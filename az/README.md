# VatPortal API Sənədləşməsi

**Azərbaycanın e-taxes iş axınınızı avtomatlaşdırın**

ERP sisteminizi birbaşa e-taxes.gov.az ilə birləşdirin. Qaimə-fakturaları yükləyin, elektron imzalayın və qaime nömrələrini avtomatik əldə edin.

---

## 👋 VatPortal API ilə yenisiz?

**5 dəqiqədə başlayın** • Mürəkkəb quraşdırma yoxdur • İstehsal üçün hazırdır

[**🚀 Sürətli Başlanğıc Bələdçisi**](./quickstart.md) &nbsp;&nbsp;&nbsp; [**🤝 Mən Partnerəm**](./partner-integration.md) &nbsp;&nbsp;&nbsp; [**📞 Dəstək Alın**](#dəstək)


---

## İnteqrasiya Yolunuzu Seçin

### 🏢 Müəssisə Müştərisi
**Tək şirkət inteqrasiyası** • Birbaşa idarəetmə • Sadə autentifikasiya

**Bir şirkətin ERP sistemini** e-taxes ilə inteqrasiya edirsinizsə idealdır.

**Sizin subdomeniniz:** `şirkətiniz.vatportal.az`

[**Müəssisə İnteqrasiyasına Başlayın →**](./quickstart.md)

### 🤝 Partner / Xidmət Təminatçısı
**Çox-müştərili inteqrasiya** • Toplu əməliyyatlar • Müştəri seqreqasiyası

**Bir neçə şirkəti idarə edirsinizsə** və onların adından əməliyyat aparmaq lazımdırsa idealdır.

**Sizin subdomeniniz:** `partneradınız.vatportal.az` • `phone` parametri tələb olunur

[**Partner İnteqrasiyasına Başlayın →**](./partner-integration.md)

---

## 🎯 Nə etmək istəyirsiniz?

**Ən çox istifadə olunan tapşırıqlar:**

- **📤 Qaimələri e-taxes-a yükləmək** → [İdxal və Yükləmə Bələdçisi](./import-upload-invoices.md)
- **📥 E-taxes-dan qaimələri endirmək** → [Endirmə Bələdçisi](./download-from-etaxes.md)
- **🔐 Autentifikasiya və tokenlər quraşdırmaq** → [Autentifikasiya Quraşdırması](./authentication.md)
- **⚠️ Xətaları aradan qaldırmaq** → [Xəta Kodları Məlumatı](./error-codes.md)
- **📋 Real nümunələr görmək** → [Kod Nümunələri](./examples.md)

---

## 🔑 Əsas Xüsusiyyətlər

- ✅ Qaimə-fakturaları e-taxes.gov.az-a idxal və yükləmə
- ✅ Elektron imza dəstəyi
- ✅ Qaime nömrələrini və qaimə məlumatlarını əldə etmə
- ✅ Onlayn sistemdən qaimələri endirmə
- ✅ Status və qaimə növünə görə təkmilləşdirilmiş filtrasiya (v8.0)
- ✅ Qaralama və gözləyən qaimələri silmə
- ✅ 2FA yeniləməsi ilə token əsaslı autentifikasiya
- ✅ 18% ƏDV və sadələşdirilmiş vergitutma dəstəyi

## 🆕 v8.0-da Yeniliklər

**Buraxılış:** 7 Avqust 2025

- Endirmə metoduna **qaimə statusuna görə** filtrasiya əlavə edildi
- Endirmə metoduna **qaimə növü/tipinə görə** filtrasiya əlavə edildi
- `inc_status` və `exc_status` parametrləri təkmilləşdirildi
- `inc_kinds` və `exc_kinds` parametrləri təkmilləşdirildi
- Toplu qaimə endirmələri üzərində daha yaxşı nəzarət

[Tam dəyişikliklər jurnalı →](./CHANGELOG.md)

---

**Versiya:** 8.0 • **Son Yenilənmə:** 7 Avqust 2025 • **Əsas URL:** `https://<şirkət>.vatportal.az/api`

## 📋 Sənədləşmə Naviqasiyası

### 🚀 Başlanğıc
- [**Sürətli Başlanğıc**](./quickstart.md) - 5 dəqiqədə işə başlayın
- [**Autentifikasiya**](./authentication.md) - İcazə məlumatlarını və tokenləri quraşdırın (istifadəçi adı/parol və token əsaslı)
- [**Partner İnteqrasiyası**](./partner-integration.md) - Partnerlər üçün çox-müştərili inteqrasiya

### 📡 API Endpointləri

#### Qaimə Əməliyyatları
- [**Qaimələri İdxal və Yükləmək**](./import-upload-invoices.md) - e-taxes.gov.az-a qaimələri yükləyin
- [**E-taxes-dan Endirmə**](./download-from-etaxes.md) - e-taxes.gov.az-dan qaimələri endirin
- [**E-taxes Qaimələrini Silmək**](./delete-etaxes-invoices.md) - Portal və e-taxes-dan qaimələri silin

#### Məlumat Sənədləri
- [**Qaimə Növləri**](./invoice-types.md) - Bütün dəstəklənən qaimə növləri və istifadəsi
- [**Xəta Kodları**](./error-codes.md) - Tam xəta məlumatı və problemlərin həlli
- [**Nümunələr**](./examples.md) - Real sorğu və cavab nümunələri

### 📚 Əlavə Resurslar
- [**Dəyişikliklər Jurnalı**](./CHANGELOG.md) - Versiya tarixi və buraxılış qeydləri
- [**Sənədləşmə Bələdçisi**](./DOCUMENTATION_GUIDE.md) - Sənədləşməyə töhfə vermək üçün bələdçi

## 📡 Mövcud Endpointlər

### Qaimə İdarəetməsi
- [`POST /api/inv/import_upload_invoices.php`](./import-upload-invoices.md#endpoint) - Qaimələri idxal və yükləmə
- [`GET /api/inv/read_invoices`](./import-upload-invoices.md#read-invoices-with-qaime-numbers) - Qaime nömrələri ilə qaimələri oxumaq
- [`DELETE /api/inv/delete`](./import-upload-invoices.md#delete-unsubmitted-invoices) - Təqdim edilməmiş qaimələri ERP ID-yə görə silmək

### Proses Monitorinqi
- [`GET /api/job/read_proc_status`](./import-upload-invoices.md#monitor-process) - Proses statusunu yoxlamaq
- [`GET /api/job/read_proc_data`](./import-upload-invoices.md#monitor-process) - Ətraflı proses məlumatlarını əldə etmək

### E-taxes İnteqrasiyası
- [`POST /api/etx/import`](./download-from-etaxes.md#addım-1-yükləmə-prosesini-başladın) - Onlayn sistemdən qaimələri endirmək (v7.0)
- [`GET /api/etx/read_invoices`](./download-from-etaxes.md#addım-2-yüklənmiş-qaimə-məlumatlarını-oxuyun) - Xam qaime məlumatlarını oxumaq (v7.0+)
- [`DELETE /api/etx/delete`](./delete-etaxes-invoices.md#endpoint) - Qaimələri seriya/nömrəyə görə silmək

### Autentifikasiya
- [`GET /api/auth/reset_start.php`](./authentication.md#addım-1-token-yeniləməsini-başladın) - Token yeniləməsini başlatmaq
- [`POST /api/auth/reset_confirm.php`](./authentication.md#addım-2-token-yeniləməsini-təsdiq-edin) - Token yeniləməsini təsdiqləmək

## 🌍 Dəstəklənən Qaimə Növləri

| Növ | Təsvir |
|------|--------|
| `defaultInvoice` | Standart mal/xidmət qaiməsi |
| `advanceInvoice` | Avans ödənişi qaiməsi |
| `goodsTransfer` | Anbarlar arası mal köçürməsi |
| `returnInvoice` | Məhsul qaytarma qaiməsi |
| `agent` | Agent/komissiya satışı |
| `exportNoteInvoice` | İxrac qeydi ilə ixrac |

[Bütün qaimə növlərinə baxın →](./invoice-types.md)

## 🔐 Autentifikasiya Üsulları

1. **İstifadəçi adı/Parol** - Sorğu gövdəsinə daxil edin
2. **Token Başlığı** - `x-vatpapikey` HTTP başlığını göndərin

```bash
# Token ilə nümunə
curl -X POST https://company.vatportal.az/api/inv/import_upload_invoices.php \
  -H "x-vatpapikey: SİZİN_TOKENİNİZ" \
  -H "Content-Type: application/json" \
  -d @request.json
```

## 📊 Cavab Formatı

Bütün endpointlər standart strukturla JSON qaytarır:

```json
{
  "res": 1,
  "err_code": 0,
  "err_msg": "",
  "data": {
    // Endpointa xas məlumat
  }
}
```

- `res`: Uğur göstəricisi (1 = uğurlu, 0 = xəta)
- `err_code`: Xəta kodu (0 = xəta yoxdur, [xəta kodlarına](./error-codes.md) baxın)
- `err_msg`: İnsan oxuya bilən xəta mesajı
- `data`: Cavab yükü

## 🛠️ SDK-lar və Alətlər

Hazırda VatPortal API birbaşa HTTP sorğuları tələb edir. İcma SDK-ları:

- **JavaScript/TypeScript** - Tezliklə
- **Python** - Tezliklə
- **C#/.NET** - Tezliklə

SDK-ya töhfə vermək istəyirsiniz? [Dəstəklə əlaqə saxlayın](#dəstək)

## 📞 Dəstək

- **Sənədləşmə Problemləri:** [Problem yaradın](#)
- **API Dəstəyi:** VatPortal hesab meneceri ilə əlaqə saxlayın
- **Texniki Suallar:** support@amr.az

## 📄 Lisenziya və İstifadə

Bu API qeydiyyatdan keçmiş VatPortal abunəçiləri üçün əlçatandır. İcazə məlumatları abunəliyin tamamlanmasından sonra verilir.

---

**Hazırlayan:** AMR Solutions
**API Versiyası:** 8.0
**Sənədləşmə Versiyası:** 8.0.0
