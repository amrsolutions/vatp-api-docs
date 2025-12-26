# VatPortal API Documentation

**Version:** 8.0  
**Last Updated:** August 7, 2025  
**Base URL:** `https://<company>.vatportal.az/api`

## 🚀 What is VatPortal API?

VatPortal API enables third-party ERP systems to programmatically interact with Azerbaijan's e-taxes.gov.az system. Upload invoices, sign them electronically, retrieve qaime numbers, and manage tax documents through a RESTful JSON API.

## 📋 Documentation Navigation

### 🚀 Getting Started
- [**Quick Start**](./quickstart.md) - Get up and running in 5 minutes
- [**Authentication**](./authentication.md) - Setup credentials and tokens (username/password and token-based)

### 📡 API Endpoints

#### Invoice Operations
- [**Import & Upload Invoices**](./import-upload-invoices.md) - Upload invoices to e-taxes.gov.az
- [**Download from E-taxes**](./download-from-etaxes.md) - Download invoices from e-taxes.gov.az
- [**Delete E-taxes Invoices**](./delete-etaxes-invoices.md) - Delete invoices from portal and e-taxes

#### Reference Documentation
- [**Invoice Types**](./invoice-types.md) - All supported invoice types and their usage
- [**Error Codes**](./error-codes.md) - Complete error reference and troubleshooting
- [**Examples**](./examples.md) - Real-world request and response examples

### 📚 Additional Resources
- [**Changelog**](./CHANGELOG.md) - Version history and release notes
- [**Documentation Guide**](./DOCUMENTATION_GUIDE.md) - How to contribute to documentation

## 🔑 Key Features

- ✅ Import and upload invoices to e-taxes.gov.az
- ✅ Electronic signature support
- ✅ Retrieve qaime numbers and invoice data
- ✅ Download invoices from online system
- ✅ Advanced filtering by status and invoice type (v8.0)
- ✅ Delete draft and pending invoices
- ✅ Token-based authentication with 2FA renewal
- ✅ Support for 18% VAT and simplified taxation

## 📡 Available Endpoints

### Invoice Management
- `POST /inv/import_upload_invoices.php` - Import and upload invoices
- `GET /inv/read_invoices` - Read invoices with qaime numbers
- `DELETE /inv/delete` - Delete unsubmitted invoices by ERP ID

### Process Monitoring
- `GET /job/read_proc_status` - Check process status
- `GET /job/read_proc_data` - Get detailed process data

### E-taxes Integration
- `POST /etx/import` - Download qaimes from online system (v7.0)
- `GET /etx/read_invoices` - Read raw qaimes data (v7.0+)
- `DELETE /etx/delete` - Delete qaimes by serie/number

### Authentication
- `GET /auth/reset_start.php` - Initiate token renewal
- `POST /auth/reset_confirm.php` - Confirm token renewal

## 🌍 Supported Invoice Types

| Type | Description |
|------|-------------|
| `defaultInvoice` | Standard goods/services invoice |
| `advanceInvoice` | Advance payment invoice |
| `goodsTransfer` | Goods transfer between warehouses |
| `returnInvoice` | Product return invoice |
| `agent` | Agent/commission sales |
| `exportNoteInvoice` | Export with export note |

[See all invoice types →](./invoice-types.md)

## 🔐 Authentication Methods

1. **Username/Password** - Include in request body
2. **Token Header** - Send `x-vatpapikey` HTTP header

```bash
# Example with token
curl -X POST https://company.vatportal.az/api/inv/import_upload_invoices.php \
  -H "x-vatpapikey: YOUR_TOKEN_HERE" \
  -H "Content-Type: application/json" \
  -d @request.json
```

## 📊 Response Format

All endpoints return JSON with standard structure:

```json
{
  "res": 1,
  "err_code": 0,
  "err_msg": "",
  "data": {
    // Endpoint-specific data
  }
}
```

- `res`: Success indicator (1 = success, 0 = error)
- `err_code`: Error code (0 = no error, see [error codes](./error-codes.md))
- `err_msg`: Human-readable error message
- `data`: Response payload

## 🆕 What's New in v8.0

**Released:** August 7, 2025

- Added filtering by **invoice status** to download method
- Added filtering by **invoice kind/type** to download method
- Enhanced `inc_status` and `exc_status` parameters
- Enhanced `inc_kinds` and `exc_kinds` parameters
- Better control over bulk invoice downloads

[Full changelog →](./CHANGELOG.md)

## 🛠️ SDKs and Tools

Currently, VatPortal API requires direct HTTP requests. Community SDKs:

- **JavaScript/TypeScript** - Coming soon
- **Python** - Coming soon
- **C#/.NET** - Coming soon

Want to contribute an SDK? [Contact support](#support)

## 📞 Support

- **Documentation Issues:** [Open an issue](#)
- **API Support:** Contact your VatPortal account manager
- **Technical Questions:** support@amr.az

## 📄 License & Usage

This API is available to registered VatPortal subscribers. Credentials are provided upon subscription completion.

---

**Built by:** AMR Solutions  
**API Version:** 8.0  
**Documentation Version:** 8.0.0
