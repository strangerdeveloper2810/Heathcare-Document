# User Management & Security
# Quản lý người dùng & Bảo mật

## Overview (Tổng quan)

User Management & Security là foundation của hệ thống HIS. Đảm bảo:
- Đúng người truy cập đúng dữ liệu
- Tuân thủ quy định bảo mật
- Audit trail đầy đủ
- Phân quyền rõ ràng

**Tại sao quan trọng?**
- Bảo vệ thông tin bệnh nhân (PHI - Protected Health Information)
- Tuân thủ quy định (VN regulations, best practices)
- Trách nhiệm rõ ràng (ai làm gì, khi nào)
- Ngăn chặn truy cập trái phép

---

## 1. User Roles & Responsibilities

### 1.1 Clinical Roles

| Role | Tiếng Việt | Responsibilities | Access Level |
|------|------------|------------------|--------------|
| **Doctor** | Bác sĩ | - Khám bệnh<br>- Kê đơn<br>- Xem kết quả xét nghiệm<br>- Ghi chép hồ sơ | Full patient data<br>Can create/modify clinical notes<br>Can prescribe |
| **Nurse** | Y tá/Điều dưỡng | - Đo vital signs<br>- Thực hiện y lệnh<br>- Ghi chép chăm sóc | Can view patient data<br>Can enter vital signs<br>Cannot prescribe |
| **Pharmacist** | Dược sĩ | - Cấp phát thuốc<br>- Kiểm tra đơn thuốc<br>- Tư vấn thuốc | Can view prescriptions<br>Can dispense medications<br>Cannot prescribe |
| **Lab Technician** | Kỹ thuật viên xét nghiệm | - Nhập kết quả xét nghiệm<br>- Quản lý lab orders | Can view lab orders<br>Can enter lab results<br>Limited patient data |

### 1.2 Administrative Roles

| Role | Tiếng Việt | Responsibilities | Access Level |
|------|------------|------------------|--------------|
| **Receptionist** | Lễ tân/Tiếp nhận | - Đăng ký bệnh nhân<br>- Đặt lịch hẹn<br>- Check-in/out | Can create/view patients<br>Can schedule appointments<br>Limited clinical data |
| **Cashier** | Thu ngân | - Thu tiền<br>- In hóa đơn<br>- Xử lý thanh toán | Can view bills<br>Can process payments<br>No clinical data |
| **Billing Staff** | Nhân viên kế toán | - Tạo hóa đơn<br>- Xử lý BHYT claims<br>- Báo cáo tài chính | Can view/create bills<br>Can submit claims<br>Limited clinical data |
| **Medical Records** | Nhân viên quản lý hồ sơ | - Quản lý hồ sơ<br>- Scan documents<br>- Cung cấp hồ sơ | Can view all records<br>Can upload documents<br>Cannot modify clinical notes |

### 1.3 Management Roles

| Role | Tiếng Việt | Responsibilities | Access Level |
|------|------------|------------------|--------------|
| **Clinic Manager** | Quản lý phòng khám | - Quản lý nhân sự<br>- Xem báo cáo<br>- Cấu hình hệ thống | Full access<br>Can manage users<br>Can view all reports |
| **IT Admin** | Quản trị viên IT | - Quản lý hệ thống<br>- Cấu hình<br>- Troubleshooting | Full system access<br>Can manage all users<br>Technical access |

---

## 2. Role-Based Access Control (RBAC)

### 2.1 Permission Model

**Concept:**
- **Role**: Tập hợp permissions (VD: Doctor, Nurse)
- **Permission**: Quyền cụ thể (VD: "View Patient", "Create Prescription")
- **Resource**: Đối tượng cần bảo vệ (VD: Patient Record, Lab Result)

**Example:**
```
Role: Doctor
Permissions:
  - View Patient (all patients)
  - Create Clinical Note
  - Prescribe Medication
  - View Lab Results
  - Modify Own Notes

Role: Nurse
Permissions:
  - View Patient (assigned patients)
  - Enter Vital Signs
  - View Lab Results
  - Cannot: Prescribe, Modify Clinical Notes
```

### 2.2 Permission Matrix

| Permission | Doctor | Nurse | Pharmacist | Receptionist | Cashier |
|------------|--------|-------|------------|--------------|---------|
| **View Patient List** | ✅ | ✅ | ❌ | ✅ | ❌ |
| **View Patient Details** | ✅ All | ✅ Assigned | ✅ Prescription only | ✅ Basic info | ❌ |
| **Create Patient** | ❌ | ❌ | ❌ | ✅ | ❌ |
| **Modify Patient** | ✅ Limited | ❌ | ❌ | ✅ | ❌ |
| **View Clinical Notes** | ✅ All | ✅ Assigned | ❌ | ❌ | ❌ |
| **Create Clinical Note** | ✅ | ❌ | ❌ | ❌ | ❌ |
| **Modify Clinical Note** | ✅ Own only | ❌ | ❌ | ❌ | ❌ |
| **Prescribe Medication** | ✅ | ❌ | ❌ | ❌ | ❌ |
| **Dispense Medication** | ❌ | ❌ | ✅ | ❌ | ❌ |
| **View Lab Results** | ✅ | ✅ | ❌ | ❌ | ❌ |
| **Enter Lab Results** | ❌ | ❌ | ❌ | ❌ | ❌ |
| **View Bills** | ✅ Own patients | ❌ | ❌ | ✅ | ✅ |
| **Create Bills** | ❌ | ❌ | ❌ | ✅ | ❌ |
| **Process Payments** | ❌ | ❌ | ❌ | ❌ | ✅ |
| **View Reports** | ✅ Clinical | ❌ | ❌ | ✅ Basic | ✅ Financial |

---

## 3. Data Model

### 3.1 Users Table

```sql
CREATE TABLE users (
  user_id INT PRIMARY KEY AUTO_INCREMENT,
  
  -- Authentication
  username VARCHAR(50) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  email VARCHAR(100),
  
  -- User info
  full_name VARCHAR(150) NOT NULL,
  employee_id VARCHAR(20) UNIQUE,              -- Mã nhân viên
  title VARCHAR(100),                            -- Chức danh: Bác sĩ, Y tá...
  
  -- Role & Department
  role_id INT NOT NULL,                        -- Foreign key to roles
  department_id INT,                            -- Khoa/phòng
  
  -- Professional info
  license_number VARCHAR(50),                   -- Số chứng chỉ hành nghề
  specialty VARCHAR(100),                       -- Chuyên khoa (nếu là bác sĩ)
  
  -- Contact
  phone_number VARCHAR(15),
  
  -- Status
  is_active BOOLEAN DEFAULT TRUE,
  last_login TIMESTAMP,
  
  -- Timestamps
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  created_by INT,
  
  FOREIGN KEY (role_id) REFERENCES roles(role_id),
  FOREIGN KEY (department_id) REFERENCES departments(department_id),
  
  INDEX idx_username (username),
  INDEX idx_role (role_id),
  INDEX idx_department (department_id)
);
```

### 3.2 Roles Table

```sql
CREATE TABLE roles (
  role_id INT PRIMARY KEY AUTO_INCREMENT,
  role_code VARCHAR(50) UNIQUE NOT NULL,        -- DOCTOR, NURSE, PHARMACIST...
  role_name_vi VARCHAR(100) NOT NULL,          -- Bác sĩ, Y tá...
  role_name_en VARCHAR(100),
  
  description TEXT,
  
  is_active BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### 3.3 Permissions Table

```sql
CREATE TABLE permissions (
  permission_id INT PRIMARY KEY AUTO_INCREMENT,
  permission_code VARCHAR(100) UNIQUE NOT NULL, -- VIEW_PATIENT, CREATE_PRESCRIPTION...
  permission_name_vi VARCHAR(255) NOT NULL,
  permission_name_en VARCHAR(255),
  
  resource_type VARCHAR(50),                    -- Patient, Prescription, Bill...
  action VARCHAR(50),                           -- View, Create, Modify, Delete
  
  description TEXT,
  
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### 3.4 Role Permissions (Many-to-Many)

```sql
CREATE TABLE role_permissions (
  role_id INT NOT NULL,
  permission_id INT NOT NULL,
  
  -- Optional: Scope restrictions
  scope VARCHAR(50),                            -- ALL, OWN, ASSIGNED, DEPARTMENT
  
  PRIMARY KEY (role_id, permission_id),
  FOREIGN KEY (role_id) REFERENCES roles(role_id),
  FOREIGN KEY (permission_id) REFERENCES permissions(permission_id)
);
```

### 3.5 User Sessions & Audit Logs

```sql
CREATE TABLE user_sessions (
  session_id VARCHAR(100) PRIMARY KEY,
  user_id INT NOT NULL,
  
  login_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  logout_time TIMESTAMP,
  ip_address VARCHAR(45),
  user_agent TEXT,
  
  is_active BOOLEAN DEFAULT TRUE,
  
  FOREIGN KEY (user_id) REFERENCES users(user_id),
  INDEX idx_user (user_id),
  INDEX idx_login_time (login_time)
);

CREATE TABLE audit_logs (
  log_id INT PRIMARY KEY AUTO_INCREMENT,
  user_id INT,
  
  -- Action details
  action_type VARCHAR(50) NOT NULL,            -- LOGIN, VIEW, CREATE, MODIFY, DELETE
  resource_type VARCHAR(50),                   -- Patient, Prescription, Bill...
  resource_id INT,                             -- ID của resource
  
  -- Details
  action_description TEXT,
  old_values JSON,                              -- Giá trị cũ (nếu modify)
  new_values JSON,                              -- Giá trị mới
  
  -- Context
  ip_address VARCHAR(45),
  user_agent TEXT,
  
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  FOREIGN KEY (user_id) REFERENCES users(user_id),
  INDEX idx_user (user_id),
  INDEX idx_action (action_type),
  INDEX idx_resource (resource_type, resource_id),
  INDEX idx_created_at (created_at)
);
```

---

## 4. Authentication & Authorization Flow

### 4.1 Login Flow

```
User truy cập hệ thống
    ↓
Nhập username/password
    ↓
System validate credentials
    ↓
Check user status:
- is_active = TRUE?
- Account locked?
- Password expired?
    ↓
  Valid:
  → Create session
  → Load user roles & permissions
  → Redirect to dashboard
  → Log login event
  
  Invalid:
  → Show error
  → Track failed attempts
  → Lock account nếu quá nhiều lần sai
```

### 4.2 Authorization Check Flow

```
User requests action (VD: View Patient)
    ↓
System checks:
1. User authenticated? (có session không?)
2. User có permission không?
   - Load user's role
   - Check role có permission "VIEW_PATIENT" không?
3. Check scope:
   - ALL: Xem tất cả
   - OWN: Chỉ xem của mình
   - ASSIGNED: Chỉ xem được assign
   - DEPARTMENT: Chỉ xem trong khoa
    ↓
  Authorized:
  → Allow action
  → Log action (audit log)
  
  Unauthorized:
  → Deny action
  → Show error: "Bạn không có quyền thực hiện hành động này"
  → Log unauthorized attempt
```

---

## 5. Permission Examples

### 5.1 View Patient Permission

```javascript
// Check if user can view patient
function canViewPatient(userId, patientId) {
  const user = getUser(userId);
  const permission = 'VIEW_PATIENT';
  
  // Check if role has permission
  if (!hasPermission(user.role_id, permission)) {
    return { allowed: false, reason: 'No permission' };
  }
  
  // Check scope
  const scope = getPermissionScope(user.role_id, permission);
  
  switch (scope) {
    case 'ALL':
      return { allowed: true };
      
    case 'OWN':
      // Check if patient assigned to this doctor
      const assignedDoctor = getAssignedDoctor(patientId);
      return { 
        allowed: assignedDoctor === userId 
      };
      
    case 'ASSIGNED':
      // Check if patient in user's assigned list
      const assignedPatients = getAssignedPatients(userId);
      return { 
        allowed: assignedPatients.includes(patientId) 
      };
      
    case 'DEPARTMENT':
      // Check if patient in same department
      const patientDept = getPatientDepartment(patientId);
      return { 
        allowed: patientDept === user.department_id 
      };
      
    default:
      return { allowed: false, reason: 'Unknown scope' };
  }
}
```

### 5.2 Create Prescription Permission

```javascript
// Check if user can prescribe
function canPrescribe(userId, patientId) {
  const user = getUser(userId);
  
  // Only doctors can prescribe
  if (user.role_code !== 'DOCTOR') {
    return { allowed: false, reason: 'Only doctors can prescribe' };
  }
  
  // Check if user has permission
  if (!hasPermission(user.role_id, 'CREATE_PRESCRIPTION')) {
    return { allowed: false, reason: 'No permission' };
  }
  
  // Check if user can view this patient (prerequisite)
  const canView = canViewPatient(userId, patientId);
  if (!canView.allowed) {
    return { allowed: false, reason: 'Cannot view patient' };
  }
  
  return { allowed: true };
}
```

---

## 6. UI/UX Design

### 6.1 Login Screen

```
┌─────────────────────────────────────────────────┐
│            HỆ THỐNG QUẢN LÝ PHÒNG KHÁM         │
│                                                 │
│  ────────────────────────────────────────────  │
│                                                 │
│  Tên đăng nhập:                                │
│  [________________________]                     │
│                                                 │
│  Mật khẩu:                                     │
│  [________________________]  [👁️]              │
│                                                 │
│  ☐ Ghi nhớ đăng nhập                           │
│                                                 │
│  [Quên mật khẩu?]                              │
│                                                 │
│           [Đăng nhập]                           │
│                                                 │
└─────────────────────────────────────────────────┘
```

### 6.2 User Management Screen (Admin)

```
┌─────────────────────────────────────────────────┐
│  QUẢN LÝ NGƯỜI DÙNG                           │
└─────────────────────────────────────────────────┘

[+ Thêm người dùng mới]  [Tìm kiếm...]  [Lọc theo vai trò ▼]

┌─────────────────────────────────────────────────┐
│ Tên đăng nhập │ Họ tên      │ Vai trò │ Trạng thái │
├─────────────────────────────────────────────────┤
│ dr.nguyen     │ Nguyễn Văn A │ Bác sĩ  │ ✅ Hoạt động│
│ nurse.tran    │ Trần Thị B   │ Y tá    │ ✅ Hoạt động│
│ pharm.le      │ Lê Văn C     │ Dược sĩ │ ✅ Hoạt động│
│ reception.vo  │ Võ Thị D     │ Lễ tân  │ ❌ Khóa     │
└─────────────────────────────────────────────────┘

[Chỉnh sửa]  [Khóa/Mở khóa]  [Đặt lại mật khẩu]  [Xóa]
```

### 6.3 Role Permission Management

```
┌─────────────────────────────────────────────────┐
│  PHÂN QUYỀN - Vai trò: Bác sĩ                  │
└─────────────────────────────────────────────────┘

Quyền truy cập:
┌─────────────────────────────────────────────────┐
│ ☑ Xem danh sách bệnh nhân                      │
│ ☑ Xem chi tiết bệnh nhân                       │
│ ☑ Tạo ghi chép lâm sàng                        │
│ ☑ Sửa ghi chép lâm sàng (chỉ của mình)        │
│ ☑ Kê đơn thuốc                                 │
│ ☑ Xem kết quả xét nghiệm                       │
│ ☑ Xem hóa đơn                                  │
│ ☐ Tạo hóa đơn                                  │
│ ☐ Xử lý thanh toán                             │
│ ☐ Quản lý người dùng                           │
└─────────────────────────────────────────────────┘

    [Hủy]  [Lưu thay đổi]
```

---

## 7. Security Best Practices

### 7.1 Password Policy

**Requirements:**
- Minimum 8 characters
- Must contain: uppercase, lowercase, number
- Cannot reuse last 5 passwords
- Expire after 90 days (optional)
- Lock account after 5 failed attempts

**Implementation:**
```javascript
function validatePassword(password) {
  const errors = [];
  
  if (password.length < 8) {
    errors.push('Mật khẩu phải có ít nhất 8 ký tự');
  }
  
  if (!/[A-Z]/.test(password)) {
    errors.push('Mật khẩu phải có chữ hoa');
  }
  
  if (!/[a-z]/.test(password)) {
    errors.push('Mật khẩu phải có chữ thường');
  }
  
  if (!/[0-9]/.test(password)) {
    errors.push('Mật khẩu phải có số');
  }
  
  return errors;
}
```

### 7.2 Session Management

**Session timeout:**
- Inactive timeout: 30 phút
- Absolute timeout: 8 giờ
- Force logout on password change

**Session security:**
- HTTPS only
- Secure cookies (HttpOnly, Secure)
- CSRF protection
- IP address validation (optional)

### 7.3 Data Encryption

**At rest:**
- Encrypt sensitive fields (password, SSN, BHYT card number)
- Database encryption

**In transit:**
- HTTPS/TLS for all communications
- Encrypt API responses nếu cần

---

## 8. Audit Trail

### 8.1 What to Log

**Authentication events:**
- Login (success/failure)
- Logout
- Password change
- Account lock/unlock

**Data access:**
- View patient record
- View lab results
- View prescriptions
- View bills

**Data modification:**
- Create patient
- Modify patient info
- Create prescription
- Modify clinical note
- Process payment

**Critical actions:**
- Delete records
- Export data
- System configuration changes

### 8.2 Audit Log Example

```sql
-- Example audit log entries
INSERT INTO audit_logs (user_id, action_type, resource_type, resource_id, action_description)
VALUES
  (123, 'LOGIN', 'SYSTEM', NULL, 'User logged in from IP 192.168.1.100'),
  (123, 'VIEW', 'PATIENT', 456, 'Viewed patient record P2023001234'),
  (123, 'CREATE', 'PRESCRIPTION', 789, 'Created prescription RX20231119001'),
  (123, 'MODIFY', 'CLINICAL_NOTE', 101, 'Modified clinical note for encounter 20231119001');
```

### 8.3 Audit Report

```
┌─────────────────────────────────────────────────┐
│  BÁO CÁO AUDIT - 19/11/2023                    │
└─────────────────────────────────────────────────┘

Thời gian    │ Người dùng      │ Hành động        │ Chi tiết
─────────────┼─────────────────┼──────────────────┼─────────────
08:15        │ dr.nguyen       │ LOGIN            │ IP: 192.168.1.100
08:20        │ dr.nguyen       │ VIEW_PATIENT     │ Patient: P2023001234
08:25        │ dr.nguyen       │ CREATE_PRESC     │ RX20231119001
08:30        │ nurse.tran      │ VIEW_PATIENT     │ Patient: P2023001234
08:35        │ nurse.tran      │ ENTER_VITALS     │ Encounter: 20231119001
09:00        │ cashier.vo      │ PROCESS_PAYMENT  │ Bill: INV20231119001
```

---

## 9. Common Security Scenarios

### 9.1 Doctor Leaves, Patient Needs Follow-up

**Scenario:** Bác sĩ nghỉ việc, bệnh nhân cần tái khám

**Solution:**
- Reassign patients to another doctor
- Transfer all appointments
- Maintain access to historical records (read-only)
- New doctor can view previous notes

### 9.2 Temporary Access (Coverage)

**Scenario:** Bác sĩ A nghỉ, bác sĩ B cover

**Solution:**
- Temporary role assignment
- Time-limited access
- Auto-revoke after period
- Audit all actions

### 9.3 Emergency Access

**Scenario:** Cấp cứu, cần access nhanh

**Solution:**
- Emergency access protocol
- Break-glass access (with approval)
- Post-access review required
- Alert admin immediately

---

## 10. Vietnam-Specific Requirements

### 10.1 License Management

**Bác sĩ phải có:**
- Chứng chỉ hành nghề (Medical License)
- Chứng chỉ chuyên khoa (Specialty Certificate)
- Cập nhật định kỳ

**System cần:**
- Store license number
- Track expiration date
- Alert before expiration
- Prevent prescribing if expired

### 10.2 Data Retention

**Quy định VN:**
- Hồ sơ bệnh án: Lưu tối thiểu 15-20 năm
- Không được xóa hồ sơ (chỉ archive)
- Backup định kỳ

---

## 11. User Onboarding & Training

### 11.1 New User Setup

```
Admin creates user account
    ↓
Set initial password (temporary)
    ↓
Assign role
    ↓
Set permissions
    ↓
Send credentials to user
    ↓
User logs in first time
    ↓
Force password change
    ↓
Complete profile
    ↓
Training (optional)
    ↓
User ready to use system
```

### 11.2 Training Checklist

**For Doctors:**
- [ ] Login & navigation
- [ ] Patient search
- [ ] Clinical note entry
- [ ] Prescription workflow
- [ ] Lab results review
- [ ] ICD-10 coding

**For Nurses:**
- [ ] Patient check-in
- [ ] Vital signs entry
- [ ] Appointment management
- [ ] Task management

**For Receptionists:**
- [ ] Patient registration
- [ ] Appointment scheduling
- [ ] BHYT validation
- [ ] Payment processing

---

## Summary

**Key takeaways:**

1. **RBAC** là foundation cho security
2. **Role-based permissions** đơn giản và hiệu quả
3. **Audit logs** bắt buộc cho compliance
4. **Session management** quan trọng cho security
5. **Password policy** bảo vệ account
6. **Scope restrictions** (ALL, OWN, ASSIGNED) linh hoạt
7. **Vietnam-specific**: License management, data retention

**For implementation:**
- Start với basic roles (Doctor, Nurse, Admin)
- Implement RBAC với permission matrix
- Audit logging cho critical actions
- Session management với timeout
- Password policy enforcement

**Next**: [08-reporting-analytics.md](08-reporting-analytics.md)

