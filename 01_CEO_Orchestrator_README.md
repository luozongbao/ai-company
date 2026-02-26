# 01 CEO Orchestrator — Weekly Planning & Coordination

| Property | Value |
|---|---|
| **File** | `01_CEO_Orchestrator.json` |
| **Trigger** | Schedule — ทุกวันจันทร์ เวลา 07:00 |
| **Version** | 1.0.0 |
| **Status** | Activate หลังจากรัน `00_SEED_Strategy` แล้ว |

---

## วัตถุประสงค์

CEO Agent คือ **ศูนย์กลางการตัดสินใจ** ของระบบ ทำงานทุกต้นสัปดาห์เพื่อ:
- ดึง KPI สัปดาห์ที่แล้วจาก Google Sheets
- รับข้อมูล market intelligence + trending topics ใหม่
- กำหนด content themes ประจำสัปดาห์สำหรับ Content Agent
- ระบุ product opportunity สำหรับ Product Agent
- บันทึก weekly plan ไว้ใน Google Sheets สำหรับ audit และ tracking

---

## การทำงาน (Node-by-Node Flow)

```
Schedule Trigger (ทุกจันทร์ 07:00)
    │
    ▼
Set Weekly Context
    │  กำหนด: week_number, current_date, planning_prompt
    ▼
CEO Agent  ←── OpenAI GPT-4o (temp: 0.4, max 3000 tokens)
    │      ←── CEO Memory (session: "ceo_agent_memory")
    │      ←── Tool: Read KPI Reports (Google Sheets → KPI_Dashboard)
    │      ←── Tool: Market Intelligence (Brave Search API)
    │
    │  Output JSON: weekNumber, priorities, departmentPlans, riskFlags, revenueTarget
    ▼
Format Weekly Plan
    │  รวม weekly_plan + week + generated_at
    ▼
Save to Google Sheets (sheet: Weekly_Plans)
    │
    ▼
[จบ — แผนรายสัปดาห์บันทึกแล้ว Agents อื่นจะ execute ตาม schedule ของตัวเอง]
```

---

## Tools ที่ใช้

| Tool | ประเภท | หน้าที่ |
|---|---|---|
| `read_department_reports` | HTTP (Google Sheets API) | ดึง KPI ของสัปดาห์ที่แล้วทุกแผนก |
| `get_market_intelligence` | HTTP (Brave Search API) | ค้นหา trending market news สำหรับ context การวางแผน |

---

## Input / Output

### Input
- Google Sheets `KPI_Dashboard` — ข้อมูล KPI ที่ Finance Agent Update ไว้
- Brave Search results — market trends ประจำสัปดาห์

### Output (JSON บันทึกลง `Weekly_Plans` sheet)
```json
{
  "weekNumber": 9,
  "contentThemes": ["AI automation for freelancers", "Prompt engineering tips", "n8n tutorials"],
  "agentTasks": {
    "contentAgent": { "topicsCount": 7, "affiliateFocus": "OpenAI API tools", "newsletterCTA": "..." },
    "productAgent": { "productType": "prompt_pack", "targetTopic": "ChatGPT for marketers", "pricePoint": 19 }
  },
  "revenueTarget": 1500,
  "trendingOpportunity": "GPT-4o tool integrations gaining traction",
  "riskFlags": []
}
```

---

## Credentials ที่ต้องตั้ง

| Credential | ใช้ที่ | วิธีตั้ง |
|---|---|---|
| OpenAI API | `OpenAI GPT-4o` node | n8n → Credentials → OpenAI API |
| Google Sheets OAuth2 | `Tool: Read KPI Reports`, `Save to Google Sheets` | n8n → Credentials → Google Sheets OAuth2 |
| Brave Search Header Auth | `Tool: Market Intelligence` | HTTP Header Auth: `X-Subscription-Token` — ดู [SETUP_BraveSearch.md](SETUP_BraveSearch.md) |
| `REPLACE_WITH_SPREADSHEET_ID` | Google Sheets nodes | เปลี่ยนใน node parameters ทุกตัว |

---

## วิธีการใช้งาน

1. Import workflow + ตั้ง credentials ให้ครบ
2. Replace `REPLACE_WITH_SPREADSHEET_ID` ด้วย ID ของ Google Sheet จริง
3. สร้าง sheet ชื่อ `KPI_Dashboard` และ `Weekly_Plans` ใน Spreadsheet
4. **Activate** workflow — ระบบจะรันอัตโนมัติทุกวันจันทร์ 07:00

> **ทดสอบก่อน activate:** คลิก Execute ใน n8n เพื่อดูว่า Agent generate plan ได้ถูกต้อง

---

## การทำงานร่วมกับ Workflow อื่น

```
00_SEED_Strategy ──→ [CEO เริ่มต้น] ──→ 01_CEO_Orchestrator
                                               │
               ┌───────────────────────────────┤
               │  อ่าน KPI จาก Finance Report  │
               ▼                              │
04_Finance_Agent ──→ อัปเดต KPI_Dashboard ───┘

CEO Plan อ่านโดย:
├── 02_Content_Agent (Content Agent)  — รู้ว่าต้องทำ content themes อะไรสัปดาห์นี้
├── 03_Product_Agent (Product Agent)      — รู้ว่า product type / topic ที่ควร launch
└── 05_SelfImprovement                  — รู้ว่า CEO prioritize อะไรเพื่อ audit ตรงจุด
```

> **หมายเหตุ:** Agent แผนกอื่นยังไม่ได้อ่าน `Weekly_Plans` sheet โดยตรงในปัจจุบัน — เป็น improvement ที่แนะนำให้เพิ่มในเฟสถัดไป

---

## Audit Checklist (ดูทุกวันจันทร์หลัง 07:00)

- [ ] มี record ใหม่ใน Google Sheets `Weekly_Plans` sheet
- [ ] `departmentPlans` มีแผนครบทุก 4 แผนก
- [ ] `revenueTarget` สมเหตุสมผล (ไม่สูงหรือต่ำผิดปกติ)
- [ ] `riskFlags` ว่างเปล่า หรือถ้ามีให้อ่านและ assess เอง
- [ ] Execution log ไม่มี error (n8n → Executions)

---

## Troubleshooting

| ปัญหา | สาเหตุ | วิธีแก้ |
|---|---|---|
| ไม่มี record ใน Weekly_Plans | Workflow ไม่ได้ active | n8n → Workflows → toggle Active ON |
| Google Sheets error | OAuth2 token หมดอายุ | n8n → Credentials → Google Sheets → Reconnect |
| CEO Plan ไม่ครบ 4 แผนก | KPI_Dashboard ว่างเปล่า | ตรวจสอบว่า Finance Agent รันแล้วหรือยัง |
| Agent loop / timeout | Context window เต็ม (maxTokens 3000) | ลด `planning_prompt` หรือเพิ่ม maxTokens |

---

## Human Support — สิ่งที่คุณต้องทำ

| ความถี่ | งาน | เวลา |
|---|---|---|
| **ทุกจันทร์** | เปิด Google Sheets `Weekly_Plans` → อ่าน plan → confirm หรือ flag ความผิดปกติ | 10 นาที |
| **รายเดือน** | ตรวจสอบว่า revenue targets มีความสมเหตุสมผลเพิ่มขึ้นตาม growth | 5 นาที |
| **ถ้า plan ผิดพลาด** | แก้ system prompt ใน `CEO Agent` node → เพิ่ม constraints หรือ context | เมื่อพบปัญหา |

---

> **v2.0 Note:** CEO Orchestrator ใน v2 นี้ coordinate **Content Media + Digital Products** business — ไม่มี Marketing department หรือ Sales department แยก แต่ทำงานผ่าน Content Agent (blog/affiliate) และ Product Agent (digital products) แทน
