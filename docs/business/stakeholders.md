# Stakeholder Register & RACI Matrix — HopHo (Họp Hộ)

**Tầng tài liệu**: Business Layer (`docs/business/stakeholders.md`)  
**Phiên bản**: 1.0  
**Ngày cập nhật**: 2026-10-15  
**Nguồn tham chiếu**: `docs/project-idea/requirements.md` §1 & §3, `docs/project-idea/idea.md`  

---

## 1. Danh mục các bên liên quan (Stakeholder Register)

### 1.1 Tác nhân nội bộ (Internal Actors & Personas)

| Mã | Vai trò / Persona | Đặc điểm & Nhu cầu chính | Mối quan tâm & Kỳ vọng |
|---|---|---|---|
| **STK-01** | **Người dùng phổ thông (Low-code / No-code)** | Nhân viên văn phòng, quản lý dự án, thư ký cuộc họp không am hiểu kỹ thuật lập trình. | Cài đặt dễ dàng (file `.msi`), giao diện trực quan, không bắt gõ lệnh dòng lệnh, không lo lỗi cài đặt Python. |
| **STK-02** | **Kỹ sư phần mềm & Power User** | Các nhà phát triển, kiến trúc sư hệ thống cần kiểm soát chi tiết phần cứng, model AI và quy trình làm việc. | Tận dụng card đồ họa rời (NVIDIA CUDA), tùy chỉnh prompt template, kết nối dữ liệu cuộc họp vào agent qua MCP server. |
| **STK-03** | **Người tham gia họp đa ngôn ngữ** | Thành viên trong các nhóm làm việc toàn cầu hoặc làm việc với đối tác quốc tế (Việt, Anh, Trung, Hàn). | Cần dịch phụ đề trực tiếp trên màn hình theo thời gian thực (Overlay), hỗ trợ tốt nói trộn tiếng Việt - Anh. |
| **STK-04** | **Quản trị viên an toàn thông tin (Security & Privacy Conscious)** | Người dùng quan ngại về bảo mật dữ liệu doanh nghiệp và bí mật công nghệ. | Cam kết 100% dữ liệu nằm trên máy cá nhân, không có bot tham gia phòng họp làm phiền người khác, không gửi dữ liệu ra ngoài khi chưa cho phép. |

### 1.2 Tác nhân & Hệ thống bên ngoài (External Systems & Systems of Engagement)

| Mã | Tên hệ thống bên ngoài | Phương thức tương tác với HopHo | Trách nhiệm tương tác |
|---|---|---|---|
| **EXT-01** | **Ứng dụng phòng họp trực tuyến (Zoom, Google Meet, Teams...)** | Thu âm qua cơ chế âm thanh hệ thống (WASAPI loopback) trên Windows. | Phát âm thanh đầu ra của cuộc họp để HopHo ghi lại mà không cần cài bot vào phòng họp. |
| **EXT-02** | **AI Agent bên ngoài (Claude Desktop, Claude Code, Codex, Antigravity)** | Kết nối qua chuẩn giao thức Model Context Protocol (`hopho-mcp`) hoặc Agent CLI headless. | Truy vấn lịch sử họp, tìm kiếm nội dung, tự động xuất action items sang các hệ thống quản trị công việc (Notion, Jira, Slack). |
| **EXT-03** | **Nhà cung cấp Cloud LLM (OpenAI-compatible endpoints)** | Kết nối bảo mật qua HTTPS API khi người dùng kích hoạt (opt-in) và nhập API Key. | Cung cấp năng lực suy luận và tóm tắt cuộc họp dài khi người dùng không muốn dùng mô hình local. |

---

## 2. Ma trận phân quyền RACI (RACI Matrix)

*Quy ước*:
- **R (Responsible)**: Người trực tiếp thực hiện công việc.
- **A (Accountable)**: Người chịu trách nhiệm phê duyệt cao nhất.
- **C (Consulted)**: Người được tham vấn ý kiến trước khi thực hiện.
- **I (Informed)**: Người được thông báo sau khi hoàn thành.

| Khối chức năng / Nhiệm vụ cốt lõi | Product Owner (PO) | Lead Architect | Desktop / Rust Dev | AI / Sidecar Dev | End User (Người dùng) |
|---|---|---|---|---|---|
| Xác định bài toán kinh doanh & Phạm vi MVP | **A** / **R** | C | I | I | C |
| Lựa chọn mô hình STT & Đánh giá Benchmark (M0) | C | **A** | I | **R** | I |
| Thiết kế kiến trúc IPC & Rust-Sidecar Lifecycle (M1) | I | **A** | **R** | **R** | I |
| Thu âm WASAPI, Ghi Opus Chunks & Phục hồi lỗi (M2) | I | A | **R** | I | I |
| Triển khai luồng STT Realtime & Batch Reprocessing (M3) | C | A | C | **R** | I |
| Triển khai Diarization & Giao diện Review sau họp (M4) | C | A | **R** | **R** | I |
| Tích hợp LLM, Dịch câu & Xử lý mẫu biên bản upload (M5) | **A** | C | I | **R** | C |
| Xây dựng máy chủ MCP Server & Tích hợp Agent CLI (M6) | C | **A** | I | **R** | I |
| Thiết kế Cửa sổ Overlay & Phím tắt toàn cục (M7) | C | A | **R** | I | C |
| Đóng gói Bộ cài `.msi`, Onboarding & Model Manager (M8) | **A** | C | **R** | **R** | C |
| Kiểm thử tải trên máy chuẩn Gen10 & Đóng gói v0.1 (M9) | **A** | A | **R** | **R** | I |
