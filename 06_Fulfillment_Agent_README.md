# 06 Fulfillment Agent — Auto-Deliver Digital Products on Purchase

| Property | Value |
|---|---|
| **File** | `06_Fulfillment_Agent.json` |
| **Trigger** | Stripe Webhook — POST `/webhook/stripe-webhook` (24/7) |
| **Version** | 1.0.0 |
| **Status** | Activate before going live with Stripe products |

---

## วัตถุประสงค์

Fulfillment Agent คือ **ระบบ delivery อัตโนมัติ** ที่ทำงานทันทีเมื่อมีลูกค้าจ่ายเงิน:

1. รับ event จาก Stripe (`checkout.session.completed`)
2. ตรวจสอบว่าจ่ายเงินสำเร็จ
3. ดึงข้อมูล product จาก Google Sheets `Product_Catalog`
4. AI เขียน delivery email พร้อม product content ครบถ้วน
5. SendGrid ส่ง email ให้ลูกค้าภายใน ~1 นาที
6. บันทึกลง `Order_Log` sheet

> **ไม่มีมนุษย์เข้ามาเกี่ยวข้องเลยตั้งแต่ต้นจนจบ**

---

## การทำงาน (Node-by-Node Flow)

```
Stripe fires checkout.session.completed event
    │
    ▼
Webhook: Stripe Payment  (POST /webhook/stripe-webhook)
    │
    ▼
Check: Is Paid?  ─── IF event=checkout.session.completed AND payment_status=paid
    │ YES                           │ NO (ignored — no action)
    ▼
Extract Order Details
    │  customer_email, customer_name, product_type, product_title
    │  amount_paid, session_id, fulfillment_prompt
    ▼
Fulfillment Agent  ←── OpenAI GPT-4o (temp: 0.3, max 4000 tokens)
    │             ←── Tool: Read Product Catalog (Google Sheets)
    │             ←── Tool: Send Delivery Email (SendGrid via toolCode)
    │
    │  Output JSON: { productFound, productTitle, emailSent, customerEmail, deliveryStatus }
    ▼
Log to Order Log  (Google Sheets append → Order_Log sheet)
```

---

## Tools ที่ใช้

| Tool | ประเภท | หน้าที่ |
|---|---|---|
| `read_product_catalog` | HTTP (Google Sheets API) | ดึง product content จาก Product_Catalog sheet |
| `send_delivery_email` | toolCode (SendGrid API) | ส่ง HTML email พร้อม full product content |

---

## Credentials ที่ต้องตั้ง

**n8n Credentials (Settings → Credentials):**

| Credential | ประเภท | ใช้ใน Node |
|---|---|---|
| OpenAI API | OpenAI API | `OpenAI GPT-4o` |
| Google Sheets OAuth2 | Google Sheets OAuth2 | `Tool: Read Product Catalog`, `Log to Order Log` |

**n8n Environment Variables (Settings → Environment Variables):**

| Variable | ค่า | อธิบาย |
|---|---|---|
| `SENDGRID_API_KEY` | `SG.xxxxx...` | SendGrid API key — ดู [SETUP_SendGrid.md](SETUP_SendGrid.md) |
| `SENDGRID_FROM_EMAIL` | `hello@yourdomain.com` | Verified sender email ใน SendGrid |
| `COMPANY_NAME` | `AI Automation Team` | ชื่อที่แสดงในอีเมล |
| `REPLACE_WITH_SPREADSHEET_ID` | Google Sheet ID | เปลี่ยนใน node parameters ของ `Tool: Read Product Catalog` และ `Log to Order Log` |

> **Note:** SendGrid sender address ต้อง verify ก่อน — ดู SETUP_SendGrid.md

---

## Stripe Setup

### 1. Create Webhook Endpoint

1. Stripe Dashboard → Developers → Webhooks → **Add endpoint**
2. Endpoint URL: `https://YOUR_N8N_URL/webhook/stripe-webhook`
3. Events to listen: `checkout.session.completed`
4. Copy **Webhook Signing Secret** (สำหรับ future signature verification)

### 2. Add Required Metadata to Stripe Products

ทุก Stripe product ที่จะ fulfill ต้องมี metadata ดังนี้:

```json
{
  "product_type": "prompt_pack",
  "product_title": "50 ChatGPT Prompts for Freelancers"
}
```

**วิธีตั้ง metadata:**
- Stripe Dashboard → Products → เลือก product → Metadata → Add
- หรือตอนสร้าง Stripe Payment Link → More options → Metadata

**ค่า `product_type` ที่รองรับ:**

| product_type | คำอธิบาย |
|---|---|
| `prompt_pack` | ชุด prompt สำหรับ GPT |
| `n8n_template` | n8n workflow JSON + setup guide |
| `ai_guide` | PDF-ready guide |
| `checklist` | Automation checklist |

---

## Google Sheets ที่ต้องมี

### Product_Catalog (สร้างโดย Product Agent — 03)

Fulfillment Agent ค้นหา product ใน sheet นี้ก่อน deliver:

| Column | ข้อมูล | ตัวอย่าง |
|---|---|---|
| `created_at` | Timestamp | `2026-02-25T09:00:00.000Z` |
| `week_number` | เลขสัปดาห์ | `9` |
| `product_result` | JSON content ของ product (AI generated) | `{"title":"50 Prompts...","content":"..."}` |
| `status` | สถานะ | `draft` / `active` |

### Order_Log (สร้างโดย Fulfillment Agent)

บันทึกทุก order ที่ fulfill แล้ว:

| Column | ข้อมูล |
|---|---|
| `fulfilled_at` | Timestamp ที่ deliver |
| `session_id` | Stripe Checkout Session ID |
| `customer_email` | Email ลูกค้า |
| `product_title` | ชื่อ product ที่ซื้อ |
| `product_type` | ประเภท product |
| `amount_paid` | จำนวนเงินที่จ่าย (USD) |
| `fulfillment_result` | JSON output จาก Fulfillment Agent |
| `status` | `delivered` |

---

## Input / Output

### Input (จาก Stripe Webhook)

```json
{
  "type": "checkout.session.completed",
  "data": {
    "object": {
      "id": "cs_live_xxxx",
      "payment_status": "paid",
      "amount_total": 1900,
      "customer_email": "customer@example.com",
      "customer_details": { "name": "John Doe", "email": "customer@example.com" },
      "metadata": {
        "product_type": "prompt_pack",
        "product_title": "50 ChatGPT Prompts for Freelancers"
      }
    }
  }
}
```

### Output (Delivery Email ที่ลูกค้าได้รับ)

```
Subject: Your 50 ChatGPT Prompts for Freelancers is here! 🎉

Hi John,

Thank you for purchasing "50 ChatGPT Prompts for Freelancers"!

Here are your prompts:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PROMPT 1: "You are a freelance rate negotiation expert..."
PROMPT 2: "Create a professional project proposal for..."
...
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Usage Tips:
1. Copy each prompt into ChatGPT 4o for best results
2. Replace [brackets] with your specific context
...

Want more AI automation tips? Subscribe to our newsletter →

The AI Automation Team
```

---

## วิธี Activate

1. Import `06_Fulfillment_Agent.json` เข้า n8n
2. ตั้งค่า credentials: OpenAI API + Google Sheets OAuth2
3. ตั้ง environment variables: `SENDGRID_API_KEY`, `SENDGRID_FROM_EMAIL`, `COMPANY_NAME`
4. เปลี่ยน `REPLACE_WITH_SPREADSHEET_ID` ใน node parameters ทั้ง 2 ที่
5. ตั้ง Stripe Webhook ให้ชี้มาที่ n8n URL
6. **Activate** workflow (toggle ON)
7. ทดสอบด้วย Stripe test webhook: Stripe CLI → `stripe trigger checkout.session.completed`

> **ลำดับ Activate:** ต้อง activate 03_Product_Agent ก่อน เพื่อให้มี products ใน Product_Catalog

---

## Troubleshooting

| ปัญหา | สาเหตุที่เป็นไปได้ | วิธีแก้ |
|---|---|---|
| Webhook ไม่รับ event | Stripe URL ผิด หรือ n8n ไม่ได้ active | ตรวจ Stripe → Webhooks → Recent deliveries |
| `product not found in catalog` | ยังไม่มี product ใน Product_Catalog sheet | รัน 03_Product_Agent ก่อน ให้มี product |
| SendGrid error 403 | API key ผิด หรือ sender ยังไม่ verify | ตรวจ SENDGRID_API_KEY + verify sender email |
| Email ไม่ถึงลูกค้า | Spam filter หรือ domain reputation | ตั้ง SPF/DKIM ใน SendGrid → ดู SETUP_SendGrid.md |
| `SENDGRID_FROM_EMAIL env var is not set` | ยังไม่ได้ตั้ง env var | n8n → Settings → Environment Variables |
| Amount = 0 | Stripe metadata ขาด `amount_total` | ตรวจ Stripe checkout session format |

---

## Audit Checklist (ดูทุกสัปดาห์)

- [ ] Order_Log มี record ใหม่สอดคล้องกับจำนวน sales ใน Stripe
- [ ] ไม่มี order ที่ `status = pending` ค้างอยู่ใน Order_Log
- [ ] n8n Executions ไม่มี error ใน Fulfillment Agent
- [ ] ทดสอบ test order ด้วย Stripe test mode เดือนละ 1 ครั้ง

---

## Human Support — สิ่งที่คุณต้องทำ

| ความถี่ | งาน | เวลา |
|---|---|---|
| **รายสัปดาห์** | เปิด Order_Log → ตรวจว่า orders fulfilled ครบ | 5 นาที |
| **ถ้ามี complaint** | ลูกค้าไม่ได้รับ email → resend manually จาก SendGrid | เมื่อพบปัญหา |
| **รายเดือน** | ตรวจ delivery rate ใน SendGrid Analytics | 5 นาที |
