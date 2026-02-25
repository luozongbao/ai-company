# 00 SEED Strategy — Market Research & Business Selection

| Property | Value |
|---|---|
| **File** | `00_SEED_Strategy.json` |
| **Trigger** | Manual (รันครั้งเดียวตอน setup) |
| **Version** | 1.0.0 |
| **Status** | Run once — ไม่ต้อง activate เป็น scheduled |

---

## วัตถุประสงค์

Workflow นี้คือ **จุดเริ่มต้นของทั้งระบบ** รันก่อน Workflow อื่นทุกตัว หน้าที่คือให้ Strategy Agent วิเคราะห์ตลาดธุรกิจ AI Automation แล้วเลือก **2 Business Model ที่ดีที่สุด** สำหรับระบบนี้ในการสร้างรายได้

---

## การทำงาน (Node-by-Node Flow)

```
Manual Trigger
    │
    ▼
Set Research Prompt
    │  กำหนด prompt: วิเคราะห์ 10 business models ที่ AI ทำได้ใน 2026
    ▼
Strategy Agent  ←── OpenAI GPT-4o (temp: 0.3)
    │           ←── Window Buffer Memory
    │           ←── Tool: Web Research (Brave Search API)
    │           ←── Tool: Score & Rank (Code — weighted scoring)
    │
    │  Output: JSON วิเคราะห์สมบูรณ์ + top 2 recommendations
    ▼
Prepare Output
    │  เพิ่ม timestamp, status: "seed_complete", next_action
    ▼
Save Strategy Decision
    │  POST ผลลัพธ์ไปยัง webhook หรือ internal endpoint
    ▼
[จบ — ดู output แล้ว activate workflows ที่ต้องการ]
```

---

## Tools ที่ใช้

| Tool | ประเภท | หน้าที่ |
|---|---|---|
| `web_research` | HTTP (Brave Search API) | ค้นหาข้อมูลตลาด, รายได้, competition |
| `score_and_rank` | Code (JavaScript) | คำนวณ composite score: revenue×0.35 + automation×0.35 + speed×0.20 + safety×0.10 |

---

## Input / Output

### Input
- ไม่มี input จากภายนอก — trigger manual โดยผู้ใช้

### Output (JSON)
```json
{
  "strategy_result": "{ ... JSON วิเคราะห์ business models ... }",
  "timestamp": "2026-02-25T07:00:00.000Z",
  "status": "seed_complete",
  "next_action": "Activate 01_CEO_Orchestrator workflow and selected business workflows"
}
```

---

## Credentials ที่ต้องตั้งก่อนรัน

| Credential | ใช้ที่ | วิธีตั้ง |
|---|---|---|
| OpenAI API | `OpenAI GPT-4o` node | n8n → Settings → Credentials → OpenAI API |
| Brave Search Header Auth | `Tool: Web Research` | n8n → Credentials → HTTP Header Auth → `X-Subscription-Token: YOUR_KEY` |

---

## วิธีการใช้งาน (Step-by-Step)

1. Import `00_SEED_Strategy.json` เข้า n8n
2. ตั้ง Credentials (OpenAI + Brave Search) ให้ครบ
3. คลิก **Execute Workflow** (manual run)
4. รอ 1–3 นาที ให้ Agent วิเคราะห์เสร็จ
5. ดู output ใน node `Prepare Output` → อ่าน `strategy_result`
6. ตัดสินใจว่าจะเลือก business model ไหน
7. Activate workflows ที่เกี่ยวข้องตามผลลัพธ์

> **ไม่ต้อง Activate workflow นี้** — รันแบบ manual ครั้งเดียวก็พอ

---

## การทำงานร่วมกับ Workflow อื่น

```
00_SEED_Strategy (รันก่อน)
    │
    │  ผลลัพธ์บอกว่าควรเลือก business model อะไร
    ▼
01_CEO_Orchestrator  ← activate หลังจากอ่าน output
02_Marketing_Agent   ← activate ถ้าเลือก Content/Newsletter model
03_Sales_Agent       ← activate ถ้าเลือก Freelance/Services model
04_Finance_Agent     ← activate เสมอ
05_SelfImprovement   ← activate เสมอ
```

---

## Audit Checklist

- [ ] Agent output มี top 2 business models พร้อม justification
- [ ] แต่ละ model มี: revenue estimate, automation score, risk score, launch plan
- [ ] ไม่มีการแนะนำธุรกิจผิดกฎหมาย/ผิดจริยธรรม
- [ ] `status` = `"seed_complete"` ใน output
- [ ] ผลลัพธ์บันทึกผ่าน `Save Strategy Decision` สำเร็จ (ดู execution log)

---

## Troubleshooting

| ปัญหา | สาเหตุ | วิธีแก้ |
|---|---|---|
| Agent ไม่ response | OpenAI credential ผิด | ตรวจสอบ API key ใน n8n Credentials |
| Web Research ไม่ทำงาน | Brave API key ผิดหรือหมด quota | ดู [api.search.brave.com](https://api.search.brave.com) → Usage |
| Output ไม่ใช่ JSON ที่ valid | Temperature สูงเกิน / prompt ไม่ชัด | ลอง execute ใหม่ หรือลด temperature เป็น 0.1 |
| `Save Strategy Decision` fail | Webhook URL ไม่ถูกต้อง | ตั้ง `N8N_HOST` env var ให้ถูกต้อง — error นี้ `continueOnFail: true` จะไม่หยุด workflow |

---

## Human Support — สิ่งที่คุณต้องทำ

| เมื่อไหร่ | งาน |
|---|---|
| **ก่อนรัน** | ตั้ง credentials ให้ครบ |
| **หลังรัน** | อ่าน output และตัดสินใจเลือก business model ด้วยตัวเอง |
| **หลังเลือก** | Activate workflows ที่เกี่ยวข้องตามที่ตัดสินใจ |
| **ถ้าต้องการ re-seed** | แก้ prompt ใน `Set Research Prompt` node แล้วรันใหม่ได้เสมอ |
