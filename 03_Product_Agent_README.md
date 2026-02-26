# 03 Product Agent — Weekly Digital Product Creation

| Property | Value |
|---|---|
| **File** | `03_Product_Agent.json` |
| **Trigger** | Schedule — ทุกวันอังคาร เวลา 09:00 |
| **Version** | 1.0.0 |
| **Status** | Activate หลังจากรัน `00_SEED_Strategy` และ `01_CEO_Orchestrator` ทำงานแล้วสัปดาห์แรก |

---

## วัตถุประสงค์

Product Agent ทำหน้าที่เป็น **นักพัฒนา digital product อัตโนมัติ** ทำงานทุกวันอังคารเพื่อ:
- ค้นหาความต้องการตลาดล่าสุดด้วย Brave Search
- เลือก product type + topic ที่มี demand signal สูงสุดสัปดาห์นี้
- สร้าง digital product ที่สมบูรณ์พร้อมขาย (prompt pack / n8n template / AI guide / checklist)
- Publish เป็น paid WordPress post draft (เพื่อให้ human review ก่อน publish จริง)
- บันทึกลง `Product_Catalog` sheet สำหรับให้ Fulfillment Agent ดึงไป deliver

---

## การทำงาน (Node-by-Node Flow)

```
Schedule Trigger (ทุกอังคาร 09:00)
    │
    ▼
Set Product Context
    │  กำหนด: product_prompt (วันที่ + คำสั่ง), product_week (เลขสัปดาห์)
    ▼
Product Agent  ←── DeepSeek Chat (temp: 0.6, max 4000 tokens)
    │          ←── Product Memory (session: "product_agent_memory")
    │          ←── Tool: Market Research (Brave Search — freshness: pw)
    │          ←── Tool: Publish to WordPress (toolCode — WordPress API)
    │          ←── Tool: Read Product Catalog (Google Sheets — หลีกเลี่ยง duplicate)
    │
    │  Output JSON: productType, title, subtitle, price, description,
    │               content, deliveryFormat, ghostPostReady, weekNumber, demandSignal
    ▼
Save to Product Catalog (Google Sheets append → sheet: Product_Catalog)
    │
    ▼
[จบ — WordPress draft รอ human review + Product_Catalog อัปเดตแล้ว]
```

---

## Tools ที่ใช้

| Tool | ประเภท | หน้าที่ |
|---|---|---|
| `Tool: Market Research` | HTTP (Brave Search API) | ค้นหา AI automation pain points + trending products สัปดาห์นี้ |
| `Tool: Publish to WordPress` | toolCode (WordPress API) | สร้าง paid post draft พร้อม product content กำหนด visibility: paid |
| `Tool: Read Product Catalog` | HTTP (Google Sheets API) | อ่าน catalog เพื่อหลีกเลี่ยงสร้าง product ซ้ำ |

---

## Product Types ที่ Agent สร้าง (หมุนเวียน)

| Product Type | เนื้อหา | ราคาแนะนำ |
|---|---|---|
| **Prompt Pack** | 30–50 prompts สำหรับ GPT-4o ตาม use case เฉพาะ | $9–$19 |
| **n8n Workflow Template** | JSON workflow + setup guide สำหรับ automation งาน | $19–$29 |
| **AI Starter Guide** | คู่มือ 1,500–2,500 คำ วิธีใช้ AI สำหรับงานเฉพาะ | $9–$19 |
| **Automation Checklist** | Checklist + resources สำหรับ automate business process | $9 |

---

## Input / Output

### Input (Set Product Context)

```json
{
  "product_prompt": "Today is Tuesday, March 4 2026. Research what AI automation digital products people are looking for this week. Then create one complete, high-quality digital product ready to sell at $9–$29.",
  "product_week": 10
}
```

### Output (Agent JSON บันทึกลง Product_Catalog)

```json
{
  "productType": "prompt_pack",
  "title": "50 ChatGPT Prompts for Freelance Writers",
  "subtitle": "Save 10+ hours per week with AI-assisted writing",
  "price": 19,
  "description": "A curated collection of 50 production-ready GPT-4o prompts...",
  "content": "[Full prompt pack content — all 50 prompts with explanations]",
  "deliveryFormat": "markdown",
  "ghostPostReady": true,
  "weekNumber": 10,
  "demandSignal": "Reddit r/freelance: multiple posts requesting writing automation this week"
}
```

---

## Credentials ที่ต้องตั้ง

**n8n Credentials (Settings → Credentials):**

| Credential | ใช้ที่ | วิธีตั้ง |
|---|---|---|
| OpenAI API | `DeepSeek Chat` | n8n → Credentials → DeepSeek API (OpenAI-compatible) |
| Google Sheets OAuth2 | `Save to Product Catalog`, `Tool: Read Product Catalog` | n8n → Credentials → Google Sheets OAuth2 |
| Brave Search Header Auth | `Tool: Market Research` | HTTP Header Auth: `X-Subscription-Token` — ดู [SETUP_BraveSearch.md](SETUP_BraveSearch.md) |

**n8n Environment Variables (Settings → Environment Variables):**

| Variable | ค่า | อธิบาย |
|---|---|---|
| `WORDPRESS_URL` | `https://your-ghost-blog.com` | URL ของ WordPress (ไม่มี trailing slash) |
| `WORDPRESS_APP_PASSWORD` | Admin API Key จาก WordPress | WordPress → Settings → Integrations → Create Custom Integration |
| `REPLACE_WITH_SPREADSHEET_ID` | Google Sheet ID | เปลี่ยนใน node parameters ของ `Tool: Read Product Catalog` และ `Save to Product Catalog` |

---

## Google Sheets ที่ต้องมี

**Product_Catalog** (Agent เขียน — Fulfillment Agent อ่าน):

| Column | ข้อมูล | ตัวอย่าง |
|---|---|---|
| `created_at` | ISO timestamp | `2026-03-04T09:12:00.000Z` |
| `week_number` | เลขสัปดาห์ | `10` |
| `product_result` | JSON output ทั้งหมดจาก Agent | `{"productType":"prompt_pack","title":"...","content":"..."}` |
| `status` | สถานะ | `draft` → คุณเปลี่ยนเป็น `published` เมื่อ approve |

> **หมายเหตุ:** `06_Fulfillment_Agent` อ่าน `product_result` จาก sheet นี้เมื่อต้อง deliver ให้ลูกค้า ดังนั้นต้อง activate workflow นี้ก่อนเปิดรับ payment

---

## วิธีการใช้งาน

1. ตั้ง env vars: `WORDPRESS_URL` + `WORDPRESS_APP_PASSWORD`
2. ตั้ง Brave Search credential ใน n8n
3. สร้าง sheet `Product_Catalog` ใน Spreadsheet (columns: created_at, week_number, product_result, status)
4. แทน `REPLACE_WITH_SPREADSHEET_ID` ด้วย Spreadsheet ID จริงในทั้ง 2 node
5. **Activate** workflow

### WordPress Setup สำหรับ Product Posts

1. WordPress Dashboard → Settings → Integrations → **Add custom integration**
2. ชื่อ: `n8n Product Agent`
3. Copy **Admin API Key** → ใส่ใน env var `WORDPRESS_APP_PASSWORD`
4. ตั้ง membership tier ใน WordPress (paid tier สำหรับ product delivery)

---

## การทำงานร่วมกับ Workflow อื่น

```
01_CEO_Orchestrator ──→ กำหนด product direction ประจำสัปดาห์ (จันทร์ 07:00 → อังคาร 09:00)
02_Content_Agent    ──→ promote product ใหม่ใน blog posts + social content
06_Fulfillment_Agent ──→ อ่าน Product_Catalog เมื่อต้อง deliver ให้ลูกค้า (Stripe webhook)
04_Finance_Agent    ──→ track product sales revenue จาก Stripe
05_SelfImprovement  ──→ audit product quality + variety + pricing ทุกอาทิตย์
```

---

## Audit Checklist (ดูทุกวันอังคารหลัง 09:00)

- [ ] มี record ใหม่ใน `Product_Catalog` sheet (week_number ตรงกับสัปดาห์นี้)
- [ ] เปิด WordPress Dashboard → ดู draft post ที่สร้าง → ตรวจ content ครบถ้วน ไม่มี placeholder
- [ ] ตรวจ price ที่ Agent กำหนด — สมเหตุสมผลกับเนื้อหา?
- [ ] **Publish WordPress post** (draft → published) เมื่อ approve แล้ว
- [ ] อัปเดต `status` ใน Product_Catalog จาก `draft` → `published`
- [ ] Execution log ไม่มี error (n8n → Executions)

---

## Troubleshooting

| ปัญหา | สาเหตุ | วิธีแก้ |
|---|---|---|
| WordPress REST API 401 | Admin API Key ผิดหรือหมดอายุ | WordPress → Settings → Integrations → regenerate key |
| WordPress REST API 422 | HTML format ไม่ถูกต้อง | ตรวจ html_content ใน toolCode output ใน n8n |
| Product ซ้ำกับสัปดาห์ก่อน | `Tool: Read Product Catalog` ไม่ทำงาน | ตรวจ Google Sheets credential + SPREADSHEET_ID |
| Brave Search ไม่มีผล | API key หมด quota | ตรวจ api.search.brave.com dashboard |
| `WORDPRESS_URL env var` error | ยังไม่ได้ตั้ง env vars | n8n → Settings → Environment Variables |

---

## Human Support — สิ่งที่คุณต้องทำ

| ความถี่ | งาน | เวลา |
|---|---|---|
| **ทุกอังคาร** | Review WordPress draft → approve หรือ edit → publish | 15 นาที |
| **ทุกอังคาร** | อัปเดต `status` ใน Product_Catalog จาก `draft` → `published` | 2 นาที |
| **รายเดือน** | ทบทวน product variety — หมุนครบ 4 types หรือยัง? | 5 นาที |
| **ถ้า WordPress error** | ดู Execution log → แก้ credential หรือ URL | เมื่อพบปัญหา |
