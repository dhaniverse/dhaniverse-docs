# Development Documentation

This section covers development workflows, testing strategies, coding standards, and debugging procedures for contributing to the Dhaniverse project.

## Table of Contents

- [Development Workflow](./development-workflow.md) - Git workflow and development processes
- [Testing](./testing.md) - Testing strategies and procedures
- [Coding Standards](./coding-standards.md) - Code style and best practices
- [Debugging](./debugging.md) - Debugging guides and tools
- [Documentation Review Checklist](./documentation-review-checklist.md) - Quality standards for documentation
- [Documentation Update Procedures](./documentation-update-procedures.md) - Keeping docs synchronized with code

## Development Overview

### Project Structure
The Dhaniverse project follows a modular architecture with clear separation of concerns:
- **Frontend**: React with TypeScript and Phaser.js
- **Backend**: Deno servers with Oak framework
- **Blockchain**: ICP canisters written in Rust
- **Packages**: Reusable TypeScript packages

### Development Principles
- **Type Safety**: Comprehensive TypeScript usage
- **Testing**: Test-driven development approach
- **Code Quality**: Automated linting and formatting
- **Documentation**: Code documentation and API specs
- **Security**: Security-first development practices

## Getting Started with Development

### First-Time Contributors
1. **Read the Workflow** - Understand our Git workflow and branching strategy
2. **Set Up Environment** - Follow the [Local Setup Guide](../setup/local-setup.md)
3. **Review Standards** - Familiarize yourself with our coding standards
4. **Run Tests** - Ensure all tests pass before making changes

### Regular Contributors
- **Stay Updated** - Pull latest changes regularly
- **Follow Conventions** - Maintain consistency with existing code
- **Write Tests** - Include tests for new features and bug fixes
- **Document Changes** - Update documentation for significant changes

## Development Workflow Summary

### Branch Strategy
- `main` - Production-ready code
- `develop` - Integration branch for features
- `feature/*` - Individual feature branches
- `hotfix/*` - Critical bug fixes

### Code Review Process
1. Create feature branch from `develop`
2. Implement changes with tests
3. Create pull request with description
4. Address review feedback
5. Merge after approval

## Quick Navigation

### Related Documentation
- [Setup & Configuration](../setup/) - Environment setup and configuration
- [API Documentation](../api/) - API endpoints and integration guides
- [Architecture](../architecture/) - System design and component relationships
- [Components](../components/) - Individual component documentation

### Development Tools
- [TypeScript Handbook](https://www.typescriptlang.org/docs/)
- [React Testing Library](https://testing-library.com/docs/react-testing-library/intro/)
- [Deno Testing](https://deno.land/manual/testing)
- [Rust Testing](https://doc.rust-lang.org/book/ch11-00-testing.html)

### Code Quality Tools
- ESLint configuration
- Prettier formatting
- Husky git hooks
- GitHub Actions CI/CD

---

[← Back to Main Documentation](../README.md)