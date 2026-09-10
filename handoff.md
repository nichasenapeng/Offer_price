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
| **Approve** | ผู้อนุมัติ (ต้องใส่รหัสผ่าน) | Approve / Reject / **มีคำถาม** แยก 2 ช่อง A1, A2 |
| **Status** | ทีมขาย | ตามผลหลังอนุมัติ ปิดการขาย ตอบคำถาม ขอขยาย VALID ขอ bidding |
| **Summary** | ทุกคน | สรุปการปิดการขายรายวัน ปริมาณ (MT) และมูลค่า (USD) |
| **Setting** | แอดมิน | ค่าเริ่มต้นฟอร์ม · **ตารางค่าเฟรทรายเดือนแยก port** · ลิงก์ระบบ |

---

## Flow สถานะ

```
ส่งใบ → pending
          ├─ A1 หรือ A2 กด Approve       → partial → (อีกคน Approve) → approved
          ├─ ใครกด Reject                → rejected → ทีมขายกด "แก้ไขและส่งใหม่" → ใบใหม่ REF ใหม่
          │                                            หรือกด "ลบใบนี้" → deleted (ซ่อน กู้คืนได้)
          └─ ใครกด "มีคำถาม"             → question → ทีมขายตอบในใบเดิม → กลับมาให้เคาะ

approved → ทีมขายเลือก
          ├─ ปิดการขายได้     → closed-won   (บันทึก shipment/qty/ราคา/margin/remark/วันที่)
          ├─ ไม่ปิดการขาย     → closed-lost  (เสนอไปแล้วแต่ไม่ได้)
          ├─ ไม่เสนอ          → not-offered  (ไม่ได้ส่งราคาให้ลูกค้าเลย)
          ├─ ขอขยาย VALID     → ext-pending → ผู้อนุมัติยืนยัน/ไม่อนุมัติ (REF เดิม ไม่ออกใบใหม่)
          │                     ระหว่างรอ ทีมขายกด "แก้ไขคำขอ" หรือ "ยกเลิกคำขอ" ได้
          └─ ลูกค้าขอ bidding → ใบเดิมเป็น superseded + เปิดฟอร์มใหม่พร้อมข้อมูลเดิม
```

**ระหว่างมีคำถามค้าง ปิดการขายไม่ได้** — ตั้งใจให้เป็นแบบนั้น

**ใบที่เลือก "ให้ไอเดียราคา" ขอขยาย VALID ไม่ได้** — เพราะไม่ได้ยืนราคาไว้ตั้งแต่ต้น

**คำขอขยาย VALID แก้ได้จนกว่าจะถูกเคาะ** — ระหว่าง `ext-pending` ทีมขายกด "แก้ไขคำขอ"
เพื่อเปิดฟอร์มเดิมที่กรอกค่าไว้แล้ว หรือ "ยกเลิกคำขอ" เพื่อกลับไปเป็น `approved`
ช่องเหตุผลเป็น textarea ขึ้นบรรทัดได้ และผู้อนุมัติมีช่อง "หมายเหตุถึงทีมขาย" ตอนเคาะ
เก็บเป็นช่องที่ 4 ของ `ext-*` (`extNote`) แถวเก่าที่มีแค่ 3 ช่องอ่านได้เหมือนเดิม

**"ลบ" คือซ่อน ไม่ใช่ลบจริง** — ใบที่ถูกปฏิเสธกดลบได้ (มี confirm) จะกลายเป็นสถานะ
`deleted` ซึ่ง `getStatus()` คืนก่อนทุกสถานะ ทำให้หายจากทุกรายการและไม่ถูกนับใน "ทั้งหมด"
แต่**แถวในชีทยังอยู่ครบ** ดูและกู้คืนได้จากปุ่มกรอง "ถังขยะ" ทั้งฝั่งผู้อนุมัติและเช็คสถานะ

ลบได้เฉพาะสถานะ `rejected` — `deleteOffer()` เช็คซ้ำก่อนทำงาน ต่อให้เรียกจากที่อื่นก็ไม่ผ่าน
ถ้าอยากลบแถวออกจากชีทจริง ต้องแก้ Apps Script ฝั่ง server ซึ่งตอนนี้เขียนได้แค่คอลัมน์สถานะ

**`closed-lost` กับ `not-offered` ไม่เหมือนกัน** — อันแรกคือเสนอไปแล้วลูกค้าไม่เอา
อันหลังคือไม่ได้เสนอเลย แยกไว้เพื่อไม่ให้ตัวเลข "แพ้ดีล" เพี้ยน
ทั้งคู่เก็บแค่ remark + วันที่ ไม่มีตัวเลขปิดการขาย และกด "แก้ไข" กลับเป็น `approved` ได้
สรุปยอดขาย (`renderSummary_`) นับเฉพาะ `closed-won` ทั้งสองสถานะนี้จึงไม่กระทบยอด

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
[KIND: idea | DATE: 10/09/2026 | VALID: 7 วัน | FRT: 3000 | MTC: 24 | FX: 32] หมายเหตุที่ทีมขายพิมพ์
```
`KIND` จะโผล่เฉพาะใบที่เลือก "ให้ไอเดียราคา" — ใบเก่าและใบเสนอราคาปกติไม่มีคีย์นี้ อ่านได้เหมือนเดิม

คอลัมน์ `สถานะรวม` เก็บสถานะแบบมี payload คั่นด้วย `|`
```
closed-won|shipment|qty|price|margin|remark|YYYY-MM-DD
closed-lost|||||remark|YYYY-MM-DD
not-offered|||||remark|YYYY-MM-DD
ext-pending|VALID ใหม่|เหตุผล
ext-approved|VALID ใหม่|เหตุผล|หมายเหตุจากผู้อนุมัติ      ext-rejected|เหมือนกัน
question|A1                         question|A1|คำตอบจากทีมขาย
superseded|เหตุผล
deleted|YYYY-MM-DD
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

**4.5 %Margin ในชีทมีหลายรูปแบบปนกัน** — ช่องกรอกเป็น free text และ Google Sheets
อ่าน `"25.33%"` ที่พิมพ์มาเป็นตัวเลข `0.2533` แล้วคืนค่านั้นกลับมา การ์ดเลยเคยขึ้น `0.2533%`

ตอนนี้กันไว้สองชั้น
- ตอนส่ง `pctPlain_()` ตัด `%` ออกถ้าค่าเป็นตัวเลขล้วน ชีทจะเก็บ `25.33` ตรง ๆ
- ตอนแสดง `fmtMargin_()` / `mgNum_()` แปลงค่าที่ไม่มี `%` และต่ำกว่า 1 เป็นเปอร์เซ็นต์ (×100)
  ข้อความอิสระอย่าง `11% (-3% เทียบ BLK)` หรือ `x` แสดงตามที่พิมพ์ ไม่เติม `%` ต่อท้าย

ถ้ามีวันที่ margin ต่ำกว่า 1% จริง ให้พิมพ์ `0.5%` (มี `%`) จะไม่ถูกคูณ 100
แถวเก่าในชีทไม่ได้ถูกแก้ ใช้การแปลงตอนแสดงผลเอา

**5. ตารางค่าเฟรท** อยู่ในตัวแปร `DEFAULT_FREIGHT` (47 port, 12 กลุ่ม)
แก้ใน Setting = เก็บใน localStorage ของเครื่องนั้นเท่านั้น
ถ้าจะอัปเดตให้ทั้งทีม ต้องแก้ `DEFAULT_FREIGHT` ในโค้ดแล้ว deploy

**6. ใบเสนอราคา vs ใบให้ไอเดียราคา** — เลือกที่หัว Step 2 เป็นของ**ทั้งใบ** ไม่ใช่รายสินค้า
เก็บในตัวแปร `offerKind` (`'offer'` | `'idea'`) สลับด้วย `setOfferKind()`

โหมด `idea` ต่างจากปกติ 4 อย่าง:
- ซ่อนช่อง DATE และ VALID (คลาส `.fld-offer-only`) และล้างค่าใน `getProdData()` กันค่าค้างจากตอนสลับโหมด
- `Shipment ที่จะเสนอ` ไม่บังคับกรอก (ข้ามเช็คทั้งใน `goNext(2)` และ `submitForm()`)
- ป้ายชื่อเปลี่ยนด้วย CSS ล้วน — `.lbl-offer` / `.lbl-idea` คู่กัน สลับตามคลาส `.kind-idea` บน `#products-container`
  (การ์ดสินค้าถูกสร้างด้วย JS ทีหลังได้เรื่อย ๆ การใช้ CSS จึงครอบคลุมทั้งการ์ดเดิมและการ์ดใหม่)
- ฝั่งผู้อนุมัติ/เช็คสถานะ ขึ้นป้าย "ไอเดียราคา" ที่หัวการ์ด เปลี่ยนคำ "ราคาเสนอ" เป็น "ไอเดียราคา"
  และซ่อนปุ่ม "ขอขยาย VALID" เพราะใบไอเดียไม่มี VALID ให้ขยาย

โหมดเป็นของทั้งใบ แต่ `savedSessions` ถูกบันทึกไว้ตอนกด "เพิ่มสินค้า" ซึ่งอาจเป็นคนละโหมด
`submitForm()` จึงเขียนโหมดปัจจุบันทับทุก session ก่อนส่ง

---

## วิธี deploy

**ทำเมื่อผู้ใช้สั่งเท่านั้น และต้องขึ้นทั้งสองที่**

**ขั้นตอนต่างกันตามเครื่อง — เช็คก่อนว่ามี `gh` / `node` ไหม**

**GitHub** — repo คือ `nichasenapeng/Offer_price`

เครื่องที่มี `gh` และ active เป็น `aiengsunfood` (push ไม่ได้) ต้องสลับก่อน:
```bash
gh auth switch -u nichasenapeng
git -c credential.helper='!gh auth git-credential' push origin main
gh auth switch -u aiengsunfood
```
เครื่องที่ไม่มี `gh` แต่มี credential ของ nichasenapeng ใน macOS keychain
(`credential.helper = osxkeychain`) push ตรงได้เลย ไม่ต้องสลับอะไร:
```bash
git push origin main
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
เช็คว่าได้บัญชีถูกด้วย `npx wrangler whoami` — ต้องขึ้น account ID `186b05df...`

**อย่ารัน `wrangler login` โดยไพป์ผ่าน `tail` หรือ `head`** — มันจะ buffer จนจบ process
ทำให้ลิงก์ authorize ไม่โผล่ระหว่างที่ยังกดทัน

เครื่องที่ไม่มี Node เลย (ไม่มี `node` / `npx` / Homebrew) ต้องให้ผู้ใช้ลง Node LTS
จาก nodejs.org ก่อน เพราะขั้นตอนนี้ต้องใช้รหัสเครื่อง ทำแทนไม่ได้
บนเครื่องแบบนั้น token จะไปอยู่ที่ `~/Library/Preferences/.wrangler/config/default.toml`

เสร็จแล้วเทียบ md5 ของทั้งสอง URL กับไฟล์ในเครื่องก่อนบอกว่าเสร็จ

---

## ผู้ใช้

**Export Sales Team** — NICHA · NET · KAN · EMMA · MAI
**Food Supply Chain** — MAY · MILK · ANCHAN · TIK · ICE
**ผู้อนุมัติ** — Approver 1 (A1), Approver 2 (A2) · รหัสผ่าน `approvexx` (ตัวแปร `PASS` ในโค้ด)
เปลี่ยนรหัส = แก้ `PASS` แล้ว deploy — โหมด Status และ Summary ไม่ต้องใช้รหัส เข้าได้เลยเหมือนเดิม

---

## ข้อจำกัดที่ยังอยู่

- **รหัสผ่านผู้อนุมัติกันได้แค่คนกดผิดหน้า ไม่ใช่ระบบยืนยันตัวตน** — ไฟล์นี้เป็น HTML ล้วน
  ที่เปิดสาธารณะ ใครกด View Source ก็เห็นค่า `PASS` ตรง ๆ และรหัสเดียวใช้ร่วมกันทั้ง A1 และ A2
  ระบบยังแยกไม่ได้ว่าใครเป็นคนกด ถ้าต้องการของจริงต้องย้ายการตรวจสอบไปฝั่ง Apps Script
- ค่าใน Setting เก็บแยกตามเครื่อง ไม่ sync ข้ามผู้ใช้
- ใบที่ส่งก่อน 10 ก.ย. 2026 และมีหลายสินค้า จะไม่เห็นหมายเหตุ (ข้อมูลค้างอยู่แถวหลัง)
- REF สุ่มด้วย `Math.random()` โอกาสซ้ำต่ำแต่ไม่เป็นศูนย์
