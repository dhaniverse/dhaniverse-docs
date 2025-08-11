# Testing Documentation

## Overview

This document outlines the comprehensive testing strategy for the Dhaniverse project, covering unit testing, integration testing, and manual testing procedures across all components including the React frontend, Rust ICP canister, Deno game servers, and TypeScript utilities.

## Testing Philosophy

### Testing Pyramid

The project follows the testing pyramid approach:

```
    /\
   /  \    E2E Tests (Few)
  /____\   
 /      \   Integration Tests (Some)
/__________\ Unit Tests (Many)
```

- **Unit Tests (70%)**: Fast, isolated tests for individual functions and components
- **Integration Tests (20%)**: Tests for component interactions and API endpoints
- **End-to-End Tests (10%)**: Full user workflow tests across the entire system

### Testing Principles

- **Test-Driven Development (TDD)**: Write tests before implementation when possible
- **Behavior-Driven Development (BDD)**: Focus on testing behavior, not implementation
- **Fast Feedback**: Tests should run quickly to enable rapid development
- **Reliable**: Tests should be deterministic and not flaky
- **Maintainable**: Tests should be easy to understand and modify

## Unit Testing

### Frontend (React/TypeScript)

#### Testing Framework

- **Test Runner**: Vitest
- **Testing Library**: React Testing Library
- **Mocking**: Vitest mocks and MSW (Mock Service Worker)
- **Coverage**: c8 (built into Vitest)

#### Setup

```bash
# Install testing dependencies (already included in package.json)
npm install

# Run tests
npm run test

# Run tests with coverage
npm run test:coverage

# Run tests in watch mode
npm run test:watch
```

#### Test Structure

```typescript
// src/components/__tests__/WalletConnector.test.tsx
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { WalletConnector } from '../WalletConnector';
import { Web3AuthService } from '../../services/web3/Web3AuthService';

// Mock external dependencies
vi.mock('../../services/web3/Web3AuthService');

describe('WalletConnector', () => {
  beforeEach(() => {
    vi.clearAllMocks();
  });

  it('should render connect button when not connected', () => {
    render(<WalletConnector />);
    expect(screen.getByText('Connect Wallet')).toBeInTheDocument();
  });

  it('should handle wallet connection', async () => {
    const mockConnect = vi.fn().mockResolvedValue({ address: '0x123' });
    vi.mocked(Web3AuthService.prototype.connect).mockImplementation(mockConnect);

    render(<WalletConnector />);
    fireEvent.click(screen.getByText('Connect Wallet'));

    await waitFor(() => {
      expect(mockConnect).toHaveBeenCalled();
    });
  });
});
```

#### Testing Guidelines

- **Component Testing**: Test component behavior, not implementation details
- **Props Testing**: Test all prop variations and edge cases
- **Event Handling**: Test user interactions and event callbacks
- **State Management**: Test state changes and side effects
- **Error Handling**: Test error states and error boundaries

#### Mock Strategies

```typescript
// Mock external services
vi.mock('../../services/ICPActorService', () => ({
  ICPActorService: {
    getInstance: vi.fn(() => ({
      getBalance: vi.fn().mockResolvedValue(100),
      transfer: vi.fn().mockResolvedValue({ success: true })
    }))
  }
}));

// Mock React hooks
vi.mock('react-router-dom', () => ({
  useNavigate: vi.fn(() => vi.fn()),
  useLocation: vi.fn(() => ({ pathname: '/test' }))
}));
```

### Backend Services (Deno/TypeScript)

#### Testing Framework

- **Test Runner**: Deno built-in test runner
- **Assertions**: Deno standard library assertions
- **Mocking**: Deno standard library mocking utilities

#### Setup

```bash
# Run game server tests
cd server/game
deno test --allow-net --allow-env --allow-read

# Run WebSocket server tests
cd server/ws
deno test --allow-net --allow-env

# Run with coverage
deno test --coverage=coverage --allow-all
deno coverage coverage
```

#### Test Structure

```typescript
// server/game/src/controllers/__tests__/PlayerController.test.ts
import { assertEquals, assertRejects } from "@std/assert";
import { describe, it, beforeEach, afterEach } from "@std/testing/bdd";
import { PlayerController } from "../PlayerController.ts";
import { MockDatabase } from "../../__mocks__/MockDatabase.ts";

describe("PlayerController", () => {
  let controller: PlayerController;
  let mockDb: MockDatabase;

  beforeEach(() => {
    mockDb = new MockDatabase();
    controller = new PlayerController(mockDb);
  });

  afterEach(() => {
    mockDb.cleanup();
  });

  it("should create a new player", async () => {
    const playerData = { username: "testuser", email: "test@example.com" };
    const result = await controller.createPlayer(playerData);
    
    assertEquals(result.success, true);
    assertEquals(result.player.username, "testuser");
  });

  it("should handle duplicate username error", async () => {
    const playerData = { username: "existing", email: "test@example.com" };
    
    await assertRejects(
      () => controller.createPlayer(playerData),
      Error,
      "Username already exists"
    );
  });
});
```

### ICP Canister (Rust)

#### Testing Framework

- **Unit Tests**: Built-in Rust testing framework
- **Integration Tests**: PocketIC for canister testing
- **Property Testing**: Proptest for property-based testing

#### Setup

```bash
cd packages/icp-canister

# Run unit tests
cargo test

# Run tests with output
cargo test -- --nocapture

# Run specific test
cargo test test_wallet_creation

# Run integration tests
cargo test --test integration_tests
```

#### Test Structure

```rust
// packages/icp-canister/src/wallet.rs
#[cfg(test)]
mod tests {
    use super::*;
    use crate::types::{WalletCreationRequest, TransferRequest};

    #[test]
    fn test_create_wallet_success() {
        let request = WalletCreationRequest {
            user_id: "test_user".to_string(),
            initial_balance: 1000,
        };

        let result = create_wallet(request);
        
        assert!(result.is_ok());
        let wallet = result.unwrap();
        assert_eq!(wallet.balance, 1000);
        assert_eq!(wallet.user_id, "test_user");
    }

    #[test]
    fn test_transfer_insufficient_funds() {
        let mut wallet = create_test_wallet(100);
        let request = TransferRequest {
            to: "recipient".to_string(),
            amount: 200,
        };

        let result = transfer(&mut wallet, request);
        
        assert!(result.is_err());
        assert_eq!(result.unwrap_err(), WalletError::InsufficientFunds);
    }
}

// Integration tests
// packages/icp-canister/tests/integration_tests.rs
use pocket_ic::PocketIc;
use candid::{Encode, Decode};

#[tokio::test]
async fn test_canister_deployment() {
    let pic = PocketIc::new();
    let canister_id = pic.create_canister();
    
    // Deploy canister
    let wasm_bytes = include_bytes!("../target/wasm32-unknown-unknown/release/rust_icp_canister.wasm");
    pic.install_canister(canister_id, wasm_bytes.to_vec(), vec![], None);

    // Test canister functionality
    let result = pic.query_call(
        canister_id,
        "get_balance",
        Encode!(&"test_user").unwrap(),
    ).await;

    assert!(result.is_ok());
}
```

## Integration Testing

### API Integration Tests

#### Game Server API Tests

```typescript
// server/game/tests/integration/api.test.ts
import { assertEquals } from "@std/assert";
import { describe, it, beforeAll, afterAll } from "@std/testing/bdd";

describe("Game Server API Integration", () => {
  let server: Deno.HttpServer;
  const baseUrl = "http://localhost:8001";

  beforeAll(async () => {
    // Start test server
    const { app } = await import("../../index.ts");
    server = Deno.serve({ port: 8001 }, app.fetch);
  });

  afterAll(() => {
    server.shutdown();
  });

  it("should authenticate user and return JWT", async () => {
    const response = await fetch(`${baseUrl}/api/auth/login`, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({
        username: "testuser",
        password: "testpass"
      })
    });

    assertEquals(response.status, 200);
    const data = await response.json();
    assertEquals(typeof data.token, "string");
  });

  it("should require authentication for protected routes", async () => {
    const response = await fetch(`${baseUrl}/api/player/profile`);
    assertEquals(response.status, 401);
  });
});
```

### Database Integration Tests

```typescript
// server/game/tests/integration/database.test.ts
import { assertEquals } from "@std/assert";
import { MongoClient } from "mongodb";
import { describe, it, beforeAll, afterAll } from "@std/testing/bdd";

describe("Database Integration", () => {
  let client: MongoClient;
  let db: any;

  beforeAll(async () => {
    client = new MongoClient("mongodb://localhost:27017");
    await client.connect();
    db = client.db("dhaniverse_test");
  });

  afterAll(async () => {
    await db.dropDatabase();
    await client.close();
  });

  it("should create and retrieve player data", async () => {
    const players = db.collection("players");
    
    const playerData = {
      username: "testplayer",
      email: "test@example.com",
      createdAt: new Date()
    };

    await players.insertOne(playerData);
    const retrieved = await players.findOne({ username: "testplayer" });

    assertEquals(retrieved.username, "testplayer");
    assertEquals(retrieved.email, "test@example.com");
  });
});
```

### Frontend-Backend Integration

```typescript
// src/services/__tests__/integration/ICPIntegration.test.ts
import { describe, it, expect, beforeAll, afterAll } from 'vitest';
import { ICPActorService } from '../ICPActorService';
import { setupTestCanister, cleanupTestCanister } from '../../__mocks__/testUtils';

describe('ICP Integration Tests', () => {
  let actorService: ICPActorService;
  let canisterId: string;

  beforeAll(async () => {
    canisterId = await setupTestCanister();
    actorService = ICPActorService.getInstance();
  });

  afterAll(async () => {
    await cleanupTestCanister(canisterId);
  });

  it('should create wallet and perform transfer', async () => {
    // Create wallet
    const wallet = await actorService.createWallet('test_user', 1000);
    expect(wallet.balance).toBe(1000);

    // Perform transfer
    const result = await actorService.transfer('test_user', 'recipient', 100);
    expect(result.success).toBe(true);

    // Verify balance
    const balance = await actorService.getBalance('test_user');
    expect(balance).toBe(900);
  });
});
```

## End-to-End Testing

### E2E Testing Framework

- **Framework**: Playwright
- **Browser Support**: Chromium, Firefox, Safari
- **Test Environment**: Isolated test environment with test data

#### Setup

```bash
# Install Playwright
npx playwright install

# Run E2E tests
npm run test:e2e

# Run E2E tests with UI
npm run test:e2e:ui

# Run specific test file
npx playwright test wallet-connection.spec.ts
```

#### Test Structure

```typescript
// tests/e2e/wallet-connection.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Wallet Connection Flow', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/');
  });

  test('should connect MetaMask wallet', async ({ page }) => {
    // Click connect wallet button
    await page.click('[data-testid="connect-wallet"]');
    
    // Select MetaMask option
    await page.click('[data-testid="metamask-option"]');
    
    // Mock MetaMask connection
    await page.evaluate(() => {
      window.ethereum = {
        request: async ({ method }) => {
          if (method === 'eth_requestAccounts') {
            return ['0x1234567890123456789012345678901234567890'];
          }
        }
      };
    });
    
    // Verify connection success
    await expect(page.locator('[data-testid="wallet-address"]')).toBeVisible();
    await expect(page.locator('[data-testid="wallet-address"]')).toContainText('0x1234');
  });

  test('should handle wallet connection error', async ({ page }) => {
    // Mock wallet rejection
    await page.evaluate(() => {
      window.ethereum = {
        request: async () => {
          throw new Error('User rejected request');
        }
      };
    });

    await page.click('[data-testid="connect-wallet"]');
    await page.click('[data-testid="metamask-option"]');

    // Verify error message
    await expect(page.locator('[data-testid="error-message"]')).toBeVisible();
    await expect(page.locator('[data-testid="error-message"]')).toContainText('Connection failed');
  });
});
```

### Game Flow E2E Tests

```typescript
// tests/e2e/game-flow.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Game Flow', () => {
  test('should complete full game session', async ({ page }) => {
    // Login
    await page.goto('/login');
    await page.fill('[data-testid="username"]', 'testuser');
    await page.fill('[data-testid="password"]', 'testpass');
    await page.click('[data-testid="login-button"]');

    // Wait for game to load
    await expect(page.locator('[data-testid="game-canvas"]')).toBeVisible();

    // Interact with ATM
    await page.click('[data-testid="atm-building"]');
    await expect(page.locator('[data-testid="atm-interface"]')).toBeVisible();

    // Check balance
    await page.click('[data-testid="check-balance"]');
    await expect(page.locator('[data-testid="balance-display"]')).toBeVisible();

    // Perform transaction
    await page.click('[data-testid="deposit-tab"]');
    await page.fill('[data-testid="amount-input"]', '100');
    await page.click('[data-testid="deposit-button"]');

    // Verify transaction success
    await expect(page.locator('[data-testid="success-message"]')).toBeVisible();
  });
});
```

## Manual Testing

### Testing Checklist

#### Pre-Release Testing

**Frontend Testing**
- [ ] All pages load without errors
- [ ] Responsive design works on mobile/tablet/desktop
- [ ] Wallet connection flows work for all supported wallets
- [ ] Game renders correctly and is playable
- [ ] All UI interactions work as expected
- [ ] Error states display appropriate messages
- [ ] Loading states show proper indicators

**Backend Testing**
- [ ] All API endpoints return expected responses
- [ ] Authentication and authorization work correctly
- [ ] Database operations complete successfully
- [ ] WebSocket connections establish and maintain properly
- [ ] Error handling returns appropriate error codes
- [ ] Rate limiting works as configured

**ICP Canister Testing**
- [ ] Canister deploys successfully to local and IC networks
- [ ] All public methods work as documented
- [ ] Wallet operations (create, transfer, balance) function correctly
- [ ] Authentication and authorization work properly
- [ ] Error handling returns appropriate error types
- [ ] Upgrade procedures work without data loss

**Integration Testing**
- [ ] Frontend communicates correctly with game server
- [ ] Game server integrates properly with ICP canister
- [ ] WebSocket messages flow correctly between components
- [ ] Database operations reflect in frontend UI
- [ ] Cross-component error handling works properly

### Browser Testing

Test on the following browsers and versions:

**Desktop**
- Chrome (latest 2 versions)
- Firefox (latest 2 versions)
- Safari (latest 2 versions)
- Edge (latest 2 versions)

**Mobile**
- iOS Safari (latest 2 versions)
- Android Chrome (latest 2 versions)

### Performance Testing

#### Frontend Performance
- [ ] Initial page load under 3 seconds
- [ ] Game initialization under 5 seconds
- [ ] Smooth 60fps gameplay
- [ ] Memory usage remains stable during extended play
- [ ] No memory leaks in long-running sessions

#### Backend Performance
- [ ] API response times under 200ms for simple operations
- [ ] Database queries complete within acceptable timeframes
- [ ] WebSocket message latency under 100ms
- [ ] Server handles expected concurrent user load
- [ ] Resource usage remains within acceptable limits

### Security Testing

#### Manual Security Checks
- [ ] Input validation prevents injection attacks
- [ ] Authentication tokens expire appropriately
- [ ] Sensitive data is not exposed in client-side code
- [ ] HTTPS is enforced in production
- [ ] CORS policies are properly configured
- [ ] Rate limiting prevents abuse
- [ ] Error messages don't leak sensitive information

## Test Data Management

### Test Data Strategy

**Unit Tests**: Use minimal, focused test data created in test setup
**Integration Tests**: Use dedicated test database with known data sets
**E2E Tests**: Use isolated test environment with controlled test data

### Test Data Cleanup

```typescript
// Automated cleanup after tests
afterEach(async () => {
  await cleanupTestData();
  await resetTestDatabase();
  await clearTestCaches();
});
```

### Mock Data Generation

```typescript
// Test data factories
export const createTestPlayer = (overrides = {}) => ({
  id: 'test-player-1',
  username: 'testuser',
  email: 'test@example.com',
  balance: 1000,
  createdAt: new Date(),
  ...overrides
});

export const createTestWallet = (overrides = {}) => ({
  id: 'test-wallet-1',
  userId: 'test-player-1',
  balance: 1000,
  currency: 'ICP',
  ...overrides
});
```

## Continuous Integration Testing

### CI Pipeline Testing

The CI pipeline runs the following test suites:

1. **Lint and Format Check**
   - ESLint for TypeScript/JavaScript
   - Rustfmt and Clippy for Rust
   - Prettier for code formatting

2. **Unit Tests**
   - Frontend unit tests with coverage reporting
   - Backend unit tests for all services
   - Rust unit tests for ICP canister

3. **Integration Tests**
   - API integration tests
   - Database integration tests
   - Cross-service integration tests

4. **Build Tests**
   - Frontend production build
   - Backend service builds
   - ICP canister WASM compilation

5. **Security Tests**
   - Dependency vulnerability scanning
   - Static code analysis
   - Security linting rules

### Test Reporting

- **Coverage Reports**: Minimum 80% code coverage required
- **Test Results**: Detailed test results with failure analysis
- **Performance Metrics**: Build time and test execution time tracking
- **Quality Gates**: Automated quality checks that must pass before merge

This comprehensive testing strategy ensures code quality, reliability, and maintainability across all components of the Dhaniverse project.