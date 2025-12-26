# Sürətli Başlanğıc Bələdçisi

VatPortal API ilə 5 dəqiqədə işə başlayın.

## İlkin Tələblər

- Aktiv VatPortal abunəliyi
- API icazə məlumatları (istifadəçi adı/parol və ya token)
- Portalda qeydiyyatdan keçmiş ASAN telefon nömrəsi
- Şirkət VÖEN nömrəsi

## Addım 1: İcazə Məlumatlarınızı Əldə Edin

Abunəlikdən sonra alacaqsınız:
- İstifadəçi adı
- Parol
- API Token (ixtiyari, başlıq əsaslı autentifikasiya üçün)

API tokeninizi yaratmaq/görmək üçün:
1. VatPortal-a daxil olun
2. **Profil → API-yə Giriş** bölməsinə keçin
3. "Avtomatik yarat" düyməsini basın və ya xüsusi hash daxil edin
4. "Yadda saxla" düyməsini basın

## Addım 2: İlk Sorğunuzu Göndərin

### Sadə Qaimə İdxalı (Yükləmə Olmadan)

```bash
curl -X POST https://yourcompany.vatportal.az/api/inv/import_upload_invoices.php \
  -H "Content-Type: application/json" \
  -d '{
    "username": "istifadəçi_adınız",
    "password": "parolunuz",
    "upload": false,
    "invoices": [{
      "id": "INV-2025-001",
      "main_info": "Test Qaiməsi",
      "seller": "1234567890",
      "buyer": "0987654321",
      "sum_subtotal1": 100,
      "sum_subtotal2": 100,
      "sum_total": 100,
      "items": [{
        "id": "PROD-001",
        "name": "Test Məhsulu",
        "unit": "ədəd",
        "unit_price": 100,
        "quantity": 1,
        "barcode": "PROD-001000",
        "subtotal1": 100,
        "subtotal2": 100,
        "total": 100
      }]
    }]
  }'
```

### Gözlənilən Cavab

```json
{
  "err_code": 0,
  "err_msg": "",
  "data": {
    "id": 12345
  }
}
```

`id` sahəsi sizin proses ID-nizdir. Statusu yoxlamaq üçün istifadə edin.

## Addım 3: Proses Statusunu Yoxlayın

```bash
curl -X POST https://yourcompany.vatportal.az/api/job/read_proc_status \
  -H "Content-Type: application/json" \
  -d '{
    "username": "istifadəçi_adınız",
    "password": "parolunuz",
    "procId": 12345
  }'
```

### Cavab

```json
{
  "res": 1,
  "err_code": 0,
  "data": {
    "status": 2,
    "status_desc": "Proses uğurla tamamlandı",
    "stats": {
      "imported": 1,
      "packet_created": 0,
      "login_done": false
    }
  }
}
```

**Status Kodları:**
- `2` = Uğurla tamamlandı
- `3` = Xəta ilə tamamlandı

## Addım 4: E-taxes-a Yükləmə (Tam Axın)

Yükləmək və imzalamaq üçün:

```bash
curl -X POST https://yourcompany.vatportal.az/api/inv/import_upload_invoices.php \
  -H "Content-Type: application/json" \
  -d '{
    "username": "istifadəçi_adınız",
    "password": "parolunuz",
    "upload": true,
    "use_old_sys": false,
    "invoices": [{
      "id": "INV-2025-001",
      "main_info": "Yükləmə üçün Real Qaimə",
      "seller": "1234567890",
      "buyer": "0987654321",
      "sum_subtotal1": 118,
      "sum_subtotal2": 118,
      "sum_apply_vat_amount": 118,
      "sum_vat_amount": 21.24,
      "sum_total": 139.24,
      "items": [{
        "id": "PROD-001",
        "name": "Məhsul Adı",
        "unit": "ədəd",
        "unit_price": 118,
        "quantity": 1,
        "barcode": "PROD-001000",
        "subtotal1": 118,
        "subtotal2": 118,
        "apply_vat_amount": 118,
        "vat_amount": 21.24,
        "total": 139.24
      }]
    }]
  }'
```

**Vacib:** `upload: true` olduqda, etməli olacaqsınız:
1. PIN1 kodunu təsdiqləyin (ASAN telefonuna göndərilir)
2. İmzalamaq üçün PIN2 kodunu təsdiqləyin

Bu addımları görmək üçün proses statusunu izləyin.

## Addım 5: Qaime Nömrəsini Əldə Edin

Uğurlu yükləmədən sonra:

```bash
curl -X POST https://yourcompany.vatportal.az/api/inv/read_invoices \
  -H "Content-Type: application/json" \
  -d '{
    "username": "istifadəçi_adınız",
    "password": "parolunuz",
    "ids": ["INV-2025-001"]
  }'
```

### Cavab

```json
{
  "res": 1,
  "err_code": 0,
  "data": {
    "invoices": [{
      "id": "INV-2025-001",
      "qaime": "AA25010000123"
    }],
    "total": 1
  }
}
```

## Token Autentifikasiyasından İstifadə

Gövdədə istifadəçi adı/parol əvəzinə, HTTP başlığından istifadə edin:

```bash
curl -X POST https://yourcompany.vatportal.az/api/inv/import_upload_invoices.php \
  -H "x-vatpapikey: SİZİN_TOKENİNİZ" \
  -H "Content-Type: application/json" \
  -d '{
    "upload": false,
    "invoices": [...]
  }'
```

## Ümumi Xətalar

### Xəta 5: Yanlış icazə məlumatları
```json
{
  "err_code": 5,
  "err_msg": ""
}
```
**Həll:** İstifadəçi adı/parolu və ya tokeni yoxlayın

### Xəta 14: Yanlış qaimə məlumatları
```json
{
  "err_code": 14,
  "err_msg": "",
  "data": {
    "invoices": [[37, 43]]
  }
}
```
**Həll:** Qaimə ID-si yoxdur (37) və məhsul adı boşdur (43). [Xəta kodlarına](./reference/error-codes.md) baxın

### Xəta 6: Boş qaimələr massivi
```json
{
  "err_code": 6,
  "err_msg": ""
}
```
**Həll:** Massivə ən azı bir qaimə daxil edin

## Növbəti Addımlar

- 📖 [Tam autentifikasiya bələdçisini oxuyun](./authentication.md)
- 🔍 [Bütün endpointləri araşdırın](./endpoints/)
- 💡 [Real nümunələrə baxın](./examples/)
- ⚠️ [Xəta kodları arayışı](./reference/error-codes.md)

## Köməyə ehtiyacınız var?

- Autentifikasiyada problem? → [Autentifikasiya Bələdçisi](./authentication.md)
- Qaimə validasiya xətaları? → [Xəta Kodları](./reference/error-codes.md)
- Tam nümunələr istəyirsiniz? → [Nümunələr](./examples/)

---

[← Sənədləşməyə qayıt](./README.md)
