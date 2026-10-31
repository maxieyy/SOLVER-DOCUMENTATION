# reCAPTCHA Solver API


## Base URL

```text
https://api.maxwell.deals
```

## Authentication

Include your API token in the `Authorization` header.

```http
Authorization: Bearer YOUR_API_TOKEN
```

## Endpoint

`POST /solve`

Submit a CAPTCHA solving request.

**Content-Type:** `application/json`

---

## Supported CAPTCHA Types

| Version | Description | Enterprise |
|---|---|---|
| `v2` | Standard checkbox CAPTCHA | Supported |
| `invisible` | Invisible reCAPTCHA v2 | Supported |
| `v3` | Score-based reCAPTCHA v3 | Supported |

---

## 1. reCAPTCHA v2 Checkbox

Solve a standard Google reCAPTCHA checkbox challenge.

**Request:**

```bash
curl -X POST https://api.maxwell.deals/solve \
  -H "Authorization: Bearer YOUR_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "type": "recaptcha",
    "version": "v2",
    "sitekey": "YOUR_SITE_KEY",
    "url": "https://example.com/login"
  }'
```

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `type` | string | Yes | `recaptcha` |
| `version` | string | Yes | `v2` |
| `sitekey` | string | Yes | CAPTCHA site key |
| `url` | string | Yes | Page URL containing CAPTCHA |
| `enterprise` | boolean | No | Enable Enterprise mode |
| `timeout_s` | integer | No | Maximum solving time |

---

## 2. Invisible reCAPTCHA v2

Solve invisible reCAPTCHA challenges.

**Request:**

```bash
curl -X POST https://api.maxwell.deals/solve \
  -H "Authorization: Bearer YOUR_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "type": "recaptcha",
    "version": "invisible",
    "sitekey": "YOUR_SITE_KEY",
    "url": "https://example.com/register"
  }'
```

For pages requiring actual page execution, you can enable:

```json
{
  "real_page": true
}
```

---

## 3. reCAPTCHA v3

Solve score-based reCAPTCHA v3 challenges.

**Request:**

```bash
curl -X POST https://api.maxwell.deals/solve \
  -H "Authorization: Bearer YOUR_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "type": "recaptcha",
    "version": "v3",
    "sitekey": "YOUR_SITE_KEY",
    "url": "https://example.com/login",
    "action": "login"
  }'
```

**Additional parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `action` | string | No | Expected reCAPTCHA action |
| `enterprise` | boolean | No | Enable Enterprise mode |
| `real_page` | boolean | No | Enable real-page execution |

---

## 4. reCAPTCHA Enterprise

Enterprise is supported for both score-based and checkbox challenges.

Enable Enterprise using:

```json
"enterprise": true
```

**Enterprise v3 example:**

```json
{
  "type": "recaptcha",
  "version": "v3",
  "enterprise": true,
  "sitekey": "YOUR_SITE_KEY",
  "url": "https://example.com",
  "action": "login"
}
```

**Enterprise v2 example:**

```json
{
  "type": "recaptcha",
  "version": "v2",
  "enterprise": true,
  "sitekey": "YOUR_SITE_KEY",
  "url": "https://example.com/form"
}
```

---

## 5. Advanced Options

The following optional parameters are available when supported by the selected solving method.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `real_page` | boolean | `false` | Enable real-page execution |
| `timeout_s` | integer | — | Maximum solving time |
| `classifier` | string | `auto` | Image classification preference |
| `pre_actions` | array | — | Actions to perform before solving |
| `post_fetch` | array | — | Requests to perform after solving |

### Classifier Options

| Value | Description |
|---|---|
| `auto` | Automatic selection |
| `hybrid` | Hybrid classification |
| `yolo` | Local classification |
| `mistral` | Vision-based classification |

For normal API usage, leave `classifier` unset.

---

## 6. API Response

All HTTP `200` responses include a `solved` boolean indicating whether the CAPTCHA was successfully solved.

### Successful Response

```json
{
  "solved": true,
  "token": "CAPTCHA_RESPONSE_TOKEN",
  "elapsed": 4.2
}
```

### v3 Response

```json
{
  "solved": true,
  "token": "CAPTCHA_RESPONSE_TOKEN",
  "score": 0.9,
  "elapsed": 4.2
}
```

### Failed Solve

```json
{
  "solved": false,
  "error": "Unable to solve CAPTCHA"
}
```

**Response fields:**

| Field | Type | Description |
|---|---|---|
| `solved` | boolean | Whether solving succeeded |
| `token` | string | Generated CAPTCHA token |
| `score` | number | Score, when available |
| `elapsed` | number | Time taken in seconds |
| `expires_in` | number | Token validity period, when available |
| `verify_success` | boolean | Verification result, when available |
| `method` | string | Solving method, when available |
| `error` | string | Error description when solving fails |

---

## 7. Error Handling

The API uses standard HTTP status codes.

| HTTP Status | Description |
|---|---|
| `200` | Request processed; check `solved` |
| `400` | Invalid request or unsupported CAPTCHA configuration |
| `408` | Request timed out |
| `422` | Invalid request body |
| `500` | Internal server error |

### Error Response

```json
{
  "detail": "Invalid sitekey or missing required parameter"
}
```

**Important:** A `200` response does not necessarily mean the CAPTCHA was solved. Always check `solved`.

- HTTP 2xx: Read `solved`.
- Non-2xx: Read `detail`.

---

## 8. Integration Example (Python)

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
        "type": "recaptcha",
        "version": "v3",
        "sitekey": "YOUR_SITE_KEY",
        "url": "https://example.com/login",
        "action": "login"
    },
    timeout=120
)

if response.ok:
    result = response.json()

    if result["solved"]:
        print("Solved:", result["token"])
    else:
        print("Solve failed:", result.get("error"))
else:
    print("API error:", response.json().get("detail"))
```

---

## Quick Reference

| CAPTCHA | Required Parameters |
|---|---|
| v2 Checkbox | `type`, `version`, `sitekey`, `url` |
| Invisible v2 | `type`, `version`, `sitekey`, `url` |
| v3 | `type`, `version`, `sitekey`, `url`, `action` |
| Enterprise v2 | Standard v2 parameters + `enterprise: true` |
| Enterprise v3 | Standard v3 parameters + `enterprise: true` |

## API Summary

| Property | Value |
|---|---|
| Method | `POST` |
| Endpoint | `/solve` |
| Content-Type | `application/json` |
| Authentication | Bearer Token |
| Response | JSON |
| Protocol | HTTPS |

**Note:** CAPTCHA tokens are subject to Google's expiration, site-key, action, and server-side validation requirements. Token generation does not guarantee acceptance by the target website.
