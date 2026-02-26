# AI Content Media & Digital Products Company
> **Project Vision:** บริษัทที่ขับเคลื่อนด้วย AI + n8n Workflow 100% สร้าง passive income จาก content, newsletter, และ digital products โดยไม่ต้องมีมนุษย์ operate

---

## Business Model

บริษัทนี้หารายได้จาก **3 ช่องทางที่ AI ทำได้ครบ loop** — ตั้งแต่ผลิตจนถึง collect เงิน โดยไม่ต้องมีคนแตะเลย:

| # | Revenue Stream | รายได้ประมาณ/เดือน | AI ทำได้ 100%? |
|---|---|---|---|
| 1 | **Affiliate Marketing** ฝังใน blog/social content | $200–2,000 | ✅ AI เขียน + publish + ฝัง link |
| 2 | **Newsletter Subscription** (WordPress paid tier $9/mo) | $500–5,000 | ✅ AI เขียน + WordPress publish อัตโนมัติ |
| 3 | **Digital Products** (prompt packs, n8n templates, AI guides) | $300–3,000 | ✅ AI สร้าง + Stripe + AI deliver ทางอีเมล |

**ทำไมถึงเลือก 3 model นี้?**
- ไม่ต้องสต็อกสินค้า — ไม่มี physical goods
- AI ผลิตสินค้าได้ (GPT-4o เขียน/generate ทุกอย่าง)
- AI deliver ได้ (WordPress, Buffer, SendGrid ทำงานผ่าน API)
- รายได้ passive — เงินเข้าแม้ระบบไม่ได้กำลัง "ทำงาน"

---

## 🗺️ Overall System Architecture

```
Human Owner (คุณ)
  Setup ครั้งแรก / Weekly 30-min Review

        ↓
  00_SEED_Strategy    ← Manual (รันครั้งเดียว)
  วิเคราะห์ niche + affiliate + product lineup

        ↓
  01_CEO_Orchestrator (ทุกจันทร์ 07:00)
  วางแผน content themes + product ประจำสัปดาห์

        ↓                        ↓
02_Content_Agent         03_Product_Agent
(ทุกวัน 08:00)           (ทุกอังคาร 09:00)
Blog + Social +          สร้าง digital
Affiliate links          product ใหม่

        ↓                        ↓
     WordPress + Buffer          WordPress post
     Affiliate revenue       Product_Catalog

             [ลูกค้าจ่ายเงิน Stripe]
                      ↓
        06_Fulfillment_Agent (Stripe Webhook 24/7)
        AI fetch product → SendGrid deliver ทันที

                      ↓
        04_Finance_Agent (ทุกศุกร์ 18:00)
        P&L: Stripe + affiliate + DeepSeek costs

                      ↓
        05_SelfImprovement (ทุกอาทิตย์ 00:00)
        Audit + propose improvements

  99_SHUTDOWN_Graceful (Manual)
  Emergency kill-switch
```

---

## Weekly Operational Cycle

| เวลา | Workflow | สิ่งที่เกิดขึ้น |
|---|---|---|
| จันทร์ 07:00 | CEO Orchestrator | อ่าน KPI → กำหนด content themes + product ประจำสัปดาห์ |
| ทุกวัน 08:00 | Content Agent | Research → เขียน blog + affiliate links → publish WordPress + Buffer |
| อังคาร 09:00 | Product Agent | Research demand → สร้าง digital product → WordPress post + log Catalog |
| ตลอด 24/7 | Fulfillment Agent | รับ Stripe webhook → fetch product → SendGrid deliver ทันที |
| ศุกร์ 18:00 | Finance Agent | Stripe + affiliate + DeepSeek costs → P&L → KPI Dashboard |
| อาทิตย์ 00:00 | Self-Improvement | Audit workflows → เสนอ improvements → log Improvement_Log |

---

## Data Flow — Google Sheets เป็นศูนย์กลาง

```
KPI_Dashboard     ◄── Finance (เขียน)  ──► CEO, Self-Imp (อ่าน)
Weekly_Plans      ◄── CEO (เขียน)
Content_Log       ◄── Content Agent (เขียน)
Product_Catalog   ◄── Product Agent (เขียน) ──► Fulfillment (อ่าน)
Order_Log         ◄── Fulfillment Agent (เขียน)
Financial_Reports ◄── Finance Agent (เขียน)
Affiliate_Income  ◄── Manual entry (คุณ) ──► Finance (อ่าน)
Improvement_Log   ◄── Self-Improvement (เขียน)
Strategy_Decisions◄── SEED (เขียน)
```

---

## Workflow Files

| ไฟล์ | Workflow | Trigger | หน้าที่ | README |
|---|---|---|---|---|
| `00_SEED_Strategy.json` | SEED: Niche Research | Manual (1x) | เลือก niche + affiliate + product lineup | [📄](00_SEED_Strategy_README.md) |
| `01_CEO_Orchestrator.json` | CEO Orchestrator | จันทร์ 07:00 | วางแผน content themes + product สัปดาห์นี้ | [📄](01_CEO_Orchestrator_README.md) |
| `02_Content_Agent.json` | Content Agent | ทุกวัน 08:00 | Blog + Social + Affiliate publishing | [📄](02_Content_Agent_README.md) |
| `03_Product_Agent.json` | Product Agent | อังคาร 09:00 | สร้าง digital products รายสัปดาห์ | [📄](03_Product_Agent_README.md) |
| `04_Finance_Agent.json` | Finance Agent | ศุกร์ 18:00 | P&L: Stripe + affiliate + DeepSeek costs | [📄](04_Finance_Agent_README.md) |
| `05_SelfImprovement_Agent.json` | Self-Improvement | อาทิตย์ 00:00 | Audit + improvement proposals | [📄](05_SelfImprovement_Agent_README.md) |
| `06_Fulfillment_Agent.json` | Fulfillment Agent | Stripe Webhook (24/7) | Auto-deliver digital products via email | [📄](06_Fulfillment_Agent_README.md) |
| `99_SHUTDOWN_Graceful.json` | Graceful Shutdown | Manual | ปิดทุก Agent + final report | [📄](99_SHUTDOWN_Graceful_README.md) |

---

## Revenue Loop — วิธีที่เงินเข้าจริง

```
Content Agent เขียน blog post ที่ฝัง affiliate link
       ↓
มีคน click affiliate link → ซื้อ tool
       ↓
Affiliate commission เข้าบัญชีอัตโนมัติ

OR ลูกค้าสมัคร newsletter paid tier $9/mo
       ↓
Stripe subscription เก็บเงินอัตโนมัติทุกเดือน

OR ลูกค้าซื้อ digital product
       ↓
Stripe payment → Webhook → 06_Fulfillment
       ↓
AI fetch product → SendGrid deliver ภายใน 1 นาที
ไม่มีมนุษย์เข้ามาเลยตั้งแต่ต้นจนจบ
```

---

## Setup Guide

### Prerequisites

```
- n8n self-hosted (Docker บน VPS)
- DeepSeek API Key (GPT-4o)
- Brave Search API Key (free 2,000 req/mo)
- Google Sheets + OAuth2
- Stripe Account
- SendGrid Account (verified sender)
- WordPress
- Buffer Account (Optional)
```

### Step 1 — Install n8n

```bash
docker run -d --restart unless-stopped \
  --name n8n -p 5678:5678 \
  -v ~/.n8n:/home/node/.n8n \
  -e N8N_BASIC_AUTH_ACTIVE=true \
  -e N8N_BASIC_AUTH_USER=admin \
  -e N8N_BASIC_AUTH_PASSWORD=your_strong_password \
  docker.n8n.io/n8nio/n8n
```

### Step 2 — สร้าง Google Sheets

สร้าง Spreadsheet 1 ไฟล์ แล้วเพิ่ม sheets ดังนี้:

| Sheet Name | ข้อมูล | ใครเขียน |
|---|---|---|
| `Strategy_Decisions` | niche/product decisions | SEED |
| `KPI_Dashboard` | KPI ภาพรวม | Finance |
| `Weekly_Plans` | แผนรายสัปดาห์ | CEO |
| `Content_Log` | blog/social posts | Content Agent |
| `Product_Catalog` | digital products ที่สร้างแล้ว | Product Agent |
| `Order_Log` | orders fulfilled | Fulfillment Agent |
| `Financial_Reports` | P&L รายสัปดาห์ | Finance Agent |
| `Affiliate_Income` | affiliate commission (manual) | คุณ |
| `Improvement_Log` | audit reports | Self-Improvement |

### Step 3 — ตั้ง Credentials ใน n8n

**n8n Credentials (Settings → Credentials):**

| Credential | ประเภท | ใช้โดย |
|---|---|---|
| OpenAI API | OpenAI API | ทุก Agent |
| Google Sheets OAuth2 | Google Sheets OAuth2 | ทุก Agent |
| Brave Search | HTTP Header Auth: X-Subscription-Token | Content, CEO, SEED — ดู [SETUP_BraveSearch.md](SETUP_BraveSearch.md) |
| Stripe | HTTP Header Auth: Authorization Bearer sk_live_... | Finance — ดู [SETUP_Stripe.md](SETUP_Stripe.md) |
| Buffer | HTTP Header Auth: Authorization Bearer TOKEN | Content Agent |
| WordPress Admin | HTTP Header Auth: Authorization WordPress ADMIN_KEY | Content + Product |

**n8n Environment Variables (Settings → Environment Variables):**

| Variable | ค่า | ใช้โดย |
|---|---|---|
| `N8N_API_KEY` | สร้างใน n8n → Settings → API | Shutdown workflow |
| `SENDGRID_API_KEY` | ดู [SETUP_SendGrid.md](SETUP_SendGrid.md) | Fulfillment Agent |
| `SENDGRID_FROM_EMAIL` | verified sender email | Fulfillment Agent |
| `COMPANY_NAME` | ชื่อแบรนด์ของคุณ | Fulfillment Agent |
| `WORDPRESS_URL` | https://your-ghost-blog.com | Content + Product |
| `WORDPRESS_APP_PASSWORD` | WordPress → Settings → Integrations | Product Agent |
| `BUFFER_TWITTER_PROFILE_ID` | Buffer dashboard → Channels | Content Agent |
| `REPLACE_WITH_SPREADSHEET_ID` | Google Sheet ID จาก URL | ทุก workflow JSON |

### Step 4 — Stripe Webhook

1. Stripe Dashboard → Developers → Webhooks → Add endpoint
2. URL: `https://YOUR_N8N_URL/webhook/stripe-webhook`
3. Event: `checkout.session.completed`
4. ตั้ง metadata บน Stripe product:
   ```json
   { "product_type": "prompt_pack", "product_title": "50 ChatGPT Prompts for Freelancers" }
   ```

### Step 5 — Activate Order

```
Day 1  → Run 00_SEED_Strategy (manual, 1x)
Day 2  → Activate 01_CEO_Orchestrator
Day 2  → Activate 04_Finance_Agent       (track costs ตั้งแต่ต้น)
Day 3  → Activate 05_SelfImprovement
          Activate 06_Fulfillment_Agent  (webhook 24/7)
Week 1 → Activate 02_Content_Agent      (ดู quality 3 วัน)
Week 2 → Activate 03_Product_Agent
```

---

## 💰 Budget Guide

### One-time Setup (~$47)

| รายการ | ค่าใช้จ่าย |
|---|---|
| VPS เดือนแรก (Hetzner CX22) | $6 |
| Domain name | $12 |
| OpenAI credit initial | $20 |
| WordPress Pro (optional, can self-host) | $9 |
| **รวม** | **~$47** |

### Monthly Burn Rate

| รายการ | ประหยัด | ปกติ |
|---|---|---|
| VPS Server | $6 | $12 |
| OpenAI API | $5–10 | $15–30 |
| Brave Search | $0 | $0 |
| SendGrid | $0 | $0 |
| WordPress | $0 (self-host) | $9 |
| Buffer | $0 | $18 |
| Domain | $1 | $1 |
| **รวม** | **$12–17** | **$55–70** |

> จุดคุ้มทุน: newsletter subscriber 2 คน ($9/mo) = คืน server cost แล้ว

---

## Setup Guides

| Service | ใช้ใน | Guide |
|---|---|---|
| Brave Search API | SEED, CEO, Content, Self-Improvement | [SETUP_BraveSearch.md](SETUP_BraveSearch.md) |
| SendGrid | Fulfillment Agent | [SETUP_SendGrid.md](SETUP_SendGrid.md) |
| Stripe | Finance + Fulfillment | [SETUP_Stripe.md](SETUP_Stripe.md) |

---

## Kill Switch

```bash
# Emergency
docker stop n8n

# Graceful: รัน 99_SHUTDOWN_Graceful workflow → final report → deactivate all
```

---

## 🗓️ Human Maintenance

### รายสัปดาห์ (~30 นาที)

| วัน | งาน | เวลา |
|---|---|---|
| จันทร์ | ดู CEO Weekly Plan ใน Weekly_Plans sheet | 10 นาที |
| พุธ | สุ่มตรวจ blog — affiliate links ถูกต้อง? | 10 นาที |
| ศุกร์ | ดู Finance Report — revenue + alerts | 10 นาที |
| อาทิตย์ | อ่าน Self-Improvement proposals → approve Priority 1 | 15 นาที |

### รายเดือน (~1 ชั่วโมง)

| งาน | รายละเอียด |
|---|---|
| Monthly P&L | Stripe + Financial_Reports รวมเดือน |
| Apply improvements | ใส่ Priority-1 ใน n8n |
| Add affiliate income | กรอก commission เข้า Affiliate_Income sheet |
| Backup | Export JSON → git commit |
| Top-up OpenAI | ถ้า balance < $5 |

### สัญญาณอันตราย

| สถานการณ์ | Action |
|---|---|
| n8n error > 20% ติดต่อกัน 2 วัน | Executions tab → แก้ error |
| DeepSeek cost > $5/วัน ช่วงแรก | ตั้ง spending limit ใน OpenAI |
| Content มี misleading claims | Deactivate Content Agent → แก้ prompt |
| Fulfillment webhook ไม่ทำงาน | ตรวจ Stripe webhook events ใน n8n |

---

## Ethical Guardrails

ทุก Agent มี FORBIDDEN list ใน system prompt:
- ห้าม spam / phishing / black-hat tactics
- ห้าม misleading affiliate claims
- ทุก affiliate recommendation ต้องเหมาะกับ context จริงๆ
- ห้ามสร้าง content ที่ผิดกฎหมาย

---

## 12-Month Revenue Projection (Conservative)

| เดือน | MRR Target | หมายเหตุ |
|---|---|---|
| 1–2 | $0 | Setup + content building |
| 3–4 | $50–200 | Affiliate เริ่มมา + subscriber แรก |
| 5–6 | $200–500 | Newsletter + digital products แรก |
| 7–9 | $500–1,500 | 3 streams เริ่มทำงานพร้อมกัน |
| 10–12 | $1,500–4,000 | Compound growth |
| Year 2 | $4,000–15,000/mo | Scale ด้วย catalog สะสม + traffic |

---

*Version 2.0 — Rewritten for fully autonomous digital business*
*Business model: Content Media + Digital Products (fully AI-executable)*
