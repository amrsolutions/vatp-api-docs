# VatPortal API Sənədləşməsi

**Versiya:** 8.0
**Son Yenilənmə:** 7 Avqust 2025
**Əsas URL:** `https://<şirkət>.vatportal.az/api`

## 🚀 VatPortal API nədir?

VatPortal API üçüncü tərəf ERP sistemlərinə Azərbaycanın e-taxes.gov.az sistemi ilə proqramlı şəkildə əlaqə qurmağa imkan verir. Qaimə-fakturaları yükləyin, onları elektron imzalayın, qaime nömrələrini əldə edin və vergi sənədlərini RESTful JSON API vasitəsilə idarə edin.

## 📋 Sürətli Keçidlər

- [**Sürətli Başlanğıc**](./quickstart.md) - 5 dəqiqədə işə başlayın
- [**Autentifikasiya**](./authentication.md) - İcazə məlumatlarını və tokenləri quraşdırın
- [**Qaimələri İdxal və Yükləmək**](./import-upload-invoices.md) - Əsas endpoint sənədləşməsi
- [**Xəta Kodları**](./error-codes.md) - Problemlərin həlli üzrə bələdçi
- [**Nümunələr**](./examples.md) - Real istifadə halları

## 🔑 Əsas Xüsusiyyətlər

- ✅ Qaimə-fakturaları e-taxes.gov.az-a idxal və yükləmə
- ✅ Elektron imza dəstəyi
- ✅ Qaime nömrələrini və qaimə məlumatlarını əldə etmə
- ✅ Onlayn sistemdən qaimələri endirmə
- ✅ Status və qaimə növünə görə təkmilləşdirilmiş filtrasiya (v8.0)
- ✅ Qaralama və gözləyən qaimələri silmə
- ✅ 2FA yeniləməsi ilə token əsaslı autentifikasiya
- ✅ 18% ƏDV və sadələşdirilmiş vergitutma dəstəyi

## 📡 Mövcud Endpointlər

### Qaimə İdarəetməsi
- `POST /inv/import_upload_invoices.php` - Qaimələri idxal və yükləmə
- `GET /inv/read_invoices` - Qaime nömrələri ilə qaimələri oxumaq
- `DELETE /inv/delete` - Təqdim edilməmiş qaimələri ERP ID-yə görə silmək

### Proses Monitorinqi
- `GET /job/read_proc_status` - Proses statusunu yoxlamaq
- `GET /job/read_proc_data` - Ətraflı proses məlumatlarını əldə etmək

### E-taxes İnteqrasiyası
- `POST /etx/import` - Onlayn sistemdən qaimələri endirmək (v7.0)
- `GET /etx/read_invoices` - Xam qaime məlumatlarını oxumaq (v7.0+)
- `DELETE /etx/delete` - Qaimələri seriya/nömrəyə görə silmək

### Autentifikasiya
- `GET /auth/reset_start.php` - Token yeniləməsini başlatmaq
- `POST /auth/reset_confirm.php` - Token yeniləməsini təsdiqləmək

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

## 🆕 v8.0-da Yeniliklər

**Buraxılış:** 7 Avqust 2025

- Endirmə metoduna **qaimə statusuna görə** filtrasiya əlavə edildi
- Endirmə metoduna **qaimə növünə/tipinə görə** filtrasiya əlavə edildi
- Təkmilləşdirilmiş `inc_status` və `exc_status` parametrləri
- Təkmilləşdirilmiş `inc_kinds` və `exc_kinds` parametrləri
- Toplu qaimə endirmələri üzərində daha yaxşı nəzarət

[Tam dəyişikliklər jurnalı →](./CHANGELOG.md)

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
