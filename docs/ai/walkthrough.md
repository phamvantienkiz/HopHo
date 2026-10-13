# Walkthrough — Tài liệu kế hoạch MVP & Spike STT

## Thay đổi

| File | Loại | Nội dung |
|---|---|---|
| [requirements.md](../project-idea/requirements.md) | Sửa | Gen10 là mốc NFR (Gen8/9 chỉ là tối thiểu); dịch bằng LLM tổng quát/chuyên dịch (FR-TRN-02/03); thêm FR-LLM-07 (user upload mẫu biên bản); model chốt qua benchmark; bỏ R6/OQ1/OQ2; đưa nhắc đồng thuận ghi âm vào out of scope |
| [mvp-plan.md](../plan/mvp-plan.md) | Mới | Quyết định kiến trúc AD-1..7 (gồm IPC localhost + token, `hopho-mcp` tách riêng), cấu trúc monorepo, sơ đồ runtime/sequence, data model, milestone M0–M9 kèm DoD, chiến lược test, rủi ro |
| [spike-stt-benchmark.md](../plan/spike-stt-benchmark.md) | Mới | Câu hỏi Q1–Q6, model ứng viên, dataset (FLEURS + FLEURS-long + custom), chỉ số đo và chuẩn hoá text, cấu trúc code + interface engine, CLI, môi trường đo, tiêu chí chọn theo gói Lite/Standard/Pro |
| `tasks/todo.md` | Sửa | Checklist M0–M9 trỏ về kế hoạch |

## Kiểm tra
- Đối chiếu 14 quyết định grill-me + 5 ý chốt thêm với cả 3 tài liệu: đã có đủ, không mâu thuẫn.
- Kiểm tra toàn bộ link tương đối giữa các tài liệu: tất cả tồn tại.
- `git status`: chỉ thay đổi trong `docs/` (riêng `tasks/` nằm trong `.gitignore`).
- Không có code hay package nào được tạo/cài.
