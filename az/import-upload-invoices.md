# Qaimələri İdxal və Yükləmək

Qaimələri VatPortal-a idxal edin və ixtiyari olaraq imzalamaq üçün e-taxes.gov.az-a yükləyin.

## Endpoint

```
POST /api/inv/import_upload_invoices.php
```

## Autentifikasiya

- Gövdədə İstifadəçi adı/Parol, VƏ YA
- `x-vatpapikey` başlığı vasitəsilə Token

## Sorğu Parametrləri

### Kök Səviyyə

| Parametr | Növ | Tələb olunur | Standart | Təsvir |
|----------|-----|--------------|----------|--------|
| `username` | string | Bəli* | - | İstifadəçi adı (token istifadə etmədikdə) |
| `password` | string | Bəli* | - | Parol (token istifadə etmədikdə) |
| `upload` | boolean | Xeyr | `false` | E-taxes-a yükləmək və ya yox |
| `use_old_sys` | boolean | Xeyr | `false` | Köhnə (yaşıl) e-taxes sistemindən istifadə et |
| `invoices` | array | Bəli | - | Qaimə obyektləri massivi |

*Token autentifikasiyası istifadə edilmədikdə tələb olunur

### Qaimə Obyekti

| Sahə | Növ | Tələb olunur | Standart | Təsvir |
|------|-----|--------------|----------|--------|
| `id` | string | Bəli | avto | ERP-nizdən unikal qaimə ID (min 7 simvol) |
| `type` | string | Xeyr | `"default"` | Qaimə növü: `default`, `avans`, `goodsTransfer` |
| `main_info` | string | Bəli | - | Əsas təsvir |
| `seller` | string | Bəli | - | Satıcı VÖEN (10 rəqəm) |
| `buyer` | string | Bəli | - | Alıcı VÖEN (10 rəqəm) |
| `sum_subtotal1` | decimal | Bəli | - | Malların ümumi qiyməti |
| `sum_subtotal2` | decimal | Bəli | - | Aksiz sonrası ümumi məbləğ |
| `sum_apply_vat_amount` | decimal | Xeyr | `0` | ƏDV tətbiq olunan məbləğ |
| `sum_vat_amount` | decimal | Xeyr | `0` | Ödənilməli ümumi ƏDV |
| `sum_total` | decimal | Bəli | - | Yekun ümumi məbləğ |
| `items` | array | Bəli | - | Sətir məhsulları massivi |

### Qaimə Məhsulu Obyekti

| Sahə | Növ | Tələb olunur | Standart | Təsvir |
|------|-----|--------------|----------|--------|
| `id` | string | Bəli | - | Məhsul kodu (10 rəqəm) |
| `name` | string | Bəli | - | Məhsul adı |
| `unit` | string | Bəli | - | Ölçü vahidi (məs., "ədəd", "kq") |
| `unit_price` | decimal | Bəli | - | Vahid qiyməti |
| `quantity` | decimal | Bəli | - | Miqdar |
| `barcode` | string | Bəli | - | Barkod (məhsul kodu + "000" ola bilər) |
| `subtotal1` | decimal | Bəli | - | Ümumi qiymət |
| `subtotal2` | decimal | Bəli | - | Aksiz sonrası ümumi |
| `apply_vat_amount` | decimal | Xeyr | `0` | ƏDV tətbiq olunan məbləğ |
| `vat_amount` | decimal | Xeyr | `0` | Ödənilməli ƏDV |
| `total` | decimal | Bəli | - | Məhsul ümumi |

## Cavab

```json
{
  "err_code": 0,
  "err_msg": "",
  "data": {
    "id": 12345
  }
}
```

## Nümunələr

### Nümunə 1: Yalnız İdxal (Yükləmə Yoxdur)

```bash
curl -X POST https://company.vatportal.az/api/inv/import_upload_invoices.php \
  -H "x-vatpapikey: SİZİN_TOKENİNİZ" \
  -H "Content-Type: application/json" \
  -d '{
    "upload": false,
    "invoices": [{
      "id": "INV-2025-001",
      "main_info": "Test satışı",
      "seller": "1234567890",
      "buyer": "0987654321",
      "sum_subtotal1": 100,
      "sum_subtotal2": 100,
      "sum_total": 100,
      "items": [{
        "id": "0000000001",
        "name": "Test Məhsulu",
        "unit": "ədəd",
        "unit_price": 100,
        "quantity": 1,
        "barcode": "0000000001000",
        "subtotal1": 100,
        "subtotal2": 100,
        "total": 100
      }]
    }]
  }'
```

### Nümunə 2: 18% ƏDV ilə Standart Qaimə

```json
{
  "username": "istifadəçi_adınız",
  "password": "parolunuz",
  "upload": true,
  "invoices": [{
    "id": "2024030110400001",
    "main_info": "Müqavilə A001; 08.01.2024",
    "seller": "1111111111",
    "buyer": "0000000000",
    "sum_subtotal1": 2008.47,
    "sum_subtotal2": 2008.47,
    "sum_apply_vat_amount": 2008.47,
    "sum_vat_amount": 361.53,
    "sum_total": 2370,
    "items": [{
      "id": "0000000000",
      "name": "Çox bahalı qutu",
      "unit": "TON",
      "unit_price": 66.94915,
      "quantity": 30,
      "barcode": "0000000000000",
      "subtotal1": 2008.4745,
      "subtotal2": 2008.4745,
      "apply_vat_amount": 2008.4745,
      "vat_amount": 361.52541,
      "total": 2370
    }]
  }]
}
```

### Nümunə 3: Sadələşdirilmiş Vergitutma (ƏDV Yoxdur)

```json
{
  "username": "istifadəçi_adınız",
  "password": "parolunuz",
  "upload": true,
  "invoices": [{
    "id": "INV-SIMPLE-001",
    "main_info": "Müqavilə A001; 08.01.2024",
    "seller": "1111111111",
    "buyer": "0000000000",
    "sum_subtotal1": 91.72,
    "sum_subtotal2": 91.72,
    "sum_total": 91.72,
    "items": [{
      "id": "9965111040",
      "name": "Sadələşdirilmiş vergitutma altında xidmət",
      "unit": "ədəd",
      "unit_price": 91.72,
      "quantity": 1,
      "barcode": "9965111040000",
      "subtotal1": 91.72,
      "subtotal2": 91.72,
      "total": 91.72
    }]
  }]
}
```

## Xəta Cavabları

### Validasiya Xətaları (err_code: 14)

```json
{
  "err_code": 14,
  "data": {
    "invoices": [[37, 43]]
  }
}
```

Massiv `[37, 43]` bu xüsusi qaimə üçün xəta kodlarını ehtiva edir:
- `37` = Qaimə ID mövcud deyil
- `43` = Məhsul adı boşdur

[Tam xəta kodlarına baxın →](../reference/error-codes.md)

## Yükləmə Axını

`upload: true` olduqda, proses bu mərhələlərdən keçir:

1. **İdxal** - Qaimə portala saxlanılır
2. **Paket Yaratma** - Qaimə e-taxes üçün paketlənir
3. **Giriş** - E-taxes ilə autentifikasiya (PIN1 tələb olunur)
4. **Yükləmə** - Paket e-taxes-a göndərilir
5. **İmzalama** - Elektron imza (PIN2 tələb olunur)
6. **Tamamlama** - Qaime nömrəsi təyin edilir

[Proses Statusunu Oxu](./read-process-status.md) endpointindən istifadə edərək tərəqqini izləyin.

## Qeydlər

- **Minimum ID Uzunluğu:** Qaimə ID ən azı 7 simvol olmalıdır
- **VÖEN Formatı:** Tam olaraq 10 rəqəm olmalıdır
- **Barkod:** Məhsul ID + "000" sonluğu ola bilər
- **ƏDV Hesablanması:** Sistem cəmlərin uyğun gəldiyini yoxlayır

## Növbəti Addımlar

- [Proses statusunu yoxlayın →](./read-process-status.md)
- [Qaimə məlumatlarını əldə edin →](./read-process-data.md)
- [Qaime nömrələrini oxuyun →](./read-invoices.md)
- [Xəta kodları arayışı →](../reference/error-codes.md)

---

[← Endpointlərə qayıt](./README.md)
