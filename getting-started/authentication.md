# Authentication

Maniac uses API keys for authentication. This guide covers how to obtain, configure, and manage your API keys securely.

## Getting Your API Key

### 1. Sign Up for Maniac

Visit [maniac.ai](https://maniac.ai) and create an account:

1. Click "Sign Up"
2. Currently, we support sign ins with GitHub and Google.

### 2. Create an API Key

In the Maniac dashboard:

1. Create a new organization and project.
2. Navigate to **Project** → **Settings** → **API Keys**
3. Click **"Create a Key"**
4. Give your key a descriptive name (e.g., "Production API", "Development")
5. Click **"Submit,"** and copy your key.

⚠️ **Important**: API keys are scoped per user and project.

## Configuring Authentication

### Method 1: Environment Variables (Recommended)

Set your API key as an environment variable:

**Linux/macOS:**

```bash
export MANIAC_API_KEY="mk-1234567890abcdef"
```

**Windows:**

```cmd
set MANIAC_API_KEY=mk-1234567890abcdef
```

**Python Code:**

```python
import os
from maniac import Maniac

# Client automatically uses MANIAC_API_KEY environment variable
client = Maniac(api_key=os.getenv("MANIAC_API_KEY"))
```

### Method 2: Configuration File

Create a configuration file at `~/.maniac/config.yaml`:

```yaml
api_key: mk-1234567890abcdef
environment: production
region: us-west-2
```

**Python Code:**

```python
from maniac import Maniac

# Client automatically loads from config file
client = Maniac(api_key=os.getenv("MANIAC_API_KEY"))
```

### Method 3: Direct in Code

```python
from maniac import Maniac

client = Maniac(api_key="MANIAC_API_KEY")
```

⚠️ **Security Warning**: Never hardcode API keys in production code or commit them to version control.

## Security Best Practices

### 1. Environment-Specific Keys

Use different API keys for different environments:

```python
import os
from maniac import Maniac

# Automatically switch based on environment
api_key = os.getenv("MANIAC_API_KEY")
environment = os.getenv("ENVIRONMENT", "development")

client = Maniac(
    api_key=api_key,
    environment=environment
)
```

### 2. Key Rotation

Regularly rotate your API keys:

1. Generate a new API key
2. Update your environment variables/config
3. Test the new key
4. Delete the old key

### 3. Least Privilege Access

Create keys with minimal required permissions:

```yaml
# Example: Read-only key for monitoring
permissions:
  - containers:read
  - metrics:read

# Example: Full access key for deployment
permissions:
  - containers:*
  - models:*
  - data:*
```

### 4. Monitoring Usage

Monitor your API key usage in the dashboard:

* Track request volumes
* Monitor error rates
* Set up usage alerts
* Review access patterns

## Managing Multiple Keys

### Team Environments

For team development, consider:

```python
import os
from maniac import Maniac

# Each team member has their own key
api_key = os.getenv(f"MANIAC_API_KEY_{os.getenv('USER', 'default').upper()}")

client = Maniac(api_key=api_key)
```

## Troubleshooting Authentication

### Common Error Messages

**Invalid API Key**

```json
{
  "error": "invalid_api_key",
  "message": "The provided API key is invalid or has been revoked"
}
```

**Solution**: Check that your API key is correct and hasn't been revoked.

**Permission Denied**

```json
{
  "error": "permission_denied",
  "message": "This API key doesn't have permission for this action"
}
```

**Solution**: Check your key's permissions in the dashboard.

**Rate Limit Exceeded**

```json
{
  "error": "rate_limit_exceeded",
  "message": "Rate limit exceeded. Try again in 60 seconds"
}
```

**Solution**: Implement rate limiting in your code or upgrade your plan.

### Debugging Authentication Issues

Enable debug logging:

```python
import logging
from maniac import Maniac

# Enable debug logging
logging.basicConfig(level=logging.DEBUG)

client = Maniac(api_key=os.getenv("MANIAC_API_KEY"))

# Test authentication
try:
    info = client.get_account_info()
    print(f"Authenticated as: {info['email']}")
except Exception as e:
    print(f"Authentication failed: {e}")
```

## Environment Variables Reference

| Variable             | Description                      | Required |
| -------------------- | -------------------------------- | -------- |
| `MANIAC_API_KEY`     | Your API key                     | Yes      |
| `MANIAC_ENVIRONMENT` | Environment (production/sandbox) | No       |
| `MANIAC_REGION`      | Preferred region                 | No       |
| `MANIAC_TIMEOUT`     | Request timeout (seconds)        | No       |
| `MANIAC_MAX_RETRIES` | Max retry attempts               | No       |

## Next Steps

* [Create your first container](quickstart.md)
* [Explore the Python SDK](/broken/pages/ptbbkbjtANQZOc4zN2ld)
* [Set up monitoring](../guides/performance.md)
