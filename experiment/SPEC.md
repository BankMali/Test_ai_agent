# Frozen Spec — Benchmark Task (ห้ามแก้ระหว่างการทดลอง)

Spec นี้ถูก freeze เป็น base commit ทั้งสอง arm (agent-team / workflow) ต้องทำ **task เดียวกันเป๊ะ** จาก base commit เดียวกัน ห้ามแก้ไฟล์นี้หลังเริ่มรัน

## เป้าหมาย (MVP slice)

สร้าง CLI tool ที่: รับ **GitHub repo + วันที่ (+ author ถ้าระบุ)** → ดึง commit และ PR ของวันนั้น → เก็บข้อมูลดิบลง local storage → generate สรุปรูปแบบ standup ด้วย Claude → พิมพ์/บันทึกสรุป

**ไม่รวมในรอบนี้**: web frontend, การ sync หลาย repo, การแก้สรุปผ่าน UI (เก็บไว้ทีหลัง)

## Stack ที่ fix (ทั้งสอง arm ต้องใช้เหมือนกัน)

- ภาษา: **Python 3**
- GitHub: REST API ผ่าน token ใน env var `GITHUB_TOKEN`
- Storage: **SQLite** (ไฟล์เดียว)
- LLM: **Claude** (`claude-*` ผ่าน Anthropic SDK) — key ใน env `ANTHROPIC_API_KEY`
- Test: `pytest`

> ถ้าจะเปลี่ยน stack เปลี่ยนได้ แต่ต้องเปลี่ยน **ทั้งสอง arm ให้เหมือนกัน** ก่อนเริ่มรัน

## Definition of Done (เกณฑ์ว่างานเสร็จ — ใช้เดียวกันทั้งสอง arm)

1. **รันได้ end-to-end**: `python -m dailysummary --repo owner/name --date 2026-07-01 [--author user]` แล้วได้สรุปออกมา
2. **Ingestion**
   - ดึง commit + PR ของวันที่ระบุครบ (จัดการ pagination)
   - อ่าน token จาก env — **ห้าม hardcode/log token**
   - จัดการ rate limit เบื้องต้น (เช่น เคารพ header / retry)
   - repo ว่าง หรือไม่มี commit ในวันนั้น → ไม่ crash, รายงานว่า "ไม่มีงาน"
3. **Storage**
   - เก็บ **ข้อมูลดิบ (commit/PR) แยก table จากสรุปที่ generate**
   - query ย้อนหลังได้อย่างน้อย: ตาม date + ตาม author
4. **Summarize**
   - output เป็น standup format: **เมื่อวานทำอะไร / วันนี้จะทำอะไร / blocker**
   - สรุปต้อง **ตรงกับข้อมูลจริง** ห้ามแต่งงานที่ไม่มีใน commit/PR (faithfulness)
   - แยกสถานะ merged vs PR ที่ยัง open
5. **Test**
   - มี test อย่างน้อย: ingestion (mock GitHub API) + summary pipeline (mock LLM)
   - test รันผ่าน ด้วย `pytest`
   - **ห้ามยิง API จริงใน test**
6. **เอกสาร**: README บอกวิธี set env + วิธีรัน + วิธีรัน test

## ข้อจำกัดการทดลอง (ให้ยุติธรรม)

- ทั้งสอง arm ใช้ subagent ชุดเดียวกันใน `.claude/agents/`
- Arm A (agent-team): orchestrate โดย main agent แบบ autonomous ที่สุดเท่าที่ทำได้ (ไม่ถามคนจุกจิก)
- Arm B (workflow): orchestrate ด้วย workflow script แบบ deterministic
- จับ **เวลา (wall-clock)** และ **token/จำนวน agent** ต่อ arm
- 1 รอบต่อ arm (single-run เชิงสังเกต ไม่ใช่ข้อสรุปสถิติ)
