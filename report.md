# BÁO CÁO THỰC HÀNH LAB DAY 22
## LangSmith + Prompt Versioning + RAGAS Evaluation + Guardrails AI

---

## 📌 THÔNG TIN HỌC VIÊN & NỘP BÀI

- **Họ và tên:** Nguyễn Duy Trọng
- **Mã sinh viên / MSSV:** `2A202601333`
- **GitHub Repository:** [https://github.com/DuyTrongK64/Track2-DAY22-NguyenDuyTrong-2A202601333](https://github.com/DuyTrongK64/Track2-DAY22-NguyenDuyTrong-2A202601333)
- **LangSmith Project URL:** [https://smith.langchain.com/o/default/projects/p/day22-lab](https://smith.langchain.com/o/default/projects/p/day22-lab)
- **LangSmith Prompt Hub URLs:**
  - **Prompt V1:** [https://smith.langchain.com/prompts/nguyen-duy-trong-rag-prompt-v1](https://smith.langchain.com/prompts/nguyen-duy-trong-rag-prompt-v1)
  - **Prompt V2:** [https://smith.langchain.com/prompts/nguyen-duy-trong-rag-prompt-v2](https://smith.langchain.com/prompts/nguyen-duy-trong-rag-prompt-v2)

---

## 1. TỔNG QUAN CÁC BƯỚC THỰC HIỆN

| Bước | Mô tả nội dung | File thực thi | Trạng thái |
|:---:|:---|:---|:---:|
| **Bước 1** | **LangSmith RAG Pipeline:** Xây dựng FAISS vectorstore (107 chunks), tạo chain LCEL, gắn `@traceable(name="rag-query")` và log 50 QA pairs kèm metadata lên LangSmith project `day22-lab`. | `src/01_langsmith_rag_pipeline.py` | ✅ **Hoàn thành (50/50 Traces Online)** |
| **Bước 2** | **Prompt Hub & A/B Routing:** Định nghĩa 2 prompt versions (`nguyen-duy-trong-rag-prompt-v1`, `v2`), push/pull từ LangSmith Prompt Hub, định tuyến A/B deterministic bằng MD5 hashing (`md5(request_id) % 100 < 50`). | `src/02_prompt_hub_ab_routing.py` | ✅ **Hoàn thành (19 V1 / 31 V2)** |
| **Bước 3** | **RAGAS Evaluation:** Đánh giá định lượng 50 QA pairs qua cả 2 phiên bản prompt với 4 metric RAGAS cốt lõi. | `src/03_ragas_evaluation.py` | ✅ **Hoàn thành (Faithfulness = 0.9280 $\ge$ 0.80)** |
| **Bước 4** | **Guardrails AI Validators:** Xây dựng `PIIDetector` (redact email, phone, ssn, credit card) và `JSONFormatter` (sửa markdown fences, single quotes, trailing commas, fallback JSON). | `src/04_guardrails_validator.py` | ✅ **Hoàn thành (100% Test Cases Pass)** |

---

## 2. KẾT QUẢ ĐÁNH GIÁ RAGAS (BƯỚC 3)

Tất cả 50 câu hỏi trong `data/eval_dataset.json` đã được đánh giá chi tiết với các chỉ số như sau:

| Metric | Prompt V1 (Concise) | Prompt V2 (Structured) | Winner | Tiêu chuẩn đánh giá |
|:---|:---:|:---:|:---:|:---:|
| **Faithfulness (Độ trung thực)** | **0.9280** | 0.9114 | **← V1** | ✅ **ĐẠT (Target $\ge$ 0.80)** |
| **Answer Relevancy (Độ liên quan)** | 0.8855 | **0.9043** | **← V2** | ✅ **ĐẠT** |
| **Context Recall (Độ bao phủ)** | 0.9756 | **1.0000** | **← V2** | ✅ **ĐẠT** |
| **Context Precision (Độ chính xác ngữ cảnh)** | **0.9125** | 0.8889 | **← V1** | ✅ **ĐẠT** |

---

## 3. PHÂN TÍCH SO SÁNH CHUYÊN SÂU: PROMPT V1 VS PROMPT V2 (BONUS ANALYSIS)

1. **Vì sao Prompt V1 thắng ở `Faithfulness` (0.9280) và `Context Precision` (0.9125):**
   - Prompt V1 chỉ thị phong cách trả lời ngắn gọn, trực tiếp (2-4 câu). Nhờ đó, LLM tập trung trích xuất chính xác các sự kiện cốt lõi từ context mà không suy diễn hay thêm các cấu trúc liên kết phức tạp. Điều này giúp tỷ lệ các claim được hỗ trợ 100% bởi context cao hơn.

2. **Vì sao Prompt V2 thắng ở `Answer Relevancy` (0.9043) và `Context Recall` (1.0000):**
   - Prompt V2 đóng vai chuyên gia phân tích với cấu trúc mạch lạc (3-5 câu), giải thích rõ cơ chế và các khía cạnh liên quan, giúp câu trả lời bao quát đầy đủ tất cả thông tin trong ground truth reference (đạt điểm tuyệt đối 100% Context Recall).

---

## 4. DANH MỤC MINH CHỨNG (EVIDENCE ARTIFACTS)

| File minh chứng | Mô tả chi tiết |
|:---|:---|
| `evidence/01_langsmith_traces.png` | Ảnh chụp dashboard LangSmith hiển thị các traces của project `day22-lab` |
| `evidence/02_prompt_hub.png` | Ảnh chụp Prompt Hub với 2 phiên bản prompt và cơ chế A/B routing |
| `evidence/02_ab_routing_log.txt` | Console log thực thi Bước 2 với tỷ lệ điều phối 19 V1 và 31 V2 |
| `evidence/03_ragas_scores.png` | Biểu đồ trực quan so sánh 4 metric RAGAS giữa V1 và V2 |
| `evidence/03_ragas_report.json` | File JSON xuất kết quả đánh giá RAGAS đầy đủ |
| `evidence/04_pii_demo_log.txt` | Console log kiểm thử PII Detector che giấu thông tin nhạy cảm |
| `evidence/04_json_demo_log.txt` | Console log kiểm thử JSON Formatter tự động sửa lỗi cú pháp |
| `evidence/README.md` | Tài liệu phân tích so sánh chi tiết giữa 2 phiên bản prompt |

---

## 5. HƯỚNG DẪN CHẠY LẠI TOÀN BỘ LAB (REPRODUCIBILITY)

```bash
# Chạy tất cả 4 bước tuần tự
python src/run_all.py

# Hoặc chạy từng bước độc lập
python src/run_all.py --step 1    # Bước 1: LangSmith Tracing
python src/run_all.py --step 2    # Bước 2: Prompt Hub & A/B Routing
python src/run_all.py --step 3    # Bước 3: RAGAS Evaluation
python src/run_all.py --step 4    # Bước 4: Guardrails AI Validators
```
