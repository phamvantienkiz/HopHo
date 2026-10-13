1. Loại sản phẩm: Desktop app sử dụng cho PC, laptop Windows, Mac (Linux)
2. Yêu cầu phần cứng: Chạy được trên máy chỉ có Only CPU (các dòng Intel, AMD), Apple Silicon, máy có GPU NVIDIA. các đời CPU tối thiểu ngang Intel Core I5 - Gen8 trở lên, ưu tiên Gen 10 trở lên (AMD ngang cấp), RAM tối thiểu 16GB.
3. Yêu cầu cài đặt: Dễ cài đặt cho các đối tượng người dùng lowcode, nocode.
4. Bản quyền và phân phối: Giai đoạn này ưu tiên Open-source với License Apache 2.0
5. Idea sản phẩm:

- Sử dụng các model STT, LLM nhỏ vừa đủ chạy local trên máy cho các tác vụ chính, có thể sử dụng các model của các bên thứ 3 với chuẩn API OpenAI. Tối ưu nhất là hybrid STT là local, LLM cloud, có thể cắm trực tiếp vào Claude Desktop/Claude code, Codex, Antigravity,... sẵn trên máy user thông qua plugin, skill, mcp tùy bên.
- Các cuộc họp (meeting) trực tiếp tại văn phòng thì ưu tiên ghi âm và chạy STT realtime để save meeting note, có tùy chọn gen trực tiếp văn bản realtime (tất nhiên cho phép độ trễ tùy vào cấu hình máy), có tùy chọn dịch nếu cuộc họp có ngôn ngữ nước ngoài (tự động dịch về ngôn ngữ native).
- Cuộc họp kết thúc thì có các tùy chọn tóm tắt, lọc ý chính, lọc ra các todo list cần làm,... viết báo cáo cuộc họp,... Đối với các user có cắm vào các Agent như Claude, Codex, Antigravity và sử dụng các model trong đó với các gói plan thì sẽ có nhiều tùy chọn hơn tùy vào yêu cầu và năng lực của Agent đó.
- Hỗ trợ đa ngôn ngữ, tập trung vào tiếng Việt, Anh, Trung, Hàn
