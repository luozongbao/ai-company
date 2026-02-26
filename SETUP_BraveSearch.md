# Brave Search API — Setup Guide

| Property | Value |
|---|---|
| **ใช้โดย Workflow** | `00_SEED`, `01_CEO`, `02_Marketing`, `03_Sales`, `05_SelfImprovement` |
| **หน้าที่** | Web search + SEO keyword suggest สำหรับทุก AI Agent |
| **ราคา** | Free tier — $0 (2,000 requests/เดือน) |
| **เวลาตั้งค่า** | ~5 นาที |

---

## Brave Search API คืออะไร?

Brave Search API คือ **Search Engine API** ของ Brave ที่ให้ผลลัพธ์การค้นหาจาก index ของตัวเอง (ไม่ใช่ Google/Bing) มีความเป็นส่วนตัวสูง ไม่มี bias จาก ad tracking และมี **free tier ที่ใช้งานได้จริง**

Project นี้ใช้เป็น "ตา" ของทุก AI Agent — ทำให้ Agent มองเห็นโลกภายนอกได้แบบ real-time แทนที่จะตอบจาก training data เพียงอย่างเดียว

---

## Workflows ที่ใช้ Brave Search

| Workflow | Tool Name | Endpoint | Params | หน้าที่ |
|---|---|---|---|---|
| `00_SEED_Strategy` | `web_research` | `/web/search` | count: 10 | วิเคราะห์ตลาด + business model research |
| `01_CEO_Orchestrator` | `get_market_intelligence` | `/web/search` | count: 5 | market news รายสัปดาห์ |
| `02_Content_Agent` | `get_trending_topics` | `/web/search` | count: 10, freshness: **pd** (past day) | trending topics ล่าสุด |
| `02_Content_Agent` | `seo_keyword_research` | `/suggest` | — | keyword suggestions สำหรับ SEO blog |
| `03_Product_Agent` | `web_search` | `/web/search` | count: 10, freshness: **pw** (past week) | ค้นหา demand signal สำหรับ digital products รายสัปดาห์ |
| `05_SelfImprovement` | `research_new_ai_tools` | `/web/search` | count: 10, freshness: **pw** (past week) | ค้นหา AI tools ใหม่ประจำสัปดาห์ |

---

## การใช้ Request Budget

ประมาณการ requests ต่อเดือน เทียบกับ free tier:

| Workflow | Requests/รัน | ความถี่ | Requests/เดือน |
|---|---|---|---|
| `02_Marketing` (web search) | ~3–5 | ทุกวัน | ~90–150 |
| `02_Marketing` (suggest) | ~2–3 | ทุกวัน | ~60–90 |
| `03_Sales` | ~3–5 | ทุกวัน | ~90–150 |
| `01_CEO` | ~2–3 | ทุกจันทร์ | ~8–12 |
| `05_SelfImprovement` | ~2–3 | ทุกอาทิตย์ | ~8–12 |
| `00_SEED` | ~3–5 | ครั้งเดียว | ~5 |
| **รวม** | | | **~261–419 req/เดือน** |

> ✅ **Free tier 2,000 req/เดือน เพียงพอมาก** — ใช้ไม่ถึง 25% ของ quota

---

## Plans

| Plan | ราคา | Requests/เดือน | เหมาะกับ |
|---|---|---|---|
| **Free** | **$0** | 2,000 | ✅ เพียงพอตลอดช่วง 1 ปีแรก |
| Basic | $5/เดือน | 20,000 | ถ้าเพิ่ม workflows มากขึ้น |
| Pro | $40/เดือน | 200,000 | Scale ใหญ่มาก |

---

## Step-by-Step: ตั้งค่า Brave Search API

### Step 1 — สมัคร Account

1. ไปที่ **[api.search.brave.com](https://api.search.brave.com)**
2. คลิก **Sign up for free**
3. กรอก email + password (หรือ Sign in with Google)
4. Verify email

### Step 2 — Subscribe to Free Plan

1. หลัง login → ไปที่ **Subscriptions**
2. เลือก **Data for Search — Free** plan
3. คลิก **Subscribe** (ไม่ต้องใส่บัตรเครดิต)

### Step 3 — สร้าง API Key

1. ไปที่ **API Keys** (เมนูซ้าย)
2. คลิก **Add Key**
3. ตั้งชื่อ: `n8n-ai-company`
4. คลิก **Create**
5. **Copy API key** ทันที

```
ตัวอย่าง API key format:
BSA...xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

### Step 4 — ตั้ง Credential ใน n8n

Brave Search ใช้ **HTTP Header Authentication** ใน n8n — ทุก workflow ใช้ credential ชุดเดียวกัน:

1. n8n → **Settings → Credentials → Add Credential**
2. เลือกประเภท: **HTTP Header Auth**
3. กรอก:
   ```
   Name:   Brave Search API
   Name:   X-Subscription-Token
   Value:  BSA...xxxxxxxx  (API key จาก Step 3)
   ```
4. **Save**

### Step 5 — เชื่อม Credential กับทุก Tool Node

ใน n8n แต่ละ workflow มี Brave Search tool node ที่ต้องผูก credential:

| Workflow | Node ที่ต้องเชื่อม |
|---|---|
| `00_SEED_Strategy` | `Tool: Web Research` |
| `01_CEO_Orchestrator` | `Tool: Market Intelligence` |
| `02_Content_Agent` | `Tool: Trending Topics`, `Tool: SEO Keywords` |
| `03_Product_Agent` | `Tool: Find Leads` |
| `05_SelfImprovement_Agent` | `Tool: Research New Tools` |

สำหรับแต่ละ node:
1. คลิกที่ node → **Credential for Header Auth** → เลือก `Brave Search API`

---

## Endpoints ที่ใช้ใน Project

### 1. `/v1/web/search` — Web Search

ใช้ใน: SEED, CEO, Marketing, Sales, SelfImprovement

```
GET https://api.search.brave.com/res/v1/web/search

Headers:
  X-Subscription-Token: YOUR_API_KEY
  Accept: application/json

Query Parameters:
  q         (string)  — search query (AI กำหนดให้)
  count     (number)  — จำนวน results (5–20)
  freshness (string)  — pd = past day | pw = past week | pm = past month
```

**ตัวอย่าง query ที่แต่ละ agent ส่ง:**
```
SEED:           "best AI automation business models 2026 revenue"
CEO:            "AI workflow automation market trends February 2026"
Marketing:      "n8n automation trending topics today"
Sales:          "site:reddit.com need help automate my business 2026"
SelfImprovement:"new n8n nodes AI tools automation update 2026"
```

### 2. `/v1/suggest` — Search Suggestions (SEO Keywords)

ใช้ใน: Marketing Agent เท่านั้น

```
GET https://api.search.brave.com/res/v1/suggest

Headers:
  X-Subscription-Token: YOUR_API_KEY

Query Parameters:
  q (string) — partial keyword (AI กำหนดให้)
```

**ตัวอย่าง response:**
```json
{
  "query": "n8n automation",
  "results": [
    "n8n automation workflow examples",
    "n8n automation tutorial 2026",
    "n8n automation vs zapier"
  ]
}
```

---

## Monitor Usage

### ดู Quota ที่เหลือ

1. [api.search.brave.com](https://api.search.brave.com) → Login
2. **Dashboard → Usage** — เห็น requests ที่ใช้ไปในเดือนนี้

### Alert ที่ควร watch

| สถานการณ์ | Action |
|---|---|
| Usage > 1,500/เดือน (75%) | ตรวจว่า workflow รัน loop ผิดปกติไหม |
| API ตอบ `429 Too Many Requests` | Quota หมด → downgrade ช่อง search count หรือ upgrade plan |
| API ตอบ `401 Unauthorized` | API key ผิดหรือ credential ไม่ได้ผูก → ตรวจ n8n Credentials |

---

## Troubleshooting

| ปัญหา | สาเหตุ | วิธีแก้ |
|---|---|---|
| `401 Unauthorized` | API key ผิด หรือ credential ไม่ได้ผูกกับ node | ตรวจ n8n → Credentials → HTTP Header Auth → ค่า `Value` ถูกต้องไหม |
| `429 Too Many Requests` | Quota หมด (2,000/เดือน) | ดู usage dashboard → upgrade plan หรือรอเดือนถัดไป |
| Tool ไม่ return results | Query ไม่มีผลหรือ freshness เข้มเกิน | ลบ `freshness` parameter ออกชั่วคราวเพื่อทดสอบ |
| SEO suggest ว่างเปล่า | q สั้นเกิน (< 2 ตัวอักษร) | ปรับ prompt ให้ Agent ส่ง keyword ที่ยาวขึ้น |
| Results ภาษาผิด | ไม่มี `country`/`search_lang` param | เพิ่ม `country=us&search_lang=en` ใน query params |

---

## เปรียบเทียบกับ Alternative APIs

| API | Free Tier | ราคา Paid | ข้อดี | ข้อเสีย |
|---|---|---|---|---|
| **Brave Search** | 2,000/เดือน | $5+/เดือน | ✅ privacy-first, index ของตัวเอง | — |
| Google Custom Search | 100/วัน | $5/1,000 queries | ✅ Google results | ❌ แพงมาก, rate limit ต่ำ |
| SerpAPI | 100/เดือน | $50+/เดือน | ✅ Google + social | ❌ ฟรี tier น้อยมาก |
| Serper.dev | 2,500/เดือน | $50+/เดือน | ✅ Google results | ❌ ไม่ได้ใน free plan นาน |
| Tavily | 1,000/เดือน | $20+/เดือน | ✅ designed for AI agents | ❌ quota ต่ำกว่า Brave |

> **Brave Search เหมาะที่สุดสำหรับ project นี้** — free tier สูงสุด, designed for developers, ไม่มี Google dependency

---

## Links ที่เกี่ยวข้อง

- [Brave Search API Portal](https://api.search.brave.com)
- [Brave Search API Documentation](https://api.search.brave.com/app/documentation/web-search/get-started)
- [Web Search API Reference](https://api.search.brave.com/app/documentation/web-search/query)
- [Suggest API Reference](https://api.search.brave.com/app/documentation/suggestions/query)
- [Usage & Billing Dashboard](https://api.search.brave.com/app/subscriptions)
