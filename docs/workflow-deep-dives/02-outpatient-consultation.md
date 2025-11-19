# Outpatient Consultation Workflow
# Quy trình khám ngoại trú

## Overview (Tổng quan)

Outpatient consultation là **core workflow** của phòng khám - nơi bác sĩ gặp bệnh nhân, chẩn đoán và điều trị. Đây là workflow quan trọng nhất cần optimize vì:
- Diễn ra nhiều lần/ngày (20-50+ patients/day per doctor)
- Ảnh hưởng trực tiếp đến chất lượng khám chữa bệnh
- Quyết định revenue của phòng khám

**Thời gian trung bình:** 15-30 phút/bệnh nhân

---

## 1. Consultation Workflow Steps

### Complete Flow (Luồng đầy đủ)

```
BƯỚC 1: Triage & Vital Signs
        (Phân loại & Đo dấu hiệu sinh tồn)
        - Y tá đo: BP, HR, Temp, Weight, Height
        - Ghi lý do khám (Chief Complaint)
        - Sơ bộ đánh giá mức độ khẩn cấp
              ↓
BƯỚC 2: Doctor Review Patient Info
        (Bác sĩ xem thông tin bệnh nhân)
        - Xem vital signs vừa đo
        - Đọc lịch sử khám trước (nếu có)
        - Review tiền sử bệnh, dị ứng thuốc
              ↓
BƯỚC 3: History Taking (Hỏi bệnh sử)
        - Chief complaint detail (triệu chứng chính)
        - HPI - History of Present Illness (bệnh sử hiện tại)
        - Past medical history (tiền sử bệnh lý)
        - Medication history (đang dùng thuốc gì)
        - Family history (tiền sử gia đình)
        - Social history (hút thuốc, uống rượu...)
              ↓
BƯỚC 4: Physical Examination
        (Khám lâm sàng)
        - General appearance
        - Khám theo hệ thống (system-based exam)
        - Ghi nhận findings (các phát hiện)
              ↓
BƯỚC 5: Preliminary Assessment
        (Đánh giá sơ bộ)
        - Bác sĩ đưa ra differential diagnosis
        - Quyết định có cần thêm tests không
              ↓
BƯỚC 6A: Order Tests (nếu cần)
         (Ra chỉ định xét nghiệm/chẩn đoán hình ảnh)
         - Lab tests
         - Imaging (X-ray, Ultrasound...)
         - Bệnh nhân đi làm tests
         - Chờ kết quả
              ↓
BƯỚC 6B: Review Test Results
         (Xem kết quả)
         - Bác sĩ đọc và interpret kết quả
              ↓
BƯỚC 7: Diagnosis (Chẩn đoán)
        - Đưa ra chẩn đoán cuối cùng
        - Mã hóa theo ICD-10
        - Giải thích cho bệnh nhân
              ↓
BƯỚC 8: Treatment Plan (Kế hoạch điều trị)
        - Kê đơn thuốc (nếu cần)
        - Chỉ định thêm tests/procedures (nếu cần)
        - Advice (lời khuyên): nghỉ ngơi, chế độ ăn...
        - Follow-up plan (tái khám khi nào)
              ↓
BƯỚC 9: Documentation (Ghi chép hồ sơ)
        - Bác sĩ hoàn thiện medical note
        - Sign and finalize encounter
              ↓
BƯỚC 10: Patient Education & Checkout
         (Giải thích và hoàn tất)
         - Trao đổi với bệnh nhân về diagnosis & treatment
         - In đơn thuốc, hướng dẫn
         - Bệnh nhân ra quầy thanh toán
```

---

## 2. SOAP Note Format

**SOAP** là framework tiêu chuẩn cho clinical documentation:

### S - Subjective (Chủ quan)
**Những gì bệnh nhân kể**

**Chief Complaint (CC)**: Lý do khám chính
- Ví dụ: "Đau bụng, sốt 3 ngày"

**History of Present Illness (HPI)**: Chi tiết triệu chứng
- Onset: Khi nào bắt đầu?
- Location: Đau ở đâu?
- Duration: Kéo dài bao lâu?
- Character: Tính chất (đau nhói, đau tức...)
- Aggravating/Relieving factors: Gì làm nặng/nhẹ hơn?
- Associated symptoms: Triệu chứng kèm theo

**Past Medical History (PMH)**: Tiền sử bệnh
- Bệnh mãn tính: Đái tháo đường, cao huyết áp...
- Phẫu thuật trước đây

**Medications**: Thuốc đang dùng

**Allergies**: Dị ứng

**Family History (FH)**: Tiền sử gia đình

**Social History (SH)**: Hút thuốc, uống rượu...

### O - Objective (Khách quan)
**Những gì bác sĩ quan sát/đo được**

**Vital Signs**:
- BP: 120/80 mmHg
- HR: 72 bpm
- Temp: 37.0°C
- RR: 16/min
- SpO2: 98%
- Weight: 65 kg
- Height: 170 cm
- BMI: 22.5

**Physical Examination Findings**:
- General: Alert, well-nourished
- HEENT: Normocephalic, pupils equal and reactive
- Cardiovascular: Regular rhythm, no murmurs
- Respiratory: Clear breath sounds bilaterally
- Abdomen: Soft, tenderness at RLQ (right lower quadrant)
- ...

**Lab/Imaging Results** (nếu có):
- CBC: WBC 12,000 (elevated)
- CXR: Clear lung fields

### A - Assessment (Đánh giá)
**Chẩn đoán**

**Diagnosis** (mã ICD-10):
- Primary diagnosis: Acute appendicitis (K35.8)
- Secondary diagnosis: ...

**Differential diagnosis** (các chẩn đoán phân biệt - nếu chưa chắc chắn):
- Acute gastroenteritis
- Urinary tract infection

### P - Plan (Kế hoạch)
**Điều trị và theo dõi**

**Medications**:
- Amoxicillin 500mg PO TID x 7 days
- Paracetamol 500mg PO PRN for pain

**Procedures**:
- Refer to surgery for appendectomy

**Tests ordered**:
- CT abdomen

**Patient education**:
- Avoid fatty foods
- Rest, increase fluid intake

**Follow-up**:
- Return in 3 days if symptoms worsen
- Recheck in 1 week

---

## 3. Data Model

### 3.1 Encounter (Visit) Table

```sql
CREATE TABLE encounters (
  encounter_id INT PRIMARY KEY AUTO_INCREMENT,
  patient_id INT NOT NULL,
  visit_number VARCHAR(20) UNIQUE NOT NULL,     -- Số phiếu khám (VN format)

  -- Visit info
  encounter_type ENUM('Outpatient', 'Inpatient', 'Emergency') DEFAULT 'Outpatient',
  visit_reason ENUM('New Visit', 'Follow-up', 'Annual Checkup', 'Emergency'),

  -- Timing
  scheduled_datetime DATETIME,                   -- Giờ hẹn (nếu có)
  arrival_datetime DATETIME,                     -- Giờ đến
  triage_datetime DATETIME,                      -- Giờ vào triage
  doctor_start_datetime DATETIME,                -- Giờ bác sĩ bắt đầu khám
  doctor_end_datetime DATETIME,                  -- Giờ kết thúc khám
  checkout_datetime DATETIME,                    -- Giờ checkout

  -- Assigned staff
  assigned_doctor_id INT,                        -- Bác sĩ khám
  triage_nurse_id INT,                           -- Y tá triage

  -- Status
  status ENUM('Scheduled', 'Checked-in', 'In Triage', 'Waiting for Doctor',
              'In Consultation', 'Pending Tests', 'Completed', 'Cancelled')
         DEFAULT 'Checked-in',

  -- Chief complaint
  chief_complaint TEXT,                          -- Lý do khám

  -- Timestamps
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

  FOREIGN KEY (patient_id) REFERENCES patients(patient_id),
  FOREIGN KEY (assigned_doctor_id) REFERENCES users(user_id),
  FOREIGN KEY (triage_nurse_id) REFERENCES users(user_id),

  INDEX idx_patient (patient_id),
  INDEX idx_doctor (assigned_doctor_id),
  INDEX idx_visit_date (arrival_datetime),
  INDEX idx_status (status)
);
```

### 3.2 Vital Signs Table

```sql
CREATE TABLE vital_signs (
  vital_sign_id INT PRIMARY KEY AUTO_INCREMENT,
  encounter_id INT NOT NULL,
  patient_id INT NOT NULL,

  -- Vital signs
  blood_pressure_systolic INT,                   -- mmHg
  blood_pressure_diastolic INT,                  -- mmHg
  heart_rate INT,                                -- bpm
  respiratory_rate INT,                          -- breaths/min
  temperature DECIMAL(4,1),                      -- °C
  spo2 INT,                                      -- %
  weight DECIMAL(5,2),                           -- kg
  height DECIMAL(5,2),                           -- cm
  bmi DECIMAL(4,2),                              -- Auto-calculated

  -- Pain scale
  pain_scale INT,                                -- 0-10

  -- Who recorded
  recorded_by INT,                               -- User ID (nurse)
  recorded_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

  -- Notes
  notes TEXT,

  FOREIGN KEY (encounter_id) REFERENCES encounters(encounter_id),
  FOREIGN KEY (patient_id) REFERENCES patients(patient_id),
  FOREIGN KEY (recorded_by) REFERENCES users(user_id),

  INDEX idx_encounter (encounter_id),
  INDEX idx_patient (patient_id)
);
```

### 3.3 Clinical Notes (SOAP Notes)

```sql
CREATE TABLE clinical_notes (
  note_id INT PRIMARY KEY AUTO_INCREMENT,
  encounter_id INT NOT NULL,
  patient_id INT NOT NULL,

  -- SOAP sections
  subjective TEXT,                               -- S: Chief complaint, HPI, PMH, etc.
  objective TEXT,                                -- O: Physical exam findings
  assessment TEXT,                               -- A: Diagnosis
  plan TEXT,                                     -- P: Treatment plan

  -- Additional sections
  history_of_present_illness TEXT,               -- HPI chi tiết
  review_of_systems TEXT,                        -- ROS
  physical_examination TEXT,                     -- PE chi tiết

  -- Diagnosis codes
  primary_diagnosis_icd10 VARCHAR(10),
  primary_diagnosis_name VARCHAR(255),

  -- Status
  status ENUM('Draft', 'Signed', 'Amended') DEFAULT 'Draft',

  -- Authorship
  author_id INT NOT NULL,                        -- Bác sĩ viết note
  signed_by INT,                                 -- Bác sĩ ký
  signed_at TIMESTAMP,

  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

  FOREIGN KEY (encounter_id) REFERENCES encounters(encounter_id),
  FOREIGN KEY (patient_id) REFERENCES patients(patient_id),
  FOREIGN KEY (author_id) REFERENCES users(user_id),
  FOREIGN KEY (signed_by) REFERENCES users(user_id),

  INDEX idx_encounter (encounter_id),
  INDEX idx_patient (patient_id),
  INDEX idx_author (author_id)
);
```

### 3.4 Diagnosis Table

```sql
CREATE TABLE encounter_diagnoses (
  diagnosis_id INT PRIMARY KEY AUTO_INCREMENT,
  encounter_id INT NOT NULL,
  patient_id INT NOT NULL,

  -- Diagnosis
  icd10_code VARCHAR(10) NOT NULL,
  diagnosis_name VARCHAR(255) NOT NULL,
  diagnosis_type ENUM('Primary', 'Secondary', 'Differential'),

  -- Severity
  severity ENUM('Mild', 'Moderate', 'Severe'),

  -- Status
  status ENUM('Active', 'Resolved', 'Ruled Out') DEFAULT 'Active',

  -- Timestamps
  diagnosed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  diagnosed_by INT,                              -- Doctor ID

  FOREIGN KEY (encounter_id) REFERENCES encounters(encounter_id),
  FOREIGN KEY (patient_id) REFERENCES patients(patient_id),
  FOREIGN KEY (diagnosed_by) REFERENCES users(user_id),

  INDEX idx_encounter (encounter_id),
  INDEX idx_patient (patient_id),
  INDEX idx_icd10 (icd10_code)
);
```

---

## 4. UI/UX Design

### 4.1 Triage Screen (Y tá)

```
┌─────────────────────────────────────────────────────┐
│  TRIAGE - Bệnh nhân: Nguyễn Văn An (P2023001234)   │
└─────────────────────────────────────────────────────┘

┌─ Vital Signs ────────────────────────────────────────┐
│  Huyết áp:  [___] / [___] mmHg                      │
│  Nhịp tim:  [___] bpm                                │
│  Nhiệt độ:  [___] °C                                 │
│  Nhịp thở:  [___] /phút                              │
│  SpO2:      [___] %                                  │
│  Cân nặng:  [___] kg   Chiều cao: [___] cm          │
│  BMI:       22.5 (auto-calculated)                   │
└──────────────────────────────────────────────────────┘

┌─ Lý do khám ─────────────────────────────────────────┐
│  [Đau bụng, sốt 3 ngày________________________]     │
└──────────────────────────────────────────────────────┘

┌─ Pain Scale (0-10) ──────────────────────────────────┐
│  [○][○][○][○][○][●][○][○][○][○][○]                 │
│   0  1  2  3  4  5  6  7  8  9  10                   │
└──────────────────────────────────────────────────────┘

    [Hủy]  [Lưu và gửi cho bác sĩ]
```

### 4.2 Doctor Consultation Screen

**Layout chính:**
```
┌────────────────────────────────────────────────────────────────┐
│  Bệnh nhân: Nguyễn Văn An - 38 tuổi - Nam                    │
│  MRN: P2023001234  |  Visit: V20231119001                    │
└────────────────────────────────────────────────────────────────┘

┌─ Left Panel ──────────┐  ┌─ Main Panel ──────────────────────┐
│                        │  │                                    │
│  📋 Thông tin tóm tắt  │  │  [S] [O] [A] [P] [Orders] [Labs]  │
│  ─────────────────────│  │  ──────────────────────────────────│
│  Vital Signs (Today):  │  │                                    │
│  • BP: 130/85          │  │  SUBJECTIVE:                       │
│  • HR: 78              │  │  CC: Đau bụng, sốt 3 ngày         │
│  • Temp: 38.2°C ⚠️     │  │                                    │
│  • Weight: 65kg        │  │  HPI:                              │
│                        │  │  [___________________________]     │
│  ⚠️ Dị ứng:            │  │  [___________________________]     │
│  • Penicillin          │  │                                    │
│                        │  │  PMH:                              │
│  Bệnh mãn tính:        │  │  • Đái tháo đường type 2 (2020)   │
│  • Đái tháo đường      │  │  • Cao huyết áp (2019)             │
│                        │  │                                    │
│  Lịch sử khám (5):     │  │  Medications:                      │
│  • 15/10 - Cảm cúm     │  │  • Metformin 500mg BID             │
│  • 01/09 - Tái khám DM │  │  • Amlodipine 5mg QD               │
│  • ...                 │  │                                    │
│                        │  │  [Save Draft]  [Next: Objective]   │
│  [Xem đầy đủ]          │  │                                    │
└────────────────────────┘  └────────────────────────────────────┘
```

**Quick Actions Bar:**
```
[📝 Templates]  [🔍 ICD-10 Search]  [💊 Prescribe]  [🧪 Order Labs]  [📄 Print]
```

### 4.3 Diagnosis Input với ICD-10 Autocomplete

```
┌─ Chẩn đoán ──────────────────────────────────────────┐
│                                                       │
│  Chẩn đoán chính:                                    │
│  [Viêm ruột thừa________________] [🔍]               │
│                                                       │
│  Suggestions:                                         │
│  ┌─────────────────────────────────────────────────┐│
│  │ K35.8 - Viêm ruột thừa cấp                      ││
│  │ K35.2 - Viêm ruột thừa có áp xe                 ││
│  │ K35.3 - Viêm ruột thừa có thủng lan tỏa         ││
│  └─────────────────────────────────────────────────┘│
│                                                       │
│  ☐ Chẩn đoán phân biệt (Differential):              │
│    • Viêm dạ dày ruột cấp (A09)                     │
│    • Nhiễm trùng đường tiết niệu (N39.0)            │
│                                                       │
└───────────────────────────────────────────────────────┘
```

---

## 5. Clinical Decision Support (CDS)

### 5.1 Drug Interaction Alerts

**Scenario**: Bác sĩ kê thuốc mới, system check tương tác với thuốc đang dùng

```javascript
// Check drug interactions
async function checkDrugInteractions(patientId, newMedicationCode) {
  // Get current medications
  const currentMeds = await getCurrentMedications(patientId);

  // Check interactions
  const interactions = await drugDatabase.checkInteractions(
    newMedicationCode,
    currentMeds
  );

  if (interactions.length > 0) {
    return {
      alert: true,
      severity: interactions[0].severity, // 'Critical', 'Moderate', 'Mild'
      message: `Cảnh báo: ${newMedicationName} có tương tác với ${interactions[0].drugName}`,
      details: interactions[0].description
    };
  }

  return { alert: false };
}
```

**UI Alert:**
```
┌─ ⚠️  CẢNH BÁO TƯƠNG TÁC THUỐC ─────────────────────┐
│                                                      │
│  Warfarin có tương tác NGHIÊM TRỌNG với Aspirin     │
│                                                      │
│  Nguy cơ: Tăng nguy cơ chảy máu                    │
│                                                      │
│  Khuyến nghị: Cân nhắc thay thế hoặc giảm liều      │
│                                                      │
│  [Xem chi tiết]  [Vẫn kê đơn]  [Hủy]               │
└──────────────────────────────────────────────────────┘
```

### 5.2 Allergy Alerts

```javascript
// Check allergies before prescribing
async function checkAllergies(patientId, medicationCode) {
  const allergies = await getPatientAllergies(patientId);

  const allergyMatch = allergies.find(allergy => {
    // Check exact drug match
    if (allergy.allergen_name === medicationCode) return true;

    // Check drug class match (e.g., patient allergic to Penicillin → alert for Amoxicillin)
    if (isDrugInSameClass(medicationCode, allergy.allergen_name)) return true;

    return false;
  });

  if (allergyMatch) {
    return {
      alert: true,
      severity: allergyMatch.severity,
      message: `Bệnh nhân DỊ ỨNG với ${allergyMatch.allergen_name}`,
      reaction: allergyMatch.reaction
    };
  }

  return { alert: false };
}
```

### 5.3 Vital Signs Alerts

**Auto-highlight abnormal values:**
```javascript
function checkVitalSignsAlerts(vitalSigns) {
  const alerts = [];

  // Blood pressure
  if (vitalSigns.systolic > 140 || vitalSigns.diastolic > 90) {
    alerts.push({
      type: 'Hypertension',
      severity: 'Warning',
      message: 'Huyết áp cao'
    });
  }

  if (vitalSigns.systolic < 90 || vitalSigns.diastolic < 60) {
    alerts.push({
      type: 'Hypotension',
      severity: 'Critical',
      message: 'Huyết áp thấp'
    });
  }

  // Temperature
  if (vitalSigns.temperature >= 38.0) {
    alerts.push({
      type: 'Fever',
      severity: 'Warning',
      message: 'Sốt'
    });
  }

  // SpO2
  if (vitalSigns.spo2 < 95) {
    alerts.push({
      type: 'Low Oxygen',
      severity: 'Critical',
      message: 'SpO2 thấp - cần can thiệp'
    });
  }

  return alerts;
}
```

---

## 6. Templates & Shortcuts

### 6.1 Clinical Note Templates

**Common templates cho các trường hợp thường gặp:**

**Template: Common Cold (Cảm cúm)**
```
S: Sốt, ho, sổ mũi [X] ngày
O:
  - Temp: [__]°C
  - Throat: Erythematous
  - Lungs: Clear
A: J00 - Acute nasopharyngitis (common cold)
P:
  - Paracetamol 500mg PO TID PRN fever
  - Increase fluid intake
  - Rest
  - Return if fever > 3 days
```

**Template: Hypertension Follow-up (Tái khám cao huyết áp)**
```
S: Tái khám cao huyết áp. Đang dùng [thuốc]. Không có triệu chứng bất thường.
O:
  - BP: [__]/[__] mmHg
  - HR: [__] bpm
A: I10 - Essential hypertension, controlled
P:
  - Continue current medications
  - Monitor BP at home
  - Low salt diet
  - Recheck in 3 months
```

### 6.2 Text Expansion / Shortcuts

**Macros để gõ nhanh:**
- `.bp` → expands to "Blood pressure: __/__ mmHg"
- `.temp` → expands to "Temperature: __°C"
- `.normal` → expands to "Within normal limits"
- `.wnl` → "Within normal limits"
- `.perrla` → "Pupils equal, round, reactive to light and accommodation"

---

## 7. Integration with Other Workflows

### 7.1 Lab Orders

**Từ consultation screen → Order labs:**
```
Doctor clicks [Order Labs]
      ↓
Lab order form opens
      ↓
Select tests (CBC, BMP, Urinalysis...)
      ↓
Submit order
      ↓
Order sent to Lab System (LIS)
      ↓
Patient goes to lab
      ↓
Results come back
      ↓
Doctor reviews results in EMR
```

### 7.2 Prescription

**Từ consultation screen → Prescribe:**
```
Doctor clicks [Prescribe]
      ↓
Medication search/select
      ↓
CDS alerts (allergies, interactions)
      ↓
Specify: dosage, route, frequency, duration
      ↓
Add instructions
      ↓
Save prescription
      ↓
E-prescription sent to pharmacy OR printed for patient
```

*Chi tiết xem [04-prescription-pharmacy.md]*

### 7.3 Referrals (Chuyển tuyến)

**Khi bệnh cần chuyên khoa:**
```
Doctor decides: Need specialist
      ↓
Click [Referral]
      ↓
Select specialty (Cardiology, Orthopedics...)
      ↓
Write referral note (reason, relevant history)
      ↓
Print referral letter for patient
      ↓
(Optional) Send electronically to specialist clinic
```

---

## 8. Performance Optimization

### 8.1 Reduce Doctor's Clicks

**Problem**: Bác sĩ phải click quá nhiều → mất thời gian, frustrating

**Solutions:**
1. **Keyboard shortcuts**:
   - Ctrl+S: Save draft
   - Ctrl+Enter: Sign note
   - Alt+T: Templates
   - Alt+D: Diagnosis search

2. **Smart defaults**:
   - Auto-populate với data from triage
   - Remember doctor's preferences (common diagnoses, medications)

3. **Voice input**: Speech-to-text cho note-taking

4. **Single-page workflow**: Không phải navigate nhiều screens

### 8.2 Fast Note Templates

**Pre-built templates for 80% common cases:**
- Upper respiratory infection
- Hypertension follow-up
- Diabetes follow-up
- Annual check-up
- Acute gastroenteritis
- ...

**Bác sĩ chỉ cần:**
1. Select template
2. Fill in blanks
3. Adjust as needed
4. Sign

### 8.3 Auto-save & Recovery

```javascript
// Auto-save draft every 30 seconds
setInterval(async () => {
  const noteData = getCurrentNoteData();
  await saveDraft(noteData);
  console.log('Auto-saved at', new Date());
}, 30000);

// On page load, check for unsaved draft
onPageLoad(async () => {
  const draft = await checkForDraft(encounterId);
  if (draft && draft.updated_at > lastSavedVersion) {
    showModal('Có bản nháp chưa lưu. Bạn có muốn khôi phục?');
  }
});
```

---

## 9. Common Issues & Solutions

### Issue 1: Bác sĩ quên sign note
**Problem**: Note ở trạng thái Draft, không finalize
**Solution**:
- End-of-day reminder: "Bạn có 5 notes chưa sign"
- Dashboard widget showing unsigned notes
- Cannot checkout patient until note is signed (configurable)

### Issue 2: Copy-paste từ note cũ
**Problem**: Bác sĩ copy note lần trước, quên update → sai thông tin
**Solution**:
- Highlight fields that are copied
- Mandatory fields phải re-enter (không cho copy)
- Warning nếu note quá giống note trước

### Issue 3: Thiếu diagnosis code
**Problem**: Bác sĩ viết diagnosis nhưng không mã ICD-10 → không claim BHYT được
**Solution**:
- Mandatory ICD-10 code trước khi sign
- Smart suggestion: AI suggest ICD-10 based on note text
- Validation: Cannot finalize without ICD-10

### Issue 4: Quá nhiều thông tin, bác sĩ overwhelmed
**Problem**: Screen quá đầy information
**Solution**:
- Collapsible sections
- Focus mode (hide non-essential info)
- Customizable layout (doctor can choose what to show)

---

## 10. Mobile Considerations

**Bác sĩ có thể dùng tablet trong khi khám:**

```
┌─────────────────────────────┐
│  👤 Nguyễn Văn An - 38T    │
│  ───────────────────────── │
│  [SOAP] [Vitals] [History] │
│                             │
│  Subjective:                │
│  [Voice Input 🎤]          │
│  "Bệnh nhân than đau bụng"  │
│                             │
│  [📝 Templates]             │
│  [💊 Quick Prescribe]       │
│  [✓ Sign & Next Patient]   │
└─────────────────────────────┘
```

**Mobile-friendly features:**
- Large touch targets
- Swipe gestures (swipe → next section)
- Voice input
- Simplified layout
- Offline mode (sync when online)

---

## Summary

**Key takeaways:**

1. **SOAP format** là standard cho clinical documentation
2. **Optimize for speed**: Templates, shortcuts, smart defaults
3. **Clinical Decision Support**: Drug interactions, allergies, vital signs alerts
4. **Integration**: Seamless với lab orders, prescriptions, referrals
5. **Mobile-friendly**: Bác sĩ có thể dùng tablet
6. **Auto-save**: Không mất dữ liệu
7. **ICD-10 mandatory**: Cho BHYT claims

**For implementation:**
- Start with basic SOAP editor
- Add templates incrementally
- Implement CDS alerts (critical for patient safety)
- Optimize UX based on doctor feedback
- Mobile support từ đầu (tablet usage is common)

**Next**: [03-diagnostics-lab-tests.md](03-diagnostics-lab-tests.md)
