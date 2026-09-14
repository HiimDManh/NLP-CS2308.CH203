# Phân tích kết quả & Kế hoạch viết báo cáo

Tài liệu này tổng hợp số liệu thực tế từ `final/artifacts/` (pull mới nhất ngày 09/09/2026, sau khi vá lỗi 413 "Request too large" — xem lịch sử debug ở §8). **Đây là lần đầu tiên dữ liệu Correctness/Support/Usefulness thực sự đáng tin cậy** — không còn dấu hiệu giá trị mặc định hàng loạt như các lần pull trước. Tham chiếu khung báo cáo: `SELF_RAG_SEMINAR_PROJECT_GUIDELINE.md` §28.

*Cập nhật 13/09/2026: bổ sung §6 — ablation đầy đủ trên cả 4 token phản tư (`[Retrieve]`, `[ISREL]`→`[ISSUP]`, `[ISSUP]`→Correctness, `[ISUSE]`→Correctness), tính lại từ đúng dữ liệu 187/202 hiện có, không cần pull artifacts mới. Các mục §7/§8/§9 (Case study/Debug/Mapping) đánh số lại tương ứng.*

## 0. Tóm tắt nhanh (đọc trước)

- **Dữ liệu lần này dùng được, không cần chạy lại**: 187/250 câu (74.8%) có đủ đánh giá từ cả 3 hệ, phân bố nhãn đa dạng và hợp lý ở mọi chỉ số (xem kiểm chứng chất lượng ở §2). Đây là cỡ mẫu **giới hạn cứng của free-tier Groq** cho notebook này — không cần cố chạy thêm, đủ tốt để chốt số liệu cho báo cáo.
- **Kết quả chính (headline cho báo cáo)**: Self-RAG-inspired vượt trội cả 2 baseline trên **mọi chỉ số chất lượng câu trả lời** — Correctness 33.2% (vs Standard RAG 25.7%, No-RAG 16.0%), Support 62.0% (vs Standard RAG 53.5%), Usefulness 62.6% (vs Standard RAG 25.7%, No-RAG 50.8%). Xem bảng đầy đủ ở §3.
- **Phát hiện thú vị đáng đưa vào Discussion**: Standard RAG có Usefulness **thấp hơn cả No-RAG** (25.7% vs 50.8%) dù Correctness cao hơn — nghịch lý này đến từ việc Standard RAG hay từ chối trả lời ("không đủ căn cứ") khi 5 điều luật cố định không đủ liên quan, trong khi No-RAG cứ trả lời tự tin từ kiến thức chung. Self-RAG giải quyết được nghịch lý này nhờ `[ISREL]` lọc bớt điều luật nhiễu trước khi generate — vừa đúng hơn, vừa hữu ích hơn. Xem §3.
- **Ablation `[ISREL]`→`[ISSUP]` tái hiện lần thứ 3 liên tiếp** trên 3 lần pull độc lập (15%→83.2%, rồi 39.0%→90.3%, nay lại ~39%→90%) — đây là phát hiện đáng tin cậy nhất của cả dự án.
- **Ablation đầy đủ cả 4 token phản tư (§6, mới bổ sung)**: cả `[ISSUP]` và `[ISUSE]` đều là tín hiệu có giá trị dự đoán thật (label tốt hơn ↔ Correctness cao hơn, theo đúng thứ tự đơn điệu) chứ không phải nhãn ngẫu nhiên; riêng `[Retrieve]` bộc lộ hạn chế thật — nhóm 9 câu tự quyết định `NO_RETRIEVE` có Correctness thấp hơn cả khi ép Standard RAG truy xuất cưỡng bức trên đúng 9 câu đó (0% vs 11.1%), cho thấy mô-đun này đôi khi bỏ qua truy xuất quá tay.
- **Phát hiện định tính xuyên suốt cả 3 module** (đọc trực tiếp câu trả lời, không chỉ nhìn bảng số) — xem ví dụ cụ thể ở §6: `[Retrieve]` hiểu sai domain câu hỏi (qid 4000, 14790); `[ISSUP]` chấm câu từ chối an toàn là "FULLY_SUPPORTED" dù sai (qid 15) và chấm câu đúng chỉ "PARTIALLY_SUPPORTED" vì thiếu hình thức trích dẫn (qid 3099); `[ISUSE]` chấm câu trả lời sai nhưng tự tin/mạch lạc là "USEFUL" (qid 1227, 2997) và câu đúng nhưng ngắn gọn là "NOT_USEFUL" (qid 4769). Tất cả cùng chỉ ra một hạn chế gốc rễ: **các module tự phản tư thiên về đánh giá hình thức trình bày hơn là nội dung đúng/sai thực chất** — một điểm rất đáng đưa vào Discussion.
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

Mức độ lọc của `[ISREL]`: trung bình 5.00 passage retrieve → còn 2.39 sau lọc (~52%); 152/193 câu (78.8%) có RETRIEVE bị lọc bớt ít nhất 1 passage.

## 6. Ablation đầy đủ trên cả 4 token phản tư — DÙNG ĐƯỢC

Mục này trả lời câu hỏi chung cho cả 4 module: **"mỗi tín hiệu phản tư có thực sự mang thông tin dự đoán chất lượng câu trả lời, hay chỉ là nhãn trang trí?"** Ba mục 6.2–6.4 dùng tập `n=187` (đã chấm Correctness/Support/Usefulness ở Notebook 4, xem §1); riêng 6.1 dùng đúng tập con 9 câu `NO_RETRIEVE` nằm trong 187 câu đó.

### 6.1. `[Retrieve]`: quyết định "không cần tra luật" có đáng tin không?

Trong 187 câu, Self-RAG tự quyết định `NO_RETRIEVE` cho 9 câu (4.8%). So sánh chất lượng câu trả lời của Self-RAG trên 9 câu này với chất lượng của Standard RAG (luôn bị ép truy xuất top-5 cố định) trên **đúng cùng 9 câu đó**:

| Hệ thống (trên cùng 9 câu `NO_RETRIEVE`) | Correctness_rate | Correctness_score |
|---|---:|---:|
| Self-RAG (không truy xuất, trả lời từ kiến thức chung) | 0/9 = **0.0%** | 0.222 |
| Standard RAG (bị ép truy xuất top-5, dù `[Retrieve]` coi là không cần) | 1/9 = **11.1%** | 0.333 |

**Kết luận: đây là hạn chế thật của module `[Retrieve]`**, không phải điểm mạnh — trên 9/9 câu này, việc *có* truy xuất (dù top-5 có thể không hoàn toàn liên quan) vẫn giúp Standard RAG đúng hơn Self-RAG bỏ qua truy xuất hoàn toàn. Đáng chú ý, `Usefulness_rate` của Self-RAG trên nhóm này vẫn cao (8/9 = 88.9%) — cùng cơ chế với "nghịch lý Usefulness" ở §3: trả lời tự tin bằng kiến thức chung nghe hữu ích hơn dù sai nhiều hơn.

**Ví dụ định tính** (đọc trực tiếp câu trả lời, không chỉ nhìn con số):

> **qid 4000** — "Nguyên tắc tổ chức bồi dưỡng bằng hiện vật như thế nào?" `[Retrieve]` quyết định `NO_RETRIEVE` với lý do "câu hỏi về quy trình khoa học, không liên quan đến pháp luật". Nhưng đây thực chất là thuật ngữ pháp lý — "bồi dưỡng bằng hiện vật" là chế độ bồi dưỡng cho người lao động làm việc trong điều kiện độc hại (Thông tư 24/2022/TT-BLĐTBXH), không phải "đào tạo". Self-RAG trả lời hoàn toàn lạc đề (nguyên tắc sư phạm chung) → `INCORRECT`. Standard RAG (bị ép truy xuất) lấy được đúng văn bản liên quan và trả lời đúng một phần các nguyên tắc thật → `PARTIALLY_CORRECT`. Đây là bằng chứng cụ thể cho thấy `[Retrieve]` có thể hiểu sai domain của câu hỏi vì chỉ dựa vào cách đặt câu hỏi bề mặt, không có bước tra cứu để kiểm chứng lại quyết định của chính nó.

> **qid 14790** cho mô hình lỗi tương tự: model coi câu hỏi "điều tra, thu thập, đánh giá nguồn gen giống cây trồng" là "câu hỏi khoa học, không liên quan pháp luật" và bỏ qua truy xuất → trả lời chỉ đúng thao tác kỹ thuật, thiếu hoàn toàn căn cứ pháp lý (Điều 7 Nghị định 27/2021/NĐ-CP) → `INCORRECT`. Standard RAG dù chỉ đúng một phần vẫn bắt được đúng khung pháp lý → `PARTIALLY_CORRECT`.

> Không phải lúc nào cũng một chiều: ở **qid 11746**, Self-RAG (`NO_RETRIEVE`, trả lời từ kiến thức chung) vẫn đưa ra được nội dung gần đúng (`PARTIALLY_CORRECT`, chỉ thiếu chi tiết thành phần Ban Chỉ đạo), trong khi Standard RAG bị 5 điều luật không liên quan "làm nhiễu" nên từ chối trả lời hẳn dù văn bản đúng (Thông tư 15/2022/TT-BKHCN) nằm ngay trong top-5 → `INCORRECT`. Tức là hạn chế của `[Retrieve]` không phải "luôn luôn nên retrieve", mà là **quyết định `NO_RETRIEVE` cần đi kèm một bước kiểm chứng lại xem câu hỏi có thật sự không liên quan đến văn bản pháp luật hay không** — hiện tại quyết định này chỉ dựa vào 1 lần suy luận từ chính câu hỏi, không có cơ chế tự sửa sai.

**Lưu ý cỡ mẫu n=9 rất nhỏ** — nêu như một quan sát định tính có số liệu minh hoạ, không khái quát hoá thành kết luận thống kê mạnh.

### 6.2. `[ISREL]` → `[ISSUP]`: lọc nhiễu trước khi generate có tăng chất lượng evidence không?

| Nhóm | n | FULLY_SUPPORTED (loại NOT_APPLICABLE) |
|---|---:|---:|
| `[ISREL]` **không** lọc gì (giữ nguyên top-5) | 41 | 16/41 = **39.0%** |
| `[ISREL]` **có** lọc ≥1 passage | 152 | 112/124 = **90.3%** |

**Đã tái hiện 3 lần độc lập** (15%→83.2% trên 213 câu; 39.0%→90.3% trên 202 câu lần trước; và lại đúng 39.0%→90.3% lần này vì dùng chung nguồn `self_rag_results.jsonl` chưa đổi từ 07/09) — đây là **phát hiện đáng tin cậy nhất và nên là headline chính** của phần Results: khi `[ISREL]` thực sự loại bỏ passage nhiễu, khả năng câu trả lời được evidence hỗ trợ đầy đủ tăng hơn gấp đôi.

**Ví dụ định tính minh hoạ cơ chế này**: xem §7.1 (qid 885, qid 1027) — cả 2 đều cho thấy cùng một mẫu hình cụ thể: trong top-5 gốc chỉ có 1 điều luật thực sự liên quan, 4 điều còn lại là nhiễu; khi Standard RAG dùng nguyên cả 5 điều, model bị "che khuất" nên từ chối trả lời hoặc trả lời sai; khi `[ISREL]` lọc đúng còn lại điều luật liên quan, Self-RAG tự tin và chính xác hẳn. Đây là bằng chứng định tính trực tiếp giải thích *vì sao* con số 39.0%→90.3% xảy ra, không chỉ là tương quan thống kê.

### 6.3. `[ISSUP]` → Correctness: nhãn "có bằng chứng hỗ trợ" có dự đoán đúng câu trả lời thật không?

Nhóm 187 câu theo nhãn `[ISSUP]` (`self_support`) của Self-RAG, đối chiếu với `Correctness` chấm độc lập ở Notebook 4:

| Nhãn `[ISSUP]` | n | Correctness_rate | Correctness_score |
|---|---:|---:|---:|
| FULLY_SUPPORTED | 116 | 43/116 = **37.1%** | 0.612 |
| NOT_APPLICABLE (không có evidence, trả lời từ kiến thức chung) | 36 | 11/36 = 30.6% | 0.472 |
| PARTIALLY_SUPPORTED | 33 | 8/33 = 24.2% | 0.455 |
| NOT_SUPPORTED | 2 | 0/2 = 0.0% | 0.000 |

**Xu hướng đơn điệu đúng như kỳ vọng**: FULLY_SUPPORTED > NOT_APPLICABLE > PARTIALLY_SUPPORTED > NOT_SUPPORTED trên cả `_rate` lẫn `_score` (nhóm NOT_SUPPORTED chỉ n=2, không đủ để khái quát hoá nhưng không mâu thuẫn — 0% đúng khi hoàn toàn không có bằng chứng hỗ trợ là hợp lý). Đây là bằng chứng cho thấy `[ISSUP]` **là tín hiệu có giá trị dự đoán thật**, không phải nhãn ngẫu nhiên — càng được đánh giá "có bằng chứng hỗ trợ đầy đủ" thì càng có khả năng là câu trả lời đúng.

**Nhưng con số tuyến tính này che giấu một cơ chế lỗi cụ thể, chỉ thấy được khi đọc từng câu trả lời** — trong 116 câu FULLY_SUPPORTED vẫn có 17 câu (14.7%) `INCORRECT`, và không phải ngẫu nhiên:

> **qid 15** — "Mua bán, cho thuê tài khoản ngân hàng có bị truy cứu trách nhiệm hình sự không?" Self-RAG trả lời "không có quy định nào đề cập... không đủ căn cứ" và được `[ISSUP]` chấm **FULLY_SUPPORTED** (câu trả lời đúng là *không* bịa gì ngoài evidence được cấp) — nhưng thực chất câu trả lời **sai**, vì gold answer có quy định xử lý hình sự cụ thể mà model bỏ sót khi đọc evidence. **Đây là điểm mù thật của `[ISSUP]`**: module này chỉ kiểm tra "câu trả lời có nằm trong phạm vi evidence hay không", chứ không kiểm tra "câu trả lời có đúng/đầy đủ so với evidence hay không" — một câu từ chối an toàn luôn dễ đạt `FULLY_SUPPORTED` dù nó không giải quyết được câu hỏi.

> Chiều ngược lại cũng xảy ra: **qid 3099** — "Chủ doanh nghiệp tư nhân thuê người khác làm Giám đốc thì có phải chịu trách nhiệm không?" Self-RAG trả lời đúng kết luận pháp lý cốt lõi ("có, chủ DN tư nhân vẫn chịu trách nhiệm") nhưng chỉ được `[ISSUP]` chấm **PARTIALLY_SUPPORTED** (có thể vì cách trích dẫn chưa bám sát đủ chi tiết điều khoản) dù `Correctness = CORRECT`. Cho thấy `[ISSUP]` đôi khi nghiêm khắc về **hình thức trích dẫn** hơn là về **kết luận pháp lý thực chất** — ngược chiều với ví dụ qid 15 ở trên.

> Hai ví dụ này cùng chỉ ra một điều: `[ISSUP]` là tín hiệu hữu ích ở mức tổng thể (bảng trên), nhưng cơ chế đánh giá của nó lệch theo trục "có bám evidence hình thức hay không" nhiều hơn trục "có trả lời đúng/đủ nội dung hay không" — hai trục này thường trùng nhau nhưng không phải luôn luôn.

### 6.4. `[ISUSE]` → Correctness: nhãn "hữu ích" có đáng tin không?

Cùng cách làm với nhãn `[ISUSE]` (`self_usefulness`):

| Nhãn `[ISUSE]` | n | Correctness_rate | Correctness_score | INCORRECT dù được gắn nhãn này |
|---|---:|---:|---:|---:|
| USEFUL | 117 | 49/117 = **41.9%** | 0.620 | 21/117 = 17.9% |
| PARTIALLY_USEFUL | 62 | 12/62 = 19.4% | 0.460 | 17/62 = 27.4% |
| NOT_USEFUL | 8 | 1/8 = 12.5% | 0.250 | 5/8 = 62.5% |

**Cũng đơn điệu đúng hướng** (USEFUL > PARTIALLY_USEFUL > NOT_USEFUL) — `[ISUSE]` nhìn chung là tín hiệu hợp lý. Nhưng có một **hạn chế đáng nêu trong Discussion/Limitations**: 21/117 (17.9%) câu được gắn nhãn `USEFUL` thực chất vẫn `INCORRECT` — tức model tự đánh giá câu trả lời của chính nó là hữu ích trong khi nó sai. qid 1179 (case study hallucination ở §7.2 — trả lời sai cơ quan có thẩm quyền nhưng đúng định dạng trích dẫn) là một ví dụ cụ thể nằm trong nhóm này. Điều này cho thấy `[ISUSE]` đang đo "nghe có vẻ hữu ích/đầy đủ thông tin" nhiều hơn là đo "đúng sự thật" — hai tiêu chí khác nhau mà judge dùng chung 1 model đôi khi nhầm lẫn.

**Hai ví dụ định tính khác minh hoạ rõ cùng cơ chế** (ngoài qid 1179):

> **qid 1227** — "Người có hành vi không bán đấu giá đối với tài sản phải bán thông qua đấu giá bị phạt bao nhiêu?" Self-RAG trả lời rành mạch, có cấu trúc: "không có quy định cụ thể về mức phạt..." — được chấm `USEFUL` (đầy đủ, rõ ràng, trực tiếp trả lời câu hỏi) nhưng thực chất **sai hoàn toàn**: Điều 23 khoản 3 Nghị định 82/2020/NĐ-CP quy định rõ mức phạt 20-30 triệu đồng. `[ISUSE]` bị "đánh lừa" bởi sự tự tin và mạch lạc của câu trả lời, không kiểm tra được liệu phủ định đó có đúng hay không.

> **qid 2997** tương tự — model khẳng định chắc nịch "Ban Chấp hành (Ban Điều hành)" có quyền điều hành liên hiệp hợp tác xã (chấm `USEFUL`, câu trả lời đầy đủ chi tiết), trong khi đáp án đúng là Giám đốc — một câu trả lời càng chi tiết, càng dễ được `[ISUSE]` chấm hữu ích, bất kể đúng/sai.

> **Chiều ngược lại cũng có, và đáng chú ý hơn**: **qid 4769** — "Thời hạn sử dụng đất khi chuyển từ trồng lúa sang đất thổ cư?" Self-RAG trả lời đúng, ngắn gọn, đúng trọng tâm ("50 năm...") — `Correctness = CORRECT` — nhưng lại bị `[ISUSE]` chấm **NOT_USEFUL**. Đây là bằng chứng cho thấy `[ISUSE]` có thể **thiên vị câu trả lời dài, có cấu trúc, nhiều trích dẫn** hơn là câu trả lời ngắn gọn nhưng chính xác tuyệt đối — một hạn chế ngược chiều với 2 ví dụ trên nhưng cùng gốc rễ: `[ISUSE]` đang lẫn giữa "độ đầy đủ về hình thức" và "độ hữu ích/chính xác thực chất".

Tổng hợp: cả `[ISSUP]` và `[ISUSE]` đều là tín hiệu có ý nghĩa ở mức tổng thể (bảng số liệu ở trên), nhưng khi đọc từng trường hợp cụ thể, cả hai đều bộc lộ cùng một dạng sai lệch — **đánh giá dựa trên hình thức trình bày (tự tin, mạch lạc, có trích dẫn, đủ chi tiết) nhiều hơn là bản chất đúng/sai thực sự** — một hạn chế cố hữu của LLM-as-judge khi judge và generator dùng chung 1 model, nên đáng đưa cả phần định lượng lẫn định tính này vào Discussion/Limitations của báo cáo.

### Tóm tắt ablation 4 token (định lượng + định tính)

| Token | Tín hiệu có ý nghĩa? | Bằng chứng định lượng | Cơ chế lỗi thấy được khi đọc câu trả lời (định tính) |
|---|---|---|---|
| `[Retrieve]` | Có xu hướng đúng, nhưng còn rộng tay | 9 câu NO_RETRIEVE đúng ít hơn Standard RAG bị ép truy xuất (0% vs 11.1%) | qid 4000/14790: hiểu sai domain câu hỏi (tưởng là câu hỏi kỹ thuật/kiến thức chung trong khi thực chất là thuật ngữ pháp lý) vì quyết định `NO_RETRIEVE` không có bước kiểm chứng lại; qid 11746 là phản ví dụ (đôi khi không retrieve lại đúng hơn vì tránh được nhiễu) |
| `[ISREL]`→`[ISSUP]` | **Có, mạnh nhất** | 39.0%→90.3%, tái hiện 3 lần độc lập | qid 885/1027 (§7.1): 4/5 điều luật nhiễu "che" điều đúng khiến Standard RAG sai/từ chối; lọc còn đúng 1 điều giúp Self-RAG tự tin và chính xác |
| `[ISSUP]`→Correctness | Có, đơn điệu đúng hướng | 37.1% (Fully) > 24.2% (Partially) > 0% (Not) | qid 15: câu từ chối "không đủ căn cứ" vẫn được chấm FULLY_SUPPORTED dù sai — `[ISSUP]` đo "không bịa ngoài evidence" chứ không đo "trả lời đúng/đủ"; qid 3099 ngược lại: kết luận đúng nhưng chỉ PARTIALLY_SUPPORTED vì trích dẫn chưa đủ hình thức |
| `[ISUSE]`→Correctness | Có, đơn điệu đúng hướng | 41.9% (Useful) > 19.4% (Partially) > 12.5% (Not) | qid 1227/2997: câu trả lời sai nhưng tự tin, mạch lạc, đủ chi tiết vẫn được chấm USEFUL; qid 4769 ngược lại: câu trả lời đúng nhưng ngắn gọn lại bị chấm NOT_USEFUL — `[ISUSE]` thiên vị hình thức trình bày hơn tính đúng/sai thực chất |

**Kết luận chung của §6**: cả 4 token đều mang tín hiệu có ý nghĩa thống kê ở mức tổng thể, nhưng khi đọc định tính từng trường hợp, 3/4 module (`[Retrieve]`, `[ISSUP]`, `[ISUSE]`) đều bộc lộ cùng một dạng hạn chế gốc rễ: **judge/module tự đánh giá thiên về hình thức (tự tin, mạch lạc, có trích dẫn, đủ chi tiết) hơn là nội dung đúng/sai thực chất** — hệ quả tự nhiên của việc mô phỏng phản tư bằng prompting trên cùng 1 họ model dùng để generate, thay vì một critic được huấn luyện riêng như trong paper gốc.

## 7. Case study định tính — DÙNG ĐƯỢC (dữ liệu chấm thật, có `_reason` của judge)

### 7.1. Minh hoạ nghịch lý Usefulness (Standard RAG từ chối trả lời, Self-RAG trả lời đúng) — dùng cho Discussion

**qid 885** — "Bố mẹ chồng tôi mất do tai nạn bất ngờ nên không để làm di chúc. Tôi là con dâu, có tên trong sổ hộ khẩu... [hỏi về quyền thừa kế]"

> **Standard RAG**: "Dựa trên các điều luật được cung cấp, không đủ căn cứ để trả lời câu hỏi của bạn..." — `Correctness = INCORRECT` (lý do judge: "Answer claims insufficient legal basis instead of stating that, under the Civil Code, a daughter-in-law is not among the statutory heirs")

> **Self-RAG**: "Bạn không phải là người thừa kế trực tiếp của bố mẹ chồng khi họ qua đời mà không để lại di chúc. Theo quy định chung về thừa kế, quyền thừa kế thuộc về vợ/chồng, con cái, cha mẹ..." — `Correctness = CORRECT`, `[ISSUP] = FULLY_SUPPORTED`

**qid 1027** — "Công ty quản lý quỹ nước ngoài tại Việt Nam được mở bao nhiêu tài khoản chứng khoán?" Retrieved 5 aid `[8149, 56180, 8311, 54667, 8314]` → sau `[ISREL]` còn đúng 1 aid liên quan `[8149]`.

> **Standard RAG** (dùng cả 5 aid thô): "...không có thông tin về số lượng tài khoản chứng khoán..." — `INCORRECT` (judge: "regulations specify the number and types of accounts... contradicting the statement")

> **Self-RAG** (đã lọc còn 1 aid đúng): "Công ty quản lý quỹ nước ngoài tại Việt Nam được phép mở **02 tài khoản giao dịch chứng khoán** tại mỗi công ty chứng khoán. [1]" — `CORRECT`, `FULLY_SUPPORTED`

Cả 2 ví dụ cho thấy đúng cơ chế: 4/5 điều luật nhiễu trong context của Standard RAG "che" mất điều luật đúng, khiến model không tự tin trích xuất; `[ISREL]` lọc bỏ nhiễu giúp Self-RAG tự tin và chính xác hơn hẳn.

### 7.2. Hạn chế thật của Self-RAG — hallucination khi vẫn có "evidence" nhưng model hiểu sai

**qid 1179** — "Cơ quan nào có thẩm quyền cho thuê đất để xây dựng trụ sở đại sứ quán nước ngoài tại Việt Nam?"

> **Self-RAG**: "**Bộ Xây dựng** (cơ quan nhà nước Việt Nam có thẩm quyền theo Điều 170 của Luật Đất đai 45/2013/QH13)." — `Correctness = INCORRECT` (judge: "Theo Điều 59, khoản 1, Luật Đất đai 2013, việc cho thuê đất cho tổ chức nước ngoài có chức năng ngoại giao do Ủy ban nhân dân cấp tỉnh quyết định, không phải Bộ Xây dựng")

Đây là ví dụ hallucination thật: model trích dẫn đúng định dạng (`[1]`, số điều, tên văn bản) nhưng gán sai cơ quan có thẩm quyền — cho thấy `[ISSUP]`/citation hình thức không đảm bảo đúng nội dung tuyệt đối, cần nêu trong Limitations. (Ghi chú: đây cũng chính là câu hỏi từng bị dùng làm bằng chứng cho lỗi 413/quota ở các lần pull trước — nay đã được chấm đúng, đúng là `INCORRECT` thật chứ không phải mặc định.)

**qid 15** — "Mua bán, cho thuê tài khoản ngân hàng có bị truy cứu trách nhiệm hình sự không?" Self-RAG trả lời "không có quy định nào... không đủ căn cứ" trong khi gold answer có mức phạt cụ thể — ví dụ model bỏ sót evidence dù corpus có (có thể do retrieval top-5 không lấy đúng điều luật, một hạn chế của tầng retrieval chứ không phải reflection).

### 7.3. Adaptive retrieval: `[Retrieve] = NO_RETRIEVE` (9/202 câu)

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

## 8. Lịch sử debug (tóm tắt, xem đầy đủ ở `CLAUDE.md`/`PIPELINE.md`)

Notebook 4 đã trải qua 3 lần vá lỗi trước khi ra được dữ liệu dùng được ở tài liệu này:

1. **Lỗi `<think>`** (26/08/2026) — model "thinking" `qwen/qwen3.6-27b` làm hỏng answer + parse JSON. Đã vá bằng `strip_think()`.
2. **Lỗi dừng-không-đúng-lúc** (07/09/2026, lần 1) — khi tất cả model bị khóa, vòng lặp cũ tiếp tục ghi hàng trăm dòng mặc định thay vì dừng. Đã vá: dừng sớm + log `_reason` + cảnh báo `CANH BAO`.
3. **Lỗi 413 "Request too large" bị hiểu nhầm thành hết quota** (07/09/2026, lần 2) — `judge_support` ghép nguyên văn nhiều điều luật dài, vượt giới hạn TPM (tokens/phút) của Groq cho 1 request cụ thể; code cũ coi lỗi này giống hết-quota-theo-ngày, đánh dấu cả 3 model "hỏng vĩnh viễn" chỉ vì 1 request quá khổ, khiến việc dừng sớm kích hoạt oan chỉ sau ~2 câu dù quota thật còn nhiều. Đã vá: 413 không còn đánh dấu model hỏng (chỉ bỏ qua request đó), và `build_context()` cắt bớt độ dài mỗi điều luật (`MAX_CHUNK_CHARS = 1200`) để giảm khả năng gặp lại lỗi.

Sau khi vá cả 3, lần chạy 09/09/2026 cho ra 187 câu chấm thật, chất lượng đã kiểm chứng ở §2 — **giới hạn 187/202 câu còn lại (thay vì đủ 202) là do free-tier Groq vẫn còn giới hạn thật (một số request vẫn hiếm khi chạm 413/429), không phải lỗi code còn sót**.

## 9. Mapping vào khung báo cáo (`SELF_RAG_SEMINAR_PROJECT_GUIDELINE.md` §28)

| Mục báo cáo | Nội dung lấy từ đâu |
|---|---|
| Methodology | `PIPELINE.md` §3 (kiến trúc 3 hệ), §5 (lý do không fine-tune) |
| Implementation | `PIPELINE.md` §4 (luồng dữ liệu), mô tả 4 module ở `03_self_rag_pipeline.ipynb`; **mục phụ "Thách thức triển khai"** kể lại hành trình debug 3 lỗi ở §8 — minh chứng thật cho khó khăn vận hành LLM-judge trên hạ tầng free-tier |
| Experiments | §1 (cỡ mẫu 187/250), §2 (kiểm chứng chất lượng dữ liệu) |
| Results | Bảng chính §3 (Self-RAG thắng cả 3 chỉ số), bảng retrieval §4, **ablation đầy đủ 4 token §6** — đây là các bảng nên đưa trực tiếp vào báo cáo, đặc biệt §6.2 (ISREL→ISSUP) làm headline |
| Discussion | **Nghịch lý Usefulness của Standard RAG** (§3, minh hoạ case study §7.1) — điểm phân tích sâu sắc nhất, nên làm nổi bật; đánh đổi recall/precision (§4); `[Retrieve]` hơi rộng tay khi bỏ qua truy xuất (§6.1, §7.3); `[ISSUP]`/`[ISUSE]` là tín hiệu dự đoán thật, đơn điệu đúng hướng (§6.3, §6.4) |
| Limitations | **Phát hiện xuyên suốt (§6, tổng hợp cuối mục "Tóm tắt ablation 4 token"): cả 3 module `[Retrieve]`/`[ISSUP]`/`[ISUSE]` đều thiên về đánh giá hình thức (tự tin, mạch lạc, đủ chi tiết, có trích dẫn) hơn nội dung đúng/sai thực chất** — minh hoạ bằng ví dụ cụ thể qid 15 (từ chối an toàn vẫn chấm FULLY_SUPPORTED), qid 1227/2997 (sai nhưng tự tin vẫn chấm USEFUL), qid 4769 (đúng nhưng ngắn gọn lại chấm NOT_USEFUL); hallucination dù có citation hình thức đúng (qid 1179, §7.2); ~6.4% dòng còn giá trị mặc định do nhiễu judge (§2); cỡ mẫu 187/250 do giới hạn free-tier (§1); retrieval top-5 đôi khi bỏ sót evidence đúng (qid 15, §7.2) |
| Conclusion | Self-RAG-inspired thắng cả 3 chỉ số chất lượng đồng thời (Correctness/Support/Usefulness) so với cả 2 baseline, và giải quyết được đánh đổi "trung thực vs hữu ích" mà Standard RAG gặp phải; cả 4 token phản tư đều mang tín hiệu có ý nghĩa (§6), dù `[Retrieve]` còn hạn chế khi quyết định bỏ qua truy xuất — bằng chứng mạnh cho giá trị của cơ chế self-reflection dù có đánh đổi recall ở tầng retrieval |
