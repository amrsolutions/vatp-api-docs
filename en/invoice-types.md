# Invoice Types & Statuses

## Invoice Types

Use these values for filtering (v8.0) and reference:

| Code | Description (EN) | Description (AZ) |
|------|------------------|------------------|
| `defaultInvoice` | Standard goods/services invoice | Malların, işlərin və xidmətlərin təqdim edilməsi barədə elektron qaimə-faktura |
| `advanceInvoice` | Advance payment invoice | Alınmış avans ödənişləri barədə elektron qaimə-faktura |
| `agent` | Agent/commission sales invoice | Malların, işlərin və xidmətlərin vəkalətverən (komitent) tərəfindən agentə (komisyonçuya) verilməsi barədə elektron qaimə-faktura |
| `resale` | Agent resale to buyer | Agent (komisyonçu) tərəfindən malların, işlərin və xidmətlərin alıcısına (subagentə) təqdim edilməsi barədə elektron qaimə-faktura |
| `recycling` | Goods for processing/storage | Malların emala yaxud saxlamaya verilməsi barədə elektron qaimə-faktura |
| `taxCodex163` | Tax code adjustment (Art. 163) | Qaytarma istisna olmaqla VM-in 163-cü maddəsinə əsasən dəqiqləşdirilməsi barədə elektron qaimə-faktura |
| `taxCodex177_5` | Tax code submission (Art. 177.5) | VM-in 177.5-ci maddəsinə əsasən təqdim edilməsi barədə elektron qaimə-faktura |
| `returnInvoice` | Product return invoice | Malların qaytarılması barədə elektron qaimə-faktura |
| `returnByAgent` | Return from agent to principal | Agent (komisyonçu) tərəfindən malların, işlərin və xidmətlərin vəkalətverənə (komitentə) qaytarılması barədə elektron qaimə-faktura |
| `returnRecycled` | Return of processed/stored goods | Emal prosesi keçmiş, yaxud saxlamaya verilmiş malların qaytarılması barədə elektron qaimə-faktura |
| `exportNoteInvoice` | Export with export note | Malların ixrac qeydi ilə satışı barədə elektron qaimə-faktura |
| `exciseGoodsTransfer` | Excise goods transfer | Aksizli malların (neft məhsulları istisna olmaqla) təsərrüfatdaxili yerdəyişməsi barədə elektron qaimə-faktura |

## Invoice Statuses

Used for filtering downloads (v8.0) and tracking invoice state:

| Code | Description (EN) | Description (AZ) | Meaning |
|------|------------------|------------------|---------|
| `approved` | Approved | Təsdiqləndi | Invoice fully approved |
| `onApproval` | Awaiting approval | Təsdiq gözləyir | Waiting for counterparty approval |
| `updateApproval` | Update approved | Düzəliş edildi | Updated and approved |
| `updateRequested` | Update requested | Düzəlişə qaytarıldı | Sent back for corrections |
| `cancelRequested` | Cancellation requested | Ləğvə göndərildi | Cancellation pending |
| `approvedBySystem` | System approved | Sistem tərəfindən təsdiqləndi | Auto-approved by system |
| `onApprovalEdited` | Edited, awaiting approval | Düzəlişə göndərildi | Edited version pending |
| `canceled` | Canceled | Ləğv edildi | Invoice canceled |
| `deleted` | Deleted | Silindi | Invoice deleted |
| `deletedBySystem` | System deleted | Sistem tərəfindən silindi | Auto-deleted by system |
| `deactivated` | Deactivated | Passiv | Invoice deactivated |

## Using Filters in API (v8.0)

### Filtering by Type

**Include specific types:**
```json
{
  "inc_kinds": "defaultInvoice,advanceInvoice"
}
```

**Exclude specific types:**
```json
{
  "exc_kinds": "taxCodex163,taxCodex177_5"
}
```

**Note:** `inc_kinds` takes precedence over `exc_kinds`.

### Filtering by Status

**Include specific statuses:**
```json
{
  "inc_status": "approved,onApproval"
}
```

**Exclude specific statuses:**
```json
{
  "exc_status": "cancelRequested,updateRequested"
}
```

**Note:** `inc_status` takes precedence over `exc_status`.

### Example: Download Only Approved Invoices

```bash
curl -X POST https://company.vatportal.az/api/etx/import \
  -H "x-vatpapikey: YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "phone": "994501234567",
    "date_from": "01.01.2025",
    "date_to": "31.01.2025",
    "dir": 1,
    "inc_status": "approved"
  }'
```

### Example: Download All Except Returns

```bash
curl -X POST https://company.vatportal.az/api/etx/import \
  -H "x-vatpapikey: YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "phone": "994501234567",
    "date_from": "01.01.2025",
    "date_to": "31.01.2025",
    "dir": 1,
    "exc_kinds": "returnInvoice,returnByAgent,returnRecycled"
  }'
```

## Invoice Lifecycle

```
┌─────────────┐
│   Created   │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ onApproval  │ ◄─────┐
└──────┬──────┘       │
       │              │
       ├──────────────┤
       │              │
       ▼              │
┌─────────────┐       │
│  approved   │       │
└──────┬──────┘       │
       │              │
       ├──────────────┘
       │   updateRequested
       ▼
┌─────────────┐
│ cancelReq'd │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│  canceled   │
└─────────────┘
```

## Common Status Flows

### Normal Flow
1. `onApproval` - Invoice sent to buyer
2. `approved` - Buyer approves
3. Done ✓

### Update Flow
1. `approved` - Currently approved
2. `updateRequested` - Buyer requests changes
3. `onApprovalEdited` - Updated version sent
4. `updateApproval` - Update approved
5. Done ✓

### Cancellation Flow
1. `approved` - Currently approved
2. `cancelRequested` - Cancellation requested
3. `canceled` - Cancellation approved
4. Done ✓

## Best Practices

### When Downloading Invoices

**✅ DO:**
- Use `inc_status: "approved"` to get only finalized invoices
- Combine status and type filters for precise results
- Use date ranges to limit result size

**❌ DON'T:**
- Download all statuses if you only need approved
- Download canceled/deleted invoices unnecessarily
- Use too broad date ranges (causes performance issues)

### When Creating Invoices

**Set correct type:**
```json
{
  "type": "advanceInvoice"  // For advance payments
}
```

**Monitor status after upload:**
```javascript
// Check if invoice was approved
const status = await getProcessStatus(processId);
if (status.stats.invoice_signed) {
  // Success - qaime generated
}
```

## API Endpoints Using Types/Statuses

| Endpoint | Uses Types | Uses Statuses |
|----------|------------|---------------|
| `/etx/import` | ✅ (v8.0) | ✅ (v8.0) |
| `/etx/read_invoices` | ✅ (returned) | ✅ (returned) |
| `/inv/import_upload_invoices` | ✅ (set) | ❌ |

## Need More Info?

- [Import & Upload Invoices →](./import-upload-invoices)
- [Real-world Examples →](./examples)
- [Error Codes Reference →](./error-codes)

---

[← Back to Reference](./README.md)
