# Báo Cáo Cá Nhân — Lab Day 08: RAG Pipeline

**Họ và tên:** Vũ Minh Khải
**Vai trò trong nhóm:** Retrieval Owner
**Ngày nộp:** 13/04/2026
**Độ dài yêu cầu:** 500–800 từ

---

## 1. Tôi đã làm gì trong lab này? (100-150 từ)

> Mô tả cụ thể phần bạn đóng góp vào pipeline:
> - Sprint nào bạn chủ yếu làm?:
>   - Em làm chủ yếu về Sprint 2 Build retrieval + grounded answer function và Sprint 3: Cài đặt Sparse search và Hybrid search.
> - Cụ thể bạn implement hoặc quyết định điều gì?
>   - Cài đặt các yêu cầu trong Sprint 2: implement retrieve_dense(), call_llm(), chỉnh sửa grounded prompt. Sprint 3: cài đặt retrieve_sparse() + retrieve_hybrid(). Về phần retrieve_sparse, em phát hiện BM25 nếu dùng trực tiếp thì sẽ ko tốt cho tiếng việt, vì tiếng việt ko giống tiếng anh do có các từ ghép,... Em sử dụng thư viện underthesea để token hóa, sau đó mới sử dụng cho BM25
> - Công việc của bạn kết nối với phần của người khác như thế nào?
>   - Kế thừa sử dụng dữ liệu đã được chunking và index vào vector database từ Sprint 1. Và phần của em được các bạn đánh giá ở Sprint 4
---

## 2. Điều tôi hiểu rõ hơn sau lab này (100-150 từ)

> - chunking: Cần căn cứ vào tài liệu thực tế để quyết định cách chunking như thế nào. Ví dụ với tài liệu cụ thể hiện tại, các file nội dung có độ tương đồng về định dạng: bắt đầu === Section ===, sau đó phân tách nội dung trong section bằng '\n',... Từ phân tích này có thể xây dựng cách chunking hiệu quả.
> - hybrid retrieval: Kết hợp kết quả từ dense retrieval và sparse retrieval. Với những câu hỏi cần thông tin cụ thể ví dụ về điều luật,... sparse search phát huy hiệu quả bằng cách tìm kiếm từ chính xác. Với những câu cần hiểu về ngữ nghĩa, dense retrieval hoạt động tốt hơn. Việc kết hợp này giúp tăng khả năng tìm thấy được tài liệu liên quan so với từng phương pháp độc lập. 

_________________

---

## 3. Điều tôi ngạc nhiên hoặc gặp khó khăn (100-150 từ)

> - Điều gì xảy ra không đúng kỳ vọng?
>   - Ban đầu cài đặt BM25 trực tiếp cho sparse search, kết quả khá kém và kéo theo kết quả hybrid search cũng bị kém theo...
> - Lỗi nào mất nhiều thời gian debug nhất?
>   - Việc tìm hiểu tại ra lỗi ở BM25 cũng mất khá nhiều thời gian. Ban đầu thấy kết quả hybrid search kém, em nghĩ đến có lẽ cách kết hợp kết quả dense search và sparse search bị sai. Sau đó thì in trực tiếp kết quả dense search, sparse search thì mới bắt đầu nhận ra kq từ sparse search có vấn đề.
> - Giả thuyết ban đầu của bạn là gì và thực tế ra sao?
>   - Giả thuyết rằng có thể BM25 không hoạt động tốt cho tiếng việt -> bắt đầu tìm hiểu -> cách token hóa của BM25 hiện tại đang dùng là split từ, trong khi tiếng Việt khác 1 chút là có những từ ghép -> Sử dụng thư viện Underthesea để token hóa câu tiếng Việt. Cuối cùng kết quả đã cải thiện cho cả sparse search và hybrid search. (giả thuyết trùng thực tế)
---

## 4. Phân tích một câu hỏi trong scorecard (150-200 từ)
**Câu hỏi:** ERR-403-AUTH là lỗi gì và cách xử lý? 
>   - Câu hỏi này không có tài liệu căn cứ để trả lời.
>   - Câu này nhóm em đã sửa lại bằng prompt rồi nên ko bị sai nữa. nhưng mà nó khá thú vị nên em để vào.

**Phân tích:**
>   - Baseline trả lời đúng. Và câu trả lời chỉ đơn giản là "Không đủ dữ liệu để trả lời. Do tài liệu không đề cập đến lỗi ERR-403-AUTH"
>   - Tuy nhiên sử dụng Variant (hybrid search) thì lại trả lời 1 loạt thông tin thừa ở phía trước, sau đó mới đưa ra thông tin rằng không có đủ dữ liệu để trả lời câu hỏi này.
>   - Vấn đề xảy ra do tài liệu baseline và variant nhận được khác nhau (dense search vs hybrid search). Sparse search tìm dữ liệu theo key word nên -> trả về các tài liệu gần giống -> LLM nhận các tài liệu gần giống -> nói lan man, sau đó mới đi vào vấn đề.
>   - Cách khắc phục là thay đổi prompt kĩ hơn cho Variant


---

## 5. Nếu có thêm thời gian, tôi sẽ làm gì? (50-100 từ)

> - Thử thêm 1 số phương pháp tiền xử lý dữ liệu trước cho ngôn ngữ tiếng việt sau đó mới chạy sparse search. Ko chắc chắn nhưng có thể sẽ cải thiện thêm về độ chính xác cho sparse search -> tăng độ chính xác cho hybrid search.
> - triển khai theo phương pháp small to big retrieval. với mỗi section là parent chunk, và các đoạn trong section là child chunk. Em cũng ko chắc về việc có thể cải thiện ko, nhưng khả năng cao là có vì nội dung embedding sẽ thành mỗi đoạn thành vì mỗi section (rộng) như hiện tại -> tăng độ chính xác.


---

*Lưu file này với tên: `reports/individual/[ten_ban].md`*
*Ví dụ: `reports/individual/nguyen_van_a.md`*
