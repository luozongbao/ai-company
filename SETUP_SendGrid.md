# SendGrid — Email Service Setup Guide

| Property | Value |
|---|---|
| **ใช้โดย Workflow** | `03_Sales_Agent.json` |
| **หน้าที่** | ส่ง personalized outreach email ไปยัง prospects |
| **ราคา** | Free tier — $0 (3,000 email/เดือน) |
| **เวลาตั้งค่า** | ~15 นาที |

---

## SendGrid คืออะไร?

SendGrid คือบริการ **Transactional Email API** ที่ใช้ส่ง email แบบ programmatic (ผ่าน API) แทนการส่งจาก Gmail/Outlook โดยตรง มาตรฐานสำหรับ developer และ SaaS ที่ต้องการ email deliverability สูง

### ทำไม Project นี้ถึงใช้ SendGrid แทน Gmail?

| | Gmail ตรงๆ | SendGrid |
|---|---|---|
| **Rate limit** | 500 email/วัน (personal) | 100/วัน free tier — เพียงพอ |
| **Deliverability** | ต่ำ — มักถูก spam filter | สูง — มี IP reputation ดี |
| **Spam risk** | สูง เมื่อส่งหลาย recipient | ต่ำ — มี compliance built-in |
| **API integration** | ต้องใช้ OAuth ซับซ้อน | API Key ตรงๆ — ง่ายกว่ามาก |
| **Tracking** | ไม่มี | มี open rate / click tracking |
| **n8n setup** | หลายขั้นตอน | ตั้ง env vars 3 ตัว เสร็จ |

---

## Plan ที่แนะนำ

| Plan | ราคา | Email/เดือน | เหมาะกับ |
|---|---|---|---|
| **Free** | **$0** | 3,000 | ✅ เดือน 1–6 (Sales Agent ส่ง ~10/วัน = ~300/เดือน) |
| Essentials 50K | $19.95/เดือน | 50,000 | เมื่อ scale > 1,000 leads/เดือน |
| Pro 100K | $89.95/เดือน | 100,000 | เมื่อมีรายได้แล้ว |

> **สรุป: เริ่มด้วย Free plan ได้เลย ไม่ต้องใส่บัตรเครดิต**

---

## Step-by-Step: ตั้งค่า SendGrid สำหรับ Project นี้

### Step 1 — สมัคร Account

1. ไปที่ **[sendgrid.com](https://sendgrid.com)**
2. คลิก **Start for Free**
3. กรอก:
   - Email address (ใช้เป็น admin account)
   - Password
   - Company name (ใช้ชื่อ project ได้)
4. Verify email ที่ SendGrid ส่งมา
5. ตอบคำถาม onboarding (เลือก "Developer" + "Transactional email")

---

### Step 2 — Verify Sender Identity (บังคับ)

SendGrid ต้องการ verify ว่า email ที่จะส่ง *จาก* นั้นเป็นของคุณจริง มี 2 แบบ:

#### แบบ A: Single Sender Verification (ง่าย — แนะนำตอนเริ่ม)

ใช้ได้แม้เป็น Gmail หรือ email ทั่วไป

1. SendGrid dashboard → **Settings → Sender Authentication**
2. คลิก **Verify a Single Sender**
3. กรอกข้อมูล:
   ```
   From Name:    ชื่อบริษัทหรือแบรนด์ของคุณ  (เช่น "AI Automation Co")
   From Email:   อีเมลที่จะใช้ส่ง             (เช่น yourname@gmail.com)
   Reply To:     อีเมลเดียวกันหรืออีเมลอื่น
   ```
4. SendGrid จะส่ง verification email ไปที่ `From Email`
5. เปิดอีเมลนั้น → คลิก **Verify Single Sender** ✅

#### แบบ B: Domain Authentication (แนะนำถ้ามี domain แล้ว)

ถ้ามี domain เช่น `yourcompany.com` — deliverability ดีกว่ามาก

1. SendGrid → **Settings → Sender Authentication → Authenticate Your Domain**
2. เลือก DNS provider (Cloudflare, Namecheap, GoDaddy ฯลฯ)
3. SendGrid จะให้ DNS records 3 ตัว (CNAME) → เพิ่มใน DNS ของ domain
4. รอ propagate (5–30 นาที) → คลิก Verify ✅

---

### Step 3 — สร้าง API Key

1. SendGrid dashboard → **Settings → API Keys**
2. คลิก **Create API Key**
3. ตั้งชื่อ: `n8n-ai-company-sales`
4. เลือก Permission: **Restricted Access**
   - **Mail Send** → Full Access ✅
   - อื่นๆ ไม่จำเป็น → ปล่อยไว้ No Access
5. คลิก **Create & View**
6. **Copy API key ทันที** — จะแสดงครั้งเดียวเท่านั้น

```
ตัวอย่าง API key format:
SG.xxxxxxxxxxxxxxxxxxxxxx.yyyyyyyyyyyyyyyyyyyyyyyyyyyyyyy
```

> ⚠️ ถ้าลืม copy → ต้องสร้าง key ใหม่ (key เดิมดูไม่ได้อีก)

---

### Step 4 — ตั้ง Environment Variables ใน n8n

1. n8n → **Settings** (ซ้ายล่าง) → **Environment Variables**
2. เพิ่มตัวแปรต่อไปนี้:

| Variable Name | ค่า | ตัวอย่าง |
|---|---|---|
| `SENDGRID_API_KEY` | API key จาก Step 3 | `SG.xxxx...` |
| `SENDGRID_FROM_EMAIL` | email ที่ verify แล้วใน Step 2 | `yourname@gmail.com` |
| `COMPANY_NAME` | ชื่อที่จะแสดงใน From field | `AI Automation Co` |

3. บันทึก → **Restart n8n** (ถ้าทำผ่าน Docker: `docker restart n8n`)

---

### Step 5 — ทดสอบก่อน Activate Sales Agent

ใน n8n เปิด `03_Sales_Agent` → **Execute Workflow** (manual) → ดู output:

```json
{
  "success": true,
  "to": "test@example.com",
  "subject": "Quick idea for you",
  "from": "yourname@gmail.com"
}
```

ถ้า success → Activate workflow ได้เลย ✅

---

## การทำงานใน 03_Sales_Agent

`Tool: Send Email` ใน workflow นี้ใช้ **Code node** (ไม่ใช่ n8n SendGrid node) ที่เรียก SendGrid API โดยตรง:

```
Sales Agent ตัดสินใจส่ง email
    │
    ▼
Tool: Send Email (Code node)
    │  อ่าน SENDGRID_API_KEY จาก $env
    │  อ่าน SENDGRID_FROM_EMAIL จาก $env
    │  อ่าน COMPANY_NAME จาก $env
    │
    ├── validate: to_email + body ต้องมี
    ├── POST https://api.sendgrid.com/v3/mail/send
    └── return { success: true, to, subject, from }
```

Agent จะเรียก tool นี้พร้อม input:
```json
{
  "to_email": "prospect@company.com",
  "subject": "Quick automation idea for [Company]",
  "body": "Hi [Name], saw your post about..."
}
```

---

## Deliverability Best Practices

สิ่งที่ระบบทำอัตโนมัติอยู่แล้ว (ใน system prompt):
- ✅ Message ไม่เกิน 150 คำ
- ✅ Value-first — ให้ tip ก่อน pitch
- ✅ ตรวจ CRM history ก่อนส่ง (ไม่ duplicate)
- ✅ Personalized ตาม pain point ของ prospect

สิ่งที่คุณต้องทำเอง:
- ✅ Verify sender domain (ถ้ามี) — เพิ่ม SPF/DKIM records
- ✅ ไม่ส่งมากกว่า 50 email/วันในช่วงแรก (warm up IP)
- ✅ ตรวจ bounce rate ใน SendGrid dashboard ทุกสัปดาห์ (เป้า < 5%)

---

## Monitor & Maintain

### SendGrid Dashboard ที่ต้องดู

| Section | ดูอะไร | ความถี่ |
|---|---|---|
| **Activity Feed** | email ที่ส่งแต่ละฉบับ — delivered / bounced / opened | ทุกสัปดาห์ |
| **Stats** | Delivery rate, Open rate, Bounce rate, Spam report | ทุกสัปดาห์ |
| **Bounces** | email address ที่ bounce → ลบออกจาก CRM | ทุกสัปดาห์ |
| **Spam Reports** | ถ้ามี → หยุด workflow ทันทีเพื่อตรวจสอบ | ทันทีที่เห็น |

### KPIs ที่ควร track

| Metric | เป้าหมาย | ถ้าต่ำกว่า → Action |
|---|---|---|
| Delivery rate | > 95% | ตรวจ domain authentication |
| Bounce rate | < 5% | ปรับ lead quality filter |
| Spam report rate | < 0.1% | แก้ outreach message ทัน |
| Open rate | > 20% | ปรับ subject line ใน system prompt |

---

## Troubleshooting

| ปัญหา | สาเหตุ | วิธีแก้ |
|---|---|---|
| `403 Forbidden` | API key ไม่มี Mail Send permission | สร้าง key ใหม่ + เพิ่ม Mail Send access |
| `Sender not verified` | from-email ไม่ได้ verify | SendGrid → Sender Authentication → verify |
| Email ถูก spam | Domain ไม่มี SPF/DKIM | ตั้ง Domain Authentication |
| `SENDGRID_API_KEY is not set` | env var ไม่ได้ตั้งใน n8n | n8n → Settings → Environment Variables |
| Account suspended | ส่งมากเกินไปหรือ spam rate สูง | ติดต่อ SendGrid support + ลด volume |
| Bounce rate > 10% | Lead email address ไม่ valid | เพิ่ม email validation ก่อนส่ง |

---

## Links ที่เกี่ยวข้อง

- [SendGrid Sign Up](https://sendgrid.com)
- [SendGrid API Keys](https://app.sendgrid.com/settings/api_keys)
- [Sender Authentication](https://app.sendgrid.com/settings/sender_auth)
- [Activity Feed (email logs)](https://app.sendgrid.com/email_activity)
- [SendGrid Mail Send API Docs](https://docs.sendgrid.com/api-reference/mail-send/mail-send)
