# Swagger UI Setup Guide

This guide explains how to set up GitHub Pages to host your interactive Swagger UI documentation.

## Files Created

1. **`openapi.yaml`** - Complete OpenAPI 3.0 specification with all VatPortal API endpoints
2. **`swagger.html`** - Interactive Swagger UI interface with dynamic subdomain configuration
3. **Updated READMEs** - Added links to Swagger UI in both English and Azerbaijani documentation

## GitHub Pages Setup

### Step 1: Enable GitHub Pages

1. Go to your GitHub repository settings
2. Navigate to **Settings** → **Pages**
3. Under "Source", select:
   - **Branch:** `main` (or your default branch)
   - **Folder:** `/apidocs` (or `/` if apidocs is your root)
4. Click **Save**

### Step 2: Access Your Documentation

After GitHub Pages is enabled, your documentation will be available at:

```
https://yourusername.github.io/your-repo-name/swagger.html
```

Or if you're using a custom domain:

```
https://docs.yourcompany.com/swagger.html
```

### Step 3: Test the Swagger UI

1. Open the Swagger URL in your browser
2. You'll see a yellow configuration box at the top
3. Enter your company subdomain (e.g., "mycompany" for `mycompany.vatportal.az`)
4. Click "Update Server"
5. The server URL will update to your subdomain
6. You can now test all API endpoints with your credentials

## Features

### Dynamic Subdomain Configuration

- Users can enter their own company or partner subdomain
- Subdomain is saved in browser localStorage
- Persists across page refreshes
- Works for both enterprise customers and partners

### Interactive Testing

- Try all API endpoints directly in the browser
- Supports both authentication methods:
  - Username/Password in request body
  - `x-vatpapikey` header token
- See real-time request/response examples
- Copy curl commands
- Download OpenAPI specification

### Complete API Coverage

The OpenAPI spec includes:
- ✅ Import & Upload Invoices (`POST /inv/import_upload_invoices.php`)
- ✅ Read Invoices (`POST /inv/read_invoices`)
- ✅ Download from E-taxes (`POST /etx/import`)
- ✅ Read Downloaded Invoices (`GET /etx/read_invoices`)
- ✅ Delete E-taxes Invoices (`DELETE /etx/delete`)
- ✅ Read Process Status (`GET /job/read_proc_status`)
- ✅ Read Process Data (`GET /job/read_proc_data`)

## Customization

### Update OpenAPI Specification

Edit `openapi.yaml` to:
- Add new endpoints
- Update examples
- Modify descriptions
- Change schema definitions

After editing, changes will be reflected immediately in Swagger UI.

### Customize Swagger UI

Edit `swagger.html` to:
- Change color scheme (update CSS in `<style>` section)
- Modify subdomain validation logic
- Add custom branding
- Change Swagger UI version (update CDN URLs)

## Using with Different Tools

### Import into Postman

1. Download the OpenAPI spec: `https://your-pages-url/openapi.yaml`
2. In Postman: **Import** → **Link** → Paste the URL
3. All endpoints will be imported as a collection

### Generate Client SDKs

Use [OpenAPI Generator](https://openapi-generator.tech/):

```bash
# JavaScript/TypeScript
openapi-generator-cli generate -i openapi.yaml -g typescript-axios -o ./client

# Python
openapi-generator-cli generate -i openapi.yaml -g python -o ./client

# C#
openapi-generator-cli generate -i openapi.yaml -g csharp -o ./client
```

### Use with Other API Tools

- **Stoplight**: Import the OpenAPI spec
- **SwaggerHub**: Upload the YAML file
- **Insomnia**: Import as OpenAPI 3.0 spec
- **VS Code REST Client**: Use with OpenAPI extension

## Troubleshooting

### Swagger UI not loading

1. Check that GitHub Pages is enabled
2. Verify the branch and folder settings
3. Wait a few minutes for GitHub Pages to deploy
4. Check browser console for errors

### CORS Issues

If testing from localhost:
- GitHub Pages should work without CORS issues
- If you see CORS errors, use the actual API endpoints (not from Swagger)

### Subdomain not updating

1. Clear browser localStorage
2. Hard refresh the page (Ctrl+Shift+R or Cmd+Shift+R)
3. Check browser console for JavaScript errors

## Best Practices

### For Enterprise Customers

1. Enter your company subdomain once
2. Use token authentication (`x-vatpapikey` header) instead of username/password
3. Test with small data sets first
4. Monitor process status using the polling endpoints

### For Partners

1. Enter your partner subdomain (not customer subdomain)
2. Always include the `phone` parameter in requests
3. Use the partner-specific examples in the OpenAPI spec
4. Test with one customer first before bulk operations

## Security Note

⚠️ **Never commit sensitive credentials to the repository**

- The Swagger UI is public (on GitHub Pages)
- Do not hardcode API tokens or passwords
- Users should enter their own credentials when testing
- Token authentication is recommended over username/password

## Updates and Maintenance

To update the documentation:

1. Edit `openapi.yaml` or `swagger.html`
2. Commit and push to GitHub
3. GitHub Pages will auto-deploy the changes
4. Changes will be live in a few minutes

## Support

For issues with:
- **OpenAPI specification**: Check the [OpenAPI 3.0 spec](https://swagger.io/specification/)
- **Swagger UI**: See [Swagger UI documentation](https://swagger.io/tools/swagger-ui/)
- **GitHub Pages**: See [GitHub Pages docs](https://docs.github.com/en/pages)

---

**Built by:** AMR Solutions
**Last Updated:** December 28, 2025
