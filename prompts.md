# Prompts ที่ใช้กับ AI

แบบทดสอบกำหนดว่าถ้าใช้ AI ช่วยเขียน ต้องแนบไฟล์นี้และระบุ prompt ที่ใช้ในแต่ละข้อ / ขั้นตอน

## เครื่องมือ

| | |
|---|---|
| AI | Claude (model Claude Opus 5.5) โดย Anthropic |
| ผ่านโปรแกรม | Claude Code (desktop app) |
| วันที่ใช้ | 24 พฤศจิกายน 2026 |

---

## ขั้นที่ 1 — ร่างคำตอบทุกข้อ

**Input:** แนบไฟล์โจทย์ `Pre-Interview Question System Analyst .pdf`

**Prompt (คำต่อคำ):**

```
นายๆช่วยเราทำแบบทดสอบนี้หน่อย
```

**ผลลัพธ์:** AI อ่านโจทย์ทั้ง 4 สถานการณ์จากไฟล์ PDF แล้วร่างคำตอบทุกไฟล์ใน repo นี้จาก prompt เดียว (ไม่ได้แยก prompt ต่อข้อ) และทดสอบว่า Mermaid diagram ทั้ง 9 รูปแสดงผลได้จริงก่อนอัปโหลด

| ข้อ | ไฟล์ | Prompt ที่ใช้ |
|---|---|---|
| 1.1 | `01-flash-sale/1.1-high-level-architecture.md` | ขั้นที่ 1 |
| 1.2 | `01-flash-sale/1.2-concurrency-logic.md` | ขั้นที่ 1 |
| 2 | `02-junior-mentoring/answer.md` | ขั้นที่ 1 |
| 3 | `03-coupon-100-no-overbooking/answer.md` | ขั้นที่ 1 |
| 4 | `04-hardcoded-secrets/answer.md` | ขั้นที่ 1 |
| — | `README.md`, `prompts.md` | ขั้นที่ 1 |

---

## ขั้นตอนอื่นที่ไม่ได้แก้เนื้อหาคำตอบ

หลังขั้นที่ 1 มีการใช้ AI ช่วยงานที่ไม่เกี่ยวกับเนื้อหาคำตอบข้อ 1–4 ได้แก่ เปิดหน้าพรีวิวบนเครื่อง ใส่ชื่อผู้สมัครและวันที่ส่งใน `README.md` สร้าง repository และ push ขึ้น GitHub
