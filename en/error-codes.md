# Error Codes Reference

Complete guide to VatPortal API error codes and how to fix them.

## General Error Codes

These errors apply to all endpoints:

| Code | Error | Description | Fix |
|------|-------|-------------|-----|
| `1` | Invalid Request Method | Request must use POST | Use `POST` instead of `GET` |
| `2` | Invalid Content-Type | Header must be `application/json` | Set `Content-Type: application/json` |
| `3` | Missing Root Elements | Required JSON fields missing | Check request structure |
| `4` | Empty Credentials | Username or password is empty | Provide valid credentials |
| `5` | Invalid Credentials | Wrong username/password or token | Verify credentials in portal |
| `6` | Empty Invoices Array | No invoices in request | Add at least one invoice |
| `7` | Authentication Error | Internal auth system error | Contact support |
| `10,11,12,16` | Internal Server Error | System error | Retry or contact support |
| `14` | Invoice Validation Error | One or more invoices invalid | See detailed errors in response |
| `15` | Missing ASAN Number | User has no ASAN phone configured | Add ASAN number in portal |
| `17` | Duplicate Invoice ID | Invoice with this ID exists | Use unique invoice ID |
| `20` | Qaime Not Found | Qaime doesn't exist or wrong folder | Verify qaime number and folder |

## Invoice-Level Errors (err_code: 14)

When `err_code: 14`, check the `data.invoices` array for specific error codes:

```json
{
  "err_code": 14,
  "data": {
    "invoices": [
      [37, 22, 43]
    ]
  }
}
```

This means invoice #1 has 3 errors: code 37, 22, and 43.

### Invoice Structure Errors

| Code | Error | Fix |
|------|-------|-----|
| `20` | VOEN not present | Add `seller` or `buyer` field |
| `21` | VOEN invalid | Use exactly 10 digits |
| `22` | Main info not present | Add `main_info` field |
| `23` | Main info empty | Provide non-empty description |
| `37` | Invoice ID not present | Add `id` field |
| `38` | Invoice ID too short | Use at least 7 characters |
| `39` | Items array missing | Add `items` array |

### Invoice Amount Errors

| Code | Error | Fix |
|------|-------|-----|
| `24` | `sum_subtotal1` not present | Add `sum_subtotal1` field |
| `25` | `sum_subtotal1` invalid | Use positive number |
| `26` | `sum_subtotal2` not present | Add `sum_subtotal2` field |
| `27` | `sum_subtotal2` invalid | Use positive number |
| `28` | `sum_total` not present | Add `sum_total` field |
| `29` | `sum_total` invalid | Use positive number |
| `30` | `sum_aksiz` invalid | Use non-negative number |
| `31` | `sum_apply_vat_amount` invalid | Use non-negative number |
| `32` | `sum_skip_vat_amount` invalid | Use non-negative number |
| `33` | `sum_vat_zero` invalid | Use non-negative number |
| `34` | `sum_vat_free` invalid | Use non-negative number |
| `35` | `sum_vat_amount` invalid | Use non-negative number |
| `36` | `sum_ttax` invalid | Use non-negative number |

### Special Invoice Type Errors

| Code | Error | Fix |
|------|-------|-----|
| `61` | Invalid invoice type | Use: `default`, `avans`, or `goodsTransfer` |
| `62` | Avans invoice not found | Verify `main_qaime` exists in portal |
| `63` | Source warehouse not found | Check `from_wh` value |
| `64` | Destination warehouse not found | Check `to_wh` value |

## Item-Level Errors

These errors indicate problems with individual line items:

### Item Structure Errors

| Code | Error | Fix |
|------|-------|-----|
| `40` | Item ID not present | Add `id` field to item |
| `41` | Item ID invalid | Use exactly 10 digits |
| `42` | Item name not present | Add `name` field |
| `43` | Item name empty | Provide product name |
| `44` | Unit not present | Add `unit` field |
| `45` | Unit empty | Provide unit (e.g., "pcs", "kg") |
| `50` | Barcode invalid | Use numeric characters only |
| `51` | Barcode not numeric | Use digits only |

### Item Amount Errors

| Code | Error | Fix |
|------|-------|-----|
| `46` | Unit price not present | Add `unit_price` field |
| `47` | Unit price invalid | Use positive number |
| `48` | Quantity not present | Add `quantity` field |
| `49` | Quantity invalid | Use positive number |
| `52-60` | Item total sums invalid | Check item amount calculations |

## Common Error Scenarios

### Scenario 1: Authentication Fails

**Error:**
```json
{
  "err_code": 5,
  "err_msg": ""
}
```

**Checklist:**
- ✅ Username/password correct?
- ✅ Token still valid?
- ✅ Using correct authentication method?
- ✅ Header name is `x-vatpapikey`?

### Scenario 2: Invalid Invoice Structure

**Error:**
```json
{
  "err_code": 14,
  "data": {
    "invoices": [[37, 43]]
  }
}
```

**Interpretation:**
- Invoice #1 has 2 errors
- Error 37: Invoice ID missing
- Error 43: Item name is empty

**Fix:**
```json
{
  "invoices": [{
    "id": "INV-2025-001",  // ← Added (fixes error 37)
    "main_info": "Sale",
    "seller": "1234567890",
    "buyer": "0987654321",
    "sum_subtotal1": 100,
    "sum_subtotal2": 100,
    "sum_total": 100,
    "items": [{
      "id": "PROD-001",
      "name": "Product Name",  // ← Added (fixes error 43)
      "unit": "pcs",
      "unit_price": 100,
      "quantity": 1,
      "barcode": "PROD-001000",
      "subtotal1": 100,
      "subtotal2": 100,
      "total": 100
    }]
  }]
}
```

### Scenario 3: VOEN Issues

**Error:**
```json
{
  "err_code": 14,
  "data": {
    "invoices": [[20]]
  }
}
```

**Fix:**
```json
{
  "seller": "1234567890",  // Exactly 10 digits
  "buyer": "0987654321"    // Exactly 10 digits
}
```

### Scenario 4: Process Errors

**Error in Process Status:**
```json
{
  "status": 3,
  "status_desc": "Process Finalized With Error",
  "stats": {
    "etx_exception": ["Invalid VOEN: 1111111111"]
  }
}
```

**Note:** This is NOT a validation error (err_code 14). The invoice structure was correct, but e-taxes rejected it for business reasons (e.g., invalid VOEN in e-taxes system).

## Debugging Tips

### 1. Check Response Structure

Always inspect the full response:

```javascript
fetch(url, options)
  .then(res => res.json())
  .then(data => {
    console.log('Full response:', JSON.stringify(data, null, 2));
    
    if (data.err_code !== 0) {
      console.error('Error:', data.err_code);
      if (data.err_code === 14) {
        console.error('Invoice errors:', data.data.invoices);
      }
    }
  });
```

### 2. Validate Before Sending

```javascript
function validateInvoice(invoice) {
  const errors = [];
  
  if (!invoice.id || invoice.id.length < 7) {
    errors.push('Invoice ID must be at least 7 characters');
  }
  
  if (!invoice.seller || invoice.seller.length !== 10) {
    errors.push('Seller VOEN must be exactly 10 digits');
  }
  
  if (!invoice.buyer || invoice.buyer.length !== 10) {
    errors.push('Buyer VOEN must be exactly 10 digits');
  }
  
  if (!invoice.main_info || invoice.main_info.trim() === '') {
    errors.push('Main info is required');
  }
  
  if (!invoice.items || invoice.items.length === 0) {
    errors.push('At least one item required');
  }
  
  invoice.items?.forEach((item, idx) => {
    if (!item.name || item.name.trim() === '') {
      errors.push(`Item ${idx + 1}: Name is required`);
    }
    if (!item.id || item.id.length !== 10) {
      errors.push(`Item ${idx + 1}: ID must be 10 digits`);
    }
  });
  
  return errors;
}

// Usage
const errors = validateInvoice(myInvoice);
if (errors.length > 0) {
  console.error('Validation errors:', errors);
  // Don't send request
} else {
  // Send request
}
```

### 3. Log All Requests

```javascript
// Wrapper for all API calls
async function callVatPortalAPI(endpoint, payload) {
  const timestamp = new Date().toISOString();
  
  console.log(`[${timestamp}] Request to ${endpoint}:`, 
    JSON.stringify(payload, null, 2));
  
  try {
    const response = await fetch(endpoint, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'x-vatpapikey': API_TOKEN
      },
      body: JSON.stringify(payload)
    });
    
    const data = await response.json();
    
    console.log(`[${timestamp}] Response:`, 
      JSON.stringify(data, null, 2));
    
    if (data.err_code !== 0) {
      console.error(`[${timestamp}] Error ${data.err_code}`);
    }
    
    return data;
  } catch (error) {
    console.error(`[${timestamp}] Network error:`, error);
    throw error;
  }
}
```

## Error Code Quick Reference

### Critical Errors (Stop Execution)
- **5** - Auth failed → Fix credentials
- **14** - Validation failed → Fix invoice data
- **15** - No ASAN → Configure in portal

### Retryable Errors
- **10, 11, 12, 16** - Server error → Retry with backoff

### Client Errors (Fix Code)
- **1, 2, 3** - Request format → Fix request structure
- **4, 6** - Missing data → Add required fields

## Need Help?

Can't figure out an error?

1. **Search this doc** - Use Ctrl+F to find error code
2. **Check examples** - See [examples section](../examples/)
3. **Validate JSON** - Use online JSON validator
4. **Test with curl** - Isolate the issue
5. **Contact support** - With full error details

---

[← Back to Documentation](../README.md) | [View Examples →](../examples/)
