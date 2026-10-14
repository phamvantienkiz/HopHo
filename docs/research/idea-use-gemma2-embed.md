I turned Google DeepMind's EmbeddingGemma 2 into a speech-to-text model. It turned out to be one of the fastest on-device transcription models you can run: an hour of audio in 13 seconds on a base MacBook Air.

Embedding models turn text, images or audio into vectors. Their main use is search, along with clustering and recommendations; transcription is normally a separate, much heavier model. But in March, Google put Gemini Embedding 2 in its API, and it maps text, images, audio and video into one shared space: a spoken word lands right next to the written word.

That gave me an idea. If the sound of a word sits next to the word itself, you should be able to read a transcript straight out of the embedding model. I couldn't try that through an API. Last week Google released EmbeddingGemma 2 as an open model, so I finally did.

The setup is simple. The model stays untouched. A small reader on top takes each slice of audio, finds the nearest word in the model's own vocabulary, and strings the words together. So one small model now gives you both the meaning and the transcript.

It worked better than I hoped:
→ 288× real time on a base M3 MacBook Air: an hour of audio in about 13 seconds. On the same machine, that is about twice as fast as IBM's Granite Speech TurboCTC.
→ Under two hours of training on 8×H100, on openly licensed speech.
→ Everything stays on the Mac.

It isn't the most accurate model out there yet; dedicated speech models still make fewer mistakes. But for an embedding model that was never trained to transcribe, it is surprisingly good.

We also built a free Mac App on it. The link is in the first comment. You can download and test it.

---

# Fine-tune EmbeddingGemma để chuyển audio thành text: quy trình thực tế

Có thể huấn luyện trên Kaggle hoặc Google Colab, nhưng cách làm trong bài LinkedIn khác khá nhiều so với fine-tune LLM thông thường bằng Unsloth.

Điểm thú vị nhất trong bài đăng là tác giả không biến EmbeddingGemma thành một mô hình sinh văn bản theo cách thông thường. Thay vào đó, họ tận dụng khả năng biểu diễn audio và text trong không gian vector, sau đó huấn luyện một bộ đọc nhỏ để chuyển biểu diễn audio thành từ ngữ.

Tôi sẽ phân tích theo ba phần: kiến trúc mô hình, dữ liệu huấn luyện cần chuẩn bị và cách triển khai trên GPU cloud.

## 1\. Hiểu đúng kiến trúc mà tác giả sử dụng

Audio đầu vào

Ví dụ: một đoạn nói dài 5 giây

EmbeddingGemma

Encoder audio → vector biểu diễn

Giữ nguyên trọng số gốc

Reader / prediction head

Được huấn luyện để dự đoán từ tương ứng

Văn bản đầu ra

Ví dụ: "Hôm nay trời đẹp"

Ý tưởng dựa trên một trực giác quan trọng:

- Một encoder audio biến âm thanh thành các vector đặc trưng.

- Một encoder text biểu diễn từ ngữ thành vector trong cùng không gian hoặc một không gian có thể so sánh.

- Nếu biểu diễn audio chứa đủ thông tin để phân biệt các từ, ta có thể học cách ánh xạ nó sang từ tương ứng.

Tuy nhiên, có một chi tiết kỹ thuật cần phân biệt: vector embedding của cả câu không đồng nghĩa với việc có thể đọc ra transcript của câu đó. Nếu mô hình chỉ cung cấp một vector tổng hợp cho cả đoạn audio, thông tin về thứ tự từ có thể đã bị mất hoặc nén quá nhiều.

Do đó, để thực hiện transcription, hệ thống phải có biểu diễn audio ở mức thời gian đủ chi tiết, hoặc một cơ chế khác để dự đoán chuỗi từ theo thứ tự. Đây là một trong những điểm quyết định thành công của phương pháp.

Theo tài liệu chính thức, EmbeddingGemma 2 có khoảng 740 triệu tham số, trong đó có encoder text khoảng 270 triệu và encoder audio khoảng 300 triệu tham số; đầu ra embedding chung có 768 chiều. Google cũng cung cấp hướng dẫn chạy mô hình trên cả Colab và Kaggle.

![](https://www.google.com/s2/favicons?domain=https://ai.google.dev&sz=32)

Google AI for Developers

+1

Lưu ý: sơ đồ trên mô tả kiến trúc khái niệm. Bài LinkedIn không cung cấp đầy đủ chi tiết về reader, hàm loss và cách căn chỉnh chuỗi, nên không thể khẳng định chính xác tác giả triển khai từng lớp như thế nào chỉ từ nội dung bài đăng.

## 2\. Dữ liệu cần chuẩn bị để huấn luyện là gì?

Điểm quan trọng nhất: bạn không chỉ cần văn bản, cũng không chỉ cần audio. Bạn cần audio và transcript tương ứng của chính audio đó.

Ví dụ, một dataset đơn giản có thể tổ chức như sau:

|

audio_path

|

transcript

|
|

`audio_001.wav`

|

Hôm nay trời đẹp.

|
|

`audio_002.wav`

|

Tôi đang học machine learning.

|
|

`audio_003.wav`

|

Chúng ta bắt đầu cuộc họp lúc chín giờ.

|

Mỗi dòng là một cặp dữ liệu giám sát (supervised pair).

### Các thành phần dữ liệu

1\. Audio

File WAV hoặc định dạng được thư viện hỗ trợ; chuẩn hóa sample rate, số kênh và âm lượng theo yêu cầu của encoder.

Nên giữ metadata về người nói, ngôn ngữ và độ dài.

2\. Transcript chuẩn

Văn bản khớp với audio, có quy tắc nhất quán về dấu câu, chữ hoa, số, từ viết tắt và cách viết từ ngoại ngữ.

Không nên dùng transcript tự động chưa kiểm tra làm nhãn chuẩn cho toàn bộ tập huấn luyện.

3\. Căn chỉnh thời gian --- tùy phương pháp

Nếu reader dự đoán từ tại từng bước thời gian, bạn có thể cần nhãn từ hoặc token gắn với từng khung thời gian. Nếu reader dự đoán chuỗi bằng CTC, cần thiết kế nhãn và hàm loss tương ứng.

Không phải mọi phương pháp đều cần file căn chỉnh thời gian thủ công.

### Có cần hàng triệu giờ audio không?

Không nhất thiết. Bài đăng nói tác giả huấn luyện trong chưa đến hai giờ trên 8 GPU H100 bằng dữ liệu speech có giấy phép mở. Điều đó không có nghĩa dataset chỉ có hai giờ audio: hai giờ là thời gian huấn luyện được báo cáo, không phải thời lượng dữ liệu.

Để bắt đầu thử nghiệm, bạn có thể dùng một tập nhỏ vài giờ audio có transcript chuẩn. Sau đó mở rộng dần lên hàng chục hoặc hàng trăm giờ tùy chất lượng, mục tiêu và ngôn ngữ. Những con số này là mốc thử nghiệm đề xuất, không phải quy mô dataset mà tác giả đã sử dụng.

Một số nguồn dữ liệu có thể cân nhắc:

- [Mozilla Common Voice](https://commonvoice.mozilla.org/) --- dữ liệu giọng nói đa ngôn ngữ.

- [Google FLEURS](https://huggingface.co/datasets/google/fleurs) --- dữ liệu speech đa ngôn ngữ có transcript.

- [VIVOS](https://huggingface.co/datasets/AILAB-VNUHCM/VIVOS) --- dữ liệu tiếng Việt, phù hợp để tìm hiểu bài toán ASR.

Hãy kiểm tra điều khoản sử dụng của từng phiên bản dataset trước khi huấn luyện hoặc phân phối mô hình.

### Một điểm khác biệt quan trọng: nhãn từ hay nhãn câu?

Có ít nhất hai cách tiếp cận khác nhau:

Cách A --- học từ audio sang từ vựng

Encoder tạo đặc trưng audio; reader dự đoán từ tiếp theo hoặc phân phối xác suất trên vocabulary. Nếu triển khai theo hướng phân loại từ ở mỗi bước, bạn cần thiết kế cách lấy nhãn theo thời gian.

Phù hợp với trực giác "tìm từ gần nhất trong vocabulary" được mô tả trong bài đăng.

Cách B --- học audio sang cả chuỗi transcript

Reader dự đoán các token theo chuỗi. Bạn có thể dùng CTC hoặc một kiến trúc giải mã phù hợp khác, nhưng phải thiết kế loss, căn chỉnh và giải mã riêng.

Không nên mặc định tác giả sử dụng CTC; nội dung bài đăng chưa đủ để xác nhận điều đó.

Đối với phương pháp được mô tả, tôi sẽ ưu tiên nghiên cứu hướng A trước, nhưng cần kiểm tra đầu ra thực tế của encoder audio và cơ chế reader trước khi chọn loss.

## 3\. Có giống fine-tune LLM bằng Unsloth không?

Giống nhau ở quy trình học có giám sát, nhưng khác nhau ở mục tiêu tối ưu và cách tạo đầu ra.

|

Tiêu chí

|

LLM fine-tune bằng Unsloth

|

EmbeddingGemma → speech-to-text

|
|

Mô hình gốc

|

LLM sinh token

|

Mô hình embedding đa phương thức

|
|

Dữ liệu

|

Prompt, response hoặc conversation

|

Audio, transcript và có thể thêm nhãn theo thời gian

|
|

Đầu ra

|

Chuỗi token được sinh tự hồi quy

|

Chuỗi từ được dự đoán hoặc giải mã từ đặc trưng audio

|
|

Loss

|

Thường là next-token cross-entropy

|

Phụ thuộc reader: cross-entropy, CTC hoặc loss phù hợp khác

|
|

Phần được huấn luyện

|

LoRA/QLoRA hoặc toàn bộ mô hình

|

Reader trước tiên; encoder có thể đóng băng hoặc fine-tune

|
|

Công cụ

|

Unsloth, Transformers, PEFT

|

PyTorch, Transformers/Sentence Transformers và mã reader tùy chỉnh

|

### Điểm khác biệt lớn nhất: không nhất thiết phải fine-tune encoder

Có ba chiến lược bạn có thể thử:

Chiến lược 1 --- Freeze encoder, train reader

Giữ nguyên toàn bộ trọng số EmbeddingGemma 2. Dùng encoder trích xuất đặc trưng audio, sau đó chỉ huấn luyện reader.

Nên thử đầu tiên

Ưu điểm: ít tốn VRAM hơn, ít rủi ro làm hỏng biểu diễn gốc, dễ thử nghiệm.

Chiến lược 2 --- Train reader rồi fine-tune một phần encoder

Khi reader đã học được cách dự đoán, mở một số lớp hoặc thành phần của encoder để cải thiện khả năng nhận dạng tiếng nói.

Đổi lại, bạn cần nhiều tài nguyên hơn và phải theo dõi độ chính xác trên tập validation.

Chiến lược 3 --- Fine-tune toàn bộ mô hình

Cho phép cập nhật cả encoder audio và những thành phần liên quan đến biểu diễn text.

Cách này linh hoạt hơn nhưng khó tối ưu, cần nhiều bộ nhớ và dễ làm giảm chất lượng embedding vốn có nếu dữ liệu không đủ đa dạng.

### Vì sao không thể mặc định dùng Unsloth như với LLM?

Unsloth tối ưu quá trình fine-tune những kiến trúc mà nó hỗ trợ. Tuy nhiên, bài toán này cần một reader có cách lấy đặc trưng và giải mã riêng. Việc tải thành công EmbeddingGemma 2 vào môi trường Python không đồng nghĩa với việc có thể áp dụng ngay quy trình LoRA dành cho LLM sinh văn bản.

Tôi sẽ dùng PyTorch để triển khai reader trước. Nếu sau này muốn fine-tune encoder, cần kiểm tra kiến trúc thực tế, các module được hỗ trợ bởi PEFT/Unsloth và khả năng truyền gradient qua nhánh audio. Không nên mặc định rằng có thể dùng nguyên notebook SFT của Unsloth.

Ngoài ra, đóng băng encoder không có nghĩa là không cần gradient ở mọi nơi: bạn có thể chạy encoder dưới `torch.no_grad()` để trích xuất đặc trưng, nhưng reader vẫn cần gradient để cập nhật trọng số.

## 4\. Có thể huấn luyện trên Kaggle hoặc Colab không?

Có. Đây là hướng tôi khuyên bạn triển khai trước, nhất là khi mục tiêu là xây dựng prototype để kiểm chứng ý tưởng.

Google có notebook chính thức hướng dẫn sử dụng EmbeddingGemma 2 trên cả hai nền tảng. Tuy nhiên, notebook hiện có tập trung vào tạo embedding đa phương thức và so sánh độ tương đồng; đó chưa phải notebook hoàn chỉnh để huấn luyện reader cho speech-to-text.

![](https://www.google.com/s2/favicons?domain=https://ai.google.dev&sz=32)

Google AI for Developers

+1

![File:Google Colaboratory SVG Logo.svg - Wikimedia Commons](https://images.openai.com/static-rsc-4/Vva6xZfcPdjvmM5Y2whHAYqkrhLfkY1DBFPaZ6osRiXnciHTCr2RgsqjlRzyLSB2jnJMRMKSL-qrxSYunjYJ9ND4ZlhbVzT3pyhhvGmKe8Ga46rP7tRKVqSQOl8YYNBfIOVrTyOYEhNE8oSmOaf6QfcgmOs4LR6pf9TDMke4rhU?purpose=inline)

Google Colab

Phù hợp để phát triển và thử nghiệm reader, kiểm tra pipeline audio, debug loss và đánh giá mô hình. Có thể dùng GPU theo gói bạn có.

Hạn chế: thời lượng phiên, VRAM và loại GPU phụ thuộc cấu hình tài khoản tại thời điểm chạy.

[Mở Google Colab](https://colab.research.google.com/)

![How can I use the Kaggle API for my projects?](https://images.openai.com/static-rsc-4/BoQWLkq3MFN6VMTIIgSpnrocjDgLEWRi0sJSW7p42pFMPgRZDbyYR-eLZzWdw2hJL7UhMhjNNPT8An7lIHBAKq8SgMHQKs3xQKSLWwxmFLTY5WCVuMrjSJPc3Nw4QhYxHOewxn63T0xvNZWs3QP8K51YxvJOY-4cJYh-HIPkq9E?purpose=inline)

Kaggle Notebooks

Phù hợp để chạy thí nghiệm có dataset cố định, lưu checkpoint và so sánh nhiều cấu hình. Có thể dùng GPU được cấp cho notebook.

Hạn chế: loại GPU, thời lượng sử dụng và dung lượng lưu trữ phụ thuộc quota và tài nguyên được cấp.

[Mở Kaggle](https://www.kaggle.com/code)

### Ước lượng tài nguyên cần thiết

Với mô hình 740M tham số, chỉ riêng trọng số ở BF16 chiếm khoảng 1,48 GB theo cách tính thập phân; thực tế còn cần bộ nhớ cho activation, đặc trưng audio, gradient, optimizer và các thành phần khác.

|

Cấu hình

|

Khả năng triển khai

|
|

T4 16 GB

|

Có thể bắt đầu với encoder đóng băng và reader nhỏ, nếu pipeline được tối ưu

|
|

L4 24 GB

|

Thoải mái hơn cho các thí nghiệm reader và batch lớn hơn

|
|

A100 40/80 GB

|

Phù hợp để thử nghiệm fine-tune thêm encoder

|
|

8×H100

|

Cấu hình được bài đăng đề cập; mạnh hơn đáng kể so với yêu cầu để làm prototype

|

Đây là đánh giá sơ bộ, không phải benchmark VRAM đã đo cho chính code của tác giả. Nếu dùng đầu vào dài, batch lớn hoặc lưu activation để backpropagation qua encoder, bộ nhớ cần thiết có thể tăng đáng kể.

Đặc biệt, tài liệu chính thức khuyến nghị dùng `bfloat16` hoặc `float32` với EmbeddingGemma 2, thay vì `float16`, vì activation có thể vượt phạm vi biểu diễn của FP16.

![](https://www.google.com/s2/favicons?domain=https://ai.google.dev&sz=32)

Google AI for Developers

+1

## 5\. Pipeline huấn luyện tôi đề xuất

Tôi sẽ chia dự án thành các giai đoạn sau để tránh tốn GPU trước khi biết phương pháp có hiệu quả hay không.

Bước 1 --- Chuẩn bị dữ liệu

Thu thập các cặp audio--transcript, chuẩn hóa audio về mono 16 kHz nếu phù hợp với pipeline, làm sạch nhãn và chia train/validation/test theo người nói hoặc nguồn ghi âm.

Bước 2 --- Kiểm tra đầu ra audio encoder

Tải `google/embeddinggemma-2`, chạy thử một đoạn audio và kiểm tra chính xác shape của đầu ra. Cần xác định có thể lấy đặc trưng theo từng bước thời gian hay chỉ nhận embedding đã pooling.

Bước 3 --- Xây dựng reader

Nếu có đặc trưng theo thời gian, thử một head tuyến tính hoặc MLP nhỏ. Sau đó xác định cách dự đoán chuỗi, vocabulary, cơ chế giải mã và loss phù hợp với cấu trúc đầu ra.

Bước 4 --- Đánh giá

Đo WER và CER trên tập test chưa từng được dùng khi huấn luyện. Kiểm tra riêng dấu câu, số, tên riêng, từ ngoài vocabulary và những câu có nhiều từ gần âm.

Bước 5 --- Tối ưu tốc độ

Sau khi đạt độ chính xác chấp nhận được, đo tốc độ inference, thử batch/chunk khác nhau và kiểm tra hiệu năng trên thiết bị đích. Chỉ sau đó mới cân nhắc fine-tune encoder.

### Một lưu ý rất quan trọng về reader

Bài đăng nói reader tìm từ gần nhất trong vocabulary của mô hình. Đây là mô tả trực giác, nhưng không nên hiểu đơn giản rằng chỉ cần lấy embedding audio, tính cosine similarity với embedding của tất cả các từ rồi ghép kết quả.

Lý do là:

- Embedding câu đã pooling có thể không giữ đủ thông tin về thứ tự từ.

- Từ gần nhất về mặt ngữ nghĩa chưa chắc là từ đúng về mặt âm học.

- Từ điển từ vựng phải có cơ chế biểu diễn phù hợp; từ và token không nhất thiết tương ứng một-một.

- Một bộ giải mã chuỗi có thể cần xử lý lặp từ, dấu câu, khoảng trắng và lỗi phát âm.

Vì vậy, bước đầu tiên nên là kiểm tra các tensor thực tế mà audio encoder trả về. Nếu chỉ có một vector tổng hợp cho cả đoạn, bạn sẽ cần tìm hiểu cách kiến trúc cho phép lấy biểu diễn chi tiết hơn hoặc chọn một phương pháp khác.

## 6\. Nếu muốn làm riêng cho tiếng Việt thì sao?

Tôi sẽ bắt đầu bằng một thí nghiệm nhỏ, tập trung vào khả năng nhận dạng tiếng Việt thay vì cố tái tạo ngay tốc độ 288× của tác giả.

### Kế hoạch thử nghiệm ban đầu

Dataset

VIVOS hoặc Common Voice tiếng Việt

Quy mô thử nghiệm

Vài giờ audio có transcript chuẩn

GPU

T4/L4, hoặc GPU tốt hơn

Mô hình gốc

EmbeddingGemma 2

Cách học đầu tiên

Freeze encoder, train reader

Metric

WER + CER

Khi đánh giá tiếng Việt, cần quy định cách tính WER nhất quán, đặc biệt với khoảng trắng, dấu câu và từ tiếng Anh xuất hiện trong câu. CER là chỉ số bổ sung hữu ích vì một lỗi dấu có thể làm thay đổi ký tự dù phần còn lại của từ đúng.

Nếu mục tiêu cuối cùng chỉ là có một hệ thống STT tiếng Việt tốt, hãy benchmark thêm một mô hình ASR chuyên dụng. Việc chuyển một embedding model thành STT là hướng nghiên cứu thú vị, nhưng không mặc nhiên tốt hơn fine-tune Whisper hoặc sử dụng một mô hình ASR đã được huấn luyện sẵn.

## Kết luận

Tôi sẽ chọn Colab hoặc Kaggle + PyTorch + EmbeddingGemma 2 đóng băng + reader nhỏ cho phiên bản đầu tiên.

Đừng bắt đầu bằng cách fine-tune toàn bộ 740M tham số hoặc cố dùng nguyên notebook SFT của Unsloth. Hãy xác minh đầu ra encoder, xác định cách reader dự đoán chuỗi và đo độ chính xác trước. Khi phương pháp hoạt động, bạn mới có cơ sở quyết định liệu có cần fine-tune encoder, tăng dữ liệu hoặc tối ưu inference hay không.
