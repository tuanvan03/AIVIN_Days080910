# Tuning Log — RAG Pipeline (Day 08 Lab)

> Template: Ghi lại mỗi thay đổi và kết quả quan sát được.
> A/B Rule: Chỉ đổi MỘT biến mỗi lần.

---

## Baseline (Sprint 2)

**Ngày:** 13/04/2026
**Config:**
```
retrieval_mode = "dense"
chunk_size = 400 tokens
overlap = 80 tokens
top_k_search = 10
top_k_select = 3
use_rerank = False
llm_model = gpt-4o mini
```

**Scorecard Baseline:**
| Metric | Average Score |
|--------|--------------|
| Faithfulness | 4.80/5 |
| Answer Relevance | 4.20/5 |
| Context Recall | 3.5/5 |
| Completeness | 3.80/5 |

**Câu hỏi yếu nhất (điểm thấp):**

> TODO: Liệt kê 2-3 câu hỏi có điểm thấp nhất và lý do tại sao.
> Ví dụ: "q07 (Approval Matrix) - context recall = 1/5 vì dense bỏ lỡ alias."

- q05 (Admin Access cho Contractor) - context recall = 0/5: Do cơ chế Dense Retrieval không tìm nạp được phần phạm vi áp dụng (Section 1), dẫn đến việc mô hình kết luận sai rằng contractor không được cấp quyền.

- gq02 & gq09 (Chi tiết kỹ thuật FAQ) - completeness = 3/5: Do pipeline bỏ lỡ các chi tiết thực thể nhỏ như tên phần mềm (Cisco AnyConnect) hoặc kênh liên lạc cụ thể, dẫn đến câu trả lời đúng trọng tâm nhưng thiếu sót thông tin bổ trợ quan trọng.

- gq07 (Anti-hallucination) - context recall = None: Đây là điểm yếu về mặt dữ liệu (thông tin không tồn tại), kiểm tra khả năng từ chối trả lời của hệ thống thay vì cố gắng suy diễn.



**Giả thuyết nguyên nhân (Error Tree):**
- [ ] Indexing: Chunking cắt giữa điều khoản
- [ ] Indexing: Metadata thiếu effective_date
- [x] Retrieval: Dense bỏ lỡ exact keyword / alias
- [x] Retrieval: Top-k quá ít → thiếu evidence
- [x] Generation: Prompt không đủ grounding
- [x] Generation: Context quá dài → lost in the middle

---

## Variant 1 (Sprint 3)

**Ngày:**13/04/2026  
**Biến thay đổi:** retrieve_sparse() + retrieve_hybrid() (Chuyển sang Hybrid Retrieval)
**Lý do chọn biến này:**
> TODO: Giải thích theo evidence từ baseline results.
> Ví dụ: "Chọn hybrid vì q07 (alias query) và q09 (mã lỗi ERR-403) đều thất bại với dense.
> Corpus có cả ngôn ngữ tự nhiên (policy) lẫn tên riêng/mã lỗi (ticket code, SLA label)."

- Bằng chứng từ Baseline: Kết quả Baseline cho thấy hệ thống gặp khó khăn lớn với các câu hỏi chứa định danh chính xác hoặc bí danh:

   - q07 (Approval Matrix): Context Recall thấp vì Dense Retrieval không nhận diện được "Approval Matrix" là bí danh (alias) của tài liệu "Access Control SOP".

   - q09 (ERR-403-AUTH): Context Recall là "None" do tìm kiếm ngữ nghĩa (Dense) không bắt được mã lỗi đặc thù, dẫn đến việc model phải trả lời "Tôi không biết".

- Đặc điểm Corpus: Bộ tài liệu là sự đan xen giữa ngôn ngữ tự nhiên (chính sách nghỉ phép, hoàn tiền) và các từ khóa kỹ thuật có độ chính xác cao (mã lỗi IT, nhãn SLA P1, tên dự án Jira).

- Mục tiêu: Sử dụng Sparse Retrieval (BM25) để ưu tiên bắt chính xác các từ khóa/mã lỗi (Keyword Matching) và kết hợp với Dense Retrieval để duy trì khả năng hiểu các câu hỏi dưới dạng ngôn ngữ tự nhiên. Việc merge kết quả bằng thuật toán RRF (Reciprocal Rank Fusion) sẽ giúp tối ưu hóa cả hai tín hiệu này, từ đó cải thiện triệt để Context Recall cho các truy vấn đặc thù.

**Config thay đổi:**
```
retrieval_mode = "hybrid"   # hoặc biến khác
# Các tham số còn lại giữ nguyên như baseline
```

**Scorecard Variant 1:**
| Metric | Baseline | Variant 1 | Delta |
|--------|----------|-----------|-------|
| Faithfulness | 4.8/5 | 5/5 | +0.2 |
| Answer Relevance | 4.2/5 | 4.9/5 | +0.7 |
| Context Recall | 3.5/5 | 4.5/5 | +1.0 |
| Completeness | 3.8/5 | 4.2/5 | +0.4 |

**Nhận xét:**
> TODO: Variant 1 cải thiện ở câu nào? Tại sao?
- gq05 (Contractor Admin Access): Cải thiện rõ rệt nhất. Baseline (Dense) hoàn toàn thất bại trong việc tìm thấy thông tin tại Section 1 (Phạm vi áp dụng), dẫn đến kết luận sai. Hybrid (BM25) đã bắt đúng từ khóa "Contractor", giúp hệ thống truy xuất đúng chứng cứ và khẳng định Contractor ĐƯỢC cấp quyền.

- gq09 (Mật khẩu): Hybrid giúp trích xuất đầy đủ cả chu kỳ đổi mật khẩu và link portal SSO. Baseline thường bỏ lỡ chi tiết link portal do nó nằm ở cuối đoạn văn bản FAQ.
> Có câu nào kém hơn không? Tại sao?
- gq05 (Completeness): Mặc dù tìm thấy đúng tài liệu (Recall tốt), nhưng điểm Completeness chỉ đạt 2/5. Nguyên nhân là do tài liệu liệt kê quá nhiều thông số (Level 1-4), khiến LLM bị "nhiễu" và trích xuất nhầm thời gian xử lý 1 ngày của Level 1 áp cho Level 4 (đúng phải là 5 ngày). Đây là lỗi nhầm lẫn thuộc tính khi ngữ cảnh chứa quá nhiều dữ liệu số tương đồng.

**Kết luận:**
> TODO: Variant 1 có tốt hơn baseline không?
- Có. Đây là bước tiến quan trọng về khả năng truy xuất chứng cứ.
> Bằng chứng là gì? (điểm số, câu hỏi cụ thể)
- Về điểm số: Điểm Context Recall tăng từ 3.5 lên 4.5 và Completeness tăng từ 3.8 lên 4.2.

- Về câu hỏi cụ thể: Giải quyết được lỗi logic nghiêm trọng ở câu gq05 (về phạm vi đối tượng Contractor) và cung cấp thông tin kỹ thuật đầy đủ hơn ở câu gq09. Hệ thống hoạt động ổn định và an toàn hơn khi xử lý các truy vấn chứa từ khóa đặc thù.

---

## Variant 2 (nếu có thời gian)

**Biến thay đổi:** ___________  
**Config:**
```
# TODO
```

**Scorecard Variant 2:**
| Metric | Baseline | Variant 1 | Variant 2 | Best |
|--------|----------|-----------|-----------|------|
| Faithfulness | ? | ? | ? | ? |
| Answer Relevance | ? | ? | ? | ? |
| Context Recall | ? | ? | ? | ? |
| Completeness | ? | ? | ? | ? |

---

## Tóm tắt học được

> TODO (Sprint 4): Điền sau khi hoàn thành evaluation.

1. **Lỗi phổ biến nhất trong pipeline này là gì?**
   > Hallucination do nhiễu ngữ cảnh (Context Noise) và Nhầm lẫn thuộc tính (Attribute Confusion): Đây là lỗi phổ biến nhất, đặc biệt khi nâng cấp lên Hybrid Retrieval. Hệ thống tìm thấy rất nhiều thông tin (Recall cao) nhưng các thông tin này thường chứa các con số hoặc thuật ngữ tương tự nhau (ví dụ: các mốc thời gian 1, 3, 5 ngày cho các cấp độ quyền khác nhau). Điều này khiến LLM bị "nhiễu" và trích xuất nhầm giá trị của đối tượng này áp cho đối tượng kia (như lỗi ở câu gq05).

2. **Biến nào có tác động lớn nhất tới chất lượng?**
   > Chiến lược Retrieval (Dense vs Hybrid): Việc chuyển đổi từ chỉ dùng Vector Search (Dense) sang kết hợp với Keyword Search (BM25) là biến số có tác động mạnh nhất. Nó giúp chỉ số Context Recall tăng vọt từ mức trung bình (~3.5/5) lên mức tuyệt đối (4.5/5), giải quyết triệt để vấn đề bỏ lỡ các mã lỗi kỹ thuật, link portal hoặc các đối tượng đặc thù như "Contractor".

3. **Nếu có thêm 1 giờ, nhóm sẽ thử gì tiếp theo?**
   >Triển khai Rerank (Cross-Encoder): Để khắc phục nhược điểm "nhiễu" mà Hybrid Retrieval mang lại, bước tiếp theo chắc chắn là thêm một module Reranker. Module này sẽ chấm điểm lại độ liên quan thực tế của các chunk được lấy về, đảm bảo chỉ những đoạn văn bản sát với ý định của câu hỏi nhất (ví dụ: đúng Level 4 Admin Access) được đưa vào Prompt, từ đó cải thiện điểm Completeness và Faithfulness.
