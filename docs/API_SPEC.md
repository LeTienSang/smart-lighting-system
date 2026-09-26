# API_SPEC — Smart Lighting System

> Đây là **communication contract** của hệ thống. Agent khi viết frontend/backend/firmware phải tham khảo file này. Không tự ý thay đổi endpoint, topic hoặc payload structure nếu không có chỉ dẫn rõ ràng — mọi thay đổi phải được phản ánh ngược lại vào file này (xem `PROJECT-RULES.md` mục Documentation Rules).

---

## Phần A — REST API

Quy ước chung: "Quyền" (Authorization) lấy theo bảng mục 3.3.3 của báo cáo gốc. "All" nghĩa là mọi user đã đăng nhập (Admin/Operator/Viewer) đều được gọi.

### A.1 Authentication

#### `POST /auth/login`
- **Authentication:** Public (không cần token)
- **Authorization:** Public
- **Purpose:** Đăng nhập
- **Request:** TODO — schema chưa được báo cáo xác định (suy đoán hợp lý: username/email + password, nhưng chưa xác nhận)
- **Response:** TODO
- **Error cases:** TODO

#### `POST /auth/logout`
- **Authentication:** Authenticated (yêu cầu token hợp lệ)
- **Authorization:** Tất cả vai trò
- **Purpose:** Đăng xuất
- **Request:** TODO
- **Response:** TODO
- **Error cases:** TODO

#### `PUT /auth/password`
- **Authentication:** Authenticated
- **Authorization:** Tất cả vai trò (trên tài khoản của chính mình)
- **Purpose:** Đổi mật khẩu
- **Request:** TODO
- **Response:** TODO
- **Error cases:** TODO

### A.2 Users

#### `GET /users`
- **Authentication:** Authenticated
- **Authorization:** Admin
- **Purpose:** Danh sách User
- **Request:** TODO
- **Response:** TODO
- **Error cases:** TODO

#### `POST /users`
- **Authentication:** Authenticated
- **Authorization:** Admin
- **Purpose:** Tạo User
- **Request:** TODO
- **Response:** TODO
- **Error cases:** TODO

#### `PUT /users/:id`
- **Authentication:** Authenticated
- **Authorization:** Admin
- **Purpose:** Cập nhật User
- **Request:** TODO
- **Response:** TODO
- **Error cases:** TODO

#### `PATCH /users/:id/status`
- **Authentication:** Authenticated
- **Authorization:** Admin
- **Purpose:** Khóa/mở User
- **Request:** TODO
- **Response:** TODO
- **Error cases:** TODO

### A.3 Devices

#### `GET /devices`
- **Authentication:** Authenticated
- **Authorization:** All
- **Purpose:** Danh sách thiết bị
- **Request:** TODO
- **Response:** TODO
- **Error cases:** TODO

#### `POST /devices`
- **Authentication:** Authenticated
- **Authorization:** Admin
- **Purpose:** Onboarding thiết bị
- **Request:** TODO
- **Response:** TODO
- **Error cases:** TODO

#### `GET /devices/:id`
- **Authentication:** Authenticated
- **Authorization:** All
- **Purpose:** Chi tiết thiết bị
- **Request:** TODO
- **Response:** TODO
- **Error cases:** TODO

#### `PUT /devices/:id`
- **Authentication:** Authenticated
- **Authorization:** Admin
- **Purpose:** Cập nhật thiết bị
- **Request:** TODO
- **Response:** TODO
- **Error cases:** TODO

#### `PATCH /devices/:id/lifecycle`
- **Authentication:** Authenticated
- **Authorization:** Admin
- **Purpose:** Đổi trạng thái vòng đời (`lifecycle_status`) — xem `ARCHITECTURE.md` mục Device Lifecycle
- **Request:** TODO — có khả năng chứa trường trạng thái mới (chưa xác nhận)
- **Response:** TODO
- **Error cases:** TODO (ví dụ: chuyển trạng thái không hợp lệ — chưa xác định trong báo cáo)

#### `PATCH /devices/:id/config`
- **Authentication:** Authenticated
- **Authorization:** Admin
- **Purpose:** Cấu hình Adaptive Lighting (`adaptive_config`)
- **Request:** TODO — có khả năng chứa các ngưỡng lux/PWM (xem `ARCHITECTURE.md` mục Adaptive Lighting), nhưng schema chính xác chưa được báo cáo xác định
- **Response:** TODO
- **Error cases:** TODO

### A.4 Telemetry

#### `GET /devices/:id/telemetry/latest`
- **Authentication:** Authenticated
- **Authorization:** All
- **Purpose:** Telemetry mới nhất
- **Request:** TODO
- **Response:** TODO — có khả năng trả về cùng các field như Telemetry Payload (Phần B), chưa xác nhận
- **Error cases:** TODO

#### `GET /devices/:id/telemetry`
- **Authentication:** Authenticated
- **Authorization:** All
- **Purpose:** Lịch sử telemetry
- **Request:** TODO — có khả năng hỗ trợ filter theo khoảng thời gian, chưa xác nhận
- **Response:** TODO
- **Error cases:** TODO

### A.5 Commands

#### `POST /devices/:id/commands`
- **Authentication:** Authenticated
- **Authorization:** Admin/Operator
- **Purpose:** Điều khiển thiết bị (gửi command, ví dụ ON/OFF/PWM)
- **Request:** TODO — có khả năng tương ứng với Command Payload (Phần B), chưa xác nhận cấu trúc body REST chính xác
- **Response:** TODO
- **Error cases:** TODO (ví dụ: Viewer gọi endpoint này phải bị từ chối — xem test case TC09 trong `PRD.md`)

### A.6 Alerts

#### `GET /alerts`
- **Authentication:** Authenticated
- **Authorization:** All
- **Purpose:** Danh sách cảnh báo
- **Request:** TODO
- **Response:** TODO
- **Error cases:** TODO

#### `PATCH /alerts/:id/acknowledge`
- **Authentication:** Authenticated
- **Authorization:** Admin/Operator
- **Purpose:** Xác nhận cảnh báo
- **Request:** TODO
- **Response:** TODO
- **Error cases:** TODO

#### `PATCH /alerts/:id/resolve`
- **Authentication:** Authenticated
- **Authorization:** Admin/Operator
- **Purpose:** Xử lý (giải quyết) cảnh báo
- **Request:** TODO
- **Response:** TODO
- **Error cases:** TODO

### A.7 Dashboard

#### `GET /dashboard/summary`
- **Authentication:** Authenticated
- **Authorization:** All
- **Purpose:** Tổng quan dashboard
- **Request:** TODO
- **Response:** TODO
- **Error cases:** TODO

#### `GET /dashboard/devices`
- **Authentication:** Authenticated
- **Authorization:** All
- **Purpose:** Trạng thái thiết bị (dùng cho dashboard)
- **Request:** TODO
- **Response:** TODO
- **Error cases:** TODO

### A.8 Audit

#### `GET /audit-logs`
- **Authentication:** Authenticated
- **Authorization:** Admin
- **Purpose:** Xem audit log
- **Request:** TODO
- **Response:** TODO
- **Error cases:** TODO

---

## Phần B — MQTT

### B.1 Topic Structure

| Topic | Chiều | Mục đích |
|---|---|---|
| `iot/device/{device_id}/telemetry` | ESP32 → Backend | Dữ liệu cảm biến |
| `iot/device/{device_id}/status` | ESP32 → Backend | Heartbeat / trạng thái online |
| `iot/device/{device_id}/command` | Backend → ESP32 | Lệnh điều khiển |
| `iot/device/{device_id}/ack` | ESP32 → Backend | ACK phản hồi lệnh |
| `iot/device/{device_id}/config` | Backend → ESP32 | Cấu hình (Adaptive Lighting) |

### B.2 Chi tiết từng topic

#### `iot/device/{device_id}/telemetry`
- **Publisher:** ESP32
- **Subscriber:** Backend
- **Direction:** ESP32 → Backend
- **Purpose:** Gửi dữ liệu cảm biến định kỳ (lux, motion, điện năng, pwm hiện tại).
- **Payload:** xem mục B.3.

#### `iot/device/{device_id}/status`
- **Publisher:** ESP32
- **Subscriber:** Backend
- **Direction:** ESP32 → Backend
- **Purpose:** Heartbeat / báo trạng thái online, dùng để backend phát hiện `DEVICE_OFFLINE` khi timeout.
- **Payload:** TODO — cấu trúc payload cụ thể chưa được báo cáo xác định.

#### `iot/device/{device_id}/command`
- **Publisher:** Backend
- **Subscriber:** ESP32
- **Direction:** Backend → ESP32
- **Purpose:** Gửi lệnh điều khiển (bật/tắt, PWM) xuống thiết bị.
- **Payload:** xem mục B.4.

#### `iot/device/{device_id}/ack`
- **Publisher:** ESP32
- **Subscriber:** Backend
- **Direction:** ESP32 → Backend
- **Purpose:** Phản hồi xác nhận đã thực hiện lệnh.
- **Payload:** xem mục B.5.

#### `iot/device/{device_id}/config`
- **Publisher:** Backend
- **Subscriber:** ESP32
- **Direction:** Backend → ESP32
- **Purpose:** Đẩy cấu hình Adaptive Lighting (`adaptive_config`) xuống thiết bị.
- **Payload:** TODO — cấu trúc payload cụ thể chưa được báo cáo xác định (khả năng chứa các ngưỡng lux/PWM tương ứng bảng trong `ARCHITECTURE.md`, chưa xác nhận).

### B.3 Telemetry Payload

Các field đã xác định (theo ví dụ trong báo cáo gốc, mục 3.2.2):

```json
{
  "device_id": "LIGHT001",
  "timestamp": "...",
  "lux": 185.2,
  "motion": true,
  "voltage": 12.1,
  "current": 0.72,
  "power": 8.71,
  "pwm": 65
}
```

| Field | Mô tả |
|---|---|
| `device_id` | Định danh thiết bị |
| `timestamp` | Thời điểm ghi nhận |
| `lux` | Cường độ ánh sáng (từ BH1750) |
| `motion` | Có/không phát hiện chuyển động (từ RCWL-0516) |
| `voltage` | Điện áp đo được (từ INA219) |
| `current` | Dòng điện đo được (từ INA219) |
| `power` | Công suất tiêu thụ (từ INA219) |
| `pwm` | Giá trị PWM hiện tại đang áp cho LED |

### B.4 Command Payload

```json
{
  "command_id": "uuid",
  "command_type": "PWM",
  "command_value": 70,
  "timestamp": "..."
}
```

| Field | Mô tả |
|---|---|
| `command_id` | UUID định danh lệnh |
| `command_type` | Loại lệnh (ví dụ: `PWM` — các giá trị khác như `ON`/`OFF`: chưa liệt kê đầy đủ trong báo cáo, `TODO` xác nhận danh sách đầy đủ) |
| `command_value` | Giá trị đi kèm lệnh (ví dụ giá trị PWM) |
| `timestamp` | Thời điểm gửi lệnh |

### B.5 ACK Payload

```json
{
  "command_id": "uuid",
  "status": "ACKNOWLEDGED",
  "timestamp": "...",
  "result": "PWM_SET"
}
```

| Field | Mô tả |
|---|---|
| `command_id` | UUID của lệnh được xác nhận (khớp với Command Payload) |
| `status` | Trạng thái ACK (ví dụ: `ACKNOWLEDGED` — các giá trị khác chưa liệt kê đầy đủ, `TODO`) |
| `timestamp` | Thời điểm ACK |
| `result` | Kết quả thực thi (ví dụ: `PWM_SET`) |

### B.6 Các thông số vận hành MQTT chưa xác định

Các mục sau **không được tự quyết định** khi implement — báo cáo gốc chưa xác định (mục 3.2.8 "Retry, Timeout và xử lý bản tin trùng" chỉ có tiêu đề, không có nội dung):

- QoS (QoS level cho từng topic): `TODO/UNDEFINED`
- Retain flag: `TODO/UNDEFINED`
- Message expiry: `TODO/UNDEFINED`
- Retry policy: `TODO/UNDEFINED`
- Timeout (heartbeat timeout để xác định `DEVICE_OFFLINE`): `TODO/UNDEFINED`
- Xử lý bản tin trùng (deduplication): `TODO/UNDEFINED`

Phụ lục C của báo cáo gốc ("API / MQTT payload đầy đủ") cũng được đánh dấu "bổ sung sau khi code thực tế" — nghĩa là các payload trên là thiết kế ban đầu, có thể còn thay đổi khi code thực tế hoàn thiện. Khi có thay đổi, cập nhật lại file này (xem `PROJECT-RULES.md`).
