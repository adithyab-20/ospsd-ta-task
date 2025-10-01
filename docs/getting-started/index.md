# Getting Started

Quick start guide for setting up and using the Email Client with Dependency Injection.

## Prerequisites

- Python 3.12+
- [uv](https://docs.astral.sh/uv/) package manager
- Google Cloud Console account (for Gmail implementation)

## Installation

### 1. Clone and Setup

```bash
git clone <repository-url>
cd ospsd-ta-task

# Install all dependencies
uv sync --extra dev --extra email --extra gmail
```

### 2. Gmail API Setup

To use the Gmail implementation:

1. Go to the [Google Cloud Console](https://console.cloud.google.com/)
2. Create a new project or select an existing one
3. Enable the Gmail API
4. Go to "APIs & Services" → "Credentials"
5. Create OAuth 2.0 credentials for a desktop application
6. Download the credentials JSON file as `credentials.json` in the project root
7. Configure OAuth consent screen and add your Gmail as a test user

## Quick Start

### Basic Usage

```python
import email_api
import gmail_impl  # noqa: F401  # Import injects implementation

# Get client via dependency injection
client = email_api.get_client()

# Fetch recent emails
for email in client.get_messages(limit=5):
    print(f"From: {email.sender}")
    print(f"Subject: {email.subject}")
    print(f"Date: {email.date_sent}")
```

### Demo Application

Run the included demo:

```bash
uv run python main.py
```

This will:
1. Open browser for OAuth2 authentication (first run only)
2. Fetch your 5 most recent emails
3. Display email content

## Development

### Running Tests

```bash
# All tests (unit + integration + e2e)
uv run pytest

# Unit tests only (fast, no credentials needed)
uv run pytest -m unit

# Integration tests (requires credentials.json)
uv run pytest -m integration

# E2E tests (requires credentials.json)
uv run pytest -m e2e
```

### Code Quality

```bash
# Type checking
uv run mypy src/

# Linting
uv run ruff check .

# Format code
uv run ruff format .

# Coverage report
uv run pytest --cov=src --cov-report=html
```

### Documentation

```bash
# Serve documentation locally at http://127.0.0.1:8000
uv run mkdocs serve

# Build static documentation
uv run mkdocs build
```

## Project Structure

```
ospsd-ta-task/
├── src/
│   ├── email_api/          # Interface component
│   └── gmail_impl/         # Gmail implementation
├── tests/
│   ├── integration/        # Integration tests
│   └── e2e/                # End-to-end tests
├── docs/                   # Documentation
├── main.py                 # Demo application
└── pyproject.toml          # Workspace configuration
```

## Next Steps

- Learn about [Dependency Injection](../architecture/dependency-injection.md)
- Understand the [Testing Strategy](../architecture/testing.md)
- Explore the [Email API Reference](../reference/email-api.md)
- Check out the [Gmail Implementation](../reference/gmail-impl.md)
