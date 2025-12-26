# Quick Start Guide

Get started with VatPortal API in 5 minutes.

## Prerequisites

- Active VatPortal subscription
- API credentials (username/password or token)
- ASAN phone number registered in portal
- Company VOEN number

## Step 1: Get Your Credentials

Upon subscription, you'll receive:
- Username
- Password
- API Token (optional, for header-based auth)

To generate/view your API token:
1. Log into VatPortal
2. Go to **Profile → Access to API**
3. Click "Generate automatically" or enter custom hash
4. Click "Save"

## Step 2: Make Your First Request

### Simple Invoice Import (Without Upload)

```bash
curl -X POST https://yourcompany.vatportal.az/api/inv/import_upload_invoices.php \
  -H "Content-Type: application/json" \
  -d '{
    "username": "your_username",
    "password": "your_password",
    "upload": false,
    "invoices": [{
      "id": "INV-2025-001",
      "main_info": "Test Invoice",
      "seller": "1234567890",
      "buyer": "0987654321",
      "sum_subtotal1": 100,
      "sum_subtotal2": 100,
      "sum_total": 100,
      "items": [{
        "id": "PROD-001",
        "name": "Test Product",
        "unit": "pcs",
        "unit_price": 100,
        "quantity": 1,
        "barcode": "PROD-001000",
        "subtotal1": 100,
        "subtotal2": 100,
        "total": 100
      }]
    }]
  }'
```

### Expected Response

```json
{
  "err_code": 0,
  "err_msg": "",
  "data": {
    "id": 12345
  }
}
```

The `id` field is your process ID. Use it to check status.

## Step 3: Check Process Status

```bash
curl -X POST https://yourcompany.vatportal.az/api/job/read_proc_status \
  -H "Content-Type: application/json" \
  -d '{
    "username": "your_username",
    "password": "your_password",
    "procId": 12345
  }'
```

### Response

```json
{
  "res": 1,
  "err_code": 0,
  "data": {
    "status": 2,
    "status_desc": "Process completed successfully",
    "stats": {
      "imported": 1,
      "packet_created": 0,
      "login_done": false
    }
  }
}
```

**Status Codes:**
- `2` = Completed successfully
- `3` = Completed with error

## Step 4: Upload to E-taxes (Full Flow)

To upload and sign:

```bash
curl -X POST https://yourcompany.vatportal.az/api/inv/import_upload_invoices.php \
  -H "Content-Type: application/json" \
  -d '{
    "username": "your_username",
    "password": "your_password",
    "upload": true,
    "use_old_sys": false,
    "invoices": [{
      "id": "INV-2025-001",
      "main_info": "Real Invoice for Upload",
      "seller": "1234567890",
      "buyer": "0987654321",
      "sum_subtotal1": 118,
      "sum_subtotal2": 118,
      "sum_apply_vat_amount": 118,
      "sum_vat_amount": 21.24,
      "sum_total": 139.24,
      "items": [{
        "id": "PROD-001",
        "name": "Product Name",
        "unit": "pcs",
        "unit_price": 118,
        "quantity": 1,
        "barcode": "PROD-001000",
        "subtotal1": 118,
        "subtotal2": 118,
        "apply_vat_amount": 118,
        "vat_amount": 21.24,
        "total": 139.24
      }]
    }]
  }'
```

**Important:** When `upload: true`, you'll need to:
1. Confirm PIN1 code (sent to ASAN phone)
2. Confirm PIN2 code for signing

Monitor process status to see these steps.

## Step 5: Retrieve Qaime Number

After successful upload:

```bash
curl -X POST https://yourcompany.vatportal.az/api/inv/read_invoices \
  -H "Content-Type: application/json" \
  -d '{
    "username": "your_username",
    "password": "your_password",
    "ids": ["INV-2025-001"]
  }'
```

### Response

```json
{
  "res": 1,
  "err_code": 0,
  "data": {
    "invoices": [{
      "id": "INV-2025-001",
      "qaime": "AA25010000123"
    }],
    "total": 1
  }
}
```

## Using Token Authentication

Instead of username/password in body, use HTTP header:

```bash
curl -X POST https://yourcompany.vatportal.az/api/inv/import_upload_invoices.php \
  -H "x-vatpapikey: YOUR_TOKEN_HERE" \
  -H "Content-Type: application/json" \
  -d '{
    "upload": false,
    "invoices": [...]
  }'
```

## Common Errors

### Error 5: Invalid credentials
```json
{
  "err_code": 5,
  "err_msg": ""
}
```
**Fix:** Check username/password or token

### Error 14: Invalid invoice data
```json
{
  "err_code": 14,
  "err_msg": "",
  "data": {
    "invoices": [[37, 43]]
  }
}
```
**Fix:** Invoice ID missing (37) and item name empty (43). See [error codes](./error-codes.md)

### Error 6: Empty invoices array
```json
{
  "err_code": 6,
  "err_msg": ""
}
```
**Fix:** Include at least one invoice in the array

## Next Steps

- 📖 [Read full authentication guide](./authentication.md)
- 🔍 [Explore main endpoint](./import-upload-invoices.md)
- 💡 [Check real-world examples](./examples.md)
- ⚠️ [Error codes reference](./error-codes.md)

## Need Help?

- Stuck on authentication? → [Authentication Guide](./authentication.md)
- Invoice validation errors? → [Error Codes](./error-codes.md)
- Want complete examples? → [Examples](./examples.md)

---

[← Back to Documentation](./README.md)
