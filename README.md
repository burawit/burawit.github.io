# Asset Scanner — คู่มือติดตั้ง

## ภาพรวม

```
มือถือ (Browser)
   │
   ├── อ่านข้อมูล ◄── Google Sheets (public read, ไม่ต้อง login)
   └── เขียนข้อมูล ──► Google Apps Script Web App ──► Google Sheets
```

## ขั้นตอนที่ 1 — เตรียม Google Sheets

1. สร้าง Google Sheets ใหม่
2. เปลี่ยนชื่อแท็บแรกเป็น `Assets`
3. Import ไฟล์ `combined_assets.csv` (File → Import → Upload → Replace current sheet)
4. ตรวจสอบว่า header row มีคอลัมน์: `Site_Id, ลำดับ, รายการอุปกรณ์, Asset, S/N, Application`
5. **แชร์แบบ public read:** Share → General access → "Anyone with the link" → **Viewer**
6. จด **Sheet ID** จาก URL:
   ```
   https://docs.google.com/spreadsheets/d/【นี่คือ SHEET_ID】/edit
   ```

## ขั้นตอนที่ 2 — ติดตั้ง Apps Script (สำหรับบันทึกข้อมูล)

1. ใน Google Sheets เดิม → Extensions → **Apps Script**
2. ลบโค้ดเดิม วางโค้ดจากไฟล์ `Code.gs` ทั้งหมด
3. กด Deploy → **New deployment** → เลือก type **Web app**
   - Description: Asset Scanner API
   - Execute as: **Me**
   - Who has access: **Anyone**
4. กด Deploy → อนุญาตสิทธิ์ (Authorize) → copy **Web App URL**
   - หน้าตา: `https://script.google.com/macros/s/XXXXX/exec`

## ขั้นตอนที่ 3 — ตั้งค่า index.html

เปิดไฟล์ `index.html` แก้ 2 บรรทัดนี้ (อยู่ใน `<script>` ส่วน CONFIG):

```javascript
const SHEET_ID   = "ใส่ Sheet ID จากขั้นตอนที่ 1";
const SCRIPT_URL = "ใส่ Web App URL จากขั้นตอนที่ 2";
```

## ขั้นตอนที่ 4 — Deploy บน GitHub Pages

1. สร้าง repo ใหม่บน GitHub (เช่น `asset-scanner`)
2. Upload `index.html` เข้า repo
3. Settings → Pages → Source: **Deploy from a branch** → Branch: `main` → Save
4. รอสักครู่ ได้ URL: `https://username.github.io/asset-scanner/`

> ⚠️ **กล้องใช้ได้เฉพาะ HTTPS** — GitHub Pages เป็น HTTPS อยู่แล้ว
> ถ้าทดสอบในเครื่องให้ใช้ `localhost` (ถือว่า secure context)

## ขั้นตอนที่ 5 — ติดตั้งบนมือถือ (แบบ PWA)

- **Android (Chrome):** เปิด URL → เมนู ⋮ → "Add to Home screen"
- **iOS (Safari):** เปิด URL → ปุ่ม Share → "Add to Home Screen"

## ฟีเจอร์

| ฟีเจอร์ | รายละเอียด |
|---------|-----------|
| สแกน Barcode/QR | ZXing-js ผ่านกล้องหลัง รองรับ Code128, EAN, QR ฯลฯ |
| ค้นหาด้วยมือ | พิมพ์ Asset No. หรือ S/N (รองรับค้นหาบางส่วน) |
| ดูรายละเอียด | ข้อมูลครบทุกคอลัมน์จาก Sheet |
| แก้ไข | Status / Room / Note → เขียนกลับเข้า Sheet |
| ตรวจสอบแล้ว ✓ | บันทึกการตรวจนับลงแท็บ AuditLog |
| ประวัติ | Log ฝั่งเครื่อง (localStorage) + log กลางใน Sheet |
| Offline | Cache ข้อมูลล่าสุดไว้ ใช้ค้นหาได้แม้ไม่มีเน็ต |

## ข้อจำกัดที่ควรรู้

- ข้อมูลใน Sheet เป็น **public read** — ใครมีลิงก์ก็อ่านได้ อย่าเก็บข้อมูลลับ
- Apps Script Web App เปิดแบบ Anyone — มีชื่อ user ใน log แต่ไม่ได้ verify ตัวตนจริง
- ถ้าต้องการความปลอดภัยสูงขึ้นภายหลัง → ย้ายไป Supabase (มี auth จริง) ได้โดยแก้แค่ส่วน loadAssets/postToSheet