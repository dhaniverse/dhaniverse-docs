# Development Workflow

## Table of Contents

- [Overview](#overview)
- [Git Workflow](#git-workflow)
- [Branching Strategy](#branching-strategy)
- [Code Review Process](#code-review-process)
- [Testing Requirements](#testing-requirements)
- [Documentation Standards](#documentation-standards)
- [Release Process](#release-process)
- [Collaboration Guidelines](#collaboration-guidelines)

## Overview

This document outlines the development workflow for the Dhaniverse project, including Git workflow, branching strategy, code review process, and collaboration guidelines. The workflow is designed to maintain code quality, enable parallel development, and ensure smooth integration of features.

## Git Workflow

### Repository Structure

The Dhaniverse project follows a monorepo structure with the following main components:

- **Frontend**: React/TypeScript application (`src/`)
- **ICP Canister**: Rust backend (`packages/icp-canister/`)
- **Game Server**: Deno/TypeScript server (`server/game/`)
- **WebSocket Server**: Deno/TypeScript server (`server/ws/`)
- **Map Optimizer**: TypeScript utility package (`packages/map-optimizer/`)

### Branching Strategy

We use a Git Flow-inspired branching model with the following branch types:

#### Main Branches

- **`main`**: Production-ready code
  - Always deployable
  - Protected branch requiring pull request reviews
  - Automatic deployment to production environment

- **`develop`**: Integration branch for features
  - Latest development changes
  - Base branch for feature branches
  - Continuous integration testing

#### Supporting Branches

- **Feature branches**: `feature/[issue-number]-[short-description]`
  - Created from `develop`
  - Merged back to `develop` via pull request
  - Example: `feature/123-wallet-integration`

- **Hotfix branches**: `hotfix/[issue-number]-[short-description]`
  - Created from `main` for critical production fixes
  - Merged to both `main` and `develop`
  - Example: `hotfix/456-security-patch`

- **Release branches**: `release/[version]`
  - Created from `develop` when preparing for release
  - Only bug fixes and release preparation commits
  - Merged to `main` and tagged with version

### Commit Message Convention

Follow the Conventional Commits specification:

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

#### Types

- **feat**: New feature
- **fix**: Bug fix
- **docs**: Documentation changes
- **style**: Code style changes (formatting, missing semicolons, etc.)
- **refactor**: Code refactoring without changing functionality
- **test**: Adding or modifying tests
- **chore**: Maintenance tasks, dependency updates

#### Examples

```
feat(wallet): add MetaMask connection support

Implement MetaMask wallet connector with error handling
and connection state management.

Closes #123
```

```
fix(canister): resolve authentication token validation

- Fix JWT token expiration check
- Add proper error handling for invalid tokens
- Update tests for edge cases

Fixes #456
```

## Code Review Process

### Pull Request Requirements

All code changes must go through the pull request process with the following requirements:

#### Before Creating a Pull Request

1. **Branch is up to date**: Rebase or merge latest `develop`
2. **Tests pass**: All automated tests must pass
3. **Code quality**: Linting and formatting checks pass
4. **Documentation**: Update relevant documentation
5. **Self-review**: Review your own changes first

#### Pull Request Template

```markdown
## Description
Brief description of changes and motivation.

## Type of Change
- [ ] Bug fix (non-breaking change which fixes an issue)
- [ ] New feature (non-breaking change which adds functionality)
- [ ] Breaking change (fix or feature that would cause existing functionality to not work as expected)
- [ ] Documentation update

## Testing
- [ ] Unit tests added/updated
- [ ] Integration tests added/updated
- [ ] Manual testing completed

## Checklist
- [ ] Code follows project coding standards
- [ ] Self-review completed
- [ ] Documentation updated
- [ ] No console.log or debug statements
- [ ] Error handling implemented
```

#### Review Process

1. **Automated Checks**: CI/CD pipeline runs automatically
   - TypeScript compilation
   - Linting (ESLint, Rust clippy)
   - Unit tests
   - Integration tests
   - Security scans

2. **Peer Review**: At least one team member review required
   - Code quality and standards compliance
   - Logic and implementation review
   - Security considerations
   - Performance implications

3. **Approval and Merge**: After approval
   - Squash and merge for feature branches
   - Merge commit for release branches
   - Delete feature branch after merge

### Review Guidelines

#### For Authors

- Keep pull requests focused and reasonably sized
- Provide clear description and context
- Respond to feedback promptly and professionally
- Test changes thoroughly before requesting review

#### For Reviewers

- Review within 24 hours when possible
- Focus on code quality, security, and maintainability
- Provide constructive feedback with suggestions
- Approve only when confident in the changes

## Development Environment Setup

### Prerequisites

Ensure you have the required tools installed as documented in `docs/setup/prerequisites.md`.

### Local Development Workflow

1. **Clone and Setup**
   ```bash
   git clone <repository-url>
   cd dhaniverse
   npm install
   ```

2. **Create Feature Branch**
   ```bash
   git checkout develop
   git pull origin develop
   git checkout -b feature/123-new-feature
   ```

3. **Development Cycle**
   ```bash
   # Start development servers
   npm run dev          # Frontend development server
   npm run server:game  # Game server with hot reload
   npm run server:ws    # WebSocket server with hot reload
   
   # For ICP canister development
   cd packages/icp-canister
   dfx start --background
   dfx deploy --local
   ```

4. **Testing and Quality Checks**
   ```bash
   # Run tests
   npm test
   cd packages/icp-canister && cargo test
   
   # Code quality checks
   npm run lint
   npm run format
   ```

5. **Commit and Push**
   ```bash
   git add .
   git commit -m "feat: implement new feature"
   git push origin feature/123-new-feature
   ```

6. **Create Pull Request**
   - Use GitHub/GitLab interface
   - Fill out pull request template
   - Request reviews from team members

## Collaboration Guidelines

### Communication

- **Daily Standups**: Brief status updates and blockers
- **Code Reviews**: Constructive feedback and knowledge sharing
- **Documentation**: Keep documentation updated with code changes
- **Issue Tracking**: Use GitHub/GitLab issues for bug reports and feature requests

### Conflict Resolution

#### Merge Conflicts

1. **Prevention**: Regularly sync with `develop` branch
2. **Resolution**: 
   ```bash
   git checkout develop
   git pull origin develop
   git checkout feature/your-branch
   git rebase develop
   # Resolve conflicts in editor
   git add .
   git rebase --continue
   ```

#### Code Conflicts

- Discuss disagreements in pull request comments
- Escalate to team lead if consensus cannot be reached
- Prioritize code quality and maintainability

### Best Practices

#### General

- Write self-documenting code with clear variable and function names
- Keep functions small and focused on single responsibility
- Use consistent naming conventions across the codebase
- Add comments for complex business logic
- Remove dead code and unused imports

#### Git Practices

- Make atomic commits with single logical changes
- Write descriptive commit messages
- Use interactive rebase to clean up commit history before merging
- Tag releases with semantic versioning

#### Code Organization

- Follow established project structure
- Group related functionality together
- Separate concerns between components, services, and utilities
- Use TypeScript interfaces for type safety
- Implement proper error handling and logging

## Continuous Integration

### Automated Workflows

The project uses GitHub Actions/GitLab CI for automated workflows:

#### On Pull Request

- **Code Quality**: ESLint, Prettier, Rust clippy
- **Type Checking**: TypeScript compilation
- **Testing**: Unit and integration tests
- **Security**: Dependency vulnerability scanning
- **Build**: Verify all components build successfully

#### On Merge to Develop

- **Integration Tests**: Full test suite execution
- **Deployment**: Deploy to staging environment
- **Documentation**: Update API documentation

#### On Release

- **Production Build**: Optimized production builds
- **Deployment**: Deploy to production environment
- **Tagging**: Create release tags and changelog

### Quality Gates

Pull requests must pass all quality gates before merging:

- All tests pass (unit, integration, end-to-end)
- Code coverage meets minimum threshold (80%)
- No linting errors or warnings
- Security scan passes
- Performance benchmarks within acceptable range
- Documentation updated for public API changes

## Release Process

### Version Management

Follow Semantic Versioning (SemVer):

- **MAJOR**: Breaking changes
- **MINOR**: New features (backward compatible)
- **PATCH**: Bug fixes (backward compatible)

### Release Steps

1. **Prepare Release Branch**
   ```bash
   git checkout develop
   git pull origin develop
   git checkout -b release/1.2.0
   ```

2. **Update Version Numbers**
   - Update `package.json` version
   - Update `Cargo.toml` version
   - Update documentation versions

3. **Final Testing**
   - Run full test suite
   - Manual testing of critical paths
   - Performance testing

4. **Create Release**
   ```bash
   git checkout main
   git merge release/1.2.0
   git tag v1.2.0
   git push origin main --tags
   ```

5. **Post-Release**
   - Merge release branch back to develop
   - Deploy to production
   - Update changelog
   - Announce release

This workflow ensures code quality, enables efficient collaboration, and maintains a stable production environment while allowing for rapid development and iteration.#
# Related Documentation

- [Coding Standards](./coding-standards.md) - Code style and best practices
- [Testing](./testing.md) - Testing strategies and procedures
- [Debugging](./debugging.md) - Debugging guides and tools
- [Local Setup](../setup/local-setup.md) - Development environment setup
- [Production Deployment](../deployment/production-deployment.md) - Release deployment process

## External Resources

- [Git Flow Workflow](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow)
- [Conventional Commits](https://www.conventionalcommits.org/)
- [GitHub Flow](https://guides.github.com/introduction/flow/)
- [Code Review Best Practices](https://google.github.io/eng-practices/review/)

---

**Navigation**: [← Development Index](./index.md) | [Main Documentation](../README.md) | [Testing →](./testing.md)