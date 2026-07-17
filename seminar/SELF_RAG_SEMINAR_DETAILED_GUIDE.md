# Self-RAG Seminar Detailed Guide

Tài liệu này tập trung riêng cho **luồng Seminar** về bài báo:

> **Self-RAG: Learning to Retrieve, Generate, and Critique through Self-Reflection**

Mục tiêu là giúp xây dựng một bài seminar có mạch rõ ràng, đủ chiều sâu học thuật, dễ thuyết trình, và thể hiện được cả nội dung bài báo lẫn ý nghĩa của chủ đề trong NLP hiện đại.

---

## 1. Mục tiêu của bài seminar

Seminar không nên chỉ tóm tắt bài báo theo thứ tự từ abstract đến experiments. Một bài seminar tốt cần làm được 4 việc:

1. Giải thích được **vấn đề thực tế**: LLM mạnh nhưng dễ hallucinate.
2. Giải thích được **giải pháp nền**: RAG giúp mô hình dựa vào tài liệu ngoài.
3. Chỉ ra được **hạn chế của RAG truyền thống**: retrieve cố định, không tự đánh giá tài liệu, không tự kiểm tra câu trả lời.
4. Làm rõ được **đóng góp của Self-RAG**: mô hình học cách retrieve, generate và critique thông qua reflection tokens.

Sau khi nghe xong, người nghe nên trả lời được:

- Self-RAG là gì?
- Self-RAG khác RAG truyền thống ở đâu?
- Reflection tokens có vai trò gì?
- Critic model và generator model được dùng như thế nào?
- Vì sao Self-RAG giúp cải thiện factuality?
- Hạn chế của Self-RAG là gì?

---

## 2. Thông điệp trung tâm

Thông điệp nên xuyên suốt toàn bài:

> RAG truyền thống giúp LLM có thêm kiến thức, nhưng retrieval vẫn còn bị động. Self-RAG làm cho retrieval trở nên có điều kiện, có kiểm soát, và có khả năng tự đánh giá.

Có thể tóm gọn bằng 3 cụm:

```text
Adaptive Retrieval
Self-Critique
Controllable Generation
```

Đây là 3 cụm nên xuất hiện ở phần mở đầu, phần method và phần kết luận.

---

## 3. Flow tổng thể của seminar

Flow nên đi từ dễ đến khó:

```text
1. LLM và hallucination
2. RAG như một hướng giải quyết
3. Hạn chế của RAG truyền thống
4. Ý tưởng Self-RAG
5. Reflection tokens
6. Training pipeline
7. Inference pipeline
8. Experiments và results
9. Analysis, limitations, discussion
10. Conclusion và Q&A
```

Nếu seminar dài **20 phút**, nên dùng khoảng **18 slide**.

Nếu seminar dài **30 phút**, nên dùng khoảng **24 slide**.

Nếu seminar dài **45 phút**, có thể dùng **28-32 slide**, thêm nhiều ví dụ và phân tích thí nghiệm hơn.

---

## 3.1. Nguyên tắc bám sát bài báo khi làm seminar

Khi trình bày Self-RAG, nên tránh nói quá chung chung kiểu "Self-RAG giúp mô hình thông minh hơn". Thay vào đó, mỗi nhận định quan trọng nên bám vào một phần cụ thể trong bài báo.

Các phần nên được trích dẫn trong slide:

| Nội dung trình bày | Vị trí trong bài báo | Cách dùng trong seminar |
|---|---|---|
| Vấn đề của LM và RAG truyền thống | Abstract, Section 1 | Dùng để mở bài và đặt motivation |
| Self-RAG học retrieve, generate, critique | Abstract, Section 1 | Dùng làm thesis chính |
| Reflection tokens | Section 3, Table 1 | Dùng cho phần method cốt lõi |
| Critic model | Section 3.1 | Dùng để giải thích cách tạo supervision |
| Generator model | Section 3.2 | Dùng để giải thích training generator |
| Inference algorithm | Section 3.3, Algorithm 1 | Dùng để giải thích adaptive retrieval khi suy luận |
| Experimental setup | Section 4 | Dùng khi giới thiệu benchmark và baseline |
| Main results | Section 5, các bảng kết quả | Dùng khi nói hiệu quả thực nghiệm |
| Analysis/ablation | Section 5.3 | Dùng để chứng minh reflection tokens có đóng góp thực sự |
| Ethical concerns/limitations | Cuối bài báo | Dùng cho phần đánh giá và thảo luận |

Quy tắc trình bày:

- Claim nào đến từ bài báo thì ghi nhỏ trong slide: `Source: Asai et al., 2023, Section/Table ...`.
- Không cần trích nguyên văn dài. Chỉ cần paraphrase chính xác và nêu vị trí.
- Với Table 1, nên đưa bảng reflection tokens gần với paper nhất có thể.
- Với kết quả thực nghiệm, nên nói xu hướng và ý nghĩa, không đọc toàn bộ số.
- Phần đánh giá cuối bài nên tách rõ: **điểm mạnh**, **hạn chế**, **khả năng cải tiến**.

---

## 3.2. Bảng claim và dẫn chứng từ bài báo

Bảng này dùng như "xương sống học thuật" cho seminar. Khi làm slide, có thể lấy từng dòng làm speaker note hoặc citation nhỏ ở cuối slide.

| Claim trong seminar | Dẫn chứng từ paper | Ý nghĩa khi thuyết trình |
|---|---|---|
| LLM có thể tạo câu trả lời không factual, và retrieval được dùng để giảm vấn đề này | Abstract và Introduction nêu vấn đề LLM thường tạo response không factual, còn retrieval-augmented LM dùng external knowledge để cải thiện factuality | Đây là lý do cần RAG và Self-RAG |
| RAG truyền thống có hạn chế vì thường retrieve một số lượng passage cố định | Introduction chỉ ra nhiều phương pháp retrieval-augmented generation retrieve passages indiscriminately, dù retrieval không phải lúc nào cũng cần | Dẫn vào adaptive retrieval |
| Self-RAG huấn luyện một LM để retrieve, generate và critique | Abstract mô tả Self-RAG là framework giúp LM tự retrieve passages on demand, generate, và critique thông qua reflection tokens | Đây là đóng góp chính |
| Reflection tokens là cơ chế trung tâm | Section 3 và Table 1 liệt kê retrieval token và critique tokens cho relevance, support, utility | Dùng để giải thích method |
| Critic model tạo supervision cho reflection tokens | Section 3.1 mô tả critic model được huấn luyện để dự đoán reflection tokens | Giải thích vì sao generator học được self-reflection |
| Generator được fine-tune để sinh cả output và reflection tokens | Section 3.2 mô tả generator model học từ data đã được chèn reflection tokens | Phân biệt critic training và generator inference |
| Inference có thể điều chỉnh behavior bằng reflection-token scores | Section 3.3 và Algorithm 1 mô tả decoding dùng reflection tokens, retrieval frequency, relevance/support/utility scores | Dẫn tới controllable generation |
| Self-RAG cải thiện factuality và citation-related quality trong long-form generation | Section 5 báo cáo kết quả trên long-form generation, human evaluation và citation/factuality metrics | Dùng trong phần results |
| Reflection/retrieval components có đóng góp qua ablation | Section 5.3 phân tích ablation và retrieval behavior | Dùng để chứng minh method không chỉ là thêm retrieval |
| Tác giả vẫn thừa nhận rủi ro về factuality và misuse | Ethical concerns nêu mô hình vẫn có thể tạo nội dung không factual và có nguy cơ misuse | Dùng cho phần hạn chế |

---

## 4. Phân bổ thời gian đề xuất

### Seminar 20 phút

| Phần | Thời lượng |
|---|---:|
| Motivation: LLM, hallucination, RAG | 4 phút |
| Problem statement | 2 phút |
| Self-RAG overview | 3 phút |
| Reflection tokens | 4 phút |
| Training và inference | 4 phút |
| Experiments, limitations, conclusion | 3 phút |

### Seminar 30 phút

| Phần | Thời lượng |
|---|---:|
| Motivation và background | 6 phút |
| Problem statement | 3 phút |
| Self-RAG overview | 4 phút |
| Reflection tokens | 6 phút |
| Training pipeline | 4 phút |
| Inference pipeline | 3 phút |
| Experiments và analysis | 3 phút |
| Limitations, conclusion, Q&A bridge | 1 phút |

### Seminar 45 phút

| Phần | Thời lượng |
|---|---:|
| Motivation và NLP context | 8 phút |
| RAG background | 6 phút |
| Standard RAG limitations | 5 phút |
| Self-RAG method | 10 phút |
| Training và inference | 7 phút |
| Experiments | 5 phút |
| Limitations và discussion | 3 phút |
| Conclusion | 1 phút |

---

## 5. Cấu trúc slide chi tiết

Phần này là khung slide-by-slide. Mỗi slide gồm:

- **Mục tiêu**: slide này cần làm rõ điều gì.
- **Nội dung nên có**: các bullet hoặc hình minh họa.
- **Script nói gợi ý**: lời thuyết trình có thể luyện theo.

---

## Slide 1. Title

### Mục tiêu

Giới thiệu chủ đề, bài báo, nhóm/tên người trình bày.

### Nội dung nên có

```text
Self-RAG
Learning to Retrieve, Generate, and Critique through Self-Reflection

Seminar for NLP Course
Presenter: ...
```

### Script nói gợi ý

```text
Hôm nay em sẽ trình bày bài báo Self-RAG: Learning to Retrieve, Generate, and Critique through Self-Reflection.
Đây là một bài báo về Retrieval-Augmented Generation, tập trung vào việc giúp language model không chỉ sinh câu trả lời, mà còn biết khi nào cần truy xuất tài liệu và tự đánh giá câu trả lời của mình.
```

---

## Slide 2. Motivation: Vì sao chủ đề này quan trọng?

### Mục tiêu

Tạo lý do để người nghe quan tâm.

### Nội dung nên có

- LLM có khả năng sinh văn bản rất mạnh.
- Nhưng LLM có thể hallucinate.
- Trong QA hoặc các tác vụ factual, câu trả lời cần có căn cứ.
- RAG là một hướng phổ biến để giảm hallucination.

### Script nói gợi ý

```text
Các mô hình ngôn ngữ lớn hiện nay có thể trả lời rất tự nhiên và thuyết phục.
Tuy nhiên, một vấn đề quan trọng là chúng có thể tạo ra thông tin sai, gọi là hallucination.
Điều này đặc biệt nguy hiểm trong các tác vụ cần độ chính xác cao như hỏi đáp, giáo dục, y tế hoặc pháp luật.
Vì vậy, một hướng nghiên cứu quan trọng là làm sao để mô hình trả lời dựa trên bằng chứng thay vì chỉ dựa vào kiến thức đã học trong tham số.
```

---

## Slide 3. Hallucination trong LLM

### Mục tiêu

Giải thích rõ vấn đề nền.

### Nội dung nên có

Định nghĩa:

```text
Hallucination = mô hình sinh ra thông tin sai hoặc không có căn cứ,
nhưng trình bày như thể đó là sự thật.
```

Ví dụ:

```text
Question: Who discovered penicillin?
Wrong answer: Marie Curie discovered penicillin.
Correct answer: Alexander Fleming discovered penicillin.
```

### Script nói gợi ý

```text
Hallucination không chỉ là việc mô hình không biết câu trả lời.
Vấn đề là mô hình có thể tạo ra một câu trả lời nghe rất hợp lý nhưng lại sai.
Vì LLM sinh token dựa trên xác suất, nó không mặc định có cơ chế kiểm chứng sự thật.
Đó là lý do các phương pháp kết hợp external knowledge trở nên quan trọng.
```

---

## Slide 4. Retrieval-Augmented Generation

### Mục tiêu

Nhắc lại RAG ở mức dễ hiểu.

### Nội dung nên có

Pipeline:

```text
Question
  -> Retriever
  -> Retrieved Documents
  -> Generator
  -> Answer
```

Ưu điểm:

- Cung cấp kiến thức ngoài.
- Có thể cập nhật bằng cách cập nhật kho tài liệu.
- Giúp answer grounded hơn.
- Có thể cung cấp nguồn/citation.

### Script nói gợi ý

```text
Retrieval-Augmented Generation, hay RAG, là cách kết hợp Information Retrieval với Text Generation.
Thay vì để LLM trả lời trực tiếp, hệ thống trước tiên truy xuất các đoạn tài liệu liên quan.
Sau đó, generator dùng câu hỏi và các tài liệu này để sinh câu trả lời.
Về ý tưởng, RAG giúp mô hình có thêm bằng chứng trước khi trả lời.
```

---

## Slide 5. Hạn chế của RAG truyền thống

### Mục tiêu

Tạo cầu nối sang Self-RAG.

### Nội dung nên có

Các hạn chế:

- Retrieve cố định cho mọi câu hỏi.
- Không phải câu hỏi nào cũng cần retrieval.
- Tài liệu retrieve có thể không liên quan.
- Generator có thể dùng sai tài liệu.
- Generator không tự biết câu trả lời có được tài liệu hỗ trợ không.

Minh họa:

```text
Question: What is 2 + 2?
Retrieval needed? No.

Question: What are the reflection tokens in Self-RAG?
Retrieval needed? Yes.
```

### Script nói gợi ý

```text
RAG rất hữu ích, nhưng RAG truyền thống vẫn còn bị động.
Thông thường hệ thống luôn retrieve top-k tài liệu rồi đưa vào prompt.
Điều này tạo ra hai vấn đề.
Thứ nhất, có những câu hỏi không cần retrieval nhưng hệ thống vẫn retrieve.
Thứ hai, nếu tài liệu retrieve không liên quan, generator có thể bị nhiễu và sinh câu trả lời sai.
```

### Dẫn chứng từ paper

- **Introduction** của bài báo nhấn mạnh rằng nhiều hệ thống RAG retrieve passages một cách không phân biệt, trong khi một số input có thể không cần retrieval.
- Đây là điểm paper dùng để dẫn tới nhu cầu **retrieve on demand** thay vì luôn retrieve cố định.
- Khi làm slide, có thể ghi citation nhỏ: `Source: Asai et al., 2023, Introduction`.

---

## Slide 6. Research Question

### Mục tiêu

Đặt câu hỏi nghiên cứu trung tâm.

### Nội dung nên có

```text
Can a language model learn to:
1. Decide when to retrieve?
2. Evaluate whether retrieved passages are relevant?
3. Generate answers grounded in evidence?
4. Critique whether its own answer is supported and useful?
```

### Script nói gợi ý

```text
Câu hỏi nghiên cứu của Self-RAG là:
Liệu một language model có thể học cách tự quyết định khi nào cần truy xuất thông tin,
tự đánh giá tài liệu truy xuất,
và tự kiểm tra câu trả lời của chính nó hay không?
Đây là điểm chuyển từ RAG bị động sang RAG có khả năng tự phản tư.
```

---

## Slide 7. Self-RAG Overview

### Mục tiêu

Giới thiệu framework Self-RAG ở mức tổng quan.

### Nội dung nên có

Self-RAG học 3 khả năng:

```text
Retrieve
Generate
Critique
```

Pipeline tổng quát:

```text
Input
  -> decide retrieve or not
  -> retrieve passages if needed
  -> evaluate passages
  -> generate answer
  -> evaluate answer
```

### Script nói gợi ý

```text
Self-RAG là framework huấn luyện language model để làm ba việc cùng lúc:
retrieve, generate và critique.
Điểm quan trọng là critique không phải bước hậu xử lý hoàn toàn tách biệt.
Nó được học thông qua các token đặc biệt gọi là reflection tokens.
Nhờ vậy, mô hình có thể kiểm soát tốt hơn quá trình trả lời.
```

---

## Slide 8. Ý tưởng cốt lõi: Reflection Tokens

### Mục tiêu

Giới thiệu thành phần quan trọng nhất.

### Nội dung nên có

Định nghĩa:

```text
Reflection tokens are special tokens that allow the model to
decide, evaluate, and control the generation process.
```

Các token chính:

- Retrieve
- ISREL
- ISSUP
- ISUSE

### Script nói gợi ý

```text
Reflection tokens là các token đặc biệt được chèn vào quá trình sinh.
Thay vì chỉ sinh câu trả lời cuối cùng, mô hình còn sinh các tín hiệu điều khiển.
Các tín hiệu này cho biết có cần retrieve không, passage có liên quan không,
câu trả lời có được hỗ trợ không, và câu trả lời có hữu ích không.
```

### Dẫn chứng từ paper

- **Section 3** giới thiệu reflection tokens như thành phần giúp LM tự kiểm soát quá trình retrieval và generation.
- **Table 1** là nguồn quan trọng nhất cho slide này vì liệt kê các loại reflection tokens và chức năng của chúng.
- Nên ghi trên slide: `Source: Table 1, Asai et al., 2023`.

---

## Slide 9. Retrieve Token

### Mục tiêu

Giải thích adaptive retrieval.

### Nội dung nên có

```text
[Retrieve] = Yes / No / Continue
```

Vai trò:

- Quyết định có cần truy xuất tài liệu.
- Tránh retrieve không cần thiết.
- Cho phép retrieve nhiều lần nếu cần.

Ví dụ:

```text
Question: What is 2 + 2?
[Retrieve] = No

Question: What is the main contribution of Self-RAG?
[Retrieve] = Yes
```

### Script nói gợi ý

```text
Retrieve token giúp mô hình quyết định khi nào cần gọi retriever.
Nếu câu hỏi đơn giản hoặc có thể trả lời bằng kiến thức chung, mô hình có thể chọn No.
Nếu câu hỏi cần bằng chứng từ tài liệu ngoài, mô hình chọn Yes.
Đây chính là adaptive retrieval.
```

---

## Slide 10. ISREL Token

### Mục tiêu

Giải thích cách Self-RAG đánh giá passage.

### Nội dung nên có

```text
[ISREL] = Relevant / Irrelevant
```

Vai trò:

- Đánh giá passage có liên quan với câu hỏi không.
- Giúp loại bỏ tài liệu nhiễu.
- Giảm nguy cơ generator dùng sai evidence.

### Script nói gợi ý

```text
Sau khi retrieve, không phải passage nào cũng hữu ích.
ISREL token dùng để đánh giá một passage có liên quan đến câu hỏi hay không.
Nếu passage không liên quan, mô hình có thể giảm ảnh hưởng hoặc bỏ qua passage đó.
Điều này rất quan trọng vì retrieval sai có thể làm generation sai.
```

---

## Slide 11. ISSUP Token

### Mục tiêu

Giải thích factual support.

### Nội dung nên có

```text
[ISSUP] = Fully supported / Partially supported / No support
```

Vai trò:

- Kiểm tra câu trả lời có được evidence hỗ trợ không.
- Giúp đánh giá factuality.
- Giảm hallucination.

### Script nói gợi ý

```text
ISSUP đánh giá mối quan hệ giữa câu trả lời và evidence.
Một câu trả lời có thể nghe hay, nhưng nếu không được tài liệu hỗ trợ thì vẫn không đáng tin.
Token này giúp Self-RAG kiểm tra xem output có grounded trong passage được retrieve hay không.
```

---

## Slide 12. ISUSE Token

### Mục tiêu

Giải thích usefulness.

### Nội dung nên có

```text
[ISUSE] = Useful / Partially useful / Not useful
```

Vai trò:

- Đánh giá câu trả lời có thực sự hữu ích với người dùng không.
- Bổ sung cho factuality.
- Một câu trả lời đúng nhưng quá thiếu chi tiết vẫn có thể chưa hữu ích.

### Script nói gợi ý

```text
Một câu trả lời được evidence hỗ trợ chưa chắc đã là câu trả lời tốt nhất.
Nó còn cần đầy đủ, rõ ràng và trả lời đúng nhu cầu của người dùng.
ISUSE token đánh giá khía cạnh này.
Vì vậy Self-RAG không chỉ kiểm tra đúng sai, mà còn kiểm tra usefulness.
```

---

## Slide 13. Bảng tổng hợp Reflection Tokens

### Mục tiêu

Củng cố toàn bộ token.

### Nội dung nên có

| Token | Chức năng | Câu hỏi được trả lời |
|---|---|---|
| Retrieve | Quyết định retrieval | Có cần truy xuất không? |
| ISREL | Đánh giá passage | Passage có liên quan không? |
| ISSUP | Đánh giá support | Answer có được evidence hỗ trợ không? |
| ISUSE | Đánh giá usefulness | Answer có hữu ích không? |

### Script nói gợi ý

```text
Tóm lại, Self-RAG dùng reflection tokens để kiểm soát nhiều điểm trong pipeline.
Retrieve kiểm soát việc có gọi retriever không.
ISREL kiểm tra tài liệu.
ISSUP kiểm tra sự hỗ trợ của evidence.
ISUSE kiểm tra chất lượng câu trả lời đối với người dùng.
```

### Dẫn chứng từ paper

Trong **Table 1**, bài báo chia reflection tokens thành 2 nhóm chính:

1. **Retrieval token**: điều khiển việc có retrieve hay không.
2. **Critique tokens**: đánh giá passage và output.

Các critique tokens gồm:

- **IsREL**: passage có relevant với input không.
- **IsSUP**: generation có được passage hỗ trợ không.
- **IsUSE**: generation có hữu ích không.

Lưu ý khi làm slide:

- Với **IsUSE**, paper dùng mức đánh giá utility theo thang điểm, không chỉ nhãn nhị phân useful/not useful.
- Nếu muốn đơn giản hóa cho seminar, có thể trình bày thành `Useful / Partially useful / Not useful`, nhưng speaker note nên ghi rõ đây là bản diễn giải đơn giản hóa từ utility score trong paper.

---

## Slide 14. Training Pipeline Overview

### Mục tiêu

Giải thích cách Self-RAG học reflection tokens.

### Nội dung nên có

Hai thành phần:

```text
Critic Model
Generator Model
```

Pipeline:

```text
Raw training data
  -> Critic annotates reflection tokens
  -> Data with reflection tokens
  -> Train generator
```

### Script nói gợi ý

```text
Self-RAG không tự nhiên biết sinh các reflection tokens.
Bài báo dùng một critic model để tạo tín hiệu huấn luyện.
Critic được huấn luyện để đánh giá retrieval, relevance, support và usefulness.
Sau đó critic được dùng để annotate dữ liệu.
Generator cuối cùng được fine-tune trên dữ liệu đã có reflection tokens.
```

### Dẫn chứng từ paper

- **Section 3.1** mô tả bước huấn luyện **critic model** để tạo reflection tokens.
- **Section 3.2** mô tả bước huấn luyện **generator model** trên dữ liệu đã có reflection tokens.
- Đây là bằng chứng để giải thích rằng Self-RAG không chỉ là prompt engineering, mà có quá trình huấn luyện để model học self-reflection.

---

## Slide 15. Critic Model

### Mục tiêu

Làm rõ vai trò của critic.

### Nội dung nên có

Critic học đánh giá:

- Retrieval necessity.
- Passage relevance.
- Answer support.
- Answer usefulness.

Output của critic:

```text
[Retrieve]
[ISREL]
[ISSUP]
[ISUSE]
```

### Script nói gợi ý

```text
Critic model có vai trò giống một bộ đánh giá.
Nó không phải thành phần chính để trả lời người dùng cuối.
Thay vào đó, critic tạo ra annotation cho dữ liệu huấn luyện.
Nhờ critic, generator học được khi nào cần retrieve và cách tự đánh giá output.
```

---

## Slide 16. Generator Model

### Mục tiêu

Giải thích generator sau khi học có thể làm gì.

### Nội dung nên có

Generator được huấn luyện để sinh:

- Câu trả lời.
- Reflection tokens.
- Các quyết định liên quan retrieval và critique.

Điểm quan trọng:

```text
At inference time, the generator can produce reflection tokens itself.
```

### Script nói gợi ý

```text
Sau khi dữ liệu đã được annotate, generator được fine-tune để sinh cả câu trả lời và reflection tokens.
Điều quan trọng là ở giai đoạn inference, generator có thể tự sinh các token này.
Nói cách khác, critic giúp dạy generator trong training, còn generator là thành phần chính được dùng khi suy luận.
```

---

## Slide 17. Inference Pipeline

### Mục tiêu

Cho thấy hệ thống chạy như thế nào với input mới.

### Nội dung nên có

```text
1. Receive user input
2. Generate [Retrieve]
3. Retrieve passages if needed
4. Generate [ISREL] for passages
5. Generate answer
6. Generate [ISSUP]
7. Generate [ISUSE]
8. Select or return final answer
```

### Script nói gợi ý

```text
Khi nhận một câu hỏi mới, Self-RAG trước tiên quyết định có cần retrieve hay không.
Nếu cần, hệ thống lấy các passage liên quan.
Sau đó mô hình đánh giá passage, sinh câu trả lời, rồi đánh giá câu trả lời có được hỗ trợ và hữu ích không.
Như vậy, inference của Self-RAG là một quá trình generate có kiểm soát.
```

### Dẫn chứng từ paper

- **Section 3.3** và **Algorithm 1** mô tả quá trình inference của Self-RAG.
- Paper cho thấy retrieval có thể được kích hoạt trong quá trình generation khi model sinh retrieval token.
- Decoding không chỉ dựa vào xác suất token thông thường, mà còn có thể dùng điểm từ critique tokens như relevance, support và utility để chọn output tốt hơn.
- Đây là phần nên dùng để giải thích cụm **controllable generation**.

---

## Slide 18. Example Walkthrough

### Mục tiêu

Giúp người nghe hiểu bằng ví dụ cụ thể.

### Nội dung nên có

Ví dụ:

```text
Question:
What are the reflection tokens in Self-RAG?

[Retrieve] Yes

Retrieved Passage:
Self-RAG uses reflection tokens to decide retrieval, evaluate relevance,
assess support, and judge usefulness.

[ISREL] Relevant

Answer:
Self-RAG uses Retrieve, ISREL, ISSUP, and ISUSE tokens.

[ISSUP] Fully supported
[ISUSE] Useful
```

### Script nói gợi ý

```text
Ví dụ này minh họa toàn bộ pipeline.
Vì câu hỏi hỏi trực tiếp về nội dung bài báo, mô hình quyết định retrieve.
Sau khi lấy được passage, nó đánh giá passage là relevant.
Tiếp theo nó sinh câu trả lời về các reflection tokens.
Cuối cùng, nó đánh giá câu trả lời được hỗ trợ đầy đủ và hữu ích.
```

---

## Slide 19. Experiments

### Mục tiêu

Trình bày bài báo đánh giá trên những loại task nào.

### Nội dung nên có

Các nhóm task:

- Open-domain Question Answering.
- Fact Verification.
- Reasoning.
- Long-form Generation.

Các baseline:

- Standard language models.
- RAG baselines.
- Retrieval-augmented models.
- Larger LLMs.

### Script nói gợi ý

```text
Bài báo đánh giá Self-RAG trên nhiều nhóm tác vụ khác nhau.
Điều này quan trọng vì Self-RAG không chỉ được thiết kế cho một benchmark đơn lẻ.
Các tác vụ gồm question answering, fact verification, reasoning và long-form generation.
Tác giả so sánh với các mô hình ngôn ngữ thông thường, các baseline RAG, và cả một số mô hình lớn hơn.
```

### Dẫn chứng từ paper

- **Section 4** trình bày experimental setup.
- Bài báo đánh giá trên nhiều nhóm tác vụ, bao gồm short-form generation như open-domain QA/fact verification/reasoning và long-form generation.
- Khi làm slide, nên ghi rõ đây không phải chỉ là một benchmark đơn lẻ, mà là nhiều nhóm task để kiểm tra factuality, reasoning và generation quality.

---

## Slide 20. Main Results

### Mục tiêu

Rút ra ý nghĩa chính từ kết quả.

### Nội dung nên có

Thông điệp:

- Self-RAG cải thiện factuality.
- Self-RAG tốt hơn nhiều RAG baseline.
- Self-RAG giúp kiểm soát retrieval linh hoạt.
- Model nhỏ hơn vẫn có thể cạnh tranh nhờ retrieval và critique.

### Script nói gợi ý

```text
Kết quả chính cho thấy Self-RAG cải thiện đáng kể khả năng trả lời dựa trên bằng chứng.
Điểm đáng chú ý không chỉ là điểm số cao hơn, mà là cơ chế self-reflection giúp mô hình kiểm soát tốt hơn quá trình generate.
Thay vì retrieve nhiều hơn một cách mù quáng, Self-RAG học cách retrieve khi cần và đánh giá thông tin được retrieve.
```

### Dẫn chứng từ paper

- **Section 5** báo cáo Self-RAG đạt kết quả mạnh trên nhiều task so với các baseline LM và RAG.
- Với long-form generation, bài báo nhấn mạnh cải thiện về factuality và citation/evidence-related quality.
- **Section 5.3** có phân tích bổ sung/ablation để cho thấy các thành phần retrieval và critique đóng vai trò thực sự, không chỉ là thêm nhiều context.
- Khi trình bày, nên tránh nói "Self-RAG luôn tốt nhất trong mọi trường hợp". Cách nói chính xác hơn:

```text
Across the evaluated tasks in the paper, Self-RAG shows consistent improvements over many baselines, especially in factuality-oriented settings.
```

---

## Slide 21. Why Self-RAG Works

### Mục tiêu

Phân tích trực giác vì sao phương pháp hiệu quả.

### Nội dung nên có

Ba lý do:

```text
1. Adaptive retrieval reduces unnecessary context.
2. Relevance critique filters noisy passages.
3. Support/usefulness critique improves grounded generation.
```

### Script nói gợi ý

```text
Self-RAG hiệu quả vì nó giải quyết nhiều điểm yếu của RAG cùng lúc.
Adaptive retrieval giúp tránh việc thêm ngữ cảnh không cần thiết.
ISREL giúp giảm passage nhiễu.
ISSUP và ISUSE giúp mô hình ưu tiên câu trả lời có căn cứ và hữu ích.
Nói ngắn gọn, Self-RAG không chỉ thêm tài liệu vào prompt, mà còn học cách sử dụng tài liệu có trách nhiệm hơn.
```

---

## Slide 22. Limitations

### Mục tiêu

Thể hiện tư duy phản biện.

### Nội dung nên có

Hạn chế:

- Phụ thuộc vào chất lượng retriever.
- Chi phí inference có thể cao hơn.
- Reflection tokens vẫn có thể sai.
- Training phức tạp hơn RAG truyền thống.
- Không loại bỏ hallucination hoàn toàn.

### Script nói gợi ý

```text
Self-RAG là một cải tiến quan trọng, nhưng không phải lời giải hoàn chỉnh.
Nếu retriever lấy sai tài liệu, hệ thống vẫn có thể bị ảnh hưởng.
Ngoài ra, việc sinh và đánh giá reflection tokens làm tăng độ phức tạp.
Quan trọng hơn, self-critique của mô hình không đảm bảo luôn đúng.
Vì vậy Self-RAG nên được hiểu là một bước tiến trong việc giảm hallucination, không phải giải pháp loại bỏ hallucination hoàn toàn.
```

### Dẫn chứng từ paper

- Phần **Ethical Concerns** của bài báo thừa nhận model vẫn có thể tạo nội dung không factual hoặc gây hại.
- Bài báo cũng cho thấy hệ thống phụ thuộc vào retrieval và critique, nên nếu evidence không được truy xuất đúng hoặc reflection sai, output vẫn có rủi ro.
- Đây là cơ sở để phần đánh giá seminar không chỉ khen phương pháp, mà còn phân tích giới hạn thực tế.

---

## Slide 23. Discussion

### Mục tiêu

Liên hệ với NLP và mở rộng suy nghĩ.

### Nội dung nên có

Câu hỏi thảo luận:

- Self-reflection có thể áp dụng cho các task NLP khác không?
- Có nên tin vào đánh giá do chính mô hình sinh ra không?
- Self-RAG khác gì với việc dùng một LLM làm judge riêng?
- Adaptive retrieval có hữu ích trong hệ thống tiếng Việt không?

### Script nói gợi ý

```text
Từ góc nhìn NLP, Self-RAG nằm ở giao điểm giữa Information Retrieval, Text Generation và Evaluation.
Điểm thú vị là mô hình không chỉ tạo văn bản, mà còn tạo tín hiệu đánh giá quá trình tạo văn bản.
Điều này mở ra hướng xây dựng các hệ thống generation có khả năng tự kiểm soát và dễ giải thích hơn.
```

---

## Slide 24. Conclusion

### Mục tiêu

Chốt lại ngắn, rõ, dễ nhớ.

### Nội dung nên có

```text
Self-RAG = RAG + Self-Reflection

Key takeaways:
1. Retrieval should be adaptive.
2. Retrieved evidence should be critiqued.
3. Generated answers should be checked for support and usefulness.
```

### Script nói gợi ý

```text
Tóm lại, Self-RAG mở rộng RAG bằng cách thêm self-reflection vào quá trình sinh.
Thay vì luôn retrieve rồi generate, mô hình học cách quyết định khi nào retrieve,
đánh giá tài liệu retrieve, và kiểm tra câu trả lời của mình.
Đóng góp chính của bài báo là đưa retrieval, generation và critique vào một framework thống nhất.
```

---

## Slide 25. Q&A

### Mục tiêu

Mời câu hỏi và chuẩn bị trả lời.

### Nội dung nên có

```text
Q&A
Thank you!
```

Có thể thêm 3 keyword:

```text
Adaptive Retrieval
Self-Critique
Controllable Generation
```

---

## 6. Hình minh họa nên có trong slide

### Hình 1. RAG truyền thống

```text
Question
   |
   v
Retriever ---> Documents
   |
   v
Generator
   |
   v
Answer
```

Mục đích:

- Giúp người nghe hiểu baseline.
- Dùng trước khi nói hạn chế.

### Hình 2. Self-RAG overview

```text
Question
   |
   v
[Retrieve?] ---- No ----> Generate Answer
   |
  Yes
   |
   v
Retriever
   |
   v
Passages
   |
   v
[ISREL]
   |
   v
Generate Answer
   |
   v
[ISSUP] + [ISUSE]
   |
   v
Final Answer
```

Mục đích:

- Là hình quan trọng nhất.
- Nên trình bày đơn giản, ít chữ.

### Hình 3. Training pipeline

```text
Raw Data
  -> Critic Model
  -> Data with Reflection Tokens
  -> Generator Training
  -> Self-RAG Generator
```

Mục đích:

- Giải thích vì sao cần critic.
- Làm rõ critic chủ yếu dùng để tạo tín hiệu training.

### Hình 4. Reflection token table

Dùng bảng:

| Token | Stage | Function |
|---|---|---|
| Retrieve | Before retrieval | Decide retrieval |
| ISREL | After retrieval | Judge passage relevance |
| ISSUP | After generation | Judge evidence support |
| ISUSE | After generation | Judge usefulness |

---

## 7. Script mở đầu hoàn chỉnh

Có thể dùng gần như nguyên văn:

```text
Trong những năm gần đây, Large Language Models đạt được kết quả rất mạnh trong nhiều tác vụ NLP như hỏi đáp, tóm tắt, hội thoại và sinh văn bản.
Tuy nhiên, một vấn đề lớn của các mô hình này là hallucination, tức là mô hình có thể sinh ra thông tin sai nhưng vẫn trình bày một cách rất tự tin.

Một hướng phổ biến để giảm hallucination là Retrieval-Augmented Generation, hay RAG.
Thay vì để mô hình trả lời trực tiếp, RAG cho mô hình truy xuất tài liệu bên ngoài rồi dùng các tài liệu này để sinh câu trả lời.

Tuy nhiên, RAG truyền thống vẫn còn hạn chế.
Hệ thống thường retrieve một số lượng tài liệu cố định cho mọi câu hỏi, dù không phải câu hỏi nào cũng cần retrieval.
Ngoài ra, nếu tài liệu retrieve không liên quan, mô hình có thể bị nhiễu và sinh câu trả lời sai.

Bài báo Self-RAG đề xuất một hướng tiếp cận mới:
thay vì chỉ retrieve rồi generate, mô hình được huấn luyện để tự quyết định khi nào cần retrieve,
tự đánh giá tài liệu retrieve, và tự kiểm tra câu trả lời của mình.
Điều này được thực hiện thông qua các token đặc biệt gọi là reflection tokens.
```

---

## 8. Script giải thích phương pháp hoàn chỉnh

```text
Điểm cốt lõi của Self-RAG là đưa self-reflection vào quá trình generation.
Ở đây self-reflection không có nghĩa là mô hình có ý thức như con người.
Nó có nghĩa là mô hình sinh ra các token đặc biệt để biểu diễn các quyết định và đánh giá trong quá trình trả lời.

Token đầu tiên là Retrieve, dùng để quyết định có cần truy xuất tài liệu hay không.
Sau khi tài liệu được truy xuất, token ISREL được dùng để đánh giá passage có liên quan đến câu hỏi không.
Tiếp theo, mô hình sinh câu trả lời.
Sau đó, token ISSUP đánh giá câu trả lời có được evidence hỗ trợ hay không,
và token ISUSE đánh giá câu trả lời có hữu ích với người dùng không.

Nhờ các token này, Self-RAG biến quá trình trả lời thành một quá trình có kiểm soát.
Mô hình không chỉ trả lời, mà còn tự tạo ra các tín hiệu để kiểm tra chất lượng của quá trình trả lời.
```

---

## 9. Script giải thích training hoàn chỉnh

```text
Về huấn luyện, Self-RAG sử dụng hai thành phần chính: critic model và generator model.

Critic model được huấn luyện để tạo ra các reflection tokens.
Nó học cách đánh giá khi nào cần retrieve, passage có relevant không,
câu trả lời có được support bởi evidence không, và câu trả lời có useful không.

Sau khi critic model được huấn luyện, nó được dùng để annotate dữ liệu huấn luyện.
Tức là từ dữ liệu câu hỏi-câu trả lời ban đầu, hệ thống thêm vào các reflection tokens tương ứng.

Sau đó, generator model được fine-tune trên dữ liệu đã được annotate này.
Kết quả là generator học cách sinh cả câu trả lời lẫn reflection tokens.
Ở giai đoạn inference, generator có thể tự sinh các token này mà không nhất thiết cần critic model chạy riêng.
```

---

## 10. Script kết luận hoàn chỉnh

```text
Tóm lại, Self-RAG là một cải tiến quan trọng của Retrieval-Augmented Generation.
RAG truyền thống giúp mô hình có thêm thông tin bên ngoài, nhưng retrieval thường còn cố định và thiếu cơ chế tự đánh giá.

Self-RAG giải quyết vấn đề này bằng cách huấn luyện mô hình để retrieve, generate và critique thông qua reflection tokens.
Nhờ vậy, mô hình có thể quyết định khi nào cần retrieve, đánh giá tài liệu retrieve, và kiểm tra câu trả lời có được hỗ trợ cũng như hữu ích hay không.

Đóng góp chính của bài báo không chỉ là dùng retrieval, mà là làm cho retrieval và generation trở nên adaptive, controllable và grounded hơn.
Đây là một hướng tiếp cận quan trọng trong việc xây dựng các hệ thống NLP đáng tin cậy hơn.
```

---

## 11. Câu hỏi Q&A nên chuẩn bị

### Câu 1. Self-RAG khác RAG truyền thống ở đâu?

RAG truyền thống thường retrieve top-k tài liệu rồi generate câu trả lời. Self-RAG cho mô hình tự quyết định khi nào cần retrieve, tự đánh giá passage có liên quan không, và tự đánh giá câu trả lời có được evidence hỗ trợ không.

### Câu 2. Reflection tokens có giống chain-of-thought không?

Không hoàn toàn. Chain-of-thought thường là chuỗi lập luận trung gian. Reflection tokens là các token điều khiển và đánh giá, dùng để quyết định retrieval, đánh giá relevance, support và usefulness.

### Câu 3. Self-RAG có loại bỏ hallucination hoàn toàn không?

Không. Self-RAG giúp giảm hallucination bằng cách tăng evidence support và factuality, nhưng vẫn phụ thuộc vào retriever và khả năng tự đánh giá của mô hình.

### Câu 4. Vì sao cần critic model?

Critic model được dùng để tạo annotation cho reflection tokens trong quá trình training. Sau khi generator được fine-tune trên dữ liệu đã annotate, generator có thể tự sinh các reflection tokens trong inference.

### Câu 5. Nếu retriever lấy sai tài liệu thì Self-RAG có còn hiệu quả không?

Self-RAG có ISREL để đánh giá passage, nên có thể giảm tác động của passage không liên quan. Tuy nhiên, nếu retriever không lấy được evidence đúng, hệ thống vẫn có thể thất bại.

### Câu 6. Self-RAG có tốn chi phí hơn RAG thường không?

Có thể tốn hơn vì có thêm bước sinh reflection tokens, đánh giá passage và đánh giá answer. Tuy nhiên, adaptive retrieval có thể tránh retrieval không cần thiết trong một số trường hợp.

### Câu 7. Có thể áp dụng Self-RAG cho tiếng Việt không?

Có thể. Ý tưởng adaptive retrieval và self-critique không phụ thuộc riêng vào tiếng Anh. Tuy nhiên, hiệu quả sẽ phụ thuộc vào chất lượng retriever tiếng Việt, embedding model, corpus, và LLM dùng để generate/judge.

### Câu 8. Self-RAG có cần fine-tune không?

Trong bài báo gốc, có fine-tune generator với dữ liệu chứa reflection tokens. Trong project nhỏ hoặc demo, có thể mô phỏng Self-RAG bằng prompt-based reflection thay vì fine-tune.

---

## 12. Các điểm dễ bị hỏi khó

### 12.1. "Mô hình tự đánh giá thì có đáng tin không?"

Câu trả lời nên cân bằng:

```text
Không thể tin tuyệt đối.
Self-critique là một tín hiệu hỗ trợ, không phải bằng chứng chắc chắn.
Tuy nhiên, khi được huấn luyện đúng và kết hợp với evidence retrieval, nó giúp cải thiện khả năng kiểm soát output.
```

### 12.2. "Self-RAG có phải chỉ là prompt engineering không?"

```text
Không. Trong bài báo gốc, Self-RAG huấn luyện mô hình để sinh reflection tokens.
Prompt engineering có thể mô phỏng ý tưởng này trong demo, nhưng đóng góp của bài báo là đưa reflection vào quá trình training và inference của model.
```

### 12.3. "Tại sao không dùng RAG rồi thêm một LLM judge ở cuối?"

```text
Cách đó là một pipeline hậu xử lý.
Self-RAG khác ở chỗ reflection tokens được tích hợp vào quá trình generation, giúp mô hình điều khiển cả retrieval và generation trong khi sinh câu trả lời.
```

### 12.4. "Nếu câu hỏi không cần retrieve, Self-RAG làm gì?"

```text
Mô hình có thể sinh [Retrieve] = No và trả lời trực tiếp.
Đây là điểm adaptive: retrieval không còn là bước bắt buộc cho mọi input.
```

---

## 13. Checklist làm slide

Slide nên có:

- [ ] Title rõ ràng.
- [ ] Motivation về hallucination.
- [ ] Pipeline RAG truyền thống.
- [ ] Hạn chế của RAG truyền thống.
- [ ] Research question.
- [ ] Overview Self-RAG.
- [ ] Bảng reflection tokens.
- [ ] Slide riêng cho Retrieve.
- [ ] Slide riêng cho ISREL.
- [ ] Slide riêng cho ISSUP.
- [ ] Slide riêng cho ISUSE.
- [ ] Training pipeline.
- [ ] Inference pipeline.
- [ ] Ví dụ walkthrough.
- [ ] Experiments.
- [ ] Main results.
- [ ] Limitations.
- [ ] Conclusion.
- [ ] Q&A.

---

## 14. Checklist luyện thuyết trình

Trước khi seminar:

- [ ] Nói thử toàn bài một lần không nhìn script.
- [ ] Canh thời gian từng phần.
- [ ] Tập giải thích RAG trong dưới 1 phút.
- [ ] Tập giải thích reflection tokens trong dưới 2 phút.
- [ ] Tập giải thích training pipeline trong dưới 2 phút.
- [ ] Chuẩn bị 5 câu Q&A quan trọng.
- [ ] Kiểm tra slide có ít chữ, nhiều diagram.
- [ ] Đảm bảo mỗi slide chỉ có một ý chính.

---

## 15. Checklist nội dung học thuật

Bài seminar nên thể hiện được:

- [ ] Hiểu đúng motivation của bài báo.
- [ ] Hiểu RAG và vấn đề của RAG.
- [ ] Hiểu reflection tokens.
- [ ] Phân biệt critic và generator.
- [ ] Hiểu training khác inference.
- [ ] Hiểu vì sao Self-RAG giúp factuality.
- [ ] Có nhận xét hạn chế.
- [ ] Có liên hệ với NLP.

---

## 16. Gợi ý thiết kế slide

Nên:

- Dùng ít chữ.
- Dùng diagram pipeline.
- Dùng bảng cho reflection tokens.
- Dùng ví dụ đơn giản.
- Tô màu khác nhau cho retrieval, generation, critique.
- Mỗi slide chỉ truyền tải một thông điệp.

Không nên:

- Copy nhiều đoạn dài từ paper.
- Đọc nguyên bảng kết quả phức tạp.
- Dùng quá nhiều công thức nếu không giải thích.
- Nhảy vào method trước khi người nghe hiểu vấn đề.
- Nói reflection như thể mô hình có ý thức thật.

---

## 17. Một flow kể chuyện ngắn gọn để ghi nhớ

Nếu bị quên script, chỉ cần nhớ mạch này:

```text
LLM có hallucination.
RAG giảm hallucination bằng retrieval.
Nhưng RAG truyền thống retrieve cố định và không tự đánh giá.
Self-RAG thêm reflection tokens.
Các token giúp mô hình quyết định retrieve, đánh giá passage, đánh giá answer.
Nhờ vậy generation trở nên adaptive, grounded và controllable hơn.
```

---

## 18. Kết luận cho phần chuẩn bị seminar

Khi triển khai seminar, nên ưu tiên làm rõ **logic của vấn đề** hơn là đọc nhiều số liệu.

Mạch quan trọng nhất:

```text
Problem: LLM hallucination
Baseline solution: RAG
Limitation: fixed retrieval and no self-critique
Proposed method: Self-RAG with reflection tokens
Impact: better factuality, controllability, and evidence use
```

Nếu bài trình bày làm rõ được mạch này, người nghe sẽ hiểu cả nội dung bài báo và ý nghĩa của chủ đề.

---

## 19. Đánh giá bài báo

Phần này nên đặt gần cuối seminar, sau khi đã trình bày method và results. Mục tiêu là thể hiện bạn không chỉ hiểu bài báo, mà còn có khả năng đánh giá học thuật.

### 19.1. Điểm mạnh của bài báo

#### Điểm mạnh 1. Vấn đề nghiên cứu rõ và quan trọng

Bài báo tập trung vào một vấn đề rất thực tế của LLM: factuality và hallucination. RAG là hướng giải quyết phổ biến, nhưng RAG truyền thống vẫn thiếu khả năng tự kiểm soát.

Dẫn chứng từ paper:

- Abstract và Introduction đặt vấn đề rằng LM có thể tạo response không factual.
- Paper nhấn mạnh retrieval-augmented methods giúp bổ sung knowledge, nhưng retrieval không nên được dùng một cách cứng nhắc cho mọi input.

Cách nói trong seminar:

```text
Điểm mạnh đầu tiên của bài báo là chọn đúng một vấn đề trung tâm của LLM hiện đại: làm sao để câu trả lời có căn cứ hơn.
Self-RAG không phủ nhận RAG, mà chỉ ra rằng RAG cần linh hoạt và có cơ chế tự đánh giá.
```

#### Điểm mạnh 2. Ý tưởng reflection tokens đơn giản nhưng hiệu quả

Reflection tokens là một thiết kế gọn: thay vì thêm một pipeline quá phức tạp, bài báo đưa các quyết định retrieval và critique vào chính chuỗi sinh của model.

Dẫn chứng từ paper:

- Table 1 định nghĩa các token cho retrieval, relevance, support và utility.
- Section 3 cho thấy các token này được dùng trong cả training và inference.

Cách nói trong seminar:

```text
Điểm hay của reflection tokens là chúng biến các quyết định như retrieve hay đánh giá evidence thành một phần của quá trình generation.
Nhờ vậy, mô hình không chỉ sinh answer mà còn sinh tín hiệu kiểm soát answer.
```

#### Điểm mạnh 3. Kết hợp retrieval, generation và critique trong một framework

Nhiều hệ thống có retrieval, hoặc có judge riêng sau generation. Self-RAG đáng chú ý vì tích hợp cả ba thành phần:

```text
Retrieve
Generate
Critique
```

Dẫn chứng từ paper:

- Abstract mô tả framework cho LM học cách retrieve, generate và critique.
- Section 3.3/Algorithm 1 cho thấy các bước này được dùng trong inference.

#### Điểm mạnh 4. Có thực nghiệm đa dạng

Paper không chỉ đánh giá trên một task. Các thí nghiệm trải rộng qua:

- Open-domain QA.
- Fact verification.
- Reasoning.
- Long-form generation.

Dẫn chứng từ paper:

- Section 4 trình bày các nhóm task và baseline.
- Section 5 trình bày kết quả và phân tích.

Cách nói trong seminar:

```text
Việc đánh giá trên nhiều nhóm task giúp kết luận của bài báo thuyết phục hơn.
Self-RAG không chỉ phù hợp với một dạng câu hỏi ngắn, mà còn được kiểm tra trên các tác vụ factual và long-form generation.
```

### 19.2. Hạn chế của bài báo

#### Hạn chế 1. Phụ thuộc vào retriever

Self-RAG có thể đánh giá passage bằng IsREL, nhưng nếu retriever không lấy được evidence đúng ngay từ đầu thì model vẫn thiếu thông tin.

Cách nói:

```text
Self-RAG cải thiện cách sử dụng evidence, nhưng không hoàn toàn giải quyết vấn đề retrieval.
Nếu evidence đúng không nằm trong top retrieved passages, hệ thống vẫn có thể trả lời thiếu hoặc sai.
```

#### Hạn chế 2. Self-critique không đảm bảo luôn đúng

Reflection tokens do model sinh ra vẫn có thể sai. Một model có thể đánh giá nhầm một passage là relevant, hoặc đánh giá câu trả lời là supported trong khi evidence chưa đủ.

Dẫn chứng từ paper:

- Ethical Concerns thừa nhận model vẫn có thể tạo nội dung không factual.
- Điều này gián tiếp cho thấy self-reflection không phải cơ chế đảm bảo tuyệt đối.

#### Hạn chế 3. Chi phí huấn luyện và suy luận cao hơn

So với RAG đơn giản, Self-RAG cần:

- Huấn luyện critic.
- Annotate dữ liệu bằng reflection tokens.
- Fine-tune generator.
- Sinh thêm reflection tokens trong inference.
- Có thể retrieve nhiều lần theo từng segment.

Cách nói:

```text
Self-RAG đổi lấy chất lượng và khả năng kiểm soát bằng chi phí hệ thống cao hơn.
Đây là trade-off quan trọng khi triển khai thực tế.
```

#### Hạn chế 4. Reflection token design còn thủ công

Các loại token như Retrieve, IsREL, IsSUP, IsUSE được thiết kế trước. Điều này hiệu quả nhưng cũng đặt câu hỏi:

- Các token này đã đủ chưa?
- Có task nào cần loại reflection khác không?
- Có thể học reflection schema tự động không?

#### Hạn chế 5. Khả năng tổng quát sang ngôn ngữ/tài liệu khác cần kiểm chứng thêm

Paper chủ yếu đánh giá trong bối cảnh benchmark tiếng Anh. Khi áp dụng cho tiếng Việt hoặc tài liệu chuyên ngành, hiệu quả có thể phụ thuộc mạnh vào:

- Retriever tiếng Việt.
- Embedding model.
- Chất lượng corpus.
- Khả năng judge relevance/support bằng tiếng Việt.

---

## 20. Khả năng cải tiến và hướng mở rộng

Phần này nên trình bày như ý tưởng thảo luận cuối seminar. Không nên nói như chắc chắn tốt hơn paper, mà nên nói là **possible future directions**.

### 20.1. Cải tiến retriever

Vì Self-RAG vẫn phụ thuộc vào evidence được retrieve, một hướng cải tiến là dùng retriever mạnh hơn:

- Hybrid retrieval: kết hợp BM25 và dense embedding.
- Reranker để sắp xếp lại top-k passages.
- Domain-specific retriever cho tài liệu chuyên ngành.
- Vietnamese retriever nếu áp dụng cho tiếng Việt.

Câu nói gợi ý:

```text
Một hướng cải tiến tự nhiên là nâng chất lượng retrieval trước khi critique.
Nếu evidence đầu vào tốt hơn, các bước IsREL và IsSUP cũng có cơ sở tốt hơn.
```

### 20.2. Multi-stage hoặc multi-agent critique

Thay vì để cùng một generator tự đánh giá, có thể dùng nhiều critic độc lập:

- Một critic đánh giá relevance.
- Một critic đánh giá factual support.
- Một critic đánh giá completeness/usefulness.

Lợi ích:

- Giảm rủi ro model tự đánh giá thiên lệch output của chính mình.
- Cho phép kiểm tra chéo giữa các judge.

Trade-off:

- Tốn chi phí inference hơn.
- Pipeline phức tạp hơn.

### 20.3. Calibration cho reflection scores

Một vấn đề quan trọng là reflection score có đáng tin không. Có thể cải tiến bằng:

- Calibration trên validation set.
- So sánh self-critique với human labels.
- Đặt threshold cho support/usefulness.
- Từ chối trả lời khi support score thấp.

Ví dụ:

```text
If support = No support, the system should abstain or ask for more evidence.
```

Điều này giúp hệ thống an toàn hơn trong các ứng dụng cần độ tin cậy cao.

### 20.4. Better citation grounding

Với long-form generation, có thể cải tiến bằng cách yêu cầu mỗi câu hoặc mỗi đoạn trả lời gắn với citation cụ thể.

Hướng mở rộng:

- Sentence-level citation.
- Claim-level evidence matching.
- Highlight span trong passage hỗ trợ từng claim.

Ý nghĩa:

- Người dùng dễ kiểm chứng.
- Giảm hallucination trong câu trả lời dài.

### 20.5. Áp dụng cho tiếng Việt

Với môn NLP, đây là hướng thảo luận rất hay:

```text
Can Self-RAG be adapted to Vietnamese document QA?
```

Các bước có thể làm:

- Dùng corpus tiếng Việt.
- Dùng Vietnamese embedding model.
- Dùng retriever hỗ trợ tiếng Việt.
- Thiết kế prompt/judge tiếng Việt cho relevance, support, usefulness.
- Đánh giá trên câu hỏi tiếng Việt.

Thách thức:

- Tách câu/tách đoạn tiếng Việt.
- Dấu câu và biến thể cách viết.
- Tài liệu chuyên ngành có thuật ngữ Anh-Việt lẫn lộn.
- Thiếu benchmark factuality tiếng Việt.

### 20.6. Efficient Self-RAG

Một hướng cải tiến thực tế là giảm chi phí:

- Chỉ retrieve khi confidence thấp.
- Cache retrieved results.
- Chỉ critique top-k nhỏ.
- Dùng model nhỏ làm critic.
- Dùng early stopping nếu support đã đủ cao.

Câu nói gợi ý:

```text
Trong triển khai thực tế, câu hỏi không chỉ là Self-RAG có chính xác hơn không,
mà còn là nó có đủ nhanh và đủ rẻ để dùng trong hệ thống thật hay không.
```

---

## 21. Slide đánh giá bài báo nên trình bày như thế nào?

Nên có 2 slide cuối trước conclusion:

### Slide: Critical Assessment

Nội dung:

```text
Strengths
- Clear problem: factuality and hallucination
- Elegant mechanism: reflection tokens
- Unified framework: retrieve, generate, critique
- Broad experiments

Limitations
- Depends on retriever quality
- Self-critique can be wrong
- Higher training/inference cost
- Reflection schema is manually designed
```

### Slide: Future Improvements

Nội dung:

```text
Possible Improvements
- Stronger retriever and reranker
- Better calibrated reflection scores
- Independent or multi-stage critics
- Claim-level citation grounding
- Vietnamese/domain-specific Self-RAG
- More efficient inference
```

Script nói gợi ý:

```text
Về tổng thể, Self-RAG là một đóng góp mạnh vì nó đưa critique vào quá trình generation.
Tuy nhiên, phương pháp vẫn phụ thuộc vào chất lượng retrieval và độ tin cậy của self-critique.
Trong tương lai, có thể cải tiến bằng retriever tốt hơn, calibration cho reflection scores,
hoặc áp dụng vào các miền/ngôn ngữ cụ thể như hỏi đáp tài liệu tiếng Việt.
```

---

## 22. Tài liệu tham chiếu cho seminar

Tài liệu chính:

- Asai et al. **Self-RAG: Learning to Retrieve, Generate, and Critique through Self-Reflection**. arXiv:2310.11511, 2023.
- File local trong project: `paper/2310.11511v1.pdf`
- Link arXiv: `https://arxiv.org/abs/2310.11511`

Các vị trí trong paper nên đánh dấu khi đọc:

- Abstract: motivation và đóng góp chính.
- Section 1: vấn đề của RAG/LM và overview.
- Section 3: phương pháp Self-RAG.
- Table 1: reflection tokens.
- Section 3.1: critic model.
- Section 3.2: generator model.
- Section 3.3 và Algorithm 1: inference.
- Section 4: experimental setup.
- Section 5: results và analysis.
- Ethical Concerns: hạn chế/rủi ro.
