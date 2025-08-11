# Coding Standards

## Overview

This document establishes coding standards and best practices for the Dhaniverse project across TypeScript, Rust, and React components. These standards ensure code consistency, maintainability, and quality across the entire codebase.

## General Principles

### Code Quality Principles

- **Readability**: Code should be self-documenting and easy to understand
- **Consistency**: Follow established patterns and conventions
- **Simplicity**: Prefer simple solutions over complex ones
- **Maintainability**: Write code that is easy to modify and extend
- **Performance**: Consider performance implications of code decisions
- **Security**: Follow security best practices and avoid common vulnerabilities

### Documentation Standards

- Document public APIs and complex business logic
- Use clear and descriptive comments
- Keep documentation up-to-date with code changes
- Include examples for complex functionality
- Document assumptions and constraints

## TypeScript Standards

### File Organization

```typescript
// File structure for TypeScript modules
// 1. Imports (external libraries first, then internal)
import React from 'react';
import { ethers } from 'ethers';

import { WalletService } from '../services/WalletService';
import { formatCurrency } from '../utils/formatters';
import type { WalletConnection } from '../types/wallet';

// 2. Type definitions
interface ComponentProps {
  balance: number;
  onConnect: () => void;
}

// 3. Constants
const DEFAULT_TIMEOUT = 5000;
const SUPPORTED_NETWORKS = ['mainnet', 'testnet'] as const;

// 4. Main implementation
export const WalletConnector: React.FC<ComponentProps> = ({ balance, onConnect }) => {
  // Implementation
};

// 5. Default export (if applicable)
export default WalletConnector;
```

### Naming Conventions

```typescript
// Variables and functions: camelCase
const userBalance = 1000;
const calculateInterest = (principal: number, rate: number) => principal * rate;

// Constants: SCREAMING_SNAKE_CASE
const MAX_RETRY_ATTEMPTS = 3;
const API_BASE_URL = 'https://api.dhaniverse.com';

// Types and interfaces: PascalCase
interface UserProfile {
  id: string;
  username: string;
}

type WalletProvider = 'metamask' | 'coinbase' | 'walletconnect';

// Classes: PascalCase
class WalletManager {
  private connections: Map<string, WalletConnection>;
  
  public async connect(provider: WalletProvider): Promise<WalletConnection> {
    // Implementation
  }
}

// Enums: PascalCase with PascalCase values
enum TransactionStatus {
  Pending = 'Pending',
  Confirmed = 'Confirmed',
  Failed = 'Failed'
}

// Files and directories: kebab-case
// wallet-connector.ts
// user-profile/
```

### Type Definitions

```typescript
// Use interfaces for object shapes
interface WalletState {
  address: string | null;
  balance: number;
  isConnected: boolean;
  provider: WalletProvider | null;
}

// Use type aliases for unions and computed types
type ConnectionStatus = 'connecting' | 'connected' | 'disconnected' | 'error';
type WalletAction = 
  | { type: 'CONNECT_START' }
  | { type: 'CONNECT_SUCCESS'; payload: { address: string; balance: number } }
  | { type: 'CONNECT_ERROR'; payload: { error: string } };

// Use generic types for reusable patterns
interface ApiResponse<T> {
  data: T;
  success: boolean;
  message?: string;
}

interface Repository<T> {
  findById(id: string): Promise<T | null>;
  create(entity: Omit<T, 'id'>): Promise<T>;
  update(id: string, updates: Partial<T>): Promise<T>;
  delete(id: string): Promise<void>;
}

// Use utility types for transformations
type CreateUserRequest = Omit<User, 'id' | 'createdAt' | 'updatedAt'>;
type UpdateUserRequest = Partial<Pick<User, 'username' | 'email'>>;
```

### Function Standards

```typescript
// Function declarations for hoisted functions
function calculateCompoundInterest(
  principal: number,
  rate: number,
  time: number,
  compoundingFrequency: number = 1
): number {
  return principal * Math.pow(1 + rate / compoundingFrequency, compoundingFrequency * time);
}

// Arrow functions for callbacks and short functions
const formatBalance = (balance: number): string => 
  new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD' }).format(balance);

// Async/await for asynchronous operations
async function connectWallet(provider: WalletProvider): Promise<WalletConnection> {
  try {
    const connection = await WalletService.connect(provider);
    return connection;
  } catch (error) {
    console.error('Wallet connection failed:', error);
    throw new WalletConnectionError('Failed to connect wallet', { cause: error });
  }
}

// Use proper error handling
class WalletConnectionError extends Error {
  constructor(message: string, options?: ErrorOptions) {
    super(message, options);
    this.name = 'WalletConnectionError';
  }
}
```

### Import/Export Standards

```typescript
// Named exports for utilities and services
export const WalletService = {
  connect: async (provider: WalletProvider) => { /* implementation */ },
  disconnect: async () => { /* implementation */ },
  getBalance: async (address: string) => { /* implementation */ }
};

export { formatCurrency, formatDate } from './formatters';

// Default exports for main components/classes
export default class GameEngine {
  // Implementation
}

// Re-exports for barrel files
export { WalletConnector } from './WalletConnector';
export { TransactionHistory } from './TransactionHistory';
export type { WalletConnection, TransactionRecord } from './types';
```

### Error Handling

```typescript
// Custom error classes
class ValidationError extends Error {
  constructor(
    message: string,
    public field: string,
    public value: unknown
  ) {
    super(message);
    this.name = 'ValidationError';
  }
}

// Result pattern for operations that can fail
type Result<T, E = Error> = 
  | { success: true; data: T }
  | { success: false; error: E };

async function safeWalletConnect(provider: WalletProvider): Promise<Result<WalletConnection>> {
  try {
    const connection = await WalletService.connect(provider);
    return { success: true, data: connection };
  } catch (error) {
    return { success: false, error: error as Error };
  }
}

// Use proper error boundaries in React
class ErrorBoundary extends React.Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error: Error): State {
    return { hasError: true };
  }

  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    console.error('Error caught by boundary:', error, errorInfo);
    // Log to error reporting service
  }

  render() {
    if (this.state.hasError) {
      return <ErrorFallback />;
    }
    return this.props.children;
  }
}
```

## React Standards

### Component Structure

```typescript
// Functional component with TypeScript
interface WalletDisplayProps {
  address: string;
  balance: number;
  onDisconnect: () => void;
  className?: string;
}

export const WalletDisplay: React.FC<WalletDisplayProps> = ({
  address,
  balance,
  onDisconnect,
  className = ''
}) => {
  // Hooks at the top
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);
  
  // Event handlers
  const handleDisconnect = useCallback(async () => {
    setIsLoading(true);
    try {
      await onDisconnect();
    } catch (err) {
      setError('Failed to disconnect wallet');
    } finally {
      setIsLoading(false);
    }
  }, [onDisconnect]);

  // Effects
  useEffect(() => {
    // Side effects
  }, []);

  // Early returns for loading/error states
  if (error) {
    return <ErrorMessage message={error} />;
  }

  // Main render
  return (
    <div className={`wallet-display ${className}`}>
      <div className="address">{formatAddress(address)}</div>
      <div className="balance">{formatBalance(balance)}</div>
      <button 
        onClick={handleDisconnect}
        disabled={isLoading}
        className="disconnect-btn"
      >
        {isLoading ? 'Disconnecting...' : 'Disconnect'}
      </button>
    </div>
  );
};
```

### Hook Standards

```typescript
// Custom hooks for reusable logic
export function useWalletConnection() {
  const [state, setState] = useState<WalletState>({
    address: null,
    balance: 0,
    isConnected: false,
    provider: null
  });

  const connect = useCallback(async (provider: WalletProvider) => {
    setState(prev => ({ ...prev, isConnecting: true }));
    
    try {
      const connection = await WalletService.connect(provider);
      setState({
        address: connection.address,
        balance: connection.balance,
        isConnected: true,
        provider
      });
    } catch (error) {
      setState(prev => ({ ...prev, isConnecting: false }));
      throw error;
    }
  }, []);

  const disconnect = useCallback(async () => {
    await WalletService.disconnect();
    setState({
      address: null,
      balance: 0,
      isConnected: false,
      provider: null
    });
  }, []);

  return { ...state, connect, disconnect };
}

// Hook for API calls
export function useApiCall<T>(
  apiCall: () => Promise<T>,
  dependencies: React.DependencyList = []
) {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    let cancelled = false;

    const fetchData = async () => {
      try {
        setLoading(true);
        setError(null);
        const result = await apiCall();
        
        if (!cancelled) {
          setData(result);
        }
      } catch (err) {
        if (!cancelled) {
          setError(err as Error);
        }
      } finally {
        if (!cancelled) {
          setLoading(false);
        }
      }
    };

    fetchData();

    return () => {
      cancelled = true;
    };
  }, dependencies);

  return { data, loading, error };
}
```

### State Management

```typescript
// Context for global state
interface AppContextType {
  user: User | null;
  wallet: WalletState;
  theme: 'light' | 'dark';
  updateUser: (user: User) => void;
  updateWallet: (wallet: WalletState) => void;
  toggleTheme: () => void;
}

export const AppContext = React.createContext<AppContextType | null>(null);

export const useAppContext = () => {
  const context = useContext(AppContext);
  if (!context) {
    throw new Error('useAppContext must be used within AppProvider');
  }
  return context;
};

// Reducer for complex state logic
type GameAction = 
  | { type: 'PLAYER_MOVE'; payload: { x: number; y: number } }
  | { type: 'PLAYER_INTERACT'; payload: { targetId: string } }
  | { type: 'UPDATE_BALANCE'; payload: { balance: number } };

interface GameState {
  player: {
    position: { x: number; y: number };
    balance: number;
  };
  npcs: NPC[];
  buildings: Building[];
}

function gameReducer(state: GameState, action: GameAction): GameState {
  switch (action.type) {
    case 'PLAYER_MOVE':
      return {
        ...state,
        player: {
          ...state.player,
          position: action.payload
        }
      };
    
    case 'UPDATE_BALANCE':
      return {
        ...state,
        player: {
          ...state.player,
          balance: action.payload.balance
        }
      };
    
    default:
      return state;
  }
}
```

## Rust Standards

### Project Structure

```rust
// lib.rs - Main library file
use ic_cdk::export_candid;
use ic_cdk_macros::{init, post_upgrade, pre_upgrade, query, update};

mod auth;
mod banking;
mod error;
mod storage;
mod types;
mod utils;
mod wallet;

// Re-export public types
pub use types::*;
pub use error::*;

// Candid export for interface generation
export_candid!();
```

### Naming Conventions

```rust
// Variables and functions: snake_case
let user_balance = 1000;
fn calculate_interest(principal: u64, rate: f64) -> u64 {
    (principal as f64 * rate) as u64
}

// Constants: SCREAMING_SNAKE_CASE
const MAX_TRANSFER_AMOUNT: u64 = 1_000_000;
const DEFAULT_WALLET_BALANCE: u64 = 0;

// Types and structs: PascalCase
#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct WalletInfo {
    pub user_id: String,
    pub balance: u64,
    pub created_at: u64,
}

// Enums: PascalCase with PascalCase variants
#[derive(Debug, Clone, Serialize, Deserialize)]
pub enum TransactionType {
    Deposit,
    Withdrawal,
    Transfer,
    Interest,
}

// Modules: snake_case
mod wallet_manager;
mod transaction_history;

// Files: snake_case
// wallet_manager.rs
// transaction_history.rs
```

### Error Handling

```rust
// Custom error types
#[derive(Debug, Clone, Serialize, Deserialize)]
pub enum WalletError {
    UserNotFound,
    InsufficientFunds,
    InvalidAmount,
    TransferFailed(String),
    AuthenticationFailed,
}

impl std::fmt::Display for WalletError {
    fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
        match self {
            WalletError::UserNotFound => write!(f, "User not found"),
            WalletError::InsufficientFunds => write!(f, "Insufficient funds"),
            WalletError::InvalidAmount => write!(f, "Invalid amount"),
            WalletError::TransferFailed(msg) => write!(f, "Transfer failed: {}", msg),
            WalletError::AuthenticationFailed => write!(f, "Authentication failed"),
        }
    }
}

// Result type alias
pub type WalletResult<T> = Result<T, WalletError>;

// Function with proper error handling
pub fn transfer_funds(
    from_user: &str,
    to_user: &str,
    amount: u64,
) -> WalletResult<TransactionRecord> {
    if amount == 0 {
        return Err(WalletError::InvalidAmount);
    }

    let mut from_wallet = get_wallet(from_user)
        .ok_or(WalletError::UserNotFound)?;

    if from_wallet.balance < amount {
        return Err(WalletError::InsufficientFunds);
    }

    let mut to_wallet = get_wallet(to_user)
        .ok_or(WalletError::UserNotFound)?;

    // Perform transfer
    from_wallet.balance -= amount;
    to_wallet.balance += amount;

    // Update storage
    update_wallet(&from_wallet);
    update_wallet(&to_wallet);

    // Create transaction record
    let transaction = TransactionRecord {
        id: generate_transaction_id(),
        from_user: from_user.to_string(),
        to_user: to_user.to_string(),
        amount,
        transaction_type: TransactionType::Transfer,
        timestamp: ic_cdk::api::time(),
    };

    Ok(transaction)
}
```

### Function Standards

```rust
// Public API functions with documentation
/// Creates a new wallet for the specified user
/// 
/// # Arguments
/// * `user_id` - Unique identifier for the user
/// * `initial_balance` - Starting balance for the wallet
/// 
/// # Returns
/// * `WalletResult<WalletInfo>` - The created wallet or an error
/// 
/// # Errors
/// * Returns `WalletError::InvalidAmount` if initial_balance is negative
/// * Returns `WalletError::UserAlreadyExists` if user already has a wallet
#[update]
pub fn create_wallet(user_id: String, initial_balance: u64) -> WalletResult<WalletInfo> {
    validate_user_id(&user_id)?;
    
    if wallet_exists(&user_id) {
        return Err(WalletError::UserAlreadyExists);
    }

    let wallet = WalletInfo {
        user_id: user_id.clone(),
        balance: initial_balance,
        created_at: ic_cdk::api::time(),
    };

    store_wallet(&wallet);
    
    Ok(wallet)
}

// Private helper functions
fn validate_user_id(user_id: &str) -> WalletResult<()> {
    if user_id.is_empty() || user_id.len() > 64 {
        return Err(WalletError::InvalidUserId);
    }
    Ok(())
}

fn wallet_exists(user_id: &str) -> bool {
    WALLETS.with(|wallets| wallets.borrow().contains_key(user_id))
}
```

### Memory Management

```rust
use ic_stable_structures::{BTreeMap, DefaultMemoryImpl, StableBTreeMap};
use std::cell::RefCell;

// Stable storage for persistence across upgrades
thread_local! {
    static WALLETS: RefCell<StableBTreeMap<String, WalletInfo, DefaultMemoryImpl>> = 
        RefCell::new(StableBTreeMap::init(DefaultMemoryImpl::default()));
    
    static TRANSACTIONS: RefCell<StableBTreeMap<String, TransactionRecord, DefaultMemoryImpl>> = 
        RefCell::new(StableBTreeMap::init(DefaultMemoryImpl::default()));
}

// Canister lifecycle hooks
#[init]
fn init() {
    // Initialize canister state
}

#[pre_upgrade]
fn pre_upgrade() {
    // Prepare for upgrade
}

#[post_upgrade]
fn post_upgrade() {
    // Restore state after upgrade
}
```

### Testing Standards

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_create_wallet_success() {
        let user_id = "test_user".to_string();
        let initial_balance = 1000;

        let result = create_wallet(user_id.clone(), initial_balance);
        
        assert!(result.is_ok());
        let wallet = result.unwrap();
        assert_eq!(wallet.user_id, user_id);
        assert_eq!(wallet.balance, initial_balance);
    }

    #[test]
    fn test_transfer_insufficient_funds() {
        // Setup test data
        let from_user = "user1".to_string();
        let to_user = "user2".to_string();
        
        create_wallet(from_user.clone(), 100).unwrap();
        create_wallet(to_user.clone(), 0).unwrap();

        // Attempt transfer with insufficient funds
        let result = transfer_funds(&from_user, &to_user, 200);
        
        assert!(result.is_err());
        assert_eq!(result.unwrap_err(), WalletError::InsufficientFunds);
    }

    #[test]
    fn test_invalid_user_id() {
        let result = create_wallet("".to_string(), 1000);
        assert!(result.is_err());
        assert_eq!(result.unwrap_err(), WalletError::InvalidUserId);
    }
}

// Property-based testing with proptest
#[cfg(test)]
mod property_tests {
    use super::*;
    use proptest::prelude::*;

    proptest! {
        #[test]
        fn test_balance_never_negative(
            initial_balance in 0u64..1_000_000,
            transfer_amount in 0u64..1_000_000
        ) {
            let user_id = "test_user".to_string();
            create_wallet(user_id.clone(), initial_balance).unwrap();
            
            let wallet = get_wallet(&user_id).unwrap();
            
            // Balance should never go negative
            prop_assert!(wallet.balance >= 0);
        }
    }
}
```

## Code Formatting and Linting

### TypeScript/JavaScript

```json
// .eslintrc.json
{
  "extends": [
    "@typescript-eslint/recommended",
    "plugin:react/recommended",
    "plugin:react-hooks/recommended"
  ],
  "rules": {
    "@typescript-eslint/no-unused-vars": "error",
    "@typescript-eslint/explicit-function-return-type": "warn",
    "react/prop-types": "off",
    "react/react-in-jsx-scope": "off",
    "prefer-const": "error",
    "no-var": "error"
  }
}

// prettier.config.js
module.exports = {
  semi: true,
  trailingComma: 'es5',
  singleQuote: true,
  printWidth: 100,
  tabWidth: 2,
  useTabs: false
};
```

### Rust

```toml
# rustfmt.toml
max_width = 100
hard_tabs = false
tab_spaces = 4
newline_style = "Unix"
use_small_heuristics = "Default"
reorder_imports = true
reorder_modules = true
remove_nested_parens = true
edition = "2021"
```

## Performance Guidelines

### TypeScript/React Performance

```typescript
// Use React.memo for expensive components
const ExpensiveComponent = React.memo<Props>(({ data, onUpdate }) => {
  // Expensive rendering logic
  return <div>{/* Complex UI */}</div>;
});

// Use useMemo for expensive calculations
const GameStats = ({ players, transactions }) => {
  const stats = useMemo(() => {
    return calculateComplexStats(players, transactions);
  }, [players, transactions]);

  return <div>{stats}</div>;
};

// Use useCallback for stable function references
const PlayerList = ({ players, onPlayerSelect }) => {
  const handlePlayerClick = useCallback((playerId: string) => {
    onPlayerSelect(playerId);
  }, [onPlayerSelect]);

  return (
    <div>
      {players.map(player => (
        <PlayerCard 
          key={player.id}
          player={player}
          onClick={handlePlayerClick}
        />
      ))}
    </div>
  );
};
```

### Rust Performance

```rust
// Use appropriate data structures
use std::collections::HashMap;
use ic_stable_structures::BTreeMap;

// Prefer borrowing over cloning
fn process_wallet(wallet: &WalletInfo) -> u64 {
    wallet.balance * 2  // No unnecessary cloning
}

// Use iterators for efficient processing
fn calculate_total_balance(wallets: &[WalletInfo]) -> u64 {
    wallets.iter()
        .map(|wallet| wallet.balance)
        .sum()
}

// Avoid unnecessary allocations
fn format_balance(balance: u64) -> String {
    format!("{:.2}", balance as f64 / 100.0)  // Only allocate when necessary
}
```

## Security Guidelines

### Input Validation

```typescript
// TypeScript input validation
function validateTransferAmount(amount: number): boolean {
  return amount > 0 && amount <= MAX_TRANSFER_AMOUNT && Number.isFinite(amount);
}

function sanitizeUserInput(input: string): string {
  return input.trim().slice(0, MAX_INPUT_LENGTH);
}
```

```rust
// Rust input validation
fn validate_transfer_request(request: &TransferRequest) -> WalletResult<()> {
    if request.amount == 0 {
        return Err(WalletError::InvalidAmount);
    }
    
    if request.amount > MAX_TRANSFER_AMOUNT {
        return Err(WalletError::AmountTooLarge);
    }
    
    if request.to_user.is_empty() || request.to_user.len() > MAX_USER_ID_LENGTH {
        return Err(WalletError::InvalidUserId);
    }
    
    Ok(())
}
```

### Authentication and Authorization

```typescript
// Secure token handling
const AuthService = {
  async validateToken(token: string): Promise<User | null> {
    try {
      const decoded = jwt.verify(token, JWT_SECRET);
      return await UserService.findById(decoded.userId);
    } catch (error) {
      return null;
    }
  },

  async requireAuth(req: Request): Promise<User> {
    const token = req.headers.authorization?.replace('Bearer ', '');
    if (!token) {
      throw new UnauthorizedError('No token provided');
    }

    const user = await this.validateToken(token);
    if (!user) {
      throw new UnauthorizedError('Invalid token');
    }

    return user;
  }
};
```

These coding standards ensure consistency, maintainability, and quality across the Dhaniverse codebase. All team members should follow these guidelines and use the provided linting and formatting tools to maintain code quality.