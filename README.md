# 🏷️ สแกนขอป้ายราคา → Google Sheet

ยิงสแกนบาร์โค้ดด้วยเครื่อง Handheld → แสดง **Art No + Descr** ทันทีจากฐานข้อมูลสินค้า → **ข้อมูลเข้า Google Sheet ทันที**
สแกนสินค้าเดิม ขนาดเดิม ซ้ำ = **เพิ่มจำนวนป้าย** ในแถวเดิม
ไม่มีหน้าตั้งค่า ไม่ต้องกดส่ง — เลือกขนาดป้าย **A3 / A4 / A5 / A6** ได้ตลอดด้วยปุ่มบนจอหรือ **F1–F4**

## โครงสร้างไฟล์
```
index.html                          ← เว็บแอป (วางบน GitHub Pages)
products.json                       ← ฐานข้อมูลสินค้า (สร้างจาก DataItem.xlsx)
data/DataItem.xlsx                  ← ไฟล์ต้นฉบับ Dept no | Barcode | Art No | Descr
tools/build_products.py             ← สคริปต์แปลง xlsx → products.json
.github/workflows/build-products.yml← อัปเดต products.json อัตโนมัติเมื่ออัปโหลด xlsx ใหม่
apps-script/Code.gs                 ← โค้ดฝั่ง Google Sheet
```

---

## ขั้นที่ 1: Google Sheet + Apps Script
1. สร้าง Google Sheet ใหม่ → เมนู **ส่วนขยาย → Apps Script**
2. วางโค้ดจาก `apps-script/Code.gs` → บันทึก
3. เลือกฟังก์ชัน `setup` → **Run** → อนุญาตสิทธิ์
   จะได้ชีต `Requests` (รายการขอป้าย), `Summary` (สรุปป้ายรอพิมพ์) และ `ScanLog` (ประวัติทุกการสแกน — ซ่อนไว้)
4. **Deploy → New deployment** → Web app → Execute as: **Me** → Who has access: **Anyone** → Deploy
5. คัดลอก **Web app URL** (ลงท้าย `/exec`)

> แก้ `Code.gs` เมื่อไหร่ ต้อง Deploy → Manage deployments → Edit → **New version** ทุกครั้ง

## ขั้นที่ 2: ใส่ URL ใน index.html
เปิด `index.html` แก้ส่วน `CONFIG` ด้านบนไฟล์:
```js
SHEET_URL: "https://script.google.com/macros/s/xxxxxxxx/exec",
BRANCH: "สาขาสุพรรณบุรี",
DEFAULT_SIZE: "A4",
```

## ขั้นที่ 3: GitHub Pages
1. สร้าง repository → อัปโหลด**ทุกไฟล์ในโฟลเดอร์** (รวม `products.json`, `data/`, `tools/`, `.github/`) ไว้ที่ root ของ `main`
   (โฟลเดอร์ `.github` ซ่อนอยู่ — ถ้าลากไฟล์ผ่านเว็บไม่ขึ้น ให้ใช้ GitHub Desktop หรือสร้างไฟล์ `.github/workflows/build-products.yml` เองแล้ววางเนื้อหา)
2. **Settings → Pages** → Deploy from a branch → `main` / `(root)` → Save
3. ได้ลิงก์ `https://<username>.github.io/<repo>/`

**หลายสาขาใช้ไฟล์เดียวกันได้** — ใส่ชื่อสาขาท้ายลิงก์ เช่น
`https://<username>.github.io/<repo>/?branch=สาขาลาดพร้าว`
(เพิ่ม `&size=A5` เพื่อกำหนดขนาดเริ่มต้นของเครื่องนั้น)

---

## ตั้งค่าเครื่อง Handheld (ครั้งเดียว)
ให้หัวสแกนส่งข้อมูลแบบ **คีย์บอร์ด (Keystroke / Keyboard Wedge)** และ **ต่อท้ายด้วย Enter**

| ยี่ห้อ | ตำแหน่งเมนู (อาจต่างตามรุ่น) |
|---|---|
| Zebra | DataWedge → Profile ผูกกับ Chrome → Keystroke output: Enabled → Send ENTER key ✓ |
| Honeywell | Settings → Scan Settings → Internal Scanner → Default profile → Data Processing → Wedge Method: Keyboard, Suffix: `\n` |
| Urovo | Scanner Settings → Output Mode: Keyboard → Append Enter ✓ |
| Sunmi | Settings → Scanner → Output mode: Simulate keyboard input → Suffix: Enter |
| iData / อื่น ๆ | ScanWedge → Output: Keyboard → Suffix: Enter |

ทดสอบ: เปิดแอปโน้ตแล้วยิง — ตัวเลขขึ้นและขึ้นบรรทัดใหม่เอง = พร้อมใช้
จากนั้นเปิดลิงก์ใน Chrome → ⋮ → **เพิ่มลงในหน้าจอหลัก**

## การใช้งาน
- ยิงได้เลย ไม่ต้องแตะจอ (จุดเขียว = เชื่อมต่อพร้อม)
- **F1/F2/F3/F4 = A3/A4/A5/A6** หรือแตะปุ่มด้านบน
- ปุ่ม − / + ตั้ง "จำนวนป้ายต่อการสแกน" ใช้กับทุกสแกนถัดไป
- เสียง: ปิ๊บสั้น = บันทึกแล้ว · ปิ๊บสองครั้ง = ไม่พบในฐานข้อมูล / ต้องเลือก Art No · เสียงยาว = ยังส่งไม่ได้
- **ไม่มีสัญญาณ:** สแกนต่อได้ ข้อมูลค้างในเครื่องแล้วส่งเองอัตโนมัติเมื่อกลับมามีสัญญาณ
- **สแกนซ้ำ = เพิ่มจำนวน**: Art No เดียวกัน + ขนาดเดียวกัน รวมเป็นแถวเดียว (บาร์โค้ดต่างกันแต่ Art No เดียวกันก็รวมกัน) ขนาดต่างกันแยกแถว
- ยิงเกิน → กด **−** ที่รายการ ลดทีละ 1 ป้าย (เหลือ 0 = สถานะในชีตเป็น "ยกเลิก")
- เมื่อเปลี่ยนสถานะในชีตเป็น "พิมพ์แล้ว" แล้ว การสแกนสินค้านั้นครั้งต่อไปจะขึ้นแถวใหม่

## อัปเดตฐานข้อมูลสินค้า
1. ใน GitHub เปิดโฟลเดอร์ `data` → **Add file → Upload files** → อัปโหลด `DataItem.xlsx` ไฟล์ใหม่ (ชื่อเดิม) → Commit
2. GitHub Actions จะสร้าง `products.json` ใหม่ให้เอง (~1 นาที ดูได้ที่แท็บ **Actions**)
   ครั้งแรกอาจต้องเปิดสิทธิ์: Settings → Actions → General → Workflow permissions → **Read and write**
3. เครื่อง Handheld จะโหลดฐานข้อมูลใหม่เองเมื่อเปิดแอปครั้งถัดไป (ดูวันที่อัปเดตได้ใต้ช่องสแกน)

> ไฟล์ต้องมีหัวคอลัมน์ `Dept no | Barcode | Art No | Descr` เหมือนเดิม
> ฐานข้อมูลเก็บไว้ในเครื่อง ใช้ค้นหาได้แม้ไม่มีสัญญาณ

## การค้นหาสินค้า
- ค้นจากบาร์โค้ด (ไม่สนใจเลข 0 นำหน้า เพราะ Excel มักตัดทิ้ง) ถ้าไม่พบจะลองค้นเป็น **Art No**
- บาร์โค้ดที่มีหลาย Art No (มี ~30 ตัวในไฟล์) จะขึ้นให้**แตะเลือก** ถ้าไม่เลือกแล้วยิงตัวถัดไป จะใช้ตัวแรกให้
- ไม่พบในฐานข้อมูล → ยังบันทึกลงชีต (Art No/Descr ว่าง) พร้อมเสียงเตือน

## ในชีต
- `Requests` — 1 แถวต่อ สินค้า+ขนาด ที่รอพิมพ์: สแกนครั้งแรก, สแกนล่าสุด, สาขา, บาร์โค้ด, Art No, Descr, Dept, ขนาด, **จำนวน**, หมายเหตุ (กรอกในชีตเองได้), สถานะ
- `ScanLog` — ทุกการกดสแกน/ลดจำนวน (ชีตซ่อน: คลิกขวาที่แถบชีต → แสดง)
  สถานะเปลี่ยนได้จาก dropdown: รอพิมพ์ / พิมพ์แล้ว / ติดแล้ว / ยกเลิก
- `Summary` — รวมป้ายที่ "รอพิมพ์" ตาม Art No แยกตามขนาด เรียงตาม Dept ใช้เป็นใบสั่งพิมพ์ได้เลย
