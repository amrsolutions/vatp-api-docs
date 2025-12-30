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
  "res": 1,
  "err_code": 0,
  "err_msg": "",
  "data": {
    "id": 12345
  }
}
```

## Nümunələr

### Nümunə 1: Yalnız İdxal (Yükləmə Yoxdur)

Qaiməni e-taxes-a yükləmədən idxal edin:

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

**Cavab:**
```json
{
  "res": 1,
  "err_code": 0,
  "err_msg": "",
  "data": {
    "id": 5001
  }
}
```

### Nümunə 2: Çox Məhsullu Qaimə 18% ƏDV və Nəqliyyat Vergisi ilə

Real nümunə çoxlu sətir məhsulları və nəqliyyat vergisi ilə:

```json
{
  "username": "istifadəçi_adınız",
  "password": "parolunuz",
  "upload": true,
  "use_old_sys": false,
  "invoices": [{
    "id": "2024030110400001",
    "main_info": "Müqavilə A001; 08.01.2024",
    "add_info": "Qaimə XYZ 251145; 28.02.2024",
    "seller": "1111111111",
    "buyer": "0000000000",
    "sum_subtotal1": 428.34,
    "sum_aksiz": 0,
    "sum_subtotal2": 428.34,
    "sum_apply_vat_amount": 428.34,
    "sum_skip_vat_amount": 0,
    "sum_vat_zero": 0,
    "sum_vat_free": 0,
    "sum_vat_amount": 77.10,
    "sum_ttax": 14.56,
    "sum_total": 520.00,
    "items": [
      {
        "id": "9947008540",
        "name": "Maye qaz (texniki butan)",
        "unit": "ton",
        "unit_price": 0.400,
        "quantity": 816.61020,
        "barcode": "9947008540000",
        "subtotal1": 326.6441,
        "aksiz": 0,
        "aksizgrade": 0,
        "subtotal2": 326.6441,
        "apply_vat_amount": 326.6441,
        "skip_vat_amount": 0,
        "vat_zero": 0,
        "vat_free": 0,
        "vat_amount": 58.7959,
        "transport_tax": 14.56,
        "total": 400.0000
      },
      {
        "id": "9947008540",
        "name": "Daşınma xərci",
        "unit": "xidmət",
        "unit_price": 0.400,
        "quantity": 254.24,
        "barcode": "9947008540000",
        "subtotal1": 101.6960,
        "aksiz": 0,
        "aksizgrade": 0,
        "subtotal2": 101.6960,
        "apply_vat_amount": 101.6960,
        "skip_vat_amount": 0,
        "vat_zero": 0,
        "vat_free": 0,
        "vat_amount": 18.3053,
        "transport_tax": 0,
        "total": 120.0013
      }
    ]
  }]
}
```

### Nümunə 3: Avans Qaiməsi

Sonradan istinad edilə bilən avans ödənişi qaiməsi:

```json
{
  "username": "istifadəçi_adınız",
  "password": "parolunuz",
  "upload": true,
  "use_old_sys": false,
  "invoices": [{
    "id": "2025010115500001",
    "type": "avans",
    "main_info": "Müqavilə B-2025/105; Avans ödənişi",
    "add_info": "50% əvvəlcədən ödəniş",
    "seller": "1111111111",
    "buyer": "2222222222",
    "sum_subtotal1": 5000.00,
    "sum_aksiz": 0,
    "sum_subtotal2": 5000.00,
    "sum_apply_vat_amount": 5000.00,
    "sum_skip_vat_amount": 0,
    "sum_vat_zero": 0,
    "sum_vat_free": 0,
    "sum_vat_amount": 900.00,
    "sum_ttax": 0,
    "sum_total": 5900.00,
    "items": [{
      "id": "1234567890",
      "name": "Kompüter avadanlıqları (avans)",
      "unit": "dəst",
      "unit_price": 2500.00,
      "quantity": 2,
      "barcode": "1234567890000",
      "subtotal1": 5000.00,
      "aksiz": 0,
      "aksizgrade": 0,
      "subtotal2": 5000.00,
      "apply_vat_amount": 5000.00,
      "skip_vat_amount": 0,
      "vat_zero": 0,
      "vat_free": 0,
      "vat_amount": 900.00,
      "transport_tax": 0,
      "total": 5900.00
    }]
  }]
}
```

### Nümunə 4: Avansa Əsaslanan Qaimə

Əvvəlki avans ödənişinə istinad edən yekun qaimə:

```json
{
  "username": "istifadəçi_adınız",
  "password": "parolunuz",
  "upload": true,
  "use_old_sys": false,
  "invoices": [{
    "id": "2025012015600001",
    "main_qaime": "AA24070000000",
    "main_info": "Malların çatdırılması (avansa əsasən)",
    "seller": "1111111111",
    "buyer": "2222222222"
  }]
}
```

**Qeyd:** `main_qaime` təqdim edildikdə və etibarlı olduqda, digər sahələrin əksəriyyəti boş ola bilər - portal avtomatik olaraq avans qaiməsindən məlumatları əldə edir.

### Nümunə 5: Sadələşdirilmiş Vergitutma - Çox Məhsullu Xidmətlər

Sadələşdirilmiş vergitutma istifadə edən şirkətlər üçün qaimə (ƏDV yoxdur):

```json
{
  "username": "istifadəçi_adınız",
  "password": "parolunuz",
  "upload": true,
  "use_old_sys": false,
  "invoices": [{
    "id": "2024030110400002",
    "main_info": "Müqavilə A001; 08.01.2024",
    "add_info": "Qaimə XYZ 251145; 28.02.2024",
    "seller": "3333333333",
    "buyer": "0000000000",
    "sum_subtotal1": 450.00,
    "sum_aksiz": 0,
    "sum_subtotal2": 450.00,
    "sum_apply_vat_amount": 0,
    "sum_skip_vat_amount": 0,
    "sum_vat_zero": 0,
    "sum_vat_free": 0,
    "sum_vat_amount": 0,
    "sum_ttax": 0,
    "sum_total": 450.00,
    "items": [
      {
        "id": "9965111040",
        "name": "Texniki xidmət № QH2222219499/9",
        "unit": "ədəd",
        "unit_price": 250.00,
        "quantity": 1,
        "barcode": "9965111040000",
        "subtotal1": 250.00,
        "aksiz": 0,
        "aksizgrade": 0,
        "subtotal2": 250.00,
        "apply_vat_amount": 0,
        "skip_vat_amount": 0,
        "vat_zero": 0,
        "vat_free": 0,
        "vat_amount": 0,
        "transport_tax": 0,
        "total": 250.00
      },
      {
        "id": "9965111041",
        "name": "Konsultasiya xidməti № QH1234535788/1",
        "unit": "saat",
        "unit_price": 50.00,
        "quantity": 4,
        "barcode": "9965111041000",
        "subtotal1": 200.00,
        "aksiz": 0,
        "aksizgrade": 0,
        "subtotal2": 200.00,
        "apply_vat_amount": 0,
        "skip_vat_amount": 0,
        "vat_zero": 0,
        "vat_free": 0,
        "vat_amount": 0,
        "transport_tax": 0,
        "total": 200.00
      }
    ]
  }]
}
```

**Qeyd:** Sadələşdirilmiş vergitutma üçün bütün ƏDV sahələri 0-a təyin edilir.

### Nümunə 6: Anbarlar Arası Malların Köçürülməsi

Şirkətin öz anbarları arasında malların daxili köçürülməsi:

```json
{
  "username": "istifadəçi_adınız",
  "password": "parolunuz",
  "upload": true,
  "use_old_sys": false,
  "invoices": [{
    "id": "2025011816700001",
    "type": "goodsTransfer",
    "main_info": "Anbar arası mal köçürülməsi",
    "add_info": "Bakı anbarından Gəncə anbarına",
    "seller": "1111111111",
    "buyer": "1111111111",
    "transp_reg_number": "10-AA-123",
    "from_wh": "WH001",
    "to_wh": "WH002",
    "wh_field": 1,
    "sum_subtotal1": 8500.00,
    "sum_aksiz": 0,
    "sum_subtotal2": 8500.00,
    "sum_apply_vat_amount": 0,
    "sum_skip_vat_amount": 0,
    "sum_vat_zero": 0,
    "sum_vat_free": 0,
    "sum_vat_amount": 0,
    "sum_ttax": 0,
    "sum_total": 8500.00,
    "items": [
      {
        "id": "7777777001",
        "name": "Plastik qablaşdırma materialı",
        "unit": "rulon",
        "unit_price": 50.00,
        "quantity": 100,
        "barcode": "7777777001000",
        "subtotal1": 5000.00,
        "aksiz": 0,
        "aksizgrade": 0,
        "subtotal2": 5000.00,
        "apply_vat_amount": 0,
        "skip_vat_amount": 0,
        "vat_zero": 0,
        "vat_free": 0,
        "vat_amount": 0,
        "transport_tax": 0,
        "total": 5000.00
      },
      {
        "id": "7777777002",
        "name": "Karton qutular (50x50x50)",
        "unit": "ədəd",
        "unit_price": 7.00,
        "quantity": 500,
        "barcode": "7777777002000",
        "subtotal1": 3500.00,
        "aksiz": 0,
        "aksizgrade": 0,
        "subtotal2": 3500.00,
        "apply_vat_amount": 0,
        "skip_vat_amount": 0,
        "vat_zero": 0,
        "vat_free": 0,
        "vat_amount": 0,
        "transport_tax": 0,
        "total": 3500.00
      }
    ]
  }]
}
```

**Qeyd:** Malların köçürülməsi üçün `seller` və `buyer` eyni VÖEN-dir (daxili köçürmə). `wh_field: 1` anbar koduna görə axtarış deməkdir.

### Nümunə 7: Partner İnteqrasiyası - Çox Məhsullu Qaimə

Partner öz müştəriləri üçün qaimə idarə edir (`phone` parametri tələb olunur):

```json
{
  "username": "partner_istifadəçi_adı",
  "password": "partner_parolu",
  "phone": "994501234567",
  "upload": true,
  "invoices": [{
    "id": "PTR-2025-001-0045",
    "main_info": "Müştəri sifarişi #0045",
    "add_info": "Partner vasitəsilə",
    "seller": "4444444444",
    "buyer": "5555555555",
    "sum_subtotal1": 1500.00,
    "sum_aksiz": 0,
    "sum_subtotal2": 1500.00,
    "sum_apply_vat_amount": 1500.00,
    "sum_skip_vat_amount": 0,
    "sum_vat_zero": 0,
    "sum_vat_free": 0,
    "sum_vat_amount": 270.00,
    "sum_ttax": 0,
    "sum_total": 1770.00,
    "items": [
      {
        "id": "8888888001",
        "name": "Notebook Lenovo ThinkPad",
        "unit": "ədəd",
        "unit_price": 800.00,
        "quantity": 1,
        "barcode": "8888888001000",
        "subtotal1": 800.00,
        "aksiz": 0,
        "aksizgrade": 0,
        "subtotal2": 800.00,
        "apply_vat_amount": 800.00,
        "skip_vat_amount": 0,
        "vat_zero": 0,
        "vat_free": 0,
        "vat_amount": 144.00,
        "transport_tax": 0,
        "total": 944.00
      },
      {
        "id": "8888888002",
        "name": "Simsiz siçan Logitech",
        "unit": "ədəd",
        "unit_price": 35.00,
        "quantity": 2,
        "barcode": "8888888002000",
        "subtotal1": 70.00,
        "aksiz": 0,
        "aksizgrade": 0,
        "subtotal2": 70.00,
        "apply_vat_amount": 70.00,
        "skip_vat_amount": 0,
        "vat_zero": 0,
        "vat_free": 0,
        "vat_amount": 12.60,
        "transport_tax": 0,
        "total": 82.60
      },
      {
        "id": "8888888003",
        "name": "USB flash 64GB",
        "unit": "ədəd",
        "unit_price": 15.00,
        "quantity": 42,
        "barcode": "8888888003000",
        "subtotal1": 630.00,
        "aksiz": 0,
        "aksizgrade": 0,
        "subtotal2": 630.00,
        "apply_vat_amount": 630.00,
        "skip_vat_amount": 0,
        "vat_zero": 0,
        "vat_free": 0,
        "vat_amount": 113.40,
        "transport_tax": 0,
        "total": 743.40
      }
    ]
  }]
}
```

**Qeyd:** Partnerlər autentifikasiya üçün `phone` parametrini (format: 994XXXXXXXXX) daxil etməlidirlər.

### Nümunə 8: Aksizli Qaimə

Aksiz vergisi tətbiq olunan mallar üçün (alkoqol, tütün, yanacaq):

```json
{
  "username": "istifadəçi_adınız",
  "password": "parolunuz",
  "upload": true,
  "use_old_sys": false,
  "invoices": [{
    "id": "2025011917800001",
    "main_info": "Aksizli məhsullar satışı",
    "add_info": "Müqavilə ALC-2025/015",
    "seller": "1111111111",
    "buyer": "6666666666",
    "sum_subtotal1": 2000.00,
    "sum_aksiz": 800.00,
    "sum_subtotal2": 2800.00,
    "sum_apply_vat_amount": 2800.00,
    "sum_skip_vat_amount": 0,
    "sum_vat_zero": 0,
    "sum_vat_free": 0,
    "sum_vat_amount": 504.00,
    "sum_ttax": 0,
    "sum_total": 3304.00,
    "items": [
      {
        "id": "2208301100",
        "name": "Viski (40% alkoqol) 0.7L",
        "unit": "şüşə",
        "unit_price": 50.00,
        "quantity": 30,
        "barcode": "2208301100000",
        "subtotal1": 1500.00,
        "aksiz": 600.00,
        "aksizgrade": 40,
        "subtotal2": 2100.00,
        "apply_vat_amount": 2100.00,
        "skip_vat_amount": 0,
        "vat_zero": 0,
        "vat_free": 0,
        "vat_amount": 378.00,
        "transport_tax": 0,
        "total": 2478.00
      },
      {
        "id": "2402201000",
        "name": "Siqaret (filter) 20 ədəd qab",
        "unit": "blok",
        "unit_price": 25.00,
        "quantity": 20,
        "barcode": "2402201000000",
        "subtotal1": 500.00,
        "aksiz": 200.00,
        "aksizgrade": 40,
        "subtotal2": 700.00,
        "apply_vat_amount": 700.00,
        "skip_vat_amount": 0,
        "vat_zero": 0,
        "vat_free": 0,
        "vat_amount": 126.00,
        "transport_tax": 0,
        "total": 826.00
      }
    ]
  }]
}
```

**Qeyd:** Formula: `subtotal1 + aksiz = subtotal2`, sonra ƏDV `subtotal2`-yə tətbiq edilir.

### Nümunə 9: Token Autentifikasiyası

İstifadəçi adı/parol əvəzinə API tokeni istifadə etmək:

```bash
curl -X POST https://company.vatportal.az/api/inv/import_upload_invoices.php \
  -H "x-vatpapikey: SİZİN_API_TOKENİNİZ" \
  -H "Content-Type: application/json" \
  -d '{
    "upload": true,
    "use_old_sys": false,
    "invoices": [{
      "id": "2025012018900001",
      "main_info": "Avtomatlaşdırılmış inteqrasiya",
      "add_info": "Token ilə autentifikasiya",
      "seller": "1111111111",
      "buyer": "7777777777",
      "sum_subtotal1": 350.00,
      "sum_aksiz": 0,
      "sum_subtotal2": 350.00,
      "sum_apply_vat_amount": 350.00,
      "sum_skip_vat_amount": 0,
      "sum_vat_zero": 0,
      "sum_vat_free": 0,
      "sum_vat_amount": 63.00,
      "sum_ttax": 0,
      "sum_total": 413.00,
      "items": [{
        "id": "5555555001",
        "name": "Proqram təminatı lisenziyası",
        "unit": "ədəd",
        "unit_price": 350.00,
        "quantity": 1,
        "barcode": "5555555001000",
        "subtotal1": 350.00,
        "aksiz": 0,
        "aksizgrade": 0,
        "subtotal2": 350.00,
        "apply_vat_amount": 350.00,
        "skip_vat_amount": 0,
        "vat_zero": 0,
        "vat_free": 0,
        "vat_amount": 63.00,
        "transport_tax": 0,
        "total": 413.00
      }]
    }]
  }'
```

**Qeyd:** Token Portal → Profil → API-yə Giriş bölməsindən alınır. Token istifadə edildikdə sorğu gövdəsində istifadəçi adı/parol olmur.

### Nümunə 10: Toplu Yükləmə - Çoxlu Qaimələr

Daha yaxşı performans üçün bir sorğuda çoxlu qaimə yükləyin:

```json
{
  "username": "istifadəçi_adınız",
  "password": "parolunuz",
  "upload": true,
  "use_old_sys": false,
  "invoices": [
    {
      "id": "2025012119000001",
      "main_info": "Müqavilə A-2025/101",
      "add_info": "Birinci qaimə",
      "seller": "1111111111",
      "buyer": "2222222222",
      "sum_subtotal1": 500.00,
      "sum_aksiz": 0,
      "sum_subtotal2": 500.00,
      "sum_apply_vat_amount": 500.00,
      "sum_skip_vat_amount": 0,
      "sum_vat_zero": 0,
      "sum_vat_free": 0,
      "sum_vat_amount": 90.00,
      "sum_ttax": 0,
      "sum_total": 590.00,
      "items": [{
        "id": "1111111001",
        "name": "Ofis mebeli - masa",
        "unit": "ədəd",
        "unit_price": 250.00,
        "quantity": 2,
        "barcode": "1111111001000",
        "subtotal1": 500.00,
        "aksiz": 0,
        "aksizgrade": 0,
        "subtotal2": 500.00,
        "apply_vat_amount": 500.00,
        "skip_vat_amount": 0,
        "vat_zero": 0,
        "vat_free": 0,
        "vat_amount": 90.00,
        "transport_tax": 0,
        "total": 590.00
      }]
    },
    {
      "id": "2025012119000002",
      "main_info": "Müqavilə B-2025/202",
      "add_info": "İkinci qaimə",
      "seller": "1111111111",
      "buyer": "3333333333",
      "sum_subtotal1": 1200.00,
      "sum_aksiz": 0,
      "sum_subtotal2": 1200.00,
      "sum_apply_vat_amount": 1200.00,
      "sum_skip_vat_amount": 0,
      "sum_vat_zero": 0,
      "sum_vat_free": 0,
      "sum_vat_amount": 216.00,
      "sum_ttax": 0,
      "sum_total": 1416.00,
      "items": [{
        "id": "2222222001",
        "name": "Printer Canon LBP",
        "unit": "ədəd",
        "unit_price": 400.00,
        "quantity": 3,
        "barcode": "2222222001000",
        "subtotal1": 1200.00,
        "aksiz": 0,
        "aksizgrade": 0,
        "subtotal2": 1200.00,
        "apply_vat_amount": 1200.00,
        "skip_vat_amount": 0,
        "vat_zero": 0,
        "vat_free": 0,
        "vat_amount": 216.00,
        "transport_tax": 0,
        "total": 1416.00
      }]
    },
    {
      "id": "2025012119000003",
      "main_info": "Müqavilə C-2025/303",
      "add_info": "Üçüncü qaimə - Sadələşdirilmiş",
      "seller": "4444444444",
      "buyer": "5555555555",
      "sum_subtotal1": 300.00,
      "sum_aksiz": 0,
      "sum_subtotal2": 300.00,
      "sum_apply_vat_amount": 0,
      "sum_skip_vat_amount": 0,
      "sum_vat_zero": 0,
      "sum_vat_free": 0,
      "sum_vat_amount": 0,
      "sum_ttax": 0,
      "sum_total": 300.00,
      "items": [{
        "id": "3333333001",
        "name": "Təmizlik xidməti",
        "unit": "xidmət",
        "unit_price": 300.00,
        "quantity": 1,
        "barcode": "3333333001000",
        "subtotal1": 300.00,
        "aksiz": 0,
        "aksizgrade": 0,
        "subtotal2": 300.00,
        "apply_vat_amount": 0,
        "skip_vat_amount": 0,
        "vat_zero": 0,
        "vat_free": 0,
        "vat_amount": 0,
        "transport_tax": 0,
        "total": 300.00
      }]
    }
  ]
}
```

**Üstünlüklər:**
- Azaldılmış API çağırışları
- İzləmək üçün vahid proses ID
- Toplu əməliyyatlar üçün daha yaxşı performans

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

[Tam xəta kodlarına baxın →](./error-codes.md)

## Yükləmə Axını

`upload: true` olduqda, proses bu mərhələlərdən keçir:

1. **İdxal** - Qaimə portala saxlanılır
2. **Paket Yaratma** - Qaimə e-taxes üçün paketlənir
3. **Giriş** - E-taxes ilə autentifikasiya (PIN1 tələb olunur)
4. **Yükləmə** - Paket e-taxes-a göndərilir
5. **İmzalama** - Elektron imza (PIN2 tələb olunur)
6. **Tamamlama** - Qaime nömrəsi təyin edilir

[`GET /job/read_proc_status`](./polling-guide.md) endpointindən istifadə edərək tərəqqini izləyin. Tövsiyə olunan tətbiq üçün [Polling Ən Yaxşı Təcrübələri](./polling-guide.md) baxın.

## Qeydlər

- **Minimum ID Uzunluğu:** Qaimə ID ən azı 7 simvol olmalıdır
- **VÖEN Formatı:** Tam olaraq 10 rəqəm olmalıdır
- **Barkod:** Məhsul ID + "000" sonluğu ola bilər
- **ƏDV Hesablanması:** Sistem cəmlərin uyğun gəldiyini yoxlayır

## Növbəti Addımlar

- [Real nümunələrə baxın →](./examples.md)
- [Qaimə növləri arayışı →](./invoice-types.md)
- [Xəta kodları arayışı →](./error-codes.md)
- [Autentifikasiya bələdçisi →](./authentication.md)

---

[← Endpointlərə qayıt](./README.md)
