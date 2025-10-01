# Dependency Injection Pattern

This project uses **monkey-patching dependency injection** - a simple and elegant pattern for loose coupling.

## How It Works

### 1. API Package (`email_api`)

The API defines the interface and a stub `get_client()` function:

```python
# email_api/client.py
def get_client() -> Client:
    """Return an instance of a Mail Client."""
    raise NotImplementedError
```

### 2. Implementation Package (`gmail_impl`)

The implementation **directly replaces** the API's `get_client()` function when imported:

```python
# gmail_impl/__init__.py
import email_api
from gmail_impl.gmail_client import GmailClient

# Dependency injection: Replace get_client with our implementation
email_api.get_client = lambda: GmailClient()
```

### 3. Application Usage

Your application code depends only on the API:

```python
import email_api
import gmail_impl  # noqa: F401  # Import injects the implementation

# Get client from API (returns GmailClient because we imported gmail_impl)
client = email_api.get_client()

for email in client.get_messages(limit=10):
    print(f"Subject: {email.subject}")
```

## Benefits

### 1. Loose Coupling
Application code depends only on the `email_api` interface, not the implementation.

### 2. Simple
No complex registry or framework - just direct function replacement.

### 3. Testable
Easy to mock by replacing `email_api.get_client`:

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

### 4. Flexible
Swap implementations by changing which package you import:

```python
import email_api

# Uncomment the implementation you want:
import gmail_impl  # noqa: F401
# import outlook_impl  # noqa: F401
# import mock_impl  # noqa: F401

client = email_api.get_client()  # Gets the imported implementation
```

## Comparison to Registry Pattern

**Monkey-Patching (This Approach):**

- ✅ Simpler - no registry logic needed
- ✅ Direct - obvious what's happening
- ✅ Less code - just function replacement

**Registry Pattern:**

- ❌ More complex - needs `register_client()` function
- ❌ Extra indirection - function calls registry
- ❌ Over-engineered for this use case

## Testing Patterns

### Unit Testing the Implementation

Use monkey-patching on internal methods:

```python
from unittest.mock import patch
import email_api

def test_get_messages():
    with patch("gmail_impl.gmail_client.build"):
        client = email_api.get_client()
        messages = list(client.get_messages(limit=5))
        assert len(messages) <= 5
```

### Integration Testing

Use `email_api.get_client()` to respect DI abstraction:

```python
import email_api
import gmail_impl  # noqa: F401

def test_real_gmail_integration():
    client = email_api.get_client()
    messages = list(client.get_messages(limit=1))
    assert isinstance(messages, list)
```

## Design Principles

This pattern follows John Ousterhout's interface design principles:

1. **Deep Modules**: Simple interface hides complex implementation
2. **Information Hiding**: Application doesn't know about Gmail API details
3. **Define Errors Out of Existence**: Import the implementation → it just works
4. **Pull Complexity Downward**: Implementation handles OAuth, API calls, parsing

## Environment Configuration

The implementation supports environment variable configuration:

```python
# Set via environment variables
export GMAIL_CREDENTIALS_PATH=/path/to/credentials.json
export GMAIL_TOKEN_PATH=/path/to/token.json

# Or pass directly (but breaks DI abstraction)
client = GmailClient(
    credentials_file="/path/to/credentials.json",
    token_file="/path/to/token.json"
)
```

For testing, use environment variables to inject test-specific configs while maintaining DI abstraction.
