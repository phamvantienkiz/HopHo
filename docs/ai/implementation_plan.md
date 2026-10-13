# Kế hoạch: Tài liệu MVP + Thiết kế Spike Benchmark STT

## Mục tiêu

Viết bộ tài liệu kế hoạch vào `docs/plan/`, gồm hai phần:

1. **Kế hoạch MVP** cho bản Windows: kiến trúc repo, các milestone, thứ tự phụ thuộc giữa chúng, tiêu chí hoàn thành (Definition of Done) cho từng milestone.
2. **Thiết kế spike benchmark STT**: mục tiêu, model ứng viên, dataset, các chỉ số đo, cấu trúc code dự kiến, tiêu chí chọn model. Phạm vi lần này **chỉ là thiết kế, chưa viết code**.

Đồng thời cập nhật [requirements.md](../project-idea/requirements.md) theo các quyết định bạn vừa chốt thêm.

**Phạm vi:** chỉ tạo/sửa file Markdown. Không tạo code, không cài package.

---

## Cần bạn xem lại

> [!IMPORTANT]
> **Kiến trúc IPC giữa Rust và Python sidecar:** mình đề xuất sidecar chạy một server **localhost (FastAPI + WebSocket)**, bind `127.0.0.1` trên port ngẫu nhiên, kèm token xác thực sinh mỗi lần chạy. Rust thu audio rồi đẩy PCM 16kHz mono qua WebSocket. Phương án thay thế là JSON-RPC qua stdio: đơn giản hơn, nhưng gửi stream nhị phân và làm nhiều tác vụ song song (realtime + chạy lại sau họp) thì kém hơn.

> [!IMPORTANT]
> **MCP server là một tiến trình riêng** (`hopho-mcp`). Claude Desktop/Codex tự khởi chạy MCP server qua stdio, không phụ thuộc HopHo có đang mở hay không. Vì vậy `hopho-mcp` đọc thẳng SQLite (chế độ WAL), và chỉ ghi vào bảng summary/notes.

> [!NOTE]
> Máy dev hiện tại (i5-13500H) mạnh hơn mốc chuẩn Gen10 khá nhiều. Kết quả spike trên máy này **không thay thế** được phép đo trên máy Gen10 thật. Tài liệu spike sẽ yêu cầu chạy lại trên ít nhất 1 máy Gen10/16GB trước khi chốt model.

---

## Thay đổi đề xuất

### 1. Cập nhật yêu cầu

#### [MODIFY] [requirements.md](../project-idea/requirements.md)

- **§1 Tổng quan:** ghi rõ i5 Gen8/9 là mức *tối thiểu, chấp nhận chạy chậm*. **Gen10 là mốc tham chiếu** cho mọi chỉ tiêu NFR.
- **§3.3 / §3.5:** thêm ghi chú "Model được chốt trong quá trình phát triển dựa trên benchmark", trỏ link tới tài liệu spike.
- **§3.5 Dịch:** viết lại FR-TRN-02/03 như sau:
  ```diff
  - FR-TRN-02 Ưu tiên dùng khả năng dịch của model STT đa ngôn ngữ nếu có hướng dịch phù hợp.
  - FR-TRN-03 Nếu STT không đáp ứng được thì user chọn engine: local LLM nhỏ hoặc cloud OpenAI-compatible.
  + FR-TRN-02 Dịch bằng LLM tổng quát hoặc LLM chuyên dụng cho dịch (local qua llama.cpp/Ollama, hoặc cloud OpenAI-compatible). Không dùng chức năng dịch của STT.
  + FR-TRN-03 User chọn engine dịch trong Settings; app đề xuất theo phần cứng.
  ```
- **§3.6:** thêm **FR-LLM-07**: user **upload mẫu biên bản** (DOCX/MD/PDF). App trích cấu trúc mẫu làm khung, LLM điền nội dung, export ra DOCX/MD. Template "Biên bản cuộc họp" có sẵn chỉ còn là mẫu mặc định chung.
- **§8 Rủi ro:** xoá R6 (đồng thuận ghi âm), xoá OQ1/OQ2 (đã chốt), viết lại R1 theo mốc Gen10.
- **§7 Out of scope:** thêm "Nhắc đồng thuận ghi âm / disclaimer".

---

### 2. Kế hoạch MVP

#### [NEW] `docs/plan/mvp-plan.md`

Dàn ý:

1. **Mục tiêu & phạm vi MVP:** bản Windows, chỉ những FR trong requirements đã đánh dấu MVP.
2. **Cấu trúc monorepo đề xuất**
   ```
   HopHo/
   ├── apps/desktop/          # Tauri 2 + React/TS (UI, Rust core: audio, storage, sidecar mgmt)
   │   ├── src/               # React UI
   │   └── src-tauri/         # Rust
   ├── sidecar/               # Python (uv): STT, diarization, LLM, translation, FastAPI server
   ├── mcp/                   # hopho-mcp (Python, dùng chung package truy cập DB với sidecar)
   ├── spikes/stt-benchmark/  # Spike (M0)
   ├── templates/             # Template prompt mặc định (Markdown)
   └── docs/
   ```
3. **Sơ đồ kiến trúc runtime** (mermaid): Rust audio, chunk writer, WebSocket gửi sang sidecar, rồi tới SQLite; và `hopho-mcp` đọc DB.
4. **Data model sơ bộ (SQLite):** `workspaces`, `meetings`, `audio_chunks`, `segments` (bản realtime/bản chuẩn), `speakers`, `bookmarks`, `notes`, `glossary_terms`, `templates`, `outputs`, `chat_messages`, kèm bảng FTS5.
5. **Milestones** (mỗi milestone có: mục tiêu, FR liên quan, phụ thuộc, DoD):

   ```mermaid
   flowchart LR
     M0["M0 Spike STT"] --> M3
     M1["M1 Skeleton"] --> M2["M2 Audio + Storage"] --> M3["M3 STT 2 lớp"]
     M3 --> M4["M4 Diarization + Review"]
     M3 --> M5["M5 LLM + Translation"]
     M5 --> M6["M6 MCP + Agent CLI"]
     M2 --> M7["M7 In-meeting UX"]
     M4 --> M8["M8 Installer + Onboarding"]
     M5 --> M8
     M6 --> M8
     M7 --> M8
     M8 --> M9["M9 Hardening + Release"]
   ```

   | MS | Nội dung chính |
   |---|---|
   | M0 | Spike benchmark STT (tài liệu riêng); ra bảng đề xuất model Lite/Standard/Pro |
   | M1 | Scaffold Tauri + React + sidecar uv; sidecar lifecycle (spawn khi cần, tắt khi rảnh — NFR-06); IPC + token; CI build Windows |
   | M2 | WASAPI mic + loopback, chunk writer Opus chống crash (NFR-05), import file (ffmpeg đóng gói), chọn thư mục data, SQLite schema + migration, màn Storage |
   | M3 | Luồng bắt đầu họp (FR-START), STT realtime + chạy lại sau họp ở nền (tạm dừng/tiếp tục), glossary vào hotwords, đo CPU/RAM theo NFR-02 |
   | M4 | Diarization sau họp, đổi nhãn speaker, playback theo segment, sửa transcript, FTS search, workspace/tag |
   | M5 | Lớp LLM provider (llama.cpp / OpenAI-compatible / Agent CLI adapter), templates + upload mẫu biên bản, chat theo cuộc họp, dịch realtime theo câu, cảnh báo gửi cloud, keychain, export MD/DOCX/PDF. **Spike phụ: benchmark model dịch** |
   | M6 | `hopho-mcp` + tool set, config/skill mẫu cho Claude Desktop/Code, Codex, Antigravity |
   | M7 | Overlay always-on-top, tray, hotkey toàn cục, bookmark, ghi chú tay có timestamp |
   | M8 | Benchmark phần cứng, wizard onboarding, Model Manager (tải tiếp được khi rớt mạng, mirror), add-on CUDA, bộ cài `.msi`, auto-update |
   | M9 | Test hiệu năng trên máy Gen10 thật, test họp 3h, xử lý lỗi, hoàn thiện i18n vi/en, tài liệu người dùng, release v0.1 |

6. **Chiến lược test:** unit test (pytest cho sidecar, `cargo test`, vitest cho UI), integration test cho IPC, test hiệu năng tự động bằng audio mẫu 3h, checklist test tay.
7. **Rủi ro & cách giảm thiểu** theo từng milestone. Ví dụ: kích thước sidecar (ưu tiên CTranslate2/ONNX, tránh torch), pyannote gated, thay đổi CLI của agent.

> Kế hoạch **không ước lượng thời gian theo ngày**, vì chưa biết quy mô team. Chỉ có thứ tự và tiêu chí hoàn thành. Nếu bạn muốn có ước lượng thì báo mình.

---

### 3. Thiết kế Spike Benchmark STT

#### [NEW] `docs/plan/spike-stt-benchmark.md`

Dàn ý:

1. **Câu hỏi cần trả lời**
   - Model nào đạt NFR-02/03 (≤50% CPU, ≤4GB RAM, trễ ≤5s) cho chế độ realtime trên máy Gen10 chỉ có CPU?
   - Model nào cho độ chính xác tốt nhất khi chạy lại sau họp mà vẫn ≤ 1x thời lượng họp (NFR-04)?
   - Chất lượng với tiếng Việt và nói trộn Việt–Anh ra sao?
   - Runtime nào gọn nhất khi đóng gói (CTranslate2 / ONNX / whisper.cpp)?
2. **Model ứng viên** (cần xác minh license và khả năng hỗ trợ ngôn ngữ trong lúc làm spike):
   | Nhóm | Ứng viên | Runtime |
   |---|---|---|
   | Realtime | Whisper tiny/base/small (int8) | faster-whisper (CTranslate2) |
   | Realtime | Zipformer streaming (vi, en, zh…) | sherpa-onnx |
   | Sau họp | Whisper large-v3-turbo, medium (int8) | faster-whisper |
   | Sau họp | PhoWhisper (small/medium/large) | chuyển sang CTranslate2 |
   | So sánh runtime | whisper.cpp (cùng model) | whisper.cpp |
3. **Dataset**
   - **FLEURS** (CC-BY 4.0), tập test cho `vi_vn`, `en_us`, `cmn_hans_cn`, `ko_kr`. Mặc định lấy mẫu N câu mỗi ngôn ngữ để chạy nhanh. Script tự tải qua `huggingface_hub`.
   - **`data/custom/`**: user thả cặp `xxx.wav` + `xxx.txt` (ghi âm họp thật, nói trộn Việt–Anh). Thư mục này nằm trong `.gitignore`.
   - **Bài test dài:** ghép các câu FLEURS thành file 30–60 phút để đo độ ổn định và RAM.
4. **Chỉ số đo**
   | Chỉ số | Cách đo |
   |---|---|
   | WER (vi, en) / CER (zh, ko) | `jiwer`, chuẩn hoá (Unicode NFC, lowercase, bỏ dấu câu) |
   | RTF (tỉ lệ thời gian xử lý / thời lượng audio) | thời gian xử lý / thời lượng audio |
   | Trễ realtime | đẩy audio theo nhịp thời gian thực (chunk 0.5–1s), đo thời gian từ cuối câu nói đến lúc ra text |
   | CPU% trung bình/đỉnh, RAM đỉnh | `psutil`, lấy mẫu mỗi 0.5s |
   | Thời gian load model, dung lượng model trên đĩa | đo trực tiếp |
   | Giới hạn thread | chạy với 2/4/tất cả thread để mô phỏng máy yếu và chừa CPU cho Zoom |
5. **Cấu trúc code dự kiến** (sẽ code ở bước sau, *không thuộc lần này*):
   ```
   spikes/stt-benchmark/
   ├── pyproject.toml         # uv, .venv cục bộ
   ├── README.md
   ├── configs/models.yaml    # danh sách model + tham số
   ├── src/stt_bench/
   │   ├── datasets.py        # FLEURS + custom loader
   │   ├── engines/           # faster_whisper.py, sherpa.py, whisper_cpp.py (cùng interface)
   │   ├── metrics.py         # WER/CER + normalizer
   │   ├── monitor.py         # CPU/RAM sampler
   │   ├── streaming.py       # mô phỏng realtime
   │   └── run.py             # CLI
   ├── tests/                 # pytest cho normalizer/metrics/loader
   └── results/               # CSV/JSON + report.md (gitignored, trừ report cuối)
   ```
   Interface chung cho các engine:
   ```python
   class SttEngine(Protocol):
       name: str
       def load(self) -> None: ...
       def transcribe(self, audio: np.ndarray, sr: int, language: str | None) -> str: ...
       def stream(self, chunks: Iterable[np.ndarray], sr: int) -> Iterator[PartialResult]: ...  # optional
   ```
6. **Tiêu chí chọn model:** với mỗi gói Lite/Standard/Pro và mỗi chế độ realtime/sau họp, chọn model có WER tiếng Việt thấp nhất trong số các model thoả NFR. Hoà nhau thì ưu tiên model nhỏ hơn và license thoáng hơn.
7. **Đầu ra của spike:** `docs/reports/stt-benchmark-<date>.md` gồm bảng kết quả, đề xuất model theo gói, và ghi chú đóng gói runtime.
8. **Môi trường test:** máy dev i5-13500H (chạy sơ bộ) và **bắt buộc** ít nhất 1 máy Gen10/16GB chỉ CPU. Có thể thêm Apple Silicon và NVIDIA nếu có máy.
9. **Ngoài phạm vi spike:** diarization, dịch (benchmark ở M4/M5).

---

### 4. Đồng bộ task

#### [MODIFY] `tasks/todo.md`
Thêm checklist ngắn: các milestone M0–M9 (chỉ tiêu đề), trỏ link tới `docs/plan/mvp-plan.md`.

---

## Kiểm tra sau khi viết

- Đọc lại toàn bộ 3 file, đối chiếu từng quyết định đã chốt (gồm 14 câu grill-me và 5 ý chốt thêm) xem đã có mặt và không mâu thuẫn nhau.
- Kiểm tra cú pháp mermaid hợp lệ, và link giữa các tài liệu trỏ đúng đường dẫn.
- `git status` để xác nhận chỉ có các file Markdown nói trên bị thay đổi.
