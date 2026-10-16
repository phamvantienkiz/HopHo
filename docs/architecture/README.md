# Architecture Documentation & System Invariants — HopHo (Họp Hộ)

**Tầng tài liệu**: Architecture Layer (`docs/architecture/README.md`)  
**Hệ thống**: HopHo (Họp Hộ) Desktop Assistant  
**Phiên bản**: 1.0  
**Ngày cập nhật**: 2026-10-15  
**Trạng thái**: Living System Invariants (Single Source of Truth)  

---

## 1. Nguyên tắc cốt lõi của Tầng Kiến trúc (Living Architecture Standard)

> ⛔ **Bất biến bất di bất dịch**: Tầng Kiến trúc (`docs/architecture/`) là **Nguồn sự thật duy nhất (Single Source of Truth)** cho toàn bộ hệ sinh thái kỹ thuật của HopHo.  
> Tầng kiến trúc **TUYỆT ĐỐI KHÔNG ĐƯỢC PHÂN CHIA THEO RELEASE**. Không tồn tại `architecture-v1/` hay `architecture-release-2/`. Chỉ có DUY NHẤT một tập tài liệu kiến trúc sống, được cập nhật liên tục tại chỗ (in-place) khi hệ thống tiến hóa.

Mọi nhà phát triển và **AI Coding Agent** khi thực hiện bất kỳ thay đổi nào trong mã nguồn bắt buộc phải tuân thủ các quy tắc bất biến được định nghĩa trong các tài liệu từ `01` đến `10`.

---

## 2. Bản đồ điều hướng & Chỉ mục tài liệu (Architecture Map)

| STT | Tài liệu kiến trúc | Trọng tâm nội dung & Bất biến kỹ thuật | Liên kết |
|---|---|---|---|
| **01** | **System Context & Scope Boundary** | Ranh giới hệ thống, tác nhân (Actors), sơ đồ Use Case và quy trình nghiệp vụ cấp cao | [01-system-context.md](./01-system-context.md) |
| **02** | **High-Level Architecture** | Topo kiến trúc tổng thể, các khối thành phần, ranh giới Rust / Python / React, phân tích đánh đổi (Trade-offs) | [02-high-level-architecture.md](./02-high-level-architecture.md) |
| **03** | **Service & Process Architecture** | Kiến trúc tiến trình (Multi-process model), vòng đời Sidecar (Lifecycle), cơ chế tự tắt khi rảnh, IPC nội bộ | [03-service-architecture.md](./03-service-architecture.md) |
| **04** | **Deployment & Packaging Architecture** | Đóng gói `.msi` Windows, phân cấp phần cứng (Lite/Standard/Pro), add-on CUDA, cấu trúc thư mục data cô lập | [04-deployment-architecture.md](./04-deployment-architecture.md) |
| **05** | **Data Architecture & Storage** | Thiết kế SQLite Schema (Rust là chủ sở hữu), DDL đầy đủ, ERD quan hệ, bảng ảo FTS5, lưu trữ chunk Opus | [05-data-architecture.md](./05-data-architecture.md) |
| **06** | **AI Pipeline & Runtimes** | Pipeline STT 2 lớp (Realtime & Batch), Diarization không gated, LLM Provider, trích mẫu biên bản, quản trị model | [06-ai-architecture.md](./06-ai-architecture.md) |
| **07** | **Security & Privacy Architecture** | Triết lý Local-first, Zero Telemetry, Outbound Consent Gate, lưu khóa qua OS Keychain, bảo mật socket nội bộ | [07-security-architecture.md](./07-security-architecture.md) |
| **08** | **Integration & Communication** | Giao thức WebSocket nội bộ kèm token, máy chủ `hopho-mcp` (stdio), adapter Agent CLI headless, Cloud LLM | [08-integration-architecture.md](./08-integration-architecture.md) |
| **09** | **Design Principles & Standards** | Tiêu chuẩn mã nguồn, Clean Architecture, SOLID, xử lý lỗi thống nhất (Error Envelope), Repository pattern | [09-design-principles.md](./09-design-principles.md) |
| **10** | **Monorepo Folder Structure** | Bố cục thư mục monorepo, ranh giới trách nhiệm giữa các gói, quy ước đặt file cho Coding Agent | [10-folder-structure.md](./10-folder-structure.md) |
| **11-14** | **Frontend System & Design Language** | Kiến trúc giao diện người dùng, hệ thống component, state management, design tokens | [11-frontend-system-design.md](./11-frontend-system-design.md) |
| **ADR** | **Architecture Decision Records** | Nhật ký quyết định kiến trúc (`ADR-001` đến `ADR-NNN`) theo chuẩn Solves / Worsens / Change it when | [adr/ADR-001.md](./adr/ADR-001.md) |

---

## 3. Rào chắn bảo vệ hệ thống (System Invariants Checklist)

Khi duyệt Pull Request hoặc sinh code tự động, phải đối chiếu với các rào chắn sau:

1. **Rust sở hữu SQLite Schema**: Python Sidecar và MCP chỉ đọc/ghi thông qua package `hopho_store`. Không tạo migration bằng Python.
2. **Sidecar chạy qua Localhost có Token**: Giao tiếp nội bộ chỉ bind `127.0.0.1`, cổng ngẫu nhiên, token bảo mật sinh mới mỗi lần khởi động sidecar.
3. **Không lưu Voice Profile**: Phân tách người nói (Diarization) chỉ chạy ngoại tuyến và xóa sạch embedding sau phiên.
4. **Không thu thập dữ liệu (Zero Telemetry)**: Tuyệt đối không gắn analytics, tracking SDK hay gửi log ra ngoài.
5. **An toàn khi Crash**: Audio phải được ghi dạng Opus chunks định kỳ kèm `fsync`.
