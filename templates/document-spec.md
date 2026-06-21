# 📐 Canonical Document Spec — ใบประเมินราคา (Quotation)

> **แหล่งความจริงเดียว (single source of truth)** ของโครงเอกสารใบประเมิน
> ทั้ง **`output.html`** (ที่ agent gen) และ **เว็บ React** (`ProjectView`) **ต้องยึดไฟล์นี้เป๊ะ** —
> ห้าม improvise โครง/ลำดับ/เนื้อหาเอง ทุกค่าต้องมาจาก `data.json` (ชื่อ field ตรงตาม schema)
>
> หลักการ: **ฉบับเต็ม = `output.html` ≡ React `audience="internal"`** · **React `audience="client"` = ฉบับเต็มลบส่วนที่ทำเครื่องหมายซ่อน**

---

## 1. โครงเอกสาร (ลำดับตายตัว)

ส่วนหัว (ไม่นับเลข section):

| บล็อก | เนื้อหา | data.json fields | client เห็น? |
|---|---|---|---|
| **Header (hero)** | kicker, ชื่อโครงการ, ลูกค้า, วันที่, ผู้จัดทำ, เวอร์ชัน | `projectName, client, date, preparedBy, docVersion` | ✅ — ตัด role ในวงเล็บของ `preparedBy`, ซ่อน `docVersion`/"draft", kicker = "ข้อเสนอโครงการ" |
| **Nav (สารบัญ sticky)** | ลิงก์ทุก section ที่โชว์ + pin ติดบนจอ | (auto) | ✅ |
| **Price hero** | ช่วงราคารวม (ก่อน VAT) | `summary.priceLow / priceHigh` | ✅ |

Section หลัก (เลข 1–9 ตายตัว · `output.html` และ React internal ต้องเรียง id ตรงกันทุกตัว):

| # | id | หัวข้อ | data.json fields | client |
|---|---|---|---|---|
| 1 | `overview` | ภาพรวม & เป้าหมายธุรกิจ | `overview`, `businessGoal{problem, successMetric}` | ✅ เต็ม |
| 2 | `scope` | ขอบเขตงาน (in/out) | `scope.in[]`, `scope.out[]` | ✅ เต็ม |
| 3 | `actors` | ผู้ใช้ & บทบาท | `actors[]{role, can}` | ✅ เต็ม |
| 4 | `usecases` | Use Case หลัก | `useCases[]{title, actor?, desc}` | ✅ เต็ม |
| 5 | `modules` | โมดูล / ฟีเจอร์ | `modules[]`, `groupLegend` | ⚠️ เห็นแค่ `name`+`desc` — ซ่อนคอลัมน์ `group`,`code`,`complexity`,`mdLow`,`mdHigh`, แถว subtotal, `groupLegend` |
| 6 | `nonfunc` | ความต้องการ Non-functional | `nonFunctional[]{key, label, detail?}` | ✅ เต็ม |
| 7 | `timeline` | Timeline & เทียบ Stack | `timeline{teamAssumption, elapsedNote, stacks[], verdict}` | ❌ **ซ่อนทั้ง section** |
| 8 | `cost` | ค่าใช้จ่าย & ราคา | `support[]`, `rates[]`, `contingencyPct`, `phasing`, **`ai{}`** + คำนวณด้วย `totals()`,`sumMd()`,`aiTotals()` | ⚠️ เห็นแค่กล่องสรุปช่วงราคา — ซ่อนตาราง support/man-day/ตารางเรต, `phasing` **และบล็อก AI ทั้งหมด** |
| 9 | `notes` | สมมติฐาน · ความเสี่ยง · คำถาม · สิ่งที่ไม่รวมราคา | `assumptions[]`, `risks[]`, `openQuestions[]`, `outOfPrice[]` | ✅ — clean รหัส `(M-0x)` ออกจาก free-text, หัวข้อ "คำถามที่ต้องถามลูกค้าเพิ่ม" → "ข้อมูลที่ต้องการเพิ่มเติม" |

**กติกาเลข section:** internal เห็น 1–9 ครบ · client ซ่อน §7 (timeline) แล้ว **รันเลขใหม่ต่อเนื่อง** (client จะมี 8 section เลข 1–8)
**ห้ามใส่:** ตาราง "วิเคราะห์รูปที่ 1–3" / เนื้อหา intake เฉพาะงาน — ไม่ใช่ section มาตรฐาน

### บล็อกย่อย "ทางเลือก: ให้ AI เขียนโค้ด" (อยู่ใน §8 `cost` — ไม่ใช่ section ใหม่)
> โหมดประเมินที่ 2 — เทียบ **คน vs AI** ในใบเดียว · ฐานคน (§1–9, `modules/support/rates`) **ไม่แตะ** เพิ่มเฉพาะบล็อกนี้
- **ตำแหน่ง:** ท้าย §8 `cost` ฝั่ง internal (หลัง `phasing`) · **คงเลข section 1–9 เดิม ไม่เพิ่ม section**
- **เงื่อนไขแสดง:** เมื่อ `p.ai` มีค่า **และ `audience==="internal"` เท่านั้น** → **client ไม่เห็น** (เหมือน §7 timeline) เอกสารลูกค้าเดิมไม่เปลี่ยน
- **เนื้อหา (จาก `ai{}`):** การ์ดเทียบ คน vs AI (man-day/เวลา/ราคา) · ตาราง AI man-day→ราคา (`aiTotals()`+`ai.rates`) · ตารางตัวเร่งราย module (ผูก `ai.modules[].code` กับ `modules[]` เดิม) · `ai.assumptions[]` · `ai.risks[]`
- **กฎเหล็ก:** AI man-day ทุกตัวต้องมี `factor` กำกับ (ที่มาของวันที่ลด) · เรต `ai.rates` = เรตคนเท่าเดิม

---

## 2. พฤติกรรม Navigation (ทั้งสองต้องเหมือนกัน)

- **Sticky/pin** — แถบสารบัญ pin ติดบนจอเมื่อเลื่อนพ้น hero (`position: sticky; top: 0`) → กดข้าม section ได้ทันที ไม่ต้องไถกลับขึ้นบน
- **Scroll-spy** — ไฮไลต์ลิงก์ของ section ที่กำลังอ่าน (IntersectionObserver)
- **Smooth scroll** + section มี `scroll-margin-top` กันหัวข้อโดนแถบ sticky บัง
- **Mobile** — เมื่อ pin ยุบเป็นแถวเดียวเลื่อนแนวนอน (`overflow-x:auto; white-space:nowrap`)
- **ปุ่ม "↑ บนสุด"** เสริม
- **Print** — `nav` กลับเป็น static / ซ่อน (กันเพี้ยนตอนพิมพ์ PDF)

---

## 3. Design tokens (สีต้องตรงกันทั้ง HTML CSS vars และ React hex)

| token | hex | ใช้ |
|---|---|---|
| `--teal` | `#3E7C76` | สีเน้นหลัก |
| `--teal-dark` | `#2F605B` | หัวข้อ/หัวตาราง |
| `--amber` | `#C98A3B` | accent / price hero |
| `--bg` | `#F4F1EA` | พื้นหลังหน้า |
| `--card` | `#FBFAF6` | พื้นการ์ด/section |
| `--line` | `#DED8C8` | เส้นขอบ |
| `--ink` / `--ink-soft` | `#2D3A3A` / `#5C6A68` | ข้อความ |
| green / green-bg | `#4F7A52` / `#E5EEE2` | in-scope / low / tip |
| red / red-bg | `#A8543F` / `#F2E0D8` | out-of-scope / high |
| yellow-bg | `#F6ECCF` | mid / callout |

---

## 4. กฎ data.json (ห้ามผิด)

- ทุกค่าที่ render ต้องมาจาก `data.json` — ชื่อ field ตรงตาม type `Project` ใน `app/lib/projects.ts`
- field ใหม่ `useCases[]`, `nonFunctional[]` เป็น **optional** → ถ้าไม่มี ให้ทั้งสอง renderer **ข้าม section นั้น** (ไม่ใช่เขียนสด)
- การคำนวณราคา/man-day ใช้ helper กลาง `totals()`, `sumMd()`, `fmtTHB()` จาก `projects.ts` เท่านั้น
- เลขทุกตัวต้องตรวจย้อนได้: ราคา ← man-day ← module (กฎเหล็กของเคาะดีญะฮ์)

---

## 5. Checklist ก่อนถือว่า "ตรง spec"

- [ ] `grep '<section id=' output.html` ได้ id เรียง: overview, scope, actors, usecases, modules, nonfunc, timeline, cost, notes
- [ ] React `/project/<id>/internal` มี section id ชุดเดียวกัน เรียงเหมือนกัน
- [ ] React `/project/<id>` (client) ซ่อน timeline + คอลัมน์/ตารางภายใน + **บล็อก AI** + ไม่มีคำต้องห้าม
- [ ] บล็อก AI (ถ้ามี `ai{}`) อยู่ใน §8 ฝั่ง internal เท่านั้น · AI man-day แต่ละ module = ceil(คน ÷ factor) · มี `factor` กำกับครบ
- [ ] สารบัญ pin ติดบนจอเมื่อเลื่อน + scroll-spy ทำงาน (ทั้ง React และ output.html)
- [ ] สีตรงตาม token ตาราง §3
