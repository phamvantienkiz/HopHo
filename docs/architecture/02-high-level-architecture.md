# High-Level Architecture — HopHo (Họp Hộ)

**Tầng tài liệu**: Architecture Layer (`docs/architecture/02-high-level-architecture.md`)  
**Hệ thống**: HopHo (Họp Hộ) Desktop Assistant  
**Phiên bản**: 1.0  
**Ngày cập nhật**: 2026-10-15  
**Trạng thái**: Living System Invariant  
**Nguồn tham chiếu**: `docs/project-idea/requirements.md` §2, `docs/plan/mvp-plan.md` §2 & §4  

---

## 1. Topo Kiến trúc Tổng thể (System Topology)

HopHo được thiết kế theo mô hình **Multi-Process Hybrid Desktop Architecture** kết hợp giữa tính an toàn, nhẹ nhàng của **Rust (Tauri 2)** cho các tác vụ hệ thống và sức mạnh phong phú của hệ sinh thái **Python** cho tính toán AI.

```mermaid
flowchart TD
    subgraph ClientHost ["Máy tính Người dùng (Windows / Local Desktop)"]
        subgraph TauriApp ["Tauri 2 Desktop Shell (Rust + Webview)"]
            UI["React 18 + TypeScript UI<br/>(Main Window + Overlay Always-on-top + Tray)"]
            
            subgraph RustCore ["Rust Core Runtime"]
                AudioCap["WASAPI Audio Capture<br/>(Mic + Loopback Native)"]
                RecEngine["Chunk Recorder & Recovery<br/>(Opus Encoder + fsync)"]
                StorageMgr["Storage Manager<br/>(SQLite Schema Owner & Migrations)"]
                SidecarCtrl["Sidecar Process Controller<br/>(Spawn, Token, Port, Lifecycle, WS Client)"]
                SysIntegrate["System Integration<br/>(Global Hotkeys, Tray, Keychain)"]
            end
            
            UI <-->|Tauri IPC Commands & Events| RustCore
        end

        subgraph SidecarProc ["Python AI Sidecar (FastAPI + Uvicorn)"]
            WS_API["Localhost WebSocket API<br/>(Token Auth @ 127.0.0.1:RandomPort)"]
            STT_RT["Realtime STT Engine<br/>(Silero VAD + faster-whisper / sherpa-onnx)"]
            STT_Batch["Batch STT Engine<br/>(Whisper large-v3-turbo / PhoWhisper)"]
            Diarize["Diarization Engine<br/>(sherpa-onnx / pyannote)"]
            Translate["Realtime Translation Engine<br/>(Sentence-level LLM / Cloud)"]
            LLM_Service["LLM & Template Processor<br/>(llama.cpp, Docx Extractor, Map-Reduce)"]
            
            WS_API --> STT_RT
            WS_API --> STT_Batch
            WS_API --> Diarize
            WS_API --> Translate
            WS_API --> LLM_Service
        end

        subgraph LocalData ["Thư mục Dữ liệu Cô lập (Local Storage)"]
            SQLiteDB[("SQLite Database (WAL Mode)<br/>hopho.db + FTS5 Index")]
            AudioStore[("Audio Files<br/>audio/*.opus")]
            ModelStore[("AI Models Cache<br/>models/*")]
            ExportStore[("Exports<br/>exports/*.docx, *.md")]
        end

        subgraph MCPProc ["hopho-mcp Server (Independent Process)"]
            MCPServer["hopho-mcp CLI<br/>(Python MCP SDK via Stdio)"]
        end

        %% Kết nối nội bộ
        AudioCap --> RecEngine
        RecEngine -- "Opus chunks" --> AudioStore
        RecEngine -- "PCM 16kHz mono (WS)" --> SidecarCtrl
        SidecarCtrl <-->|WebSocket IPC Loopback| WS_API
        StorageMgr <-->|rusqlite / WAL| SQLiteDB
        SidecarProc <-->|hopho_store (Python ORM)| SQLiteDB
        MCPServer <-->|hopho_store (Read-mostly)| SQLiteDB
    end

    subgraph ExternalEcosystem ["Hệ sinh thái bên ngoài"]
        ExtAudio["🎧 Âm thanh Hệ điều hành<br/>(Zoom / Meet / Teams / Mic vật lý)"]
        ExtAgents["🤖 External AI Agents<br/>(Claude Desktop / Claude Code / Codex / Antigravity)"]
        CloudLLM["☁️ Cloud LLM Endpoints<br/>(OpenAI-compatible via HTTPS)"]
        AgentCLI["💻 Local Agent CLI<br/>(claude -p / codex exec / agy)"]
    end

    ExtAudio --> AudioCap
    ExtAgents <-->|JSON-RPC via Stdio| MCPServer
    LLM_Service -. "User Opt-in (Consent Gate)" .-> CloudLLM
    LLM_Service -. "User Opt-in (Headless Call)" .-> AgentCLI
```

---

## 2. Phân rã Hệ thống & Ranh giới Trách nhiệm (Subsystem Decomposition)

Hệ thống được chia thành 4 thành phần thực thi độc lập với ranh giới trách nhiệm cực kỳ rành mạch:

### 2.1 Tauri Desktop Shell (Rust Core + React UI)
- **Công nghệ**: Tauri 2 (Rust), React 18, TypeScript, Tailwind CSS, Vite.
- **Trách nhiệm chính**:
  1. **Hiển thị giao diện**: Quản lý cửa sổ chính (Main window), cửa sổ phụ đề nổi (Overlay always-on-top trong suốt, không viền) và biểu tượng khay hệ thống (System Tray).
  2. **Thu nhận âm thanh native**: Sử dụng Windows Core Audio APIs (WASAPI loopback + mic), resample âm thanh về chuẩn 16,000 Hz, 16-bit mono.
  3. **Lưu trữ âm thanh an toàn**: Nén Opus và ghi tuần tự các chunk (2-5s) xuống đĩa cứng kèm lệnh `fsync`. Quản lý cơ chế phục hồi crash khi mở lại app.
  4. **Chủ sở hữu Schema cơ sở dữ liệu**: Thực thi các migration tạo bảng và nâng cấp phiên bản SQLite thông qua Rust.
  5. **Quản lý vòng đời Sidecar**: Khởi chạy tiến trình Python khi cần, cấp phát cổng ngẫu nhiên, sinh token bí mật, kiểm tra health-check và tự động tắt khi không dùng.

### 2.2 Python AI Sidecar (`hopho_sidecar`)
- **Công nghệ**: Python 3.10+, FastAPI, Uvicorn, CTranslate2, ONNX Runtime, llama.cpp nhúng (`llama-cpp-python`), python-docx.
- **Trách nhiệm chính**:
  1. **Realtime STT**: Nhận luồng PCM qua WebSocket, áp dụng VAD ngắt câu, giải mã bản nháp tức thời và đẩy kết quả partial/final về UI.
  2. **Batch STT & Diarization**: Xử lý lại file âm thanh sau cuộc họp bằng mô hình lớn; phân cụm người nói ngoại tuyến không lưu voice profile.
  3. **Realtime Translation**: Dịch các câu hoàn chỉnh sang ngôn ngữ đích qua mô hình dịch chuyên dụng hoặc LLM nhỏ.
  4. **Post-meeting Processing**: Tóm tắt, trích xuất action items, xử lý tài liệu họp dài qua kỹ thuật Map-Reduce, trích xuất cấu trúc mẫu biên bản người dùng tải lên và điền nội dung vào file DOCX.

### 2.3 Máy chủ MCP Độc lập (`hopho-mcp`)
- **Công nghệ**: Python, MCP SDK chính thức.
- **Trách nhiệm chính**:
  1. Vận hành dưới dạng tiến trình console độc lập giao tiếp qua `stdio`.
  2. Kết nối trực tiếp vào file SQLite ở chế độ đọc an toàn (WAL mode) thông qua package thư viện dùng chung `packages/hopho_store`.
  3. Cung cấp 6 công cụ (Tools) chuẩn cho các AI Agent bên ngoài: `list_meetings`, `get_meeting`, `get_transcript`, `search_meetings`, `get_action_items`, `save_summary`.

### 2.4 Thư viện dùng chung (`packages/hopho_store`)
- **Trách nhiệm chính**: Package Python nội bộ cung cấp mô hình dữ liệu (Data Access Layer / Repository) chuẩn hóa để cả Sidecar và MCP truy cập cơ sở dữ liệu SQLite mà không bị trùng lặp mã nguồn hoặc lệch logic schema.

---

## 3. Tính toán Năng lực & Tài nguyên (Back-of-the-Envelope Capacity - BOTEC)

Các chỉ số dưới đây là **ước lượng lý thuyết, chưa đo thực tế** cho cấu hình chuẩn (**Intel Core i5 Gen 10 / 16GB RAM, Only CPU**). Căn cứ chính thức là kết quả benchmark M0 ([spike-stt-benchmark.md](../plan/spike-stt-benchmark.md)); nếu không đạt NFR-02/03 thì áp dụng phương án dự phòng của R1 (gợi ý "Chỉ ghi âm" hoặc dịch bằng cloud):

| Thông số kỹ thuật | Công thức tính toán | Giá trị định lượng | Tác động kiến trúc |
|---|---|---|---|
| **Luồng PCM thô đầu vào** | $16,000 \text{ samples/s} \times 2 \text{ bytes (16-bit)} \times 1 \text{ channel}$ | **32 KB/s** ($1.92 \text{ MB/phút}$, $\approx 115.2 \text{ MB/giờ}$) | Băng thông loopback localhost cực nhỏ, không gây nghẽn socket |
| **Dung lượng nén Opus** | Bitrate trung bình $32 \text{ kbps} - 64 \text{ kbps}$ | **$\approx 30 \text{ MB/giờ}$** (Cuộc họp 3h $\approx 90 \text{ MB}$) | Cho phép lưu trữ hàng trăm giờ họp mà chỉ tốn vài GB ổ đĩa |
| **Tần suất ghi đĩa (Disk I/O)** | 1 chunk Opus mỗi $3 \text{ giây}$ | $\approx 25 \text{ KB/lần ghi} + \text{fsync}$ | Tải I/O đĩa cực thấp ($< 0.01\%$), an toàn tuyệt đối khi crash |
| **Dung lượng văn bản cuộc họp** | Trung bình 150 từ/phút $\times 60 \text{ phút} \approx 9,000 \text{ từ}$ | $\approx 50 \text{ KB text/giờ}$ ($\approx 3,000 \text{ segments/3h}$) | SQLite lưu trữ thoải mái, FTS5 tìm kiếm dưới $200 \text{ ms}$ |
| **Độ trễ Realtime STT** | VAD cắt câu ($0.5-1.5\text{s}$) + CTranslate2 int8 decode ($1.0-2.5\text{s}$) | **P95 $\approx 3.5 - 4.2 \text{ giây}$** | Ước lượng nằm trong ngưỡng **NFR-03 ($<= 5\text{s}$)** trên CPU Gen 10 |
| **Tải CPU khi chạy Realtime** | Rust WASAPI ($1-2\%$) + Whisper Small int8 ($30-40\%$) + UI ($2\%$) | **Tổng tải $\approx 35 - 45\%$ CPU** | Ước lượng nằm trong ngưỡng **NFR-02 ($<= 50\%$ CPU)**, chừa CPU cho Zoom |
| **Bộ nhớ RAM tiêu thụ** | Tauri/Webview ($180\text{MB}$) + Sidecar + STT Model ($1.5-2.2\text{GB}$) | **Tổng RAM $\approx 2.4 - 2.8\text{GB}$** | Ước lượng nằm trong ngưỡng **NFR-02 ($<= 4\text{GB}$ RAM)** trên máy 16GB |

---

## 4. Đánh giá Đánh đổi Kiến trúc (Architectural Trade-Offs Rubric)

Mọi quyết định kiến trúc cốt lõi (AD-1 đến AD-7) đều được cân nhắc kỹ lưỡng theo ma trận 4 cột chuẩn:

| Quyết định | What it solves (Giải quyết được gì) | What it worsens (Gây bất lợi gì) | Change it when (Khi nào cần thay đổi) |
|---|---|---|---|
| **AD-1: Tauri 2 + Python Sidecar (thay vì Electron hoặc Pure Rust)** | Bộ cài shell siêu nhẹ ($< 30\text{MB}$ so với $> 150\text{MB}$ của Electron); tận dụng trọn vẹn hệ sinh thái mô hình AI hàng đầu của Python mà không phải viết lại C/C++ bindings phức tạp. | Phải quản lý tiến trình phụ (Sidecar); bộ cài sidecar nặng hơn thuần Rust; tốn thêm IPC giữa 2 tiến trình. | Khi các framework AI trên Rust (như `candle`, `burn`) đạt độ chín và hỗ trợ 100% các model STT/Diarization/LLM mới nhất. |
| **AD-2: WebSocket nội bộ trên `127.0.0.1` kèm Token (thay vì Named Pipes hay Stdio)** | Hỗ trợ luồng nhị phân hai chiều mượt mà (gửi PCM stream và nhận text partial đồng thời); hỗ trợ chạy song song nhiều tác vụ; dễ debug bằng trình duyệt devtools. | Tốn cổng mạng cục bộ; phải có cơ chế cấp phát cổng ngẫu nhiên tránh xung đột; cần cơ chế token xác thực bảo vệ. | Nếu hệ điều hành Windows chặn socket loopback do chính sách bảo mật nội bộ khắt khe ở môi trường ngân hàng/quân sự. |
| **AD-3: `hopho-mcp` là tiến trình stdio riêng (thay vì nhúng trong app GUI)** | Cho phép các AI Agent (Claude, Codex, Antigravity) tự động triệu gọi dữ liệu họp bất kỳ lúc nào ngay cả khi người dùng không mở cửa sổ HopHo desktop. | Phải đảm bảo tiến trình MCP truy cập SQLite an toàn khi app desktop cũng đang ghi dữ liệu (cần SQLite WAL mode). | Khi tất cả các Agent chuẩn hóa việc giao tiếp qua Server-Sent Events (SSE) hoặc WebSocket daemon từ xa. |
| **AD-4: Rust là chủ sở hữu duy nhất của SQLite Schema** | Ngăn chặn hoàn toàn hiện tượng lệch phiên bản migration giữa Rust và Python; đảm bảo tính toàn vẹn dữ liệu ở tầng thấp nhất. | Python Sidecar không thể tự tiện sửa cấu trúc bảng; phải phát triển package `hopho_store` riêng cho Python truy cập. | Không có lý do thay đổi. Đây là bất biến sống còn của hệ thống. |
| **AD-5: Ưu tiên CTranslate2 / ONNX / llama.cpp (tránh PyTorch)** | Giảm kích thước bộ cài Sidecar từ $\approx 2\text{GB}$ (PyTorch full) xuống còn $\approx 300-500\text{MB}$; tối ưu hóa CPU cực tốt qua lệnh int8 SIMD (AVX2/AVX-512). | Một số mô hình nghiên cứu mới nhất trên Hugging Face cần thời gian chuyển đổi (convert) sang định dạng ONNX/CTranslate2. | Khi xuất hiện mô hình AI mang tính đột phá vượt trội nhưng bắt buộc phải chạy trên runtime PyTorch nguyên bản. |
| **AD-6: Sidecar On-Demand & Tự tắt khi rảnh (NFR-06)** | Đảm bảo 0% CPU khi không họp; máy tính người dùng chạy nhẹ nhàng, không gây nóng máy hao pin khi để app chạy ngầm. | Lần đầu tiên bắt đầu họp sẽ mất khoảng $1.5 - 2.5\text{ giây}$ để khởi động Sidecar và nạp mô hình vào RAM. | Khi người dùng bật tùy chọn "Luôn giữ mô hình trong RAM để phản hồi tức thì (Fast Start)". |
| **AD-7: Quyết định Model bằng Benchmark M0 thực tế** | Loại bỏ hoàn toàn sự cảm tính trong việc chọn model; đảm bảo model được chọn chắc chắn chạy mượt trên phần cứng thực tế của người dùng. | Tốn thêm thời gian thực hiện Milestone M0 để xây dựng harness đo lường trước khi code sâu vào tính năng. | Khi các tiêu chuẩn phần cứng chuẩn được nâng cấp đồng loạt (ví dụ máy tính phổ thông đều có NPU tích hợp). |
