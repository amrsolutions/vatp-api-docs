# Real-World Examples

Practical examples for common VatPortal API use cases.

## Example 1: Simple Product Sale (18% VAT)

**Scenario:** Sell one product for 100 AZN + 18 AZN VAT = 118 AZN total

```bash
curl -X POST https://company.vatportal.az/api/inv/import_upload_invoices.php \
  -H "x-vatpapikey: YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "upload": true,
    "invoices": [{
      "id": "SALE-2025-001",
      "main_info": "Contract №12345, dated 15.01.2025",
      "seller": "1234567890",
      "buyer": "0987654321",
      "sum_subtotal1": 100,
      "sum_subtotal2": 100,
      "sum_apply_vat_amount": 100,
      "sum_vat_amount": 18,
      "sum_total": 118,
      "items": [{
        "id": "PROD-123",
        "name": "Laptop Dell Inspiron 15",
        "unit": "pcs",
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

**Response:**
```json
{
  "err_code": 0,
  "err_msg": "",
  "data": {
    "id": 5001
  }
}
```

---

## Example 2: Multi-Item Invoice with Transport

**Scenario:** Sale of multiple products with transport tax

```json
{
  "username": "your_username",
  "password": "your_password",
  "upload": true,
  "invoices": [{
    "id": "INV-2025-042",
    "main_info": "Contract A001; 08.01.2024",
    "add_info": "Delivery note XYZ 251145; 28.02.2024",
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
        "name": "Liquefied gas (technical butane)",
        "unit": "ton",
        "unit_price": 0.400,
        "quantity": 816.61020,
        "barcode": "9947008540000",
        "subtotal1": 326.6441,
        "aksiz": 0,
        "aksizgrade": 0,
        "subtotal2": 326.6441,
        "apply_vat_amount": 326.6441,
        "vat_amount": 58.7959,
        "transport_tax": 14.56,
        "total": 400.0000
      },
      {
        "id": "9947008540",
        "name": "Transport service",
        "unit": "service",
        "unit_price": 0.400,
        "quantity": 254.24,
        "barcode": "9947008540000",
        "subtotal1": 101.6960,
        "aksiz": 0,
        "aksizgrade": 0,
        "subtotal2": 101.6960,
        "apply_vat_amount": 101.6960,
        "vat_amount": 18.3053,
        "transport_tax": 0,
        "total": 120.0013
      }
    ]
  }]
}
```

---

## Example 3: Simplified Taxation (No VAT)

**Scenario:** Small business without VAT registration

```json
{
  "username": "your_username",
  "password": "your_password",
  "upload": true,
  "invoices": [{
    "id": "SIMPLE-2025-001",
    "main_info": "Contract A001; 08.01.2024",
    "add_info": "Payment reference: 251145",
    "seller": "1111111111",
    "buyer": "0000000000",
    "sum_subtotal1": 91.72,
    "sum_subtotal2": 91.72,
    "sum_total": 91.72,
    "items": [{
      "id": "9965111040",
      "name": "Consulting services under simplified taxation",
      "unit": "service",
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

**Key differences:**
- No VAT-related fields required
- Simpler structure
- All VAT amounts are 0 or omitted

---

## Example 4: Advance Payment Invoice

**Scenario:** Customer pays 50% upfront

**Step 1: Create advance invoice**
```json
{
  "username": "your_username",
  "password": "your_password",
  "upload": true,
  "invoices": [{
    "id": "AVANS-2025-001",
    "type": "avans",
    "main_info": "50% advance payment for order #12345",
    "seller": "1111111111",
    "buyer": "0000000000",
    "sum_subtotal1": 500,
    "sum_subtotal2": 500,
    "sum_apply_vat_amount": 500,
    "sum_vat_amount": 90,
    "sum_total": 590,
    "items": [{
      "id": "0000000001",
      "name": "Advance for office furniture",
      "unit": "service",
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

**Result:** Qaime number `AA25010000123`

**Step 2: Create final invoice based on advance**
```json
{
  "username": "your_username",
  "password": "your_password",
  "upload": true,
  "invoices": [{
    "id": "FINAL-2025-001",
    "main_qaime": "AA25010000123",
    "main_info": "Final delivery of office furniture",
    "seller": "1111111111",
    "buyer": "0000000000",
    "sum_subtotal1": 1000,
    "sum_subtotal2": 1000,
    "sum_apply_vat_amount": 1000,
    "sum_vat_amount": 180,
    "sum_total": 1180,
    "items": [{
      "id": "FURNITURE-001",
      "name": "Complete office furniture set",
      "unit": "set",
      "unit_price": 1000,
      "quantity": 1,
      "barcode": "FURNITURE-001000",
      "subtotal1": 1000,
      "subtotal2": 1000,
      "apply_vat_amount": 1000,
      "vat_amount": 180,
      "total": 1180
    }]
  }]
}
```

---

## Example 5: Goods Transfer Between Warehouses

**Scenario:** Move inventory from main warehouse to branch

```json
{
  "username": "your_username",
  "password": "your_password",
  "upload": true,
  "invoices": [{
    "id": "TRANSFER-2025-001",
    "type": "goodsTransfer",
    "main_info": "Transfer to Sumgait branch",
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
      "name": "Building materials - cement 50kg",
      "unit": "bag",
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

---

## Example 6: Complete Workflow with Status Checking

```javascript
// Step 1: Import and upload invoice
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
          main_info: 'Product sale',
          seller: '1234567890',
          buyer: '0987654321',
          sum_subtotal1: 100,
          sum_subtotal2: 100,
          sum_apply_vat_amount: 100,
          sum_vat_amount: 18,
          sum_total: 118,
          items: [{
            id: 'PROD-001',
            name: 'Product',
            unit: 'pcs',
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
    throw new Error(`Upload failed: ${data.err_code}`);
  }
  
  return data.data.id; // Process ID
}

// Step 2: Poll for completion
async function waitForCompletion(processId) {
  const maxAttempts = 60; // 5 minutes max
  const pollInterval = 5000; // 5 seconds
  
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
    
    // Still processing
    await sleep(pollInterval);
  }
  
  throw new Error('Timeout waiting for completion');
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

// Step 3: Get qaime number
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

// Complete workflow
async function completeInvoiceFlow() {
  try {
    // Upload
    console.log('Uploading invoice...');
    const processId = await uploadInvoice();
    console.log(`Process ID: ${processId}`);
    
    // Wait for completion
    console.log('Waiting for completion...');
    const result = await waitForCompletion(processId);
    
    if (!result.success) {
      console.error('Upload failed:', result.error);
      return;
    }
    
    console.log('Invoice uploaded successfully!');
    
    // Get qaime number
    const qaime = await getQaimeNumber('SALE-2025-100');
    console.log(`Qaime number: ${qaime}`);
    
    // Store qaime in your database
    await updateInvoiceInDatabase('SALE-2025-100', { qaime });
    
  } catch (error) {
    console.error('Error:', error);
  }
}

function sleep(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}
```

---

## Example 7: Download and Process Invoices (v8.0)

```python
import requests
import time
from datetime import datetime, timedelta

API_BASE = 'https://company.vatportal.az/api'
API_TOKEN = 'your_token_here'
PHONE = '994501234567'

def download_approved_invoices():
    """Download only approved invoices from last month"""
    
    # Calculate date range
    today = datetime.now()
    last_month = today - timedelta(days=30)
    
    # Start download
    response = requests.post(
        f'{API_BASE}/etx/import',
        headers={'x-vatpapikey': API_TOKEN},
        json={
            'phone': PHONE,
            'date_from': last_month.strftime('%d.%m.%Y'),
            'date_to': today.strftime('%d.%m.%Y'),
            'dir': 1,  # Sent folder
            'inc_status': 'approved',  # Only approved
            'exc_kinds': 'returnInvoice'  # Exclude returns
        }
    )
    
    data = response.json()
    
    if data['err_code'] != 0:
        raise Exception(f"Download failed: {data['err_code']}")
    
    process_id = data['data']['id']
    print(f"Download started. Process ID: {process_id}")
    
    # Wait for completion
    while True:
        status = check_download_status(process_id)
        
        if status['status'] == 2:
            print("Download complete!")
            break
        elif status['status'] == 3:
            raise Exception("Download failed")
        
        time.sleep(5)
    
    # Read downloaded data
    invoices = read_downloaded_invoices(process_id)
    print(f"Downloaded {len(invoices)} invoices")
    
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

# Usage
if __name__ == '__main__':
    try:
        invoices = download_approved_invoices()
        
        # Process invoices
        for inv in invoices:
            print(f"Qaime: {inv['qaime']}")
            print(f"Total: {inv['net_amount']}")
            print(f"Items: {len(inv['lines'])}")
            print("---")
            
    except Exception as e:
        print(f"Error: {e}")
```

---

## Error Handling Best Practices

```javascript
async function safeApiCall(endpoint, payload) {
  try {
    const response = await fetch(endpoint, {
      method: 'POST',
      headers: {
        'x-vatpapikey': process.env.VATPORTAL_TOKEN,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(payload)
    });
    
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}`);
    }
    
    const data = await response.json();
    
    // Check API error code
    if (data.err_code !== 0) {
      if (data.err_code === 14) {
        // Validation errors
        const errors = data.data.invoices[0];
        throw new Error(`Validation failed: ${errors.join(', ')}`);
      }
      
      throw new Error(`API error ${data.err_code}`);
    }
    
    return data;
    
  } catch (error) {
    console.error('API call failed:', error);
    
    // Log for debugging
    console.error('Endpoint:', endpoint);
    console.error('Payload:', JSON.stringify(payload, null, 2));
    
    throw error;
  }
}
```

---

## Next Steps

- [Import & Upload Invoices →](./import-upload-invoices)
- [Error codes reference →](./error-codes)
- [Authentication guide →](./authentication)

---

[← Back to Documentation](../README.md)
