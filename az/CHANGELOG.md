# Dəyişikliklər Jurnalı

VatPortal API-nin bütün əhəmiyyətli dəyişiklikləri burada sənədləşdirilir.

## [8.0] - 2025-08-07

### Əlavə Edildi
- **Qaimə statusuna görə filtrasiya** endirmə metodunda (`/api/etx/import`)
  - Yeni parametrlər: `inc_status` və `exc_status`
  - Yalnız xüsusi qaimə statuslarını endirin (məs., "approved", "onApproval")
  - İstənməyən statusları istisna edin (məs., "canceled", "deleted")
  
- **Qaimə növünə/tipinə görə filtrasiya** endirmə metodunda (`/api/etx/import`)
  - Yeni parametrlər: `inc_kinds` və `exc_kinds`
  - Yalnız xüsusi qaimə növlərini endirin (məs., "defaultInvoice", "advanceInvoice")
  - İstənməyən növləri istisna edin (məs., "returnInvoice")

### İstifadə Halları
```json
// Yalnız təsdiqlənmiş standart qaimələri endirin
{
  "inc_status": "approved",
  "inc_kinds": "defaultInvoice"
}

// Qaytarmalar və ləğv edilmiş istisna olmaqla hamısını endirin
{
  "exc_kinds": "returnInvoice,returnByAgent",
  "exc_status": "canceled,deleted"
}
```

### Pozucu Dəyişikliklər
Yoxdur - bütün yeni parametrlər ixtiyaridir və geriyə uyğundur.

---

## [7.0] - 2025-06-27

### Əlavə Edildi
- **Onlayn sistemdən qaimələri endirmək** (`/api/etx/import`)
  - Qaimələri birbaşa e-taxes.gov.az-dan əldə edin
  - İzləmək üçün proses ID qaytarır
  - ASAN telefon autentifikasiyası tələb edir
  
- **Xam qaime məlumatlarını oxumaq** (`/api/etx/read_invoices`)
  - Bütün sətir məhsulları daxil olmaqla tam qaimə təfərrüatlarını əldə edin
  - Proses ID və ya tarix aralığına görə filtr edin
  - Tam strukturlaşdırılmış qaimə məlumatları qaytarır

### İstifadə Halları
```json
// Endirməni başlatın
POST /api/etx/import
{
  "phone": "994501234567",
  "date_from": "01.01.2025",
  "date_to": "31.01.2025",
  "dir": 1
}

// Endirilmiş məlumatları oxuyun
GET /api/etx/read_invoices
{
  "phone": "994501234567",
  "procId": 123
}
```

---

## [6.0] - 2024-11-01

### Əlavə Edildi
- **Təqdim edilməmiş qaimələri ERP ID-yə görə silmək** (`/api/inv/delete`)
  - E-taxes-a təqdim edilməmiş qaimələri silin
  - Toplu silmə dəstəyi
  - Təqdim edilmiş qaimələrin təsadüfi silinməsinin qarşısını alır

---

## [5.0] - 2024-10-19

### Əlavə Edildi
- **Autentifikasiya bölməsi** sənədləşməyə
  - Hər iki autentifikasiya metodu üçün əhatəli bələdçi
  - Nümunələrlə token yeniləmə axını
  - Təhlükəsizlik ən yaxşı təcrübələri

---

## [4.0] - 2024-10-04

### Əlavə Edildi
- **Token əsaslı autentifikasiya**
  - İstifadəçi adı/parol əvəzinə `x-vatpapikey` HTTP başlığından istifadə edin
  - Təkmilləşdirilmiş təhlükəsizlik (sorğu gövdəsində icazə məlumatları yoxdur)
  - Asanlaşdırılmış icazə məlumatları rotasiyası

### Miqrasiya Bələdçisi
```bash
# Köhnə metod (hələ də dəstəklənir)
curl -X POST /api/inv/read_invoices \
  -d '{"username":"user","password":"pass",...}'

# Yeni metod (tövsiyə olunur)
curl -X POST /api/inv/read_invoices \
  -H "x-vatpapikey: SİZİN_TOKENİNİZ" \
  -d '{...}'
```

---

## Miqrasiya Bələdçiləri

### v8.0-a v7.0-dan Yüksəltmə

Pozucu dəyişikliklər yoxdur. `/api/etx/import` istifadə edirsinizsə, indi filtrlər əlavə edə bilərsiniz:

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

### v7.0-a v6.0-dan Yüksəltmə

Yeni endpointlər əlçatandır. Mövcud endpointlərdə dəyişikliklər yoxdur.

### v4.0-a v3.x-dən Yüksəltmə

**Tövsiyə olunur:** Token autentifikasiyasına keçin:

1. Portalda token yaradın (Profil → API-yə Giriş)
2. İstifadəçi adı/parolu başlıqla əvəz edin:
   ```bash
   -H "x-vatpapikey: SİZİN_TOKENİNİZ"
   ```
3. Sorğu gövdəsindən `username` və `password` silin

Köhnə metod hələ də işləyir, lakin token daha təhlükəsizdir.

## Versiya Dəstək Siyasəti

- **Cari versiya (8.0):** Tam dəstəklənir
- **Əvvəlki versiya (7.0):** 8.0 buraxılışından 6 ay sonra dəstəklənir
- **Köhnə versiyalar:** İşləyə bilər, lakin rəsmi dəstəklənmir

## Köhnəlmə Bildirişləri

**Hazırda yoxdur.** İstifadəçi adı/parol autentifikasiyası token autentifikasiyası ilə yanaşı dəstəklənməyə davam edəcək.

## Planlaşdırılan Xüsusiyyətlər

🔮 Yol xəritəsi (dəyişdirilə bilər):

- Qaimə statusu dəyişiklikləri üçün Webhook bildirişləri
- Toplu qaimə emalı üçün Toplu əməliyyatlar API-si
- Rəsmi SDK-lar (JavaScript, Python, C#)
- GraphQL endpoint alternativ
- Təkmilləşdirilmiş ətraflı validasiya rəyi ilə xəta mesajları

## Rəy və Xüsusiyyət Tələbləri

Gələcək versiyalar üçün təklifləriniz var?

- Hesab meneceri ilə əlaqə saxlayın
- Email: support@amrsolutions.az
- Portalda bəyənmə/bəyənməmə düymələrindən istifadə edin

---

[← Sənədləşməyə qayıt](./README.md)
