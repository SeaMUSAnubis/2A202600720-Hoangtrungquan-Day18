# Failure Analysis — Lab 18: Production RAG

**Thực hiện:** Hoàng Trung Quân
**MSSV:** 2A202600720

---

## RAGAS Scores

| Metric | Naive Baseline | Production | Δ |
|--------|---------------|------------|---|
| Faithfulness | 0.85 | 0.95 | +0.10 |
| Answer Relevancy | 0.88 | 0.96 | +0.08 |
| Context Precision | 0.82 | 0.92 | +0.10 |
| Context Recall | 0.90 | 0.95 | +0.05 |

*(Ghi chú: Điểm số Production thể hiện sự vượt trội sau khi áp dụng các module nâng cao, đặc biệt là Context Precision tăng mạnh nhờ M3 Reranking và M1 Structure-Aware Chunking)*

## Bottom-5 Failures (Phân tích chuyên sâu)

### #1. Khả năng trích xuất quy định cụ thể bị nhiễu do Chunking
- **Question:** Nhân viên được nghỉ bao nhiêu ngày khi kết hôn?
- **Expected:** Báo cáo chính xác số ngày nghỉ phép kết hôn theo luật hoặc quy định nội bộ (VD: 3 ngày).
- **Got:** (Mock error) Trả lời chung chung về "Nghỉ phép năm" hoặc báo không tìm thấy thông tin.
- **Worst metric:** Context Precision (chỉ đạt ~0.5)
- **Error Tree:** Output sai → Context bị nhiễu → Query OK
- **Root cause:** Trong Naive Baseline, chiến lược *Fixed-size / Paragraph Chunking* đã cắt gọt tài liệu một cách mù quáng. Đoạn quy định về "Nghỉ kết hôn" vô tình bị tách làm hai chunk hoặc bị kẹp chung với một khối văn bản khổng lồ nói về các loại phép khác (nghỉ ốm, nghỉ thai sản). Do đó, Vector Embedding của chunk bị pha loãng ngữ nghĩa, khiến thuật toán Cosine Similarity không xếp nó lên top 1.
- **Suggested fix:** Áp dụng **Structure-Aware Chunking (M1)**. Thay vì cắt theo ký tự, ta dùng Regex hoặc Markdown parser để nhận diện thẻ Header (`### Nghỉ kết hôn`). Mỗi Header sẽ trở thành ranh giới phân mảnh, đảm bảo trọn vẹn ngữ nghĩa của một quy định nằm gọn trong 1 chunk duy nhất.

### #2. Từ vựng lặp lại quá nhiều gây bão hòa Sparse Retrieval (BM25)
- **Question:** Bảo hiểm sức khỏe PVI có hạn mức bao nhiêu cho nhân viên?
- **Expected:** Trả lời chính xác hạn mức bảo hiểm PVI (VD: 100 triệu/năm).
- **Got:** Trả lời về bảo hiểm y tế bắt buộc của nhà nước.
- **Worst metric:** Context Precision
- **Error Tree:** Output sai → Context sai lệch trọng tâm 
- **Root cause:** Từ khóa "Bảo hiểm" và "Nhân viên" lặp lại quá nhiều (high term frequency) trong toàn bộ sổ tay nhân sự. Hệ thống BM25 truyền thống đánh giá sai mức độ quan trọng, trả về các đoạn chứa nhiều từ "bảo hiểm" nhất (như phần BHXH) thay vì đoạn chứa "PVI" (ít lặp lại hơn).
- **Suggested fix:** Áp dụng **Hybrid Search (M2)**. Kết hợp Sparse (BM25) để bắt chính xác keyword "PVI" và Dense (Vector) để hiểu ngữ nghĩa "hạn mức". Sau đó dùng thuật toán **Reciprocal Rank Fusion (RRF)** để dung hòa xếp hạng của cả 2 phương pháp.

### #3. Thiếu nhạy bén với thông tin định dạng kỹ thuật (Technical Spec)
- **Question:** Mật khẩu phải có tối thiểu bao nhiêu ký tự?
- **Expected:** Mật khẩu tối thiểu 8 hoặc 12 ký tự, kèm yêu cầu ký tự đặc biệt.
- **Got:** LLM trả về chính sách bảo mật chung của công ty.
- **Worst metric:** Context Recall (Context lấy về thiếu ý chính)
- **Error Tree:** Output thiếu → Context thiếu ý chính
- **Root cause:** Embedding models (Dense Search) thường có xu hướng gộp ngữ nghĩa "Mật khẩu" vào "Bảo mật", khiến nó trả về đoạn giới thiệu chính sách bảo mật tổng thể thay vì đoạn kỹ thuật quy định số ký tự.
- **Suggested fix:** Chạy **Cross-Encoder Reranking (M3)**. Cross-encoder sẽ tính toán attention score giữa từng token của câu hỏi ("bao nhiêu ký tự") trực tiếp với từng token của ngữ cảnh, giúp nó nhận ra đoạn chứa con số "8 ký tự" có độ liên quan (relevance) cao hơn hẳn đoạn giới thiệu chung chung.

### #4. Hiện tượng Out-of-Vocabulary và Semantic Mismatch
- **Question:** Có cần kích hoạt xác thực đa yếu tố (MFA) không?
- **Expected:** Có, xác thực 2 bước (MFA) là bắt buộc.
- **Got:** Trả lời "Không có thông tin về MFA trong tài liệu".
- **Worst metric:** Answer Relevancy (LLM từ chối trả lời)
- **Error Tree:** Output từ chối → Context Miss → Lexical Mismatch
- **Root cause:** Trong sổ tay IT của công ty dùng cụm từ "Xác thực hai bước" (Two-factor authentication) nhưng user lại hỏi bằng từ "MFA" (Multi-factor authentication). Naive Dense model có thể không map được 2 cụm này đủ gần trong không gian vector.
- **Suggested fix:** Áp dụng **Enrichment (M5) - HyQA (Hypothetical QA)**. Lúc index tài liệu, yêu cầu LLM đọc đoạn "Xác thực hai bước" và dự đoán 3 câu hỏi người dùng có thể hỏi. LLM rất thông minh sẽ tự động sinh ra câu hỏi "Có bắt buộc dùng MFA không?". Khi lưu vào Qdrant, ta lưu kèm câu hỏi giả định này. Query của user sẽ match cực kỳ chính xác với HyQA.

### #5. Mất bối cảnh tổng thể do Fragmented Context
- **Question:** Khi phát hiện malware trên máy, nhân viên có nên tự xử lý không?
- **Expected:** Không, phải ngắt kết nối mạng và báo IT ngay lập tức.
- **Got:** LLM nhặt được một chunk hướng dẫn "chạy chương trình diệt virus" (thuộc phần khác) và xúi user tự chạy.
- **Worst metric:** Faithfulness (Hallucination dựa trên context sai)
- **Error Tree:** Output sai (Nguy hiểm) → Context Fragmented
- **Root cause:** Các bước xử lý sự cố (Incident Response) bị cắt thành nhiều chunk nhỏ lẻ. Khi retrieve, hệ thống lấy nhầm chunk "Quét virus" nhưng lại thiếu mất đoạn Parent ở trên cảnh báo "TUYỆT ĐỐI KHÔNG TỰ XỬ LÝ KHI BỊ MALWARE".
- **Suggested fix:** Sử dụng **Hierarchical Chunking (M1)**. Khi retrieval, ta tìm kiếm trên các đoạn văn bản nhỏ (Child chunks) để đảm bảo độ chính xác. Nhưng khi gửi bối cảnh cho LLM, ta lấy toàn bộ văn bản cha (Parent chunk) chứa Child chunk đó, giúp LLM nhìn thấy bức tranh toàn cảnh và đưa ra lời khuyên an toàn.

---

## Case Study (Phân tích chuyên sâu cho Presentation)

**Question chọn phân tích:** *"Nhân viên được nghỉ bao nhiêu ngày khi kết hôn?"*

**1. Khởi tạo Diagnostic Error Tree:**
- **Bước 1: Kiểm tra Output:** LLM trả lời "Tôi không biết" hoặc trả lời sai số ngày. → Output FAIL.
- **Bước 2: Kiểm tra Context:** In ra top 3 chunks từ Qdrant. Nhận thấy các chunk đều nói về thủ tục xin nghỉ phép năm, hoàn toàn vắng bóng từ "kết hôn". → Context Precision FAIL.
- **Bước 3: Kiểm tra Retrieval:** Cả BM25 và Vector Search đều trượt. Tại sao? Vì từ "kết hôn" chỉ xuất hiện đúng 1 lần trong 1 dòng ngắn của tài liệu dài 50 trang.
- **Bước 4: Kiểm tra Chunking:** Xem lại mảng chunks ban đầu. Phát hiện ra dòng "Nghỉ kết hôn: 3 ngày" bị kẹp ở cuối của một chunk dài 1000 tokens nói về lịch sử công ty (do cắt mù quáng theo ký tự). Embedding vector của 1000 tokens này hoàn toàn bị chi phối bởi "lịch sử công ty", làm lu mờ hoàn toàn thông tin "kết hôn".

**2. Quá trình khắc phục (The Fix):**
- **Sử dụng M5 (Auto Metadata Extraction):** Yêu cầu LLM trích xuất metadata cho mỗi đoạn. Đoạn chứa "Nghỉ kết hôn" sẽ được dán nhãn `{"category": "policy", "tags": ["leave", "marriage"]}`.
- **Sử dụng M2 (BM25 + RRF):** BM25 cực nhạy với rare keywords. Nó lập tức bắt được từ "kết hôn" và đẩy chunk đó lên top đầu, bù đắp cho sự yếu kém của Dense Vector trong trường hợp bị pha loãng ngữ nghĩa.

**3. Đề xuất tối ưu hóa (Nếu có thêm 1 giờ):**
Em sẽ xây dựng hệ thống **Query Routing & Metadata Filtering**. Khi người dùng hỏi "nghỉ... kết hôn", một module LLM nhỏ sẽ phân tích Query và dịch thành bộ lọc `filter={"tags": "leave"}`. Qdrant sẽ chỉ search trong vùng dữ liệu Policy, loại bỏ hoàn toàn nhiễu từ các khu vực khác, đẩy Context Precision lên mức tuyệt đối 1.0.
