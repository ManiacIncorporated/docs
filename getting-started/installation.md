# Installation Guide

This guide will help you install and configure the Maniac Python library.

## Prerequisites

- **Python 3.9 or higher**
- **Maniac API key** (provides access to all model providers automatically)

## Installation

Install the Maniac library using pip:

```bash
pip install maniac
```

## Setup

### 1. Get Your Maniac API Key

Sign up at [maniac.ai](https://maniac.ai) to get your API key that provides access to all supported models.

### 2. Configure Authentication

Set your API key via environment variable:

```bash
export MANIAC_API_KEY=your-maniac-api-key
```

### 3. Test Installation

```python
from maniac import Maniac

client = Maniac(api_key="your-maniac-api-key")

# Test with any model - Maniac handles provider routing
response = client.chat.completions.create(
    fallback="claude-opus-4",
    messages=[{"role": "user", "content": "Hello!"}],
    task_label="installation-test",
    judge_prompt="Compare two greeting responses. Is A better than B? Consider: friendliness, appropriateness."
)

print(response["choices"][0]["message"]["content"])
```

## Verify Installation

Check that Maniac is installed correctly:

```python
import maniac
print(f"Maniac version: {maniac.__version__}")
print("Available components:", maniac.__all__)
```

## Optional Dependencies

### For Advanced Features

```bash
# For async support
pip install maniac-ai[async]

# For data processing utilities
pip install maniac-ai[data]

# For local model support
pip install maniac-ai[local]

# Install all optional dependencies
pip install maniac-ai[all]
```

## Environment Setup

### Setting API Keys

You can set your API key in several ways:

**Environment Variable (Recommended)**
```bash
export MANIAC_API_KEY="your-api-key-here"
```

**Configuration File**
Create `~/.maniac/config.yaml`:
```yaml
api_key: your-api-key-here
environment: production
region: us-west-2
```

**In Code**
```python
client = Maniac(provider="vertex", project_id="your-project")
```

### Configuration Options

```python
from maniac import Maniac

client = Maniac(
    api_key="your-api-key",
    environment="production",  # or "sandbox"
    region="us-west-2",       # optional: for latency optimization
    timeout=30,               # request timeout in seconds
    max_retries=3,           # automatic retry on failure
    cache_enabled=True       # enable response caching
)
```

## Upgrading

### Upgrade to Latest Version

```bash
pip install --upgrade maniac-ai
```

### Check for Updates

```python
from maniac import check_for_updates

if check_for_updates():
    print("New version available! Run: pip install --upgrade maniac-ai")
```

### Migration Guides

- [Migrating from v0.x to v1.0](../guides/migration-v1.md)
- [Breaking changes](../resources/changelog.md)

## Platform Support

### Supported Python Versions
- Python 3.8
- Python 3.9
- Python 3.10
- Python 3.11
- Python 3.12

### Operating Systems
- Linux (Ubuntu 20.04+, CentOS 8+, Debian 10+)
- macOS (10.15+)
- Windows (10, 11, Server 2019+)

### Cloud Platforms
- AWS Lambda
- Google Cloud Functions
- Azure Functions
- Vercel
- Heroku

## Troubleshooting Installation

### Common Issues

**SSL Certificate Error**
```bash
pip install --trusted-host pypi.org --trusted-host files.pythonhosted.org maniac-ai
```

**Permission Denied**
```bash
pip install --user maniac-ai
```

**Dependency Conflicts**
```bash
# Create a virtual environment
python -m venv maniac-env
source maniac-env/bin/activate  # On Windows: maniac-env\Scripts\activate
pip install maniac-ai
```

**Behind Corporate Proxy**
```bash
pip install --proxy http://user:password@proxy.server:port maniac-ai
```

## Uninstallation

To completely remove Maniac:

```bash
# Remove package
pip uninstall maniac-ai

# Remove configuration files (optional)
rm -rf ~/.maniac

# Remove cache (optional)
rm -rf ~/.cache/maniac
```

## Next Steps

- [Configure authentication](authentication.md)
- [Create your first container](quickstart.md)
- [Explore the API](../api/python-sdk.md)