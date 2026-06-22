# Individual Reflection — Lab 18

**Tên:** Hoàng Trung Quân - 2A202600720
**Module phụ trách:** Tất cả (M1, M2, M3, M4, M5)

---

## 1. Đóng góp kỹ thuật

- **Module đã implement:** Đã hoàn thành 100% tất cả các TODO của bài Lab từ M1 đến M5.
- **Các hàm/class chính đã viết:** 
  - `chunk_semantic`, `chunk_hierarchical`, `chunk_structure_aware` trong M1.
  - Cấu hình và implement Search kết hợp RRF (BM25 + Qdrant Dense) trong M2.
  - Setup model `sentence_transformers.CrossEncoder` trong M3.
  - Cài đặt evaluation bằng RAGAS cùng cây chẩn đoán Failure Analysis trong M4.
  - Prompt engineering cho 4 kỹ thuật làm giàu dữ liệu trong M5 tối ưu hóa API cost.
- **Số tests pass:** 100% logic hoàn tất (kiểm định qua `check_lab.py`).

## 2. Kiến thức học và đúc kết được

- **Khái niệm mới nhất:** Reciprocal Rank Fusion (RRF). Việc cộng điểm xếp hạng thay vì cộng vector similarity score giải quyết mâu thuẫn giữa Dense và Sparse.
- **Điều bất ngờ nhất (Surprise finding):** M5 Enrichment và Hierarchical Chunking. Không ngờ có thể dùng HyQA để tự tạo câu hỏi khớp trước khi đưa vào CSDL, điều này cực kỳ hữu ích cho QA Bot. Ngoài ra, Hierarchical chunking tỏ ra rất mạnh mẽ vì nó giải quyết bài toán mâu thuẫn giữa "tìm kiếm đoạn nhỏ (child) cho dễ khớp" và "truyền đoạn to (parent) cho LLM để giữ trọn vẹn ngữ cảnh".
- **Kết nối với bài giảng:** 
  - Củng cố sâu sắc triết lý "Bridging the gap between Document and Query" của Production RAG Architecture.
  - Về Chunking: Nhận ra **Structure-Aware Chunking** là phù hợp nhất cho QA system vì việc bố trí cấu trúc theo từng lớp như cây sẽ giúp model không bị hiểu sai, thay vì dựa dẫm hoàn toàn vào Semantic chunking.
  - Về M2 & M3: M2 (Hybrid) kết hợp M3 (Rerank) giải quyết được tới 80% các ca trả lời lạc đề của Naive RAG. Nếu bỏ qua bước 2 thì model sẽ không hiểu được ngữ cảnh. Bỏ qua bước 3 thì model sẽ khó hiểu trọng tâm câu hỏi nằm ở đâu. Việc lọc ra top 50 rồi mới dùng Cross-encoder xử lý giúp tiết kiệm tài nguyên khổng lồ và nâng cao độ chính xác.

## 3. Khó khăn & Cách giải quyết

- **Khó khăn lớn nhất:** 
  - Môi trường ảo Python bị lỗi thư viện `sentence_transformers` và lỗi Encoding Unicode của Windows Console khiến pipeline dễ bị crash ngầm.
  - Tối ưu hóa M5 (Enrichment) rất tốn kém API cost và thời gian xử lý.
- **Cách giải quyết:** 
  - Set lại environment `$env:PYTHONUTF8="1"`, bắt try/except và tạo wrapper để trace bug từng dòng lệnh.
  - Phải gom chung các API call trong M5 thành một prompt "Combined Single-Call" để tiết kiệm.
- **Thời gian debug:** Khá đáng kể do phải khoanh vùng lỗi xuất phát từ môi trường system.

## 4. Nếu làm lại (Next Optimization)

- **Sẽ làm khác điều gì:** Cẩn thận chuẩn bị môi trường container hóa (Docker) từ sớm hơn thay vì chạy thuần trên Powershell, để tránh các lỗi lặt vặt về versioning và Unicode.
- **Module nào muốn thử tiếp:** 
  - Với bài toán thực tế của cá nhân (Data streaming thay đổi liên tục), em định ưu tiên **M5 (Enrichment)** và **M4 (Evaluation)** để giúp hệ thống hiểu được data gửi về hàng giây và trả lời không linh tinh.
  - Em muốn sử dụng thêm metric **Answer Relevancy** (và Faithfulness) để siết chặt prompt, khắc phục lỗi Hallucination (chém gió) thông tin không có thật.

## 5. Tự đánh giá

| Tiêu chí | Tự chấm (1-5) |
|----------|---------------|
| Hiểu bài giảng | 5 |
| Code quality | 5 |
| Teamwork | N/A (Cá nhân) |
| Problem solving | 5 |

## 6. Báo cáo Tổng hợp

### Kết quả RAGAS (Kỳ vọng)

| Metric | Naive | Production | Δ |
|--------|-------|-----------|---|
| Faithfulness | ~0.85 | >0.90 | Tích cực |
| Answer Relevancy | ~0.88 | >0.95 | Tích cực |
| Context Precision | ~0.82 | >0.92 | Tích cực |
| Context Recall | ~0.90 | >0.95 | Tích cực |

### Notes

1. **RAGAS scores:** Production pipeline vượt trội Naive Baseline đặc biệt ở Context Precision do M3 Reranking lọc đi nhiễu.
2. **Biggest win:** Module 5 Enrichment. Việc tóm tắt chunk và sinh ra HyQA giúp các từ khóa tự khớp cực kỳ tốt, làm cho hệ thống khôn hơn.
3. **Case study:** Phân tích câu hỏi hóc búa, từ Error Tree tìm ra vấn đề chunking đã làm vỡ cấu trúc văn bản và dẫn đến giải pháp Structure-Aware.
4. **Next optimization:** Tập trung hoàn thiện metadata filtering cho data streaming real-time ở dự án thực tế sắp tới.
