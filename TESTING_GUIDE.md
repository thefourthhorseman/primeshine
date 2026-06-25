# PrimeShine Testing Guide

## Overview

This guide covers comprehensive non-regression testing strategies for the PrimeShine cleaning service platform. Our testing approach follows the **Test Pyramid** methodology to ensure robust, maintainable, and fast test suites.

## Testing Strategy

### 1. Test Pyramid

```
    /\
   /  \     E2E Tests (5-10% of tests)
  /____\     - Critical user journeys
 /      \    - Cross-browser testing
/________\   - Business value validation
Unit Tests (70-80% of tests)
- Fast execution (< 100ms per test)
- High coverage
- Isolated testing

Integration Tests (10-20% of tests)
- API endpoint testing
- Database interactions
- Component integration
```

### 2. Test Categories

#### Unit Tests
- **Purpose**: Test individual functions, methods, and components in isolation
- **Tools**: Jest, React Testing Library
- **Coverage**: 70-80% of codebase
- **Speed**: < 100ms per test

#### Integration Tests
- **Purpose**: Test interactions between components and services
- **Tools**: Jest + Supertest (backend), React Testing Library (frontend)
- **Coverage**: API endpoints, database operations
- **Speed**: 1-5 seconds per test

#### End-to-End Tests
- **Purpose**: Test complete user journeys
- **Tools**: Cypress, Playwright
- **Coverage**: Critical business flows
- **Speed**: 10-60 seconds per test

## Backend Testing

### Directory Structure
```
primeshine-back/
├── tests/
│   ├── setup.js                 # Global test configuration
│   ├── unit/                    # Unit tests
│   │   ├── controllers/
│   │   ├── services/
│   │   └── utils/
│   ├── integration/             # Integration tests
│   │   ├── api/
│   │   ├── database/
│   │   └── auth/
│   └── fixtures/                # Test data
└── jest.config.js              # Jest configuration
```

### Running Tests

```bash
# Run all tests
npm test

# Run tests in watch mode
npm run test:watch

# Run with coverage
npm run test:coverage

# Run only unit tests
npm run test:unit

# Run only integration tests
npm run test:integration

# Run tests for CI/CD
npm run test:ci
```

### Unit Testing Best Practices

#### Controller Testing
```javascript
// Example: authController.test.js
describe('AuthController', () => {
  it('should register a new user successfully', async () => {
    // Arrange
    const userData = { email: 'test@example.com', password: 'password123' };
    
    // Act
    const response = await request(app)
      .post('/api/auth/register')
      .send(userData);
    
    // Assert
    expect(response.status).toBe(201);
    expect(response.body).toHaveProperty('token');
  });
});
```

#### Service Testing
```javascript
// Example: taskGenerationService.test.js
describe('TaskGenerationService', () => {
  it('should generate tasks for a 3-bedroom house', () => {
    const service = new TaskGenerationService();
    const tasks = service.generateTasks('HOUSE', 3, 2, 2000);
    
    expect(tasks).toHaveLength(expect.any(Number));
    expect(tasks[0]).toHaveProperty('name');
    expect(tasks[0]).toHaveProperty('duration');
  });
});
```

### Integration Testing Best Practices

#### API Testing
```javascript
// Example: booking.test.js
describe('Booking API', () => {
  let authToken;
  
  beforeEach(async () => {
    // Setup test user and get auth token
    authToken = await createTestUser();
  });
  
  it('should create a booking with valid data', async () => {
    const bookingData = {
      customerInfo: { firstName: 'John', lastName: 'Doe' },
      propertyDetails: { address: '123 Test St' }
    };
    
    const response = await request(app)
      .post('/api/bookings')
      .set('Authorization', `Bearer ${authToken}`)
      .send(bookingData);
    
    expect(response.status).toBe(201);
    expect(response.body).toHaveProperty('id');
  });
});
```

### Database Testing
- Use separate test database
- Clean database between tests
- Use transactions for test isolation
- Mock external services (Stripe, Google Maps)

## Frontend Testing

### Directory Structure
```
primeshine-front/
├── src/
│   ├── components/
│   │   ├── __tests__/           # Component tests
│   │   └── ComponentName.js
│   ├── contexts/
│   │   ├── __tests__/           # Context tests
│   │   └── ContextName.js
│   ├── pages/
│   │   ├── __tests__/           # Page tests
│   │   └── PageName.js
│   └── utils/
│       ├── __tests__/           # Utility tests
│       └── utilityName.js
```

### Running Tests

```bash
# Run all tests
npm test

# Run tests in watch mode
npm run test:watch

# Run with coverage
npm run test:coverage

# Run tests for CI/CD
npm run test:ci
```

### Component Testing Best Practices

#### Component Testing
```javascript
// Example: FloatingActionButton.test.js
describe('FloatingActionButton', () => {
  it('renders with text and icon', () => {
    render(<FloatingActionButton text="Book Now" icon="Add" />);
    
    expect(screen.getByText('Book Now')).toBeInTheDocument();
    expect(screen.getByTestId('AddIcon')).toBeInTheDocument();
  });
  
  it('calls onClick when clicked', () => {
    const mockOnClick = jest.fn();
    render(<FloatingActionButton onClick={mockOnClick} />);
    
    fireEvent.click(screen.getByRole('button'));
    expect(mockOnClick).toHaveBeenCalledTimes(1);
  });
});
```

#### Context Testing
```javascript
// Example: BookingContext.test.js
describe('BookingContext', () => {
  it('provides initial booking state', () => {
    render(
      <BookingProvider>
        <TestComponent />
      </BookingProvider>
    );
    
    expect(screen.getByTestId('current-step')).toHaveTextContent('1');
  });
});
```

### Testing Utilities

#### Custom Render Function
```javascript
// tests/test-utils.js
import { render } from '@testing-library/react';
import { BrowserRouter } from 'react-router-dom';
import { ThemeProvider } from '@mui/material/styles';
import theme from '../src/theme';

const AllTheProviders = ({ children }) => {
  return (
    <BrowserRouter>
      <ThemeProvider theme={theme}>
        {children}
      </ThemeProvider>
    </BrowserRouter>
  );
};

const customRender = (ui, options) =>
  render(ui, { wrapper: AllTheProviders, ...options });

export * from '@testing-library/react';
export { customRender as render };
```

## Test Data Management

### Fixtures
```javascript
// tests/fixtures/users.js
export const testUsers = {
  client: {
    email: 'client@test.com',
    password: 'password123',
    firstName: 'John',
    lastName: 'Doe',
    role: 'CLIENT'
  },
  maid: {
    email: 'maid@test.com',
    password: 'password123',
    firstName: 'Jane',
    lastName: 'Smith',
    role: 'MAID'
  }
};
```

### Factory Functions
```javascript
// tests/factories/bookingFactory.js
export const createBooking = (overrides = {}) => ({
  customerInfo: {
    firstName: 'John',
    lastName: 'Doe',
    email: 'john@example.com'
  },
  propertyDetails: {
    address: '123 Test St',
    propertyType: 'HOUSE',
    bedrooms: 3
  },
  ...overrides
});
```

## Continuous Integration

### GitHub Actions Workflow
```yaml
# .github/workflows/test.yml
name: Tests
on: [push, pull_request]

jobs:
  test-backend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-node@v2
        with:
          node-version: '18'
      - run: npm ci
      - run: npm run test:ci
      - uses: codecov/codecov-action@v1

  test-frontend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-node@v2
        with:
          node-version: '18'
      - run: npm ci
      - run: npm run test:ci
      - uses: codecov/codecov-action@v1
```

## Coverage Requirements

### Backend Coverage
- **Statements**: 80%
- **Branches**: 75%
- **Functions**: 80%
- **Lines**: 80%

### Frontend Coverage
- **Statements**: 70%
- **Branches**: 65%
- **Functions**: 70%
- **Lines**: 70%

## Testing Best Practices

### 1. Test Naming
```javascript
// Good
describe('User Registration', () => {
  it('should create a new user with valid data', () => {
    // test implementation
  });
  
  it('should return error for duplicate email', () => {
    // test implementation
  });
});

// Bad
describe('register', () => {
  it('works', () => {
    // test implementation
  });
});
```

### 2. Arrange-Act-Assert Pattern
```javascript
it('should calculate booking total correctly', () => {
  // Arrange
  const basePrice = 100;
  const addOns = [{ price: 25 }, { price: 15 }];
  const taxRate = 0.08;
  
  // Act
  const total = calculateBookingTotal(basePrice, addOns, taxRate);
  
  // Assert
  expect(total).toBe(151.20);
});
```

### 3. Test Isolation
- Each test should be independent
- Clean up after each test
- Don't rely on test execution order
- Use beforeEach/afterEach hooks

### 4. Mocking Strategy
```javascript
// Mock external dependencies
jest.mock('stripe');
jest.mock('../../services/emailService');

// Mock API calls
jest.mock('axios', () => ({
  get: jest.fn(),
  post: jest.fn()
}));
```

### 5. Error Testing
```javascript
it('should handle API errors gracefully', async () => {
  // Mock API to throw error
  axios.get.mockRejectedValue(new Error('Network error'));
  
  const { getByText } = render(<BookingForm />);
  
  fireEvent.click(getByText('Submit'));
  
  await waitFor(() => {
    expect(getByText('Something went wrong')).toBeInTheDocument();
  });
});
```

## Performance Testing

### Load Testing
```javascript
// tests/performance/loadTest.js
import autocannon from 'autocannon';

const test = autocannon({
  url: 'http://localhost:3001/api/bookings',
  connections: 10,
  duration: 10,
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': 'Bearer test-token'
  },
  body: JSON.stringify(testBookingData)
});

autocannon.track(test, { renderProgressBar: true });
```

## Security Testing

### Authentication Tests
```javascript
describe('Authentication Security', () => {
  it('should reject requests without valid JWT', async () => {
    const response = await request(app)
      .get('/api/bookings')
      .expect(401);
  });
  
  it('should reject expired tokens', async () => {
    const expiredToken = generateExpiredToken();
    
    const response = await request(app)
      .get('/api/bookings')
      .set('Authorization', `Bearer ${expiredToken}`)
      .expect(401);
  });
});
```

## Monitoring and Maintenance

### Test Metrics
- Test execution time
- Coverage trends
- Flaky test detection
- Test failure analysis

### Regular Maintenance
- Update test dependencies
- Refactor test code
- Remove obsolete tests
- Review and update test data

## Resources

### Documentation
- [Jest Documentation](https://jestjs.io/docs/getting-started)
- [React Testing Library](https://testing-library.com/docs/react-testing-library/intro/)
- [Supertest Documentation](https://github.com/visionmedia/supertest)

### Tools
- **Test Runners**: Jest, Mocha
- **Assertion Libraries**: Jest, Chai
- **Mocking**: Jest, Sinon
- **Coverage**: Istanbul, Jest
- **E2E Testing**: Cypress, Playwright

### Best Practices
- Write tests first (TDD)
- Keep tests simple and focused
- Use descriptive test names
- Maintain test data consistency
- Regular test reviews and updates 