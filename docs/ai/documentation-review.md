# Báo cáo kiểm tra bộ tài liệu 3 tầng so với requirements / MVP plan / spike

> Ngày: 2026-10-15 · Chuẩn đối chiếu: [requirements.md](../project-idea/requirements.md), [mvp-plan.md](../plan/mvp-plan.md), [spike-stt-benchmark.md](../plan/spike-stt-benchmark.md), skill `documentation` v2.2.

## 0. Phạm vi & kết luận

**Đã đọc toàn bộ:** `business/brd`, `business/roadmap`, `architecture/01–10 + README + ADR-001`, `implementation/release-1/{prd, srs, technical-design, story-index}`, và US-005, US-016, US-020. Quét tự động cấu trúc của cả 20 story.
**Chưa đối chiếu sâu:** `glossary.md`, `stakeholders.md`, nội dung chi tiết 17 story còn lại.

**Kết luận:** Khung 3 tầng đúng skill, các quyết định cốt lõi (AD-1…AD-7, Rust sở hữu schema, token loopback, no voice profile, no telemetry) được giữ nhất quán. Tuy nhiên bộ tài liệu có **3 nhóm vấn đề cần sửa trước khi dùng làm nguồn cho coding agent**:

1. Nhiều "quyết định/kết quả" bị **viết như đã chốt** trong khi plan nói phải chốt bằng benchmark (M0/M4/M5).
2. **Mâu thuẫn nội bộ** giữa các tài liệu (giao thức WS, token, timeout, đường dẫn báo cáo…).
3. **Lệch traceability** so với requirements (ID FR, danh sách FR, Out-of-scope bị thiếu).

---

## 1. Nghiêm trọng (sai lệch so với requirements/plan hoặc tự mâu thuẫn)

| # | Vấn đề | Bằng chứng | Đề xuất |
|---|---|---|---|
| C1 | **OQ-01…05 bị ghi `Resolved`** dù chưa chạy M0/M4/M5. Requirements R1–R5 và plan coi đây là rủi ro còn mở. | [prd.md L227-231](../implementation/release-1/prd.md), [story-index.md L53-57](../implementation/release-1/backlog/story-index.md) | Đổi thành `Open – mitigated by M0/M4/M5`; chỉ đóng khi có báo cáo benchmark. |
| C2 | **BOTEC viết như đã đạt NFR** ("P95 ≈ 3.5–4.2s", "CPU 35–45%", "Đạt chỉ tiêu"). Đây là ước tính; R1 nói có thể **không đạt**. | [02-high-level L127-129](../architecture/02-high-level-architecture.md) | Gắn nhãn "ước lượng, chờ xác nhận M0"; bỏ chữ "Đạt". |
| C3 | **Model cụ thể bị ghim sớm** (Whisper small, Qwen2.5 1.5B/3B, sherpa làm mặc định diarization, "tiny.en/vi") trái AD-7 và FR-TRN-03 (model dịch chốt ở M5; diarization chốt ở M4). `tiny.en` là model chỉ tiếng Anh. | [02 L36,L129](../architecture/02-high-level-architecture.md), [04 L57-59](../architecture/04-deployment-architecture.md), [06 L115](../architecture/06-ai-architecture.md), [srs.md L55](../implementation/release-1/srs.md) | Ghi "ứng viên", trỏ tới báo cáo M0/M4/M5; sửa `tiny.en/vi`. |
| C4 | **Bảng license trong 06 tuyên bố đã xác minh** (OQ-04 Resolved), trong khi spike yêu cầu xác minh lúc làm M0. Cần kiểm tra lại, ví dụ Qwen2.5 **3B** (không phải mọi cỡ đều Apache 2.0), PhoWhisper, model 3D-Speaker. | [06 L163-175](../architecture/06-ai-architecture.md), [spike L28](../plan/spike-stt-benchmark.md) | Đánh dấu "chưa xác minh"; xác minh thật ở M0. |
| C5 | **Token "dùng một lần"** (PRD/03) lệch plan AD-2 "token sinh **mỗi lần chạy**". Một lần dùng sẽ hỏng khi reconnect hoặc khi mở song song WS realtime + batch. | [prd.md L213](../implementation/release-1/prd.md), [03 L99](../architecture/03-service-architecture.md), [mvp-plan L30](../plan/mvp-plan.md) | Sửa thành "token theo phiên sidecar (per-launch)". |
| C6 | **Giao thức WS không thống nhất**: endpoint `/ws/session` (08) vs `/ws/stt` (US-005); action `start_session` (08) vs `start_stream` (TDD); event `{event:"partial_transcript"}` (08) vs `{type:"stt_result"}` (03). `api/v1.md` đang rỗng. | [08 L40-98](../architecture/08-integration-architecture.md), [US-005 L55](../implementation/release-1/backlog/stories/US-005.md), [TDD L50-52](../implementation/release-1/technical-design.md), [03 L118-126](../architecture/03-service-architecture.md) | Chọn 08 làm nguồn duy nhất; sửa 03/TDD/US-005; điền REST/WS contract của sidecar (batch/pause…). |
| C7 | **Thiếu Codex** trong integration: FR-MCP-02 và plan liệt kê Claude Desktop, Claude Code, **Codex**, Antigravity; 08 và 10 chỉ có 3. | [08 L117-140](../architecture/08-integration-architecture.md), [10 L173-176](../architecture/10-folder-structure.md), [mvp-plan L64](../plan/mvp-plan.md) | Thêm `codex` config. Kiểm tra lại format config từng agent (Claude Code dùng `.mcp.json`, không phải `mcp_config.json`). |
| C8 | **Thiếu mục Out-of-scope "Nhắc đồng thuận ghi âm / disclaimer pháp lý"** của requirements §7. | [requirements L253](../project-idea/requirements.md) vs [brd L72-79](../business/brd.md), [prd L124-132](../implementation/release-1/prd.md) | Bổ sung vào BRD/PRD. |
| C9 | **KPI "WER ≤ 12%" không có trong requirements/plan**; lặp lại ở 06. Spike chỉ dùng WER để xếp hạng, không đặt ngưỡng. | [brd L51](../business/brd.md), [06 L78](../architecture/06-ai-architecture.md) | Xoá hoặc ghi "TBD sau M0" và hỏi PO. |
| C10 | **Batch checkpoint không có trong schema**: 06 nói lưu `last_processed_ms` "vào database", TDD nói "vào cache"; DDL không có cột/bảng nào. `meetings.status` cũng không biểu diễn "paused". | [06 L82](../architecture/06-ai-architecture.md), [TDD L136](../implementation/release-1/technical-design.md), [05 DDL](../architecture/05-data-architecture.md) | Thêm bảng/cột checkpoint (vd `batch_jobs`) vào 05 + migration; sửa TDD. |
| C11 | **FTS chỉ index segment `final`** → họp chỉ chạy realtime (user từ chối chạy lại) sẽ **không tìm được** (FR-REV-04). Chưa định nghĩa realtime segment có bị thay bằng final hay không. | [05 L314-319](../architecture/05-data-architecture.md) | Quy định vòng đời `realtime`→`final` và index cả hai (hoặc promote realtime). |
| C12 | **Action items & settings không có nơi lưu**: `get_action_items` trả mảng có cấu trúc (việc/người/hạn) nhưng `outputs` chỉ là `content_md`; không có bảng settings (mặc định FR-START, vị trí overlay US-005 AC3, ngưỡng xoá audio FR-REV-05). | [08 L114](../architecture/08-integration-architecture.md), [05](../architecture/05-data-architecture.md) | Quyết định: bảng `action_items` hay parse từ output; thêm settings store vào 05. |

## 2. Trung bình (lệch traceability / vi phạm skill)

| # | Vấn đề | Bằng chứng |
|---|---|---|
| M1 | **ID bị lệch so với requirements**: `FR-LLM-08` không tồn tại (requirements chỉ 01–07); `FR-MCP-02/03/04` bị đánh số lại (requirements: 02=plugin/config, 03=connector ủy quyền, 04=RAG/search ủy quyền; SRS: 02=tool list, 03=config, 04=connector; mất ý RAG). `FR-START-01` cũng là ID mới. | [srs L80,L86-89](../implementation/release-1/srs.md) vs [requirements L135-160](../project-idea/requirements.md) |
| M2 | **PRD §8 nói "22 FR"**; requirements có ~60 FR. PRD §5 chỉ liệt kê một phần, nhưng story-index vẫn trích "PRD FR Ref" cho các FR không có trong bảng đó (FR-STT-02, FR-UX-04…). | [prd L240](../implementation/release-1/prd.md), [story-index L15-34](../implementation/release-1/backlog/story-index.md) |
| M3 | **PRD In-scope thiếu** chat hỏi–đáp, glossary, workspace/tag, bookmark/ghi chú, export, quản lý storage/privacy, dù plan §1 nói phạm vi = toàn bộ FR. | [prd L112-122](../implementation/release-1/prd.md) |
| M4 | **Ma trận truy vết SRS lệch plan**: M1 gán `FR-INS-01` (plan: M8) và `FR-DATA-07` (plan: M5); M2 thiếu `FR-DATA-05`; M5 gán NFR-01 không có trong plan. | [srs L161-165](../implementation/release-1/srs.md) vs [mvp-plan L169-211](../plan/mvp-plan.md) |
| M5 | **FR chưa thấy được map vào story** trong cột FR Ref của index: FR-UX-01, FR-UX-06, FR-REV-03, và NFR-01/04/06/07. | [story-index](../implementation/release-1/backlog/story-index.md) |
| M6 | **Story-index nói "100% story có đầy đủ guardrails 05/07/08/09/10"** nhưng quét thực tế: US-016 **không cite 09** (skill bắt buộc 09+10 cho cấu trúc code); US-002/014/016/019 không cite 05; hầu hết story không cite 07/08. Tuyên bố bị sai. | [story-index L46](../implementation/release-1/backlog/story-index.md), [US-016 L55-61](../implementation/release-1/backlog/stories/US-016.md) |
| M7 | **Story Points gán sẵn** (3 SP cho US-005/016/020…) trái plan §9 "không ước lượng thời gian vì chưa biết quy mô team". | [US-005 L7](../implementation/release-1/backlog/stories/US-005.md), [mvp-plan L288](../plan/mvp-plan.md) |
| M8 | **Timeout Agent CLI mâu thuẫn**: 60s (sơ đồ) vs 120s (text/09). Cờ lệnh (`claude -p --no-stream`, `codex exec --quiet`, `agy run --headless`) chưa xác minh trong khi R5 cảnh báo cú pháp thay đổi. | [08 L152-164](../architecture/08-integration-architecture.md) |
| M9 | **Đường dẫn báo cáo benchmark mâu thuẫn**: plan/spike → `docs/reports/stt-benchmark-<date>.md`; 10 → `spikes/{name}/`. Thư mục `docs/reports` (và `plan/`, `project-idea/`, `research/`, `ai/`) không có trong cấu trúc docs của 10. | [mvp-plan L167](../plan/mvp-plan.md), [10 L194-195](../architecture/10-folder-structure.md) |
| M10 | **10 thiếu PDF export** (FR-LLM-06) trong `export/` (chỉ docx/md). Cây thư mục data ở 04 bỏ `data/` và đặt `hopho.db` ở root (FR-DATA-02 liệt kê `data/ models/ audio/ exports/ logs/`). | [10 L134-136](../architecture/10-folder-structure.md), [04 L90-114](../architecture/04-deployment-architecture.md) |
| M11 | **Vi phạm layer integrity**: kiến trúc (05) viện dẫn story `US-017`; BRD (business) chứa thuật ngữ kỹ thuật (WASAPI, sidecar, FTS5, `kill -9`, `.msi`) — skill nói BRD không mô tả công nghệ. | [05 L293](../architecture/05-data-architecture.md), [brd L42,L55-56](../business/brd.md) |
| M12 | **Update check vs "no phone-home"**: 07 cấm mọi ping tự động, nhưng 04/US-020 có Tauri Updater kiểm tra GitHub; tải model từ Hugging Face cũng là egress. 01-system-context không liệt kê Hugging Face/GitHub/Agent CLI là hệ thống ngoài. | [07 L57](../architecture/07-security-architecture.md), [04 L120](../architecture/04-deployment-architecture.md), [01 L18-21](../architecture/01-system-context.md) |
| M13 | **Trạng thái "Approved" tự gán** (BRD, PRD "96/100") trong khi requirements còn "Draft v1 — chờ review". Chưa có cổng duyệt của bạn. | [brd L7](../business/brd.md), [prd L8-9](../implementation/release-1/prd.md), [requirements L5](../project-idea/requirements.md) |

## 3. Nhẹ / dọn dẹp

- 7 file **0 byte**: `architecture/11–14`, `release-1/{migration, api/v1, frontend/ui-spec, qa/test-cases}`, `views/README.md`. Architecture README vẫn trỏ tới 11–14 như đã có nội dung, trong khi 09/10 đã chốt stack FE (Zustand, Tailwind, shadcn, pnpm) mà requirements/plan không nêu. Plan §7 còn yêu cầu checklist test tay trong `docs/`.
- Skill đặt tên `{feature}-story-index.md`; thực tế là `story-index.md`.
- Ước lượng chưa có nguồn: "tăng tốc 3–5x", "RTF < 0.2" (04, US-020), "Tauri 15–25MB".
- **Trùng lặp SSoT**: mvp-plan §3/§5 (cấu trúc repo, data model) và requirements giờ trùng với 05/10; nên đánh dấu plan/requirements là nguồn lịch sử và trỏ sang tài liệu tầng Architecture.
- FR-LLM-02 yêu cầu template là *file* Markdown, kiến trúc lưu `prompt_md` trong DB — cần chốt một cách.
- Công thức LaTeX (`$...$`) trong tài liệu kiến trúc có thể không hiển thị trên mọi renderer.

## 4. Những điểm đã khớp tốt

- Roadmap M0–M9 + phụ thuộc khớp mvp-plan §6.
- AD-1…AD-7, bảng tier Lite/Standard/Pro khớp spike §7, chuỗi pause/resume, no voice profile, Windows Job Object, keychain, consent gate.
- R1–R5 → OQ-01…05 ánh xạ đúng nội dung (chỉ sai trạng thái, xem C1).
- 20 story đều có User Story, 4 AC Given-When-Then, mục Architecture Guardrails, Fact-list; không có HTML nhúng (đúng Pure Markdown).

## 5. Thứ tự sửa đề xuất

1. C1–C4, C9 (sự thật/trạng thái) → 2. C5–C7, M8 (mâu thuẫn giao thức) → 3. C10–C12 (khoảng trống schema) → 4. M1–M6 (traceability) → 5. còn lại.

## 6. Câu hỏi cần bạn quyết định

1. KPI WER ≤ 12%: giữ, bỏ, hay đặt TBD sau M0?
2. Action items: bảng riêng `action_items` hay parse từ `outputs`?
3. Có muốn tôi sửa trực tiếp các tài liệu theo danh sách trên (từng nhóm), hay chỉ giữ báo cáo này?
