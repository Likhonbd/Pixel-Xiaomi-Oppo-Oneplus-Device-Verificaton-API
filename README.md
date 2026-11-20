# Pixel Verification API — Developer Integration Guide

> **Base URL**: `https://api.pixelverification.me`
>
> **Web Portal (API Keys & Credits)**: [`https://pixelverification.me`](https://pixelverification.me)
>
> **Telegram Bot**: [`@Pixel_verification_bot`](https://t.me/Pixel_verification_bot)
>
> **Owner & Support**: Telegram [`@Tanvir124`](https://t.me/Tanvir124) — for custom pricing, higher rate limits, manual key issuance, or new plan updates.

---

## Contents

1. [Overview & Available Products](#overview--available-products)
2. [Getting Your API Key & Credits (Web Portal)](#getting-your-api-key--credits-web-portal)
3. [Authentication](#authentication)
4. [Quick Start](#quick-start)
5. [Endpoint Reference](#endpoint-reference)
   - [`GET /api/health`](#get-apihealth)
   - [`GET /api/pricing`](#get-apipricing)
   - [`POST /api/order`](#post-apiorder)
   - [`GET /api/order/{order_id}`](#get-apiorderorder_id)
   - [`GET /api/order-by-email`](#get-apiorder-by-email)
   - [`GET /api/credits`](#get-apicredits)
6. [The 4 Available Products](#the-4-available-products)
7. [Normal vs Full Mode](#normal-vs-full-mode)
8. [Order Lifecycle & Polling](#order-lifecycle--polling)
9. [Progress Tracking](#progress-tracking)
   - [Queue Position (While Pending)](#queue-position-while-pending)
10. [Credits & Billing](#credits--billing)
11. [Error Codes & Troubleshooting](#error-codes--troubleshooting)
12. [Rate Limiting](#rate-limiting)
13. [Idempotency & Timeout Recovery](#idempotency--timeout-recovery)
14. [Client Examples (Python, Node.js, cURL)](#client-examples)
15. [Integration Checklist](#integration-checklist)
16. [FAQ](#faq)
17. [Support & Admin Contact](#support--admin-contact)

---

## Overview & Available Products

The API automates **Google One / Google AI Pro offer verification** for Gmail accounts. Depending on the product chosen, it extracts the subscription redemption link (Normal Mode) or fully activates the subscription on the account (Full Mode).

There are **4 primary products** available for bot integration:

| Product                        | Flow Type  | Validity | What You Get                               | Initial Credit Cost |
| ------------------------------ | ---------- | -------- | ------------------------------------------ | ------------------- |
| **1. Normal Mode (12 Months)** | `"normal"` | `12`     | 12-Month Google One redemption link        | **NOT AVAILABLE**   |
| **2. Normal Mode (3 Months)**  | `"normal"` | `3`      | 3-Month Google One redemption link         | **1 credits**       |
| **3. Full Mode (12 Months)**   | `"full"`   | `12`     | 12-Month link **+ activated subscription** | **NOT AVAILABLE**   |
| **4. Full Mode (3 Months)**    | `"full"`   | `3`      | 3-Month link **+ activated subscription**  | **1.5 credits**     |

> 📌 **Pricing & Custom Deals**: The initial standard rates are listed above. If you need updated pricing, custom volume rates, or if new validity plans are introduced, the Admin will notify you or you can contact the Admin on Telegram at **`@Tanvir124`**.

### How the API Works

Everything is **asynchronous**:
1. Your bot submits an order with the user's Gmail credentials, `flow_type`, and `validity`.
2. The API immediately responds with an `order_id` and deducts credits upfront.
3. Your bot polls `GET /api/order/{order_id}` until the status becomes `success` or `failed`.
4. **Automatic Refunds**: If an order fails for any reason (wrong credentials, ineligible account, timeout, etc.), the credits charged for that order are **automatically refunded** to your balance immediately.

```
Your Bot / Backend  ──POST /api/order──►  API Gateway  ──►  Worker Queue
       │                                     │
       └──GET /api/order/{id}────────────────┘   (Poll until success / failed)
```

---

## Getting Your API Key & Credits (Web Portal)

You can generate your API key, view your rates, and fund your API credit balance directly from the official web portal at **[`https://pixelverification.me`](https://pixelverification.me)**.

### Step-by-Step Guide

1. **Start the Telegram Bot First**
   Open [`@Pixel_verification_bot`](https://t.me/Pixel_verification_bot) on Telegram and send `/start` if you have not already done so.

2. **Log In to the Website with the Same Telegram Account**
   Go to **[`https://pixelverification.me`](https://pixelverification.me)** and click **Sign in with Telegram**.
   > ⚠️ **Important — Use the Same Telegram ID**: You **must** sign in with the **exact same Telegram account (Telegram ID)** that you used to start `@Pixel_verification_bot`. The website authenticates using your numeric Telegram ID so your Bot Wallet balance, lifetime top-up history, and API keys sync automatically. Logging in with a different Telegram account will create a separate empty wallet.

3. **Open the `API` Tab**
   After signing in, open the **`API`** tab (**API Keys & Credits**) from the navigation bar.

4. **Generate Your API Key**
   - **Automatic Self-Serve Generation**: Once your account has **more than `$50.00` in lifetime USD top-ups**, self-serve API key generation unlocks automatically. Click **`Generate My API Key`** to instantly create and link your personal `pvsk_...` key (Pixel Verification enabled, default `20 req/min` rate limit).
   - **Not unlocked yet?** The **`API`** tab displays your lifetime top-up progress and the remaining amount needed. Click **`Top Up Wallet`** (or go to the **`Top Up`** tab) to deposit funds via **Binance Pay, USDT on-chain, bKash, or Nagad**. Once your lifetime top-up exceeds `$50.00`, return to the **`API`** tab and click **`Generate My API Key`**.
   - **Want to Start with testing or few dollers??** Contact Owner Tanvir on Telegram at [`@Tanvir124`](https://t.me/Tanvir124).
   - **Need custom bulk pricing?** Contact Owner Tanvir on Telegram at [`@Tanvir124`](https://t.me/Tanvir124).

5. **Reveal & Copy Your `X-API-Key`**
   On the **`API`** tab, click the **eye icon** next to the `X-API-Key` box to reveal your full `pvsk_...` key, then tap/click the key box to copy it.

6. **Transfer Credits to Your API Key**
   Your Bot Wallet balance (USD) and External API balance (Credits) are managed separately on the portal:
   - In the **Move Bot Wallet → API Credits** section on the **`API`** tab, enter the USD amount (or pick `$1`, `$5`, `$10`, `$25`, or `Max`) and click **`Transfer`**.
   - Conversion rate is **`1 USD = 1 API Credit`** (instant, one-way transfer from Bot Wallet → External API Credits).

---

## Authentication

Every protected endpoint requires your API key passed in the `X-API-Key` header:

```http
X-API-Key: pvsk_a3b4c5d6e7f8g9h0j1k2l3m4n5o6p7q8
```

API keys start with `pvsk_` and can be generated directly from **[`https://pixelverification.me`](https://pixelverification.me)** (under the **`API`** tab) or issued manually by the Admin ([`@Tanvir124`](https://t.me/Tanvir124)).

| Scenario | HTTP Status | Response |
|----------|-------------|----------|
| Missing header | `401` | `{ "error": "Missing X-API-Key header" }` |
| Key invalid or deactivated | `401` | `{ "error": "Invalid or inactive API key" }` |
| Key lacks order permission | `403` | `{ "error": "API key does not have pixel verification permission" }` |
| Not enough credits | `402` | `{ "error": "Insufficient credits", "credits_remaining": 0.5, "credits_required": 1.0 }` |

> ⚠️ **Security Reminder**: Keep your API key strictly on your backend/server. Never expose it in client-side code, frontend JavaScript, or public repositories.

---

## Quick Start

### 1. Check Your Balance
```bash
curl https://api.pixelverification.me/api/credits \
  -H "X-API-Key: YOUR_API_KEY"
```

### 2. Submit an Order (e.g., 12-Month Normal Mode)
```bash
curl -X POST https://api.pixelverification.me/api/order \
  -H "Content-Type: application/json" \
  -H "X-API-Key: YOUR_API_KEY" \
  -H "X-Idempotency-Key: 550e8400-e29b-41d4-a716-446655440000" \
  -d '{
    "email": "user@gmail.com",
    "password": "user_password",
    "totp_secret": "JBSWY3DPEHPK3PXP",
    "flow_type": "normal",
    "validity": 12
  }'
```

**Response (`201 Created`):**
```json
{
  "success": true,
  "order_id": "e7c3a1f8-9d4b-4e2a-8f1c-6b5a3d2e1f0c",
  "flow_type": "normal",
  "validity": 12,
  "credit_cost": 0.7,
  "credits_remaining": 49.3,
  "message": "Order queued for processing"
}
```

### 3. Poll for the Result
```bash
curl https://api.pixelverification.me/api/order/e7c3a1f8-9d4b-4e2a-8f1c-6b5a3d2e1f0c \
  -H "X-API-Key: YOUR_API_KEY"
```

When `status` is `"success"`, extract `result.link`. When `"failed"`, inspect `error.message`.

---

## Endpoint Reference

| Method | Path | Auth Required | Deducts Credits | Rate Limited |
|--------|------|---------------|-----------------|--------------|
| `GET` | `/api/health` | No | No | No |
| `GET` | `/api/pricing` | No | No | No |
| `POST` | `/api/order` | **Yes** | **Yes** | **Yes** |
| `GET` | `/api/order/{id}` | **Yes** | No | No |
| `GET` | `/api/order-by-email?email=` | **Yes** | No | No |
| `GET` | `/api/credits` | **Yes** | No | No |

---

### `GET /api/health`

Public health check endpoint.
```json
{
  "status": "online",
  "timestamp": 1717020000000
}
```

---

### `GET /api/pricing`

Returns the active server pricing and validities.
```json
{
  "success": true,
  "version": 1,
  "updated_at": 1754800000000,
  "default_validity": 12,
  "allowed_validities": [3, 12],
  "pricing": {
    "normal": { "3": 0.6, "12": 0.7 },
    "full":   { "3": 0.9, "12": 1.0 }
  }
}
```

---

### `POST /api/order`

Submit a verification order for one of the 4 products.

**Headers:**
- `Content-Type: application/json` (Required)
- `X-API-Key: pvsk_...` (Required)
- `X-Idempotency-Key: <UUID>` (Recommended — prevents duplicate charges on network retry)

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `email` | string | **Yes** | Target Gmail address |
| `password` | string | **Yes** | Account password |
| `totp_secret` | string | **Yes** | Base32 2FA secret key (setup key, not the 6-digit code) |
| `flow_type` | string | **Yes** | `"normal"` or `"full"` |
| `validity` | integer | **Yes** | `3` or `12` |

**Success Response (`201 Created`):**
```json
{
  "success": true,
  "order_id": "e7c3a1f8-9d4b-4e2a-8f1c-6b5a3d2e1f0c",
  "flow_type": "full",
  "validity": 12,
  "credit_cost": 1.0,
  "credits_remaining": 45.0,
  "message": "Order queued for processing"
}
```

**Common Error Responses:**
- `400 Bad Request`: Missing field or invalid validity/combination (`{ "error": "Missing required field: email" }`).
- `401 Unauthorized`: Missing or invalid API key.
- `402 Payment Required`: Insufficient credits (`{ "error": "Insufficient credits", "credits_remaining": 0.5, "credits_required": 1.0 }`).
- `403 Forbidden`: API key does not have verification permission.
- `429 Too Many Requests`: Rate limit exceeded (`{ "error": "Rate limit exceeded", "retry_after_seconds": 30 }`).

---

### `GET /api/order/{order_id}`

Retrieves current status, queue position, progress, and results for an order.

**Pending / Queued Order Response (`200 OK` with `status: "pending"`):**
```json
{
  "success": true,
  "order": {
    "order_id": "e7c3a1f8-9d4b-4e2a-8f1c-6b5a3d2e1f0c",
    "email": "john.doe@gmail.com",
    "status": "pending",
    "queue_position": 3,
    "flow_type": "normal",
    "validity": 12,
    "credit_cost": 0.7,
    "created_at": 1717020000000,
    "started_at": null,
    "completed_at": null,
    "result": null,
    "error": null,
    "progress": null,
    "credit_refunded": false
  }
}
```

**Success Response (`200 OK`):**
```json
{
  "success": true,
  "order": {
    "order_id": "e7c3a1f8-9d4b-4e2a-8f1c-6b5a3d2e1f0c",
    "email": "john.doe@gmail.com",
    "status": "success",
    "queue_position": null,
    "flow_type": "normal",
    "validity": 12,
    "credit_cost": 0.7,
    "created_at": 1717020000000,
    "started_at": 1717020005000,
    "completed_at": 1717020180000,
    "result": {
      "link": "https://one.google.com/partner-eft-onboard/9YKTBN49CWCG3EBKYCYH",
      "link_type": "partner-eft-onboard",
      "retries_used": 1
    },
    "error": null,
    "progress": {
      "current_step": 8,
      "total_steps": 8,
      "step_label": "Finalizing the Process",
      "status": "success"
    },
    "credit_refunded": false
  }
}
```

**Failed Order Response (`200 OK` with `status: "failed"`):**
```json
{
  "success": true,
  "order": {
    "order_id": "e7c3a1f8-9d4b-4e2a-8f1c-6b5a3d2e1f0c",
    "email": "john.doe@gmail.com",
    "status": "failed",
    "queue_position": null,
    "flow_type": "full",
    "validity": 12,
    "credit_cost": 1.0,
    "error": {
      "code": "WRONG_PASSWORD",
      "message": "Wrong password",
      "step": 3,
      "user_message": "Please double-check the password and try again"
    },
    "result": null,
    "credit_refunded": true
  }
}
```

---

### `GET /api/order-by-email`

Finds your key's most recent order for a specific email submitted within the last 10 minutes (useful for recovery if network dropped during submission).

```http
GET /api/order-by-email?email=user@gmail.com
```

---

### `GET /api/credits`

Returns your account details, current credit balance, and rate limit.

```json
{
  "success": true,
  "credits_remaining": 49.3,
  "rate_limit": 20,
  "total_orders": 120,
  "name": "Developer Key 1"
}
```

---

## The 4 Available Products

When building your Telegram bot or web service, we recommend presenting the service as **4 distinct product options** in your user interface / buttons:

```
┌────────────────────────────────────────────────────────┐
│             SELECT VERIFICATION PRODUCT                │
├────────────────────────────┬───────────────────────────┤
│ [1] 12-Month Normal Mode   │ [2] 3-Month Normal Mode   │
│ (Link Only) — 0.7 credits  │ (Link Only) — 0.6 credits │
├────────────────────────────┼───────────────────────────┤
│ [3] 12-Month Full Mode     │ [4] 3-Month Full Mode     │
│ (Link + Active) — 1.0 cr   │ (Link + Active) — 0.9 cr  │
└────────────────────────────┴───────────────────────────┘
```

### Product Mapping for Bot Submission

| Product Option in Your Bot | Payload parameters to send |
|----------------------------|----------------------------|
| **12-Month Normal** | `{"flow_type": "normal", "validity": 12}` |
| **3-Month Normal** | `{"flow_type": "normal", "validity": 3}` |
| **12-Month Full** | `{"flow_type": "full", "validity": 12}` |
| **3-Month Full** | `{"flow_type": "full", "validity": 3}` |

*If any new validities or products are released, Admin Tanvir (`@Tanvir124`) will notify all developers with the updated details.*

---

## Normal vs Full Mode

| Feature | Normal Mode | Full Mode |
|---------|-------------|-----------|
| `flow_type` parameter | `"normal"` | `"full"` |
| Execution Time | ~1 to 2 minutes | ~3 to 5 minutes |
| What is Delivered | Google One redemption link | Redemption link **+ fully activated on account** |
| Response fields | `result.link` | `result.link` + `activation_status` |
| Recommended for | Users who want to redeem links themselves | Users who want turnkey automatic activation |

### Full Mode Activation Status (`activation_status`)

On full mode orders, check `order.activation_status`:
- `activated`: The subscription was successfully activated on the account.
- `already_redeemed`: The offer was already activated/redeemed on this account.
- `item_not_available`: Account is ineligible for this offer (credit fully refunded).
- `failed` / `invalid` / `error`: Activation step encountered an issue (credit fully refunded).

---

## Order Lifecycle & Polling

```
           POST /api/order (credits deducted)
                         │
                         ▼
                    ┌─────────┐
                    │ PENDING │   (Waiting for worker capacity)
                    └────┬────┘
                         │
                         ▼
                   ┌──────────┐
                   │PROCESSING│   (Running verification steps)
                   └─────┬────┘
                         │
                    ┌────┴────┐
                    ▼         ▼
              ┌─────────┐ ┌────────┐
              │ SUCCESS │ │ FAILED │ (Credit automatically refunded)
              └─────────┘ └────────┘
```

### Polling Best Practices
- When `status == "pending"`: Poll every **8–10 seconds**.
- When `status == "processing"`: Poll every **4–5 seconds**.
- When `status == "success"` or `status == "failed"`: **Stop polling immediately.**
- Set a client-side timeout of **10 minutes** for Normal Mode and **15 minutes** for Full Mode.

---

## Progress Tracking

### Queue Position (While Pending)

While `status` is `"pending"`, `GET /api/order/{order_id}` includes `order.queue_position` so you can show your users where their order stands in line:

- **`queue_position`** (`integer | null`): A **1-based** position among active pending orders ordered by creation time (`1` = next in line).
- Once the order moves to `"processing"` or finishes (`"success"` / `"failed"`), `queue_position` becomes `null`.
- Always handle `null` gracefully (e.g., display a standard `"Order Queued"` message without a position number if `queue_position` is `null`).
- **Update Behavior**: Queue positions are cached and refreshed approximately every **60 seconds**, so `queue_position` may take up to a minute to reflect orders ahead leaving the queue. When updating live chat/Telegram messages, only edit the message when `queue_position` changes (or every ~1–2 minutes) to avoid redundant UI edits.

### Processing Steps

While `status` is `"processing"`, the `order.progress` object provides live status updates you can show to your bot users:

```json
"progress": {
  "current_step": 4,
  "total_steps": 8,
  "step_label": "Verifying 2FA code",
  "status": "start"
}
```

**Steps Overview (1 to 8):**
1. Initializing device connection
2. Signing into Google account
3. Entering password
4. Verifying 2FA code
5. Completing sign-in
6. Opening offer page
7. Extracting offer URL *(Normal)* / Adding Payment Method *(Full)*
8. Finalizing the Process *(Normal)* / Activating Google One *(Full)*

---

## Credits & Billing

1. **Self-Serve Top-Ups & Transfers**: Deposit USD to your synced Bot Wallet via **[`https://pixelverification.me`](https://pixelverification.me)** (Binance Pay, USDT on-chain, bKash, or Nagad) — making sure to sign in with the same Telegram account used to start [`@Pixel_verification_bot`](https://t.me/Pixel_verification_bot) — then transfer USD to API Credits (`1 USD = 1 API Credit`) under the **`API`** tab.
2. **Automatic Refund Guarantee**: You only pay for successful orders. If an order fails for any reason (wrong credentials, Google block, ineligible account, server error), the exact amount charged is **instantly refunded** back to your API key balance.
3. **Fractional Credits Supported**: Balance and deduction values support decimals (e.g. `0.6`, `0.7`, `0.9`, `1.0`).
4. **Atomic Balance Updates**: Balance deductions and refunds are atomic; concurrent requests are safe from race conditions.

---

## Error Codes & Troubleshooting

When an order fails, `order.error.code` provides the exact reason:

| Error Code | Meaning | What your bot should tell the user | Retryable? |
|------------|---------|------------------------------------|------------|
| `WRONG_PASSWORD` | Password was incorrect | Ask user to check their password | No |
| `WRONG_2FA_CODE` | TOTP secret key is invalid | Ask user to re-check 2FA key | No |
| `WRONG_EMAIL` | Email format invalid | Verify the Gmail address | No |
| `2FA_WARNING` | 2FA not set up properly on account | Ask user to set up 2FA correctly | No |
| `PAYMENT_WARNING` | Payment profile issue on Google | User needs to close payment profile | No |
| `CAPTCHA_DETECTED` | Google triggered a manual Captcha | User should sign in once manually | No |
| `ACTIVATION_ITEM_NOT_AVAILABLE` | This specific Offer is ineligible for this Gmail | Do NOT resubmit this Gmail For this offer(blacklisted). But you can try other validity offer | No |
| `OTHER_ERROR` | Generic unexpected error | Order is refunded; safe to retry | Yes |

---

## Rate Limiting

- Default rate limit is **20 order submissions per minute** per API key (can be increased by Admin on request).
- Polling (`GET /api/order/{id}`), balance check (`GET /api/credits`), and health check endpoints are **unlimited** and not rate-limited.
- If you exceed the submission limit, you receive `429 Too Many Requests` with a `retry_after_seconds` field.

---

## Idempotency & Timeout Recovery

To prevent accidental duplicate orders if your bot encounters a network timeout:
1. Generate a UUID `X-Idempotency-Key` header for each order.
2. If your request times out, retry with the **same** `X-Idempotency-Key`.
3. If the server already created the order, it returns `200 OK` with the existing `order_id` without charging you again.

---

## Client Examples

### Python Integration Example

```python
"""Pixel Verification API - Python Bot Integration Helper."""

import time
import uuid
import requests

BASE_URL = "https://api.pixelverification.me"
API_KEY = "pvsk_your_api_key_here"
HEADERS = {"Content-Type": "application/json", "X-API-Key": API_KEY}

# The 4 standard products
PRODUCTS = {
    "12M_NORMAL": {"flow_type": "normal", "validity": 12, "name": "12-Month Normal"},
    "3M_NORMAL":  {"flow_type": "normal", "validity": 3,  "name": "3-Month Normal"},
    "12M_FULL":   {"flow_type": "full",   "validity": 12, "name": "12-Month Full Mode"},
    "3M_FULL":    {"flow_type": "full",   "validity": 3,  "name": "3-Month Full Mode"},
}


def submit_verification_order(email, password, totp_secret, product_key="12M_NORMAL"):
    """Submits an order for one of the 4 defined products."""
    product = PRODUCTS.get(product_key, PRODUCTS["12M_NORMAL"])
    payload = {
        "email": email.strip(),
        "password": password,
        "totp_secret": totp_secret.strip(),
        "flow_type": product["flow_type"],
        "validity": product["validity"],
    }
    headers = {**HEADERS, "X-Idempotency-Key": str(uuid.uuid4())}

    res = requests.post(f"{BASE_URL}/api/order", json=payload, headers=headers, timeout=30)
    data = res.json()

    if res.status_code in (200, 201):
        return data["order_id"], data.get("credit_cost"), data.get("credits_remaining")
    else:
        raise RuntimeError(f"Order failed ({res.status_code}): {data.get('error')}")


def poll_order(order_id, on_progress=None, on_queue=None, timeout=900):
    """Polls the order until terminal success or failure."""
    start_time = time.time()
    last_queue_pos = None
    while time.time() - start_time < timeout:
        res = requests.get(f"{BASE_URL}/api/order/{order_id}", headers=HEADERS, timeout=30)
        if res.status_code != 200:
            time.sleep(5)
            continue

        order = res.json().get("order", {})
        status = order.get("status")

        if status == "success":
            return order

        if status == "failed":
            error_info = order.get("error", {})
            raise RuntimeError(f"Order Failed: {error_info.get('user_message') or error_info.get('message')}")

        if status == "processing":
            if on_progress:
                prog = order.get("progress", {})
                on_progress(prog.get("current_step", 0), prog.get("step_label", ""))
            time.sleep(4)
        else:
            # Pending state
            q_pos = order.get("queue_position")
            if on_queue and q_pos and q_pos != last_queue_pos:
                last_queue_pos = q_pos
                on_queue(q_pos)
            time.sleep(8)

    raise TimeoutError(f"Order {order_id} timed out.")


# ─── Quick Test ─────────────────────────────────────────────────────────────
if __name__ == "__main__":
    email = "test@gmail.com"
    pwd = "password123"
    totp = "JBSWY3DPEHPK3PXP"

    print("Submitting 12-Month Normal Mode order...")
    order_id, cost, remaining = submit_verification_order(email, pwd, totp, "12M_NORMAL")
    print(f"Order {order_id} created. Cost: {cost} cr | Balance: {remaining} cr")

    def queue_callback(position):
        print(f"  -> Queue Position: #{position}")

    def progress_callback(step, label):
        if label:
            print(f"  -> Step {step}/8: {label}")

    result = poll_order(order_id, on_progress=progress_callback, on_queue=queue_callback)
    print(f"✅ Success! Link: {result['result']['link']}")
```

---

### Node.js Integration Example

```javascript
/**
 * Pixel Verification API - Node.js Bot Integration Helper
 */
const BASE_URL = "https://api.pixelverification.me";
const API_KEY = "pvsk_your_api_key_here";
const HEADERS = { "Content-Type": "application/json", "X-API-Key": API_KEY };

const PRODUCTS = {
  NORMAL_12M: { flow_type: "normal", validity: 12 },
  NORMAL_3M:  { flow_type: "normal", validity: 3 },
  FULL_12M:   { flow_type: "full",   validity: 12 },
  FULL_3M:    { flow_type: "full",   validity: 3 },
};

async function submitOrder({ email, password, totpSecret, product = "NORMAL_12M" }) {
  const { flow_type, validity } = PRODUCTS[product] || PRODUCTS.NORMAL_12M;
  const idempotencyKey = crypto.randomUUID();

  const res = await fetch(`${BASE_URL}/api/order`, {
    method: "POST",
    headers: { ...HEADERS, "X-Idempotency-Key": idempotencyKey },
    body: JSON.stringify({
      email: email.trim(),
      password,
      totp_secret: totpSecret.trim(),
      flow_type,
      validity,
    }),
  });

  const data = await res.json();
  if (res.status === 200 || res.status === 201) {
    return data;
  }
  throw new Error(`Order submission failed: ${data.error}`);
}

async function pollOrder(orderId, { onProgress, onQueue, timeoutMs = 900000 } = {}) {
  const start = Date.now();
  let lastQueuePos = null;

  while (Date.now() - start < timeoutMs) {
    const res = await fetch(`${BASE_URL}/api/order/${orderId}`, { headers: HEADERS });
    if (!res.ok) {
      await new Promise(r => setTimeout(r, 5000));
      continue;
    }

    const { order } = await res.json();
    if (order.status === "success") return order;

    if (order.status === "failed") {
      throw new Error(`Order failed: ${order.error?.user_message || order.error?.message}`);
    }

    if (order.status === "processing") {
      onProgress?.(order.progress?.current_step, order.progress?.step_label);
      await new Promise(r => setTimeout(r, 4000));
    } else {
      if (order.queue_position && order.queue_position !== lastQueuePos) {
        lastQueuePos = order.queue_position;
        onQueue?.(order.queue_position);
      }
      await new Promise(r => setTimeout(r, 8000));
    }
  }
  throw new Error(`Order ${orderId} timed out.`);
}
```

---

### cURL Examples for the 4 Products

```bash
# 1. Normal Mode - 12 Months (0.7 cr)
curl -X POST https://api.pixelverification.me/api/order \
  -H "Content-Type: application/json" \
  -H "X-API-Key: pvsk_your_key" \
  -d '{"email":"user@gmail.com","password":"pwd","totp_secret":"2FAKEY","flow_type":"normal","validity":12}'

# 2. Normal Mode - 3 Months (0.6 cr)
curl -X POST https://api.pixelverification.me/api/order \
  -H "Content-Type: application/json" \
  -H "X-API-Key: pvsk_your_key" \
  -d '{"email":"user@gmail.com","password":"pwd","totp_secret":"2FAKEY","flow_type":"normal","validity":3}'

# 3. Full Mode - 12 Months (1.0 cr)
curl -X POST https://api.pixelverification.me/api/order \
  -H "Content-Type: application/json" \
  -H "X-API-Key: pvsk_your_key" \
  -d '{"email":"user@gmail.com","password":"pwd","totp_secret":"2FAKEY","flow_type":"full","validity":12}'

# 4. Full Mode - 3 Months (0.9 cr)
curl -X POST https://api.pixelverification.me/api/order \
  -H "Content-Type: application/json" \
  -H "X-API-Key: pvsk_your_key" \
  -d '{"email":"user@gmail.com","password":"pwd","totp_secret":"2FAKEY","flow_type":"full","validity":3}'
```

---

## Integration Checklist

- [ ] Start [`@Pixel_verification_bot`](https://t.me/Pixel_verification_bot) on Telegram and log in to [`https://pixelverification.me`](https://pixelverification.me) with that **exact same Telegram account (Telegram ID)** to generate/copy your API key and fund API credits.
- [ ] Point your client `BASE_URL` to `https://api.pixelverification.me`.
- [ ] Store API keys safely on your backend (never exposed to users or client code).
- [ ] Provide 4 separate menu / button options to your bot users (12M Normal, 3M Normal, 12M Full, 3M Full).
- [ ] Send corresponding `flow_type` (`"normal"` or `"full"`) and `validity` (`12` or `3`).
- [ ] Include `X-Idempotency-Key` (UUID) with each order request.
- [ ] Poll `GET /api/order/{id}` at reasonable intervals (8s pending, 4s processing).
- [ ] Optionally display `order.queue_position` while `status` is `"pending"` (handle `null` gracefully and only update UI when it changes).
- [ ] Stop polling once status reaches `success` or `failed`.
- [ ] Display `order.error.user_message` to your users on failed orders.

---

## FAQ

**Q: How do I get an API key and add credits?**
A: Start [`@Pixel_verification_bot`](https://t.me/Pixel_verification_bot) on Telegram, then visit **[`https://pixelverification.me`](https://pixelverification.me)** and sign in with **that same Telegram account (Telegram ID)**. Open the **`API`** tab to generate your `pvsk_...` key (automatically unlocked once your lifetime USD top-ups exceed `$50.00`) and transfer USD from your Bot Wallet into API Credits (`1 USD = 1 Credit`).

**Q: Why is my wallet balance showing `$0` on `pixelverification.me`?**
A: Make sure you logged in using the **same Telegram ID** you used to start `@Pixel_verification_bot`. Signing in with a different Telegram account opens a separate account.

**Q: Are failed orders refunded?**
A: **Yes, 100% automatically.** If an order fails for any reason, the exact credit cost is refunded to your balance immediately.

**Q: What if I need custom pricing or higher volume?**
A: Contact Owner Tanvir on Telegram at [`@Tanvir124`](https://t.me/Tanvir124) to arrange custom rates or package pricing for your API key.

**Q: Will more validity plans be added in the future?**
A: Yes, if new plans or validities become available, the Admin will notify all developers with their parameters and pricing.

**Q: How do I check my remaining balance?**
A: Send a `GET` request to `/api/credits` with your `X-API-Key` header, or view it live on the **`API`** tab at [`https://pixelverification.me`](https://pixelverification.me).

---

## Support & Admin Contact

For self-serve API keys, wallet top-ups, and credit transfers:

- **Web Portal**: [`https://pixelverification.me`](https://pixelverification.me) *(Sign in with the same Telegram account used for `@Pixel_verification_bot`)*
- **API Base URL**: `https://api.pixelverification.me`
- **Telegram Bot**: [`@Pixel_verification_bot`](https://t.me/Pixel_verification_bot)

For custom pricing, higher rate limits, manual key setup, or technical inquiries:

- **Telegram Admin**: [`@Tanvir124`](https://t.me/Tanvir124)
- **Service**: Pixel Verification API

---
*Developer Integration Guide — Updated November 2026*
