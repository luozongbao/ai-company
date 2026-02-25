# 03 Sales Agent — Daily Lead Generation & Outreach

| Property | Value |
|---|---|
| **File** | `03_Sales_Agent.json` |
| **Trigger** | Schedule — ทุกวัน เวลา 09:00 |
| **Version** | 1.1.0 |
| **Status** | Activate หลังจาก `02_Marketing_Agent` ผ่าน QC อย่างน้อย 1 สัปดาห์แล้ว |

---

## วัตถุประสงค์

Sales Agent ทำหน้าที่ **หาลูกค้าใหม่และส่ง outreach อัตโนมัติ** ทุกวัน เป้าหมาย 10 leads/วัน ผ่านการค้นหาบน Reddit, Twitter, LinkedIn แล้วเขียน email ที่ personalised และส่งผ่าน SendGrid โดยทุก message ต้องเป็น value-first ไม่ใช่ spam

---

## การทำงาน (Node-by-Node Flow)

```
Schedule Trigger (ทุกวัน 09:00)
    │
    ▼
Set Sales Context
    │  กำหนด sales_prompt, target_leads_count = 10
    ▼
Sales Agent  ←── OpenAI GPT-4o (temp: 0.5, max 3500 tokens)
    │         ←── Sales Memory (session: "sales_agent_memory")
    │         ←── Tool: Find Leads (Brave Search — site:reddit.com, freshness: pw)
    │         ←── Tool: Send Email (toolCode → SendGrid API)
    │         ←── Tool: CRM History (Google Sheets → CRM sheet)
    │
    │  Output JSON: leadsFound, outreachMessages, followUps, dailySummary
    ▼
Log to CRM Sheet (Google Sheets → CRM)
    │  บันทึก: date, daily_summary, leads_found count, outreach_sent count
    ▼
[จบ — Sales Agent จะ call Tool: Send Email ระหว่าง agent loop]
```

---

## Tools ที่ใช้

| Tool | ประเภท | หน้าที่ |
|---|---|---|
| `find_leads` | HTTP (Brave Search) | ค้นหา lead บน Reddit/Twitter — freshness: past week, count: 15 |
| `send_outreach_email` | Code (toolCode) | ส่ง email จริงผ่าน SendGrid API (อ่าน from-address จาก env vars) |
| `check_crm_history` | HTTP (Google Sheets API) | ตรวจว่าเคย contact prospect คนนี้ไปแล้วหรือยัง — ป้องกัน duplicate |

---

## Services ที่ขาย (ตามที่กำหนดใน system prompt)

| Service | ราคา |
|---|---|
| n8n Workflow Setup | $500–2,000/project |
| AI Agent Monthly Retainer | $300–1,000/month |
| Content Automation Package | $200–500/month |
| Custom Automation Consulting | $150/hour |

---

## Environment Variables ที่ต้องตั้ง (สำคัญมาก)

| Variable | ค่า | หน้าที่ |
|---|---|---|
| `SENDGRID_API_KEY` | API key จาก SendGrid | ใช้ในการส่ง email จริง |
| `SENDGRID_FROM_EMAIL` | verified sender email ของคุณ | ปรากฏใน From field |
| `COMPANY_NAME` | ชื่อบริษัท/แบรนด์ | ปรากฏใน From name |

> ตั้งที่: n8n → Settings → Environment Variables
>
> 📄 **ดู setup guide ฉบับเต็ม:** [SETUP_SendGrid.md](SETUP_SendGrid.md)

---

## Credentials ที่ต้องตั้ง

| Credential | ใช้ที่ | วิธีตั้ง |
|---|---|---|
| OpenAI API | `OpenAI GPT-4o` | n8n → Credentials → OpenAI API |
| Google Sheets OAuth2 | `Tool: CRM History`, `Log to CRM Sheet` | n8n → Credentials → Google Sheets OAuth2 |
| Brave Search Header Auth | `Tool: Find Leads` | HTTP Header Auth: `X-Subscription-Token` — ดู [SETUP_BraveSearch.md](SETUP_BraveSearch.md) |

---

## Google Sheets CRM Structure

สร้าง sheet ชื่อ `CRM` ใน Spreadsheet พร้อม columns:

| Column | ข้อมูล |
|---|---|
| `date` | วันที่ log |
| `daily_summary` | JSON output จาก Sales Agent |
| `leads_found` | จำนวน leads ที่หาได้ |
| `outreach_sent` | จำนวน email ที่ส่ง |

---

## วิธีการใช้งาน

1. **ตั้ง env vars ก่อน** (`SENDGRID_API_KEY`, `SENDGRID_FROM_EMAIL`, `COMPANY_NAME`)
2. ตั้ง credentials (OpenAI, Google Sheets, Brave)
3. สร้าง `CRM` sheet ใน Spreadsheet
4. **Verify sender email** ใน SendGrid dashboard ก่อน activate (สำคัญมาก — ไม่เช่นนั้นส่ง email ไม่ได้)
5. Execute manual ครั้งแรก — **ตรวจดู outreach messages ก่อน activate**
6. ถ้า message quality ดี → Activate workflow

> ⚠️ **คำเตือน:** ตรวจ `leadsFound` และ `outreachMessages` ใน output ก่อน activate เพื่อให้แน่ใจว่า Agent ไม่ส่ง spam

---

## ตัวอย่าง Outreach Email ที่ดี (ที่ Agent ควร generate)

```
Subject: Quick automation idea for your business

Hi [Name],

Saw your post about spending hours on manual data entry —
that's exactly what AI automation solves.

Here's one free tip: n8n + a simple webhook can cut that
process from 2 hours to 5 minutes automatically.

Happy to walk you through the full setup — no pitch, just help.
Worth a 15-min chat?

Best,
[COMPANY_NAME]
```

---

## การทำงานร่วมกับ Workflow อื่น

```
02_Marketing_Agent
    │  สร้าง content → build credibility → ทำให้ lead อยากตอบ outreach
    ▼
03_Sales_Agent
    │  อ่าน CRM history (ไม่ส่ง duplicate)
    │  ส่ง outreach → รอ reply จาก prospect
    │  บันทึก results ลง CRM sheet
    ▼
04_Finance_Agent
    │  ดึงรายได้จาก Stripe ที่มาจาก closed deals
    ▼
05_SelfImprovement_Agent
    │  วิเคราะห์ leads_found trend, outreach_sent vs deals closed
    │  เสนอปรับ target คุณลักษณะ lead หรือ offer
```

---

## Audit Checklist (ดูทุกสัปดาห์)

- [ ] `CRM` sheet มี record ใหม่ทุกวัน
- [ ] `leads_found` เฉลี่ย ≥ 5/วัน (ถ้าต่ำกว่า: ปรับ search query)
- [ ] `outreach_sent` ≥ 1/วัน (ถ้าเป็น 0: ตรวจ SendGrid connection)
- [ ] ไม่มี duplicate เลข email เดิมใน CRM ติดกัน 2 วัน
- [ ] outreach messages ใน daily_summary ไม่ใช่ generic template
- [ ] SendGrid dashboard: bounce rate < 5%, spam report = 0

---

## Troubleshooting

| ปัญหา | สาเหตุ | วิธีแก้ |
|---|---|---|
| Email ไม่ส่ง / error 403 | `SENDGRID_FROM_EMAIL` ไม่ได้ verify | SendGrid → Sender Authentication → verify domain/email |
| `leads_found` = 0 ทุกวัน | Brave Search ไม่พบ results | ปรับ search query ใน `find_leads` tool description |
| Email ถูก flag เป็น spam | From domain ไม่มี SPF/DKIM | ตั้ง DNS records ตาม SendGrid guide |
| `SENDGRID_API_KEY not set` error | env var ไม่ได้ตั้ง | n8n → Settings → Environment Variables → เพิ่ม key |
| CRM มี lead ซ้ำ | `check_crm_history` ไม่ match correctly | ตรวจ sheet structure ว่ามี column ที่ใช้ match ครบ |

---

## Human Support — สิ่งที่คุณต้องทำ

| ความถี่ | งาน | เวลา |
|---|---|---|
| **ทุกวัน (ช่วงแรก)** | ตรวจ CRM sheet ดู outreach quality + รับ reply จาก prospect ด้วยตัวเอง | 10–15 นาที |
| **ทุกสัปดาห์** | ดู conversion rate: leads → reply → deal | 10 นาที |
| **เมื่อมี reply** | **ตอบ reply เองหรือ setup email auto-responder** (Agent ส่งครั้งแรกได้ แต่ follow-up conversation ยังต้องมนุษย์) | ตามสถานการณ์ |
| **รายเดือน** | Review SendGrid deliverability report | 10 นาที |
| **ถ้าถูก block/ban** | หยุด workflow ทันที → ตรวจสอบปัญหา → แก้ system prompt | เมื่อพบ |
