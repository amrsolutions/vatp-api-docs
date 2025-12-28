# Polling Best Practices

**Available since:** v1.0 (June 1, 2021)

## Overview

Many VatPortal API operations are asynchronous, meaning they return a process ID immediately and continue working in the background. To check the status of these operations, you need to poll the [`GET /job/read_proc_status`](./import-upload-invoices.md#monitor-process) endpoint.

This guide provides best practices for implementing efficient and reliable polling.

---

## When to Use Polling

Poll the process status endpoint after initiating any of these operations:

- **Import & Upload Invoices** - [`POST /inv/import_upload_invoices.php`](./import-upload-invoices.md)
- **Download from E-taxes** - [`POST /etx/import`](./download-from-etaxes.md)
- **Delete E-taxes Invoices** - [`DELETE /etx/delete`](./delete-etaxes-invoices.md)

All these endpoints return a process ID in the response:

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

Use this `id` to poll for status updates.

---

## Recommended Polling Strategy

### Progressive Intervals (Recommended)

**Most VatPortal operations take 3-5 minutes to complete.** Using progressive intervals (starting fast, then slowing down) provides the best balance between responsiveness and server efficiency:

```javascript
async function pollProcessStatus(procId, maxAttempts = 120) {
  let interval = 2000; // Start at 2 seconds for responsiveness
  const maxInterval = 10000; // Cap at 10 seconds
  const startTime = Date.now();

  for (let attempt = 1; attempt <= maxAttempts; attempt++) {
    const response = await fetch('https://company.vatportal.az/api/job/read_proc_status', {
      method: 'GET',
      headers: {
        'Content-Type': 'application/json',
        'x-vatpapikey': 'YOUR_TOKEN'
      },
      body: JSON.stringify({ procId })
    });

    const data = await response.json();

    // Check if process is complete
    if (data.data.status === 2) {
      console.log('✅ Process completed successfully');
      return { success: true, data: data.data };
    }

    if (data.data.status === 3) {
      console.log('❌ Process completed with errors');
      return { success: false, data: data.data };
    }

    // Adjust interval based on elapsed time
    const elapsedTime = Date.now() - startTime;

    if (elapsedTime > 120000) {
      // After 2 minutes, slow to 10 seconds
      interval = 10000;
    } else if (elapsedTime > 60000) {
      // After 1 minute, slow to 5 seconds
      interval = 5000;
    }
    // First minute: keep at 2 seconds

    console.log(`⏳ Attempt ${attempt}/${maxAttempts} - Next poll in ${interval/1000}s...`);
    await new Promise(resolve => setTimeout(resolve, interval));
  }

  // Timeout reached
  throw new Error('Polling timeout: Process did not complete in time');
}
```

**Why progressive intervals?**
- **Fast response initially**: 2-second intervals catch quick operations
- **Reduced server load**: Slower intervals for long-running processes (3-5 min avg)
- **60-70% fewer requests** for operations over 2 minutes
- **Better user experience**: Quick feedback when possible, efficient when needed

### Alternative: Exponential Backoff

For scenarios where you want mathematically increasing intervals:

```javascript
async function pollWithExponentialBackoff(procId, maxAttempts = 60) {
  let interval = 1000; // Start with 1 second
  const maxInterval = 10000; // Cap at 10 seconds

  for (let attempt = 1; attempt <= maxAttempts; attempt++) {
    const response = await fetch('https://company.vatportal.az/api/job/read_proc_status', {
      method: 'GET',
      headers: {
        'Content-Type': 'application/json',
        'x-vatpapikey': 'YOUR_TOKEN'
      },
      body: JSON.stringify({ procId })
    });

    const data = await response.json();

    // Check terminal states
    if (data.data.status === 2) {
      return { success: true, data: data.data };
    }

    if (data.data.status === 3) {
      return { success: false, data: data.data };
    }

    // Wait with exponential backoff
    console.log(`⏳ Attempt ${attempt} - Waiting ${interval}ms before retry...`);
    await new Promise(resolve => setTimeout(resolve, interval));

    // Increase interval (exponential backoff: 1s → 1.5s → 2.25s → 3.37s → 5s → 7.5s → 10s)
    interval = Math.min(interval * 1.5, maxInterval);
  }

  throw new Error('Polling timeout');
}
```

### Simple Fixed Interval (Not Recommended for Production)

Only use fixed intervals for quick operations or testing:

```javascript
async function pollWithFixedInterval(procId, maxAttempts = 60) {
  const pollInterval = 2000; // 2 seconds (fixed)

  for (let attempt = 1; attempt <= maxAttempts; attempt++) {
    const response = await fetch('https://company.vatportal.az/api/job/read_proc_status', {
      method: 'GET',
      headers: {
        'Content-Type': 'application/json',
        'x-vatpapikey': 'YOUR_TOKEN'
      },
      body: JSON.stringify({ procId })
    });

    const data = await response.json();

    if (data.data.status === 2) {
      return { success: true, data: data.data };
    }

    if (data.data.status === 3) {
      return { success: false, data: data.data };
    }

    await new Promise(resolve => setTimeout(resolve, pollInterval));
  }

  throw new Error('Polling timeout');
}
```

**Warning:** Fixed intervals are inefficient for VatPortal's 3-5 minute operations and create unnecessary server load.

---

## Polling Intervals by Operation Type

Different operations have different typical completion times. **Most operations average 3-5 minutes**, so progressive intervals are recommended:

| Operation | Typical Duration | Initial Interval | Progressive Schedule |
|-----------|------------------|------------------|----------------------|
| **Import & Upload** (upload: true) | **3-5 minutes** | 2 seconds | 2s (0-1min) → 5s (1-2min) → 10s (2min+) |
| **Download from E-taxes** | 3-5 minutes | 2 seconds | 2s (0-1min) → 5s (1-2min) → 10s (2min+) |
| **Delete E-taxes Invoices** | 3-5 minutes | 2 seconds | 2s (0-1min) → 5s (1-2min) → 10s (2min+) |
| **Import Only** (upload: false) | 30-60 seconds | 2 seconds | 2s (fixed) - completes quickly |

**Note:** Actual duration depends on:
- Number of invoices being processed
- E-taxes system response time
- Network latency
- Whether PIN codes need to be entered
- Server load during peak hours

---

## Process Status Values

When polling, check the `status` field in the response:

```json
{
  "res": 1,
  "err_code": 0,
  "data": {
    "status": 2,
    "status_desc": "Process Finalized Successfully",
    "stats": {
      "imported": 5,
      "packet_created": 1,
      "login_done": true
    }
  }
}
```

### Status Codes

| Status | Description | Action |
|--------|-------------|--------|
| `1` | Process running | Continue polling |
| `2` | Process completed successfully | Stop polling - Success! |
| `3` | Process completed with errors | Stop polling - Check error details |

**Important:** Only `2` and `3` are terminal states. Keep polling for any other status.

---

## Handling Upload Operations with PIN Codes

When `upload: true`, the process requires PIN code confirmation:

### Monitoring for PIN1 (Login)

```javascript
async function monitorForPIN1(procId) {
  while (true) {
    const response = await checkProcessStatus(procId);

    if (response.data.stats.pin1_code) {
      console.log(`📱 PIN1 code received: ${response.data.stats.pin1_code}`);
      console.log('⚠️ User must confirm this code via ASAN login');

      // Notify user to check their phone
      notifyUser('Please check your ASAN phone and confirm PIN1 code');

      // Continue polling to detect when login is done
      while (!response.data.stats.login_done) {
        await sleep(2000);
        response = await checkProcessStatus(procId);
      }

      console.log('✅ Login confirmed');
      break;
    }

    await sleep(2000);
  }
}
```

### Monitoring for PIN2 (Signing)

```javascript
async function monitorForPIN2(procId) {
  while (true) {
    const response = await checkProcessStatus(procId);

    if (response.data.stats.pin2_code) {
      console.log(`📱 PIN2 code received: ${response.data.stats.pin2_code}`);
      console.log('⚠️ User must confirm this code to sign invoices');

      // Notify user to check their phone
      notifyUser('Please check your ASAN phone and confirm PIN2 code to sign');

      // Continue polling to detect when signing is done
      while (!response.data.stats.invoice_signed) {
        await sleep(2000);
        response = await checkProcessStatus(procId);
      }

      console.log('✅ Invoices signed');
      break;
    }

    await sleep(2000);
  }
}
```

---

## Complete Example: Full Workflow with Polling

```javascript
async function uploadInvoicesWithPolling(invoices) {
  try {
    // Step 1: Initiate upload
    console.log('📤 Starting invoice upload...');
    const uploadResponse = await fetch('https://company.vatportal.az/api/inv/import_upload_invoices.php', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'x-vatpapikey': 'YOUR_TOKEN'
      },
      body: JSON.stringify({
        upload: true,
        use_old_sys: false,
        invoices: invoices
      })
    });

    const uploadData = await uploadResponse.json();

    if (uploadData.err_code !== 0) {
      throw new Error(`Upload failed: ${uploadData.err_msg}`);
    }

    const procId = uploadData.data.id;
    console.log(`✅ Upload initiated. Process ID: ${procId}`);

    // Step 2: Monitor for import completion
    console.log('⏳ Monitoring import progress...');
    await waitForImport(procId);

    // Step 3: Monitor for PIN1
    console.log('⏳ Waiting for PIN1 code...');
    await monitorForPIN1(procId);

    // Step 4: Monitor for packet upload
    console.log('⏳ Uploading packets to e-taxes...');
    await waitForUpload(procId);

    // Step 5: Monitor for PIN2
    console.log('⏳ Waiting for PIN2 code...');
    await monitorForPIN2(procId);

    // Step 6: Wait for final completion
    console.log('⏳ Finalizing...');
    const finalStatus = await pollUntilComplete(procId);

    if (finalStatus.status === 2) {
      console.log('🎉 All invoices uploaded and signed successfully!');
      console.log(`📊 Stats: ${JSON.stringify(finalStatus.stats, null, 2)}`);
      return { success: true, stats: finalStatus.stats };
    } else {
      console.log('❌ Process completed with errors');
      console.log(`⚠️ Error: ${finalStatus.stats.etx_exception}`);
      return { success: false, error: finalStatus.stats.etx_exception };
    }

  } catch (error) {
    console.error('💥 Error:', error.message);
    throw error;
  }
}

// Helper function to poll until complete
async function pollUntilComplete(procId, maxAttempts = 60) {
  let interval = 2000; // 2 seconds

  for (let attempt = 1; attempt <= maxAttempts; attempt++) {
    const response = await checkProcessStatus(procId);

    if (response.data.status === 2 || response.data.status === 3) {
      return response.data;
    }

    console.log(`⏳ Attempt ${attempt}/${maxAttempts}...`);
    await sleep(interval);
  }

  throw new Error('Polling timeout');
}

function sleep(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}
```

---

## Best Practices

### ✅ DO

- **Use progressive or exponential backoff** - Essential for 3-5 minute operations to reduce server load
- **Start with 2-second intervals** - Provides good responsiveness initially
- **Slow down after 1-2 minutes** - Most operations are still running, reduce polling frequency
- **Set reasonable timeouts** - 5-10 minutes for most operations (120 max attempts with progressive intervals)
- **Monitor terminal states** (status 2 or 3) to stop polling
- **Handle errors gracefully** - check `etx_exception` field for e-taxes errors
- **Notify users** when PIN codes are required
- **Log polling attempts** for debugging and monitoring
- **Use token authentication** instead of username/password for better security

### ❌ DON'T

- **Don't use fixed intervals for long operations** - Creates unnecessary load for 3-5 minute processes
- **Don't poll faster than 1 second** - Minimum interval should be 1 second
- **Don't poll indefinitely** - Always set a maximum attempt limit
- **Don't ignore status codes** - Check both `status` and `err_code`
- **Don't skip error handling** - Process may complete with status 3 (errors)
- **Don't poll after completion** - Stop when status is 2 or 3
- **Don't use POST method** - `read_proc_status` is a GET endpoint
- **Don't poll all processes simultaneously** - Sequential processing reduces server load

---

## Error Handling During Polling

### Check for E-taxes Errors

```javascript
async function checkForErrors(procId) {
  const response = await checkProcessStatus(procId);

  // Check for e-taxes exceptions
  if (response.data.stats.etx_exception) {
    const errors = response.data.stats.etx_exception;

    if (errors.includes('NEED RETRY')) {
      console.log('⚠️ E-taxes system busy - operation will be retried automatically');
      // Continue polling - system will retry
    } else {
      console.log('❌ E-taxes error:', errors);
      // These are business logic errors (invalid VOEN, amounts don't match, etc.)
      // Stop polling and report error
      throw new Error(`E-taxes error: ${errors.join(', ')}`);
    }
  }
}
```

### Handling Network Errors

```javascript
async function pollWithRetry(procId) {
  const maxNetworkRetries = 3;
  let networkRetry = 0;

  while (networkRetry < maxNetworkRetries) {
    try {
      const response = await checkProcessStatus(procId);
      networkRetry = 0; // Reset on success

      // Process response...

    } catch (error) {
      networkRetry++;
      console.log(`⚠️ Network error (${networkRetry}/${maxNetworkRetries}): ${error.message}`);

      if (networkRetry >= maxNetworkRetries) {
        throw new Error('Network error: Failed to connect after retries');
      }

      // Wait before retry
      await sleep(2000 * networkRetry); // Increasing backoff
    }
  }
}
```

---

## Performance Optimization

### Batch Processing

When uploading multiple batches, don't poll all processes simultaneously:

```javascript
// ❌ BAD: Polling all at once
const processes = await Promise.all(batches.map(batch => uploadBatch(batch)));
await Promise.all(processes.map(proc => pollUntilComplete(proc.id)));

// ✅ GOOD: Sequential processing
for (const batch of batches) {
  const proc = await uploadBatch(batch);
  await pollUntilComplete(proc.id);
  console.log(`✅ Batch ${batch.id} complete`);
}
```

### Use Process Data for Detailed Info

For detailed progress information, use [`GET /job/read_proc_data`](./import-upload-invoices.md#read-process-data):

```javascript
async function getDetailedProgress(procId) {
  const response = await fetch('https://company.vatportal.az/api/job/read_proc_data', {
    method: 'GET',
    headers: {
      'Content-Type': 'application/json',
      'x-vatpapikey': 'YOUR_TOKEN'
    },
    body: JSON.stringify({ procId })
  });

  const data = await response.json();

  // Get detailed invoice processing info
  console.log('Invoices processed:', data.data.invoices);
  console.log('E-tax invoices:', data.data.etax_invoices);

  return data.data;
}
```

---

## Testing Your Polling Implementation

### Test with Import-Only First

Start by testing with `upload: false` to verify basic polling:

```javascript
// Simpler test case - no PIN codes required
const testResponse = await uploadInvoices({
  upload: false,  // No e-taxes interaction
  invoices: [testInvoice]
});

const result = await pollUntilComplete(testResponse.data.id);
console.log('Test result:', result);
```

### Mock Response for Development

```javascript
// Mock for testing polling logic
function mockProcessStatus(attempt) {
  if (attempt < 5) {
    return { data: { status: 1, stats: {} } }; // Running
  } else if (attempt < 10) {
    return { data: { status: 1, stats: { imported: 1 } } }; // Import done
  } else {
    return { data: { status: 2, stats: { imported: 1 } } }; // Complete
  }
}
```

---

## Monitoring and Logging

### Recommended Log Format

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

### Metrics to Track

- Total polling attempts
- Average time to completion by operation type
- Frequency of timeout errors
- Frequency of e-taxes errors
- Average time for PIN code confirmation

---

## Related Documentation

- [Import & Upload Invoices](./import-upload-invoices.md) - Main upload endpoint
- [Download from E-taxes](./download-from-etaxes.md) - Download endpoint
- [Delete E-taxes Invoices](./delete-etaxes-invoices.md) - Delete endpoint
- [Error Codes](./error-codes.md) - Complete error reference

---

[← Back to Documentation](./README.md)
