# Phân tích kết quả & Kế hoạch viết báo cáo

Tài liệu này tổng hợp toàn bộ số liệu thực tế từ `final/artifacts/` (pull về ngày 26/08/2026), phân tách rõ **số liệu dùng được** và **số liệu lỗi cần chạy lại**, kèm ablation analysis và case study cụ thể để đưa thẳng vào báo cáo cuối kỳ. Tham chiếu khung báo cáo: `SELF_RAG_SEMINAR_PROJECT_GUIDELINE.md` §28.

## 0. Tóm tắt nhanh (đọc trước)

- **Phát hiện 1 lỗi nghiêm trọng đã sửa**: cột `Correctness` (cả 3 hệ) và `Standard RAG Support/Usefulness`, `No-RAG Usefulness` trong `final_comparison_table.csv` hiện tại **không dùng được** — 100% là giá trị mặc định do lỗi parse, không phải đánh giá thật. Nguyên nhân + cách sửa ở §2.
- **Số liệu dùng được ngay**: toàn bộ chỉ số retrieval (Recall/Precision/MRR, có/không có `[ISREL]`), phân bố `[Retrieve]`/`[ISSUP]`/`[ISUSE]` thật của Self-RAG (tính từ Notebook 3, không bị lỗi), và một phát hiện ablation khá mạnh: **khi `[ISREL]` thực sự lọc bớt passage, tỷ lệ FULLY_SUPPORTED tăng từ 15% lên 83%** (§4).
- **Việc cần làm trước khi chốt số cho báo cáo**: xoá `evaluation_details.jsonl` + `final_comparison_table.csv`, chạy lại §7-§10 của `04_evaluation_report.ipynb` (đã vá lỗi) — không cần chạy lại Notebook 2/3. Chi tiết ở §6.

## 1. Cỡ mẫu thực tế

| Hệ thống | Số câu hoàn thành | Ghi chú |
|---|---:|---|
| Dev set cố định (Notebook 1) | 250 | Toàn bộ Recall@k/Precision@k/MRR ở Notebook 1 tính trên 250 câu này |
| Standard RAG (Notebook 2) | 250 / 250 (100%) | Hoàn thành đủ |
| Self-RAG-inspired (Notebook 3) | 213 / 250 (85.2%) | 37 câu chưa chạy (rate-limit) |
| No-RAG (sinh trong Notebook 4) | 213 / 213 | Chỉ sinh trên phần giao Standard ∩ Self-RAG |
| **Tập dùng để so sánh 3 hệ** | **213** | Giao của cả 3 hệ |

213/250 (85%) là cỡ mẫu khá tốt, không phải vấn đề — không cần cố chạy nốt 37 câu còn lại trừ khi muốn có con số tròn 250.

Đối chiếu để kiểm tra tính đại diện của subset 213 so với full 250 (Notebook 1):

| | Recall@5 | Precision@5 | MRR |
|---|---:|---:|---:|
| Full dev set (250 câu, Notebook 1) | 0.498 | 0.122 | 0.420 |
| Subset 213 câu (Notebook 4, retriever giống hệt) | 0.505 | 0.126 | 0.417 |

Hai bộ số gần như trùng khớp → **subset 213 câu đại diện tốt cho toàn bộ dev set**, không có thiên lệch đáng kể do 37 câu bị thiếu là ngẫu nhiên (do rate-limit, không liên quan nội dung câu hỏi).

## 2. Lỗi đã phát hiện và sửa: model "thinking" (`qwen/qwen3.6-27b`) làm hỏng judge

### Hiện tượng phát hiện được

Trong `final_comparison_table.csv` hiện tại:

| Hệ thống | Correctness_rate | Support_rate | Usefulness_rate |
|---|---:|---:|---:|
| No-RAG | 0.0 | N/A | 0.0 |
| Standard RAG | 0.0 | 0.0 | 0.0 |
| Self-RAG-inspired | 0.0 | 0.704 | 0.563 |

`Correctness_rate = 0.0` cho **cả 3 hệ** vì **100% trong 213 câu đều nhận đúng một nhãn `PARTIALLY_CORRECT`** — không có một ngoại lệ nào. Tương tự, `standard_support` 100% `PARTIALLY_SUPPORTED`, `standard_usefulness` và `norag_usefulness` 100% `PARTIALLY_USEFUL`. Đây chính xác là giá trị **mặc định (fallback)** được code trả về khi không parse được JSON từ model — xác suất một judge thật cho ra đúng 1 nhãn suốt 213 câu hỏi đa dạng là gần như bằng 0, nên đây là **bằng chứng thống kê rõ ràng của lỗi hệ thống**, không phải trùng hợp.

### Nguyên nhân gốc (đã xác minh trực tiếp trong dữ liệu)

Kiểm tra `no_rag_answer`/`standard_answer` thô trong `evaluation_details.jsonl`, phát hiện chúng chứa nguyên khối:

```text
<think>
The user asks: "..."
... (hàng nghìn ký tự suy luận) ...
</think>

<câu trả lời thật>
```

`qwen/qwen3.6-27b` — model đứng đầu `CANDIDATE_MODELS` lúc đó — là model kiểu **"thinking"**: luôn sinh khối suy luận `<think>...</think>` trước nội dung thật, **kể cả khi prompt yêu cầu "chỉ trả về JSON, không giải thích gì thêm"**. Hệ quả:

1. Câu trả lời final (No-RAG/Standard RAG) bị lẫn cả đoạn suy luận dài vào `generated_answer`.
2. `extract_json()` (dùng cho `judge_correctness`/`judge_support`/`judge_usefulness`) không tách được JSON thật ra khỏi khối `<think>` một cách đáng tin cậy — đặc biệt khi model "nhắc lại" ví dụ định dạng JSON của prompt ngay trong lúc suy luận, khiến regex tham lam `\{.*\}` khớp nhầm. Kết quả: hầu hết lệnh gọi rơi vào nhánh mặc định.
3. Trường hợp xấu nhất (gặp thật ở qid 85): chuỗi suy luận quá dài, bị cắt cụt do giới hạn token đầu ra **trước khi** model kịp đóng `</think>` — toàn bộ câu trả lời chỉ còn là suy luận dở dang, không cứu được.

Đo được tỷ lệ nhiễm thực tế trên dữ liệu đã pull:

| File | Tổng | Có `<think>` |
|---|---:|---:|
| `standard_rag_results.jsonl` | 250 | 82 (32.8%) |
| `self_rag_results.jsonl` | 213 | 29 (13.6%) |
| `no_rag_results.jsonl` | 213 | 158 (74.2%) |

Tỷ lệ khác nhau giữa các file vì cơ chế xoay vòng model (`exhausted_models`) khiến các lần chạy khác nhau rơi trúng `qwen` với tần suất khác nhau tuỳ quota còn lại tại thời điểm chạy.

**Vì sao `self_support`/`self_usefulness` (Self-RAG) vẫn đúng?** Vì 2 cột này được Notebook 4 **tái dùng nguyên** giá trị đã tính từ Notebook 3 (`issup_label`/`isuse_label`), không tính lại — và lần chạy Notebook 3 đó tỷ lệ nhiễm `<think>` thấp hơn (13.6%) nên phần lớn vẫn parse đúng. Đây là may mắn ngẫu nhiên, không phải do thiết kế miễn nhiễm.

### Đã sửa những gì (trong `02_generator_baseline.ipynb`, `03_self_rag_pipeline.ipynb`, `04_evaluation_report.ipynb`)

1. Thêm hàm `strip_think()` — cắt bỏ `<think>...</think>` bằng regex trước khi:
   - Parse JSON trong `extract_json()`.
   - Lưu/dùng làm `generated_answer` (áp dụng ngay tại điểm dùng trong Notebook 4, nên **không cần chạy lại Notebook 2/3** dù dữ liệu nguồn của chúng vẫn còn nhiễm — Notebook 4 tự làm sạch khi đọc).
2. Đổi thứ tự `CANDIDATE_MODELS`: đưa `qwen/qwen3.6-27b` xuống **cuối cùng** (trước đó đứng đầu), ưu tiên `openai/gpt-oss-120b` → `openai/gpt-oss-20b` — hai model này không gặp vấn đề "thinking" nên giảm hẳn tần suất gặp lại lỗi lớp này, kể cả với các câu chưa parse lỗi hoàn toàn (JSON kẹp trong `<think>` không đóng vẫn là rủi ro tiềm ẩn không thể cứu 100% bằng `strip_think`).

### Giới hạn còn lại (thành thật, để ghi vào phần Limitations của báo cáo)

`strip_think()` chỉ cứu được khi model **đóng** `</think>` trước khi hết token — nếu bị cắt cụt giữa chừng (trường hợp qid 85), không có cách khôi phục câu trả lời. Hạ ưu tiên `qwen` xuống cuối giảm tần suất chứ không loại bỏ hoàn toàn rủi ro này (vẫn có thể bị chọn khi 2 model kia cạn quota).

## 3. Số liệu retrieval — DÙNG ĐƯỢC (tính thuần Python, không qua LLM-judge)

| | Recall | Precision | MRR |
|---|---:|---:|---:|
| Standard RAG (top-5 cố định) | 0.505 | 0.126 | 0.417 |
| Self-RAG **trước** `[ISREL]` (top-5 thô) | 0.502 | 0.126 | — |
| Self-RAG **sau** `[ISREL]` (đã lọc) | 0.417 | 0.285 | 0.410 |

**Diễn giải cho báo cáo**: `[ISREL]` giúp **Precision tăng +126% tương đối** (0.126 → 0.285) nhưng **Recall giảm 17% tương đối** (0.502 → 0.417). Đây **không phải** kết quả lý tưởng kiểu "giữ nguyên recall, chỉ tăng precision" như kỳ vọng ban đầu trong `PIPELINE.md` — mà là một **đánh đổi thật**: mô hình lọc khá mạnh tay, có xu hướng loại bỏ luôn một số passage đúng cùng với passage nhiễu. Đây là điểm phân tích trung thực và có giá trị cho phần Discussion (đúng tinh thần guideline: "nếu Self-RAG-inspired không luôn tốt hơn baseline, vẫn có thể phân tích lý do — đó là một phần tốt của báo cáo khoa học").

Thống kê mức độ lọc:
- Trung bình mỗi câu: **4.88 passage được retrieve → còn 2.29 passage sau lọc** (lọc bỏ ~53%).
- Trong số 208 câu có `[Retrieve]=RETRIEVE`, `[ISREL]` lọc bớt ít nhất 1 passage ở **173 câu (83.2%)** — module này hoạt động tích cực, không phải no-op.

## 4. Phân bố phản tư (reflection) thật của Self-RAG — DÙNG ĐƯỢC (n=213, từ Notebook 3)

**`[Retrieve]`**:

| Quyết định | Số câu | Tỷ lệ |
|---|---:|---:|
| RETRIEVE | 208 | 97.7% |
| NO_RETRIEVE | 5 | 2.3% |

Phù hợp với phát hiện đã ghi ở `CLAUDE.md`: hầu hết câu hỏi thật trong `train.json` đều cần retrieval (đúng bản chất dữ liệu — câu hỏi tư vấn luật). Module vẫn hoạt động đúng logic (5 câu hỏi mang tính "phương pháp/tổ chức chung" được nhận diện đúng là không cần tra luật — xem case study §5).

**`[ISSUP]`** (Self-RAG, giá trị thật — không bị lỗi §2):

| Nhãn | Số câu | Tỷ lệ |
|---|---:|---:|
| FULLY_SUPPORTED | 150 | 70.4% |
| PARTIALLY_SUPPORTED | 39 | 18.3% |
| NOT_APPLICABLE (không có evidence) | 23 | 10.8% |
| NOT_SUPPORTED | 1 | 0.5% |

**`[ISUSE]`** (Self-RAG, giá trị thật):

| Nhãn | Số câu | Tỷ lệ |
|---|---:|---:|
| USEFUL | 120 | 56.3% |
| PARTIALLY_USEFUL | 81 | 38.0% |
| NOT_USEFUL | 12 | 5.6% |

### Ablation: `[ISREL]` có lọc bớt passage hay không ↔ chất lượng `[ISSUP]`

Đây là phân tích ablation mạnh nhất hiện có — **hoàn toàn hợp lệ** vì chỉ dùng `self_support` (giá trị thật) và số lượng aid (đếm thuần, không qua judge):

| Nhóm | n | FULLY_SUPPORTED | PARTIALLY_SUPPORTED | NOT_APPLICABLE | NOT_SUPPORTED |
|---|---:|---:|---:|---:|---:|
| `[ISREL]` **không** lọc gì (giữ nguyên top-5) | 40 | 6 (15.0%) | 29 (72.5%) | 5 (12.5%) | 0 |
| `[ISREL]` **có** lọc ≥1 passage | 173 | 144 (83.2%) | 10 (5.8%) | 18 (10.4%) | 1 (0.6%) |

**Đây là bằng chứng định lượng trực tiếp và mạnh cho luận điểm cốt lõi của Self-RAG**: khi module `[ISREL]` thực sự loại bỏ passage nhiễu, tỷ lệ câu trả lời được hỗ trợ đầy đủ bởi evidence (`FULLY_SUPPORTED`) tăng từ **15% lên 83.2%** — chênh lệch rất lớn, đáng để làm biểu đồ/callout riêng trong phần Results. Đây là kết quả nên **headline** trong báo cáo, vì nó không phụ thuộc vào phần dữ liệu bị lỗi ở §2.

## 5. Case study định tính — DÙNG ĐƯỢC (đã làm sạch `<think>`)

### 5.1. Adaptive retrieval: `[Retrieve] = NO_RETRIEVE` đúng chỗ (5/213 câu)

| qid | Câu hỏi | Lý do model đưa ra |
|---|---|---|
| 10614 | Nội dung thực hành chuyên môn của người phụ trách công tác dược lâm sàng gồm những gì? | "question about general professional duties, not legal regulation" |
| 13009 | Xây dựng kế hoạch, chương trình đối thoại với thanh niên? | "Câu hỏi về lập kế hoạch chung, không liên quan đến quy định pháp luật" |
| 14023 | Thành phần bản vẽ trong hồ sơ đồ án quy hoạch chuyên ngành hạ tầng kỹ thuật đô thị...? | "technical planning question, not legal regulation" |
| 14790 | Điều tra, thu thập, đánh giá nguồn gen giống cây trồng lâm nghiệp như thế nào? | "question about general methodology, not legal regulation" |
| 15779 | Các đơn vị nào giúp việc Giám đốc Học viện Chính trị quốc gia Hồ Chí Minh? | "question about organizational units, not legal regulation" |

**Nhận xét quan trọng cho phần Discussion/Limitations**: nhìn kỹ 5 câu này, thực ra **đa số vẫn có thể tra cứu được trong corpus luật** (ví dụ cơ cấu tổ chức Học viện Chính trị quốc gia HCM thường được quy định trong quyết định/nghị định cụ thể) — mô hình có xu hướng gắn nhãn NO_RETRIEVE hơi rộng tay cho các câu hỏi *nghe* có vẻ về "quy trình/tổ chức chung" dù thực chất vẫn có căn cứ pháp lý. Đây là ví dụ thật cho hạn chế "self-critique không đảm bảo đúng tuyệt đối" đã nêu trong `SELF_RAG_SEMINAR_DETAILED_GUIDE.md` mục 12.5.

### 5.2. `[ISREL]` lọc nhiễu giúp câu trả lời súc tích và có căn cứ hơn

**qid 32** — "Số ngày tính lãi trong tháng đối với lãi suất tiền gửi... tại Kho bạc Nhà nước được xác định?"
Retrieved 5 aid `[1017, 12802, 12803, 13126, 12801]` → sau `[ISREL]` còn đúng 1 aid liên quan `[1017]`.

> **Self-RAG**: "Số ngày tính lãi trong tháng đối với lãi suất tiền gửi tại Kho bạc Nhà nước được xác định bằng **số ngày thực tế của tháng**... **Tham khảo:** [1] (Văn bản: 18/2020/TT-BTC)." — `[ISSUP] = FULLY_SUPPORTED`

> **Standard RAG** (cùng 5 aid thô, không lọc): "...được xác định là **số ngày thực tế của tháng**... [1]" — nội dung tương tự nhưng do đưa cả 5 văn bản (4 văn bản không liên quan) vào context, không có tín hiệu nào cho thấy model đã tự tin lọc bỏ nhiễu — về hình thức câu trả lời "trông giống nhau" nhưng Self-RAG có thêm tầng kiểm chứng tường minh (biết văn bản nào thực sự được dùng).

**qid 85** — minh hoạ rõ nhất vấn đề `<think>` không đóng: `standard_answer` của câu này **hoàn toàn là một khối suy luận dở dang, không có câu trả lời thật** (do bị cắt token giữa chừng) — trong khi `self_answer` cho cùng câu hỏi lại trả lời đầy đủ, có cấu trúc, trích dẫn `[1]` rõ ràng, `[ISSUP] = FULLY_SUPPORTED`. Đây là ví dụ tốt để minh hoạ hạn chế hạ tầng free-tier trong báo cáo (không phải do phương pháp Self-RAG kém hơn).

## 6. Việc cần làm trước khi chốt số liệu cuối cho báo cáo

1. Trên Google Drive (thư mục `NLP-CS2308.CH203-data/artifacts/`), **xoá 2 file**: `evaluation_details.jsonl` và `final_comparison_table.csv`. Bắt buộc — nếu không xoá, cơ chế resume-safe của `04_evaluation_report.ipynb` sẽ coi 213 câu cũ là "đã xong" và **không chấm lại**, giữ nguyên số liệu lỗi.
2. Mở `04_evaluation_report.ipynb` (đã vá `strip_think()` + đổi thứ tự `CANDIDATE_MODELS`), chạy lại từ đầu. **Không cần chạy lại Notebook 2/3.**
3. Notebook sẽ tự in cỡ mẫu (`n = ...`) — kỳ vọng vẫn quanh 213 câu (trừ khi muốn chạy thêm 37 câu Self-RAG còn thiếu bằng cách chạy lại Notebook 3 trước).
4. Sau khi có `final_comparison_table.csv` mới, đối chiếu lại `Correctness_rate` — nếu vẫn thấy một giá trị lặp lại bất thường ở tỷ lệ cao, báo lại để kiểm tra tiếp (có thể do quota cạn giữa chừng chứ không phải lỗi cũ).
5. Ghi đè phần "Kết quả" trong báo cáo bằng bảng mới; **các phần §3 và §4 của tài liệu này (retrieval + ablation ISREL) không cần chờ chạy lại**, dùng thẳng được ngay.

## 7. Mapping vào khung báo cáo (`SELF_RAG_SEMINAR_PROJECT_GUIDELINE.md` §28)

| Mục báo cáo | Nội dung lấy từ đâu |
|---|---|
| Methodology | `PIPELINE.md` §3 (kiến trúc 3 hệ), §5 (lý do không fine-tune) |
| Implementation | `PIPELINE.md` §4 (luồng dữ liệu), mô tả 4 module ở `03_self_rag_pipeline.ipynb`; **thêm mục phụ "Thách thức triển khai"** kể lại lỗi `<think>` ở §2 tài liệu này — đây là nội dung Implementation thật, đáng đưa vào |
| Experiments | §1 (cỡ mẫu, kiểm tra tính đại diện), bảng dev-set/model ở `CLAUDE.md` |
| Results | Bảng retrieval §3, bảng phân bố reflection §4, bảng ablation ISREL §4, `final_comparison_table.csv` (sau khi chạy lại theo §6) |
| Discussion | Đánh đổi recall/precision (§3), nhận xét NO_RETRIEVE hơi rộng tay (§5.1), case qid 85 minh hoạ giới hạn hạ tầng free-tier (§5.2) |
| Limitations | Lỗi `<think>` + giới hạn của `strip_think()` khi bị cắt cụt (§2), quota Groq free-tier khiến chỉ hoàn thành 213/250 câu (§1), self-critique không tuyệt đối chính xác (§5.1) |
| Conclusion | Câu chốt đề xuất: cải thiện ISREL đáng kể mối liên hệ evidence–answer (15%→83.2% FULLY_SUPPORTED) là bằng chứng mạnh nhất cho giá trị của cơ chế self-reflection, dù có đánh đổi ở recall |
