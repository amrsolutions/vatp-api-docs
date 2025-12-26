# Autentifikasiya

VatPortal API iki autentifikasiya metodunu dəstəkləyir:

1. **İstifadəçi Adı/Parol** - JSON gövdəsində icazə məlumatları
2. **Token əsaslı** - HTTP başlığında API açarı

## Metod 1: İstifadəçi Adı/Parol

Hər sorğu gövdəsinə icazə məlumatlarını daxil edin:

```json
{
  "username": "istifadəçi_adınız",
  "password": "parolunuz",
  // ... digər parametrlər
}
```

### Nümunə

```bash
curl -X POST https://company.vatportal.az/api/inv/read_invoices \
  -H "Content-Type: application/json" \
  -d '{
    "username": "john_doe",
    "password": "secure_password_123",
    "ids": ["INV-001"]
  }'
```

### Üstünlüklər və Çatışmazlıqlar

✅ Tətbiq etmək sadədir
✅ Token idarəetməsinə ehtiyac yoxdur
❌ Hər sorğuda icazə məlumatları
❌ Parolları dəyişdirmək çətindir

## Metod 2: Token əsaslı (Tövsiyə olunur)

API tokenini HTTP başlığı vasitəsilə göndərin:

```bash
curl -X POST https://company.vatportal.az/api/inv/read_invoices \
  -H "x-vatpapikey: SİZİN_TOKENİNİZ" \
  -H "Content-Type: application/json" \
  -d '{
    "ids": ["INV-001"]
  }'
```

### Üstünlüklər və Çatışmazlıqlar

✅ Daha təhlükəsizdir (sorğularda parol yoxdur)
✅ Tokenləri dəyişdirmək asandır
✅ Daha təmiz sorğu yükləri
✅ Müstəqil olaraq ləğv edilə bilər
❌ İlkin quraşdırma tələb olunur

## API Tokeninizi Əldə Etmək

### Addım 1: Portal Parametrlərinə Daxil Olun

1. `https://yourcompany.vatportal.az` ünvanında VatPortal-a daxil olun
2. **Profil → API-yə Giriş** bölməsinə keçin

### Addım 2: Token Yaradın

İki seçiminiz var:

**Seçim A: Avtomatik yaratma**
```
1. "Avtomatik yarat" düyməsini basın
2. "Yadda saxla" düyməsini basın
3. Yaradılmış hash açarını kopyalayın
```

**Seçim B: Xüsusi hash**
```
1. Öz hash açarınızı daxil edin (təhlükəsiz mətn)
2. "Yadda saxla" düyməsini basın
3. Bu hash-dən token kimi istifadə edin
```

### Addım 3: Tokendən İstifadə Edin

Tokeni sorğu başlıqlarına əlavə edin:

```
x-vatpapikey: yaradılmış_tokeniniz
```

## Token Yeniləməsi (2FA Axını)

Tokeninizi proqramlı şəkildə yeniləmək/yenidən yaratmaq lazımdırsa:

### Addım 1: Yeniləməni Başladın

```bash
curl -X GET "https://company.vatportal.az/api/auth/reset_start.php?login=istifadəçi_adınız"
```

**Cavab:**
```json
{
  "plain_value": "təsadüfi_çağırış_mətni_xyz"
}
```

### Addım 2: İmzanı Hesablayın

Çağırışı imzalamaq üçün HMAC-SHA256 istifadə edin:

```javascript
// JavaScript nümunəsi
const crypto = require('crypto');

const plainValue = "təsadüfi_çağırış_mətni_xyz";
const secretHash = "portaldan_gizli_hash";

const signature = crypto
  .createHmac('sha256', secretHash)
  .update(plainValue)
  .digest('hex');

console.log(signature); // Bunu 'key' kimi göndərin
```

```python
# Python nümunəsi
import hmac
import hashlib

plain_value = "təsadüfi_çağırış_mətni_xyz"
secret_hash = "portaldan_gizli_hash"

signature = hmac.new(
    secret_hash.encode(),
    plain_value.encode(),
    hashlib.sha256
).hexdigest()

print(signature)  # Bunu 'key' kimi göndərin
```

### Addım 3: Yeniləməni Təsdiqləyin

```bash
curl -X POST https://company.vatportal.az/api/auth/reset_confirm.php \
  -H "Content-Type: application/json" \
  -d '{
    "login": "istifadəçi_adınız",
    "key": "hesablanmış_imza"
  }'
```

**Cavab:**
```json
{
  "token": "yeni_api_tokeniniz"
}
```

### Addım 4: Tətbiqinizi Yeniləyin

Tətbiqinizin konfiqurasiyasında köhnə tokeni yeni tokenlə əvəz edin.

## Təhlükəsizlik Ən Yaxşı Təcrübələri

### ✅ EDİN

- **Tokenləri təhlükəsiz saxlayın** - Mühit dəyişənlərindən və ya gizli menecerlərindən istifadə edin
- **Yalnız HTTPS istifadə edin** - Heç vaxt HTTP üzərindən icazə məlumatları göndərməyin
- **Tokenləri müntəzəm dəyişdirin** - Minimum 90 gündə bir dəyişdirin
- **Token metodundan istifadə edin** - İstifadəçi adı/parol əvəzinə başlıq əsaslı autentifikasiyaya üstünlük verin
- **İstifadəni izləyin** - Şübhəli fəaliyyət üçün API çağırışlarını izləyin
- **Xəbərdarlıqlar qurun** - Autentifikasiya uğursuzluqları barədə bildiriş alın

### ❌ ETMƏYİN

- **İcazə məlumatlarını kodda yazmayın** - Heç vaxt parol/tokenləri git-ə commit etməyin
- **Tokenləri paylaşmayın** - Hər tətbiqin öz tokeni olmalıdır
- **İcazə məlumatlarını qeyd edin** - Həssas məlumatları silmək üçün qeydləri təmizləyin
- **Parolları yenidən istifadə edin** - VatPortal API üçün unikal parol istifadə edin
- **Tokenləri URL-lərdə göstərməyin** - Həmişə tokenlər üçün başlıqlardan istifadə edin

## Xəta İdarəetməsi

### Yanlış İcazə Məlumatları (Xəta 5)

```json
{
  "err_code": 5,
  "err_msg": "Yanlış istifadəçi adı və ya parol"
}
```

**Səbəblər:**
- Yanlış istifadəçi adı/parol
- Yanlış və ya vaxtı keçmiş token
- Token portal konfiqurasiyasına uyğun gəlmir

**Həll:**
1. Portalda icazə məlumatlarını yoxlayın
2. Tokenlər üçün: yenidən yaradın və yeniləyin
3. Parol üçün: portal vasitəsilə sıfırlayın

### Autentifikasiya Xətası (Xəta 7)

```json
{
  "err_code": 7,
  "err_msg": "Autentifikasiya xətası"
}
```

**Səbəblər:**
- Daxili autentifikasiya sistemi problemi
- Hesab deaktiv edilib

**Həll:**
- VatPortal dəstəyi ilə əlaqə saxlayın
- Portalda hesab statusunu yoxlayın

## Kod Nümunələri

### Node.js Token ilə

```javascript
const axios = require('axios');

const API_BASE = 'https://company.vatportal.az/api';
const API_TOKEN = process.env.VATPORTAL_TOKEN;

async function getInvoices(invoiceIds) {
  try {
    const response = await axios.post(
      `${API_BASE}/inv/read_invoices`,
      { ids: invoiceIds },
      {
        headers: {
          'Content-Type': 'application/json',
          'x-vatpapikey': API_TOKEN
        }
      }
    );
    return response.data;
  } catch (error) {
    console.error('API Xətası:', error.response?.data);
    throw error;
  }
}
```

### Python Token ilə

```python
import os
import requests

API_BASE = 'https://company.vatportal.az/api'
API_TOKEN = os.getenv('VATPORTAL_TOKEN')

def get_invoices(invoice_ids):
    response = requests.post(
        f'{API_BASE}/inv/read_invoices',
        json={'ids': invoice_ids},
        headers={
            'Content-Type': 'application/json',
            'x-vatpapikey': API_TOKEN
        }
    )
    response.raise_for_status()
    return response.json()

# İstifadə
invoices = get_invoices(['INV-001', 'INV-002'])
```

### C# Token ilə

```csharp
using System;
using System.Net.Http;
using System.Text;
using System.Threading.Tasks;
using Newtonsoft.Json;

public class VatPortalClient
{
    private readonly HttpClient _client;
    private readonly string _apiToken;
    private const string ApiBase = "https://company.vatportal.az/api";

    public VatPortalClient(string apiToken)
    {
        _apiToken = apiToken;
        _client = new HttpClient();
        _client.DefaultRequestHeaders.Add("x-vatpapikey", _apiToken);
    }

    public async Task<dynamic> GetInvoices(string[] invoiceIds)
    {
        var payload = new { ids = invoiceIds };
        var content = new StringContent(
            JsonConvert.SerializeObject(payload),
            Encoding.UTF8,
            "application/json"
        );

        var response = await _client.PostAsync(
            $"{ApiBase}/inv/read_invoices",
            content
        );

        response.EnsureSuccessStatusCode();
        var json = await response.Content.ReadAsStringAsync();
        return JsonConvert.DeserializeObject(json);
    }
}
```

## Autentifikasiyanı Sınaqdan Keçirmək

İcazə məlumatlarınızı yoxlamaq üçün sürətli test:

```bash
# İstifadəçi adı/parolu test edin
curl -X POST https://company.vatportal.az/api/inv/read_invoices \
  -H "Content-Type: application/json" \
  -d '{
    "username": "test",
    "password": "test",
    "ids": [],
    "date_from": "01.01.2025",
    "date_to": "01.01.2025"
  }'

# Tokeni test edin
curl -X POST https://company.vatportal.az/api/inv/read_invoices \
  -H "x-vatpapikey: tokeniniz" \
  -H "Content-Type: application/json" \
  -d '{
    "ids": [],
    "date_from": "01.01.2025",
    "date_to": "01.01.2025"
  }'
```

Uğurlu olsa, `err_code: 0` cavabı alacaqsınız.

## Problemlərin Həlli

| Problem | Həll |
|---------|------|
| "Yanlış icazə məlumatları" | Portalda istifadəçi adı/parolu yoxlayın |
| "Token işləmir" | Portal parametrlərində tokeni yenidən yaradın |
| "Boş cavab" | Token yeniləmə prosesinin uğurlu olduğunu yoxlayın |
| "401 İcazəsiz" | `x-vatpapikey` başlığının düzgün təyin edildiyinə əmin olun |

## Növbəti Addımlar

- [İlk API çağırışınızı edin →](./quickstart.md)
- [API endpointlərini araşdırın →](./endpoints/)
- [Kod nümunələrinə baxın →](./examples/)

---

[← Sənədləşməyə qayıt](./README.md)
