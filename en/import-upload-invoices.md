# Import and Upload Invoices

Import invoices to VatPortal and optionally upload them to e-taxes.gov.az for signing.

## Endpoint

```
POST /api/inv/import_upload_invoices.php
```

## Authentication

- Username/Password in body, OR
- Token via `x-vatpapikey` header

## Request Parameters

### Root Level

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `username` | string | Yes* | - | Username (if not using token) |
| `password` | string | Yes* | - | Password (if not using token) |
| `upload` | boolean | No | `false` | Whether to upload to e-taxes |
| `use_old_sys` | boolean | No | `false` | Use old (green) e-taxes system |
| `invoices` | array | Yes | - | Array of invoice objects |

*Required only when not using token authentication

### Invoice Object

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `id` | string | Yes | auto | Unique invoice ID from your ERP (min 7 chars) |
| `type` | string | No | `"default"` | Invoice type: `default`, `avans`, `goodsTransfer` |
| `with_agent` | boolean | No | `false` | Whether items managed by agent |
| `transp_reg_number` | string | No | - | Transport registration number |
| `from_wh` | string | No | - | Source warehouse (for goods transfer) |
| `to_wh` | string | No | - | Destination warehouse (for goods transfer) |
| `wh_field` | integer | No | `1` | Warehouse lookup: `1` = by code, `2` = by name |
| `main_qaime` | string | No | - | Base invoice for "avans" type (e.g., "AA24070000000") |
| `main_info` | string | Yes | - | Main description |
| `add_info` | string | No | - | Additional notes |
| `seller` | string | Yes | - | Seller VOEN (10 digits) |
| `buyer` | string | Yes | - | Buyer VOEN (10 digits) |
| `sum_subtotal1` | decimal | Yes | - | Total price of goods |
| `sum_aksiz` | decimal | No | `0` | Total excise amount |
| `sum_subtotal2` | decimal | Yes | - | Total amount after excise |
| `sum_apply_vat_amount` | decimal | No | `0` | Amount subject to VAT |
| `sum_skip_vat_amount` | decimal | No | `0` | Amount not subject to VAT |
| `sum_vat_zero` | decimal | No | `0` | Amount with 0% VAT |
| `sum_vat_free` | decimal | No | `0` | VAT-exempt amount |
| `sum_vat_amount` | decimal | No | `0` | Total VAT to pay |
| `sum_ttax` | decimal | No | `0` | Road tax amount |
| `sum_total` | decimal | Yes | - | Final total amount |
| `items` | array | Yes | - | Array of line items |

### Invoice Item Object

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `id` | string | Yes | - | Product code (10 digits) |
| `name` | string | Yes | - | Product name |
| `unit` | string | Yes | - | Unit of measurement (e.g., "pcs", "kg") |
| `unit_price` | decimal | Yes | - | Price per unit |
| `quantity` | decimal | Yes | - | Quantity |
| `barcode` | string | Yes | - | Barcode (can be product code + "000") |
| `subtotal1` | decimal | Yes | - | Total price |
| `aksiz` | decimal | No | `0` | Excise amount |
| `aksizgrade` | decimal | No | `0` | Excise rate |
| `subtotal2` | decimal | Yes | - | Total after excise |
| `apply_vat_amount` | decimal | No | `0` | Amount subject to VAT |
| `skip_vat_amount` | decimal | No | `0` | Amount not subject to VAT |
| `vat_zero` | decimal | No | `0` | Amount with 0% VAT |
| `vat_free` | decimal | No | `0` | VAT-exempt amount |
| `vat_amount` | decimal | No | `0` | VAT to pay |
| `transport_tax` | decimal | No | `0` | Road tax |
| `total` | decimal | Yes | - | Item total |

## Response

```json
{
  "err_code": 0,
  "err_msg": "",
  "data": {
    "id": 12345
  }
}
```

| Field | Type | Description |
|-------|------|-------------|
| `err_code` | integer | Error code (`0` = success) |
| `err_msg` | string | Error message (usually empty) |
| `data.id` | integer | Process ID for tracking |

## Examples

### Example 1: Import Only (No Upload)

Import invoice without uploading to e-taxes:

```bash
curl -X POST https://company.vatportal.az/api/inv/import_upload_invoices.php \
  -H "x-vatpapikey: YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "upload": false,
    "invoices": [{
      "id": "INV-2025-001",
      "main_info": "Test sale",
      "seller": "1234567890",
      "buyer": "0987654321",
      "sum_subtotal1": 100,
      "sum_subtotal2": 100,
      "sum_total": 100,
      "items": [{
        "id": "0000000001",
        "name": "Test Product",
        "unit": "pcs",
        "unit_price": 100,
        "quantity": 1,
        "barcode": "0000000001000",
        "subtotal1": 100,
        "subtotal2": 100,
        "total": 100
      }]
    }]
  }'
```

**Response:**
```json
{
  "err_code": 0,
  "err_msg": "",
  "data": {
    "id": 5001
  }
}
```

### Example 2: Standard Invoice with 18% VAT

```json
{
  "username": "your_username",
  "password": "your_password",
  "upload": true,
  "use_old_sys": false,
  "invoices": [{
    "id": "2024030110400001",
    "main_info": "Contract A001; 08.01.2024",
    "add_info": "Qaime XYZ 251145; 28.02.2024",
    "seller": "1111111111",
    "buyer": "0000000000",
    "sum_subtotal1": 2008.47,
    "sum_aksiz": 0,
    "sum_subtotal2": 2008.47,
    "sum_apply_vat_amount": 2008.47,
    "sum_skip_vat_amount": 0,
    "sum_vat_zero": 0,
    "sum_vat_free": 0,
    "sum_vat_amount": 361.53,
    "sum_ttax": 0,
    "sum_total": 2370,
    "items": [{
      "id": "0000000000",
      "name": "Very expensive box",
      "unit": "TON",
      "unit_price": 66.94915,
      "quantity": 30,
      "barcode": "0000000000000",
      "subtotal1": 2008.4745,
      "aksiz": 0,
      "aksizgrade": 0,
      "subtotal2": 2008.4745,
      "apply_vat_amount": 2008.4745,
      "skip_vat_amount": 0,
      "vat_zero": 0,
      "vat_free": 0,
      "vat_amount": 361.52541,
      "transport_tax": 0,
      "total": 2370
    }]
  }]
}
```

### Example 3: Advance (Avans) Invoice

```json
{
  "username": "your_username",
  "password": "your_password",
  "upload": true,
  "invoices": [{
    "id": "INV-AVANS-001",
    "type": "avans",
    "main_info": "Advance payment",
    "seller": "1111111111",
    "buyer": "0000000000",
    "sum_subtotal1": 1000,
    "sum_subtotal2": 1000,
    "sum_apply_vat_amount": 1000,
    "sum_vat_amount": 180,
    "sum_total": 1180,
    "items": [{
      "id": "0000000001",
      "name": "Advance for goods",
      "unit": "service",
      "unit_price": 1000,
      "quantity": 1,
      "barcode": "0000000001000",
      "subtotal1": 1000,
      "subtotal2": 1000,
      "apply_vat_amount": 1000,
      "vat_amount": 180,
      "total": 1180
    }]
  }]
}
```

### Example 4: Invoice Based on Avans

If you created an "avans" invoice with qaime "AA24070000000", create regular invoice:

```json
{
  "username": "your_username",
  "password": "your_password",
  "upload": true,
  "invoices": [{
    "id": "INV-FINAL-001",
    "main_qaime": "AA24070000000"
  }]
}
```

**Note:** When `main_qaime` is provided for avans-based invoice, other fields can be empty - they'll be pulled from the avans invoice.

### Example 5: Simplified Taxation (No VAT)

```json
{
  "username": "your_username",
  "password": "your_password",
  "upload": true,
  "invoices": [{
    "id": "INV-SIMPLE-001",
    "main_info": "Contract A001; 08.01.2024",
    "seller": "1111111111",
    "buyer": "0000000000",
    "sum_subtotal1": 91.72,
    "sum_aksiz": 0,
    "sum_subtotal2": 91.72,
    "sum_total": 91.72,
    "items": [{
      "id": "9965111040",
      "name": "Service under simplified taxation",
      "unit": "pcs",
      "unit_price": 91.72,
      "quantity": 1,
      "barcode": "9965111040000",
      "subtotal1": 91.72,
      "subtotal2": 91.72,
      "total": 91.72
    }]
  }]
}
```

**Note:** For simplified taxation, VAT fields are not required (or set to 0).

### Example 6: Goods Transfer Between Warehouses

```json
{
  "username": "your_username",
  "password": "your_password",
  "upload": true,
  "invoices": [{
    "id": "TRANSFER-001",
    "type": "goodsTransfer",
    "main_info": "Transfer to branch office",
    "from_wh": "WAREHOUSE-001",
    "to_wh": "WAREHOUSE-002",
    "wh_field": 1,
    "seller": "1111111111",
    "buyer": "1111111111",
    "sum_subtotal1": 500,
    "sum_subtotal2": 500,
    "sum_total": 500,
    "items": [{
      "id": "PROD-123",
      "name": "Product for transfer",
      "unit": "pcs",
      "unit_price": 500,
      "quantity": 1,
      "barcode": "PROD-123000",
      "subtotal1": 500,
      "subtotal2": 500,
      "total": 500
    }]
  }]
}
```

## Error Responses

### Validation Errors (err_code: 14)

If invoice data is invalid:

```json
{
  "res": 0,
  "err_code": 14,
  "err_msg": "",
  "data": {
    "invoices": [
      [37, 43]
    ]
  }
}
```

The array `[37, 43]` contains error codes for this specific invoice:
- `37` = Invoice ID not present
- `43` = Item name is empty

See [complete error codes →](../reference/error-codes.md)

### Common Errors

| Code | Description | Solution |
|------|-------------|----------|
| `5` | Invalid username/password | Check credentials |
| `6` | Invoices array is empty | Add at least one invoice |
| `14` | Invoice validation error | Check invoice data |
| `15` | ASAN number not assigned | Configure ASAN phone in portal |
| `17` | Invoice ID already exists | Use unique invoice ID |

## Upload Flow

When `upload: true`, the process goes through these stages:

1. **Import** - Invoice saved to portal
2. **Packet Creation** - Invoice packaged for e-taxes
3. **Login** - Authenticate with e-taxes (PIN1 required)
4. **Upload** - Send packet to e-taxes
5. **Signing** - Electronic signature (PIN2 required)
6. **Complete** - Qaime number assigned

Monitor progress using [Read Process Status](./read-process-status.md) endpoint.

## Notes

- **Minimum ID Length:** Invoice ID must be at least 7 characters
- **VOEN Format:** Must be exactly 10 digits
- **Barcode:** Can be product ID + "000" suffix
- **VAT Calculation:** System validates that sums match
- **Portal Validation:** Only format/structure is validated, not business rules
- **E-taxes Validation:** Full validation happens when uploading (if `upload: true`)

## Next Steps

- [Check process status →](./read-process-status.md)
- [Get invoice data →](./read-process-data.md)
- [Read qaime numbers →](./read-invoices.md)
- [Error codes reference →](../reference/error-codes.md)

---

[← Back to Endpoints](./README.md)
