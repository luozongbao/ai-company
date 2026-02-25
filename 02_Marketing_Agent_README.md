# 02 Marketing Agent — Daily Content Creation & Publishing

| Property | Value |
|---|---|
| **File** | `02_Marketing_Agent.json` |
| **Trigger** | Schedule — ทุกวัน เวลา 08:00 |
| **Version** | 1.1.0 |
| **Status** | Activate หลังจาก `01_CEO_Orchestrator` รันแล้วอย่างน้อย 1 ครั้ง |

---

## วัตถุประสงค์

Marketing Agent สร้าง **content อัตโนมัติทุกวัน** สำหรับทุกช่องทาง ตั้งแต่ค้นหา trending topics จนถึง publish จริงผ่าน Buffer (Twitter/LinkedIn) และ Ghost (newsletter draft) โดยไม่ต้องมีมนุษย์ช่วย

---

## การทำงาน (Node-by-Node Flow)

```
Schedule Trigger (ทุกวัน 08:00)
    │
    ▼
Set Daily Context
    │  กำหนด today (formatted date), content_prompt
    ▼
Marketing Agent  ←── OpenAI GPT-4o (temp: 0.7, max 4000 tokens)
    │            ←── Marketing Memory (session: "marketing_agent_memory")
    │            ←── Tool: Trending Topics (Brave Search — freshness: last 24h)
    │            ←── Tool: SEO Keywords (Brave Suggest API)
    │
    │  Output JSON: trendingTopics, twitterThread, linkedinPost,
    │               newsletterSection, blogOutline, publishingSchedule
    ▼
Content Generated? (IF node)
    │
    ├── [YES] Log Content to Sheets (Google Sheets → Content_Log)
    │              │
    │              ▼
    │         Parse Content for Publishing (Code node)
    │              │  แปลง JSON output → publishing payloads
    │              │
    │         ┌────┴────┐
    │         ▼         ▼
    │   Publish via   Publish to
    │   Buffer API    Ghost Blog
    │  (Twitter/      (newsletter
    │  LinkedIn)       draft)
    │
    └── [NO]  Log Error (Google Sheets → Error_Log)
```

---

## Tools ที่ใช้

| Tool | ประเภท | หน้าที่ |
|---|---|---|
| `get_trending_topics` | HTTP (Brave Search) | ค้นหา trending topics ใน AI, automation, passive income — freshness: วันนี้ |
| `seo_keyword_research` | HTTP (Brave Suggest) | ดึง keyword suggestions สำหรับ SEO blog optimization |

---

## Output หลัก

### Google Sheets `Content_Log`
| Column | ข้อมูล |
|---|---|
| `date` | ISO timestamp ของวันที่ generate |
| `content_package` | JSON เต็มจาก Agent |
| `status` | `"published"` |

### Buffer (Twitter/LinkedIn)
- Twitter thread (8–12 tweets รวมกันเป็น thread)
- LinkedIn post (300–500 คำ)
- Scheduled 2 ชั่วโมงหลัง generate

### Ghost Blog
- สร้าง post ใหม่เป็น **draft** อัตโนมัติ
- ต้องการ human review ก่อน publish จริง (แนะนำ)

---

## Credentials ที่ต้องตั้ง

| Credential / Env Var | ใช้ที่ | วิธีตั้ง |
|---|---|---|
| OpenAI API | `OpenAI GPT-4o` | n8n → Credentials → OpenAI API |
| Google Sheets OAuth2 | `Log Content to Sheets`, `Log Error` | n8n → Credentials → Google Sheets OAuth2 |
| Brave Search Header Auth | `Tool: Trending Topics`, `Tool: SEO Keywords` | HTTP Header Auth: `X-Subscription-Token` — ดู [SETUP_BraveSearch.md](SETUP_BraveSearch.md) |
| Buffer HTTP Header Auth | `Publish via Buffer` | `Authorization: Bearer BUFFER_ACCESS_TOKEN` |
| Ghost HTTP Header Auth | `Publish to Ghost Blog` | `Authorization: Ghost YOUR_ADMIN_API_KEY` |
| `BUFFER_TWITTER_PROFILE_ID` | env var | n8n → Settings → Environment Variables |
| `GHOST_API_URL` | env var | n8n → Settings → Environment Variables (เช่น `https://your-blog.com`) |

---

## วิธีการใช้งาน

1. ตั้งค่า credentials และ environment variables ทั้งหมดให้ครบ
2. สร้าง sheet `Content_Log` และ `Error_Log` ใน Google Spreadsheet
3. **ทดสอบก่อน activate:** Execute manual 1 ครั้ง → ตรวจ content ใน Sheets
4. ตรวจ Ghost draft ว่า format ถูกต้อง
5. ตรวจ Buffer ว่า post schedule แล้ว
6. **Activate** workflow

> **แนะนำ:** ช่วง 2 สัปดาห์แรก ดู Ghost drafts ก่อน publish เพื่อ QC content quality

---

## Content ที่ Agent สร้างทุกวัน

| ชิ้นงาน | Platform | ความยาว | เนื้อหา |
|---|---|---|---|
| Twitter Thread | Twitter/X (via Buffer) | 8–12 tweets | Educational, value-driven, CTA |
| LinkedIn Post | LinkedIn (via Buffer) | 300–500 คำ | Professional insight, business angle |
| Newsletter Section | Ghost Blog (draft) | 400–600 คำ | Deep dive 1 topic |
| Blog Post Outline | Google Sheets | Structured H2s | Long-tail SEO keyword |

---

## การทำงานร่วมกับ Workflow อื่น

```
01_CEO_Orchestrator
    │  กำหนด priority topics + content direction รายสัปดาห์
    ▼
02_Marketing_Agent (รันทุกวัน)
    │
    │  Content → ดึง traffic → leads เข้าสู่ funnel
    ▼
03_Sales_Agent
    │  ใช้ content ที่ Marketing สร้างเป็น social proof ในการ outreach
    │
    ▼
05_SelfImprovement_Agent
    │  ตรวจ Content_Log ว่า publish ครบทุกวันไหม
    │  วิเคราะห์ topic engagement + เสนอ content strategy ใหม่
```

---

## Audit Checklist (ดูทุกวัน หรืออย่างน้อยสัปดาห์ละครั้ง)

- [ ] มี record ใหม่ใน `Content_Log` sheet ทุกวัน
- [ ] `Error_Log` ว่างเปล่า (ถ้ามี error → ตรวจสอบทันที)
- [ ] Ghost Blog มี draft post ใหม่อย่างน้อย 1 ชิ้น/วัน
- [ ] Buffer มี post scheduled ไว้
- [ ] Content ไม่มีข้อมูลผิดพลาด, misleading, หรือ hallucinated stats
- [ ] SEO blog outline มี H2 structure ชัดเจน
- [ ] ไม่มี spam / clickbait ใน copy

---

## Troubleshooting

| ปัญหา | สาเหตุ | วิธีแก้ |
|---|---|---|
| `Content Generated?` → NO ทุกวัน | Agent output ว่างเปล่า | ดู execution log → ตรวจ OpenAI connection |
| Buffer ไม่รับ post | Profile ID ผิด หรือ token หมดอายุ | ตรวจ `BUFFER_TWITTER_PROFILE_ID` + refresh token |
| Ghost post ไม่สร้าง | Admin API key ผิด หรือ URL ผิด | ดู Ghost Admin → Settings → Integrations → copy key ใหม่ |
| Content ซ้ำ หรือ topic เดิมทุกวัน | Memory buffer เก็บ context ซ้ำ | ลบ session `marketing_agent_memory` ใน PostgreSQL แล้วรันใหม่ |
| Brave API ไม่ตอบ | Rate limit (2000 req/mo free) | อัปเกรด plan หรือ cache results ด้วย Code node |

---

## Human Support — สิ่งที่คุณต้องทำ

| ความถี่ | งาน | เวลา |
|---|---|---|
| **ทุกวัน** (optional) | สุ่มตรวจ content 1 ชิ้นจาก Content_Log | 5 นาที |
| **ทุกสัปดาห์** | Review Ghost drafts + publish ที่ดีที่สุด 2–3 ชิ้น | 15 นาที |
| **ทุกสัปดาห์** | ตรวจ Error_Log ว่ามี error สะสมไหม | 5 นาที |
| **รายเดือน** | ตรวจว่า Buffer/Ghost credentials ยังใช้งานได้ | 5 นาที |
| **เมื่อพบ content ผิดพลาด** | Deactivate → แก้ system prompt → Activate ใหม่ | 20 นาที |
