# Database Migration — Release 1 (MVP Windows v0.1)

**Tầng tài liệu**: Implementation Layer (`docs/implementation/release-1/migration.md`)  
**Release**: release-1  
**Ngày cập nhật**: 2026-10-15  
**Nguồn tham chiếu**: [05-data-architecture.md](../../architecture/05-data-architecture.md)

---

## 1. Phạm vi

Release 1 là bản đầu tiên nên **không có breaking change** và không cần di chuyển dữ liệu từ phiên bản cũ.

## 2. Migration của Release 1

| File (`apps/desktop/src-tauri/src/storage/migrations/`) | Nội dung |
|---|---|
| `001_initial_schema.sql` | Toàn bộ bảng trong 05 §3: `workspaces`, `meetings`, `meeting_tags`, `audio_chunks`, `speakers`, `segments`, `bookmarks`, `notes`, `glossary_terms`, `templates`, `outputs`, `chat_messages`, `batch_jobs`, `action_items`, `app_settings` |
| `002_fts5_indexes.sql` | `segments_fts`, `outputs_fts` và các trigger đồng bộ (05 §4) |

## 3. Quy tắc

1. Chỉ **Rust Core** tạo/chạy migration (AD-4); Python (`hopho_store`) cập nhật model tương ứng trong cùng thay đổi.
2. Migration chỉ tiến (forward-only), mỗi file idempotent (`IF NOT EXISTS`).
3. Mọi thay đổi schema sau này phải cập nhật [05-data-architecture.md](../../architecture/05-data-architecture.md) trước, rồi thêm file migration mới và ghi vào tài liệu của release tương ứng.
