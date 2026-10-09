# Báo cáo Phân tích Đánh giá A/B Testing & RAGAS Evaluation

**Học viên:** Nguyễn Thị Thúy Hiền  
**MSSV:** 2A202602896  
**Lớp / Khóa:** K4-L3 Track 2 - Day 22: LangSmith + Prompt Versioning  
**LangSmith Project:** `day22-lab`  

---

## 1. Mục tiêu Thử nghiệm

Đánh giá tác động của kỹ thuật thiết kế Prompt (Prompt Engineering) đến hiệu năng của hệ thống Retrieval-Augmented Generation (RAG) thông qua framework RAGAS (Retrieval Augmented Generation Assessment) và giám sát tập trung trên LangSmith Prompt Hub.

---

## 2. Thiết kế Prompt: V1 vs V2

Hai phiên bản prompt được thiết kế với sự khác biệt rõ rệt về phong cách hướng dẫn mô hình nhưng cùng giữ nguyên tham số ngữ cảnh tài liệu `{context}`:

- **Phiên bản V1 (`nguyen-thi-thuy-hien-rag-prompt-v1`) - Phong cách Ngắn gọn (Concise & Direct):**
  - **System Prompt:**
    ```text
    Bạn là trợ lý AI thân thiện. Trả lời ngắn gọn (2-4 câu), chỉ dựa trên context được cung cấp. Nếu không có thông tin, hãy nói thẳng là không biết, không suy đoán.

    Context:
    {context}
    ```
  - **Đặc điểm:** Hạn chế độ dài (2-4 câu), ưu tiên tính súc tích và tránh hallucination bằng cách quy định rõ việc thừa nhận thiếu thông tin.

- **Phiên bản V2 (`nguyen-thi-thuy-hien-rag-prompt-v2`) - Phong cách Chuyên gia có Cấu trúc (Structured & Analytical):**
  - **System Prompt:**
    ```text
    Bạn là chuyên gia phân tích thông tin. Đọc kỹ context, xác định các facts liên quan, rồi viết câu trả lời rõ ràng và có tổ chức (3-5 câu). Tuyệt đối không suy đoán ngoài context.

    Context:
    {context}
    ```
  - **Đặc điểm:** Yêu cầu mô hình đóng vai chuyên gia phân tích, trích xuất facts liên quan và tổ chức câu trả lời chặt chẽ, mạch lạc (3-5 câu).

---

## 3. Bảng Điểm Thực Tế & Phân tích Định lượng (RAGAS)

Thử nghiệm được thực hiện trên toàn bộ **50 cặp câu hỏi - đáp án chuẩn (`QA_PAIRS`)** qua cả hai phiên bản prompt. Kết quả định lượng chi tiết:

| Chỉ số (Metric) | Prompt V1 (Ngắn gọn) | Prompt V2 (Chuyên gia) | Chênh lệch (V1 - V2) | Đánh giá xuất sắc |
|---|:---:|:---:|:---:|:---:|
| **Faithfulness** | **0.9620** | **0.9416** | +0.0204 | Cả 2 đạt $\ge 0.90$ (Đạt điểm thưởng) |
| **Answer Relevancy** | **0.9077** | **0.8977** | +0.0100 | V1 vượt trội nhờ câu trả lời trực diện |
| **Context Recall** | **1.0000** | **1.0000** | 0.0000 | Hoàn hảo 100% tài liệu đối chiếu |
| **Context Precision**| **0.9450** | **0.9417** | +0.0033 | Retriever FAISS xếp hạng tài liệu chuẩn xác |

### Phân tích chuyên sâu: Vì sao V1 đạt điểm cao hơn V2?

1. **Về Faithfulness (0.9620 so với 0.9416):**
   - Prompt V1 yêu cầu rõ ràng: *"Trả lời ngắn gọn (2-4 câu)... Nếu không có thông tin, hãy nói thẳng là không biết, không suy đoán"*. Việc giới hạn độ dài ngắn giúp mô hình chỉ nêu đúng các mệnh đề cốt lõi nhất được trích trực tiếp từ context, giảm thiểu việc mô hình sinh thêm các từ nối hoặc câu mở rộng có thể bị RAGAS coi là thông tin ngoại vi không được trực tiếp chứng minh.
   - Prompt V2 yêu cầu *"viết câu trả lời rõ ràng và có tổ chức (3-5 câu)"*, điều này khiến câu trả lời dài hơn và diễn giải nhiều hơn. Mặc dù câu trả lời rất chi tiết và đúng đắn, việc có thêm các chi tiết diễn đạt phong phú làm tăng nhẹ xác suất xuất hiện các câu phụ ngữ cảnh.

2. **Về Answer Relevancy (0.9077 so với 0.8977):**
   - Chỉ số Answer Relevancy của RAGAS đo lường bằng cách sinh lại câu hỏi từ câu trả lời và so sánh độ tương đồng cosine với câu hỏi gốc. Câu trả lời của V1 rất ngắn gọn, đi thẳng vào trọng tâm câu hỏi người dùng mà không có phần mở đầu/kết luận rườm rà, do đó câu hỏi tái tạo tương đồng rất cao với câu hỏi ban đầu.

3. **Về Context Recall & Precision (1.0000 & ~0.94):**
   - Cả 2 phiên bản đều đạt Recall tuyệt đối 1.0 và Precision xấp xỉ 0.94, chứng minh chiến lược chunking (500 ký tự, overlap 50) và mô hình embedding `text-embedding-3-small` với retriever $k=3$ hoạt động cực kỳ hiệu quả trên knowledge base.

---

## 4. Kết luận & Khuyến nghị Triển khai Production

1. **Hiệu quả của Prompt Versioning:** Việc quản trị prompt qua LangSmith Prompt Hub giúp tách rời mã nguồn ứng dụng và logic prompt, hỗ trợ cập nhật linh hoạt không cần redeploy code.
2. **Deterministic A/B Routing:** Kỹ thuật hash MD5 trên `request_id` đảm bảo tính nhất quán (cùng người dùng/phiên luôn nhận cùng phiên bản prompt), tạo điều kiện thu thập phản hồi khách quan.
3. **Lựa chọn mô hình cho Production:** Với hệ thống hỏi đáp tra cứu tài liệu nhanh, Prompt V1 là lựa chọn tối ưu nhờ độ chính xác cao nhất (Faithfulness 0.962), độ phản hồi nhanh hơn và tiết kiệm token đầu ra.
4. **Guardrails AI:** Đảm bảo dữ liệu đầu ra an toàn tuyệt đối với PII Masking (`PIIDetector`) và tự động chuẩn hóa cú pháp JSON (`JSONFormatter`), sẵn sàng tích hợp an toàn vào luồng API production.
