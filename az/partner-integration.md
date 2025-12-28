# Partner İnteqrasiya Bələdçisi

**Mövcuddur:** Partner Proqramı
**Subdomen Formatı:** `partnername.vatportal.az`

## Ümumi Baxış

VatPortal Partner Proqramı inteqrasiya partnyorlarına tək API inteqrasiyası vasitəsilə bir neçə son istifadəçini idarə etməyə imkan verir. Partnerlər vasitəçi kimi çıxış edir və müştəriləri adından e-taxes əməliyyatlarını idarə edirlər.

## Partner və Müəssisə Modeli

| Xüsusiyyət | Müəssisə Müştəriləri | Partnerlər |
|------------|---------------------|------------|
| **Subdomen** | `company.vatportal.az` | `partnername.vatportal.az` |
| **Müştəri Konteksti** | Tək şirkət (implicit) | Çoxlu müştərilər (`phone` vasitəsilə açıq) |
| **Autentifikasiya** | İstifadəçi adı/parol və ya token | İstifadəçi adı/parol və ya token |
| **Phone Parametri** | Tələb olunmur | **BÜTÜN API çağırışlarında tələb olunur** |
| **İstifadə Halı** | Bir şirkət üçün birbaşa inteqrasiya | Bir neçə şirkəti idarə edən xidmət təminatçısı |

## Partner İnteqrasiyası Necə İşləyir

### 1. Müştəri Qeydiyyat Axını

```
┌─────────────┐
│   Partner   │
└──────┬──────┘
       │
       │ 1. Müştərini portalda qeydiyyatdan keçir
       ├──────────────────────────────────────┐
       │                                      │
       │                              ┌───────▼────────┐
       │                              │  VatPortal     │
       │                              │  Admin İcmal   │
       │                              └───────┬────────┘
       │                                      │
       │ 2. Müştəri təsdiqləndi               │
       │◄─────────────────────────────────────┤
       │                                      │
       │ 3. Müştəri üçün ASAN                 │
       │      telefon nömrələri əlavə et      │
       ├──────────────────────────────────────►
       │                                      │
       │ 4. Müştəri adından əməliyyat         │
       │      aparmaq üçün API-də             │
       │      telefonu istifadə et            │
       └──────────────────────────────────────┘
```

### 2. Telefon Nömrəsi İdarəetməsi

- Partner tərəfindən qeydiyyatdan keçirilmiş hər müştəri **bir və ya bir neçə ASAN telefon nömrəsinə** malik ola bilər
- Telefon nömrələri istifadə edilməzdən əvvəl VatPortal-da qeydiyyatdan keçməlidir
- API çağırışlarındaki `phone` parametri əməliyyatın hansı müştəri üçün olduğunu müəyyən edir
- Format: `994XXXXXXXXX` (Azərbaycan ASAN formatı)

### 3. API Autentifikasiyası

Partnerlər müəssisə müştəriləri ilə eyni autentifikasiya üsullarından istifadə edirlər:

**Variant 1: Sorğu Gövdəsində İstifadəçi Adı/Parol**
```json
{
  "username": "partner_username",
  "password": "partner_password",
  "phone": "994501234567",
  ...
}
```

**Variant 2: HTTP Header-də Token**
```bash
curl -X POST https://partnername.vatportal.az/api/inv/import_upload_invoices.php \
  -H "x-vatpapikey: PARTNER_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "phone": "994501234567",
    "invoices": [...]
  }'
```

**Əsas Fərq:** Partnerlər hər API çağırışında **həmişə `phone` parametrini daxil etməlidirlər**.

## API Çağırışlarına Tələb Olunan Dəyişikliklər

### Müəssisə Müştəriləri Üçün (Cari)
```json
{
  "username": "company_user",
  "password": "company_pass",
  "invoices": [...]
}
```

### Partnerlər Üçün (Phone Parametri ilə)
```json
{
  "username": "partner_user",
  "password": "partner_pass",
  "phone": "994501234567",  // ← TƏLƏБ OLUNUR: Müştərini müəyyən edir
  "invoices": [...]
}
```

## Bütün Endpointlər Phone Parametri Tələb Edir

Partner icazə məlumatlarından istifadə edərkən **BÜTÜN** API endpointlərində `phone` parametri daxil edilməlidir:

### Qaimə Əməliyyatları
- ✅ `POST /inv/import_upload_invoices.php` - `phone` daxil edin
- ✅ `GET /inv/read_invoices` - `phone` daxil edin
- ✅ `DELETE /inv/delete` - `phone` daxil edin

### E-taxes İnteqrasiyası
- ✅ `POST /etx/import` - `phone` daxil edin
- ✅ `GET /etx/read_invoices` - `phone` daxil edin
- ✅ `DELETE /etx/delete` - `phone` daxil edin

### Proses Monitorinqi
- ✅ `GET /job/read_proc_status` - `phone` daxil edin
- ✅ `GET /job/read_proc_data` - `phone` daxil edin

### Autentifikasiya Token Yeniləməsi
- ✅ `GET /auth/reset_start.php` - Sorğu sətirində `phone` daxil edin
- ✅ `POST /auth/reset_confirm.php` - `phone` daxil edin

## Xəta İşləmə

### Phone Parametri Çatışmır

Əgər partner `phone` parametri olmadan API çağırışı etsə:

```json
{
  "res": 0,
  "err_code": 15,
  "err_msg": "Partner hesabları üçün telefon nömrəsi tələb olunur",
  "data": null
}
```

### Yanlış Telefon Nömrəsi

Əgər telefon nömrəsi partner üçün qeydiyyatdan keçməyibsə:

```json
{
  "res": 0,
  "err_code": 16,
  "err_msg": "Telefon nömrəsi tapılmadı və ya bu partner üçün icazəli deyil",
  "data": null
}
```

### Telefon Nömrəsi Format Xətası

```json
{
  "res": 0,
  "err_code": 17,
  "err_msg": "Yanlış telefon nömrəsi formatı. 994XXXXXXXXX istifadə edin",
  "data": null
}
```

## Tam Nümunə: Partner Qaimə Yükləmə

```json
{
  "username": "partner_integration",
  "password": "secure_password",
  "phone": "994501234567",
  "upload": true,
  "invoices": [
    {
      "id": "CUST001-INV-2024-001",
      "main_info": "Müştəri A-ya satış",
      "seller": "1234567890",
      "buyer": "0987654321",
      "sum_subtotal1": 1000.00,
      "sum_subtotal2": 1000.00,
      "sum_apply_vat_amount": 1000.00,
      "sum_vat_amount": 180.00,
      "sum_total": 1180.00,
      "items": [
        {
          "id": "PROD001",
          "name": "Məhsul Adı",
          "unit": "ədəd",
          "unit_price": 10.00,
          "quantity": 100,
          "barcode": "PROD001000",
          "subtotal1": 1000.00,
          "subtotal2": 1000.00,
          "apply_vat_amount": 1000.00,
          "vat_amount": 180.00,
          "total": 1180.00
        }
      ]
    }
  ]
}
```

## Partnerlər Üçün Ən Yaxşı Təcrübələr

### 1. Müştəri İdentifikasiyası
- API çağırışlarından əvvəl həmişə telefon nömrələrini təsdiqləyin
- Sisteminizə müştəri-telefon əlaqələndirməsini saxlayın
- Lazım gələrsə hər müştəri üçün bir neçə telefon nömrəsi idarə edin

### 2. Xəta İşləmə
- Telefon nömrəsi təsdiqləmə xətaları üçün yenidən cəhd məntiqini tətbiq edin
- Audit üçün bütün API çağırışlarını əlaqəli telefon nömrələri ilə qeyd edin
- Son istifadəçilərə aydın xəta mesajları verin

### 3. Təhlükəsizlik
- Partner icazə məlumatlarını təhlükəsiz saxlayın
- İstehsal üçün token əsaslı autentifikasiyadan istifadə edin
- Tokenləri müntəzəm olaraq dəyişdirin
- Telefon nömrələrini heç vaxt icazəsiz istifadəçilərə açmayın

### 4. Test
- Bir neçə müştəri telefon nömrəsi ilə test edin
- Əməliyyatların müştəri başına təcrid olunduğunu yoxlayın
- Kənar halları test edin (yanlış telefon, icazəsiz telefon)

## Müəssisədən Partnerə Miqrasiya

Əgər müəssisə inteqrasiyasından partner inteqrasiyasına keçirsinizsə:

### Addım 1: Partner kimi Qeydiyyatdan Keçin
Hesabınızı partner hesabına çevirmək üçün VatPortal dəstək komandası ilə əlaqə saxlayın.

### Addım 2: Müştərilərinizi Qeydiyyatdan Keçirin
Bütün müştərilərinizi VatPortal partner portalı vasitəsilə əlavə edin.

### Addım 3: İnteqrasiyanızı Yeniləyin
**Bütün** API çağırışlarına `phone` parametrini əlavə edin:

```javascript
// Əvvəl (Müəssisə)
const request = {
  username: "myuser",
  password: "mypass",
  invoices: [...]
};

// Sonra (Partner)
const request = {
  username: "myuser",
  password: "mypass",
  phone: getCustomerPhone(customerId), // ← Bunu əlavə edin
  invoices: [...]
};
```

### Addım 4: Ətraflı Test Edin
Canlı rejimə keçməzdən əvvəl bütün endpointləri müxtəlif müştəri telefon nömrələri ilə test edin.

## Dəstək

Partner proqramı sorğuları üçün:
- **Satış:** Hesab menecərinizlə əlaqə saxlayın
- **Texniki Dəstək:** support@amr.az
- **Sənədləşmə Məsələləri:** [GitHub Issues](https://github.com/amrsolutions/vatp-api-docs/issues)

---

**Son Yenilənmə:** 28 Dekabr 2025
**Tətbiq olunur:** Partner Proqramı iştirakçılarına
