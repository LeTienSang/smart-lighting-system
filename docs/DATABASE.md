# DATABASE — Smart Lighting System

> Đây là database specification. Agent phải tham khảo file này **trước khi** thay đổi schema hoặc viết database-related code. Không tự thêm bảng, không tự quyết định data type/index/cascade behavior/constraint nếu chưa được xác định — đánh dấu `TODO/UNDEFINED`.

## 1. Database Overview

- **Hệ quản trị:** PostgreSQL (phiên bản 18 theo bảng công nghệ trong `ARCHITECTURE.md`).
- **Vai trò:** lưu trữ toàn bộ dữ liệu quản lý và vận hành của hệ thống. Backend là thành phần **duy nhất** được phép truy cập trực tiếp PostgreSQL.
- **Dữ liệu quản lý:**
  - Telemetry — dữ liệu cảm biến gửi định kỳ từ thiết bị.
  - Command — lệnh điều khiển gửi tới thiết bị và trạng thái/kết quả.
  - Alert — cảnh báo phát sinh (lamp fault, device offline...).
  - Audit — nhật ký các thao tác quan trọng trong hệ thống.
  - Ngoài ra còn User, Role, Device phục vụ quản lý người dùng, phân quyền và thiết bị.
- Cấu hình Adaptive Lighting (`adaptive_config`) dùng kiểu **JSONB**, lưu trong bảng `DEVICE`.

## 2. Entities

Đúng 7 entity hiện tại (không tự thêm bảng):

```text
ROLE
USER
DEVICE
TELEMETRY
COMMAND
ALERT
AUDIT_LOG
```

## 3. Table Specification — quy ước chung

Với mỗi bảng dưới đây:
- Cột đầu tiên trong danh sách field là khóa chính (primary key), theo quy ước đặt tên trong báo cáo gốc.
- **Data type, độ dài, index, cascade behavior, và constraint cụ thể** (NOT NULL, UNIQUE, CHECK...) **không được báo cáo gốc xác định** ở mức chi tiết SQL → đánh dấu `TODO/UNDEFINED` cho từng bảng, cần nhóm bổ sung trước khi viết migration.

## 4. ROLE

- **Table name:** `ROLE`
- **Purpose:** Định nghĩa các vai trò trong hệ thống (Admin, Operator, Viewer).
- **Columns:**

| Column | Ghi chú |
|---|---|
| `role_id` | Primary key |
| `role_name` | Tên vai trò (Admin/Operator/Viewer) |
| `description` | Mô tả vai trò |

- **Primary key:** `role_id`
- **Foreign keys:** không có
- **Relationships:** `ROLE → USER` (1:N)
- **Constraints:** TODO/UNDEFINED (ví dụ: `role_name` có UNIQUE hay không — chưa xác định)

## 5. USER

- **Table name:** `USER`
- **Purpose:** Lưu thông tin tài khoản người dùng.
- **Columns:**

| Column | Ghi chú |
|---|---|
| `user_id` | Primary key |
| `username` | Tên đăng nhập |
| `email` | Email |
| `password_hash` | Mật khẩu đã băm (bcryptjs) — không lưu plaintext |
| `role_id` | Foreign key → `ROLE.role_id` |
| `status` | Trạng thái tài khoản (khóa/mở — giá trị cụ thể: TODO) |
| `created_at` | Thời điểm tạo |
| `updated_at` | Thời điểm cập nhật gần nhất |

- **Primary key:** `user_id`
- **Foreign keys:** `role_id` → `ROLE.role_id`
- **Relationships:** `USER → DEVICE` (1:N, qua `assigned_user_id`), `USER → COMMAND` (1:N), `USER → ALERT` (1:N, qua `acknowledged_by`/`resolved_by`), `USER → AUDIT_LOG` (1:N)
- **Constraints:** TODO/UNDEFINED (UNIQUE cho `username`/`email`, độ dài `password_hash`, giá trị enum cho `status`... chưa xác định)

## 6. DEVICE

- **Table name:** `DEVICE`
- **Purpose:** Lưu thông tin và trạng thái từng thiết bị ESP32 trong hệ thống.
- **Columns:**

| Column | Ghi chú |
|---|---|
| `device_id` | Primary key |
| `device_name` | Tên thiết bị |
| `location` | Vị trí lắp đặt |
| `assigned_user_id` | Foreign key → `USER.user_id` (người phụ trách/gán thiết bị) |
| `device_status` | Trạng thái hoạt động hiện tại: ONLINE / OFFLINE / FAULT |
| `lifecycle_status` | Trạng thái vòng đời: REGISTERED / PROVISIONED / ACTIVE / MAINTENANCE / DECOMMISSIONED (xem `ARCHITECTURE.md`) |
| `firmware_version` | Phiên bản firmware hiện tại |
| `credential_hash` | Credential/khóa xác thực của thiết bị (đã băm) |
| `adaptive_config` | Cấu hình Adaptive Lighting (kiểu JSONB) |
| `last_seen_at` | Thời điểm cuối cùng nhận được tín hiệu từ thiết bị |
| `created_at` | Thời điểm tạo |
| `updated_at` | Thời điểm cập nhật gần nhất |

- **Primary key:** `device_id`
- **Foreign keys:** `assigned_user_id` → `USER.user_id`
- **Relationships:** `DEVICE → TELEMETRY` (1:N), `DEVICE → COMMAND` (1:N), `DEVICE → ALERT` (1:N), `DEVICE → AUDIT_LOG` (1:N)
- **Constraints:** TODO/UNDEFINED (`device_status`/`lifecycle_status` nên là enum/CHECK constraint theo danh sách giá trị đã biết, nhưng báo cáo chưa xác nhận cơ chế ràng buộc cụ thể ở tầng DB)

> Ghi chú quan trọng: `device_status` (ONLINE/OFFLINE/FAULT) và `lifecycle_status` (REGISTERED/PROVISIONED/ACTIVE/MAINTENANCE/DECOMMISSIONED) là **hai trường độc lập**, không được gộp chung hay nhầm lẫn.

## 7. TELEMETRY

- **Table name:** `TELEMETRY`
- **Purpose:** Lưu dữ liệu cảm biến gửi định kỳ từ thiết bị.
- **Columns:**

| Column | Ghi chú |
|---|---|
| `telemetry_id` | Primary key |
| `device_id` | Foreign key → `DEVICE.device_id` |
| `timestamp` | Thời điểm ghi nhận dữ liệu |
| `lux` | Cường độ ánh sáng |
| `motion` | Có/không phát hiện chuyển động |
| `voltage` | Điện áp |
| `current` | Dòng điện |
| `power` | Công suất |
| `pwm` | Giá trị PWM tại thời điểm ghi nhận |

- **Primary key:** `telemetry_id`
- **Foreign keys:** `device_id` → `DEVICE.device_id`
- **Relationships:** thuộc về 1 `DEVICE`
- **Constraints:** TODO/UNDEFINED (index theo `device_id` + `timestamp` để truy vấn lịch sử hiệu quả — hợp lý về mặt kỹ thuật nhưng chưa được báo cáo xác nhận, cần nhóm quyết định)

## 8. COMMAND

- **Table name:** `COMMAND`
- **Purpose:** Lưu lệnh điều khiển gửi tới thiết bị và kết quả thực thi.
- **Columns:**

| Column | Ghi chú |
|---|---|
| `command_id` | Primary key |
| `device_id` | Foreign key → `DEVICE.device_id` |
| `user_id` | Foreign key → `USER.user_id` (người gửi lệnh) |
| `command_type` | Loại lệnh (ví dụ: PWM...) |
| `command_value` | Giá trị đi kèm |
| `status` | Trạng thái lệnh (ví dụ ACKNOWLEDGED — danh sách đầy đủ: TODO) |
| `sent_at` | Thời điểm gửi lệnh |
| `ack_at` | Thời điểm nhận ACK |
| `result` | Kết quả thực thi |

- **Primary key:** `command_id`
- **Foreign keys:** `device_id` → `DEVICE.device_id`, `user_id` → `USER.user_id`
- **Relationships:** thuộc về 1 `DEVICE` và 1 `USER`
- **Constraints:** TODO/UNDEFINED

## 9. ALERT

- **Table name:** `ALERT`
- **Purpose:** Lưu cảnh báo phát sinh trong hệ thống (ví dụ LAMP_FAULT, DEVICE_OFFLINE).
- **Columns:**

| Column | Ghi chú |
|---|---|
| `alert_id` | Primary key |
| `device_id` | Foreign key → `DEVICE.device_id` |
| `alert_type` | Loại cảnh báo (ví dụ: LAMP_FAULT, DEVICE_OFFLINE) |
| `severity` | Mức độ nghiêm trọng (giá trị cụ thể: TODO) |
| `message` | Nội dung cảnh báo |
| `status` | Trạng thái xử lý cảnh báo (giá trị cụ thể: TODO) |
| `created_at` | Thời điểm tạo cảnh báo |
| `acknowledged_at` | Thời điểm được xác nhận |
| `resolved_at` | Thời điểm được giải quyết |
| `acknowledged_by` | Foreign key → `USER.user_id` |
| `resolved_by` | Foreign key → `USER.user_id` |

- **Primary key:** `alert_id`
- **Foreign keys:** `device_id` → `DEVICE.device_id`; `acknowledged_by` → `USER.user_id`; `resolved_by` → `USER.user_id`
- **Relationships:** thuộc về 1 `DEVICE`; liên kết tới `USER` qua `acknowledged_by`/`resolved_by` (1:N mỗi chiều)
- **Constraints:** TODO/UNDEFINED (danh sách giá trị hợp lệ của `alert_type`, `severity`, `status` chưa được báo cáo liệt kê đầy đủ)

## 10. AUDIT_LOG

- **Table name:** `AUDIT_LOG`
- **Purpose:** Ghi lại các thao tác quan trọng trong hệ thống phục vụ truy vết.
- **Columns:**

| Column | Ghi chú |
|---|---|
| `log_id` | Primary key |
| `user_id` | Foreign key → `USER.user_id` |
| `device_id` | Foreign key → `DEVICE.device_id` (nếu liên quan tới thiết bị) |
| `action` | Hành động được thực hiện |
| `description` | Mô tả chi tiết |
| `ip_address` | Địa chỉ IP thực hiện hành động |
| `created_at` | Thời điểm ghi log |

- **Primary key:** `log_id`
- **Foreign keys:** `user_id` → `USER.user_id`; `device_id` → `DEVICE.device_id`
- **Relationships:** thuộc về 1 `USER` và (tùy chọn) 1 `DEVICE`
- **Constraints:** TODO/UNDEFINED (liệu `device_id` có bắt buộc NOT NULL hay optional — chưa xác định)

## 11. Relationships (tổng hợp)

| Quan hệ | Cardinality |
|---|---|
| ROLE → USER | 1:N |
| USER → DEVICE | 1:N (qua `assigned_user_id`) |
| DEVICE → TELEMETRY | 1:N |
| DEVICE → COMMAND | 1:N |
| USER → COMMAND | 1:N |
| DEVICE → ALERT | 1:N |
| USER → ALERT | 1:N (qua `acknowledged_by` / `resolved_by`) |
| USER → AUDIT_LOG | 1:N |
| DEVICE → AUDIT_LOG | 1:N |

## 12. ERD

```mermaid
erDiagram
    ROLE ||--o{ USER : has
    USER ||--o{ DEVICE : "assigned_user_id"
    DEVICE ||--o{ TELEMETRY : generates
    DEVICE ||--o{ COMMAND : receives
    USER ||--o{ COMMAND : sends
    DEVICE ||--o{ ALERT : triggers
    USER ||--o{ ALERT : "acknowledged_by / resolved_by"
    USER ||--o{ AUDIT_LOG : performs
    DEVICE ||--o{ AUDIT_LOG : "related to"

    ROLE {
        pk role_id
        role_name
        description
    }
    USER {
        pk user_id
        username
        email
        password_hash
        fk role_id
        status
        created_at
        updated_at
    }
    DEVICE {
        pk device_id
        device_name
        location
        fk assigned_user_id
        device_status
        lifecycle_status
        firmware_version
        credential_hash
        adaptive_config
        last_seen_at
        created_at
        updated_at
    }
    TELEMETRY {
        pk telemetry_id
        fk device_id
        timestamp
        lux
        motion
        voltage
        current
        power
        pwm
    }
    COMMAND {
        pk command_id
        fk device_id
        fk user_id
        command_type
        command_value
        status
        sent_at
        ack_at
        result
    }
    ALERT {
        pk alert_id
        fk device_id
        alert_type
        severity
        message
        status
        created_at
        acknowledged_at
        resolved_at
        fk acknowledged_by
        fk resolved_by
    }
    AUDIT_LOG {
        pk log_id
        fk user_id
        fk device_id
        action
        description
        ip_address
        created_at
    }
```

> Không tự thêm bảng ngoài 7 entity trên. Data type SQL cụ thể, index, cascade behavior (ON DELETE/ON UPDATE) và các constraint chi tiết đều **chưa được báo cáo gốc xác định** — mỗi mục trong bảng này đã được đánh dấu `TODO/UNDEFINED` tương ứng, cần nhóm bổ sung khi thiết kế migration thực tế.
