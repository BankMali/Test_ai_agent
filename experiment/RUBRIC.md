# Quality Rubric — ใช้ให้คะแนนทั้งสอง arm (blind)

Judge agent อิสระให้คะแนนโดย **ไม่รู้ว่า diff ไหนมาจาก arm ไหน** (label เป็น X / Y เท่านั้น) ใช้ rubric เดียวกันกับทั้งสอง

## เกณฑ์และน้ำหนัก

| # | เกณฑ์ | น้ำหนัก | ให้คะแนน 1–5 จาก |
|---|-------|---------|------------------|
| 1 | **Correctness & รันได้** | 25% | รัน end-to-end ได้ไหม, logic ถูก, edge case (ไม่มี commit, pagination, timezone) จัดการครบ |
| 2 | **Completeness vs DoD** | 20% | ทำครบ Definition of Done ใน SPEC.md กี่ข้อ |
| 3 | **Summary faithfulness** ⭐ | 25% | สรุปตรง commit/PR จริงไหม มี hallucination/ตกหล่นไหม (ให้ `summary-quality-verifier` ตรวจข้อนี้) |
| 4 | **Code quality & security** | 15% | การจัดการ token ปลอดภัย, error handling, ไม่มี bug ร้ายแรง (จาก `code-reviewer`) |
| 5 | **Maintainability** | 15% | โครงสร้างชัด, แยก concern, อ่าน/ต่อยอดง่าย, test มีความหมาย |

**คะแนนรวม** = ผลรวมถ่วงน้ำหนัก (เต็ม 5.00)

## วิธีให้คะแนน

- แต่ละเกณฑ์ให้ 1 (แย่มาก) – 5 (ดีเยี่ยม) พร้อม**เหตุผลสั้น ๆ + หลักฐาน** (ชี้ไฟล์:บรรทัด)
- ข้อ 3 (faithfulness) ต้อง cross-check สรุปที่ output กับ commit/PR จริงในข้อมูลทดสอบ
- ห้ามเดา — ถ้ารันไม่ได้/ไม่มีหลักฐาน ให้คะแนนต่ำและระบุเหตุผล
- ปิดท้ายด้วยตารางสรุปคะแนนของ X และ Y เทียบกัน

## เกณฑ์เสริม (รายงานแยก ไม่รวมในคะแนนคุณภาพ)

- เวลา (wall-clock) ต่อ arm
- Token / จำนวน agent ที่ spawn ต่อ arm
- จำนวนครั้งที่ต้องถามคน (human-in-loop overhead)
