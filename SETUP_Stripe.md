# SETUP: Stripe — Payment & Revenue Tracking

Stripe เป็น Payment Processing platform ที่ใช้ใน **04 Finance Agent** เพื่อดึงข้อมูล transactions รายสัปดาห์และวิเคราะห์ Revenue ของบริษัท

---

## Stripe คืออะไร

Stripe เป็น Payment Infrastructure ที่รับชำระเงินออนไลน์ได้ทั่วโลก ลูกค้าจ่ายเงินเข้า Stripe Dashboard ของคุณ แล้ว Finance Agent ดึงข้อมูลออกมาผ่าน Stripe API ทุกศุกร์เพื่อสร้าง P&L Report อัตโนมัติ

**บทบาทในโปรเจค:**
- เป็นแหล่ง "ความจริง" ของ Revenue ทั้งหมด
- Finance Agent อ่าน charges / refunds ย้อนหลัง 7 วัน
- Agent จัดหมวดรายได้เป็น 5 streams: Freelance, Content, Affiliate, SaaS, Consulting
- คำนวณ MRR trend + เปรียบเทียบกับ week ก่อน

---

## แผนและราคา

| แผน | ค่าธรรมเนียมต่อ Transaction | API Access | เหมาะสำหรับ |
|---|---|---|---|
| **Standard** | 2.9% + $0.30 | ✅ ฟรี | ทุกโปรเจค — ไม่มีค่าสมัครรายเดือน |
| **Plus** | 2.7% + $0.30 | ✅ ฟรี | ยอดขาย > $50K/เดือน |
| **Custom** | ต่อรองได้ | ✅ ฟรี | Enterprise |

> **สรุป:** Stripe API ฟรีทุกแผน คุณจ่ายเฉพาะ % ของ transaction — ถ้ายังไม่มี revenue ก็ยังไม่มีค่าใช้จ่าย

---

## วิธี Setup (5 ขั้นตอน)

### ขั้นตอนที่ 1 — สร้างบัญชี Stripe

1. ไปที่ [https://dashboard.stripe.com/register](https://dashboard.stripe.com/register)
2. สมัครด้วย email + ยืนยัน
3. กรอกข้อมูลธุรกิจ (ชื่อ, ประเทศ, ประเภทธุรกิจ)
4. ยืนยันตัวตน (เพื่อ Payout — ใช้ Passport หรือ ID Card)

> **หมายเหตุ:** สำหรับ **ทดสอบ** ไม่ต้องยืนยันตัวตนก็ได้ ใช้ Test Mode API Key ได้เลย

---

### ขั้นตอนที่ 2 — ดึง API Key

1. เข้า [https://dashboard.stripe.com/apikeys](https://dashboard.stripe.com/apikeys)
2. คุณจะเห็น 2 ชุด key:

| Mode | Key Format | ใช้เมื่อ |
|---|---|---|
| **Test** | `sk_test_...` | ทดสอบ workflow โดยไม่ใช้เงินจริง |
| **Live** | `sk_live_...` | Production — เงินจริง |

3. คลิก **Reveal live key** (หรือ test key) → Copy

> ⚠️ **ความปลอดภัย:** Secret Key (`sk_`) มีสิทธิ์เต็ม ห้ามใส่ใน code หรือ commit ลง Git เด็ดขาด — ใช้ n8n Credential เท่านั้น

---

### ขั้นตอนที่ 3 — สร้าง Credential ใน n8n

ไป **n8n → Settings → Credentials → New Credential**

1. เลือก **"Header Auth"**
2. ตั้งค่า:

```
Name:   Stripe API
Header Name:   Authorization
Header Value:  Bearer sk_live_XXXXXXXXXXXXXXXXXXXXXXXX
```

3. กด **Save**
4. จด Credential ID ที่ได้ (จะใส่ใน `04_Finance_Agent.json` ทีหลัง)

> **ทำไมไม่ใช้ Stripe Node ของ n8n?** Finance Agent ใช้ `toolHttpRequest` ซึ่ง bind กับ HTTP Header Auth โดยตรง — ยืดหยุ่นกว่าและ Agent เรียกเองได้อัตโนมัติ

---

### ขั้นตอนที่ 4 — อัปเดต Credential ID ใน Workflow

เปิด `04_Finance_Agent.json` หาโหนด `Tool: Stripe Revenue` ตรง parameter `credentials` — ใส่ ID ที่ได้จากขั้นตอนที่ 3 (แต่สำหรับ toolHttpRequest จะ bind ผ่าน `genericAuthType: "httpHeaderAuth"` ไม่ได้ระบุ credential ID โดยตรง ให้เลือก credential ใน n8n UI ตอน import แทน)

---

### ขั้นตอนที่ 5 — ทดสอบ API

ทดสอบด้วย curl หรือ terminal:

```powershell
$headers = @{ Authorization = "Bearer sk_test_XXXXXXXXXXXX" }
Invoke-RestMethod -Uri "https://api.stripe.com/v1/charges?limit=5" -Headers $headers
```

ผลลัพธ์ที่ถูกต้อง:
```json
{
  "object": "list",
  "data": [...],
  "has_more": false
}
```

---

## API ที่ใช้ในโปรเจค

### `GET /v1/charges`

Finance Agent เรียก endpoint นี้ทุกศุกร์ 18:00

| Parameter | ค่าใน Workflow | ความหมาย |
|---|---|---|
| `limit` | `100` | ดึงมากสุด 100 transactions |
| `created[gte]` | Unix timestamp (7 วันที่แล้ว) | กรองเฉพาะ transaction สัปดาห์นี้ |

**ตัวอย่าง URL จริง:**
```
GET https://api.stripe.com/v1/charges?limit=100&created[gte]=1737590400
```

**ตัวอย่าง Response:**
```json
{
  "object": "list",
  "data": [
    {
      "id": "ch_3abc123",
      "amount": 19900,
      "currency": "usd",
      "status": "succeeded",
      "description": "SaaS Monthly Subscription",
      "created": 1737676800,
      "refunded": false,
      "metadata": {
        "product": "automation_starter",
        "customer_tier": "small_business"
      }
    }
  ],
  "has_more": false
}
```

> **หน่วยเงิน:** Stripe ส่งค่าเป็น cents (19900 = $199.00) — Finance Agent จะหาร 100 ในการวิเคราะห์

---

## Revenue Streams ที่ Finance Agent จัดหมวด

Finance Agent จะอ่าน `description` และ `metadata` ของแต่ละ charge แล้วจัดหมวดให้อัตโนมัติ:

| Stream | ตัวอย่าง Description | เป้าหมายรายสัปดาห์ |
|---|---|---|
| **Freelance** | "Automation consulting - Client A" | $500–1,500 |
| **Content** | "Newsletter sponsorship", "Course sale" | $200–800 |
| **Affiliate** | "Referral commission - n8n", "Affiliate payout" | $100–500 |
| **SaaS** | "Monthly subscription - Starter Plan" | $500–2,000 |
| **Consulting** | "Strategy session", "Implementation fee" | $300–1,000 |

---

## แนะนำการตั้ง Metadata ใน Stripe

เพื่อให้ Finance Agent จัดหมวดได้แม่นยำขึ้น ตั้งค่า metadata ตอนสร้าง Charge หรือ Payment Link:

```json
{
  "metadata": {
    "stream": "saas",
    "product": "automation_starter",
    "billing_cycle": "monthly"
  }
}
```

> **ใช้ Payment Links:** ใน Stripe Dashboard → **Payment Links** → Create Link → ตั้ง metadata ให้แต่ละ product ไว้ล่วงหน้า

---

## Webhook (Optional — สำหรับ Real-time)

ปัจจุบัน Finance Agent **ดึงข้อมูลแบบ Polling** ทุกศุกร์ ถ้าต้องการ real-time alerts เพิ่มในภายหลัง:

1. Stripe Dashboard → **Developers → Webhooks → Add endpoint**
2. ใส่ URL: `https://YOUR_N8N_URL/webhook/stripe-payment`
3. Events ที่น่าสนใจ: `payment_intent.succeeded`, `charge.refunded`, `invoice.payment_failed`

---

## Test Mode vs Live Mode

| | Test Mode | Live Mode |
|---|---|---|
| Key | `sk_test_...` | `sk_live_...` |
| เงิน | ไม่จริง | จริง |
| Test Card | `4242 4242 4242 4242` | บัตรจริงเท่านั้น |
| ใช้เมื่อ | ทดสอบ workflow | Production |

**แนะนำ:** ใช้ Test Mode ก่อนทุกครั้งที่แก้ workflow Finance Agent — ป้องกัน noise ใน data จริง

---

## KPI ที่ควร Monitor

เข้าใช้งานที่ [https://dashboard.stripe.com](https://dashboard.stripe.com)

| Metric | ความหมาย | เป้าหมาย |
|---|---|---|
| **Gross Volume** | รายรับรวมก่อนหัก fee | สูงขึ้นทุกสัปดาห์ |
| **Net Volume** | หลังหัก Stripe fee | ≥ 95% ของ Gross |
| **Refund Rate** | % ที่ถูก refund | < 2% |
| **Failed Payments** | บัตรปฏิเสธ | < 5% ของ attempts |
| **MRR** | Monthly Recurring Revenue | Trend ขึ้น |

---

## Troubleshooting

### `401 Unauthorized`
- API Key ผิดหรือหมดอายุ
- ตรวจสอบว่าใช้ `sk_` (Secret Key) ไม่ใช่ `pk_` (Publishable Key)
- ตรวจสอบ Header: `Authorization: Bearer sk_...` (มี space หลัง Bearer)

### `429 Too Many Requests`
- Stripe rate limit: 100 req/sec (Standard), 1,000 req/sec (Plus)
- ไม่น่าเกิดในโปรเจคนี้เพราะเรียกแค่ครั้งเดียวต่อสัปดาห์

### Finance Agent ได้ข้อมูลไม่ครบ
- ตรวจสอบ `created[gte]` ว่า Unix timestamp ถูกต้อง
- ถ้ามี > 100 transactions/สัปดาห์ ต้องเพิ่ม pagination: `starting_after` parameter
- ใช้ Test Mode ยืนยันว่า API Key ทำงาน

### ไม่มี transaction ใน data
- ใน Test Mode ต้องสร้าง test payment ก่อน: Dashboard → Products → Create Test Payment
- ตรวจสอบ timezone: `$now` ใน n8n ใช้ UTC

### `charge.refunded` ไม่ถูกหัก
- `/v1/charges` endpoint ส่ง `refunded: true` และ `amount_refunded` มาด้วย
- Finance Agent system prompt สั่งให้หักออกจาก net revenue แล้ว

---

## ลิงก์ที่เป็นประโยชน์

- [Stripe Dashboard](https://dashboard.stripe.com)
- [Stripe API Reference — Charges](https://stripe.com/docs/api/charges/list)
- [Stripe Test Cards](https://stripe.com/docs/testing#cards)
- [Stripe API Keys](https://dashboard.stripe.com/apikeys)
- [Stripe Webhooks](https://stripe.com/docs/webhooks)
- [04 Finance Agent README](04_Finance_Agent_README.md)
