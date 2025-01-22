# Installation Guide

This guide will walk you through the process of installing and configuring the Teable-Client library in your Python environment.

## Requirements

- Python 3.7 or higher
- pip (Python package installer)
- A Teable instance and API key

## Installation Methods

### Using pip (Recommended)

The simplest way to install Teable-Client is using pip:

```bash
pip install teable-client
```

### Installing from Source

If you want to install from source, you can clone the repository and install using pip:

```bash
git clone https://github.com/yourusername/teable-client.git
cd teable-client
pip install -e .
```

## Configuration

After installation, you'll need to configure the client with your Teable instance details.

### Basic Configuration

```python
from teable import TeableClient, TeableConfig

# Initialize with basic configuration
config = TeableConfig(
    api_url="https://your-teable-instance.com/api",
    api_key="your-api-key"
)
client = TeableClient(config)
```

### Advanced Configuration

You can customize the client behavior with additional configuration options:

```python
config = TeableConfig(
    api_url="https://your-teable-instance.com/api",
    api_key="your-api-key",
    timeout=30,  # Request timeout in seconds
    max_retries=3,  # Number of retry attempts for failed requests
    cache_enabled=True  # Enable response caching
)
```

## Verifying Installation

To verify that the installation was successful, you can run a simple test:

```python
from teable import TeableClient, TeableConfig

# Initialize the client
config = TeableConfig(
    api_url="https://your-teable-instance.com/api",
    api_key="your-api-key"
)
client = TeableClient(config)

# Test the connection
try:
    # Attempt to get user profile
    profile = client.get_user_profile()
    print(f"Connected successfully as: {profile.name}")
except Exception as e:
    print(f"Connection failed: {str(e)}")
```

## Environment Variables

For better security, you can use environment variables to store sensitive configuration:

```python
import os
from teable import TeableClient, TeableConfig

config = TeableConfig(
    api_url=os.getenv("TEABLE_API_URL"),
    api_key=os.getenv("TEABLE_API_KEY")
)
```

Set these environment variables in your shell:

```bash
export TEABLE_API_URL="https://your-teable-instance.com/api"
export TEABLE_API_KEY="your-api-key"
```

## Troubleshooting

### Common Installation Issues

1. **SSL Certificate Verification Failed**
   ```python
   # Add certificate verification
   config = TeableConfig(
       api_url="https://your-teable-instance.com/api",
       api_key="your-api-key",
       verify_ssl=True,
       ssl_ca_cert="/path/to/certificate"
   )
   ```

2. **Proxy Configuration**
   ```python
   # Configure proxy settings
   config = TeableConfig(
       api_url="https://your-teable-instance.com/api",
       api_key="your-api-key",
       proxy={
           "http": "http://proxy.example.com:8080",
           "https": "https://proxy.example.com:8080"
       }
   )
   ```

### Version Conflicts

If you encounter package version conflicts, you can create a virtual environment:

```bash
# Create a virtual environment
python -m venv teable-env

# Activate the environment
# On Windows:
teable-env\Scripts\activate
# On Unix or MacOS:
source teable-env/bin/activate

# Install teable-client in the virtual environment
pip install teable-client
```

## Next Steps

Now that you have installed and configured Teable-Client, proceed to the [Quick Start Guide](quickstart.md) to learn how to perform basic operations with the library.
