# Báo Cáo Đánh Giá & So Sánh RAG Pipeline (Day 22 Lab)

**Học viên:** Nguyễn Duy Trọng  
**Mã học viên / ID:** 2A202601333  
**Repository:** `DuyTrongK64/Track2-DAY22-NguyenDuyTrong-2A202601333`  
**Dự án LangSmith:** `day22-lab`  

---

## 1. Tổng quan các bước thực hiện

| Bước | Nội dung | File thực thi | Trạng thái | Minh chứng |
|:---|:---|:---|:---:|:---|
| **Bước 1** | LangSmith RAG Pipeline (50 queries, FAISS retriever, OpenRouter LLM) | `src/01_langsmith_rag_pipeline.py` | ✅ Hoàn thành | `evidence/01_langsmith_traces.png` |
| **Bước 2** | Prompt Hub & Deterministic A/B Routing (MD5 hashing) | `src/02_prompt_hub_ab_routing.py` | ✅ Hoàn thành | `evidence/02_prompt_hub.png`<br>`evidence/02_ab_routing_log.txt` |
| **Bước 3** | RAGAS 4-Metric Evaluation (Faithfulness, Relevancy, Recall, Precision) | `src/03_ragas_evaluation.py` | ✅ Hoàn thành | `evidence/03_ragas_scores.png`<br>`evidence/03_ragas_report.json`<br>`data/ragas_report.json` |
| **Bước 4** | Guardrails AI Validators (PII Redaction & JSON Repair) | `src/04_guardrails_validator.py` | ✅ Hoàn thành | `evidence/04_pii_demo_log.txt`<br>`evidence/04_json_demo_log.txt` |

---

## 2. Kết quả đánh giá RAGAS (Bước 3)

Tất cả 50 cặp QA trong bộ dữ liệu chuẩn (`data/eval_dataset.json`) đã được đánh giá qua cả 2 phiên bản prompt:

| Metric | Prompt V1 (Concise) | Prompt V2 (Structured/Expert) | Winner | Đạt chuẩn (Target ≥ 0.8) |
|:---|:---:|:---:|:---:|:---:|
| **Faithfulness (Độ trung thực)** | **0.9280** | 0.9114 | **← V1** | ✅ ĐẠT (≥ 0.80) |
| **Answer Relevancy (Độ liên quan câu trả lời)** | 0.8855 | **0.9043** | **← V2** | ✅ ĐẠT |
| **Context Recall (Độ bao phủ ngữ cảnh)** | 0.9756 | **1.0000** | **← V2** | ✅ ĐẠT |
| **Context Precision (Độ chính xác ngữ cảnh)** | **0.9125** | 0.8889 | **← V1** | ✅ ĐẠT |

> **Kết luận mục tiêu:** Cả 2 phiên bản prompt đều vượt xa ngưỡng yêu cầu tối thiểu **`faithfulness ≥ 0.80`** (V1 đạt 0.9280, V2 đạt 0.9114).

---

## 3. Phân tích so sánh chuyên sâu: Prompt V1 vs Prompt V2 (Bonus Analysis)

### 3.1. Thiết kế của hai Prompt
- **Prompt V1 (`nguyen-duy-trong-rag-prompt-v1`):**
  - *System Persona:* Trợ lý AI thân thiện, trả lời ngắn gọn, trực tiếp, súc tích (2-4 câu).
  - *Mục tiêu:* Tối ưu tốc độ đọc, hạn chế tối đa hallucination bằng cách chỉ nêu trực diện câu trả lời.
- **Prompt V2 (`nguyen-duy-trong-rag-prompt-v2`):**
  - *System Persona:* Chuyên gia phân tích thông tin & AI, diễn giải có cấu trúc mạch lạc, chuẩn xác (3-5 câu).
  - *Mục tiêu:* Đào sâu các khía cạnh kỹ thuật, cung cấp câu trả lời toàn diện, giải thích nguyên lý rõ ràng.

---

### 3.2. Lý do Prompt V1 thắng ở `Faithfulness` và `Context Precision`

1. **Faithfulness (V1: 0.9280 vs V2: 0.9114):**
   - V1 có chỉ thị chặt chẽ về độ dài (2-4 câu) và phong cách trả lời trực tiếp. Nhờ đó, LLM tập trung trích xuất chính xác các mệnh đề (claims) xuất hiện trong context mà không thêm thắt các từ nối phức tạp hoặc giải thích phái sinh.
   - Khi RAGAS phân rã câu trả lời thành từng atomic claim để đối chiếu với context, tỷ lệ các claim của V1 được context hỗ trợ 100% cao hơn.

2. **Context Precision (V1: 0.9125 vs V2: 0.8889):**
   - Do V1 trả lời cô đọng, câu trả lời phản ánh sát nhất các đoạn context có độ tương đồng cosine cao nhất ở top 1.

---

### 3.3. Lý do Prompt V2 thắng ở `Answer Relevancy` và `Context Recall`

1. **Answer Relevancy (V2: 0.9043 vs V1: 0.8855):**
   - V2 được hướng dẫn phân tích đầy đủ các luận điểm, nên câu trả lời làm rõ được bản chất câu hỏi, ví dụ không chỉ định nghĩa thuật ngữ mà còn nêu rõ cơ chế hoạt động và ứng dụng thực tế.
   - Khi RAGAS sinh câu hỏi ngược từ câu trả lời của V2 và tính cosine similarity với câu hỏi gốc của người dùng, độ tương thích ngữ nghĩa đạt tới **0.9043**.

2. **Context Recall (V2: 1.0000 vs V1: 0.9756):**
   - V2 đạt điểm tuyệt đối **1.0000 (100%)** ở Context Recall.
   - Do ground truth reference chứa nhiều chi tiết kỹ thuật, cấu trúc 3-5 câu của V2 đã bao quát đầy đủ tất cả các thông tin trong reference mà context cung cấp, trong khi V1 đôi khi lược bỏ một số chi tiết phụ để giữ câu trả lời ngắn gọn (2-4 câu).

---

### 3.4. Đề xuất ứng dụng thực tế (Production Recommendation)
- **Cho người dùng cuối / chatbot tương tác nhanh:** Sử dụng **Prompt V1** để tối ưu hóa latency, tiết kiệm token và đảm bảo độ trung thực cao nhất.
- **Cho báo cáo kỹ thuật / enterprise Q&A:** Sử dụng **Prompt V2** để có câu trả lời đầy đủ, bao phủ 100% ngữ cảnh thông tin cần thiết.

---

## 4. Danh mục minh chứng (Evidence Artifacts)

- `evidence/01_langsmith_traces.png`: Ảnh chụp theo dõi 50 traces trên LangSmith project `day22-lab`.
- `evidence/02_prompt_hub.png`: Ảnh chụp Prompt Hub với 2 phiên bản V1 và V2 cùng cơ chế A/B routing.
- `evidence/02_ab_routing_log.txt`: Console log thực thi Bước 2 (19 queries V1, 31 queries V2).
- `evidence/03_ragas_scores.png`: Biểu đồ so sánh 4 metric RAGAS giữa V1 và V2.
- `evidence/03_ragas_report.json`: Dữ liệu JSON điểm số RAGAS chi tiết.
- `evidence/04_pii_demo_log.txt`: Console log kiểm thử PII Detector (redact email, phone, ssn, card).
- `evidence/04_json_demo_log.txt`: Console log kiểm thử JSON Formatter (sửa fences, single quotes, trailing commas).
- `evidence/README.md`: Báo cáo phân tích chuyên sâu (file hiện tại).
