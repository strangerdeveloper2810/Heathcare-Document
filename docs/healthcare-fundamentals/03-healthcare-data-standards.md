# Healthcare Data Standards

## Tại sao dev cần hiểu Healthcare Data Standards?

Healthcare data standards là những quy chuẩn để:
- **Trao đổi dữ liệu** giữa các hệ thống khác nhau (EMR ↔ Lab ↔ Pharmacy...)
- **Mã hóa thông tin y tế** một cách nhất quán (bệnh, thuốc, xét nghiệm...)
- **Đảm bảo interoperability** (khả năng tương tác) giữa các phần mềm

**Nếu không có standards**:
- Mỗi vendor tự định nghĩa format → không tích hợp được
- Dữ liệu không nhất quán → khó phân tích
- Bệnh nhân chuyển bệnh viện → không share được hồ sơ

---

## 1. Messaging Standards (Chuẩn trao đổi thông điệp)

### HL7 (Health Level 7)

**HL7 là gì?**
- Chuẩn quốc tế để trao đổi dữ liệu giữa các hệ thống y tế
- "Level 7" = Application layer trong OSI model
- Được sử dụng rộng rãi nhất trên thế giới

**Phiên bản chính:**

#### HL7 v2.x (Legacy, nhưng vẫn phổ biến nhất)
**Format**: Pipe-delimited text messages (dùng ký tự | để phân cách)

**Ví dụ HL7 v2 message** (ADT^A01 - Patient Admission):
```
MSH|^~\&|SendingApp|SendingFacility|ReceivingApp|ReceivingFacility|20231119120000||ADT^A01|MSG00001|P|2.5
EVN|A01|20231119120000
PID|1||123456^^^Hospital^MR||Nguyen^Van^A||19850315|M|||123 Le Loi Street^Apt 4^HCMC^VN^70000||0909123456
PV1|1|I|Ward 5^Room 501^Bed A|||^Nguyen^Thi^B^Dr|||Internal Medicine
```

**Giải thích:**
- **MSH** (Message Header): Thông tin message
- **EVN** (Event Type): Loại sự kiện
- **PID** (Patient Identification): Thông tin bệnh nhân
- **PV1** (Patient Visit): Thông tin lần khám

**Message types phổ biến:**
| Message Type | Mô tả | Khi nào dùng |
|--------------|-------|--------------|
| ADT^A01 | Admit patient (Nhập viện) | Khi bệnh nhân nhập viện |
| ADT^A03 | Discharge patient (Xuất viện) | Khi bệnh nhân xuất viện |
| ADT^A08 | Update patient info (Cập nhật thông tin) | Khi sửa thông tin bệnh nhân |
| ORM^O01 | Order message (Yêu cầu xét nghiệm/thuốc) | Bác sĩ ra lệnh xét nghiệm |
| ORU^R01 | Observation result (Kết quả xét nghiệm) | Lab trả kết quả về |
| SIU^S12 | Schedule appointment (Đặt lịch hẹn) | Tạo/cập nhật lịch hẹn |

**Ưu điểm:**
- Được hỗ trợ rộng rãi
- Flexible, extensible

**Nhược điểm:**
- Text-based → khó parse
- Không có schema validation cứng
- Mỗi vendor implement hơi khác nhau

#### HL7 v3 (XML-based, ít phổ biến hơn)
- Dùng XML format
- Strict schema (RIM - Reference Information Model)
- Phức tạp hơn, ít được adopt

#### HL7 FHIR (Fast Healthcare Interoperability Resources) - The Future
**FHIR là gì?**
- Chuẩn mới nhất của HL7 (từ 2014)
- RESTful API-based
- JSON/XML format
- Dễ implement hơn nhiều so với v2/v3

**Ví dụ FHIR Patient resource (JSON)**:
```json
{
  "resourceType": "Patient",
  "id": "123456",
  "identifier": [
    {
      "system": "http://hospital.example.com/patients",
      "value": "123456"
    }
  ],
  "name": [
    {
      "family": "Nguyen",
      "given": ["Van", "A"]
    }
  ],
  "gender": "male",
  "birthDate": "1985-03-15",
  "address": [
    {
      "line": ["123 Le Loi Street", "Apt 4"],
      "city": "HCMC",
      "country": "VN",
      "postalCode": "70000"
    }
  ],
  "telecom": [
    {
      "system": "phone",
      "value": "0909123456"
    }
  ]
}
```

**FHIR Resources quan trọng:**
- **Patient**: Thông tin bệnh nhân
- **Practitioner**: Thông tin bác sĩ/nhân viên y tế
- **Encounter**: Lần khám/nhập viện
- **Observation**: Kết quả xét nghiệm, vital signs
- **MedicationRequest**: Đơn thuốc
- **DiagnosticReport**: Báo cáo chẩn đoán
- **Appointment**: Lịch hẹn

**Ưu điểm FHIR:**
- RESTful → dễ integrate (GET, POST, PUT, DELETE)
- JSON → developer-friendly
- Modular (resources độc lập)
- Modern web standards

**Xu hướng:**
- US, EU đang chuyển dần sang FHIR
- VN chưa áp dụng rộng rãi, nhưng nên học

**⚠️ Recommendation cho dev:**
- Hiện tại: Hỗ trợ HL7 v2 (vì phổ biến ở VN)
- Tương lai: Chuẩn bị cho FHIR (thiết kế API RESTful từ đầu)

---

## 2. Clinical Terminology Standards (Chuẩn thuật ngữ lâm sàng)

### ICD (International Classification of Diseases)

**ICD là gì?**
- Hệ thống mã hóa bệnh tật và vấn đề sức khỏe
- Do WHO (World Health Organization) phát hành
- Dùng để ghi chẩn đoán, thống kê, thanh toán bảo hiểm

**Phiên bản:**

#### ICD-10 (Hiện tại)
- Khoảng 68,000 mã
- Format: Ký tự đầu (A-Z) + số

**Ví dụ ICD-10 codes:**
| Mã ICD-10 | Tên tiếng Anh | Tên tiếng Việt |
|-----------|---------------|----------------|
| J00 | Acute nasopharyngitis (common cold) | Viêm mũi họng cấp (cảm lạnh) |
| E11 | Type 2 diabetes mellitus | Đái tháo đường type 2 |
| I10 | Essential (primary) hypertension | Tăng huyết áp nguyên phát |
| J18.9 | Pneumonia, unspecified | Viêm phổi không xác định |
| K29.0 | Acute gastritis | Viêm dạ dày cấp |
| M79.3 | Panniculitis | Viêm mô mỡ dưới da |
| A09 | Diarrhea and gastroenteritis | Tiêu chảy và viêm dạ dày ruột |

**Cấu trúc ICD-10:**
- **Chương** (Chapter): A00-A99 (Infectious diseases), E00-E89 (Endocrine...)
- **Category**: 3 ký tự (E11 = Type 2 DM)
- **Subcategory**: 4+ ký tự (E11.9 = Type 2 DM without complications)

#### ICD-11 (Mới nhất, từ 2022)
- Cải tiến, cập nhật hơn
- Thêm nhiều mã mới (COVID-19, gaming disorder...)
- VN chưa adopt rộng rãi

**Ở VN:**
- Bảo hiểm y tế (BHYT) yêu cầu mã ICD-10 khi thanh toán
- Bệnh viện công phải report ICD-10 cho Bộ Y tế

**⚠️ Implementation tips:**
- Có ICD-10 database trong hệ thống
- Search tiếng Việt + tiếng Anh
- Suggest codes cho bác sĩ (autocomplete)
- Validate mã trước khi submit BHYT

---

### ICD-10-PCS (Procedure Coding System)

**ICD-10-PCS là gì?**
- Hệ thống mã hóa các thủ thuật/phẫu thuật
- Chủ yếu dùng ở US cho inpatient procedures
- VN ít dùng ICD-10-PCS, thường dùng catalog riêng

---

### SNOMED CT (Systematized Nomenclature of Medicine - Clinical Terms)

**SNOMED CT là gì?**
- Hệ thống thuật ngữ y học toàn diện nhất thế giới
- Hơn 350,000 concepts (khái niệm y học)
- Cover: diseases, symptoms, procedures, medications, anatomy...

**Ví dụ SNOMED CT concepts:**
- 38341003: Hypertension (Tăng huyết áp)
- 73211009: Diabetes mellitus (Đái tháo đường)
- 22298006: Myocardial infarction (Nhồi máu cơ tim)

**Khi nào dùng:**
- Clinical decision support systems
- Detailed clinical documentation
- Research

**Ở VN:**
- Chưa được sử dụng rộng rãi
- ICD-10 đơn giản hơn, đủ cho hầu hết trường hợp

---

### LOINC (Logical Observation Identifiers Names and Codes)

**LOINC là gì?**
- Chuẩn mã hóa cho xét nghiệm và quan sát lâm sàng
- Dùng để identify lab tests, vital signs, clinical observations

**Ví dụ LOINC codes:**
| LOINC Code | Tên xét nghiệm | Giải thích |
|------------|----------------|------------|
| 2339-0 | Glucose [Mass/volume] in Blood | Đường huyết |
| 8480-6 | Systolic blood pressure | Huyết áp tâm thu |
| 8462-4 | Diastolic blood pressure | Huyết áp tâm trương |
| 8867-4 | Heart rate | Nhịp tim |
| 2160-0 | Creatinine [Mass/volume] in Serum or Plasma | Creatinine máu |
| 718-7 | Hemoglobin [Mass/volume] in Blood | Hemoglobin |

**Tại sao cần LOINC:**
- Lab machines từ vendors khác nhau gọi tên test khác nhau
- LOINC cung cấp unique identifier
- Giúp standardize và exchange lab results

**Ở VN:**
- Ít được sử dụng
- Lab thường dùng tên riêng (không chuẩn hóa)
- Nếu tích hợp international labs → cần LOINC

---

### RxNorm (Medication Terminology)

**RxNorm là gì?**
- Chuẩn mã hóa thuốc của US National Library of Medicine
- Normalized names cho medications

**Ví dụ RxNorm:**
- Amoxicillin 500 MG Oral Capsule
- Metformin 500 MG Oral Tablet

**Ở VN:**
- Không sử dụng RxNorm
- Dùng **Danh mục thuốc quốc gia** của Bộ Y tế
- Mỗi thuốc có mã riêng

---

## 3. Medical Imaging Standards

### DICOM (Digital Imaging and Communications in Medicine)

**DICOM là gì?**
- Chuẩn quốc tế cho medical imaging
- Định nghĩa format lưu trữ và truyền tải hình ảnh y khoa
- Được sử dụng cho: X-ray, CT, MRI, Ultrasound...

**DICOM format:**
- File extension: `.dcm`
- Chứa cả hình ảnh + metadata (patient info, study info...)

**DICOM metadata ví dụ:**
```
(0010,0010) Patient's Name: NGUYEN^VAN^A
(0010,0020) Patient ID: 123456
(0008,0020) Study Date: 20231119
(0008,0060) Modality: CT
(0020,000D) Study Instance UID: 1.2.840.113619.2.1.1.1
```

**DICOM Services:**
- **C-STORE**: Lưu ảnh lên PACS
- **C-FIND**: Tìm kiếm ảnh
- **C-MOVE**: Truy xuất ảnh
- **C-GET**: Download ảnh

**⚠️ For dev:**
- Nếu làm PACS/RIS → phải hiểu DICOM
- Dùng libraries: dcm4che (Java), pydicom (Python), cornerstone.js (Web viewer)
- DICOM images rất nặng (MRI có thể 100+ MB/study)

---

## 4. Vietnam-Specific Standards

### Danh mục Quốc gia (National Catalogs)

Bộ Y tế Việt Nam có các danh mục chuẩn:

#### 1. Danh mục Bệnh (ICD-10 Vietnamese version)
- ICD-10 được dịch và điều chỉnh cho VN
- Bắt buộc cho BHYT

#### 2. Danh mục Thuốc Quốc gia
- Tất cả thuốc lưu hành tại VN
- Mỗi thuốc có:
  - Mã thuốc
  - Tên hoạt chất (generic name)
  - Tên biệt dược (brand name)
  - Hàm lượng
  - Đơn vị
  - Nhà sản xuất

#### 3. Danh mục Dịch vụ Kỹ thuật (Service Catalog)
- Tất cả dịch vụ y tế (xét nghiệm, thủ thuật, phẫu thuật...)
- Có mã và giá

**Ví dụ:**
- Mã: 09.1840 - Công thức máu (CBC) - Giá: 50,000 VND
- Mã: 09.1870 - X-quang phổi - Giá: 80,000 VND

#### 4. Danh mục CSYT (Medical Facilities)
- Mã số cơ sở y tế
- Dùng để BHYT biết bệnh nhân khám ở đâu

### Yêu cầu từ BHXH (Vietnam Social Security)

**Định dạng XML cho BHYT:**
- BHXH có format XML riêng để hospitals submit claims
- XML schemas do BHXH cung cấp
- Bao gồm: patient info, diagnosis (ICD-10), services, medications, costs

**XML BHYT gồm các phần:**
1. **XML1**: Thông tin hành chính (patient demographics, insurance info)
2. **XML2**: Chi tiết chi phí (itemized billing)
3. **XML3**: Thuốc
4. **XML4**: Dịch vụ kỹ thuật
5. **XML5**: Máu và chế phẩm máu

**⚠️ Critical for VN clinics:**
- Nếu làm BHYT → phải generate đúng XML format
- Sai format → reject → mất tiền
- BHXH update format thường xuyên → cần maintain

---

## 5. Data Types & Formats

### Patient Identifiers

**Các loại ID trong healthcare:**
- **MRN (Medical Record Number)**: ID bệnh án trong 1 bệnh viện
- **National ID**: CMND/CCCD (VN)
- **Insurance ID**: Số thẻ BHYT (VN)
- **Passport Number**: Cho bệnh nhân nước ngoài

**Best practice:**
- Dùng composite identifier (nhiều loại ID)
- Có algorithm check duplicate patients (patient matching)

### Date/Time Formats

**ISO 8601 (Recommended):**
- Date: YYYY-MM-DD (2023-11-19)
- DateTime: YYYY-MM-DDThh:mm:ss (2023-11-19T14:30:00)
- With timezone: 2023-11-19T14:30:00+07:00 (VN timezone)

**HL7 v2 format:**
- YYYYMMDD (20231119)
- YYYYMMDDHHmmss (20231119143000)

**⚠️ Important:**
- Always store in UTC, display in local timezone
- Vietnam timezone: +07:00 (ICT - Indochina Time)

### Name Formats

**Vietnamese names:**
- Format: Họ + Tên đệm + Tên
- Ví dụ: Nguyễn Văn An
  - Family name: Nguyễn
  - Middle name: Văn
  - Given name: An

**Database design:**
```sql
CREATE TABLE patients (
  id INT PRIMARY KEY,
  family_name VARCHAR(100),  -- Họ
  middle_name VARCHAR(100),  -- Tên đệm
  given_name VARCHAR(100),   -- Tên
  full_name VARCHAR(255)     -- Họ và tên đầy đủ (for search)
);
```

**Diacritics (Dấu tiếng Việt):**
- Store with diacritics: Nguyễn
- Also index without diacritics for search: Nguyen
- Use full-text search với collation support Vietnamese

---

## 6. Security & Privacy Standards

### HIPAA (US Standard - For Reference)

**HIPAA là gì?**
- Health Insurance Portability and Accountability Act (US law)
- Bảo vệ PHI (Protected Health Information)

**Key HIPAA rules:**
1. **Privacy Rule**: Who can access PHI
2. **Security Rule**: How to protect ePHI (electronic PHI)
3. **Breach Notification Rule**: Report security breaches

**HIPAA requirements:**
- Encryption at rest and in transit
- Access controls (role-based)
- Audit logs (who accessed what, when)
- Business Associate Agreements (BAAs)
- Automatic logout after inactivity
- Data backup and disaster recovery

### GDPR (EU Standard - For Reference)

**GDPR key concepts:**
- Right to access (patient can request their data)
- Right to erasure ("right to be forgotten")
- Data portability (export data in standard format)
- Consent management (explicit consent required)

### Vietnam Data Privacy

**Luật An toàn thông tin mạng 2018:**
- Bảo vệ dữ liệu cá nhân
- Chưa có quy định cụ thể về healthcare data như HIPAA

**Best practices cho VN:**
- Encrypt patient data
- Role-based access control
- Audit logs
- Backup regularly
- Patient consent cho việc sử dụng dữ liệu

---

## 7. Practical Implementation Guide

### Choosing Standards for Your Project

**For a clinic in Vietnam (your case):**

| Component | Standard to use | Priority |
|-----------|----------------|----------|
| Diagnosis coding | ICD-10 (Vietnamese version) | **MUST** (for BHYT) |
| Lab results | Custom codes + LOINC (optional) | Custom for now |
| Medications | Danh mục thuốc Quốc gia | **MUST** |
| Services | Danh mục dịch vụ Quốc gia | **MUST** |
| Messaging | HL7 v2 (if integrating with other systems) | Nice-to-have |
| API | RESTful JSON (FHIR-like structure) | Recommended |
| Medical imaging | DICOM (if doing radiology) | If applicable |
| BHYT claims | BHXH XML format | **MUST** (if accepting BHYT) |

### Database Design Tips

**Use lookup tables for standards:**
```sql
-- ICD-10 codes
CREATE TABLE icd10_codes (
  code VARCHAR(10) PRIMARY KEY,
  name_en VARCHAR(500),
  name_vi VARCHAR(500),
  category VARCHAR(50),
  chapter VARCHAR(100)
);

-- Medications
CREATE TABLE medications (
  code VARCHAR(20) PRIMARY KEY,
  generic_name VARCHAR(255),
  brand_name VARCHAR(255),
  dosage_form VARCHAR(50),  -- tablet, capsule, syrup...
  strength VARCHAR(50),      -- 500mg, 10ml...
  unit VARCHAR(20)
);

-- Services
CREATE TABLE services (
  code VARCHAR(20) PRIMARY KEY,
  name_vi VARCHAR(255),
  category VARCHAR(100),
  base_price DECIMAL(10,2),
  bhyt_price DECIMAL(10,2)
);
```

### API Design Best Practices

**RESTful + FHIR-inspired:**
```
GET    /api/patients              # List patients
GET    /api/patients/:id          # Get patient
POST   /api/patients              # Create patient
PUT    /api/patients/:id          # Update patient

GET    /api/patients/:id/encounters       # Patient's visits
GET    /api/patients/:id/observations     # Lab results, vitals
GET    /api/patients/:id/medications      # Medications
```

**Response format (JSON):**
```json
{
  "resourceType": "Patient",
  "id": "123456",
  "identifier": [
    {
      "type": "MRN",
      "value": "P000123456"
    },
    {
      "type": "NationalID",
      "value": "001234567890"
    }
  ],
  "name": {
    "family": "Nguyen",
    "middle": "Van",
    "given": "A",
    "full": "Nguyen Van A"
  },
  "gender": "male",
  "birthDate": "1985-03-15",
  "telecom": [
    {
      "system": "phone",
      "value": "0909123456"
    }
  ]
}
```

---

## 8. Common Pitfalls & How to Avoid

### ❌ Pitfall 1: Hard-coding medical terms
**Problem**: `if (diagnosis == "Cao huyết áp")` → không scalable, dễ lỗi
**Solution**: Dùng ICD-10 codes: `if (diagnosis_code == "I10")`

### ❌ Pitfall 2: Không validate codes
**Problem**: Nhập sai mã ICD-10 → BHYT reject
**Solution**: Validate against danh mục, có autocomplete

### ❌ Pitfall 3: Custom format không có document
**Problem**: Mỗi developer làm khác nhau
**Solution**: Define API schema rõ ràng (OpenAPI/Swagger)

### ❌ Pitfall 4: Không versioning API
**Problem**: Update API → break clients
**Solution**: API versioning (`/api/v1/patients`)

### ❌ Pitfall 5: Lưu tên không có dấu
**Problem**: "Nguyen Van A" → không tìm được "Nguyễn Văn A"
**Solution**: Lưu cả có dấu + không dấu, search cả 2

---

## Summary

**Key takeaways:**

1. **Messaging**: HL7 v2 (current), FHIR (future)
2. **Diagnosis**: ICD-10 (mandatory for VN)
3. **Lab results**: LOINC (international), custom (VN)
4. **Medications**: RxNorm (US), Danh mục Quốc gia (VN)
5. **Imaging**: DICOM (if doing radiology)
6. **Vietnam-specific**: BHXH XML format (critical for BHYT)
7. **Security**: Encryption, access control, audit logs

**For your clinic project:**
- Start with: ICD-10, Danh mục thuốc, Danh mục dịch vụ
- Design RESTful API with FHIR-like structure
- Prepare for BHYT XML format
- Keep standards in lookup tables
- Plan for FHIR migration in future

**Next step**: Part 2 - Deep dive vào từng workflow cụ thể.
