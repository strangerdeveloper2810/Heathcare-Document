# Healthcare Domain Knowledge Base

# Kho kiến thức lĩnh vực Y tế

Tài liệu toàn diện về healthcare domain dành cho PM và Dev team xây dựng hệ thống EMR/HIS cho phòng khám tại Việt Nam.

---

## 📋 Mục đích (Purpose)

Bộ tài liệu này được tạo ra để:

- **Empowering Dev team**: Giúp developers hiểu sâu nghiệp vụ healthcare để tự làm BA
- **Knowledge transfer**: Onboard dev mới nhanh chóng
- **Foundation để rebuild**: Cơ sở để refactor/thiết kế lại hệ thống từ đầu
- **Long-term reference**: Tài liệu tham khảo lâu dài

**Target audience**: PM (non-technical, non-domain) + Dev team

**Scope**: EMR (Electronic Medical Records) + Patient Web App cho phòng khám (primary care, outpatient focus)

---

## 🗂️ Cấu trúc tài liệu (Document Structure)

### Part 1: Healthcare Fundamentals (Kiến thức nền tảng)

Các khái niệm cơ bản cần nắm trước khi bắt đầu.

#### 📘 [01. Medical Terminology Basics](docs/healthcare-fundamentals/01-medical-terminology.md)

- Cấu trúc thuật ngữ y học (prefix, root, suffix)
- Thuật ngữ theo 6 hệ thống cơ thể
- Vital signs & common abbreviations
- Patient types & visit types
- Medication terminology
- **Thời gian đọc**: 30-40 phút

#### 📘 [02. Healthcare System Overview](docs/healthcare-fundamentals/02-healthcare-system-overview.md)

- Levels of Care (Primary → Quaternary)
- Healthcare system ở VN (phân tuyến, public vs private)
- So sánh international systems (US, UK, Singapore)
- Key stakeholders (clinical + administrative + external)
- Healthcare workflows (outpatient vs inpatient)
- Data flow và integration points
- Payment models & Regulations overview
- **Thời gian đọc**: 45-60 phút

#### 📘 [03. Healthcare Data Standards](docs/healthcare-fundamentals/03-healthcare-data-standards.md)

- Messaging standards: HL7 v2, FHIR (với examples)
- Clinical terminology: ICD-10, SNOMED CT, LOINC, RxNorm
- Medical imaging: DICOM
- **Vietnam-specific**: BHYT XML format, Danh mục Quốc gia
- Security standards: HIPAA, GDPR overview
- Practical implementation guide cho VN clinics
- **Thời gian đọc**: 40-50 phút

**📌 Recommended reading order**: 01 → 02 → 03

---

### Part 2: Workflow Deep-Dives (Chi tiết quy trình nghiệp vụ)

Deep dive vào từng workflow cụ thể của phòng khám, từ đăng ký đến thanh toán.

#### 📗 [01. Patient Registration & Admission](docs/workflow-deep-dives/01-patient-registration.md)

**Nội dung:**

- 3 types of registration (new, returning, appointment-based)
- Complete data model (SQL schemas: patients, insurance, allergies...)
- UI design examples
- Business logic: MRN generation, BHYT validation, duplicate detection
- Integration points
- Common issues & solutions

**Key learnings:**

- Registration là foundation - sai ở đây → sai cả workflow sau
- Validate thoroughly: BHYT card, National ID, duplicates
- Duplicate patient detection algorithms
- Vietnam-specific: BHYT validation, mã CSYT

**Thời gian đọc**: 35-45 phút

---

#### 📗 [02. Outpatient Consultation Workflow](docs/workflow-deep-dives/02-outpatient-consultation.md)

**Nội dung:**

- Complete consultation flow (10 steps: Triage → Documentation)
- **SOAP Note format** (Subjective, Objective, Assessment, Plan)
- Data model: encounters, vital signs, clinical notes, diagnoses
- UI/UX design cho doctor consultation screen
- Clinical Decision Support (CDS): Drug interactions, allergy alerts
- Templates & shortcuts để optimize doctor's time
- Integration với lab orders, prescriptions, referrals

**Key learnings:**

- SOAP format là standard cho clinical documentation
- Optimize for speed: Templates, keyboard shortcuts, smart defaults
- CDS alerts (drug interactions, allergies) are CRITICAL for patient safety
- ICD-10 mandatory cho BHYT claims
- Mobile-friendly (bác sĩ dùng tablet)

**Thời gian đọc**: 40-50 phút

---

#### 📗 [03. Diagnostics & Lab Tests Workflow](docs/workflow-deep-dives/03-diagnostics-lab-tests.md)

**Nội dung:**

- Complete lab workflow (9 steps: Order → Result → Review)
- Common lab tests & panels: CBC, BMP, LFT, Lipid, HbA1c, Urinalysis
- Data model: lab_orders, lab_order_items, lab_results, lab_test_catalog
- Normal ranges & interpretation
- HL7 messaging cho LIS integration
- **Critical value alerts** (must notify doctor immediately)
- Imaging orders (X-ray, Ultrasound, CT, MRI)
- Quality control & result amendments

**Key learnings:**

- Critical values cần alert bác sĩ NGAY LẬP TỨC (patient safety)
- HL7 v2 messaging cho auto-import results từ lab machines
- Result display phải có flags (High, Low, Critical)
- Trending results (so với previous tests)

**Thời gian đọc**: 30-40 phút

---

#### 📗 [04. Prescription & Pharmacy Workflow](docs/workflow-deep-dives/04-prescription-pharmacy.md)

**Nội dung:**

- Complete prescription flow (9 steps: Select → Dispense → Counsel)
- Prescription notation (SIG codes): Route, Frequency, Duration
- Data model: prescriptions, prescription_items, medications catalog
- **Clinical Decision Support**:
  - Drug-drug interaction checking
  - Drug-allergy checking
  - Contraindication checking
  - Duplicate therapy checking
- E-prescribing format (XML)
- Pharmacy dispensing workflow
- Medication label generation
- Inventory management (stock control, reorder alerts)
- Medication adherence tracking

**Key learnings:**

- Safety checks (allergies, interactions) are MANDATORY
- SIG codes: PO, BID, TID, QD, PRN, AC, PC...
- E-prescribing > paper prescriptions (eliminate handwriting errors)
- Pharmacist review là final safety checkpoint
- Inventory management: lot tracking, expiry dates

**Thời gian đọc**: 35-45 phút

---

#### 📗 [05. Billing & Insurance Workflow](docs/workflow-deep-dives/05-billing-insurance.md)

**Nội dung:**

- Complete billing flow (10 steps: Service → Claim → Payment)
- Data model: bills, bill_items, payments, service_price_list
- **BHYT (Vietnam Social Insurance) specifics**:
  - Coverage levels (80%, 95%, 100%)
  - Card validation (15-digit format)
  - **BHYT Claim XML generation** (XML1, XML2, XML3...)
  - Co-payment calculation
- Billing calculations với BHYT
- UI design: Bill summary, payment screen, receipt
- Revenue cycle management: Collection rate, Days in AR, Denial management
- Discounts & packages
- Payment gateway integration (cards)
- Reports: Daily revenue, service revenue, aging reports

**Key learnings:**

- BHYT XML format là CRITICAL cho VN clinics (sai format → không được thanh toán)
- Coverage calculation: 80%, 95%, 100% based on card prefix
- Co-payment (cùng chi trả) for non-exempt patients
- Claim denial management workflow
- Revenue metrics tracking

**Thời gian đọc**: 40-50 phút

---

#### 📗 [06. Appointment Scheduling Workflow](docs/workflow-deep-dives/06-appointment-scheduling.md)

**Nội dung:**

- Types of appointments (New Visit, Follow-up, Consultation, Annual Checkup)
- Booking channels (Walk-in, Phone, Online, In-person)
- Complete appointment scheduling flow (9 steps)
- Data model: appointments, doctor_schedules, appointment slots
- Availability checking logic
- Online booking flow & Phone booking flow
- Appointment status management (Scheduled → Completed/Cancelled/No-show)
- Reminder system (24h before, 2h before)
- Cancellation & rescheduling workflows
- Queue management (Appointment vs Walk-in)
- No-show management & policies

**Key learnings:**

- Appointment scheduling giúp tối ưu thời gian và cải thiện UX
- Multiple booking channels cần được hỗ trợ
- Reminder system giảm no-show rate
- Queue management cân bằng appointment vs walk-in
- Integration với registration và consultation

**Thời gian đọc**: 35-45 phút

---

#### 📗 [07. User Management & Security](docs/workflow-deep-dives/07-user-management-security.md)

**Nội dung:**

- User roles & responsibilities (Clinical, Administrative, Management)
- Role-Based Access Control (RBAC) model
- Permission matrix (View, Create, Modify, Delete)
- Data model: users, roles, permissions, role_permissions, audit_logs
- Authentication & authorization flow
- Permission examples (View Patient, Create Prescription)
- UI/UX design (Login, User Management, Role Permission Management)
- Security best practices (Password policy, Session management, Data encryption)
- Audit trail (What to log, Audit log examples)
- Common security scenarios (Doctor leaves, Temporary access, Emergency access)
- Vietnam-specific requirements (License management, Data retention)

**Key learnings:**

- RBAC là foundation cho security
- Role-based permissions đơn giản và hiệu quả
- Audit logs bắt buộc cho compliance
- Session management quan trọng cho security
- Scope restrictions (ALL, OWN, ASSIGNED) linh hoạt

**Thời gian đọc**: 40-50 phút

---

#### 📗 [08. Reporting & Analytics](docs/workflow-deep-dives/08-reporting-analytics.md)

**Nội dung:**

- Types of reports (Clinical, Operational, Financial, Administrative)
- Clinical reports (Patient Summary, Diagnosis Statistics, Treatment Outcomes)
- Operational reports (Daily Activity, Appointment Statistics, Doctor Productivity)
- Financial reports (Daily/Monthly Revenue, BHYT Claims, Outstanding Bills)
- Report generation flow (Real-time vs Scheduled)
- Data model: report_definitions, report_executions, report_schedules
- Analytics & dashboards (KPIs, Visual metrics)
- Data export (PDF, Excel, CSV, JSON)
- Report access control (Role-based)
- Performance considerations (Optimization, Pre-calculated summary tables)

**Key learnings:**

- Multiple report types phục vụ different needs
- Real-time vs Scheduled reports tùy nhu cầu
- Role-based access đảm bảo security
- Performance optimization quan trọng với large dataset
- Dashboards cung cấp quick insights

**Thời gian đọc**: 35-45 phút

---

#### 📗 [09. Vietnam Compliance & Regulations](docs/workflow-deep-dives/09-vietnam-compliance-regulations.md)

**Nội dung:**

- Medical Records Regulations (Required information, Format, Retention)
- BHYT Requirements (Card validation, Coverage rules, Claim submission)
- ICD-10 Coding Requirements (Mandatory usage, Implementation)
- Prescription Regulations (Required information, Controlled substances)
- Lab Test Regulations (Required information, Critical values)
- Data Privacy & Security (Patient data protection, Consent management)
- License & Certification (Medical license, Facility license)
- Reporting Requirements (Mandatory reports, Incident reporting)
- Quality Standards (Clinical quality indicators, Continuous improvement)
- Implementation checklist

**Key learnings:**

- Medical records phải đầy đủ và lưu trữ lâu dài (15-20 năm)
- BHYT compliance là bắt buộc cho thanh toán
- ICD-10 coding bắt buộc cho tất cả chẩn đoán
- Prescription regulations nghiêm ngặt
- Data privacy phải được bảo vệ
- License management quan trọng

**Thời gian đọc**: 40-50 phút

---

#### 📗 [10. System Architecture Overview](docs/workflow-deep-dives/10-system-architecture-overview.md)

**Nội dung:**

- System Components (Core modules, Module descriptions)
- Data Flow (Complete patient journey, Data dependencies)
- Module Interactions (Registration ↔ Appointment, Consultation ↔ Lab, etc.)
- Integration Points (BHYT, LIS, PACS)
- Database Overview (Core tables, Key relationships)
- User Roles & Access (Role-based module access, Data access scope)
- Workflow Integration (Registration → Appointment → Consultation)
- System Boundaries (What's inside/outside HIS)
- Data Consistency (Master data, Transactional data)
- Scalability Considerations (Horizontal/Vertical scaling, Caching)
- Security Architecture (Authentication, Authorization, Data protection)
- Deployment Architecture (Typical deployment, Integration layer)

**Key learnings:**

- Modular architecture: Mỗi module độc lập nhưng tích hợp chặt chẽ
- Data flow rõ ràng từ registration → consultation → billing
- Integration points qua standard protocols (HL7, DICOM, XML)
- Role-based access: Mỗi role có quyền truy cập phù hợp
- Master data: Single source of truth
- Scalability: Stateless modules có thể scale horizontal

**Thời gian đọc**: 30-40 phút

---

## 🎯 Quick Start Guide

### Cho PM/Team Lead:

1. Đọc [Healthcare System Overview](docs/healthcare-fundamentals/02-healthcare-system-overview.md) trước → big picture
2. Đọc [Medical Terminology](docs/healthcare-fundamentals/01-medical-terminology.md) → nắm thuật ngữ cơ bản
3. Đọc workflow relevant đến features đang làm

### Cho Developers (New to healthcare):

**Week 1 - Fundamentals:**

- Day 1-2: [Medical Terminology](docs/healthcare-fundamentals/01-medical-terminology.md)
- Day 3-4: [Healthcare System Overview](docs/healthcare-fundamentals/02-healthcare-system-overview.md)
- Day 5: [Healthcare Data Standards](docs/healthcare-fundamentals/03-healthcare-data-standards.md)

**Week 2 - Core Workflows:**

- Day 1: [Patient Registration](docs/workflow-deep-dives/01-patient-registration.md)
- Day 2: [Appointment Scheduling](docs/workflow-deep-dives/06-appointment-scheduling.md)
- Day 3: [Outpatient Consultation](docs/workflow-deep-dives/02-outpatient-consultation.md)
- Day 4: [Diagnostics & Lab Tests](docs/workflow-deep-dives/03-diagnostics-lab-tests.md)
- Day 5: [Prescription & Pharmacy](docs/workflow-deep-dives/04-prescription-pharmacy.md)

**Week 3 - Supporting Workflows:**

- Day 1: [Billing & Insurance](docs/workflow-deep-dives/05-billing-insurance.md)
- Day 2: [User Management & Security](docs/workflow-deep-dives/07-user-management-security.md)
- Day 3: [Reporting & Analytics](docs/workflow-deep-dives/08-reporting-analytics.md)
- Day 4: [Vietnam Compliance & Regulations](docs/workflow-deep-dives/09-vietnam-compliance-regulations.md)
- Day 5: [System Architecture Overview](docs/workflow-deep-dives/10-system-architecture-overview.md)

**Week 3+**: Apply to actual development, refer back khi cần

### Cho Developers (Feature-specific):

- **Làm Registration module** → Đọc [01-patient-registration.md](docs/workflow-deep-dives/01-patient-registration.md)
- **Làm Appointment scheduling** → Đọc [06-appointment-scheduling.md](docs/workflow-deep-dives/06-appointment-scheduling.md)
- **Làm Doctor consultation** → Đọc [02-outpatient-consultation.md](docs/workflow-deep-dives/02-outpatient-consultation.md)
- **Làm Lab integration** → Đọc [03-diagnostics-lab-tests.md](docs/workflow-deep-dives/03-diagnostics-lab-tests.md) + [03-healthcare-data-standards.md](docs/healthcare-fundamentals/03-healthcare-data-standards.md) (HL7 section)
- **Làm Prescription** → Đọc [04-prescription-pharmacy.md](docs/workflow-deep-dives/04-prescription-pharmacy.md)
- **Làm Billing/BHYT** → Đọc [05-billing-insurance.md](docs/workflow-deep-dives/05-billing-insurance.md)
- **Làm User management/Security** → Đọc [07-user-management-security.md](docs/workflow-deep-dives/07-user-management-security.md)
- **Làm Reporting** → Đọc [08-reporting-analytics.md](docs/workflow-deep-dives/08-reporting-analytics.md)
- **Làm Compliance** → Đọc [09-vietnam-compliance-regulations.md](docs/workflow-deep-dives/09-vietnam-compliance-regulations.md)
- **Thiết kế Architecture** → Đọc [10-system-architecture-overview.md](docs/workflow-deep-dives/10-system-architecture-overview.md)

---

## 💡 Key Takeaways

### Healthcare-specific considerations:

1. **Patient Safety First**: Sai số liệu y tế có thể nguy hiểm tính mạng

   - Drug interactions, allergy alerts are MANDATORY
   - Critical value alerts for lab results
   - Validation everywhere

2. **Data Standards Matter**:

   - ICD-10 for diagnosis (required for BHYT)
   - HL7/FHIR for interoperability
   - BHYT XML format (Vietnam-specific, CRITICAL)

3. **Workflow Complexity**:

   - Healthcare có nhiều stakeholders (doctor, nurse, pharmacist, admin, patient...)
   - Integration points: EMR ↔ Lab ↔ Pharmacy ↔ Billing ↔ Patient Portal
   - Real-world workflow không tuyến tính (có loops, có exceptions)

4. **Vietnam-Specific**:

   - BHYT (Bảo hiểm y tế) là core feature (80%+ patients có BHYT)
   - BHYT XML format phải đúng 100% (sai → reject → mất tiền)
   - Danh mục Quốc gia (thuốc, dịch vụ, ICD-10 Vietnamese version)
   - Mã CSYT (medical facility code)

5. **UX/Performance**:
   - Doctors hate slow systems → optimize for speed
   - Minimize clicks → templates, shortcuts, smart defaults
   - Mobile-friendly (tablets are common)
   - Auto-save (never lose data)

---

## 🔧 Technical Implementation Priority

**Phase 1 - MVP (Core workflows):**

1. ✅ Patient Registration (new + returning)
2. ✅ Outpatient Consultation (basic SOAP notes)
3. ✅ Prescription (e-prescribing với CDS alerts)
4. ✅ Billing (basic fee-for-service)
5. ✅ Payment collection & receipts

**Phase 2 - BHYT Integration:**

1. ✅ BHYT card validation
2. ✅ BHYT coverage calculation
3. ✅ BHYT XML generation
4. ✅ Claim submission workflow
5. ✅ Claim tracking & denial management

**Phase 3 - Advanced Features:**

1. ✅ Appointment scheduling (online booking, reminders)
2. ✅ Lab orders & results (với HL7 integration)
3. ✅ Imaging orders
4. ✅ Inventory management (pharmacy)
5. ✅ Patient portal/app
6. ✅ Reports & analytics
7. ✅ User management & RBAC

**Phase 4 - Optimization:**

1. ✅ Performance optimization
2. ✅ Mobile app (doctor + patient)
3. ✅ AI/ML features (diagnosis suggestions, drug interaction predictions...)
4. ✅ Telemedicine

---

## 📚 Additional Resources

### For Learning:

- **MedlinePlus** (NIH): https://medlineplus.gov - Medical dictionary, health topics
- **ICD-10 Browser** (WHO): https://icd.who.int - Official ICD-10 codes
- **HL7 Standards**: https://www.hl7.org - Healthcare data standards
- **FHIR Documentation**: https://www.hl7.org/fhir - Modern healthcare API standard

### Vietnam-Specific:

- **Cổng thông tin Bộ Y Tế**: https://moh.gov.vn - Regulations, guidelines
- **BHXH Portal**: https://baohiemxahoi.gov.vn - Social insurance info
- **VssID App**: Mobile app để check BHYT online

### For Development:

- **FHIR Libraries**: HAPI FHIR (Java), fhir.js (JavaScript), fhirclient.py (Python)
- **HL7 Parsers**: hl7apy (Python), hl7parser (Java), simple-hl7 (Node.js)
- **DICOM Libraries**: dcm4che (Java), pydicom (Python), cornerstone.js (Web DICOM viewer)

---

## 🤝 Contributing & Feedback

Tài liệu này là living document - sẽ được cập nhật khi có:

- Thay đổi regulations (BHYT format changes, new laws...)
- Feedback từ team (phần nào chưa rõ, cần bổ sung...)
- Best practices mới từ real-world usage

**How to contribute:**

1. Tìm thấy lỗi/thiếu sót → tạo issue hoặc sửa trực tiếp
2. Có câu hỏi → ask PM/team lead
3. Học được best practice mới → document và share

---

## 📞 Contact

**PM**: Ethan Nguyễn
**Team**: Healthcare EMR Development Team
**Last Updated**: November 2025

---

## 📄 License

Internal use only - Confidential

---

**Happy Learning! 🎉**

Remember: Healthcare software development requires both technical skills AND domain knowledge. Take time to understand the "why" behind workflows, not just the "what". Patient safety depends on it.
