# Câu hỏi dự kiến từ giảng viên & câu trả lời

Đồ án cuối kỳ CS2308.CH203 · Self-RAG-Inspired Legal QA

Tài liệu này tập trung vào các câu hỏi **học thuật/phương pháp luận/kết quả** — mang tính chất phản biện, có thể bị hỏi xoáy trong buổi bảo vệ. Câu hỏi về **cách notebook hoạt động cụ thể** (chunking, FAISS, prompt, checkpoint...) đã có riêng ở `final/PIPELINE.md` §8.1–8.4, không lặp lại ở đây. Trả lời trung thực, kể cả khi câu trả lời là "chưa làm được, đây là hạn chế" — giám khảo đánh giá cao sự trung thực hơn là né tránh.

---

## A. Về ý tưởng cốt lõi & so sánh với paper gốc

**1. Đồ án có thực sự cài đặt Self-RAG không, hay chỉ là RAG có thêm vài bước lọc?**

Đồ án **không** cài đặt Self-RAG nguyên bản — paper gốc fine-tune một critic model để tự sinh reflection token *xen kẽ với quá trình generate*, còn đồ án **mô phỏng 4 tín hiệu đó bằng prompting** trên một LLM instruction-tuned có sẵn, gọi rời rạc thành từng bước (không tích hợp vào token stream của generator). Vì vậy tên gọi chính xác là "Self-RAG-**inspired**" — lấy đúng tinh thần (retrieve thích ứng + tự phê bình) chứ không phải bản sao kiến trúc.

**2. Vì sao không fine-tune một critic model nhỏ (LoRA) thay vì chỉ prompting?**

Fine-tune cần dữ liệu huấn luyện có nhãn reflection-token (paper gốc tự sinh nhãn này bằng GPT-4 rồi distill về model nhỏ) — đồ án không có ngân sách để gọi GPT-4 với số lượng lớn để tạo nhãn, và Colab free không đảm bảo runtime đủ dài để train ổn định. Đây là lý do chính ở §5 `PIPELINE.md`, và cũng là hướng phát triển đầu tiên được đề xuất ở slide Kết luận.

**3. Paper gốc có kết quả tốt hơn đồ án này bao nhiêu?**

Không so sánh trực tiếp được — paper gốc đánh giá trên các benchmark tiếng Anh khác hẳn (PopQA, TriviaQA, ARC-Challenge...), không có baseline chung với miền pháp luật tiếng Việt. Đồ án không có tham vọng "đánh bại" paper gốc mà kiểm chứng xem **ý tưởng cốt lõi** của paper (retrieve thích ứng, tự phê bình) có còn tác dụng khi implement bằng cách rẻ hơn nhiều (prompting, không fine-tune) hay không.

---

## B. Về phương pháp luận & thiết kế thí nghiệm

**4. Model dùng để sinh câu trả lời và model dùng để chấm điểm (judge) có phải cùng một model không? Có bị thiên vị không?**

Đúng, cả generate lẫn 3 module judge hậu kiểm đều lấy chung một model từ cùng danh sách `CANDIDATE_MODELS` qua Groq. Đây là một hạn chế thật: nghiên cứu về LLM-as-judge từng chỉ ra hiện tượng **self-preference bias** — model có xu hướng chấm câu trả lời do chính họ model sinh ra cao hơn thực tế. Đồ án chưa kiểm soát được yếu tố này vì chỉ có một nguồn model miễn phí khả dụng ổn định; đây là hạn chế đã nêu và hướng khắc phục là dùng judge từ một họ model khác với generator.

**5. Vì sao không kiểm định ý nghĩa thống kê (statistical significance) giữa các con số, ví dụ 25,7% và 33,2%?**

Đồ án chưa làm kiểm định thống kê hình thức (bootstrap CI, McNemar's test theo từng cặp câu hỏi) do giới hạn thời gian. Điều củng cố độ tin cậy thay thế: kết quả ablation ISREL→ISSUP **tái hiện đúng chiều qua 3 lần đánh giá độc lập** trong suốt quá trình làm đồ án (15%→83%, rồi 39%→90%, rồi lại ~39%→90%) — coi như một dạng kiểm chứng bằng lặp lại (replication) thay vì kiểm định hình thức. Nếu có thêm thời gian, McNemar's test trên từng cặp hệ thống cùng bộ câu hỏi sẽ là bước hợp lý tiếp theo.

**6. Vì sao Self-RAG lại có Recall thấp hơn Standard RAG — đây không phải là điểm yếu sao?**

Đúng là một đánh đổi thật, không che giấu. `[ISREL]` chỉ có thể **lọc bớt** trong số điều luật đã retrieve (top-5), không thể bổ sung evidence mới — nên nếu lọc nhầm loại luôn điều đúng, Recall giảm là hệ quả tất yếu. Điểm quan trọng: dù Recall giảm, chất lượng câu trả lời cuối (Correctness, Usefulness) vẫn tăng — nghĩa là với generator, "ít nhưng đúng" có giá trị hơn "nhiều nhưng lẫn nhiễu". Đây chính là nội dung ablation ở slide 14.

**7. Tại sao không so sánh thêm với các kỹ thuật RAG khác (BM25, hybrid search, cross-encoder rerank)?**

Nằm ngoài phạm vi câu hỏi nghiên cứu chính (self-reflection có cải thiện RAG *cố định* hay không) — nhóm giữ nguyên một retriever cố định xuyên suốt 3 hệ để cô lập đúng biến cần đo (tác động của reflection), tránh nhiễu do đổi retriever. Thêm rerank bằng cross-encoder là hướng mở rộng hợp lý, có thể cải thiện luôn cả 3 hệ.

**8. Vì sao Standard RAG luôn dùng top-5 mà không thử top-3, top-10 để tìm cấu hình tốt nhất?**

Đây là biến được **cố định có chủ đích** để so sánh công bằng, không phải quên tối ưu — nếu thay đổi top-k cho từng hệ khác nhau sẽ không tách bạch được đóng góp thực sự đến từ cơ chế phản tư hay từ việc chọn k tốt hơn. Việc dò k tối ưu là một thí nghiệm ablation khác, chưa nằm trong phạm vi đồ án.

---

## C. Về dữ liệu

**9. Dev set 250 câu có đại diện cho 2190 câu của `train.json` không?**

Có kiểm chứng gián tiếp: khi so retrieval Recall/Precision/MRR giữa toàn bộ dev set 250 câu và các subset nhỏ hơn phát sinh trong quá trình đánh giá (213 câu, rồi 202 câu), số liệu luôn rất gần nhau (ví dụ Recall@5 dao động quanh 0,49–0,50 across nhiều lần đo) — không thấy dấu hiệu thiên lệch lớn. Tuy nhiên nhóm **chưa** làm stratified sampling theo lĩnh vực luật (dân sự, hình sự, hành chính...) để đảm bảo đại diện đều theo chủ đề — đây là một giới hạn có thể nêu nếu bị hỏi sâu.

**10. Corpus pháp luật có bị lỗi thời (luật đã sửa đổi/thay thế) không?**

Corpus là snapshot do ban tổ chức VLSP2025 DRiLL thu thập, đồ án dùng nguyên trạng, không tự cập nhật. Đây là hạn chế chung của mọi hệ RAG dựa trên corpus tĩnh — nhưng cũng chính là **điểm mạnh của kiến trúc RAG so với việc nhồi kiến thức vào tham số model**: muốn cập nhật luật mới chỉ cần thay corpus và re-index, không cần train lại LLM.

**11. Vì sao không dùng toàn bộ 2190 câu của `train.json` để đánh giá thay vì chỉ 250?**

Cân bằng giữa chi phí API (mỗi câu ở Self-RAG tốn ~5 lệnh gọi, ở Notebook 4 tốn thêm ~6 lệnh gọi/câu) và độ tin cậy thống kê cần thiết cho một đồ án cuối kỳ trên hạ tầng miễn phí. 250 câu với seed cố định đã đủ để so sánh có ý nghĩa; việc chạy hết 2190 câu vượt xa khả năng của Groq free tier trong khung thời gian môn học.

---

## D. Về LLM-as-judge & độ tin cậy đánh giá

**12. Có đánh giá độ tương đồng giữa LLM-judge và con người (inter-rater agreement) không?**

Chưa — đây là thiếu sót thật cần thừa nhận thẳng thắn. Nhóm chưa có baseline con người chấm điểm (dù chỉ trên mẫu nhỏ ~20-30 câu) để tính Cohen's kappa hay % đồng thuận với LLM-judge. Độ tin cậy hiện tại chỉ được củng cố gián tiếp qua việc đọc thủ công case study (slide 16-17) thấy nhãn "hợp lý" khi đối chiếu với gold answer — đây là hướng cải thiện rõ ràng và dễ làm nhất nếu có thêm thời gian.

**13. Vì sao tin được là ~6,4% dòng còn nhiễu nhưng vẫn dùng cả bộ 187 câu, không loại bỏ 6,4% đó?**

Nhóm **có thể xác định** dòng nào bị mặc định toàn bộ (nhờ đã thêm log `_reason`), nhưng cố tình không loại bỏ để tránh **selection bias** — nếu chỉ giữ lại các câu "chấm đẹp", bức tranh tổng thể sẽ méo đi theo hướng lạc quan giả tạo. Giữ nguyên toàn bộ 187 câu (kể cả phần nhiễu) và báo cáo trung thực tỷ lệ nhiễu minh bạch hơn là làm sạch thủ công.

**14. `Correctness` so sánh câu trả lời với gold answer tự do (không phải đáp án trắc nghiệm) — làm sao đảm bảo judge chấm nhất quán?**

Prompt của judge cố định `temperature=0` (giảm ngẫu nhiên) và yêu cầu so sánh "kết luận pháp lý cốt lõi", không yêu cầu trùng từ ngữ — nhưng đây vẫn là điểm yếu cố hữu của LLM-as-judge cho văn bản tự do: không có ground-truth tuyệt đối cho "mức độ đúng", chỉ có xấp xỉ tốt nhất trong điều kiện không có ngân sách thuê người chấm tay 187 câu × 3 hệ.

---

## E. Về hạn chế & khả năng ứng dụng thực tế

**15. Với Correctness chỉ ~33%, hệ thống có thể triển khai thực tế cho luật sư/người dân dùng không?**

Chưa — 33% còn quá thấp để dùng như một kênh tư vấn độc lập, đáng tin cậy. Hệ thống ở giai đoạn hiện tại chỉ phù hợp làm **công cụ hỗ trợ nháp** (draft gợi ý + trích dẫn để chuyên gia rà soát lại), không thay thế con người, và bắt buộc phải có disclaimer rõ ràng về giới hạn độ chính xác nếu triển khai thật.

**16. Nếu hệ thống tư vấn sai và người dùng làm theo, ai chịu trách nhiệm?**

Đây là câu hỏi về đạo đức/pháp lý ứng dụng AI, ngoài phạm vi kỹ thuật của đồ án, nhưng là vấn đề thực tế quan trọng cần nhìn nhận nghiêm túc — đúng lý do vì sao ở mức chất lượng hiện tại, hệ thống chỉ nên đóng vai trò hỗ trợ có giám sát của chuyên gia, không nên để người dùng cuối tự ra quyết định pháp lý dựa hoàn toàn vào câu trả lời của hệ thống.

**17. Ý tưởng này có áp dụng được cho lĩnh vực khác (y tế, tài chính) hoặc ngôn ngữ khác không?**

Về nguyên tắc có — kiến trúc 4 module phản tư không phụ thuộc đặc thù pháp luật hay tiếng Việt. Nhưng đồ án **chưa kiểm chứng thực nghiệm** trên lĩnh vực/ngôn ngữ khác, nên đây chỉ là suy luận hợp lý (plausible), không phải kết luận đã được chứng minh — cần nêu rõ ranh giới này khi trả lời để tránh nói quá.

**18. Nếu có thêm 1 tháng và ngân sách không giới hạn, việc đầu tiên nhóm sẽ làm là gì?**

Ưu tiên theo tác động: (1) đánh giá con người trên một mẫu để hiệu chỉnh/kiểm tra độ tin cậy LLM-judge — vá đúng lỗ hổng lớn nhất hiện tại; (2) chạy lại trên toàn bộ 250 câu với một API trả phí ổn định để loại hoàn toàn giới hạn rate-limit; (3) thử judge model khác họ với generator để giảm self-preference bias.

---

## Ghi chú khi trả lời

- Nếu bị hỏi điều **chưa từng nghĩ tới**: thừa nhận thẳng ("đây là điểm nhóm em chưa kiểm chứng, cảm ơn Thầy/Cô đã gợi ý, đây sẽ là hướng cải thiện tiếp theo") — tốt hơn nhiều so với suy đoán/bịa câu trả lời tại chỗ.
- Luôn quay lại **được cái gì, đánh đổi cái gì** — giám khảo đánh giá cao việc trình bày trung thực đánh đổi (recall↓ precision↑, đúng hơn nhưng ít evidence hơn) hơn là chỉ khoe điểm mạnh.
- Với câu hỏi về con số cụ thể, có thể mở nhanh `final/RESULTS_ANALYSIS.md` (đã có sẵn mọi bảng + cách tính) nếu cần tra lại tại chỗ.
