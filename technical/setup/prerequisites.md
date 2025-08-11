# Prerequisites

This document outlines all system requirements and dependencies needed to run the Dhaniverse project locally.

## System Requirements

### Operating System
- **Windows**: Windows 10 or later
- **macOS**: macOS 10.15 (Catalina) or later
- **Linux**: Ubuntu 18.04+ or equivalent distribution

### Hardware Requirements
- **RAM**: Minimum 8GB, recommended 16GB
- **Storage**: At least 5GB free space
- **CPU**: Multi-core processor (Intel i5/AMD Ryzen 5 or better)
- **Network**: Stable internet connection for package downloads and blockchain operations

## Core Dependencies

### Node.js and npm
- **Node.js**: Version 18.0.0 or later
- **npm**: Version 8.0.0 or later (comes with Node.js)

**Installation:**
- Download from [nodejs.org](https://nodejs.org/)
- Verify installation: `node --version` and `npm --version`

### Deno Runtime
- **Deno**: Version 1.40.0 or later
- Required for game server and WebSocket server

**Installation:**
```bash
# Windows (PowerShell)
irm https://deno.land/install.ps1 | iex

# macOS/Linux
curl -fsSL https://deno.land/install.sh | sh

# Verify installation
deno --version
```

### Rust and Cargo
- **Rust**: Version 1.70.0 or later
- **Cargo**: Comes with Rust installation
- Required for ICP canister development

**Installation:**
```bash
# Install Rust via rustup
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# Add to PATH (restart terminal or run)
source ~/.cargo/env

# Verify installation
rustc --version
cargo --version
```

### DFX (DFINITY SDK)
- **DFX**: Version 0.15.0 or later
- Required for Internet Computer canister deployment

**Installation:**
```bash
# Install DFX
sh -ci "$(curl -fsSL https://sdk.dfinity.org/install.sh)"

# Verify installation
dfx --version
```

## Database Requirements

### MongoDB
You need access to a MongoDB database. Choose one of the following options:

#### Option 1: MongoDB Atlas (Recommended)
- Create a free account at [MongoDB Atlas](https://cloud.mongodb.com/)
- Create a cluster and database user
- Obtain connection string for configuration

#### Option 2: Local MongoDB Installation
- **MongoDB**: Version 5.0 or later
- Download from [mongodb.com](https://www.mongodb.com/try/download/community)
- Ensure MongoDB service is running

## Development Tools

### Git
- **Git**: Version 2.30.0 or later
- Required for version control and repository cloning

**Installation:**
- Download from [git-scm.com](https://git-scm.com/)
- Verify: `git --version`

### Code Editor (Recommended)
- **Visual Studio Code** with extensions:
  - TypeScript and JavaScript Language Features
  - Rust Analyzer
  - Deno extension
  - Candid Language Support

## Browser Requirements

### Supported Browsers
- **Chrome**: Version 90 or later (recommended)
- **Firefox**: Version 88 or later
- **Safari**: Version 14 or later
- **Edge**: Version 90 or later

### Required Browser Features
- WebGL 2.0 support
- WebSocket support
- Local Storage support
- ES2020 module support

## Network Requirements

### Ports
Ensure the following ports are available:
- **3000**: Frontend development server (Vite)
- **4943**: DFX local replica
- **8000**: Game server (Deno)
- **8001**: WebSocket server (Deno)

### Firewall Configuration
- Allow outbound connections to:
  - npm registry (registry.npmjs.org)
  - Deno registry (deno.land)
  - MongoDB Atlas (if using cloud database)
  - Internet Computer network (ic0.app)

## Optional Dependencies

### Wallet Integration
For blockchain features, you may need:
- **Internet Identity**: ICP authentication
- **Plug Wallet**: Browser extension for ICP
- **MetaMask**: For Ethereum-based features (future)

### Development Utilities
- **Tiled Map Editor**: For map editing (if contributing maps)
- **MongoDB Compass**: GUI for database management
- **Postman**: API testing tool

## Verification Checklist

Before proceeding to setup, verify you have:

- [ ] Node.js 18+ installed and accessible
- [ ] Deno runtime installed and in PATH
- [ ] Rust and Cargo installed
- [ ] DFX installed and configured
- [ ] Git installed and configured
- [ ] MongoDB access (Atlas or local)
- [ ] Supported browser installed
- [ ] Required ports available
- [ ] Stable internet connection

## Troubleshooting Common Issues

### Permission Issues
- **Windows**: Run terminal as Administrator if needed
- **macOS/Linux**: Use `sudo` for system-wide installations
- **Node modules**: Never use `sudo` with npm in project directory

### Path Issues
- Ensure all tools are added to system PATH
- Restart terminal after installations
- Check PATH with `echo $PATH` (Unix) or `echo %PATH%` (Windows)

### Version Conflicts
- Use Node Version Manager (nvm) for Node.js version management
- Use rustup for Rust toolchain management
- Keep DFX updated with `dfx upgrade`

## Next Steps

Once all prerequisites are installed and verified, proceed to the [Local Setup Guide](local-setup.md) for detailed installation instructions.