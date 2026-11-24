# Pre-Interview Question — System Analyst (Persec 2026)

| | |
|---|---|
| **ผู้สมัคร** | Sittikron Poti |
| **ตำแหน่ง** | System Analyst |
| **วันที่ส่ง** | 24 พฤศจิกายน 2026 |

---

## โครงสร้างไฟล์ (แยกตามข้อ)

```
.
├── README.md                              ← หน้านี้: สารบัญ + สรุปคำตอบ
├── 01-flash-sale/
│   ├── 1.1-high-level-architecture.md     ← สถานการณ์ 1 คำถามที่ 1
│   └── 1.2-concurrency-logic.md           ← สถานการณ์ 1 คำถามที่ 2
├── 02-junior-mentoring/
│   └── answer.md                          ← สถานการณ์ 2
├── 03-coupon-100-no-overbooking/
│   └── answer.md                          ← สถานการณ์ 3
├── 04-hardcoded-secrets/
│   └── answer.md                          ← สถานการณ์ 4
└── prompts.md                             ← Prompt ที่ใช้กับ AI
```

| ข้อ | เรื่อง | ไฟล์ |
|---|---|---|
| 1.1 | High-Level Architecture ของ Flash Sale 1,000 สิทธิ์ / 50,000 คน | [01-flash-sale/1.1-high-level-architecture.md](01-flash-sale/1.1-high-level-architecture.md) |
| 1.2 | Logic แก้ Race Condition — ไม่แจกซ้ำ ไม่แจกเกิน | [01-flash-sale/1.2-concurrency-logic.md](01-flash-sale/1.2-concurrency-logic.md) |
| 2 | Mentor น้อง Junior ที่ API รายงานยอดขายค้าง / Error 500 | [02-junior-mentoring/answer.md](02-junior-mentoring/answer.md) |
| 3 | แจกคูปอง 100 สิทธิ์ ให้ DB ไม่ล่มและไม่ Overbooking | [03-coupon-100-no-overbooking/answer.md](03-coupon-100-no-overbooking/answer.md) |
| 4 | API Key / URL ถูก Hardcode ใน Source Code | [04-hardcoded-secrets/answer.md](04-hardcoded-secrets/answer.md) |
| — | Prompt ที่ใช้กับ AI | [prompts.md](prompts.md) |

ไดอะแกรมทั้งหมดเขียนด้วย **Mermaid** — GitHub แสดงเป็นภาพให้อัตโนมัติ

---

## สรุปคำตอบแบบสั้น

### ข้อ 1.1 — Architecture Flash Sale
คน 98% จะไม่ได้โค้ด จึงออกแบบให้ **ปฏิเสธเร็วและถูกที่สุดก่อนถึง Database**:
CDN + WAF → API Gateway (JWT, rate limit) → Claim Service แบบ stateless ที่ pre-scale ไว้ → **Redis ตัดสินสิทธิ์** → Stream → Worker → PostgreSQL (เขียนแค่ ~1,000 แถว) พร้อม monitoring, load test และ kill switch
ไล่หา **Single Point of Failure** ทุก component (จุดเดียวคือ Redis primary ของแคมเปญ → fail closed), หาขนาดระบบจากสูตร + load test และกำหนด **NFR 10 ข้อที่วัดได้** เป็นเกณฑ์ Go / No-go

### ข้อ 1.2 — Concurrency / Race Condition
ป้องกันหลายชั้น โดยความถูกต้องอยู่ที่ 2 ชั้นล่าง:
- **Redis Lua script** ทำ "เช็กว่าเคยได้หรือยัง → `LPOP` โค้ดจาก LIST ที่มี 1,000 ตัว → บันทึกผู้ได้ → ส่ง event" ในขั้นเดียวแบบ atomic
- **Database constraint** `UNIQUE (campaign_id, user_id)`, `UNIQUE (code)` + FK ไปยังตารางโค้ด 1,000 ตัว เป็นด่านสุดท้าย
- เตรียม **กรณีเลวร้ายที่สุด** ที่ Redis ล่มก่อน replicate ด้วย `WAIT`, audit log ของทุกคำตอบ `WON` และ reserve code ที่ตกลงกับฝ่ายธุรกิจไว้ก่อน

### ข้อ 2 — Mentor น้อง Junior
ลดความเครียดและรับเรื่องเวลาไว้เองก่อน → **น้องพิมพ์ พี่ถาม** → Reproduce → อ่าน log / stack trace → แบ่งชั้นจับเวลา → ตั้งสมมติฐานพิสูจน์ทีละข้อ (ใช้ Hint Ladder เมื่อติด) → น้องแก้เองและวัดผล → มุม SA ทบทวน requirement ของรายงาน → สรุปบทเรียนเป็น Debugging Checklist พร้อม **ตัวอย่างบทสนทนาจริง** ที่น้องเจอสาเหตุเอง

### ข้อ 3 — คูปอง 100 สิทธิ์
**Funnel** กรองคำขอทีละชั้นจน Database อยู่นอก hot path (preload ทุกอย่าง, ไม่มี read/write DB ระหว่างแคมเปญ, worker pool เล็ก, fail closed)
กัน overbooking ด้วย **token 100 ใบใน Redis + Lua atomic + `WAIT` replica** และ DB ที่มี **100 แถว ใบละหนึ่งเจ้าของ** มี NFR ที่วัด "DB ไม่ล่ม" และ "ไม่ overbooking" เป็นตัวเลข พร้อมเสนอทางเลือกเชิงธุรกิจแบบ **ลงทะเบียน + จับฉลาก**

### ข้อ 4 — Hardcoded API Key / URL
คุยแบบไม่ตำหนิ ทำให้เห็นผลกระทบจริง (Git history, บิล, PDPA, สลับ environment ไม่ได้) พร้อมตัวอย่างบทสนทนาที่ตอบข้อโต้แย้ง "repo เป็น private" → **ถือว่ารั่วแล้ว rotate ทันที** → secret ไป Secret Manager, URL ไป config ตาม environment → กันซ้ำด้วย gitleaks pre-commit + GitHub Push Protection + CI

---

## สมมติฐานที่ใช้

- ใช้ AWS เป็นตัวอย่างของ cloud — แต่ละ component มีบริการเทียบเท่าใน Azure / GCP / on-premise (ระบุไว้ในข้อ 1.1)
- ผู้ใช้ต้อง login ก่อนกดรับสิทธิ์ และ 1 บัญชีได้ 1 สิทธิ์ต่อแคมเปญ
- โค้ดส่วนลด / คูปองสร้างไว้ล่วงหน้าก่อนเปิดแคมเปญ
- ตัวเลข capacity เป็นค่าตั้งต้นจากการประมาณ ต้องยืนยันด้วย load test ก่อนใช้งานจริง
- เป้า NFR เป็นข้อเสนอให้ทีมและฝ่ายธุรกิจตกลงร่วมกันก่อนเริ่มพัฒนา

## การใช้ AI

ใช้ AI ช่วยร่างคำตอบ แล้วตรวจทานอีก 4 รอบแบบ SA (ความถูกต้องทางเทคนิค, NFR, การสื่อสาร, ความสอดคล้องทั้ง repo) — prompt ทุกขั้นอยู่ใน [prompts.md](prompts.md) และสิ่งที่เปลี่ยนในแต่ละรอบดูได้จาก commit history
