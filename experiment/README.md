# การทดลอง: Agent Team vs Workflow

เปรียบเทียบ **วิธี orchestrate** สอง แบบบน task เดียวกัน (SPEC.md) จาก base commit เดียวกัน
เพื่อดูความต่างด้าน **คุณภาพ** และ **เวลา/ต้นทุน**

- `SPEC.md` — โจทย์ที่ freeze แล้ว (ห้ามแก้ระหว่างทดลอง)
- `RUBRIC.md` — เกณฑ์ให้คะแนนคุณภาพ (judge อิสระ, blind)
- `RESULTS.md` — ตารางเก็บผล เติมหลังรันแต่ละ arm

## ขั้นตอน

1. **Base** — spec/rubric ถูก commit บน `main` เป็นจุดตั้งต้นร่วม
2. **Arm A — Agent Team** (branch `exp/agent-team`)
   - main agent orchestrate แบบ dynamic/autonomous delegate ให้ subagent ตาม flow ใน CLAUDE.md
   - จับเวลาเริ่ม-จบ, นับ token/agent, นับครั้งที่ถามคน
   - commit ผลงาน
3. **Arm B — Workflow** (branch `exp/workflow`)
   - รัน workflow script แบบ deterministic (plan → implement → test → review → verify)
   - จับ metric เดียวกัน
   - commit ผลงาน
4. **Judge** — spawn judge agent อิสระ ให้คะแนนทั้งสอง diff แบบ blind (label X/Y) ด้วย RUBRIC.md
5. **สรุป** — เติม RESULTS.md เทียบเวลา/ต้นทุน/คุณภาพ

## กติกาความยุติธรรม

- subagent ชุดเดียวกัน, stack เดียวกัน, DoD เดียวกัน, base commit เดียวกัน
- ตัวแปรที่ต่างมีตัวเดียว: **วิธี orchestrate**
- วัด agent wall-clock + token เป็นหลัก (human thinking time ไม่นับรวม รายงาน human overhead แยก)
- single-run: ผลเป็นแนวโน้มเชิงสังเกต ไม่ใช่ข้อสรุปสถิติ
