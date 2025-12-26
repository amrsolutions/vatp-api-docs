# Qaimə Növləri və Statusları

## Qaimə Növləri

Filtrasiya (v8.0) və arayış üçün bu dəyərlərdən istifadə edin:

| Kod | Təsvir |
|-----|--------|
| `defaultInvoice` | Malların, işlərin və xidmətlərin təqdim edilməsi barədə elektron qaimə-faktura |
| `advanceInvoice` | Alınmış avans ödənişləri barədə elektron qaimə-faktura |
| `agent` | Malların, işlərin və xidmətlərin vəkalətverən (komitent) tərəfindən agentə (komisyonçuya) verilməsi barədə elektron qaimə-faktura |
| `resale` | Agent (komisyonçu) tərəfindən malların, işlərin və xidmətlərin alıcısına (subagentə) təqdim edilməsi barədə elektron qaimə-faktura |
| `recycling` | Malların emala yaxud saxlamaya verilməsi barədə elektron qaimə-faktura |
| `taxCodex163` | Qaytarma istisna olmaqla VM-in 163-cü maddəsinə əsasən dəqiqləşdirilməsi barədə elektron qaimə-faktura |
| `taxCodex177_5` | VM-in 177.5-ci maddəsinə əsasən təqdim edilməsi barədə elektron qaimə-faktura |
| `returnInvoice` | Malların qaytarılması barədə elektron qaimə-faktura |
| `returnByAgent` | Agent (komisyonçu) tərəfindən malların, işlərin və xidmətlərin vəkalətverənə (komitentə) qaytarılması barədə elektron qaimə-faktura |
| `returnRecycled` | Emal prosesi keçmiş, yaxud saxlamaya verilmiş malların qaytarılması barədə elektron qaimə-faktura |
| `exportNoteInvoice` | Malların ixrac qeydi ilə satışı barədə elektron qaimə-faktura |
| `exciseGoodsTransfer` | Aksizli malların (neft məhsulları istisna olmaqla) təsərrüfatdaxili yerdəyişməsi barədə elektron qaimə-faktura |

## Qaimə Statusları

Endirmələri filtrləmək (v8.0) və qaimə vəziyyətini izləmək üçün istifadə olunur:

| Kod | Təsvir | Mənası |
|-----|--------|--------|
| `approved` | Təsdiqləndi | Qaimə tam təsdiqləndi |
| `onApproval` | Təsdiq gözləyir | Qarşı tərəfin təsdiqi gözlənilir |
| `updateApproval` | Düzəliş edildi | Yeniləndi və təsdiqləndi |
| `updateRequested` | Düzəlişə qaytarıldı | Düzəlişlər üçün geri göndərildi |
| `cancelRequested` | Ləğvə göndərildi | Ləğv gözlənilir |
| `approvedBySystem` | Sistem tərəfindən təsdiqləndi | Sistem tərəfindən avtomatik təsdiqləndi |
| `onApprovalEdited` | Düzəlişə göndərildi | Redaktə edilmiş versiya gözlənilir |
| `canceled` | Ləğv edildi | Qaimə ləğv edildi |
| `deleted` | Silindi | Qaimə silindi |
| `deletedBySystem` | Sistem tərəfindən silindi | Sistem tərəfindən avtomatik silindi |
| `deactivated` | Passiv | Qaimə deaktiv edildi |

## API-də Filtrlərdən İstifadə (v8.0)

### Növə Görə Filtrasiya

**Xüsusi növləri daxil et:**
```json
{
  "inc_kinds": "defaultInvoice,advanceInvoice"
}
```

**Xüsusi növləri istisna et:**
```json
{
  "exc_kinds": "taxCodex163,taxCodex177_5"
}
```

**Qeyd:** `inc_kinds` `exc_kinds`-dən üstündür.

### Statusa Görə Filtrasiya

**Xüsusi statusları daxil et:**
```json
{
  "inc_status": "approved,onApproval"
}
```

**Xüsusi statusları istisna et:**
```json
{
  "exc_status": "cancelRequested,updateRequested"
}
```

**Qeyd:** `inc_status` `exc_status`-dan üstündür.

### Nümunə: Yalnız Təsdiqlənmiş Qaimələri Endirin

```bash
curl -X POST https://company.vatportal.az/api/etx/import \
  -H "x-vatpapikey: SİZİN_TOKENİNİZ" \
  -H "Content-Type: application/json" \
  -d '{
    "phone": "994501234567",
    "date_from": "01.01.2025",
    "date_to": "31.01.2025",
    "dir": 1,
    "inc_status": "approved"
  }'
```

### Nümunə: Qaytarmalar İstisna Olmaqla Hamısını Endirin

```bash
curl -X POST https://company.vatportal.az/api/etx/import \
  -H "x-vatpapikey": SİZİN_TOKENİNİZ" \
  -H "Content-Type: application/json" \
  -d '{
    "phone": "994501234567",
    "date_from": "01.01.2025",
    "date_to": "31.01.2025",
    "dir": 1,
    "exc_kinds": "returnInvoice,returnByAgent,returnRecycled"
  }'
```

## Qaimə Həyat Dövrü

```
┌─────────────┐
│  Yaradıldı  │
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

## Ümumi Status Axınları

### Normal Axın
1. `onApproval` - Qaimə alıcıya göndərildi
2. `approved` - Alıcı təsdiqlədi
3. Tamamlandı ✓

### Yeniləmə Axını
1. `approved` - Hazırda təsdiqlənmiş
2. `updateRequested` - Alıcı dəyişikliklər istəyir
3. `onApprovalEdited` - Yenilənmiş versiya göndərildi
4. `updateApproval` - Yeniləmə təsdiqləndi
5. Tamamlandı ✓

### Ləğv Axını
1. `approved` - Hazırda təsdiqlənmiş
2. `cancelRequested` - Ləğv tələb olundu
3. `canceled` - Ləğv təsdiqləndi
4. Tamamlandı ✓

## Ən Yaxşı Təcrübələr

### Qaimə Endirərkən

**✅ EDİN:**
- Yalnız yekunlaşdırılmış qaimələr almaq üçün `inc_status: "approved"` istifadə edin
- Dəqiq nəticələr üçün status və növ filtrlərini birləşdirin
- Nəticə ölçüsünü məhdudlaşdırmaq üçün tarix aralıqlarından istifadə edin

**❌ ETMƏYİN:**
- Yalnız təsdiqlənmiş lazımdırsa, bütün statusları endirməyin
- Ləğv edilmiş/silinmiş qaimələri lazımsız endirməyin
- Çox geniş tarix aralıqları istifadə etməyin (performans problemlərinə səbəb olur)

### Qaimə Yaradarkən

**Düzgün növü təyin edin:**
```json
{
  "type": "advanceInvoice"  // Avans ödənişləri üçün
}
```

**Yükləmədən sonra statusu izləyin:**
```javascript
// Qaimənin təsdiqlənib-təsdiqlənmədiyini yoxlayın
const status = await getProcessStatus(processId);
if (status.stats.invoice_signed) {
  // Uğur - qaime yaradıldı
}
```

## Növlər/Statuslar İstifadə Edən API Endpointləri

| Endpoint | Növləri İstifadə Edir | Statusları İstifadə Edir |
|----------|------------------------|--------------------------|
| `/etx/import` | ✅ (v8.0) | ✅ (v8.0) |
| `/etx/read_invoices` | ✅ (qaytarılır) | ✅ (qaytarılır) |
| `/inv/import_upload_invoices` | ✅ (təyin edilir) | ❌ |

## Daha Çox Məlumat Lazımdır?

- [Qaimələri İdxal və Yükləmək →](./import-upload-invoices.md)
- [Real nümunələr →](./examples.md)
- [Xəta Kodları Arayışı →](./error-codes.md)

---

[← Arayışa qayıt](./README.md)
