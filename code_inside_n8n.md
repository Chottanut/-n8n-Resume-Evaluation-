# 1) OCR command 
> ใช้กับ **Execute Command** ใน n8n เพื่ออ่านไฟล์จาก `/tmp` แล้วส่งคืน `{"ocr_text": "..."}`

```bash
python -c "import os, json; os.environ['TYPHOON_OCR_API_KEY']='<TYPHOON_OCR_API_KEY>'; from typhoon_ocr import ocr_document; input_path='/tmp/{{ $binary.data.fileName }}'; text=ocr_document(input_path); print(json.dumps({'ocr_text': text}, ensure_ascii=False))"
```
* โค้ดนี้จะอ่าน PDF/ภาพจาก `/tmp/{{ $binary.data.fileName }}` → ส่งให้ Typhoon OCR → ได้ข้อความ → พิมพ์ JSON ออกมา
* อย่าใส่คีย์จริงลงในไฟล์โปรเจ็กต์ ใช้ `<TYPHOON_OCR_API_KEY>` แล้วไปตั้งใน env จะปลอดภัยกว่า


# 2) Prompt วิเคราะห์เรซูเม่ (พร้อมวาง)
> ใช้กับโหนด LLM (เช่น Vertex AI / OpenAI) โดยส่ง `OCR_TEXT` เข้าตามบล็อกด้านล่าง

```
คุณคือผู้ช่วยวิเคราะห์ความตรงของเรซูเม่ต่อ JD สำหรับตำแหน่ง “AI & Data Solution Intern” ของบริษัทเทคฯ (โซลูชัน HR/การศึกษาโดยใช้ AI)

## กติกาสำคัญ
- วิเคราะห์จาก **OCR_TEXT เท่านั้น** ห้ามเดา/ห้ามฮัลลูซิเนต
- ทุกการตัดสิน (present / partial / missing) ต้องมี **evidence** เป็นคำ/วลี/โปรเจกต์/เครื่องมือที่พบจริงใน OCR (คัดวลีสั้น ๆ)
- ถ้าไม่พบให้ใส่ค่าว่าง "" หรือ [] ตามชนิดข้อมูล
- **ต้องส่งออกเป็น JSON ออบเจกต์เดียว** และต้อง **สอดคล้องกับ JSON Schema ของ Structured Output Parser ที่ให้ไว้** (ห้ามมีคีย์เกิน)

## JD (เกณฑ์อ้างอิง)
### Responsibilities (matches.responsibilities: ครบทั้ง 6 ข้อ)
1) เก็บ/เข้าใจความต้องการกับผู้ใช้/ธุรกิจ
2) ออกแบบ/พัฒนา/ปรับแต่ง prompt
3) ออกแบบโครงสร้าง/กระบวนงาน/การใช้งานของ AI application
4) ใช้ LLMs เพื่อพัฒนา AI application
5) ทดสอบ/ประเมินประสิทธิภาพของ AI application
6) ทำงานร่วมวิศวกรซอฟต์แวร์เพื่อนำขึ้นระบบจริง

### Qualifications (matches.qualifications: ครบทุกหัวข้อ)
- Python, prompt engineering, context engineering
- คิดวิเคราะห์/แก้ปัญหา, สนใจ AI/automation/data-driven
- พื้นฐาน NLP และแนวคิด Machine Learning
- API, JSON, หรือ automation pipelines
- พิเศษ: n8n, SQL, Docker, Cloud

## คำอธิบายการตัดสิน
- present = มีหลักฐานตรงตัว
- partial = กล่าวถึงใกล้เคียง/บางส่วน
- missing = ไม่พบหลักฐาน
- ต้องมี **confidence** (0–1, ปัดทศนิยม 2 ตำแหน่ง)

## Must-Have Checklist (ต้องพบ ≥ partial และบันทึกใน RESULT.reason)
1) Python
2) Prompt engineering หรือการใช้ LLM (≥1 อย่าง)
3) อย่างน้อยหนึ่งใน API / JSON / automation pipeline

## การให้คะแนน (รวม 100)
A) ทักษะหลัก 40 → Python 10, Prompt 12, Context/RAG 10, NLP/ML 8
B) ประสบการณ์ทำ AI/LLM App 30 → ใช้ LLM 10, โครงสร้าง/เวิร์กโฟลว์ 8, ทดสอบประเมิน 6, ทำงานร่วมทีม 6
C) เครื่องมือ/เทคโนโลยี 15 → API/JSON/Automation 5, n8n 4, SQL/Docker/Cloud 6
D) สอดคล้อง Responsibilities 15 → requirements 4, prompt 4, design 3, test 2, collaboration 2

**หลักการให้คะแนน**
- missing = 0
- partial ≈ ครึ่งน้ำหนัก
- present ≈ เต็มน้ำหนัก
- รวมและปัดเป็นจำนวนเต็ม 0–100 → ใส่ใน RESULT.point

## วิธีแมปข้อมูลเข้ากับ Schema
- candidate: name, email, links (ไม่พบให้ "" หรือ [])
- education: degree, field, institution, graduation_year, evidence
- experience: role, organization, duration/period, highlights (array), evidence (หลายหลักฐานคั่น `; `)
- skills: technical[], soft[]
- knowledge[], tools[] (เช่น NLP, LangChain, SQL, Docker, GCP ที่ “พบจริง”)
- matches: responsibilities (6 ข้อ) และ qualifications (ครบทุกหัวข้อ) → ใส่ match_level, evidence, confidence
- notes: strengths, gaps, recommendations (ยึดหลักฐานจริง)
- RESULT:
  - name = candidate.name
  - point = คะแนนรวม 0–100
  - reason = เหตุผลย่อ คั่น `;` เช่น "มี Python; มี SQL; ไม่มี Docker; เคยเชื่อม LLM API"

## รูปแบบ Output
- ส่งออกเป็น **ออบเจกต์ JSON เดียว** (ไม่ใส่โค้ดบล็อก ไม่ใส่ข้อความอื่น)
- ค่าและข้อความทั้งหมดต้องเป็น **ภาษาไทย**

## OCR_TEXT
""" {{ $json["stdout"] }} """
```

# 3) JSON Schema ของผลลัพธ์ (พร้อมวาง)
> เอาไปใส่ใน **Structured Output Parser / JSON Schema** ได้เลย

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "additionalProperties": false,
  "properties": {
    "candidate": {
      "type": "object",
      "properties": {
        "name": { "type": "string" },
        "email": { "type": "string" },
        "links": { "type": "array", "items": { "type": "string" } }
      },
      "required": ["name", "email", "links"],
      "additionalProperties": false
    },
    "education": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "degree": { "type": "string" },
          "field": { "type": "string" },
          "institution": { "type": "string" },
          "graduation_year": { "type": "string" },
          "evidence": { "type": "string" }
        },
        "required": ["degree", "field", "institution", "graduation_year", "evidence"],
        "additionalProperties": false
      }
    },
    "experience": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "role": { "type": "string" },
          "organization": { "type": "string" },
          "duration": { "type": "string" },
          "period": { "type": "string" },
          "highlights": { "type": "array", "items": { "type": "string" } },
          "evidence": { "type": "string" }
        },
        "required": ["role", "organization", "highlights", "evidence"],
        "additionalProperties": false
      }
    },
    "skills": {
      "type": "object",
      "properties": {
        "technical": { "type": "array", "items": { "type": "string" } },
        "soft": { "type": "array", "items": { "type": "string" } }
      },
      "required": ["technical", "soft"],
      "additionalProperties": false
    },
    "knowledge": { "type": "array", "items": { "type": "string" } },
    "tools": { "type": "array", "items": { "type": "string" } },
    "matches": {
      "type": "object",
      "properties": {
        "responsibilities": {
          "type": "array",
          "items": {
            "type": "object",
            "properties": {
              "item": { "type": "string" },
              "match_level": { "type": "string", "enum": ["present", "partial", "missing"] },
              "evidence": { "type": "string" },
              "confidence": { "type": "number", "minimum": 0, "maximum": 1 }
            },
            "required": ["item", "match_level", "evidence", "confidence"],
            "additionalProperties": false
          }
        },
        "qualifications": {
          "type": "array",
          "items": {
            "type": "object",
            "properties": {
              "item": { "type": "string" },
              "match_level": { "type": "string", "enum": ["present", "partial", "missing"] },
              "evidence": { "type": "string" },
              "confidence": { "type": "number", "minimum": 0, "maximum": 1 }
            },
            "required": ["item", "match_level", "evidence", "confidence"],
            "additionalProperties": false
          }
        }
      },
      "required": ["responsibilities", "qualifications"],
      "additionalProperties": false
    },
    "notes": {
      "type": "object",
      "properties": {
        "strengths": { "type": "array", "items": { "type": "string" } },
        "gaps": { "type": "array", "items": { "type": "string" } },
        "recommendations": { "type": "array", "items": { "type": "string" } }
      },
      "required": ["strengths", "gaps", "recommendations"],
      "additionalProperties": false
    },
    "RESULT": {
      "type": "object",
      "properties": {
        "name": { "type": "string" },
        "point": { "type": "integer", "minimum": 0, "maximum": 100 },
        "reason": { "type": "string" }
      },
      "required": ["name", "point", "reason"],
      "additionalProperties": false
    }
  },
  "required": [
    "candidate",
    "education",
    "experience",
    "skills",
    "knowledge",
    "tools",
    "matches",
    "notes",
    "RESULT"
  ]
}
```

# 4) คำอธิบาย
## 📌 ภาพรวมการทำงาน
1. **ดึงไฟล์** → โหลด Resume ที่อัปเข้า Google Drive
2. **OCR** → แปลงไฟล์ PDF เป็นข้อความดิบ (OCR\_TEXT)
3. **LLM วิเคราะห์** → เปรียบเทียบข้อความนั้นกับ Job Description (JD)
4. **ผลลัพธ์ JSON** → ได้ข้อมูลที่เป็นโครงสร้าง พร้อมคะแนน / หลักฐาน / เหตุผลสั้น ๆ
5. **ใช้งานต่อ** → เช่น append เข้า Google Sheets

## 📑 หลักการวิเคราะห์
* **หลักฐาน**: ทุกอย่างต้องมาจาก OCR ที่เจอจริง ๆ เท่านั้น
  * ถ้าไม่เจอเลย ให้ใส่ `""` หรือ `[]` แล้วเขียนว่า *missing*
* **การให้คะแนน**
  * เจอครบถ้วน = ให้คะแนนเต็มตามข้อกำหนด
  * เจอแค่บางส่วน = ให้คะแนนครึ่งหนึ่งของข้อกำหนด
  * ไม่เจอเลย = 0
  * รวมแล้วปัดเป็นจำนวนเต็ม 0–100
  * 
## 📝 เหตุผลสั้น

* เขียนเป็น **ข้อความสั้น ๆ ต่อกันด้วย `;`**
* เช่น:
  ```
  มี Python; ใช้ LLM API; ไม่มี Docker; เขียน SQL
  ```
---
## ⚠️ ข้อควรระวัง
1. **อย่าเพิ่ม key ที่ไม่มีใน schema**
2. ค่า **confidence** ต้องอยู่ระหว่าง `0–1` และปัด 2 ตำแหน่ง (เช่น `0.82`)
3. ข้อความทั้งหมดต้องเป็น **ภาษาไทย**
4. ถ้าไม่เจอข้อมูลให้ใส่ *missing* ชัดเจน
---

