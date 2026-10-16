# Ubiquitous Domain Glossary — HopHo (Họp Hộ)

**Tầng tài liệu**: Business Layer (`docs/business/glossary.md`)  
**Phiên bản**: 1.0  
**Ngày cập nhật**: 2026-10-15  
**Nguồn tham chiếu**: `docs/project-idea/requirements.md`, `docs/plan/mvp-plan.md`  

---

## 1. Thuật ngữ miền nghiệp vụ & Khái niệm sản phẩm

| Thuật ngữ | Khái niệm & Định nghĩa | Ngữ cảnh sử dụng trong HopHo |
|---|---|---|
| **HopHo (Họp Hộ)** | Tên sản phẩm chính thức. Ứng dụng trợ lý AI họp hành cục bộ (Local-first) trên máy tính người dùng. | Toàn hệ thống |
| **Local-First** | Nguyên tắc thiết kế ưu tiên toàn bộ dữ liệu (audio, văn bản, cấu hình) và tính toán AI diễn ra trên máy cá nhân của người dùng, không phụ thuộc vào máy chủ trung tâm. | Triết lý kiến trúc cốt lõi |
| **Realtime Transcript** | Luồng phiên âm trực tiếp dạng nháp (draft) được hiển thị theo thời gian thực trong khi cuộc họp đang diễn ra. | Chế độ họp trực tiếp |
| **Batch Reprocessing (Chạy lại sau họp)** | Quá trình sử dụng mô hình nhận diện giọng nói lớn hơn, chính xác hơn để xử lý lại toàn bộ file ghi âm sau khi cuộc họp kết thúc. | Xử lý sau cuộc họp |
| **Speaker Diarization** | Kỹ thuật AI phân tách và nhận dạng danh tính các người nói khác nhau ("ai nói gì vào thời điểm nào"). Trong HopHo, tính năng này chỉ chạy ngoại tuyến sau họp. | Xử lý sau cuộc họp |
| **Code-switching** | Hiện tượng chuyển đổi hoặc nói trộn giữa các ngôn ngữ trong cùng một câu/cuộc hội thoại (đặc biệt phổ biến là trộn tiếng Việt và tiếng Anh). | Nhận diện giọng nói STT |
| **Overlay Always-on-top** | Cửa sổ giao diện bán trong suốt luôn nổi trên cùng màn hình, hiển thị phụ đề hoặc bản dịch mà không làm gián đoạn cửa sổ ứng dụng họp. | Trải nghiệm trong lúc họp |
| **Workspace Glossary** | Từ điển chuyên ngành và danh sách tên riêng được người dùng cấu hình theo từng không gian làm việc để định hướng mô hình STT và LLM nhận diện chính xác. | Không gian làm việc |
| **Action Items** | Danh sách các đầu việc phát sinh từ cuộc họp, bao gồm nội dung việc, người phụ trách (assignee) và hạn chót (deadline). | Kết quả xuất từ LLM |
| **Uploaded Minutes Template** | Mẫu biên bản họp thực tế (định dạng DOCX, Markdown, PDF) do người dùng tải lên để ứng dụng trích xuất cấu trúc bố cục và điền nội dung tương ứng. | Mẫu biên bản cuộc họp |

---

## 2. Thuật ngữ Kỹ thuật & Hạ tầng tích hợp

| Thuật ngữ | Định nghĩa kỹ thuật | Vai trò trong hệ thống HopHo |
|---|---|---|
| **WASAPI Loopback** | Windows Audio Session API ở chế độ Loopback, cho phép ghi lại âm thanh phát ra từ các ứng dụng khác trên máy tính (như Zoom, Google Meet) mà không cần cài đặt Virtual Audio Cable. | Audio Capture trên Windows |
| **Opus Chunks** | Các đoạn tệp âm thanh được nén theo định dạng Opus và ghi tuần tự liên tục xuống đĩa cứng mỗi vài giây. Đảm bảo nếu ứng dụng bị crash thì toàn bộ âm thanh trước đó không bị mất. | Lưu trữ âm thanh an toàn |
| **Python Sidecar** | Tiến trình chạy ngầm bằng ngôn ngữ Python được đóng gói độc lập, chịu trách nhiệm thực thi các mô hình AI (STT, Diarization, LLM, Dịch). Giao tiếp với ứng dụng chính qua WebSocket nội bộ. | Hạ tầng AI Runtime |
| **MCP (Model Context Protocol)** | Giao thức tiêu chuẩn mở cho phép các trợ lý AI bên ngoài (Claude Desktop, Claude Code, Codex, Antigravity) kết nối và truy xuất dữ liệu an toàn từ HopHo. | Tích hợp Agent bên ngoài |
| **Agent CLI** | Cơ chế gọi các công cụ dòng lệnh của AI Agent (như `claude -p`, `codex exec`, `agy`) ở chế độ headless (không giao diện) để tận dụng gói tài khoản và mô hình sẵn có của người dùng. | Provider xử lý LLM |
| **WER (Word Error Rate)** | Tỷ lệ lỗi từ, chỉ số tiêu chuẩn đánh giá độ chính xác của hệ thống nhận dạng tiếng nói (STT). WER càng thấp thì bản phiên âm càng chính xác. | Benchmark & Chất lượng STT |
| **RTF (Real-Time Factor)** | Tỷ số giữa thời gian xử lý âm thanh trên tổng thời lượng của đoạn âm thanh đó. RTF <= 1 nghĩa là xử lý nhanh hơn hoặc bằng thời gian phát thực tế. | Hiệu năng xử lý sau họp |
| **Hardware Tiers (Lite / Standard / Pro)** | Phân hạng cấu hình phần cứng do hệ thống đề xuất: Lite (máy yếu Gen8/9), Standard (máy chuẩn Gen10 Only CPU), Pro (máy có Apple Silicon hoặc NVIDIA GPU). | Onboarding & Quản trị model |
