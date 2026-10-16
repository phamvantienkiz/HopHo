# Sổ cái Backlog (Story Index) — HopHo MVP Windows (Release 1)

**Tầng tài liệu**: Implementation Layer (`docs/implementation/release-1/backlog/story-index.md`)  
**Release**: release-1 (MVP Windows v0.1)  
**Nguồn tham chiếu**: [prd.md](../prd.md), [srs.md](../srs.md), [requirements.md](../../../project-idea/requirements.md), [mvp-plan.md](../../../plan/mvp-plan.md)  
**Ngày cập nhật**: 2026-10-15  
**Trạng thái**: Draft — chờ Product Owner duyệt  

---

## 1. Ma trận phân bổ User Story & Traceability

| Mã Story | Tiêu đề User Story | Persona | Ưu tiên | PRD FR Ref | Số AC | Rào chắn Kiến trúc (Guardrails) | Trạng thái |
|---|---|---|---|---|---|---|---|
| [**US-001**](./stories/US-001.md) | Khởi tạo phiên họp & Cấu hình nguồn âm | Người dùng văn phòng | **P0 (Must)** | FR-START-01, FR-STT-02, FR-AUD-04 | 4 ACs | 05-data, 07-security, 09-design, 10-folder | Ready for Dev |
| [**US-002**](./stories/US-002.md) | Ghi âm Microphone & WASAPI Loopback hệ thống | Người dùng văn phòng | **P0 (Must)** | FR-AUD-01, FR-AUD-02 | 4 ACs | 07-security, 09-design, 10-folder | Ready for Dev |
| [**US-003**](./stories/US-003.md) | Ghi âm phân đoạn Opus an toàn & Phục hồi crash | Hệ thống | **P0 (Must)** | FR-AUD-05, NFR-05 | 4 ACs | 05-data, 09-design, 10-folder | Ready for Dev |
| [**US-004**](./stories/US-004.md) | Nhập (Import) tệp âm thanh/video có sẵn | Quản lý dự án | **P1 (Should)** | FR-AUD-03 | 4 ACs | 04-deploy, 05-data, 09-design, 10-folder | Ready for Dev |
| [**US-005**](./stories/US-005.md) | Phiên âm thời gian thực (Realtime Transcript) trên Overlay | Thành viên dự họp | **P0 (Must)** | FR-STT-01..04, FR-UX-02 | 4 ACs | 05-data, 08-integ, 09-design, 10-folder | Ready for Dev |
| [**US-006**](./stories/US-006.md) | Phiên dịch câu trực tiếp trên cửa sổ Overlay | Người tham gia đa ngữ | **P1 (Should)** | FR-TRN-01..05 | 4 ACs | 05-data, 06-ai, 09-design, 10-folder | Ready for Dev |
| [**US-007**](./stories/US-007.md) | Đánh dấu Bookmark & Ghi chú tay có Timestamp | Thư ký cuộc họp | **P1 (Should)** | FR-UX-03, 04, 05 | 4 ACs | 05-data, 07-security, 09-design, 10-folder | Ready for Dev |
| [**US-008**](./stories/US-008.md) | Chạy lại STT chuẩn sau họp hỗ trợ Tạm dừng/Chạy tiếp | Thư ký cuộc họp | **P0 (Must)** | FR-STT-01, 05, 07, NFR-04 | 4 ACs | 03-service, 05-data, 09-design, 10-folder | Ready for Dev |
| [**US-009**](./stories/US-009.md) | Phân tách người nói (Diarization) & Đổi tên hàng loạt | Thư ký cuộc họp | **P0 (Must)** | FR-DIA-01..04 | 4 ACs | 05-data, 07-security, 09-design, 10-folder | Ready for Dev |
| [**US-010**](./stories/US-010.md) | Nghe lại theo câu (Playback) & Sửa transcript inline | Người dùng văn phòng | **P0 (Must)** | FR-REV-01, FR-REV-02 | 4 ACs | 05-data, 08-integ, 09-design, 10-folder | Ready for Dev |
| [**US-011**](./stories/US-011.md) | Quản lý từ điển Workspace Glossary nạp vào STT & LLM | Kỹ sư phần mềm | **P1 (Should)** | FR-GLO-01, 02, FR-STT-06 | 4 ACs | 05-data, 06-ai, 09-design, 10-folder | Ready for Dev |
| [**US-012**](./stories/US-012.md) | Sinh Tóm tắt & Action Items theo Template tiêu chuẩn | Quản lý dự án | **P0 (Must)** | FR-LLM-01, 02, 04, 06 | 4 ACs | 05-data, 06-ai, 09-design, 10-folder | Ready for Dev |
| [**US-013**](./stories/US-013.md) | Trích xuất cấu trúc & Điền nội dung theo mẫu biên bản tải lên | Thư ký cuộc họp | **P0 (Must)** | FR-LLM-07 | 4 ACs | 05-data, 06-ai, 09-design, 10-folder | Ready for Dev |
| [**US-014**](./stories/US-014.md) | Tích hợp linh hoạt 3 nhóm Provider LLM (Local, Cloud, Agent) | Kỹ sư phần mềm | **P0 (Must)** | FR-LLM-08, FR-DATA-06, 07 | 4 ACs | 07-security, 09-design, 10-folder | Ready for Dev |
| [**US-015**](./stories/US-015.md) | Trò chuyện hỏi đáp (Chat in-meeting) dựa trên transcript | Quản lý dự án | **P1 (Should)** | FR-LLM-03 | 4 ACs | 05-data, 06-ai, 09-design, 10-folder | Ready for Dev |
| [**US-016**](./stories/US-016.md) | Cung cấp máy chủ `hopho-mcp` cho Agent ngoài qua Stdio | Kỹ sư phần mềm | **P0 (Must)** | FR-MCP-01..04 | 4 ACs | 03-service, 05-data, 07-security, 08-integ, 09-design, 10-folder | Ready for Dev |
| [**US-017**](./stories/US-017.md) | Tìm kiếm toàn văn FTS5 & Quản trị lưu trữ dọn dẹp | Người dùng văn phòng | **P0 (Must)** | FR-REV-04, 05, FR-DATA-04 | 4 ACs | 05-data, 09-design, 10-folder | Ready for Dev |
| [**US-018**](./stories/US-018.md) | Quản lý Thư mục dữ liệu cô lập & Chính sách No-telemetry | Quản trị bảo mật | **P0 (Must)** | FR-DATA-01, 02, 03, 05 | 4 ACs | 04-deploy, 05-data, 07-security, 09-design, 10-folder | Ready for Dev |
| [**US-019**](./stories/US-019.md) | Cài đặt ứng dụng `.msi` & Trình hướng dẫn Onboarding Wizard | Người dùng văn phòng | **P0 (Must)** | FR-INS-01, FR-INS-02, FR-STT-03 | 4 ACs | 04-deploy, 09-design, 10-folder | Ready for Dev |
| [**US-020**](./stories/US-020.md) | Quản lý Mô hình AI (Model Manager) & Add-on CUDA | Kỹ sư phần mềm | **P0 (Must)** | FR-INS-03, 04, 05, 06 | 4 ACs | 04-deploy, 05-data, 06-ai, 09-design, 10-folder | Ready for Dev |
| [**US-021**](./stories/US-021.md) | Cửa sổ chính, quản lý Workspace/Tag & Không chạy ngầm khi rảnh | Người dùng văn phòng | **P0 (Must)** | FR-UX-01, FR-REV-03, FR-UX-06, NFR-06 | 4 ACs | 03-service, 05-data, 09-design, 10-folder | Ready for Dev |
| [**US-022**](./stories/US-022.md) | Độ bền phiên họp 3 giờ & Báo lỗi tường minh | Người dùng văn phòng | **P0 (Must)** | NFR-01, NFR-07 | 4 ACs | 03-service, 07-security, 09-design, 10-folder | Ready for Dev |

---

## 2. Thống kê & Độ bao phủ Kịch bản kiểm thử (Coverage Metrics)

- **Tổng số User Stories**: 22 Stories độc lập đạt chuẩn INVEST.
- **Tổng số Acceptance Criteria (ACs)**: 88 Kịch bản kiểm thử chi tiết Given-When-Then.
- **Phân bổ kịch bản**: mỗi story có 4 AC gồm happy path và các kịch bản validation, edge case, lỗi/ràng buộc phi chức năng (chi tiết xem từng story).
- **Độ bao phủ Rào chắn Kiến trúc**: 100% User Story có mục Architecture Guardrails và đều cite tối thiểu `09-design-principles` và `10-folder-structure`; cột Guardrails ở bảng trên liệt kê các tài liệu kiến trúc thực sự được trích dẫn trong từng story.

---

## 3. Danh mục Câu hỏi mở (Open — chốt ở M0/M4/M5)

Toàn bộ các câu hỏi mở từ bản khảo sát yêu cầu ban đầu (OQ-01 đến OQ-05 trong `prd.md`) đã có phương án giảm thiểu trong các User Story (chỉ đóng khi milestone tương ứng hoàn tất):
- `[ ] OQ-01`: Được xử lý theo thiết kế trong [US-001](./stories/US-001.md) & [US-005](./stories/US-005.md) — Ứng dụng đưa ra khuyến nghị chế độ "Chỉ ghi âm" hoặc dùng Cloud LLM dịch nếu CPU yếu.
- `[ ] OQ-02`: Được xử lý theo thiết kế trong [US-020](./stories/US-020.md) — Sử dụng runtime CTranslate2 và ONNX, cô lập torch để gói sidecar gọn nhẹ.
- `[ ] OQ-03`: Được xử lý theo thiết kế trong [US-009](./stories/US-009.md) — Mặc định sử dụng engine `sherpa-onnx` không gated cho người dùng phổ thông.
- `[ ] OQ-04`: Được xử lý theo thiết kế trong [US-020](./stories/US-020.md) — Kiểm tra giấy phép tương thích Apache 2.0 trước khi tải model vào máy.
- `[ ] OQ-05`: Được xử lý theo thiết kế trong [US-014](./stories/US-014.md) — Bổ sung cơ chế version probe trước khi kích hoạt lệnh Agent CLI.

---

## 4. Truy vết NFR & Yêu cầu xuyên suốt

| NFR / FR | Story chính |
|---|---|
| NFR-01 (3 giờ) | [US-022](./stories/US-022.md) |
| NFR-02, NFR-03 | [US-005](./stories/US-005.md) |
| NFR-04 | [US-008](./stories/US-008.md) |
| NFR-05 | [US-003](./stories/US-003.md) |
| NFR-06 | [US-021](./stories/US-021.md) |
| NFR-07 | [US-022](./stories/US-022.md) |
| FR-UX-01, FR-UX-06, FR-REV-03 | [US-021](./stories/US-021.md) |
