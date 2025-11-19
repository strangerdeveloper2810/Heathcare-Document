# Vietnam Compliance & Regulations
# Tuân thủ Quy định Việt Nam

## Overview (Tổng quan)

Hệ thống HIS tại Việt Nam phải tuân thủ các quy định của:
- Bộ Y tế (Ministry of Health - MOH)
- Bảo hiểm Y tế (Social Health Insurance)
- Luật Khám chữa bệnh
- Quy định về bảo mật thông tin bệnh nhân

**Tại sao quan trọng?**
- Bắt buộc pháp lý
- Được thanh toán BHYT
- Tránh phạt và rủi ro pháp lý
- Đảm bảo chất lượng chăm sóc

---

## 1. Medical Records Regulations (Quy định Hồ sơ Bệnh án)

### 1.1 Required Information

**Theo Thông tư 46/2018/TT-BYT:**

**Thông tin bắt buộc trong hồ sơ bệnh án:**
- Thông tin nhận dạng bệnh nhân
- Lý do vào viện/khám
- Bệnh sử, tiền sử
- Khám lâm sàng
- Chẩn đoán
- Phương án điều trị
- Diễn biến bệnh
- Kết quả điều trị
- Chữ ký bác sĩ

### 1.2 Medical Record Format

**Cấu trúc hồ sơ bệnh án:**

```
1. PHẦN HÀNH CHÍNH
   - Thông tin bệnh nhân
   - Thông tin BHYT
   - Ngày giờ vào viện/khám

2. PHẦN LÂM SÀNG
   - Lý do khám
   - Bệnh sử
   - Khám lâm sàng
   - Chẩn đoán (ICD-10)
   - Phương án điều trị

3. PHẦN CẬN LÂM SÀNG
   - Xét nghiệm
   - Chẩn đoán hình ảnh
   - Kết quả

4. PHẦN ĐIỀU TRỊ
   - Đơn thuốc
   - Y lệnh
   - Diễn biến

5. PHẦN KẾT THÚC
   - Kết quả điều trị
   - Hướng dẫn xuất viện/tái khám
   - Chữ ký bác sĩ
```

### 1.3 Retention Requirements

**Thời gian lưu trữ:**
- **Hồ sơ ngoại trú**: Tối thiểu 15 năm
- **Hồ sơ nội trú**: Tối thiểu 20 năm
- **Hồ sơ trẻ em**: Đến khi 18 tuổi + 15 năm

**Implementation:**
- Không được xóa hồ sơ (chỉ archive)
- Backup định kỳ
- Disaster recovery plan

---

## 2. BHYT (Social Health Insurance) Requirements

### 2.1 BHYT Card Validation

**Required checks:**
- Card number format (13 digits)
- Expiration date
- Coverage status (active/inactive)
- Patient information match

**Validation flow:**
```
Bệnh nhân cung cấp thẻ BHYT
    ↓
System validate:
1. Format: 13 chữ số
2. Expiration date: Còn hiệu lực?
3. Status: Active?
4. Patient info: Khớp với thông tin?
    ↓
  Valid:
  → Apply BHYT coverage
  → Calculate co-payment
  → Generate BHYT claim
  
  Invalid:
  → Show error
  → Request valid card
  → Process as self-pay
```

### 2.2 BHYT Coverage Rules

**Coverage levels:**
- **100%**: Trẻ em < 6 tuổi, người nghèo, người có công
- **95%**: Người hưu trí, người tàn tật
- **80%**: Người lao động, người tham gia BHYT
- **Co-payment**: 20% (tùy đối tượng)

**Covered services:**
- Khám bệnh
- Thuốc (theo danh mục)
- Xét nghiệm (theo danh mục)
- Chẩn đoán hình ảnh (theo danh mục)
- Điều trị nội trú

**Not covered:**
- Dịch vụ thẩm mỹ
- Khám sức khỏe định kỳ (một số trường hợp)
- Thuốc ngoài danh mục

### 2.3 BHYT Claim Submission

**Required information:**
- Patient information
- BHYT card number
- Services provided (with codes)
- Diagnosis (ICD-10)
- Dates of service
- Amounts

**Claim format:**
- XML format (theo quy định BHYT)
- Digital signature
- Submit to BHYT portal

**Claim workflow:**
```
Complete service
    ↓
Generate BHYT claim
    ↓
Validate claim data
    ↓
Format as XML
    ↓
Digital signature
    ↓
Submit to BHYT portal
    ↓
Track status:
- Submitted
- Approved
- Rejected (with reason)
- Paid
```

---

## 3. ICD-10 Coding Requirements

### 3.1 Mandatory ICD-10 Usage

**Quy định:**
- Tất cả chẩn đoán phải có mã ICD-10
- Bắt buộc từ 2018
- Sử dụng phiên bản mới nhất

**Common ICD-10 codes:**
```
I10 - Essential (primary) hypertension
E11 - Type 2 diabetes mellitus
J11 - Influenza with other respiratory manifestations
J40 - Bronchitis, not specified as acute or chronic
R51 - Headache
K29 - Gastritis and duodenitis
M25 - Other joint disorders, not elsewhere classified
```

### 3.2 ICD-10 Implementation

**System requirements:**
- ICD-10 catalog (updated)
- Search functionality
- Auto-suggest
- Validation

**Coding workflow:**
```
Doctor enters diagnosis
    ↓
Search ICD-10 code
    ↓
Select appropriate code
    ↓
System validates:
- Code exists?
- Code active?
- Appropriate for patient?
    ↓
Save diagnosis with ICD-10 code
```

---

## 4. Prescription Regulations

### 4.1 Prescription Requirements

**Required information:**
- Patient information
- Date
- Doctor information (name, license)
- Medications (name, dosage, quantity)
- Instructions (SIG)
- Doctor signature

**Prescription format:**
```
ĐƠN THUỐC
─────────────────────────────────────
Họ tên BN: Nguyễn Văn A
Tuổi: 43
Ngày: 19/11/2023

STT │ Tên thuốc        │ Hàm lượng │ SL │ Cách dùng
────┼──────────────────┼───────────┼────┼─────────────
1   │ Amlodipine       │ 5mg       │ 30 │ 1 viên/ngày
2   │ Losartan         │ 50mg      │ 30 │ 1 viên/ngày

Chữ ký bác sĩ: [Signature]
Tên BS: Dr. Nguyễn Văn B
Số chứng chỉ: 12345
```

### 4.2 Controlled Substances

**Quy định thuốc kiểm soát:**
- Morphine, Opioids: Prescription đặc biệt
- Antibiotics: Cần chẩn đoán rõ ràng
- Psychotropic drugs: Quy định nghiêm ngặt

**System requirements:**
- Flag controlled substances
- Require additional documentation
- Track dispensing
- Report to authorities (nếu cần)

---

## 5. Lab Test Regulations

### 5.1 Lab Test Requirements

**Required information:**
- Patient information
- Ordering doctor
- Test name (with code)
- Date/time ordered
- Date/time performed
- Results
- Reference ranges
- Performed by (technician)

### 5.2 Critical Values

**Quy định:**
- Critical values phải báo ngay cho bác sĩ
- Document notification
- Follow-up required

**Critical value workflow:**
```
Lab result entered
    ↓
System checks: Critical value?
    ↓
  Yes:
  → Alert immediately
  → Notify ordering doctor
  → Document notification
  → Require acknowledgment
  
  No:
  → Normal workflow
```

---

## 6. Data Privacy & Security

### 6.1 Patient Data Protection

**Quy định:**
- Bảo mật thông tin bệnh nhân
- Chỉ người có thẩm quyền mới được truy cập
- Audit trail bắt buộc

**Implementation:**
- Role-based access control
- Encryption (at rest & in transit)
- Audit logs
- Data backup

### 6.2 Consent Management

**Required consents:**
- Consent for treatment
- Consent for data sharing (nếu cần)
- Consent for research (nếu có)

**System requirements:**
- Store consent records
- Track consent status
- Expiration management

---

## 7. License & Certification

### 7.1 Medical License Requirements

**Bác sĩ phải có:**
- Chứng chỉ hành nghề (Medical License)
- Chứng chỉ chuyên khoa (Specialty Certificate)
- Cập nhật định kỳ

**System requirements:**
- Store license information
- Track expiration dates
- Alert before expiration
- Prevent prescribing if expired

### 7.2 Facility License

**Phòng khám phải có:**
- Giấy phép hoạt động
- Giấy phép hành nghề
- Đăng ký với Sở Y tế

---

## 8. Reporting Requirements

### 8.1 Mandatory Reports

**Báo cáo bắt buộc:**
- Báo cáo hoạt động (hàng tháng/quý)
- Báo cáo dịch bệnh (nếu có)
- Báo cáo tai biến y khoa
- Báo cáo BHYT claims

**Report submission:**
- Format theo quy định
- Submit to Sở Y tế
- Deadline compliance

### 8.2 Incident Reporting

**Tai biến y khoa:**
- Phải báo cáo ngay
- Document đầy đủ
- Follow-up investigation

**System requirements:**
- Incident reporting form
- Alert system
- Tracking & follow-up

---

## 9. Quality Standards

### 9.1 Clinical Quality Indicators

**Chỉ số chất lượng:**
- Patient satisfaction
- Treatment outcomes
- Infection rates
- Medication errors
- Readmission rates

**System requirements:**
- Track quality indicators
- Generate quality reports
- Benchmark comparison

### 9.2 Continuous Improvement

**Quy trình:**
- Regular audits
- Quality reviews
- Corrective actions
- Documentation

---

## 10. Implementation Checklist

### 10.1 Medical Records

- [ ] Hồ sơ bệnh án đầy đủ thông tin bắt buộc
- [ ] ICD-10 coding cho tất cả chẩn đoán
- [ ] Chữ ký điện tử bác sĩ
- [ ] Lưu trữ tối thiểu 15-20 năm
- [ ] Backup định kỳ

### 10.2 BHYT

- [ ] Validate BHYT card
- [ ] Calculate coverage correctly
- [ ] Generate BHYT claims (XML format)
- [ ] Submit claims to BHYT portal
- [ ] Track claim status
- [ ] Handle rejections

### 10.3 Prescriptions

- [ ] Prescription format đúng quy định
- [ ] Doctor license validation
- [ ] Controlled substances tracking
- [ ] E-prescribing compliance

### 10.4 Lab Tests

- [ ] Lab test codes (LOINC)
- [ ] Critical value alerts
- [ ] Result documentation
- [ ] Technician tracking

### 10.5 Security & Privacy

- [ ] Role-based access control
- [ ] Audit logs
- [ ] Data encryption
- [ ] Consent management
- [ ] Data backup & recovery

### 10.6 Licenses

- [ ] Doctor license tracking
- [ ] License expiration alerts
- [ ] Facility license management

### 10.7 Reporting

- [ ] Mandatory reports generation
- [ ] Report submission workflow
- [ ] Incident reporting system

---

## 11. Common Compliance Issues

### Issue 1: Missing ICD-10 Codes
**Problem:** Chẩn đoán không có mã ICD-10
**Solution:** Require ICD-10 code before saving diagnosis

### Issue 2: Invalid BHYT Claims
**Problem:** BHYT claims bị reject
**Solution:** Validate claim data before submission, check format

### Issue 3: Expired Licenses
**Problem:** Bác sĩ kê đơn với license hết hạn
**Solution:** Check license expiration, prevent prescribing if expired

### Issue 4: Missing Signatures
**Problem:** Hồ sơ thiếu chữ ký
**Solution:** Require digital signature before completing record

### Issue 5: Data Breach
**Problem:** Thông tin bệnh nhân bị lộ
**Solution:** Access control, encryption, audit logs

---

## Summary

**Key takeaways:**

1. **Medical records** phải đầy đủ và lưu trữ lâu dài
2. **BHYT compliance** là bắt buộc cho thanh toán
3. **ICD-10 coding** bắt buộc cho tất cả chẩn đoán
4. **Prescription regulations** nghiêm ngặt
5. **Data privacy** phải được bảo vệ
6. **License management** quan trọng
7. **Reporting** bắt buộc định kỳ

**For implementation:**
- Start với BHYT validation và claims
- Implement ICD-10 coding
- Medical record format compliance
- Security & audit logging
- License tracking

**Next**: [10-system-architecture-overview.md](10-system-architecture-overview.md)

