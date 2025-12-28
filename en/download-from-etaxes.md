# Download Invoices from E-taxes

**Available since:** v7.0 (June 27, 2025)
**Enhanced in:** v8.0 (August 7, 2025)

## Overview

The VatPortal API allows you to download invoices (qaimes) directly from Azerbaijan's e-taxes.gov.az system into your portal. This is the **reverse direction** of the upload flow - instead of sending invoices TO e-taxes, you retrieve invoices FROM e-taxes.

This feature is useful for:
- Synchronizing invoices created outside your ERP system
- Retrieving invoices from business partners
- Backing up invoice data from e-taxes
- Reconciling invoice records between systems

## Two-Step Process

Downloading invoices from e-taxes involves two API calls:

1. **Start Download** (`POST /etx/import`) - Initiates the download process
2. **Read Downloaded Data** (`GET /etx/read_invoices`) - Retrieves the complete invoice data

---

## Authentication & Customer Context

### For Enterprise Customers
- **No `phone` parameter needed** - operates on your own company data
- Use `username`/`password` or token authentication
- Subdomain format: `company.vatportal.az`

### For Partners
- **`phone` parameter is REQUIRED** in all API calls
- Identifies which customer you're operating on behalf of
- Format: `994XXXXXXXXX` (ASAN phone number)
- Subdomain format: `partnername.vatportal.az`

See the [Partner Integration Guide](./partner-integration.md) for more details.

---

## Step 1: Start Download Process

### Endpoint

```
POST https://<company>.vatportal.az/api/etx/import
```

### Request Parameters

| Parameter | Description | Required | Default |
|-----------|-------------|----------|---------|
| `username` | Portal username | Yes | - |
| `password` | Portal password | Yes | - |
| `phone` | ASAN phone number (format: 994XXXXXXXXX) **[Partners only]** | Partners: Yes<br>Enterprise: No | - |
| `date_from` | Start date when qaime was created in e-taxes (dd.mm.yyyy or dd.mm.yyyy hh:mm) | Yes | - |
| `date_to` | End date when qaime was created in e-taxes (dd.mm.yyyy or dd.mm.yyyy hh:mm) | Yes | - |
| `dir` | Folder type: `0` = Incoming, `1` = Sent | Yes | `0` |
| `inc_kinds` | **[v8.0]** Include only these invoice types (comma-separated) | No | - |
| `exc_kinds` | **[v8.0]** Exclude these invoice types (comma-separated) | No | - |
| `inc_status` | **[v8.0]** Include only these statuses (comma-separated) | No | - |
| `exc_status` | **[v8.0]** Exclude these statuses (comma-separated) | No | - |

**Note:** `inc_kinds` overrides `exc_kinds`. `inc_status` overrides `exc_status`.

### Request Example

**Enterprise Customer:**
```json
{
  "username": "myusername",
  "password": "mypassword",
  "date_from": "01.08.2025",
  "date_to": "07.08.2025",
  "dir": 1,
  "inc_status": "approved,onApproval",
  "exc_kinds": "returnInvoice"
}
```

**Partner:**
```json
{
  "username": "partner_username",
  "password": "partner_password",
  "phone": "994501234567",
  "date_from": "01.08.2025",
  "date_to": "07.08.2025",
  "dir": 1,
  "inc_status": "approved,onApproval",
  "exc_kinds": "returnInvoice"
}
```

### Response

```json
{
  "res": 1,
  "err_code": 0,
  "err_msg": "",
  "data": {
    "id": 123
  }
}
```

The `id` field contains the process ID. Use this to monitor the download progress with the [`GET /job/read_proc_status`](./polling-guide.md) endpoint. See [Polling Best Practices](./polling-guide.md) for recommended implementation.

---

## Step 2: Read Downloaded Invoice Data

### Endpoint

```
GET https://<company>.vatportal.az/api/etx/read_invoices
```

### Request Parameters

You can search by either **process ID** or **date range**:

#### Option A: Search by Process ID

| Parameter | Description | Required |
|-----------|-------------|----------|
| `username` | Portal username | Yes |
| `password` | Portal password | Yes |
| `phone` | ASAN phone number (format: 994XXXXXXXXX) **[Partners only]** | Partners: Yes<br>Enterprise: No |
| `procId` | Process ID from download request | Yes |
| `dir` | Folder type: `0` = Incoming, `1` = Sent | No (default: 0) |

#### Option B: Search by Date Range

| Parameter | Description | Required |
|-----------|-------------|----------|
| `username` | Portal username | Yes |
| `password` | Portal password | Yes |
| `phone` | ASAN phone number (format: 994XXXXXXXXX) **[Partners only]** | Partners: Yes<br>Enterprise: No |
| `date_from` | Start date (dd.mm.yyyy) | Yes |
| `date_to` | End date (dd.mm.yyyy) | Yes |
| `dir` | Folder type: `0` = Incoming, `1` = Sent | No (default: 0) |
| `from` | Page number (starts at 1) | No (default: 1) |

**Pagination:** Results are returned in pages of 5000 records.

### Request Example (by Process ID)

**Enterprise Customer:**
```json
{
  "username": "myusername",
  "password": "mypassword",
  "procId": 123,
  "dir": 1
}
```

**Partner:**
```json
{
  "username": "partner_username",
  "password": "partner_password",
  "phone": "994501234567",
  "procId": 123,
  "dir": 1
}
```

### Request Example (by Date Range)

**Enterprise Customer:**
```json
{
  "username": "myusername",
  "password": "mypassword",
  "date_from": "01.08.2025",
  "date_to": "07.08.2025",
  "dir": 1,
  "from": 1
}
```

**Partner:**
```json
{
  "username": "partner_username",
  "password": "partner_password",
  "phone": "994501234567",
  "date_from": "01.08.2025",
  "date_to": "07.08.2025",
  "dir": 1,
  "from": 1
}
```

### Response

```json
{
  "res": 1,
  "err_code": 0,
  "err_msg": "",
  "data": {
    "invoices": [
      {
        "id": 12345,
        "main_subject": "Contract A001",
        "add_subject": "Additional info",
        "serial": "AA",
        "number": "24080000001",
        "qaime": "AA24080000001",
        "cmp_from": "Company LLC",
        "cmp_to": "Buyer Company",
        "tin_from": "1111111111",
        "tin_to": "0000000000",
        "datetime": "07.08.2025 14:30",
        "ini_amount": 1000.00,
        "ini_amount1": 1000.00,
        "ini_amount2": 1000.00,
        "excise_amount": 0,
        "apply_amount": 1000.00,
        "skip_amount": 0,
        "zero_amount": 0,
        "free_amount": 0,
        "vat_amount": 180.00,
        "transp_amount": 0,
        "net_amount": 1180.00,
        "status": "approved",
        "type": "defaultInvoice",
        "lines": [
          {
            "id": 1,
            "line": 1,
            "prod_code": "1234567890",
            "prod_name": "Product Name",
            "barcode": "1234567890000",
            "unit_name": "pcs",
            "unit_price": 10.00,
            "unit_qty": 100,
            "ini_amount": 1000.00,
            "ini_amount2": 1000.00,
            "grade": 0,
            "total_excise": 0,
            "apply_amount": 1000.00,
            "skip_amount": 0,
            "zero_amount": 0,
            "free_amount": 0,
            "vat_amount": 180.00,
            "transp_amount": 0,
            "net_amount": 1180.00
          }
        ]
      }
    ],
    "total": 1
  }
}
```

---

## Filtering Options (v8.0)

### Filter by Invoice Status

Use `inc_status` or `exc_status` with these values:

| Status Code | Description |
|-------------|-------------|
| `approved` | Approved |
| `onApproval` | Waiting for approval |
| `updateApproval` | Update approved |
| `updateRequested` | Update requested |
| `cancelRequested` | Cancellation requested |
| `approvedBySystem` | System approved |
| `onApprovalEdited` | Sent for update |
| `canceled` | Canceled |
| `deleted` | Deleted |
| `deletedBySystem` | Deleted by system |
| `deactivated` | Deactivated |

**Example:** Download only approved and pending invoices:
```json
"inc_status": "approved,onApproval"
```

### Filter by Invoice Type

Use `inc_kinds` or `exc_kinds` with these values:

| Type Code | Description |
|-----------|-------------|
| `defaultInvoice` | Standard goods/services invoice |
| `agent` | Agent/commission invoice |
| `resale` | Agent resale invoice |
| `recycling` | Goods sent for processing |
| `taxCodex163` | Tax code 163 adjustment |
| `taxCodex177_5` | Tax code 177.5 |
| `returnInvoice` | Product return |
| `returnByAgent` | Agent return |
| `returnRecycled` | Return from processing |
| `exportNoteInvoice` | Export with export note |
| `exciseGoodsTransfer` | Excise goods transfer |
| `advanceInvoice` | Advance payment invoice |

**Example:** Exclude return invoices:
```json
"exc_kinds": "returnInvoice,returnByAgent"
```

---

## Filter Usage Examples

### Example 1: Download Only Approved Invoices

Download only invoices that have been approved, excluding those waiting for approval or updates:

**Enterprise Customer:**
```bash
curl -X POST https://company.vatportal.az/api/etx/import \
  -H "Content-Type: application/json" \
  -d '{
    "username": "myuser",
    "password": "mypass",
    "date_from": "01.01.2025",
    "date_to": "31.01.2025",
    "dir": 0,
    "inc_status": "approved"
  }'
```

**Partner:**
```bash
curl -X POST https://partnername.vatportal.az/api/etx/import \
  -H "Content-Type: application/json" \
  -d '{
    "username": "partner_user",
    "password": "partner_pass",
    "phone": "994501234567",
    "date_from": "01.01.2025",
    "date_to": "31.01.2025",
    "dir": 0,
    "inc_status": "approved"
  }'
```

### Example 2: Exclude Canceled and Deleted Invoices

Download all invoices except those that have been canceled or deleted:

**Enterprise Customer:**
```bash
curl -X POST https://company.vatportal.az/api/etx/import \
  -H "Content-Type: application/json" \
  -d '{
    "username": "myuser",
    "password": "mypass",
    "date_from": "01.01.2025",
    "date_to": "31.01.2025",
    "dir": 1,
    "exc_status": "canceled,deleted,deletedBySystem"
  }'
```

**Partner (add `phone` parameter):**
```bash
curl -X POST https://partnername.vatportal.az/api/etx/import \
  -H "Content-Type: application/json" \
  -d '{
    "username": "partner_user",
    "password": "partner_pass",
    "phone": "994501234567",
    "date_from": "01.01.2025",
    "date_to": "31.01.2025",
    "dir": 1,
    "exc_status": "canceled,deleted,deletedBySystem"
  }'
```

> **Note for Examples 3-7:** For brevity, examples below show enterprise customer format only.
> **Partners:** Add `"phone": "994XXXXXXXXX"` parameter and use subdomain `partnername.vatportal.az`.

### Example 3: Download Only Standard Invoices

Download only standard goods/services invoices, excluding all other types:

```bash
curl -X POST https://company.vatportal.az/api/etx/import \
  -H "Content-Type: application/json" \
  -d '{
    "username": "myuser",
    "password": "mypass",
    "date_from": "01.01.2025",
    "date_to": "31.01.2025",
    "dir": 0,
    "inc_kinds": "defaultInvoice"
  }'
```

### Example 4: Exclude All Return-Related Invoices

Download all invoices except various types of returns:

```bash
curl -X POST https://company.vatportal.az/api/etx/import \
  -H "Content-Type: application/json" \
  -d '{
    "username": "myuser",
    "password": "mypass",
    "date_from": "01.01.2025",
    "date_to": "31.01.2025",
    "dir": 0,
    "exc_kinds": "returnInvoice,returnByAgent,returnRecycled"
  }'
```

### Example 5: Combine Status and Type Filters

Download only approved standard invoices and advance invoices:

```bash
curl -X POST https://company.vatportal.az/api/etx/import \
  -H "Content-Type: application/json" \
  -d '{
    "username": "myuser",
    "password": "mypass",
    "date_from": "01.01.2025",
    "date_to": "31.01.2025",
    "dir": 1,
    "inc_status": "approved,approvedBySystem",
    "inc_kinds": "defaultInvoice,advanceInvoice"
  }'
```

### Example 6: Download Pending Approvals (Incoming Invoices)

Get all incoming invoices that are waiting for your approval:

```bash
curl -X POST https://company.vatportal.az/api/etx/import \
  -H "Content-Type: application/json" \
  -d '{
    "username": "myuser",
    "password": "mypass",
    "date_from": "01.12.2024",
    "date_to": "31.12.2024",
    "dir": 0,
    "inc_status": "onApproval,onApprovalEdited"
  }'
```

### Example 7: Export Invoices (No Returns, No Canceled)

Download export invoices that are active (not canceled or deleted):

```bash
curl -X POST https://company.vatportal.az/api/etx/import \
  -H "Content-Type: application/json" \
  -d '{
    "username": "myuser",
    "password": "mypass",
    "date_from": "01.01.2025",
    "date_to": "31.03.2025",
    "dir": 1,
    "inc_kinds": "exportNoteInvoice",
    "exc_status": "canceled,deleted,deletedBySystem,deactivated"
  }'
```

### Filter Priority Rules

**Important:** When using filters, remember:

1. **`inc_*` parameters override `exc_*` parameters**
   - If both `inc_status` and `exc_status` are provided, only `inc_status` is used
   - If both `inc_kinds` and `exc_kinds` are provided, only `inc_kinds` is used

2. **Example of override behavior:**
   ```json
   {
     "inc_status": "approved",
     "exc_status": "canceled"
   }
   ```
   Result: Only `inc_status` is applied, `exc_status` is ignored. You will get only approved invoices.

3. **Comma-separated values:**
   ```json
   {
     "inc_status": "approved,onApproval,updateApproval"
   }
   ```
   Result: Invoices matching ANY of these statuses will be included (OR logic).

---

## Complete Workflow Example

**Enterprise Customer:**
```bash
# Step 1: Start download process
curl -X POST https://mycompany.vatportal.az/api/etx/import \
  -H "Content-Type: application/json" \
  -d '{
    "username": "myuser",
    "password": "mypass",
    "date_from": "01.08.2025",
    "date_to": "07.08.2025",
    "dir": 1,
    "inc_status": "approved"
  }'

# Response: {"err_code": 0, "data": {"id": 123}}

# Step 2: Check process status
curl -X GET https://mycompany.vatportal.az/api/job/read_proc_status \
  -H "Content-Type: application/json" \
  -d '{
    "username": "myuser",
    "password": "mypass",
    "procId": 123
  }'

# Response: {"status": 2, "status_desc": "Process Finalized Successfully"}

# Step 3: Read downloaded invoice data
curl -X GET https://mycompany.vatportal.az/api/etx/read_invoices \
  -H "Content-Type: application/json" \
  -d '{
    "username": "myuser",
    "password": "mypass",
    "procId": 123,
    "dir": 1
  }'
```

**Partner (add `phone` parameter to Steps 1 and 3):**
```bash
# Step 1: Include phone
curl -X POST https://partnername.vatportal.az/api/etx/import \
  -d '{"username": "...", "password": "...", "phone": "994501234567", ...}'

# Step 3: Include phone
curl -X GET https://partnername.vatportal.az/api/etx/read_invoices \
  -d '{"username": "...", "password": "...", "phone": "994501234567", "procId": 123, ...}'
```

---

## Important Notes

- **Phone Parameter (Partners Only):** Partners must include the `phone` parameter to identify which customer they're operating on behalf of. Enterprise customers do not need this parameter.
- **ASAN Configuration:** ASAN phone numbers must be configured in the portal before use
- **Authentication:** The download process will require PIN1 confirmation via SMS/ASAN login
- **Pagination:** Large result sets are returned in pages of 5000 records
- **Date Format:** Use `dd.mm.yyyy` or `dd.mm.yyyy hh:mm` format
- **Filter Priority:** `inc_*` parameters override `exc_*` parameters

---

## Error Handling

Common errors when downloading from e-taxes:

| Error Code | Description | Solution |
|------------|-------------|----------|
| 4 | Empty username or password | Provide credentials |
| 5 | Invalid username or password | Check credentials |
| 15 | ASAN number not assigned | Configure ASAN phone in portal |

---

## Next Steps

- [Import & Upload Invoices →](./import-upload-invoices.md)
- [Error Codes Reference →](./error-codes.md)
- [Authentication Guide →](./authentication.md)

---

[← Back to Documentation](./README.md)
