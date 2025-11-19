# Appointment Scheduling Workflow

# Quy trình đặt lịch hẹn khám

## Overview (Tổng quan)

Appointment Scheduling là workflow quan trọng giúp:

- Giảm thời gian chờ đợi của bệnh nhân
- Tối ưu lịch làm việc của bác sĩ
- Quản lý capacity của phòng khám
- Cải thiện trải nghiệm bệnh nhân

**Tại sao quan trọng?**

- Bệnh nhân không muốn chờ đợi lâu
- Bác sĩ cần quản lý thời gian hiệu quả
- Phòng khám cần tối ưu utilization rate
- Online booking là xu hướng hiện đại

---

## 1. Types of Appointments (Các loại lịch hẹn)

### 1.1 By Appointment Type

| Appointment Type   | Mô tả        | Thời gian  | Khi nào dùng            |
| ------------------ | ------------ | ---------- | ----------------------- |
| **New Visit**      | Khám lần đầu | 30-45 phút | Bệnh nhân mới           |
| **Follow-up**      | Tái khám     | 15-20 phút | Bệnh nhân cũ, theo dõi  |
| **Consultation**   | Hội chẩn     | 30-60 phút | Cần nhiều bác sĩ        |
| **Annual Checkup** | Khám định kỳ | 60-90 phút | Khám sức khỏe tổng quát |
| **Emergency**      | Cấp cứu      | Immediate  | Không cần đặt lịch      |

### 1.2 By Booking Channel

| Channel       | Mô tả              | Ưu điểm          | Nhược điểm            |
| ------------- | ------------------ | ---------------- | --------------------- |
| **Walk-in**   | Đến trực tiếp      | Linh hoạt        | Phải chờ đợi          |
| **Phone**     | Gọi điện           | Dễ dùng          | Cần nhân viên trả lời |
| **Online**    | Website/App        | 24/7, tự phục vụ | Cần internet          |
| **In-person** | Đặt tại phòng khám | Tư vấn trực tiếp | Phải đến phòng khám   |

---

## 2. Appointment Scheduling Workflow

### 2.1 Complete Flow

```
BƯỚC 1: Patient Requests Appointment
        (Bệnh nhân yêu cầu đặt lịch)
        - Online: Điền form trên website/app
        - Phone: Gọi điện đến phòng khám
        - In-person: Đến trực tiếp
              ↓
BƯỚC 2: Check Availability
        (Kiểm tra lịch trống)
        - Chọn bác sĩ (hoặc để hệ thống gợi ý)
        - Chọn ngày giờ mong muốn
        - System check: Lịch trống? Bác sĩ available?
              ↓
BƯỚC 3: Select Time Slot
        (Chọn khung giờ)
        - Hiển thị các slot trống
        - Patient chọn slot phù hợp
              ↓
BƯỚC 4: Provide Patient Info
        (Cung cấp thông tin)
        - New patient: Đăng ký mới
        - Returning patient: Tìm trong hệ thống
        - Confirm: Tên, SĐT, Lý do khám
              ↓
BƯỚC 5: Confirm Appointment
        (Xác nhận lịch hẹn)
        - System tạo appointment record
        - Generate appointment number
        - Send confirmation (SMS/Email)
              ↓
BƯỚC 6: Reminder (Before Appointment)
        (Nhắc nhở trước lịch hẹn)
        - 24h trước: SMS reminder
        - 2h trước: SMS reminder
        - Confirm: Bệnh nhân có đến không?
              ↓
BƯỚC 7: Check-in (Day of Appointment)
        (Check-in ngày hẹn)
        - Patient arrives
        - Front desk checks in appointment
        - Update status: "Arrived"
              ↓
BƯỚC 8: Consultation
        (Khám bệnh)
        - Doctor sees patient
        - Complete consultation
              ↓
BƯỚC 9: Complete Appointment
        (Hoàn tất)
        - Mark appointment as "Completed"
        - Schedule follow-up nếu cần
```

---

## 3. Data Model

### 3.1 Appointments Table

```sql
CREATE TABLE appointments (
  appointment_id INT PRIMARY KEY AUTO_INCREMENT,
  patient_id INT NOT NULL,

  -- Appointment details
  appointment_number VARCHAR(20) UNIQUE NOT NULL,  -- Số phiếu hẹn
  appointment_type ENUM('New Visit', 'Follow-up', 'Consultation', 'Annual Checkup', 'Emergency'),

  -- Scheduling
  doctor_id INT,                                   -- Bác sĩ khám (NULL = any available)
  department_id INT,                               -- Khoa/phòng khám
  appointment_date DATE NOT NULL,
  appointment_time TIME NOT NULL,
  duration_minutes INT DEFAULT 30,                 -- Thời gian dự kiến (phút)

  -- Status
  status ENUM('Scheduled', 'Confirmed', 'Checked-in', 'In Progress',
              'Completed', 'Cancelled', 'No-show') DEFAULT 'Scheduled',

  -- Patient info
  patient_name VARCHAR(150),
  patient_phone VARCHAR(15),
  reason_for_visit TEXT,                          -- Lý do khám

  -- Booking info
  booking_channel ENUM('Walk-in', 'Phone', 'Online', 'In-person'),
  booked_by INT,                                  -- User ID (nếu book bởi staff)
  booked_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

  -- Check-in
  checked_in_at TIMESTAMP,
  actual_start_time TIMESTAMP,
  actual_end_time TIMESTAMP,

  -- Cancellation
  cancelled_at TIMESTAMP,
  cancellation_reason TEXT,
  cancelled_by INT,                               -- User ID

  -- Notes
  notes TEXT,

  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

  FOREIGN KEY (patient_id) REFERENCES patients(patient_id),
  FOREIGN KEY (doctor_id) REFERENCES users(user_id),
  FOREIGN KEY (department_id) REFERENCES departments(department_id),

  INDEX idx_patient (patient_id),
  INDEX idx_doctor (doctor_id),
  INDEX idx_date (appointment_date),
  INDEX idx_status (status),
  INDEX idx_appointment_datetime (appointment_date, appointment_time)
);
```

### 3.2 Doctor Schedule Table

```sql
CREATE TABLE doctor_schedules (
  schedule_id INT PRIMARY KEY AUTO_INCREMENT,
  doctor_id INT NOT NULL,

  -- Schedule period
  schedule_date DATE NOT NULL,
  day_of_week TINYINT,                            -- 1=Monday, 7=Sunday

  -- Time slots
  start_time TIME NOT NULL,                        -- Giờ bắt đầu (VD: 08:00)
  end_time TIME NOT NULL,                          -- Giờ kết thúc (VD: 12:00)

  -- Availability
  is_available BOOLEAN DEFAULT TRUE,
  max_appointments INT DEFAULT 20,                 -- Số lượng appointment tối đa
  appointment_duration INT DEFAULT 30,             -- Thời gian mỗi appointment (phút)

  -- Location
  department_id INT,
  room_number VARCHAR(20),

  -- Notes
  notes TEXT,

  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

  FOREIGN KEY (doctor_id) REFERENCES users(user_id),
  FOREIGN KEY (department_id) REFERENCES departments(department_id),

  INDEX idx_doctor_date (doctor_id, schedule_date),
  INDEX idx_date (schedule_date)
);
```

### 3.3 Appointment Slots (Generated)

```sql
-- View hoặc tính toán động từ doctor_schedules
-- Ví dụ: Doctor có schedule 08:00-12:00, duration 30 phút
-- → Tạo slots: 08:00, 08:30, 09:00, 09:30, 10:00, 10:30, 11:00, 11:30

-- Logic tính toán:
-- start_time = 08:00
-- end_time = 12:00
-- duration = 30 phút
-- → Slots: 08:00, 08:30, 09:00, 09:30, 10:00, 10:30, 11:00, 11:30
```

---

## 4. Appointment Booking Flow

### 4.1 Online Booking Flow

```
Patient truy cập website/app
        ↓
Chọn "Đặt lịch khám"
        ↓
Chọn loại khám:
- Khám mới
- Tái khám
- Khám định kỳ
        ↓
Chọn bác sĩ (hoặc "Bất kỳ bác sĩ nào")
        ↓
Chọn ngày (calendar picker)
        ↓
Hiển thị các slot trống:
┌─────────────────────────────┐
│ 19/11/2023                 │
│ ───────────────────────────│
│ 08:00  [Trống]  [Chọn]    │
│ 08:30  [Trống]  [Chọn]    │
│ 09:00  [Đã đặt]            │
│ 09:30  [Trống]  [Chọn]    │
│ 10:00  [Đã đặt]            │
│ ...                        │
└─────────────────────────────┘
        ↓
Patient chọn slot
        ↓
Nhập thông tin:
- Họ tên
- Số điện thoại
- Lý do khám (optional)
        ↓
Xác nhận thông tin
        ↓
System tạo appointment
        ↓
Gửi SMS/Email confirmation:
"Bạn đã đặt lịch khám thành công!
Số phiếu: APT20231119001
Bác sĩ: Dr. Nguyễn Văn A
Ngày: 19/11/2023, 09:30
Địa chỉ: 123 Lê Loi, Q1, TPHCM"
        ↓
Done
```

### 4.2 Phone Booking Flow

```
Patient gọi điện đến phòng khám
        ↓
Receptionist trả lời
        ↓
Hỏi thông tin:
- Loại khám? (Mới/Tái khám)
- Bác sĩ cụ thể? (hoặc để gợi ý)
- Ngày giờ mong muốn?
        ↓
Receptionist check lịch trên system
        ↓
Đề xuất các slot trống:
"Tôi có các slot sau:
- 19/11, 09:30
- 19/11, 14:00
- 20/11, 08:00
Bạn muốn chọn slot nào?"
        ↓
Patient chọn slot
        ↓
Receptionist nhập thông tin vào system
        ↓
Xác nhận với patient:
"Đã đặt lịch thành công!
Số phiếu: APT20231119001
Ngày: 19/11/2023, 09:30
Sẽ gửi SMS xác nhận đến số 0909123456"
        ↓
System gửi SMS confirmation
        ↓
Done
```

---

## 5. Availability Checking Logic

### 5.1 Check Available Slots

```javascript
// Pseudo-code để check available slots
function getAvailableSlots(doctorId, date) {
  // 1. Get doctor's schedule for that date
  const schedule = getDoctorSchedule(doctorId, date);

  if (!schedule || !schedule.is_available) {
    return []; // No schedule = no slots
  }

  // 2. Generate all possible slots
  const slots = generateTimeSlots(
    schedule.start_time, // 08:00
    schedule.end_time, // 12:00
    schedule.appointment_duration // 30 minutes
  );
  // Result: [08:00, 08:30, 09:00, 09:30, 10:00, 10:30, 11:00, 11:30]

  // 3. Get existing appointments for that date
  const existingAppointments = getAppointments(doctorId, date);

  // 4. Filter out booked slots
  const availableSlots = slots.filter((slot) => {
    const isBooked = existingAppointments.some((apt) => {
      return (
        apt.appointment_time === slot.time &&
        apt.status !== "Cancelled" &&
        apt.status !== "No-show"
      );
    });

    return !isBooked;
  });

  // 5. Check max appointments limit
  if (existingAppointments.length >= schedule.max_appointments) {
    return []; // Đã đủ số lượng
  }

  return availableSlots;
}
```

### 5.2 Slot Generation Example

```
Doctor Schedule:
- Start: 08:00
- End: 12:00
- Duration: 30 phút

Generated Slots:
┌─────────────┬──────────┬──────────────┐
│ Slot Time   │ Status   │ Action       │
├─────────────┼──────────┼──────────────┤
│ 08:00       │ Available│ [Book]       │
│ 08:30       │ Booked   │ -            │
│ 09:00       │ Available│ [Book]       │
│ 09:30       │ Available│ [Book]       │
│ 10:00       │ Booked   │ -            │
│ 10:30       │ Available│ [Book]       │
│ 11:00       │ Available│ [Book]       │
│ 11:30       │ Available│ [Book]       │
└─────────────┴──────────┴──────────────┘
```

---

## 6. UI/UX Design

### 6.1 Online Booking Form

```
┌─────────────────────────────────────────────────┐
│  ĐẶT LỊCH KHÁM                                  │
└─────────────────────────────────────────────────┘

Loại khám:
○ Khám mới (30 phút)
○ Tái khám (15 phút)
○ Khám định kỳ (60 phút)

Bác sĩ:
[▼ Chọn bác sĩ...] hoặc [○ Bất kỳ bác sĩ nào]

Ngày khám:
[📅 Chọn ngày] 19/11/2023

Khung giờ trống:
┌─────────────────────────────────────────────────┐
│  Sáng (08:00 - 12:00)                          │
│  ───────────────────────────────────────────── │
│  [09:00] [09:30] [10:00] [10:30] [11:00]      │
│                                                 │
│  Chiều (13:00 - 17:00)                         │
│  ───────────────────────────────────────────── │
│  [13:00] [13:30] [14:00] [14:30] [15:00]      │
└─────────────────────────────────────────────────┘

Thông tin bệnh nhân:
Họ tên: [________________________]
Số điện thoại: [________________]
Lý do khám: [___________________] (tùy chọn)

    [Hủy]  [Đặt lịch]
```

### 6.2 Appointment Calendar View (Doctor)

```
┌─────────────────────────────────────────────────┐
│  LỊCH KHÁM - Dr. Nguyễn Văn A                  │
│  Ngày: 19/11/2023                              │
└─────────────────────────────────────────────────┘

┌─────────────┬──────────────┬──────────────────┐
│ Thời gian   │ Bệnh nhân    │ Trạng thái       │
├─────────────┼──────────────┼──────────────────┤
│ 08:00       │ Trần Văn B   │ ✓ Đã khám        │
│ 08:30       │ Lê Thị C     │ ✓ Đã khám        │
│ 09:00       │ Phạm Văn D   │ ⏳ Đang chờ      │
│ 09:30       │ Nguyễn Thị E │ ⏳ Đang chờ      │
│ 10:00       │ -            │ ○ Trống          │
│ 10:30       │ Hoàng Văn F  │ ⏳ Đang chờ      │
│ 11:00       │ -            │ ○ Trống          │
│ 11:30       │ -            │ ○ Trống          │
└─────────────┴──────────────┴──────────────────┘

Tổng: 5 appointments | Đã khám: 2 | Đang chờ: 3
```

### 6.3 Appointment List (Receptionist)

```
┌─────────────────────────────────────────────────┐
│  DANH SÁCH LỊCH HẸN - 19/11/2023               │
└─────────────────────────────────────────────────┘

Bộ lọc:
[▼ Tất cả bác sĩ] [▼ Tất cả trạng thái] [Tìm kiếm...]

┌─────────────────────────────────────────────────┐
│ 09:00  │ Phạm Văn D      │ Dr. Nguyễn Văn A   │
│        │ 0909123456     │ [Đã đến] [Đang khám]│
├─────────────────────────────────────────────────┤
│ 09:30  │ Nguyễn Thị E    │ Dr. Trần Văn B     │
│        │ 0912345678     │ [Chưa đến] [Check-in]│
├─────────────────────────────────────────────────┤
│ 10:30  │ Hoàng Văn F     │ Dr. Nguyễn Văn A   │
│        │ 0923456789     │ [Chưa đến]          │
└─────────────────────────────────────────────────┘

Tổng: 15 appointments | Đã đến: 8 | Chưa đến: 7
```

---

## 7. Appointment Status Management

### 7.1 Status Flow

```
Scheduled (Đã đặt)
    ↓
Confirmed (Đã xác nhận) - sau khi gửi reminder
    ↓
Checked-in (Đã đến) - khi patient check-in
    ↓
In Progress (Đang khám) - khi doctor bắt đầu
    ↓
Completed (Hoàn tất) - khi khám xong

Hoặc:

Scheduled/Confirmed
    ↓
Cancelled (Đã hủy) - nếu patient hủy

Hoặc:

Scheduled/Confirmed
    ↓
No-show (Không đến) - nếu đến giờ nhưng không đến
```

### 7.2 Status Definitions

| Status          | Mô tả       | Khi nào           | Action                    |
| --------------- | ----------- | ----------------- | ------------------------- |
| **Scheduled**   | Đã đặt lịch | Sau khi book      | Gửi confirmation          |
| **Confirmed**   | Đã xác nhận | Sau reminder      | -                         |
| **Checked-in**  | Đã đến      | Patient check-in  | Gửi cho doctor            |
| **In Progress** | Đang khám   | Doctor bắt đầu    | -                         |
| **Completed**   | Hoàn tất    | Khám xong         | Có thể đặt follow-up      |
| **Cancelled**   | Đã hủy      | Patient/staff hủy | Free up slot              |
| **No-show**     | Không đến   | Quá giờ không đến | Free up slot, có thể phạt |

---

## 8. Reminder System

### 8.1 Reminder Schedule

```
Appointment: 19/11/2023, 09:30

Reminder 1: 18/11/2023, 09:30 (24h trước)
SMS: "Nhắc nhở: Bạn có lịch khám ngày mai 19/11 lúc 09:30
với Dr. Nguyễn Văn A. Vui lòng đến đúng giờ.
Hủy lịch: Reply CANCEL"

Reminder 2: 19/11/2023, 07:30 (2h trước)
SMS: "Nhắc nhở: Bạn có lịch khám hôm nay lúc 09:30
với Dr. Nguyễn Văn A. Địa chỉ: 123 Lê Loi, Q1, TPHCM"

Confirmation: 19/11/2023, 08:00 (1.5h trước)
SMS: "Xác nhận bạn có đến khám lúc 09:30 không?
Reply YES để xác nhận, NO để hủy"
```

### 8.2 Reminder Logic

```javascript
// Daily job: Check appointments tomorrow
function sendReminders() {
  const tomorrow = addDays(today(), 1);
  const appointments = getAppointmentsForDate(tomorrow);

  for (const apt of appointments) {
    // Check if already sent reminder
    if (apt.reminder_sent) continue;

    // Send 24h reminder
    sendSMS(apt.patient_phone, {
      message: `Nhắc nhở: Bạn có lịch khám ngày mai ${formatDate(
        apt.appointment_date
      )} lúc ${formatTime(apt.appointment_time)} với ${
        apt.doctor_name
      }. Vui lòng đến đúng giờ.`,
    });

    // Mark as sent
    markReminderSent(apt.appointment_id);
  }
}

// Hourly job: Check appointments in 2 hours
function send2HourReminders() {
  const in2Hours = addHours(now(), 2);
  const appointments = getAppointmentsAroundTime(in2Hours);

  for (const apt of appointments) {
    if (apt.status === "Checked-in" || apt.status === "Completed") continue;

    sendSMS(apt.patient_phone, {
      message: `Nhắc nhở: Bạn có lịch khám trong 2 giờ nữa lúc ${formatTime(
        apt.appointment_time
      )} với ${apt.doctor_name}.`,
    });
  }
}
```

---

## 9. Cancellation & Rescheduling

### 9.1 Cancellation Flow

```
Patient muốn hủy lịch
    ↓
Chọn phương thức:
- Online: Click "Hủy lịch" trên website/app
- Phone: Gọi điện đến phòng khám
    ↓
Xác nhận hủy:
"Bạn có chắc chắn muốn hủy lịch khám
ngày 19/11/2023, 09:30 không?"
    ↓
Chọn lý do hủy (optional):
- Thay đổi kế hoạch
- Bận việc
- Đã khỏi bệnh
- Khác
    ↓
Confirm cancellation
    ↓
Update appointment status = "Cancelled"
    ↓
Free up slot (có thể cho người khác đặt)
    ↓
Gửi SMS confirmation:
"Đã hủy lịch khám thành công.
Số phiếu: APT20231119001
Ngày: 19/11/2023, 09:30"
```

### 9.2 Rescheduling Flow

```
Patient muốn đổi lịch
    ↓
Chọn appointment cần đổi
    ↓
Hiển thị lịch mới available
    ↓
Chọn slot mới
    ↓
Confirm reschedule
    ↓
Update appointment:
- Old slot: Free up
- New slot: Book
- Status: "Scheduled"
    ↓
Gửi SMS:
"Đã đổi lịch khám thành công.
Lịch mới: 20/11/2023, 14:00
Lịch cũ: 19/11/2023, 09:30"
```

---

## 10. Queue Management

### 10.1 Walk-in Queue

```
Walk-in patient đến
    ↓
Check có appointment không?
    ↓
  Có appointment:
  → Check-in appointment
  → Priority queue (ưu tiên hơn walk-in)

  Không có appointment:
  → Add to walk-in queue
  → Wait for available slot
    ↓
Doctor available
    ↓
Call next patient:
- Appointment patients first (theo thứ tự giờ hẹn)
- Walk-in patients sau
```

### 10.2 Queue Display

```
┌─────────────────────────────────────────────────┐
│  HÀNG ĐỢI KHÁM - 19/11/2023                    │
└─────────────────────────────────────────────────┘

Đang khám:
┌─────────────────────────────────────────────────┐
│ [A001] Nguyễn Văn A - Dr. Nguyễn Văn B        │
│ Bắt đầu: 09:15                                 │
└─────────────────────────────────────────────────┘

Đang chờ (có hẹn):
┌─────────────────────────────────────────────────┐
│ [A002] Trần Thị C - 09:30 - Dr. Nguyễn Văn B  │
│ [A003] Lê Văn D  - 10:00 - Dr. Nguyễn Văn B   │
│ [A004] Phạm Thị E - 10:30 - Dr. Nguyễn Văn B  │
└─────────────────────────────────────────────────┘

Đang chờ (không hẹn):
┌─────────────────────────────────────────────────┐
│ [W001] Hoàng Văn F - Đến lúc 09:20            │
│ [W002] Võ Thị G   - Đến lúc 09:25            │
└─────────────────────────────────────────────────┘

Tổng: 6 người | Đang khám: 1 | Chờ: 5
```

---

## 11. No-show Management

### 11.1 No-show Detection

```
Appointment time: 09:30
Current time: 09:45 (15 phút sau)
    ↓
Check status:
- Status = "Scheduled" hoặc "Confirmed"
- Patient chưa check-in
    ↓
Mark as "No-show"
    ↓
Free up slot (có thể cho walk-in)
    ↓
Gửi SMS:
"Bạn đã không đến khám theo lịch hẹn
ngày 19/11/2023, 09:30.
Vui lòng đặt lịch lại nếu cần."
```

### 11.2 No-show Policy

**Common policies:**

- **First no-show**: Warning, vẫn cho đặt lịch
- **Multiple no-shows**: Yêu cầu đặt cọc hoặc chỉ nhận walk-in
- **No-show rate tracking**: Track để đánh giá

---

## 12. Business Rules & Validation

### 12.1 Booking Rules

**Time restrictions:**

- Không thể đặt lịch quá khứ
- Không thể đặt lịch quá xa (VD: tối đa 3 tháng)
- Phải đặt trước tối thiểu X giờ (VD: 2 giờ)

**Capacity limits:**

- Mỗi doctor có max appointments/ngày
- Mỗi slot chỉ 1 appointment
- Check double-booking

**Patient restrictions:**

- Một patient không thể đặt 2 appointments cùng giờ
- Có thể đặt nhiều appointments khác giờ

### 12.2 Validation Examples

```javascript
function validateAppointmentBooking(appointment) {
  const errors = [];

  // 1. Check appointment date not in past
  if (appointment.appointment_date < today()) {
    errors.push("Không thể đặt lịch trong quá khứ");
  }

  // 2. Check minimum advance booking (2 hours)
  const appointmentDateTime = combineDateTime(
    appointment.appointment_date,
    appointment.appointment_time
  );
  const minAdvanceTime = addHours(now(), 2);

  if (appointmentDateTime < minAdvanceTime) {
    errors.push("Phải đặt lịch trước tối thiểu 2 giờ");
  }

  // 3. Check doctor availability
  const availableSlots = getAvailableSlots(
    appointment.doctor_id,
    appointment.appointment_date
  );

  const slotExists = availableSlots.some(
    (slot) => slot.time === appointment.appointment_time
  );

  if (!slotExists) {
    errors.push("Khung giờ này không còn trống");
  }

  // 4. Check patient doesn't have conflicting appointment
  const conflictingAppt = getPatientAppointments(
    appointment.patient_id,
    appointment.appointment_date
  ).find(
    (apt) =>
      apt.appointment_time === appointment.appointment_time &&
      apt.status !== "Cancelled"
  );

  if (conflictingAppt) {
    errors.push("Bạn đã có lịch hẹn khác vào giờ này");
  }

  return errors;
}
```

---

## 13. Integration Points

### 13.1 Integration với Registration

```
Appointment → Check-in → Registration
    ↓
Patient có appointment đến
    ↓
Front desk check-in appointment
    ↓
Nếu new patient → Registration flow
Nếu returning patient → Quick check-in
    ↓
Proceed to consultation
```

### 13.2 Integration với Consultation

```
Appointment → Consultation
    ↓
Doctor opens appointment
    ↓
Load patient info từ appointment
    ↓
Start consultation
    ↓
Complete consultation
    ↓
Mark appointment as "Completed"
```

### 13.3 Integration với Billing

```
Appointment → Consultation → Billing
    ↓
Appointment có thể có:
- Consultation fee (phí khám)
- Service fees (nếu book kèm dịch vụ)
    ↓
Khi complete appointment
    ↓
Auto-create bill items
```

---

## 14. Common Issues & Solutions

### Issue 1: Double-booking

**Problem**: 2 patients được đặt cùng slot
**Solution**:

- Database constraint: Unique (doctor_id, appointment_date, appointment_time)
- Check availability trước khi book
- Lock slot khi đang book

### Issue 2: No-show nhiều

**Problem**: Bệnh nhân không đến, lãng phí thời gian bác sĩ
**Solution**:

- Reminder system
- No-show tracking
- Policy: Yêu cầu confirm trước giờ hẹn
- Có thể yêu cầu đặt cọc

### Issue 3: Walk-in vs Appointment conflict

**Problem**: Walk-in đến nhiều, appointment phải chờ
**Solution**:

- Priority queue: Appointment trước, walk-in sau
- Reserve buffer time cho walk-in
- Limit số walk-in/ngày

### Issue 4: Doctor late/cancelled

**Problem**: Bác sĩ trễ hoặc hủy lịch
**Solution**:

- Notification system cho patients
- Auto-reschedule hoặc offer alternative doctor
- Compensation policy

---

## 15. Performance Considerations

### 15.1 Optimize Slot Availability Check

**Problem**: Check availability chậm khi có nhiều appointments

**Solutions:**

1. **Cache available slots** (refresh mỗi 5 phút)
2. **Pre-calculate slots** cho ngày hôm sau
3. **Index database** properly:
   ```sql
   INDEX idx_doctor_date_time (doctor_id, appointment_date, appointment_time);
   ```

### 15.2 Handle Concurrent Bookings

**Problem**: Nhiều users cùng book slot cuối cùng

**Solution:**

- Database transaction với proper locking
- Optimistic locking: Check version trước khi update
- Queue system cho booking requests

---

## Summary

**Key takeaways:**

1. **Appointment scheduling** giúp tối ưu thời gian và cải thiện UX
2. **Multiple booking channels**: Online, phone, walk-in
3. **Availability checking** là core logic
4. **Reminder system** giảm no-show rate
5. **Queue management** cân bằng appointment vs walk-in
6. **Status management** track toàn bộ lifecycle
7. **Integration** với registration, consultation, billing

**For implementation:**

- Start với basic scheduling (doctor schedule + appointments)
- Add online booking sau
- Implement reminder system
- Queue management cho walk-in
- No-show tracking và policies

**Next**: [07-user-management-security.md](07-user-management-security.md)
