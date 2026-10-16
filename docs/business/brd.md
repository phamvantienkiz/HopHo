# Business Requirement Document (BRD) — HopHo (Họp Hộ)

**Tầng tài liệu**: Business Layer (`docs/business/brd.md`)  
**Dự án**: HopHo (Họp Hộ) — "Để AI họp hộ bạn."  
**Phiên bản**: 1.0  
**Ngày cập nhật**: 2026-10-15  
**Trạng thái**: Draft — chờ Product Owner duyệt  
**Nguồn tham chiếu**: `docs/project-idea/requirements.md`, `docs/project-idea/idea.md`  

---

## 1. Tầm nhìn & Tuyên bố sứ mệnh (Vision & Mission)

### 1.1 Tuyên bố sứ mệnh
HopHo là ứng dụng desktop trợ lý cuộc họp cá nhân thế hệ mới, vận hành theo triết lý **Local-first**, hỗ trợ người dùng ghi âm, phiên dịch trực tiếp và tự động hóa toàn bộ công tác tổng hợp biên bản, trích xuất hành động (action items) mà không cần bot can thiệp vào phòng họp.

> *"Để AI họp hộ bạn — An toàn dữ liệu tuyệt đối, trực tiếp trên máy tính của bạn."*

### 1.2 Bối cảnh bài toán & Nỗi đau người dùng (Problem Statement)
Trong môi trường làm việc hiện đại, các cuộc họp trực tuyến (Zoom, Google Meet, Microsoft Teams) và trực tiếp tại văn phòng diễn ra với mật độ dày đặc:
1. **Quá tải ghi chép và mất tập trung**: Người tham gia phải vừa nghe, vừa thảo luận, vừa ghi chú chép tay dẫn đến bỏ sót các quyết định hoặc hành động quan trọng.
2. **Rào cản ngôn ngữ trong cuộc họp đa quốc gia**: Các cuộc họp sử dụng tiếng Anh, Trung, Hàn hoặc giao tiếp nói trộn Việt–Anh (code-switching) gây khó khăn cho việc thấu hiểu tức thời.
3. **Mối lo ngại về quyền riêng tư & bảo mật**: Hầu hết các công cụ họp AI hiện nay đều gửi bot ảo (meeting bot) tham gia phòng họp và đẩy toàn bộ âm thanh lên máy chủ đám mây của bên thứ ba, tiềm ẩn nguy cơ lộ bí mật kinh doanh và bị người tham gia phản đối.
4. **Độ trễ và rào cản kỹ thuật**: Các giải pháp mã nguồn mở hiện có thường đòi hỏi cài đặt phức tạp (Python, CUDA, dòng lệnh), gây trở ngại lớn cho người dùng phổ thông (low-code / no-code).

---

## 2. Nguyên tắc sản phẩm cốt lõi (Core Principles)

Mọi tính năng và quyết định thiết kế của HopHo phải tuân thủ nghiêm ngặt 4 nguyên tắc:

1. **User quyết định mọi thứ (User-Driven)**: 
   - Ứng dụng không tự động ghi âm ngầm, không tự bật phiên dịch hay gửi dữ liệu lên cloud.
   - Mọi hành vi quan trọng đều phải được người dùng kích hoạt hoặc cấu hình rõ ràng trước phiên họp.
2. **Ưu tiên xử lý cục bộ (Local-First)**:
   - Mặc định 100% dữ liệu âm thanh, văn bản phiên âm (transcript) và cấu trúc lưu trữ nằm trên máy tính cá nhân.
   - Cloud LLM chỉ được kích hoạt khi người dùng chủ động chọn (opt-in) và có cảnh báo minh bạch về điểm đến dữ liệu.
3. **Tiết kiệm tài nguyên (Resource-Light when Idle)**:
   - Khi không trong phiên ghi âm, không duy trì bất kỳ tiến trình nền nặng nề nào gây hao tốn CPU/RAM.
   - Thành phần xử lý AI chỉ khởi động khi thực sự cần và tự động giải phóng tài nguyên khi rảnh rỗi.
4. **Thân thiện với người dùng phổ thông (Low-code / No-code Friendly)**:
   - Cung cấp bộ cài đặt trọn gói duy nhất trên Windows.
   - Không yêu cầu người dùng tự cài đặt môi trường Python, CUDA toolkit hay gõ lệnh terminal.

---

## 3. Mục tiêu kinh doanh & Chỉ số thành công (Business Goals & KPIs)

| Chỉ số (KPI) | Mục tiêu định lượng | Phương pháp đo lường |
|---|---|---|
| **Chất lượng phiên âm tiếng Việt (WER)** | Ngưỡng WER mục tiêu **chưa đặt cứng**; sẽ chốt sau khi có kết quả benchmark đầu tiên (M0). Trước đó WER là tiêu chí xếp hạng khi chọn model | Benchmark tự động (FLEURS + dữ liệu custom) |
| **Độ trễ phiên âm thời gian thực** | <= 5 giây từ lúc dứt câu đến khi hiển thị text trên máy chuẩn (i5 Gen10/16GB) | Đo thời gian end-to-end trong phiên realtime |
| **Tải tài nguyên khi chạy realtime** | Tải CPU <= 50% tổng năng lực CPU; RAM <= 4GB trên cấu hình chuẩn | Lấy mẫu hệ thống liên tục qua monitor tiến trình |
| **Tốc độ xử lý chuẩn sau họp** | Tổng thời gian chạy lại STT + Diarization <= 1x thời lượng cuộc họp | Đo tổng thời gian xử lý file ghi âm 1h <= 60 phút |
| **An toàn dữ liệu tuyệt đối (Zero Data Loss)** | 0% sự cố mất mát âm thanh khi ứng dụng hoặc hệ điều hành bị crash | Kiểm thử tắt ứng dụng đột ngột khi ghi âm |
| **Tiêu thụ tài nguyên khi rảnh (Idle)** | Không có tiến trình AI nền tốn tài nguyên khi không ghi âm; thành phần AI tự tắt sau thời gian chờ | Giám sát tiến trình nền khi ứng dụng ở chế độ chờ |

---

## 4. Ranh giới sản phẩm (Product Boundaries)

### 4.1 Trong phạm vi (In-Scope)
- Hỗ trợ ghi âm microphone (họp trực tiếp) và loopback âm thanh hệ thống (họp qua Zoom/Meet/Teams) không dùng bot.
- Cho phép nhập (import) các tệp âm thanh/video có sẵn để xử lý như một phiên họp.
- Kiến trúc Speech-to-Text 2 lớp: Bản nháp realtime nhẹ nhàng và bản chuẩn hóa độ chính xác cao sau cuộc họp.
- Tùy chọn dịch realtime hiển thị trên cửa sổ phụ đề nổi (Overlay always-on-top).
- Phân biệt người nói (Speaker Diarization) ngoại tuyến sau họp, cho phép đổi tên người nói hàng loạt.
- Tóm tắt cuộc họp, trích xuất việc cần làm (action items), lập biên bản theo mẫu mặc định hoặc mẫu người dùng tải lên (`.docx`, `.md`, `.pdf`).
- Tích hợp chuẩn giao thức Model Context Protocol (MCP) và Agent CLI (`claude -p`, `codex exec`, `agy`) để các trợ lý bên ngoài truy xuất lịch sử họp.
- Quản lý từ điển thuật ngữ (Glossary) theo từng không gian làm việc (Workspace) để tối ưu độ chính xác của STT và LLM.

### 4.2 Ngoài phạm vi (Out-of-Scope)
- **Tuyệt đối không phát triển bot tham gia phòng họp**: Không can thiệp API phòng họp hay gửi tài khoản bot vào Zoom/Meet/Teams.
- **Không thực hiện Diarization realtime**: Không phân tách danh tính người nói trong luồng streaming trực tiếp để bảo vệ tài nguyên máy.
- **Không lưu trữ nhận diện giọng nói (Voice Profile)** xuyên suốt giữa các cuộc họp khác nhau nhằm đảm bảo quyền riêng tư.
- **Không tự xây dựng connector trực tiếp** tới Notion, Jira, Slack, Trello: Mọi kết nối chuyển tiếp dữ liệu được giao quyền cho các Agent thông qua máy chủ MCP của HopHo.
- **Không xây dựng bộ máy tìm kiếm ngữ nghĩa (RAG) toàn cục phức tạp** bên trong ứng dụng: Ứng dụng tập trung vào tìm kiếm toàn văn (Full-Text Search), tính năng RAG mở rộng sẽ do Agent thực hiện qua MCP.
- **Không đồng bộ đám mây tự động (No Auto Cloud Sync)**.
- **Không tự động theo dõi tiến trình hệ thống để phát hiện cuộc họp đang diễn ra**.

---

## 5. Cấu hình phần cứng mục tiêu

- **Phần cứng chuẩn (Baseline đo lường NFR)**: Intel Core i5 Gen 10 / AMD tương đương, RAM 16GB, Only CPU (không GPU rời). Mọi cam kết về độ trễ và tài nguyên được định mức trên cấu hình này.
- **Phần cứng tối thiểu**: Intel Core i5 Gen 8/9 / AMD tương đương, RAM 16GB, Only CPU — Chấp nhận tốc độ xử lý chậm hơn, khuyến nghị sử dụng chế độ "Chỉ ghi âm".
- **Tăng tốc phần cứng (Tùy chọn)**: Hỗ trợ gói cài đặt bổ sung NVIDIA CUDA cho máy có card đồ họa rời và Apple Silicon (Metal) cho phiên bản macOS.
