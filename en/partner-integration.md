# Partner Integration Guide

**Available for:** Partner Program
**Subdomain Format:** `partnername.vatportal.az`

## Overview

The VatPortal Partner Program allows integration partners to manage multiple end-customers through a single API integration. Partners act as intermediaries, handling e-taxes operations on behalf of their customers.

## Partner vs Enterprise Model

| Feature | Enterprise Customers | Partners |
|---------|---------------------|----------|
| **Subdomain** | `company.vatportal.az` | `partnername.vatportal.az` |
| **Customer Context** | Single company (implicit) | Multiple customers (explicit via `phone`) |
| **Authentication** | Username/password or token | Username/password or token |
| **Phone Parameter** | Not required | **Required in ALL API calls** |
| **Use Case** | Direct integration for one company | Service provider managing multiple companies |

## How Partner Integration Works

### 1. Customer Registration Flow

```
┌─────────────┐
│   Partner   │
└──────┬──────┘
       │
       │ 1. Register customer in portal
       ├──────────────────────────────────────┐
       │                                      │
       │                              ┌───────▼────────┐
       │                              │  VatPortal     │
       │                              │  Admin Review  │
       │                              └───────┬────────┘
       │                                      │
       │ 2. Customer approved                 │
       │◄─────────────────────────────────────┤
       │                                      │
       │ 3. Add ASAN phone numbers            │
       │      for customer                    │
       ├──────────────────────────────────────►
       │                                      │
       │ 4. Use phone in API calls            │
       │      to operate on behalf            │
       │      of customer                     │
       └──────────────────────────────────────┘
```

### 2. Phone Number Management

- Each customer registered by a partner can have **one or more ASAN phone numbers**
- Phone numbers must be registered in the VatPortal before use
- The `phone` parameter in API calls identifies which customer the operation is for
- Format: `994XXXXXXXXX` (Azerbaijan ASAN format)

### 3. API Authentication

Partners use the same authentication methods as enterprise customers:

**Option 1: Username/Password in Request Body**
```json
{
  "username": "partner_username",
  "password": "partner_password",
  "phone": "994501234567",
  ...
}
```

**Option 2: Token in HTTP Header**
```bash
curl -X POST https://partnername.vatportal.az/api/inv/import_upload_invoices.php \
  -H "x-vatpapikey: PARTNER_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "phone": "994501234567",
    "invoices": [...]
  }'
```

**Key Difference:** Partners must **always include the `phone` parameter** in every API call.

## Required Changes to API Calls

### For Enterprise Customers (Current)
```json
{
  "username": "company_user",
  "password": "company_pass",
  "invoices": [...]
}
```

### For Partners (With Phone Parameter)
```json
{
  "username": "partner_user",
  "password": "partner_pass",
  "phone": "994501234567",  // ← REQUIRED: Identifies customer
  "invoices": [...]
}
```

## All Endpoints Require Phone Parameter

The `phone` parameter must be included in **ALL** API endpoints when using partner credentials:

### Invoice Operations
- ✅ `POST /inv/import_upload_invoices.php` - Include `phone`
- ✅ `GET /inv/read_invoices` - Include `phone`
- ✅ `DELETE /inv/delete` - Include `phone`

### E-taxes Integration
- ✅ `POST /etx/import` - Include `phone`
- ✅ `GET /etx/read_invoices` - Include `phone`
- ✅ `DELETE /etx/delete` - Include `phone`

### Process Monitoring
- ✅ `GET /job/read_proc_status` - Include `phone`
- ✅ `GET /job/read_proc_data` - Include `phone`

### Authentication Token Renewal
- ✅ `GET /auth/reset_start.php` - Include `phone` in query string
- ✅ `POST /auth/reset_confirm.php` - Include `phone`

## Error Handling

### Missing Phone Parameter

If a partner makes an API call without the `phone` parameter:

```json
{
  "res": 0,
  "err_code": 15,
  "err_msg": "Phone number required for partner accounts",
  "data": null
}
```

### Invalid Phone Number

If the phone number is not registered for the partner:

```json
{
  "res": 0,
  "err_code": 16,
  "err_msg": "Phone number not found or not authorized for this partner",
  "data": null
}
```

### Phone Number Format Error

```json
{
  "res": 0,
  "err_code": 17,
  "err_msg": "Invalid phone number format. Use 994XXXXXXXXX",
  "data": null
}
```

## Complete Example: Partner Upload Invoice

```json
{
  "username": "partner_integration",
  "password": "secure_password",
  "phone": "994501234567",
  "upload": true,
  "invoices": [
    {
      "id": "CUST001-INV-2024-001",
      "main_info": "Sale to Customer A",
      "seller": "1234567890",
      "buyer": "0987654321",
      "sum_subtotal1": 1000.00,
      "sum_subtotal2": 1000.00,
      "sum_apply_vat_amount": 1000.00,
      "sum_vat_amount": 180.00,
      "sum_total": 1180.00,
      "items": [
        {
          "id": "PROD001",
          "name": "Product Name",
          "unit": "pcs",
          "unit_price": 10.00,
          "quantity": 100,
          "barcode": "PROD001000",
          "subtotal1": 1000.00,
          "subtotal2": 1000.00,
          "apply_vat_amount": 1000.00,
          "vat_amount": 180.00,
          "total": 1180.00
        }
      ]
    }
  ]
}
```

## Best Practices for Partners

### 1. Customer Identification
- Always validate phone numbers before making API calls
- Store customer-to-phone mapping in your system
- Handle multiple phone numbers per customer if needed

### 2. Error Handling
- Implement retry logic for phone number validation errors
- Log all API calls with associated phone numbers for audit
- Provide clear error messages to your end-users

### 3. Security
- Store partner credentials securely
- Use token-based authentication for production
- Rotate tokens regularly
- Never expose phone numbers to unauthorized users

### 4. Testing
- Test with multiple customer phone numbers
- Verify operations are isolated per customer
- Test edge cases (invalid phone, unauthorized phone)

## Migration from Enterprise to Partner

If you're migrating from an enterprise integration to a partner integration:

### Step 1: Register as Partner
Contact VatPortal support to convert your account to a partner account.

### Step 2: Register Your Customers
Add all your customers through the VatPortal partner portal.

### Step 3: Update Your Integration
Add the `phone` parameter to **all** API calls:

```javascript
// Before (Enterprise)
const request = {
  username: "myuser",
  password: "mypass",
  invoices: [...]
};

// After (Partner)
const request = {
  username: "myuser",
  password: "mypass",
  phone: getCustomerPhone(customerId), // ← Add this
  invoices: [...]
};
```

### Step 4: Test Thoroughly
Test all endpoints with different customer phone numbers before going live.

## Support

For partner program inquiries:
- **Sales:** Contact your account manager
- **Technical Support:** support@amr.az
- **Documentation Issues:** [GitHub Issues](https://github.com/amrsolutions/vatp-api-docs/issues)

---

**Last Updated:** December 28, 2025
**Applies to:** Partner Program participants
