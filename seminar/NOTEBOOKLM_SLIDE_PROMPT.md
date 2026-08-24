# Prompt cho NotebookLM tạo slide Seminar Self-RAG

Sử dụng prompt dưới đây trong NotebookLM sau khi đã upload các nguồn:

1. `paper/2310.11511v1.pdf`
2. `seminar/SELF_RAG_SEMINAR_DETAILED_GUIDE.md`

---

## Prompt chính

```text
Bạn là một trợ lý học thuật chuyên hỗ trợ tạo slide seminar cho môn Xử lý ngôn ngữ tự nhiên.

Tôi cần bạn tạo một bộ slide seminar bằng tiếng Việt về bài báo:
"Self-RAG: Learning to Retrieve, Generate, and Critique through Self-Reflection"
của Asai et al., arXiv:2310.11511.

Hãy sử dụng chặt chẽ các tài liệu tôi đã upload, đặc biệt là:
1. File paper gốc Self-RAG.
2. File hướng dẫn seminar chi tiết SELF_RAG_SEMINAR_DETAILED_GUIDE.md.

Yêu cầu quan trọng:
- Nội dung phải bám sát bài báo, không suy diễn quá mức.
- Mỗi claim quan trọng cần có dẫn chứng từ paper, ví dụ: Abstract, Section 1, Section 3, Table 1, Algorithm 1, Section 4, Section 5, Ethical Concerns.
- Không bịa số liệu nếu nguồn không nêu rõ.
- Không trích nguyên văn quá dài từ paper; hãy diễn giải lại bằng tiếng Việt học thuật, dễ hiểu.
- Tập trung làm rõ: motivation, hạn chế của RAG truyền thống, ý tưởng Self-RAG, reflection tokens, critic model, generator model, inference, experiments, results, limitations, critical assessment, future improvements.
- Slide cần phục vụ thuyết trình seminar, không phải báo cáo văn bản dài.

Hãy tạo một bộ slide khoảng 25-26 slide cho bài seminar 25-30 phút, tính cả slide Q&A.

Với mỗi slide, hãy trình bày theo đúng format sau:

Slide <số>. <Tiêu đề slide>

Mục tiêu slide:
- Nêu rõ slide này cần giúp người nghe hiểu điều gì.

Nội dung trên slide:
- Chỉ viết các bullet ngắn gọn, phù hợp để đưa vào PowerPoint.
- Mỗi slide tối đa 4-6 bullet.
- Nếu cần, hãy đề xuất bảng hoặc sơ đồ thay vì quá nhiều chữ.

Gợi ý hình ảnh/sơ đồ:
- Mô tả cụ thể nên vẽ gì trên slide.
- Ví dụ: pipeline RAG, pipeline Self-RAG, bảng reflection tokens, training flow, inference flow.

Speaker notes:
- Viết lời thuyết trình bằng tiếng Việt cho slide đó.
- Giọng văn rõ ràng, học thuật nhưng dễ hiểu.
- Speaker notes nên đủ để tôi luyện nói.

Dẫn chứng từ paper:
- Ghi rõ phần paper nên cite cho slide này.
- Ví dụ: "Source: Asai et al., 2023, Table 1" hoặc "Source: Section 3.3, Algorithm 1".
- Nếu slide dựa nhiều vào file guide thay vì paper, hãy ghi "Based on seminar guide; verify with paper sections ...".

Luồng slide mong muốn:
1. Title
2. Thông tin bài báo: title, authors, affiliations, venue, publication type
3. Motivation: LLM và hallucination
4. Vì sao cần Retrieval-Augmented Generation
5. RAG truyền thống hoạt động như thế nào
6. Hạn chế của RAG truyền thống
7. Research question của Self-RAG
8. Tổng quan Self-RAG
9. Reflection tokens là gì
10. Retrieve token
11. ISREL token
12. ISSUP token
13. ISUSE token
14. Bảng tổng hợp reflection tokens
15. Training pipeline tổng quan
16. Critic model
17. Generator model
18. Inference pipeline
19. Example walkthrough
20. Experimental setup
21. Main results
22. Why Self-RAG works
23. Limitations
24. Critical assessment
25. Future improvements và conclusion
26. Q&A

Yêu cầu riêng cho slide 2 - Thông tin bài báo:
- Ghi đúng tên paper: Self-RAG: Learning to Retrieve, Generate, and Critique through Self-Reflection.
- Ghi đúng tác giả: Akari Asai, Zeqiu Wu, Yizhong Wang, Avirup Sil, Hannaneh Hajishirzi.
- Ghi affiliations: University of Washington, Allen Institute for AI, IBM Research AI.
- Ghi venue: ICLR 2024 - The Twelfth International Conference on Learning Representations.
- Ghi publication type: Conference paper.
- Citation: trang đầu của bản conference paper ghi "Published as a conference paper at ICLR 2024".

Yêu cầu riêng cho phần Critical Assessment:
- Đánh giá điểm mạnh của bài báo.
- Đánh giá hạn chế của bài báo.
- Phân biệt rõ nhận xét của tác giả trong paper và nhận xét phân tích của người trình bày.

Yêu cầu riêng cho phần Future Improvements:
- Đề xuất các hướng cải tiến có cơ sở, không phóng đại.
- Có thể gồm: stronger retriever/reranker, calibrated reflection scores, independent critic, claim-level citation grounding, abstention when evidence is insufficient, conflict-aware evidence checking, query rewriting, áp dụng cho tiếng Việt, efficient Self-RAG.

Sau khi tạo outline 25-26 slide, hãy tạo thêm:
1. Một bảng tóm tắt toàn bộ slide gồm: số slide, tiêu đề, mục tiêu, nguồn paper liên quan.
2. Một checklist những slide bắt buộc phải có citation.
3. Một script mở đầu 1 phút.
4. Một script kết luận 1 phút.
5. 15-20 câu hỏi Q&A có thể bị hỏi, kèm câu trả lời ngắn; chia theo nhóm background, method, experiments, limitations, future work.

Lưu ý:
- Toàn bộ output bằng tiếng Việt.
- Giữ thuật ngữ quan trọng bằng tiếng Anh kèm giải thích tiếng Việt, ví dụ: "adaptive retrieval", "self-critique", "controllable generation", "reflection tokens".
- Nội dung phải phù hợp cho sinh viên cao học/đại học ngành NLP.
- Không biến slide thành bài đọc dài; slide phải gọn, speaker notes mới là phần giải thích chi tiết.
```

---

## Prompt rút gọn nếu NotebookLM trả lời quá dài

```text
Hãy tạo outline 25-26 slide tiếng Việt cho seminar về bài báo Self-RAG dựa trên các nguồn tôi đã upload.

Với mỗi slide, hãy cung cấp:
1. Tiêu đề.
2. 4-6 bullet ngắn để đưa vào slide.
3. Speaker notes khoảng 100-150 từ.
4. Gợi ý hình/sơ đồ.
5. Dẫn chứng từ paper: Abstract/Section/Table/Algorithm tương ứng.

Luồng slide:
Title, thông tin bài báo, Motivation, Hallucination, RAG, hạn chế RAG, research question, Self-RAG overview, reflection tokens, Retrieve, ISREL, ISSUP, ISUSE, training, critic, generator, inference, example, experiments, results, why it works, limitations, critical assessment, future improvements, conclusion, Q&A.

Yêu cầu:
- Bám sát paper Self-RAG.
- Không bịa số liệu.
- Có dẫn chứng từ paper cho claim quan trọng.
- Cuối cùng tạo thêm 15-20 câu Q&A, chia theo nhóm background, method, experiments, limitations, future work.
```

---

## Prompt tạo slide chi tiết từng phần

Nếu muốn NotebookLM làm từng phần để kiểm soát chất lượng, có thể dùng các prompt nhỏ sau.

### 1. Prompt cho phần mở đầu

```text
Dựa trên paper Self-RAG và file seminar guide, hãy tạo nội dung slide 1-6 cho phần mở đầu seminar.

Các slide gồm:
1. Title
2. Thông tin bài báo: authors, affiliations, venue, publication type
3. Motivation: LLM và hallucination
4. Hallucination trong LLM
5. Retrieval-Augmented Generation
6. Hạn chế của RAG truyền thống
7. Research question của Self-RAG

Với mỗi slide, hãy có: nội dung slide, speaker notes, gợi ý hình minh họa, dẫn chứng từ paper.
```

### 2. Prompt cho phần phương pháp

```text
Dựa trên paper Self-RAG và file seminar guide, hãy tạo nội dung slide 8-19 cho phần phương pháp.

Các slide gồm:
8. Self-RAG overview
9. Reflection tokens
10. Retrieve token
11. ISREL token
12. ISSUP token
13. ISUSE token
14. Bảng tổng hợp reflection tokens
15. Training pipeline
16. Critic model
17. Generator model
18. Inference pipeline
19. Example walkthrough

Yêu cầu:
- Bám sát Section 3, Table 1 và Algorithm 1 của paper.
- Giải thích rõ critic model khác generator model ở đâu.
- Làm rõ training khác inference.
- Với mỗi slide, cung cấp bullets, speaker notes, visual suggestion và citation.
```

### 3. Prompt cho phần kết quả và đánh giá

```text
Dựa trên paper Self-RAG và file seminar guide, hãy tạo nội dung slide 20-26 cho phần results, đánh giá và kết luận.

Các slide gồm:
20. Experimental setup
21. Main results
22. Why Self-RAG works
23. Limitations
24. Discussion
25. Critical assessment / Future improvements / Conclusion
26. Q&A

Yêu cầu:
- Bám sát Section 4, Section 5 và Ethical Concerns của paper.
- Không bịa số liệu.
- Với kết quả, hãy diễn giải xu hướng chính và ý nghĩa thay vì đọc toàn bộ bảng.
- Phần Critical Assessment phải có điểm mạnh và hạn chế.
- Phần Future Improvements cần đề xuất cải tiến có cơ sở.
- Với mỗi slide, cung cấp bullets, speaker notes, visual suggestion và citation.
```

---

## Prompt kiểm tra chất lượng slide sau khi NotebookLM tạo xong

```text
Hãy review outline slide vừa tạo theo vai trò một giảng viên NLP.

Kiểm tra các tiêu chí:
1. Có bám sát paper Self-RAG không?
2. Claim quan trọng đã có citation từ paper chưa?
3. Có giải thích rõ RAG truyền thống và hạn chế của nó chưa?
4. Reflection tokens đã được giải thích đúng theo Table 1 chưa?
5. Training pipeline có phân biệt critic model và generator model chưa?
6. Inference pipeline có bám Algorithm 1 chưa?
7. Results có tránh bịa số liệu không?
8. Phần đánh giá bài báo có đủ điểm mạnh, hạn chế và hướng cải tiến không?
9. Slide có quá nhiều chữ không?
10. Flow 25-30 phút có hợp lý không?

Hãy chỉ ra các slide cần sửa và đề xuất phiên bản tốt hơn.
```
