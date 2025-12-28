# Updating Documentation Links in OpenAPI Spec

Once you've deployed your documentation to GitHub Pages, you can update the OpenAPI specification to include clickable links to your documentation.

## Your GitHub Pages URL

After enabling GitHub Pages, your documentation will be available at:

```
https://YOUR_USERNAME.github.io/YOUR_REPO_NAME/
```

For example, if your GitHub username is `amrsolutions` and repository is `vatportal-api-docs`, the URL would be:

```
https://amrsolutions.github.io/vatportal-api-docs/
```

## Update the OpenAPI Spec

Edit `/opt/vatp/apidocs/openapi.yaml` and replace the generic text references with actual links:

### 1. Update Partner Integration Reference (Line ~30)

**Current:**
```yaml
See the Partner Integration Guide in the documentation for details.
```

**Update to:**
```yaml
See [Partner Integration Guide](https://amrsolutions.github.io/vatp-api-docs/en/partner-integration.html) for details.
```

### 2. Update Polling Guide References (Lines ~82, ~632)

**Current:**
```yaml
See the Polling Best Practices guide in the documentation for implementation details.
```

**Update to:**
```yaml
See [Polling Best Practices](https://amrsolutions.github.io/vatp-api-docs/en/polling-guide.html) for implementation details.
```

### 3. Update Download Guide Reference (Line ~332)

**Current:**
```yaml
See the Download from E-taxes guide in the documentation for complete details.
```

**Update to:**
```yaml
See [Download from E-taxes Guide](https://amrsolutions.github.io/vatp-api-docs/en/download-from-etaxes.html) for complete details.
```

### 4. Update Delete Guide Reference (Line ~540)

**Current:**
```yaml
See the Delete E-taxes Invoices guide in the documentation for complete usage details.
```

**Update to:**
```yaml
See [Delete E-taxes Invoices Guide](https://amrsolutions.github.io/vatp-api-docs/en/delete-etaxes-invoices.html) for complete usage details.
```

## Quick Replace Script

Once you know your GitHub Pages URL, you can use this sed command to update all references at once:

```bash
# Set your GitHub Pages base URL
GITHUB_PAGES_URL="https://amrsolutions.github.io/vatp-api-docs/en/README.html"

# Update the openapi.yaml file
sed -i "s|See the Partner Integration Guide in the documentation for details.|See [Partner Integration Guide](${GITHUB_PAGES_URL}/en/partner-integration.html) for details.|g" openapi.yaml

sed -i "s|See the Polling Best Practices guide in the documentation for implementation details.|See [Polling Best Practices](${GITHUB_PAGES_URL}/en/polling-guide.html) for implementation details.|g" openapi.yaml

sed -i "s|See the Polling Best Practices guide in the documentation for detailed implementation examples.|See [Polling Best Practices](${GITHUB_PAGES_URL}/en/polling-guide.html) for detailed implementation examples.|g" openapi.yaml

sed -i "s|See the Download from E-taxes guide in the documentation for complete details.|See [Download from E-taxes Guide](${GITHUB_PAGES_URL}/en/download-from-etaxes.html) for complete details.|g" openapi.yaml

sed -i "s|See the Delete E-taxes Invoices guide in the documentation for complete usage details.|See [Delete E-taxes Invoices Guide](${GITHUB_PAGES_URL}/en/delete-etaxes-invoices.html) for complete usage details.|g" openapi.yaml
```

## Update Contact Info

Also update the contact information in the `info` section (around line 33-38):

```yaml
info:
  title: VatPortal API
  version: 1.0.0
  contact:
    name: VatPortal Support
    url: https://vatportal.az
    email: support@amr.az
  license:
    name: MIT
    url: https://opensource.org/licenses/MIT
```

Change `url` to your actual support page or company website.

## Testing

After updating the links:

1. Save the `openapi.yaml` file
2. Refresh the Swagger UI page (`swagger.html`)
3. Click on any endpoint to expand it
4. The description should now show clickable blue links
5. Click the links to verify they point to the correct documentation pages

## Note on Markdown Links

GitHub Pages serves Markdown files as `.html` files, so:
- Markdown file: `en/polling-guide.md`
- URL to use: `en/polling-guide.html`

Always use the `.html` extension in your links, even though the source file is `.md`.

---

**Last Updated:** December 28, 2025
