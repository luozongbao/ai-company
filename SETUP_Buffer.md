# Buffer API — Setup Guide

| Property | Value |
|---|---|
| **ใช้โดย Workflow** | `02_Content_Agent` |
| **หน้าที่** | กำหนดเวลา + โพสต์ Twitter/X อัตโนมัติ ทุกวัน |
| **ราคา** | Essentials — $6/เดือน (API ต้องใช้แผนชำระเงิน) |
| **เวลาตั้งค่า** | ~15 นาที |

---

## Buffer คืออะไร?

Buffer คือ Social Media Scheduling platform ที่ให้โพสต์ Twitter/X และ LinkedIn ในเวลาที่กำหนดผ่าน API  
Content Agent ใช้ Buffer เพื่อส่ง Twitter thread ทุกวันอัตโนมัติ โดยกำหนดเวลาโพสต์ล่วงหน้า 2 ชั่วโมงจากที่ Agent รัน  
(Agent รัน 08:00 → โพสต์ลง Twitter 10:00)

> **หมายเหตุ:** Buffer API ต้องใช้แผน Essentials ($6/เดือน) ขึ้นไป — Free plan ไม่รองรับ API access

---

## ขั้นตอนตั้งค่า

### Step 1 — สร้าง Buffer Account และอัปเกรดแผน

1. ไปที่ [buffer.com](https://buffer.com) → สร้าง account
2. Settings → Billing → เลือกแผน **Essentials ($6/เดือน)**
3. เชื่อมต่อ Twitter/X account ใน Buffer (Channels tab)

---

### Step 2 — สร้าง Buffer App

1. ไปที่ [buffer.com/developers/apps](https://buffer.com/developers/apps)
2. คลิก **Create an App** และกรอก:
   - App Name: `n8n AI Company`
   - Callback URL: `https://YOUR_N8N_URL/rest/oauth2-credential/callback`
3. บันทึก **Client ID** และ **Client Secret**

---

### Step 3 — รับ Access Token

**วิธี 1 — ผ่าน Authorization URL (แนะนำ)**

เปิด browser ไปที่:
```
https://bufferapp.com/oauth2/authorize?
  client_id=YOUR_CLIENT_ID
  &redirect_uri=https://YOUR_N8N_URL/rest/oauth2-credential/callback
  &response_type=code
```

หลัง authorize แล้ว exchange code เป็น token:
```
POST https://api.bufferapp.com/1/oauth2/token.json
  client_id=CLIENT_ID
  client_secret=CLIENT_SECRET
  redirect_uri=CALLBACK_URL
  code=AUTHORIZATION_CODE
  grant_type=authorization_code
```

**วิธี 2 — ผ่าน n8n OAuth2 (ง่ายกว่า)**

1. n8n → Settings → Credentials → Add Credential → **OAuth2 API**
2. กรอก:
   - Auth URL: `https://bufferapp.com/oauth2/authorize`
   - Access Token URL: `https://api.bufferapp.com/1/oauth2/token.json`
   - Client ID / Secret: ค่าจาก Step 2
3. คลิก **Connect** → login Buffer → อนุญาต scope

---

### Step 4 — หา Twitter Profile ID

```
GET https://api.bufferapp.com/1/profiles.json
Authorization: Bearer YOUR_ACCESS_TOKEN
```

Response:
```json
[
  {
    "id": "abc123def456",
    "service": "twitter",
    "service_username": "@yourhandle"
  }
]
```

บันทึก `"id"` ของ Twitter/X profile

---

### Step 5 — ตั้งค่า Credential ใน n8n

1. n8n → Settings → Credentials → Add Credential → **Header Auth**
2. กรอก:
   - Name: `Buffer API`
   - Name (header key): `Authorization`
   - Value: `Bearer YOUR_ACCESS_TOKEN`
3. บันทึก → จด Credential ID

---

### Step 6 — ตั้ง Environment Variable

เพิ่มใน Docker run command หรือ `.env` file:
```bash
BUFFER_TWITTER_PROFILE_ID=abc123def456
```

---

### Step 7 — อัปเดต Workflow JSON

ใน `02_Content_Agent.json` แทนที่ placeholder:
- `REPLACE_WITH_BUFFER_CREDENTIAL_ID` → Credential ID จาก Step 5

---

## Node ที่ใช้ใน Workflow

| Workflow | Node Name | Endpoint | หน้าที่ |
|---|---|---|---|
| `02_Content_Agent` | `Publish via Buffer` | `POST /1/updates/create.json` | โพสต์ Twitter thread ทุกวัน 10:00 |

### ตัวอย่าง Request ที่ Workflow ยิง:
```
POST https://api.bufferapp.com/1/updates/create.json
Content-Type: application/x-www-form-urlencoded
Authorization: Bearer YOUR_TOKEN

text=Thread+content...
profile_ids[]=abc123def456
scheduled_at=2026-02-26T10:00:00Z
```

---

## ต้องการ LinkedIn ด้วยไหม?

ปัจจุบัน workflow โพสต์แค่ Twitter/X ถ้าต้องการ LinkedIn:

1. หา LinkedIn Profile ID จาก `/1/profiles.json`
2. ตั้ง env var: `BUFFER_LINKEDIN_PROFILE_ID=xyz789`
3. เพิ่ม `profile_ids[]` ที่สองใน body ของ node `Publish via Buffer`

---

## ทางเลือกฟรีแทน Buffer

| ทางเลือก | ราคา | หมายเหตุ |
|---|---|---|
| Twitter/X API v2 Free | $0 | 1,500 tweet/เดือน, ต้องเปลี่ยน node |
| Twitter/X API Basic | $100/เดือน | 3,000 tweet/เดือน |
| LinkedIn API (Direct) | ฟรี | ต้องสมัคร LinkedIn Developer App แยก |

> ถ้าต้องการเปลี่ยนเป็น Twitter/X API โดยตรง (ไม่ผ่าน Buffer) แจ้งได้เลย
