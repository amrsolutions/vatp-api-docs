# Polling Ən Yaxşı Təcrübələri

**Əlçatan olduğu tarix:** v1.0 (1 İyun, 2021)

## İcmal

Bir çox VatPortal API əməliyyatları asinxrondur, yəni dərhal proses ID-si qaytarır və fonda işləməyə davam edir. Bu əməliyyatların statusunu yoxlamaq üçün [`GET /job/read_proc_status`](./import-upload-invoices.md#monitor-process) endpointindən istifadə etməlisiniz.

Bu bələdçi effektiv və etibarlı pollinqin tətbiqi üçün ən yaxşı təcrübələri təqdim edir.

---

## Polling-dən Nə Vaxt İstifadə Etmək Lazımdır

Aşağıdakı əməliyyatlardan hər hansı birini başlatdıqdan sonra proses statusu endpointini poll edin:

- **Qaimələri İdxal və Yükləmək** - [`POST /inv/import_upload_invoices.php`](./import-upload-invoices.md)
- **E-taxes-dan Endirmək** - [`POST /etx/import`](./download-from-etaxes.md)
- **E-taxes Qaimələrini Silmək** - [`DELETE /etx/delete`](./delete-etaxes-invoices.md)

Bütün bu endpointlər cavabda proses ID-si qaytarır:

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

Status yeniləmələri üçün poll etmək üçün bu `id`-dən istifadə edin.

---

## Tövsiyə Olunan Polling Strategiyası

### Əsas Polling Nümunəsi

```javascript
async function pollProcessStatus(procId, maxAttempts = 60) {
  const pollInterval = 2000; // 2 saniyə

  for (let attempt = 1; attempt <= maxAttempts; attempt++) {
    const response = await fetch('https://company.vatportal.az/api/job/read_proc_status', {
      method: 'GET',
      headers: {
        'Content-Type': 'application/json',
        'x-vatpapikey': 'SİZİN_TOKENİNİZ'
      },
      body: JSON.stringify({ procId })
    });

    const data = await response.json();

    // Prosesin tamamlanıb-tamamlanmadığını yoxlayın
    if (data.data.status === 2) {
      console.log('✅ Proses uğurla tamamlandı');
      return { success: true, data: data.data };
    }

    if (data.data.status === 3) {
      console.log('❌ Proses xətalarla tamamlandı');
      return { success: false, data: data.data };
    }

    // Proses hələ də işləyir, növbəti poll-dan əvvəl gözləyin
    console.log(`⏳ Cəhd ${attempt}/${maxAttempts} - Proses hələ də işləyir...`);
    await new Promise(resolve => setTimeout(resolve, pollInterval));
  }

  // Vaxt bitdi
  throw new Error('Polling vaxt bitməsi: Proses vaxtında tamamlanmadı');
}
```

### Eksponensial Geri Çəkilmə Nümunəsi

Daha yaxşı effektivlik və azaldılmış server yükü üçün eksponensial geri çəkilmədən istifadə edin:

```javascript
async function pollWithBackoff(procId, maxAttempts = 30) {
  let interval = 1000; // 1 saniyə ilə başlayın
  const maxInterval = 10000; // 10 saniyədə məhdudlaşdırın

  for (let attempt = 1; attempt <= maxAttempts; attempt++) {
    const response = await fetch('https://company.vatportal.az/api/job/read_proc_status', {
      method: 'GET',
      headers: {
        'Content-Type': 'application/json',
        'x-vatpapikey': 'SİZİN_TOKENİNİZ'
      },
      body: JSON.stringify({ procId })
    });

    const data = await response.json();

    // Terminal vəziyyətləri yoxlayın
    if (data.data.status === 2) {
      return { success: true, data: data.data };
    }

    if (data.data.status === 3) {
      return { success: false, data: data.data };
    }

    // Eksponensial geri çəkilmə ilə gözləyin
    console.log(`⏳ Cəhd ${attempt} - Yenidən cəhddən əvvəl ${interval}ms gözləyir...`);
    await new Promise(resolve => setTimeout(resolve, interval));

    // İntervalı artırın (eksponensial geri çəkilmə)
    interval = Math.min(interval * 1.5, maxInterval);
  }

  throw new Error('Polling vaxt bitməsi');
}
```

---

## Əməliyyat Növünə Görə Polling İntervalları

Müxtəlif əməliyyatların müxtəlif tipik tamamlanma müddətləri var. Polling strategiyanızı buna uyğun tənzimləyin:

| Əməliyyat | Tipik Müddət | Tövsiyə Olunan İlkin İnterval | Tövsiyə Olunan Maks İnterval |
|-----------|--------------|-------------------------------|------------------------------|
| **Yalnız İdxal** (upload: false) | 5-15 saniyə | 1 saniyə | 3 saniyə |
| **İdxal və Yükləmə** (upload: true) | 30-90 saniyə | 2 saniyə | 5 saniyə |
| **İdxal və Yükləmə (böyük dəstə)** | 1-5 dəqiqə | 3 saniyə | 10 saniyə |
| **E-taxes-dan Endirmə** | 20-60 saniyə | 2 saniyə | 5 saniyə |
| **E-taxes Qaimələrini Silmək** | 10-30 saniyə | 1 saniyə | 3 saniyə |

**Qeyd:** Faktiki müddət aşağıdakılardan asılıdır:
- Emal olunan qaimələrin sayı
- E-taxes sistem cavab vaxtı
- Şəbəkə gecikmə
- PIN kodlarının daxil edilməsinin tələb olunub-olunmaması

---

## Proses Status Dəyərləri

Poll edərkən cavabdakı `status` sahəsini yoxlayın:

```json
{
  "res": 1,
  "err_code": 0,
  "data": {
    "status": 2,
    "status_desc": "Proses Uğurla Tamamlandı",
    "stats": {
      "imported": 5,
      "packet_created": 1,
      "login_done": true
    }
  }
}
```

### Status Kodları

| Status | Təsvir | Hərəkət |
|--------|--------|---------|
| `1` | Proses işləyir | Poll etməyə davam edin |
| `2` | Proses uğurla tamamlandı | Poll etməyi dayandırın - Uğur! |
| `3` | Proses xətalarla tamamlandı | Poll etməyi dayandırın - Xəta təfərrüatlarını yoxlayın |

**Vacib:** Yalnız `2` və `3` terminal vəziyyətlərdir. Hər hansı digər status üçün poll etməyə davam edin.

---

## PIN Kodları ilə Yükləmə Əməliyyatlarını İdarə Etmək

`upload: true` olduqda, proses PIN kodu təsdiqini tələb edir:

### PIN1 (Giriş) üçün Monitoring

```javascript
async function monitorForPIN1(procId) {
  while (true) {
    const response = await checkProcessStatus(procId);

    if (response.data.stats.pin1_code) {
      console.log(`📱 PIN1 kodu alındı: ${response.data.stats.pin1_code}`);
      console.log('⚠️ İstifadəçi bu kodu ASAN girişi vasitəsilə təsdiq etməlidir');

      // İstifadəçiyə telefonunu yoxlaması üçün bildiriş göndərin
      notifyUser('ASAN telefonunuzu yoxlayın və PIN1 kodunu təsdiq edin');

      // Girişin nə vaxt edilməsini aşkar etmək üçün poll etməyə davam edin
      while (!response.data.stats.login_done) {
        await sleep(2000);
        response = await checkProcessStatus(procId);
      }

      console.log('✅ Giriş təsdiqləndi');
      break;
    }

    await sleep(2000);
  }
}
```

### PIN2 (İmzalama) üçün Monitoring

```javascript
async function monitorForPIN2(procId) {
  while (true) {
    const response = await checkProcessStatus(procId);

    if (response.data.stats.pin2_code) {
      console.log(`📱 PIN2 kodu alındı: ${response.data.stats.pin2_code}`);
      console.log('⚠️ İstifadəçi qaimələri imzalamaq üçün bu kodu təsdiq etməlidir');

      // İstifadəçiyə telefonunu yoxlaması üçün bildiriş göndərin
      notifyUser('ASAN telefonunuzu yoxlayın və imzalamaq üçün PIN2 kodunu təsdiq edin');

      // İmzalamanın nə vaxt edilməsini aşkar etmək üçün poll etməyə davam edin
      while (!response.data.stats.invoice_signed) {
        await sleep(2000);
        response = await checkProcessStatus(procId);
      }

      console.log('✅ Qaimələr imzalandı');
      break;
    }

    await sleep(2000);
  }
}
```

---

## Tam Nümunə: Polling ilə Tam İş Axını

```javascript
async function uploadInvoicesWithPolling(invoices) {
  try {
    // Addım 1: Yükləməni başlat
    console.log('📤 Qaimə yükləməsi başlayır...');
    const uploadResponse = await fetch('https://company.vatportal.az/api/inv/import_upload_invoices.php', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'x-vatpapikey': 'SİZİN_TOKENİNİZ'
      },
      body: JSON.stringify({
        upload: true,
        use_old_sys: false,
        invoices: invoices
      })
    });

    const uploadData = await uploadResponse.json();

    if (uploadData.err_code !== 0) {
      throw new Error(`Yükləmə uğursuz oldu: ${uploadData.err_msg}`);
    }

    const procId = uploadData.data.id;
    console.log(`✅ Yükləmə başladıldı. Proses ID: ${procId}`);

    // Addım 2: İdxal tamamlanması üçün monitoring
    console.log('⏳ İdxal gedişatı izlənir...');
    await waitForImport(procId);

    // Addım 3: PIN1 üçün monitoring
    console.log('⏳ PIN1 kodu gözlənilir...');
    await monitorForPIN1(procId);

    // Addım 4: Paket yükləməsi üçün monitoring
    console.log('⏳ Paketlər e-taxes-a yüklənir...');
    await waitForUpload(procId);

    // Addım 5: PIN2 üçün monitoring
    console.log('⏳ PIN2 kodu gözlənilir...');
    await monitorForPIN2(procId);

    // Addım 6: Son tamamlanmağı gözləyin
    console.log('⏳ Tamamlanır...');
    const finalStatus = await pollUntilComplete(procId);

    if (finalStatus.status === 2) {
      console.log('🎉 Bütün qaimələr uğurla yükləndi və imzalandı!');
      console.log(`📊 Statistika: ${JSON.stringify(finalStatus.stats, null, 2)}`);
      return { success: true, stats: finalStatus.stats };
    } else {
      console.log('❌ Proses xətalarla tamamlandı');
      console.log(`⚠️ Xəta: ${finalStatus.stats.etx_exception}`);
      return { success: false, error: finalStatus.stats.etx_exception };
    }

  } catch (error) {
    console.error('💥 Xəta:', error.message);
    throw error;
  }
}

// Tamamlanana qədər poll etmək üçün köməkçi funksiya
async function pollUntilComplete(procId, maxAttempts = 60) {
  let interval = 2000; // 2 saniyə

  for (let attempt = 1; attempt <= maxAttempts; attempt++) {
    const response = await checkProcessStatus(procId);

    if (response.data.status === 2 || response.data.status === 3) {
      return response.data;
    }

    console.log(`⏳ Cəhd ${attempt}/${maxAttempts}...`);
    await sleep(interval);
  }

  throw new Error('Polling vaxt bitməsi');
}

function sleep(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}
```

---

## Ən Yaxşı Təcrübələr

### ✅ EDİN

- Server yükünü azaltmaq üçün **eksponensial geri çəkilmədən istifadə edin**
- **Ağlabatan vaxt məhdudiyyətləri təyin edin** (əksər əməliyyatlar üçün 2-5 dəqiqə)
- Poll etməyi dayandırmaq üçün **terminal vəziyyətləri izləyin** (status 2 və ya 3)
- **Xətaları qeyri-adi şəkildə idarə edin** - e-taxes xətaları üçün `etx_exception` sahəsini yoxlayın
- PIN kodları tələb olunduqda **istifadəçilərə bildiriş göndərin**
- Debugging və monitoring üçün **polling cəhdlərini qeyd edin**
- Daha yaxşı təhlükəsizlik üçün istifadəçi adı/parol əvəzinə **token autentifikasiyasından istifadə edin**

### ❌ ETMƏYİN

- **Çox tez-tez poll etməyin** - Minimum 1 saniyə intervallar
- **Qeyri-müəyyən müddətə poll etməyin** - Həmişə maksimum cəhd limiti təyin edin
- **Status kodlarını nəzərə almayın** - Həm `status`, həm də `err_code` yoxlayın
- **Xəta idarəetməsini atlayın** - Proses status 3 (xətalar) ilə tamamlana bilər
- **Tamamlandıqdan sonra poll etməyin** - Status 2 və ya 3 olduqda dayandırın
- **POST metodundan istifadə etməyin** - `read_proc_status` GET endpointidir

---

## Polling Zamanı Xəta İdarəetməsi

### E-taxes Xətalarını Yoxlayın

```javascript
async function checkForErrors(procId) {
  const response = await checkProcessStatus(procId);

  // E-taxes istisnalarını yoxlayın
  if (response.data.stats.etx_exception) {
    const errors = response.data.stats.etx_exception;

    if (errors.includes('NEED RETRY')) {
      console.log('⚠️ E-taxes sistemi məşğuldur - əməliyyat avtomatik olaraq təkrar cəhd ediləcək');
      // Poll etməyə davam edin - sistem təkrar cəhd edəcək
    } else {
      console.log('❌ E-taxes xətası:', errors);
      // Bunlar biznes məntiq xətalarıdır (yanlış VÖEN, məbləğlər uyğun gəlmir və s.)
      // Poll etməyi dayandırın və xətanı bildirin
      throw new Error(`E-taxes xətası: ${errors.join(', ')}`);
    }
  }
}
```

### Şəbəkə Xətalarını İdarə Etmək

```javascript
async function pollWithRetry(procId) {
  const maxNetworkRetries = 3;
  let networkRetry = 0;

  while (networkRetry < maxNetworkRetries) {
    try {
      const response = await checkProcessStatus(procId);
      networkRetry = 0; // Uğur halında sıfırlayın

      // Cavabı emal edin...

    } catch (error) {
      networkRetry++;
      console.log(`⚠️ Şəbəkə xətası (${networkRetry}/${maxNetworkRetries}): ${error.message}`);

      if (networkRetry >= maxNetworkRetries) {
        throw new Error('Şəbəkə xətası: Təkrar cəhdlərdən sonra qoşulmaq alınmadı');
      }

      // Təkrar cəhddən əvvəl gözləyin
      await sleep(2000 * networkRetry); // Artan geri çəkilmə
    }
  }
}
```

---

## Performans Optimallaşdırması

### Dəstə Emalı

Bir neçə dəstə yükləyərkən, bütün prosesləri eyni vaxtda poll etməyin:

```javascript
// ❌ PİS: Hamısını eyni anda poll etmək
const processes = await Promise.all(batches.map(batch => uploadBatch(batch)));
await Promise.all(processes.map(proc => pollUntilComplete(proc.id)));

// ✅ YAXŞI: Ardıcıl emal
for (const batch of batches) {
  const proc = await uploadBatch(batch);
  await pollUntilComplete(proc.id);
  console.log(`✅ Dəstə ${batch.id} tamamlandı`);
}
```

### Ətraflı Məlumat üçün Proses Məlumatından İstifadə Edin

Ətraflı gedişat məlumatı üçün [`GET /job/read_proc_data`](./import-upload-invoices.md#read-process-data)-dan istifadə edin:

```javascript
async function getDetailedProgress(procId) {
  const response = await fetch('https://company.vatportal.az/api/job/read_proc_data', {
    method: 'GET',
    headers: {
      'Content-Type': 'application/json',
      'x-vatpapikey': 'SİZİN_TOKENİNİZ'
    },
    body: JSON.stringify({ procId })
  });

  const data = await response.json();

  // Ətraflı qaimə emal məlumatını əldə edin
  console.log('Emal edilmiş qaimələr:', data.data.invoices);
  console.log('E-tax qaimələri:', data.data.etax_invoices);

  return data.data;
}
```

---

## Polling Tətbiqinizi Test Etmək

### Əvvəlcə Yalnız İdxal ilə Test Edin

Əsas pollinqi yoxlamaq üçün `upload: false` ilə test etməyə başlayın:

```javascript
// Sadə test halı - PIN kodları tələb olunmur
const testResponse = await uploadInvoices({
  upload: false,  // E-taxes əlaqəsi yoxdur
  invoices: [testInvoice]
});

const result = await pollUntilComplete(testResponse.data.id);
console.log('Test nəticəsi:', result);
```

### İnkişaf üçün Mock Cavab

```javascript
// Polling məntiqini test etmək üçün mock
function mockProcessStatus(attempt) {
  if (attempt < 5) {
    return { data: { status: 1, stats: {} } }; // İşləyir
  } else if (attempt < 10) {
    return { data: { status: 1, stats: { imported: 1 } } }; // İdxal edildi
  } else {
    return { data: { status: 2, stats: { imported: 1 } } }; // Tamamlandı
  }
}
```

---

## Monitoring və Qeydiyyat

### Tövsiyə Olunan Qeyd Formatı

```javascript
function logPollAttempt(attempt, procId, status) {
  const timestamp = new Date().toISOString();
  console.log(JSON.stringify({
    timestamp,
    event: 'poll_attempt',
    attempt,
    procId,
    status: status.status,
    status_desc: status.status_desc,
    stats: status.stats
  }));
}
```

### İzləmə üçün Metrikalar

- Ümumi polling cəhdləri
- Əməliyyat növünə görə orta tamamlanma vaxtı
- Vaxt bitməsi xətalarının tezliyi
- E-taxes xətalarının tezliyi
- PIN kodu təsdiqi üçün orta vaxt

---

## Əlaqəli Sənədlər

- [Qaimələri İdxal və Yükləmək](./import-upload-invoices.md) - Əsas yükləmə endpointi
- [E-taxes-dan Endirmək](./download-from-etaxes.md) - Endirmə endpointi
- [E-taxes Qaimələrini Silmək](./delete-etaxes-invoices.md) - Silmə endpointi
- [Xəta Kodları](./error-codes.md) - Tam xəta arayışı

---

[← Sənədləşməyə qayıt](./README.md)
