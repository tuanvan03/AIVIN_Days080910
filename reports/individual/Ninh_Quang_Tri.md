# Báo Cáo Cá Nhân — Lab Day 08: RAG Pipeline

**Họ và tên:** Ninh Quang Trí <br>
**Vai trò trong nhóm:** Eval Owner <br>
**Ngày nộp:** 13/04/2026 <br>
**Độ dài yêu cầu:** 500–800 từ

---

## 1. Tôi đã làm gì trong lab này? (100-150 từ)

Tôi phụ trách phần đánh giá toàn bộ pipeline trong Sprint 4. Tôi đã đọc và chỉnh sửa `eval.py` để xây dựng scorecard với 4 metric: Faithfulness, Relevance, Context Recall và Completeness. Tôi cũng chạy `run_scorecard()` cho cả cấu hình baseline và variant, so sánh A/B, và thiết kế prompt đánh giá LLM sao cho trả về JSON rõ ràng. Ngoài ra tôi theo dõi test questions, grading question, kết nối kết quả đánh giá với việc chọn variant hybrid và đảm bảo các kết quả scorecard lưu ra `results/` va `logs\` để nhóm dùng cho báo cáo cuối cùng.

---

## 2. Điều tôi hiểu rõ hơn sau lab này (100-150 từ)

Tôi hiểu rõ hơn về vòng lặp evaluation trong RAG pipeline và cách phân biệt chất lượng của retrieval với chất lượng của generation. Tôi nhận ra rằng một câu trả lời có vẻ “hợp lý” chưa chắc là grounded nếu nguồn evidence không được retrieve đúng nên `Context Recall` là metric quan trọng để đánh giá retrieval riêng. Tôi cũng hiểu rõ rằng prompt đánh giá phải yêu cầu model trả về JSON thuần để tránh parsing lỗi khi dùng LLM-as-Judge trong `eval.py`.

---

## 3. Điều tôi ngạc nhiên hoặc gặp khó khăn (100-150 từ)

Khó khăn lớn nhất là alignment giữa source trong chunk và expected_sources của test question. Câu hỏi thiếu context như `ERR-403-AUTH` đòi hỏi pipeline không chỉ retrieve tốt mà còn phải biết abstain hoặc trả lời “không đủ dữ liệu”. Debug phần parsing JSON từ response LLM trong `score_faithfulness` và `score_completeness` tốn thời gian vì model đôi khi trả thêm văn bản ngoài JSON.

---

## 4. Phân tích một câu hỏi trong scorecard (150-200 từ)

**Câu hỏi:** Approval Matrix để cấp quyền hệ thống là tài liệu nào?

**Phân tích:** Đây là câu hỏi alias nên baseline dense search dễ bỏ qua nếu truy vấn không có exact keyword. Baseline có thể trả lời sai hoặc trả nguồn không đúng vì chỉ dựa vào embedding mà không tận dụng được alias “Approval Matrix”. Variant hybrid cải thiện được nhờ kết hợp sparse keyword và dense semantic, giúp tìm đúng document `access-control-sop.md` dù query dùng tên cũ. Điểm `Context Recall` của variant cao hơn baseline, chứng tỏ retrieval tốt hơn, và `Faithfulness` cũng cải thiện vì answer dựa trên đúng chunk. Nếu variant vẫn sai thì lỗi chủ yếu nằm ở retrieval (chưa lấy đúng nguồn) chứ không phải generation, vì expected source đã rõ ràng.

---

## 5. Nếu có thêm thời gian, tôi sẽ làm gì? (50-100 từ)

Tôi sẽ mở rộng evaluation bằng 2 cải tiến: (1) bổ sung test questions cho các query alias và trường hợp “no answer” để stress-test retrieval + abstain, và (2) tự động hóa matching `expected_sources` bằng map alias source/partial path để giảm false negative trong `Context Recall`.

---

*Lưu file này với tên: `reports/individual/[ten_ban].md`*
*Ví dụ: `reports/individual/nguyen_van_a.md`*
