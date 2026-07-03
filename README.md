# THAMC PR ORDER — คู่มือติดตั้ง (ไฟล์พร้อมใช้งาน)

ระบบเสนอความต้องการซื้อ/จ้าง/เช่า — เมนูยืดหดได้ + รองรับมือถือเต็มรูปแบบ
สถาปัตยกรรม: **หน้าเว็บ (index.html) โฮสต์บน GitHub Pages** เรียก **Backend บน Google Apps Script**

```
GitHub Pages  ──(fetch JSON)──▶  Apps Script Web App  ──▶  Google Sheets + Drive
   index.html                        Code.gs
```

---

## ไฟล์ในชุดนี้
| ไฟล์ | หน้าที่ | วางไว้ที่ |
|---|---|---|
| `index.html` | หน้าเว็บทั้งระบบ (UI, เมนูยืดหด, โหมดมือถือ, เอกสาร A4 + QR, ดึง Excel SAP) | GitHub |
| `Code.gs` | Backend API router + ฟังก์ชันอ่าน/เขียน Google Sheets | Apps Script |
| `README.md` | คู่มือนี้ | — |

> **เปิดดูได้ทันที:** เปิด `index.html` แบบดับเบิลคลิกจะเข้า **โหมดสาธิต (Demo)** ด้วยข้อมูลตัวอย่าง (ยังไม่ต้องต่อ backend) ล็อกอินด้วยค่าที่กรอกไว้ให้แล้วได้เลย

---

## ขั้นตอนที่ 1 — เตรียม Google Sheets
สร้างสเปรดชีต 1 ไฟล์ มีชีตชื่อ: `Users`, `Departments`, `PurchaseRequests`, `PurchaseRequestItems`, `TransactionLog`, `ItemTypes`, `StaffAssignments`
(โครงสร้างคอลัมน์เหมือนระบบเดิมของคุณทุกประการ — ใช้ต่อได้เลย)

## ขั้นตอนที่ 2 — วาง Backend บน Apps Script
1. เปิด https://script.google.com → **New project**
2. วางเนื้อหา `Code.gs` ลงไป **แล้ววางฟังก์ชันธุรกิจเดิม** (submitPurchaseRequest, updatePurchaseRequestStatus, getUsersForAdmin, updateUserStatusByAdmin, getNotifications ฯลฯ) ต่อท้าย
3. แก้ค่าด้านบนไฟล์:
   - `SHEET_ID` = ID ของสเปรดชีต
   - `ATTACHMENT_FOLDER_ID`, `PDF_EXPORT_FOLDER_ID` = ID โฟลเดอร์ Drive
4. **Deploy → New deployment → Web app**
   - Execute as: **Me**
   - Who has access: **Anyone**
5. กด Deploy แล้ว **คัดลอกลิงก์** ที่ลงท้ายด้วย `/exec`

## ขั้นตอนที่ 3 — ต่อหน้าเว็บเข้ากับ Backend
เปิด `index.html` แก้บรรทัดบนสุดของ `<script>`:
```js
const CONFIG = {
  WEB_APP_URL: "https://script.google.com/macros/s/XXXXX/exec",  // ← วางลิงก์ที่ได้
};
```
เมื่อใส่ค่านี้แล้ว โหมดสาธิตจะปิดอัตโนมัติ และระบบจะเรียกข้อมูลจริง

## ขั้นตอนที่ 4 — วางหน้าเว็บบน GitHub Pages
1. สร้าง repository ใหม่ → อัป `index.html`
2. **Settings → Pages → Source: main / root → Save**
3. รอสักครู่จะได้ลิงก์ `https://<user>.github.io/<repo>/`

เสร็จแล้ว! เปิดลิงก์ GitHub Pages ใช้งานได้จากคอมพิวเตอร์และมือถือ (เพิ่มลง Home Screen ได้เหมือนแอป)

---

## หมายเหตุเรื่อง CORS
หน้าเว็บส่งคำขอแบบ `Content-Type: text/plain` เพื่อให้เป็น *simple request* (เลี่ยง preflight) — Apps Script Web App จะ redirect ไป `googleusercontent.com` ที่อนุญาต cross-origin จึงอ่านผลลัพธ์ได้ตามปกติ ไม่ต้องตั้ง proxy เพิ่ม

## การเชื่อมต่อในอนาคต
- **SAP:** ปุ่ม "ดึง Excel ทำ PR (SAP)" สร้างไฟล์ .xlsx 19 คอลัมน์ (Type…Department) ตามรูปแบบที่ใช้ import เข้า SAP ได้ทันที
- **ฐานข้อมูล:** เมื่อข้อมูลสะสมมาก แนะนำย้าย backend จาก Google Sheets ไป PostgreSQL/Supabase โดยหน้าเว็บ (index.html) ใช้ต่อได้ เพียงเปลี่ยนปลายทางใน `api()`
