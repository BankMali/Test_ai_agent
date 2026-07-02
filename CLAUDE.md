# Daily Dev Summary

เครื่องมือดึง commit/PR จาก **GitHub** มาสรุปงานรายวันของ dev เก็บย้อนหลังได้ และเตรียมสรุปให้พร้อมสำหรับ daily standup เพื่อกระชับการประชุม

- **Provider**: GitHub (REST/GraphQL API + personal access token / GitHub App)
- **หน่วยข้อมูล**: commit และ pull request (PR)
- **Positioning**: ผู้ช่วยจำ + กระชับประชุม — **ไม่ใช่** เครื่องมือจับผิด productivity (ห้ามทำ leaderboard/นับ commit เป็น KPI)

## Agent team

โปรเจกต์นี้ทำงานแบบ agent team — main agent ทำหน้าที่ orchestrator และ delegate ให้ subagent ที่เหมาะ (นิยามใน `.claude/agents/`):

| Agent | หน้าที่ | เขียนโค้ด? |
|-------|---------|-----------|
| `planner-architect` | วางแผน/สถาปัตยกรรม/แตก task | ไม่ (วางแผนอย่างเดียว) |
| `integration-agent` | ต่อ GitHub API: commit, PR, diff, auth, pagination, rate limit | ใช่ |
| `data-storage-agent` | schema + เก็บย้อนหลัง (by dev/date/repo/team) | ใช่ |
| `llm-prompt-agent` | prompt สรุป (ใช้ Claude เป็น backend) | ใช่ |
| `summary-quality-verifier` | ตรวจสรุปตรงกับ commit/PR จริง (กัน hallucination) | ไม่ (ตรวจอย่างเดียว) |
| `frontend-presenter` | หน้าดูย้อนหลัง + standup view + แก้ด้วยมือ | ใช่ |
| `code-reviewer` | ตรวจ bug/security/ความซับซ้อน | ไม่ (ตรวจอย่างเดียว) |
| `tester` | test end-to-end | ใช่ |
| `pr-author` | push + เปิด/เขียน PR บน GitHub (what/why/how + test plan) | ใช่ (git/gh) |

## Flow มาตรฐานของทีม

```
planner-architect → (integration | data-storage | llm-prompt | frontend) → tester → code-reviewer → done
                                          ↑                                               |
                                          └──────── verifier/review loop ─────────────────┘
```

กติกาสำหรับ orchestrator:
1. **งานใหม่ที่ยังไม่ชัด** → เริ่มด้วย `planner-architect` เสมอ ก่อนลงมือเขียน
2. **แยกคนเขียนออกจากคนตรวจ** → โค้ดที่เขียนโดย implementer ต้องผ่าน `code-reviewer` (คนละ agent) ก่อนถือว่าเสร็จ
3. **ทุกครั้งที่แตะ logic การสรุป** → ต้องให้ `summary-quality-verifier` ตรวจว่าสรุปยังตรง commit จริง
4. **งาน implement ที่เป็นอิสระต่อกัน** → spawn หลาย agent พร้อมกันได้ (parallel)
5. **ก่อน commit งานที่มี runtime** → ให้ `tester` ยืนยัน end-to-end ก่อน
6. **หลัง push branch** → ให้ `pr-author` เปิด/อัปเดต PR บน GitHub (ผ่าน `code-reviewer` + `tester` ก่อนถือว่าพร้อม review)

## หลักการทางเทคนิคที่ทั้งทีมยึด

- **ความน่าเชื่อถือของสรุป > จำนวนฟีเจอร์** — สรุปเพี้ยนแม้ครั้งเดียวความเชื่อถือพัง
- **เก็บข้อมูลดิบแยกจากสรุปที่ generate** — re-generate ได้เมื่อ prompt ดีขึ้นโดยไม่ต้องดึง API ซ้ำ
- **ความปลอดภัย token** — ห้าม hardcode/log GitHub token ใช้ env/secret เสมอ
- **dev แก้สรุปด้วยมือได้** — AI ไม่ใช่ source of truth เดี่ยว
- backend สรุปใช้ **Claude** — เรื่อง model/param/caching อ่าน skill `claude-api` ก่อน อย่าเดา
