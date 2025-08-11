# Configuration Guide

This document provides comprehensive information about all configuration files, environment variables, and settings used in the Dhaniverse project.

## Configuration Files Overview

The project uses multiple configuration files for different components:

- **Frontend**: `vite.config.ts`, `tsconfig.json`, `tailwind.config.ts`
- **Game Server**: `server/game/.env`, `server/game/deno.json`
- **WebSocket Server**: `server/ws/.env`
- **ICP Canister**: `dfx.json`, `packages/icp-canister/Cargo.toml`
- **Map Optimizer**: `packages/map-optimizer/package.json`, `packages/map-optimizer/tsconfig.json`
- **Root Project**: `package.json`, `tsconfig.json`

## Environment Variables

### Game Server Configuration (`server/game/.env`)

#### Database Configuration
```bash
# MongoDB connection string
MONGODB_URI=mongodb+srv://username:password@cluster.mongodb.net/dhaniverse?retryWrites=true&w=majority
# Local alternative: mongodb://localhost:27017/dhaniverse
```

**Description**: MongoDB database connection string. Use MongoDB Atlas for production or local MongoDB for development.

**Required**: Yes

**Format**: Standard MongoDB connection URI

#### Authentication Configuration
```bash
# JWT secret for token signing
JWT_SECRET=your-super-secure-jwt-secret-key-here
```

**Description**: Secret key used for signing and verifying JWT tokens. Must be the same across all services.

**Required**: Yes

**Security**: Use a strong, randomly generated string (minimum 32 characters)

**Generation**: 
```bash
# Generate secure JWT secret
node -e "console.log(require('crypto').randomBytes(64).toString('hex'))"
```

#### Server Configuration
```bash
# Environment mode
NODE_ENV=development
DENO_ENV=development

# Server port
PORT=8000
```

**NODE_ENV**: Determines the runtime environment
- `development`: Development mode with verbose logging
- `production`: Production mode with optimized performance
- `test`: Testing mode with test-specific configurations

**DENO_ENV**: Deno-specific environment variable
- `development`: Development mode
- `production`: Production mode

**PORT**: Port number for the game server (default: 8000)

#### CORS Configuration
```bash
# Allowed origins for CORS
ALLOWED_ORIGINS=http://localhost:3000,http://localhost:5173,http://localhost:4173
```

**Description**: Comma-separated list of allowed origins for Cross-Origin Resource Sharing (CORS).

**Development**: Include all local development server URLs
**Production**: Include only production domain URLs

### WebSocket Server Configuration (`server/ws/.env`)

#### Server Configuration
```bash
# WebSocket server port
PORT=8001

# Environment
DENO_ENV=development

# Server domain
SERVER_DOMAIN=localhost
```

**PORT**: Port number for WebSocket server (default: 8001)

**SERVER_DOMAIN**: Domain name for the server (localhost for development)

#### Authentication Configuration
```bash
# JWT secret (must match game server)
JWT_SECRET=your-super-secure-jwt-secret-key-here
```

**Description**: Must be identical to the game server JWT_SECRET for token verification.

#### CORS Configuration
```bash
# Allowed origins
ALLOWED_ORIGINS=http://localhost:3000,http://localhost:5173,http://localhost:4173
```

**Description**: Same as game server CORS configuration.

#### Service URLs
```bash
# Auth server URLs
AUTH_SERVER_URL=http://localhost:8000
PRODUCTION_AUTH_SERVER_URL=https://dhaniverseapi.deno.dev
```

**AUTH_SERVER_URL**: URL of the game server for development
**PRODUCTION_AUTH_SERVER_URL**: Production URL of the game server

## Configuration Files

### Frontend Configuration

#### Vite Configuration (`vite.config.ts`)
```typescript
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  server: {
    port: 5173,
    host: true
  },
  build: {
    outDir: 'dist',
    sourcemap: true
  }
})
```

**Key Settings**:
- **port**: Development server port (5173)
- **host**: Allow external connections
- **outDir**: Build output directory
- **sourcemap**: Generate source maps for debugging

#### TypeScript Configuration (`tsconfig.json`)
```json
{
  "compilerOptions": {
    "target": "ES2020",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true
  },
  "include": ["src"],
  "references": [{ "path": "./tsconfig.node.json" }]
}
```

#### Tailwind Configuration (`tailwind.config.ts`)
```typescript
import type { Config } from 'tailwindcss'

const config: Config = {
  content: [
    "./index.html",
    "./src/**/*.{js,ts,jsx,tsx}",
  ],
  theme: {
    extend: {
      // Custom theme extensions
    },
  },
  plugins: [],
}

export default config
```

### Game Server Configuration

#### Deno Configuration (`server/game/deno.json`)
```json
{
  "tasks": {
    "dev": "deno run --allow-net --allow-env --allow-read --env-file=.env --watch index.ts",
    "start": "deno run --allow-net --allow-env --allow-read --env-file=.env index.ts"
  },
  "imports": {
    "@std/assert": "jsr:@std/assert@1",
    "oak": "https://deno.land/x/oak@v17.1.5/mod.ts",
    "cors": "https://deno.land/x/cors@v1.2.2/mod.ts",
    "djwt": "https://deno.land/x/djwt@v3.0.1/mod.ts",
    "mongodb": "npm:mongodb@5.6.0"
  },
  "compilerOptions": {
    "strict": true,
    "strictNullChecks": true
  }
}
```

**Key Settings**:
- **tasks**: Predefined Deno tasks for development and production
- **imports**: Import map for external dependencies
- **compilerOptions**: TypeScript compiler options

### ICP Canister Configuration

#### DFX Configuration (`dfx.json`)
```json
{
  "version": 1,
  "canisters": {
    "dhaniverse_backend": {
      "type": "rust",
      "package": "rust-icp-canister",
      "candid": "packages/icp-canister/rust_icp_canister.did",
      "wasm": "target/wasm32-unknown-unknown/release/rust_icp_canister.wasm"
    },
    "dhaniverse_frontend": {
      "type": "assets",
      "source": ["dist/"],
      "dependencies": ["dhaniverse_backend"]
    }
  },
  "networks": {
    "local": {
      "bind": "127.0.0.1:4943",
      "type": "ephemeral"
    },
    "ic": {
      "providers": ["https://ic0.app"],
      "type": "persistent"
    }
  }
}
```

**Key Settings**:
- **canisters**: Defines backend (Rust) and frontend (assets) canisters
- **networks**: Local development and IC mainnet configurations
- **bind**: Local replica binding address and port

#### Cargo Configuration (`packages/icp-canister/Cargo.toml`)
```toml
[package]
name = "rust-icp-canister"
version = "0.1.0"
edition = "2021"

[lib]
crate-type = ["cdylib"]

[dependencies]
ic-cdk = "0.13"
ic-cdk-macros = "0.9"
candid = { version = "0.10", features = ["value"] }
serde = { version = "1.0", features = ["derive"] }
```

**Key Settings**:
- **crate-type**: Compiled as C dynamic library for WASM
- **dependencies**: ICP SDK and required crates

## Advanced Configuration

### Database Configuration

#### MongoDB Connection Options
```bash
# Full connection string with options
MONGODB_URI=mongodb+srv://username:password@cluster.mongodb.net/dhaniverse?retryWrites=true&w=majority&maxPoolSize=10&serverSelectionTimeoutMS=5000
```

**Connection Parameters**:
- `retryWrites=true`: Enable retryable writes
- `w=majority`: Write concern for data durability
- `maxPoolSize=10`: Maximum connection pool size
- `serverSelectionTimeoutMS=5000`: Server selection timeout

#### Local MongoDB Configuration
```bash
# Local MongoDB with authentication
MONGODB_URI=mongodb://username:password@localhost:27017/dhaniverse?authSource=admin

# Local MongoDB without authentication
MONGODB_URI=mongodb://localhost:27017/dhaniverse
```

### Security Configuration

#### JWT Configuration
```bash
# JWT token expiration (in seconds)
JWT_EXPIRES_IN=3600

# JWT algorithm
JWT_ALGORITHM=HS256

# Refresh token expiration
REFRESH_TOKEN_EXPIRES_IN=604800
```

#### CORS Configuration
```bash
# Development CORS (permissive)
ALLOWED_ORIGINS=http://localhost:3000,http://localhost:5173,http://localhost:4173,http://127.0.0.1:3000

# Production CORS (restrictive)
ALLOWED_ORIGINS=https://dhaniverse.vercel.app,https://dhaniverse.com
```

### Performance Configuration

#### Server Performance
```bash
# Request timeout (milliseconds)
REQUEST_TIMEOUT=30000

# Maximum request body size (bytes)
MAX_BODY_SIZE=1048576

# Rate limiting
RATE_LIMIT_WINDOW=900000
RATE_LIMIT_MAX_REQUESTS=100
```

#### Database Performance
```bash
# Connection pool settings
DB_MIN_POOL_SIZE=5
DB_MAX_POOL_SIZE=20
DB_MAX_IDLE_TIME=30000
```

### Logging Configuration

#### Log Levels
```bash
# Log level (error, warn, info, debug)
LOG_LEVEL=info

# Enable request logging
ENABLE_REQUEST_LOGGING=true

# Log format (json, text)
LOG_FORMAT=json
```

## Environment-Specific Configurations

### Development Environment
```bash
NODE_ENV=development
DENO_ENV=development
LOG_LEVEL=debug
ENABLE_REQUEST_LOGGING=true
MONGODB_URI=mongodb://localhost:27017/dhaniverse
ALLOWED_ORIGINS=http://localhost:3000,http://localhost:5173,http://localhost:4173
```

### Production Environment
```bash
NODE_ENV=production
DENO_ENV=production
LOG_LEVEL=warn
ENABLE_REQUEST_LOGGING=false
MONGODB_URI=mongodb+srv://prod_user:secure_password@prod-cluster.mongodb.net/dhaniverse?retryWrites=true&w=majority
ALLOWED_ORIGINS=https://dhaniverse.vercel.app
```

### Testing Environment
```bash
NODE_ENV=test
DENO_ENV=test
LOG_LEVEL=error
MONGODB_URI=mongodb://localhost:27017/dhaniverse_test
JWT_SECRET=test-jwt-secret-key
```

## Configuration Validation

### Environment Variable Validation
The application validates required environment variables on startup:

```typescript
// Required variables
const requiredVars = [
  'MONGODB_URI',
  'JWT_SECRET',
  'NODE_ENV'
];

// Validation function
function validateEnvironment() {
  for (const variable of requiredVars) {
    if (!Deno.env.get(variable)) {
      throw new Error(`Missing required environment variable: ${variable}`);
    }
  }
}
```

### Configuration Schema
```typescript
interface ServerConfig {
  mongodb: {
    uri: string;
    options: MongoClientOptions;
  };
  jwt: {
    secret: string;
    expiresIn: string;
    algorithm: string;
  };
  server: {
    port: number;
    cors: {
      origins: string[];
    };
  };
}
```

## Troubleshooting Configuration Issues

### Common Configuration Problems

#### Invalid MongoDB URI
```bash
# Error: Invalid connection string
# Solution: Check URI format and credentials
MONGODB_URI=mongodb+srv://user:pass@cluster.mongodb.net/db
```

#### JWT Secret Issues
```bash
# Error: JWT secret too short
# Solution: Use minimum 32 character secret
JWT_SECRET=$(openssl rand -hex 32)
```

#### Port Conflicts
```bash
# Error: Port already in use
# Solution: Change port or kill existing process
PORT=8080  # Use different port
```

#### CORS Issues
```bash
# Error: CORS policy violation
# Solution: Add frontend URL to allowed origins
ALLOWED_ORIGINS=http://localhost:3000,http://localhost:5173
```

### Configuration Testing

#### Test Database Connection
```bash
# Test MongoDB connection
deno run --allow-net --allow-env test-db-connection.ts
```

#### Test Environment Variables
```bash
# Print all environment variables
deno run --allow-env -e "console.log(Deno.env.toObject())"
```

#### Validate Configuration
```bash
# Run configuration validation
npm run validate-config
```

## Best Practices

### Security Best Practices
1. Never commit `.env` files to version control
2. Use strong, randomly generated JWT secrets
3. Rotate secrets regularly in production
4. Use environment-specific configurations
5. Validate all configuration inputs

### Performance Best Practices
1. Use connection pooling for databases
2. Set appropriate timeouts
3. Configure rate limiting
4. Use caching where appropriate
5. Monitor configuration performance impact

### Maintenance Best Practices
1. Document all configuration changes
2. Use configuration templates
3. Validate configurations in CI/CD
4. Monitor configuration drift
5. Regular security audits

## Next Steps

After configuring your environment:

1. Test all configurations with [Local Setup Guide](local-setup.md)
2. Review [Troubleshooting Guide](troubleshooting.md) for common issues
3. Check [Development Workflow](../development/development-workflow.md) for development practices
4. Explore [API Documentation](../api/) for integration details