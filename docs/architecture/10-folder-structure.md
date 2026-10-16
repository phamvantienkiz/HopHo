# Monorepo Folder Structure & Module Boundaries — HopHo (Họp Hộ)

**Tầng tài liệu**: Architecture Layer (`docs/architecture/10-folder-structure.md`)  
**Hệ thống**: HopHo (Họp Hộ) Desktop Assistant  
**Phiên bản**: 1.0  
**Ngày cập nhật**: 2026-10-15  
**Trạng thái**: Living System Invariant  
**Nguồn tham chiếu**: `docs/plan/mvp-plan.md` §3  

---

## 1. Cấu trúc Tổng thể Monorepo (Root Layout)

HopHo được tổ chức theo cấu trúc **Monorepo đa ngôn ngữ (Rust, Python, TypeScript)** với ranh giới phân tách độc lập giữa các ứng dụng và gói thư viện dùng chung:

```
HopHo/
├── apps/
│   └── desktop/                 # Ứng dụng Desktop chính (Tauri 2 + React 18 + TypeScript)
│
├── sidecar/                     # Dịch vụ tính toán AI độc lập (Python 3.10+, FastAPI, uv)
│
├── packages/
│   └── hopho_store/             # Thư viện Python dùng chung truy cập SQLite Database
│
├── mcp/                         # Máy chủ Model Context Protocol (hopho-mcp stdio)
│
├── templates/                   # Bộ mẫu prompt Markdown mặc định của hệ thống
│
├── spikes/
│   └── stt-benchmark/           # Code thực thi đo lường Benchmark STT (Milestone M0)
│
├── docs/                        # Hệ thống tài liệu 3 tầng (Business, Architecture, Implementation)
│   ├── business/                # Tầng Business (BRD, Roadmap, Glossary, Stakeholders)
│   ├── architecture/            # Tầng Architecture (Living System Invariants)
│   ├── implementation/          # Tầng Implementation (Tổ chức theo release-N)
│   ├── views/                   # Tầng Visual Views (Các tệp HTML trực quan độc lập)
│   ├── plan/                    # Kế hoạch MVP & spike (đầu vào, ngoài 3 tầng)
│   ├── project-idea/            # Ý tưởng & đặc tả yêu cầu gốc (đầu vào, ngoài 3 tầng)
│   ├── reports/                 # Báo cáo kết quả benchmark/spike
│   ├── research/                # Tài liệu nghiên cứu tham khảo
│   └── ai/                      # Tài liệu do AI Agent sinh ra
│
├── tasks/                       # Quản lý tiến độ dự án (todo.md, lessons.md, workflow.md)
├── AGENTS.md                    # Bản chỉ dẫn trí tuệ cho AI Coding Agents
└── README.md                    # Giới thiệu tổng quan dự án & Hướng dẫn cài đặt
```

---

## 2. Chi tiết Cấu trúc các Ứng dụng & Module con

### 2.1 Ứng dụng Desktop (`apps/desktop/`)

```
apps/desktop/
├── package.json                 # Quản lý dependencies frontend (pnpm)
├── vite.config.ts               # Cấu hình Vite bundler
├── tailwind.config.js           # Cấu hình giao diện Tailwind CSS
├── src/                         # Mã nguồn giao diện người dùng (React + TypeScript)
│   ├── main.tsx                 # Điểm khởi chạy React app
│   ├── App.tsx                  # Bộ định tuyến cửa sổ chính (Main App Router)
│   ├── components/              # Các thành phần giao diện theo miền nghiệp vụ
│   │   ├── common/              # Button, Modal, Toast, Input dùng chung
│   │   ├── session/             # Hộp thoại bắt đầu họp, VU meter
│   │   ├── overlay/             # Cửa sổ phụ đề nổi Always-on-top
│   │   ├── transcript/          # Danh sách câu transcript, audio playback inline
│   │   ├── chat/                # Khung chat hỏi đáp trong cuộc họp
│   │   ├── settings/            # Màn hình cài đặt, Storage, Model Manager
│   │   ├── onboarding/          # Trình hướng dẫn Onboarding Wizard
│   │   └── workspace/           # Quản lý Workspace & từ điển Glossary
│   ├── stores/                  # Zustand state stores (sessionStore, settingsStore)
│   ├── hooks/                   # Custom React hooks giao tiếp Tauri IPC
│   └── types/                   # Định nghĩa TypeScript interfaces & types
│
└── src-tauri/                   # Mã nguồn Rust Core Engine
    ├── Cargo.toml               # Quản lý dependencies Rust (crates)
    ├── tauri.conf.json          # Cấu hình quyền hạn, cửa sổ, updater của Tauri
    ├── build.rs                 # Script build nhúng binary
    └── src/
        ├── main.rs              # Điểm khởi chạy Rust runtime & Tauri builder
        ├── audio/               # Thu nhận âm thanh native
        │   ├── mod.rs
        │   └── wasapi.rs        # Thu âm Microphone & Loopback âm thanh hệ thống Windows
        ├── recorder/            # Ghi âm và nén phân đoạn an toàn
        │   ├── mod.rs
        │   ├── chunk_writer.rs  # Nén Opus và ghi đĩa kèm fsync định kỳ
        │   ├── recovery.rs      # Quét và phục hồi phiên họp bị gián đoạn crash
        │   └── import.rs        # Trích xuất âm thanh từ file media qua ffmpeg
        ├── storage/             # Quản lý cơ sở dữ liệu SQLite
        │   ├── mod.rs
        │   ├── db.rs            # Khởi tạo SQLite connection pool (WAL mode)
        │   ├── path.rs          # Quản lý đường dẫn thư mục data cô lập
        │   └── migrations/      # Các tệp SQL migration nâng cấp phiên bản
        │       ├── 001_initial_schema.sql
        │       └── 002_fts5_indexes.sql
        ├── sidecar/             # Điều khiển vòng đời Python Sidecar
        │   ├── mod.rs
        │   ├── process.rs       # Spawn process, cấp phát cổng, Windows Job Object
        │   ├── client.rs        # WebSocket client kết nối Sidecar
        │   └── watchdog.rs      # Giám sát sức khỏe, tự tắt khi rảnh, tự phục hồi
        └── system/              # Tương tác hệ điều hành Windows
            ├── mod.rs
            ├── hotkey.rs        # Đăng ký phím tắt toàn cục (Global Hotkeys)
            ├── tray.rs          # Quản lý biểu tượng khay hệ thống (System Tray)
            ├── keychain.rs      # Đọc/ghi khóa bảo mật vào Windows Credential Manager
            └── downloader.rs    # Tải mô hình AI hỗ trợ resume & kiểm tra SHA-256
```

### 2.2 Dịch vụ Tính toán AI Sidecar (`sidecar/`)

```
sidecar/
├── pyproject.toml               # Quản lý dependencies Python bằng uv
├── src/
│   └── hopho_sidecar/
│       ├── __init__.py
│       ├── main.py              # Điểm khởi chạy FastAPI & Uvicorn CLI
│       ├── api/                 # Tầng tiếp nhận API mạng cục bộ
│       │   ├── router.py        # Định tuyến các endpoints
│       │   ├── websocket.py     # Endpoint WebSocket nhận PCM stream & trả text
│       │   └── health.py        # Endpoint kiểm tra sức khỏe (/health)
│       ├── stt/                 # Bộ máy nhận dạng tiếng nói
│       │   ├── vad.py           # Silero VAD cắt câu
│       │   ├── realtime.py      # Engine streaming (faster-whisper int8 / sherpa-onnx)
│       │   └── batch.py         # Engine chạy lại sau họp (Whisper large / PhoWhisper)
│       ├── diarization/         # Phân tách người nói ngoại tuyến
│       │   ├── engine.py        # sherpa-onnx 3D-speaker & pyannote wrapper
│       │   └── cluster.py       # Thuật toán gom cụm và gán nhãn Speaker N
│       ├── translation/         # Dịch thuật trực tiếp theo câu
│       │   └── translator.py    # Dịch qua LLM nhỏ local hoặc Cloud endpoint
│       ├── llm/                 # Xử lý ngôn ngữ lớn & Trợ lý
│       │   ├── base.py          # Abstract LlmProvider Protocol
│       │   ├── providers/       # llama.cpp nhúng, Cloud OpenAI, Agent CLI adapters
│       │   └── map_reduce.py    # Thuật toán tóm tắt cuộc họp dài 3h
│       ├── templates/           # Xử lý biểu mẫu & Mẫu biên bản tải lên
│       │   ├── renderer.py      # Render prompt Markdown với biến template
│       │   └── extractor.py     # Trích xuất cấu trúc tệp mẫu .docx/.pdf
│       └── export/              # Xuất bản tài liệu
│           ├── docx_writer.py   # Tạo file Word giữ nguyên mẫu
│           ├── md_writer.py     # Tạo file Markdown
│           └── pdf_writer.py    # Xuất PDF
```

### 2.3 Gói Dữ liệu Dùng chung (`packages/hopho_store/`)

```
packages/hopho_store/
├── pyproject.toml
└── src/
    └── hopho_store/
        ├── __init__.py
        ├── connection.py        # Kết nối SQLite với cấu hình WAL & busy_timeout
        ├── models.py            # Pydantic v2 data models ánh xạ bảng SQLite
        └── repositories/        # Repository pattern truy vấn dữ liệu
            ├── meetings.py      # Thao tác bảng meetings & audio_chunks
            ├── segments.py      # Thao tác bảng segments & FTS5 search
            ├── templates.py     # Thao tác bảng templates & outputs
            └── glossary.py      # Thao tác bảng glossary_terms
```

### 2.4 Máy chủ MCP Độc lập (`mcp/`)

```
mcp/
├── pyproject.toml
├── README.md                    # Hướng dẫn cấu hình cho Claude Desktop & Antigravity
├── src/
│   └── hopho_mcp/
│       ├── __init__.py
│       ├── server.py            # Máy chủ MCP Stdio
│       └── tools/               # 6 Tools tiêu chuẩn
│           ├── list_meetings.py
│           ├── get_meeting.py
│           ├── get_transcript.py
│           ├── search_meetings.py
│           ├── get_action_items.py
│           └── save_summary.py
└── integrations/                # File cấu hình mẫu cho các Agent
    ├── claude_desktop_config.json
    ├── claude_code.md
    ├── codex_config.toml
    └── antigravity_config.json
```

---

## 3. Quy tắc Đặt File cho AI Coding Agent (File Placement Invariants)

Khi được yêu cầu viết mã nguồn mới, Coding Agent **BẮT BUỘC** phải tuân theo bảng chỉ dẫn vị trí sau, tuyệt đối không tự tạo thư mục lạ ở root:

| Yêu cầu nghiệp vụ / Kỹ thuật | Thư mục đặt file bắt buộc |
|---|---|
| Thêm giao diện người dùng mới (Component / View) | `apps/desktop/src/components/{domain}/` |
| Thêm trạng thái toàn cục UI (State Store) | `apps/desktop/src/stores/{domain}Store.ts` |
| Thêm logic thu âm / phần cứng Windows mới | `apps/desktop/src-tauri/src/audio/` hoặc `system/` |
| Thêm bảng cơ sở dữ liệu mới / Migration SQL | Tạo file migration trong `apps/desktop/src-tauri/src/storage/migrations/` VÀ cập nhật model trong `packages/hopho_store/` |
| Bổ sung mô hình AI mới (STT / LLM / Diarization) | `sidecar/src/hopho_sidecar/{stt|llm|diarization}/` |
| Thêm công cụ mới cho MCP Server | `mcp/src/hopho_mcp/tools/` |
| Thêm mẫu prompt Markdown mặc định | `templates/{ten-mau}.md` |
| Code spike / benchmark | `spikes/{spike-name}/` (báo cáo kết quả đặt tại `docs/reports/`) |
| Tài liệu kỹ thuật hệ thống | Tuân thủ 3 tầng trong `docs/` (`business/`, `architecture/`, `implementation/`) |
