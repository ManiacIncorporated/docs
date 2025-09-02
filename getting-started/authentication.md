# Authentication

Maniac uses API keys for authentication. This guide covers how to obtain, configure, and manage your API keys securely.

## Getting Your API Key

### 1. Sign Up for Maniac

Visit [maniac.ai](https://maniac.ai) and create an account:

1. Click "Sign Up"
2. Enter your email and create a password
3. Verify your email address
4. Complete your profile

### 2. Create an API Key

In the Maniac dashboard:

1. Navigate to **Settings** → **API Keys**
2. Click **"Create New API Key"**
3. Give your key a descriptive name (e.g., "Production API", "Development")
4. Select the appropriate permissions
5. Click **"Generate Key"**

⚠️ **Important**: Copy and store your API key immediately. It won't be shown again for security reasons.

## API Key Types

### Development Keys
- Limited to 1,000 requests per day
- Access to sandbox environment
- Perfect for testing and development

### Production Keys
- Higher rate limits based on your plan
- Access to production environment
- Required for live applications

### Service Keys
- Designed for server-to-server communication
- No rate limits (within plan bounds)
- Enhanced security features

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
from maniac import ManiacClient

# Client automatically uses MANIAC_API_KEY environment variable
client = ManiacClient()
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
from maniac import ManiacClient

# Client automatically loads from config file
client = ManiacClient()
```

### Method 3: Direct in Code

```python
from maniac import ManiacClient

client = ManiacClient(api_key="mk-1234567890abcdef")
```

⚠️ **Security Warning**: Never hardcode API keys in production code or commit them to version control.

## Security Best Practices

### 1. Environment-Specific Keys

Use different API keys for different environments:

```python
import os
from maniac import ManiacClient

# Automatically switch based on environment
api_key = os.getenv("MANIAC_API_KEY")
environment = os.getenv("ENVIRONMENT", "development")

client = ManiacClient(
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

- Track request volumes
- Monitor error rates
- Set up usage alerts
- Review access patterns

## Managing Multiple Keys

### Team Environments

For team development, consider:

```python
import os
from maniac import ManiacClient

# Each team member has their own key
api_key = os.getenv(f"MANIAC_API_KEY_{os.getenv('USER', 'default').upper()}")

client = ManiacClient(api_key=api_key)
```

### Service Accounts

For production services:

```python
from maniac import ManiacClient

# Use service-specific keys
client = ManiacClient(
    api_key=os.getenv("MANIAC_SERVICE_KEY"),
    user_agent="MyApp/1.0.0"  # Helps with monitoring
)
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
from maniac import ManiacClient

# Enable debug logging
logging.basicConfig(level=logging.DEBUG)

client = ManiacClient(api_key="your-key")

# Test authentication
try:
    info = client.get_account_info()
    print(f"Authenticated as: {info['email']}")
except Exception as e:
    print(f"Authentication failed: {e}")
```

## Environment Variables Reference

| Variable | Description | Required |
|----------|-------------|----------|
| `MANIAC_API_KEY` | Your API key | Yes |
| `MANIAC_ENVIRONMENT` | Environment (production/sandbox) | No |
| `MANIAC_REGION` | Preferred region | No |
| `MANIAC_TIMEOUT` | Request timeout (seconds) | No |
| `MANIAC_MAX_RETRIES` | Max retry attempts | No |

## Next Steps

- [Create your first container](quickstart.md)
- [Explore the Python SDK](../api/python-sdk.md)
- [Set up monitoring](../guides/performance.md)