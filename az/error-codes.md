# Xəta Kodları Arayışı

VatPortal API xəta kodlarına və onların həlli üçün tam bələdçi.

## Ümumi Xəta Kodları

Bu xətalar bütün endpointlər üçün tətbiq olunur:

| Kod | Xəta | Təsvir | Həll |
|-----|------|--------|------|
| `1` | Yanlış Sorğu Metodu | Sorğu POST istifadə etməlidir | `GET` əvəzinə `POST` istifadə edin |
| `2` | Yanlış Content-Type | Başlıq `application/json` olmalıdır | `Content-Type: application/json` təyin edin |
| `3` | Kök Elementlər Çatışmır | Tələb olunan JSON sahələri yoxdur | Sorğu strukturunu yoxlayın |
| `4` | Boş İcazə Məlumatları | İstifadəçi adı və ya parol boşdur | Etibarlı icazə məlumatları təmin edin |
| `5` | Yanlış İcazə Məlumatları | Səhv istifadəçi adı/parol və ya token | Portalda icazə məlumatlarını yoxlayın |
| `6` | Boş Qaimələr Massivi | Sorğuda qaimə yoxdur | Ən azı bir qaimə əlavə edin |
| `7` | Autentifikasiya Xətası | Daxili autentifikasiya sistemi xətası | Dəstəklə əlaqə saxlayın |
| `10,11,12,16` | Daxili Server Xətası | Sistem xətası | Yenidən cəhd edin və ya dəstəklə əlaqə saxlayın |
| `14` | Qaimə Validasiya Xətası | Bir və ya daha çox qaimə yanlışdır | Cavabda ətraflı xətalara baxın |
| `15` | ASAN Nömrəsi Yoxdur | İstifadəçinin konfiqurasiya olunmuş ASAN telefonu yoxdur | Portalda ASAN nömrəsi əlavə edin |
| `17` | Təkrar Qaimə ID | Bu ID ilə qaimə mövcuddur | Unikal qaimə ID istifadə edin |
| `20` | Qaime Tapılmadı | Qaime mövcud deyil və ya səhv qovluq | Qaime nömrəsini və qovluğu yoxlayın |

## Qaimə Səviyyəli Xətalar (err_code: 14)

`err_code: 14` olduqda, xüsusi xəta kodları üçün `data.invoices` massivinə baxın:

```json
{
  "err_code": 14,
  "data": {
    "invoices": [
      [37, 22, 43]
    ]
  }
}
```

Bu o deməkdir ki, qaimə #1-də 3 xəta var: kod 37, 22 və 43.

### Qaimə Strukturu Xətaları

| Kod | Xəta | Həll |
|-----|------|------|
| `20` | VÖEN mövcud deyil | `seller` və ya `buyer` sahəsi əlavə edin |
| `21` | VÖEN yanlışdır | Tam olaraq 10 rəqəm istifadə edin |
| `22` | Əsas məlumat mövcud deyil | `main_info` sahəsi əlavə edin |
| `23` | Əsas məlumat boşdur | Boş olmayan təsvir təmin edin |
| `37` | Qaimə ID mövcud deyil | `id` sahəsi əlavə edin |
| `38` | Qaimə ID çox qısadır | Ən azı 7 simvol istifadə edin |
| `39` | Məhsul massivi yoxdur | `items` massivi əlavə edin |

### Qaimə Məbləğ Xətaları

| Kod | Xəta | Həll |
|-----|------|------|
| `24` | `sum_subtotal1` mövcud deyil | `sum_subtotal1` sahəsi əlavə edin |
| `25` | `sum_subtotal1` yanlışdır | Müsbət ədəd istifadə edin |
| `26` | `sum_subtotal2` mövcud deyil | `sum_subtotal2` sahəsi əlavə edin |
| `27` | `sum_subtotal2` yanlışdır | Müsbət ədəd istifadə edin |
| `28` | `sum_total` mövcud deyil | `sum_total` sahəsi əlavə edin |
| `29` | `sum_total` yanlışdır | Müsbət ədəd istifadə edin |
| `30` | `sum_aksiz` yanlışdır | Qeyri-mənfi ədəd istifadə edin |
| `31` | `sum_apply_vat_amount` yanlışdır | Qeyri-mənfi ədəd istifadə edin |
| `32` | `sum_skip_vat_amount` yanlışdır | Qeyri-mənfi ədəd istifadə edin |
| `33` | `sum_vat_zero` yanlışdır | Qeyri-mənfi ədəd istifadə edin |
| `34` | `sum_vat_free` yanlışdır | Qeyri-mənfi ədəd istifadə edin |
| `35` | `sum_vat_amount` yanlışdır | Qeyri-mənfi ədəd istifadə edin |
| `36` | `sum_ttax` yanlışdır | Qeyri-mənfi ədəd istifadə edin |

### Xüsusi Qaimə Növü Xətaları

| Kod | Xəta | Həll |
|-----|------|------|
| `61` | Yanlış qaimə növü | İstifadə edin: `default`, `avans`, və ya `goodsTransfer` |
| `62` | Avans qaiməsi tapılmadı | `main_qaime` portalda mövcud olduğunu yoxlayın |
| `63` | Mənbə anbarı tapılmadı | `from_wh` dəyərini yoxlayın |
| `64` | Təyinat anbarı tapılmadı | `to_wh` dəyərini yoxlayın |

## Məhsul Səviyyəli Xətalar

Bu xətalar ayrı-ayrı sətir məhsulları ilə problemləri göstərir:

### Məhsul Strukturu Xətaları

| Kod | Xəta | Həll |
|-----|------|------|
| `40` | Məhsul ID mövcud deyil | Məhsula `id` sahəsi əlavə edin |
| `41` | Məhsul ID yanlışdır | Tam olaraq 10 rəqəm istifadə edin |
| `42` | Məhsul adı mövcud deyil | `name` sahəsi əlavə edin |
| `43` | Məhsul adı boşdur | Məhsul adı təmin edin |
| `44` | Ölçü vahidi mövcud deyil | `unit` sahəsi əlavə edin |
| `45` | Ölçü vahidi boşdur | Ölçü vahidi təmin edin (məs., "ədəd", "kq") |
| `50` | Barkod yanlışdır | Yalnız rəqəm simvolları istifadə edin |
| `51` | Barkod rəqəm deyil | Yalnız rəqəmlər istifadə edin |

### Məhsul Məbləğ Xətaları

| Kod | Xəta | Həll |
|-----|------|------|
| `46` | Vahid qiymət mövcud deyil | `unit_price` sahəsi əlavə edin |
| `47` | Vahid qiymət yanlışdır | Müsbət ədəd istifadə edin |
| `48` | Miqdar mövcud deyil | `quantity` sahəsi əlavə edin |
| `49` | Miqdar yanlışdır | Müsbət ədəd istifadə edin |
| `52-60` | Məhsul cəm məbləğləri yanlışdır | Məhsul məbləğ hesablamalarını yoxlayın |

## Ümumi Xəta Ssenariləri

### Ssenarı 1: Autentifikasiya Uğursuz Oldu

**Xəta:**
```json
{
  "err_code": 5,
  "err_msg": ""
}
```

**Yoxlama siyahısı:**
- ✅ İstifadəçi adı/parol düzdür?
- ✅ Token hələ etibarlıdır?
- ✅ Düzgün autentifikasiya metodundan istifadə edilir?
- ✅ Başlıq adı `x-vatpapikey` dir?

### Ssenarı 2: Yanlış Qaimə Strukturu

**Xəta:**
```json
{
  "err_code": 14,
  "data": {
    "invoices": [[37, 43]]
  }
}
```

**Şərh:**
- Qaimə #1-də 2 xəta var
- Xəta 37: Qaimə ID yoxdur
- Xəta 43: Məhsul adı boşdur

**Həll:**
```json
{
  "invoices": [{
    "id": "INV-2025-001",  // ← Əlavə edildi (xəta 37 həll olundu)
    "main_info": "Satış",
    "seller": "1234567890",
    "buyer": "0987654321",
    "sum_subtotal1": 100,
    "sum_subtotal2": 100,
    "sum_total": 100,
    "items": [{
      "id": "PROD-001",
      "name": "Məhsul Adı",  // ← Əlavə edildi (xəta 43 həll olundu)
      "unit": "ədəd",
      "unit_price": 100,
      "quantity": 1,
      "barcode": "PROD-001000",
      "subtotal1": 100,
      "subtotal2": 100,
      "total": 100
    }]
  }]
}
```

## Debaq Məsləhətləri

### 1. Cavab Strukturunu Yoxlayın

Həmişə tam cavabı yoxlayın:

```javascript
fetch(url, options)
  .then(res => res.json())
  .then(data => {
    console.log('Tam cavab:', JSON.stringify(data, null, 2));

    if (data.err_code !== 0) {
      console.error('Xəta:', data.err_code);
      if (data.err_code === 14) {
        console.error('Qaimə xətaları:', data.data.invoices);
      }
    }
  });
```

### 2. Göndərməzdən Əvvəl Yoxlayın

```javascript
function validateInvoice(invoice) {
  const errors = [];

  if (!invoice.id || invoice.id.length < 7) {
    errors.push('Qaimə ID ən azı 7 simvol olmalıdır');
  }

  if (!invoice.seller || invoice.seller.length !== 10) {
    errors.push('Satıcı VÖEN tam olaraq 10 rəqəm olmalıdır');
  }

  if (!invoice.buyer || invoice.buyer.length !== 10) {
    errors.push('Alıcı VÖEN tam olaraq 10 rəqəm olmalıdır');
  }

  if (!invoice.main_info || invoice.main_info.trim() === '') {
    errors.push('Əsas məlumat tələb olunur');
  }

  if (!invoice.items || invoice.items.length === 0) {
    errors.push('Ən azı bir məhsul tələb olunur');
  }

  invoice.items?.forEach((item, idx) => {
    if (!item.name || item.name.trim() === '') {
      errors.push(`Məhsul ${idx + 1}: Ad tələb olunur`);
    }
    if (!item.id || item.id.length !== 10) {
      errors.push(`Məhsul ${idx + 1}: ID 10 rəqəm olmalıdır`);
    }
  });

  return errors;
}

// İstifadə
const errors = validateInvoice(myInvoice);
if (errors.length > 0) {
  console.error('Validasiya xətaları:', errors);
  // Sorğu göndərməyin
} else {
  // Sorğu göndərin
}
```

## Xəta Kodu Sürətli Arayış

### Kritik Xətalar (İcranı Dayandırın)
- **5** - Autentifikasiya uğursuz oldu → İcazə məlumatlarını düzəldin
- **14** - Validasiya uğursuz oldu → Qaimə məlumatlarını düzəldin
- **15** - ASAN yoxdur → Portalda konfiqurasiya edin

### Təkrar Cəhd Oluna Bilən Xətalar
- **10, 11, 12, 16** - Server xətası → Gecikmə ilə yenidən cəhd edin

### Müştəri Xətaları (Kodu Düzəldin)
- **1, 2, 3** - Sorğu formatı → Sorğu strukturunu düzəldin
- **4, 6** - Məlumat çatışmır → Tələb olunan sahələri əlavə edin

## Köməyə ehtiyacınız var?

Xətanı başa düşə bilmirsiniz?

1. **Bu sənədi axtarın** - Xəta kodunu tapmaq üçün Ctrl+F istifadə edin
2. **Nümunələri yoxlayın** - [Nümunələr bölməsinə](../examples/) baxın
3. **JSON-u yoxlayın** - Onlayn JSON validatorundan istifadə edin
4. **curl ilə test edin** - Problemi təcrid edin
5. **Dəstəklə əlaqə saxlayın** - Tam xəta təfərrüatları ilə

---

[← Sənədləşməyə qayıt](../README.md) | [Nümunələrə baxın →](../examples/)
