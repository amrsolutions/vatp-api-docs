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
  "res": 1,
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
  "res": 1,
  "err_code": 0,
  "err_msg": "",
  "data": {
    "id": 5001
  }
}
```

### Example 2: Multi-item Invoice with 18% VAT & Transport Tax

Real-world example with multiple line items and transport tax:

```json
{
  "username": "your_username",
  "password": "your_password",
  "upload": true,
  "use_old_sys": false,
  "invoices": [{
    "id": "2024030110400001",
    "main_info": "Müqavilə A001; 08.01.2024",
    "add_info": "Qaimə XYZ 251145; 28.02.2024",
    "seller": "1111111111",
    "buyer": "0000000000",
    "sum_subtotal1": 428.34,
    "sum_aksiz": 0,
    "sum_subtotal2": 428.34,
    "sum_apply_vat_amount": 428.34,
    "sum_skip_vat_amount": 0,
    "sum_vat_zero": 0,
    "sum_vat_free": 0,
    "sum_vat_amount": 77.10,
    "sum_ttax": 14.56,
    "sum_total": 520.00,
    "items": [
      {
        "id": "9947008540",
        "name": "Maye qaz (texniki butan)",
        "unit": "ton",
        "unit_price": 0.400,
        "quantity": 816.61020,
        "barcode": "9947008540000",
        "subtotal1": 326.6441,
        "aksiz": 0,
        "aksizgrade": 0,
        "subtotal2": 326.6441,
        "apply_vat_amount": 326.6441,
        "skip_vat_amount": 0,
        "vat_zero": 0,
        "vat_free": 0,
        "vat_amount": 58.7959,
        "transport_tax": 14.56,
        "total": 400.0000
      },
      {
        "id": "9947008540",
        "name": "Daşınma xərci",
        "unit": "xidmət",
        "unit_price": 0.400,
        "quantity": 254.24,
        "barcode": "9947008540000",
        "subtotal1": 101.6960,
        "aksiz": 0,
        "aksizgrade": 0,
        "subtotal2": 101.6960,
        "apply_vat_amount": 101.6960,
        "skip_vat_amount": 0,
        "vat_zero": 0,
        "vat_free": 0,
        "vat_amount": 18.3053,
        "transport_tax": 0,
        "total": 120.0013
      }
    ]
  }]
}
```

### Example 3: Advance (Avans) Invoice

Advance payment invoice that can be referenced later:

```json
{
  "username": "your_username",
  "password": "your_password",
  "upload": true,
  "use_old_sys": false,
  "invoices": [{
    "id": "2025010115500001",
    "type": "avans",
    "main_info": "Müqavilə B-2025/105; Avans ödənişi",
    "add_info": "50% əvvəlcədən ödəniş",
    "seller": "1111111111",
    "buyer": "2222222222",
    "sum_subtotal1": 5000.00,
    "sum_aksiz": 0,
    "sum_subtotal2": 5000.00,
    "sum_apply_vat_amount": 5000.00,
    "sum_skip_vat_amount": 0,
    "sum_vat_zero": 0,
    "sum_vat_free": 0,
    "sum_vat_amount": 900.00,
    "sum_ttax": 0,
    "sum_total": 5900.00,
    "items": [{
      "id": "1234567890",
      "name": "Kompüter avadanlıqları (avans)",
      "unit": "dəst",
      "unit_price": 2500.00,
      "quantity": 2,
      "barcode": "1234567890000",
      "subtotal1": 5000.00,
      "aksiz": 0,
      "aksizgrade": 0,
      "subtotal2": 5000.00,
      "apply_vat_amount": 5000.00,
      "skip_vat_amount": 0,
      "vat_zero": 0,
      "vat_free": 0,
      "vat_amount": 900.00,
      "transport_tax": 0,
      "total": 5900.00
    }]
  }]
}
```

### Example 4: Invoice Based on Avans

Final invoice that references a previous advance payment:

```json
{
  "username": "your_username",
  "password": "your_password",
  "upload": true,
  "use_old_sys": false,
  "invoices": [{
    "id": "2025012015600001",
    "main_qaime": "AA24070000000",
    "main_info": "Malların çatdırılması (avansa əsasən)",
    "seller": "1111111111",
    "buyer": "2222222222"
  }]
}
```

**Note:** When `main_qaime` is provided and valid, most other fields can be empty - the portal automatically retrieves data from the advance invoice.

### Example 5: Simplified Taxation - Multi-item Services

Invoice for companies using simplified taxation (no VAT):

```json
{
  "username": "your_username",
  "password": "your_password",
  "upload": true,
  "use_old_sys": false,
  "invoices": [{
    "id": "2024030110400002",
    "main_info": "Müqavilə A001; 08.01.2024",
    "add_info": "Qaimə XYZ 251145; 28.02.2024",
    "seller": "3333333333",
    "buyer": "0000000000",
    "sum_subtotal1": 450.00,
    "sum_aksiz": 0,
    "sum_subtotal2": 450.00,
    "sum_apply_vat_amount": 0,
    "sum_skip_vat_amount": 0,
    "sum_vat_zero": 0,
    "sum_vat_free": 0,
    "sum_vat_amount": 0,
    "sum_ttax": 0,
    "sum_total": 450.00,
    "items": [
      {
        "id": "9965111040",
        "name": "Texniki xidmət № QH2222219499/9",
        "unit": "ədəd",
        "unit_price": 250.00,
        "quantity": 1,
        "barcode": "9965111040000",
        "subtotal1": 250.00,
        "aksiz": 0,
        "aksizgrade": 0,
        "subtotal2": 250.00,
        "apply_vat_amount": 0,
        "skip_vat_amount": 0,
        "vat_zero": 0,
        "vat_free": 0,
        "vat_amount": 0,
        "transport_tax": 0,
        "total": 250.00
      },
      {
        "id": "9965111041",
        "name": "Konsultasiya xidməti № QH1234535788/1",
        "unit": "saat",
        "unit_price": 50.00,
        "quantity": 4,
        "barcode": "9965111041000",
        "subtotal1": 200.00,
        "aksiz": 0,
        "aksizgrade": 0,
        "subtotal2": 200.00,
        "apply_vat_amount": 0,
        "skip_vat_amount": 0,
        "vat_zero": 0,
        "vat_free": 0,
        "vat_amount": 0,
        "transport_tax": 0,
        "total": 200.00
      }
    ]
  }]
}
```

**Note:** For simplified taxation, all VAT fields are set to 0.

### Example 6: Goods Transfer Between Warehouses

Internal transfer of goods between company's own warehouses:

```json
{
  "username": "your_username",
  "password": "your_password",
  "upload": true,
  "use_old_sys": false,
  "invoices": [{
    "id": "2025011816700001",
    "type": "goodsTransfer",
    "main_info": "Anbar arası mal köçürülməsi",
    "add_info": "Bakı anbarından Gəncə anbarına",
    "seller": "1111111111",
    "buyer": "1111111111",
    "transp_reg_number": "10-AA-123",
    "from_wh": "WH001",
    "to_wh": "WH002",
    "wh_field": 1,
    "sum_subtotal1": 8500.00,
    "sum_aksiz": 0,
    "sum_subtotal2": 8500.00,
    "sum_apply_vat_amount": 0,
    "sum_skip_vat_amount": 0,
    "sum_vat_zero": 0,
    "sum_vat_free": 0,
    "sum_vat_amount": 0,
    "sum_ttax": 0,
    "sum_total": 8500.00,
    "items": [
      {
        "id": "7777777001",
        "name": "Plastik qablaşdırma materialı",
        "unit": "rulon",
        "unit_price": 50.00,
        "quantity": 100,
        "barcode": "7777777001000",
        "subtotal1": 5000.00,
        "aksiz": 0,
        "aksizgrade": 0,
        "subtotal2": 5000.00,
        "apply_vat_amount": 0,
        "skip_vat_amount": 0,
        "vat_zero": 0,
        "vat_free": 0,
        "vat_amount": 0,
        "transport_tax": 0,
        "total": 5000.00
      },
      {
        "id": "7777777002",
        "name": "Karton qutular (50x50x50)",
        "unit": "ədəd",
        "unit_price": 7.00,
        "quantity": 500,
        "barcode": "7777777002000",
        "subtotal1": 3500.00,
        "aksiz": 0,
        "aksizgrade": 0,
        "subtotal2": 3500.00,
        "apply_vat_amount": 0,
        "skip_vat_amount": 0,
        "vat_zero": 0,
        "vat_free": 0,
        "vat_amount": 0,
        "transport_tax": 0,
        "total": 3500.00
      }
    ]
  }]
}
```

**Note:** For goods transfer, `seller` and `buyer` are the same VOEN (internal transfer). `wh_field: 1` means lookup by warehouse code.

### Example 7: Partner Integration - Multi-item Invoice

Partner managing invoices for their customers (requires `phone` parameter):

```json
{
  "username": "partner_username",
  "password": "partner_password",
  "phone": "994501234567",
  "upload": true,
  "invoices": [{
    "id": "PTR-2025-001-0045",
    "main_info": "Müştəri sifarişi #0045",
    "add_info": "Partner vasitəsilə",
    "seller": "4444444444",
    "buyer": "5555555555",
    "sum_subtotal1": 1500.00,
    "sum_aksiz": 0,
    "sum_subtotal2": 1500.00,
    "sum_apply_vat_amount": 1500.00,
    "sum_skip_vat_amount": 0,
    "sum_vat_zero": 0,
    "sum_vat_free": 0,
    "sum_vat_amount": 270.00,
    "sum_ttax": 0,
    "sum_total": 1770.00,
    "items": [
      {
        "id": "8888888001",
        "name": "Notebook Lenovo ThinkPad",
        "unit": "ədəd",
        "unit_price": 800.00,
        "quantity": 1,
        "barcode": "8888888001000",
        "subtotal1": 800.00,
        "aksiz": 0,
        "aksizgrade": 0,
        "subtotal2": 800.00,
        "apply_vat_amount": 800.00,
        "skip_vat_amount": 0,
        "vat_zero": 0,
        "vat_free": 0,
        "vat_amount": 144.00,
        "transport_tax": 0,
        "total": 944.00
      },
      {
        "id": "8888888002",
        "name": "Simsiz siçan Logitech",
        "unit": "ədəd",
        "unit_price": 35.00,
        "quantity": 2,
        "barcode": "8888888002000",
        "subtotal1": 70.00,
        "aksiz": 0,
        "aksizgrade": 0,
        "subtotal2": 70.00,
        "apply_vat_amount": 70.00,
        "skip_vat_amount": 0,
        "vat_zero": 0,
        "vat_free": 0,
        "vat_amount": 12.60,
        "transport_tax": 0,
        "total": 82.60
      },
      {
        "id": "8888888003",
        "name": "USB flash 64GB",
        "unit": "ədəd",
        "unit_price": 15.00,
        "quantity": 42,
        "barcode": "8888888003000",
        "subtotal1": 630.00,
        "aksiz": 0,
        "aksizgrade": 0,
        "subtotal2": 630.00,
        "apply_vat_amount": 630.00,
        "skip_vat_amount": 0,
        "vat_zero": 0,
        "vat_free": 0,
        "vat_amount": 113.40,
        "transport_tax": 0,
        "total": 743.40
      }
    ]
  }]
}
```

**Note:** Partners must include the `phone` parameter (format: 994XXXXXXXXX) for authentication.

### Example 8: Invoice with Excise Tax

For goods subject to excise tax (alcohol, tobacco, fuel):

```json
{
  "username": "your_username",
  "password": "your_password",
  "upload": true,
  "use_old_sys": false,
  "invoices": [{
    "id": "2025011917800001",
    "main_info": "Aksizli məhsullar satışı",
    "add_info": "Müqavilə ALC-2025/015",
    "seller": "1111111111",
    "buyer": "6666666666",
    "sum_subtotal1": 2000.00,
    "sum_aksiz": 800.00,
    "sum_subtotal2": 2800.00,
    "sum_apply_vat_amount": 2800.00,
    "sum_skip_vat_amount": 0,
    "sum_vat_zero": 0,
    "sum_vat_free": 0,
    "sum_vat_amount": 504.00,
    "sum_ttax": 0,
    "sum_total": 3304.00,
    "items": [
      {
        "id": "2208301100",
        "name": "Viski (40% alkoqol) 0.7L",
        "unit": "şüşə",
        "unit_price": 50.00,
        "quantity": 30,
        "barcode": "2208301100000",
        "subtotal1": 1500.00,
        "aksiz": 600.00,
        "aksizgrade": 40,
        "subtotal2": 2100.00,
        "apply_vat_amount": 2100.00,
        "skip_vat_amount": 0,
        "vat_zero": 0,
        "vat_free": 0,
        "vat_amount": 378.00,
        "transport_tax": 0,
        "total": 2478.00
      },
      {
        "id": "2402201000",
        "name": "Siqaret (filter) 20 ədəd qab",
        "unit": "blok",
        "unit_price": 25.00,
        "quantity": 20,
        "barcode": "2402201000000",
        "subtotal1": 500.00,
        "aksiz": 200.00,
        "aksizgrade": 40,
        "subtotal2": 700.00,
        "apply_vat_amount": 700.00,
        "skip_vat_amount": 0,
        "vat_zero": 0,
        "vat_free": 0,
        "vat_amount": 126.00,
        "transport_tax": 0,
        "total": 826.00
      }
    ]
  }]
}
```

**Note:** Formula: `subtotal1 + aksiz = subtotal2`, then VAT is applied to `subtotal2`.

### Example 9: Token-based Authentication

Using API token instead of username/password:

```bash
curl -X POST https://company.vatportal.az/api/inv/import_upload_invoices.php \
  -H "x-vatpapikey: YOUR_API_TOKEN_HERE" \
  -H "Content-Type: application/json" \
  -d '{
    "upload": true,
    "use_old_sys": false,
    "invoices": [{
      "id": "2025012018900001",
      "main_info": "Avtomatlaşdırılmış inteqrasiya",
      "add_info": "Token ilə autentifikasiya",
      "seller": "1111111111",
      "buyer": "7777777777",
      "sum_subtotal1": 350.00,
      "sum_aksiz": 0,
      "sum_subtotal2": 350.00,
      "sum_apply_vat_amount": 350.00,
      "sum_skip_vat_amount": 0,
      "sum_vat_zero": 0,
      "sum_vat_free": 0,
      "sum_vat_amount": 63.00,
      "sum_ttax": 0,
      "sum_total": 413.00,
      "items": [{
        "id": "5555555001",
        "name": "Proqram təminatı lisenziyası",
        "unit": "ədəd",
        "unit_price": 350.00,
        "quantity": 1,
        "barcode": "5555555001000",
        "subtotal1": 350.00,
        "aksiz": 0,
        "aksizgrade": 0,
        "subtotal2": 350.00,
        "apply_vat_amount": 350.00,
        "skip_vat_amount": 0,
        "vat_zero": 0,
        "vat_free": 0,
        "vat_amount": 63.00,
        "transport_tax": 0,
        "total": 413.00
      }]
    }]
  }'
```

**Note:** Token obtained from Portal → Profile → Access to API. No username/password in request body when using token.

### Example 10: Batch Upload - Multiple Invoices

Upload multiple invoices in a single request for better performance:

```json
{
  "username": "your_username",
  "password": "your_password",
  "upload": true,
  "use_old_sys": false,
  "invoices": [
    {
      "id": "2025012119000001",
      "main_info": "Müqavilə A-2025/101",
      "add_info": "Birinci qaimə",
      "seller": "1111111111",
      "buyer": "2222222222",
      "sum_subtotal1": 500.00,
      "sum_aksiz": 0,
      "sum_subtotal2": 500.00,
      "sum_apply_vat_amount": 500.00,
      "sum_skip_vat_amount": 0,
      "sum_vat_zero": 0,
      "sum_vat_free": 0,
      "sum_vat_amount": 90.00,
      "sum_ttax": 0,
      "sum_total": 590.00,
      "items": [{
        "id": "1111111001",
        "name": "Ofis mebeli - masa",
        "unit": "ədəd",
        "unit_price": 250.00,
        "quantity": 2,
        "barcode": "1111111001000",
        "subtotal1": 500.00,
        "aksiz": 0,
        "aksizgrade": 0,
        "subtotal2": 500.00,
        "apply_vat_amount": 500.00,
        "skip_vat_amount": 0,
        "vat_zero": 0,
        "vat_free": 0,
        "vat_amount": 90.00,
        "transport_tax": 0,
        "total": 590.00
      }]
    },
    {
      "id": "2025012119000002",
      "main_info": "Müqavilə B-2025/202",
      "add_info": "İkinci qaimə",
      "seller": "1111111111",
      "buyer": "3333333333",
      "sum_subtotal1": 1200.00,
      "sum_aksiz": 0,
      "sum_subtotal2": 1200.00,
      "sum_apply_vat_amount": 1200.00,
      "sum_skip_vat_amount": 0,
      "sum_vat_zero": 0,
      "sum_vat_free": 0,
      "sum_vat_amount": 216.00,
      "sum_ttax": 0,
      "sum_total": 1416.00,
      "items": [{
        "id": "2222222001",
        "name": "Printer Canon LBP",
        "unit": "ədəd",
        "unit_price": 400.00,
        "quantity": 3,
        "barcode": "2222222001000",
        "subtotal1": 1200.00,
        "aksiz": 0,
        "aksizgrade": 0,
        "subtotal2": 1200.00,
        "apply_vat_amount": 1200.00,
        "skip_vat_amount": 0,
        "vat_zero": 0,
        "vat_free": 0,
        "vat_amount": 216.00,
        "transport_tax": 0,
        "total": 1416.00
      }]
    },
    {
      "id": "2025012119000003",
      "main_info": "Müqavilə C-2025/303",
      "add_info": "Üçüncü qaimə - Sadələşdirilmiş",
      "seller": "4444444444",
      "buyer": "5555555555",
      "sum_subtotal1": 300.00,
      "sum_aksiz": 0,
      "sum_subtotal2": 300.00,
      "sum_apply_vat_amount": 0,
      "sum_skip_vat_amount": 0,
      "sum_vat_zero": 0,
      "sum_vat_free": 0,
      "sum_vat_amount": 0,
      "sum_ttax": 0,
      "sum_total": 300.00,
      "items": [{
        "id": "3333333001",
        "name": "Təmizlik xidməti",
        "unit": "xidmət",
        "unit_price": 300.00,
        "quantity": 1,
        "barcode": "3333333001000",
        "subtotal1": 300.00,
        "aksiz": 0,
        "aksizgrade": 0,
        "subtotal2": 300.00,
        "apply_vat_amount": 0,
        "skip_vat_amount": 0,
        "vat_zero": 0,
        "vat_free": 0,
        "vat_amount": 0,
        "transport_tax": 0,
        "total": 300.00
      }]
    }
  ]
}
```

**Benefits:**
- Reduced API calls
- Single process ID to monitor
- Better performance for bulk operations

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

See [complete error codes →](./error-codes.md)

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

Monitor progress using the [`GET /job/read_proc_status`](./polling-guide.md) endpoint. See [Polling Best Practices](./polling-guide.md) for recommended implementation.

## Notes

- **Minimum ID Length:** Invoice ID must be at least 7 characters
- **VOEN Format:** Must be exactly 10 digits
- **Barcode:** Can be product ID + "000" suffix
- **VAT Calculation:** System validates that sums match
- **Portal Validation:** Only format/structure is validated, not business rules
- **E-taxes Validation:** Full validation happens when uploading (if `upload: true`)

## Next Steps

- [View real-world examples →](./examples.md)
- [Invoice types reference →](./invoice-types.md)
- [Error codes reference →](./error-codes.md)
- [Authentication guide →](./authentication.md)

---

[← Back to Endpoints](./README.md)
