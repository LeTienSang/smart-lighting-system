# PROJECT-RULES — Smart Lighting System

> Đây là quy tắc dành cho AI coding agent khi thực hiện thay đổi code. File này **không** mô tả requirements (xem `PRD.md` cho requirements).

## 1. General Rules

- Không tự ý mở rộng scope (không thêm AI/ML, OTA, RGB, báo cháy, báo trộm, camera, mobile app riêng — xem `PRD.md` mục Out of Scope).
- Không tự ý thay đổi architecture đã mô tả trong `ARCHITECTURE.md` (ví dụ: không cho Frontend truy cập trực tiếp PostgreSQL hoặc MQTT Broker).
- Không xóa functionality hiện có trừ khi được yêu cầu rõ ràng.
- Ưu tiên thay đổi nhỏ và có kiểm soát; tránh refactor lớn không cần thiết cho task hiện tại.
- Kiểm tra code hiện tại trước khi sửa — không giả định code đã tồn tại hoặc đã hoạt động đúng.

## 2. Backend Rules

- Sử dụng **TypeScript** và **Express** đúng theo stack đã chọn (xem `ARCHITECTURE.md`).
- API structure phải tuân thủ đúng danh sách endpoint trong `API_SPEC.md`; không tự thêm/xóa endpoint.
- Validation: input phải được kiểm tra và làm sạch trước khi xử lý (theo yêu cầu bảo mật trong báo cáo gốc, mục 3.6.1).
- Error handling: cần có xử lý lỗi rõ ràng cho các API; format lỗi cụ thể — báo cáo chưa xác định → agent đề xuất và ghi rõ trong code/PR, không tự coi là chuẩn chính thức.
- Authentication: dùng JWT để xác thực API.
- Authorization: RBAC phải được kiểm tra **tại backend/API**, không chỉ ẩn phần tử trên giao diện.
- MQTT integration: dùng MQTT.js để giao tiếp với Mosquitto; topic và payload phải đúng theo `API_SPEC.md` Phần B.
- Database access: chỉ Backend được truy cập PostgreSQL trực tiếp; không cho phép Frontend hoặc thành phần khác truy cập DB.
- Logging: audit log cho các thao tác quan trọng phải được ghi vào bảng `AUDIT_LOG` (xem `DATABASE.md`).

## 3. Frontend Rules

- Sử dụng **React + TypeScript + Vite + Tailwind CSS** đúng theo stack đã chọn.
- API integration: Frontend chỉ giao tiếp với Backend qua REST/Socket.IO, không kết nối trực tiếp PostgreSQL hoặc MQTT Broker.
- Authentication state: **[ĐÃ BỔ SUNG]** Quản lý trạng thái đăng nhập/token ở phía client: lưu Access Token trong bộ nhớ ứng dụng (React Auth Context/State), lưu Refresh Token trong HttpOnly Cookie; header mọi request bảo vệ bằng `Authorization: Bearer <access_token>`. Khi Access Token hết hạn, client tự động gọi `POST /auth/refresh` để nhận token mới.
  - *Lý do/Rationale:* Đây là kiến trúc xác thực chuẩn để lập trình Http Client Interceptor (Axios/Fetch) và React Auth Guard, giúp bảo vệ token trước nguy cơ tấn công XSS (không lưu refresh token ở localStorage).
- RBAC UI: giao diện nên ẩn/hiện chức năng theo vai trò để trải nghiệm tốt hơn, **nhưng đây chỉ là UX**, không thay thế cho kiểm tra quyền ở backend.
- Realtime: dùng Socket.IO để nhận cập nhật dữ liệu thời gian thực (dashboard, trạng thái thiết bị, telemetry mới).
- Biểu đồ dùng Recharts (theo `ARCHITECTURE.md`).

## 4. Firmware Rules

- Nền tảng: ESP32, firmware viết bằng C++ với PlatformIO/Arduino Framework.
- Sensor handling: đọc dữ liệu từ BH1750 (lux), RCWL-0516 (motion), INA219 (voltage/current/power) theo đúng vai trò đã định nghĩa trong `ARCHITECTURE.md`.
- MQTT: publish telemetry lên `iot/device/{device_id}/telemetry`, publish heartbeat/status lên `iot/device/{device_id}/status`, subscribe `iot/device/{device_id}/command` và `iot/device/{device_id}/config`, publish ACK lên `iot/device/{device_id}/ack`. Không tự đổi cấu trúc topic.
- Reconnect: firmware phải tự động kết nối lại khi mất kết nối mạng/MQTT (yêu cầu reliability trong `PRD.md`).
- Heartbeat: gửi định kỳ để backend phát hiện `DEVICE_OFFLINE` khi timeout.
- Command handling: xử lý lệnh nhận từ topic `command`, áp dụng thay đổi (ví dụ PWM), sau đó gửi ACK.
- ACK: mọi lệnh nhận được phải có phản hồi ACK tương ứng qua topic `ack`, theo đúng payload trong `API_SPEC.md`.
- PWM: Adaptive Lighting và lệnh điều khiển thủ công đều tác động lên PWM của LED qua IRLZ44N; Adaptive Lighting chạy **cục bộ trên ESP32** (không phụ thuộc backend để hoạt động).

## 5. Database Rules

- Không tự ý đổi schema (tên bảng, tên cột) đã liệt kê trong `DATABASE.md`.
- Không tạo bảng mới nếu không thực sự cần thiết và chưa được xác nhận — hiện tại chỉ có đúng 7 entity.
- Migration phải có kiểm soát (versioned migration, không sửa trực tiếp schema production).
- Không hard-code credentials trong code hoặc migration.
- Không commit secret (connection string, password) vào Git.

## 6. API Rules

- API phải tuân thủ `API_SPEC.md`.
- Không tự ý đổi endpoint (method, path).
- Không tự ý đổi request/response contract khi đã được xác định.
- Breaking change đối với contract hiện có phải được xác nhận với người dùng/nhóm trước khi thực hiện, và phải cập nhật `API_SPEC.md` cùng lúc.

## 7. MQTT Rules

- Topic phải tuân thủ đúng danh sách trong `API_SPEC.md` Phần B.1.
- Payload phải tuân thủ đúng contract trong `API_SPEC.md` Phần B.3–B.5.
- Không tự ý đổi topic hoặc payload structure.
- QoS, retain, message expiry, retry, timeout, deduplication: các thông số vận hành này đã được chuẩn hóa tại `ARCHITECTURE.md` mục 6.3 và `API_SPEC.md` mục B.6 (đánh dấu `[ĐÃ BỔ SUNG]`, kèm rationale) — implement đúng theo các giá trị đó. Đây là **giá trị đề xuất ban đầu của nhóm để có thể bắt đầu code**, chưa qua đo đạc/hiệu chỉnh thực tế trên phần cứng; nếu cần thay đổi các con số này, phải cập nhật lại đồng thời cả hai file trên, không tự ý đổi rải rác.

## 8. Security Rules

- Password hashing: dùng **bcryptjs**, không lưu plaintext.
- JWT: dùng để xác thực API.
- RBAC: kiểm tra quyền tại backend cho mọi API, theo ma trận quyền trong `PRD.md`.
- Input validation: dữ liệu đầu vào phải được kiểm tra và làm sạch.
- Device credentials: mỗi thiết bị có `device_id` và `credential_hash` riêng, không dùng chung credential.
- Environment variables: secret/credential lưu qua biến môi trường hoặc cơ chế cấu hình an toàn.
- Secrets: không commit secret lên GitHub (bao gồm `.env`).
- Audit logs: các thao tác quan trọng phải được ghi vào `AUDIT_LOG`.
- HTTPS/TLS: sử dụng khi phù hợp với môi trường triển khai thực tế (báo cáo gốc ghi đây là điều "cân nhắc" cho môi trường thật, chưa phải yêu cầu bắt buộc ở giai đoạn prototype — xem `ARCHITECTURE.md` mục 9 và Hướng phát triển).
- **Không được tự tuyên bố** rằng một cơ chế bảo mật đã được triển khai (ví dụ "hệ thống đã có HTTPS/TLS") nếu code thực tế chưa có — phải phản ánh đúng trạng thái implementation thực tế.

## 9. Git Rules

- Không commit `.env` (chỉ dùng `.env.example` làm mẫu).
- Không commit secret dưới bất kỳ hình thức nào.
- Commit phải tập trung vào một thay đổi rõ ràng (tránh commit gộp nhiều việc không liên quan).
- Không sửa lịch sử Git (force push, rebase lịch sử đã chia sẻ...) nếu không được yêu cầu rõ ràng.
- Quy ước đặt tên commit message: **[ĐÃ BỔ SUNG]** Tuân thủ chuẩn Conventional Commits: `<type>(<scope>): <mô tả ngắn>` (Ví dụ: `feat(backend): add telemetry ingestion handler`, `fix(firmware): fix reconnection logic`, `docs(api): update MQTT payload spec`). Các type hợp lệ: `feat`, `fix`, `docs`, `refactor`, `test`, `chore`.
  - *Lý do/Rationale:* Cần thống nhất quy ước commit ngay từ đầu để giữ lịch sử Git rõ ràng và hỗ trợ sinh changelog tự động.

## 10. Testing Rules

- Agent phải kiểm tra phần bị ảnh hưởng bởi thay đổi trước khi coi task là hoàn thành.
- Bao gồm khi phù hợp:
  - Unit test cho logic nghiệp vụ (ví dụ: logic tính alert, logic Adaptive Lighting).
  - API test cho các endpoint bị ảnh hưởng.
  - MQTT test cho luồng telemetry/command/ack bị ảnh hưởng.
  - Integration test cho luồng end-to-end liên quan.
  - Security/RBAC test (ví dụ: đảm bảo Viewer bị từ chối khi gọi API điều khiển — tương ứng TC09 trong `PRD.md`).
- Không được tạo kết quả test giả hoặc báo cáo "test passed" khi chưa thực sự chạy test.
- Lưu ý: báo cáo gốc hiện **chưa có kết quả kiểm thử thực tế** (các test case TC01–TC10 và các chỉ tiêu phi chức năng trong `PRD.md` mục Acceptance Criteria đều đang ở trạng thái "chưa đo/chưa chạy"). Agent không được điền số liệu hoặc kết quả thay cho nhóm.

## 11. Documentation Rules

Nếu thay đổi:

- **API** → cập nhật `API_SPEC.md`.
- **Database** → cập nhật `DATABASE.md`.
- **Architecture** → cập nhật `ARCHITECTURE.md`.
- **Requirement** → cập nhật `PRD.md`.

Thay đổi code và cập nhật tài liệu tương ứng nên nằm trong cùng một thay đổi (cùng PR/commit liên quan), để tránh tài liệu bị lệch khỏi thực tế implementation.
