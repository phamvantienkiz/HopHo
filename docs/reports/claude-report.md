Tôi đã sửa trực tiếp các tài liệu theo báo cáo review. Tôi đã quét lại bằng grep những chuỗi sai cũ (OQ Resolved, "dùng một
lần", /ws/stt, WER ≤ 12%…) và không còn sót. Chưa chạy trình kiểm tra Markdown hay link.

## Hai điểm bạn nhờ tôi quyết định

• KPI WER ≤ 12%: đây là ngưỡng chất lượng nhận dạng giọng nói tiếng Việt, và con số 12% không có trong requirements hay
plan. Tôi bỏ ngưỡng cứng. WER chỉ dùng để xếp hạng model, và ngưỡng mục tiêu sẽ chốt sau khi có kết quả benchmark M0. Đã
sửa trong BRD, tài liệu 06 và US-008.
• Action items: tôi chọn bảng riêng action_items (task, assignee, due_date, done). Khi sinh theo template "Action items",
LLM trả JSON và sidecar ghi vào bảng này, đồng thời lưu bản Markdown vào outputs. Tool MCP get_action_items đọc từ bảng đó.
Tóm tắt do agent ngoài gửi qua save_summary chỉ ghi vào outputs với provider='external_agent'.
• Nhắc đồng thuận ghi âm: giữ nguyên như bạn yêu cầu, không thêm vào Out-of-scope.

## Các nhóm đã sửa

• Trạng thái và kết quả chưa chốt:
• OQ-01…05 đổi thành Open, kèm milestone chốt (M0/M4/M5).
• BRD, PRD, TDD đổi từ Approved sang Draft — chờ duyệt.
• BOTEC được ghi rõ là ước lượng chưa đo.
• Model và license chuyển thành "ứng viên / chưa xác minh". Sửa tiny.en, và đánh dấu Qwen cỡ 3B cần kiểm tra license
riêng.
• Mâu thuẫn:
• Token đổi thành "theo phiên sidecar".
• Giao thức WebSocket thống nhất theo tài liệu 08, và bổ sung mục REST batch.
• Timeout Agent CLI thống nhất 120s; cờ lệnh ghi là cấu hình adapter. Adapter còn lại phải xác minh bằng version probe.
• Đường dẫn báo cáo benchmark thống nhất là docs/reports/.
• Schema:
• Thêm bảng batch_jobs (checkpoint pause/resume), action_items và app_settings.
• Thêm cột prompt_path vào templates.
• FTS giờ index cả segment realtime và final, kèm quy tắc ưu tiên final khi truy vấn.
• Traceability:
• Thêm FR-LLM-08 và FR-START-01 vào requirements.md.
• Đánh số lại FR-MCP trong SRS cho khớp requirements.
• Sửa lại ma trận truy vết, phạm vi trong PRD và thêm Codex cho MCP.
• Bỏ Story Points và bỏ viện dẫn US/PRD trong tầng Architecture.
• Bổ sung:
• Thêm US-021 (cửa sổ chính, Workspace/Tag, không chạy ngầm) và US-022 (phiên 3 giờ, báo lỗi tường minh) để phủ các
FR/NFR còn thiếu. Tổng giờ là 22 story, 88 AC.
• Cột Guardrails của index được tạo lại từ nội dung story thật, và mọi story đều cite tài liệu 09 và 10.
• Điền nội dung cho 9 file đang trống (architecture/11–14, migration.md, api/v1.md, ui-spec.md, test-cases.md,
views/README.md).

## Cần bạn lưu ý

• Tài liệu 14 (Design Language): để TBD cho bảng màu, kiểu chữ và lưới, vì chưa có quyết định nào để ghi.
• Tên file story-index.md: tôi giữ nguyên. Skill gợi ý {feature}-story-index.md, nhưng file này là sổ cái cho cả release
chứ không riêng một feature.
• Chưa kiểm cấu hình MCP và cờ CLI: cấu hình MCP cho Codex, Claude Code và các cờ CLI chưa được kiểm tra với phiên bản thật
của từng agent. Tôi đã ghi chú trong tài liệu 08.
• Cập nhật kiểm tra phiên bản (mới): tôi đặt mặc định tắt, hỏi ở Onboarding, để tránh mâu thuẫn với nguyên tắc không phone-
home. Bạn có muốn đổi mặc định không?
• Báo cáo review: docs/ai/documentation-review.md vẫn mô tả trạng thái trước khi sửa.
• Git: thư mục docs mới chưa được commit. Bạn nên commit trước khi tiếp tục để có điểm quay lại.
