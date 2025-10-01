# Email Client with Dependency Injection

Professional email client implementation using component-based architecture with monkey-patching dependency injection.

## Architecture

### Component-Based Design

The project is structured as a workspace with independent components:

- **email_api**: Interface component defining contracts using Abstract Base Classes
- **gmail_impl**: Gmail implementation with OAuth2 authentication

Each component can be independently developed, tested, and published.

### Dependency Injection Pattern

This project uses **monkey-patching dependency injection** for simplicity and testability:

#### 1. API Package (`email_api`)

Defines the interface and a stub `get_client()` function:

```python
# email_api/client.py
def get_client() -> Client:
    """Return an instance of a Mail Client."""
    raise NotImplementedError
```

#### 2. Implementation Package (`gmail_impl`)

Directly replaces the API's `get_client()` function when imported:

```python
# gmail_impl/__init__.py
import email_api
from gmail_impl.gmail_client import GmailClient

# Dependency injection: Replace get_client with our implementation
email_api.get_client = lambda: GmailClient()
```

#### 3. Application Usage

Your code depends only on the API interface:

```python
import email_api
import gmail_impl  # noqa: F401  # Import injects the implementation

# Get client (returns GmailClient because we imported gmail_impl)
client = email_api.get_client()

for email in client.get_messages(limit=10):
    print(f"From: {email.sender}")
    print(f"Subject: {email.subject}")
```

### Benefits

1. **Loose Coupling**: Application code depends only on `email_api` interface
2. **Simple**: No complex registry, just direct function replacement
3. **Testable**: Easy to mock by replacing `email_api.get_client`
4. **Flexible**: Swap implementations by changing which package you import

## Project Structure

```
ospsd-ta-task/
├── src/
│   ├── email_api/              # Interface component
│   │   ├── src/email_api/
│   │   │   ├── __init__.py
│   │   │   └── client.py       # Client, Email, EmailAddress
│   │   ├── tests/              # Unit tests
│   │   └── pyproject.toml
│   └── gmail_impl/             # Gmail implementation
│       ├── src/gmail_impl/
│       │   ├── __init__.py     # Performs DI injection
│       │   └── gmail_client.py # GmailClient implementation
│       ├── tests/              # Unit tests
│       └── pyproject.toml
├── tests/                      # Integration and E2E tests
│   ├── integration/            # Real Gmail API tests
│   └── e2e/                    # Subprocess execution tests
├── main.py                     # Demo application
└── pyproject.toml              # Workspace configuration
```

## Setup

### Prerequisites

- Python 3.12+
- [uv](https://docs.astral.sh/uv/) package manager
- Gmail API credentials (for integration/E2E tests)

### Installation

```bash
git clone <repository-url>
cd ospsd-ta-task

# Install dependencies
uv sync --extra dev --extra email --extra gmail
```

### Gmail API Setup

1. Create a project in [Google Cloud Console](https://console.cloud.google.com/)
2. Enable Gmail API
3. Create OAuth 2.0 credentials
4. Download credentials as `credentials.json` in project root

## Development

```bash
# Run all tests (unit + integration + e2e)
uv run pytest

# Run specific test types
uv run pytest -m unit           # Unit tests only
uv run pytest -m integration    # Integration tests only
uv run pytest -m e2e            # E2E tests only

# Component-specific tests
uv run pytest src/email_api/tests/
uv run pytest src/gmail_impl/tests/

# Type checking
uv run mypy src/

# Code quality
uv run ruff check .
uv run ruff format .

# Coverage
uv run pytest --cov=src --cov-report=html

# Demo
uv run python main.py
```

## Testing Strategy

### Unit Tests (`src/*/tests/`)
- Test data models and business logic
- Mock external dependencies (Gmail API)
- Each test uses individual `with patch()` statements
- Fast, isolated, no network calls

### Integration Tests (`tests/integration/`)
- Test real Gmail API integration
- Validate OAuth2 flow and token management
- Use `email_api.get_client()` to respect DI abstraction
- Require credentials.json

### E2E Tests (`tests/e2e/`)
- Execute `main.py` via subprocess
- Validate complete user workflows
- Test syntax, imports, and file structure
- True end-to-end testing with process isolation

## Testing Pattern

Replace `get_client()` for testing:

```python
import email_api
from unittest.mock import Mock

# Inject mock client
mock_client = Mock(spec=email_api.Client)
email_api.get_client = lambda: mock_client

# Test your code
client = email_api.get_client()
assert client is mock_client
```

## Quality Standards

- **Type Safety**: Full mypy strict mode compliance
- **Code Quality**: All ruff rules enabled with documented exceptions
- **Test Coverage**: Comprehensive unit, integration, and E2E tests
- **Professional Standards**: Clean, maintainable, production-ready code

## Technology Stack

- **Language**: Python 3.12+
- **Package Management**: uv (workspace-based monorepo)
- **Testing**: pytest with coverage
- **Type Checking**: mypy (strict mode)
- **Code Quality**: ruff (linter + formatter)
- **Architecture**: Component-based with monkey-patching DI

## Design Principles

This architecture follows interface design best practices:

1. **Deep Modules**: Simple interface hides complex implementation
2. **Information Hiding**: Application doesn't know about Gmail API details
3. **Define Errors Out of Existence**: Import the implementation → it just works
4. **Pull Complexity Downward**: Implementation handles OAuth, API calls, parsing

## License

MIT License - See LICENSE file for details.
