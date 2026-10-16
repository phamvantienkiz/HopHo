# QA Test Cases — Release 1

**Tầng tài liệu**: Implementation Layer (`docs/implementation/release-1/qa/test-cases.md`)  
**Release**: release-1  
**Ngày cập nhật**: 2026-10-15  
**Nguồn tham chiếu**: [mvp-plan.md](../../../plan/mvp-plan.md) §6–§7 (DoD từng milestone), [srs.md](../srs.md) §3

Test tự động (pytest, cargo test, vitest) theo [technical-design.md](../technical-design.md) §4. Dưới đây là checklist test thủ công / hiệu năng trên máy chuẩn (i5 Gen10 / 16GB, chỉ CPU).

| ID | Milestone | Yêu cầu | Các bước | Kết quả mong đợi |
|---|---|---|---|---|
| TC-01 | M1 | NFR-06 | Chạy app, UI gọi sidecar; đóng app | Sidecar tắt theo; không tiến trình mồ côi |
| TC-02 | M2 | FR-AUD-05, NFR-05 | Ghi âm, kill process giữa chừng, mở lại app | Meeting `interrupted`, audio còn tới chunk cuối |
| TC-03 | M2 | NFR-01 | Ghi liên tục 3 giờ | Không lỗi, RAM không tăng dần |
| TC-04 | M2 | FR-AUD-03 | Import mp3/mp4/m4a/wav | Tạo meeting, audio chuẩn hóa Opus |
| TC-05 | M3 | NFR-02, NFR-03 | Realtime với audio mẫu 1h trên máy chuẩn | CPU ≤ 50%, RAM ≤ 4GB, trễ p95 ≤ 5s |
| TC-06 | M3 | NFR-04, FR-STT-07 | Chạy lại STT 1h audio; pause/resume | Hoàn tất ≤ 1x thời lượng; resume đúng checkpoint |
| TC-07 | M4 | FR-DIA-01..04, FR-REV-01..05 | Diarization, đổi tên speaker, click câu để nghe, tìm kiếm | Nhãn hợp lý; đổi tên áp dụng toàn bộ; search đúng |
| TC-08 | M5 | FR-LLM-07 | Upload mẫu .docx, sinh biên bản | DOCX đúng khung mẫu |
| TC-09 | M5 | FR-DATA-06 | Gọi cloud/agent | Luôn hiện hộp xác nhận điểm đến dữ liệu |
| TC-10 | M6 | FR-MCP-01..04 | Từ từng agent hỏi "tóm tắt cuộc họp sáng nay"; `save_summary` | Kết quả đúng; summary hiện trong app |
| TC-11 | M7 | FR-UX-02, FR-UX-03 | Overlay trên Zoom/Meet/Teams (windowed/borderless); hotkey khi app không focus | Overlay hiển thị; hotkey hoạt động |
| TC-12 | M8 | FR-INS-01..06 | Máy Windows sạch: cài `.msi` → wizard → ghi và tóm tắt | Hoàn tất không cần dòng lệnh |
| TC-13 | M9 | NFR-07 | Gây lỗi: mất quyền mic, rớt mạng khi tải model, đầy ổ | Mỗi lỗi có mã + hướng dẫn; không im lặng |
