# Báo Cáo Cá Nhân — Lab Day 08: RAG Pipeline

**Họ và tên:** Lê Nguyễn Thanh Bình
**Vai trò trong nhóm:** Documentation Owner  
**Ngày nộp:** 13/04/2026  
**Độ dài yêu cầu:** 500–800 từ

---

## 1. Tôi đã làm gì trong lab này? (100-150 từ)

> Mô tả cụ thể phần bạn đóng góp vào pipeline:
> - Sprint nào bạn chủ yếu làm?
> - Cụ thể bạn implement hoặc quyết định điều gì?
> - Công việc của bạn kết nối với phần của người khác như thế nào?

> Trong Lab này, với vai trò Documentation Owner, tôi chịu trách nhiệm chính trong việc "chuyển hóa" các con số kỹ thuật thành logic nghiệp vụ. Tôi tập trung chủ yếu vào Sprint 3 và 4 để ghi chép nhật ký thử nghiệm A/B (Tuning Log) và hoàn thiện hồ sơ kiến trúc hệ thống (architecture.md). Cụ thể, tôi đã cùng Tech Lead phân tích các mã lỗi phát sinh để xây dựng "Error Tree", từ đó đưa ra các giả thuyết về việc tại sao hệ thống từ chối trả lời hoặc trả lời sai. Tôi cũng phụ trách việc cấu trúc lại các file Markdown để đảm bảo tính minh bạch cho quá trình Evaluation. Công việc của tôi kết nối trực tiếp với Eval Owner để lấy dữ liệu Scorecard và với Retrieval Owner để hiểu rõ các tham số cấu hình như dense_weight hay sparse_weight nhằm giải trình trong báo cáo nhóm.

_________________

---

## 2. Điều tôi hiểu rõ hơn sau lab này (100-150 từ)

> Chọn 1-2 concept từ bài học mà bạn thực sự hiểu rõ hơn sau khi làm lab.
> Ví dụ: chunking, hybrid retrieval, grounded prompt, evaluation loop.
> Giải thích bằng ngôn ngữ của bạn — không copy từ slide.

>Sau Lab này, hai khái niệm tôi hiểu sâu sắc nhất là Hybrid Retrieval và Evaluation Loop. Trước đây, tôi nghĩ RAG chỉ đơn thuần là tìm kiếm bằng vector (Dense). Tuy nhiên, qua thực tế xử lý các tài liệu chứa nhiều mã lỗi kỹ thuật như "ERR-403-AUTH", tôi nhận ra Dense Search thường bỏ lỡ các keyword chính xác này. Hybrid Retrieval (kết hợp BM25) là giải pháp cứu cánh để bắt trọn các thực thể định danh. Thứ hai là vòng lặp đánh giá (Evaluation Loop): điểm số Scorecard không phải là kết quả cuối cùng mà là "la bàn" để chúng ta quay lại sửa Indexing hoặc Prompting. RAG không phải là một quy trình tuyến tính, mà là một chu kỳ tinh chỉnh liên tục dựa trên bằng chứng (evidence-based tuning). 

_________________

---

## 3. Điều tôi ngạc nhiên hoặc gặp khó khăn (100-150 từ)

> Điều gì xảy ra không đúng kỳ vọng?
> Lỗi nào mất nhiều thời gian debug nhất?
> Giả thuyết ban đầu của bạn là gì và thực tế ra sao?

>Điều khiến tôi ngạc nhiên nhất là sự sụt giảm điểm Faithfulness khi chúng tôi cố gắng tăng Context Recall. Ban đầu, giả thuyết của tôi là "càng lấy được nhiều thông tin (Recall cao) thì câu trả lời càng đúng". Tuy nhiên, thực tế ở Variant 1 (Hybrid), khi hệ thống truy xuất được rất nhiều chunk chứa các từ khóa trùng lặp nhưng thuộc các cấp độ khác nhau (như Level 1 và Level 4 trong Access Control), LLM đã bị "nhiễu" và trích xuất nhầm giá trị. Khó khăn lớn nhất là việc debug lỗi "Attribute Confusion" này. Chúng tôi đã mất nhiều thời gian để nhận ra rằng việc cung cấp quá nhiều ngữ cảnh không liên quan (noise) từ BM25 có thể phản tác dụng, gây ra hiện tượng ảo tưởng (hallucination) ngay cả khi thông tin đúng đã nằm trong context. 

_________________

---

## 4. Phân tích một câu hỏi trong scorecard (150-200 từ)

> Chọn 1 câu hỏi trong test_questions.json mà nhóm bạn thấy thú vị.
> Phân tích:
> - Baseline trả lời đúng hay sai? Điểm như thế nào?
> - Lỗi nằm ở đâu: indexing / retrieval / generation?
> - Variant có cải thiện không? Tại sao có/không?

**Câu hỏi:** "gq05: Contractor từ bên ngoài công ty có thể được cấp quyền Admin Access không? Nếu có, cần bao nhiêu ngày và có yêu cầu đặc biệt gì?"

**Phân tích:**
- Đây là câu hỏi thú vị nhất vì nó bộc lộ rõ sự đánh đổi giữa các chiến lược. Ở giai đoạn Baseline (Dense Search), câu hỏi này nhận điểm Recall là 1/5 và Completeness là 0. Lý do là vì tài liệu quy định về phạm vi đối tượng (Contractor) nằm ở Section 1, trong khi chi tiết về Admin Access (Level 4) nằm ở Section 4. Dense Search chỉ lấy được các đoạn ở giữa, dẫn đến việc LLM kết luận sai rằng Contractor không được cấp quyền.

- Khi chuyển sang Variant 1 (Hybrid), điểm Recall đã tăng lên 5/5 nhờ BM25 bắt đúng từ khóa "Contractor". Tuy nhiên, điểm Completeness lại chỉ đạt 2/5. Tại sao? Vì trong context lúc này chứa quá nhiều mốc thời gian (1 ngày cho Level 1, 2 ngày cho Level 2, 5 ngày cho Level 4). LLM đã bị nhầm lẫn và trả lời là "1 ngày" (của Level 1) thay vì "5 ngày". Lỗi này nằm ở khâu Generation bị quá tải bởi nhiễu từ Retrieval. Điều này khẳng định rằng chỉ tăng Recall là chưa đủ, mà cần một bước Rerank để sắp xếp lại độ ưu tiên của các bằng chứng.

_________________

---

## 5. Nếu có thêm thời gian, tôi sẽ làm gì? (50-100 từ)

> 1-2 cải tiến cụ thể bạn muốn thử.
> Không phải "làm tốt hơn chung chung" mà phải là:
> "Tôi sẽ thử X vì kết quả eval cho thấy Y."

>Tôi sẽ đề xuất nhóm thử nghiệm module Rerank (Cross-Encoder). Kết quả đánh giá hiện tại cho thấy Hybrid Retrieval đang lấy về quá nhiều thông tin gây nhiễu dẫn đến lỗi nhầm lẫn thuộc tính. Tôi muốn áp dụng Reranker để lọc lại Top-3 chunk có độ liên quan cao nhất từ danh sách Top-10 ứng viên. Tôi tin rằng việc làm sạch context trước khi đưa vào LLM sẽ giúp cải thiện đáng kể điểm Completeness (từ 3.2 lên trên 4.5) và loại bỏ hoàn toàn các lỗi nhầm lẫn số liệu như ở câu gq05. 

_________________

---

*Lưu file này với tên: `reports/individual/[ten_ban].md`*
*Ví dụ: `reports/individual/nguyen_van_a.md`*
