# 04 Finance Agent — Weekly Revenue Tracking & Reporting

| Property | Value |
|---|---|
| **File** | `04_Finance_Agent.json` |
| **Trigger** | Schedule — ทุกวันศุกร์ เวลา 18:00 |
| **Version** | 1.1.0 |
| **Status** | Activate ตั้งแต่วันแรก (ก่อน Marketing/Sales เพื่อ track costs) |

---

## วัตถุประสงค์

Finance Agent ทำหน้าที่เป็น **CFO อัตโนมัติ** รายงานสถานะการเงินทุกสัปดาห์ ดึงข้อมูล revenue จาก Stripe, ดึง LLM costs จาก OpenAI, เปรียบเทียบกับสัปดาห์ก่อน และส่ง alerts ถ้ามีสิ่งผิดปกติ บันทึกผลลัพธ์ใน Google Sheets 2 ที่พร้อมกัน

---

## การทำงาน (Node-by-Node Flow)

```
Schedule Trigger (ทุกศุกร์ 18:00)
    │
    ▼
Set Finance Context
    │  กำหนด report_prompt (week ending date), week_end timestamp
    ▼
Finance Agent  ←── OpenAI GPT-4o (temp: 0.1, max 3000 tokens)
    │           ←── Finance Memory (session: "finance_agent_memory")
    │           ←── Tool: Stripe Revenue (GET /v1/charges — 7 วันล่าสุด)
    │           ←── Tool: API Cost Tracking (GET /v1/organization/usage/completions)
    │           ←── Tool: Historical Reports (Google Sheets → Financial_Reports)
    │
    │  Output JSON: weeklyRevenue, weeklyExpenses, netProfit,
    │               revenueByStream, expenseBreakdown, forecast, alerts, recommendations
    ▼
Save Financial Report
    │  append ลง Google Sheets → Financial_Reports sheet
    ▼
Update KPI Dashboard
    │  update Google Sheets → KPI_Dashboard sheet (last_updated + latest_report)
    ▼
[จบ — CEO Agent จะอ่าน KPI_Dashboard ในวันจันทร์ถัดไป]
```

---

## Tools ที่ใช้

| Tool | ประเภท | หน้าที่ |
|---|---|---|
| `get_stripe_transactions` | HTTP (Stripe API) | ดึง charges 100 รายการล่าสุดใน 7 วัน |
| `get_openai_usage` | HTTP (OpenAI Admin API) | ดึง token usage + cost ช่วง 7 วัน (`/v1/organization/usage/completions`) |
| `read_previous_reports` | HTTP (Google Sheets API) | ดึง report สัปดาห์ก่อนเพื่อ trend comparison |

---

## Output Report Structure (JSON)

```json
{
  "weeklyRevenue": 1500,
  "weeklyExpenses": {
    "llmApi": 18.50,
    "infrastructure": 6.00,
    "sendgrid": 0,
    "total": 24.50
  },
  "netProfit": 1475.50,
  "revenueByStream": {
    "freelance": 1000,
    "content": 300,
    "affiliate": 200
  },
  "forecast": {
    "nextWeek": 1700,
    "month4": 6000,
    "trend": "growing +13%"
  },
  "alerts": [],
  "recommendations": ["Scale content output to 2x to reach $3000/mo by month 3"]
}
```

---

## Alert Conditions (built-in ใน system prompt)

| Condition | Alert |
|---|---|
| Expenses > 30% of revenue | ⚠️ Cost ratio too high |
| LLM cost > $20/day | 🚨 API cost spike |
| Revenue < previous week by > 20% | ⚠️ Revenue declining |

---

## Credentials ที่ต้องตั้ง

| Credential | ใช้ที่ | วิธีตั้ง |
|---|---|---|
| OpenAI API | `OpenAI GPT-4o` + `Tool: API Cost Tracking` | n8n → Credentials → OpenAI API |
| Google Sheets OAuth2 | `Save Financial Report`, `Update KPI Dashboard`, `Tool: Historical Reports` | n8n → Credentials → Google Sheets OAuth2 |
| Stripe Secret Key (Header Auth) | `Tool: Stripe Revenue` | n8n → Credentials → HTTP Header Auth → `Authorization: Bearer sk_live_...` — ดู [SETUP_Stripe.md](SETUP_Stripe.md) |
| `REPLACE_WITH_SPREADSHEET_ID` | ทุก Google Sheets node | เปลี่ยนใน node parameters |

---

## Google Sheets Structure ที่ต้องมี

| Sheet Name | Columns | ใช้โดย |
|---|---|---|
| `Financial_Reports` | `week_end`, `report` (JSON), `generated_at` | Finance Agent → append |
| `KPI_Dashboard` | `last_updated`, `latest_report` | Finance Agent → update / CEO Agent → read |

---

## วิธีการใช้งาน

1. ตั้ง Stripe credential (Header Auth: `Authorization: Bearer sk_live_YOUR_KEY`)
2. ตั้ง OpenAI + Google Sheets credentials
3. Replace `REPLACE_WITH_SPREADSHEET_ID` ทุกที่
4. สร้าง sheets `Financial_Reports` และ `KPI_Dashboard`
5. **Activate ก่อน Marketing และ Sales** — เพื่อให้มี baseline cost tracking ตั้งแต่ต้น
6. รอถึงวันศุกร์ หรือ Execute manual เพื่อทดสอบ

> **Note:** ช่วงแรกที่ยังไม่มี Stripe transactions — Agent จะ generate report พร้อม `weeklyRevenue: 0` ซึ่งเป็นปกติ

---

## การทำงานร่วมกับ Workflow อื่น

```
03_Sales_Agent
    │  ปิด deal → รับเงินผ่าน Stripe
    ▼
04_Finance_Agent (ทุกศุกร์)
    │  ดึง Stripe data → คำนวณ P&L
    │
    ├── อัปเดต KPI_Dashboard
    │       │
    │       ▼
    │   01_CEO_Orchestrator (จันทร์ถัดไป)
    │       │  อ่าน KPI → วางแผนสัปดาห์ใหม่
    │
    └── อัปเดต Financial_Reports
            │
            ▼
        05_SelfImprovement_Agent (อาทิตย์)
            │  ดึง Financial_Reports → วิเคราะห์ trend → เสนอปรับ strategy
```

---

## Audit Checklist (ดูทุกวันศุกร์หลัง 18:00)

- [ ] มี record ใหม่ใน `Financial_Reports` sheet
- [ ] `KPI_Dashboard` มี `last_updated` สัปดาห์นี้
- [ ] `weeklyRevenue` ตรงกับ Stripe dashboard (ตรวจ manually เป็นระยะ)
- [ ] `weeklyExpenses.llmApi` สมเหตุสมผล (ไม่ spike ผิดปกติ)
- [ ] ถ้ามี `alerts` → อ่านและ assess เอง
- [ ] `forecast` trend direction สอดคล้องกับ activity จริง

---

## Troubleshooting

| ปัญหา | สาเหตุ | วิธีแก้ |
|---|---|---|
| Stripe 401 error | Secret key ผิดหรือใช้ test key | ดู Stripe dashboard → API Keys → ใช้ live key สำหรับ production |
| OpenAI usage data ว่าง | Admin API ต้องการ Org-level key | สร้าง API key ที่มี `org:read` permission |
| `Financial_Reports` ไม่ append | Sheet ไม่มีหรือ SpreadsheetID ผิด | ตรวจ SpreadsheetID และ sheet name ให้ตรงกัน |
| Revenue ไม่ตรงกับ Stripe | Stripe time zone หรือ payout status ต่างกัน | ตรวจ `created[gte]` timestamp calculation |
| Report JSON ไม่ valid | Temperature 0.1 ยังมี hallucination | เพิ่ม strict output schema ใน system prompt |

---

## Human Support — สิ่งที่คุณต้องทำ

| ความถี่ | งาน | เวลา |
|---|---|---|
| **ทุกศุกร์** | เปิด `Financial_Reports` sheet → อ่าน `alerts` + `recommendations` | 15 นาที |
| **ทุกศุกร์** | เปรียบ Stripe dashboard กับ `weeklyRevenue` ใน report | 5 นาที |
| **ทุกเดือน** | Export `Financial_Reports` เป็น backup | 5 นาที |
| **ถ้า alert ขึ้น** | ตรวจสอบและ action ภายใน 24 ชั่วโมง | ตามสถานการณ์ |
| **ตั้ง spending limit** | OpenAI dashboard → Billing → Usage limits → ตั้ง $50/month หรือตามที่เห็นสมควร | ครั้งเดียว |
