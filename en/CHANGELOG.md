# Changelog

All notable changes to VatPortal API are documented here.

## [8.0] - 2025-08-07

### Added
- **Filtering by invoice status** in download method (`/api/etx/import`)
  - New parameters: `inc_status` and `exc_status`
  - Download only specific invoice statuses (e.g., "approved", "onApproval")
  - Exclude unwanted statuses (e.g., "canceled", "deleted")
  
- **Filtering by invoice type/kind** in download method (`/api/etx/import`)
  - New parameters: `inc_kinds` and `exc_kinds`
  - Download only specific invoice types (e.g., "defaultInvoice", "advanceInvoice")
  - Exclude unwanted types (e.g., "returnInvoice")

### Use Cases
```json
// Download only approved default invoices
{
  "inc_status": "approved",
  "inc_kinds": "defaultInvoice"
}

// Download all except returns and canceled
{
  "exc_kinds": "returnInvoice,returnByAgent",
  "exc_status": "canceled,deleted"
}
```

### Breaking Changes
None - all new parameters are optional and backward compatible.

---

## [7.0] - 2025-06-27

### Added
- **Download qaimes from online system** (`/api/etx/import`)
  - Fetch invoices directly from e-taxes.gov.az
  - Returns process ID for monitoring
  - Requires ASAN phone authentication
  
- **Read raw qaime data** (`/api/etx/read_invoices`)
  - Get complete invoice details including all line items
  - Filter by process ID or date range
  - Returns full structured invoice data

### Use Cases
```json
// Start download
POST /api/etx/import
{
  "phone": "994501234567",
  "date_from": "01.01.2025",
  "date_to": "31.01.2025",
  "dir": 1
}

// Read downloaded data
GET /api/etx/read_invoices
{
  "phone": "994501234567",
  "procId": 123
}
```

### Breaking Changes
None - entirely new endpoints.

---

## [6.0] - 2024-11-01

### Added
- **Delete unsubmitted invoices by ERP ID** (`/api/inv/delete`)
  - Remove invoices that haven't been submitted to e-taxes
  - Bulk deletion support
  - Prevents accidental deletion of submitted invoices

### Use Cases
```json
// Delete draft invoices
DELETE /api/inv/delete
{
  "ids": ["INV-2024-001", "INV-2024-002"]
}
```

---

## [5.0] - 2024-10-19

### Added
- **Authentication chapter** in documentation
  - Comprehensive guide for both auth methods
  - Token renewal flow with examples
  - Security best practices

---

## [4.0] - 2024-10-04

### Added
- **Token-based authentication**
  - Use `x-vatpapikey` HTTP header instead of username/password
  - Improved security (no credentials in request body)
  - Easier credential rotation

### Migration Guide
```bash
# Old method (still supported)
curl -X POST /api/inv/read_invoices \
  -d '{"username":"user","password":"pass",...}'

# New method (recommended)
curl -X POST /api/inv/read_invoices \
  -H "x-vatpapikey: YOUR_TOKEN" \
  -d '{...}'
```

---

## [3.3] - 2024-08-13

### Fixed
- **Double quotes in sample request** corrected

### Added
- **Warehouse mapping parameter** (`wh_field`)
  - Map warehouses by code (`wh_field: 1`) or name (`wh_field: 2`)
  - Used for goods transfer invoices

---

## [3.2] - 2024-08-03

### Added
Four new optional fields for invoice import:
- Expanded data collection for goods transfer
- Better support for complex invoice scenarios

---

## [3.1] - 2024-07-22

### Added
Two additional optional fields for invoice import method:
- Enhanced invoice metadata support

---

## [3.0] - 2024-07-05

### Added
- **New API endpoints** (specific endpoints not documented)

---

## [2.6] - 2024-04-02

### Added
- **New error code** for better error handling

---

## [2.5] - 2024-03-14

### Added
- **Example for Read Invoices with Qaime Numbers API**
- **Example for request with no VAT** (simplified taxation)

---

## [2.4] - 2024-03-12

### Added
- **API call to retrieve invoice IDs with corresponding qaime numbers**
  - `/api/inv/read_invoices` endpoint
  - Map ERP invoice IDs to e-taxes qaime numbers

---

## [2.3] - 2024-03-04

### Added
- **Example request with result qaime**

---

## [2.2] - 2024-03-01

### Added
- **Localized descriptions** of input fields
  - Added Azerbaijani translations for parameter names

---

## [2.1] - 2024-01-20

### Added
- **Additional parameter** to import/upload method

---

## [2.0] - 2021-06-10

### Added
- **Error codes** for all API methods
- **Sample requests and responses with errors**

---

## [1.0] - 2021-06-01

### Initial Release
- **Import and upload invoices** to e-taxes
- **Read process status** endpoint
- **Read process data** endpoint
- **Username/password authentication**
- Basic invoice structure with VAT support

---

## Migration Guides

### Upgrading to v8.0 from v7.0

No breaking changes. If you use `/api/etx/import`, you can now add filters:

```diff
{
  "phone": "994501234567",
  "date_from": "01.01.2025",
  "date_to": "31.01.2025",
  "dir": 1,
+ "inc_status": "approved",
+ "inc_kinds": "defaultInvoice"
}
```

### Upgrading to v7.0 from v6.0

New endpoints available. No changes to existing endpoints.

### Upgrading to v4.0 from v3.x

**Recommended:** Switch to token authentication:

1. Generate token in portal (Profile → Access to API)
2. Replace username/password with header:
   ```bash
   -H "x-vatpapikey: YOUR_TOKEN"
   ```
3. Remove `username` and `password` from request body

Old method still works but token is more secure.

## Version Support Policy

- **Current version (8.0):** Fully supported
- **Previous version (7.0):** Supported for 6 months after 8.0 release
- **Older versions:** May work but not officially supported

## Deprecation Notices

**None currently.** Username/password authentication will continue to be supported alongside token authentication.

## Planned Features

🔮 Roadmap (subject to change):

- Webhook notifications for invoice status changes
- Batch operations API for bulk invoice processing
- Official SDKs (JavaScript, Python, C#)
- GraphQL endpoint alternative
- Enhanced error messages with detailed validation feedback

## Feedback & Feature Requests

Have suggestions for future versions?

- Contact your account manager
- Email: support@amrsolutions.az
- Use the thumbs down/up buttons in portal

---

[← Back to Documentation](./README.md)
