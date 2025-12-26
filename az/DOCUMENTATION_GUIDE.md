# VatPortal API Sənədləşməsi v8.0

**Yaradılıb:** 26 Dekabr 2025
**Format:** Markdown
**Mənbə:** PDF versiya 6.0/8.0-dan yenidən strukturlaşdırılıb

---

## 📦 Nələr Daxildir

Bu sənədləşmə paketi bunları ehtiva edir:

```
vatportal-api-docs/
├── README.md                    # Əsas giriş nöqtəsi
├── quickstart.md                # 5 dəqiqəlik başlanğıc bələdçisi
├── authentication.md            # Tam autentifikasiya bələdçisi
├── CHANGELOG.md                 # Versiya tarixçəsi
├── endpoints/
│   └── import-upload-invoices.md  # Ətraflı endpoint sənədləri
├── reference/
│   ├── error-codes.md          # Tam xəta arayışı
│   └── invoice-types.md        # Növlər və statuslar
└── examples/
    └── README.md               # Real kod nümunələri
```

## 🎯 PDF-ə Nisbətən Əsas Təkmilləşdirmələr

### ✅ Həll Edilmiş Problemlər
- **Versiya Uyğunluğu** - Bütün istinadlar v8.0 deyir (6.0 deyil)
- **Tək Dil** - Hər yerdə Azərbaycan (cədvəllərdən İngilis dili silindi)
- **Strukturlaşdırılmış Naviqasiya** - Aydın iyerarxiya və keçidlər
- **İnteqrasiya Edilmiş Nümunələr** - Hər endpoint ilə kod nümunələri
- **Sürətli Başlanğıc** - 5 dəqiqədə işə başlayın
- **Axtarıla Bilən** - Ctrl+F, Google indeksləməsi ilə işləyir
- **Kopyalana Bilən** - Bütün kod nümunələri istifadəyə hazırdır

### ✨ Yeni Xüsusiyyətlər
- **Sürətli Başlanğıc Bələdçisi** - Sürətli qoşulma
- **Konsolidasiya Edilmiş Autentifikasiya** - Hər iki metod + token yeniləməsi bir yerdə
- **Xəta Kodlarının Həlləri** - Yalnız kodlar deyil, həm də onları necə həll etmək
- **Real Nümunələr** - Kod ilə tam iş axınları
- **Versiya Tarixçəsi** - Miqrasiya bələdçiləri ilə aydın dəyişikliklər jurnalı
- **Tərtibatçı Fokuslu** - Texniki auditoriya üçün yazılıb

## 🚀 Bu Sənədləşmədən Necə İstifadə Etmək

### Seçim 1: Lokal Oxuyun (Markdown Görüntüləyici)
1. `README.md`-ni VS Code, Obsidian və ya istənilən markdown görüntüləyicidə açın
2. Səhifələr arasında naviqasiya etmək üçün keçidlərə klikləyin
3. Kod nümunələrini birbaşa kopyalayın

### Seçim 2: GitHub Pages-də Yerləşdirin (Tövsiyə olunur)
```bash
# 1. GitHub reposu yaradın
gh repo create vatportal-api-docs --public

# 2. Sənədləşməni push edin
cd vatportal-api-docs
git init
git add .
git commit -m "İlkin sənədləşmə v8.0"
git remote add origin https://github.com/your-org/vatportal-api-docs.git
git push -u origin main

# 3. GitHub Pages-i aktivləşdirin
# Repo Settings → Pages → Mənbə: main branch keçin
# Sənədləriniz buradadır: https://your-org.github.io/vatportal-api-docs/
```

### Seçim 3: Statik Sayt Generatorundan İstifadə Edin

**MkDocs istifadə edərək (Python):**
```bash
pip install mkdocs mkdocs-material

# mkdocs.yml yaradın
cat > mkdocs.yml << EOF
site_name: VatPortal API Sənədləşməsi
theme:
  name: material
  palette:
    primary: blue
nav:
  - Ana səhifə: README.md
  - Sürətli Başlanğıc: quickstart.md
  - Autentifikasiya: authentication.md
  - Endpointlər:
    - İdxal və Yükləmə: endpoints/import-upload-invoices.md
  - Arayış:
    - Xəta Kodları: reference/error-codes.md
    - Qaimə Növləri: reference/invoice-types.md
  - Nümunələr: examples/README.md
  - Dəyişikliklər Jurnalı: CHANGELOG.md
EOF

# Lokal olaraq göstərin
mkdocs serve
# Ziyarət edin: http://localhost:8000

# Yerləşdirmə üçün quraşdırın
mkdocs build
```

## 🔄 Sənədləşməni Yenilənmiş Saxlamaq

### API Dəyişdikdə
1. Müvafiq `.md` faylını redaktə edin
2. `CHANGELOG.md`-ni yeniləyin
3. Bütün istinadlarda versiyanı yeniləyin
4. Aydın mesajla commit edin

### Versiya Yüksəltmə Yoxlama Siyahısı
```bash
# 1. Versiya istinadlarını yeniləyin
grep -r "Versiya.*8.0" . --include="*.md"
# Yeni versiya ilə əvəz edin

# 2. Dəyişikliklər jurnalı girişi əlavə edin
# CHANGELOG.md redaktə edin

# 3. Commit
git add .
git commit -m "docs: v9.0-a yüksəlt"
git tag v9.0
git push --tags
```

## 💡 Tövsiyələr

### ✅ EDİN
- **Onlayn yerləşdirin** - GitHub Pages və ya GitBook istifadə edin
- **Axtarışdan istifadə edin** - Statik sayt generatorları axtarış əlavə edir
- **Naviqasiya əlavə edin** - Məzmun cədvəlindən istifadə edin
- **Versiya nəzarəti** - Git-də saxlayın
- **Töhfələri qəbul edin** - Tərtibatçılara sənədləri təkmilləşdirməyə icazə verin

### ❌ ETMƏYİN
- **Yalnız PDF qalın** - Saxlamaq və axtarmaq çətindir
- **Nümunələri buraxmayın** - Tərtibatçılara işləyən kod lazımdır
- **Dəyişikliklər jurnalını unutmayın** - Dəyişənləri izləyin
- **Rəyi nəzərə almayın** - İstifadəçilər səhvləri tapır

## 📖 Növbəti Addımlar

1. **Sənədləri nəzərdən keçirin** - Oxuyun və dəqiqliyi yoxlayın
2. **Hostinq seçin** - Yuxarıdakı seçimlərdən birini seçin
3. **Komanda ilə paylaşın** - Tərtibatçılardan rəy alın
4. **Çatışmayan endpointləri əlavə edin** - Qalan endpointləri sənədləşdirin
5. **SDK-lar yaradın** - Dilə xas sarğılar quraşdırın
6. **Rəy toplayın** - İstifadəçi ehtiyaclarına əsasən təkrarlayın

## 📞 Dəstək

- **Sənədləşmə Problemləri:** GitHub problemi yaradın
- **API Sualları:** support@amrsolutions.az
- **Xüsusiyyət Tələbləri:** Hesab meneceri ilə əlaqə saxlayın

## 📝 Lisenziya

Sənədləşmə: CC BY 4.0
API: Xüsusi (AMR Solutions)

---

**Tərtibatçılar üçün ❤️ ilə hazırlanıb**
**AMR Solutions - VatPortal API Komandası**
