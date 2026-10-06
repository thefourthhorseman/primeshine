# Square Payments Integration — Design & Roadmap

**Date:** 2026-10-05
**Status:** Phases 0–1 (invoices) DEPLOYED to production 2026-10-05 (backend 18b8ffd, frontend 497eb15, migrations run). Next: connect the 5 Fairies Square account in production; then Phases 2–5.
**Scope:** Replace Stripe with Square as PrimeShine's card processor. Each tenant connects its own Square account (OAuth). Square covers **saved-card recurring billing** and **invoice / quote pay links**; PrimeShine keeps its own invoice emails and public pay page.

---

## 1. Decisions

### Locked (user, 2026-10-05)
| # | Decision | Consequence |
|---|---|---|
| D1 | **Square replaces Stripe** | All Stripe code, packages and env vars are removed at cutover |
| D2 | **Scenarios:** saved-card recurring billing + invoice/quote pay links | These are the two flows designed in detail (§5.2, §5.3) |
| D3 | **Each tenant connects its own Square account** via OAuth | PrimeShine registers one Square *application*; per-tenant tokens are stored encrypted (§4) |
| D4 | **PrimeShine pay page**, not Square Invoices | Square only processes the card (Web Payments SDK); invoices, emails and Odoo stay as they are |
| D5 | **Website booking checkout moves to Square** (was O1) | `PaymentStep.js` and `QuotationRequest.js` switch to `SquareCardForm`; Phase 4 is in scope (§5.4) |
| D6 | **Square plan tier and fees checked** by the user in the Square Dashboard (was O2) | §9 cost comparison stands |

### Still open (need an answer before Phase 2)
| # | Question | Recommendation |
|---|---|---|
| O3 | Does PrimeShine (the platform) take a cut of each tenant's payments? `Booking.platformFee` exists. | **No for now.** Square supports it later via `app_fee_money` + the `PAYMENTS_WRITE_ADDITIONAL_RECIPIENTS` scope, without redesign. |
| O4 | Should customers save a card **only by explicit opt-in** on the pay page, or can an admin send a "save your card" link? | **Both**: an opt-in checkbox on the pay page plus an admin-sent card-setup link (§5.2). Consent text and timestamp are stored either way. |

---

## 2. Starting point (audit 2026-10-05)

**Production has nothing to migrate.** Read-only check on the Neon DB: 0 rows in `payments`, 0 `RecurringSchedules`, 0 `clientprofiles.stripeCustomerId`, 1 tenant (5 Fairies, `stripeAccountId` null). Stripe live keys were never finished in production. So there's **no card import, no dual-run period and no historical Stripe refunds to support.**

**Stripe is hard-coded everywhere.** There's no provider abstraction:
- Every provider ID column is named `stripe*`.
- Keys are global env vars (`STRIPE_SECRET_KEY`, `REACT_APP_STRIPE_*`), baked into the frontend build.
- The backend calls the SDK directly in 4 controllers: `paymentController`, `bookingController`, `publicDocumentController` and `stripeWebhookController`.

Touchpoints to replace:

| Area | Stripe today | File(s) |
|---|---|---|
| Checkout intent | `POST /api/payments/create-intent` / `confirm` / `status/:id` | `routes/payments.js`, `controllers/paymentController.js` |
| Booking creation | Idempotency + paid status keyed on `stripePaymentIntentId` | `controllers/bookingController.js` (`createBooking`, `createBookingFromQuotation`, `createVoucherBooking`, `commitBookingDraft`) |
| Saved card | `setup_future_usage`, `findOrCreateCustomerForUser` | `services/stripeService.js` |
| Recurring | hourly cron → `chargeOffSession`, dunning ×3 → PAST_DUE | `services/recurringBillingService.js`, `server.js:85-100` |
| Invoice/quote pay | `/api/public/documents/:token/{pay-intent,confirm-payment}` | `controllers/publicDocumentController.js` |
| Refunds | `POST /api/bookings/:id/refund`, cancel + refund | `bookingController.js` (`issueRefundForBooking`) |
| Webhooks | `/api/webhooks/stripe` (raw body, before `express.json`) | `routes/stripeWebhookRoutes.js`, `controllers/stripeWebhookController.js`, `app.js:69-74` |
| Odoo | Payment memo `Stripe <id>` | `services/odooSyncService.js:419-480` |
| Frontend | `CardElement` + `confirmCardPayment` | `booking/steps/PaymentStep.js`, `QuotationRequest.js`, `documents/PublicDocumentPayForm.js`, `config/stripe.js` (+ unused `PaymentModal.js`) |

**Existing bugs this rewrite removes:**
- Code still reads `paymentIntent.charges.data`, which was removed from the Stripe API version that `stripe@18` uses. `createBookingFromQuotation` and `createVoucherBooking` can throw because of it.
- `commitBookingDraft` and `createBookingFromQuotation` mark bookings RELEASED (paid) without checking the payment with the provider.

---

## 3. Architecture

```
 Browser (customer)                     PrimeShine backend (Heroku)                 Square (per tenant)
 ───────────────────                    ──────────────────────────                  ───────────────────
 Square Web Payments SDK  ─token──▶  paymentGateway (Square impl)  ──REST──▶  Payments / Cards / Customers / Refunds
 (card iframe, PCI SAQ-A)            ├─ squareConnectionService  ──OAuth──▶  token obtain / refresh / revoke
                                     ├─ recurringBillingService (cron)
                                     └─ /api/webhooks/square  ◀──HMAC──  payment.updated, refund.updated, card.*, oauth.*
```

**Principles**
1. **The server decides the amount and charges the card.** The browser only tokenizes the card. Square's `CreatePayment` (with `autocomplete: true`) is synchronous, so there's no intent/confirm round-trip. This removes the "payment succeeded but booking failed" orphan class at its source.
2. **A thin `paymentGateway` interface** (`services/payments/gateway.js`) with one implementation (`squareGateway.js`). Controllers call `gateway.charge()`, `gateway.saveCard()`, `gateway.chargeSavedCard()`, `gateway.refund()` and never import a provider SDK. The next provider switch then touches one folder, not four controllers.
3. **Everything is per tenant.** Every gateway call takes a `tenantId` and resolves that tenant's Square connection (token + location). No global secret key.
4. **Every write carries an idempotency key.** Square requires one on payments, refunds and cards. Keys are deterministic where a retry must not double-charge (recurring, §5.3).
5. **Additive data changes** (feedback: isolate new features): new provider-neutral columns are added. `stripe*` columns are dropped only in the final cleanup migration.

---

## 4. Tenant connection (OAuth)

PrimeShine registers **one Square application** (Developer Dashboard), with a sandbox and a production version. Each tenant admin clicks **Connect Square** and approves; PrimeShine stores that seller's tokens.

**Flow (OAuth code flow, confidential client)**
1. `GET /api/admin/square/connect` (tenant admin)
   - Creates a random `state`, stored server-side with `tenantId` and a 10-min TTL.
   - Redirects to `https://connect.squareup.com/oauth2/authorize?client_id=…&scope=…&session=false&state=…`.
   - `session=false` is sent in **production only**: the sandbox supports only the default (`true`) and shows a blank page otherwise.
2. `GET /api/square/oauth/callback?code&state`
   - Validates `state`.
   - `ObtainToken` (code → access + refresh token) and `ListLocations`.
   - Stores the connection and redirects to the admin page.
3. Admin chooses the **location** that receives payments (auto-selected when there's only one).
4. `POST /api/admin/square/disconnect`: `RevokeToken`, then status → `REVOKED`.

**Token lifecycle**
- Access tokens expire after **30 days**. Code-flow refresh tokens don't expire. Square's guidance is to refresh **every 7 days or less**.
- A daily cron (`SQUARE_TOKEN_REFRESH_CRON_ENABLED`) refreshes any token older than 6 days. A refresh failure alerts `ADMIN_ALERT_EMAIL`.
- Tokens are encrypted at rest with **AES-256-GCM**, using `SQUARE_TOKEN_ENC_KEY` (Heroku config). The key never goes in the DB or the repo.

**Scopes (least privilege):**
- `MERCHANT_PROFILE_READ` (locations)
- `PAYMENTS_READ`, `PAYMENTS_WRITE`
- `CUSTOMERS_READ`, `CUSTOMERS_WRITE` (customers + cards on file)
- *Later, only if O3 = yes:* `PAYMENTS_WRITE_ADDITIONAL_RECIPIENTS`

Confirm against Square's OAuth permissions reference when registering the app.

**New table `SquareConnections`**
| Column | Notes |
|---|---|
| `id` UUID PK | |
| `tenantId` INT, unique | one active connection per tenant |
| `merchantId` | Square seller ID; routes incoming webhooks to the tenant |
| `locationId`, `locationName` | where payments land |
| `accessTokenEnc`, `refreshTokenEnc` | AES-256-GCM ciphertext (iv + tag included) |
| `accessTokenExpiresAt`, `lastRefreshedAt` | |
| `scopes` TEXT[] | |
| `status` ENUM(ACTIVE, REVOKED, ERROR) | |
| `environment` ENUM(sandbox, production) | |
| `connectedByUserId`, timestamps | audit |

**Admin UI:** a "Square payments" card in *Integrations & Tools*. It shows Connect / Disconnect, the connected business name, the location picker, and the last token refresh. A banner warns when the status isn't ACTIVE, because pay pages and recurring charges are disabled until it's fixed.

---

## 5. Payment flows

### 5.1 Invoice / quote pay link (D2)
1. The customer opens `/documents/:token` (existing `DocumentView`) and clicks **Pay online**.
2. `GET /api/public/documents/:token/pay-config` returns:
   - `{ applicationId, locationId, environment, amountCents, currency, canSaveCard }`
   - `applicationId` is the platform's app ID; `locationId` is the tenant's. Nothing is baked into the build anymore.
3. **Card form.** The `SquareCardForm` component loads the SDK and renders the card iframe. It then calls `card.tokenize()` and `payments.verifyBuyer()`:
   - intent `CHARGE`, or `CHARGE_AND_STORE` when the customer ticks **"Save this card for future cleanings"**.
4. `POST /api/public/documents/:token/pay` with `{ sourceId, verificationToken, idempotencyKey, saveCard, consentText }`. The server:
   1. Recomputes the amount from the document (the existing logic in `createPayIntent`; never from the client).
   2. Calls `CreatePayment`:
      - `source_id`, `amount_money`, `autocomplete: true`, `location_id`
      - `reference_id` = job/booking ID
      - `note` = invoice number
      - `customer_id` when saving the card
      - `idempotency_key` from the client. It's generated once per pay-page load, so a double click can't charge twice.
   3. On `COMPLETED`, marks the job/booking paid, writes a `payments` row (provider fields, card brand/last4, Square `receipt_url`), and sends the existing receipt email.
   4. If `saveCard`, finds or creates the Square customer, calls `CreateCard` with `source_id = payment.id`, and stores the card plus the consent (§5.2).
   5. On decline, returns Square's error code mapped to the existing retry messages.

Quotes keep today's path: accept → pay → `create-from-quotation`. That endpoint now verifies the Square payment server-side instead of trusting the client, which fixes one of the §2 bugs.

### 5.2 Saving a card (recurring prerequisite)
There are two entry points, both producing a **card on file** in the tenant's Square account:
- **At payment:** the opt-in checkbox from §5.1 (`CHARGE_AND_STORE`).
- **Card-setup link (no charge):**
  - An admin clicks **"Send card-setup link"** on a customer or recurring schedule. The customer gets an email/SMS link to `/payment-method/:token`.
  - The page uses intent `STORE`, then calls `POST /api/public/payment-method/:token`, which runs `CreateCard`.
  - It reuses the existing public-token pattern from `/api/public/documents`.

**Customer mapping:** `ClientProfile` is already per tenant. A new column, `squareCustomerId`, is set on first save:
- Look up the Square customer by `reference_id` = clientProfile ID, then by email.
- Otherwise call `CreateCustomer`.

**Consent** (card-network rules for merchant-initiated charges): store `cardConsentText`, `cardConsentAt` and `cardConsentIp` with the card. The recurring schedule can only be activated when consent exists.

### 5.3 Recurring billing (D2)
Keep `recurringBillingService`, its cron, charge timing (BEFORE_24H / AFTER_24H), dunning ×3 → PAST_DUE and the admin panel. Only the charge call and the stored references change:
- `RecurringSchedules`:
  - New columns: `provider` (`SQUARE`), `providerCustomerId`, `providerCardId`.
  - `stripeCustomerId` and `stripePaymentMethodId` become nullable (they're NOT NULL today).
- Each charge:
  - `CreatePayment` with `source_id = providerCardId`, `customer_id`, `autocomplete: true`, `reference_id` = occurrence booking ID.
  - **`idempotency_key = rs-{scheduleId}-{occurrenceDate}`**: deterministic, so a cron re-run, overlapping dynos or a crash between charge and DB write can never double-charge one occurrence. Today's Stripe code has no such guard.
- Before charging, check that the tenant connection is ACTIVE. If it isn't, skip without counting a dunning failure, and alert the admin.
- `card.disabled` / expired card: the webhook marks the schedule `NEEDS_CARD` and automatically sends the card-setup link (§5.2).

### 5.4 Website booking checkout (D5)
- `PaymentStep.js` swaps `CardElement` for `SquareCardForm`.
- `POST /api/bookings/create-booking` receives `{ sourceId, verificationToken, idempotencyKey, saveCard }` and runs **create pending booking → charge with `reference_id = bookingId` → finalize** in one request.
- `POST /api/payments/create-intent`, `/confirm` and `/status/:id` are retired.
- `QuotationRequest.js` gets the same swap.

### 5.5 Refunds
- `gateway.refund(tenantId, paymentId, amountCents, reason)` calls `RefundPayment` with `idempotency_key = rf-{paymentRowId}-{amountCents}`. It's used by the existing refund endpoint and by cancel + refund.
- Partial refunds become possible (Square supports them). The UI keeps full refunds for v1.
- **Refunds issued from the Square Dashboard** reach PrimeShine through the `refund.updated` webhook and are mirrored (§5.6).

### 5.6 Webhooks
- **Endpoint:** `POST /api/webhooks/square`. It's mounted with `express.raw()` before `express.json()`, where the Stripe route is today.
- **One subscription at the application level** receives events for every connected seller. Each event carries `merchant_id`, which maps to `SquareConnections` → tenant.

**Signature check**
- Header `x-square-hmacsha256-signature`, compared to base64 HMAC-SHA256(**notification URL + raw body**) keyed with `SQUARE_WEBHOOK_SIGNATURE_KEY`.
- Compare in constant time (`crypto.timingSafeEqual`).
- The URL must exactly match the one registered in the dashboard (scheme, host, path, trailing slash). Configure it as `SQUARE_WEBHOOK_URL` rather than rebuilding it from the request behind Heroku's router.

**Deduplication:** new table `SquareWebhookEvents(eventId PK, type, merchantId, receivedAt, processedAt, error)`. Duplicates return 200 immediately; Square retries on non-2xx.

| Event | Action |
|---|---|
| `payment.updated` | Reconcile the `payments` row by Square payment ID (COMPLETED / FAILED / CANCELED). Alert on an orphan COMPLETED payment after a 5-minute grace period (same logic as today's Stripe handler). |
| `refund.created`, `refund.updated` | On COMPLETED, mark the payment and its job/booking REFUNDED. This covers Dashboard-initiated refunds. |
| `card.disabled`, `card.automatically_updated` | Disabled: schedule → NEEDS_CARD and send the setup link. Auto-updated: refresh brand/last4/expiry. |
| `dispute.created` | Email the admin with job, customer and amount. |
| `oauth.authorization.revoked` | Connection → REVOKED, pause that tenant's recurring charges, email the admin. |

### 5.7 Receipts & Odoo
- **Receipts:** Square returns `receipt_url` on every payment. It's stored in `payments.receiptUrl`, so the existing receipt links in `BookingDetails` and `CustomerOverviewPanel` keep working.
- **Odoo:** `syncPayments` reads the new `providerPaymentId`, and the memo becomes `Square <id>`. Use a separate **"Square" payment journal** in Odoo, so bank deposits (Square payouts) reconcile against it.

---

## 6. Frontend

**New `components/payments/SquareCardForm.js`** (one component, reused everywhere)
- Loads `https://web.squarecdn.com/v1/square.js`, or `https://sandbox.web.squarecdn.com/v1/square.js` in sandbox, chosen from `pay-config.environment`.
- Renders the card iframe, exposes `tokenize()` and `verifyBuyer()`, and follows the existing loading-button and error patterns.
- Props: `config`, `intent` (`CHARGE` / `STORE` / `CHARGE_AND_STORE`), `amountCents`, `onToken`.

**Where it's used:**
- `PublicDocumentPayForm`
- the new `PaymentMethodSetupPage` (`/payment-method/:token`)
- `PaymentStep` and `QuotationRequest` (D5)

**Removed:**
- `@stripe/react-stripe-js`, `@stripe/stripe-js`
- `config/stripe.js` and `REACT_APP_STRIPE_*`
- the unused `PaymentModal.js`

**CSP:** if a Content-Security-Policy is added later, allow `web.squarecdn.com`, `*.squarecdn.com` and `pci-connect.squareup.com` (iframe/script/connect).

**Labels:** the admin "Stripe payments" label in `OdooSyncDialog`, and `hasSavedCard` in the customer API, which becomes `!!providerCardId`.

---

## 7. Data model changes (additive)

| Table | Add | Change later (cleanup migration) |
|---|---|---|
| `SquareConnections` | new (§4) | — |
| `SquareWebhookEvents` | new (§5.6) | — |
| `payments` | `provider` ('SQUARE'), `providerPaymentId`, `providerCustomerId`, `providerRefundId` | drop `stripePaymentIntentId`, `stripeCustomerId` |
| `clientprofiles` | `squareCustomerId` | drop `stripeCustomerId` |
| `RecurringSchedules` | `provider`, `providerCustomerId`, `providerCardId`, `cardBrand`, `cardLast4`, `cardExpMonth/Year`, `cardConsentText/At/Ip`; status `NEEDS_CARD` | drop the `stripe*` columns |
| `Bookings`, `Jobs` | `providerPaymentId` | drop `stripePaymentIntentId` |
| `TenantProfiles` | — | drop the unused `stripeAccountId` |

Amount units stay as they are: cents on `payments`, `Jobs` and `RecurringSchedules`; dollars (DECIMAL) on `Bookings` and `Quotations`. All conversions happen inside the gateway, which converts dollars to cents in exactly one place.

Migrations run **locally first** and go to production only with approval (local-first and prod-write policies).

---

## 8. Configuration

| Env var (Heroku backend) | Purpose |
|---|---|
| `SQUARE_ENVIRONMENT` | `sandbox` \| `production` |
| `SQUARE_APPLICATION_ID`, `SQUARE_APPLICATION_SECRET` | platform OAuth app (one per environment) |
| `SQUARE_OAUTH_REDIRECT_URL` | e.g. `https://primeshine-back-…herokuapp.com/api/square/oauth/callback` |
| `SQUARE_WEBHOOK_SIGNATURE_KEY`, `SQUARE_WEBHOOK_URL` | webhook verification (§5.6) |
| `SQUARE_TOKEN_ENC_KEY` | 32-byte key for token encryption |
| `SQUARE_TOKEN_REFRESH_CRON_ENABLED` | daily token refresh |
| `SQUARE_SANDBOX_ACCESS_TOKEN` | **local dev only**: the sandbox test seller's token, for scripts/tests without OAuth; never set in production |
| `ADMIN_ALERT_EMAIL` | already planned for Stripe; reused |
| *removed* | `STRIPE_SECRET_KEY`, `STRIPE_*_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET`; the frontend `REACT_APP_STRIPE_*` |

The frontend needs **no Square env var**: `applicationId`, `locationId` and environment come from `pay-config` at runtime.

The `process.exit(1)` when the Stripe key is missing (`config/stripe.js`) goes away. A tenant without a Square connection just can't take card payments: pay buttons show "Online payment not available — contact us".

---

## 9. Costs

These are approximate US rates from public sources (2026); confirm in the Square Dashboard.

| Charge type | Square | Stripe (reference) |
|---|---|---|
| Customer pays online (pay link, checkout) | ≈ 3.3% + 30¢ on Free; ≈ 2.9% + 30¢ on Plus/Premium | 2.9% + 30¢ |
| Saved card charged by PrimeShine (recurring) | ≈ **3.5% + 15¢** on all plans | 2.9% + 30¢ |

On a typical **$220 recurring cleaning**, Square charges ≈ $7.85 against Stripe's ≈ $6.68: about **+$1.17 per recurring charge**. That's worth knowing before moving many customers to auto-pay. Pay-link payments cost the same as Stripe on Plus.

---

## 10. Roadmap

### Phase 0 — Foundation (≈ 1.5 days)
- Register the Square application (sandbox + production) and add the env vars.
- Add the `square` Node SDK and `services/payments/{gateway,squareGateway}.js`.
- `SquareConnections` migration and model, token encryption helper, OAuth connect/callback/disconnect, refresh cron.
- Admin "Square payments" card.
- **Exit:** the 5 Fairies sandbox seller connects, a location is chosen, and a token refresh is proven. ✅ Met 2026-10-05 (disconnect not yet exercised against Square, to keep the test connection for Phase 1).
- **Built 2026-10-05:** `SquareConnections` migration (run locally) + model, `utils/secretBox.js`, `services/payments/squareConnectionService.js`, `controllers/squareConnectionController.js`, `routes/squareRoutes.js` (`/api/admin/square/*`, `/api/square/oauth/callback`), refresh cron in `server.js`, `SquarePaymentsDialog` in Integrations & Tools. The `gateway.js` interface moves to Phase 1, where its first methods land.

### Phase 1 — Invoice/quote pay links (≈ 2 days) ← first customer-visible win
- `SquareCardForm`, plus `pay-config` and `pay` endpoints for job, booking and quote documents.
- `payments` provider columns, receipt email, Odoo memo/journal.
- **Exit:** pay a job invoice and a quote in sandbox with test cards (success, decline, 3-DS challenge); paid status, receipt email and `receipt_url` are correct; a double-click doesn't double-charge.
- **Built 2026-10-05 (invoices only; quotes keep the old path):** `services/payments/gateway.js`, `controllers/publicDocumentPaymentController.js` (`GET /:token/pay-config`, `POST /:token/pay`, row lock, charged-but-unrecorded admin alert), migration `20261005100000` (payments.provider/providerPaymentId + **fixes payments.bookingId NOT NULL, which 20260924000000 failed to drop — also broken in production**), `SquareInvoicePayForm` on the invoice page, "View & Pay Invoice" email. Sandbox server-side test passed: decline 402, pay 200 → PAID + Payment row + receipt email, repeat 409.

### Phase 2 — Saved cards & recurring (≈ 2.5 days)
- Customer mapping, `CHARGE_AND_STORE` checkbox, card-setup link page, consent capture.
- `RecurringSchedules` columns, `chargeSavedCard` with the deterministic idempotency key, `NEEDS_CARD`.
- **Exit:** schedule → cron charge → occurrence booking + payment + email. Re-running the cron for the same occurrence returns the same payment (no double charge). Three declines lead to PAST_DUE.

### Phase 3 — Webhooks & refunds (≈ 1.5 days)
- `/api/webhooks/square` with signature check and event dedupe; all events in §5.6.
- Gateway refunds wired to the refund and cancel endpoints.
- **Exit:** a refund from the Square sandbox Dashboard is mirrored; revoking the app pauses recurring; a forged signature is rejected (401).

### Phase 4 — Checkout swap (≈ 1 day)
- `PaymentStep` and `QuotationRequest` use `SquareCardForm`; the single-request charge + booking (§5.4).

### Phase 5 — Stripe removal & go-live (≈ 1 day)
- Delete the Stripe routes, controllers, services, webhook and packages; run the cleanup migration.
- Production Square app, live webhook subscription, connect the real 5 Fairies account.
- Prod migrations (with approval), then one live $1 payment refunded end to end.

**Total ≈ 8.5 dev days.** There's no data migration: production has no Stripe payments or saved cards.

---

## 11. Testing

- **Sandbox:** sandbox seller account plus Square's sandbox test cards (approve, decline, CVV fail, 3-DS / verification challenge). The OAuth sandbox authorize URL is `https://connect.squareupsandbox.com`.
- **Local webhooks and OAuth redirect:** need a public HTTPS URL to the local backend (a tunnel such as ngrok or cloudflared). Set `SQUARE_WEBHOOK_URL` and `SQUARE_OAUTH_REDIRECT_URL` to it.
- **Unit:** amount computation (dollars ↔ cents), signature verification (including the trailing-slash case), idempotency-key generation, token encryption round trip, webhook → tenant routing by `merchant_id`.
- **Integration:** recurring double-run and no-double-charge; revoked connection → graceful "payments unavailable"; client-supplied amount ignored.
- **Coverage:** keep the 70% project threshold on the new `services/payments/` folder.

---

## 12. Security checklist

- [ ] Card data never touches PrimeShine (Square iframe tokenization keeps PCI scope at SAQ-A).
- [ ] Amounts always computed server-side; the client amount is ignored.
- [ ] OAuth `state` validated; tokens AES-256-GCM encrypted; `SQUARE_TOKEN_ENC_KEY` only in Heroku config.
- [ ] Webhook HMAC verified in constant time against the exact registered URL; events deduped.
- [ ] Idempotency key on every payment, card and refund write; deterministic for recurring.
- [ ] Admin-only access to connect/disconnect, refunds and card-setup links; public pay endpoints scoped to their document token.
- [ ] Recurring charges only with stored consent; customers can remove their saved card (via card-setup link or admin).
