# Software Requirements Specification (SRS): HopHo MVP Windows (v0.1)

**Tầng tài liệu**: Implementation Layer (`docs/implementation/release-1/srs.md`)  
**Release**: release-1 (MVP Windows v0.1)  
**Phiên bản**: 1.0  
**Ngày cập nhật**: 2026-10-15  
**Nguồn tham chiếu**: `docs/project-idea/requirements.md`, `docs/plan/mvp-plan.md`  

---

## 1. Giới thiệu & Phạm vi tài liệu (Introduction & Scope)

Tài liệu Đặc tả Yêu cầu Phần mềm (SRS) này chi tiết hóa toàn bộ các Yêu cầu Chức năng (Functional Requirements - FR) và Yêu cầu Phi chức năng (Non-Functional Requirements - NFR) cho đợt phát hành Release 1 (phiên bản Windows MVP v0.1). Tài liệu là căn cứ pháp lý và kỹ thuật phục vụ việc thiết kế kiến trúc, phân rã User Story và nghiệm thu kiểm thử (QA/QC).

---

## 2. Đặc tả yêu cầu chức năng chi tiết (Functional Requirements)

### 2.1 Thu nhận & Xử lý Nguồn âm thanh (FR-AUD)

| Mã yêu cầu | Tên chức năng | Chi tiết yêu cầu kỹ thuật & Hành vi hệ thống | Tiêu chí nghiệm thu (DoD) |
|---|---|---|---|
| **FR-AUD-01** | Ghi âm Microphone | Thu âm thanh trực tiếp từ thiết bị microphone mặc định hoặc được chọn. Phục vụ cho các cuộc họp trực tiếp tại văn phòng. | Thu được luồng âm thanh rõ ràng, không bị méo tiếng, lưu đúng định dạng. |
| **FR-AUD-02** | Ghi System Audio + Mic | Thu âm thanh loopback từ ứng dụng họp (Zoom, Google Meet, Teams...) kết hợp luồng âm microphone cá nhân. Không sử dụng bot tham gia phòng họp. | Thu đồng thời 2 luồng, tách kênh rõ ràng để phục vụ phân tách người nói sau này. |
| **FR-AUD-03** | Nhập file âm thanh/video | Cho phép người dùng kéo thả hoặc chọn file media (`.mp3`, `.wav`, `.m4a`, `.mp4`) có sẵn để phân tích như một cuộc họp. | Tự động chuyển đổi qua ffmpeg thành file Opus chuẩn và tạo phiên họp tương ứng. |
| **FR-AUD-04** | Kiểm tra thiết bị & VU Meter | Cho phép chọn thiết bị mic/loopback trong Settings hoặc trước khi họp; hiển thị thanh đo âm lượng (VU meter) trực quan. | Thanh VU meter phản hồi mượt mà theo thời gian thực khi có tín hiệu âm thanh vào. |
| **FR-AUD-05** | Phân đoạn lưu Opus an toàn | Luồng âm thanh được mã hóa Opus và ghi xuống đĩa cứng theo từng đoạn (chunk) định kỳ mỗi vài giây kèm lệnh `fsync`. | Khi ứng dụng hoặc máy tính bị ngắt điện đột ngột, toàn bộ âm thanh đến chunk cuối cùng được bảo toàn. |

### 2.2 Khởi tạo Phiên họp (FR-START)

Trước mỗi phiên họp, hệ thống hiển thị hộp thoại cấu hình (cho phép lưu cấu hình làm mặc định):
1. **Chế độ**: `Chỉ ghi âm` (để xử lý ngoại tuyến sau) **hoặc** `Realtime Transcript`.
2. **Phiên dịch trực tiếp**: Tùy chọn `Bật/Tắt`. Nếu Bật, cho phép chọn ngôn ngữ đích (mặc định lấy theo ngôn ngữ native trong Settings).
3. **Ngôn ngữ cuộc họp**: Cho phép chọn cụ thể (Tiếng Việt, Tiếng Anh, Tiếng Trung, Tiếng Hàn) hoặc `Auto-detect`.
4. **Nguồn âm**: `Chỉ Microphone` hoặc `Microphone + System Audio`.
5. **Không gian làm việc**: Chọn `Workspace/Project` và nhập các thẻ phân loại (`tags`).

### 2.3 Nhận diện Giọng nói — Speech-to-Text (FR-STT)

| Mã yêu cầu | Tên chức năng | Chi tiết yêu cầu kỹ thuật & Hành vi hệ thống |
|---|---|---|
| **FR-STT-01** | Kiến trúc STT 2 lớp | (a) Lớp Realtime sử dụng mô hình nhẹ/streaming đẩy bản nháp tức thời; (b) Lớp Batch sau họp chạy lại bằng mô hình chính xác cao. |
| **FR-STT-02** | Quyền tự quyết của người dùng | Cả hai chế độ Realtime và Chạy lại sau họp đều là tùy chọn độc lập do người dùng quyết định bật/tắt. |
| **FR-STT-03** | Đề xuất mô hình theo phần cứng | Ứng dụng tự động đề xuất gói mô hình phù hợp dựa trên kết quả khảo sát phần cứng ban đầu; cho phép người dùng tùy chọn đổi mô hình. |
| **FR-STT-04** | Định dạng phân câu Realtime | Luồng transcript trực tiếp chỉ thực hiện ngắt câu bằng dấu chấm/phẩy theo VAD; tuyệt đối không phân tách người nói khi đang realtime. |
| **FR-STT-05** | Hỗ trợ hội thoại trộn (Code-switching) | Tối ưu hóa khả năng nhận diện các cuộc hội thoại sử dụng xen kẽ tiếng Việt và tiếng Anh. |
| **FR-STT-06** | Tích hợp từ điển chuyên ngành | Tự động nạp danh mục thuật ngữ (Glossary) của Workspace vào danh sách `hotwords` hoặc `initial_prompt` của mô hình STT. |
| **FR-STT-07** | Tiến trình chạy lại sau họp an toàn | Tác vụ chạy lại sau họp vận hành ở chế độ nền với mức ưu tiên CPU thấp; hỗ trợ nút **Tạm dừng (Pause)** và **Chạy tiếp (Resume)** theo checkpoint từng đoạn. |

### 2.4 Phân tách Người nói — Diarization (FR-DIA)

| Mã yêu cầu | Tên chức năng | Chi tiết yêu cầu kỹ thuật & Hành vi hệ thống |
|---|---|---|
| **FR-DIA-01** | Chạy ngoại tuyến sau họp | Tính năng Diarization chỉ thực thi sau khi cuộc họp kết thúc; tuyệt đối không chạy realtime trong phiên họp. |
| **FR-DIA-02** | Lựa chọn Engine linh hoạt | Hỗ trợ 2 engine phân tách: `sherpa-onnx` (nhẹ, ứng viên mặc định) và `pyannote` (chính xác cao, tùy chọn nâng cao). Engine mặc định được chốt ở mini-spike M4, chọn theo phần cứng. |
| **FR-DIA-03** | Đổi tên người nói hàng loạt | Cho phép người dùng chỉnh sửa nhãn mặc định `Speaker 1`, `Speaker 2` thành tên thật; hệ thống tự động cập nhật đồng bộ cho toàn bộ transcript. |
| **FR-DIA-04** | Bảo mật giọng nói | Tuyệt đối không lưu trữ profile giọng nói (voice embedding) giữa các cuộc họp khác nhau nhằm đảm bảo quyền riêng tư. |

### 2.5 Dịch thuật Trực tiếp (FR-TRN)

| Mã yêu cầu | Tên chức năng | Chi tiết yêu cầu kỹ thuật & Hành vi hệ thống |
|---|---|---|
| **FR-TRN-01** | Kích hoạt theo nhu cầu | Dịch thuật là tùy chọn độc lập, chỉ kích hoạt khi người dùng xác nhận ở hộp thoại bắt đầu phiên họp. |
| **FR-TRN-02** | Dịch thuật bằng mô hình LLM | Sử dụng LLM cục bộ (llama.cpp nhúng) hoặc Cloud LLM để dịch; không sử dụng tính năng dịch giới hạn của Whisper (vốn chỉ hỗ trợ dịch sang tiếng Anh). |
| **FR-TRN-03** | Cấu hình Engine dịch | Người dùng có thể cấu hình engine dịch riêng biệt trong Settings; hệ thống đề xuất gói phù hợp cấu hình máy. |
| **FR-TRN-04** | Dịch theo câu hoàn chỉnh | Hệ thống đợi VAD ngắt một câu hoàn chỉnh mới gửi dịch; không dịch rời rạc từng từ ngữ. |
| **FR-TRN-05** | Trải nghiệm hiển thị dịch | Cửa sổ Overlay chỉ hiển thị bản dịch để người dùng dễ theo dõi; nội dung gốc vẫn được lưu đầy đủ trong cơ sở dữ liệu. |

### 2.6 Xử lý Hậu kỳ bằng Mô hình Ngôn ngữ lớn (FR-LLM)

| Mã yêu cầu | Tên chức năng | Chi tiết yêu cầu kỹ thuật & Hành vi hệ thống |
|---|---|---|
| **FR-LLM-01** | Bộ Template tiêu chuẩn có sẵn | Cung cấp sẵn các mẫu prompt: (1) Tóm tắt điều hành, (2) Ý chính & Quyết định, (3) Danh sách việc cần làm (Action Items: Việc - Người - Deadline; lưu có cấu trúc vào bảng `action_items`), (4) Biên bản họp chuẩn Việt Nam, (5) Báo cáo tiến độ. |
| **FR-LLM-02** | Quản lý Template tùy chỉnh | Cho phép người dùng tạo mới, chỉnh sửa, nhân bản các mẫu prompt cá nhân; mỗi template là một file Markdown trong `templates/custom/` (nguồn sự thật), CSDL lưu bản sao để truy vấn nhanh. |
| **FR-LLM-03** | Trò chuyện hỏi đáp (Chat in-meeting) | Giao diện hội thoại tương tác cho phép người dùng hỏi đáp chuyên sâu dựa trên toàn bộ nội dung transcript của cuộc họp đó. |
| **FR-LLM-04** | Kết hợp Ghi chú & Đánh dấu | Khi sinh tóm tắt, tự động tích hợp các đoạn ghi chú tay (manual notes) và các điểm đánh dấu (bookmarks) của người dùng vào ngữ cảnh prompt. |
| **FR-LLM-05** | Tối ưu hóa từ vựng với Glossary | Nạp danh sách từ vựng chuyên ngành vào prompt để LLM tự động sửa các lỗi chính tả danh từ riêng hoặc thuật ngữ kỹ thuật. |
| **FR-LLM-06** | Xuất bản đa định dạng | Hỗ trợ xuất kết quả ra tệp tin định dạng Markdown (`.md`), Word (`.docx`), PDF (`.pdf`) hoặc sao chép nhanh vào khay nhớ tạm (Clipboard). |
| **FR-LLM-07** | Trích xuất theo Mẫu biên bản người dùng tải lên | Người dùng tải lên mẫu văn bản có sẵn (`.docx`, `.md`, `.pdf`). Ứng dụng trích xuất cấu trúc đề mục, bảng biểu làm khung sườn, yêu cầu LLM điền nội dung và xuất ra file `.docx` giữ nguyên mẫu. |
| **FR-LLM-08** | Đa dạng hóa Nhà cung cấp LLM | Hỗ trợ 3 nhóm kết nối: (1) Local nhúng qua llama.cpp, (2) Cloud tương thích chuẩn OpenAI, (3) Agent CLI (`claude -p`, `codex exec`, `agy`). Khuyến nghị mô hình Hybrid. |

### 2.7 Tích hợp Trợ lý Ngoài qua Model Context Protocol (FR-MCP)

| Mã yêu cầu | Tên chức năng | Chi tiết yêu cầu kỹ thuật & Hành vi hệ thống |
|---|---|---|
| **FR-MCP-01** | Máy chủ `hopho-mcp` độc lập | Cung cấp tiến trình MCP (stdio) kết nối trực tiếp SQLite ở chế độ WAL, với tối thiểu 6 tool: `list_meetings`, `get_meeting`, `get_transcript`, `search_meetings`, `get_action_items`, `save_summary`. |
| **FR-MCP-02** | Plugin/skill/cấu hình tích hợp | Cung cấp sẵn cấu hình mẫu cho Claude Desktop, Claude Code, Codex, Antigravity kèm hướng dẫn cài 1-click hoặc copy-paste (nút copy trong Settings). |
| **FR-MCP-03** | Ủy thác kết nối ứng dụng bên ngoài | Việc đẩy dữ liệu sang Notion, Jira, Slack, Trello do Agent thực hiện qua MCP; HopHo không xây dựng connector riêng. |
| **FR-MCP-04** | Tìm kiếm/RAG xuyên lịch sử | RAG xuyên toàn bộ lịch sử họp do Agent đảm nhận qua MCP (tool `search_meetings`); trong app chỉ có full-text search FTS5. |

### 2.8 Trải nghiệm Người dùng trong Lúc họp (FR-UX)

| Mã yêu cầu | Tên chức năng | Chi tiết yêu cầu kỹ thuật & Hành vi hệ thống |
|---|---|---|
| **FR-UX-01** | Cửa sổ ứng dụng chính | Quản lý danh sách cuộc họp, chi tiết transcript, xem tóm tắt, cài đặt hệ thống và quản trị mô hình. |
| **FR-UX-02** | Cửa sổ Overlay nổi Always-on-top | Cửa sổ phụ đề bán trong suốt luôn nổi trên cùng màn hình, hiển thị phụ đề hoặc bản dịch, nút Start/Stop nhanh. Cho phép kéo thả vị trí, chỉnh kích thước và độ trong suốt. |
| **FR-UX-03** | Khay hệ thống & Phím tắt toàn cục | Biểu tượng trên System Tray kết hợp hệ thống Global Hotkeys (Bắt đầu/Dừng ghi âm, Tạo bookmark) hoạt động ngay cả khi HopHo đang chạy nền. |
| **FR-UX-04** | Đánh dấu thời điểm quan trọng | Phím tắt bấm tạo Bookmark tức thời trong cuộc họp để đánh dấu mốc thời gian đáng chú ý. |
| **FR-UX-05** | Ghi chú nhanh gắn Timestamp | Cho phép người dùng vừa nghe vừa nhập ghi chú tóm tắt; mỗi dòng ghi chú tự động gắn mốc thời gian tương ứng với âm thanh. |
| **FR-UX-06** | Không chạy ngầm theo dõi | Ứng dụng không tự động quét tiến trình hệ thống để phát hiện cuộc họp đang diễn ra nhằm tránh tiêu tốn tài nguyên máy. |

### 2.9 Quản lý & Hiệu chỉnh Sau cuộc họp (FR-REV)

| Mã yêu cầu | Tên chức năng | Chi tiết yêu cầu kỹ thuật & Hành vi hệ thống |
|---|---|---|
| **FR-REV-01** | Nghe lại theo từng câu (Audio Playback) | Nhấp chuột vào bất kỳ đoạn transcript nào trên màn hình sẽ tự động phát lại chính xác đoạn âm thanh tương ứng. |
| **FR-REV-02** | Chỉnh sửa văn bản trực tiếp | Cho phép chỉnh sửa thủ công câu từ trong transcript; câu đã sửa được đánh dấu trạng thái `edited`. |
| **FR-REV-03** | Phân nhóm Workspace & Thẻ tag | Phân loại cuộc họp theo không gian làm việc (Workspace) và gắn nhãn (tags) linh hoạt để lọc dữ liệu. |
| **FR-REV-04** | Tìm kiếm toàn văn siêu tốc | Tích hợp bộ tìm kiếm toàn văn SQLite FTS5 cho phép tra cứu tức thời từ khóa trên toàn bộ transcript và summary. |
| **FR-REV-05** | Chính sách lưu trữ âm thanh | Cho phép xóa thủ công hoặc cấu hình tự động xóa file audio sau N ngày nhằm tiết kiệm ổ đĩa nhưng vẫn giữ nguyên vẹn transcript và summary. |

### 2.10 Từ điển Chuyên ngành (FR-GLO)

- **FR-GLO-01**: Cung cấp giao diện quản lý danh mục thuật ngữ, tên riêng theo từng Workspace.
- **FR-GLO-02**: Tự động liên kết dữ liệu Glossary cho cả luồng STT (dưới dạng hotwords) và luồng LLM (dưới dạng prompt định hướng chính tả).

### 2.11 Lưu trữ & Quyền riêng tư (FR-DATA)

| Mã yêu cầu | Tên chức năng | Chi tiết yêu cầu kỹ thuật & Hành vi hệ thống |
|---|---|---|
| **FR-DATA-01** | Lưu trữ cục bộ toàn phần | Toàn bộ siêu dữ liệu, transcript lưu trong SQLite; file âm thanh lưu định dạng nén Opus (~30MB/giờ họp). |
| **FR-DATA-02** | Thư mục dữ liệu duy nhất & cô lập | Cho phép người dùng tự chọn thư mục lưu trữ dữ liệu tập trung khi cài đặt. Mặc định tự động quét và gợi ý ổ đĩa còn nhiều dung lượng trống nhất, không ép buộc ổ C. |
| **FR-DATA-03** | Di chuyển thư mục dữ liệu | Cung cấp chức năng chuyển đổi toàn bộ thư mục dữ liệu sang ổ đĩa khác trực tiếp từ màn hình Settings. |
| **FR-DATA-04** | Quản lý dung lượng trực quan | Màn hình Storage hiển thị biểu đồ dung lượng theo từng nhóm (models, audio, transcript, exports) kèm nút mở thư mục và dọn dẹp. |
| **FR-DATA-05** | Không thu thập dữ liệu (No Telemetry) | Tuyệt đối không gửi bất kỳ dữ liệu ẩn, nhật ký sử dụng hay thống kê nào về máy chủ từ xa. |
| **FR-DATA-06** | Cảnh báo trước khi gửi Cloud | Luôn hiển thị hộp thoại xác nhận minh bạch về điểm đến dữ liệu trước khi gửi nội dung lên Cloud LLM hoặc Agent CLI. |
| **FR-DATA-07** | Bảo vệ khóa bí mật | Khóa API Key của Cloud LLM được mã hóa và lưu trữ an toàn trong Windows Credential Manager. |

### 2.12 Bộ cài đặt & Quản lý Mô hình (FR-INS)

| Mã yêu cầu | Tên chức năng | Chi tiết yêu cầu kỹ thuật & Hành vi hệ thống |
|---|---|---|
| **FR-INS-01** | Gói cài đặt chuẩn Windows | Phân phối qua file cài đặt `.msi` duy nhất, tự động đóng gói sẵn toàn bộ môi trường thực thi cần thiết. |
| **FR-INS-02** | Trình hướng dẫn cấu hình (Wizard) | Khảo sát cấu hình máy tính lần đầu -> Đề xuất gói mô hình (Lite / Standard / Pro) -> Tải mô hình -> Kiểm tra âm thanh. |
| **FR-INS-03** | Tải mô hình thông minh | Hỗ trợ tạm dừng/tải tiếp khi mất kết nối mạng (resume download), hiển thị thanh tiến độ và hỗ trợ mirror Hugging Face. |
| **FR-INS-04** | Trình quản lý mô hình (Model Manager) | Màn hình trực quan cho phép xem danh sách, dung lượng, kiểm tra bản cập nhật và xóa các mô hình AI. |
| **FR-INS-05** | Gói mở rộng tăng tốc GPU (CUDA) | Gói thư viện CUDA được tách thành add-on tải riêng; chỉ hiển thị đề xuất khi hệ thống phát hiện có card đồ họa NVIDIA. |
| **FR-INS-06** | Tự động cập nhật ứng dụng | Tích hợp cơ chế kiểm tra và cập nhật ứng dụng tự động qua Tauri Updater; yêu cầu người dùng xác nhận trước khi cập nhật. |

---

## 3. Đặc tả yêu cầu phi chức năng (Non-Functional Requirements - NFR)

| Mã NFR | Tiêu chuẩn chất lượng | Chỉ số đo lường & Ràng buộc kỹ thuật | Phương pháp kiểm thử |
|---|---|---|---|
| **NFR-01** | Độ bền phiên họp | Hỗ trợ phiên họp liên tục lên tới **3 giờ** mà không bị tràn bộ nhớ (memory leak) hay suy giảm tốc độ. | Kiểm thử tải chạy audio giả lập 3h |
| **NFR-02** | Tải tài nguyên hệ thống | Chế độ Realtime tiêu thụ **<= 50% CPU** và **<= 4GB RAM** trên máy chuẩn (i5 Gen10/16GB). Tuyệt đối không làm đơ giật Zoom/Meet. | Giám sát tài nguyên qua `psutil` |
| **NFR-03** | Độ trễ Realtime | Độ trễ hiển thị phiên âm trực tiếp **<= 5 giây** (tính từ thời điểm kết thúc câu nói trong âm thanh đến khi chữ hiện lên màn hình). | Đo lường độ trễ p95 bằng công cụ spike |
| **NFR-04** | Tốc độ xử lý ngoại tuyến | Tổng thời gian chạy lại STT bản chuẩn kết hợp Diarization **<= 1x thời lượng họp** (RTF <= 1.0) trên cấu hình chuẩn. | Đo tổng thời gian xử lý file mẫu 1h |
| **NFR-05** | Khả năng tự phục hồi | Khi ứng dụng bị tắt đột ngột, mở lại hệ thống tự động phát hiện phiên họp dở dang và phục hồi toàn bộ audio đến chunk cuối. | Thực hiện kill tiến trình ngẫu nhiên |
| **NFR-06** | Trạng thái rảnh nhẹ nhàng | Khi không trong phiên ghi âm: **0% CPU** tiêu thụ bởi sidecar AI; sidecar tự động tắt sau khoảng thời gian rảnh rỗi định trước. | Giám sát tác vụ nền khi app ở chế độ chờ |
| **NFR-07** | Xử lý lỗi tường minh | Mọi sự cố (mất quyền microphone, rớt mạng khi tải model, lỗi card âm thanh) phải được thông báo rõ ràng cho người dùng kèm mã lỗi và hướng dẫn khắc phục. | Rà soát toàn bộ các luồng ngoại lệ |

---

## 4. Ma trận truy vết yêu cầu (Traceability Matrix)

| Nhóm yêu cầu | Mã Yêu cầu Chức năng (FR) | Ràng buộc Phi chức năng (NFR) | Cột mốc thực thi (Milestone) |
|---|---|---|---|
| Khảo sát & Chọn Model AI | FR-STT-01, FR-STT-05, FR-STT-06 | NFR-02, NFR-03, NFR-04 | **M0 (Spike STT Benchmark)** |
| Khung ứng dụng & IPC | (hạ tầng, không có FR nghiệp vụ) | NFR-06, NFR-07 | **M1 (Skeleton & Infrastructure)** |
| Thu âm & Quản lý File | FR-AUD-01..05, FR-DATA-01..05 | NFR-01, NFR-05 | **M2 (Audio Capture & Storage)** |
| Nhận diện giọng nói STT | FR-START-01, FR-STT-01..07, FR-GLO-01..02 | NFR-02, NFR-03, NFR-04 | **M3 (Two-Tier STT Pipeline)** |
| Phân tách người nói & Review | FR-DIA-01..04, FR-REV-01..05 | NFR-04 | **M4 (Diarization & Review UI)** |
| Xử lý LLM & Dịch thuật | FR-TRN-01..05, FR-LLM-01..08, FR-DATA-06..07 | NFR-07 | **M5 (LLM, Templates & Translation)** |
| Tích hợp Agent ngoài | FR-MCP-01..04 | NFR-06, NFR-07 | **M6 (MCP Server & Agent CLI)** |
| Giao diện trong cuộc họp | FR-UX-01..06 | NFR-02, NFR-07 | **M7 (In-Meeting UX & Overlay)** |
| Đóng gói & Quản lý Model | FR-INS-01..06 | NFR-06, NFR-07 | **M8 (Installer & Model Manager)** |
| Nghiệm thu & Phát hành v0.1 | Toàn bộ FR-01 đến FR-INS-06 | Đạt 100% NFR-01 đến NFR-07 | **M9 (Hardening & Release v0.1)** |
