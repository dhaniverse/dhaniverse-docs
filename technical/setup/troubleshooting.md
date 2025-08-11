# Troubleshooting Guide

This guide covers common setup issues and their solutions for the Dhaniverse project.

## General Troubleshooting Steps

Before diving into specific issues, try these general troubleshooting steps:

1. **Restart all services** - Stop and restart all development servers
2. **Clear caches** - Clear npm, Deno, and browser caches
3. **Check logs** - Review console output and error logs
4. **Verify environment** - Ensure all environment variables are set correctly
5. **Update dependencies** - Make sure all dependencies are up to date

## Installation Issues

### Node.js and npm Issues

#### Problem: `npm install` fails with permission errors
```bash
Error: EACCES: permission denied, mkdir '/usr/local/lib/node_modules'
```

**Solution (macOS/Linux):**
```bash
# Fix npm permissions
sudo chown -R $(whoami) ~/.npm
sudo chown -R $(whoami) /usr/local/lib/node_modules

# Or use a Node version manager (recommended)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
nvm install node
nvm use node
```

**Solution (Windows):**
```bash
# Run PowerShell as Administrator
# Or use Chocolatey/Scoop for package management
```

#### Problem: Node.js version compatibility issues
```bash
Error: This package requires Node.js version 18.0.0 or higher
```

**Solution:**
```bash
# Check current version
node --version

# Update Node.js
# Download from nodejs.org or use version manager
nvm install 18
nvm use 18
```

### Deno Installation Issues

#### Problem: Deno command not found
```bash
deno: command not found
```

**Solution:**
```bash
# Add Deno to PATH (macOS/Linux)
echo 'export PATH="$HOME/.deno/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc

# Windows - Add to system PATH
# %USERPROFILE%\.deno\bin
```

#### Problem: Deno permission errors
```bash
Error: Requires net access to "deno.land"
```

**Solution:**
```bash
# Grant necessary permissions
deno run --allow-net --allow-env --allow-read script.ts

# Or use -A for all permissions (development only)
deno run -A script.ts
```

### Rust and Cargo Issues

#### Problem: Rust installation fails
```bash
error: could not download file from 'https://forge.rust-lang.org/...'
```

**Solution:**
```bash
# Use alternative installation method
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# Or download from rust-lang.org manually
# Add to PATH after installation
source ~/.cargo/env
```

#### Problem: Cargo build fails with linker errors
```bash
error: linking with `cc` failed: exit status: 1
```

**Solution (macOS):**
```bash
# Install Xcode command line tools
xcode-select --install
```

**Solution (Linux):**
```bash
# Install build essentials
sudo apt-get update
sudo apt-get install build-essential
```

**Solution (Windows):**
```bash
# Install Visual Studio Build Tools
# Or use rustup-init.exe with MSVC toolchain
```

### DFX Installation Issues

#### Problem: DFX installation fails
```bash
error: failed to download dfx
```

**Solution:**
```bash
# Try alternative installation
DFX_VERSION=0.15.0 sh -ci "$(curl -fsSL https://sdk.dfinity.org/install.sh)"

# Or download manually from GitHub releases
# https://github.com/dfinity/sdk/releases
```

#### Problem: DFX version conflicts
```bash
Error: dfx version 0.14.0 is not supported
```

**Solution:**
```bash
# Update DFX
dfx upgrade

# Or install specific version
DFX_VERSION=0.15.0 sh -ci "$(curl -fsSL https://sdk.dfinity.org/install.sh)"
```

## Database Connection Issues

### MongoDB Atlas Issues

#### Problem: Connection timeout
```bash
Error: connection timed out
```

**Solution:**
1. **Check IP whitelist** - Add your current IP to MongoDB Atlas
2. **Verify connection string** - Ensure correct username, password, and cluster name
3. **Check firewall** - Ensure port 27017 is not blocked
4. **Test connection:**
   ```bash
   # Test with mongosh
   mongosh "mongodb+srv://cluster.mongodb.net/dhaniverse" --username your_username
   ```

#### Problem: Authentication failed
```bash
Error: Authentication failed
```

**Solution:**
1. **Verify credentials** - Check username and password in connection string
2. **Check database user permissions** - Ensure user has read/write access
3. **URL encode special characters** - Encode @ % : characters in password
   ```bash
   # Example: password with @ symbol
   # Original: user@123
   # Encoded: user%40123
   MONGODB_URI=mongodb+srv://username:user%40123@cluster.mongodb.net/dhaniverse
   ```

### Local MongoDB Issues

#### Problem: MongoDB service not running
```bash
Error: connect ECONNREFUSED 127.0.0.1:27017
```

**Solution (macOS):**
```bash
# Start MongoDB service
brew services start mongodb-community

# Or start manually
mongod --config /usr/local/etc/mongod.conf
```

**Solution (Linux):**
```bash
# Start MongoDB service
sudo systemctl start mongod
sudo systemctl enable mongod

# Check status
sudo systemctl status mongod
```

**Solution (Windows):**
```bash
# Start MongoDB service
net start MongoDB

# Or from Services panel
# Services -> MongoDB -> Start
```

## Server Startup Issues

### Port Conflicts

#### Problem: Port already in use
```bash
Error: listen EADDRINUSE: address already in use :::5173
```

**Solution:**
```bash
# Find process using port (macOS/Linux)
lsof -i :5173
kill -9 <PID>

# Find process using port (Windows)
netstat -ano | findstr :5173
taskkill /PID <PID> /F

# Or use different port
PORT=3001 npm run dev
```

### Environment Variable Issues

#### Problem: Missing environment variables
```bash
Error: Missing required environment variable: JWT_SECRET
```

**Solution:**
1. **Check .env file exists** - Ensure `.env` files are created from templates
2. **Verify variable names** - Check for typos in variable names
3. **Check file location** - Ensure `.env` files are in correct directories
4. **Test environment loading:**
   ```bash
   # Test environment variables
   cd server/game
   deno run --allow-env -e "console.log(Deno.env.get('JWT_SECRET'))"
   ```

#### Problem: Environment variables not loading
```bash
Error: Cannot read properties of undefined
```

**Solution:**
```bash
# Ensure .env file is in correct location
ls -la server/game/.env
ls -la server/ws/.env

# Check file permissions
chmod 644 server/game/.env
chmod 644 server/ws/.env

# Verify Deno is loading env file
deno run --allow-env --env-file=.env script.ts
```

## ICP Canister Issues

### DFX Replica Issues

#### Problem: DFX replica fails to start
```bash
Error: Failed to start the replica
```

**Solution:**
```bash
# Clean DFX state
dfx stop
rm -rf .dfx
dfx start --clean --background

# Check port availability
lsof -i :4943

# Try different port
dfx start --host 127.0.0.1:8080 --background
```

#### Problem: Canister deployment fails
```bash
Error: Failed to build canister
```

**Solution:**
```bash
# Check Rust installation
rustc --version
cargo --version

# Clean and rebuild
cd packages/icp-canister
cargo clean
dfx build
dfx deploy --network local

# Check for compilation errors
cargo check
```

### Candid Interface Issues

#### Problem: Candid file generation fails
```bash
Error: Failed to generate candid file
```

**Solution:**
```bash
# Regenerate candid file
cd packages/icp-canister
cargo run --bin generate_candid > rust_icp_canister.did

# Verify candid syntax
dfx canister metadata dhaniverse_backend candid:service
```

## Frontend Issues

### Vite Development Server Issues

#### Problem: Vite server fails to start
```bash
Error: Failed to resolve import
```

**Solution:**
```bash
# Clear Vite cache
rm -rf node_modules/.vite
npm run dev

# Check for TypeScript errors
npx tsc --noEmit

# Verify imports
npm run build
```

#### Problem: Module resolution errors
```bash
Error: Cannot resolve module './component'
```

**Solution:**
1. **Check file paths** - Verify relative import paths
2. **Check file extensions** - Ensure correct file extensions
3. **Check TypeScript config** - Verify `tsconfig.json` paths
4. **Clear cache and reinstall:**
   ```bash
   rm -rf node_modules
   rm package-lock.json
   npm install
   ```

### React/TypeScript Issues

#### Problem: TypeScript compilation errors
```bash
Error: Property 'x' does not exist on type 'y'
```

**Solution:**
1. **Check type definitions** - Ensure correct TypeScript types
2. **Update dependencies** - Update `@types/*` packages
3. **Check tsconfig.json** - Verify TypeScript configuration
4. **Restart TypeScript server** - In VS Code: Ctrl+Shift+P -> "TypeScript: Restart TS Server"

## Game-Specific Issues

### Phaser.js Issues

#### Problem: Game canvas not rendering
```bash
Error: WebGL context lost
```

**Solution:**
1. **Check browser support** - Ensure WebGL 2.0 support
2. **Update graphics drivers** - Update GPU drivers
3. **Try different browser** - Test in Chrome/Firefox
4. **Check console errors** - Look for WebGL-related errors

#### Problem: Asset loading failures
```bash
Error: Failed to load texture
```

**Solution:**
1. **Check file paths** - Verify asset file paths
2. **Check file permissions** - Ensure assets are accessible
3. **Check network tab** - Look for 404 errors in browser dev tools
4. **Verify asset formats** - Ensure supported image formats

### WebSocket Connection Issues

#### Problem: WebSocket connection fails
```bash
Error: WebSocket connection failed
```

**Solution:**
1. **Check WebSocket server** - Ensure server is running on correct port
2. **Check firewall** - Ensure WebSocket port is not blocked
3. **Check CORS settings** - Verify allowed origins
4. **Test connection:**
   ```bash
   # Test WebSocket endpoint
   curl -i -N -H "Connection: Upgrade" -H "Upgrade: websocket" \
        -H "Sec-WebSocket-Key: test" -H "Sec-WebSocket-Version: 13" \
        http://localhost:8001/ws
   ```

## Performance Issues

### Slow Build Times

#### Problem: npm install takes too long
```bash
# Taking several minutes to install
```

**Solution:**
```bash
# Use npm cache
npm cache clean --force
npm install --prefer-offline

# Use yarn instead
yarn install

# Use pnpm for faster installs
npm install -g pnpm
pnpm install
```

#### Problem: Vite build is slow
```bash
# Build taking too long
```

**Solution:**
```bash
# Enable build optimizations
# In vite.config.ts
export default defineConfig({
  build: {
    minify: 'esbuild',
    sourcemap: false,
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['react', 'react-dom'],
          phaser: ['phaser']
        }
      }
    }
  }
})
```

### Memory Issues

#### Problem: Out of memory errors
```bash
Error: JavaScript heap out of memory
```

**Solution:**
```bash
# Increase Node.js memory limit
export NODE_OPTIONS="--max-old-space-size=4096"
npm run build

# Or in package.json scripts
"build": "NODE_OPTIONS='--max-old-space-size=4096' vite build"
```

## Network Issues

### Firewall and Proxy Issues

#### Problem: Cannot connect to external services
```bash
Error: connect ETIMEDOUT
```

**Solution:**
1. **Check firewall settings** - Allow outbound connections
2. **Configure proxy** - Set HTTP_PROXY and HTTPS_PROXY if needed
3. **Check corporate network** - May need to whitelist domains
4. **Test connectivity:**
   ```bash
   # Test external connectivity
   curl -I https://registry.npmjs.org
   curl -I https://deno.land
   ```

### DNS Issues

#### Problem: Cannot resolve hostnames
```bash
Error: getaddrinfo ENOTFOUND
```

**Solution:**
```bash
# Flush DNS cache (macOS)
sudo dscacheutil -flushcache

# Flush DNS cache (Windows)
ipconfig /flushdns

# Try different DNS servers
# Use 8.8.8.8 or 1.1.1.1
```

## Getting Additional Help

### Diagnostic Information

When reporting issues, include:

```bash
# System information
node --version
npm --version
deno --version
rustc --version
dfx --version

# Operating system
uname -a  # macOS/Linux
systeminfo  # Windows

# Project information
git rev-parse HEAD
npm list --depth=0
```

### Log Collection

```bash
# Collect logs
npm run dev > logs/frontend.log 2>&1 &
npm run server:game > logs/game-server.log 2>&1 &
npm run server:ws > logs/ws-server.log 2>&1 &
dfx logs dhaniverse_backend > logs/canister.log 2>&1 &
```

### Support Channels

1. **GitHub Issues** - For bug reports and feature requests
2. **Documentation** - Check other documentation files
3. **Community Discord** - For real-time help
4. **Stack Overflow** - For general programming questions

### Creating Bug Reports

Include the following information:

1. **Environment details** - OS, Node.js version, etc.
2. **Steps to reproduce** - Exact steps that cause the issue
3. **Expected behavior** - What should happen
4. **Actual behavior** - What actually happens
5. **Error messages** - Full error messages and stack traces
6. **Configuration** - Relevant configuration files (sanitized)
7. **Logs** - Relevant log output

## Prevention Tips

### Regular Maintenance

1. **Keep dependencies updated** - Regular `npm update` and `cargo update`
2. **Monitor security advisories** - Check for security updates
3. **Clean build artifacts** - Regular cleanup of build directories
4. **Backup configurations** - Keep configuration templates updated
5. **Document changes** - Keep track of configuration changes

### Development Best Practices

1. **Use version managers** - nvm for Node.js, rustup for Rust
2. **Consistent environments** - Use Docker or similar for consistency
3. **Automated testing** - Set up CI/CD for early issue detection
4. **Code reviews** - Review configuration changes
5. **Monitoring** - Set up monitoring for production environments