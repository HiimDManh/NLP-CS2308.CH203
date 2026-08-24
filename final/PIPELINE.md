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
| `public_test.json` / `private_test.json` | Cùng schema nhưng nhãn bị ẩn — chỉ dùng khi nộp leaderboard chính thức | 312 / 627 câu |

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
│   ├── 02_generator_baseline.ipynb   [CHƯA LÀM]
│   ├── 03_self_rag_pipeline.ipynb    [CHƯA LÀM]
│   ├── 04_evaluation_report.ipynb    [CHƯA LÀM]
│   └── 05_drill_submission.ipynb     [CHƯA LÀM — phương án (b), làm sau cùng]
└── artifacts/              # SINH RA từ notebook, không commit git (.gitignore)
    ├── chunks.faiss                  # FAISS index của ~60k chunk điều luật
    ├── chunks_meta.json              # metadata chunk (aid, law_id, text)
    ├── dev_split_qids.json           # 250 qid cố định dùng chung mọi notebook
    ├── retrieval_baseline_metrics.csv
    ├── generation_results.csv        # (Notebook 02+) câu trả lời + judge scores từng qid
    └── final_comparison_table.csv    # (Notebook 04) bảng so sánh 3 hệ, dùng thẳng cho báo cáo
```

Nguyên tắc: **artifacts luôn tái tạo được từ notebook + dataset gốc**, không bao giờ sửa tay. Vì `dev_split_qids.json` được lưu lại từ notebook 01, mọi notebook sau load lại đúng file này để đảm bảo 3 hệ thống được so trên cùng một tập câu hỏi.

## 5. Vì sao không fine-tune (ràng buộc Colab free)

Paper gốc train một critic model + fine-tune generator (7B+) để tự sinh reflection token. Trên Colab free (T4 15GB, session giới hạn giờ, không đảm bảo persistent runtime), việc này không khả thi. Pipeline dùng **API LLM miễn phí** (Google Gemini free tier hoặc Groq free tier) cho toàn bộ generate + 4 module judge, để:

- Không tốn GPU quota của Colab cho suy luận (dành GPU cho bước encode embedding một lần).
- Không phụ thuộc việc giữ session sống lâu để load một model lớn.

GPU trên Colab (nếu có) chỉ dùng cho bước embedding corpus (`01_retrieval_baseline.ipynb`) — bước duy nhất tốn compute đáng kể và chỉ chạy một lần nhờ cơ chế lưu/load lại `chunks.faiss`.

## 6. Hướng dẫn chạy trên Google Colab

### 6.1. Đưa repo + dataset lên Colab

Dataset (`legal_corpus.json` ~117MB, `legal_dataset.json` ~134MB) **không nằm trong git** — vượt giới hạn 100MB/file của GitHub, và đã được thêm vào `.gitignore` (`final/dataset/VLQA/*.json`). Vì vậy cần 2 bước tách biệt: clone code bằng git, rồi upload dataset lên Drive thủ công **một lần duy nhất** (Drive giữ nguyên giữa các phiên Colab nên không phải lặp lại).

**Bước 1 — clone code (nhanh, không dính giới hạn size vì dataset đã bị loại khỏi git):**

```python
from google.colab import drive
drive.mount('/content/drive')

%cd /content/drive/MyDrive
!git clone https://github.com/HiimDManh/NLP-CS2308.CH203.git
```

Lần sau chỉ cần `%cd /content/drive/MyDrive/NLP-CS2308.CH203 && !git pull` để lấy code/notebook mới nhất.

Nếu repo là **private**, cần Personal Access Token: `!git clone https://<username>:<token>@github.com/HiimDManh/NLP-CS2308.CH203.git` (không paste token vào cell rồi commit/share notebook — chỉ dùng trong phiên Colab).

**Bước 2 — upload dataset vào đúng vị trí (một lần duy nhất, làm trên trình duyệt, không phải trong Colab):**

Sau khi clone xong, trên Drive sẽ có `MyDrive/NLP-CS2308.CH203/final/dataset/VLQA/` nhưng thư mục này **rỗng phần json** (chỉ có `readme.md` được clone theo git). Vào Google Drive trên trình duyệt, mở đúng thư mục đó, kéo-thả 5 file json từ máy bạn (`legal_corpus.json`, `legal_dataset.json`, `train.json`, `public_test.json`, `private_test.json`) vào. Vì đây là bước thủ công một lần, không cần lặp lại mỗi khi mở Colab — notebook sẽ tự tìm thấy dataset ở đúng path `REPO_DIR/final/dataset/VLQA/` như đã cấu hình (không cần sửa code notebook).

**Cách khác (không dùng git cho code):** upload thẳng cả thư mục `final/` (dataset + notebooks) lên Drive qua giao diện web, rồi mở notebook trực tiếp từ Drive. Đơn giản hơn nhưng phải tự cập nhật thủ công mỗi khi code thay đổi.

### 6.2. Mỗi lần mở một notebook

1. `Runtime > Change runtime type > T4 GPU` (cần cho bước embedding ở notebook 01; các notebook sau chủ yếu gọi API nên GPU không bắt buộc, để CPU cũng chạy được).
2. Chạy cell đầu tiên (`pip install`) rồi cell mount Drive — **sửa biến `REPO_DIR`** trong notebook cho đúng path bạn vừa clone/copy ở bước 6.1 (mặc định notebook giả định `/content/drive/MyDrive/NLP-CS2308.CH203`).
3. `Runtime > Run all`.

### 6.3. Thứ tự chạy notebook

Phải chạy theo đúng thứ tự vì mỗi notebook phụ thuộc artifact của notebook trước:

1. `01_retrieval_baseline.ipynb` → sinh `chunks.faiss`, `chunks_meta.json`, `dev_split_qids.json`.
2. `02_generator_baseline.ipynb` → dùng lại index từ bước 1, cần API key LLM (lưu trong Colab Secrets, không hardcode trong notebook).
3. `03_self_rag_pipeline.ipynb` → dùng lại index + generator prompt từ bước 2, thêm 4 module judge.
4. `04_evaluation_report.ipynb` → chạy cả 3 hệ trên dev set, xuất `final_comparison_table.csv` + biểu đồ cho báo cáo.
5. `05_drill_submission.ipynb` → (làm sau cùng, tùy chọn) chạy hệ tốt nhất trên `public_test.json`/`private_test.json`, format đúng chuẩn nộp bài DRiLL.

### 6.4. Xử lý sự cố Colab thường gặp

| Sự cố | Cách xử lý |
|---|---|
| Session bị ngắt giữa chừng lúc encode embedding | Chạy lại notebook 01 — index đã lưu một phần sẽ không tự resume, nhưng vì mất ít hơn ~30-60 phút cho 60k chunk nên chấp nhận chạy lại từ đầu; **không** để mất do quên mount Drive trước khi encode |
| Hết GPU quota free | Chuyển runtime về CPU — embedding vẫn chạy được, chỉ chậm hơn; các notebook 02-05 không cần GPU |
| Rate-limit API LLM (Gemini/Groq free tier) | Thêm retry/backoff (đã tính trong thiết kế notebook 02+), giảm batch câu hỏi mỗi lần chạy, hoặc chia dev set thành nhiều lần chạy nhỏ |
| `REPO_DIR` sai path → `DATA_DIR exists: False` | Kiểm tra lại đường dẫn Drive thực tế bằng `!ls /content/drive/MyDrive` trước khi sửa biến |
| `DATA_DIR exists: True` nhưng notebook báo thiếu `legal_corpus.json`/`train.json` | Quên bước 2 ở §6.1 (upload dataset thủ công) — `git clone` không kéo các file json vì đã bị `.gitignore`; vào Drive kiểm tra `final/dataset/VLQA/` có đủ 5 file json chưa |

## 7. Kết quả cuối cùng của đồ án sẽ là gì

Khi hoàn thành, đồ án gồm các thành phần sau (ánh xạ vào khung báo cáo guideline §28):

1. **Hệ thống chạy được** (notebook 01-04, tái chạy được từ đầu trên Colab free): nhập một câu hỏi pháp luật tiếng Việt, hệ Self-RAG-inspired trả về câu trả lời + danh sách điều luật trích dẫn (`law_id`, số điều) + reflection report (Retrieve/ISREL/ISSUP/ISUSE).
2. **Bảng so sánh định lượng 3 hệ thống** trên cùng 250 câu dev set cố định — ví dụ khung bảng (số liệu thật sẽ điền sau khi chạy notebook 04):

   | Hệ thống | Recall@5 (retrieval) | MRR | Answer quality (LLM-judge) | Support rate (ISSUP) | Usefulness rate (ISUSE) |
   |---|---|---|---|---|---|
   | No-RAG | — | — | ? | ? | ? |
   | Standard RAG | ? | ? | ? | ? | ? |
   | Self-RAG-inspired | ? | ? | ? | ? | ? |

3. **Phân tích case cụ thể**: ví dụ câu hỏi mà Self-RAG-inspired lọc được điều luật nhiễu (RAG chuẩn không lọc), và ví dụ câu hỏi mà self-critique phát hiện answer không được hỗ trợ (hallucination) — dùng cho phần Discussion của báo cáo.
4. **Báo cáo cuối kỳ** theo khung §28 của guideline (Introduction → Related Work → Methodology → Implementation → Experiments → Results → Discussion → Conclusion), dùng trực tiếp bảng/case ở trên.
5. **Demo** chạy trực tiếp trong Colab (nhập câu hỏi trong 1 cell, hoặc UI Gradio đơn giản nếu có thời gian).
6. **(Tùy chọn, phương án (b))** một bản nộp lên leaderboard VLSP2025 DRiLL bằng hệ tốt nhất — cho điểm cộng "được đánh giá độc lập" ngoài phạm vi môn học.

## 8. Trạng thái hiện tại

- [x] Dataset đã verify, hiểu rõ schema (`train.json` có nhãn thật, `public_test`/`private_test` nhãn ẩn).
- [x] `01_retrieval_baseline.ipynb` — chunking, embedding, FAISS index, Recall@k/Precision@k/MRR trên dev set.
- [ ] `02_generator_baseline.ipynb` — baseline RAG generator qua LLM API.
- [ ] `03_self_rag_pipeline.ipynb` — 4 module reflection.
- [ ] `04_evaluation_report.ipynb` — chạy 3 hệ, xuất bảng so sánh cuối.
- [ ] `05_drill_submission.ipynb` — nộp leaderboard (làm sau).
- [ ] Báo cáo + slide (tái sử dụng nội dung từ `seminar/SELF_RAG_SEMINAR_DETAILED_GUIDE.md` cho phần liên quan tới paper gốc).
