# SunFood — Offer & Approval System

**Handoff** · อัปเดต 10 กันยายน 2026

ระบบเสนอราคา–อนุมัติ–ปิดการขาย ของทีม Export Sales และ Food Supply Chain
ทั้งระบบอยู่ในไฟล์เดียว: `index.html` (ไม่มี build step ไม่มี dependency นอกจาก Google Fonts)

---

## URL ใช้งานจริง

| ที่ | ลิงก์ | อัปเดตยังไง |
|---|---|---|
| GitHub Pages | https://nichasenapeng.github.io/Offer_price/ | push แล้ว build เอง |
| Cloudflare Worker | https://sunfood-offer.nichasenapeng13.workers.dev | ต้อง `wrangler deploy` เอง |

**ลิงก์เข้าโหมดตรง** — ต่อท้าย `?mode=offer` · `approve` · `status` · `summary` · `setting`
**ดูพร้อมข้อมูลตัวอย่าง** — ต่อท้าย `?demo=1` (ใช้ร่วมกับ `mode` ได้)

---

## 5 โหมดในไฟล์เดียว

หน้าแรกเป็นการ์ดสี่เหลี่ยมให้เลือกโหมด กดโลโก้ SUNFOOD ที่มุมซ้ายบนเพื่อกลับหน้านี้เสมอ

| โหมด | ใคร | ทำอะไร |
|---|---|---|
| **Offer** | ทีมขาย | กรอกใบเสนอราคา 3 ขั้น (Customer → Inquiry → สรุป & ส่ง) |
| **Approve** | ผู้อนุมัติ (ไม่ต้องใช้รหัส กด Enter เข้าได้เลย) | Approve / Reject / **มีคำถาม** แยก 2 ช่อง A1, A2 |
| **Status** | ทีมขาย | ตามผลหลังอนุมัติ ปิดการขาย ตอบคำถาม ขอขยาย VALID ขอ bidding |
| **Summary** | ทุกคน | สรุปการปิดการขายรายวัน ปริมาณ (MT) และมูลค่า (USD) |
| **Setting** | แอดมิน | ค่าเริ่มต้นฟอร์ม · **ตารางค่าเฟรทรายเดือนแยก port** · ลิงก์ระบบ |

---

## Flow สถานะ

```
ส่งใบ → pending
          ├─ A1 หรือ A2 กด Approve       → partial → (อีกคน Approve) → approved
          ├─ ใครกด Reject                → rejected → ทีมขายกด "แก้ไขและส่งใหม่" → ใบใหม่ REF ใหม่
          └─ ใครกด "มีคำถาม"             → question → ทีมขายตอบในใบเดิม → กลับมาให้เคาะ

approved → ทีมขายเลือก
          ├─ ปิดการขายได้     → closed-won   (บันทึก shipment/qty/ราคา/margin/remark/วันที่)
          ├─ ไม่ปิดการขาย     → closed-lost
          ├─ ขอขยาย VALID     → ext-pending → ผู้อนุมัติยืนยัน/ไม่อนุมัติ (REF เดิม ไม่ออกใบใหม่)
          └─ ลูกค้าขอ bidding → ใบเดิมเป็น superseded + เปิดฟอร์มใหม่พร้อมข้อมูลเดิม
```

**ระหว่างมีคำถามค้าง ปิดการขายไม่ได้** — ตั้งใจให้เป็นแบบนั้น

---

## Backend: Google Sheets + Apps Script 2 ตัว

**Script 1 — เขียนใบเสนอราคา** (ผูกกับ Sheet "EXPORT SALES _ OFFER", tab `Inquiry Requests`)
`https://script.google.com/macros/s/AKfycbyAfBG-BNJfw7y7MnbC6t4zC4Vckewu-VGQBBgSnEhAXqHEYRyFxjmKuvoLZxpM-Lw/exec`
รับ GET `?data=<JSON>` → เขียน 1 แถวต่อ 1 สินค้า
สำเนาโค้ดอยู่ใน `Code.gs` แต่ **ตัวที่ deploy จริงใหม่กว่า อย่า copy ทับทั้งไฟล์**

**Script 2 — อ่านทั้งหมด + เขียนผลอนุมัติ** (ผูกกับ Sheet "EXPORT SALES _ APPROVE")
`https://script.google.com/macros/s/AKfycbzaa16CiQbLUhCNTiIkVSuFhCfEgrYsz342t6GmOc1c-IY3KiOEmvS4ZWXcZHwMAtTS/exec`
- GET → คืนใบทั้งหมด จัดกลุ่มตาม REF พร้อม `products[]`
- POST (no-cors) → บันทึก A1/A2/หมายเหตุ/สถานะรวม

Sheet: [Offer](https://docs.google.com/spreadsheets/d/15fxIAqk32f898h4nN-CG1HstFe3GPSy-33ALiJlJHwU/edit) · [Approval](https://docs.google.com/spreadsheets/d/1ZKrNVK1Mgffl1UqTGw8QxMld3hpTnlfZgpcgrwoNyHU/edit)

---

## จุดที่ต้องรู้ก่อนแก้โค้ด

**1. ข้อมูลบางอย่างฝากไว้ในคอลัมน์ที่มีอยู่ เพราะเพิ่มคอลัมน์ในชีทไม่ได้**

คอลัมน์ `หมายเหตุ` เก็บ meta ไว้ต้นข้อความ แล้วแกะออกตอนอ่านด้วย `stripMeta_()`
```
[DATE: 10/09/2026 | VALID: 7 วัน | FRT: 3000 | MTC: 24 | FX: 32] หมายเหตุที่ทีมขายพิมพ์
```

คอลัมน์ `สถานะรวม` เก็บสถานะแบบมี payload คั่นด้วย `|`
```
closed-won|shipment|qty|price|margin|remark|YYYY-MM-DD
closed-lost|||||remark|YYYY-MM-DD
ext-pending|VALID ใหม่|เหตุผล      ext-approved|... ext-rejected|...
question|A1                         question|A1|คำตอบจากทีมขาย
superseded|เหตุผล
```
ค่าที่ผู้ใช้พิมพ์จะถูกล้าง `|` ออกก่อนเสมอ เพื่อไม่ให้ข้อมูลเพี้ยน

**2. ส่งข้อมูลต้อง encode 2 ชั้น**
`encodeURIComponent(encodeURIComponent(json))` — เพราะ Apps Script decode ซ้ำอีกรอบ
ถ้า encode ชั้นเดียว ค่าที่มี `%` (เช่น "48.6%") จะทำให้ server พังทั้งแถวแบบเงียบ ๆ

**3. ใช้ `fetch` ไม่ใช่ `Image` beacon**
หน้า "ส่งสำเร็จ" ขึ้นเฉพาะเมื่อ server ตอบ ok จริง ถ้าล้มเหลวจะเตือนให้ส่งใหม่

**4. หมายเหตุรวมต้องติดไปทุกสินค้า**
`submitForm()` เขียน `overallRemark` ลงทุก session — เพราะ Script 2 อ่านค่าจากแถวแรกของ REF
ถ้าติดเฉพาะสินค้าชิ้นสุดท้าย ผู้อนุมัติจะไม่เห็นหมายเหตุเลย

**5. ตารางค่าเฟรท** อยู่ในตัวแปร `DEFAULT_FREIGHT` (47 port, 12 กลุ่ม)
แก้ใน Setting = เก็บใน localStorage ของเครื่องนั้นเท่านั้น
ถ้าจะอัปเดตให้ทั้งทีม ต้องแก้ `DEFAULT_FREIGHT` ในโค้ดแล้ว deploy

---

## วิธี deploy

**ทำเมื่อผู้ใช้สั่งเท่านั้น และต้องขึ้นทั้งสองที่**

**GitHub** — repo คือ `nichasenapeng/Offer_price` แต่ `gh` ในเครื่อง active เป็น `aiengsunfood` ซึ่ง push ไม่ได้
```bash
gh auth switch -u nichasenapeng
git -c credential.helper='!gh auth git-credential' push origin main
gh auth switch -u aiengsunfood
```

**Cloudflare** — worker `sunfood-offer` อยู่บนบัญชีของ nicha (`186b05df927d0192ed92ec799dc1b2a0`)
copy `index.html` ไปเป็น `public/index.html` คู่กับ `wrangler.toml`:
```toml
name = "sunfood-offer"
compatibility_date = "2026-01-01"
[assets]
directory = "./public"
```
```bash
CLOUDFLARE_ACCOUNT_ID=186b05df927d0192ed92ec799dc1b2a0 npx wrangler deploy
```
token หมดอายุบ่อย และเวลา refresh เองมันเด้งกลับไปเป็นบัญชี ai.eng.sunfood ซึ่งเข้า worker ของ nicha ไม่ได้
ต้อง `npx wrangler login` ใหม่ แล้วให้ผู้ใช้กด Allow **ภายใน 2 นาที** โดยเบราว์เซอร์ต้อง sign in เป็น nicha อยู่ก่อน

เสร็จแล้วเทียบ md5 ของทั้งสอง URL กับไฟล์ในเครื่องก่อนบอกว่าเสร็จ

---

## ผู้ใช้

**Export Sales Team** — NICHA · NET · KAN · EMMA · MAI
**Food Supply Chain** — MAY · MILK · ANCHAN · TIK · ICE
**ผู้อนุมัติ** — Approver 1 (A1), Approver 2 (A2) · ไม่มีรหัสผ่านแล้ว ใครมีลิงก์เข้าได้

---

## ข้อจำกัดที่ยังอยู่

- ไม่มีระบบยืนยันตัวตนจริง ใครมีลิงก์กด Approve ได้
- ค่าใน Setting เก็บแยกตามเครื่อง ไม่ sync ข้ามผู้ใช้
- ใบที่ส่งก่อน 10 ก.ย. 2026 และมีหลายสินค้า จะไม่เห็นหมายเหตุ (ข้อมูลค้างอยู่แถวหลัง)
- REF สุ่มด้วย `Math.random()` โอกาสซ้ำต่ำแต่ไม่เป็นศูนย์
