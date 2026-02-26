# Google Sheets API — Setup Guide

| Property | Value |
|---|---|
| **ใช้โดย Workflow** | ทุก Workflow (00–06, 99) |
| **หน้าที่** | Central Database — เก็บ strategy, KPI, content, products, orders, reports ทั้งหมด |
| **ราคา** | ฟรี (Google Sheets API ฟรีในโควต้าปกติ) |
| **เวลาตั้งค่า** | ~20 นาที |

---

## Google Sheets ใช้ทำอะไรในโปรเจคต์นี้?

ทุก Agent ใช้ Google Sheets เป็น **central data store** แทน database:

| Sheet Name | ใช้โดย Agent | เก็บข้อมูล |
|---|---|---|
| `Strategy_Decisions` | 00 SEED | ผล niche research, affiliate programs ที่เลือก |
| `KPI_Dashboard` | 01 CEO, 05 Self-Improvement | metrics รวม: traffic, revenue, subscribers |
| `Weekly_Plans` | 01 CEO | weekly content themes + agent tasks |
| `Content_Log` | 02 Content | ทุก content package ที่ publish + วันที่ |
| `Product_Catalog` | 03 Product, 06 Fulfillment | ทุก digital product + เนื้อหาจัดส่ง |
| `Order_Log` | 06 Fulfillment | ทุก order + delivery status |
| `Financial_Reports` | 04 Finance, 99 Shutdown | weekly P&L reports |
| `Affiliate_Income` | 04 Finance | รายได้ affiliate แยกตาม program |
| `Improvement_Log` | 05 Self-Improvement | audit reports รายสัปดาห์ |
| `Shutdown_Log` | 99 Shutdown | final report เมื่อปิดระบบ |

---

## ขั้นตอนตั้งค่า

### Step 1 — สร้าง Google Cloud Project

1. ไปที่ [console.cloud.google.com](https://console.cloud.google.com)
2. คลิก **Select a project** (มุมบนซ้าย) → **New Project**
3. ตั้งชื่อ: `ai-company-n8n`
4. คลิก **Create**

---

### Step 2 — เปิดใช้งาน Google Sheets API

1. ใน Google Cloud Console → ไปที่ **APIs & Services** → **Library**
2. ค้นหา `Google Sheets API` → คลิก **Enable**
3. ค้นหา `Google Drive API` → คลิก **Enable** (จำเป็นสำหรับ n8n OAuth2)

---

### Step 3 — สร้าง OAuth2 Credentials

1. APIs & Services → **Credentials** → **Create Credentials** → **OAuth client ID**

2. ถ้ายังไม่ได้ตั้ง **consent screen**:
   - กด **Configure consent screen**
   - เลือก **External** → กรอก App name: `n8n AI Company`
   - เพิ่ม scope: `https://www.googleapis.com/auth/spreadsheets`
   - เพิ่ม scope: `https://www.googleapis.com/auth/drive.file`
   - บันทึก

3. กลับมาสร้าง OAuth client ID:
   - Application type: **Web application**
   - Name: `n8n-google-sheets`
   - Authorized redirect URIs: เพิ่ม `https://YOUR_N8N_URL/rest/oauth2-credential/callback`

4. บันทึก **Client ID** และ **Client Secret**

---

### Step 4 — ตั้งค่า Credential ใน n8n

1. n8n → Settings → Credentials → Add Credential
2. เลือก **Google Sheets OAuth2 API**
3. กรอก:
   - Client ID: ค่าจาก Step 3
   - Client Secret: ค่าจาก Step 3
4. คลิก **Sign in with Google** → login และอนุญาต permission
5. บันทึก → จด **Credential ID**

---

### Step 5 — สร้าง Google Spreadsheet และ Sheets ทั้งหมด

#### 5.1 สร้าง Spreadsheet ใหม่

1. ไปที่ [sheets.google.com](https://sheets.google.com) → สร้าง Spreadsheet ใหม่
2. ตั้งชื่อ: `AI Company — Central Database`
3. คัดลอก **Spreadsheet ID** จาก URL:
   ```
   https://docs.google.com/spreadsheets/d/SPREADSHEET_ID_HERE/edit
   ```

#### 5.2 สร้าง Sheets (tabs) ทั้งหมด

เปลี่ยนชื่อ Sheet1 และเพิ่ม sheets ตามรายการนี้ (ต้องตรงชื่อ **100%**):

| Sheet Name | Header row แนะนำ |
|---|---|
| `Strategy_Decisions` | timestamp, niche, affiliatePrograms, decision |
| `KPI_Dashboard` | week, pageViews, subscribers, affiliateClicks, productSales, revenue |
| `Weekly_Plans` | week_number, date, contentThemes, agentTasks, revenueTarget, riskFlags |
| `Content_Log` | date, content_package, status |
| `Product_Catalog` | created_at, week_number, product_result, status |
| `Order_Log` | timestamp, session_id, customer_email, product_title, amount, delivery_status |
| `Financial_Reports` | week_end, report, weeklyRevenue, netProfit, alerts |
| `Affiliate_Income` | date, program, clicks, conversions, revenue |
| `Improvement_Log` | audit_date, report, priority_actions |
| `Shutdown_Log` | shutdown_time, reason, final_report |

> **วิธีเพิ่ม sheet:** คลิก **+** ที่ด้านล่าง → คลิกขวาที่ tab → Rename

---

### Step 6 — อัปเดต Workflow JSONs

แทนที่ placeholder `REPLACE_WITH_SPREADSHEET_ID` ด้วย Spreadsheet ID จริง ใน **ทุก** workflow JSON
(มีใน 00, 01, 02, 03, 04, 05, 06, 99)

และแทนที่ `REPLACE_WITH_YOUR_GSHEETS_CREDENTIAL_ID` ด้วย Credential ID จาก Step 4

---

### Step 7 — ทดสอบ

ใน n8n เปิด `04_Finance_Agent` → Execute ด้วย test data → ดูว่า `Financial_Reports` sheet มีข้อมูลเพิ่มเข้ามา

---

## การใช้ API Quota

Google Sheets API มี quota สูงมาก ปกติไม่มีปัญหา:

| Limit | ค่า | การใช้งานในโปรเจคต์ |
|---|---|---|
| Read requests/นาที | 300 | ใช้ ~10–20/วัน |
| Write requests/นาที | 300 | ใช้ ~5–10/วัน |
| Requests/วัน | 1,000,000 | ใช้ < 100/วัน |

---

## Troubleshooting

| ปัญหา | สาเหตุ | วิธีแก้ |
|---|---|---|
| `403 Forbidden` | Credential ไม่มี permission | ตรวจสอบ scope ใน OAuth consent screen |
| `Spreadsheet not found` | Spreadsheet ID ผิด | copy ID จาก URL อีกครั้ง |
| `Sheet not found` | ชื่อ sheet ไม่ตรง | ตรวจสอบชื่อ tab ใน Sheets ให้ตรงกับรายการด้านบน |
| Token expired | OAuth token หมดอายุ | n8n จะ refresh อัตโนมัติ ถ้าไม่ได้ → reconnect credential |
