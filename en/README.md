# VatPortal API Documentation

**Automate your Azerbaijan e-taxes workflow**

Connect your ERP system directly to e-taxes.gov.az. Upload invoices, sign electronically, and retrieve qaime numbers automatically.

---

## 👋 New to VatPortal API?

**Get started in 5 minutes** • No complex setup • Production-ready

[**🚀 Quick Start Guide**](./quickstart.md) &nbsp;&nbsp;&nbsp; [**🤝 I'm a Partner**](./partner-integration.md) &nbsp;&nbsp;&nbsp; [**📞 Get Support**](#support)

### 🧪 Interactive API Testing

**[Try the API in your browser →](../swagger.html)**

Test all endpoints with your own subdomain and credentials using our interactive Swagger UI documentation.


---

## Choose Your Integration Path

### 🏢 Enterprise Customer
**Single company integration** • Direct control • Simple authentication

Perfect if you're integrating **one company's ERP** with e-taxes.

**Your subdomain:** `yourcompany.vatportal.az`

[**Start Enterprise Integration →**](./quickstart.md)

### 🤝 Partner / Service Provider
**Multi-customer integration** • Bulk operations • Customer segregation

Perfect if you **manage multiple companies** and need to operate on their behalf.

**Your subdomain:** `partnername.vatportal.az` • Requires `phone` parameter

[**Start Partner Integration →**](./partner-integration.md)

---

## 🎯 What do you want to do?

**Most common tasks:**

- **📤 Upload invoices to e-taxes** → [Import & Upload Guide](./import-upload-invoices.md)
- **📥 Download invoices from e-taxes** → [Download Guide](./download-from-etaxes.md)
- **🔐 Setup authentication & tokens** → [Authentication Setup](./authentication.md)
- **⚠️ Troubleshoot errors** → [Error Codes Reference](./error-codes.md)
- **📋 See real examples** → [Code Examples](./examples.md)

---

## 🔑 Key Features

- ✅ Import and upload invoices to e-taxes.gov.az
- ✅ Electronic signature support
- ✅ Retrieve qaime numbers and invoice data
- ✅ Download invoices from online system
- ✅ Advanced filtering by status and invoice type (v8.0)
- ✅ Delete draft and pending invoices
- ✅ Token-based authentication with 2FA renewal
- ✅ Support for 18% VAT and simplified taxation

## 🆕 What's New in v8.0

**Released:** August 7, 2025

- Added filtering by **invoice status** to download method
- Added filtering by **invoice kind/type** to download method
- Enhanced `inc_status` and `exc_status` parameters
- Enhanced `inc_kinds` and `exc_kinds` parameters
- Better control over bulk invoice downloads

[Full changelog →](./CHANGELOG.md)

---

**Version:** 8.0 • **Last Updated:** August 7, 2025 • **Base URL:** `https://<company>.vatportal.az/api`

## 📋 Documentation Navigation

### 🚀 Getting Started
- [**Quick Start**](./quickstart.md) - Get up and running in 5 minutes
- [**Authentication**](./authentication.md) - Setup credentials and tokens (username/password and token-based)
- [**Partner Integration**](./partner-integration.md) - Multi-customer integration for partners

### 📡 API Endpoints

#### Invoice Operations
- [**Import & Upload Invoices**](./import-upload-invoices.md) - Upload invoices to e-taxes.gov.az
- [**Download from E-taxes**](./download-from-etaxes.md) - Download invoices from e-taxes.gov.az
- [**Delete E-taxes Invoices**](./delete-etaxes-invoices.md) - Delete invoices from portal and e-taxes

#### Reference Documentation
- [**Invoice Types**](./invoice-types.md) - All supported invoice types and their usage
- [**Error Codes**](./error-codes.md) - Complete error reference and troubleshooting
- [**Polling Best Practices**](./polling-guide.md) - How to efficiently poll for process status
- [**Examples**](./examples.md) - Real-world request and response examples

### 📚 Additional Resources
- [**Changelog**](./CHANGELOG.md) - Version history and release notes
- [**Documentation Guide**](./DOCUMENTATION_GUIDE.md) - How to contribute to documentation

## 📡 Available Endpoints

### Invoice Management
- [`POST /inv/import_upload_invoices.php`](./import-upload-invoices.md#endpoint) - Import and upload invoices
- [`GET /inv/read_invoices`](./import-upload-invoices.md#read-invoices-with-qaime-numbers) - Read invoices with qaime numbers
- [`DELETE /inv/delete`](./import-upload-invoices.md#delete-unsubmitted-invoices) - Delete unsubmitted invoices by ERP ID

### Process Monitoring
- [`GET /job/read_proc_status`](./polling-guide.md) - Check process status
- [`GET /job/read_proc_data`](./polling-guide.md) - Get detailed process data

💡 **Learn best practices:** [Polling Guide](./polling-guide.md) - Recommended polling intervals, exponential backoff, and error handling

### E-taxes Integration
- [`POST /etx/import`](./download-from-etaxes.md#step-1-start-download-process) - Download qaimes from online system (v7.0)
- [`GET /etx/read_invoices`](./download-from-etaxes.md#step-2-read-downloaded-invoice-data) - Read raw qaimes data (v7.0+)
- [`DELETE /etx/delete`](./delete-etaxes-invoices.md#endpoint) - Delete qaimes by serie/number

### Authentication
- [`GET /auth/reset_start.php`](./authentication.md#step-1-initiate-token-renewal) - Initiate token renewal
- [`POST /auth/reset_confirm.php`](./authentication.md#step-2-confirm-token-renewal) - Confirm token renewal

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

## 🛠️ SDKs and Tools

### Interactive API Documentation

- **[Swagger UI (OpenAPI 3.0)](../swagger.html)** - Test API endpoints directly in your browser with your subdomain
- **[OpenAPI Specification](../openapi.yaml)** - Download the complete API specification

### Community SDKs

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
