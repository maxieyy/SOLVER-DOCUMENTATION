# AWS WAF Token API

Retrieve AWS WAF token status through the AWS WAF integration endpoint.

## Base URL

```text
https://api.maxwell.deals
```

## Endpoint

```http
POST /solve
```

**Full endpoint:** `https://api.maxwell.deals/solve`

**Content-Type:** `application/json`

## Authentication

The supplied implementation notes do not specify an authentication requirement. If API authentication is enabled, include your configured authorization header.

## Request

```json
{
  "type": "awswaf",
  "url": "https://example.com",
  "timeout_s": 60
}
```

## Request Parameters

| Field | Required | Default | Description |
|---|---|---|---|
| `type` | Yes | — | Integration type. Must be `awswaf`. |
| `url` | Yes | — | Target page URL. |
| `timeout_s` | No | `60` | Maximum processing time in seconds. |
| `pre_actions` | No | — | Optional actions executed before processing. |
| `post_fetch` | No | — | Optional actions executed after processing. |

## cURL Example

```bash
curl -X POST "https://api.maxwell.deals/solve" \
  -H "Content-Type: application/json" \
  -d '{
    "type": "awswaf",
    "url": "https://example.com",
    "timeout_s": 60
  }'
```

## Response

The API uses a consistent `solved` boolean to indicate the outcome.

**Successful response**

```json
{
  "solved": true,
  "success": true,
  "method": "silent-challenge",
  "elapsed": 3.9
}
```

**Unsuccessful response**

```json
{
  "solved": false,
  "error": "Unable to complete the request"
}
```

## Response Fields

| Field | Type | Description |
|---|---|---|
| `solved` | Boolean | Indicates whether processing succeeded. |
| `success` | Boolean | Additional success indicator, when returned. |
| `method` | String | Processing method reported by the API. |
| `elapsed` | Number | Processing time in seconds. |
| `error` | String | Explains why processing was unsuccessful. |

## Error Handling

HTTP `200` means the request was processed. Always inspect `solved` to determine the outcome.

Non-2xx responses indicate that the request could not be completed.

| HTTP Status | Description |
|---|---|
| `400` | Invalid request, missing required parameters, unsupported type, or blocked URL. |
| `408` | Request timed out. |
| `500` | Internal processing error. |

Example error response:

```json
{
  "detail": "Request timed out"
}
```

**Important:** A successful HTTP response does not necessarily mean the operation succeeded. Always check `solved`.

## Limitations

- Supports silent AWS WAF JavaScript challenges only.
- Interactive visual challenges are not supported.
- Some protected pages may reject requests based on their security policies.
- Processing time depends on the target page and challenge requirements.

## Python Example

```python
import requests

API_URL = "https://api.maxwell.deals/solve"

payload = {
    "type": "awswaf",
    "url": "https://example.com",
    "timeout_s": 60
}

try:
    response = requests.post(
        API_URL,
        json=payload,
        timeout=70
    )

    response.raise_for_status()
    result = response.json()

    if result.get("solved"):
        print("Request processed successfully")
    else:
        print("Processing unsuccessful:", result.get("error"))

except requests.exceptions.HTTPError as exc:
    print("API error:", exc.response.text)

except requests.exceptions.Timeout:
    print("Client request timed out")

except requests.exceptions.RequestException as exc:
    print("Connection error:", str(exc))

