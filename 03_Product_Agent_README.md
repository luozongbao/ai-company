# 03 Product Agent  Weekly Digital Product Creation

ทกองคาร 09:00  AI วเคราะหความตองการตลาด แลวสราง digital product ใหม 1 ชน พรอม publish ขน Ghost เปน paid post

---

## Overview

| รายละเอยด | คา |
|---|---|
| **Trigger** | ทกองคาร 09:00 (Weekly) |
| **Agent** | Product Agent (GPT-4o, temp: 0.6) |
| **Input** | วนท + week number |
| **Output** | Digital product content + Ghost paid draft + Product_Catalog log |
| **Runtime** | ~35 นาท |

---

## Node Flow

```
Schedule: Every Tuesday 9am
        
Set Product Context  (product_prompt + week_number)
        
Product Agent  Tool: Market Research (Brave Search)
             Tool: Publish to Ghost (paid post draft)
             Tool: Read Product Catalog (avoid duplicates)
        
Save to Product Catalog (Google Sheets)
```

---

## Tools ท Agent ใช

| Tool | ชนด | หนาท |
|---|---|---|
| `Tool: Market Research` | HTTP (Brave Search) | คนหา AI automation pain points + trending products สปดาหน |
| `Tool: Publish to Ghost` | toolCode (Ghost Admin API) | สราง paid post draft พรอม product content |
| `Tool: Read Product Catalog` | HTTP (Google Sheets API) | อาน catalog เพอหลกเลยงสราง product ซำ |

---

## Product Types ท Agent สราง (หมนเวยน)

| Product Type | เนอหา | ราคาแนะนำ |
|---|---|---|
| **Prompt Pack** | 3050 prompts สำหรบ GPT-4o ตาม use case เฉพาะ | $9$19 |
| **n8n Workflow Template** | JSON workflow + setup guide สำหรบ automation งาน | $19$29 |
| **AI Starter Guide** | คมอ 1,5002,500 คำ วธใช AI สำหรบงานเฉพาะ | $9$19 |
| **Automation Checklist** | Checklist + resources สำหรบ automate business process | $9 |

---

## Input / Output

**Input (Set Product Context):**
```json
{
  "product_prompt": "Today is Tuesday, March 4 2026. Research what AI automation digital products people are looking for...",
  "product_week": 10
}
```

**Output (Agent JSON):**
```json
{
  "productType": "prompt_pack",
  "title": "50 ChatGPT Prompts for Freelance Writers",
  "subtitle": "Save 10+ hours per week with AI-assisted writing",
  "price": 19,
  "description": "A curated collection of 50 production-ready GPT-4o prompts...",
  "content": "[Full prompt pack content  all 50 prompts with explanations]",
  "deliveryFormat": "markdown",
  "ghostPostReady": true,
  "weekNumber": 10,
  "demandSignal": "Reddit r/freelance: multiple posts requesting writing automation this week"
}
```

---

## Google Sheets

**Product_Catalog** (columns ท Agent เขยน):
| Column | คา |
|---|---|
| `created_at` | ISO timestamp |
| `week_number` | เลขสปดาห |
| `product_result` | JSON output ทงหมด |
| `status` | `draft` |

> **หมายเหต:** Fulfillment Agent (06) อาน `product_result` จาก sheet นเมอตอง deliver ใหลกคา

---

## Credentials ทตองตง

| Credential | Node | วธตง |
|---|---|---|
| OpenAI API | `OpenAI GPT-4o` | n8n  Credentials  OpenAI API |
| Google Sheets OAuth2 | `Save to Product Catalog`, `Tool: Read Product Catalog` | n8n  Credentials  Google Sheets |
| Brave Search Header Auth | `Tool: Market Research` | HTTP Header Auth: `X-Subscription-Token`  ด [SETUP_BraveSearch.md](SETUP_BraveSearch.md) |

**Environment Variables:**
| Variable | คา |
|---|---|
| `GHOST_API_URL` | `https://your-ghost-blog.com` |
| `GHOST_ADMIN_API_KEY` | Ghost  Settings  Integrations  Create Custom Integration |

---

## วธใชงาน

### ครงแรก
1. ตง env vars `GHOST_API_URL` + `GHOST_ADMIN_API_KEY`
2. ตง Brave Search credential
3. สราง sheet `Product_Catalog` ใน Spreadsheet (columns: created_at, week_number, product_result, status)
4. แทน `REPLACE_WITH_SPREADSHEET_ID` ดวย Spreadsheet ID จรง
5. Activate workflow

### Ghost Setup สำหรบ Product Posts
1. Ghost Dashboard  Settings  Integrations  **Add custom integration**
2. ชอ: "n8n Product Agent"
3. Copy **Admin API Key**
4. ตง Stripe membership tier ใน Ghost ($9/mo หรอ one-time price)

---

## ความสมพนธกบ Workflow อน

```
01_CEO_Orchestrator   กำหนด product direction สปดาหน (CEO วางแผนจนทร, Product Agent ทำองคาร)
02_Content_Agent      promote products ใหมใน blog posts + social
06_Fulfillment_Agent  อาน Product_Catalog เมอตอง deliver ใหลกคา
04_Finance_Agent      track product sales revenue ใน Stripe
05_SelfImprovement    audit product quality + suggest improvements
```

---

## Troubleshooting

| ปญหา | สาเหต | วธแก |
|---|---|---|
| Ghost API 401 | Admin API Key ผดหรอหมดอาย | Ghost  Settings  Integrations  regenerate key |
| Ghost API 422 | HTML format ไมถกตอง | ตรวจ html_content ใน toolCode output |
| Product content ซำกบสปดาหกอน | Read Catalog tool ไมทำงาน | ตรวจ Google Sheets credential + SPREADSHEET_ID |
| Brave Search ไมมผล | API key หมด quota | ตรวจ api.search.brave.com dashboard |

---

## Human Review Checklist

หลง Product Agent รนทกองคาร:
- [ ] เปด Ghost Dashboard  Review draft post ทสราง
- [ ] ตรวจวา product content ครบถวน ไมใช placeholder
- [ ] ตรวจ price ท Agent กำหนด  สมเหตสมผลกบเนอหา?
- [ ] Publish Ghost post (change status: draft  published) เมอ approve แลว
- [ ] อปเดต `status` ใน Product_Catalog sheet จาก `draft`  `published`
