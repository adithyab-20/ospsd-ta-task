# Email Client with Dependency Injection

Professional email client implementation using component-based architecture with monkey-patching dependency injection.

## Overview

A modular email client implementing clean architecture principles with a simple yet powerful dependency injection pattern. Built with professional software development standards including comprehensive testing, type safety, and maintainable code structure.

## Key Features

- **Component-Based Architecture**: Independent email_api and gmail_impl components
- **Monkey-Patching DI**: Simple, direct function replacement for dependency injection
- **Type Safety**: Full mypy strict mode compliance
- **Comprehensive Testing**: Unit, integration, and E2E tests
- **OAuth2 Authentication**: Secure Gmail API access with token caching

## Quick Start

### Installation

```bash
git clone <repository-url>
cd ospsd-ta-task

# Install dependencies
uv sync --extra dev --extra email --extra gmail
```

### Basic Usage

```python
import email_api
import gmail_impl  # noqa: F401  # Import injects implementation

# Get client (returns GmailClient via DI)
client = email_api.get_client()

# Fetch emails
for email in client.get_messages(limit=10):
    print(f"From: {email.sender}")
    print(f"Subject: {email.subject}")
```

## Architecture Highlights

### Component Structure

```
ospsd-ta-task/
├── src/
│   ├── email_api/          # Interface component
│   │   └── src/email_api/
│   │       ├── __init__.py
│   │       └── client.py   # Client ABC, Email, EmailAddress
│   └── gmail_impl/         # Implementation component
│       └── src/gmail_impl/
│           ├── __init__.py  # DI injection happens here
│           └── gmail_client.py
├── tests/
│   ├── integration/        # Real Gmail API tests
│   └── e2e/                # Subprocess execution tests
└── main.py                 # Demo application
```

### Dependency Injection

The project uses **monkey-patching dependency injection**:

1. **email_api** defines `get_client()` that raises `NotImplementedError`
2. **gmail_impl** imports email_api and replaces `get_client` with `lambda: GmailClient()`
3. Application code imports both packages and calls `email_api.get_client()`

Benefits:
- Loose coupling
- Simple implementation
- Easy to test
- Flexible

[Learn more about the DI pattern →](architecture/dependency-injection.md)

## Testing Strategy

- **Unit Tests**: Fast, isolated, mock external dependencies
- **Integration Tests**: Real Gmail API, validate OAuth2 and data contracts
- **E2E Tests**: Execute main.py via subprocess, validate complete workflows

[Learn more about testing →](architecture/testing.md)

## Development

```bash
# Run all tests
uv run pytest

# Run specific test types
uv run pytest -m unit
uv run pytest -m integration
uv run pytest -m e2e

# Type checking
uv run mypy src/

# Code quality
uv run ruff check .

# Generate docs
uv run mkdocs serve
```

## Technology Stack

- **Language**: Python 3.12+
- **Package Manager**: uv (workspace-based monorepo)
- **Testing**: pytest with coverage
- **Type Checking**: mypy (strict mode)
- **Linting**: ruff
- **Docs**: mkdocs-material

## License

MIT License - See LICENSE file for details.
