# Self-RAG for Vietnamese Legal QA — Pipeline dự án

Tài liệu này mô tả **hướng đi tổng thể, kiến trúc pipeline, và cách vận hành trên Google Colab** cho đồ án cuối kỳ. Xem thêm bối cảnh nền tảng ở [`SELF_RAG_SEMINAR_PROJECT_GUIDELINE.md`](../SELF_RAG_SEMINAR_PROJECT_GUIDELINE.md) (khung ý tưởng gốc) — tài liệu này là bản cụ thể hóa cho dataset thật đang có.

## 1. Mục tiêu đồ án

Xây dựng một hệ thống hỏi-đáp pháp luật tiếng Việt theo hướng **Self-RAG-inspired** (mô phỏng reflection tokens bằng prompting có cấu trúc, không fine-tune model — xem lý do ở §5), rồi so sánh định lượng với baseline RAG chuẩn và baseline không-RAG, trên bộ dữ liệu thật **VLSP2025 DRiLL**.

Câu hỏi nghiên cứu (bám theo guideline §22, thu hẹp còn 1 câu chính):

> Trên miền văn bản pháp luật Việt Nam, việc thêm cơ chế self-reflection (quyết định retrieve, đánh giá relevance, đánh giá support, đánh giá usefulness) vào RAG có cải thiện độ chính xác truy xuất và chất lượng câu trả lời so với RAG cố định hay không?

## 2. Dataset (đã verify, xem chi tiết ở [`CLAUDE.md`](../CLAUDE.md))

Shared task **VLSP2025 DRiLL** (https://vlsp.org.vn/vlsp2025/eval/drill), nằm ở `final/dataset/VLQA/`:

| File | Vai trò | Số lượng |
|---|---|---|
| `legal_corpus.json` / `legal_dataset.json` | Corpus tri thức (2 dạng biểu diễn của cùng 2157 văn bản luật) | 59 636 điều luật (`aid` duy nhất toàn cục) |
| `train.json` | Câu hỏi có nhãn thật: `question` + `relevant_laws` (gold `aid`) + `answer` (câu trả lời người viết) | 2190 câu |
| `public_test.json` / `private_test.json` | Cùng schema nhưng nhãn bị ẩn. Vốn chỉ dùng khi nộp leaderboard chính thức, nhưng **hạn nộp VLSP2025 DRiLL (12/08/2025) đã qua** — không còn cách nào chấm 2 file này, kể cả cục bộ lẫn qua leaderboard. Coi như không dùng được, chỉ giữ lại cho đủ bộ dữ liệu | 312 / 627 câu |

Vì `public_test`/`private_test` không dùng để tự đánh giá được, pipeline dùng một **dev set cố định 250 câu** tách từ `train.json` (seed 42) cho mọi so sánh nội bộ — xem §4.

## 3. Kiến trúc pipeline

Ba hệ thống được xây dựng và so sánh trên cùng dev set:

```text
Hệ 1 — No-RAG (baseline dưới)
Question -> LLM -> Answer

Hệ 2 — Standard RAG (baseline giữa)
Question -> Retrieve top-k điều luật -> Prompt LLM (question + context) -> Answer

Hệ 3 — Self-RAG-inspired (hệ chính của đồ án)
Question
  -> [Retrieve?] quyết định có cần truy xuất không
  -> Retrieve top-k điều luật (nếu cần)
  -> [ISREL] đánh giá từng điều luật có liên quan không -> loại điều không liên quan
  -> Generate answer dựa trên các điều còn lại (kèm citation law_id + aid)
  -> [ISSUP] đánh giá answer có được các điều luật đã cite hỗ trợ không
  -> [ISUSE] đánh giá answer có hữu ích/đầy đủ với câu hỏi không
  -> Answer + Reflection report (Retrieve/ISREL/ISSUP/ISUSE + nguồn trích dẫn)
```

Bảng ánh xạ token phản tư của paper → module code (xem thêm [`CLAUDE.md`](../CLAUDE.md) mục "Self-RAG core concepts"):

| Token trong paper | Module trong Hệ 3 | Input | Output |
|---|---|---|---|
| `Retrieve` | retrieval-decision | câu hỏi | Yes / No |
| `ISREL` | relevance judge | (câu hỏi, 1 điều luật) | Relevant / Irrelevant |
| `ISSUP` | support judge | (answer, các điều luật đã cite) | Fully / Partially / No support |
| `ISUSE` | usefulness judge | (câu hỏi, answer) | Useful / Partially / Not useful |

Cả 4 module đều là **một LLM instruction-following gọi qua prompt JSON có cấu trúc**, không phải model riêng — đây là điểm khác paper gốc (paper fine-tune generator để tự sinh các token này).

## 4. Luồng dữ liệu & artifacts

```text
final/
├── dataset/VLQA/          # dữ liệu gốc (đọc, không ghi) — *.json KHÔNG nằm trong git (.gitignore),
│   │                      # vì legal_corpus.json/legal_dataset.json vượt giới hạn 100MB của GitHub.
│   │                      # Chỉ tồn tại local + trên Google Drive, xem cách đưa lên Drive ở §6.1.
│   ├── legal_corpus.json
│   ├── legal_dataset.json
│   ├── train.json
│   ├── public_test.json
│   ├── private_test.json
│   └── readme.md          # duy nhất file được commit trong thư mục này (mô tả schema)
├── notebooks/             # code — nguồn sự thật của pipeline, chạy trên Colab
│   ├── 01_retrieval_baseline.ipynb   [ĐÃ CÓ]
│   ├── 02_generator_baseline.ipynb   [ĐÃ CÓ]
│   ├── 03_self_rag_pipeline.ipynb    [ĐÃ CÓ]
│   ├── 04_evaluation_report.ipynb    [ĐÃ CÓ]
│   (05_drill_submission.ipynb — ĐÃ HỦY, xem §8: hạn nộp VLSP2025 DRiLL 12/08/2025 đã qua)
└── artifacts/              # SINH RA từ notebook, không commit git (.gitignore)
    ├── chunks.faiss                  # FAISS index của ~60k chunk điều luật
    ├── chunks_meta.json              # metadata chunk (aid, law_id, text)
    ├── dev_split_qids.json           # 250 qid cố định dùng chung mọi notebook
    ├── retrieval_baseline_metrics.csv
    ├── standard_rag_results.jsonl    # (Notebook 02) câu trả lời Hệ 2 — checkpoint theo từng qid, resume-safe
    ├── self_rag_results.jsonl        # (Notebook 03) câu trả lời Hệ 3 + reflection report từng qid
    ├── no_rag_results.jsonl          # (Notebook 04) câu trả lời Hệ 1 (No-RAG), sinh trong chính notebook 04
    ├── evaluation_details.jsonl      # (Notebook 04) Correctness/Support/Usefulness hậu kiểm từng câu, cả 3 hệ
    └── final_comparison_table.csv    # (Notebook 04) bảng so sánh 3 hệ, dùng thẳng cho báo cáo
```

Nguyên tắc: **artifacts luôn tái tạo được từ notebook + dataset gốc**, không bao giờ sửa tay. Vì `dev_split_qids.json` được lưu lại từ notebook 01, mọi notebook sau load lại đúng file này để đảm bảo 3 hệ thống được so trên cùng một tập câu hỏi.

## 5. Vì sao không fine-tune (ràng buộc Colab free)

Paper gốc train một critic model + fine-tune generator (7B+) để tự sinh reflection token. Trên Colab free (T4 15GB, session giới hạn giờ, không đảm bảo persistent runtime), việc này không khả thi. Pipeline dùng **Groq API (free tier, không yêu cầu setup billing)** qua package `groq`. `GENERATOR_MODEL` không hardcode — notebook tự dò model chat/instruct thật sự khả dụng trên tài khoản qua `client.models.list()` (tài khoản này không có quyền dùng Llama trên Groq; ưu tiên `qwen/qwen3.6-27b` → `openai/gpt-oss-120b` → `openai/gpt-oss-20b` → `groq/compound-mini` → `groq/compound`, tự động xoay vòng khi một model bị khóa quota dài hạn) — cho toàn bộ generate + 4 module judge, để:

> Ban đầu định dùng Google Gemini, nhưng tài khoản Google của người thực hiện đồ án bị yêu cầu khai báo billing mới cấp API key (chính sách Google có thể đổi theo tài khoản/khu vực) — chuyển sang Groq vì free tier ở đây xác nhận không cần thẻ/billing.

- Không tốn GPU quota của Colab cho suy luận (dành GPU cho bước encode embedding một lần).
- Không phụ thuộc việc giữ session sống lâu để load một model lớn.

GPU trên Colab (nếu có) chỉ dùng cho bước embedding corpus (`01_retrieval_baseline.ipynb`) — bước duy nhất tốn compute đáng kể và chỉ chạy một lần nhờ cơ chế lưu/load lại `chunks.faiss`.

## 6. Hướng dẫn chạy trên Google Colab

### 6.1. Đưa code + dataset lên Colab

**Quan trọng: không `git clone`/`git pull` vào thư mục Google Drive đã mount trong Colab.** Đã thử và gặp lỗi thật: Google Drive mount trong Colab dùng FUSE, không tương thích với cách git ghi file pack tạm khi giải nén object — lỗi điển hình:

```text
fatal: could not open '/content/drive/MyDrive/.../.git/objects/pack/tmp_pack_XXXXXX' for reading: No such file or directory
fatal: fetch-pack: invalid index-pack output
```

Đây là giới hạn đã biết của Drive FUSE, không phải do dataset hay do file `.gitignore` — kể cả clone một repo nhỏ (như lúc chỉ có 20MB code) cũng có thể lỗi. Cách né hoàn toàn vấn đề này: **không git-clone trong Colab nữa** — tách riêng code (lấy thẳng từ GitHub, không qua Drive) và dữ liệu (Drive, không qua git):

**Code — mở notebook trực tiếp từ GitHub, không clone:**

Colab đọc thẳng notebook từ GitHub qua URL, không cần git:

```text
https://colab.research.google.com/github/HiimDManh/NLP-CS2308.CH203/blob/main/final/notebooks/01_retrieval_baseline.ipynb
```

Hoặc trong Colab: `File > Open notebook > GitHub tab` → nhập `HiimDManh/NLP-CS2308.CH203` → chọn nhánh `main` → chọn file trong `final/notebooks/`. Mỗi lần mở lại là bản mới nhất trên GitHub, không cần `git pull`. Muốn lưu chỉnh sửa ngược lại GitHub: `File > Save a copy in GitHub` (Colab tự xử lý qua OAuth, không đụng tới git-trên-Drive nên không gặp lỗi trên).

**Dataset — upload thẳng lên một thư mục Drive thường (không phải git repo), một lần duy nhất:**

```python
from google.colab import drive
drive.mount('/content/drive')
```

Trên trình duyệt, vào Google Drive, tạo thư mục `MyDrive/NLP-CS2308.CH203-data/VLQA/`, kéo-thả 5 file json từ máy bạn vào (`legal_corpus.json`, `legal_dataset.json`, `train.json`, `public_test.json`, `private_test.json`). Chỉ cần làm một lần — Drive giữ nguyên giữa các phiên Colab. Đây cũng đúng là lý do dataset bị loại khỏi git (`.gitignore` — 2 file vượt 100MB) nên vốn dĩ không có lựa chọn "kéo theo dataset khi clone code" ngay từ đầu.

### 6.2. Mỗi lần mở notebook

1. `Runtime > Change runtime type > T4 GPU` (cần cho bước embedding ở notebook 01; các notebook sau chủ yếu gọi API nên GPU không bắt buộc, để CPU cũng chạy được).
2. Mở notebook theo cách ở §6.1 (link GitHub trực tiếp, không clone).
3. Chạy cell `pip install`, rồi cell mount Drive — **kiểm tra biến `DRIVE_DATA_ROOT`** trong notebook khớp đúng thư mục bạn tạo ở §6.1 (mặc định `/content/drive/MyDrive/NLP-CS2308.CH203-data`).
4. `Runtime > Run all`.

### 6.3. Thứ tự chạy notebook

Phải chạy theo đúng thứ tự vì mỗi notebook phụ thuộc artifact của notebook trước:

1. `01_retrieval_baseline.ipynb` → sinh `chunks.faiss`, `chunks_meta.json`, `dev_split_qids.json`.
2. `02_generator_baseline.ipynb` → dùng lại index từ bước 1, cần secret `GROQ_API_KEY` (Colab Secrets, không hardcode trong notebook) → sinh `standard_rag_results.jsonl`, checkpoint theo từng câu nên an toàn khi bị ngắt session giữa chừng.
3. `03_self_rag_pipeline.ipynb` → thêm 4 module reflection (Retrieve-decision, ISREL batch, ISSUP, ISUSE) lên trên cùng hạ tầng retrieve/generate → sinh `self_rag_results.jsonl`. Tốn ~5 lần gọi API/câu hỏi (so với 1 lần ở bước 2), nên thử `MAX_QUESTIONS` nhỏ trước khi chạy full dev set.
4. `04_evaluation_report.ipynb` → không phụ thuộc GPU/embedding; sinh thêm Hệ 1 (No-RAG), rồi chấm Correctness/Support/Usefulness thống nhất cho cả 3 hệ trên phần giao nhau các câu đã xong (không giả định đủ 250 câu) → xuất `final_comparison_table.csv`.
~~5. `05_drill_submission.ipynb`~~ — đã hủy, xem §8.

### 6.4. Xử lý sự cố Colab thường gặp

| Sự cố | Cách xử lý |
|---|---|
| `fatal: could not open '.../tmp_pack_XXXXXX' for reading` / `invalid index-pack output` khi `git clone`/`git pull` vào Drive | Đừng làm vậy — Google Drive FUSE không tương thích với ghi pack-object của git. Mở notebook trực tiếp từ GitHub (§6.1), không clone vào Drive. Nếu cần git thật (ví dụ để dev code), làm trên máy local hoặc ổ đĩa local `/content/` của Colab (ổ tạm, mất khi hết session), không phải trên `/content/drive/...` |
| Session bị ngắt giữa chừng lúc encode embedding | Chạy lại notebook 01 — index đã lưu một phần sẽ không tự resume, nhưng vì mất ít hơn ~30-60 phút cho 60k chunk nên chấp nhận chạy lại từ đầu; **không** để mất do quên mount Drive trước khi encode |
| Hết GPU quota free | Chuyển runtime về CPU — embedding vẫn chạy được, chỉ chậm hơn; các notebook 02-05 không cần GPU |
| Rate-limit API LLM (Groq free tier, `Retry-After` vài giây) | Bình thường (per-minute) — notebook 02 đã đọc header `Retry-After` để chờ đúng thời gian, tự qua |
| Rate-limit API LLM (Groq free tier, `Retry-After` hàng trăm giây, lặp lại ở nhiều câu hỏi liên tiếp) | Đã gặp thật — dấu hiệu quota giờ/ngày của **model đó** đã cạn, không phải per-minute. Notebook 02/03 tự phát hiện (`LONG_WAIT_THRESHOLD`) và chuyển sang model tiếp theo trong `CANDIDATE_MODELS` thay vì ngồi chờ. Nếu **toàn bộ** model trong danh sách đều bị khóa cùng lúc, không còn cách chờ trong phiên hiện tại — hạ `MAX_QUESTIONS`, chạy tiếp vào phiên/ngày khác |
| `403 ... is blocked at the project level` khi gọi `groq/compound`/`groq/compound-mini` | Đã gặp thật — 2 model "agentic" này route ngầm qua `llama-3.3-70b-versatile`, model bị khóa cấp project trên tài khoản. Đã bỏ hẳn 2 model này khỏi `CANDIDATE_MODELS` trong cả notebook 02 và 03 |
| Một lỗi 403/4xx bất kỳ làm cả câu hỏi trả về rỗng dù `CANDIDATE_MODELS` còn model khác chưa thử | Bug thật đã sửa: nhánh lỗi không-retry-được trước đây `return ""` thoát thẳng khỏi hàm thay vì nhảy sang model kế tiếp trong hàng đợi. Đã sửa thành `exhausted_models.add(model)` rồi `break` để tiếp tục vòng lặp model — ảnh hưởng cả `generate_answer()` (notebook 02) và `chat()` (notebook 03) |
| Gemini bắt setup billing mới cấp API key | Đã gặp thật, chuyển hẳn sang Groq (free tier không cần billing) — xem §5 |
| Groq báo `model_not_found` cho model được liệt kê là "production" trong docs | Đã gặp thật — model khả dụng khác nhau theo tài khoản/thời điểm. Notebook 02 tự gọi `client.models.list()` để dò model thật sự dùng được thay vì hardcode tên, không cần sửa gì thêm |
| `DRIVE_DATA_ROOT` sai path → `DATA_DIR exists: False` | Kiểm tra lại đường dẫn Drive thực tế bằng `!ls /content/drive/MyDrive` trước khi sửa biến |
| `DATA_DIR exists: True` nhưng notebook báo thiếu `legal_corpus.json`/`train.json` | Quên upload dataset thủ công ở §6.1 — vào Drive kiểm tra `NLP-CS2308.CH203-data/VLQA/` có đủ 5 file json chưa |

## 7. Kết quả cuối cùng của đồ án sẽ là gì

Khi hoàn thành, đồ án gồm các thành phần sau (ánh xạ vào khung báo cáo guideline §28):

1. **Hệ thống chạy được** (notebook 01-04, tái chạy được từ đầu trên Colab free): nhập một câu hỏi pháp luật tiếng Việt, hệ Self-RAG-inspired trả về câu trả lời + danh sách điều luật trích dẫn (`law_id`, số điều) + reflection report (Retrieve/ISREL/ISSUP/ISUSE).
2. **Bảng so sánh định lượng 3 hệ thống** (`final_comparison_table.csv`, notebook 04) — trên phần giao nhau các câu đã chạy xong cả 3 hệ (không nhất thiết đủ 250 câu, tùy quota Groq free tier còn lại khi chạy):

   | Hệ thống | Recall | Precision | MRR | Correctness_rate/score | Support_rate | Usefulness_rate/score |
   |---|---|---|---|---|---|---|
   | No-RAG | — | — | — | ? | — (N/A, không có evidence) | ? |
   | Standard RAG | ? | ? | ? | ? | ? | ? |
   | Self-RAG-inspired | ? (sau ISREL) | ? (sau ISREL) | ? | ? | ? | ? |

   Notebook 04 cũng in ra so sánh Recall **trước/sau** `[ISREL]` của Self-RAG — bằng chứng định lượng cho việc lọc nhiễu có giữ được evidence đúng hay không.

3. **Phân tích case cụ thể**: ví dụ câu hỏi mà Self-RAG-inspired lọc được điều luật nhiễu (RAG chuẩn không lọc), và ví dụ câu hỏi mà self-critique phát hiện answer không được hỗ trợ (hallucination) — dùng cho phần Discussion của báo cáo.
4. **Báo cáo cuối kỳ** theo khung §28 của guideline (Introduction → Related Work → Methodology → Implementation → Experiments → Results → Discussion → Conclusion), dùng trực tiếp bảng/case ở trên.
5. **Demo** chạy trực tiếp trong Colab (nhập câu hỏi trong 1 cell, hoặc UI Gradio đơn giản nếu có thời gian).
6. ~~Nộp lên leaderboard VLSP2025 DRiLL~~ — đã hủy: hạn nộp hệ thống là 12/08/2025, đã qua từ lâu tính đến thời điểm làm đồ án (xem §8).

## 8. Trạng thái hiện tại

- [x] Dataset đã verify, hiểu rõ schema (`train.json` có nhãn thật, `public_test`/`private_test` nhãn ẩn).
- [x] `01_retrieval_baseline.ipynb` — chunking, embedding, FAISS index, Recall@k/Precision@k/MRR trên dev set (đã chạy).
- [x] `02_generator_baseline.ipynb` — baseline RAG generator qua Groq API (đổi từ Gemini vì Gemini bắt setup billing; model tự dò qua `client.models.list()`, ưu tiên `qwen/qwen3.6-27b`, tự xoay vòng khi bị khóa quota dài hạn), checkpoint JSONL resume-safe → `standard_rag_results.jsonl`.
- [x] `03_self_rag_pipeline.ipynb` — 4 module reflection (Retrieve-decision, ISREL batch, ISSUP, ISUSE), sinh `self_rag_results.jsonl`.
- [x] `04_evaluation_report.ipynb` — sinh Hệ No-RAG, chấm Correctness/Support/Usefulness thống nhất cho cả 3 hệ trên phần giao nhau đã chạy xong, xuất `final_comparison_table.csv`.
- [x] Demo — mục "11. Demo" trong `03_self_rag_pipeline.ipynb`, đặt **ngay sau Orchestrator (§10) và trước vòng lặp dev set nặng (§12)** một cách có chủ đích: muốn demo (ví dụ lúc báo cáo cuối kỳ) chỉ cần chạy notebook từ đầu tới hết §11 (Colab: chuột phải cell demo → "Run before"), không phải đợi qua vòng lặp 250 câu ở §12. Sửa `DEMO_QUESTIONS` rồi chạy lại cell là ra ngay answer + nguồn trích dẫn + reflection report.
- [x] ~~`05_drill_submission.ipynb`~~ — **đã hủy**. Fetch trang chính thức https://vlsp.org.vn/vlsp2025/eval/drill xác nhận hạn nộp hệ thống là **12/08/2025 23:59 UTC** (nộp qua Codabench, chấm bằng Recall/Precision/Macro-F2) — đã qua hơn 1 năm tính đến thời điểm làm đồ án này, không còn đường nộp thật. Quyết định (do người dùng chọn): dừng hẳn, không build notebook dự đoán trên `public_test.json`/`private_test.json` nữa, tập trung thời gian còn lại cho báo cáo/slide.
- [ ] Báo cáo + slide (tái sử dụng nội dung từ `seminar/SELF_RAG_SEMINAR_DETAILED_GUIDE.md` cho phần liên quan tới paper gốc).
