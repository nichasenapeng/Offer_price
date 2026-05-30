[Uploading README.md…]()
# SunFood — Offer & Approval System

ระบบจัดการใบเสนอราคาและอนุมัติสำหรับทีม Export Sales ของ SunFood International

---

## ระบบประกอบด้วยสองส่วน

| ส่วน | URL | ผู้ใช้งาน |
|---|---|---|
| Offer Form | https://nichasenapeng.github.io/Offer_price/ | ทีมขาย |
| Approve Dashboard | https://nichasenapeng.github.io/Approve_price/ | หัวหน้า / ผู้อนุมัติ |

---

## วิธีใช้งาน

### ทีมขาย — Offer Form

1. เลือกชื่อผู้ใช้ (NICHA / NET / KAN / EMMA)
2. กรอก Customer: Company, Port, Incoterm, Payment Term, Order Type
3. กรอก Inquiry: สินค้า, Qty, Idea Price, Shipment, ราคาเสนอ, %Margin
4. ตรวจสอบสรุป → กด **ส่งคำขอใบเสนอราคา**
5. ระบบสร้าง REF เช่น `SFD-2026-XXXX` อัตโนมัติ

> เพิ่มสินค้าหลายรายการได้โดยกด **"เพิ่มสินค้า"** ใน Step 3

### หัวหน้า — Approve Dashboard

1. ใส่รหัสผ่าน: `362254`
2. กรองดู offer ตามสถานะ: รออนุมัติ / Approve 1 แล้ว / อนุมัติครบ / ปฏิเสธ
3. กด **Approve** หรือ **Reject** ใน slot A1 หรือ A2
4. ใส่หมายเหตุ (ถ้ามี) → ระบบบันทึกลง Google Sheets อัตโนมัติ

---

## Status Logic

| เงื่อนไข | Status |
|---|---|
| ยังไม่มีการ vote | Pending |
| Vote แล้ว 1 คน | Partial |
| A1 และ A2 = approved | Approved |
| A1 หรือ A2 = rejected | Rejected |

---

## Architecture

```
ทีมขาย → Offer Form → Apps Script 1 → Sheet 1 (Offer)
                                               │
                                               │ GET
                                               ▼
หัวหน้า ← Approve Dashboard ← Apps Script 2 ──┘
              │
              │ POST (approve/reject)
              ▼
         Apps Script 2 → Sheet 2 (Approval)
```

Apps Script 2 ทำสองหน้าที่ — GET ดึงข้อมูล Offer และ POST บันทึกผลการอนุมัติ

---

## Google Sheets

| Sheet | Link |
|---|---|
| Sheet 1 — Offer | https://docs.google.com/spreadsheets/d/15fxIAqk32f898h4nN-CG1HstFe3GPSy-33ALiJlJHwU/edit#gid=954990774 |
| Sheet 2 — Approval | https://docs.google.com/spreadsheets/d/1ZKrNVK1Mgffl1UqTGw8QxMld3hpTnlfZgpcgrwoNyHU/edit#gid=0 |

---

## Repositories

| Repo | คำอธิบาย |
|---|---|
| [Offer_price](https://github.com/nichasenapeng/Offer_price) | Offer Form (ทีมขาย) |
| [Approve_price](https://github.com/nichasenapeng/Approve_price) | Approve Dashboard (หัวหน้า) |

ทั้งสองเป็น Single-page HTML ไม่มี dependencies นอกจาก Google Fonts และ Apps Script

---

## สิ่งที่ควรรู้

- รหัสผ่านฝังอยู่ใน HTML โดยตรง — ควรอัปเดตหากต้องการเพิ่มความปลอดภัย
- Apps Script POST ใช้ `mode: no-cors` — บันทึกสำเร็จแต่ไม่ได้รับ response กลับ
- Approval state โหลดจาก Sheet ทุกครั้งที่รีเฟรช ไม่หายหากปิดหน้าต่าง
- REF สร้างแบบ random — โอกาสซ้ำต่ำมาก แต่ไม่เป็นศูนย์

---

*SunFood International · Export Sales Team · 2026*
