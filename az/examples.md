# Real Nümunələr

Ümumi VatPortal API istifadə halları üçün praktik nümunələr.

## Nümunə 1: Sadə Məhsul Satışı (18% ƏDV)

**Ssenarı:** 100 AZN + 18 AZN ƏDV = 118 AZN cəmi üçün bir məhsul satın

```bash
curl -X POST https://company.vatportal.az/api/inv/import_upload_invoices.php \
  -H "x-vatpapikey: SİZİN_TOKENİNİZ" \
  -H "Content-Type: application/json" \
  -d '{
    "upload": true,
    "invoices": [{
      "id": "SALE-2025-001",
      "main_info": "Müqavilə №12345, 15.01.2025 tarixli",
      "seller": "1234567890",
      "buyer": "0987654321",
      "sum_subtotal1": 100,
      "sum_subtotal2": 100,
      "sum_apply_vat_amount": 100,
      "sum_vat_amount": 18,
      "sum_total": 118,
      "items": [{
        "id": "PROD-123",
        "name": "Noutbuk Dell Inspiron 15",
        "unit": "ədəd",
        "unit_price": 100,
        "quantity": 1,
        "barcode": "PROD-123000",
        "subtotal1": 100,
        "subtotal2": 100,
        "apply_vat_amount": 100,
        "vat_amount": 18,
        "total": 118
      }]
    }]
  }'
```

## Nümunə 2: Sadələşdirilmiş Vergitutma (ƏDV Yoxdur)

**Ssenarı:** ƏDV qeydiyyatı olmayan kiçik biznes

```json
{
  "upload": true,
  "invoices": [{
    "id": "SIMPLE-2025-001",
    "main_info": "Müqavilə A001; 08.01.2024",
    "seller": "1111111111",
    "buyer": "0000000000",
    "sum_subtotal1": 91.72,
    "sum_subtotal2": 91.72,
    "sum_total": 91.72,
    "items": [{
      "id": "9965111040",
      "name": "Sadələşdirilmiş vergitutma altında konsaltinq xidmətləri",
      "unit": "xidmət",
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

## Nümunə 3: Avans Ödənişi Qaiməsi

**Ssenarı:** Müştəri 50% əvvəlcədən ödəyir

**Addım 1: Avans qaiməsi yaradın**
```json
{
  "upload": true,
  "invoices": [{
    "id": "AVANS-2025-001",
    "type": "avans",
    "main_info": "Sifariş #12345 üçün 50% avans ödənişi",
    "seller": "1111111111",
    "buyer": "0000000000",
    "sum_subtotal1": 500,
    "sum_subtotal2": 500,
    "sum_apply_vat_amount": 500,
    "sum_vat_amount": 90,
    "sum_total": 590,
    "items": [{
      "id": "0000000001",
      "name": "Ofis mebeli üçün avans",
      "unit": "xidmət",
      "unit_price": 500,
      "quantity": 1,
      "barcode": "0000000001000",
      "subtotal1": 500,
      "subtotal2": 500,
      "apply_vat_amount": 500,
      "vat_amount": 90,
      "total": 590
    }]
  }]
}
```

**Nəticə:** Qaime nömrəsi `AA25010000123`

## Nümunə 4: Anbarlar Arası Mal Köçürməsi

**Ssenarı:** İnventarı əsas anbardan filialaseçin

```json
{
  "upload": true,
  "invoices": [{
    "id": "TRANSFER-2025-001",
    "type": "goodsTransfer",
    "main_info": "Sumqayıt filialına köçürmə",
    "transp_reg_number": "10-AA-123",
    "from_wh": "WH-BAKU-MAIN",
    "to_wh": "WH-SUMGAIT-01",
    "wh_field": 1,
    "seller": "1111111111",
    "buyer": "1111111111",
    "sum_subtotal1": 5000,
    "sum_subtotal2": 5000,
    "sum_total": 5000,
    "items": [{
      "id": "PROD-567",
      "name": "Tikinti materialları - sement 50kq",
      "unit": "çuval",
      "unit_price": 25,
      "quantity": 200,
      "barcode": "PROD-567000",
      "subtotal1": 5000,
      "subtotal2": 5000,
      "total": 5000
    }]
  }]
}
```

## Nümunə 5: Status Yoxlaması ilə Tam İş Axını

```javascript
// Addım 1: Qaiməni yükləyin
async function uploadInvoice() {
  const response = await fetch(
    'https://company.vatportal.az/api/inv/import_upload_invoices.php',
    {
      method: 'POST',
      headers: {
        'x-vatpapikey': process.env.VATPORTAL_TOKEN,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        upload: true,
        invoices: [{
          id: 'SALE-2025-100',
          main_info: 'Məhsul satışı',
          seller: '1234567890',
          buyer: '0987654321',
          sum_subtotal1: 100,
          sum_subtotal2: 100,
          sum_apply_vat_amount: 100,
          sum_vat_amount: 18,
          sum_total: 118,
          items: [{
            id: 'PROD-001',
            name: 'Məhsul',
            unit: 'ədəd',
            unit_price: 100,
            quantity: 1,
            barcode: 'PROD-001000',
            subtotal1: 100,
            subtotal2: 100,
            apply_vat_amount: 100,
            vat_amount: 18,
            total: 118
          }]
        }]
      })
    }
  );
  
  const data = await response.json();
  
  if (data.err_code !== 0) {
    throw new Error(`Yükləmə uğursuz: ${data.err_code}`);
  }
  
  return data.data.id; // Proses ID
}

// Addım 2: Tamamlanma üçün sorğu göndərin
async function waitForCompletion(processId) {
  const maxAttempts = 60; // Maksimum 5 dəqiqə
  const pollInterval = 5000; // 5 saniyə
  
  for (let i = 0; i < maxAttempts; i++) {
    const status = await checkStatus(processId);
    
    if (status.status === 2) {
      return { success: true, status };
    }
    
    if (status.status === 3) {
      return { 
        success: false, 
        error: status.stats.etx_exception 
      };
    }
    
    // Hələ emal edilir
    await sleep(pollInterval);
  }
  
  throw new Error('Tamamlanma gözləməsi üçün vaxt bitdi');
}

async function checkStatus(processId) {
  const response = await fetch(
    'https://company.vatportal.az/api/job/read_proc_status',
    {
      method: 'POST',
      headers: {
        'x-vatpapikey': process.env.VATPORTAL_TOKEN,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({ procId: processId })
    }
  );
  
  const data = await response.json();
  return data.data;
}

// Addım 3: Qaime nömrəsini əldə edin
async function getQaimeNumber(invoiceId) {
  const response = await fetch(
    'https://company.vatportal.az/api/inv/read_invoices',
    {
      method: 'POST',
      headers: {
        'x-vatpapikey': process.env.VATPORTAL_TOKEN,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({ ids: [invoiceId] })
    }
  );
  
  const data = await response.json();
  
  if (data.data.invoices.length > 0) {
    return data.data.invoices[0].qaime;
  }
  
  return null;
}

// Tam iş axını
async function completeInvoiceFlow() {
  try {
    // Yüklə
    console.log('Qaimə yüklənir...');
    const processId = await uploadInvoice();
    console.log(`Proses ID: ${processId}`);
    
    // Tamamlanma gözlə
    console.log('Tamamlanma gözlənilir...');
    const result = await waitForCompletion(processId);
    
    if (!result.success) {
      console.error('Yükləmə uğursuz:', result.error);
      return;
    }
    
    console.log('Qaimə uğurla yükləndi!');
    
    // Qaime nömrəsini əldə et
    const qaime = await getQaimeNumber('SALE-2025-100');
    console.log(`Qaime nömrəsi: ${qaime}`);
    
    // Qaime-ni verilənlər bazanızda saxlayın
    await updateInvoiceInDatabase('SALE-2025-100', { qaime });
    
  } catch (error) {
    console.error('Xəta:', error);
  }
}

function sleep(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}
```

## Nümunə 6: Qaimələri Endirmək və Emal Etmək (v8.0)

```python
import requests
import time
from datetime import datetime, timedelta

API_BASE = 'https://company.vatportal.az/api'
API_TOKEN = 'tokeniniz'
PHONE = '994501234567'

def download_approved_invoices():
    """Yalnız təsdiqlənmiş qaimələri son aydan endirin"""
    
    # Tarix aralığını hesablayın
    today = datetime.now()
    last_month = today - timedelta(days=30)
    
    # Endirməni başladın
    response = requests.post(
        f'{API_BASE}/etx/import',
        headers={'x-vatpapikey': API_TOKEN},
        json={
            'phone': PHONE,
            'date_from': last_month.strftime('%d.%m.%Y'),
            'date_to': today.strftime('%d.%m.%Y'),
            'dir': 1,  # Göndərilmiş qovluğu
            'inc_status': 'approved',  # Yalnız təsdiqlənmiş
            'exc_kinds': 'returnInvoice'  # Qaytarmaları istisna et
        }
    )
    
    data = response.json()
    
    if data['err_code'] != 0:
        raise Exception(f"Endirmə uğursuz: {data['err_code']}")
    
    process_id = data['data']['id']
    print(f"Endirmə başladı. Proses ID: {process_id}")
    
    # Tamamlanmanı gözləyin
    while True:
        status = check_download_status(process_id)
        
        if status['status'] == 2:
            print("Endirmə tamamlandı!")
            break
        elif status['status'] == 3:
            raise Exception("Endirmə uğursuz oldu")
        
        time.sleep(5)
    
    # Endirilmiş məlumatları oxuyun
    invoices = read_downloaded_invoices(process_id)
    print(f"{len(invoices)} qaimə endirildi")
    
    return invoices

def check_download_status(process_id):
    response = requests.post(
        f'{API_BASE}/job/read_proc_status',
        headers={'x-vatpapikey': API_TOKEN},
        json={'procId': process_id}
    )
    return response.json()['data']

def read_downloaded_invoices(process_id):
    response = requests.post(
        f'{API_BASE}/etx/read_invoices',
        headers={'x-vatpapikey': API_TOKEN},
        json={
            'phone': PHONE,
            'procId': process_id
        }
    )
    
    data = response.json()
    return data['data']['invoices']

# İstifadə
if __name__ == '__main__':
    try:
        invoices = download_approved_invoices()
        
        # Qaimələri emal edin
        for inv in invoices:
            print(f"Qaime: {inv['qaime']}")
            print(f"Cəm: {inv['net_amount']}")
            print(f"Məhsullar: {len(inv['lines'])}")
            print("---")
            
    except Exception as e:
        print(f"Xəta: {e}")
```

## Növbəti Addımlar

- [Qaimələri İdxal və Yükləmək →](./import-upload-invoices)
- [Xəta kodları arayışı →](./error-codes)
- [Autentifikasiya bələdçisi →](./authentication)

---

[← Sənədləşməyə qayıt](../README.md)
