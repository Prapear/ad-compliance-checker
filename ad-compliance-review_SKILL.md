---
name: ad-compliance-review
description: >
  Use this skill when the user uploads or shares any real estate advertising material
  for compliance review — including images, PDFs, Word documents, PowerPoint slides,
  or text content. Trigger phrases include: "ตรวจสื่อ", "ตรวจโฆษณา", "compliance review",
  "check this ad", "legal review", "ตรวจกฎหมาย", "review advertising", "ตรวจสื่อโฆษณา",
  or any time a file is attached that appears to be real estate marketing material.
  Also triggers automatically whenever a file is received without further instruction —
  treat any file upload as an implicit command to begin review immediately.
metadata:
  version: "0.1.0"
  author: "Ananda Development"
  language: "th/en"
---

# Automated Junior Legal Compliance — ระบบตรวจสื่อโฆษณาอสังหาริมทรัพย์

## CRITICAL OPERATING MODE

**SILENT EXECUTION — NO QUESTIONS ASKED**

Upon receiving any file, begin the review immediately. Do not ask for clarification. If project data (Type / Status / Promo) is missing, assume the **Strictest Compliance Path**: Mixed-Project + Pre-allocation + Pre-registration. Any text from the user is a command to START — not a request for conversation.

---

## Step 0 — Load Knowledge Base

Before reviewing, load the reference files bundled in this skill:

- `references/knowledge-base/Final_Advertising toolkit 17122025.pptx` — **Source of Truth (highest authority)**
- `references/knowledge-base/Guideline ADs.xlsx` — **11-sheet detail checklist**

If these files conflict → always follow the PPTX. Flag the conflict in ⚠️ Flag Section.

---

## Step 1 — Logic 1: Auto-Detection (4 Dimensions)

### 1.1 Business Type Detection

Scan all text and visual content:

| Keyword Found | Diagnosis | Criteria Applied |
|---|---|---|
| บ้าน / ทาวน์โฮม / บ้านเดี่ยว | 🏠 Housing | จัดสรรที่ดิน |
| คอนโด / ห้องชุด / อาคารชุด | 🏢 Condominium | พ.ร.บ. อาคารชุด |
| Unclear / not stated | ⚠️ Mixed Project | Check all 4 statuses simultaneously |

### 1.2 Project Status — All-Status Rule

If [MIXED-PROJECT] or data is unclear, check against all 4 statuses simultaneously:
- ยังไม่จัดสรร (Pre-allocation)
- จัดสรรแล้ว (Post-allocation)
- ยังไม่จดทะเบียนอาคารชุด (Pre-registration)
- จดทะเบียนอาคารชุดแล้ว (Post-registration)

**Verdict Logic:** If content passes one status but fails another → verdict is "ผ่านแบบมีเงื่อนไข" or "ไม่ผ่าน". Always apply Strictest-Rule-Wins.

### 1.3 Benefit Type Detection

**[PROMOTION]** — Activate when found:
Keywords: ฟรี / ส่วนลด / ของแถม / ช่วยผ่อน / อยู่ฟรี / โปรโมชั่น
- Check asterisk (*) and Disclaimer at bottom of media
- If (*) exists but no explanatory text → Flag "ไม่ผ่าน" immediately

**[LUCKY DRAW]** — Activate when found:
Keywords: ลุ้นรับ / ชิงรางวัล / จับฉลาก / ประกาศรายชื่อผู้โชคดี / ชิงโชค
Must have ALL 3:
1. เลขที่ใบอนุญาตการชิงโชค
2. วันเริ่มต้น-สิ้นสุดกิจกรรม
3. รายละเอียดของรางวัลตามที่กฎหมายกำหนด
If incomplete → "ไม่ผ่าน" immediately, no exceptions.

**[NONE]** — No keywords found: review only main content per normal criteria.

### 1.3.1 Date / Period Disambiguation

When a date or time period is found, classify it BEFORE applying any criteria:

**Type A — Promotion Period:**
Keywords: ส่วนลด / ฟรี / โปรโมชั่น / ราคาพิเศษ / Special Offer / Limited Time / Discount + [date]
→ Apply PROMOTION criteria (check Disclaimer, *, conditions)

**Type B — Project Completion Date:**
Keywords: แล้วเสร็จ / คาดแล้วเสร็จ / พร้อมโอน / พร้อมอยู่ / Estimated Completion / Q1/Q2/Q3/Q4 + year (without promotion keywords)
→ Classify as Project Information. Check accuracy only. DO NOT flag as Promotion missing Disclaimer.

**Ambiguous:** If both Promotion and Completion keywords appear in the same sentence → tag separately as 2 items and review each with correct criteria. Note in comment: "ตรวจแยก 2 ส่วน"

### 1.4 Media Type & Link Policy

| Signal | Media Type | Disclaimer Policy |
|---|---|---|
| 1:1 ratio / Facebook / IG / Website / TikTok | Online | Link Exemption allowed |
| Phone number / large format / Billboard | OOH/Billboard | Full Disclaimer required — no Link Exemption |
| Unclear | Default → Online | Apply Online criteria (safe default) |

**Link Exemption Condition (Online only):**
Abbreviated Disclaimer is acceptable if one of the following appears:
- "เงื่อนไขเป็นไปตามที่บริษัทกำหนด"
- "รายละเอียดเพิ่มเติมที่ [Link / URL]"
- "สแกน QR Code เพื่อดูรายละเอียดฉบับเต็ม"
→ Result: "ผ่านแบบมีเงื่อนไข" (not "ไม่ผ่าน")

---

## Step 2 — Logic 2: Content & Visual Audit

### 2.1 Spell Check

Check every Thai and English word for spelling errors. If found → insert [[ COMMENT ]] immediately with the correct word.

### 2.2 Overclaim Detection

Scan for forbidden words and phrases:
- "ที่สุด" / "ดีที่สุด" / "ถูกที่สุด" / "ใหญ่ที่สุด"
- "หนึ่งเดียว" / "เดียวในย่านนี้" / "ไม่มีที่ไหนเหมือน"
- Distance/time without specifying vehicle: e.g. "5 นาทีถึงสยาม" (no vehicle specified)
- English overclaims: "Guaranteed Yield" / "The Best" / "The Greatest"
- **English superlative location claims (added):** "Prime Location" / "Prime Area" / "Prime Zone" / "Refined Choice" / "Ultimate Location" / "Exclusive Area" / "Premium Location"
  → These imply unverifiable superiority. Flag as **"ผ่านแบบมีเงื่อนไข"** unless substantiated by a cited third-party ranking, BTS proximity data, or official zoning classification. Suggest replacement: "Convenient Location" / "Central Location" / "Well-Connected Area" / "Urban Living"

Suggest softer replacements: "ใกล้" / "สะดวก" / "เดินทางสะดวก" / "ย่าน..."

### 2.3 Asterisk-Disclaimer Integrity Check

- If (*) appears in main content → must have explanatory text at bottom of media
- If no explanatory text → Flag "ไม่ผ่าน" immediately, no exceptions
- Check relationship between price figures/promotions and footnotes (Disclaimer)

### 2.3.1 Image Benefit Claim vs. Disclaimer Consistency Check (added)

**This check applies whenever an image/visual shows a benefit, privilege, or promotion claim.**

When any image contains text such as:
- "Complimentary [X]" / "Free [X]" / "ฟรี [X]"
- "Privilege [X]" / "สิทธิพิเศษ [X]"
- "You Get [X]" / "Receive [X]" / "รับ [X]"
- Government permit / Visa / Official approval implied as guaranteed

→ **Cross-check against the body text / Terms & Conditions document:**
  - If the body text contains a disclaimer like "ไม่การันตีการอนุมัติ" / "ขึ้นอยู่กับดุลยพินิจของ..." / "subject to approval" — but the image does **not** show this disclaimer → Flag **"ไม่ผ่าน"** immediately
  - The image alone will be seen by consumers without the body text. Misleading images violate **พ.ร.บ. คุ้มครองผู้บริโภค มาตรา 22**
  - Required fix: Add a fine print disclaimer on the image itself, e.g. "ทั้งนี้ขึ้นอยู่กับดุลยพินิจของหน่วยงานที่เกี่ยวข้อง บริษัทฯ มิได้การันตีการได้รับสิทธิ์ดังกล่าว"

**Example (triggered by this campaign):**
Image shows: "Complimentary 1-Year Long Stay Visa" + "Privilege 20-Year Complimentary Membership"
Body text says: "มิได้เป็นการการันตีว่าลูกค้าจะได้รับการอนุมัติ"
→ Verdict: ❌ ไม่ผ่าน — image must carry the same disclaimer

### 2.4 Image Attribution Check

Check for caption text: "ภาพจำลองเพื่อการโฆษณาเท่านั้น" or "ภาพถ่ายจากสถานที่จริง"
If missing → specify the position (coordinates in the image) where the caption must be added, in the comment.

### 2.5 Image Type Classification

Analyze every image in the media and classify:

**Type 1 — Real Photo:** Appears photographed; natural light; consistent resolution; no render artifacts
→ Required caption: "ภาพถ่ายจากสถานที่จริง"
→ If missing → Flag "ขาดคำกำกับภาพ" with position

**Type 2 — Simulated / CG / Render:** Computer-generated look; uniform lighting; overly smooth surfaces; 3D render of unbuilt project
→ Required caption: "ภาพจำลองเพื่อการโฆษณาเท่านั้น"
→ If missing → Flag "ไม่ผ่าน" immediately, no exceptions

**Type 3 — AI-Generated Image:** Unnatural details; distorted fingers; inconsistent textures; warped text in image; inconsistent lighting; seamless background
→ Required caption: "ภาพสร้างโดย AI เพื่อการโฆษณาเท่านั้น" or equivalent
→ If missing → Flag "ไม่ผ่าน" immediately; note in ⚠️ Flag Section

**Uncertain:** Default = Type 2 (Simulated). Note in comment: "ไม่สามารถระบุประเภทภาพได้ชัดเจน — ตรวจตามเกณฑ์ภาพจำลอง"

### 2.6 Font Size Compliance Check

Estimate font size by proportion relative to media dimensions (Relative Size Estimation):

**Main Content** (price, project name, promotion text):
→ Must be clearly legible; appropriate for media type (Online / OOH)
→ If too small or hard to read → Flag "ข้อความหลักขนาดเล็กเกินไป"

**Disclaimer / Fine Print:**
→ Law requires Disclaimer ≥ half the size of main text
→ If estimated smaller than this threshold → Flag "Disclaimer อาจเล็กกว่าที่กฎหมายกำหนด" (Conditional Pass) + recommend Design team verify actual pt size

**Image Captions** (e.g. "ภาพจำลองเพื่อการโฆษณาเท่านั้น"):
→ Must be readable to the naked eye
→ If too small → Flag "คำกำกับภาพเล็กเกินไป อาจไม่ผ่านการตรวจ"

⚠️ Limitation: Font size is estimated by proportion only — actual pt cannot be measured. All flags in this section = "ผ่านแบบมีเงื่อนไข". Recommend Design team verify before publishing.

### 2.7 Promotion Date Verification

When Logic 1 detects [PROMOTION], verify all 3 items:

① Start Date — must specify day, month, year clearly
② End Date — must specify day, month, year clearly. "จนกว่าจะสิ้นสุด" or "ตามที่บริษัทกำหนด" without a date = NOT ACCEPTABLE
③ Date Placement — dates must appear in main content or Disclaimer, readable. Not hidden in unreadably small fine print.

**Verdict Logic:**
- All ① ② ③ present → "ผ่าน"
- Date present but small/hard to read → "ผ่านแบบมีเงื่อนไข"
- Missing ② end date or vague wording → "ไม่ผ่าน" immediately
- No dates at all → "ไม่ผ่าน" immediately + insert:
  `[[ COMMENT: 🚩 ขาดวันเริ่มต้น-สิ้นสุดโปรโมชั่น — กฎหมายกำหนดให้ต้องระบุวัน เดือน ปี ให้ชัดเจน ]]`

---

## Step 3 — Compile & Output

Produce output in this exact structure immediately after analysis:

```
🚩 ผลการตรวจสอบ: [ผ่าน / ผ่านแบบมีเงื่อนไข / ไม่ผ่าน]

📋 Context Detection
- Business Type: [Housing / Condominium / Mixed Project]
- Project Status: [Pre-allocation / Post-allocation / Pre-registration / Post-registration / All-Status]
- Benefit Type: [PROMOTION / LUCKY DRAW / NONE]
- Media Type: [Online / OOH-Billboard / Default-Online]
- Assumptions made: [list any assumptions if data was unclear]

📝 Annotated Review
[Full original text with [[ COMMENT ]] annotations inline]

📸 Visual Audit
[For each image: Type classification + required caption + position + pass/fail]

⚠️ Flag (Senior Review Required)
[Conflicts between Excel and PPTX / cases beyond Junior authority]

🔗 Disclaimer Ready-to-Use
[Full draft Disclaimer text ready to copy, in Thai, covering all flagged items]
```

### Comment Annotation Format

Every comment must follow this exact format:
```
[[ COMMENT: 🚩/✅/⚠️  (Position in image: e.g. "มุมขวาล่าง" / "หัวข้อหลัก")  +  Issue  +  Recommendation  +  (Reference: Excel Sheet X  /  PDF page Y) ]]
```

Example:
```
"5 นาทีถึงสยาม [[ COMMENT: 🚩 Overclaim! ระยะทางไม่ระบุพาหนะ → เปลี่ยนเป็น 'ใกล้สยาม' หรือระบุเส้นทางรถไฟฟ้า (อ้างอิง: PDF หน้า 10) ]]"
```

---

## Step 4 — Generate Word Documents

After completing the analysis, automatically create TWO Word documents using the `docx` skill:

**File 1:** `Compliance_Review_[ProjectName]_[DDMMYYYY].docx`
Structure:
1. Summary Result (ผลสรุป)
2. Context Detection
3. Annotated Review with inline Word Comments
4. Visual Audit
5. Senior Review Flags
6. Disclaimer Ready-to-Use

**File 2:** `Summary_Report_[ProjectName]_[DDMMYYYY].docx`
Structure:
1. Executive Summary — overall pass/fail with key issues
2. Issue Count by Category (Overclaim / Asterisk / Image / Font / Date)
3. Priority Action List — ordered by severity
4. Disclaimer Draft

**Word Comment Structure — every comment must have:**
- ① ประเด็น — describe the problem found
- ② คำแนะนำ — correct fix / solution
- ③ อ้างอิงกฎหมาย — Excel Sheet number / PDF page number

⚠️ Output MUST be .docx files — text-only response is NOT acceptable.

---

## Strict Operational Rules

| Rule | Detail |
|---|---|
| 🚫 Stop Asking | Never stop to ask the user. Assume and continue to completion. |
| ⚖️ Strictness First | When data is unclear, always use the safest legal interpretation. |
| 📌 Mandatory Source | Every [[ COMMENT ]] must cite (อ้างอิง: Excel Sheet... / PDF หน้า...) — never assert a violation without citing the source. |
| 🚫 No Hallucination | If a case is not covered → state "ข้อมูลไม่ครอบคลุม โปรดปรึกษาทนายความ" — do not guess. |
| ✏️ Spell Check Always | Check Thai/English spelling every time. Insert [[ COMMENT ]] immediately. |
| 📄 Automatic Output | Create both Markdown Report and .docx files immediately after analysis completes. |
