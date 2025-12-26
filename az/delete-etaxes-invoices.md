# E-taxes Qaimələrinin Silinməsi

**Mövcuddur:** v1.0 (1 İyun 2021)

## Ümumi Baxış

VatPortal API sizə həm portaldan, həm də e-taxes.gov.az sistemindən qaimələri silməyə imkan verir. Bu, qaralama statusunda olan və ya imzalanmış və qarşı tərəfin təsdiqini gözləyən qaimələri silən güclü əməliyyatdır.

## Vacib Qeydlər

⚠️ **Kritik Xəbərdarlıqlar:**
- Bu əməliyyat qaimələri **HƏM** portaldan, **HƏM də** e-taxes.gov.az-dan silir
- Silmə **hamısı-və-ya-heç biri** prinsipi ilə işləyir - ya sorğudakı bütün qaimələr silinir, ya da heç biri
- Yalnız xüsusi qaimə statusları silinə bilər (aşağıya baxın)
- Bu əməliyyat tamamlandıqdan sonra **geri qaytarıla bilməz**
- Silmə gedişatını izləmək üçün proses ID qaytarır

## Bu Endpointdən Nə Vaxt İstifadə Etmək Olar

E-taxes qaime nömrəsi (seriya/nömrə formatı) ilə qaimələri silmək lazım olduqda `DELETE /api/etx/delete` istifadə edin.

### Silinə Bilən Qaimə Statusları

Qaimələr silinə bilər əgər onlar:
1. **Qaralama qaimələr** - Hələ e-taxes-a təqdim edilməyib
2. **İmzalanmış qaimələr** - Təqdim və imzalanıb, lakin qarşı tərəfin təsdiqini gözləyir

### Silinə Bilməz

Siz **silə bilməzsiniz** qaimələri ki:
- ❌ Artıq hər iki tərəf tərəfindən təsdiqlənib
- ❌ Rəsmi ləğv prosesi ilə ləğv edilib
- ❌ İstənilən yekun/tamamlanmış statusdadır

## Digər Silmə Endpointləri ilə Müqayisə

| Xüsusiyyət | `/api/etx/delete` | `/api/inv/delete` |
|------------|-------------------|-------------------|
| **Qaimələri müəyyənləşdirir** | Qaime seriya/nömrə | ERP ID |
| **Silinir** | Portal + E-taxes | Yalnız portal |
| **İmzalanmış qaimələri silə bilər** | ✅ Bəli (təsdiq gözləyirsə) | ❌ Xeyr |
| **Qaralama qaimələri silə bilər** | ✅ Bəli | ✅ Bəli |
| **Proses ID qaytarır** | ✅ Bəli (asinxron əməliyyat) | ❌ Xeyr (birbaşa) |
| **İstifadə halı** | E-taxes sistemindən silmək | Yalnız yerli qaralamaları silmək |

---

## Endpoint

```
DELETE https://<company>.vatportal.az/api/etx/delete
```

## Sorğu Parametrləri

| Parametr | Təsvir | Tələb olunur | Standart |
|----------|--------|--------------|----------|
| `username` | Portal istifadəçi adı | Bəli | - |
| `password` | Portal parolu | Bəli | - |
| `serialNumbers` | Silinəcək qaime seriya/nömrələri massivi | Bəli | - |
| `is_signed` | Massivdəki qaimələrin imzalanıb-imzalanmadığı | Xeyr | `false` |

### Seriya Nömrəsi Formatı

`serialNumbers` parametri qaime nömrələrini **"SERİYA NÖMRƏ"** formatında ehtiva etməlidir:

- **Format:** `"SERİYA NÖMRƏ"` (seriya, boşluq, nömrə)
- **Nümunə:** `"MT2401 10000000"`
- **Seriya:** 2-4 böyük hərf (məs., AA, MT, ABC)
- **Nömrə:** 8-10 rəqəm

**Vacib:** `serialNumbers` massivindəki bütün qaimələr eyni qovluqda olmalıdır (ya hamısı qaralama qovluğunda, ya da hamısı göndərilmiş qovluğunda).

### is_signed Parametri

- `false` (standart) - Qaimələr **qaralama** qovluğundadır
- `true` - Qaimələr **göndərilmiş** qovluğundadır (imzalanmış, təsdiq gözləyir)

---

## Sorğu Nümunələri

### Nümunə 1: Qaralama Qaimələri Silmək

```json
{
  "username": "istifadəçi_adım",
  "password": "parolum",
  "serialNumbers": [
    "AA2401 10000001",
    "AA2401 10000002",
    "AA2401 10000003"
  ],
  "is_signed": false
}
```

### Nümunə 2: İmzalanmış Qaimələri Silmək (Təsdiq Gözləyir)

```json
{
  "username": "istifadəçi_adım",
  "password": "parolum",
  "serialNumbers": [
    "MT2412 25000100",
    "MT2412 25000101"
  ],
  "is_signed": true
}
```

### Nümunə 3: Token Autentifikasiyasından İstifadə

```bash
curl -X DELETE https://company.vatportal.az/api/etx/delete \
  -H "x-vatpapikey: SİZİN_TOKENİNİZ" \
  -H "Content-Type: application/json" \
  -d '{
    "serialNumbers": ["AA2501 10000050"],
    "is_signed": false
  }'
```

---

## Cavab

### Uğurlu Cavab

```json
{
  "err_code": 0,
  "err_msg": "",
  "data": {
    "id": 12345
  }
}
```

**Cavab Sahələri:**

| Sahə | Təsvir |
|------|--------|
| `err_code` | Xəta kodu (0 = uğur) |
| `err_msg` | Xəta mesajı (uğur zamanı boş) |
| `data.id` | Silməni izləmək üçün proses ID |

### Xəta Cavabı

```json
{
  "err_code": 20,
  "err_msg": "Qaimə tapılmadı və ya göstərilən qovluqda deyil"
}
```

---

## Silmə Prosesini İzləmək

Silmə asinxron əməliyyat olduğundan, gedişatı izləmək üçün proses ID-dən istifadə edin:

### Proses Statusunu Yoxlayın

```bash
curl -X POST https://company.vatportal.az/api/job/read_proc_status \
  -H "Content-Type: application/json" \
  -d '{
    "username": "istifadəçi_adım",
    "password": "parolum",
    "procId": 12345
  }'
```

### Status Cavabı

```json
{
  "res": 1,
  "err_code": 0,
  "data": {
    "status": 2,
    "status_desc": "Proses Uğurla Tamamlandı",
    "stats": {
      "deleted": 3
    }
  }
}
```

**Status Kodları:**
- `2` - Proses uğurla tamamlandı
- `3` - Proses xətalarla tamamlandı

---

## Tam İş Axını Nümunəsi

```bash
# Addım 1: Silməni başlatın
curl -X DELETE https://company.vatportal.az/api/etx/delete \
  -H "Content-Type: application/json" \
  -d '{
    "username": "istifadəçim",
    "password": "parolum",
    "serialNumbers": [
      "AA2501 10000001",
      "AA2501 10000002"
    ],
    "is_signed": false
  }'

# Cavab: {"err_code": 0, "data": {"id": 123}}

# Addım 2: Silmə gedişatını izləyin
curl -X POST https://company.vatportal.az/api/job/read_proc_status \
  -H "Content-Type: application/json" \
  -d '{
    "username": "istifadəçim",
    "password": "parolum",
    "procId": 123
  }'

# Cavab: {"status": 2, "status_desc": "Proses Uğurla Tamamlandı"}
```

---

## Xəta İdarəetməsi

### Ümumi Xətalar

| Xəta Kodu | Təsvir | Həll |
|-----------|--------|------|
| 4 | Boş istifadəçi adı və ya parol | Etibarlı icazə məlumatları təmin edin |
| 5 | Yanlış istifadəçi adı və ya parol | İcazə məlumatlarını yoxlayın |
| 20 | Qaimə tapılmadı və ya göstərilən qovluqda deyil | Qaime nömrələrini və `is_signed` parametrini yoxlayın |

### Xəta Kodu 20: Qaimə Tapılmadı

Bu xəta baş verir:
- Qaime seriya/nömrə mövcud deyil
- Qaimə yanlış qovluqdadır (`is_signed` parametrini yoxlayın)
- Qaimə statusu silməyə icazə vermir (artıq təsdiqlənib, və s.)

**Həll:**
1. Qaime seriya/nömrənin düzgün olduğunu yoxlayın
2. Qaimələrin imzalanmış və ya qaralama olduğunu yoxlayın (`is_signed` düzgün təyin edin)
3. Portalda qaimə statusunu yoxlayın
4. Massivdəki bütün qaimələrin eyni qovluqda olduğuna əmin olun

### Hamısı-və-ya-Heç Biri Silmə

**Vacib:** Silmə əməliyyatı tranzaksiyalıdır. Əgər `serialNumbers` massivindəki HƏR HANS BİR qaimə silinə bilmirsə:
- **HEÇ BİR qaimə** silinməyəcək
- Bütün sorğu uğursuz olacaq
- Xəta kodu problemi göstərəcək

```json
// Nümunə: Bir yanlış qaimə hamısının uğursuz olmasına səbəb olur
{
  "serialNumbers": [
    "AA2501 10000001",  // ✅ Etibarlı, silinə bilər
    "AA2501 10000002",  // ✅ Etibarlı, silinə bilər
    "XX9999 99999999"   // ❌ Etibarsız - bütün sorğu uğursuz olur
  ]
}
// Nəticə: Qaimələrin HEÇ BİRİ silinmir
```

---

## Ən Yaxşı Təcrübələr

### ✅ EDİN

- **Silmədən əvvəl qaimə statusunu yoxlayın**
- **Düzgün `is_signed` parametrindən istifadə edin** (qovluğa uyğun gəlsin)
- **Qaytarılan proses ID istifadə edərək prosesi izləyin**
- **Bütün qaimələri eyni qovluqda saxlayın** (hamısı qaralama VƏ YA hamısı imzalanmış)
- **Audit məqsədləri üçün silmə sorğularını qeyd edin**
- **İstifadəçilərə təsdiqləməzdən əvvəl silmənin tamamlandığını yoxlayın**

### ❌ ETMƏYİN

- **Eyni sorğuda qaralama və imzalanmış qaimələri qarışdırmayın**
- **Yoxlamadan silməyin** - əməliyyat geri qaytarıla bilməz
- **Proses monitorinqini nəzərdən qaçırmayın** - silmə səssiz şəkildə uğursuz ola bilər
- **Xəta zamanı dərhal yenidən cəhd etməyin** - əvvəlcə səbəbi araşdırın
- **Təsdiqlənmiş qaimələri silməyin** - əvəzinə rəsmi ləğv prosesindən istifadə edin

---

## Təhlükəsizlik Mülahizələri

### İcazə

- Yalnız icazəli portal istifadəçiləri qaimələri silə bilər
- Təhlükəsizlik üçün token əsaslı autentifikasiya tövsiyə olunur
- Silmə əməliyyatları sistemdə qeyd edilir

### Audit İzi

Sistem aşağıdakıları qeyd edir:
- Qaimələri kim sildi
- Nə vaxt silindilər
- Hansı qaimələr silindi
- Silmə uğurlu oldu və ya olmadı

---

## İstifadə Halları

### İstifadə Halı 1: Qaralama Qaimələri Toplu Silmək

**Ssenarı:** ERP sistemi xətalı qaimələr yaratdı, silmək və yenidən yaratmaq lazımdır

```json
{
  "username": "erp_istifadəçi",
  "password": "təhlükəsiz_parol",
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

### İstifadə Halı 2: Təsdiq Gözləyən İmzalanmış Qaiməni Silmək

**Ssenarı:** İmzalanmış qaimə qarşı tərəfə göndərilib, lakin xəta var. Silmək və düzəldilmiş versiya göndərmək lazımdır.

```json
{
  "username": "maliyyə_istifadəçi",
  "password": "təhlükəsiz_parol",
  "serialNumbers": [
    "MT2412 25000100"
  ],
  "is_signed": true
}
```

### İstifadə Halı 3: Test Qaimələrini Təmizləmək

**Ssenarı:** İnkişaf/test zamanı yaradılmış qaimələri silmək lazımdır

```json
{
  "username": "test_istifadəçi",
  "password": "test_parol",
  "serialNumbers": [
    "TEST01 99990001",
    "TEST01 99990002"
  ],
  "is_signed": false
}
```

---

## Miqrasiya Bələdçisi

Əgər hazırda `/api/inv/delete` istifadə edirsiniz və e-taxes-dan silmək lazımdırsa:

### Əvvəl (Yalnız Portal)

```json
DELETE /api/inv/delete
{
  "username": "istifadəçi",
  "password": "parol",
  "ids": ["ERP-INV-001", "ERP-INV-002"]  // ERP ID-ləri
}
```

### Sonra (Portal + E-taxes)

```json
DELETE /api/etx/delete
{
  "username": "istifadəçi",
  "password": "parol",
  "serialNumbers": ["AA2501 10000001", "AA2501 10000002"],  // Qaime nömrələri
  "is_signed": false
}
```

**Əsas Fərqlər:**
1. ERP ID əvəzinə qaime seriya/nömrə istifadə edin
2. Qaimələrin imzalanıb-imzalanmadığını göstərin
3. Proses ID istifadə edərək izləyin
4. Yalnız portaldan deyil, e-taxes-dan da silir

---

## Əlaqəli Endpointlər

- [Qaimələri İdxal və Yükləmək](./import-upload-invoices.md) - Qaimələr yaradın və yükləyin
- [E-taxes-dan Endirmə](./download-from-etaxes.md) - E-taxes-dan qaimələri endirin
- [Proses Statusunu Oxuyun](./import-upload-invoices.md#proses-statusunu-izləyin) - Proses gedişatını izləyin
- [Xəta Kodları](./error-codes.md) - Tam xəta arayışı

---

## Növbəti Addımlar

- [Qaimə statusları haqqında oxuyun →](./invoice-types.md)
- [Xəta idarəetməsi haqqında öyrənin →](./error-codes.md)
- [Autentifikasiya metodlarına baxın →](./authentication.md)

---

[← Sənədləşməyə qayıt](./README.md)
