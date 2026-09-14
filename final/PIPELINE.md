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
│   (05_drill_submission.ipynb — ĐÃ HỦY, xem §9: hạn nộp VLSP2025 DRiLL 12/08/2025 đã qua)
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

Paper gốc train một critic model + fine-tune generator (7B+) để tự sinh reflection token. Trên Colab free (T4 15GB, session giới hạn giờ, không đảm bảo persistent runtime), việc này không khả thi. Pipeline dùng **Groq API (free tier, không yêu cầu setup billing)** qua package `groq`. `GENERATOR_MODEL` không hardcode — notebook tự dò model chat/instruct thật sự khả dụng trên tài khoản qua `client.models.list()` (tài khoản này không có quyền dùng Llama trên Groq; ưu tiên `openai/gpt-oss-120b` → `openai/gpt-oss-20b` → `qwen/qwen3.6-27b`, tự động xoay vòng khi một model bị khóa quota dài hạn — `qwen` cố tình để cuối vì là model "thinking" từng gây lỗi parse JSON hàng loạt, xem `CLAUDE.md` mục Notebook 02 và `RESULTS_ANALYSIS.md`) — cho toàn bộ generate + 4 module judge, để:

> Ban đầu định dùng Google Gemini, nhưng tài khoản Google của người thực hiện đồ án bị yêu cầu khai báo billing mới cấp API key (chính sách Google có thể đổi theo tài khoản/khu vực) — chuyển sang Groq vì free tier ở đây xác nhận không cần thẻ/billing.
>
> **Đã thử quay lại Gemini một lần (2026-08-26) rồi quay về Groq trong chính ngày đó**: sau khi sửa lỗi billing, gặp tiếp 2 vòng lỗi model khác nhau (404 do `gemini-2.0-flash`/`gemini-2.0-flash-lite` bị khai tử; rồi 403 `PERMISSION_DENIED` do project bị hạn chế quyền dù model có trong `client.models.list()` thật của tài khoản) — kể cả sau khi đổi sang alias `-latest` để né việc đoán tên model, Gemini free tier vẫn hết quota token quá nhanh so với nhu cầu chạy 250 câu × nhiều lần gọi API/câu (đặc biệt Notebook 3 tốn ~5 lần gọi/câu). Kết luận: **Groq phù hợp hơn cho quy mô free-tier của đồ án này**, không nên thử lại Gemini trừ khi có thay đổi rõ ràng về giới hạn tài khoản.

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
~~5. `05_drill_submission.ipynb`~~ — đã hủy, xem §9.

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
| `<think>...</think>` lẫn vào `generated_answer`, và các judge (Correctness/ISSUP/ISUSE/ISREL/Retrieve-decision) parse JSON sai âm thầm — rơi về nhãn mặc định | Đã gặp thật (26/08/2026), phát hiện qua thống kê 100% một nhãn trên 213 câu — xem `RESULTS_ANALYSIS.md` (lưu ý: tài liệu này được viết lại mỗi lần có pull mới, số mục §... có thể đổi giữa các lần — đọc lại thay vì tin số mục cũ). Nguyên nhân: `qwen/qwen3.6-27b` là model "thinking", luôn bọc suy luận trước output kể cả khi yêu cầu chỉ trả JSON. Đã thêm `strip_think()` (cắt trước khi parse JSON và trước khi lưu answer) ở cả 3 notebook 02/03/04, và hạ `qwen` xuống cuối `CANDIDATE_MODELS`. Nếu suy luận bị cắt cụt do hết token trước khi đóng `</think>`, không cứu được câu trả lời đó — chấp nhận như một giới hạn còn lại. Đã giảm hẳn (còn ~0-3.5%) sau khi hạ `qwen` xuống cuối danh sách |
| **Correctness ≈ 0.5 điểm cho cả 3 hệ, nhưng KHÔNG phải do `<think>`** (đã gặp thật lần 2, 07/09/2026, sau khi lỗi `<think>` ở trên đã giảm hẳn) | Chứng minh được: chỉ 2/202 dòng đầu trong `evaluation_details.jsonl` có nhãn thật, 200 dòng còn lại default 100% — `judge_*()` âm thầm rơi về nhãn mặc định. Đã sửa notebook 04 §8 (dừng sớm khi `model_queue()` rỗng, lưu `_reason` từng judge, tự in cảnh báo `CANH BAO`). **Chẩn đoán ban đầu (hết quota theo ngày do Notebook 2/3 dùng trước) SAI** — xem dòng bên dưới |
| `413 "Request too large"` bị hiểu nhầm thành hết quota, khiến `model_queue()` trống oan và dừng sớm chỉ sau ~2 câu | Đã gặp thật (07/09/2026): log lỗi thật là `Request too large ... on tokens per minute (TPM): Limit 8000, Requested 12543` cho cả 3 model — một request cụ thể (evidence_block của `judge_support`, ghép nguyên văn nhiều điều luật dài) vượt TPM, **không phải hết ngân sách theo ngày** (TPM reset mỗi phút, không phải mỗi ngày). Lỗi code cũ: `chat()` coi 413 giống mọi lỗi 4xx khác, đưa cả 3 model vào `exhausted_models` vĩnh viễn cho phiên đó dù chúng vẫn dùng tốt cho request nhỏ hơn. Đã sửa ở cả 3 notebook 02/03/04: (1) 413 chỉ chuyển model **cho câu hỏi hiện tại**, không đánh dấu model hỏng; (2) `build_context()` cắt mỗi điều luật về tối đa `MAX_CHUNK_CHARS` (1200 ký tự) để giảm khả năng gặp lại lỗi này. **Không cần chờ quota reset qua ngày khác nữa** — chạy lại ngay sau khi vá |
| `DRIVE_DATA_ROOT` sai path → `DATA_DIR exists: False` | Kiểm tra lại đường dẫn Drive thực tế bằng `!ls /content/drive/MyDrive` trước khi sửa biến |
| `DATA_DIR exists: True` nhưng notebook báo thiếu `legal_corpus.json`/`train.json` | Quên upload dataset thủ công ở §6.1 — vào Drive kiểm tra `NLP-CS2308.CH203-data/VLQA/` có đủ 5 file json chưa |

## 7. Kết quả cuối cùng của đồ án sẽ là gì

Khi hoàn thành, đồ án gồm các thành phần sau (ánh xạ vào khung báo cáo guideline §28):

1. **Hệ thống chạy được** (notebook 01-04, tái chạy được từ đầu trên Colab free): nhập một câu hỏi pháp luật tiếng Việt, hệ Self-RAG-inspired trả về câu trả lời + danh sách điều luật trích dẫn (`law_id`, số điều) + reflection report (Retrieve/ISREL/ISSUP/ISUSE).
2. **Bảng so sánh định lượng 3 hệ thống** (`final_comparison_table.csv`, notebook 04) — trên phần giao nhau các câu đã chạy xong cả 3 hệ (n=187/250, giới hạn free-tier Groq — xem `RESULTS_ANALYSIS.md`, dữ liệu đã kiểm chứng và dùng được cho báo cáo, số liệu chốt ngày 09/09/2026):

   | Hệ thống | Recall | Precision | MRR | Correctness_rate/score | Support_rate | Usefulness_rate/score |
   |---|---|---|---|---|---|---|
   | No-RAG | — | — | — | 16.0% / 0.441 | — (N/A, không có evidence) | 50.8% / 0.749 |
   | Standard RAG | 0.497 | 0.127 | 0.421 | 25.7% / 0.444 | 53.5% | 25.7% / 0.497 |
   | Self-RAG-inspired | 0.425 (sau ISREL) | 0.280 (sau ISREL) | 0.422 | **33.2% / 0.551** | **62.0%** | **62.6% / 0.791** |

   Self-RAG-inspired thắng cả 2 baseline trên mọi chỉ số chất lượng câu trả lời. Điểm đáng chú ý: Standard RAG có Usefulness thấp hơn cả No-RAG (hay từ chối trả lời khi 5 điều luật cố định không đủ liên quan) — Self-RAG giải quyết nghịch lý này nhờ `[ISREL]` lọc bớt nhiễu trước khi generate. Chi tiết diễn giải ở `RESULTS_ANALYSIS.md` §3.

   Notebook 04 cũng in ra so sánh Recall **trước/sau** `[ISREL]` của Self-RAG — bằng chứng định lượng cho việc lọc nhiễu có giữ được evidence đúng hay không.

3. **Phân tích case cụ thể**: ví dụ câu hỏi mà Self-RAG-inspired lọc được điều luật nhiễu (RAG chuẩn không lọc), và ví dụ câu hỏi mà self-critique phát hiện answer không được hỗ trợ (hallucination) — dùng cho phần Discussion của báo cáo.
4. **Báo cáo cuối kỳ** theo khung §28 của guideline (Introduction → Related Work → Methodology → Implementation → Experiments → Results → Discussion → Conclusion), dùng trực tiếp bảng/case ở trên.
5. **Demo** chạy trực tiếp trong Colab (nhập câu hỏi trong 1 cell, hoặc UI Gradio đơn giản nếu có thời gian).
6. ~~Nộp lên leaderboard VLSP2025 DRiLL~~ — đã hủy: hạn nộp hệ thống là 12/08/2025, đã qua từ lâu tính đến thời điểm làm đồ án (xem §9).

## 8. Giải thích chi tiết từng notebook (chuẩn bị hỏi đáp khi bảo vệ)

Mục này đi sâu vào **cách từng notebook hoạt động và vì sao lại thiết kế như vậy**, dùng để trả lời khi giảng viên hỏi xoáy vào pipeline. Mỗi notebook có 3 phần: *Mục đích & luồng xử lý*, *Quyết định thiết kế quan trọng (kèm lý do)*, và *Câu hỏi thường gặp*.

### 8.1. Notebook 01 — `01_retrieval_baseline.ipynb`

**Mục đích & luồng xử lý**: biến `legal_corpus.json` (2157 văn bản, 59636 điều luật) thành một FAISS index có thể tìm kiếm ngữ nghĩa, rồi đo chất lượng retrieval thuần (chưa có generation).

1. **Chunking**: bộ tách ưu tiên tách theo đoạn/khoản có sẵn trong văn bản luật (paragraph-first); nếu một điều luật không có cấu trúc đoạn rõ ràng hoặc quá dài, dùng fallback cắt cố định 800 ký tự, overlap 150 ký tự (overlap để tránh cắt đứt ý ngay ranh giới chunk).
2. **Embedding**: mỗi chunk được encode bằng `bkai-foundation-models/vietnamese-bi-encoder` (Sentence-BERT dạng bi-encoder, encode câu hỏi và điều luật độc lập thành vector rồi so bằng similarity — khác cross-encoder phải chạy cặp (câu hỏi, điều luật) cùng lúc, không khả thi khi cần so 1 câu hỏi với 59636 điều luật). Vector được **chuẩn hoá (normalize)** trước khi lưu.
3. **Index**: FAISS `IndexFlatIP` (Inner Product trên vector đã chuẩn hoá = cosine similarity). `Flat` nghĩa là tìm kiếm brute-force chính xác tuyệt đối (không xấp xỉ như HNSW/IVF) — chấp nhận được vì quy mô ~60k vector vẫn đủ nhanh trên CPU, không cần đánh đổi độ chính xác lấy tốc độ.
4. **Retrieve**: với mỗi câu hỏi, lấy `top_chunks=50` chunk gần nhất theo cosine similarity, sau đó **khử trùng theo `aid`** (một điều luật có thể bị chia thành nhiều chunk, hoặc nhiều chunk khác nhau map về cùng điều luật) để chỉ giữ `max_k=5` **điều luật khác nhau** đầu tiên — tránh việc "top-5 kết quả" thực chất chỉ là 2-3 điều luật lặp lại.
5. **Dev split**: tách cố định 250 câu từ `train.json` bằng `random.seed(42)`, lưu lại `dev_split_qids.json` — mọi notebook sau đọc lại đúng file này để đảm bảo cả 3 hệ được so sánh trên **cùng một tập câu hỏi**, không phải suy luận lại từ đầu mỗi lần.
6. **Đánh giá**: Recall@5 (top-5 có chứa **ít nhất 1** `aid` đúng không), Precision@5 (bao nhiêu % trong top-5 là đúng), MRR (vị trí của kết quả đúng đầu tiên, nghịch đảo rồi lấy trung bình).

**Quyết định thiết kế quan trọng**:
- Dùng **bi-encoder** thay vì cross-encoder rerank: vì cross-encoder chính xác hơn nhưng phải chạy inference cho từng cặp (câu hỏi, điều luật) — không khả thi ở quy mô 60k điều luật/câu hỏi trên Colab free. Đây là hạn chế đã biết, có thể nêu ở phần Limitations/Future work ("thêm tầng rerank bằng cross-encoder cho top-50 trước khi chọn top-5 cuối").
- Chọn `bkai-foundation-models/vietnamese-bi-encoder` (không phải multilingual model tổng quát): model này pretrain/fine-tune riêng cho tiếng Việt, cho similarity chính xác hơn multilingual model chung chung với văn bản pháp luật (nhiều từ Hán-Việt, cấu trúc câu đặc thù).
- `top_chunks=50` rồi mới lọc còn `max_k=5`: đảm bảo đủ ứng viên để khử trùng `aid` mà vẫn còn đủ 5 điều luật khác nhau (nếu chỉ lấy top-5 chunk thô ngay từ đầu, có thể chỉ còn 2-3 điều luật khác nhau do trùng lặp).

**Câu hỏi thường gặp**:
- *"Tại sao không dùng model retrieval học sẵn end-to-end (như DPR)?"* — Không đủ dữ liệu cặp (câu hỏi, điều luật đúng) để fine-tune riêng một retriever cho domain pháp luật trong khuôn khổ đồ án; dùng bi-encoder pretrained sẵn (zero-shot) là lựa chọn thực tế cho quy mô đồ án.
- *"Recall@5 chỉ ~0.5 có thấp không?"* — Đúng là còn thấp nếu đứng riêng, nhưng đây là **giới hạn tầng retrieval** (không phải lỗi), và chính giới hạn này tạo động lực cho `[ISREL]` ở Hệ 3: lọc nhiễu trong top-5 sẵn có, không thể tạo ra evidence không nằm trong top-5 ngay từ đầu — đây cũng là lý do Self-RAG **giảm Recall** so với Standard RAG (đã phân tích ở `RESULTS_ANALYSIS.md`).

### 8.2. Notebook 02 — `02_generator_baseline.ipynb` (Hệ 2 — Standard RAG)

**Mục đích & luồng xử lý**: baseline RAG "chuẩn" — luôn lấy đúng top-5 điều luật (không lọc, không tự đánh giá gì thêm) rồi generate thẳng.

1. Load lại FAISS index + `dev_split_qids.json` từ Notebook 01 (không build lại).
2. Với mỗi câu hỏi: `retrieve()` lấy 5 điều luật → ghép vào `PROMPT_TEMPLATE` (câu hỏi + context) → gọi LLM qua Groq → lưu câu trả lời + `retrieved_aids` vào `standard_rag_results.jsonl`.
3. Checkpoint theo từng câu (ghi JSONL, `flush()` ngay) — nếu Colab bị ngắt giữa chừng, chạy lại tự bỏ qua câu đã xong.

**Quyết định thiết kế quan trọng**:
- **Prompt ép model từ chối khi thiếu căn cứ** ("nếu các điều luật không đủ thông tin, hãy nói rõ không đủ căn cứ thay vì suy đoán") thay vì để model tự do trả lời: mục đích là giảm hallucination và tạo điều kiện để `[ISSUP]` ở Notebook 4 có thể phân biệt được câu trả lời có bám evidence hay không. Đây cũng chính là nguyên nhân của "nghịch lý Usefolness" phát hiện được ở `RESULTS_ANALYSIS.md` — khi cả 5 điều luật không liên quan, Standard RAG từ chối trả lời thay vì bịa, nên trung thực hơn nhưng bị chấm kém hữu ích hơn.
- **Không hardcode tên model** (`CANDIDATE_MODELS` + `client.models.list()`): vì đã gặp thật lỗi `model_not_found` dù Groq tài liệu ghi model đó là "production" — quyền truy cập model thay đổi theo tài khoản/thời điểm.
- **Rate-limit 2 tầng**: `Retry-After` ngắn (giây) → chờ; dài (hàng trăm giây, lặp lại) → hiểu là quota giờ/ngày của **model đó** đã cạn, tự động xoay sang model tiếp theo trong `CANDIDATE_MODELS` (không phải chờ vô ích).
- **Vì sao cần API key/Groq**: xem giải thích chi tiết ở lượt hỏi trước (tóm tắt: sinh câu trả lời tiếng Việt tự nhiên không thể làm bằng rule-based, và Colab free không đủ mạnh để tự host một LLM đủ tốt).

**Câu hỏi thường gặp**:
- *"Tại sao Standard RAG luôn dùng đúng 5 điều luật dù có thể không liên quan?"* — Đây chính là điểm baseline: mô phỏng RAG "ngây thơ" kinh điển (retrieve cố định rồi generate), làm nền để so sánh với Self-RAG có thêm bước lọc `[ISREL]`.
- *"Vì sao không dùng OpenAI/Anthropic mà dùng Groq?"* — Groq free tier không yêu cầu khai báo billing (đã thử Gemini, bị bắt setup billing); phù hợp ràng buộc "không tốn tiền" của đồ án sinh viên.

### 8.3. Notebook 03 — `03_self_rag_pipeline.ipynb` (Hệ 3 — Self-RAG-inspired, hệ chính)

**Mục đích & luồng xử lý**: thêm 4 module phản tư (mô phỏng reflection tokens của paper) lên trên cùng hạ tầng retrieve/generate.

```text
Question -> [Retrieve] -> (nếu RETRIEVE) retrieve top-5 -> [ISREL] lọc -> Generate (adaptive) -> [ISSUP] -> [ISUSE]
```

1. **`[Retrieve]` (`judge_retrieve`)**: 1 lệnh gọi LLM, hỏi "câu này có cần tra luật không?" → `RETRIEVE`/`NO_RETRIEVE`. Mặc định an toàn khi parse lỗi: `RETRIEVE` (thà tra thừa còn hơn bỏ sót, vì đa số câu hỏi thật trong `train.json` đều cần).
2. **Retrieve** (nếu `RETRIEVE`): giống hệt Notebook 01/02, lấy top-5.
3. **`[ISREL]` (`judge_relevance_batch`)**: **1 lệnh gọi duy nhất cho cả 5 điều luật** (không phải 5 lệnh riêng như paper mô tả) — model trả về JSON list gồm 5 nhãn `RELEVANT`/`IRRELEVANT`. Mặc định an toàn: `RELEVANT` (không âm thầm bỏ evidence chỉ vì parse lỗi).
4. **Generate (`generate_answer`, thích ứng)**: nếu còn điều luật sau lọc → `WITH_CONTEXT_PROMPT`; nếu không (do `NO_RETRIEVE` hoặc `[ISREL]` lọc sạch) → `NO_CONTEXT_PROMPT` (trả lời từ kiến thức chung, tự nói rõ không có trích dẫn cụ thể). Đây là điểm khác biệt hành vi thật so với Standard RAG (luôn có context cố định).
5. **`[ISSUP]` (`judge_support`)**: chỉ gọi khi có evidence còn lại; nếu không có evidence, gán thẳng `NOT_APPLICABLE` (không hỏi LLM đánh giá support với evidence rỗng — vô nghĩa).
6. **`[ISUSE]` (`judge_usefulness`)**: luôn gọi, không phụ thuộc có evidence hay không (đánh giá trải nghiệm người hỏi độc lập với việc có trích dẫn).
7. Ghi tất cả vào `self_rag_results.jsonl` (checkpoint theo câu, resume-safe).

**Quyết định thiết kế quan trọng**:
- **`[ISREL]` gộp thành 1 lệnh gọi batch** (không làm đúng như paper — đánh giá độc lập từng passage): đánh đổi có chủ đích, vì mỗi câu hỏi đã tốn ~5 lệnh gọi (Retrieve + ISREL + Generate + ISSUP + ISUSE), nếu tách ISREL thành 5 lệnh riêng sẽ thành ~9 lệnh/câu — áp lực rate-limit free-tier không chịu nổi ở quy mô 250 câu.
- **`chat()` là hàm generic** (nhận prompt bất kỳ) thay vì viết riêng cho từng module: vì cả 4 module + generator đều cần cùng cơ chế gọi API + rate-limit-failover, tách hàm dùng chung tránh lặp code 5 lần.
- **Mặc định an toàn khác nhau cho từng module khi parse JSON lỗi** — không phải ngẫu nhiên: `RETRIEVE` (thà thừa), `RELEVANT` (thà giữ), đều thiên về "không mất thông tin" hơn là "chính xác tuyệt đối" — triết lý: một lỗi parse ngẫu nhiên không nên làm mất hẳn khả năng trả lời của cả pipeline.
- **Mục Demo (§11) đặt ngay sau Orchestrator (§10), trước vòng lặp 250 câu (§12)**: để demo trực tiếp lúc bảo vệ chỉ cần `Runtime > Run before` tới cell demo, không phải đợi/tốn quota chạy hết 250 câu.

**Câu hỏi thường gặp**:
- *"Vì sao không tự sinh reflection token bằng cách fine-tune như paper gốc?"* — Trả lời bằng lý do ở §5 (Colab free không đủ để train + host critic model 7B+); đồ án mô phỏng bằng prompting một LLM instruction-following có sẵn, đóng vai "giám khảo" — đánh đổi: không cần dữ liệu huấn luyện + không cần GPU train, nhưng chất lượng judge phụ thuộc hoàn toàn vào khả năng làm theo prompt của model có sẵn (không được huấn luyện chuyên biệt cho việc này).
- *"Reflection token trong code có đúng 4 loại như paper không?"* — Đúng cả 4 (`Retrieve`, `ISREL`, `ISSUP`, `ISUSE`), nhưng bỏ token `Continue`/multi-segment của paper (paper còn dùng để quyết định retrieve tiếp giữa chừng khi sinh đoạn dài) — đồ án chỉ làm quyết định retrieve **một lần** ở đầu, phù hợp với câu hỏi QA ngắn (không phải sinh văn bản dài nhiều đoạn).
- *"Tại sao Self-RAG lại có Recall thấp hơn Standard RAG?"* — Vì `[ISREL]` lọc dựa trên **cùng top-5** đã lấy ở bước retrieve (không lấy thêm evidence mới), nên chỉ có thể giữ nguyên hoặc giảm số điều luật đúng, không thể tăng — đây là đánh đổi Precision-Recall thật đã đo được (xem `RESULTS_ANALYSIS.md` §4).

### 8.4. Notebook 04 — `04_evaluation_report.ipynb` (đánh giá & so sánh 3 hệ)

**Mục đích & luồng xử lý**: không retrieval/embedding gì thêm — chỉ tổng hợp, sinh thêm Hệ 1 (No-RAG), rồi chấm điểm hậu kiểm thống nhất.

1. **Tính giao (`common_qids`)**: lấy phần giao `qid` giữa `standard_rag_results.jsonl` và `self_rag_results.jsonl` (không giả định đủ 250 câu, vì rate-limit có thể khiến 2 notebook trước hoàn thành số câu khác nhau).
2. **Sinh No-RAG** ngay trong notebook này (không có notebook riêng vì đây là hệ đơn giản nhất — chỉ hỏi thẳng LLM, không cần retrieve gì): tái dùng đúng `NO_CONTEXT_PROMPT` của Notebook 03 để đảm bảo nhất quán phương pháp giữa các hệ.
3. **Giao lại lần 2 (`final_qids`)**: giao `common_qids` với các câu No-RAG đã sinh xong → đây là tập câu hỏi **cuối cùng** dùng để so sánh cả 3 hệ, đảm bảo công bằng (apple-to-apple, cùng 1 tập câu hỏi cho cả 3).
4. **Dựng lại text điều luật từ `aid`**: `standard_rag_results.jsonl` chỉ lưu `retrieved_aids` (số), không lưu nguyên văn — phải tra lại `legal_corpus.json` để có text làm evidence khi chấm `[ISSUP]` cho Standard RAG.
5. **3 judge hậu kiểm áp dụng thống nhất cho cả 3 hệ**:
   - `judge_correctness` (**mới**, Notebook 02/03 chưa có): so trực tiếp với `answer` gold (văn bản tự do do người viết, không phải đoạn trích) — **tính mới hoàn toàn cho cả 3 hệ**.
   - `judge_support`/`judge_usefulness`: **tính lại (fresh)** cho No-RAG (support luôn `NOT_APPLICABLE`) và Standard RAG (evidence dựng lại từ `retrieved_aids`), nhưng **tái dùng nguyên** `issup_label`/`isuse_label` đã có sẵn từ Notebook 03 cho Self-RAG — có chủ đích, không phải thiếu sót: cùng prompt/temperature=0 đã tính rồi, tính lại chỉ tốn thêm API call mà không đổi kết quả.
6. Xuất `evaluation_details.jsonl` (chi tiết từng câu, cả 3 hệ) + `final_comparison_table.csv` (bảng tổng hợp, có cả `*_rate` — tỷ lệ đạt nhãn tốt nhất — và `*_score` — điểm trọng số 1/0.5/0, sắc thái hơn khi phần lớn câu chỉ đạt "một phần").
7. Tính lại Recall/Precision/MRR **thuần Python** (không qua LLM) cho Standard RAG và Self-RAG **cả trước lẫn sau `[ISREL]`** — đây là cơ sở cho ablation chính của đồ án.

**Quyết định thiết kế quan trọng**:
- **Vì sao chấm lại Correctness/Support/Usefulness ở một notebook riêng thay vì làm ngay trong Notebook 02/03?** Vì cần một bộ tiêu chí **thống nhất** áp dụng đồng đều cho cả 3 hệ (kể cả No-RAG, hệ chưa tồn tại lúc Notebook 02/03 chạy) — làm riêng lẻ trong từng notebook sẽ không so sánh được công bằng.
- **Vì sao Correctness dùng LLM-judge thay vì so khớp chuỗi (BLEU/ROUGE/exact match)?** `answer` trong `train.json` là văn bản tư vấn tự do do người viết (không phải đoạn trích nguyên văn từ luật), nên hai câu trả lời có thể **đúng cùng một ý nghĩa pháp lý** dù diễn đạt hoàn toàn khác câu chữ — metric lexical overlap (BLEU/ROUGE) sẽ chấm sai trong trường hợp này. LLM-judge có thể so sánh **kết luận pháp lý cốt lõi**, không cần trùng từ ngữ (ghi rõ trong `CORRECTNESS_PROMPT`).
- **Đã gặp 3 lỗi thật khi xây notebook này** (đáng kể nhất trong toàn bộ đồ án, nên chuẩn bị kỹ nếu giảng viên hỏi về "khó khăn gặp phải"):
  1. Model "thinking" `qwen/qwen3.6-27b` bọc `<think>...</think>` trước output, làm hỏng cả answer lẫn JSON parse của judge — phát hiện qua thống kê bất thường (100% câu trả lời cùng 1 nhãn, không thể xảy ra nếu judge chấm thật). Sửa bằng `strip_think()`.
  2. Khi tất cả model bị khóa (rate-limit dài hạn), vòng lặp cũ **không dừng** mà tiếp tục ghi hàng trăm dòng giá trị mặc định — làm bảng kết quả trông hợp lệ nhưng gần như toàn bộ là rác. Sửa: dừng sớm + log `_reason` + cảnh báo tự động.
  3. Lỗi **413 "Request too large"** (một request cụ thể — `evidence_block` ghép 5 điều luật dài — vượt giới hạn token/phút của Groq) từng bị hiểu nhầm thành hết quota theo ngày, khiến code đánh dấu nhầm cả 3 model "hỏng vĩnh viễn". Sửa: 413 chỉ bỏ qua **request đó**, không đánh dấu model hỏng; đồng thời cắt bớt độ dài mỗi điều luật trước khi ghép prompt.

**Câu hỏi thường gặp**:
- *"Cỡ mẫu 187/250 có đủ tin cậy không?"* — 74.8% dev set gốc, đây là **giới hạn cứng của free-tier Groq** (đã kiểm chứng qua 3 lần pull độc lập, không phải lỗi code còn sót) — đủ lớn để rút kết luận có ý nghĩa cho một đồ án cuối kỳ; chi tiết kiểm chứng chất lượng dữ liệu ở `RESULTS_ANALYSIS.md` §2.
- *"Vì sao Self-RAG Support/Usefulness reuse từ Notebook 03 mà không chấm lại?"* — Tiết kiệm API call có chủ đích (đã giải thích ở trên) — không phải vì thiếu thời gian implement.
- *"`*_rate` và `*_score` khác nhau chỗ nào, dùng cái nào cho báo cáo?"* — `rate` chỉ đếm tỷ lệ đạt nhãn cao nhất tuyệt đối (nghiêm khắc), `score` cho điểm một phần (1/0.5/0) nên phản ánh sắc thái tốt hơn khi phần lớn câu trả lời chỉ "một phần đúng/hỗ trợ/hữu ích" — nên trình bày cả 2 trong báo cáo, `score` phù hợp hơn để so sánh xu hướng tổng thể.
- *"Ablation chỉ làm trên `[ISREL]`, còn 3 token kia (`[Retrieve]`, `[ISSUP]`, `[ISUSE]`) có đánh giá không?"* — Có, bổ sung ở `RESULTS_ANALYSIS.md` §6, tính hoàn toàn từ dữ liệu 187/202 sẵn có (không cần chạy lại notebook nào): (1) `[Retrieve]` — nhóm 9 câu tự quyết định `NO_RETRIEVE` có Correctness thấp hơn cả khi ép Standard RAG truy xuất trên đúng 9 câu đó (0% vs 11.1%) — cho thấy module này đôi khi bỏ qua truy xuất quá tay, dù n=9 rất nhỏ; (2) `[ISSUP]` và `[ISUSE]` đối chiếu với `Correctness` chấm độc lập ở Notebook 04 đều cho xu hướng đơn điệu đúng hướng (nhãn tốt hơn ↔ Correctness cao hơn) — chứng minh 2 module này là tín hiệu dự đoán thật, không phải nhãn ngẫu nhiên; hạn chế đi kèm: 17.9% câu được `[ISUSE]` gắn nhãn USEFUL vẫn `INCORRECT`, tức module này đo "nghe có vẻ hữu ích" nhiều hơn "đúng sự thật". **Ngoài bảng số (định lượng), §6 còn có phân tích định tính đọc trực tiếp từng câu trả lời thật** (qid cụ thể: 4000, 14790, 11746 cho `[Retrieve]`; 15, 3099 cho `[ISSUP]`; 1227, 2997, 4769 cho `[ISUSE]`) — phát hiện chung: cả 3 module đều có xu hướng chấm dựa trên **hình thức trình bày** (tự tin, mạch lạc, đủ chi tiết, có trích dẫn) nhiều hơn **nội dung đúng/sai thực chất**, ví dụ rõ nhất là câu từ chối an toàn "không đủ căn cứ" vẫn được `[ISSUP]` chấm FULLY_SUPPORTED dù sai (qid 15), và câu trả lời sai nhưng tự tin vẫn được `[ISUSE]` chấm USEFUL (qid 1227). Nên trình bày cả 2 lớp bằng chứng này trong báo cáo — bảng số cho biết "có ý nghĩa hay không", ví dụ định tính giải thích "vì sao/cơ chế nào".

## 9. Trạng thái hiện tại

- [x] Dataset đã verify, hiểu rõ schema (`train.json` có nhãn thật, `public_test`/`private_test` nhãn ẩn).
- [x] `01_retrieval_baseline.ipynb` — chunking, embedding, FAISS index, Recall@k/Precision@k/MRR trên dev set (đã chạy).
- [x] `02_generator_baseline.ipynb` — baseline RAG generator qua Groq API (đổi từ Gemini vì Gemini bắt setup billing; model tự dò qua `client.models.list()`, ưu tiên `openai/gpt-oss-120b`, tự xoay vòng khi bị khóa quota dài hạn), checkpoint JSONL resume-safe → `standard_rag_results.jsonl`.
- [x] `03_self_rag_pipeline.ipynb` — 4 module reflection (Retrieve-decision, ISREL batch, ISSUP, ISUSE), sinh `self_rag_results.jsonl`.
- [x] `04_evaluation_report.ipynb` — sinh Hệ No-RAG, chấm Correctness/Support/Usefulness thống nhất cho cả 3 hệ trên phần giao nhau đã chạy xong, xuất `final_comparison_table.csv`. Đã vá 3 lỗi riêng biệt (xem `RESULTS_ANALYSIS.md` §8): lỗi `<think>` (26/08/2026), lỗi dừng sớm khi judge bị mặc định hàng loạt (07/09/2026), và lỗi 413 "Request too large" bị hiểu nhầm thành hết quota (07/09/2026). **Lần pull 09/09/2026 (187/250 câu) là lần đầu tiên dữ liệu Correctness/Support/Usefulness đáng tin cậy — đã kiểm chứng chất lượng, dùng được cho báo cáo, không cần chạy lại nữa.**
- [x] `RESULTS_ANALYSIS.md` — phân tích kết quả từ lần pull mới nhất (hiện tại: 09/09/2026, 187/250 câu, **dữ liệu cuối cùng dùng cho báo cáo**): bảng so sánh chính (Self-RAG thắng cả 3 chỉ số Correctness/Support/Usefulness), số liệu retrieval, **ablation đầy đủ trên cả 4 token phản tư ở §6** (bổ sung 13/09/2026: `[Retrieve]` — 9 câu NO_RETRIEVE thua cả Standard RAG bị ép truy xuất; `[ISREL]`→`[ISSUP]` — tái hiện nhất quán qua 3 lần pull độc lập, 39%→90%; `[ISSUP]`/`[ISUSE]` → Correctness — cả 2 đơn điệu đúng hướng, nhưng `[ISUSE]` có 17.9% nhãn USEFUL vẫn sai), phát hiện nghịch lý Usefulness của Standard RAG (thấp hơn cả No-RAG do hay từ chối trả lời), case study cụ thể với `_reason` thật của judge, mapping vào khung báo cáo §28. **Tài liệu này được viết lại (không phải nối thêm) mỗi lần có pull artifacts mới — luôn đọc lại bản mới nhất, đừng giả định số mục §... giữ nguyên giữa các lần.**
- [x] Demo — mục "11. Demo" trong `03_self_rag_pipeline.ipynb`, đặt **ngay sau Orchestrator (§10) và trước vòng lặp dev set nặng (§12)** một cách có chủ đích: muốn demo (ví dụ lúc báo cáo cuối kỳ) chỉ cần chạy notebook từ đầu tới hết §11 (Colab: chuột phải cell demo → "Run before"), không phải đợi qua vòng lặp 250 câu ở §12. Sửa `DEMO_QUESTIONS` rồi chạy lại cell là ra ngay answer + nguồn trích dẫn + reflection report.
- [x] ~~`05_drill_submission.ipynb`~~ — **đã hủy**. Fetch trang chính thức https://vlsp.org.vn/vlsp2025/eval/drill xác nhận hạn nộp hệ thống là **12/08/2025 23:59 UTC** (nộp qua Codabench, chấm bằng Recall/Precision/Macro-F2) — đã qua hơn 1 năm tính đến thời điểm làm đồ án này, không còn đường nộp thật. Quyết định (do người dùng chọn): dừng hẳn, không build notebook dự đoán trên `public_test.json`/`private_test.json` nữa, tập trung thời gian còn lại cho báo cáo/slide.
- [ ] Báo cáo + slide (tái sử dụng nội dung từ `seminar/SELF_RAG_SEMINAR_DETAILED_GUIDE.md` cho phần liên quan tới paper gốc).
