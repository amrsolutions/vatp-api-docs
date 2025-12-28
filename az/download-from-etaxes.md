# E-taxes-dan Qaimələrin Yüklənməsi

**Mövcuddur:** v7.0 (27 İyun 2025)
**Təkmilləşdirilib:** v8.0 (7 Avqust 2025)

## Ümumi Baxış

VatPortal API sizə Azərbaycanın e-taxes.gov.az sistemindən birbaşa qaimələri portalınıza yükləməyə imkan verir. Bu yükləmə axınının **əks istiqamətidir** - qaimələri e-taxes-a göndərmək əvəzinə, e-taxes-dan qaimələri əldə edirsiniz.

Bu funksiya aşağıdakılar üçün faydalıdır:
- ERP sisteminiz xaricində yaradılmış qaimələri sinxronlaşdırmaq
- Biznes tərəfdaşlarından qaimələri əldə etmək
- E-taxes-dan qaimə məlumatlarını ehtiyat nüsxəsini götürmək
- Sistemlər arasında qaimə qeydlərini uzlaşdırmaq

## İki Addımlı Proses

E-taxes-dan qaimələrin yüklənməsi iki API çağırışı tələb edir:

1. **Yükləməni Başlatın** (`POST /etx/import`) - Yükləmə prosesini başladır
2. **Yüklənmiş Məlumatları Oxuyun** (`GET /etx/read_invoices`) - Tam qaimə məlumatlarını əldə edir

---

## Autentifikasiya və Müştəri Konteksti

### Korporativ Müştərilər üçün
- **`phone` parametri lazım deyil** - öz şirkətinizin məlumatları üzərində əməliyyat aparır
- `username`/`password` və ya token autentifikasiyasından istifadə edin
- Subdomen formatı: `company.vatportal.az`

### Partnerlər üçün
- **`phone` parametri bütün API çağırışlarında TƏLƏb OLUNUR**
- Hansı müştəri adından əməliyyat apardığınızı müəyyən edir
- Format: `994XXXXXXXXX` (ASAN telefon nömrəsi)
- Subdomen formatı: `partnername.vatportal.az`

Daha ətraflı məlumat üçün [Partner İnteqrasiya Bələdçisi](./partner-integration.md) baxın.

---

## Addım 1: Yükləmə Prosesini Başlatın

### Endpoint

```
POST https://<company>.vatportal.az/api/etx/import
```

### Sorğu Parametrləri

| Parametr | Təsvir | Tələb olunur | Standart |
|----------|--------|--------------|----------|
| `username` | Portal istifadəçi adı | Bəli | - |
| `password` | Portal parolu | Bəli | - |
| `phone` | ASAN telefon nömrəsi (format: 994XXXXXXXXX) **[Yalnız partnerlər üçün]** | Partnerlər: Bəli<br>Korporativ: Xeyr | - |
| `date_from` | E-taxes-da qaimənin yaradılma başlanğıc tarixi (dd.mm.yyyy və ya dd.mm.yyyy hh:mm) | Bəli | - |
| `date_to` | E-taxes-da qaimənin yaradılma bitmə tarixi (dd.mm.yyyy və ya dd.mm.yyyy hh:mm) | Bəli | - |
| `dir` | Qovluq növü: `0` = Daxil olan, `1` = Göndərilmiş | Bəli | `0` |
| `inc_kinds` | **[v8.0]** Yalnız bu qaimə növlərini daxil edin (vergüllə ayrılmış) | Xeyr | - |
| `exc_kinds` | **[v8.0]** Bu qaimə növlərini çıxarın (vergüllə ayrılmış) | Xeyr | - |
| `inc_status` | **[v8.0]** Yalnız bu statusları daxil edin (vergüllə ayrılmış) | Xeyr | - |
| `exc_status` | **[v8.0]** Bu statusları çıxarın (vergüllə ayrılmış) | Xeyr | - |

**Qeyd:** `inc_kinds` parametri `exc_kinds` parametrini ləğv edir. `inc_status` parametri `exc_status` parametrini ləğv edir.

### Sorğu Nümunəsi

**Korporativ Müştəri:**
```json
{
  "username": "istifadəçi_adım",
  "password": "parolum",
  "date_from": "01.08.2025",
  "date_to": "07.08.2025",
  "dir": 1,
  "inc_status": "approved,onApproval",
  "exc_kinds": "returnInvoice"
}
```

**Partner:**
```json
{
  "username": "partner_adı",
  "password": "partner_parolu",
  "phone": "994501234567",
  "date_from": "01.08.2025",
  "date_to": "07.08.2025",
  "dir": 1,
  "inc_status": "approved,onApproval",
  "exc_kinds": "returnInvoice"
}
```

### Cavab

```json
{
  "res": 1,
  "err_code": 0,
  "err_msg": "",
  "data": {
    "id": 123
  }
}
```

`id` sahəsi proses ID-ni ehtiva edir. Yükləmə gedişatını izləmək üçün [Proses Statusunu Oxuyun](./import-upload-invoices.md#proses-statusunu-izləyin) endpointindən istifadə edin.

---

## Addım 2: Yüklənmiş Qaimə Məlumatlarını Oxuyun

### Endpoint

```
GET https://<company>.vatportal.az/api/etx/read_invoices
```

### Sorğu Parametrləri

**Proses ID** və ya **tarix aralığı** üzrə axtarış edə bilərsiniz:

#### Variant A: Proses ID üzrə Axtarış

| Parametr | Təsvir | Tələb olunur |
|----------|--------|--------------|
| `username` | Portal istifadəçi adı | Bəli |
| `password` | Portal parolu | Bəli |
| `phone` | ASAN telefon nömrəsi (format: 994XXXXXXXXX) **[Yalnız partnerlər üçün]** | Partnerlər: Bəli<br>Korporativ: Xeyr |
| `procId` | Yükləmə sorğusundan proses ID | Bəli |
| `dir` | Qovluq növü: `0` = Daxil olan, `1` = Göndərilmiş | Xeyr (standart: 0) |

#### Variant B: Tarix Aralığı üzrə Axtarış

| Parametr | Təsvir | Tələb olunur |
|----------|--------|--------------|
| `username` | Portal istifadəçi adı | Bəli |
| `password` | Portal parolu | Bəli |
| `phone` | ASAN telefon nömrəsi (format: 994XXXXXXXXX) **[Yalnız partnerlər üçün]** | Partnerlər: Bəli<br>Korporativ: Xeyr |
| `date_from` | Başlanğıc tarixi (dd.mm.yyyy) | Bəli |
| `date_to` | Bitmə tarixi (dd.mm.yyyy) | Bəli |
| `dir` | Qovluq növü: `0` = Daxil olan, `1` = Göndərilmiş | Xeyr (standart: 0) |
| `from` | Səhifə nömrəsi (1-dən başlayır) | Xeyr (standart: 1) |

**Səhifələmə:** Nəticələr 5000 qeyd səhifələrində qaytarılır.

### Sorğu Nümunəsi (Proses ID üzrə)

**Korporativ Müştəri:**
```json
{
  "username": "istifadəçi_adım",
  "password": "parolum",
  "procId": 123,
  "dir": 1
}
```

**Partner:**
```json
{
  "username": "partner_adı",
  "password": "partner_parolu",
  "phone": "994501234567",
  "procId": 123,
  "dir": 1
}
```

### Sorğu Nümunəsi (Tarix Aralığı üzrə)

**Korporativ Müştəri:**
```json
{
  "username": "istifadəçi_adım",
  "password": "parolum",
  "date_from": "01.08.2025",
  "date_to": "07.08.2025",
  "dir": 1,
  "from": 1
}
```

**Partner:**
```json
{
  "username": "partner_adı",
  "password": "partner_parolu",
  "phone": "994501234567",
  "date_from": "01.08.2025",
  "date_to": "07.08.2025",
  "dir": 1,
  "from": 1
}
```

### Cavab

```json
{
  "res": 1,
  "err_code": 0,
  "err_msg": "",
  "data": {
    "invoices": [
      {
        "id": 12345,
        "main_subject": "Müqavilə A001",
        "add_subject": "Əlavə məlumat",
        "serial": "AA",
        "number": "24080000001",
        "qaime": "AA24080000001",
        "cmp_from": "Şirkət MMC",
        "cmp_to": "Alıcı Şirkət",
        "tin_from": "1111111111",
        "tin_to": "0000000000",
        "datetime": "07.08.2025 14:30",
        "ini_amount": 1000.00,
        "ini_amount1": 1000.00,
        "ini_amount2": 1000.00,
        "excise_amount": 0,
        "apply_amount": 1000.00,
        "skip_amount": 0,
        "zero_amount": 0,
        "free_amount": 0,
        "vat_amount": 180.00,
        "transp_amount": 0,
        "net_amount": 1180.00,
        "status": "approved",
        "type": "defaultInvoice",
        "lines": [
          {
            "id": 1,
            "line": 1,
            "prod_code": "1234567890",
            "prod_name": "Məhsul Adı",
            "barcode": "1234567890000",
            "unit_name": "ədəd",
            "unit_price": 10.00,
            "unit_qty": 100,
            "ini_amount": 1000.00,
            "ini_amount2": 1000.00,
            "grade": 0,
            "total_excise": 0,
            "apply_amount": 1000.00,
            "skip_amount": 0,
            "zero_amount": 0,
            "free_amount": 0,
            "vat_amount": 180.00,
            "transp_amount": 0,
            "net_amount": 1180.00
          }
        ]
      }
    ],
    "total": 1
  }
}
```

---

## Filtrasiya Seçimləri (v8.0)

### Qaimə Statusu üzrə Filtr

`inc_status` və ya `exc_status` bu dəyərlərlə istifadə edin:

| Status Kodu | Təsvir |
|-------------|--------|
| `approved` | Təsdiqlənmiş |
| `onApproval` | Təsdiq gözləyir |
| `updateApproval` | Yeniləmə təsdiqlənmiş |
| `updateRequested` | Yeniləmə tələb edilmiş |
| `cancelRequested` | Ləğv tələb edilmiş |
| `approvedBySystem` | Sistem tərəfindən təsdiqlənmiş |
| `onApprovalEdited` | Yeniləmə üçün göndərilmiş |
| `canceled` | Ləğv edilmiş |
| `deleted` | Silinmiş |
| `deletedBySystem` | Sistem tərəfindən silinmiş |
| `deactivated` | Deaktiv edilmiş |

**Nümunə:** Yalnız təsdiqlənmiş və gözləyən qaimələri yükləyin:
```json
"inc_status": "approved,onApproval"
```

### Qaimə Növü üzrə Filtr

`inc_kinds` və ya `exc_kinds` bu dəyərlərlə istifadə edin:

| Növ Kodu | Təsvir |
|----------|--------|
| `defaultInvoice` | Standart mal/xidmət qaiməsi |
| `agent` | Agent/komissiya qaiməsi |
| `resale` | Agent təkrar satış qaiməsi |
| `recycling` | Emal üçün göndərilmiş mallar |
| `taxCodex163` | Vergi məcəlləsi 163 düzəliş |
| `taxCodex177_5` | Vergi məcəlləsi 177.5 |
| `returnInvoice` | Məhsul qaytarılması |
| `returnByAgent` | Agent qaytarılması |
| `returnRecycled` | Emaldan qaytarılma |
| `exportNoteInvoice` | İxrac qeydi ilə ixrac |
| `exciseGoodsTransfer` | Aksiz mallarının köçürülməsi |
| `advanceInvoice` | Avans ödəniş qaiməsi |

**Nümunə:** Qaytarılma qaimələrini çıxarın:
```json
"exc_kinds": "returnInvoice,returnByAgent"
```

---

## Filtrasiya İstifadə Nümunələri

### Nümunə 1: Yalnız Təsdiqlənmiş Qaimələri Yükləyin

Yalnız təsdiqlənmiş qaimələri yükləyin, təsdiq və ya yeniləmə gözləyənləri çıxararaq:

**Korporativ Müştəri:**
```bash
curl -X POST https://company.vatportal.az/api/etx/import \
  -H "Content-Type: application/json" \
  -d '{
    "username": "istifadəçim",
    "password": "parolum",
    "date_from": "01.01.2025",
    "date_to": "31.01.2025",
    "dir": 0,
    "inc_status": "approved"
  }'
```

**Partner:**
```bash
curl -X POST https://partnername.vatportal.az/api/etx/import \
  -H "Content-Type: application/json" \
  -d '{
    "username": "partner_adı",
    "password": "partner_parolu",
    "phone": "994501234567",
    "date_from": "01.01.2025",
    "date_to": "31.01.2025",
    "dir": 0,
    "inc_status": "approved"
  }'
```

### Nümunə 2: Ləğv Edilmiş və Silinmiş Qaimələri Çıxarın

Ləğv edilmiş və ya silinmiş qaimələr istisna olmaqla bütün qaimələri yükləyin:

**Korporativ Müştəri:**
```bash
curl -X POST https://company.vatportal.az/api/etx/import \
  -H "Content-Type: application/json" \
  -d '{
    "username": "istifadəçim",
    "password": "parolum",
    "date_from": "01.01.2025",
    "date_to": "31.01.2025",
    "dir": 1,
    "exc_status": "canceled,deleted,deletedBySystem"
  }'
```

**Partner (`phone` parametri əlavə edin):**
```bash
curl -X POST https://partnername.vatportal.az/api/etx/import \
  -H "Content-Type: application/json" \
  -d '{
    "username": "partner_adı",
    "password": "partner_parolu",
    "phone": "994501234567",
    "date_from": "01.01.2025",
    "date_to": "31.01.2025",
    "dir": 1,
    "exc_status": "canceled,deleted,deletedBySystem"
  }'
```

> **Nümunələr 3-7 üçün qeyd:** Qısalıq üçün aşağıdakı nümunələr yalnız korporativ müştəri formatında göstərilib.
> **Partnerlər:** `"phone": "994XXXXXXXXX"` parametrini əlavə edin və `partnername.vatportal.az` subdomenindən istifadə edin.

### Nümunə 3: Yalnız Standart Qaimələri Yükləyin

Yalnız standart mal/xidmət qaimələrini yükləyin, digər növləri çıxararaq:

```bash
curl -X POST https://company.vatportal.az/api/etx/import \
  -H "Content-Type: application/json" \
  -d '{
    "username": "istifadəçim",
    "password": "parolum",
    "date_from": "01.01.2025",
    "date_to": "31.01.2025",
    "dir": 0,
    "inc_kinds": "defaultInvoice"
  }'
```

### Nümunə 4: Bütün Qaytarılma Qaimələrini Çıxarın

Müxtəlif qaytarılma növləri istisna olmaqla bütün qaimələri yükləyin:

```bash
curl -X POST https://company.vatportal.az/api/etx/import \
  -H "Content-Type: application/json" \
  -d '{
    "username": "istifadəçim",
    "password": "parolum",
    "date_from": "01.01.2025",
    "date_to": "31.01.2025",
    "dir": 0,
    "exc_kinds": "returnInvoice,returnByAgent,returnRecycled"
  }'
```

### Nümunə 5: Status və Növ Filtrlərini Birləşdirin

Yalnız təsdiqlənmiş standart və avans qaimələrini yükləyin:

```bash
curl -X POST https://company.vatportal.az/api/etx/import \
  -H "Content-Type: application/json" \
  -d '{
    "username": "istifadəçim",
    "password": "parolum",
    "date_from": "01.01.2025",
    "date_to": "31.01.2025",
    "dir": 1,
    "inc_status": "approved,approvedBySystem",
    "inc_kinds": "defaultInvoice,advanceInvoice"
  }'
```

### Nümunə 6: Təsdiq Gözləyən Qaimələri Yükləyin (Daxil Olan)

Təsdiqlənmənizi gözləyən bütün daxil olan qaimələri əldə edin:

```bash
curl -X POST https://company.vatportal.az/api/etx/import \
  -H "Content-Type: application/json" \
  -d '{
    "username": "istifadəçim",
    "password": "parolum",
    "date_from": "01.12.2024",
    "date_to": "31.12.2024",
    "dir": 0,
    "inc_status": "onApproval,onApprovalEdited"
  }'
```

### Nümunə 7: İxrac Qaimələri (Qaytarılma və Ləğv Edilməmiş)

Aktiv (ləğv edilməmiş və silinməmiş) ixrac qaimələrini yükləyin:

```bash
curl -X POST https://company.vatportal.az/api/etx/import \
  -H "Content-Type: application/json" \
  -d '{
    "username": "istifadəçim",
    "password": "parolum",
    "date_from": "01.01.2025",
    "date_to": "31.03.2025",
    "dir": 1,
    "inc_kinds": "exportNoteInvoice",
    "exc_status": "canceled,deleted,deletedBySystem,deactivated"
  }'
```

### Filtrasiya Prioritet Qaydaları

**Vacib:** Filtrlərdən istifadə edərkən nəzərə alın:

1. **`inc_*` parametrləri `exc_*` parametrlərini üstələyir**
   - Həm `inc_status`, həm də `exc_status` təqdim edilərsə, yalnız `inc_status` istifadə olunur
   - Həm `inc_kinds`, həm də `exc_kinds` təqdim edilərsə, yalnız `inc_kinds` istifadə olunur

2. **Üstələmə davranışı nümunəsi:**
   ```json
   {
     "inc_status": "approved",
     "exc_status": "canceled"
   }
   ```
   Nəticə: Yalnız `inc_status` tətbiq olunur, `exc_status` nəzərə alınmır. Yalnız təsdiqlənmiş qaimələri əldə edəcəksiniz.

3. **Vergüllə ayrılmış dəyərlər:**
   ```json
   {
     "inc_status": "approved,onApproval,updateApproval"
   }
   ```
   Nəticə: Bu statuslardan HƏR HANSİ BİRİNƏ uyğun gələn qaimələr daxil ediləcək (VƏ YA məntiqi).

---

## Tam İş Axını Nümunəsi

**Korporativ Müştəri:**
```bash
# Addım 1: Yükləmə prosesini başlatın
curl -X POST https://mycompany.vatportal.az/api/etx/import \
  -H "Content-Type: application/json" \
  -d '{
    "username": "istifadəçim",
    "password": "parolum",
    "date_from": "01.08.2025",
    "date_to": "07.08.2025",
    "dir": 1,
    "inc_status": "approved"
  }'

# Cavab: {"err_code": 0, "data": {"id": 123}}

# Addım 2: Proses statusunu yoxlayın
curl -X GET https://mycompany.vatportal.az/api/job/read_proc_status \
  -H "Content-Type: application/json" \
  -d '{
    "username": "istifadəçim",
    "password": "parolum",
    "procId": 123
  }'

# Cavab: {"status": 2, "status_desc": "Proses Uğurla Tamamlandı"}

# Addım 3: Yüklənmiş qaimə məlumatlarını oxuyun
curl -X GET https://mycompany.vatportal.az/api/etx/read_invoices \
  -H "Content-Type: application/json" \
  -d '{
    "username": "istifadəçim",
    "password": "parolum",
    "procId": 123,
    "dir": 1
  }'
```

**Partner (Addım 1 və 3-ə `phone` parametri əlavə edin):**
```bash
# Addım 1: phone parametri daxil edin
curl -X POST https://partnername.vatportal.az/api/etx/import \
  -d '{"username": "...", "password": "...", "phone": "994501234567", ...}'

# Addım 3: phone parametri daxil edin
curl -X GET https://partnername.vatportal.az/api/etx/read_invoices \
  -d '{"username": "...", "password": "...", "phone": "994501234567", "procId": 123, ...}'
```

---

## Vacib Qeydlər

- **Telefon Parametri (Yalnız Partnerlər üçün):** Partnerlər hansı müştəri adından əməliyyat apardıqlarını müəyyən etmək üçün `phone` parametrini daxil etməlidirlər. Korporativ müştərilər bu parametrə ehtiyac duymur.
- **ASAN Konfiqurasiyası:** ASAN telefon nömrələri istifadədən əvvəl portalda konfiqurasiya edilməlidir
- **Autentifikasiya:** Yükləmə prosesi SMS/ASAN giriş vasitəsilə PIN1 təsdiqi tələb edəcək
- **Səhifələmə:** Böyük nəticə dəstləri 5000 qeyd səhifələrində qaytarılır
- **Tarix Formatı:** `dd.mm.yyyy` və ya `dd.mm.yyyy hh:mm` formatından istifadə edin
- **Filtr Prioriteti:** `inc_*` parametrləri `exc_*` parametrlərini ləğv edir

---

## Xəta İdarəetməsi

E-taxes-dan yükləmə zamanı ümumi xətalar:

| Xəta Kodu | Təsvir | Həll |
|-----------|--------|------|
| 4 | Boş istifadəçi adı və ya parol | İcazə məlumatları təmin edin |
| 5 | Yanlış istifadəçi adı və ya parol | İcazə məlumatlarını yoxlayın |
| 15 | ASAN nömrəsi təyin edilməyib | Portalda ASAN telefonu konfiqurasiya edin |

---

## Növbəti Addımlar

- [Qaimələri İdxal və Yükləmək →](./import-upload-invoices.md)
- [Xəta Kodları Arayışı →](./error-codes.md)
- [Autentifikasiya Bələdçisi →](./authentication.md)

---

[← Sənədləşməyə qayıt](./README.md)
