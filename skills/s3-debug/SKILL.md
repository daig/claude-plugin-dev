---
name: s3-debug
description: Debug S3/MinIO presigned URL issues, signature mismatches, and 403 errors
triggers:
  - presigned url
  - S3 403
  - MinIO 403
  - SignatureDoesNotMatch
  - presigned upload failed
---

# S3/MinIO Presigned URL Debugging Guide

This skill helps diagnose and fix presigned URL issues, particularly signature mismatches that manifest as 403 errors.

## Quick Diagnosis

**Always test with curl first** to see the actual error response (browsers hide the XML error body):

```bash
# Test a presigned PUT URL
curl -v -X PUT "PRESIGNED_URL_HERE" \
  -H "Content-Type: image/png" \
  --data-binary @test-file.png

# Or with minimal test data
curl -v -X PUT "PRESIGNED_URL_HERE" \
  -H "Content-Type: image/png" \
  -d "test"
```

Look for the `<Code>` in the XML response - `SignatureDoesNotMatch` is the most common issue.

## Understanding Presigned URL Signatures

A presigned URL includes these critical query parameters:

| Parameter | Purpose |
|-----------|---------|
| `X-Amz-Algorithm` | Signing algorithm (usually AWS4-HMAC-SHA256) |
| `X-Amz-Credential` | Access key + date + region + service |
| `X-Amz-Date` | Timestamp when URL was signed |
| `X-Amz-Expires` | URL validity duration in seconds |
| `X-Amz-SignedHeaders` | **Headers included in signature** |
| `X-Amz-Signature` | The actual signature |

**Critical**: `X-Amz-SignedHeaders` lists every header that was included in the signature calculation. When making a request, these headers must match **exactly** what was used during signing.

Example: `X-Amz-SignedHeaders=content-length;content-type;host`

This means the signature was computed using:
- The `Content-Length` header value
- The `Content-Type` header value
- The `Host` header value

If any of these differ in the actual request, you get `SignatureDoesNotMatch`.

## Common SignatureDoesNotMatch Causes

### 1. Host Header Mismatch (Docker/Localhost)

**The #1 cause**: URL generated with internal Docker hostname, but accessed from browser with localhost.

```python
# WRONG - Signs with internal hostname, breaks when accessed externally
s3_client = boto3.client('s3', endpoint_url='http://minio:9000', ...)
url = s3_client.generate_presigned_url(...)
# URL contains host=minio:9000 in signature
# Browser sends Host: localhost:9000 -> SIGNATURE MISMATCH

# ALSO WRONG - String replacing breaks the signature
url = s3_client.generate_presigned_url(...)
browser_url = url.replace('http://minio:9000', 'http://localhost:9000')
# URL still signed with host=minio:9000 -> SIGNATURE MISMATCH
```

**Correct approach**: Sign with the endpoint the client will actually use:

```python
# CORRECT - Sign with browser-accessible endpoint from the start
s3_client = boto3.client(
    's3',
    endpoint_url='http://localhost:9000',  # What browser will access
    aws_access_key_id='minioadmin',
    aws_secret_access_key='minioadmin',
    region_name='us-east-1'
)
url = s3_client.generate_presigned_url(...)
# URL signed with host=localhost:9000, browser sends same -> WORKS
```

For Docker Compose, your backend needs network access to MinIO via `localhost:9000` (not `minio:9000`). Options:
- Run backend outside Docker with `network_mode: host`
- Use Docker's host.docker.internal
- Expose MinIO on a port accessible from both contexts

### 2. Content-Type Mismatch

If `content-type` is in `X-Amz-SignedHeaders`, the request must send the exact same Content-Type.

```python
# Signing with Content-Type
url = s3_client.generate_presigned_url(
    'put_object',
    Params={
        'Bucket': 'my-bucket',
        'Key': 'file.png',
        'ContentType': 'image/png'  # Included in signature
    },
    ExpiresIn=3600
)
```

```javascript
// Request MUST use exact same Content-Type
await fetch(presignedUrl, {
    method: 'PUT',
    headers: { 'Content-Type': 'image/png' },  // Must match exactly
    body: file
});
```

**Fix for flexible Content-Type**: Omit ContentType from Params entirely:

```python
url = s3_client.generate_presigned_url(
    'put_object',
    Params={
        'Bucket': 'my-bucket',
        'Key': 'file.png'
        # No ContentType - client can send any
    },
    ExpiresIn=3600
)
```

### 3. Content-Length Mismatch

If `content-length` is in `X-Amz-SignedHeaders` (rare but possible), the request body size must match exactly.

### 4. URL Modification After Signing

**NEVER modify a presigned URL after generation**. Any change invalidates the signature:

```python
# ALL OF THESE BREAK THE SIGNATURE:
url.replace('http://', 'https://')      # Protocol change
url.replace('minio:9000', 'localhost')  # Host change
url + '&extra=param'                    # Adding parameters
```

## CORS vs Signature Errors

| Symptom | Likely Cause | How to Verify |
|---------|--------------|---------------|
| 403 in browser, works in curl | CORS issue | Check browser Network tab for preflight |
| 403 everywhere + SignatureDoesNotMatch in XML | Header mismatch | Compare signed headers to request headers |
| No response/network error in browser | CORS blocking | Check Console for CORS error |
| `host` in SignedHeaders + different endpoint used | Docker/localhost mismatch | Check endpoint_url in client config |

**CORS debugging**:
```bash
# Check CORS headers
curl -v -X OPTIONS "http://localhost:9000/bucket/key" \
  -H "Origin: http://localhost:3000" \
  -H "Access-Control-Request-Method: PUT"
```

## Debug Workflow

```bash
# Step 1: Get the presigned URL and inspect X-Amz-SignedHeaders
echo "$PRESIGNED_URL" | grep -o 'X-Amz-SignedHeaders=[^&]*'

# Step 2: Test with curl, matching all signed headers
curl -v -X PUT "$PRESIGNED_URL" \
  -H "Content-Type: image/png" \
  -d "test"

# Step 3: Check the error response XML
# Look for <Code>SignatureDoesNotMatch</Code>

# Step 4: Compare StringToSign in error with what you expected
# The error often includes the canonical request that was computed
```

## Fix Patterns Summary

| Problem | Fix |
|---------|-----|
| Docker hostname in URL | Generate URL with browser-accessible endpoint |
| Content-Type mismatch | Match exactly OR omit from Params |
| URL string replacement | Never do this - regenerate with correct endpoint |
| CORS errors | Configure MinIO/S3 CORS policy |

## MinIO CORS Configuration

```bash
# Set CORS policy via mc client
mc admin config set myminio api cors_allow_origin="*"

# Or via environment variable in docker-compose
MINIO_API_CORS_ALLOW_ORIGIN: "*"
```

## Python boto3 Complete Example

```python
import boto3
from botocore.config import Config

# Create client with browser-accessible endpoint
s3_client = boto3.client(
    's3',
    endpoint_url='http://localhost:9000',  # Browser-accessible
    aws_access_key_id='minioadmin',
    aws_secret_access_key='minioadmin',
    region_name='us-east-1',
    config=Config(signature_version='s3v4')
)

# Generate presigned URL without Content-Type constraint
presigned_url = s3_client.generate_presigned_url(
    'put_object',
    Params={
        'Bucket': 'uploads',
        'Key': f'avatars/{user_id}.png'
        # Intentionally omit ContentType for flexibility
    },
    ExpiresIn=3600
)

# Client can now upload with any Content-Type
```

## JavaScript/Fetch Upload Example

```javascript
async function uploadToPresignedUrl(presignedUrl, file) {
    const response = await fetch(presignedUrl, {
        method: 'PUT',
        body: file,
        // Only include Content-Type if it was signed
        // headers: { 'Content-Type': file.type }
    });

    if (!response.ok) {
        const text = await response.text();
        console.error('Upload failed:', text);
        // Parse XML to see actual error code
    }

    return response.ok;
}
```
