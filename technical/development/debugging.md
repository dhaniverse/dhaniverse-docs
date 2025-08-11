# Debugging Guide

## Overview

This document provides comprehensive debugging guides for each component of the Dhaniverse project, including the React frontend, Deno game servers, ICP Rust canister, and integration between components. It covers debugging tools, techniques, and common issues with their solutions.

## General Debugging Principles

### Debugging Methodology

1. **Reproduce the Issue**: Ensure you can consistently reproduce the problem
2. **Isolate the Problem**: Narrow down the scope to identify the specific component
3. **Gather Information**: Collect logs, error messages, and system state
4. **Form Hypotheses**: Develop theories about the root cause
5. **Test Systematically**: Test one hypothesis at a time
6. **Document Findings**: Record solutions for future reference

### Logging Best Practices

- Use appropriate log levels (error, warn, info, debug)
- Include contextual information (user ID, transaction ID, timestamps)
- Avoid logging sensitive information (passwords, private keys)
- Use structured logging for better searchability
- Include stack traces for errors

## Frontend Debugging (React/TypeScript)

### Browser Developer Tools

#### Console Debugging

```typescript
// Use console methods effectively
console.log('Basic logging');
console.warn('Warning message');
console.error('Error occurred:', error);
console.table(arrayOfObjects);  // Display arrays/objects in table format
console.group('Wallet Connection');
console.log('Attempting connection...');
console.log('Provider:', provider);
console.groupEnd();

// Conditional logging
const DEBUG = process.env.NODE_ENV === 'development';
if (DEBUG) {
  console.log('Debug info:', debugData);
}

// Performance timing
console.time('API Call');
await apiCall();
console.timeEnd('API Call');
```

#### Network Tab Debugging

Monitor network requests to identify:
- Failed API calls
- Slow response times
- Incorrect request payloads
- CORS issues
- Authentication failures

#### React Developer Tools

Install React Developer Tools browser extension for:
- Component tree inspection
- Props and state examination
- Performance profiling
- Hook debugging

### React-Specific Debugging

#### Component Debugging

```typescript
// Debug component renders
const WalletConnector: React.FC<Props> = ({ provider, onConnect }) => {
  // Log renders in development
  useEffect(() => {
    if (process.env.NODE_ENV === 'development') {
      console.log('WalletConnector rendered with:', { provider, onConnect });
    }
  });

  // Debug state changes
  const [connectionState, setConnectionState] = useState<ConnectionState>('disconnected');
  
  useEffect(() => {
    console.log('Connection state changed:', connectionState);
  }, [connectionState]);

  return (
    <div>
      {/* Add debug info in development */}
      {process.env.NODE_ENV === 'development' && (
        <div style={{ background: 'yellow', padding: '4px', fontSize: '12px' }}>
          Debug: {JSON.stringify({ provider, connectionState })}
        </div>
      )}
      {/* Component content */}
    </div>
  );
};
```

#### Hook Debugging

```typescript
// Custom hook with debugging
export function useWalletConnection() {
  const [state, setState] = useState<WalletState>(initialState);

  // Debug hook state changes
  useEffect(() => {
    if (process.env.NODE_ENV === 'development') {
      console.log('useWalletConnection state changed:', state);
    }
  }, [state]);

  const connect = useCallback(async (provider: WalletProvider) => {
    console.log('Attempting to connect with provider:', provider);
    
    try {
      const connection = await WalletService.connect(provider);
      console.log('Connection successful:', connection);
      setState(prev => ({ ...prev, ...connection, isConnected: true }));
    } catch (error) {
      console.error('Connection failed:', error);
      setState(prev => ({ ...prev, error: error.message }));
      throw error;
    }
  }, []);

  return { ...state, connect };
}
```

#### Error Boundary Debugging

```typescript
class ErrorBoundary extends React.Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    // Log detailed error information
    console.error('Error Boundary caught an error:', {
      error: error.message,
      stack: error.stack,
      componentStack: errorInfo.componentStack,
      errorBoundary: this.constructor.name
    });

    // Send to error reporting service in production
    if (process.env.NODE_ENV === 'production') {
      ErrorReportingService.report(error, errorInfo);
    }
  }

  render() {
    if (this.state.hasError) {
      return (
        <div className="error-boundary">
          <h2>Something went wrong</h2>
          {process.env.NODE_ENV === 'development' && (
            <details>
              <summary>Error Details</summary>
              <pre>{this.state.error?.stack}</pre>
            </details>
          )}
        </div>
      );
    }

    return this.props.children;
  }
}
```

### Common Frontend Issues

#### Issue: Component Not Re-rendering

**Symptoms**: UI doesn't update when state changes

**Debugging Steps**:
1. Check if state is actually changing using React DevTools
2. Verify state updates are immutable
3. Check dependency arrays in useEffect/useMemo/useCallback

```typescript
// Incorrect - mutating state
const addItem = (item) => {
  items.push(item);  // This won't trigger re-render
  setItems(items);
};

// Correct - immutable update
const addItem = (item) => {
  setItems(prev => [...prev, item]);
};
```

#### Issue: Infinite Re-renders

**Symptoms**: Browser becomes unresponsive, "Maximum update depth exceeded" error

**Debugging Steps**:
1. Check for missing dependency arrays
2. Look for state updates in render function
3. Verify useEffect dependencies

```typescript
// Problematic code
const Component = () => {
  const [count, setCount] = useState(0);
  
  // This causes infinite re-renders
  useEffect(() => {
    setCount(count + 1);  // Missing dependency array
  });

  return <div>{count}</div>;
};

// Fixed code
const Component = () => {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    setCount(prev => prev + 1);
  }, []);  // Empty dependency array for one-time effect

  return <div>{count}</div>;
};
```

## Backend Debugging (Deno/TypeScript)

### Deno Debugging Tools

#### Built-in Debugger

```bash
# Start with debugger
deno run --inspect-brk --allow-all server/game/index.ts

# Connect with Chrome DevTools
# Open chrome://inspect in Chrome browser
```

#### Logging Configuration

```typescript
// server/game/src/utils/logger.ts
export enum LogLevel {
  ERROR = 0,
  WARN = 1,
  INFO = 2,
  DEBUG = 3
}

class Logger {
  private level: LogLevel;

  constructor(level: LogLevel = LogLevel.INFO) {
    this.level = level;
  }

  error(message: string, meta?: any) {
    if (this.level >= LogLevel.ERROR) {
      console.error(`[ERROR] ${new Date().toISOString()} ${message}`, meta || '');
    }
  }

  warn(message: string, meta?: any) {
    if (this.level >= LogLevel.WARN) {
      console.warn(`[WARN] ${new Date().toISOString()} ${message}`, meta || '');
    }
  }

  info(message: string, meta?: any) {
    if (this.level >= LogLevel.INFO) {
      console.info(`[INFO] ${new Date().toISOString()} ${message}`, meta || '');
    }
  }

  debug(message: string, meta?: any) {
    if (this.level >= LogLevel.DEBUG) {
      console.debug(`[DEBUG] ${new Date().toISOString()} ${message}`, meta || '');
    }
  }
}

export const logger = new Logger(
  Deno.env.get('LOG_LEVEL') === 'debug' ? LogLevel.DEBUG : LogLevel.INFO
);
```

#### API Endpoint Debugging

```typescript
// server/game/src/routes/player.ts
import { Router } from 'oak';
import { logger } from '../utils/logger.ts';

const router = new Router();

router.post('/api/player/create', async (ctx) => {
  const requestId = crypto.randomUUID();
  logger.info(`[${requestId}] Creating player`, { 
    ip: ctx.request.ip,
    userAgent: ctx.request.headers.get('user-agent')
  });

  try {
    const body = await ctx.request.body().value;
    logger.debug(`[${requestId}] Request body:`, body);

    // Validate input
    if (!body.username || !body.email) {
      logger.warn(`[${requestId}] Invalid input:`, body);
      ctx.response.status = 400;
      ctx.response.body = { error: 'Username and email required' };
      return;
    }

    // Create player
    const player = await PlayerController.create(body);
    logger.info(`[${requestId}] Player created successfully:`, { playerId: player.id });

    ctx.response.body = { success: true, player };
  } catch (error) {
    logger.error(`[${requestId}] Error creating player:`, {
      error: error.message,
      stack: error.stack
    });

    ctx.response.status = 500;
    ctx.response.body = { error: 'Internal server error' };
  }
});
```

#### Database Debugging

```typescript
// server/game/src/db/mongo.ts
import { MongoClient } from 'mongodb';
import { logger } from '../utils/logger.ts';

export class DatabaseManager {
  private client: MongoClient;
  private db: any;

  async connect() {
    try {
      logger.info('Connecting to MongoDB...');
      this.client = new MongoClient(Deno.env.get('MONGODB_URI')!);
      await this.client.connect();
      this.db = this.client.db('dhaniverse');
      logger.info('MongoDB connected successfully');
    } catch (error) {
      logger.error('MongoDB connection failed:', error);
      throw error;
    }
  }

  async findPlayer(id: string) {
    const startTime = Date.now();
    
    try {
      logger.debug(`Finding player: ${id}`);
      const player = await this.db.collection('players').findOne({ _id: id });
      
      const duration = Date.now() - startTime;
      logger.debug(`Player query completed in ${duration}ms`);
      
      return player;
    } catch (error) {
      logger.error(`Error finding player ${id}:`, error);
      throw error;
    }
  }
}
```

### Common Backend Issues

#### Issue: Database Connection Failures

**Symptoms**: "Connection refused" or timeout errors

**Debugging Steps**:
1. Check database server status
2. Verify connection string and credentials
3. Check network connectivity
4. Review firewall settings

```typescript
// Add connection retry logic
async function connectWithRetry(maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      await dbManager.connect();
      logger.info('Database connected successfully');
      return;
    } catch (error) {
      logger.warn(`Database connection attempt ${i + 1} failed:`, error.message);
      
      if (i === maxRetries - 1) {
        logger.error('All database connection attempts failed');
        throw error;
      }
      
      // Wait before retry
      await new Promise(resolve => setTimeout(resolve, 1000 * (i + 1)));
    }
  }
}
```

#### Issue: Memory Leaks

**Symptoms**: Increasing memory usage over time

**Debugging Steps**:
1. Monitor memory usage with `deno run --v8-flags=--expose-gc`
2. Check for unclosed database connections
3. Look for event listeners that aren't removed
4. Review caching mechanisms

```typescript
// Proper cleanup example
class WebSocketManager {
  private connections = new Map<string, WebSocket>();

  addConnection(id: string, ws: WebSocket) {
    // Clean up existing connection if any
    this.removeConnection(id);
    
    this.connections.set(id, ws);
    
    ws.addEventListener('close', () => {
      this.removeConnection(id);
    });
  }

  removeConnection(id: string) {
    const ws = this.connections.get(id);
    if (ws) {
      ws.close();
      this.connections.delete(id);
      logger.debug(`Connection ${id} cleaned up`);
    }
  }

  cleanup() {
    for (const [id, ws] of this.connections) {
      ws.close();
    }
    this.connections.clear();
    logger.info('All WebSocket connections cleaned up');
  }
}
```

## ICP Canister Debugging (Rust)

### Rust Debugging Tools

#### Local Development

```bash
# Start local replica with verbose logging
dfx start --clean --verbose

# Deploy with debug info
dfx deploy --with-cycles 1000000000000 --verbose

# Check canister logs
dfx canister logs rust_icp_canister

# Call canister methods for testing
dfx canister call rust_icp_canister get_balance '("test_user")'
```

#### Debugging Macros

```rust
// src/lib.rs
use ic_cdk::println;

#[update]
pub fn create_wallet(user_id: String, initial_balance: u64) -> WalletResult<WalletInfo> {
    println!("Creating wallet for user: {}, balance: {}", user_id, initial_balance);
    
    // Validate input
    if user_id.is_empty() {
        println!("Error: Empty user ID provided");
        return Err(WalletError::InvalidUserId);
    }

    // Check if wallet already exists
    let exists = WALLETS.with(|wallets| {
        wallets.borrow().contains_key(&user_id)
    });

    if exists {
        println!("Error: Wallet already exists for user: {}", user_id);
        return Err(WalletError::WalletAlreadyExists);
    }

    let wallet = WalletInfo {
        user_id: user_id.clone(),
        balance: initial_balance,
        created_at: ic_cdk::api::time(),
    };

    // Store wallet
    WALLETS.with(|wallets| {
        wallets.borrow_mut().insert(user_id.clone(), wallet.clone());
    });

    println!("Wallet created successfully for user: {}", user_id);
    Ok(wallet)
}
```

#### Error Handling and Logging

```rust
// src/error.rs
use ic_cdk::println;

#[derive(Debug, Clone)]
pub enum WalletError {
    UserNotFound,
    InsufficientFunds,
    InvalidAmount,
    TransferFailed(String),
}

impl WalletError {
    pub fn log(&self) {
        match self {
            WalletError::UserNotFound => {
                println!("Error: User not found in wallet system");
            }
            WalletError::InsufficientFunds => {
                println!("Error: Insufficient funds for transaction");
            }
            WalletError::InvalidAmount => {
                println!("Error: Invalid amount specified");
            }
            WalletError::TransferFailed(msg) => {
                println!("Error: Transfer failed - {}", msg);
            }
        }
    }
}

// Helper function for error logging
pub fn log_and_return_error<T>(error: WalletError) -> WalletResult<T> {
    error.log();
    Err(error)
}
```

#### State Inspection

```rust
// Debug queries for state inspection
#[query]
pub fn debug_get_all_wallets() -> Vec<(String, WalletInfo)> {
    WALLETS.with(|wallets| {
        wallets.borrow()
            .iter()
            .map(|(k, v)| (k.clone(), v.clone()))
            .collect()
    })
}

#[query]
pub fn debug_get_wallet_count() -> u64 {
    WALLETS.with(|wallets| wallets.borrow().len() as u64)
}

#[query]
pub fn debug_get_total_balance() -> u64 {
    WALLETS.with(|wallets| {
        wallets.borrow()
            .iter()
            .map(|(_, wallet)| wallet.balance)
            .sum()
    })
}
```

### Testing and Debugging

```rust
// src/tests.rs
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_wallet_creation_debug() {
        println!("Starting wallet creation test");
        
        let user_id = "test_user".to_string();
        let initial_balance = 1000;

        println!("Creating wallet for user: {}", user_id);
        let result = create_wallet(user_id.clone(), initial_balance);

        match &result {
            Ok(wallet) => {
                println!("Wallet created successfully: {:?}", wallet);
                assert_eq!(wallet.user_id, user_id);
                assert_eq!(wallet.balance, initial_balance);
            }
            Err(error) => {
                println!("Wallet creation failed: {:?}", error);
                panic!("Unexpected error: {:?}", error);
            }
        }
    }

    #[test]
    fn test_transfer_with_debug() {
        println!("Starting transfer test");
        
        // Setup
        let user1 = "user1".to_string();
        let user2 = "user2".to_string();
        
        create_wallet(user1.clone(), 1000).unwrap();
        create_wallet(user2.clone(), 500).unwrap();
        
        println!("Initial wallets created");
        
        // Perform transfer
        let transfer_amount = 200;
        println!("Transferring {} from {} to {}", transfer_amount, user1, user2);
        
        let result = transfer_funds(&user1, &user2, transfer_amount);
        
        match result {
            Ok(transaction) => {
                println!("Transfer successful: {:?}", transaction);
                
                // Verify balances
                let balance1 = get_balance(&user1).unwrap();
                let balance2 = get_balance(&user2).unwrap();
                
                println!("Final balances - {}: {}, {}: {}", user1, balance1, user2, balance2);
                
                assert_eq!(balance1, 800);
                assert_eq!(balance2, 700);
            }
            Err(error) => {
                println!("Transfer failed: {:?}", error);
                panic!("Unexpected transfer error: {:?}", error);
            }
        }
    }
}
```

### Common Canister Issues

#### Issue: Canister Upgrade Failures

**Symptoms**: Upgrade fails or data is lost after upgrade

**Debugging Steps**:
1. Check pre_upgrade and post_upgrade hooks
2. Verify stable memory usage
3. Test upgrade process locally

```rust
use ic_stable_structures::{StableBTreeMap, DefaultMemoryImpl};

#[pre_upgrade]
fn pre_upgrade() {
    println!("Starting canister upgrade...");
    
    // Verify data integrity before upgrade
    let wallet_count = WALLETS.with(|wallets| wallets.borrow().len());
    println!("Preserving {} wallets during upgrade", wallet_count);
}

#[post_upgrade]
fn post_upgrade() {
    println!("Canister upgrade completed");
    
    // Verify data integrity after upgrade
    let wallet_count = WALLETS.with(|wallets| wallets.borrow().len());
    println!("Restored {} wallets after upgrade", wallet_count);
}
```

#### Issue: Out of Cycles

**Symptoms**: Canister stops responding, "out of cycles" errors

**Debugging Steps**:
1. Check cycle balance: `dfx canister status rust_icp_canister`
2. Monitor cycle consumption
3. Optimize expensive operations

```rust
#[query]
pub fn get_cycles_balance() -> u64 {
    ic_cdk::api::canister_balance()
}

// Add cycle monitoring to expensive operations
#[update]
pub fn expensive_operation() -> String {
    let cycles_before = ic_cdk::api::canister_balance();
    println!("Cycles before operation: {}", cycles_before);
    
    // Perform operation
    let result = perform_complex_calculation();
    
    let cycles_after = ic_cdk::api::canister_balance();
    let cycles_used = cycles_before - cycles_after;
    println!("Cycles used: {}", cycles_used);
    
    result
}
```

## Integration Debugging

### Cross-Component Communication

#### Frontend to Backend API Debugging

```typescript
// Add request/response interceptors
const apiClient = axios.create({
  baseURL: 'http://localhost:8000',
  timeout: 10000,
});

// Request interceptor
apiClient.interceptors.request.use(
  (config) => {
    console.log('API Request:', {
      method: config.method,
      url: config.url,
      data: config.data,
      headers: config.headers,
    });
    return config;
  },
  (error) => {
    console.error('API Request Error:', error);
    return Promise.reject(error);
  }
);

// Response interceptor
apiClient.interceptors.response.use(
  (response) => {
    console.log('API Response:', {
      status: response.status,
      data: response.data,
      headers: response.headers,
    });
    return response;
  },
  (error) => {
    console.error('API Response Error:', {
      status: error.response?.status,
      data: error.response?.data,
      message: error.message,
    });
    return Promise.reject(error);
  }
);
```

#### WebSocket Debugging

```typescript
// Frontend WebSocket debugging
class DebugWebSocketManager {
  private ws: WebSocket | null = null;
  private messageLog: Array<{ timestamp: Date; type: 'sent' | 'received'; data: any }> = [];

  connect(url: string) {
    console.log('Connecting to WebSocket:', url);
    
    this.ws = new WebSocket(url);
    
    this.ws.onopen = (event) => {
      console.log('WebSocket connected:', event);
    };
    
    this.ws.onmessage = (event) => {
      const data = JSON.parse(event.data);
      console.log('WebSocket message received:', data);
      
      this.messageLog.push({
        timestamp: new Date(),
        type: 'received',
        data
      });
    };
    
    this.ws.onerror = (error) => {
      console.error('WebSocket error:', error);
    };
    
    this.ws.onclose = (event) => {
      console.log('WebSocket closed:', { code: event.code, reason: event.reason });
    };
  }

  send(data: any) {
    if (this.ws && this.ws.readyState === WebSocket.OPEN) {
      console.log('Sending WebSocket message:', data);
      this.ws.send(JSON.stringify(data));
      
      this.messageLog.push({
        timestamp: new Date(),
        type: 'sent',
        data
      });
    } else {
      console.error('WebSocket not connected');
    }
  }

  getMessageLog() {
    return this.messageLog;
  }
}
```

#### ICP Integration Debugging

```typescript
// Frontend ICP canister debugging
class DebugICPActorService {
  private actor: any;

  async initializeActor() {
    try {
      console.log('Initializing ICP actor...');
      
      const agent = new HttpAgent({
        host: process.env.NODE_ENV === 'development' 
          ? 'http://localhost:4943' 
          : 'https://ic0.app'
      });

      if (process.env.NODE_ENV === 'development') {
        await agent.fetchRootKey();
        console.log('Root key fetched for local development');
      }

      this.actor = Actor.createActor(idlFactory, {
        agent,
        canisterId: CANISTER_ID,
      });

      console.log('ICP actor initialized successfully');
    } catch (error) {
      console.error('Failed to initialize ICP actor:', error);
      throw error;
    }
  }

  async callCanisterMethod(methodName: string, args: any[]) {
    console.log(`Calling canister method: ${methodName}`, args);
    
    try {
      const result = await this.actor[methodName](...args);
      console.log(`Canister method ${methodName} result:`, result);
      return result;
    } catch (error) {
      console.error(`Canister method ${methodName} failed:`, error);
      throw error;
    }
  }
}
```

## Performance Debugging

### Frontend Performance

```typescript
// Performance monitoring
class PerformanceMonitor {
  static measureRender(componentName: string, renderFn: () => JSX.Element) {
    const startTime = performance.now();
    const result = renderFn();
    const endTime = performance.now();
    
    console.log(`${componentName} render time: ${endTime - startTime}ms`);
    return result;
  }

  static measureAsync(operationName: string, asyncFn: () => Promise<any>) {
    return async (...args: any[]) => {
      const startTime = performance.now();
      try {
        const result = await asyncFn(...args);
        const endTime = performance.now();
        console.log(`${operationName} completed in ${endTime - startTime}ms`);
        return result;
      } catch (error) {
        const endTime = performance.now();
        console.error(`${operationName} failed after ${endTime - startTime}ms:`, error);
        throw error;
      }
    };
  }
}

// Usage
const measuredApiCall = PerformanceMonitor.measureAsync('API Call', apiCall);
```

### Memory Usage Monitoring

```typescript
// Memory usage tracking
class MemoryMonitor {
  static logMemoryUsage(label: string) {
    if ('memory' in performance) {
      const memory = (performance as any).memory;
      console.log(`Memory usage (${label}):`, {
        used: `${Math.round(memory.usedJSHeapSize / 1024 / 1024)} MB`,
        total: `${Math.round(memory.totalJSHeapSize / 1024 / 1024)} MB`,
        limit: `${Math.round(memory.jsHeapSizeLimit / 1024 / 1024)} MB`
      });
    }
  }

  static startMemoryMonitoring(interval: number = 5000) {
    setInterval(() => {
      this.logMemoryUsage('Periodic Check');
    }, interval);
  }
}
```

## Debugging Tools and Setup

### Development Environment

```bash
# Install debugging tools
npm install -g @types/node
npm install --save-dev source-map-support

# Browser extensions
# - React Developer Tools
# - Redux DevTools (if using Redux)
# - Web Vitals Extension
```

### IDE Configuration

#### VS Code Settings

```json
// .vscode/launch.json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Debug Frontend",
      "type": "node",
      "request": "launch",
      "program": "${workspaceFolder}/node_modules/.bin/vite",
      "args": ["dev"],
      "console": "integratedTerminal",
      "env": {
        "NODE_ENV": "development"
      }
    },
    {
      "name": "Debug Game Server",
      "type": "node",
      "request": "launch",
      "program": "deno",
      "args": ["run", "--inspect-brk", "--allow-all", "server/game/index.ts"],
      "console": "integratedTerminal"
    }
  ]
}
```

### Monitoring and Alerting

```typescript
// Error reporting service integration
class ErrorReporter {
  static report(error: Error, context?: any) {
    if (process.env.NODE_ENV === 'production') {
      // Send to error reporting service
      console.error('Error reported:', { error: error.message, context });
    } else {
      console.error('Development error:', error, context);
    }
  }

  static reportPerformanceIssue(metric: string, value: number, threshold: number) {
    if (value > threshold) {
      console.warn(`Performance issue detected: ${metric} = ${value} (threshold: ${threshold})`);
    }
  }
}
```

This comprehensive debugging guide provides the tools and techniques needed to effectively debug all components of the Dhaniverse project. Regular use of these debugging practices will help maintain code quality and quickly resolve issues as they arise.