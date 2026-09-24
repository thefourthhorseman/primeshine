# Odoo Community Integration — Analysis, Plan & Functional Roadmap

**Date:** 2026-07-10
**Status:** IMPLEMENTED locally (Phases 0–3 + 5) on 2026-07-10 — all syncs tested and idempotent against local Odoo 18 Docker stack. Not yet committed/pushed; production deployment (Hetzner VPS + Heroku env vars) pending.
**Scope:** Integrate Odoo Community Edition (open-source, LGPL, free) with the PrimeShine / 5 Fairies platform to fill the financial back-office gaps.

---

## 1. Why Odoo, and why now

A full audit of both repos (frontend pages/components/contexts, backend models/routes/services) shows PrimeShine is a **complete field-operations platform** but is **missing the entire financial back office**:

### What PrimeShine already does well (do NOT duplicate in Odoo)
| Capability | Where it lives |
|---|---|
| Job scheduling, calendar, timezone handling | `Job`, `ServiceSchedulePage`, FullCalendar |
| Maid portal (kanban, available jobs, earnings, en-route/arrived) | `MaidPortalPage`, `JobAssignment` |
| Customer portal & booking flow | `Booking`, `UnifiedBookingFlow`, customer pages |
| Quote generation + public signed links | `Quotation`, `documentService.js`, `/quote/:token` |
| HCP + Smoobu job/customer import | `hcpJobImportService`, `smoobuJobImportService` |
| Stripe payment intents on bookings | `paymentController.js`, `Payment` model |
| Teams, service areas, auto-assignment, maps | `Team`, `ServiceArea`, `autoAssignmentService` |
| Analytics dashboard (operational metrics) | `Analytics.js`, `dashboardController` |

### What PrimeShine is missing (the gaps Odoo fills)
| Gap | Today's state | Odoo Community module that fills it |
|---|---|---|
| **Customer invoicing & AR** | PDF invoice + email exists; no ledger, no aging, no partial payments, no follow-ups | **Invoicing** (`account`) — invoices, credit notes, AR aging, payment registration |
| **Maid payouts / AP** | `JobAssignment.paymentAmount` with a paid flag; manual tracking in `PaymentManagement.js`; no audit trail, no 1099 basis | **Vendor Bills** (`account`) + **Purchase** — each maid = vendor, each payout = vendor bill; annual totals per maid = 1099 basis |
| **General ledger & financial statements** | Nothing — analytics only, not GAAP | Community accounting engine (journal entries are created automatically by invoices/bills) + **OCA `account_financial_report`** for P&L / Balance Sheet / GL views |
| **Job profitability** | `Job.totalAmount` only; costs not tracked | **Analytic accounting** (`analytic`) — tag every invoice line & vendor bill with the job → profitability = job sale price − maid payouts, per job/customer |
| **Email marketing** | Placeholder button in dashboard | **Email Marketing** (`mass_mailing`) — campaigns to customer/lead lists |
| **CRM pipeline / forecasting** | Questionnaire leads list, quotation statuses | **CRM** (`crm`) — optional; PrimeShine's lead flow already works, so this is a later nice-to-have |

### Out of scope (decided 2026-07-10)
- **Expense tracking** — not needed.
- **Supplies inventory & purchasing** — not needed.
- **Job profitability is strictly:** job sale price (customer invoice) − maid payout amounts (vendor bills). No other cost allocation.

### Honest limitations — what Odoo Community does NOT give us (Enterprise-only)
- **Payroll** — not in Community. Not a problem: **all maids are confirmed 1099 contractors** (decided 2026-07-10), so vendor bills are the correct model, not payroll.
- **Field Service app** — Enterprise. Irrelevant: PrimeShine *is* our field-service app.
- **Full Accounting app UI** (bank sync, fancy reports) — Enterprise. Mitigated by OCA community modules; bank statements can be imported as CSV.
- **Subscriptions (recurring billing)** — Enterprise. PrimeShine already owns recurrence (`Booking.isRecurring`); we just generate an invoice per completed job.
- **No REST API** — Odoo exposes **JSON-RPC/XML-RPC**, which works fine from Node via axios (same as our other integrations).

---

## 2. Architecture principle: system-of-record split

**PrimeShine stays the operational system of record. Odoo becomes the financial system of record. Sync is one-directional per object type, additive, and isolated** — zero changes to any working production flow (per our isolation policy).

```
┌────────────────────────┐         JSON-RPC (axios)        ┌──────────────────────┐
│  PrimeShine (existing) │ ──────────────────────────────► │  Odoo Community 18   │
│  operations SoR        │                                 │  financial SoR       │
│                        │  customers  → res.partner       │                      │
│  Jobs / Bookings       │  services   → product.product   │  Invoicing (AR)      │
│  Scheduling / Portals  │  done jobs  → account.move (inv)│  Vendor bills (AP)   │
│  Stripe / HCP / Smoobu │  payments   → payment registr.  │  GL + OCA reports    │
│                        │  maid dues  → vendor bills      │  Email marketing     │
│                        │ ◄──────────────────────────────  │                      │
│                        │   paid-status read-back (poll)  │                      │
└────────────────────────┘                                 └──────────────────────┘
```

Key design rules:
1. **Push, don't share a database.** New `odooSyncService.js` follows the exact HCP/Smoobu pattern (service → controller → routes → dashboard import-manager card).
2. **Idempotent sync with external-ID mapping.** New table `odoo_sync_map` (`localModel`, `localId`, `odooModel`, `odooId`, `lastSyncedAt`, `syncHash`) — never create duplicates, re-sync only on change.
3. **Nothing in the customer/maid-facing flows calls Odoo.** Sync runs from admin actions + cron; if Odoo is down, operations are unaffected.
4. **Money enters via Stripe exactly as today.** Odoo only *records* payments (payment registration on invoices), it never charges anyone.

### Deployment (decided 2026-07-10)
- **PrimeShine stays on Heroku unchanged.** Odoo is NOT deployed on Heroku — dynos have an ephemeral filesystem (Odoo's attachment filestore and sessions would be wiped on every daily dyno restart), Odoo needs 1–2 GB RAM (Performance-dyno pricing), and the image exceeds classic slug limits. Fighting all of that costs more than proper hosting.
- **Odoo Community 18 runs on a small dedicated VPS (~$6–12/mo, e.g. Hetzner / DigitalOcean / Lightsail, 2 GB RAM)** via Docker Compose: official `odoo:18` image + `postgres:16` container + Caddy/nginx for TLS, with persistent volumes for the filestore and DB, and a nightly `pg_dump` backup to S3.
- The Heroku backend reaches Odoo over HTTPS JSON-RPC; config is 4 Heroku env vars (`ODOO_URL`, `ODOO_DB`, `ODOO_USER`, `ODOO_API_KEY`) — same pattern as HCP/Smoobu keys.
- Security: TLS, strong master password, API-key auth for the sync user; optionally IP-restrict access.
- Apps to install: Invoicing, Contacts, Email Marketing + OCA `account_financial_report`.

---

## 3. Data mapping

| PrimeShine (source) | Odoo (target) | Notes |
|---|---|---|
| `User`+`ClientProfile` (customers) | `res.partner` (customer_rank=1) | name, email, phone, address; tag with tenant |
| `User`+`MaidProfile` (maids) | `res.partner` (supplier_rank=1) | vendors for AP; store payout method in notes/custom field |
| `ServiceCatalog` entries | `product.product` (type=service) | price, income account default |
| Completed `Job` / `Booking` | `account.move` (out_invoice) | 1 line per service + add-ons; analytic tag = job ref (`hcpJobNumber`/OrderID); date = completion date |
| `Payment` (Stripe) records | `account.payment` + reconcile | marks invoice paid; keeps receipt URL in memo |
| `JobAssignment` (paymentAmount, per maid) | `account.move` (in_invoice, vendor bill) | batched weekly per maid; analytic tag = job → cost side of job profitability (sale price − payouts) |

---

## 4. Functional roadmap

### Phase 0 — Foundation (≈ 1 day)
1. Docker Compose stack for Odoo 18 Community + Postgres; install apps; configure company (5 Fairies, address, logo, fiscal year), minimal chart of accounts.
2. Backend: `src/services/odooClient.js` — thin JSON-RPC client (login, `execute_kw`, retry/backoff), env-driven (`ODOO_URL`, `ODOO_DB`, `ODOO_USER`, `ODOO_API_KEY`).
3. Migration for `odoo_sync_map` table. **Local first** — no production touch until Phase 1 is validated.

**Deliverable:** Odoo reachable, authenticated round-trip test endpoint `/api/admin/odoo/status`.

### Phase 1 — Customers & catalog sync (≈ 1–2 days)
1. `odooSyncService.syncCustomers()` — push all `ClientProfile` users → `res.partner`; idempotent via sync map.
2. `syncMaidsAsVendors()` — push maids → vendor partners.
3. `syncServiceCatalog()` — services → products.
4. Dashboard card "Odoo Sync" in **Integrations & Tools** (same UX as HCP Import Manager): preview / run / status counts.

**Deliverable:** All contacts & products visible in Odoo; re-running sync creates zero duplicates.

### Phase 2 — Customer invoicing / AR (≈ 2–3 days) ← **the core win**
1. `syncInvoices()` — every `COMPLETED` job (LOCAL, HCP, Smoobu) → draft or posted customer invoice in Odoo, with analytic account per job.
2. `syncPayments()` — existing Stripe `Payment` records → registered payments reconciled against their invoices; unpaid invoices surface in Odoo AR aging automatically.
3. Cron (env-gated, same pattern as auto-assignment cron): nightly incremental sync of newly completed jobs.
4. Read-back: small poller updates a `odooInvoiceStatus` badge on Job History page (paid/open) — display-only, additive.

**Deliverable:** Real AR aging, revenue by month/customer/service in Odoo pivot views; every completed job is an accounted invoice.

### Phase 3 — Maid payouts / AP (≈ 2 days)
1. `syncMaidPayables()` — unpaid `JobAssignment` amounts → weekly vendor bill per maid (one line per job, analytic-tagged).
2. When admin marks a payout paid in PrimeShine (`PaymentManagement.js` flow unchanged), register the payment on the Odoo bill; or vice-versa via read-back — **one direction chosen at build time to avoid conflict logic** (recommend: PrimeShine remains where "paid" is clicked, Odoo mirrors).
3. Year-end: Odoo vendor-bill totals per maid = 1099-NEC amounts.

**Deliverable:** Full AP trail per maid; job profitability = invoice (revenue) − vendor bills (labor) per analytic account.

### Phase 4 — Back-office adoption, no code (ongoing)
Used directly in the Odoo UI by the admin (Kathleen): OCA P&L / Balance Sheet / GL reports, AR aging follow-ups, CSV bank statement import for reconciliation. (Expenses and Inventory/Purchase deliberately excluded — out of scope.)

**Deliverable:** Monthly P&L without a bookkeeper spreadsheet.

### Phase 5 — Optional / later
- **Email Marketing:** push questionnaire leads + customers into Odoo mailing lists; replaces the placeholder dashboard button.
- **CRM pipeline:** push `QuestionnaireProgress` leads → `crm.lead` for pipeline/forecast views.
- **Webhooks:** Odoo automated actions → `POST /api/webhooks/odoo` for real-time paid-status instead of polling.

**Total estimated effort for Phases 0–3: ~7–9 dev days.**

---

## 5. Decisions

**Resolved 2026-07-10:**
1. ✅ **All maids are 1099 contractors** → vendor-bill payout model.
2. ✅ **Hosting**: PrimeShine stays on Heroku; Odoo self-hosted on a small VPS via Docker Compose (see §2 Deployment).
3. ✅ **Out of scope**: expense tracking, supplies inventory & purchasing. Job profitability = sale price − maid payouts only.

4. ✅ **Historical backfill**: full history — sync all completed jobs, no cutoff. (Implemented with an optional `ODOO_BACKFILL_START_DATE` env var so a cutoff can be applied later without code changes.)
5. ✅ **Invoice posting**: all synced invoices land as **draft-for-review** in Odoo; admin posts them from the Odoo UI.
6. ✅ **VPS provider**: **Hetzner** (2 GB instance, e.g. CX22).

## 6. Verification approach
- Phase 0: `/api/admin/odoo/status` returns Odoo version + authenticated uid.
- Each sync: preview (dry-run) endpoint first — same pattern as HCP `preview-import`; run against **local Odoo + local DB** before any production data.
- Reconciliation check script: sum of Odoo invoices for a month == sum of `Job.totalAmount` completed that month; flag mismatches.
- Idempotency test: run every sync twice, assert zero new records on second pass.
