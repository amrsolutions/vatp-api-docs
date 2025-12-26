# Authentication

VatPortal API supports two authentication methods:

1. **Username/Password** - Credentials in JSON body
2. **Token-based** - API key in HTTP header

## Method 1: Username/Password

Include credentials in every request body:

```json
{
  "username": "your_username",
  "password": "your_password",
  // ... other parameters
}
```

### Example

```bash
curl -X POST https://company.vatportal.az/api/inv/read_invoices \
  -H "Content-Type: application/json" \
  -d '{
    "username": "john_doe",
    "password": "secure_password_123",
    "ids": ["INV-001"]
  }'
```

### Pros & Cons

✅ Simple to implement  
✅ No token management needed  
❌ Credentials in every request  
❌ Harder to rotate passwords  

## Method 2: Token-based (Recommended)

Send API token via HTTP header:

```bash
curl -X POST https://company.vatportal.az/api/inv/read_invoices \
  -H "x-vatpapikey: YOUR_TOKEN_HERE" \
  -H "Content-Type: application/json" \
  -d '{
    "ids": ["INV-001"]
  }'
```

### Pros & Cons

✅ More secure (no password in requests)  
✅ Easy to rotate tokens  
✅ Cleaner request payloads  
✅ Can be revoked independently  
❌ Requires initial setup  

## Getting Your API Token

### Step 1: Access Portal Settings

1. Log into VatPortal at `https://yourcompany.vatportal.az`
2. Navigate to **Profile → Access to API**

### Step 2: Generate Token

You have two options:

**Option A: Auto-generate**
```
1. Click "Generate automatically"
2. Click "Save"
3. Copy the generated hash key
```

**Option B: Custom hash**
```
1. Enter your own hash key (secure string)
2. Click "Save"
3. Use this hash as your token
```

### Step 3: Use Token

Add the token to request headers:

```
x-vatpapikey: your_generated_token_here
```

## Token Renewal (2FA Flow)

If you need to renew/regenerate your token programmatically:

### Step 1: Initiate Renewal

```bash
curl -X GET "https://company.vatportal.az/api/auth/reset_start.php?login=your_username"
```

**Response:**
```json
{
  "plain_value": "random_challenge_string_xyz"
}
```

### Step 2: Calculate Signature

Use HMAC-SHA256 to sign the challenge:

```javascript
// JavaScript example
const crypto = require('crypto');

const plainValue = "random_challenge_string_xyz";
const secretHash = "your_secret_hash_from_portal";

const signature = crypto
  .createHmac('sha256', secretHash)
  .update(plainValue)
  .digest('hex');

console.log(signature); // Send this as 'key'
```

```python
# Python example
import hmac
import hashlib

plain_value = "random_challenge_string_xyz"
secret_hash = "your_secret_hash_from_portal"

signature = hmac.new(
    secret_hash.encode(),
    plain_value.encode(),
    hashlib.sha256
).hexdigest()

print(signature)  # Send this as 'key'
```

### Step 3: Confirm Renewal

```bash
curl -X POST https://company.vatportal.az/api/auth/reset_confirm.php \
  -H "Content-Type: application/json" \
  -d '{
    "login": "your_username",
    "key": "calculated_signature_here"
  }'
```

**Response:**
```json
{
  "token": "your_new_api_token"
}
```

### Step 4: Update Your Application

Replace old token with new token in your application configuration.

## Security Best Practices

### ✅ DO

- **Store tokens securely** - Use environment variables or secret managers
- **Use HTTPS only** - Never send credentials over HTTP
- **Rotate tokens regularly** - Change every 90 days minimum
- **Use token method** - Prefer header-based auth over username/password
- **Monitor usage** - Track API calls for suspicious activity
- **Set up alerts** - Get notified of authentication failures

### ❌ DON'T

- **Hardcode credentials** - Never commit passwords/tokens to git
- **Share tokens** - Each application should have its own token
- **Log credentials** - Sanitize logs to remove sensitive data
- **Reuse passwords** - Use unique password for VatPortal API
- **Expose tokens in URLs** - Always use headers for tokens

## Error Handling

### Invalid Credentials (Error 5)

```json
{
  "err_code": 5,
  "err_msg": "Invalid username or password"
}
```

**Causes:**
- Wrong username/password
- Invalid or expired token
- Token not matching portal configuration

**Fix:**
1. Verify credentials in portal
2. For tokens: regenerate and update
3. For password: reset via portal

### Authentication Error (Error 7)

```json
{
  "err_code": 7,
  "err_msg": "Authentication error"
}
```

**Causes:**
- Internal authentication system issue
- Account disabled

**Fix:**
- Contact VatPortal support
- Check account status in portal

## Code Examples

### Node.js with Token

```javascript
const axios = require('axios');

const API_BASE = 'https://company.vatportal.az/api';
const API_TOKEN = process.env.VATPORTAL_TOKEN;

async function getInvoices(invoiceIds) {
  try {
    const response = await axios.post(
      `${API_BASE}/inv/read_invoices`,
      { ids: invoiceIds },
      {
        headers: {
          'Content-Type': 'application/json',
          'x-vatpapikey': API_TOKEN
        }
      }
    );
    return response.data;
  } catch (error) {
    console.error('API Error:', error.response?.data);
    throw error;
  }
}
```

### Python with Token

```python
import os
import requests

API_BASE = 'https://company.vatportal.az/api'
API_TOKEN = os.getenv('VATPORTAL_TOKEN')

def get_invoices(invoice_ids):
    response = requests.post(
        f'{API_BASE}/inv/read_invoices',
        json={'ids': invoice_ids},
        headers={
            'Content-Type': 'application/json',
            'x-vatpapikey': API_TOKEN
        }
    )
    response.raise_for_status()
    return response.json()

# Usage
invoices = get_invoices(['INV-001', 'INV-002'])
```

### C# with Token

```csharp
using System;
using System.Net.Http;
using System.Text;
using System.Threading.Tasks;
using Newtonsoft.Json;

public class VatPortalClient
{
    private readonly HttpClient _client;
    private readonly string _apiToken;
    private const string ApiBase = "https://company.vatportal.az/api";

    public VatPortalClient(string apiToken)
    {
        _apiToken = apiToken;
        _client = new HttpClient();
        _client.DefaultRequestHeaders.Add("x-vatpapikey", _apiToken);
    }

    public async Task<dynamic> GetInvoices(string[] invoiceIds)
    {
        var payload = new { ids = invoiceIds };
        var content = new StringContent(
            JsonConvert.SerializeObject(payload),
            Encoding.UTF8,
            "application/json"
        );

        var response = await _client.PostAsync(
            $"{ApiBase}/inv/read_invoices",
            content
        );

        response.EnsureSuccessStatusCode();
        var json = await response.Content.ReadAsStringAsync();
        return JsonConvert.DeserializeObject(json);
    }
}
```

## Testing Authentication

Quick test to verify your credentials:

```bash
# Test username/password
curl -X POST https://company.vatportal.az/api/inv/read_invoices \
  -H "Content-Type: application/json" \
  -d '{
    "username": "test",
    "password": "test",
    "ids": [],
    "date_from": "01.01.2025",
    "date_to": "01.01.2025"
  }'

# Test token
curl -X POST https://company.vatportal.az/api/inv/read_invoices \
  -H "x-vatpapikey: your_token" \
  -H "Content-Type: application/json" \
  -d '{
    "ids": [],
    "date_from": "01.01.2025",
    "date_to": "01.01.2025"
  }'
```

If successful, you'll get `err_code: 0` response.

## Troubleshooting

| Issue | Solution |
|-------|----------|
| "Invalid credentials" | Verify username/password in portal |
| "Token not working" | Regenerate token in portal settings |
| "Empty response" | Check if token renewal process succeeded |
| "401 Unauthorized" | Ensure `x-vatpapikey` header is set correctly |

## Next Steps

- [Make your first API call →](./quickstart.md)
- [Explore API endpoints →](./endpoints/)
- [View code examples →](./examples/)

---

[← Back to Documentation](./README.md)
