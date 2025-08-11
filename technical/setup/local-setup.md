# Local Development Setup

## Table of Contents

- [Prerequisites](#prerequisites)
- [Project Setup](#project-setup)
- [Environment Configuration](#environment-configuration)
- [Database Setup](#database-setup)
- [ICP Canister Setup](#icp-canister-setup)
- [Server Setup](#server-setup)
- [Frontend Setup](#frontend-setup)
- [Verification](#verification)
- [Troubleshooting](#troubleshooting)

This guide provides step-by-step instructions for setting up the Dhaniverse project in your local development environment.

## Prerequisites

Before starting, ensure you have completed all requirements listed in [Prerequisites](./prerequisites.md).

## Project Setup

### 1. Clone the Repository

```bash
# Clone the repository
git clone https://github.com/Gursimrxn/dhaniverse.git
cd dhaniverse

# Verify project structure
ls -la
```

### 2. Install Dependencies

#### Install Node.js Dependencies
```bash
# Install root dependencies
npm install

# Install map-optimizer package dependencies
cd packages/map-optimizer
npm install
cd ../..
```

#### Verify Rust Installation
```bash
# Navigate to ICP canister directory
cd packages/icp-canister

# Verify Rust dependencies
cargo check

# Return to root
cd ../..
```

### 3. Environment Configuration

#### Game Server Environment
```bash
# Navigate to game server directory
cd server/game

# Copy environment template
cp .env.example .env

# Edit environment variables
# Use your preferred editor (nano, vim, code, etc.)
nano .env
```

Configure the following variables in `server/game/.env`:
```bash
# Database Configuration
MONGODB_URI=mongodb+srv://username:password@cluster.mongodb.net/dhaniverse?retryWrites=true&w=majority

# JWT Secret (generate a secure random string)
JWT_SECRET=your-super-secure-jwt-secret-key-here

# Server Configuration
NODE_ENV=development
DENO_ENV=development
PORT=8000

# CORS Configuration
ALLOWED_ORIGINS=http://localhost:3000,http://localhost:5173,http://localhost:4173
```

#### WebSocket Server Environment
```bash
# Navigate to WebSocket server directory
cd ../ws

# Copy environment template
cp .env.example .env

# Edit environment variables
nano .env
```

Configure the following variables in `server/ws/.env`:
```bash
# Server configuration
PORT=8001
DENO_ENV=development
SERVER_DOMAIN=localhost

# JWT configuration (must match game server)
JWT_SECRET=your-super-secure-jwt-secret-key-here

# CORS settings
ALLOWED_ORIGINS=http://localhost:3000,http://localhost:5173,http://localhost:4173

# Auth server URLs
AUTH_SERVER_URL=http://localhost:8000
PRODUCTION_AUTH_SERVER_URL=https://dhaniverseapi.deno.dev
```

Return to project root:
```bash
cd ../..
```

### 4. Database Setup

#### Option A: MongoDB Atlas (Recommended)

1. **Create MongoDB Atlas Account**
   - Visit [MongoDB Atlas](https://cloud.mongodb.com/)
   - Sign up for a free account
   - Create a new cluster

2. **Configure Database Access**
   - Create a database user with read/write permissions
   - Add your IP address to the IP whitelist
   - Note down the connection string

3. **Update Environment Variables**
   - Replace the `MONGODB_URI` in both `.env` files with your Atlas connection string
   - Ensure the database name is set to `dhaniverse`

#### Option B: Local MongoDB

1. **Start MongoDB Service**
   ```bash
   # macOS (with Homebrew)
   brew services start mongodb-community

   # Linux (systemd)
   sudo systemctl start mongod

   # Windows
   # Start MongoDB service from Services panel or command line
   ```

2. **Update Environment Variables**
   ```bash
   # Use local connection string
   MONGODB_URI=mongodb://localhost:27017/dhaniverse
   ```

### 5. ICP Canister Setup

#### Start Local ICP Replica
```bash
# Start DFX in background
dfx start --background

# Verify DFX is running
dfx ping
```

#### Deploy Canister Locally
```bash
# Navigate to canister directory
cd packages/icp-canister

# Build the canister
dfx build

# Deploy to local replica
dfx deploy --network local

# Verify deployment
dfx canister status dhaniverse_backend --network local

# Return to root
cd ../..
```

### 6. Build and Start Services

#### Build Frontend Assets
```bash
# Build the frontend for development
npm run build
```

#### Start Development Servers

Open three separate terminal windows/tabs:

**Terminal 1: Frontend Development Server**
```bash
# Start Vite development server
npm run dev

# Server will start on http://localhost:5173
```

**Terminal 2: Game Server**
```bash
# Start Deno game server
npm run server:game

# Server will start on http://localhost:8000
```

**Terminal 3: WebSocket Server**
```bash
# Start WebSocket server
npm run server:ws

# Server will start on http://localhost:8001
```

Alternatively, use the batch script (Windows):
```bash
# Start all servers simultaneously
npm run server
```

### 7. Verification

#### Test Frontend Access
1. Open browser and navigate to `http://localhost:5173`
2. Verify the landing page loads correctly
3. Check browser console for any errors

#### Test API Endpoints
```bash
# Test game server health
curl http://localhost:8000/health

# Test WebSocket connection (if available)
curl http://localhost:8001/health
```

#### Test Database Connection
1. Check server logs for successful MongoDB connection
2. Verify no authentication errors in console

#### Test ICP Integration
1. Access the game and try wallet connection features
2. Check DFX logs: `dfx logs dhaniverse_backend`

## Development Workflow

### Making Changes

#### Frontend Development
- Edit files in `src/` directory
- Vite will automatically reload changes
- Check browser console for TypeScript errors

#### Backend Development
- Edit files in `server/game/src/` or `server/ws/`
- Deno will automatically restart with `--watch` flag
- Check terminal output for errors

#### ICP Canister Development
- Edit files in `packages/icp-canister/src/`
- Rebuild and redeploy:
  ```bash
  cd packages/icp-canister
  dfx build
  dfx deploy --network local
  cd ../..
  ```

### Testing Changes

#### Run Frontend Tests
```bash
# If tests are available
npm run test
```

#### Test API Endpoints
```bash
# Test specific endpoints
curl -X POST http://localhost:8000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"test","password":"test"}'
```

#### Test Canister Functions
```bash
# Test canister methods
dfx canister call dhaniverse_backend get_balance '()'
```

## Common Development Tasks

### Updating Dependencies
```bash
# Update Node.js dependencies
npm update

# Update Deno dependencies
cd server/game
deno cache --reload index.ts
cd ../ws
deno cache --reload ws.ts
cd ../..

# Update Rust dependencies
cd packages/icp-canister
cargo update
cd ../..
```

### Resetting Development Environment
```bash
# Stop all servers (Ctrl+C in each terminal)

# Clean build artifacts
npm run clean  # if available
rm -rf node_modules
rm -rf dist

# Clean DFX state
dfx stop
dfx start --clean --background

# Reinstall and rebuild
npm install
npm run build
dfx deploy --network local
```

### Database Management
```bash
# Connect to MongoDB (if local)
mongosh dhaniverse

# View collections
show collections

# Clear test data (if needed)
db.users.deleteMany({})
db.games.deleteMany({})
```

## Troubleshooting

### Port Conflicts
If ports are already in use:
```bash
# Find process using port
lsof -i :5173  # macOS/Linux
netstat -ano | findstr :5173  # Windows

# Kill process if needed
kill -9 <PID>  # macOS/Linux
taskkill /PID <PID> /F  # Windows
```

### Permission Issues
```bash
# Fix npm permissions (macOS/Linux)
sudo chown -R $(whoami) ~/.npm

# Fix Deno permissions
deno cache --reload --allow-all index.ts
```

### DFX Issues
```bash
# Reset DFX completely
dfx stop
rm -rf .dfx
dfx start --clean --background
dfx deploy --network local
```

## Next Steps

Once your local environment is running successfully:

1. Review the [Configuration Guide](configuration.md) for advanced settings
2. Check the [Development Workflow](../development/development-workflow.md) for contribution guidelines
3. Explore the [API Documentation](../api/) for integration details
4. See [Troubleshooting Guide](troubleshooting.md) for common issues

## Getting Help

If you encounter issues not covered in this guide:

1. Check the [Troubleshooting Guide](troubleshooting.md)
2. Review existing GitHub issues
3. Create a new issue with detailed error information
4. Join the community Discord for real-time help## Relate
d Documentation

- [Prerequisites](./prerequisites.md) - System requirements and dependencies
- [Configuration](./configuration.md) - Environment variables and configuration files
- [Troubleshooting](./troubleshooting.md) - Common setup issues and solutions
- [Development Workflow](../development/development-workflow.md) - Development process and Git workflow
- [Local Deployment](../deployment/local-deployment.md) - Local deployment procedures

## Next Steps

After completing the local setup:

1. **Start Development** - Follow the [Development Workflow](../development/development-workflow.md)
2. **Run Tests** - See [Testing Guide](../development/testing.md)
3. **Deploy Locally** - Follow [Local Deployment](../deployment/local-deployment.md)
4. **Explore APIs** - Check [API Documentation](../api/)

---

**Navigation**: [← Setup Index](./index.md) | [Prerequisites](./prerequisites.md) | [Configuration →](./configuration.md)