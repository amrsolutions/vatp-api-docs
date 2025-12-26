# Delete E-taxes Invoices

**Available since:** v1.0 (June 1, 2021)

## Overview

The VatPortal API allows you to delete invoices (qaimes) from both the portal and the e-taxes.gov.az system. This is a powerful operation that removes invoices that are either in draft status or have been signed and are awaiting counterparty approval.

## Important Notes

⚠️ **Critical Warnings:**
- This operation deletes invoices from **BOTH** the portal and e-taxes.gov.az
- Deletion is **all-or-nothing** - either all invoices in the request are deleted or none
- Only specific invoice statuses can be deleted (see below)
- This operation is **irreversible** once completed
- Returns a process ID for monitoring the deletion progress

## When to Use This Endpoint

Use `DELETE /api/etx/delete` when you need to delete invoices by their e-taxes qaime number (serie/number format).

### Deletable Invoice Statuses

Invoices can be deleted if they are:
1. **Draft invoices** - Not yet submitted to e-taxes
2. **Signed invoices** - Submitted and signed, but waiting for approval from counterparty

### Cannot Delete

You **cannot** delete invoices that are:
- ❌ Already approved by both parties
- ❌ Cancelled through the official cancellation process
- ❌ In any final/completed status

## Comparison with Other Deletion Endpoints

| Feature | `/api/etx/delete` | `/api/inv/delete` |
|---------|-------------------|-------------------|
| **Identifies invoices by** | Qaime serie/number | ERP ID |
| **Deletes from** | Portal + E-taxes | Portal only |
| **Can delete signed invoices** | ✅ Yes (if pending approval) | ❌ No |
| **Can delete drafts** | ✅ Yes | ✅ Yes |
| **Returns process ID** | ✅ Yes (async operation) | ❌ No (direct) |
| **Use case** | Delete from e-taxes system | Delete local drafts only |

---

## Endpoint

```
DELETE https://<company>.vatportal.az/api/etx/delete
```

## Request Parameters

| Parameter | Description | Required | Default |
|-----------|-------------|----------|---------|
| `username` | Portal username | Yes | - |
| `password` | Portal password | Yes | - |
| `serialNumbers` | Array of qaime serie/numbers to delete | Yes | - |
| `is_signed` | Whether the invoices in the array are signed | No | `false` |

### Serial Number Format

The `serialNumbers` parameter must contain qaime numbers in the **"SERIE NUMBER"** format:

- **Format:** `"SERIE NUMBER"` (serie, space, number)
- **Example:** `"MT2401 10000000"`
- **Series:** 2-4 uppercase letters (e.g., AA, MT, ABC)
- **Number:** 8-10 digits

**Important:** All invoices in the `serialNumbers` array must be in the same folder (either all in draft folder or all in sent folder).

### is_signed Parameter

- `false` (default) - Invoices are in the **draft** folder
- `true` - Invoices are in the **sent** folder (signed, awaiting approval)

---

## Request Examples

### Example 1: Delete Draft Invoices

```json
{
  "username": "myusername",
  "password": "mypassword",
  "serialNumbers": [
    "AA2401 10000001",
    "AA2401 10000002",
    "AA2401 10000003"
  ],
  "is_signed": false
}
```

### Example 2: Delete Signed Invoices (Awaiting Approval)

```json
{
  "username": "myusername",
  "password": "mypassword",
  "serialNumbers": [
    "MT2412 25000100",
    "MT2412 25000101"
  ],
  "is_signed": true
}
```

### Example 3: Using Token Authentication

```bash
curl -X DELETE https://company.vatportal.az/api/etx/delete \
  -H "x-vatpapikey: YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "serialNumbers": ["AA2501 10000050"],
    "is_signed": false
  }'
```

---

## Response

### Success Response

```json
{
  "err_code": 0,
  "err_msg": "",
  "data": {
    "id": 12345
  }
}
```

**Response Fields:**

| Field | Description |
|-------|-------------|
| `err_code` | Error code (0 = success) |
| `err_msg` | Error message (empty on success) |
| `data.id` | Process ID for monitoring the deletion |

### Error Response

```json
{
  "err_code": 20,
  "err_msg": "Qaime not found or not in specified folder"
}
```

---

## Monitoring the Deletion Process

Since deletion is an asynchronous operation, use the process ID to monitor progress:

### Check Process Status

```bash
curl -X POST https://company.vatportal.az/api/job/read_proc_status \
  -H "Content-Type: application/json" \
  -d '{
    "username": "myusername",
    "password": "mypassword",
    "procId": 12345
  }'
```

### Status Response

```json
{
  "res": 1,
  "err_code": 0,
  "data": {
    "status": 2,
    "status_desc": "Process Finalized Successfully",
    "stats": {
      "deleted": 3
    }
  }
}
```

**Status Codes:**
- `2` - Process completed successfully
- `3` - Process completed with errors

---

## Complete Workflow Example

```bash
# Step 1: Initiate deletion
curl -X DELETE https://company.vatportal.az/api/etx/delete \
  -H "Content-Type: application/json" \
  -d '{
    "username": "myuser",
    "password": "mypass",
    "serialNumbers": [
      "AA2501 10000001",
      "AA2501 10000002"
    ],
    "is_signed": false
  }'

# Response: {"err_code": 0, "data": {"id": 123}}

# Step 2: Monitor deletion progress
curl -X POST https://company.vatportal.az/api/job/read_proc_status \
  -H "Content-Type: application/json" \
  -d '{
    "username": "myuser",
    "password": "mypass",
    "procId": 123
  }'

# Response: {"status": 2, "status_desc": "Process Finalized Successfully"}
```

---

## Error Handling

### Common Errors

| Error Code | Description | Solution |
|------------|-------------|----------|
| 4 | Empty username or password | Provide valid credentials |
| 5 | Invalid username or password | Check credentials |
| 20 | Qaime not found or not in specified folder | Verify qaime numbers and `is_signed` parameter |

### Error Code 20: Qaime Not Found

This error occurs when:
- The qaime serie/number doesn't exist
- The qaime is in the wrong folder (check `is_signed` parameter)
- The qaime status doesn't allow deletion (already approved, etc.)

**Solution:**
1. Verify the qaime serie/number is correct
2. Check if invoices are signed or draft (set `is_signed` correctly)
3. Verify invoice status in portal
4. Ensure all invoices in the array are in the same folder

### All-or-Nothing Deletion

**Important:** The deletion operation is transactional. If ANY invoice in the `serialNumbers` array cannot be deleted:
- **NO invoices** will be deleted
- The entire request fails
- Error code will indicate the problem

```json
// Example: One invalid qaime causes all to fail
{
  "serialNumbers": [
    "AA2501 10000001",  // ✅ Valid, can be deleted
    "AA2501 10000002",  // ✅ Valid, can be deleted
    "XX9999 99999999"   // ❌ Invalid - entire request fails
  ]
}
// Result: NONE of the invoices are deleted
```

---

## Best Practices

### ✅ DO

- **Verify invoice status** before deletion
- **Use the correct `is_signed` parameter** (match the folder)
- **Monitor the process** using the returned process ID
- **Keep all invoices in same folder** (all draft OR all signed)
- **Log deletion requests** for audit purposes
- **Verify deletion completed** before confirming to users

### ❌ DON'T

- **Mix draft and signed invoices** in the same request
- **Delete without verification** - operation is irreversible
- **Ignore process monitoring** - deletion may fail silently
- **Retry immediately** on error - investigate the cause first
- **Delete approved invoices** - use official cancellation instead

---

## Security Considerations

### Authorization

- Only authorized portal users can delete invoices
- Token-based authentication is recommended for security
- Deletion operations are logged in the system

### Audit Trail

The system maintains logs of:
- Who deleted the invoices
- When they were deleted
- Which invoices were deleted
- Whether deletion succeeded or failed

---

## Use Cases

### Use Case 1: Bulk Delete Draft Invoices

**Scenario:** ERP system created invoices with errors, need to delete and recreate

```json
{
  "username": "erp_user",
  "password": "secure_pass",
  "serialNumbers": [
    "AA2501 10000010",
    "AA2501 10000011",
    "AA2501 10000012",
    "AA2501 10000013",
    "AA2501 10000014"
  ],
  "is_signed": false
}
```

### Use Case 2: Delete Signed Invoice Pending Approval

**Scenario:** Signed invoice sent to counterparty, but contains error. Delete and resend corrected version.

```json
{
  "username": "finance_user",
  "password": "secure_pass",
  "serialNumbers": [
    "MT2412 25000100"
  ],
  "is_signed": true
}
```

### Use Case 3: Cleanup Test Invoices

**Scenario:** Development/testing created invoices that need removal

```json
{
  "username": "test_user",
  "password": "test_pass",
  "serialNumbers": [
    "TEST01 99990001",
    "TEST01 99990002"
  ],
  "is_signed": false
}
```

---

## Migration Guide

If you're currently using `/api/inv/delete` and need to delete from e-taxes:

### Before (Portal Only)

```json
DELETE /api/inv/delete
{
  "username": "user",
  "password": "pass",
  "ids": ["ERP-INV-001", "ERP-INV-002"]  // ERP IDs
}
```

### After (Portal + E-taxes)

```json
DELETE /api/etx/delete
{
  "username": "user",
  "password": "pass",
  "serialNumbers": ["AA2501 10000001", "AA2501 10000002"],  // Qaime numbers
  "is_signed": false
}
```

**Key Differences:**
1. Use qaime serie/number instead of ERP ID
2. Specify if invoices are signed
3. Monitor using process ID
4. Deletes from e-taxes, not just portal

---

## Related Endpoints

- [Import & Upload Invoices](./import-upload-invoices.md) - Create and upload invoices
- [Download from E-taxes](./download-from-etaxes.md) - Download invoices from e-taxes
- [Read Process Status](./import-upload-invoices.md#monitor-process) - Monitor process progress
- [Error Codes](./error-codes.md) - Complete error reference

---

## Next Steps

- [Read about invoice statuses →](./invoice-types.md)
- [Learn about error handling →](./error-codes.md)
- [See authentication methods →](./authentication.md)

---

[← Back to Documentation](./README.md)
