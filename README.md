# AI Company — One Click Business
> **Project Vision:** บริษัทที่ขับเคลื่อนด้วย AI Agent + n8n Workflow 100% สามารถหารายได้ พัฒนาตัวเอง และดำเนินธุรกิจได้โดยไม่ต้องมี Human in the loop

---

## Feasibility Assessment ✅

| ความสามารถ | เป็นไปได้? | หมายเหตุ |
|---|---|---|
| AI Agent แต่ละแผนกทำงานอัตโนมัติ | ✅ | n8n AI Agent node + LLM |
| Seed Workflow หาธุรกิจที่เหมาะสม | ✅ | Web research + LLM analysis |
| สร้าง Workflow ธุรกิจใหม่ได้เอง | ✅ | n8n API + Code generation |
| Self-assessment & self-improvement | ✅ | Monitoring + feedback loop |
| หารายได้จริง (Freelance, SaaS, Content) | ✅ | ต้องมี payment gateway |
| ไม่ผิดจริยธรรม / กฎหมาย | ✅ | กำหนด guardrails ไว้ใน system prompt |
| ปิดระบบได้ทุกเวลา | ✅ | Kill-switch workflow |

---

## Organization Architecture

```
CEO Agent (Orchestrator)
├── Strategy Agent          ← วิเคราะห์ตลาด, ตัดสินใจทิศทางธุรกิจ
├── Operations Agent        ← ควบคุมการทำงานภายใน, จัดการ Workflow
├── Marketing Agent         ← สร้าง Content, SEO, Social Media, Email
├── Sales Agent             ← Lead gen, Outreach, Follow-up
├── Product/Dev Agent       ← สร้างผลิตภัณฑ์ / Service deliverable
├── Finance Agent           ← ติดตามรายได้, จัดการค่าใช้จ่าย, รายงาน
├── HR/Talent Agent         ← (เฟส 2) Hire freelancers, manage outsource
└── Self-Improvement Agent  ← Monitor KPI, audit workflow, propose upgrades
```

---

## Business Model ที่เลือก (AI-Executable)

เลือกธุรกิจที่ **AI + n8n ทำได้ 100%** โดยไม่ต้องใช้มนุษย์:

### Primary Revenue Streams
| # | โมเดลธุรกิจ | รายได้ประมาณ/เดือน | AI ทำได้ไหม |
|---|---|---|---|
| 1 | **AI Content & Newsletter** (Substack/Ghost) | $500–3,000 | ✅ สร้าง + publish อัตโนมัติ |
| 2 | **Faceless YouTube / TikTok** (video automation) | $300–5,000 | ✅ script + voiceover + edit |
| 3 | **AI Freelance Services** (Fiverr/Upwork — copywriting, SEO, data) | $1,000–8,000 | ✅ LLM + templates |
| 4 | **Niche Affiliate / SEO Blog** | $500–4,000 | ✅ content pipeline |
| 5 | **Workflow-as-a-Service** (ขาย n8n workflow ให้คนอื่น) | $1,000–10,000 | ✅ product ที่บริษัทนี้สร้างเอง |

### Launch Order
1. **เฟส 0 (เดือน 1):** Seed → Strategy Agent วิเคราะห์ตลาด → เลือก top 2 streams
2. **เฟส 1 (เดือน 1–3):** Content + Freelance → สร้างรายได้เริ่มต้น
3. **เฟส 2 (เดือน 3–6):** YouTube automation + SEO Blog
4. **เฟส 3 (เดือน 6–12):** Workflow-as-a-Service + Scale ด้วยรายได้จากเฟสก่อน
5. **เฟส 4 (ปีที่ 2+):** Self-funded expansion + hire AI-assisted freelancers

---

## Step-by-Step: วิธีเริ่มต้น

### Prerequisites
```
- n8n self-hosted (Docker บน Server ของคุณ)
- LLM API Key: OpenAI GPT-4o หรือ Anthropic Claude
- Database: PostgreSQL (สำหรับ memory + logging)
- Storage: S3 หรือ local folder (สำหรับไฟล์งาน)
- Email: SMTP / SendGrid (Marketing Agent)
- Payment: Stripe account (รับเงิน)
- Social accounts: Twitter/X, LinkedIn, YouTube (ตั้งไว้ล่วงหน้า)
```

### Step 1 — ติดตั้ง n8n
```bash
docker run -it --rm \
  --name n8n \
  -p 5678:5678 \
  -v ~/.n8n:/home/node/.n8n \
  -e N8N_BASIC_AUTH_ACTIVE=true \
  -e N8N_BASIC_AUTH_USER=admin \
  -e N8N_BASIC_AUTH_PASSWORD=your_password \
  docker.n8n.io/n8nio/n8n
```
เปิด `http://localhost:5678` → Import `original organization.json`

### Step 2 — กำหนด Credentials
ใน n8n → Settings → Credentials เพิ่ม:
- `OpenAI API` (หรือ Anthropic)
- `PostgreSQL`
- `SendGrid` / Gmail SMTP
- `HTTP Header Auth` สำหรับ Stripe API
- `Google Sheets` (Financial tracking)

### Step 3 — Import Workflow ทั้งหมดเข้า n8n

ใน n8n → Workflows → Import from File → import ตามลำดับ:

| ไฟล์ | Workflow | ทริกเกอร์ | หน้าที่ | README |
|------|----------|-----------|---------|--------|
| `00_SEED_Strategy.json` | Seed: Market Research | Manual (รันครั้งแรกครั้งเดียว) | วิเคราะห์ตลาด เลือกธุรกิจ | [📄](00_SEED_Strategy_README.md) |
| `01_CEO_Orchestrator.json` | CEO Orchestrator | จันทร์ 07:00 | วางแผนรายสัปดาห์ | [📄](01_CEO_Orchestrator_README.md) |
| `02_Marketing_Agent.json` | Marketing Agent | ทุกวัน 08:00 | สร้าง Content ทุกช่องทาง | [📄](02_Marketing_Agent_README.md) |
| `03_Sales_Agent.json` | Sales Agent | ทุกวัน 09:00 | หาลูกค้า ส่ง Outreach | [📄](03_Sales_Agent_README.md) |
| `04_Finance_Agent.json` | Finance Agent | ศุกร์ 18:00 | รายงานการเงินรายสัปดาห์ | [📄](04_Finance_Agent_README.md) |
| `05_SelfImprovement_Agent.json` | Self-Improvement Agent | อาทิตย์ 00:00 | Audit + เสนอการปรับปรุง | [📄](05_SelfImprovement_Agent_README.md) |
| `99_SHUTDOWN_Graceful.json` | Graceful Shutdown | Manual (ปิดระบบ) | ปิดทุก Agent + สรุปผล | [📄](99_SHUTDOWN_Graceful_README.md) |

### Step 4 — รัน Seed Workflow ก่อน
Workflow แรกที่ต้องรัน: **"00_SEED_Strategy"**
- Strategy Agent จะ research 10 business ideas
- วิเคราะห์ automation feasibility, competition, revenue potential
- output: `business_decision.json` → เลือก top 2 models
- Activate workflows ที่เกี่ยวข้องอัตโนมัติ

### Step 5 — Activate Workflows และปล่อยให้ระบบทำงาน
หลัง Seed workflow เสร็จ Activate workflows ทั้งหมดแล้วแต่ละ Agent จะ trigger ตาม schedule:
```
CEO Agent          — ทุกวันจันทร์  07:00  (weekly planning)
Marketing Agent    — ทุกวัน        08:00  (content publish)
Sales Agent        — ทุกวัน        09:00  (outreach)
Finance Agent      — ทุกวันศุกร์   18:00  (weekly report)
Self-Improvement   — ทุกวันอาทิตย์ 00:00  (weekly audit)
```

### Step 6 — Monitor Dashboard
Finance Agent สร้าง Google Sheet รายงานอัตโนมัติ:
- รายได้รายสัปดาห์ / รายเดือน
- Workflow error rate
- KPI ของแต่ละ Agent
- Self-Improvement recommendations

---

## Kill Switch — วิธีปิดระบบ

### Emergency Stop (ทันที)
```bash
# ปิด n8n container ทันที
docker stop n8n
```

### Graceful Shutdown (แนะนำ)
- รัน Workflow **"99_SHUTDOWN_Graceful"** ใน n8n
- ระบบจะ: หยุด schedule ทั้งหมด → บันทึก state → export รายงานสุดท้าย → ปิด

### Deactivate เฉพาะ Agent
- n8n → Workflows → toggle "Active" ปิดเฉพาะแผนกที่ต้องการ

---

## Self-Improvement Loop

```
ทุกสัปดาห์ (Self-Improvement Agent):
1. ดึง logs จาก PostgreSQL
2. วิเคราะห์ error rate, revenue per workflow, bottleneck
3. สร้าง improvement report
4. Propose: ปรับ prompt / เพิ่ม tool / สร้าง workflow ใหม่
5. รอ Human approval (หรือ auto-approve ถ้า risk score ต่ำ)
6. Apply changes → version control
```

---

## 12-Month Projection (Conservative)

| เดือน | Revenue Target | Action |
|---|---|---|
| 1 | $0 (setup) | Seed + install + test |
| 2–3 | $200–500 | Content + Freelance เริ่ม |
| 4–6 | $1,000–3,000 | YouTube + Blog เพิ่ม traffic |
| 7–9 | $3,000–8,000 | Scale + Workflow-as-a-Service launch |
| 10–12 | $8,000–20,000 | Multi-stream consolidated |
| **Year 2** | **$20,000–50,000/mo** | Self-funded, self-scaling |

---

## Tech Stack

| Layer | Tool |
|---|---|
| Automation | n8n (self-hosted) |
| LLM | OpenAI GPT-4o / Claude 3.5 Sonnet |
| Memory | PostgreSQL + n8n memory nodes |
| Storage | Local / S3 |
| Web scraping | Browserless / Playwright (via HTTP node) |
| Email | SendGrid |
| Payments | Stripe |
| Analytics | Google Sheets (simple) / Metabase (advanced) |
| Version control | Git (workflow JSON export) |

---

## Ethical Guardrails (Built-in)

ทุก Agent มี system prompt ที่ระบุ:
```
FORBIDDEN ACTIONS:
- ห้ามส่ง spam หรือ phishing
- ห้ามสร้าง content เท็จ / misleading
- ห้ามทำธุรกรรมที่ผิดกฎหมาย
- ห้าม access ข้อมูลส่วนตัวโดยไม่ได้รับอนุญาต
- ทุกการใช้จ่ายเงิน > $50 ต้องรายงาน Finance Agent ก่อน
```

---

## ✅ Workflow Audit — ความสอดคล้องกับ About.md

### ผลการตรวจสอบ (Alignment Check)

| หัวข้อจาก About.md | Workflow ที่รองรับ | สถานะ | หมายเหตุ |
|---|---|---|---|
| AI Agent ทุกแผนกทำงาน 100% | 01–05 ทุก Workflow | ✅ ผ่าน | ครบทุก Agent มี LLM + Tools |
| Seed Workflow หาธุรกิจก่อน | `00_SEED_Strategy` | ✅ ผ่าน | วิเคราะห์ + จัดอันดับ Business Model |
| หารายได้ได้จริง | Marketing + Sales + Finance | ✅ ผ่าน | Stripe + Email outreach พร้อม |
| เช็คตัวเอง + พัฒนาได้ต่อเนื่อง | `05_SelfImprovement` | ✅ ผ่าน | Weekly audit + improvement proposals |
| ถูกต้องตามกฎหมาย/จริยธรรม | ทุก Agent (system prompt) | ✅ ผ่าน | FORBIDDEN list กำหนดไว้ทุก Agent |
| ปิดระบบได้ทุกเวลา | `99_SHUTDOWN_Graceful` | ✅ ผ่าน | Manual kill-switch + final report |
| ไม่ต้องมี Human in the loop | ทุก Workflow (schedule-based) | ✅ ผ่าน | Self-triggering ตาม cron schedule |

**สรุป: Workflow ครอบคลุมทุกข้อใน About.md ครบถ้วน ✅**

---

### ✅ Known Gaps — ช่องว่างที่พบและแก้ไขแล้ว

| Workflow | ปัญหาที่พบ | สถานะ | การแก้ไข |
|---|---|---|---|
| **02 Marketing** | Generate content แต่ไม่ได้ publish จริง (แค่ log ไป Google Sheets) | ✅ แก้แล้ว | เพิ่ม node `Parse Content for Publishing` → `Publish via Buffer` (Twitter/LinkedIn) + `Publish to Ghost Blog` (newsletter draft) |
| **02 Marketing** | ไม่มี Ghost / Substack publishing node | ✅ แก้แล้ว | เพิ่ม `Publish to Ghost Blog` HTTP node — POST ไปยัง Ghost Admin API `/ghost/api/admin/posts/` เป็น draft อัตโนมัติ |
| **03 Sales** | `send_outreach_email` tool มี `hello@yourcompany.com` hardcoded — AI ไม่สามารถรู้ from-address จริง | ✅ แก้แล้ว | เปลี่ยนเป็น `toolCode` ที่อ่าน from-address จาก env vars: `SENDGRID_FROM_EMAIL`, `SENDGRID_API_KEY`, `COMPANY_NAME` |
| **03 Sales** | ไม่มี LinkedIn / Upwork API integration สำหรับ outreach | ⚠️ ยังคงอยู่ | LinkedIn API ถูก restrict โดย LinkedIn; ใช้ Phantombuster/Dux-Soup หรือ manual post ก่อน |
| **04 Finance** | OpenAI `/v1/usage` endpoint deprecated | ✅ แก้แล้ว | เปลี่ยนเป็น `/v1/organization/usage/completions` พร้อม `start_time`/`end_time` Unix timestamp params |
| **05 Self-Improvement** | Output เป็น `pending_review` ไม่มี auto-apply | ✅ เจตนาดี | Human oversight คือ design decision — อย่า auto-apply high-risk changes |
| **99 Shutdown** | JS node แค่ return instructions ไม่ได้ call n8n API จริง | ✅ แก้แล้ว | เปลี่ยนเป็น `this.helpers.httpRequest` จริง — GET workflow list → PATCH `{active: false}` ทุก workflow อัตโนมัติ |

> **Credentials เพิ่มเติมที่ต้องตั้งหลังการแก้ไข:**
> - `SENDGRID_API_KEY` — n8n → Settings → Environment Variables
> - `SENDGRID_FROM_EMAIL` — อีเมล verified sender ใน SendGrid
> - `COMPANY_NAME` — ชื่อบริษัท (ใช้ใน email from-name)
> - `BUFFER_TWITTER_PROFILE_ID` — ดูจาก Buffer dashboard → Channels
> - `GHOST_API_URL` — URL ของ Ghost instance (เช่น `https://your-blog.com`)
> - Buffer credential (HTTP Header Auth: `Authorization: Bearer BUFFER_ACCESS_TOKEN`)
> - Ghost credential (HTTP Header Auth: `Authorization: Ghost YOUR_ADMIN_API_KEY`)

---

## 💰 Budget & Investment Guide

### งบประมาณเริ่มต้น (One-time Setup Cost)

| รายการ | ค่าใช้จ่าย | จำเป็น? | หมายเหตุ |
|---|---|---|---|
| **VPS/Server** (เดือนแรก) | $5–8 | ✅ จำเป็น | Hetzner CX22 (~€4.35/mo) หรือ DigitalOcean $6/mo |
| **Domain name** | $10–15/ปี (~$1/เดือน) | แนะนำ | สำหรับ branding + email ที่น่าเชื่อถือ |
| **OpenAI API credit** (initial top-up) | $20 | ✅ จำเป็น | ซื้อ credit ไว้ก่อนเพื่อไม่ให้ interrupt |
| **Brave Search API** | $0 | ✅ จำเป็น | Free tier 2,000 queries/month เพียงพอช่วงแรก |
| **Google Sheets** | $0 | ✅ จำเป็น | ใช้ Google account ปกติได้เลย |
| **Stripe account** | $0 | ✅ จำเป็น | ฟรี เปิดบัญชี; จ่ายเมื่อมีรายได้ (2.9% + $0.30/tx) |
| **SendGrid** | $0 | ✅ จำเป็น | Free tier 3,000 email/month เพียงพอช่วงแรก |
| **Social accounts** (Twitter/X, LinkedIn, YouTube) | $0 | แนะนำ | สร้างล่วงหน้าก่อนเปิด Sales + Marketing |
| **Buffer** (ถ้าต้องการ auto-post social จริง) | $0–18/month | ⚠️ Optional | Free: 3 channels/10 posts; Essentials: $6/channel/mo |
| **Ghost** (newsletter, ถ้าต้องการ) | $0–9/month | ⚠️ Optional | Ghost(Pro) Starter $9/mo หรือ self-host บน server เดิม |
| **เวลาของคุณ (setup)** | ~2–4 ชั่วโมง | ✅ | ตั้งค่า credentials + import + test workflows |

> **งบประมาณเริ่มต้นที่แนะนำ: $50–70 (ครั้งแรก รวมทุกอย่าง)**

---

### ค่าใช้จ่ายรายเดือน (Monthly Burn Rate)

| รายการ | Tier ประหยัด | Tier ปกติ | หมายเหตุ |
|---|---|---|---|
| **Server (Hetzner/DigitalOcean)** | $5 | $8–12 | CX22 หรือ DO Basic Droplet |
| **OpenAI GPT-4o API** | $5–10 | $15–30 | ขึ้นกับ usage จริง; ช่วงแรกต่ำ |
| **Brave Search API** | $0 | $0–5 | Free tier 2K queries/mo เพียงพอช่วงแรก |
| **SendGrid Email** | $0 | $0–20 | Free 3K/mo เพียงพอช่วงแรก |
| **Buffer Social Publishing** | $0 | $18 | ถ้าต้องการ auto-post จริง |
| **Ghost Newsletter** | $0 | $9 | Optional |
| **Domain (pro-rated)** | ~$1 | ~$1 | $10–15/ปี |
| **รวม** | **$11–16/เดือน** | **$55–95/เดือน** | ใช้ tier ประหยัดช่วงแรกได้ |

> **ประมาณการ 3 เดือนแรก (ก่อนมีรายได้): $50–150 รวมทั้งหมด**
> **จุดคุ้มทุน:** Sales Agent ปิด deal ได้ 1 งาน ($500 workflow setup) = คุ้มทุน 3–10 เดือน

---

### ต้นทุนแต่ละส่วน (Investment by Component)

| ส่วน | สิ่งที่ต้องลงทุน | ค่าใช้จ่าย/เดือน |
|---|---|---|
| **Infrastructure** | VPS Server + Domain + SSL (Let's Encrypt ฟรี) | $5–10 |
| **AI Brain (LLM)** | OpenAI GPT-4o API — เพิ่ม/ลดตาม usage | $5–30 |
| **Research & Data** | Brave Search API | $0 (free tier) |
| **Content Distribution** | Buffer (social) + Ghost (newsletter) | $0–27 |
| **Revenue Processing** | Stripe | $0 + 2.9% per sale |
| **Email/Outreach** | SendGrid SMTP | $0 (free tier) |
| **Data & Logging** | Google Sheets + PostgreSQL (on server) | $0 |
| **Version Control** | Git + GitHub (free) | $0 |

---

## 🗓️ Human Maintenance Guide

แม้ระบบจะ autonomous แต่ **คุณยังต้องทำบางอย่าง** เพื่อให้ระบบทำงานได้อย่างมีประสิทธิภาพและปลอดภัย

### รายสัปดาห์ (ใช้เวลา ~30–60 นาที/สัปดาห์)

| วัน | งาน | ที่ไหน | เวลาที่ใช้ |
|---|---|---|---|
| **จันทร์** (หลัง 07:00) | ดู Weekly Plan ที่ CEO Agent สร้าง — ตรวจความสมเหตุสมผล | Google Sheets → `Weekly_Plans` | 10 นาที |
| **จันทร์–ศุกร์** | สุ่มตรวจ content ที่ Marketing generate — หาข้อผิดพลาดหรือเนื้อหาไม่เหมาะสม | Google Sheets → `Content_Log` | 5–10 นาที |
| **ศุกร์** (หลัง 18:00) | ดู Finance Report รายสัปดาห์ — เช็ค revenue, cost, alerts | Google Sheets → `Financial_Reports` | 15 นาที |
| **อาทิตย์** (ตอนเช้า) | ดู Self-Improvement Audit report — review improvement proposals | Google Sheets → `Improvement_Log` | 15–20 นาที |
| **อาทิตย์** | Approve/Reject improvement proposals Priority 1 — apply changes เองใน n8n | n8n + Git | 10–15 นาที |

**สัญญาณอันตรายที่ต้องสังเกตทันที:**
- Error rate > 20% ใน n8n → Executions tab
- OpenAI cost spike ผิดปกติ (>$5/วัน ช่วงแรก) → OpenAI dashboard
- Finance Agent ส่ง alert ใน report (expenses > 30% revenue)
- Content มีข้อมูลผิดพลาด misleading หรือผิดจริยธรรม

---

### รายเดือน (ใช้เวลา ~1–2 ชั่วโมง/เดือน)

| งาน | รายละเอียด | เวลาที่ใช้ |
|---|---|---|
| **Review Monthly P&L** | ดู Stripe dashboard + Google Sheets `Financial_Reports` ครบเดือน | 20 นาที |
| **API Key Management** | ตรวจสอบ OpenAI / Brave / SendGrid keys ยังใช้ได้ ไม่หมดอายุ | 5 นาที |
| **Apply Workflow Improvements** | นำ Priority-1 improvements จาก Self-Improvement Agent มา implement ใน n8n | 30–45 นาที |
| **Content Quality Audit** | สุ่ม review 5–10 content pieces ที่ generate ใน sheet | 15 นาที |
| **Social Account Health Check** | ตรวจว่า Twitter/LinkedIn accounts ไม่ถูก suspend หรือ flag | 5 นาที |
| **Backup Workflows** | Export workflows จาก n8n → commit ไปยัง Git repo | 10 นาที |
| **Stripe/Financial Review** | ดู payout, refund, dispute ใน Stripe Dashboard | 10 นาที |
| **Top-up API Credits** | ถ้า OpenAI credit < $5 ให้ top-up เพื่อป้องกัน workflow interrupt | 5 นาที |

---

### เมื่อไหรที่ต้องเข้าไป Intervene ทันที

| สถานการณ์ | Action |
|---|---|
| Workflow error ติดต่อกัน > 3 วัน | n8n → Executions → ดู error log → แก้ไข |
| AI generate content ผิดพลาด/ไม่เหมาะสม | Deactivate Marketing Agent → แก้ system prompt → Activate ใหม่ |
| ค่าใช้จ่าย API spike ผิดปกติ | OpenAI dashboard → ดู usage → ตั้ง spending limit |
| Sales Agent ส่ง email ที่ไม่เหมาะสม | Deactivate Sales Agent → review outreach template → Activate |
| รายได้หายหรือ Stripe มีปัญหา | Stripe dashboard → ดู events → ติดต่อ Stripe support |
| ต้องการหยุดระบบทั้งหมด | รัน `99_SHUTDOWN_Graceful` workflow → follow instructions ใน output |

---

## 📋 System-wide Operating Instructions

### หลักการสำคัญ (Operating Principles)

```
1. HUMAN IS THE OWNER
   AI ทำงาน แต่คุณเป็นผู้รับผิดชอบทางกฎหมายและจริยธรรมทั้งหมด

2. AUDIT WEEKLY — ไม่ปล่อยระบบทิ้งไว้โดยไม่ดูนาน
   ดู Self-Improvement report ทุกสัปดาห์

3. START SLOW — เปิด workflow ทีละชั้น
   รัน Marketing + Finance ก่อน → verify quality → ค่อยเปิด Sales

4. MONITOR COSTS — ตั้ง Spending Limit ใน OpenAI Dashboard
   แนะนำ: $30/month hard limit ช่วงแรก

5. BACKUP ALWAYS — export JSON ก่อนเปลี่ยนแปลงใดๆ
   git commit ทุกครั้งก่อนแก้ workflow

6. NEVER AUTO-APPROVE HIGH-RISK CHANGES
   improvements ที่ risk = "high" ต้อง manual review เสมอ

7. CONTENT IS YOUR REPUTATION
   ตรวจ content ก่อน publish จริง — AI อาจ hallucinate ได้
```

---

### ลำดับการเปิดระบบที่แนะนำ (Safe Activation Order)

```
วันที่ 1  → รัน 00_SEED_Strategy (manual, ครั้งเดียว) — อ่าน output ก่อน
วันที่ 2  → Activate 01_CEO_Orchestrator
วันที่ 2  → Activate 04_Finance_Agent  (track costs ตั้งแต่วันแรก)
วันที่ 3  → Activate 05_SelfImprovement_Agent
สัปดาห์ 1 → Activate 02_Marketing_Agent (ดู content output 3 วัน ก่อน activate Sales)
สัปดาห์ 2 → Activate 03_Sales_Agent (เมื่อ content quality ผ่านการตรวจแล้ว)
```

---

### Google Sheets Structure ที่ต้องสร้างก่อน Activate

สร้าง Google Spreadsheet 1 ไฟล์ แล้วเพิ่ม sheets ต่อไปนี้:

| Sheet Name | ใช้โดย Workflow | ข้อมูลที่เก็บ |
|---|---|---|
| `KPI_Dashboard` | CEO + Self-Improvement | KPI ภาพรวมทุก Agent |
| `Weekly_Plans` | CEO Orchestrator | Weekly plan รายสัปดาห์ |
| `Content_Log` | Marketing Agent | Content ที่ generate รายวัน |
| `Error_Log` | Marketing Agent | Error logs |
| `CRM` | Sales Agent | Leads + outreach history |
| `Financial_Reports` | Finance Agent | Weekly P&L |
| `Improvement_Log` | Self-Improvement Agent | Audit reports + proposals |
| `Shutdown_Log` | Shutdown Workflow | Final report เมื่อปิดระบบ |

---

### Credentials Checklist (ก่อน Activate workflows)

**n8n Credentials (Settings → Credentials):**
- [ ] `REPLACE_WITH_YOUR_OPENAI_CREDENTIAL_ID` → เปลี่ยนใน n8n Settings → Credentials → OpenAI
- [ ] `REPLACE_WITH_YOUR_GSHEETS_CREDENTIAL_ID` → เปลี่ยนใน n8n Settings → Credentials → Google Sheets OAuth2
- [ ] `REPLACE_WITH_SPREADSHEET_ID` → ใส่ Google Sheet ID จริง (ดูจาก URL: `spreadsheets/d/[ID]/edit`)
- [ ] Brave Search API Key → สมัครที่ [api.search.brave.com](https://api.search.brave.com) → เพิ่มใน n8n Header Auth
- [ ] Stripe Secret Key → ดูที่ [dashboard.stripe.com/apikeys](https://dashboard.stripe.com/apikeys)
- [ ] Buffer credential → HTTP Header Auth: `Authorization: Bearer BUFFER_ACCESS_TOKEN` (ดูที่ buffer.com → Account → API)
- [ ] Ghost credential → HTTP Header Auth: `Authorization: Ghost YOUR_GHOST_ADMIN_API_KEY` (ดูที่ Ghost Admin → Settings → Integrations)

**n8n Environment Variables (Settings → Environment Variables):**
- [ ] `N8N_API_KEY` → สร้างใน n8n Settings → n8n API → Create API Key (ใช้โดย Shutdown workflow)
- [ ] `N8N_HOST` → ตั้งใน Docker: `-e N8N_HOST=http://your-server-ip:5678`
- [ ] `SENDGRID_API_KEY` → จาก [app.sendgrid.com/settings/api_keys](https://app.sendgrid.com/settings/api_keys)
- [ ] `SENDGRID_FROM_EMAIL` → อีเมล verified sender ของคุณใน SendGrid
- [ ] `COMPANY_NAME` → ชื่อบริษัท/แบรนด์ (ปรากฏใน email from-name)
- [ ] `BUFFER_TWITTER_PROFILE_ID` → ดูจาก Buffer dashboard → Channels → Twitter profile ID
- [ ] `GHOST_API_URL` → URL ของ Ghost site (เช่น `https://your-ghost-blog.com`)

---

*Generated: February 2026 | Version: 1.1.0 | Last updated: February 25, 2026*
