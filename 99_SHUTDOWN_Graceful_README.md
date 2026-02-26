# 99 SHUTDOWN — Graceful System Shutdown

| Property | Value |
|---|---|
| **File** | `99_SHUTDOWN_Graceful.json` |
| **Trigger** | Manual (รันเมื่อต้องการปิดระบบ) |
| **Version** | 1.1.0 |
| **Status** | ไม่ต้อง Activate — รัน manual เมื่อต้องการ shut down เท่านั้น |

---

## วัตถุประสงค์

Workflow นี้คือ **ปุ่มปิดระบบแบบปลอดภัย** ทำสิ่งต่อไปนี้ตามลำดับ:
1. Generate รายงานสรุปบริษัททั้งหมด (final report)
2. บันทึก report ลง Google Sheets `Shutdown_Log`
3. **Deactivate ทุก workflow อัตโนมัติ** ผ่าน n8n Admin API
4. แสดงสถานะ shutdown สมบูรณ์

---

## การทำงาน (Node-by-Node Flow)

```
Manual Trigger (คุณกดรัน)
    │
    ▼
Set Shutdown Context
    │  กำหนด: shutdown_reason, shutdown_time,
    │          workflow_ids_to_deactivate (list)
    ▼
Generate Final Report  ←── DeepSeek Chat (temp: 0.2, max 4000 tokens)
    │                  ←── Shutdown Memory
    │                  ←── Tool: Final Financial Data (Google Sheets → Financial_Reports)
    │
    │  Output JSON: operational summary, content produced, sales results,
    │               system health, lessons learned, restart recommendations
    ▼
Save Final Report
    │  append ลง Google Sheets → Shutdown_Log sheet
    ▼
Deactivate All Workflows  (Code node — เรียก n8n Admin API จริง)
    │  GET /api/v1/workflows (list active)
    │  PATCH /api/v1/workflows/{id} → { active: false }  ทุก workflow
    │  Return: deactivated[], skipped[], failed[]
    ▼
Shutdown Complete
    │  SHUTDOWN_STATUS: "COMPLETE" หรือ "PARTIAL" (ถ้ามี failed)
    ▼
[จบ — ระบบทั้งหมดปิดแล้ว]
```

---

## Tools ที่ใช้

| Tool | ประเภท | หน้าที่ |
|---|---|---|
| `get_final_financial_data` | HTTP (Google Sheets API) | ดึง Financial_Reports ทั้งหมดสำหรับ final summary |

---

## Final Report Structure (JSON)

```json
{
  "operationalSummary": {
    "totalRuntime": "45 days",
    "totalRevenue": 12500,
    "totalExpenses": 380,
    "netProfit": 12120,
    "peakWeek": "Week 8 — $2,300 revenue"
  },
  "contentProduced": {
    "total": 315,
    "estimatedReach": 25000
  },
  "revenueResults": {
    "digitalProductsSold": 52,
    "newsletterSubscribers": 180,
    "affiliateClicks": 4300,
    "totalFromProducts": 9500
  },
  "systemHealthAtShutdown": {
    "mostReliable": "Finance Agent (99% success)",
    "mostProblematic": "Fulfillment Agent (SendGrid bounces week 3)"
  },
  "keyLessons": [...],
  "restartRecommendations": [...],
  "finalStatus": "SHUTDOWN COMPLETE"
}
```

---

## Credentials และ Env Vars ที่ต้องมี

| Credential / Env | ใช้ที่ | วิธีตั้ง |
|---|---|---|
| OpenAI API | `DeepSeek Chat` | n8n → Credentials → DeepSeek API (OpenAI-compatible) |
| Google Sheets OAuth2 | `Tool: Final Financial Data`, `Save Final Report` | n8n → Credentials → Google Sheets OAuth2 |
| `N8N_API_KEY` | `Deactivate All Workflows` (Code node) | n8n Settings → n8n API → Create API Key → ตั้งเป็น env var |
| `N8N_HOST` | `Deactivate All Workflows` (Code node) | Docker `-e N8N_HOST=http://your-server-ip:5678` |

> ⚠️ ถ้าไม่ตั้ง `N8N_API_KEY` — Deactivate node จะ throw error และ workflows **จะไม่ถูกปิดอัตโนมัติ** ต้องปิด manual แทน

---

## วิธีการใช้งาน — Graceful Shutdown

```
ขั้นตอนที่แนะนำ:

1. เปิด n8n → Workflows → "99 SHUTDOWN — Graceful System Shutdown"
2. คลิก Execute Workflow
3. รอ 1–2 นาทีให้ทุกขั้นตอนเสร็จ
4. ดู output ของ "Shutdown Complete" node:
   - SHUTDOWN_STATUS: "COMPLETE" → ทุก workflow ปิดแล้ว ✅
   - SHUTDOWN_STATUS: "PARTIAL" → ดู "workflows_failed" list → ปิด manual
5. เปิด Google Sheets → Shutdown_Log → อ่าน final report
6. (Optional) docker stop n8n  เพื่อปิด server ด้วย
```

---

## วิธีการใช้งาน — Emergency Stop (ทันที)

```bash
# ปิด n8n container ทันที — ไม่ generate report
docker stop n8n

# หรือถ้าต้องการ restart
docker restart n8n
```

---

## วิธี Deactivate เฉพาะ Agent (ไม่ต้อง shutdown ทั้งระบบ)

n8n → Workflows → ค้นหา workflow ที่ต้องการ → toggle **Active** เป็น OFF

ใช้เมื่อ: Agent ตัวหนึ่งมีปัญหา แต่ไม่ต้องการปิดทั้งระบบ

---

## การทำงานร่วมกับ Workflow อื่น

```
ระบบปกติ:
01_CEO ── 02_Content ── 03_Product ── 04_Finance ── 05_SelfImprovement
                                                           │
                                                   ทุกอาทิตย์ audit

เมื่อรัน 99_SHUTDOWN:
    │  อ่านข้อมูลจาก Financial_Reports (ที่ 04_Finance เขียนไว้)
    │  Deactivate 01, 02, 03, 04, 05 ทั้งหมด
    ▼
    ระบบหยุดทำงาน
```

---

## Google Sheets Structure

| Sheet Name | Columns | ใช้โดย |
|---|---|---|
| `Shutdown_Log` | `shutdown_time`, `reason`, `final_report` (JSON) | Shutdown workflow → append |

---

## Audit Checklist (หลัง Shutdown)

- [ ] `Shutdown_Log` มี record ใหม่
- [ ] `final_report.finalStatus` = `"SHUTDOWN COMPLETE"`
- [ ] `SHUTDOWN_STATUS` = `"COMPLETE"` (ไม่ใช่ PARTIAL)
- [ ] ตรวจ n8n → Workflows ว่าทุกตัวอยู่สถานะ Inactive
- [ ] Export Google Sheets ทุก sheet เป็น backup ก่อนปิด server
- [ ] Commit workflow JSON ทั้งหมดไปยัง Git

---

## Troubleshooting

| ปัญหา | สาเหตุ | วิธีแก้ |
|---|---|---|
| `N8N_API_KEY environment variable is not set` | ไม่ได้ตั้ง env var | n8n Settings → n8n API → สร้าง key → ตั้ง env var `N8N_API_KEY` |
| `SHUTDOWN_STATUS: "PARTIAL"` | บาง workflow deactivate ไม่ได้ | ดู `workflows_failed` list → ปิด manual ใน n8n UI |
| Final Report ไม่ครบ | Financial_Reports ว่างเปล่า | Finance Agent ต้องรันอย่างน้อย 1 ครั้งก่อน shutdown |
| Google Sheets error | OAuth2 หมดอายุ | Reconnect credential ก่อน shutdown |

---

## Human Support — สิ่งที่คุณต้องทำ

| สถานการณ์ | งาน |
|---|---|
| **ก่อน shutdown** | Export Google Sheets ทั้งหมด + commit JSON workflows ไปยัง Git |
| **หลัง shutdown** | อ่าน `final_report.keyLessons` + `restartRecommendations` เก็บไว้ใช้ครั้งหน้า |
| **ถ้า PARTIAL** | ปิด workflow ที่เหลือ manual ใน n8n UI |
| **ถ้าต้องการ restart** | แก้ตาม `restartRecommendations` → import workflows → รัน `00_SEED_Strategy` ใหม่ |

---

## ⚠️ ข้อควรระวัง

- **อย่ารัน Shutdown workflow โดยไม่ตั้งใจ** — Workflow นี้จะปิดระบบทั้งหมด
- **ไม่ควร Activate** workflow นี้เป็น scheduled — ตั้งให้เป็น manual trigger เท่านั้น
- **ทำ backup ก่อนเสมอ** — Export workflows JSON + Google Sheets ก่อนกด Execute
