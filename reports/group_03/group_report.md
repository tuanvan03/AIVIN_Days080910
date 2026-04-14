# Báo Cáo Nhóm — Lab Day 08: Full RAG Pipeline

**Tên nhóm:** Group 03  
**Thành viên:**
| Tên | Vai trò | Email |
|-----|---------|-------|
| Đoàn Văn Tuấn | Tech Lead | doantuanvan2003@gmail.com |
| Vũ Minh Khải | Retrieval Owner | vmkqa2@gmail.com |
| Ninh Quang Trí | Eval Owner | nq.tri2511@gmail.com |
| Lê Nguyễn Thanh Bình | Documentation Owner | thanhbinh.lenguyen.1208@gmail.com |

**Ngày nộp:** 13/04/2026  
**Repo:** https://github.com/tuanvan03/AIVIN_Days080910/tree/main 
**Độ dài khuyến nghị:** 600–900 từ

---

> **Hướng dẫn nộp group report:**
>
> - File này nộp tại: `reports/group_report.md`
> - Deadline: Được phép commit **sau 18:00** (xem SCORING.md)
> - Tập trung vào **quyết định kỹ thuật cấp nhóm** — không trùng lặp với individual reports
> - Phải có **bằng chứng từ code, scorecard, hoặc tuning log** — không mô tả chung chung

---

## 1. Pipeline nhóm đã xây dựng (150–200 từ)

> Mô tả ngắn gọn pipeline của nhóm:
> - Chunking strategy: size, overlap, phương pháp tách (by paragraph, by section, v.v.)
> - Embedding model đã dùng
> - Retrieval mode: dense / hybrid / rerank (Sprint 3 variant)

**Chunking decision:**
> VD: "Nhóm dùng chunk_size=500, overlap=50, tách theo section headers vì tài liệu có cấu trúc rõ ràng."

- Nhóm sử dụng chiến lược Recursive Character Splitting với chunk_size=400 và overlap=80. Chúng tôi ưu tiên tách theo cấu trúc tự nhiên của tài liệu (paragaphs) nhưng giữ size nhỏ để đảm bảo mỗi chunk tập trung vào một quy định cụ thể, tránh việc trộn lẫn nhiều điều khoản khác nhau trong cùng một ngữ cảnh.

_________________

**Embedding model:**

- Sử dụng model text-embedding-3-small của OpenAI. Model này cung cấp sự cân bằng tối ưu giữa chi phí và khả năng biểu diễn ngữ nghĩa cho các tài liệu chính sách tiếng Việt.

_________________

**Retrieval variant (Sprint 3):**
> Nêu rõ variant đã chọn (hybrid / rerank / query transform) và lý do ngắn gọn.

- Nhóm chọn Hybrid Retrieval (Dense + BM25). Lý do là vì tập tài liệu chứa nhiều từ khóa đặc thù (mã lỗi, tên phần mềm, chức danh) mà Dense Retrieval thuần túy thường bỏ sót do không có sự tương đồng cao về vector ngữ nghĩa.

_________________

---

## 2. Quyết định kỹ thuật quan trọng nhất (200–250 từ)

> Chọn **1 quyết định thiết kế** mà nhóm thảo luận và đánh đổi nhiều nhất trong lab.
> Phải có: (a) vấn đề gặp phải, (b) các phương án cân nhắc, (c) lý do chọn.

**Quyết định:** Chuyển đổi từ Baseline (Dense) sang Variant 1 (Hybrid Retrieval).

**Bối cảnh vấn đề:**
- Ở Sprint 2, hệ thống Baseline gặp lỗi nghiêm trọng về Context Recall (chỉ đạt ~3.5/5). Cụ thể, khi người dùng hỏi về "Contractor" hoặc các mã lỗi kỹ thuật, Vector Search không tìm thấy đúng đoạn văn bản quy định vì các từ khóa này bị "loãng" trong không gian embedding.

_________________

**Các phương án đã cân nhắc:**

| Phương án | Ưu điểm | Nhược điểm |
|-----------|---------|-----------|
| Tăng Top-K (Dense) | Dễ triển khai, lấy được nhiều context hơn. | Tăng nhiễu (noise) cho LLM, tốn token. |
| Hybrid Retrieval | Bắt chính xác từ khóa (BM25) và ngữ nghĩa (Dense). | Cần thêm logic fusion (RRF), tốn tài nguyên tính toán. |

**Phương án đã chọn và lý do:**
- Nhóm chọn Hybrid Retrieval. Bằng chứng từ quá trình debug cho thấy các câu hỏi thất bại của Baseline đều rơi vào các đoạn văn bản có chứa danh từ riêng hoặc mã số. Hybrid giúp bù đắp khiếm khuyết của Dense trong việc xử lý "exact match".

_________________

**Bằng chứng từ scorecard/tuning-log:**
- Sau khi chuyển sang Hybrid, điểm Context Recall tăng từ 3.5 lên 4.5. Hệ thống đã tìm thấy tài liệu chính xác cho câu gq05 (Contractor) và gq07 (Alias tài liệu), những câu mà Baseline trước đó đã thất bại.

_________________

---

## 3. Kết quả grading questions (100–150 từ)

> Sau khi chạy pipeline với grading_questions.json (public lúc 17:00):
> - Câu nào pipeline xử lý tốt nhất? Tại sao?
> - Câu nào pipeline fail? Root cause ở đâu (indexing / retrieval / generation)?
> - Câu gq07 (abstain) — pipeline xử lý thế nào?

**Ước tính điểm raw:** 93 / 98

**Câu tốt nhất:** ID: gq01 và gq09 — Lý do: Lý do: Pipeline xử lý hoàn hảo việc trích xuất đa thông số (90 ngày, 7 ngày, link portal) và so sánh phiên bản (v2025.3 vs v2026.1) nhờ metadata effective_date được đánh dấu kỹ từ khâu Indexing.

**Câu fail:** ID: gq05 — Root cause: **Generation**. Dù Retrieval lấy đúng tài liệu, nhưng LLM bị nhầm lẫn giữa các mốc thời gian của Level 1 (1 ngày) và Level 4 (5 ngày) do chúng nằm quá gần nhau trong context.

**Câu gq07 (abstain):** Pipeline xử lý rất tốt. Nhờ Grounded Prompt nghiêm ngặt, khi tài liệu không đề cập đến mức phạt vi phạm SLA, hệ thống đã trả lời "Không đủ dữ liệu" thay vì tự suy diễn (Hallucination resistance đạt 5/5).

---

## 4. A/B Comparison — Baseline vs Variant (150–200 từ)

> Dựa vào `docs/tuning-log.md`. Tóm tắt kết quả A/B thực tế của nhóm.

**Biến đã thay đổi (chỉ 1 biến):** Chiến lược Retrieval (Dense $\rightarrow$ Hybrid).

| Metric | Baseline | Variant 1 | Delta |
|--------|----------|-----------|-------|
| Faithfulness | 4.8/5 | 5/5 | +0.2 |
| Answer Relevance | 4.2/5 | 4.9/5 | +0.7 |
| Context Recall | 3.5/5 | 4.5/5 | +1.0 |
| Completeness | 3.8/5 | 4.2/5 | +0.4 |

**Kết luận:**
> Variant tốt hơn hay kém hơn? Ở điểm nào?
- Variant (Hybrid) tốt hơn rõ rệt. Sự cải thiện lớn nhất nằm ở độ ổn định của việc tìm kiếm chứng cứ. Dù điểm Completeness chưa tuyệt đối do lỗi nhầm lẫn thuộc tính ở một số câu khó, nhưng Hybrid đã tạo ra một nền tảng dữ liệu đầu vào cực kỳ vững chắc cho LLM.

_________________

---

## 5. Phân công và đánh giá nhóm (100–150 từ)

> Đánh giá trung thực về quá trình làm việc nhóm.

**Phân công thực tế:**

| Thành viên | Phần đã làm | Sprint |
|------------|-------------|--------|
| Đoàn Văn Tuấn | Build Index | 1 |
|  Vũ Minh Khải | Baseline Retrieval + Answer | 2 + 3 |
| Ninh Quang Trí | Tuning Tối Thiểu | 3 + 4 |
| Lê Nguyễn Thanh Bình | Evaluation + Docs + Report | 4 |

**Điều nhóm làm tốt:**
- Phối hợp nhịp nhàng giữa khâu Eval và Retrieval; khi Eval phát hiện lỗi, Retrieval lập tức điều chỉnh metadata để hỗ trợ filter.

_________________

**Điều nhóm làm chưa tốt:**
- Quá trình xử lý nhiễu (noise) sau khi Hybrid lấy về quá nhiều chunk chưa được tối ưu, dẫn đến một vài lỗi nhỏ về độ đầy đủ của câu trả lời.

_________________

---

## 6. Nếu có thêm 1 ngày, nhóm sẽ làm gì? (50–100 từ)

> 1–2 cải tiến cụ thể với lý do có bằng chứng từ scorecard.
- Nhóm sẽ triển khai module Rerank (Cross-Encoder). Kết quả scorecard câu gq05 cho thấy Hybrid đang kéo về quá nhiều thông tin tương đồng gây nhiễu cho LLM. Việc thêm Reranker sẽ giúp lọc lại Top-3 chunk thực sự liên quan nhất, từ đó giải quyết triệt để lỗi "nhầm lẫn thuộc tính" và nâng điểm Completeness lên tối đa.

_________________

---

*File này lưu tại: `reports/group_report.md`*  
*Commit sau 18:00 được phép theo SCORING.md*
