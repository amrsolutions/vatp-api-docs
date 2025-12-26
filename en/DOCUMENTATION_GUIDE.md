# VatPortal API Documentation Guide

**Version:** 8.0
**Last Updated:** December 26, 2025

---

## 📖 Welcome to VatPortal API Documentation

This comprehensive guide will help you integrate your ERP system with Azerbaijan's e-taxes.gov.az through the VatPortal API.

## 🌍 Available Languages

This documentation is available in two languages:

- **English** - [Browse English Documentation](./README.md)
- **Azərbaycan** - [Azərbaycan Sənədləşməsinə baxın](../az/README.md)

You can switch between languages at any time from the main page.

## 📚 Documentation Structure

Our documentation is organized to help you get started quickly and find what you need:

### Getting Started
1. **[Quick Start Guide](./quickstart.md)** - Get up and running in 5 minutes
2. **[Authentication](./authentication.md)** - Set up your credentials and tokens
3. **[Examples](./examples.md)** - Real-world code samples

### API Reference
- **[Import & Upload Invoices](./import-upload-invoices.md)** - Main endpoint documentation
- **[Error Codes](./error-codes.md)** - Complete error reference with solutions
- **[Invoice Types](./invoice-types.md)** - All supported invoice types and statuses

### Additional Resources
- **[Changelog](./CHANGELOG.md)** - Version history and updates

## 🚀 Recommended Learning Path

### For New Developers
1. Start with [Quick Start Guide](./quickstart.md)
2. Review [Authentication](./authentication.md) methods
3. Try the [Examples](./examples.md)
4. Refer to [Error Codes](./error-codes.md) when troubleshooting

### For Integration Teams
1. Read [Authentication](./authentication.md) for setup
2. Study [Import & Upload Invoices](./import-upload-invoices.md) endpoint
3. Review [Invoice Types](./invoice-types.md) for business logic
4. Use [Examples](./examples.md) for implementation patterns

## 💡 Best Practices

### Security
- Always use **token-based authentication** (recommended over username/password)
- Store tokens securely in environment variables
- Never commit credentials to version control
- Use HTTPS for all API requests
- Rotate tokens regularly (every 90 days minimum)

### Error Handling
- Always check `err_code` in API responses
- Implement retry logic for network errors
- Log errors with full context for debugging
- Validate invoice data before sending to API

### Performance
- Use batch operations when possible
- Implement proper timeout handling
- Cache responses when appropriate
- Monitor API call rates

## 🆘 Getting Help

### Documentation Issues
If you find errors or have suggestions for improving this documentation:
- Contact us at: **support@amr.az**

### Technical Support
For API integration support:
- **Email:** support@amr.az
- **Response Time:** Within 24 hours on business days

### Feature Requests
Have ideas for new API features? Contact your account manager or email us.

## 📄 Terms of Use

This API documentation is provided for registered VatPortal subscribers. Usage of the API is subject to your VatPortal subscription agreement.

## 🔔 Staying Updated

### Version Updates
Check the [Changelog](./CHANGELOG.md) regularly for:
- New features and endpoints
- Breaking changes
- Security updates
- Bug fixes

### Current Version
**API Version:** 8.0
**Documentation Version:** 8.0.0

---

**Need Help?** Contact us at support@amr.az

**Built by AMR Solutions**
