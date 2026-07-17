# Self-RAG: Guideline Seminar va Project cuoi ki

Tai lieu nay tong hop guideline chi tiet cho chu de **Self-RAG: Learning to Retrieve, Generate, and Critique through Self-Reflection**. Noi dung duoc chia thanh 2 luong:

1. **Luong Seminar**: dung de thuyet trinh bai bao khoa hoc trong mon hoc.
2. **Luong Project cuoi ki**: dung de phat trien thanh san pham/thu nghiem thuc hanh dua tren y tuong Self-RAG.

Tai lieu nen duoc dung nhu ban khung de tiep tuc viet slide, script noi, bao cao, demo va phan Q&A.

---

## 1. Tong quan bai bao Self-RAG

### Thong tin bai bao

- Ten bai bao: **Self-RAG: Learning to Retrieve, Generate, and Critique through Self-Reflection**
- Ma arXiv: **2310.11511v1**
- File local: `paper/2310.11511v1.pdf`
- Chu de chinh: Retrieval-Augmented Generation, Large Language Models, Self-Reflection, Factuality, Hallucination Reduction.

### Tom tat nhanh

Self-RAG cai tien RAG truyen thong bang cach cho mo hinh tu quyet dinh:

- Khi nao can truy xuat tai lieu.
- Tai lieu truy xuat co lien quan hay khong.
- Cau tra loi co duoc ho tro boi bang chung hay khong.
- Cau tra loi co huu ich voi nguoi dung hay khong.

Thay vi chi lam:

```text
Question -> Retrieve documents -> Generate answer
```

Self-RAG lam:

```text
Question
  -> Decide whether to retrieve
  -> Retrieve if needed
  -> Critique retrieved passages
  -> Generate answer
  -> Critique answer support and usefulness
  -> Final answer
```

Y tuong cot loi cua bai bao la dua co che **self-reflection** vao qua trinh generation thong qua cac **reflection tokens**.

---

# PHAN A. LUONG SEMINAR

## 2. Muc tieu seminar

Seminar khong nen chi doc lai bai bao. Muc tieu la giup nguoi nghe hieu duoc van de, cach giai quyet va y nghia cua Self-RAG.

Sau seminar, nguoi nghe nen nam duoc:

- RAG la gi.
- Vi sao RAG truyen thong chua du.
- Self-RAG khac RAG truyen thong o diem nao.
- Reflection tokens la gi.
- Self-RAG huan luyen va suy luan nhu the nao.
- Ket qua thuc nghiem chung minh dieu gi.
- Han che va y nghia cua Self-RAG trong NLP/LLM.

Bon cau hoi can tra loi xuyen suot bai:

1. Vi sao LLM can retrieval?
2. Vi sao retrieval co dinh cua RAG truyen thong co van de?
3. Self-RAG dung self-reflection de cai thien RAG nhu the nao?
4. Phuong phap nay co y nghia gi doi voi viec xay dung he thong NLP dang tin cay?

---

## 3. Cau chuyen chinh cua seminar

Mach ke nen di theo logic sau:

```text
LLM rat manh nhung de hallucinate.
RAG giup LLM truy xuat tai lieu ngoai de tra loi chinh xac hon.
Nhung RAG truyen thong thuong retrieve co dinh, khong biet khi nao can retrieve,
khong biet tai lieu co huu ich khong, va khong tu kiem tra cau tra loi.
Self-RAG giai quyet bang cach cho mo hinh tu phan tu:
tu quyet dinh retrieve, tu danh gia tai lieu, tu danh gia cau tra loi.
```

Thong diep nen duoc lap lai trong seminar:

> Self-RAG khong chi la retrieve roi generate. No bien retrieval thanh mot hanh dong co dieu kien, co kiem soat va co tu danh gia.

---

## 4. Kien thuc nen trinh bay truoc khi vao Self-RAG

### 4.1. Large Language Model

Noi ngan gon:

- LLM sinh van ban bang cach du doan token tiep theo.
- LLM manh trong QA, summarization, reasoning, dialogue va text generation.
- Tuy nhien, kien thuc cua LLM phu thuoc vao du lieu huan luyen.
- LLM co the sinh thong tin sai nhung nghe rat thuyet phuc.

Diem nhan:

> Van de lon cua LLM trong cac tac vu can kien thuc thuc te la hallucination.

### 4.2. Hallucination

Hallucination la hien tuong mo hinh tao ra thong tin:

- Sai su that.
- Khong co can cu.
- Khong duoc ho tro boi bang chung.
- Nhung van duoc trinh bay mot cach tu tin.

Vi du:

```text
Question: Who discovered penicillin?
Wrong answer: Marie Curie discovered penicillin.
Correct answer: Alexander Fleming discovered penicillin.
```

Noi voi nguoi nghe:

> Hallucination nguy hiem vi nguoi dung co the tin vao cau tra loi sai, dac biet trong cac linh vuc nhu giao duc, y te, phap luat hoac tai chinh.

### 4.3. Retrieval-Augmented Generation

RAG la co che ket hop truy xuat thong tin va sinh ngon ngu.

Pipeline co ban:

```text
User Question
  -> Retriever searches external documents
  -> Retrieved passages are added to prompt
  -> Generator produces final answer
```

Uu diem cua RAG:

- Bo sung kien thuc ben ngoai cho LLM.
- Giam phu thuoc vao kien thuc trong tham so mo hinh.
- Co the cap nhat tri thuc bang cach cap nhat kho tai lieu.
- Co the cung cap bang chung hoac citation.
- Giam hallucination trong nhieu truong hop.

Han che cua RAG truyen thong:

- Thuong retrieve voi chien luoc co dinh.
- Khong phai cau hoi nao cung can retrieve.
- Neu retrieve sai tai lieu, generator co the bi nhieu.
- Generator khong tu biet tai lieu co lien quan hay khong.
- Generator khong tu kiem tra cau tra loi co duoc tai lieu ho tro hay khong.

---

## 5. Van de nghien cuu

Slide nay nen dat cau hoi:

> Lam the nao de language model co the tu quyet dinh khi nao can truy xuat thong tin, su dung thong tin nao, va tu kiem tra xem cau tra loi cua minh co duoc ho tro boi bang chung hay khong?

Co the trinh bay cac van de cua RAG truyen thong:

- Retrieval co dinh.
- Retrieval khong can thiet voi cau hoi don gian.
- Tai lieu truy xuat co the khong lien quan.
- Generator co the dung sai evidence.
- Khong co co che critique output.

Vi du:

```text
Question: What is 2 + 2?
```

Cau nay khong can retrieval.

```text
Question: What is the main contribution of Self-RAG?
```

Cau nay can retrieval vi lien quan den noi dung bai bao.

Ket luan cua phan nay:

> Retrieval nen la hanh dong adaptive, khong phai buoc co dinh.

---

## 6. Gioi thieu Self-RAG

Self-RAG la framework giup language model:

- Tu quyet dinh co can retrieve hay khong.
- Tu danh gia tai lieu truy xuat co lien quan khong.
- Tu danh gia cau tra loi co duoc ho tro boi tai lieu khong.
- Tu danh gia cau tra loi co huu ich khong.

Diem moi:

> Self-RAG tich hop retrieval, generation va critique vao cung mot qua trinh thong qua reflection tokens.

Tu "self-reflection" o day nen hieu theo nghia ki thuat:

- Mo hinh sinh them token dac biet.
- Cac token nay dung de dieu khien va danh gia qua trinh sinh.
- Khong nen giai thich nhu mo hinh that su co y thuc hay suy nghi nhu con nguoi.

---

## 7. Reflection tokens

Day la phan cot loi cua seminar. Nen danh 3-4 slide cho muc nay.

### 7.1. Bang tong hop reflection tokens

| Token | Vai tro | Cau hoi ma token tra loi |
|---|---|---|
| Retrieve | Quyet dinh truy xuat | Co can lay tai lieu ngoai khong? |
| ISREL | Danh gia tai lieu | Tai lieu nay co lien quan khong? |
| ISSUP | Danh gia ho tro | Cau tra loi co duoc evidence ho tro khong? |
| ISUSE | Danh gia huu ich | Cau tra loi co huu ich khong? |

### 7.2. Retrieve token

Dung de quyet dinh co can retrieval khong.

```text
[Retrieve] = Yes / No / Continue
```

Y nghia:

- `Yes`: can truy xuat tai lieu.
- `No`: khong can truy xuat.
- `Continue`: tiep tuc sinh, chua can retrieve them.

### 7.3. ISREL token

Danh gia tai lieu truy xuat co lien quan khong.

```text
[ISREL] = Relevant / Irrelevant
```

Y nghia:

- Neu tai lieu lien quan, mo hinh co the dung lam evidence.
- Neu tai lieu khong lien quan, mo hinh co the bo qua.

### 7.4. ISSUP token

Danh gia cau tra loi co duoc ho tro boi tai lieu khong.

```text
[ISSUP] = Fully supported / Partially supported / No support
```

Y nghia:

- Kiem tra factuality.
- Giam kha nang sinh thong tin khong co bang chung.

### 7.5. ISUSE token

Danh gia cau tra loi co huu ich khong.

```text
[ISUSE] = Useful / Partially useful / Not useful
```

Y nghia:

- Cau tra loi khong chi can dung, ma con can phu hop voi cau hoi.
- Giup danh gia tinh day du va gia tri cua output.

---

## 8. Pipeline Self-RAG

### 8.1. RAG truyen thong

```text
Input x
  -> Retriever
  -> Documents
  -> Generator
  -> Output y
```

### 8.2. Self-RAG

```text
Input x
  -> Generator decides [Retrieve]
  -> If needed: retrieve documents
  -> Generator checks [ISREL]
  -> Generate answer segment
  -> Check [ISSUP]
  -> Check [ISUSE]
  -> Final answer
```

Thong diep:

> Self-RAG khong retrieve mot cach mu quang. No retrieve khi can, loc tai lieu, sinh cau tra loi, roi tu danh gia cau tra loi.

---

## 9. Training cua Self-RAG

Self-RAG khong tu nhien biet sinh reflection tokens. Bai bao huan luyen qua 2 thanh phan:

1. Critic model.
2. Generator model.

### 9.1. Buoc 1: Huan luyen critic model

Critic model duoc huan luyen de du doan reflection tokens.

Critic hoc cach tra loi:

- Khi nao can retrieve?
- Document co relevant khong?
- Output co supported khong?
- Output co useful khong?

Critic co the duoc huan luyen tu du lieu co annotation hoac du lieu duoc tao bang mo hinh manh hon.

### 9.2. Buoc 2: Gan reflection tokens vao du lieu

Sau khi critic hoc duoc cach danh gia, no duoc dung de annotate du lieu huan luyen.

Du lieu ban dau:

```text
Question: Who wrote the novel 1984?
Answer: George Orwell wrote 1984.
```

Du lieu sau khi gan reflection tokens:

```text
Question: Who wrote the novel 1984?
[Retrieve: Yes]
Document: 1984 is a dystopian novel by George Orwell.
[ISREL: Relevant]
Answer: George Orwell wrote 1984.
[ISSUP: Fully supported]
[ISUSE: Useful]
```

### 9.3. Buoc 3: Huan luyen generator model

Generator duoc fine-tune tren du lieu da co reflection tokens.

Ket qua:

- Generator hoc cach sinh cau tra loi.
- Generator hoc cach sinh reflection tokens.
- Khi inference, generator co the tu kiem soat retrieval va generation.
- Critic khong nhat thiet phai duoc dung truc tiep luc inference.

Diem can nhan manh:

> Critic duoc dung trong training de tao tin hieu phan tu. Generator sau khi hoc co the tu sinh reflection tokens trong inference.

---

## 10. Inference cua Self-RAG

Khi co cau hoi moi, Self-RAG xu ly theo cac buoc:

1. Nhan input tu nguoi dung.
2. Sinh token `[Retrieve]`.
3. Neu can, goi retriever de lay tai lieu.
4. Danh gia tung tai lieu bang `[ISREL]`.
5. Sinh cau tra loi dua tren tai lieu lien quan.
6. Danh gia cau tra loi bang `[ISSUP]`.
7. Danh gia do huu ich bang `[ISUSE]`.
8. Chon cau tra loi tot nhat.

Vi du:

```text
Question: What is the main contribution of Self-RAG?

[Retrieve: Yes]
Retrieved passage: Self-RAG trains a LM to retrieve, generate, and critique...
[ISREL: Relevant]
Answer: The main contribution is a framework that enables LMs to adaptively retrieve passages and self-critique generations using reflection tokens.
[ISSUP: Fully supported]
[ISUSE: Useful]
```

---

## 11. Ket qua thuc nghiem

Self-RAG duoc danh gia tren nhieu nhom tac vu:

- Question Answering.
- Open-domain QA.
- Fact verification.
- Reasoning.
- Long-form generation.

Baseline co the gom:

- Standard LM.
- RAG-based models.
- Retrieval-augmented baselines.
- Larger LLMs.

Thong diep ket qua:

1. Self-RAG cai thien factuality.
2. Self-RAG tot hon RAG truyen thong trong nhieu tac vu.
3. Self-RAG giup retrieval linh hoat hon.
4. Mo hinh nho hon co the canh tranh voi mo hinh lon hon nho retrieval va self-reflection.

Cach noi khi thuyet trinh:

> Ket qua cho thay viec them co che self-reflection khong chi giup mo hinh sinh cau tra loi chinh xac hon, ma con giup mo hinh biet khi nao can dua vao tai lieu va khi nao khong. Day la diem khac biet so voi RAG truyen thong.

---

## 12. Dong gop chinh

Slide "Main Contributions" co the gom:

1. De xuat framework Self-RAG ket hop retrieval, generation va critique.
2. Gioi thieu reflection tokens de dieu khien qua trinh sinh.
3. Cho phep adaptive retrieval thay vi retrieval co dinh.
4. Cai thien factuality va citation quality.
5. Cho phep kiem soat output thong qua relevance, support va usefulness.

Diem nhan:

> Dong gop khong chi nam o viec dung retrieval, ma o viec mo hinh hoc cach tu kiem soat retrieval va tu danh gia output.

---

## 13. Han che

Mot seminar tot nen co phan phan tich han che.

### 13.1. Phu thuoc vao chat luong retriever

Neu retriever lay tai lieu sai hoac thieu, Self-RAG van bi anh huong.

### 13.2. Chi phi inference cao hon

Do co retrieval va danh gia nhieu tai lieu, thoi gian xu ly co the tang.

### 13.3. Reflection tokens khong dam bao dung tuyet doi

Mo hinh co the tu danh gia sai.

### 13.4. Training phuc tap hon RAG thong thuong

Can critic model, du lieu duoc annotate va fine-tuning.

### 13.5. Chua loai bo hoan toan hallucination

Self-RAG giam hallucination nhung khong giai quyet triet de.

Cau noi ket:

> Self-RAG la mot buoc tien quan trong, nhung khong phai loi giai hoan chinh cho factuality trong LLM.

---

## 14. Lien he voi mon hoc NLP

Self-RAG nam o giao diem cua nhieu mang NLP:

- Information Retrieval.
- Question Answering.
- Text Generation.
- Natural Language Understanding.
- Factual Consistency.
- Evaluation of Generated Text.
- Self-evaluation trong LLM.

Cau noi goi y:

> Self-RAG nam o giao diem giua IR va NLG: no dung retrieval de cung cap kien thuc, dung generation de tao cau tra loi, va dung self-critique de danh gia chat luong ngon ngu sinh ra.

---

## 15. Cau truc slide goi y

Neu seminar khoang 20-30 phut, co the dung 18-24 slide.

1. Title  
   Self-RAG: Learning to Retrieve, Generate, and Critique through Self-Reflection.

2. Motivation  
   LLMs are powerful but hallucinate.

3. Background: LLM  
   LLM sinh van ban nhung co gioi han kien thuc.

4. Background: RAG  
   Retrieve tai lieu roi generate cau tra loi.

5. Limitation of Standard RAG  
   Retrieval co dinh, khong tu danh gia evidence.

6. Research Problem  
   Lam sao de mo hinh biet khi nao retrieve va tu kiem tra cau tra loi?

7. Self-RAG Overview  
   Framework tong quan.

8. Key Idea: Reflection Tokens  
   Gioi thieu token dac biet.

9. Retrieve Token  
   Khi nao can truy xuat?

10. ISREL Token  
    Tai lieu co lien quan khong?

11. ISSUP Token  
    Cau tra loi co duoc ho tro khong?

12. ISUSE Token  
    Cau tra loi co huu ich khong?

13. Training Pipeline  
    Critic model + generator model.

14. Critic Model  
    Hoc cach annotate reflection tokens.

15. Generator Model  
    Fine-tune de sinh cau tra loi va reflection tokens.

16. Inference Pipeline  
    Quy trinh khi gap cau hoi moi.

17. Example  
    Minh hoa mot cau hoi cu the.

18. Experiments  
    Cac nhom task duoc danh gia.

19. Results  
    Self-RAG vuot baseline o nhieu task.

20. Why It Works  
    Adaptive retrieval + self-critique + controllability.

21. Limitations  
    Chi phi, phu thuoc retriever, reflection co the sai.

22. Discussion  
    Y nghia voi NLP va LLM.

23. Conclusion  
    Tong ket dong gop.

24. Q&A  
    Cau hoi thao luan.

---

## 16. Flow thuyet trinh goi y

### 16.1. Mo dau

```text
Trong nhung nam gan day, Large Language Models dat ket qua rat manh trong nhieu tac vu NLP.
Tuy nhien, mot van de lon la hallucination, tuc la mo hinh co the sinh ra thong tin sai nhung van rat tu tin.
Retrieval-Augmented Generation, hay RAG, la mot huong pho bien de giam van de nay bang cach cho mo hinh truy xuat tai lieu ben ngoai.
Tuy nhien, RAG truyen thong van con han che vi thuong truy xuat mot cach co dinh va khong tu danh gia tai lieu hay cau tra loi.
Bai bao Self-RAG de xuat mot huong tiep can moi: cho mo hinh tu quyet dinh khi nao can truy xuat, tu danh gia tai lieu, va tu phe binh cau tra loi thong qua cac reflection tokens.
```

### 16.2. Chuyen sang phuong phap

```text
Diem cot loi cua Self-RAG la bien qua trinh generate tu mot qua trinh mot chieu thanh mot qua trinh co kiem soat.
Mo hinh khong chi sinh answer, ma con sinh ra cac tin hieu phan tu de kiem tra tung buoc.
```

### 16.3. Noi ve ket qua

```text
Ket qua thuc nghiem cho thay Self-RAG khong chi cai thien do chinh xac, ma con cai thien factuality va kha nang su dung evidence.
Dieu nay cho thay van de khong chi la retrieve nhieu hon, ma la retrieve dung luc va biet kiem tra thong tin duoc retrieve.
```

### 16.4. Ket luan

```text
Tom lai, Self-RAG la mot cai tien quan trong cua RAG.
Thay vi coi retrieval la mot buoc co dinh ben ngoai mo hinh, Self-RAG dua retrieval va critique vao chinh qua trinh generation.
Day la huong tiep can co y nghia trong viec xay dung cac he thong NLP dang tin cay hon, dac biet voi cac tac vu can thong tin chinh xac va co can cu.
```

---

## 17. Phan nen nhan manh khi seminar

Ba diem quan trong nhat:

### 17.1. Adaptive Retrieval

Self-RAG khong retrieve moi luc. No hoc khi nao can retrieve.

### 17.2. Self-Critique

Mo hinh tu danh gia tai lieu va cau tra loi.

### 17.3. Controllable Generation

Reflection tokens giup dieu khien output theo relevance, support va usefulness.

Neu nguoi nghe chi nho 3 cum nay sau seminar, bai trinh bay da dat muc tieu:

```text
Adaptive Retrieval
Self-Critique
Controllable Generation
```

---

## 18. Cau hoi Q&A nen chuan bi

### 18.1. Self-RAG khac RAG truyen thong o dau?

RAG truyen thong thuong retrieve truoc roi generate. Self-RAG cho mo hinh tu quyet dinh khi nao retrieve, danh gia tai lieu co lien quan khong, va danh gia cau tra loi co duoc ho tro boi tai lieu khong.

### 18.2. Reflection tokens co phai chain-of-thought khong?

Khong hoan toan. Reflection tokens la cac token dieu khien va danh gia qua trinh generation. Chung khong nhat thiet la lap luan chi tiet tung buoc nhu chain-of-thought.

### 18.3. Self-RAG co loai bo hallucination hoan toan khong?

Khong. No giup giam hallucination bang cach tang factuality va evidence support, nhung van phu thuoc vao retriever va kha nang tu danh gia cua mo hinh.

### 18.4. Vi sao can critic model neu cuoi cung chi dung generator?

Critic model dung trong training de tao annotation cho reflection tokens. Sau khi generator duoc fine-tune, generator co the tu sinh cac token nay trong inference.

### 18.5. Self-RAG co ton chi phi hon RAG thuong khong?

Co the co, vi mo hinh can sinh them reflection tokens, truy xuat co dieu kien, va danh gia nhieu doan tai lieu. Tuy nhien, no co the tranh retrieval khong can thiet trong mot so truong hop.

---

## 19. Checklist chuan bi seminar

Truoc ngay trinh bay, can co:

- Slide hoan chinh khoang 20-24 slide.
- Mot hinh pipeline RAG truyen thong.
- Mot hinh pipeline Self-RAG.
- Mot bang reflection tokens.
- Mot vi du minh hoa inference.
- Mot slide ket qua chinh.
- Mot slide han che.
- Mot slide ket luan.
- Script noi ngan cho tung slide.
- 3-5 cau hoi Q&A da chuan bi.

---

## 20. Cach lam bai noi muot hon

Dung bat dau bang cong thuc hoac bang ket qua. Hay bat dau bang van de:

```text
LLM co the tra loi rat hay, nhung lam sao biet no dang noi dung?
```

Sau do dan vao RAG:

```text
Mot cach pho bien la cho mo hinh doc tai lieu truoc khi tra loi.
```

Roi dan vao Self-RAG:

```text
Nhung neu tai lieu sai thi sao?
Neu cau hoi khong can tai lieu thi sao?
Neu mo hinh khong biet cau tra loi cua no co duoc tai lieu ho tro khong thi sao?
```

Cach dan nay giup seminar co cau chuyen ro hon va de nghe hon.

---

# PHAN B. LUONG PROJECT CUOI KI

## 21. Muc tieu project cuoi ki

Project cuoi ki nen bien y tuong trong bai bao thanh mot he thong thuc nghiem nho, de chung minh ban hieu Self-RAG khong chi ve ly thuyet ma ca ve trien khai.

Muc tieu phu hop:

> Xay dung mot prototype RAG co co che self-reflection don gian, trong do he thong co the truy xuat tai lieu, sinh cau tra loi, va tu danh gia relevance/support/usefulness cua cau tra loi.

Project khong nhat thiet phai fine-tune lai mot LLM nhu bai bao goc. Voi dieu kien mon hoc, co the lam phien ban **Self-RAG-inspired**:

- Dung retriever co san.
- Dung LLM/API/local model de generate.
- Dung prompt hoac scoring rules de tao reflection signals.
- Danh gia ket qua so voi RAG baseline.

---

## 22. Cau hoi nghien cuu cho project

Nen chon 1-2 cau hoi ro rang:

1. Them self-reflection vao RAG co giup cau tra loi factual hon khong?
2. Adaptive retrieval co giup giam retrieval khong can thiet khong?
3. Reflection-based filtering co giup loai tai lieu nhieu khong?
4. Self-critique co giup phat hien cau tra loi khong duoc evidence ho tro khong?

Phien ban gon nhat:

> So sanh RAG truyen thong va Self-RAG-inspired RAG tren cung tap cau hoi, dua tren answer correctness, evidence support va usefulness.

---

## 23. Pham vi project de kha thi

Nen gioi han project trong mot mien cu the:

- QA tren noi dung bai bao Self-RAG.
- QA tren mot tap bai bao khoa hoc NLP.
- QA tren tai lieu mon hoc.
- QA tren document tieng Viet.
- QA tren knowledge base tu file PDF/Markdown.

Voi project hien tai, goi y tot nhat:

> Xay dung he thong hoi dap dua tren bai bao Self-RAG va/hoac tap tai lieu seminar, co baseline RAG va phien ban Self-RAG-inspired.

Ly do:

- Du lieu co san trong project.
- Chu de khop seminar.
- De demo.
- De viet bao cao.

---

## 24. Kien truc project goi y

### 24.1. Baseline RAG

```text
User question
  -> Embed question
  -> Retrieve top-k chunks
  -> Prompt LLM with question + chunks
  -> Generate answer
```

### 24.2. Self-RAG-inspired system

```text
User question
  -> Decide retrieval needed?
  -> If yes: retrieve top-k chunks
  -> Judge relevance of each chunk
  -> Keep relevant chunks
  -> Generate answer
  -> Judge support
  -> Judge usefulness
  -> Return answer + reflection report
```

---

## 25. Cac module nen co trong project

### 25.1. Document ingestion

Chuc nang:

- Doc PDF/Markdown/TXT.
- Tach van ban thanh chunks.
- Luu metadata: ten file, trang, chi so chunk.

Output:

```text
[
  {
    "chunk_id": "paper_001",
    "text": "...",
    "source": "2310.11511v1.pdf",
    "page": 1
  }
]
```

### 25.2. Embedding va vector store

Chuc nang:

- Tao embedding cho moi chunk.
- Luu vao vector database hoac index local.

Lua chon don gian:

- FAISS.
- Chroma.
- scikit-learn TF-IDF neu khong dung embedding.

### 25.3. Retriever

Input:

```text
Question
```

Output:

```text
Top-k relevant chunks
```

Nen ho tro:

- `top_k = 3`
- `top_k = 5`
- Hien thi source cua moi chunk.

### 25.4. Baseline generator

Prompt co ban:

```text
You are an assistant answering based on the provided context.
Use only the context when possible.

Context:
{retrieved_chunks}

Question:
{question}

Answer:
```

### 25.5. Retrieval decision module

Tuong ung voi token `[Retrieve]`.

Muc tieu:

- Quyet dinh cau hoi co can retrieve khong.

Cach lam don gian:

Prompt LLM:

```text
Given the question, decide whether external documents are needed.
Return only one label: RETRIEVE or NO_RETRIEVE.

Question: {question}
```

Vi du:

```text
Question: What is 2 + 2?
Decision: NO_RETRIEVE
```

```text
Question: What are the reflection tokens in Self-RAG?
Decision: RETRIEVE
```

### 25.6. Relevance judge

Tuong ung voi `[ISREL]`.

Prompt:

```text
Question:
{question}

Passage:
{chunk}

Is the passage relevant to answering the question?
Return one label: RELEVANT or IRRELEVANT.
Also provide a short reason.
```

Output:

```text
{
  "label": "RELEVANT",
  "reason": "The passage explains reflection tokens used by Self-RAG."
}
```

### 25.7. Support judge

Tuong ung voi `[ISSUP]`.

Prompt:

```text
Question:
{question}

Answer:
{answer}

Evidence:
{selected_chunks}

Is the answer supported by the evidence?
Return one label: FULLY_SUPPORTED, PARTIALLY_SUPPORTED, or NOT_SUPPORTED.
Also provide a short reason.
```

### 25.8. Usefulness judge

Tuong ung voi `[ISUSE]`.

Prompt:

```text
Question:
{question}

Answer:
{answer}

Is the answer useful and complete for the user?
Return one label: USEFUL, PARTIALLY_USEFUL, or NOT_USEFUL.
Also provide a short reason.
```

### 25.9. Final response formatter

Nen tra ve:

- Final answer.
- Retrieved sources.
- Reflection report.

Vi du:

```text
Answer:
Self-RAG proposes a framework where the model learns to retrieve, generate,
and critique its own output through reflection tokens.

Sources:
- paper/2310.11511v1.pdf, page 1
- paper/2310.11511v1.pdf, page 3

Reflection:
- Retrieve: Yes
- Relevant passages: 2/3
- Support: Fully supported
- Usefulness: Useful
```

---

## 26. Demo project nen co gi

Demo nen rat ro rang, khong nen qua phuc tap.

### 26.1. Man hinh/CLI demo toi thieu

Input:

```text
What is the main idea of Self-RAG?
```

Output:

```text
Retrieve: Yes
Relevant passages: 3/5
Answer: ...
Support: Fully supported
Usefulness: Useful
Sources: ...
```

### 26.2. Cau hoi demo nen chuan bi

Nhom cau hoi can retrieve:

```text
What are the reflection tokens used in Self-RAG?
How does Self-RAG differ from standard RAG?
What is the role of the critic model in Self-RAG?
What tasks were used to evaluate Self-RAG?
```

Nhom cau hoi khong can retrieve:

```text
What is 2 + 2?
Translate "retrieval" into Vietnamese.
What does NLP stand for?
```

Nhom cau hoi de kiem tra hallucination:

```text
Did Self-RAG completely eliminate hallucination?
Does Self-RAG always retrieve documents for every question?
```

---

## 27. Thuc nghiem va danh gia

### 27.1. Baselines

Nen co it nhat 2 he:

1. **No-RAG LLM**
   - Chi hoi LLM, khong retrieval.

2. **Standard RAG**
   - Retrieve top-k chunks.
   - Generate answer.

3. **Self-RAG-inspired**
   - Decide retrieval.
   - Filter relevant chunks.
   - Generate answer.
   - Judge support/usefulness.

### 27.2. Tap cau hoi danh gia

Nen tao 20-50 cau hoi.

Chia thanh:

- Factual questions.
- Conceptual questions.
- Comparison questions.
- Questions not requiring retrieval.
- Trick/unsupported questions.

Vi du:

```text
1. What problem does Self-RAG address?
2. What are reflection tokens?
3. What does ISREL evaluate?
4. Why is adaptive retrieval useful?
5. Does Self-RAG require retrieval for every input?
6. What is the difference between critic and generator?
7. What are the limitations of Self-RAG?
8. Is Self-RAG guaranteed to remove hallucination?
```

### 27.3. Metrics

Co the dung metric dinh tinh va dinh luong nho:

- Answer correctness.
- Evidence support.
- Usefulness.
- Retrieval precision.
- Number of retrieved chunks used.
- Hallucination rate.

Bang danh gia thu cong:

| Question | No-RAG Correct | RAG Correct | Self-RAG Correct | Support | Notes |
|---|---:|---:|---:|---|---|
| Q1 | 0 | 1 | 1 | Fully | Self-RAG cites right passage |
| Q2 | 0 | 1 | 1 | Fully | Better explanation |
| Q3 | 1 | 1 | 1 | Not needed | Retrieval skipped |

### 27.4. Ky vong ket qua

Ky vong hop ly:

- Self-RAG-inspired tot hon No-RAG o cau hoi can knowledge.
- Self-RAG-inspired co the loc bot chunk khong lien quan so voi RAG.
- Self-RAG-inspired co reflection report de giai thich cau tra loi.
- Khong nhat thiet luon cao hon RAG ve moi metric.

Quan trong:

> Neu Self-RAG-inspired khong luon tot hon baseline, van co the phan tich ly do. Do la mot phan tot cua bao cao khoa hoc.

---

## 28. Bao cao project cuoi ki

Cau truc bao cao goi y:

1. Introduction
   - Gioi thieu LLM, hallucination, RAG.
   - Ly do chon Self-RAG.

2. Related Work
   - LLM.
   - RAG.
   - Self-RAG.

3. Methodology
   - Mo ta baseline RAG.
   - Mo ta Self-RAG-inspired pipeline.
   - Mo ta reflection modules.

4. Implementation
   - Du lieu.
   - Chunking.
   - Retriever.
   - Generator.
   - Reflection prompts.
   - System interface.

5. Experiments
   - Cau hoi danh gia.
   - Baselines.
   - Metrics.

6. Results
   - Bang ket qua.
   - Vi du cau tra loi.
   - Phan tich case dung/sai.

7. Discussion
   - Vi sao co cai thien.
   - Khi nao he thong that bai.
   - Chi phi va han che.

8. Conclusion
   - Tong ket bai hoc.
   - Huong phat trien.

---

## 29. Ke hoach thuc hien project theo tuan

### Tuan 1: Doc bai bao va chot scope

- Doc ki abstract, method, experiment.
- Chot cau hoi nghien cuu.
- Chot du lieu dung cho project.
- Chot project la CLI, notebook hay web demo.

Deliverable:

- 1 trang problem statement.
- Danh sach module.

### Tuan 2: Lam baseline RAG

- Doc/tach tai lieu.
- Tao chunks.
- Lam retriever.
- Lam prompt generator.
- Test voi 10 cau hoi.

Deliverable:

- Baseline RAG chay duoc.

### Tuan 3: Them reflection modules

- Retrieval decision.
- Relevance judge.
- Support judge.
- Usefulness judge.
- Format output co reflection report.

Deliverable:

- Self-RAG-inspired pipeline chay duoc.

### Tuan 4: Danh gia va viet bao cao

- Tao tap 20-50 cau hoi.
- Chay No-RAG, RAG, Self-RAG-inspired.
- Lap bang ket qua.
- Viet phan analysis.

Deliverable:

- Bao cao ket qua.
- Demo script.

### Tuan 5: Hoan thien seminar/project

- Chinh slide.
- Chuan bi demo.
- Chuan bi Q&A.
- Tap thuyet trinh.

Deliverable:

- Slide cuoi.
- Bao cao cuoi.
- Demo cuoi.

---

## 30. Phan chia cong viec neu lam nhom

Neu project lam theo nhom 3-4 nguoi:

### Thanh vien 1: Paper va seminar

- Doc bai bao.
- Lam slide method.
- Chuan bi script.
- Giai thich reflection tokens.

### Thanh vien 2: Data va retriever

- Xu ly PDF/document.
- Chunking.
- Embedding.
- Vector store.
- Retrieval evaluation.

### Thanh vien 3: Generator va reflection

- Prompt generation.
- Retrieval decision.
- Relevance/support/usefulness judge.
- Output formatting.

### Thanh vien 4: Experiment va report

- Tao tap cau hoi.
- Chay baseline.
- Lap bang ket qua.
- Viet analysis va limitation.

---

## 31. Risk va cach xu ly

### 31.1. Khong extract duoc PDF tot

Cach xu ly:

- Dung text tu abstract/method duoc copy thu cong.
- Dung file Markdown tom tat bai bao lam knowledge base.
- Dung mot tap tai lieu nho hon.

### 31.2. Retriever tra ve chunk nhieu

Cach xu ly:

- Giam kich thuoc chunk.
- Them overlap.
- Dung top-k nho hon.
- Dung relevance judge de loc.

### 31.3. LLM judge khong on dinh

Cach xu ly:

- Bat output theo label co dinh.
- Yeu cau tra ve JSON.
- Dat temperature thap.
- Them vi du trong prompt.

### 31.4. Self-RAG-inspired cham hon RAG

Cach xu ly:

- Giai thich day la trade-off giua chat luong va chi phi.
- Cache ket qua retrieval.
- Chi judge top-k nho.

---

## 32. Demo script project

Demo nen di theo flow:

1. Gioi thieu van de.
2. Chay RAG baseline voi mot cau hoi.
3. Chay Self-RAG-inspired voi cung cau hoi.
4. So sanh output.
5. Chay cau hoi khong can retrieve.
6. Chay cau hoi trick/unsupported.
7. Ket luan.

Vi du script noi:

```text
Dau tien, chung ta hoi he thong: "What are the reflection tokens in Self-RAG?"
Voi RAG baseline, he thong retrieve top-k chunks va sinh cau tra loi truc tiep.
Voi Self-RAG-inspired, he thong dau tien quyet dinh co can retrieve khong.
Sau do no danh gia tung passage bang relevance judge.
Cuoi cung, sau khi sinh answer, no tu danh gia answer co duoc evidence ho tro khong va co huu ich khong.
```

---

## 33. Ket noi seminar va project

Seminar va project nen ho tro lan nhau.

### Trong seminar

Ban trinh bay bai bao goc:

- Dong co.
- Phuong phap.
- Reflection tokens.
- Training/inference.
- Ket qua.
- Han che.

### Trong project

Ban trien khai phien ban thu nho:

- Khong fine-tune model lon.
- Dung prompt de mo phong reflection tokens.
- So sanh voi RAG baseline.
- Danh gia tren tap cau hoi nho.

Thong diep lien ket:

> Seminar giai thich Self-RAG ve mat khoa hoc; project chung minh y tuong Self-RAG co the duoc hien thuc hoa trong mot prototype RAG co self-reflection.

---

## 34. Ket luan chung

Self-RAG la mot chu de tot cho seminar va project vi no ket hop duoc nhieu noi dung quan trong cua NLP hien dai:

- LLM.
- Retrieval.
- Question Answering.
- Text Generation.
- Factuality.
- Evaluation.
- Self-critique.

Neu trinh bay tot, bai seminar se co mach rat ro:

```text
LLM hallucination
  -> RAG as a solution
  -> Limitations of standard RAG
  -> Self-RAG with reflection tokens
  -> Better factuality and controllability
```

Neu lam project tot, san pham cuoi ki co the chung minh:

```text
RAG khong chi can retrieve.
RAG can biet khi nao retrieve, dung evidence nao, va cau tra loi co dang tin hay khong.
```

