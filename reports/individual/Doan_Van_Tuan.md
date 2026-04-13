# Báo Cáo Cá Nhân — Lab Day 08: RAG Pipeline

**Họ và tên:** Đoàn Văn Tuấn 
**MSSV:** 2A202600046
**Vai trò trong nhóm:** Tech Lead
**Ngày nộp:** 14/04/2026

---

## 1. Tôi đã làm gì trong lab này? (100-150 từ)

> Trong lab hôm nay, trong vai trò là teach lead, em đã cùng các thành viên trong nhóm thảo luận và đưa ra các quyết định cho pipeline rag. Cụ thể những việc em đã làm hôm nay:
- Thiết lập và cài đặt project chung của nhóm 
- Thảo luận chọn model embedding và các phương pháp retrieval, reranking
- Triển khai nhanh phương pháp index, reranking (sử dụng crossencoder) cho nhóm
- Giữ nhịp spint theo đúng timeline và đảm bảo các thành viên hoàn thành nhiệm vụ được giao
- Tiến hành merge code + hỗ trợ review + debug code cho các thành viên
- Hỗ trợ test questions + đánh giá kết quả 

---

## 2. Điều tôi hiểu rõ hơn sau lab này (100-150 từ)

> Sau lab này thì em hiểu rõ hơn về cách thức đánh giá mô hình RAG, đặc biệt là các metrics như faithfulness, answer relevance, context relevance, citation recall, citation precision và phương pháp sử dụng LLM để chấm điểm cho LLM (LLM as a judge). Em cũng hiểu rõ hơn về vai trò của từng thành phần trong pipeline RAG và cách chúng tương tác với nhau.

_________________

---

## 3. Điều tôi ngạc nhiên hoặc gặp khó khăn (100-150 từ)

> Việc debug + merge code + giải quyết conflic trong quá trình làm việc nhóm là điều em gặp khó khăn nhất. Em cần đảm bảo phải hiểu rõ các luồng hoạt động + logic của từng thành viên để có thể hỗ trợ kịp thời chỉnh sửa và thống nhất hướng đi cho cả nhóm. 

_________________

---

## 4. Phân tích một câu hỏi trong scorecard (150-200 từ)

**Câu hỏi:** Nếu cần hoàn tiền khẩn cấp cho khách hàng VIP, quy trình có khác không?

**Phân tích:**
- Baseline trả lời thiếu tài liệu so với phương pháp hybrid thực tế của nhóm 
- Lỗi này nằm ở việc thiếu source ở trong bước retrieval
- Cách giải quyết có thể tăng topk lên hoặc sử dụng phương pháp child-parent chunking để giải quyết. (Em đã thử nghiệm với topk=10 và thấy kết quả cải thiện đáng kể)

_________________

---

## 5. Nếu có thêm thời gian, tôi sẽ làm gì? (50-100 từ)

> Sử dụng các bộ dữ liệu lớn hơn, như các văn bản dài hay paper cho đa dạng về thông tin. Mục đích em muốn thử nghiệm và đánh giá với Encoder + Crossencoder và so sánh chúng với các phương pháp Hybrid khác. Ngoài ra, Query Transformation cũng khá là hay, đây là một khái niệm mới đối với bản thân em, em muốn tìm hiểu sâu hơn về nó và thử nghiệm nó trong pipeline RAG của nhóm.

_________________
