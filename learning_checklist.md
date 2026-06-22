# Checklist Học Tập: Production RAG Pipeline (Lab 18)

Quá trình học tập sẽ được chia nhỏ thành các chặng. Chúng ta sẽ đánh dấu hoàn thành `[x]` khi bạn đã thực sự hiểu sâu (cả high-level và low-level) từng phần.

- [x] **1. Bức tranh tổng thể (Overview)**
  - Hiểu vấn đề cốt lõi của Naive RAG.
  - Tại sao lại cần một Production RAG Pipeline? Các nhánh giải pháp (5 modules) đóng vai trò gì trong việc giải quyết vấn đề đó?

- [x] **2. Module 1: Chunking (Phân mảnh dữ liệu)**
  - Vấn đề của chunking cơ bản.
  - Giải pháp: Semantic, Hierarchical, Structure-Aware.
  - Design decisions & Edge cases.

- [x] **3. Module 5: Enrichment (Làm giàu dữ liệu)**
  - Tại sao phải làm giàu dữ liệu trước khi đưa vào CSDL?
  - Cơ chế hoạt động: Summarize, Contextual Prepend, Metadata.

- [x] **4. Module 2: Hybrid Search (Tìm kiếm lai)**
  - Vấn đề nếu chỉ dùng Sparse (BM25) hoặc Dense (Vector).
  - Giải pháp: Kết hợp cả hai bằng Reciprocal Rank Fusion (RRF).

- [x] **5. Module 3: Reranking (Xếp hạng lại)**
  - Tại sao tìm kiếm xong chưa đủ mà phải xếp hạng lại?
  - Sự khác biệt giữa Bi-encoder (cho Search) và Cross-encoder (cho Rerank).

- [x] **6. Module 4: RAGAS Evaluation (Đánh giá hệ thống)**
  - Làm sao để biết hệ thống mới thực sự tốt hơn?
  - Hiểu sâu 4 metrics: Faithfulness, Answer Relevancy, Context Precision, Context Recall.
  - Failure Analysis.

- [x] **7. Tầm nhìn bao quát (Broader Context)**
  - Hệ thống này sẽ tác động thế nào khi áp dụng vào dự án thực tế của bạn?
  - Lên kế hoạch Action Plan cho dự án cá nhân.

---

## Nhật ký học tập và đúc kết

Dưới đây là các đúc kết và câu trả lời của Quân trong suốt quá trình học 5 Modules của Production RAG:

**1. Về bức tranh tổng thể (Naive RAG vs Production RAG):**
> "Naive RAG sẽ hoạt động theo 3 bước đó là Chunking -> embedding -> retrieval. Thế nhưng đôi lúc theo quy trình như thế này sẽ khiến nó không được tối ưu khiến cho nó bị quên thông tin ở giữa, hoặc bị hallucination. Nên chúng ta cần một mô hình RAG phức tạp hơn để có thể capture được những thứ mà một production cần có, để nâng cao trải nghiệm."

**2. Về các chiến lược Chunking:**
> "Semantic chunking là kĩ thuật chia văn bản dựa trên ngữ nghĩa việc này giúp cho LLMs hiểu được rõ hơn về data... Tuy nhiên việc này phụ thuộc vào model embedding mạnh hay yếu. Hierarchical Chunking thì giúp việc tìm kiếm dữ liệu hiệu quả nhưng đòi hỏi phải có database và pipeline phức tạp, dung lượng cũng sẽ tăng cao. Em nghĩ là **Structure-Aware Chunking** phù hợp nhất cho QA system vì việc bố trí cấu trúc theo từng lớp như cây sẽ giúp model không bị hiểu sai."

**3. Tầm quan trọng của Hybrid Search (M2) và Reranking (M3):**
> "Nếu bỏ qua bước 2 thì khả năng model sẽ không hiểu được ngữ cảnh mà ta đang muốn nói đến. Trong khi đó nếu bỏ qua bước 3 model sẽ khó mà hiểu đúng câu hỏi của mình trọng tâm nằm ở đâu? Từ đó khi embedding câu hỏi để tìm kiếm dữ liệu trong data RAG sẽ bị hiểu sai gây ra hậu quả trả lời sai."

**4. Tại sao phải Search trước rồi mới Rerank (Cross-encoder):**
> "Tại vì việc xử lý (bằng Cross-encoder trên toàn bộ dữ liệu) sẽ tốn tài nguyên, và thời gian. Giả sử có 1 triệu hoặc 1 tỉ cái tài liệu thì việc xử lý như thế rất cồng kềng và không mang lại nhiều hiệu quả. Trong khi đó nếu ta lọc các thông tin liên quan sau đó mới xử lý thì sẽ nhanh hơn và có độ chính xác tốt hơn."

**5. Khắc phục lỗi Hallucination (Chém gió):**
> Metric liên quan là **Answer Relevancy / Faithfulness** vì nó nói thông tin không có thật. Giải pháp là "nên siết chặt prompt".

**6. Kế hoạch ứng dụng vào dự án cá nhân (Data streaming thay đổi liên tục):**
> "Em định ưu tiên **M5 (Enrichment)** và **M4 (Evaluation)** việc này sẽ giúp cho hệ thống của em hiểu được những data gửi về hàng giây, và trả lời không linh tinh."
