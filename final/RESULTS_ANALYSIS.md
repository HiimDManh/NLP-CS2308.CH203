# Phân tích kết quả & Kế hoạch viết báo cáo

Tài liệu này tổng hợp số liệu thực tế từ `final/artifacts/` (pull mới nhất ngày 07/09/2026, sau khi quay lại dùng Groq), phân tách rõ **số liệu dùng được** và **số liệu lỗi cần chạy lại**, kèm ablation analysis và case study cụ thể để đưa thẳng vào báo cáo cuối kỳ. Tham chiếu khung báo cáo: `SELF_RAG_SEMINAR_PROJECT_GUIDELINE.md` §28.

## 0. Tóm tắt nhanh (đọc trước)

- **Lỗi `<think>` (đã ghi nhận từ lần trước) nay gần như hết**: sau khi hạ ưu tiên `qwen/qwen3.6-27b` xuống cuối `CANDIDATE_MODELS`, tỷ lệ nhiễm `<think>` giảm còn 0/250 (Standard RAG), 0/202 (No-RAG), 7/202 = 3.5% (Self-RAG) — không còn là vấn đề đáng kể.
- **Phát hiện 1 lỗi MỚI, nghiêm trọng hơn**: `Correctness` của **cả 3 hệ**, và `Support`/`Usefulness` mới tính riêng cho No-RAG/Standard RAG **không dùng được** khi quota Groq cạn giữa chừng — chỉ vài câu đầu là chấm thật, phần còn lại rơi vào giá trị mặc định mà không có cảnh báo. Đã vá code (dừng sớm + log `_reason` + tự cảnh báo) — chi tiết §2.
- **Sửa lại chẩn đoán trên (quan trọng)**: chẩn đoán "hết quota theo ngày" ở trên **không chính xác**. Log lỗi thật thu được sau đó cho thấy nguyên nhân là **413 "Request too large"** (vượt giới hạn TPM — tokens per minute — của một request cụ thể, do `judge_support` ghép nguyên văn nhiều điều luật dài vào 1 lần gọi), **không phải hết ngân sách theo ngày**. Code cũ coi 413 giống mọi lỗi 4xx khác nên đánh dấu nhầm cả 3 model là "hỏng vĩnh viễn" chỉ vì 1 request quá khổ, khiến `model_queue()` trống oan và notebook dừng sớm dù quota thật vẫn còn nguyên. Đã sửa tận gốc — xem §2 (mục "Lỗi 413") bên dưới. **Khuyến nghị "chờ quota reset qua ngày hôm sau" ở các lần trả lời trước không còn cần thiết** — có thể chạy lại ngay sau khi vá.
- **Số liệu dùng được ngay** (không bị ảnh hưởng bởi lỗi trên): toàn bộ chỉ số retrieval (Recall/Precision/MRR, có/không có `[ISREL]`), phân bố `[Retrieve]`/`[ISSUP]`/`[ISUSE]` thật của Self-RAG (từ Notebook 3, ghi thẳng vào `self_rag_results.jsonl`, không đi qua đoạn code bị lỗi của Notebook 4), và ablation `[ISREL]` → `[ISSUP]` vẫn cho thấy hiệu ứng rất mạnh (§4) — các số liệu này **không cần chờ** Correctness/Support/Usefulness được sửa xong.
- **Việc cần làm trước khi chốt số cho báo cáo**: chạy lại `04_evaluation_report.ipynb` (đã vá lỗi 413 + cắt bớt độ dài evidence) — không cần chờ quota reset theo ngày như trước đây từng khuyến nghị nhầm. Chi tiết ở §6.

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

### Cập nhật quan trọng: chẩn đoán "hết quota theo ngày" ở trên là SAI — nguyên nhân thật là lỗi 413

Sau khi vá 3 điểm trên và chạy lại, log lỗi thật thu được không phải "hết quota" mà là:

```text
Model openai/gpt-oss-120b loi khong the retry (status 413): Error code: 413 - {'error': {'message':
'Request too large for model `openai/gpt-oss-120b` ... on tokens per minute (TPM): Limit 8000, Requested 12543 ...'}}
```

Cả 3 model trong `CANDIDATE_MODELS` đều báo 413 cho **cùng một request** (~11500-12500 token, vượt xa TPM 7000-8000 của cả 3 model) — đây là lỗi **kích thước của một request cụ thể** (gần như chắc chắn là `judge_support` ghép nguyên văn 5 điều luật dài vào `evidence_block`), **không phải hết ngân sách theo ngày**. TPM (tokens/phút) reset mỗi 60 giây, không phải hàng ngày — nên khuyến nghị "chờ qua ngày hôm sau" ở các câu trả lời trước là **sai**, dựa trên chẩn đoán nhầm.

Lỗi nặng hơn nằm ở cách `chat()` xử lý 413: code cũ coi 413 giống mọi lỗi 4xx khác (`else: exhausted_models.add(model); break`), tức là **đánh dấu cả 3 model "hỏng vĩnh viễn cho cả phiên"** chỉ vì một request quá khổ — trong khi thực tế các model đó vẫn hoàn toàn dùng tốt cho các câu hỏi khác có request nhỏ hơn. Đây là lý do notebook "dừng sớm" (`DUNG SOM`) chỉ sau 2 câu dù quota thật sự vẫn còn nhiều.

**Đã sửa tận gốc (04_evaluation_report.ipynb §4 + §2, tương tự cho Notebook 02/03)**:
4. **`chat()` không còn đánh dấu model vào `exhausted_models` khi gặp lỗi 413** — chỉ chuyển sang model khác **cho câu hỏi hiện tại**; các model đó vẫn khả dụng bình thường cho câu hỏi tiếp theo.
5. **`build_context()` cắt bớt mỗi điều luật về tối đa 1200 ký tự** (`MAX_CHUNK_CHARS`) trước khi ghép vào prompt judge — giảm hẳn khả năng một request vượt TPM ngay từ đầu.

**Khuyến nghị vận hành đúng (thay thế khuyến nghị "chờ qua ngày" trước đó)**: chạy lại ngay sau khi vá, không cần chờ gì cả. Nếu vẫn thấy dòng `DUNG SOM`/`CANH BAO`, đọc kỹ thông báo lỗi gốc được in ra ngay phía trên (status code, message) trước khi kết luận nguyên nhân — 413 (kích thước request) và 429 với `Retry-After` dài (quota thật) là hai vấn đề khác nhau, cần xử lý khác nhau.

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

1. Chạy lại `04_evaluation_report.ipynb` (đã vá lỗi 413 + cắt bớt độ dài evidence) — **không cần chờ quota reset qua ngày khác**, khuyến nghị đó dựa trên chẩn đoán sai (xem mục đính chính ở §2). Không cần xoá `evaluation_details.jsonl`/`final_comparison_table.csv` trước, cơ chế resume-safe sẽ tự tiếp tục từ chỗ dừng.
2. Sau khi chạy xong, **đọc kỹ output cuối cùng của cell §8**:
   - Nếu thấy log lỗi có `status 413` — không nên còn xảy ra nữa sau khi vá, nhưng nếu vẫn thấy, có thể cần giảm `MAX_CHUNK_CHARS` trong `build_context()` xuống thấp hơn 1200.
   - Nếu thấy log lỗi `RateLimitError`/429 với `Retry-After` dài (hàng trăm giây) lặp lại liên tục — đây mới thực sự là dấu hiệu hết quota theo giờ/ngày, lúc đó khuyến nghị "đợi" mới áp dụng.
   - Nếu thấy `CANH BAO: X/Y cau bi mac dinh...` — vẫn còn vấn đề parse JSON, kiểm tra `_reason` trong `evaluation_details.jsonl` để biết chi tiết.
3. Nếu chạy trơn tru (không có 2 loại cảnh báo trên), có thể đặt `MAX_QUESTIONS = None` để chạy hết một lượt tất cả các câu còn thiếu trong `final_qids`.
4. Sau khi có `final_comparison_table.csv` sạch (không có cảnh báo, `So_cau` đủ lớn để có ý nghĩa thống kê — tối thiểu vài chục câu), ghi đè phần "Kết quả" trong báo cáo; **§3 và §4 của tài liệu này (retrieval + ablation ISREL) dùng thẳng được ngay, không cần chờ**.

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
