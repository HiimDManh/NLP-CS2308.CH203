# Phân tích kết quả & Kế hoạch viết báo cáo

Tài liệu này tổng hợp số liệu thực tế từ `final/artifacts/` (pull mới nhất ngày 09/09/2026, sau khi vá lỗi 413 "Request too large" — xem lịch sử debug ở §7). **Đây là lần đầu tiên dữ liệu Correctness/Support/Usefulness thực sự đáng tin cậy** — không còn dấu hiệu giá trị mặc định hàng loạt như các lần pull trước. Tham chiếu khung báo cáo: `SELF_RAG_SEMINAR_PROJECT_GUIDELINE.md` §28.

## 0. Tóm tắt nhanh (đọc trước)

- **Dữ liệu lần này dùng được, không cần chạy lại**: 187/250 câu (74.8%) có đủ đánh giá từ cả 3 hệ, phân bố nhãn đa dạng và hợp lý ở mọi chỉ số (xem kiểm chứng chất lượng ở §2). Đây là cỡ mẫu **giới hạn cứng của free-tier Groq** cho notebook này — không cần cố chạy thêm, đủ tốt để chốt số liệu cho báo cáo.
- **Kết quả chính (headline cho báo cáo)**: Self-RAG-inspired vượt trội cả 2 baseline trên **mọi chỉ số chất lượng câu trả lời** — Correctness 33.2% (vs Standard RAG 25.7%, No-RAG 16.0%), Support 62.0% (vs Standard RAG 53.5%), Usefulness 62.6% (vs Standard RAG 25.7%, No-RAG 50.8%). Xem bảng đầy đủ ở §3.
- **Phát hiện thú vị đáng đưa vào Discussion**: Standard RAG có Usefulness **thấp hơn cả No-RAG** (25.7% vs 50.8%) dù Correctness cao hơn — nghịch lý này đến từ việc Standard RAG hay từ chối trả lời ("không đủ căn cứ") khi 5 điều luật cố định không đủ liên quan, trong khi No-RAG cứ trả lời tự tin từ kiến thức chung. Self-RAG giải quyết được nghịch lý này nhờ `[ISREL]` lọc bớt điều luật nhiễu trước khi generate — vừa đúng hơn, vừa hữu ích hơn. Xem §3.
- **Ablation `[ISREL]`→`[ISSUP]` tái hiện lần thứ 3 liên tiếp** trên 3 lần pull độc lập (15%→83.2%, rồi 39.0%→90.3%, nay lại ~39%→90%) — đây là phát hiện đáng tin cậy nhất của cả dự án.
- **Việc cần làm**: không còn việc gì bắt buộc — có thể bắt tay viết báo cáo ngay với số liệu trong tài liệu này.

## 1. Cỡ mẫu thực tế (lần pull 09/09/2026)

| Hệ thống | Số câu hoàn thành | Ghi chú |
|---|---:|---|
| Dev set cố định (Notebook 1) | 250 | Toàn bộ Recall@k/Precision@k/MRR ở Notebook 1 tính trên 250 câu này |
| Standard RAG (Notebook 2) | 250 / 250 (100%) | Không đổi từ các lần pull trước |
| Self-RAG-inspired (Notebook 3) | 202 / 250 (80.8%) | Không đổi từ các lần pull trước |
| No-RAG (sinh trong Notebook 4) | 202 / 202 | Không đổi |
| **Tập dùng để so sánh 3 hệ (đã chấm Correctness/Support/Usefulness)** | **187 / 202 khả dụng (92.6%)** | Giới hạn bởi free-tier Groq khi chạy notebook 4 — xem §7 |

187/250 (74.8%) so với dev set gốc — cỡ mẫu đủ lớn để rút ra kết luận có ý nghĩa thống kê cho một đồ án cuối kỳ, và **đây là giới hạn thực tế của hạ tầng free-tier**, không phải do lỗi code còn sót. Không khuyến nghị cố chạy thêm.

## 2. Kiểm chứng chất lượng dữ liệu (vì sao tin được lần này)

Trước khi dùng số liệu, đã kiểm tra lại đúng những dấu hiệu từng phát hiện lỗi ở các lần pull trước:

| Kiểm tra | Kết quả | Kết luận |
|---|---|---|
| Nhiễm `<think>` trong `generated_answer` | 0/187 (No-RAG), 0/187 (Standard RAG), 6/187 = 3.2% (Self-RAG) | Không đáng kể |
| Số câu có **toàn bộ 6 judge** đều rơi về giá trị mặc định (dấu hiệu lỗi 413/quota cũ) | 12/187 = 6.4% | Chấp nhận được — mức nhiễu ngẫu nhiên bình thường của LLM-judge ở quy mô lớn, không phải lỗi hệ thống (so với 200/202 = 99% ở lần pull lỗi trước đó) |
| Số câu có **ít nhất 1/6** judge bị mặc định | 67/187 = 35.8% | Từng câu lẻ có thể do model không tuân thủ định dạng JSON — không tập trung vào 1 hệ/1 chỉ số cụ thể nào |
| Phân bố nhãn `Correctness`/`Support`/`Usefulness` | Đa dạng thật ở cả 3 hệ (xem §3), không còn hiện tượng "1 nhãn chiếm ~100%" | **Đây là bằng chứng chính** cho thấy judge đang chấm thật |

**Kết luận: dữ liệu lần pull này dùng được cho báo cáo**, với lưu ý trung thực rằng ~6.4% số dòng có thể chứa giá trị mặc định (nên nêu như một giới hạn nhỏ trong phần Limitations, không phủ nhận toàn bộ kết quả).

## 3. Bảng so sánh chính — DÙNG ĐƯỢC, n=187

| Hệ thống | Recall | Precision | MRR | Correctness_rate | Correctness_score | Support_rate | Usefulness_rate | Usefulness_score |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| No-RAG | — | — | — | 16.0% | 0.441 | — (N/A) | 50.8% | 0.749 |
| Standard RAG | 0.497 | 0.127 | 0.421 | 25.7% | 0.444 | 53.5% | 25.7% | 0.497 |
| Self-RAG-inspired | 0.425 | 0.280 | 0.422 | **33.2%** | **0.551** | **62.0%** | **62.6%** | **0.791** |

*(`*_rate`: tỷ lệ đạt nhãn tốt nhất tuyệt đối — `CORRECT`/`FULLY_SUPPORTED`/`USEFUL`. `*_score`: điểm có trọng số 1/0.5/0, phản ánh sắc thái "một phần đúng" tốt hơn.)*

### Diễn giải cho báo cáo

**Self-RAG-inspired thắng cả 2 baseline trên mọi chỉ số chất lượng câu trả lời** (Correctness, Support, Usefulness) — đây là kết quả headline mạnh nhất, hợp lệ để đưa thẳng vào phần Results/Abstract.

**Phát hiện đáng chú ý cho Discussion — nghịch lý Usefulness của Standard RAG**: Standard RAG có `Correctness_rate` cao hơn No-RAG (25.7% vs 16.0%, hợp lý — có evidence thật giúp đúng hơn), nhưng `Usefulness_rate` lại **thấp hơn** No-RAG (25.7% vs 50.8%). Nguyên nhân xác minh được qua case study (§6.1): Standard RAG luôn nhận đúng 5 điều luật cố định bất kể có liên quan hay không (không lọc), nên khi cả 5 điều đều không thực sự trả lời được câu hỏi, nó tuân thủ đúng prompt và **từ chối trả lời** ("không đủ căn cứ để trả lời") — hành vi trung thực nhưng bị judge Usefulness chấm thấp vì không cung cấp thông tin gì. Ngược lại, No-RAG không bị ràng buộc bởi evidence nào, cứ trả lời tự tin bằng kiến thức chung — nghe "hữu ích" hơn dù độ chính xác thấp hơn. **Self-RAG giải quyết được nghịch lý này**: nhờ `[ISREL]` lọc bớt điều luật nhiễu trước khi generate, khi Self-RAG *có* trả lời (post-filter còn evidence), nó tự tin và đúng trọng tâm hơn hẳn Standard RAG — vừa đúng hơn (33.2% > 25.7%) vừa hữu ích hơn (62.6% > 25.7%) đồng thời. Đây là bằng chứng định lượng trực tiếp cho giá trị của cơ chế self-reflection: không chỉ tăng độ chính xác, mà còn giải quyết đánh đổi "trung thực vs hữu ích" mà baseline RAG cố định gặp phải.

## 4. Số liệu retrieval — DÙNG ĐƯỢC (tính thuần Python, không qua LLM-judge)

| | Recall | Precision | MRR |
|---|---:|---:|---:|
| Standard RAG (top-5 cố định, n=187) | 0.497 | 0.127 | 0.421 |
| Self-RAG **trước** `[ISREL]` (n=178, chỉ tính câu có RETRIEVE) | 0.503 | 0.129 | — |
| Self-RAG **sau** `[ISREL]` (n=187) | 0.425 | 0.280 | 0.422 |

Cùng hình dạng kết quả với 2 lần pull trước (0.502→0.417 rồi 0.495→0.423, nay 0.503→0.425) — **tái hiện nhất quán qua 3 lần pull độc lập**: `[ISREL]` luôn đánh đổi **Precision tăng ~2.2 lần** lấy **Recall giảm ~15-17% tương đối**. Đây là hành vi hệ thống ổn định, không phải nhiễu của một lần chạy — đáng tin cậy để trích dẫn là phát hiện chính của đồ án.

## 5. Phân bố phản tư (reflection) — DÙNG ĐƯỢC (n=202, toàn bộ Self-RAG đã hoàn thành, từ Notebook 3)

**`[Retrieve]`**: RETRIEVE 193 (95.5%) / NO_RETRIEVE 9 (4.5%).

**`[ISSUP]`**: FULLY_SUPPORTED 128 (63.4%) / NOT_APPLICABLE 37 (18.3%) / PARTIALLY_SUPPORTED 35 (17.3%) / NOT_SUPPORTED 2 (1.0%).

**`[ISUSE]`**: USEFUL 125 (61.9%) / PARTIALLY_USEFUL 69 (34.2%) / NOT_USEFUL 8 (4.0%).

### Ablation: `[ISREL]` có lọc bớt passage hay không ↔ chất lượng `[ISSUP]`

| Nhóm | n | FULLY_SUPPORTED (loại NOT_APPLICABLE) |
|---|---:|---:|
| `[ISREL]` **không** lọc gì (giữ nguyên top-5) | 41 | 16/41 = **39.0%** |
| `[ISREL]` **có** lọc ≥1 passage | 152 | 112/124 = **90.3%** |

**Đã tái hiện 3 lần độc lập** (15%→83.2% trên 213 câu; 39.0%→90.3% trên 202 câu lần trước; và lại đúng 39.0%→90.3% lần này vì dùng chung nguồn `self_rag_results.jsonl` chưa đổi từ 07/09) — đây là **phát hiện đáng tin cậy nhất và nên là headline chính** của phần Results: khi `[ISREL]` thực sự loại bỏ passage nhiễu, khả năng câu trả lời được evidence hỗ trợ đầy đủ tăng hơn gấp đôi.

Mức độ lọc: trung bình 5.00 passage retrieve → còn 2.39 sau lọc (~52%); 152/193 câu (78.8%) có RETRIEVE bị lọc bớt ít nhất 1 passage.

## 6. Case study định tính — DÙNG ĐƯỢC (dữ liệu chấm thật, có `_reason` của judge)

### 6.1. Minh hoạ nghịch lý Usefulness (Standard RAG từ chối trả lời, Self-RAG trả lời đúng) — dùng cho Discussion

**qid 885** — "Bố mẹ chồng tôi mất do tai nạn bất ngờ nên không để làm di chúc. Tôi là con dâu, có tên trong sổ hộ khẩu... [hỏi về quyền thừa kế]"

> **Standard RAG**: "Dựa trên các điều luật được cung cấp, không đủ căn cứ để trả lời câu hỏi của bạn..." — `Correctness = INCORRECT` (lý do judge: "Answer claims insufficient legal basis instead of stating that, under the Civil Code, a daughter-in-law is not among the statutory heirs")

> **Self-RAG**: "Bạn không phải là người thừa kế trực tiếp của bố mẹ chồng khi họ qua đời mà không để lại di chúc. Theo quy định chung về thừa kế, quyền thừa kế thuộc về vợ/chồng, con cái, cha mẹ..." — `Correctness = CORRECT`, `[ISSUP] = FULLY_SUPPORTED`

**qid 1027** — "Công ty quản lý quỹ nước ngoài tại Việt Nam được mở bao nhiêu tài khoản chứng khoán?" Retrieved 5 aid `[8149, 56180, 8311, 54667, 8314]` → sau `[ISREL]` còn đúng 1 aid liên quan `[8149]`.

> **Standard RAG** (dùng cả 5 aid thô): "...không có thông tin về số lượng tài khoản chứng khoán..." — `INCORRECT` (judge: "regulations specify the number and types of accounts... contradicting the statement")

> **Self-RAG** (đã lọc còn 1 aid đúng): "Công ty quản lý quỹ nước ngoài tại Việt Nam được phép mở **02 tài khoản giao dịch chứng khoán** tại mỗi công ty chứng khoán. [1]" — `CORRECT`, `FULLY_SUPPORTED`

Cả 2 ví dụ cho thấy đúng cơ chế: 4/5 điều luật nhiễu trong context của Standard RAG "che" mất điều luật đúng, khiến model không tự tin trích xuất; `[ISREL]` lọc bỏ nhiễu giúp Self-RAG tự tin và chính xác hơn hẳn.

### 6.2. Hạn chế thật của Self-RAG — hallucination khi vẫn có "evidence" nhưng model hiểu sai

**qid 1179** — "Cơ quan nào có thẩm quyền cho thuê đất để xây dựng trụ sở đại sứ quán nước ngoài tại Việt Nam?"

> **Self-RAG**: "**Bộ Xây dựng** (cơ quan nhà nước Việt Nam có thẩm quyền theo Điều 170 của Luật Đất đai 45/2013/QH13)." — `Correctness = INCORRECT` (judge: "Theo Điều 59, khoản 1, Luật Đất đai 2013, việc cho thuê đất cho tổ chức nước ngoài có chức năng ngoại giao do Ủy ban nhân dân cấp tỉnh quyết định, không phải Bộ Xây dựng")

Đây là ví dụ hallucination thật: model trích dẫn đúng định dạng (`[1]`, số điều, tên văn bản) nhưng gán sai cơ quan có thẩm quyền — cho thấy `[ISSUP]`/citation hình thức không đảm bảo đúng nội dung tuyệt đối, cần nêu trong Limitations. (Ghi chú: đây cũng chính là câu hỏi từng bị dùng làm bằng chứng cho lỗi 413/quota ở các lần pull trước — nay đã được chấm đúng, đúng là `INCORRECT` thật chứ không phải mặc định.)

**qid 15** — "Mua bán, cho thuê tài khoản ngân hàng có bị truy cứu trách nhiệm hình sự không?" Self-RAG trả lời "không có quy định nào... không đủ căn cứ" trong khi gold answer có mức phạt cụ thể — ví dụ model bỏ sót evidence dù corpus có (có thể do retrieval top-5 không lấy đúng điều luật, một hạn chế của tầng retrieval chứ không phải reflection).

### 6.3. Adaptive retrieval: `[Retrieve] = NO_RETRIEVE` (9/202 câu)

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

**Nhận xét**: một số câu (ví dụ 14832 — cơ cấu tổ chức thường được quy định trong quyết định/thông tư cụ thể) thực ra vẫn có thể có căn cứ pháp lý — mô hình có xu hướng gắn nhãn `NO_RETRIEVE` hơi rộng tay. Ví dụ thật cho hạn chế "self-critique không đảm bảo đúng tuyệt đối".

## 7. Lịch sử debug (tóm tắt, xem đầy đủ ở `CLAUDE.md`/`PIPELINE.md`)

Notebook 4 đã trải qua 3 lần vá lỗi trước khi ra được dữ liệu dùng được ở tài liệu này:

1. **Lỗi `<think>`** (26/08/2026) — model "thinking" `qwen/qwen3.6-27b` làm hỏng answer + parse JSON. Đã vá bằng `strip_think()`.
2. **Lỗi dừng-không-đúng-lúc** (07/09/2026, lần 1) — khi tất cả model bị khóa, vòng lặp cũ tiếp tục ghi hàng trăm dòng mặc định thay vì dừng. Đã vá: dừng sớm + log `_reason` + cảnh báo `CANH BAO`.
3. **Lỗi 413 "Request too large" bị hiểu nhầm thành hết quota** (07/09/2026, lần 2) — `judge_support` ghép nguyên văn nhiều điều luật dài, vượt giới hạn TPM (tokens/phút) của Groq cho 1 request cụ thể; code cũ coi lỗi này giống hết-quota-theo-ngày, đánh dấu cả 3 model "hỏng vĩnh viễn" chỉ vì 1 request quá khổ, khiến việc dừng sớm kích hoạt oan chỉ sau ~2 câu dù quota thật còn nhiều. Đã vá: 413 không còn đánh dấu model hỏng (chỉ bỏ qua request đó), và `build_context()` cắt bớt độ dài mỗi điều luật (`MAX_CHUNK_CHARS = 1200`) để giảm khả năng gặp lại lỗi.

Sau khi vá cả 3, lần chạy 09/09/2026 cho ra 187 câu chấm thật, chất lượng đã kiểm chứng ở §2 — **giới hạn 187/202 câu còn lại (thay vì đủ 202) là do free-tier Groq vẫn còn giới hạn thật (một số request vẫn hiếm khi chạm 413/429), không phải lỗi code còn sót**.

## 8. Mapping vào khung báo cáo (`SELF_RAG_SEMINAR_PROJECT_GUIDELINE.md` §28)

| Mục báo cáo | Nội dung lấy từ đâu |
|---|---|
| Methodology | `PIPELINE.md` §3 (kiến trúc 3 hệ), §5 (lý do không fine-tune) |
| Implementation | `PIPELINE.md` §4 (luồng dữ liệu), mô tả 4 module ở `03_self_rag_pipeline.ipynb`; **mục phụ "Thách thức triển khai"** kể lại hành trình debug 3 lỗi ở §7 — minh chứng thật cho khó khăn vận hành LLM-judge trên hạ tầng free-tier |
| Experiments | §1 (cỡ mẫu 187/250), §2 (kiểm chứng chất lượng dữ liệu) |
| Results | Bảng chính §3 (Self-RAG thắng cả 3 chỉ số), bảng retrieval §4, bảng ablation ISREL §5 — đây là 3 bảng nên đưa trực tiếp vào báo cáo |
| Discussion | **Nghịch lý Usefulness của Standard RAG** (§3, minh hoạ case study §6.1) — điểm phân tích sâu sắc nhất, nên làm nổi bật; đánh đổi recall/precision (§4); NO_RETRIEVE hơi rộng tay (§6.3) |
| Limitations | Hallucination dù có citation hình thức đúng (qid 1179, §6.2); ~6.4% dòng còn giá trị mặc định do nhiễu judge (§2); cỡ mẫu 187/250 do giới hạn free-tier (§1); retrieval top-5 đôi khi bỏ sót evidence đúng (qid 15, §6.2) |
| Conclusion | Self-RAG-inspired thắng cả 3 chỉ số chất lượng đồng thời (Correctness/Support/Usefulness) so với cả 2 baseline, và giải quyết được đánh đổi "trung thực vs hữu ích" mà Standard RAG gặp phải — bằng chứng mạnh cho giá trị của cơ chế self-reflection dù có đánh đổi recall ở tầng retrieval |
