# AGENTS.md

> Entry point dành cho AI coding agent (ví dụ: agent chạy trong VS Code).
> Đọc file này **trước tiên** trước khi thực hiện bất kỳ thay đổi code nào trong repository.

## 1. Project Overview

- **Tên project:** Smart Lighting System
- **Tên GitHub repository:** `smart-lighting-system`
- **Loại hệ thống:** Hệ thống IoT chiếu sáng thông minh, giám sát và điều khiển qua MQTT.
- **Mục tiêu:** Thu thập dữ liệu cảm biến (ánh sáng, chuyển động, điện năng) từ thiết bị ESP32, cho phép điều khiển đèn LED từ xa qua Web, tự động điều chỉnh độ sáng (Adaptive Lighting), và phát cảnh báo khi thiết bị hoạt động bất thường hoặc mất kết nối.
- **Các thành phần chính:**
  - Firmware ESP32 (cảm biến + điều khiển PWM + MQTT client)
  - MQTT Broker (Mosquitto)
  - Backend (Node.js + Express + TypeScript)
  - Database (PostgreSQL)
  - Frontend (React + TypeScript + Vite + Tailwind CSS)
  - Realtime layer (Socket.IO)

Đây là một **đồ án/prototype học phần IoT** (nhóm 2 thành viên), không phải sản phẩm thương mại hoàn chỉnh. Các thông số kỹ thuật, hợp đồng API/MQTT, ngưỡng cảnh báo và cấu trúc CSDL đã được **[ĐÃ BỔ SUNG]** chuẩn hóa đầy đủ trong toàn bộ các file tài liệu để làm căn cứ triển khai mã nguồn.

## 2. Repository Structure

Cấu trúc repository lấy đúng theo báo cáo gốc (mục 4.1.1):

```text
smart-lighting-system/
├── firmware/       # ESP32: src/{sensors, actuators, mqtt, config}, platformio.ini
├── backend/        # Express + TS: src/{controllers, services, repositories, mqtt, realtime, routes}
├── frontend/       # React + Vite: src/{components, pages, hooks, services, context}
├── database/       # migrations, seeds, schema.sql
├── deployment/     # Mosquitto config, Dockerfile, docker-compose.yml
├── hardware/       # BOM, pinout wiring diagram, ảnh prototype
├── tests/          # e2e, integration, mqtt-simulator
├── docs/           # PRD, ARCHITECTURE, API_SPEC, DATABASE, PROJECT-RULES, AGENTS
├── README.md
├── .env.example
└── docker-compose.yml
```

> **[ĐÃ BỔ SUNG]** Cấu trúc con bên trong các module chính (`backend/`, `frontend/`, `firmware/`) đã được định hình chi tiết theo kiến trúc Modular Monolith để phân tách rõ trách nhiệm giữa các layer.

## 3. Documentation Map

| File | Nội dung |
|---|---|
| `docs/PRD.md` | Requirements và chức năng (WHAT/WHY) |
| `docs/ARCHITECTURE.md` | Kiến trúc và các thành phần hệ thống (HOW ở mức tổ chức) |
| `docs/API_SPEC.md` | REST API và MQTT contract (communication contract) |
| `docs/DATABASE.md` | Database schema và relationships |
| `docs/PROJECT-RULES.md` | Quy tắc phát triển dành cho agent khi sửa code |

Agent nên đọc file tương ứng với loại task **trước khi** viết hoặc sửa code, không suy đoán từ trí nhớ.

## 4. Agent Workflow

Khi nhận một task, agent thực hiện theo thứ tự:

1. Đọc `AGENTS.md` (file này).
2. Xác định loại task (feature backend? sửa firmware? đổi API? thêm bảng DB? sửa UI?).
3. Đọc tài liệu liên quan trong `docs/` tương ứng với loại task đó.
4. Kiểm tra code hiện tại trong repo (không giả định code đã tồn tại hay đã hoạt động đúng).
5. **Không tự ý** thay đổi architecture, API contract, MQTT contract hoặc database schema nếu không được yêu cầu rõ ràng.
6. Thực hiện thay đổi tối thiểu cần thiết để hoàn thành task.
7. Kiểm tra/test phần bị ảnh hưởng sau khi thay đổi (xem `docs/PROJECT-RULES.md` mục Testing Rules).
8. Nếu thay đổi làm ảnh hưởng đến API, database hoặc architecture, **cập nhật tài liệu tương ứng** trong cùng lần thay đổi.

## 5. Source of Truth

Khi có mâu thuẫn thông tin, agent ưu tiên theo thứ tự sau:

1. **Explicit user instruction** — chỉ dẫn trực tiếp từ người dùng trong phiên làm việc hiện tại.
2. **Existing project implementation** — code hiện có trong repository.
3. **Approved project documentation** — các file trong `docs/` và `AGENTS.md`.
4. **Original project report** — báo cáo đồ án gốc (nguồn dữ liệu ban đầu để tạo ra các file docs này).
5. **Assumption** — chỉ dùng khi không có gì ở 4 mức trên, và phải nêu rõ đây là giả định.

**Nguyên tắc quan trọng:** nếu một thông tin chưa được xác định ở bất kỳ mức nào, agent **không được** tự ý biến assumption thành requirement chính thức. Phải hỏi lại người dùng hoặc đánh dấu `TODO` trong code/tài liệu.

## 6. Important Constraints

- Đây là **prototype phạm vi hẹp**: 1 node ESP32, không phải hệ thống multi-node.
- Công nghệ đã được lựa chọn cố định (xem `docs/ARCHITECTURE.md`), agent **không tự ý thay thế** (ví dụ không đổi PostgreSQL sang MongoDB, không đổi MQTT sang HTTP polling).
- Hệ thống dùng **RBAC ba vai trò**: Admin, Operator, Viewer. Mọi thay đổi liên quan đến quyền phải tuân thủ đúng ma trận quyền trong `docs/PRD.md`.
- Giao tiếp thiết bị dùng **MQTT** qua Mosquitto, theo cấu trúc topic cố định trong `docs/API_SPEC.md`.
- Thiết bị vật lý là **ESP32** với các cảm biến/cơ cấu chấp hành cố định (BH1750, RCWL-0516, INA219, IRLZ44N, LED 12V).
- Database là **PostgreSQL**, đúng 7 entity đã định nghĩa trong `docs/DATABASE.md`. Không tự thêm bảng.
- **Không tự ý mở rộng phạm vi**: các hạng mục sau nằm ngoài phạm vi hiện tại và **không được thêm vào** trừ khi có chỉ dẫn rõ ràng: AI/ML, OTA firmware, RGB lighting, báo cháy, báo trộm, camera, mobile app riêng.
- Nhiều nội dung trong báo cáo gốc còn để trống hoặc đánh dấu "bổ sung sau" (kết quả kiểm thử, số liệu độ trễ, đường dẫn GitHub, chi tiết triển khai thực tế). Các phần này được giữ nguyên là `TODO/UNDEFINED` trong toàn bộ tài liệu — agent không được tự bịa số liệu hoặc kết quả.
