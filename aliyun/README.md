# Aliyun Captcha Solver API

Solve Aliyun Captcha 2.0 slide-puzzle challenges.

The API returns the generated CAPTCHA verification token, ready for submission to the target application's verification endpoint.

## Base URL

```text
https://api.maxwell.deals
```

## Authentication

Include your API token in the request header.

```http
Authorization: Bearer YOUR_API_TOKEN
```

## Endpoint

`POST /solve`

**Content-Type:** `application/json`

---

## 1. Solve Aliyun Captcha

Submit an Aliyun Captcha 2.0 challenge using its `scene_id` and `prefix`.

### Request

```bash
curl -X POST https://api.maxwell.deals/solve \
  -H "Authorization: Bearer YOUR_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "type": "aliyun",
    "scene_id": "1r7eif79x",
    "prefix": "13lbkb5",
    "region": "sgp",
    "timeout_s": 90
  }'
```

### Request Parameters

| Parameter | Type | Required | Default | Description |
|---|---|---|---|---|
| `type` | string | Yes | — | Must be `aliyun` |
| `scene_id` | string | Yes | — | Aliyun CAPTCHA Scene ID |
| `prefix` | string | Yes | — | CAPTCHA endpoint prefix |
| `region` | string | No | `sgp` | Aliyun service region |
| `proxy` | string | No | — | Optional proxy URL |
| `timeout_s` | integer | No | — | Maximum solving time |

### Supported Regions

| Value | Region |
|---|---|
| `sgp` | Singapore (default) |
| `cn` | China |
| `intl` | International |

---

## 2. API Response

### Successful Response

```json
{
  "type": "aliyun",
  "solved": true,
  "token": {
    "sceneId": "1r7eif79x",
    "certifyId": "nVk57gcoC0",
    "deviceToken": "SG_WEB#...",
    "data": "JRMnbQ9RGiMx..."
  },
  "verify_code": "T001",
  "method": "quadratic-slide",
  "attempts": 6,
  "elapsed": 8.4
}
```

### Response Parameters

| Field | Type | Description |
|---|---|---|
| `type` | string | CAPTCHA type |
| `solved` | boolean | Whether the challenge was successfully solved |
| `token` | object | Generated Aliyun verification credentials |
| `token.sceneId` | string | CAPTCHA Scene ID |
| `token.certifyId` | string | Certification identifier |
| `token.deviceToken` | string | Device token |
| `token.data` | string | Verification data |
| `verify_code` | string | Verification result code |
| `method` | string | Solving method |
| `attempts` | integer | Number of attempts |
| `elapsed` | number | Total solving time in seconds |

### Verification Codes

| Code | Description |
|---|---|
| `T001` | Verification successful |
| `F001` | Challenge rejected |
| `F002` | Slide verification incomplete |
| `F015` | Incorrect slide position |

---

## 3. Using the Generated Token

After successfully solving the challenge, submit the returned `token` object to the target application's Aliyun verification endpoint, typically `VerifyCaptchaV3`.

**Important:**

- Submit the token immediately after solving.
- The token is session-bound and intended for one-time use.
- `deviceToken` is time-bound.
- If a proxy was used during solving, perform the subsequent verification request from the same IP address.
- Token generation does not guarantee acceptance if the target session or verification context has changed.

The exact verification request format depends on the target application's integration.

---

## 4. Failed Requests

A challenge that cannot be solved returns:

```json
{
  "type": "aliyun",
  "solved": false,
  "last_verify_code": "F015",
  "error": "Unable to solve CAPTCHA"
}
```

Always check the `solved` field before using the returned token.

---

## 5. Error Handling

The API uses standard HTTP status codes.

| HTTP Status | Description |
|---|---|
| `200` | Request processed; check `solved` |
| `400` | Invalid request or missing parameters |
| `408` | Request timed out |
| `422` | Invalid request body |
| `500` | Internal server error |

### Error Response

```json
{
  "detail": "Missing required parameter: scene_id"
}
```

**Response handling:**

- HTTP 2xx: Read `solved`.
- Non-2xx: Read `detail`.
- Only use the token when `solved` is `true`.

---

## 6. Python Integration

Install the required dependency:

```bash
pip install requests
```

### Example

```python
import requests

API_URL = "https://api.maxwell.deals/solve"
API_TOKEN = "YOUR_API_TOKEN"

response = requests.post(
    API_URL,
    headers={
        "Authorization": f"Bearer {API_TOKEN}",
        "Content-Type": "application/json"
    },
    json={
        "type": "aliyun",
        "scene_id": "1r7eif79x",
        "prefix": "13lbkb5",
        "region": "sgp",
        "timeout_s": 90
    },
    timeout=120
)

if response.ok:
    result = response.json()

    if result.get("solved"):
        token = result["token"]
        print("CAPTCHA solved successfully")
        print("Token:", token)
    else:
        print("Solve failed:", result.get("error"))
else:
    print("API error:", response.json().get("detail"))
```

---

## API Summary

| Property | Value |
|---|---|
| Method | `POST` |
| Endpoint | `/solve` |
| CAPTCHA Type | Aliyun Captcha 2.0 |
| Authentication | Bearer Token |
| Content-Type | `application/json` |
| Response Format | JSON |
| Required Parameters | `scene_id`, `prefix` |
| Optional Proxy | Supported |

**Important:** Aliyun verification tokens are session-bound, time-sensitive, and single-use. Use them immediately and submit them through the corresponding target application's verification flow.

Use this API only for authorized integrations and testing.
