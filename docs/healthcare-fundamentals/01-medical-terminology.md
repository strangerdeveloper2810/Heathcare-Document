# Medical Terminology Basics

## Tại sao dev cần hiểu Medical Terminology?

Khi làm phần mềm y tế, bạn sẽ gặp vô số thuật ngữ y học trong:
- Requirements từ doctors/nurses
- Database schema (tên bảng, tên field)
- API endpoints và response data
- Bug reports từ users

Nếu không hiểu terminology, bạn sẽ:
- Design sai data model
- Hiểu sai requirements
- Khó communicate với stakeholders
- Tạo ra bugs nguy hiểm (vì y tế liên quan đến tính mạng con người)

---

## 1. Cấu trúc của thuật ngữ y học

Thuật ngữ y học thường được tạo từ **3 thành phần**:

### Prefix (Tiền tố) - Vị trí/Hướng
- **hyper-**: cao, tăng (hypertension = tăng huyết áp)
- **hypo-**: thấp, giảm (hypoglycemia = hạ đường huyết)
- **tachy-**: nhanh (tachycardia = nhịp tim nhanh)
- **brady-**: chậm (bradycardia = nhịp tim chậm)
- **poly-**: nhiều (polydipsia = uống nhiều nước)
- **dys-**: khó khăn, bất thường (dyspnea = khó thở)
- **a-/an-**: không có (anemia = thiếu máu)

### Root (Gốc từ) - Cơ quan/Hệ thống
- **card/cardio**: tim (cardiology = khoa tim mạch)
- **hepat**: gan (hepatitis = viêm gan)
- **nephro**: thận (nephrology = khoa thận)
- **gastro**: dạ dày (gastritis = viêm dạ dày)
- **pneumo**: phổi (pneumonia = viêm phổi)
- **derm/dermato**: da (dermatology = khoa da liễu)
- **neuro**: thần kinh (neurology = khoa thần kinh)
- **osteo**: xương (osteoporosis = loãng xương)
- **hemo/hemato**: máu (hematology = khoa huyết học)

### Suffix (Hậu tố) - Triệu chứng/Điều trị/Bệnh
- **-itis**: viêm (appendicitis = viêm ruột thừa)
- **-osis**: tình trạng bệnh lý (cirrhosis = xơ gan)
- **-ology**: chuyên khoa (oncology = khoa ung bướu)
- **-ectomy**: cắt bỏ (appendectomy = phẫu thuật cắt ruột thừa)
- **-otomy**: rạch, mở (tracheotomy = mở khí quản)
- **-plasty**: tạo hình (rhinoplasty = phẫu thuật mũi)
- **-scopy**: soi, nội soi (endoscopy = nội soi)
- **-gram**: hình ảnh (electrocardiogram/ECG = điện tâm đồ)

### Ví dụ thực tế:
- **Hypertension** = hyper (tăng) + tension (áp lực) = Tăng huyết áp
- **Tachycardia** = tachy (nhanh) + cardia (tim) = Nhịp tim nhanh
- **Hepatitis** = hepat (gan) + itis (viêm) = Viêm gan
- **Gastrectomy** = gastr (dạ dày) + ectomy (cắt bỏ) = Cắt dạ dày

---

## 2. Thuật ngữ theo hệ thống cơ thể

### Hệ tim mạch (Cardiovascular System)
| Thuật ngữ | Ý nghĩa | Ghi chú cho dev |
|-----------|---------|-----------------|
| Hypertension | Tăng huyết áp | BP > 140/90 mmHg |
| Hypotension | Hạ huyết áp | BP < 90/60 mmHg |
| MI (Myocardial Infarction) | Nhồi máu cơ tim | Cần emergency workflow |
| Arrhythmia | Rối loạn nhịp tim | |
| CHF (Congestive Heart Failure) | Suy tim sung huyết | Chronic condition |
| BP (Blood Pressure) | Huyết áp | Format: systolic/diastolic (120/80) |
| HR (Heart Rate) | Nhịp tim | BPM (beats per minute) |

### Hệ hô hấp (Respiratory System)
| Thuật ngữ | Ý nghĩa | Ghi chú cho dev |
|-----------|---------|-----------------|
| Dyspnea | Khó thở | Subjective symptom |
| Asthma | Hen suyễn | Chronic condition |
| COPD | Bệnh phổi tắc nghẽn mãn tính | Common in elderly |
| Pneumonia | Viêm phổi | Requires antibiotics |
| SpO2 | Độ bão hòa oxy trong máu | Normal: 95-100% |
| RR (Respiratory Rate) | Nhịp thở | Breaths per minute |

### Hệ tiêu hóa (Gastrointestinal System)
| Thuật ngữ | Ý nghĩa | Ghi chú cho dev |
|-----------|---------|-----------------|
| Gastritis | Viêm dạ dày | |
| GERD | Trào ngược dạ dày thực quản | Chronic condition |
| Hepatitis | Viêm gan | A, B, C types |
| Cirrhosis | Xơ gan | End-stage liver disease |
| Appendicitis | Viêm ruột thừa | Emergency surgery needed |
| IBD | Bệnh viêm ruột | Includes Crohn's, UC |

### Hệ thần kinh (Neurological System)
| Thuật ngữ | Ý nghĩa | Ghi chú cho dev |
|-----------|---------|-----------------|
| CVA/Stroke | Đột quỵ | Emergency condition |
| Seizure | Co giật | |
| Migraine | Đau nửa đầu | |
| Dementia | Mất trí nhớ, sa sút trí tuệ | Alzheimer's là type phổ biến nhất |
| GCS (Glasgow Coma Scale) | Thang điểm hôn mê | Score: 3-15 |

### Hệ tiết niệu (Urinary System)
| Thuật ngữ | Ý nghĩa | Ghi chú cho dev |
|-----------|---------|-----------------|
| UTI (Urinary Tract Infection) | Nhiễm trùng đường tiết niệu | Very common |
| Nephritis | Viêm thận | |
| CKD (Chronic Kidney Disease) | Bệnh thận mãn tính | 5 stages |
| Dialysis | Lọc máu, thẩm phân | For kidney failure |

### Hệ nội tiết (Endocrine System)
| Thuật ngữ | Ý nghĩa | Ghi chú cho dev |
|-----------|---------|-----------------|
| DM (Diabetes Mellitus) | Đái tháo đường | Type 1, Type 2 |
| Hyperglycemia | Tăng đường huyết | High blood sugar |
| Hypoglycemia | Hạ đường huyết | Emergency if severe |
| Thyroid disorders | Rối loạn tuyến giáp | Hyper/Hypothyroidism |

---

## 3. Vital Signs (Dấu hiệu sinh tồn)

**Vital signs** là những chỉ số cơ bản nhất để đánh giá tình trạng sức khỏe. Trong EMR, đây là data phải collect ở mọi lần khám.

| Vital Sign | Viết tắt | Đơn vị | Giá trị bình thường | Ghi chú kỹ thuật |
|------------|----------|--------|---------------------|------------------|
| Huyết áp | BP | mmHg | 120/80 | Format: systolic/diastolic |
| Nhịp tim | HR/Pulse | BPM | 60-100 | Beats per minute |
| Nhịp thở | RR | /min | 12-20 | Breaths per minute |
| Nhiệt độ | Temp | °C | 36.5-37.5 | Hoặc °F (US) |
| SpO2 | SpO2 | % | 95-100 | Oxygen saturation |
| Cân nặng | Weight | kg | - | Track changes |
| Chiều cao | Height | cm | - | Calculate BMI |
| BMI | BMI | kg/m² | 18.5-24.9 | Auto-calculate từ height/weight |

### Cảnh báo cho dev:
- **Validation rules**: Vital signs phải có range validation (ví dụ: HR không thể là 300 BPM)
- **Unit conversion**: VN dùng °C, US dùng °F → cần convert
- **Critical values**: Một số giá trị cần alert ngay (ví dụ: SpO2 < 90%)
- **Trending**: Vital signs quan trọng ở việc theo dõi xu hướng (tăng/giảm theo thời gian)

---

## 4. Common Abbreviations trong Healthcare

Trong healthcare, mọi thứ đều được viết tắt để tiết kiệm thời gian. Bạn sẽ thấy chúng trong:
- Requirements docs
- Database fields
- Medical reports
- Doctor's notes

### General Abbreviations
| Viết tắt | Đầy đủ | Ý nghĩa tiếng Việt |
|----------|--------|-------------------|
| Pt/PT | Patient | Bệnh nhân |
| Dx | Diagnosis | Chẩn đoán |
| Tx | Treatment | Điều trị |
| Rx | Prescription | Đơn thuốc |
| Sx | Symptoms | Triệu chứng |
| Hx | History | Tiền sử |
| PMH | Past Medical History | Tiền sử bệnh lý |
| FH | Family History | Tiền sử gia đình |
| CC | Chief Complaint | Lý do khám chính |
| HPI | History of Present Illness | Bệnh sử hiện tại |
| ROS | Review of Systems | Khám hệ thống |
| PE | Physical Examination | Khám lâm sàng |

### Timing & Frequency
| Viết tắt | Đầy đủ | Ý nghĩa |
|----------|--------|---------|
| stat | Immediately | Ngay lập tức (emergency) |
| PRN | Pro re nata | Khi cần thiết |
| QD | Quaque die | Mỗi ngày 1 lần |
| BID | Bis in die | Ngày 2 lần |
| TID | Ter in die | Ngày 3 lần |
| QID | Quater in die | Ngày 4 lần |
| AC | Ante cibum | Trước ăn |
| PC | Post cibum | Sau ăn |
| HS | Hora somni | Trước khi ngủ |

**⚠️ Critical for dev**: Khi build medication module, phải support đầy đủ các frequency này.

### Lab & Diagnostics
| Viết tắt | Đầy đủ | Ý nghĩa |
|----------|--------|---------|
| CBC | Complete Blood Count | Công thức máu |
| BMP | Basic Metabolic Panel | Xét nghiệm chuyển hóa cơ bản |
| CMP | Comprehensive Metabolic Panel | Xét nghiệm chuyển hóa toàn diện |
| LFT | Liver Function Test | Xét nghiệm chức năng gan |
| RFT | Renal Function Test | Xét nghiệm chức năng thận |
| ECG/EKG | Electrocardiogram | Điện tâm đồ |
| CXR | Chest X-Ray | X-quang ngực |
| CT | Computed Tomography | Chụp CT |
| MRI | Magnetic Resonance Imaging | Chụp cộng hưởng từ |
| US | Ultrasound | Siêu âm |

### Departments
| Viết tắt | Đầy đủ | Ý nghĩa tiếng Việt |
|----------|--------|-------------------|
| ER/ED | Emergency Room/Department | Khoa cấp cứu |
| ICU | Intensive Care Unit | Khoa hồi sức tích cực |
| OR | Operating Room | Phòng mổ |
| OPD | Outpatient Department | Khoa ngoại trú |
| IPD | Inpatient Department | Khoa nội trú |
| L&D | Labor and Delivery | Khoa sản |
| NICU | Neonatal ICU | Hồi sức sơ sinh |
| Peds | Pediatrics | Khoa nhi |
| OB/GYN | Obstetrics/Gynecology | Khoa sản phụ |

---

## 5. Patient Types & Visit Types

Hiểu patient types rất quan trọng cho workflow design.

### Patient Types
| Type | Mô tả | Workflow implications |
|------|-------|----------------------|
| **Outpatient (OPD)** | Bệnh nhân ngoại trú - khám xong về | Simple workflow: registration → consult → payment |
| **Inpatient (IPD)** | Bệnh nhân nội trú - nhập viện | Complex: admission → daily rounds → discharge |
| **Emergency** | Bệnh nhân cấp cứu | Priority workflow, skip some steps |
| **Day case** | Phẫu thuật không qua đêm | Hybrid: như IPD nhưng discharge trong ngày |

### Visit Types cho Clinic/Phòng khám
| Visit Type | Mô tả | Trong hệ thống |
|------------|-------|----------------|
| First Visit | Lần đầu khám | Cần thu thập đầy đủ history |
| Follow-up | Tái khám | Reference previous visits |
| Annual Check-up | Khám định kỳ | Comprehensive exam |
| Consultation | Hội chẩn | Multiple doctors involved |

---

## 6. Medication Terminology

### Routes of Administration (Đường dùng thuốc)
| Route | Viết tắt | Mô tả |
|-------|----------|-------|
| Oral | PO | Uống |
| Intravenous | IV | Tiêm tĩnh mạch |
| Intramuscular | IM | Tiêm bắp |
| Subcutaneous | SC/SubQ | Tiêm dưới da |
| Topical | TOP | Bôi ngoài da |
| Inhalation | INH | Hít |
| Rectal | PR | Đặt hậu môn |
| Sublingual | SL | Ngậm dưới lưỡi |

### Dosage Forms
- Tablet (viên nén)
- Capsule (viên nang)
- Syrup (siro)
- Injection (tiêm)
- Cream/Ointment (kem/thuốc mỡ)
- Suppository (viên đặt)
- Inhaler (ống hít)

**Dev note**: Trong database, cần store:
- Medication name
- Dosage (liều lượng): số + đơn vị (mg, ml, IU...)
- Route
- Frequency (QD, BID, TID...)
- Duration (bao lâu)
- Instructions (hướng dẫn sử dụng)

---

## 7. Severity & Status Terms

Các thuật ngữ về mức độ nghiêm trọng và tình trạng:

### Severity
- **Acute**: Cấp tính (đột ngột, nghiêm trọng, ngắn hạn)
- **Chronic**: Mãn tính (kéo dài, lâu dài)
- **Mild**: Nhẹ
- **Moderate**: Trung bình
- **Severe**: Nặng
- **Critical**: Nguy kịch

### Status
- **Active**: Đang điều trị
- **Resolved**: Đã khỏi
- **Remission**: Thuyên giảm (bệnh mạn tính tạm thời không có triệu chứng)
- **Stable**: Ổn định
- **Worsening**: Tiến triển xấu
- **Improving**: Đang tiến triển tốt

---

## 8. Practical Tips cho Dev

### 1. Khi đọc requirements:
- Tra Google với "medical term + meaning"
- Dùng https://www.medicinenet.com hoặc https://medlineplus.gov
- Hỏi PM/BA để clarify, đừng tự đoán

### 2. Khi design database:
- **Đừng viết tắt quá mức**: `patient_blood_pressure` tốt hơn `pt_bp`
- **Standardize units**: Quy định rõ đơn vị (mmHg, mg/dl...)
- **Use enums/lookups**: Cho status, severity, visit types...
- **Validate ranges**: Vital signs phải có min/max

### 3. Khi implement:
- **Critical data validation**: Sai data y tế = nguy hiểm tính mạng
- **Audit trail**: Ai sửa gì, khi nào → compliance requirement
- **Terminology consistency**: Cùng một term trong toàn hệ thống

### 4. Common pitfalls:
- ❌ Dùng `text` field cho tất cả → Không search/filter được
- ❌ Không validate range → Nhập HR = 999 vẫn được
- ❌ Hard-code medical terms → Không flexible
- ✅ Dùng reference tables cho terminology
- ✅ Implement proper validation
- ✅ Support multi-language (EN/VN medical terms)

---

## 9. Resources để học thêm

### Online Resources:
- **MedlinePlus**: https://medlineplus.gov (NIH - official)
- **Medical Dictionary**: https://www.medicinenet.com
- **WHO ICD-10**: https://icd.who.int (Disease classification)

### Vietnamese Resources:
- Từ điển y học Anh-Việt
- Cổng thông tin Bộ Y Tế: https://moh.gov.vn

### Practice:
- Đọc medical reports thực tế (nếu có access)
- Hỏi stakeholders (doctors, nurses) giải thích terms
- Build personal glossary khi làm dự án

---

## Summary

**Key takeaways:**
1. Medical terms có cấu trúc: prefix + root + suffix
2. Hiểu vital signs là must-have cho EMR
3. Healthcare đầy abbreviations - cần tra và note lại
4. Patient types và visit types ảnh hưởng workflow
5. Medication có nhiều thuộc tính phức tạp
6. Data validation trong y tế là CRITICAL - sai số liệu = nguy hiểm

**Next step**: Sau khi nắm terminology, chúng ta sẽ tìm hiểu Healthcare System Overview để hiểu big picture.
