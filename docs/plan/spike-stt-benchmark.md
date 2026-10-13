# Spike M0 — Benchmark Speech-to-Text

> Thuộc: [mvp-plan.md](./mvp-plan.md) — Milestone M0
> Yêu cầu liên quan: [requirements.md](../project-idea/requirements.md) §3.3, NFR-02/03/04
> Trạng thái: **Thiết kế (chưa code)** — 2026-10-13

---

## 1. Mục tiêu

Chọn **model + runtime STT** cho hai lớp (realtime, chạy lại sau họp) theo ba gói phần cứng **Lite / Standard / Pro**, dựa trên số đo thực tế thay vì cảm tính.

### Câu hỏi cần trả lời

| # | Câu hỏi | Chỉ tiêu liên quan |
|---|---|---|
| Q1 | Model nào chạy realtime trên máy **Gen10/16GB chỉ CPU** mà vẫn ≤ 50% CPU, ≤ 4GB RAM, trễ ≤ 5s? | NFR-02, NFR-03 |
| Q2 | Model nào cho độ chính xác cao nhất khi chạy lại sau họp mà vẫn có RTF ≤ 1 (để chừa thời gian cho diarization, nhắm RTF ≤ 0.7)? | NFR-04 |
| Q3 | Chất lượng tiếng Việt và **nói trộn Việt–Anh** của từng model ra sao? | FR-STT-05 |
| Q4 | Glossary (hotwords/initial prompt) giúp cải thiện tên riêng/thuật ngữ được bao nhiêu? | FR-STT-06 |
| Q5 | Runtime nào gọn nhất khi đóng gói (CTranslate2, ONNX/sherpa-onnx, whisper.cpp)? | AD-5, R2 |
| Q6 | Mỗi model ổn định ra sao trên audio dài (30–60 phút): RAM có tăng dần không, có lặp/hallucination không? | NFR-01 |

---

## 2. Model ứng viên

Danh sách mở. License và khả năng hỗ trợ ngôn ngữ phải được **xác minh** trong lúc làm spike, rồi ghi vào bảng license trong báo cáo.

| Nhóm | Ứng viên | Runtime | Ghi chú |
|---|---|---|---|
| Realtime | Whisper `tiny`, `base`, `small` (int8) | faster-whisper (CTranslate2) | Đa ngôn ngữ; chạy kiểu chunked + VAD |
| Realtime | Zipformer streaming (vi, en, zh, ko — model nào có sẵn) | sherpa-onnx | Streaming thật, rất nhẹ; chỉ có cho một số ngôn ngữ |
| Sau họp | Whisper `medium`, `large-v3-turbo` (int8) | faster-whisper | Baseline đa ngôn ngữ |
| Sau họp | PhoWhisper `small` / `medium` / `large` | chuyển sang CTranslate2 | Fine-tune cho tiếng Việt |
| So sánh runtime | Cùng model Whisper ở dạng GGML | whisper.cpp | So tốc độ/RAM với CTranslate2 |
| Tùy chọn | Model mới xuất hiện trong lúc làm spike | — | Bổ sung nếu có model hứa hẹn cho vi/zh/ko |

Mọi engine cài trong `.venv` cục bộ của spike (`uv`). Model tải vào thư mục cache đặt trong spike, **không** dùng cache mặc định ở ổ C.

---

## 3. Dataset

| Bộ | Nguồn | License | Dùng để |
|---|---|---|---|
| **FLEURS** test | `google/fleurs` trên Hugging Face — `vi_vn`, `en_us`, `cmn_hans_cn`, `ko_kr` | CC-BY 4.0 | WER/CER chuẩn trên 4 ngôn ngữ |
| **FLEURS-long** | Ghép các câu FLEURS cùng ngôn ngữ, chèn khoảng lặng ngẫu nhiên 0.3–2s, tạo file 30 và 60 phút | Như trên | Đo độ ổn định, RAM, RTF trên audio dài (Q6) |
| **Custom** | `data/custom/` do user thả vào: cặp `xxx.wav` + `xxx.txt` (transcript chuẩn), tuỳ chọn `xxx.glossary.txt` | Của user, **gitignored** | Họp thật, nói trộn Việt–Anh, glossary (Q3, Q4) |

- Mặc định lấy ngẫu nhiên **N = 200 câu/ngôn ngữ** (seed cố định) để chạy nhanh. Tham số `--full` để chạy toàn bộ tập test.
- Script tải dataset dùng `huggingface_hub`, cache trong `spikes/stt-benchmark/data/cache/`.
- Không dùng các bộ có license NC (ví dụ VIVOS).

---

## 4. Chỉ số đo

| Chỉ số | Định nghĩa / cách đo |
|---|---|
| **WER** (vi, en) | `jiwer` sau khi chuẩn hoá |
| **CER** (zh, ko) | `jiwer.cer` sau khi chuẩn hoá |
| **RTF** (Real-Time Factor) | thời gian xử lý / thời lượng audio (không tính thời gian load model) |
| **Load time** | thời gian load model từ đĩa |
| **Trễ realtime** | Phát audio theo đúng nhịp thời gian thực (chunk 0.5–1s). Với mỗi câu, trễ = thời điểm câu xuất hiện ở dạng final − thời điểm kết thúc câu trong audio. Báo cáo p50 / p95 |
| **CPU%** | `psutil`, lấy mẫu mỗi 0.5s, quy về % trên tổng số core logic; báo cáo trung bình và p95 |
| **RAM** | RSS đỉnh của tiến trình (gồm cả tiến trình con nếu có) |
| **Disk** | dung lượng model trên đĩa |
| **Hallucination** | số đoạn lặp lại hoặc bịa text trong khoảng lặng (đếm theo heuristic: n-gram lặp ≥ 3 lần, text xuất hiện trong đoạn VAD = im lặng) |

### Chuẩn hoá text trước khi tính WER/CER
1. Unicode **NFC** (quan trọng cho dấu tiếng Việt).
2. Chuyển thành chữ thường.
3. Bỏ dấu câu và ký hiệu; gộp các khoảng trắng liên tiếp.
4. zh: bỏ khoảng trắng, tính CER theo ký tự. ko: tính CER, bỏ khoảng trắng.
5. **Không** chuẩn hoá số thành chữ ở vòng đầu. Ghi chú sai lệch do số trong báo cáo.

### Ma trận cấu hình
- Thread: `2`, `4`, `all`. Mức 2 và 4 mô phỏng việc chừa CPU cho Zoom/Meet và máy yếu.
- Compute type: `int8` (mặc định), thêm `int8_float16` / `float16` khi có GPU.
- Beam size: `1` (realtime), `5` (sau họp).
- Có/không có VAD (Silero VAD tích hợp trong faster-whisper).
- Có/không có glossary (chỉ với bộ custom).

---

## 5. Cấu trúc code (sẽ code ở bước triển khai M0)

```
spikes/stt-benchmark/
├── pyproject.toml            # uv; .venv cục bộ; extras theo engine: [faster-whisper], [sherpa], [whispercpp]
├── README.md                 # cách chạy, cách thả dữ liệu custom
├── configs/
│   └── models.yaml           # danh sách model + tham số + nhóm (realtime/batch)
├── src/stt_bench/
│   ├── datasets.py           # FLEURS loader, FLEURS-long builder, custom loader
│   ├── normalize.py          # chuẩn hoá text theo ngôn ngữ
│   ├── metrics.py            # WER/CER, hallucination heuristic
│   ├── monitor.py            # CPU/RAM sampler (thread nền)
│   ├── streaming.py          # phát audio theo thời gian thực, đo trễ
│   ├── engines/
│   │   ├── base.py           # SttEngine Protocol
│   │   ├── faster_whisper.py
│   │   ├── sherpa.py
│   │   └── whisper_cpp.py
│   ├── report.py             # gom kết quả → CSV + Markdown
│   └── run.py                # CLI: download | batch | stream | long | report
├── tests/                    # pytest: normalize, metrics, datasets (dữ liệu giả nhỏ)
├── data/                     # cache + custom (gitignored)
└── results/                  # raw JSON/CSV (gitignored)
```

### Interface engine

```python
from typing import Iterable, Iterator, Protocol
import numpy as np

class PartialResult:
    text: str
    is_final: bool
    audio_end_ms: int   # vị trí trong audio mà kết quả này phủ tới

class SttEngine(Protocol):
    name: str

    def load(self) -> None: ...

    def transcribe(self, audio: np.ndarray, sr: int, language: str | None,
                   hotwords: list[str] | None = None) -> str: ...

    def stream(self, chunks: Iterable[np.ndarray], sr: int,
               language: str | None) -> Iterator[PartialResult]: ...
```

Engine nào không hỗ trợ streaming thật sẽ được bọc bằng `ChunkedStreamer` dùng chung: VAD cắt câu rồi `transcribe` từng câu. Như vậy phép đo trễ công bằng giữa các engine.

### CLI dự kiến

```powershell
uv run stt-bench download --langs vi,en,zh,ko
uv run stt-bench batch  --models configs/models.yaml --group batch --threads 4
uv run stt-bench stream --model whisper-small-int8 --lang vi --threads 2
uv run stt-bench long   --model large-v3-turbo-int8 --minutes 60
uv run stt-bench report --out ../../docs/reports/stt-benchmark-<date>.md
```

---

## 6. Môi trường chạy

| Máy | Vai trò | Bắt buộc |
|---|---|---|
| Máy dev: i5-13500H / 16GB, chỉ CPU | Chạy sơ bộ, loại bớt ứng viên | Có |
| **Máy Gen10 / 16GB, chỉ CPU** | Số đo chính để chốt model theo NFR | **Có** |
| Apple Silicon | Tham khảo cho Phase 2 | Không |
| NVIDIA GPU | Đề xuất gói Pro/CUDA | Không |

Quy trình đo: cắm sạc, chế độ nguồn *Best performance*, đóng app nặng, chạy mỗi cấu hình 2 lần và lấy lần thứ 2 (để loại hiệu ứng cache/warm-up). Báo cáo ghi rõ CPU, RAM, OS, phiên bản runtime.

---

## 7. Tiêu chí chọn

1. **Lọc cứng** theo NFR trên máy Gen10:
   - Realtime: CPU trung bình ≤ 50% (với thread giới hạn), RAM ≤ 4GB, trễ p95 ≤ 5s.
   - Sau họp: RTF ≤ 0.7.
2. Trong các model qua được vòng lọc, xếp hạng theo **WER tiếng Việt** (FLEURS vi + custom). Hoà nhau thì xét CER zh/ko, WER en.
3. Hoà nữa thì ưu tiên: model nhỏ hơn → license thoáng hơn → runtime đóng gói gọn hơn.
4. **Phân gói:**
   - **Lite**: máy Gen8/9 hoặc RAM eo hẹp, có thể chỉ khuyến nghị chế độ "Chỉ ghi âm".
   - **Standard**: máy Gen10 chỉ CPU.
   - **Pro**: Apple Silicon / NVIDIA.

---

## 8. Đầu ra

- `docs/reports/stt-benchmark-<date>.md` gồm:
  - Cấu hình máy và phiên bản phần mềm.
  - Bảng kết quả (WER/CER, RTF, trễ, CPU, RAM, disk) theo model × cấu hình.
  - Bảng license model + runtime.
  - **Đề xuất model cho từng gói và từng lớp** (realtime / sau họp), kèm cấu hình (thread, beam, VAD).
  - Ghi chú đóng gói: runtime nào, kích thước ước tính.
  - Hiệu quả của glossary (nếu có dữ liệu custom).
- Code spike giữ trong `spikes/stt-benchmark/` để chạy lại khi có model mới.

---

## 9. Ngoài phạm vi

- Diarization (mini-spike ở M4).
- Dịch / LLM (mini-spike ở M5).
- Tích hợp vào app (M3).
