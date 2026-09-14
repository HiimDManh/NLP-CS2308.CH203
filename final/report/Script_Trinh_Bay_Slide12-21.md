# Script thuyết trình — Slide 12 đến hết (bản rút gọn)

Đồ án cuối kỳ CS2308.CH203 · Self-RAG-Inspired Legal QA · Vũ Đức Mạnh · Nguyễn Hoàng Nam

*Bản rút gọn để tiết kiệm thời gian trình bày — mỗi slide chỉ giữ 1 thông điệp chính + 1 lý do vì sao nó quan trọng. Câu chữ có thể điều chỉnh linh hoạt khi nói, không cần đọc nguyên văn.*

---

## Slide 12 — Kết quả chính

Đây là bảng kết quả trên 187/250 câu (giới hạn do dùng API free — em sẽ nói lại ở phần hạn chế). No-RAG chỉ đúng 16% — cho thấy rõ vấn đề hallucination nếu không tra cứu. Standard RAG nhích lên 25,7% nhờ có evidence. Self-RAG-inspired đạt 33,2% — và quan trọng hơn, nó dẫn đầu **cả 3 trục cùng lúc**: Correctness, Support (62%), Usefulness (62,6%) — không đánh đổi trục này lấy trục kia. Đây là bằng chứng chính cho luận điểm của đồ án: self-reflection cải thiện chất lượng câu trả lời toàn diện, dù chỉ mô phỏng bằng prompting.

## Slide 13 — Retrieval trước/sau ISREL

Trước lọc, Self-RAG giống hệt Standard RAG (cùng retriever). Sau khi ISREL lọc: Precision tăng gấp 2,2 lần (0,129 → 0,28), đổi lại Recall giảm ~15-17%. Đây là đánh đổi thật, đã kiểm chứng lại 3 lần độc lập nên tin cậy được, không phải nhiễu ngẫu nhiên.

## Slide 14 — Ablation ISREL → ISSUP (phát hiện quan trọng nhất)

Đây là phát hiện tâm đắc nhất của đồ án. Chia câu hỏi thành 2 nhóm: ISREL không lọc gì, và ISREL lọc bớt ≥1 điều nhiễu — rồi so tỷ lệ câu trả lời có bằng chứng hỗ trợ đầy đủ. Kết quả: 39% ở nhóm không lọc, **90,3%** ở nhóm có lọc — tăng hơn gấp đôi. Đây gần như là bằng chứng nhân quả: chính việc loại bỏ nhiễu, chứ không phải có nhiều evidence, mới quyết định chất lượng câu trả lời. Đã tái hiện đúng xu hướng này qua 3 lần đánh giá độc lập trong quá trình làm đồ án.

## Slide 15 — Phân bố tín hiệu phản tư

Nhanh qua: RETRIEVE nhận biết đúng 95,5% câu cần tra luật; ISSUP cho 63,4% câu Fully Supported, chỉ 1% không được hỗ trợ; ISUSE đánh giá 62% câu hữu ích, chỉ 4% không hữu ích. Các module hoạt động hợp lý, không đoán bừa.

## Slide 16 — Case study: minh chứng thành công

Hai ví dụ nhanh. Một, câu hỏi thừa kế của con dâu: Standard RAG từ chối trả lời (sai), Self-RAG trả lời đúng có trích dẫn. Hai, câu hỏi tài khoản chứng khoán — trong 5 điều truy xuất chỉ 1 điều đúng: Standard RAG bị 4 điều nhiễu che khuất nên sai, Self-RAG nhờ ISREL lọc đúng 1 điều nên trả lời chính xác. Đây là minh hoạ trực quan cho con số ở slide 12/14.

## Slide 17 — Case study: hạn chế thực tế

Hai kiểu lỗi còn tồn tại. Một, hallucination dù trích dẫn đúng format — Self-RAG nói sai "Bộ Xây dựng" thay vì UBND cấp tỉnh, cho thấy ISSUP chỉ kiểm tra có trích dẫn chứ không đảm bảo hiểu đúng nội dung. Hai, lỗi tầng truy xuất — điều luật đúng chưa từng nằm trong top-5 nên phản tư không cứu được. Giới hạn này thuộc về retrieval, không phải self-reflection.

## Slide 18 — Nghịch lý Usefulness của Standard RAG

Phát hiện thú vị: Standard RAG có Usefulness (25,7%) còn thấp hơn No-RAG (50,8%). Vì Standard RAG luôn nhận đúng 5 điều cố định, khi không đủ liên quan thì từ chối trả lời — trung thực nhưng bị chấm vô ích. No-RAG thì cứ trả lời tự tin nên nghe hữu ích hơn dù sai nhiều hơn. Self-RAG giải quyết được nghịch lý này: nhờ ISREL lọc trước, nó vừa đúng hơn (33,2%) vừa hữu ích hơn (62,6%) Standard RAG cùng lúc — bằng chứng trực tiếp cho giá trị của self-reflection.

## Slide 19 — Demo hệ thống

Em xin demo trực tiếp trên Colab (notebook 03, mục 11): nhập câu hỏi → RETRIEVE quyết định → truy xuất + ISREL lọc → generate kèm trích dẫn → ISSUP/ISUSE tự chấm.

*(Demo trực tiếp — dùng lại câu hỏi ở slide 16 để dễ đối chiếu.)*

Đây là hệ thống "hộp trắng" — thấy rõ từng bước suy luận, không chỉ nhận câu trả lời cuối.

## Slide 20 — Kết luận & hướng phát triển

Self-reflection vẫn có giá trị dù chỉ mô phỏng bằng prompting, không fine-tune như paper gốc. Ba bằng chứng: thắng cả 2 baseline trên mọi chỉ số cùng lúc; ISREL tăng hơn gấp đôi tỷ lệ có căn cứ, tái hiện qua nhiều lần đánh giá; giải quyết được nghịch lý trung thực-vs-hữu ích. Hướng tới: tinh chỉnh một critic model tiếng Việt nhẹ thay cho prompting-judge, đánh giá trên mẫu lớn hơn, và đưa phản tư vào cấp độ token khi sinh văn bản.

## Slide 21 — Hạn chế & lời cảm ơn

Nêu nhanh 4 hạn chế: (1) phản tư mô phỏng bằng prompting, không phải critic huấn luyện riêng; (2) dùng LLM làm giám khảo, ~6,4% dòng còn nhiễu; (3) chỉ đánh giá được 187/250 câu do rate-limit API free — giới hạn hạ tầng, không phải phương pháp luận; (4) mới kiểm chứng trên một lĩnh vực, một ngôn ngữ, chưa rõ khả năng khái quát hoá.

Em xin cảm ơn Thầy/Cô đã lắng nghe, rất mong nhận được góp ý ạ.
