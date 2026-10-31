# Cloudflare Turnstile Solver API



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

## Solving Modes

The API supports three solving modes depending on your integration requirements.

### 1. Standard Solve

The default solving mode. Suitable for websites where Turnstile is rendered directly when the page loads.

**Request:**

```bash
curl -X POST https://api.maxwell.deals/solve \
  -H "Authorization: Bearer YOUR_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "type": "turnstile",
    "sitekey": "0x4AAAAAA...",
    "url": "https://example.com/login"
  }'
```

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `type` | string | Yes | Must be `turnstile` |
| `sitekey` | string | Yes | Cloudflare Turnstile sitekey |
| `url` | string | Yes | Page URL containing Turnstile |
| `timeout_s` | integer | No | Maximum solving time |

**Note:** Some websites may reject tokens generated through the standard mode. For supported integrations requiring the actual page context, use `real_page: true`.

### 2. Solve and Verify

Solves Turnstile and submits the verification request using the same session.

Useful when your integration requires the token to be submitted immediately to a verification endpoint.

**Request:**

```bash
curl -X POST https://api.maxwell.deals/solve \
  -H "Authorization: Bearer YOUR_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "type": "turnstile",
    "sitekey": "0x4AAAAAA...",
    "verify_url": "https://example.com/api/verify-captcha",
    "verify_payload": {
      "token": "__TOKEN__"
    },
    "page_url": "https://example.com/login"
  }'
```

**Additional Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `verify_url` | string | Yes | Verification API endpoint |
| `verify_payload` | object | Yes | JSON payload sent for verification |
| `page_url` | string | No | Page URL; defaults to `verify_url` |

**Token Injection**

Use the `__TOKEN__` placeholder wherever the generated token should be inserted.

Example:

```json
{
  "verify_payload": {
    "token": "__TOKEN__"
  }
}
```

The API automatically replaces the placeholder with the generated Turnstile token.

### 3. Real-Page Solve

Use real-page mode when:

- Turnstile appears after user interaction.
- The website requires additional actions before displaying Turnstile.
- Standard solving mode is not accepted.
- You need to execute additional API requests after solving.

**Request:**

```bash
curl -X POST https://api.maxwell.deals/solve \
  -H "Authorization: Bearer YOUR_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "type": "turnstile",
    "url": "https://example.com/login",
    "timeout_s": 60,
    "pre_actions": [
      {
        "type": "click",
        "selector": "text=Continue with Email"
      },
      {
        "type": "wait",
        "value": "2"
      },
      {
        "type": "fill",
        "selector": "input[type=email]",
        "value": "user@example.com"
      },
      {
        "type": "click",
        "selector": "text=Continue"
      }
    ],
    "post_fetch": [
      {
        "url": "https://example.com/api/verify-captcha",
        "method": "POST",
        "body": {
          "token": "__TOKEN__"
        }
      }
    ]
  }'
```

---

## Pre-Actions

Pre-actions allow you to interact with the target page before solving.

### Supported Action Types

| Type | Required Fields | Description |
|---|---|---|
| `click` | `selector` | Click an element |
| `fill` | `selector`, `value` | Fill an input field |
| `select` | `selector`, `value` | Select a dropdown option |
| `wait` | `value` | Wait for a specified number of seconds |
| `press` | `value` | Press a keyboard key |

### Action Parameters

| Parameter | Type | Required | Description |
|---|---|---|---|
| `type` | string | Yes | Action type |
| `selector` | string | Depends | Target element selector |
| `value` | string | Depends | Input value, option, key, or wait duration |
| `timeout` | integer | No | Element wait timeout in milliseconds; defaults to `10000` |

### Supported Selector Formats

| Format | Example | Description |
|---|---|---|
| CSS | `input[type=email]` | CSS selector |
| XPath | `//button[@type='submit']` | XPath selector |
| Text | `text=Continue` | Text-based selector |
| Regex | `regex=Continue.*Email` | Regular expression selector |
| Role | `role=button[name='Submit']` | Accessible role selector |

---

## Post-Fetch

Post-fetch allows you to execute additional API requests after solving, using the same session.

This is useful when your application requires token verification or subsequent API calls.

### Parameters

| Parameter | Type | Required | Description |
|---|---|---|---|
| `url` | string | Yes | API endpoint URL |
| `method` | string | No | HTTP method; defaults to `POST` |
| `body` | object | No | JSON request body |

### Example

```json
{
  "post_fetch": [
    {
      "url": "https://example.com/api/verify",
      "method": "POST",
      "body": {
        "token": "__TOKEN__"
      }
    },
    {
      "url": "https://example.com/api/continue",
      "method": "POST",
      "body": {
        "status": "verified"
      }
    }
  ]
}
```

The `__TOKEN__` placeholder is automatically replaced with the generated token.

---

## API Response

All solving modes return a consistent JSON response.

### Successful Response

```json
{
  "solved": true,
  "token": "1.0eWQ2WW8abXvc...",
  "expires_in": 300,
  "method": "route",
  "elapsed": 22.8,
  "verify_success": true
}
```

### Response Parameters

| Field | Type | Description |
|---|---|---|
| `solved` | boolean | Whether solving succeeded |
| `token` | string | Generated Turnstile token |
| `expires_in` | integer | Token validity period, when available |
| `method` | string | Solving method, when available |
| `elapsed` | number | Time taken in seconds |
| `verify_success` | boolean | Verification result, when applicable |
| `error` | string | Error description when solving fails |

### Failed Solve

```json
{
  "solved": false,
  "error": "Unable to generate Turnstile token"
}
```

### Post-Fetch Response

When using real-page mode with `post_fetch`, the response may also include:

```json
{
  "solved": true,
  "token": "1.0eWQ2WW8abXvc...",
  "post_fetch": [
    {
      "url": "https://example.com/api/verify",
      "status": 200,
      "body": "..."
    }
  ]
}
```

---

## Error Handling

The API uses a consistent error contract.

| HTTP Status | Description |
|---|---|
| `200` | Request processed; check `solved` |
| `400` | Invalid request, unsupported type, missing parameters, or blocked URL |
| `408` | Request timed out |
| `422` | Invalid request body |
| `500` | Internal server error |

### Error Response

```json
{
  "detail": "Missing required parameter: sitekey"
}
```

### Important

- HTTP 2xx: Check the `solved` field.
- Non-2xx: Check the `detail` field.
- A `200` response does not necessarily mean solving succeeded.

---

## Python Integration

Install the required dependency:

```bash
pip install requests
```

### Standard Solve

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
        "type": "turnstile",
        "sitekey": "0x4AAAAAA...",
        "url": "https://example.com/login"
    },
    timeout=120
)

if response.ok:
    result = response.json()

    if result["solved"]:
        print("Token:", result["token"])
    else:
        print("Solve failed:", result.get("error"))
else:
    print("API error:", response.json().get("detail"))
```

### Solve and Verify

```python
import requests

response = requests.post(
    "https://api.maxwell.deals/solve",
    headers={
        "Authorization": "Bearer YOUR_API_TOKEN"
    },
    json={
        "type": "turnstile",
        "sitekey": "0x4AAAAAA...",
        "verify_url": "https://example.com/api/verify",
        "verify_payload": {
            "token": "__TOKEN__"
        },
        "page_url": "https://example.com/login"
    },
    timeout=120
)

print(response.json())
```

---

## Quick Reference

| Mode | Required Parameters | Use Case |
|---|---|---|
| Standard | `type`, `sitekey`, `url` | Standard Turnstile integration |
| Solve + Verify | `type`, `sitekey`, `verify_url`, `verify_payload` | Immediate verification |
| Real Page | `type`, `url` | Pages requiring additional interactions |

## API Summary

| Property | Value |
|---|---|
| Method | `POST` |
| Endpoint | `/solve` |
| Authentication | Bearer Token |
| Content-Type | `application/json` |
| Response Format | JSON |
| Supported CAPTCHA | Cloudflare Turnstile |

**Important:** Turnstile tokens are subject to Cloudflare's expiration and validation requirements. A generated token must still be validated by the website's server-side verification process. Use this API only for authorized integrations and testing.
