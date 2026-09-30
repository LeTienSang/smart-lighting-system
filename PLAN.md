# PLAN — Tiến độ triển khai Smart Lighting System

> **File này chỉ ghi nhận tiến độ, không phải nguồn xác nhận trạng thái.** Theo đúng thứ tự ưu tiên "Source of Truth" trong `AGENTS.md`, file này đứng **dưới** cả code hiện có lẫn tài liệu trong `docs/`. Một dòng đánh dấu `Done` phải đi kèm bằng chứng cụ thể (đường dẫn file/commit, hoặc kết quả chạy test) — không tự đánh dấu `Done` chỉ vì đã yêu cầu agent code việc đó, hoặc vì agent tự báo cáo đã xong. Nếu nghi ngờ, coi như `In progress` và tự kiểm tra lại code/file thật trước khi đổi trạng thái.
>
> File này không lặp lại nội dung contract (schema, endpoint, payload...) — mọi chi tiết kỹ thuật vẫn nằm trong `docs/`. Ở đây chỉ trỏ link và ghi trạng thái.
>
> **Nguyên tắc cấu trúc:** giữ nguyên **9 giai đoạn** và thứ tự hiện tại. Các dependency hoặc quyết định bổ sung chỉ được thể hiện trong Definition of Done hoặc Known Open Items, không tự tạo thêm giai đoạn mới.

---

## 1. Bảng giai đoạn

| # | Giai đoạn                                              | Tài liệu tham chiếu                                                                       | Status      |
| - | ------------------------------------------------------ | ----------------------------------------------------------------------------------------- | ----------- |
| 1 | Database & Schema                                      | `docs/DATABASE.md`                                                                        | Not started |
| 2 | Backend Core (Auth, RBAC)                              | `docs/API_SPEC.md` Phần A.1–A.2, `docs/PROJECT-RULES.md` mục 2, 8                         | Not started |
| 3 | MQTT Integration                                       | `docs/API_SPEC.md` Phần B, `docs/ARCHITECTURE.md` mục 6.3, 7                              | Not started |
| 4 | REST API (Devices, Commands, Alerts, Dashboard, Audit) | `docs/API_SPEC.md` Phần A.3–A.8                                                           | Not started |
| 5 | Socket.IO Realtime                                     | `docs/API_SPEC.md` Phần C                                                                 | Not started |
| 6 | Frontend Core (Auth, Dashboard, Control)               | `docs/PROJECT-RULES.md` mục 3                                                             | Not started |
| 7 | Firmware ESP32                                         | `docs/API_SPEC.md` Phần B, `docs/ARCHITECTURE.md` mục 4, 7, `docs/PROJECT-RULES.md` mục 4 | Not started |
| 8 | Deployment (Docker Compose)                            | `docs/ARCHITECTURE.md` mục 3, `.env.example`                                              | Not started |
| 9 | Testing & Integration                                  | `docs/PRD.md` mục Acceptance Criteria, `docs/PROJECT-RULES.md` mục 10                     | Not started |

Trạng thái hợp lệ: `Not started` / `In progress` / `Done`. Không dùng các nhãn khác (ví dụ "Mostly done", "90%") — nếu chưa đạt đủ điều kiện `Done` ở mục 2 bên dưới thì vẫn là `In progress`.

---

## 2. Definition of Done cho từng giai đoạn

### Giai đoạn 1 — Database & Schema

Done khi:

- Migration đã chạy thành công trên môi trường local (`docker-compose up` + migration script không lỗi).
- Đủ 7 bảng đúng tên, đúng cột như `docs/DATABASE.md`, không thừa/thiếu bảng.
- Seed data: 3 role (`ADMIN`/`OPERATOR`/`VIEWER`) + 1 user admin mặc định đã insert được và query lại thấy đúng.
- Password của user admin seed phải được cung cấp qua environment (`SEED_ADMIN_PASSWORD`) hoặc cơ chế local-only tương đương; không hard-code secret trong migration/seed.
- Đã tự chạy thử ít nhất 1 câu lệnh vi phạm CHECK constraint (ví dụ insert `command_type='PWM'` với `command_value=NULL`) và xác nhận bị DB từ chối đúng như thiết kế ở mục E (CHECK liên trường).

### Giai đoạn 2 — Backend Core

Done khi:

- `POST /auth/login`, `POST /auth/refresh`, `POST /auth/logout`, `PUT /auth/password` chạy được qua Postman/curl, trả đúng status code như spec.
- RBAC middleware: tự test bằng tài khoản Viewer gọi một API chỉ-Admin (ví dụ `POST /users`) và xác nhận bị `403`.
- JWT dùng đúng `HS256`, claims đúng `{sub, role, iat, exp}` (xem `docs/PROJECT-RULES.md` mục 8) — tự decode 1 token thật để xác nhận.

### Giai đoạn 3 — MQTT Integration

Done khi:

- Backend kết nối được tới Mosquitto (log connect thành công).
- Có thể tự publish một bản tin telemetry giả (qua `mosquitto_pub` hoặc script) và thấy backend nhận + lưu vào bảng TELEMETRY.
- Retry/dedup logic đã tự test với ít nhất 1 kịch bản: publish trùng `command_id` và xác nhận ESP32/mock không thực thi lại lần hai.
- **Cơ chế xác thực MQTT cho ESP32 (`device_key` ↔ `credential_hash`) phải được quyết định và kiểm chứng đủ để coi Giai đoạn 3 là `Done`.** Việc dựng/test Backend–Mosquitto connectivity có thể thực hiện trước bằng credential/mock phù hợp, nhưng không được đánh dấu `Done` khi cơ chế xác thực thiết bị vẫn còn `TODO/UNDEFINED`.

### Giai đoạn 4 — REST API

Done khi mỗi endpoint trong `docs/API_SPEC.md` Phần A đã:

- Có route thực tế trả đúng status code cho ít nhất 1 case thành công và 1 case lỗi đã liệt kê trong spec.
- Mỗi endpoint/feature có ít nhất 1 test case (`TCxx`) liên quan đã chạy thử thủ công ít nhất 1 lần; các TC phải được xác định theo feature/endpoint thực tế, không dùng danh sách mơ hồ hoặc bỏ sót bằng dấu `...`.
- Kết quả test liên quan trong `docs/PRD.md` mục Acceptance Criteria phải được ghi lại (không cần số liệu định lượng, chỉ cần pass/fail).

### Giai đoạn 5 — Socket.IO Realtime

Done khi:

- Cả 4 event (`telemetry:update`, `device:status`, `command:update`, `alert:new`) đã tự bắn thử và xác nhận client nhận được đúng payload.
- Đã tự test: client chưa có JWT hợp lệ bị từ chối kết nối.
- Khi client reload/reconnect, dữ liệu trạng thái hiện tại phải được lấy từ REST/Database như **source of truth**; Socket.IO chỉ dùng để cập nhật realtime, không được coi là persistence hoặc cơ chế đảm bảo client luôn nhận được mọi event.

### Giai đoạn 6 — Frontend Core

Done khi:

- Login flow chạy được end-to-end với backend thật (không phải mock).
- Dashboard hiển thị được telemetry realtime qua Socket.IO (tự quan sát số liệu thay đổi khi giả lập gửi telemetry mới).
- RBAC UI: tự đăng nhập bằng cả 3 role và xác nhận đúng menu/nút hiển thị khác nhau.

### Giai đoạn 7 — Firmware ESP32

Done khi:

- Nạp được firmware lên board thật (không phải chỉ compile thành công).
- Đọc được cả 3 cảm biến (BH1750, RCWL-0516, INA219) và publish telemetry thấy backend nhận đúng.
- Nhận lệnh PWM/ON/OFF từ backend, đèn thực sự đổi độ sáng, ACK gửi về đúng.
- Manual override hoạt động đúng như `docs/ARCHITECTURE.md` mục 7.2 (tự test bằng tay: đặt PWM thủ công, xác nhận Adaptive không ghi đè cho đến khi motion đổi).
- Mất Wi-Fi/MQTT → thiết bị tự reconnect → sau khi reconnect thành công phải publish trạng thái `ONLINE` đúng theo contract; backend phải cập nhật trạng thái thiết bị đúng.

### Giai đoạn 8 — Deployment

Done khi:

- `docker compose up` trên máy mới có Docker Engine + Docker Compose, chưa có source/dependency/runtime setup nào khác, khởi động được đủ Backend + Mosquitto + PostgreSQL + Frontend.
- Người test được phép và phải tạo `.env` từ `.env.example`, sau đó **điền các giá trị hợp lệ cho toàn bộ biến bắt buộc** (bao gồm các credential/secret cần thiết); không coi việc chỉ copy file mẫu sang `.env` là đủ.
- Không cần sửa tay source code, Dockerfile, compose file hoặc cấu hình ứng dụng sau khi checkout; chỉ cần điền `.env` theo hướng dẫn của project.

### Giai đoạn 9 — Testing & Integration

Done khi:

- Toàn bộ TC01–TC10 trong `docs/PRD.md` đã chạy ít nhất 1 lần trên hệ thống thật (không phải mock), kết quả pass/fail được ghi lại — không cần đạt 100% pass để coi là "đã chạy", nhưng phải ghi rõ TC nào fail và lý do.

---

## 3. Known open items (các điểm còn mở, chưa chặn code nhưng cần nhớ)

| Điểm                                                              | Ghi ở đâu                                                            | Ghi chú                                                                                                                    |
| ----------------------------------------------------------------- | -------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| Cơ chế xác thực MQTT cho ESP32 (`device_key` ↔ `credential_hash`) | `docs/API_SPEC.md` mục A.3, `docs/DATABASE.md` cột `credential_hash` | `TODO/UNDEFINED` — cần quyết định dùng auth plugin nào cho Mosquitto hoặc lớp trung gian, trước khi đánh dấu Giai đoạn 3/7 `Done`. |
| Scope Socket.IO (room theo device hay broadcast toàn bộ)          | `docs/API_SPEC.md` Phần C.3                                          | `TODO/UNDEFINED` — chưa cấp thiết vì chỉ có 1 thiết bị.                                                                    |
| Cơ chế xác thực Socket.IO cụ thể                                  | `docs/API_SPEC.md` Phần C                                             | `TODO/UNDEFINED` — hành vi bắt buộc là client không có JWT hợp lệ phải bị từ chối; cơ chế truyền/kiểm tra JWT cụ thể được chốt khi triển khai Giai đoạn 5. |
| Lưu `manual_override`/`last_manual_pwm` qua reboot (NVS)          | `docs/ARCHITECTURE.md` mục 7.2                                       | Hiện coi là biến RAM, mất khi ESP32 khởi động lại.                                                                         |
| Endpoint REST xem lịch sử/trạng thái command                      | `docs/API_SPEC.md` mục A.5                                           | Chưa có, không tự thêm khi chưa xác nhận.                                                                                  |
| Công thức map PWM % → duty cycle LEDC                             | `docs/DATABASE.md` mục 3                                             | Để cho tầng firmware tự quyết định khi code, miễn giữ nguyên tắc 0–100% ở tầng dữ liệu.                                    |
| Format lỗi REST chuẩn (`{code, message, details}` hay khác)       | `docs/PROJECT-RULES.md` mục 2                                        | Agent tự đề xuất khi code, phải ghi rõ trong code/PR, không coi là chuẩn chính thức cho tới khi xác nhận.                  |
| Danh sách đầy đủ `AUDIT_LOG.action` enum                          | `docs/DATABASE.md` mục AUDIT_LOG                                     | Chưa liệt kê, agent tự đề xuất khi code phần ghi log tương ứng.                                                            |
| Số liệu benchmark hiệu năng (độ trễ, tỷ lệ mất tin, uptime)       | `docs/PRD.md` mục NFR, Acceptance Criteria                           | `TODO — chờ đo thực tế`, không tự bịa số liệu ở bất kỳ giai đoạn nào.                                                      |
| Retry/timeout/dedup (5s/1s/2s/30s/60s/50 cache)                   | `docs/ARCHITECTURE.md` mục 6.3, `docs/API_SPEC.md` mục B.6           | Giá trị đề xuất ban đầu để bắt đầu code, cần hiệu chỉnh sau khi đo trên phần cứng thật — không phải số liệu đã kiểm chứng. |

---

## 4. Nhật ký cập nhật

> Mỗi khi đổi trạng thái một giai đoạn, thêm 1 dòng vào đây: ngày, giai đoạn, trạng thái mới, bằng chứng và loại bằng chứng. Không đánh dấu `Done` nếu chưa có evidence kiểm chứng thực tế.

| Ngày | Giai đoạn | Trạng thái mới | Bằng chứng | Loại evidence |
| ---- | --------- | -------------- | --------- | -------------- |
| —    | —         | —              | (chưa có mục nào) | — |
