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
| `phone` | ASAN phone number for e-taxes authentication (format: 994XXXXXXXXX) | Yes | - |
| `date_from` | Start date when qaime was created in e-taxes (dd.mm.yyyy or dd.mm.yyyy hh:mm) | Yes | - |
| `date_to` | End date when qaime was created in e-taxes (dd.mm.yyyy or dd.mm.yyyy hh:mm) | Yes | - |
| `dir` | Folder type: `0` = Incoming, `1` = Sent | Yes | `0` |
| `inc_kinds` | **[v8.0]** Include only these invoice types (comma-separated) | No | - |
| `exc_kinds` | **[v8.0]** Exclude these invoice types (comma-separated) | No | - |
| `inc_status` | **[v8.0]** Include only these statuses (comma-separated) | No | - |
| `exc_status` | **[v8.0]** Exclude these statuses (comma-separated) | No | - |

**Note:** `inc_kinds` overrides `exc_kinds`. `inc_status` overrides `exc_status`.

### Request Example

```json
{
  "username": "myusername",
  "password": "mypassword",
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
  "err_code": 0,
  "err_msg": "",
  "data": {
    "id": 123
  }
}
```

The `id` field contains the process ID. Use this to monitor the download progress with the [Read Process Status](./import-upload-invoices.md#monitor-process) endpoint.

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
| `phone` | ASAN phone number (format: 994XXXXXXXXX) | Yes |
| `procId` | Process ID from download request | Yes |
| `dir` | Folder type: `0` = Incoming, `1` = Sent | No (default: 0) |

#### Option B: Search by Date Range

| Parameter | Description | Required |
|-----------|-------------|----------|
| `username` | Portal username | Yes |
| `password` | Portal password | Yes |
| `phone` | ASAN phone number (format: 994XXXXXXXXX) | Yes |
| `date_from` | Start date (dd.mm.yyyy) | Yes |
| `date_to` | End date (dd.mm.yyyy) | Yes |
| `dir` | Folder type: `0` = Incoming, `1` = Sent | No (default: 0) |
| `from` | Page number (starts at 1) | No (default: 1) |

**Pagination:** Results are returned in pages of 5000 records.

### Request Example (by Process ID)

```json
{
  "username": "myusername",
  "password": "mypassword",
  "phone": "994501234567",
  "procId": 123,
  "dir": 1
}
```

### Request Example (by Date Range)

```json
{
  "username": "myusername",
  "password": "mypassword",
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

## Complete Workflow Example

```bash
# Step 1: Start download process
curl -X POST https://mycompany.vatportal.az/api/etx/import \
  -H "Content-Type: application/json" \
  -d '{
    "username": "myuser",
    "password": "mypass",
    "phone": "994501234567",
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
    "phone": "994501234567",
    "procId": 123,
    "dir": 1
  }'
```

---

## Important Notes

- **Phone Number Required:** You must configure an ASAN phone number in the portal before using this feature
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
