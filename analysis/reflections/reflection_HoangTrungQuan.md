# Individual Reflection — Lab 18

**Tên:** Hoàng Trung Quân - 2A202600720
**Module phụ trách:** Tất cả (M1, M2, M3, M4, M5)

---

# Individual Reflection — Lab 18

**Tên:** Hoàng Trung Quân - 2A202600720
**Module phụ trách:** Tất cả (M1, M2, M3, M4, M5)

---

### Phần 1: Mapping bài giảng

| Lecture Concept | Module | Hàm cụ thể | Observation |
|----------------|--------|-------------|-------------|
| Semantic chunking | M1 | `chunk_semantic()`, `chunk_structure_aware()` | Structure-Aware chia theo header giúp tránh cắt mù quáng, khắc phục lỗi mất ngữ cảnh. |
| BM25 + Dense fusion | M2 | `reciprocal_rank_fusion()` | RRF cộng điểm rank giúp BM25 và Dense (khác hệ quy chiếu) hòa hợp, giải quyết tới 80% câu trả lời lạc đề. |
| Cross-encoder reranking | M3 | `CrossEncoderReranker.rerank()` | Reranking giúp đẩy đúng chunk có chứa từ khóa ngắn lên top 1 một cách chính xác tuyệt đối. |
| RAGAS 4 metrics | M4 | `evaluate_ragas()` | Metric Context Precision phản ánh rõ nhất độ nhiễu của chunks. Answer Relevancy giúp phát hiện Hallucination. |
| Contextual embeddings | M5 | `generate_hypothesis_questions()` | HyQA sinh trước câu hỏi giả định giúp chống lại hiện tượng Lexical Mismatch cực kỳ xuất sắc. |

### Phần 2: Khó khăn & giải quyết

- **Lỗi gặp phải:** `UnicodeEncodeError: 'charmap' codec can't encode characters in position 2-3` và `ModuleNotFoundError/Crashes` khi load `sentence_transformers` trong quá trình chạy `python main.py`.
- **Cách debug:** Mở file log unbuffered, dò tìm nguyên nhân từ console encoding trên Windows và từ thư viện PyTorch bị lỗi DLL collision.
- **Kiến thức thiếu → cách bổ sung:** Ban đầu em chưa rành việc fix conflict môi trường C++ DLL trên Windows. Em đã bổ sung bằng cách đọc document setup biến môi trường `$env:PYTHONUTF8="1"` và hiểu rõ nguyên lý chạy Virtual Environment độc lập.

### Phần 3: Action Plan cho project

## Project: Hệ thống QA Bot cho Data Streaming Real-time

### Hiện tại
- **RAG pipeline hiện tại:** Chỉ mới dùng Naive RAG (cắt chữ đều đặn) với text vector search cơ bản, rất hay trả lời chậm và chém gió.
- **Known issues:** Trả lời lạc đề nhiều, dữ liệu streaming sinh ra liên tục khiến embedding cũ bị lỗi thời. Không hiểu được các keyword chuyên ngành (Out of Vocabulary).

### Plan áp dụng
1. [x] **Chunking strategy:** Structure-aware kết hợp Hierarchical. Tại vì data stream về thường có cấu trúc file (log, json), chia theo metadata/header sẽ không bị gãy ý.
2. [x] **Search:** Hybrid Search (Qdrant Dense + BM25). Tại vì BM25 cực nhạy với keyword hiếm, kết hợp Dense để hiểu ngữ nghĩa.
3. [x] **Reranking:** Có, model `BAAI/bge-reranker-v2-m3`. Để lọc top 50 từ bước 2 xuống top 3 chính xác nhất.
4. [x] **Evaluation:** RAGAS metrics, tập trung vào Answer Relevancy để bắt lỗi chém gió.
5. [x] **Enrichment:** Sử dụng Metadata Extraction và HyQA (Hypothetical QA) để gán thẻ ngay từ lúc dòng dữ liệu mới đổ về, giúp query routing cực kỳ nhanh chóng.

### Timeline
- **Tuần 1:** Setup lại toàn bộ pipeline bằng Docker (tránh lỗi môi trường cũ). Xây dựng M1 và M5.
- **Tuần 2:** Hoàn thiện M2 và M3. Kết nối với data stream qua Kafka/RabbitMQ.
- **Tuần 3:** Tích hợp M4 RAGAS để đo đạc và tuning prompt cho HyQA. Ra mắt bản thử nghiệm.
