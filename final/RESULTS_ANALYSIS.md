# Phân tích kết quả & Kế hoạch viết báo cáo

Tài liệu này tổng hợp số liệu thực tế từ `final/artifacts/` (pull mới nhất ngày 07/09/2026, sau khi quay lại dùng Groq), phân tách rõ **số liệu dùng được** và **số liệu lỗi cần chạy lại**, kèm ablation analysis và case study cụ thể để đưa thẳng vào báo cáo cuối kỳ. Tham chiếu khung báo cáo: `SELF_RAG_SEMINAR_PROJECT_GUIDELINE.md` §28.

## 0. Tóm tắt nhanh (đọc trước)

- **Lỗi `<think>` (đã ghi nhận từ lần trước) nay gần như hết**: sau khi hạ ưu tiên `qwen/qwen3.6-27b` xuống cuối `CANDIDATE_MODELS`, tỷ lệ nhiễm `<think>` giảm còn 0/250 (Standard RAG), 0/202 (No-RAG), 7/202 = 3.5% (Self-RAG) — không còn là vấn đề đáng kể.
- **Phát hiện 1 lỗi MỚI, nghiêm trọng hơn, chưa có trong lần phân tích trước**: `Correctness` của **cả 3 hệ**, và `Support`/`Usefulness` mới tính riêng cho No-RAG/Standard RAG trong `final_comparison_table.csv` hiện tại **không dùng được** — chỉ 2/202 câu là chấm thật, 200/202 câu (99%) rơi vào giá trị mặc định do hết quota Groq giữa chừng. Chi tiết + đã vá code ở §2.
- **Số liệu dùng được ngay** (không bị ảnh hưởng bởi lỗi trên): toàn bộ chỉ số retrieval (Recall/Precision/MRR, có/không có `[ISREL]`), phân bố `[Retrieve]`/`[ISSUP]`/`[ISUSE]` thật của Self-RAG (từ Notebook 3, ghi thẳng vào `self_rag_results.jsonl`, không đi qua đoạn code bị lỗi của Notebook 4), và ablation `[ISREL]` → `[ISSUP]` vẫn cho thấy hiệu ứng rất mạnh (§4).
- **Việc cần làm trước khi chốt số cho báo cáo**: xoá `evaluation_details.jsonl` + `final_comparison_table.csv`, chạy lại `04_evaluation_report.ipynb` (đã vá: tự dừng sớm khi hết quota + tự cảnh báo nếu có câu bị mặc định) **vào lúc quota Groq còn nhiều, tách riêng khỏi lần chạy Notebook 2/3** — chi tiết ở §6.

## 1. Cỡ mẫu thực tế (lần pull 07/09/2026)

| Hệ thống | Số câu hoàn thành | Ghi chú |
|---|---:|---|
| Dev set cố định (Notebook 1) | 250 | Toàn bộ Recall@k/Precision@k/MRR ở Notebook 1 tính trên 250 câu này |
| Standard RAG (Notebook 2) | 250 / 250 (100%) | Hoàn thành đủ |
| Self-RAG-inspired (Notebook 3) | 202 / 250 (80.8%) | 48 câu chưa chạy (rate-limit/quota) |
| No-RAG (sinh trong Notebook 4) | 202 / 202 | Chỉ sinh trên phần giao Standard ∩ Self-RAG |
| **Tập dùng để so sánh 3 hệ** | **202** | Giao của cả 3 hệ |

202/250 (81%) vẫn là cỡ mẫu chấp nhận được cho một free-tier project — không bắt buộc phải chạy nốt 48 câu còn lại trừ khi muốn con số tròn hơn.

## 2. Lỗi MỚI phát hiện: quota Groq cạn giữa chừng làm hỏng Correctness (cả 3 hệ) + Support/Usefulness (No-RAG/Standard RAG)

### Hiện tượng phát hiện được

`final_comparison_table.csv` (trước khi vá):

| Hệ thống | Correctness_rate | Correctness_score | Support_rate | Usefulness_rate |
|---|---:|---:|---:|---:|
| No-RAG | 0.005 | 0.502 | — | 0.010 |
| Standard RAG | 0.0 | 0.498 | 0.005 | 0.0 |
| Self-RAG-inspired | 0.0 | 0.498 | 0.634 | 0.619 |

`Correctness_score ≈ 0.5` cho **cả 3 hệ** — dấu hiệu y hệt lần trước (giá trị mặc định của `judge_correctness` khi parse JSON thất bại là `PARTIALLY_CORRECT` = 0.5 điểm). Khác lần trước ở chỗ: `Support_rate`/`Usefulness_rate` của **Self-RAG vẫn cho số hợp lý** (0.634/0.619) trong khi No-RAG/Standard RAG gần như bằng 0 — gợi ý lỗi không nằm ở `<think>` (đã sửa) mà ở một nguyên nhân khác, chỉ ảnh hưởng tới các giá trị **tính mới trong Notebook 4**.

### Nguyên nhân gốc (xác minh trực tiếp: soi thứ tự các dòng trong `evaluation_details.jsonl`)

Kiểm tra theo đúng thứ tự ghi (append-only), chỉ **2 dòng đầu tiên** (qid 15, qid 32) có nhãn judge đa dạng thật (`CORRECT`, `INCORRECT`, `USEFUL`, `FULLY_SUPPORTED`...) — **toàn bộ 200 dòng còn lại** (qid thứ 3 trở đi) có **100% cả 6 giá trị judge tính mới** (`norag_correctness`, `norag_usefulness`, `standard_correctness`, `standard_support`, `standard_usefulness`, `self_correctness`) đều là giá trị mặc định.

Giải thích: Notebook 4 tốn **6 lệnh gọi LLM/câu** — nhiều hơn hẳn Notebook 2 (1 lệnh/câu) và Notebook 3 (~5 lệnh/câu). Khi chạy Notebook 4 **sau khi** Notebook 2 (250 câu) và Notebook 3 (202 câu × ~5 lệnh) đã tiêu tốn phần lớn quota Groq free-tier trong cùng phiên/ngày, quota cạn hẳn chỉ sau 1-2 câu đầu của Notebook 4. Từ đó, `chat()` trả về `""` (tất cả model trong `CANDIDATE_MODELS` đều nằm trong `exhausted_models`) cho **mọi** lệnh gọi tiếp theo, khiến mọi `judge_*()` lặng lẽ rơi vào nhánh mặc định — không có lỗi/exception nào để phát hiện, notebook vẫn chạy xong "bình thường" và xuất ra một bảng CSV **trông có vẻ hợp lệ**.

**Vì sao `self_support`/`self_usefulness` (Self-RAG) không bị ảnh hưởng?** Vì 2 cột này **không tính lại trong Notebook 4** — chúng được copy nguyên văn từ `issup_label`/`isuse_label` đã lưu sẵn trong `self_rag_results.jsonl`, vốn được tính từ một phiên chạy Notebook 3 **riêng biệt, trước đó, với quota còn đủ**. Tương tự, toàn bộ số liệu retrieval (Recall/Precision/MRR) không đi qua LLM-judge nào cả (tính thuần Python trên `retrieved_aids`/`relevant_aids`) nên cũng không bị ảnh hưởng.

**Bằng chứng cụ thể (soi tay 2 câu, vì bản ghi cũ không lưu `reason` nên phải đọc trực tiếp câu trả lời)**:
- **qid 11575** ("Địa điểm kinh doanh của hộ kinh doanh được quy định ra sao?"): câu trả lời Standard RAG gần như **trùng khớp hoàn toàn** với gold answer (cùng trích đúng Điều 86 Nghị định 01/2021/NĐ-CP, cùng nội dung) — nhưng bị chấm `PARTIALLY_CORRECT` thay vì `CORRECT`.
- **qid 1179** ("Cơ quan nào có thẩm quyền cho thuê đất để xây dựng trụ sở đại sứ quán nước ngoài?"): câu trả lời Self-RAG nói **"Bộ Xây dựng"** có thẩm quyền, trong khi gold answer nói rõ là **"Ủy ban nhân dân cấp tỉnh"** (căn cứ Điều 59 Luật Đất đai 2013) — hai câu trả lời **mâu thuẫn trực tiếp**, nhưng vẫn bị chấm `PARTIALLY_CORRECT` thay vì `INCORRECT`.

Cả 2 ví dụ đều không thể giải thích bằng "judge chấm đúng nhưng khắt khe" — chúng là bằng chứng trực tiếp cho việc judge **không thực sự chạy**, chỉ trả về giá trị mặc định.

### Đã sửa gì (trong `04_evaluation_report.ipynb`, mục §8, 07/09/2026)

1. **Dừng sớm khi hết quota**: vòng lặp đánh giá giờ kiểm tra `model_queue()` trước mỗi câu — nếu rỗng (mọi model đều bị khóa), dừng ngay và in rõ số câu đã chấm thật trong lần chạy đó, thay vì tiếp tục ghi hàng trăm dòng mặc định vô nghĩa. Resume-safe như cũ — chạy lại sau khi quota reset sẽ tiếp tục đúng chỗ dừng.
2. **Lưu lại `_reason` của từng judge** (`norag_correctness_reason`, `standard_support_reason`, ...) vào `evaluation_details.jsonl` — trước đây không lưu, nên lần này phải soi tay câu trả lời mới phát hiện ra vấn đề.
3. **Tự đếm và cảnh báo**: sau mỗi lần chạy, in dòng `CANH BAO: X/Y cau bi mac dinh toan bo...` nếu phát hiện câu nào có toàn bộ 6 judge đều trả về đúng reason mặc định — giúp phát hiện sự cố này ngay lập tức ở lần chạy tiếp theo, không cần phân tích thủ công như lần này.

### Bài học rút ra cho việc chạy lại (đưa thẳng vào §6)

Nguyên nhân sâu xa không phải lỗi code (đã đúng từ trước) mà là **thứ tự/thời điểm chạy**: dồn cả Notebook 2 + 3 + 4 trong cùng một phiên/ngày trên cùng một tài khoản Groq free-tier khiến notebook tốn quota nhiều nhất (Notebook 4, 6 lệnh/câu) luôn là nạn nhân cuối cùng hết quota. Nên **chạy Notebook 4 riêng, vào lúc quota còn nguyên** (ví dụ đầu ngày mới, chưa chạy gì khác).

## 3. Số liệu retrieval — DÙNG ĐƯỢC (tính thuần Python, không qua LLM-judge, n=202)

| | Recall | Precision | MRR |
|---|---:|---:|---:|
| Standard RAG (top-5 cố định) | 0.490 | 0.124 | 0.411 |
| Self-RAG **trước** `[ISREL]` (top-5 thô, n=193 câu có RETRIEVE) | 0.495 | 0.125 | — |
| Self-RAG **sau** `[ISREL]` (đã lọc) | 0.423 | 0.278 | 0.413 |

**Diễn giải cho báo cáo**: cùng một hình dạng kết quả như lần trước — `[ISREL]` giúp **Precision tăng ~2.2 lần** (0.125 → 0.278) nhưng **Recall giảm ~15% tương đối** (0.495 → 0.423). Đây là một **đánh đổi thật**, không phải "lọc nhiễu miễn phí" — mô hình filter khá mạnh tay, loại bỏ luôn một phần passage đúng cùng với passage nhiễu. Nhất quán với lần phân tích trước (0.502→0.417 recall, 0.126→0.285 precision) — củng cố đây là hành vi hệ thống, không phải nhiễu ngẫu nhiên của một lần chạy.

Thống kê mức độ lọc:
- Trung bình mỗi câu (khi có RETRIEVE): **5.00 passage được retrieve → còn 2.39 passage sau lọc** (lọc bỏ ~52%).
- Trong số 193 câu có `[Retrieve]=RETRIEVE`, `[ISREL]` lọc bớt ít nhất 1 passage ở **152 câu (78.8%)**.

## 4. Phân bố phản tư (reflection) thật của Self-RAG — DÙNG ĐƯỢC (n=202, từ Notebook 3, không qua đoạn code lỗi của Notebook 4)

**`[Retrieve]`**:

| Quyết định | Số câu | Tỷ lệ |
|---|---:|---:|
| RETRIEVE | 193 | 95.5% |
| NO_RETRIEVE | 9 | 4.5% |

**`[ISSUP]`**:

| Nhãn | Số câu | Tỷ lệ |
|---|---:|---:|
| FULLY_SUPPORTED | 128 | 63.4% |
| NOT_APPLICABLE (không có evidence) | 37 | 18.3% |
| PARTIALLY_SUPPORTED | 35 | 17.3% |
| NOT_SUPPORTED | 2 | 1.0% |

**`[ISUSE]`**:

| Nhãn | Số câu | Tỷ lệ |
|---|---:|---:|
| USEFUL | 125 | 61.9% |
| PARTIALLY_USEFUL | 69 | 34.2% |
| NOT_USEFUL | 8 | 4.0% |

### Ablation: `[ISREL]` có lọc bớt passage hay không ↔ chất lượng `[ISSUP]`

Vẫn là phân tích ablation mạnh nhất và hoàn toàn hợp lệ (chỉ dùng `issup_label` thật + đếm số aid, không qua judge của Notebook 4):

| Nhóm | n | FULLY_SUPPORTED | PARTIALLY_SUPPORTED | NOT_APPLICABLE | NOT_SUPPORTED | FULLY_SUPPORTED (loại NOT_APPLICABLE) |
|---|---:|---:|---:|---:|---:|---:|
| `[ISREL]` **không** lọc gì (giữ nguyên top-5) | 41 | 16 (39.0%) | 25 (61.0%) | 0 | 0 | 16/41 = 39.0% |
| `[ISREL]` **có** lọc ≥1 passage | 152 | 112 (73.7%) | 10 (6.6%) | 28 (18.4%) | 2 (1.3%) | 112/124 = **90.3%** |

**Kết luận không đổi so với lần trước, thậm chí nhất quán trên một tập câu hỏi khác (202 câu, không trùng hoàn toàn với 213 câu lần trước)**: khi `[ISREL]` thực sự loại bỏ passage nhiễu, tỷ lệ câu trả lời được hỗ trợ đầy đủ (`FULLY_SUPPORTED`, loại các câu không có evidence để so sánh) tăng từ **39.0% lên 90.3%** — đây vẫn là kết quả headline mạnh nhất cho phần Results, và việc tái hiện được xu hướng này trên 2 lần chạy độc lập (lần trước 15%→83.2% trên 213 câu, lần này 39.0%→90.3% trên 202 câu, phương pháp tính rate hơi khác nhưng chiều hướng và độ lớn hiệu ứng giống hệt) càng làm tăng độ tin cậy của phát hiện này.

## 5. Case study định tính — DÙNG ĐƯỢC

### 5.1. Adaptive retrieval: `[Retrieve] = NO_RETRIEVE` (9/202 câu, n tăng so với 5/213 lần trước)

| qid | Câu hỏi | Lý do model đưa ra |
|---|---|---|
| 4000 | Nguyên tắc tổ chức bồi dưỡng bằng hiện vật như thế nào? | "không đề cập đến quy định pháp luật, quyền lợi hay thủ tục pháp lý" |
| 1373 | Giải pháp tăng cường đổi mới cơ chế phân cấp ngân sách nhà nước... | "question about policy, not legal text" |
| 14790 | Điều tra, thu thập, đánh giá nguồn gen giống cây trồng lâm nghiệp như thế nào? | "câu hỏi về quy trình khoa học, không liên quan đến pháp luật" |
| 14832 | Các đơn vị sự nghiệp công lập nào thuộc Sở Tài nguyên và Môi trường? | "yêu cầu liệt kê đơn vị công lập, không liên quan đến quy định pháp luật cụ thể" |
| 11417 | Định hướng nhập khẩu hàng hóa trong Chiến lược xuất nhập khẩu... | "định hướng chiến lược, không yêu cầu tra cứu văn bản luật cụ thể" |
| 2184 | Người học cử nhân Răng Hàm Mặt ở nước ngoài phải thi đầu vào bao nhiêu bài? | "question about educational admission requirements, not legal" |
| 11746 | Việc tổ chức thực hiện của Ban Chủ nhiệm Chương trình hỗ trợ doanh nghiệp... | "question about program organization, not legal regulation" |
| 14121 | Nội dung nhiệm vụ quy hoạch phân khu xây dựng khu chức năng đặc thù... | "kiến thức chung về quy hoạch, không yêu cầu truy xuất văn bản luật cụ thể" |
| 13009 | Xây dựng kế hoạch, chương trình đối thoại với thanh niên? | "question about general program planning, not legal regulation" |

**Nhận xét không đổi so với lần trước**: nhiều câu trong số này (ví dụ 14832 — cơ cấu tổ chức thường được quy định trong quyết định/thông tư cụ thể của Sở/Bộ) thực ra vẫn có thể có căn cứ pháp lý — mô hình có xu hướng gắn nhãn `NO_RETRIEVE` hơi rộng tay cho câu hỏi *nghe* như "quy trình/tổ chức chung". Ví dụ thật cho hạn chế "self-critique không đảm bảo đúng tuyệt đối".

### 5.2. Minh hoạ cụ thể cho lỗi §2 (dùng cho phần Implementation/Limitations, không phải Results)

Hai ví dụ dưới đây **không nên dùng làm minh hoạ chất lượng hệ thống** (vì Correctness lúc đó bị lỗi), mà dùng để minh hoạ chính lỗi hạ tầng free-tier ở §2:

- **qid 11575**: Standard RAG trả lời gần như trùng khớp gold answer (đúng Điều 86 NĐ 01/2021) nhưng bị chấm sai (`PARTIALLY_CORRECT` thay vì `CORRECT`) do judge bị mặc định.
- **qid 1179**: Self-RAG trả lời "Bộ Xây dựng" mâu thuẫn trực tiếp với gold "UBND cấp tỉnh", cũng bị chấm sai (`PARTIALLY_CORRECT` thay vì `INCORRECT`) cùng lý do.

### 5.3. `[ISREL]` lọc nhiễu (giữ nguyên từ lần phân tích trước, vẫn đúng vì không phụ thuộc Correctness)

**qid 32** — "Số ngày tính lãi trong tháng đối với lãi suất tiền gửi... tại Kho bạc Nhà nước được xác định?" Retrieved 5 aid → sau `[ISREL]` còn đúng 1 aid liên quan. `[ISSUP] = FULLY_SUPPORTED`, câu trả lời súc tích, có trích dẫn rõ ràng — đối lập với Standard RAG (giữ nguyên 5 aid thô, không có tầng lọc/kiểm chứng tường minh).

## 6. Việc cần làm trước khi chốt số liệu cuối cho báo cáo

1. Trên Google Drive (`NLP-CS2308.CH203-data/artifacts/`), **xoá 2 file**: `evaluation_details.jsonl` và `final_comparison_table.csv`. Bắt buộc.
2. **Chạy Notebook 4 vào lúc quota Groq free-tier còn nguyên** — tốt nhất là đầu một ngày mới, **không chạy ngay sau khi vừa chạy lại Notebook 2/3 trong cùng phiên**. Đây là bài học chính rút ra từ lỗi ở §2 (không phải lỗi code, mà là lỗi trình tự/thời điểm chạy).
3. Cân nhắc đặt `MAX_QUESTIONS` ở §8 xuống một số vừa phải (ví dụ 100) thay vì `None` (chạy hết 202 câu) — thà có 100 câu chấm **thật** còn hơn 202 câu mà phần lớn bị mặc định. Có thể chạy nhiều lần liên tiếp (các ngày khác nhau) để cộng dồn tới khi đủ số câu mong muốn, nhờ cơ chế resume-safe.
4. Sau khi chạy xong, **đọc kỹ output cuối cùng của cell §8** — nếu thấy dòng `CANH BAO: X/Y cau bi mac dinh...`, nghĩa là vẫn còn vấn đề quota, cần chạy lại vào lúc khác trước khi tin số liệu.
5. Sau khi có `final_comparison_table.csv` sạch (không có cảnh báo), ghi đè phần "Kết quả" trong báo cáo; **§3 và §4 của tài liệu này (retrieval + ablation ISREL) dùng thẳng được ngay, không cần chờ**.

## 7. Mapping vào khung báo cáo (`SELF_RAG_SEMINAR_PROJECT_GUIDELINE.md` §28)

| Mục báo cáo | Nội dung lấy từ đâu |
|---|---|
| Methodology | `PIPELINE.md` §3 (kiến trúc 3 hệ), §5 (lý do không fine-tune, lịch sử đổi Gemini↔Groq) |
| Implementation | `PIPELINE.md` §4 (luồng dữ liệu), mô tả 4 module ở `03_self_rag_pipeline.ipynb`; **thêm mục phụ "Thách thức triển khai"** kể lại cả 2 lỗi: `<think>` (§2 bản cũ, nay đã vá) và quota cạn giữa chừng làm hỏng judge (§2 tài liệu này) — cả hai đều là nội dung Implementation thật, đáng đưa vào như minh chứng cho khó khăn triển khai trên hạ tầng free-tier |
| Experiments | §1 (cỡ mẫu), bảng dev-set/model ở `CLAUDE.md` |
| Results | Bảng retrieval §3, bảng phân bố reflection §4, bảng ablation ISREL §4, `final_comparison_table.csv` (sau khi chạy lại sạch theo §6) |
| Discussion | Đánh đổi recall/precision (§3), nhận xét NO_RETRIEVE hơi rộng tay (§5.1), ablation ISREL→ISSUP là luận điểm cốt lõi (§4) |
| Limitations | Quota Groq free-tier hạn chế cỡ mẫu (202/250) và từng làm hỏng một lần chạy đánh giá hoàn toàn (§2) — minh hoạ rõ ràng cho ràng buộc hạ tầng free-tier nêu ở `PIPELINE.md` §5; lỗi `<think>` đã vá; self-critique không tuyệt đối chính xác (§5.1) |
| Conclusion | Câu chốt đề xuất: cải thiện ISREL đáng kể mối liên hệ evidence–answer (39.0%→90.3% FULLY_SUPPORTED, tái hiện nhất quán qua 2 lần chạy độc lập) là bằng chứng mạnh nhất cho giá trị của cơ chế self-reflection, dù có đánh đổi ở recall |
