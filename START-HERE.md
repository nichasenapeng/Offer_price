# เปิดโปรเจกต์นี้ในเครื่องใหม่

1. แตกไฟล์ zip แล้วเปิด Claude Code ที่โฟลเดอร์ `Offer_price`
2. อ่าน `handoff.md` ก่อน — อธิบายระบบทั้งหมด, flow สถานะ, จุดที่ต้องระวังก่อนแก้โค้ด และวิธี deploy
3. ทดสอบในเครื่อง: เปิด `index.html` ด้วยเบราว์เซอร์ได้เลย (ไม่ต้อง build ไม่ต้องลง dependency)
   ดูพร้อมข้อมูลตัวอย่าง: เปิดด้วย `?demo=1`

## ในซิปนี้มีอะไร

| ไฟล์ | คืออะไร |
|---|---|
| `index.html` | ทั้งระบบอยู่ในไฟล์นี้ไฟล์เดียว |
| `handoff.md` | คู่มือระบบฉบับล่าสุด — อ่านก่อนแก้ |
| `Code.gs` | สำเนา Apps Script ฝั่ง Offer (ตัวที่ deploy จริงใหม่กว่า อย่า copy ทับทั้งไฟล์) |
| `README.md` | บันทึกเก่าจากตอนเริ่มโปรเจกต์ |
| `.git/` | ประวัติทั้งหมด + remote — push ต่อจากเครื่องใหม่ได้เลย |
| `.claude/memory/` | ความจำของ Claude เรื่องกติกาการ deploy และบัญชีที่ใช้ |
| `.claude/launch.json` | ตั้งค่า preview server |

## สิ่งที่ไม่ได้มาในซิป (ต้องตั้งใหม่ในเครื่องนั้น)

- **สิทธิ์ push GitHub** — repo คือ `nichasenapeng/Offer_price` ต้อง login เป็น `nichasenapeng`
- **token Cloudflare** — ต้อง `npx wrangler login` เป็นบัญชีของ nicha
- ทั้งสองเรื่องมีขั้นตอนละเอียดใน `handoff.md` หัวข้อ "วิธี deploy"

## ความจำที่อยากให้ Claude เครื่องใหม่รู้

ไฟล์ใน `.claude/memory/` เป็นรูปแบบความจำของ Claude Code
ถ้าเครื่องใหม่ไม่ได้อ่านอัตโนมัติ ให้บอก Claude ว่า "อ่าน .claude/memory/ แล้วจำไว้"
สาระสำคัญคือ: **แก้เสร็จอย่าเพิ่ง deploy รอให้สั่งก่อน แล้วเวลา deploy ต้องขึ้นทั้ง GitHub Pages และ Cloudflare**
