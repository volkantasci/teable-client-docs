# Installation Guide

This guide will walk you through the process of installing and configuring the Teable Client Library.

## System Requirements

Before installing the Teable Client Library, ensure your system meets these requirements:

- Python 3.7 or higher
- pip (Python package installer)
- Access to a Teable instance
- Teable API key
- For advanced operations: Teable account credentials

## Installation Methods

### 1. Using pip (Recommended)

The simplest way to install the Teable Client Library is using pip:

```bash
pip install teable-client
```

### 2. Installing from Source

For the latest development version, you can install directly from the source:

```bash
git clone https://github.com/your-organization/teable-client.git
cd teable-client
pip install -e .
```

## Environment Setup

### 1. API Key Configuration

You'll need a Teable API key to use the library. There are several ways to configure it:

#### Option 1: Environment Variables

Create a `.env` file in your project root:

```bash
TEABLE_API_KEY=your_api_key_here
TEABLE_API_URL=https://api.teable.io  # Optional, defaults to this value
```

Then in your code:

```python
from teable import TeableClient
client = TeableClient.from_env()
```

#### Option 2: Direct Configuration

```python
from teable import TeableClient, TeableConfig

config = TeableConfig(
    api_key="your_api_key_here",
    api_url="https://api.teable.io"  # Optional
)
client = TeableClient(config)
```

### 2. Authentication Setup

For operations requiring full access (like space management), you'll need to authenticate:

```python
# After client initialization
client.auth.signin(
    email="your-email@example.com",
    password="your-password"
)
```

## Verification

To verify your installation and configuration:

```python
from teable import TeableClient, TeableConfig

# Initialize client
config = TeableConfig(
    api_key="your_api_key_here"
)
client = TeableClient(config)

# Test authentication
try:
    # Sign in (required for full access)
    client.auth.signin(
        email="your-email@example.com",
        password="your-password"
    )
    print("Authentication successful!")
    
    # Test API access
    spaces = client.spaces.get_spaces()
    print(f"Found {len(spaces)} spaces")
    
except Exception as e:
    print(f"Setup verification failed: {str(e)}")
```

## Development Installation

If you're planning to contribute to the library or need to run tests:

1. Clone the repository:
```bash
git clone https://github.com/your-organization/teable-client.git
cd teable-client
```

2. Install development dependencies:
```bash
pip install -r requirements-test.txt
```

3. Set up test environment:
Create a `.env` file in the `tests` directory:
```bash
TEABLE_API_KEY=your_api_key_here
TEABLE_API_URL=https://api.teable.io
TEABLE_EMAIL=your-email@example.com
TEABLE_PASSWORD=your-password
```

4. Run tests:
```bash
pytest tests/
```

## Troubleshooting

### Common Issues

1. **ImportError: No module named 'teable'**
   - Verify the installation: `pip list | grep teable`
   - Try reinstalling: `pip install --force-reinstall teable-client`

2. **Authentication Errors**
   - Verify your API key is correct
   - Check if your account credentials are valid
   - Ensure you're using the correct API URL

3. **Permission Errors**
   - Verify you've signed in for operations requiring authentication
   - Check your account has the necessary permissions

### Getting Help

If you encounter issues:

1. Check the [error handling documentation](../advanced/error-handling.md)
2. Review the [API Reference](../api/client.md)
3. Search existing GitHub issues
4. Create a new issue with:
   - Python version
   - Library version
   - Error message
   - Minimal code example

## Next Steps

Once you've completed the installation:

1. Follow the [Quickstart Guide](quickstart.md)
2. Review [Best Practices](../advanced/best-practices.md)
3. Explore the [API Reference](../api/client.md)
