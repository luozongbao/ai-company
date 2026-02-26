# 05 Self-Improvement Agent — Weekly Audit & System Evolution

| Property | Value |
|---|---|
| **File** | `05_SelfImprovement_Agent.json` |
| **Trigger** | Schedule — ทุกวันอาทิตย์ เวลา 00:00 (เที่ยงคืน) |
| **Version** | 1.0.0 |
| **Status** | Activate พร้อมกับ `04_Finance_Agent` ตั้งแต่ต้น |

---

## วัตถุประสงค์

Self-Improvement Agent คือ **ระบบภูมิคุ้มกันของบริษัท** ทำงานทุกสัปดาห์เพื่อ:
- Audit health ของทุก workflow: Content Agent, Product Agent, Fulfillment Agent, Finance, CEO
- วิเคราะห์ KPI performance: page views, newsletter subscribers, product sales, affiliate clicks
- ตรวจ content quality: affiliate links per post, keyword optimization, CTA effectiveness
- ตรวจ product quality: completeness, pricing, duplicate detection
- หา system gaps ที่ยังไม่ได้ทำ
- เสนอ improvement proposals แบบ prioritized (Priority 1/2/3)
- ค้นหา AI tools ใหม่ที่อาจเป็นประโยชน์
- Review business model — grow, maintain, หรือ pivot

> **ขอบเขต v2.0:** ระบบนี้ audit 3 revenue streams: Affiliate, Newsletter Subscription, Digital Products

Output ทั้งหมดบันทึกลง `Improvement_Log` sheet รอ **human review** ก่อน apply

---

## การทำงาน (Node-by-Node Flow)

```
Schedule Trigger (ทุกอาทิตย์ 00:00)
    │
    ▼
Set Audit Context
    │  กำหนด audit_prompt (week ending date), audit_date
    ▼
Self-Improvement Agent  ←── OpenAI GPT-4o (temp: 0.2, max 5000 tokens)
    │                   ←── Improvement Memory (session: "self_improvement_memory")
    │                   ←── Tool: n8n Execution Logs (n8n API /api/v1/executions)
    │                   ←── Tool: All KPI Data (Google Sheets → KPI_Dashboard)
    │                   ←── Tool: Research New Tools (Brave Search — pw freshness)
    │
    │  Output JSON (7 sections — ดูด้านล่าง)
    ▼
Save Improvement Report
    │  append ลง Google Sheets → Improvement_Log sheet
    │  status: "pending_review"
    ▼
[จบ — รอ human อ่าน และ apply Priority 1 changes]
```

---

## Tools ที่ใช้

| Tool | ประเภท | หน้าที่ |
|---|---|---|
| `get_n8n_execution_logs` | HTTP (n8n Admin API) | ดึง execution history 100 รายการล่าสุด — สถานะ success/error |
| `get_all_kpi_data` | HTTP (Google Sheets API) | ดึง KPI ทุกแผนกจาก KPI_Dashboard |
| `research_new_ai_tools` | HTTP (Brave Search) | ค้นหา AI tools, n8n updates, automation techniques ใหม่ประจำสัปดาห์ |

---

## Output Report Structure (JSON — 7 sections)

```json
{
  "workflowHealth": {
    "contentAgent": { "successRate": 0.95, "avgExecutionMs": 12000, "errors": [] },
    "productAgent": { "successRate": 1.0, "errors": [] },
    "fulfillmentAgent": { "successRate": 0.98, "errors": [] },
    "financeAgent": { "successRate": 1.0, "errors": [] },
    "ceoOrchestrator": { "successRate": 1.0, "errors": [] }
  },
  "performanceSummary": {
    "contentPublished": 7,
    "productsCreated": 1,
    "ordersDelivered": 4,
    "newsletterSubscribers": 82,
    "affiliateClicks": 143,
    "revenueThisWeek": 1500
  },
  "systemGaps": [
    "No LinkedIn direct posting capability",
    "Stripe dispute handling not automated"
  ],
  "improvementProposals": [
    {
      "problem": "Content Agent generates content but blog post not linked to Stripe product page",
      "solution": "Add WordPress REST API node after Ghost draft creation",
      "impact": "Increase organic traffic 2x in 2 months",
      "effort": "low",
      "risk": "low",
      "priority": 1
    }
  ],
  "promptOptimizations": [],
  "businessModelReview": {
    "affiliate": "growing — 143 clicks this week",
    "newsletterSubscription": "flat — increase CTA frequency in posts",
    "digitalProducts": "new — 1 product created, 4 sold",
    "recommendation": "grow all 3 streams; prioritize newsletter CTA optimization"
  },
  "priorityActions": ["Apply WordPress API integration this week", "Fix SendGrid bounce issue"]
}
```

---

## Priority System

| Priority | ความหมาย | ใครทำ | เมื่อไหร่ |
|---|---|---|---|
| **1** | ต้องทำสัปดาห์นี้ — impact สูง, effort ต่ำ | Human (คุณ) + apply ใน n8n | ภายใน 7 วัน |
| **2** | ทำเดือนหน้า — สำคัญแต่ไม่เร่ง | Human review + plan | เดือนหน้า |
| **3** | Future idea — worth noting | บันทึกไว้ใน backlog | อนาคต |

> **สำคัญ:** ระบบไม่ auto-apply changes ใดๆ — ทุก proposal ต้องผ่าน human approval เสมอ

---

## Credentials ที่ต้องตั้ง

| Credential | ใช้ที่ | วิธีตั้ง |
|---|---|---|
| OpenAI API | `OpenAI GPT-4o` | n8n → Credentials → OpenAI API |
| Google Sheets OAuth2 | `Tool: All KPI Data`, `Save Improvement Report` | n8n → Credentials → Google Sheets OAuth2 |
| Brave Search Header Auth | `Tool: Research New Tools` | HTTP Header Auth: `X-Subscription-Token` — ดู [SETUP_BraveSearch.md](SETUP_BraveSearch.md) |
| n8n API Key (Header Auth) | `Tool: n8n Execution Logs` | n8n Settings → n8n API → Create key → ใช้ Header Auth |

---

## Google Sheets Structure ที่ต้องมี

| Sheet Name | Columns | ใช้โดย |
|---|---|---|
| `KPI_Dashboard` | `last_updated`, `latest_report` | Finance Agent → write, Self-Improvement → read |
| `Improvement_Log` | `audit_date`, `full_report` (JSON), `status` | Self-Improvement → append |

---

## วิธีการใช้งาน

1. ตั้ง credentials ทั้งหมด
2. สร้าง `Improvement_Log` sheet
3. สร้าง `N8N_API_KEY` ที่ n8n Settings → n8n API → Create API Key
4. ตั้ง Header Auth credential ด้วย key นั้น → ผูกกับ `Tool: n8n Execution Logs`
5. Activate workflow

> **ทดสอบ:** Execute manual หลังจาก Content Agent + Product Agent + Finance Agent รันไปแล้วอย่างน้อย 1 รอบ เพื่อให้มีข้อมูลให้วิเคราะห์

---

## การทำงานร่วมกับ Workflow อื่น

```
01_CEO_Orchestrator ──┐
02_Content_Agent  ──┤  ทุกสัปดาห์สร้าง execution logs + KPI data
03_Product_Agent      ──┤
04_Finance_Agent    ──┘
                        │
                        ▼
              05_SelfImprovement_Agent
                        │  อ่าน logs ทั้งหมด → audit → เสนอปรับปรุง
                        │
                        ▼
                   Improvement_Log (Google Sheets)
                        │
                        ▼
                   Human Review (คุณ)
                        │
                        ▼
                   Apply changes → 01/02/03/04 workflows ดีขึ้น
```

---

## Audit Checklist (ดูทุกวันอาทิตย์หรือจันทร์เช้า)

- [ ] มี record ใหม่ใน `Improvement_Log` sheet
- [ ] `workflowHealth` ครบทุก workflow (contentAgent, productAgent, fulfillmentAgent, financeAgent, ceoOrchestrator)
- [ ] `successRate` ทุก workflow > 80% (ถ้าต่ำกว่า: action ทันที)
- [ ] `improvementProposals` มี priority ชัดเจน
- [ ] อ่าน `priorityActions` → implement Priority 1 ภายในสัปดาห์นี้
- [ ] `businessModelReview` — stream ไหน flat หรือ declining?

---

## Troubleshooting

| ปัญหา | สาเหตุ | วิธีแก้ |
|---|---|---|
| `Tool: n8n Execution Logs` auth error | n8n API key ผิดหรือไม่ได้ตั้ง | n8n Settings → n8n API → verify key → update credential |
| KPI data ว่างเปล่า | Finance Agent ยังไม่รัน | รัน `04_Finance_Agent` ก่อน อย่างน้อย 1 ครั้ง |
| Report มีแค่ guesses ไม่มีข้อมูลจริง | Execution logs + KPI ยังว่าง | รอให้ทุก workflow รันอย่างน้อย 1 สัปดาห์ก่อน audit |
| maxTokens 5000 ไม่พอ | Report ยาวมาก / หลาย workflows | เพิ่ม maxTokens หรือ แบ่ง audit เป็นส่วนๆ |

---

## Human Support — สิ่งที่คุณต้องทำ

| ความถี่ | งาน | เวลา |
|---|---|---|
| **ทุกอาทิตย์/จันทร์** | เปิด `Improvement_Log` → อ่าน `priorityActions` + Priority 1 proposals | 15–20 นาที |
| **ทุกอาทิตย์** | Implement Priority 1 changes ใน n8n (แก้ node, prompt, หรือ add tool) | 10–30 นาที |
| **ทุกเดือน** | Review Priority 2 proposals ว่าจะทำอะไรบ้าง | 15 นาที |
| **ทุกเดือน** | Export `Improvement_Log` เป็น backup + commit JSON ไปยัง Git | 10 นาที |
| **ถ้า successRate < 50%** | หยุด workflow ที่มีปัญหา → debug → fix ก่อน re-activate | ทันที |
