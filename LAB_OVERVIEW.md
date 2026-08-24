# TỔNG QUAN NỘI DUNG & Ý NGHĨA BÀI LAB DAY 22
## Xây dựng, Giám sát, Đánh giá và Bảo vệ Hệ thống RAG chuẩn Production (Production-Ready RAG)

---

## 🎯 1. Mục tiêu cốt lõi của bài Lab

Bài lab **Day 22** tập trung vào việc đưa một hệ thống **RAG (Retrieval-Augmented Generation)** từ giai đoạn thử nghiệm (PoC - Proof of Concept) sang môi trường vận hành thực tế (**Production**). 

Hệ thống được xây dựng và hoàn thiện dựa trên **4 trụ cột quan trọng nhất** trong quy trình phát triển ứng dụng GenAI chuyên nghiệp:

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                                 PRODUCTION-READY RAG                                   │
└────────────────────────────────────────────────────────────────────────────────────────┘
         │                           │                          │                     │
         ▼                           ▼                          ▼                     ▼
┌──────────────────┐       ┌──────────────────┐       ┌──────────────────┐  ┌──────────────────┐
│  Observability   │       │ Prompt Versioning│       │    Evaluation    │  │ Safety & Quality │
│   (LangSmith)    │       │   & A/B Testing  │       │     (RAGAS)      │  │ (Guardrails AI)  │
└──────────────────┘       └──────────────────┘       └──────────────────┘  └──────────────────┘
```

---

## 🏛️ 2. Chi tiết 4 Trụ Cột Triển Khai

### 1️⃣ Trụ Cột 1: Giám sát & Quan sát hệ thống (Observability with LangSmith) — *Bước 1*
- **File thực thi:** `src/01_langsmith_rag_pipeline.py`
- **Nội dung kỹ thuật:**
  - Tích hợp decorator `@traceable(name="rag-query")` vào pipeline RAG kết hợp giữa FAISS Vectorstore (107 text chunks) và LLM (OpenRouter / GPT-4o-mini).
  - Tự động ghi lại metadata cho từng truy vấn (`question_index`, `provider`, `k=3`).
- **Ý nghĩa thực tế:**
  - Cung cấp khả năng giám sát thời gian thực: thời gian phản hồi (latency), số lượng token tiêu thụ, chi phí, chi tiết input/output qua từng node (Retriever và Generator).
  - Giúp kỹ sư phát hiện và gỡ lỗi (debug) các truy vấn chậm hoặc bị lỗi ngay lập tức trên dashboard LangSmith.

---

### 2️⃣ Trụ Cột 2: Quản lý phiên bản Prompt & Thử nghiệm A/B (Prompt Hub & A/B Routing) — *Bước 2*
- **File thực thi:** `src/02_prompt_hub_ab_routing.py`
- **Nội dung kỹ thuật:**
  - Quản lý tập trung các template prompt trên **LangSmith Prompt Hub** với định danh duy nhất:
    - **Prompt V1 (`nguyen-duy-trong-rag-prompt-v1`):** Phong cách trợ lý thân thiện, trả lời ngắn gọn, trực diện (2-4 câu).
    - **Prompt V2 (`nguyen-duy-trong-rag-prompt-v2`):** Phong cách chuyên gia phân tích AI, trả lời có cấu trúc, mạch lạc (3-5 câu).
  - Triển khai cơ chế điều phối **A/B Testing Deterministic** dựa trên mã băm MD5 của `request_id` (`md5(request_id) % 100 < 50 → V1, ngược lại → V2`).
- **Ý nghĩa thực tế:**
  - Tách biệt logic câu lệnh (prompt) ra khỏi mã nguồn ứng dụng (decoupling), cho phép cập nhật hoặc rollback prompt trên production mà không cần deploy lại code.
  - Phân luồng người dùng thử nghiệm A/B một cách nhất quán (cùng một user/request ID sẽ luôn nhận cùng một phiên bản prompt).

---

### 3️⃣ Trụ Cột 3: Đánh giá chất lượng tự động (RAG Evaluation with RAGAS) — *Bước 3*
- **File thực thi:** `src/03_ragas_evaluation.py`
- **Nội dung kỹ thuật:**
  - Sử dụng framework **RAGAS** để đánh giá định lượng 50 cặp QA chuẩn (`data/eval_dataset.json`) qua cả 2 phiên bản Prompt với 4 chỉ số cốt lõi:
    1. **Faithfulness (Độ trung thực):** Đo lường xem câu trả lời có hoàn toàn dựa trên context được cung cấp hay không, chống hiện tượng "bịa đặt" (hallucination). *(Yêu cầu $\ge 0.80$)*.
    2. **Answer Relevancy (Độ liên quan):** Đo mức độ câu trả lời giải quyết đúng và trúng trọng tâm câu hỏi của người dùng.
    3. **Context Recall (Độ bao phủ ngữ cảnh):** Đo tỷ lệ thông tin trong đáp án chuẩn (ground truth reference) được bao phủ bởi context và câu trả lời.
    4. **Context Precision (Độ chính xác truy xuất):** Đánh giá mức độ liên quan và thứ tự ưu tiên của các đoạn tài liệu được retriever tìm kiếm.
- **Ý nghĩa thực tế:**
  - Giúp chuyển đổi quá trình đánh giá từ cảm tính sang **Data-Driven** (dựa trên số liệu khoa học), từ đó chứng minh được phiên bản prompt nào vượt trội hơn trước khi rollout toàn diện.

---

### 4️⃣ Trụ Cột 4: Bảo mật & Chuẩn hóa đầu ra (Safety & Output Validation with Guardrails AI) — *Bước 4*
- **File thực thi:** `src/04_guardrails_validator.py`
- **Nội dung kỹ thuật:**
  - Xây dựng lớp "lá chắn bảo vệ" (Guardrails) với 2 custom validators:
    1. **`PIIDetector`:** Tự động phát hiện và che giấu (redact) thông tin định danh cá nhân nhạy cảm (Email, Số điện thoại, Số CCCD/SSN, Thẻ tín dụng) thành các thẻ `[TYPE_REDACTED]`.
    2. **`JSONFormatter`:** Tự động kiểm tra và tự sửa lỗi (auto-repair) các cú pháp JSON lỗi thường gặp do LLM sinh ra (loại bỏ markdown fences ```` ```json ````, chuyển nháy đơn `'` thành nháy kép `"`, loại bỏ dấu phẩy thừa trước `}` hoặc `]`, fallback an toàn).
- **Ý nghĩa thực tế:**
  - Tuân thủ các quy định bảo mật dữ liệu người dùng (GDPR, bảo mật thông tin cá nhân).
  - Ngăn ngừa lỗi hệ thống (system crashes) khi các API backend tiêu thụ dữ liệu JSON trả về từ LLM.

---

## 📊 3. Bảng Tổng Hợp Kết Quả Đạt Được

| Tiêu chí | Kết quả Prompt V1 | Kết quả Prompt V2 | Đánh giá so sánh |
|:---|:---:|:---:|:---|
| **Faithfulness** | **0.9280** | 0.9114 | **V1 thắng** (Câu trả lời súc tích, trung thực cao hơn) |
| **Answer Relevancy** | 0.8855 | **0.9043** | **V2 thắng** (Phân tích sâu, giải quyết toàn diện câu hỏi) |
| **Context Recall** | 0.9756 | **1.0000** | **V2 thắng** (Đạt tuyệt đối 100% độ bao phủ thông tin chuẩn) |
| **Context Precision** | **0.9125** | 0.8889 | **V1 thắng** (Trích xuất cô đọng top ngữ cảnh liên quan) |
| **Mục tiêu Faithfulness $\ge 0.8$** | ✅ **ĐẠT** | ✅ **ĐẠT** | Cả 2 phiên bản đều vượt xa ngưỡng tối thiểu |

---

## 🌟 4. Giá trị và Ứng dụng thực tiễn

Toàn bộ các kỹ thuật trong bài Lab Day 22 chính là bộ kỹ năng tiêu chuẩn mà một **GenAI / LLM Engineer** áp dụng để:
1. Xây dựng trợ lý ảo thông minh cho doanh nghiệp (Enterprise AI Assistant).
2. Kiểm soát rủi ro pháp lý và bảo mật dữ liệu khách hàng.
3. Liên tục tối ưu hóa trải nghiệm người dùng thông qua đo lường định lượng và A/B testing tự động.
