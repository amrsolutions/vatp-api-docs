# VatPortal API Sənədləşmə Bələdçisi

**Versiya:** 8.0
**Son Yenilənmə:** 26 Dekabr 2025

---

## 📖 VatPortal API Sənədləşməsinə Xoş Gəlmisiniz

Bu əhatəli bələdçi sizə ERP sisteminizi VatPortal API vasitəsilə Azərbaycanın e-taxes.gov.az sistemi ilə inteqrasiya etməyə kömək edəcək.

## 🌍 Mövcud Dillər

Bu sənədləşmə iki dildə mövcuddur:

- **English** - [İngilis Sənədləşməsinə baxın](../en/README.md)
- **Azərbaycan** - [Azərbaycan Sənədləşməsinə baxın](./README.md)

İstənilən vaxt əsas səhifədən dillər arasında keçid edə bilərsiniz.

## 📚 Sənədləşmə Strukturu

Sənədləşməmiz sizə sürətli başlamağa və lazımı məlumatı tapmağa kömək etmək üçün təşkil edilib:

### Başlanğıc
1. **[Sürətli Başlanğıc Bələdçisi](./quickstart.md)** - 5 dəqiqədə işə başlayın
2. **[Autentifikasiya](./authentication.md)** - İcazə məlumatlarınızı və tokenlərinizi quraşdırın
3. **[Nümunələr](./examples.md)** - Real kod nümunələri

### API Arayış
- **[Qaimələri İdxal və Yükləmək](./import-upload-invoices.md)** - Əsas endpoint sənədləşməsi
- **[Xəta Kodları](./error-codes.md)** - Həll yolları ilə tam xəta arayışı
- **[Qaimə Növləri](./invoice-types.md)** - Bütün dəstəklənən qaimə növləri və statusları

### Əlavə Resurslar
- **[Dəyişikliklər Jurnalı](./CHANGELOG.md)** - Versiya tarixçəsi və yeniləmələr

## 🚀 Tövsiyə Olunan Öyrənmə Yolu

### Yeni Tərtibatçılar Üçün
1. [Sürətli Başlanğıc Bələdçisi](./quickstart.md) ilə başlayın
2. [Autentifikasiya](./authentication.md) metodlarını nəzərdən keçirin
3. [Nümunələri](./examples.md) sınayın
4. Problemlərin həlli zamanı [Xəta Kodları](./error-codes.md)-na müraciət edin

### İnteqrasiya Komandaları Üçün
1. Quraşdırma üçün [Autentifikasiya](./authentication.md) oxuyun
2. [Qaimələri İdxal və Yükləmək](./import-upload-invoices.md) endpointini öyrənin
3. Biznes məntiqiüçün [Qaimə Növləri](./invoice-types.md) nəzərdən keçirin
4. Tətbiq nümunələri üçün [Nümunələrdən](./examples.md) istifadə edin

## 💡 Ən Yaxşı Təcrübələr

### Təhlükəsizlik
- Həmişə **token əsaslı autentifikasiya** istifadə edin (istifadəçi adı/parol əvəzinə tövsiyə olunur)
- Tokenləri mühit dəyişənlərində təhlükəsiz saxlayın
- Heç vaxt icazə məlumatlarını versiya nəzarətinə commit etməyin
- Bütün API sorğuları üçün HTTPS istifadə edin
- Tokenləri müntəzəm dəyişdirin (minimum 90 gündə bir)

### Xəta İdarəetməsi
- API cavablarında həmişə `err_code` yoxlayın
- Şəbəkə xətaları üçün təkrar cəhd məntiqini tətbiq edin
- Debaq üçün tam kontekstlə xətaları qeyd edin
- API-yə göndərməzdən əvvəl qaimə məlumatlarını yoxlayın

### Performans
- Mümkün olduqda toplu əməliyyatlardan istifadə edin
- Düzgün timeout idarəetməsini tətbiq edin
- Uyğun olduqda cavabları keşləyin
- API çağırış sürətlərini izləyin

## 🆘 Kömək Almaq

### Sənədləşmə Problemləri
Xətalar tapsanız və ya bu sənədləşməni təkmilləşdirmək üçün təklifləriniz varsa:
- Bizimlə əlaqə saxlayın: **support@amr.az**

### Texniki Dəstək
API inteqrasiyası dəstəyi üçün:
- **Email:** support@amr.az
- **Cavab Müddəti:** İş günlərində 24 saat ərzində

### Xüsusiyyət Tələbləri
Yeni API xüsusiyyətləri üçün ideyalarınız var? Hesab meneceri ilə əlaqə saxlayın və ya bizə email göndərin.

## 📄 İstifadə Şərtləri

Bu API sənədləşməsi qeydiyyatdan keçmiş VatPortal abunəçiləri üçün təmin edilir. API-nin istifadəsi VatPortal abunəlik müqavilənizdə təyin edilmiş şərtlərə tabedir.

## 🔔 Yenilənmiş Qalmaq

### Versiya Yeniləmələri
[Dəyişikliklər Jurnalı](./CHANGELOG.md)-nı müntəzəm yoxlayın:
- Yeni xüsusiyyətlər və endpointlər
- Pozucu dəyişikliklər
- Təhlükəsizlik yeniləmələri
- Səhv düzəlişləri

### Cari Versiya
**API Versiyası:** 8.0
**Sənədləşmə Versiyası:** 8.0.0

---

**Köməyə ehtiyacınız var?** Bizimlə support@amr.az ünvanında əlaqə saxlayın

**AMR Solutions tərəfindən hazırlanıb**
