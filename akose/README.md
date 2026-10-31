# Arkose FunCaptcha Integration

Integrate Arkose Labs FunCaptcha verification into your application using the API.

## API Configuration

**Base URL**
```text
https://api.maxwell.deals
```

**Authentication**

Include your API token in the `Authorization` header:

```http
Authorization: Bearer YOUR_API_TOKEN
```

## Endpoint

```http
POST /solve
```

## Request

```json
{
  "type": "arkose",
  "public_key": "YOUR_ARKOSE_PUBLIC_KEY",
  "page_url": "https://example.com/login",
  "surl": "https://client-api.arkoselabs.com",
  "timeout_s": 120
}
```

## Request Parameters

| Field | Required | Description |
|---|---|---|
| `type` | Yes | Integration type: `arkose` |
| `public_key` | Yes | Arkose public key configured for your application |
| `page_url` | Yes | URL of the page containing the Arkose integration |
| `surl` | No | Arkose service URL, when specified by your integration |
| `timeout_s` | No | Request timeout in seconds |

## Response

The response should reflect the result of the authorized verification flow.

**Successful verification**
```json
{
  "type": "arkose",
  "solved": true,
  "verification_status": "verified",
  "elapsed": 2.4
}
```

**Unsuccessful verification**
```json
{
  "type": "arkose",
  "solved": false,
  "verification_status": "failed",
  "error": "Verification could not be completed"
}
```

## Error Handling

| HTTP Status | Meaning |
|---|---|
| `200` | Request processed; inspect `solved` |
| `400` | Invalid request parameters |
| `401` | Missing or invalid API token |
| `403` | Access denied |
| `408` | Request timed out |
| `422` | Invalid or unsupported integration configuration |
| `429` | Rate limit exceeded |
| `500` | Internal server error |

## cURL Example

```bash
curl -X POST "https://api.maxwell.deals/solve" \
  -H "Authorization: Bearer YOUR_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "type": "arkose",
    "public_key": "YOUR_ARKOSE_PUBLIC_KEY",
    "page_url": "https://example.com/login",
    "timeout_s": 120
  }'
```

## Python Example

```python
import requests

API_URL = "https://api.maxwell.deals/solve"
API_TOKEN = "YOUR_API_TOKEN"

payload = {
    "type": "arkose",
    "public_key": "YOUR_ARKOSE_PUBLIC_KEY",
    "page_url": "https://example.com/login",
    "timeout_s": 120
}

response = requests.post(
    API_URL,
    headers={
        "Authorization": f"Bearer {API_TOKEN}",
        "Content-Type": "application/json"
    },
    json=payload,
    timeout=130
)

response.raise_for_status()
result = response.json()

if result.get("solved"):
    print("Verification successful")
else:
    print("Verification failed:", result.get("error"))
```

