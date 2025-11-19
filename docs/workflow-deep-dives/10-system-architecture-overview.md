# System Architecture Overview
# Tổng quan Kiến trúc Hệ thống

## Overview (Tổng quan)

Document này mô tả kiến trúc tổng thể của hệ thống HIS, tập trung vào **nghiệp vụ** và **workflow**, không đi sâu vào technical implementation details.

**Mục đích:**
- Hiểu cách các module tương tác với nhau
- Nắm được luồng dữ liệu giữa các workflow
- Làm cơ sở cho việc thiết kế và phát triển

---

## 1. System Components (Các thành phần hệ thống)

### 1.1 Core Modules

```
┌─────────────────────────────────────────────────┐
│           HỆ THỐNG QUẢN LÝ PHÒNG KHÁM          │
└─────────────────────────────────────────────────┘

┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│ Registration │  │ Appointment │  │ Consultation │
│   Module     │  │   Module    │  │   Module     │
└──────────────┘  └──────────────┘  └──────────────┘
       │                 │                 │
       └─────────────────┼─────────────────┘
                         │
       ┌─────────────────┼─────────────────┐
       │                 │                 │
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│   Lab &     │  │ Prescription │  │   Billing    │
│ Diagnostics │  │   Module     │  │   Module     │
└──────────────┘  └──────────────┘  └──────────────┘
       │                 │                 │
       └─────────────────┼─────────────────┘
                         │
       ┌─────────────────┼─────────────────┐
       │                 │                 │
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│  Reporting  │  │ User &       │  │ Integration  │
│   Module    │  │ Security     │  │   Module     │
└──────────────┘  └──────────────┘  └──────────────┘
```

### 1.2 Module Descriptions

| Module | Chức năng chính | Workflows liên quan |
|--------|-----------------|---------------------|
| **Registration** | Đăng ký bệnh nhân | Patient Registration |
| **Appointment** | Đặt lịch hẹn | Appointment Scheduling |
| **Consultation** | Khám bệnh | Outpatient Consultation |
| **Lab & Diagnostics** | Xét nghiệm, chẩn đoán | Diagnostics & Lab Tests |
| **Prescription** | Kê đơn, cấp phát thuốc | Prescription & Pharmacy |
| **Billing** | Thanh toán, hóa đơn | Billing & Insurance |
| **Reporting** | Báo cáo, phân tích | Reporting & Analytics |
| **User & Security** | Quản lý người dùng, phân quyền | User Management & Security |
| **Integration** | Tích hợp với hệ thống bên ngoài | External Systems |

---

## 2. Data Flow (Luồng dữ liệu)

### 2.1 Complete Patient Journey

```
PATIENT ARRIVES
    ↓
┌─────────────────────────────────────────┐
│ 1. REGISTRATION MODULE                  │
│    - Check if existing patient          │
│    - Create/Update patient record      │
│    - Validate BHYT card                │
└─────────────────────────────────────────┘
    ↓
┌─────────────────────────────────────────┐
│ 2. APPOINTMENT MODULE                   │
│    - Check-in appointment (if any)    │
│    - Or add to walk-in queue           │
└─────────────────────────────────────────┘
    ↓
┌─────────────────────────────────────────┐
│ 3. CONSULTATION MODULE                  │
│    - Doctor sees patient               │
│    - Enter clinical notes               │
│    - Record vital signs                │
│    - Make diagnosis (ICD-10)           │
└─────────────────────────────────────────┘
    ↓
    ├─→ ┌─────────────────────────────────┐
    │   │ 4A. LAB MODULE                  │
    │   │    - Order lab tests            │
    │   │    - Receive results            │
    │   └─────────────────────────────────┘
    │
    ├─→ ┌─────────────────────────────────┐
    │   │ 4B. PRESCRIPTION MODULE          │
    │   │    - Create prescription         │
    │   │    - Check drug interactions    │
    │   │    - Send to pharmacy           │
    │   └─────────────────────────────────┘
    │
    └─→ ┌─────────────────────────────────┐
        │ 4C. DIAGNOSTICS MODULE          │
        │    - Order imaging              │
        │    - Receive results            │
        └─────────────────────────────────┘
    ↓
┌─────────────────────────────────────────┐
│ 5. PHARMACY MODULE                     │
│    - Receive prescription              │
│    - Check inventory                   │
│    - Dispense medication               │
└─────────────────────────────────────────┘
    ↓
┌─────────────────────────────────────────┐
│ 6. BILLING MODULE                      │
│    - Calculate charges                 │
│    - Apply BHYT coverage              │
│    - Generate bill                     │
│    - Process payment                   │
└─────────────────────────────────────────┘
    ↓
PATIENT LEAVES
```

### 2.2 Data Dependencies

```
PATIENT DATA
    ├─→ Used by: Registration, Appointment, Consultation, Billing
    └─→ Source: Registration Module

APPOINTMENT DATA
    ├─→ Used by: Consultation, Billing
    └─→ Source: Appointment Module

CLINICAL NOTES
    ├─→ Used by: Consultation, Reporting, Lab, Prescription
    └─→ Source: Consultation Module

LAB RESULTS
    ├─→ Used by: Consultation, Reporting
    └─→ Source: Lab Module

PRESCRIPTIONS
    ├─→ Used by: Pharmacy, Billing, Reporting
    └─→ Source: Consultation Module

BILLS
    ├─→ Used by: Billing, Reporting
    └─→ Source: Billing Module
```

---

## 3. Module Interactions (Tương tác giữa các module)

### 3.1 Registration ↔ Appointment

**Interaction:**
- Registration creates patient record
- Appointment uses patient data for booking
- Appointment check-in triggers registration update

**Data flow:**
```
Registration Module
    ↓ (creates patient)
Patient Record
    ↓ (used by)
Appointment Module
    ↓ (check-in)
Update Patient Visit History
```

### 3.2 Appointment ↔ Consultation

**Interaction:**
- Appointment provides patient context
- Consultation marks appointment as completed
- Consultation can schedule follow-up appointment

**Data flow:**
```
Appointment Module
    ↓ (patient arrives)
Check-in Appointment
    ↓ (load patient data)
Consultation Module
    ↓ (complete consultation)
Mark Appointment Completed
    ↓ (if needed)
Schedule Follow-up Appointment
```

### 3.3 Consultation ↔ Lab

**Interaction:**
- Consultation orders lab tests
- Lab results update consultation record
- Consultation reviews lab results

**Data flow:**
```
Consultation Module
    ↓ (doctor orders test)
Create Lab Order
    ↓ (lab performs test)
Lab Module
    ↓ (enter results)
Lab Results
    ↓ (notify doctor)
Consultation Module (review results)
```

### 3.4 Consultation ↔ Prescription

**Interaction:**
- Consultation creates prescription
- Prescription validates against patient allergies
- Prescription sends to pharmacy

**Data flow:**
```
Consultation Module
    ↓ (doctor prescribes)
Create Prescription
    ↓ (check interactions/allergies)
Prescription Module
    ↓ (validate)
Send to Pharmacy
    ↓ (dispense)
Pharmacy Module
```

### 3.5 Consultation ↔ Billing

**Interaction:**
- Consultation generates billable items
- Billing calculates charges
- Billing applies BHYT coverage

**Data flow:**
```
Consultation Module
    ↓ (services provided)
Billable Items
    ↓ (calculate charges)
Billing Module
    ↓ (apply BHYT)
Generate Bill
    ↓ (process payment)
Payment Record
```

---

## 4. Integration Points (Điểm tích hợp)

### 4.1 External Systems

```
┌─────────────────────────────────────────────────┐
│           HỆ THỐNG HIS (Nội bộ)                │
└─────────────────────────────────────────────────┘
         │              │              │
         │              │              │
    ┌────▼────┐    ┌────▼────┐    ┌────▼────┐
    │  BHYT   │    │   LIS   │    │  PACS   │
    │ Portal  │    │ System  │    │ System  │
    └─────────┘    └─────────┘    └─────────┘
         │              │              │
    Submit Claims  Lab Orders   Imaging Orders
    Get Status     Get Results  Get Images
```

### 4.2 BHYT Integration

**Purpose:** Submit claims, validate cards

**Data flow:**
```
Billing Module
    ↓ (generate claim)
BHYT Claim (XML)
    ↓ (submit)
BHYT Portal
    ↓ (response)
Claim Status
    ↓ (update)
Billing Module
```

### 4.3 LIS Integration

**Purpose:** Send lab orders, receive results

**Data flow:**
```
Consultation Module
    ↓ (order lab test)
Lab Order (HL7)
    ↓ (send to LIS)
LIS System
    ↓ (perform test)
Lab Results (HL7)
    ↓ (receive)
Lab Module
    ↓ (update)
Consultation Module
```

### 4.4 PACS Integration

**Purpose:** Send imaging orders, receive images

**Data flow:**
```
Consultation Module
    ↓ (order imaging)
Imaging Order (DICOM)
    ↓ (send to PACS)
PACS System
    ↓ (store images)
DICOM Images
    ↓ (retrieve)
Consultation Module
```

---

## 5. Database Overview (Tổng quan Database)

### 5.1 Core Tables

```
PATIENTS
    ├─→ PATIENT_INSURANCE
    ├─→ PATIENT_ALLERGIES
    └─→ PATIENT_MEDICAL_HISTORY

APPOINTMENTS
    └─→ APPOINTMENT_HISTORY

ENCOUNTERS (Consultations)
    ├─→ CLINICAL_NOTES
    ├─→ VITAL_SIGNS
    ├─→ DIAGNOSES
    └─→ ENCOUNTER_ITEMS

LAB_ORDERS
    ├─→ LAB_ORDER_ITEMS
    └─→ LAB_RESULTS

PRESCRIPTIONS
    ├─→ PRESCRIPTION_ITEMS
    └─→ MEDICATION_DISPENSING

BILLS
    ├─→ BILL_ITEMS
    └─→ PAYMENTS

USERS
    ├─→ ROLES
    ├─→ PERMISSIONS
    └─→ AUDIT_LOGS
```

### 5.2 Key Relationships

```
PATIENT (1) ──→ (N) APPOINTMENT
PATIENT (1) ──→ (N) ENCOUNTER
ENCOUNTER (1) ──→ (N) LAB_ORDER
ENCOUNTER (1) ──→ (N) PRESCRIPTION
ENCOUNTER (1) ──→ (1) BILL
PRESCRIPTION (1) ──→ (1) MEDICATION_DISPENSING
```

---

## 6. User Roles & Access (Vai trò & Truy cập)

### 6.1 Role-Based Module Access

```
┌─────────────────────────────────────────────────┐
│           MODULE ACCESS MATRIX                  │
└─────────────────────────────────────────────────┘

Module              │ Doctor │ Nurse │ Reception │ Cashier │ Manager
────────────────────┼────────┼───────┼───────────┼─────────┼────────
Registration        │   ❌   │   ❌   │    ✅     │   ❌    │   ✅
Appointment         │   ✅   │   ✅   │    ✅     │   ❌    │   ✅
Consultation        │   ✅   │   ✅   │    ❌     │   ❌    │   ✅
Lab & Diagnostics   │   ✅   │   ✅   │    ❌     │   ❌    │   ✅
Prescription        │   ✅   │   ❌   │    ❌     │   ❌    │   ✅
Pharmacy            │   ❌   │   ❌   │    ❌     │   ❌    │   ✅
Billing             │   ✅   │   ❌   │    ✅     │   ✅    │   ✅
Reporting           │   ✅   │   ❌   │    ✅     │   ✅    │   ✅
User Management     │   ❌   │   ❌   │    ❌     │   ❌    │   ✅
```

### 6.2 Data Access Scope

**Doctor:**
- Full access to clinical data
- Own patients + assigned patients
- Can create/modify clinical notes
- Can prescribe medications

**Nurse:**
- View assigned patients
- Enter vital signs
- Cannot prescribe or modify clinical notes

**Receptionist:**
- Patient registration
- Appointment scheduling
- Basic patient info (no clinical data)

**Cashier:**
- View bills
- Process payments
- No clinical data access

**Manager:**
- Full access to all modules
- Reporting & analytics
- User management

---

## 7. Workflow Integration (Tích hợp Workflow)

### 7.1 Registration → Appointment → Consultation

```
REGISTRATION
    ↓ (create patient)
PATIENT RECORD
    ↓ (book appointment)
APPOINTMENT
    ↓ (check-in)
CONSULTATION
    ↓ (complete)
UPDATE PATIENT HISTORY
```

### 7.2 Consultation → Lab → Results Review

```
CONSULTATION
    ↓ (order lab test)
LAB ORDER
    ↓ (perform test)
LAB RESULTS
    ↓ (notify doctor)
CONSULTATION (review)
    ↓ (update diagnosis/treatment)
UPDATED CLINICAL NOTES
```

### 7.3 Consultation → Prescription → Pharmacy → Billing

```
CONSULTATION
    ↓ (prescribe)
PRESCRIPTION
    ↓ (send to pharmacy)
PHARMACY DISPENSING
    ↓ (dispense)
BILLABLE ITEMS
    ↓ (calculate)
BILL
    ↓ (process payment)
PAYMENT
```

---

## 8. System Boundaries (Ranh giới hệ thống)

### 8.1 What's Inside HIS

**Core functions:**
- Patient management
- Clinical documentation
- Appointment scheduling
- Lab order management
- Prescription management
- Billing & insurance
- Reporting & analytics

### 8.2 What's Outside HIS

**External systems:**
- **LIS**: Lab Information System (external lab)
- **PACS**: Picture Archiving System (imaging)
- **BHYT Portal**: Insurance claims submission
- **Payment Gateway**: Credit card processing
- **Email/SMS Gateway**: Notifications

**Integration via:**
- HL7 messages (Lab)
- DICOM (Imaging)
- XML/API (BHYT, Payment)
- REST API (General)

---

## 9. Data Consistency (Tính nhất quán dữ liệu)

### 9.1 Master Data

**Single source of truth:**
- **Patient data**: Managed by Registration Module
- **User data**: Managed by User Management Module
- **Medication catalog**: Managed by Pharmacy Module
- **Service catalog**: Managed by Billing Module

**Rules:**
- Other modules reference master data
- Updates only through master module
- No duplicate data storage

### 9.2 Transactional Data

**Module-specific:**
- **Appointments**: Managed by Appointment Module
- **Clinical notes**: Managed by Consultation Module
- **Lab results**: Managed by Lab Module
- **Prescriptions**: Managed by Prescription Module
- **Bills**: Managed by Billing Module

**Rules:**
- Each module owns its data
- Cross-module access via APIs
- Audit trail for all changes

---

## 10. Scalability Considerations (Khả năng mở rộng)

### 10.1 Horizontal Scaling

**Stateless modules:**
- Consultation Module
- Lab Module
- Prescription Module
- Billing Module

**Can scale by:**
- Adding more application servers
- Load balancing

### 10.2 Vertical Scaling

**Database:**
- Centralized database
- Can scale by upgrading hardware
- Consider read replicas for reporting

### 10.3 Caching Strategy

**Cache frequently accessed data:**
- Patient lookup
- Medication catalog
- Service catalog
- User permissions

**Benefits:**
- Reduce database load
- Faster response times

---

## 11. Security Architecture (Kiến trúc Bảo mật)

### 11.1 Authentication

**Single Sign-On (SSO):**
- Centralized authentication
- All modules use same user session
- Session timeout management

### 11.2 Authorization

**Role-Based Access Control (RBAC):**
- Roles defined centrally
- Permissions per module
- Scope restrictions (ALL, OWN, ASSIGNED)

### 11.3 Data Protection

**Encryption:**
- At rest: Database encryption
- In transit: HTTPS/TLS

**Audit Logging:**
- All data access logged
- All modifications tracked
- Compliance reporting

---

## 12. Deployment Architecture (Kiến trúc Triển khai)

### 12.1 Typical Deployment

```
┌─────────────────────────────────────────────────┐
│              CLIENT DEVICES                      │
│  (Web Browsers, Tablets, Mobile Apps)           │
└─────────────────────────────────────────────────┘
                    │
                    │ HTTPS
                    │
┌─────────────────────────────────────────────────┐
│            LOAD BALANCER                         │
└─────────────────────────────────────────────────┘
                    │
        ┌───────────┼───────────┐
        │           │           │
┌───────▼───┐  ┌───▼────┐  ┌───▼────┐
│   Web     │  │  Web   │  │  Web   │
│  Server 1 │  │ Server 2│  │ Server 3│
└───────┬───┘  └───┬────┘  └───┬────┘
        │           │           │
        └───────────┼───────────┘
                    │
┌─────────────────────────────────────────────────┐
│         APPLICATION SERVER                       │
│  (Business Logic, API Layer)                    │
└─────────────────────────────────────────────────┘
                    │
┌─────────────────────────────────────────────────┐
│            DATABASE SERVER                       │
│  (MySQL/PostgreSQL)                             │
└─────────────────────────────────────────────────┘
```

### 12.2 Integration Layer

```
┌─────────────────────────────────────────────────┐
│         APPLICATION SERVER                       │
└─────────────────────────────────────────────────┘
                    │
        ┌───────────┼───────────┐
        │           │           │
┌───────▼───┐  ┌───▼────┐  ┌───▼────┐
│   BHYT    │  │   LIS  │  │  PACS  │
│  Gateway  │  │Gateway │  │Gateway │
└───────────┘  └────────┘  └────────┘
```

---

## Summary

**Key takeaways:**

1. **Modular architecture**: Mỗi module độc lập nhưng tích hợp chặt chẽ
2. **Data flow**: Rõ ràng từ registration → consultation → billing
3. **Integration points**: BHYT, LIS, PACS qua standard protocols
4. **Role-based access**: Mỗi role có quyền truy cập phù hợp
5. **Master data**: Single source of truth cho patient, user, catalog
6. **Scalability**: Stateless modules có thể scale horizontal
7. **Security**: Centralized authentication, RBAC, audit logging

**For implementation:**
- Start với core modules (Registration, Consultation, Billing)
- Add integration points sau
- Implement RBAC từ đầu
- Plan for scalability
- Security & audit logging

**Next**: Review all workflows and ensure consistency

