# Day 14 — Exercises

## AI Evaluation & Benchmarking · Lab Worksheet

**Thời gian làm bài:** 9:15–12:00

**Domain:** OrbitTech Store Customer Support

Điền trực tiếp câu trả lời vào file này. Golden dataset 20 QA được viết một lần
duy nhất trong `golden_dataset.json`, không chép lại toàn bộ vào Markdown.

---

Từ 9:15–9:30, cài môi trường và chạy baseline tests theo `guide_lab.md`.

---

## Part 1 — Warm-up (9:30–9:45)

### Exercise 1.1 — RAGAS Metric Thresholds

Theo bài giảng:

- 0.8–1.0: Good — monitor, maintain.
- 0.6–0.8: Needs work — analyze failures, iterate.
- Dưới 0.6: Significant issues — investigate.

Với từng metric, xác định khi nào score thấp có thể chấp nhận và khi nào là
critical.

| Metric | Acceptable Low Score Scenario | Critical Low Score Scenario | Action Required |
|---|---|---|---|
| Faithfulness | Score thấp do câu trả lời có thêm kiến thức phổ biến hoặc diễn giải ngoài context nhưng vẫn đúng. | Model tạo thông tin không có trong context, hallucination hoặc mâu thuẫn với tài liệu nguồn. | Kiểm tra grounding, prompt và retrieved context; yêu cầu model trả lời dựa trên context. |
| Answer Relevance | Câu trả lời đúng nhưng có thêm giải thích, ví dụ hoặc thông tin phụ không cần thiết. | Câu trả lời lệch câu hỏi, không giải quyết user intent hoặc sai chủ đề. | Tối ưu prompt, làm rõ query/user intent và giảm nội dung không liên quan. |
| Context Recall | Một số tài liệu liên quan không được retrieve nhưng context hiện tại vẫn đủ để trả lời đúng. | Retriever bỏ sót thông tin quan trọng khiến model không thể trả lời đúng hoặc đầy đủ. | Cải thiện retrieval: embedding, chunking, `top-k`, query rewriting hoặc hybrid search. |
| Context Precision | Retrieve thêm một số chunk không liên quan nhưng các chunk cần thiết vẫn xuất hiện. | Phần lớn context không liên quan, gây nhiễu hoặc làm thông tin quan trọng bị loại khỏi context window. | Cải thiện ranking/reranking, giảm `top-k`, cải thiện embedding và filtering. |
| Completeness | Thiếu một vài chi tiết phụ nhưng vẫn trả lời đầy đủ phần chính của câu hỏi. | Bỏ sót các ý hoặc thông tin thiết yếu khiến câu trả lời không đáp ứng yêu cầu. | Kiểm tra context có đủ dữ liệu không; cải thiện retrieval và prompt để bao phủ đầy đủ các ý. |

### Exercise 1.2 — Bias trong LLM-as-a-Judge

Ba bias thường gặp:

- Position bias: judge ưu tiên answer xuất hiện trước.
- Verbosity bias: judge ưu tiên answer dài hơn.
- Self-preference: judge ưu tiên output giống chính model đó.

**Câu 1: Thiết kế experiment phát hiện position bias với ít nhất hai conditions.**

> Sử dụng cùng hai câu trả lời **A** và **B**, nhưng thay đổi thứ tự:
>
> - **Condition 1:** A xuất hiện trước, B xuất hiện sau → Judge chọn answer tốt hơn.
> - **Condition 2:** B xuất hiện trước, A xuất hiện sau → Judge tiếp tục đánh giá.
>
> Nếu kết quả thay đổi đáng kể khi chỉ đổi thứ tự A/B, ví dụ A thắng khi đứng trước nhưng B thắng khi B đứng trước, thì có dấu hiệu **position bias**.
>
> Có thể đo bằng **flip rate**:
>
> `Flip Rate = Số lần kết quả thay đổi khi đảo vị trí / Tổng số cặp đánh giá`
>
> Flip rate càng cao → position bias càng lớn.


**Câu 2: Làm thế nào giảm verbosity bias bằng rubric design?**

> Thiết kế rubric tập trung vào **chất lượng thay vì độ dài**, ví dụ:
>
> - Chấm riêng **correctness**, **relevance**, **faithfulness** và **completeness**.
> - Không cộng điểm chỉ vì answer dài hoặc chi tiết hơn.
> - Quy định rõ thông tin dư thừa, lặp lại hoặc không liên quan **không làm tăng điểm**.
> - Có thể thêm tiêu chí **conciseness** để phạt nội dung dài dòng không cần thiết.
>
> Như vậy, một answer ngắn nhưng đúng và đầy đủ vẫn có thể đạt điểm cao hơn answer dài nhưng chứa nhiều thông tin thừa.


**Câu 3: Tại sao cần calibrate LLM judge với human labels?**

> Cần calibrate LLM judge với **human labels** để kiểm tra điểm của LLM có phù hợp với đánh giá của con người hay không.
>
> Human labels đóng vai trò **reference/gold standard**, giúp:
>
> - Phát hiện các bias của LLM judge.
> - Kiểm tra mức độ tương quan giữa LLM và human evaluation.
> - Điều chỉnh prompt, rubric hoặc scoring threshold.
> - Tránh trường hợp judge cho điểm cao nhưng con người đánh giá answer có chất lượng thấp.
>
> Nếu mức độ agreement giữa LLM judge và human labels thấp, cần cải thiện hoặc calibrate lại judge trước khi sử dụng để đánh giá hệ thống ở quy mô lớn.

### Exercise 1.3 — Evaluation trong CI/CD

**Câu 1: Chọn threshold để block deployment.**

| Metric | Threshold | Lý do |
|---|---:|---|
| Faithfulness | < 0.8 → Block | Faithfulness thấp cho thấy answer có nguy cơ không dựa trên context hoặc hallucination. Đây là metric quan trọng nên cần threshold cao. |
| Answer Relevance | < 0.7 → Block | Score thấp cho thấy answer không giải quyết đúng câu hỏi hoặc user intent. Mức 0.7 cho phép một lượng nhỏ thông tin phụ nhưng vẫn yêu cầu answer đủ liên quan. |
| Completeness | < 0.7 → Block | Score thấp cho thấy answer bỏ sót nhiều thông tin quan trọng. Có thể chấp nhận thiếu một số chi tiết phụ nhưng không được thiếu nội dung chính. |

**Câu 2: Khi nào dùng offline evaluation, online evaluation và human review?**


> - Offline evaluation: Dùng trước deployment hoặc khi phát triển model/pipeline. Chạy trên dataset/test set cố định để so sánh các phiên bản, prompt, model hoặc RAG pipeline.
>
> - Online evaluation: Dùng sau deployment trên dữ liệu và tương tác thực tế của người dùng. Giúp theo dõi performance, phát hiện regression và các failure case trong production.
>
> - Human review: Dùng khi cần đánh giá những trường hợp khó, mơ hồ hoặc có rủi ro cao, đặc biệt khi automated metrics hoặc LLM judge không đủ đáng tin cậy.
>

---

## Part 2 — Core Coding (9:45–10:40)

Hoàn thiện các TODO bắt buộc trong `template.py`.

### Task 1 — Data Models

- `QAPair`: question, expected answer, gold context, metadata và retrieved contexts.
- `EvalResult`: answer-side scores, optional retrieval scores, pass/failure fields.
- `overall_score()`: trung bình Faithfulness, Relevance và Completeness.

### Task 2 — RAGASEvaluator

Answer-side:

- `evaluate_faithfulness(answer, context)`
- `evaluate_relevance(answer, question)`
- `evaluate_completeness(answer, expected)`

Retrieval-side:

- `evaluate_context_recall(contexts, expected)`
- `evaluate_context_precision(contexts, expected)`

Full pipeline:

- `run_full_eval(..., contexts=None)` luôn tính ba answer metrics.
- Nếu có `contexts`, tính và lưu thêm Context Recall và Context Precision.
- Retrieval scores không làm thay đổi `overall_score()` và pass rule gốc.

### Task 3 — LLMJudge

- `score_response(question, answer, rubric)`
- `detect_bias(scores_batch)`

### Task 4 — BenchmarkRunner

- `run(qa_pairs, agent_fn, evaluator)`
- `generate_report(results)`
- `run_regression(new_results, baseline_results)`
- `identify_failures(results, threshold)`

`BenchmarkRunner.run()` phải truyền `pair.retrieved_contexts` vào
`run_full_eval()`. Report phải có average của hai retrieval metrics.

### Task 5 — FailureAnalyzer

- `categorize_failures(failures)`
- `find_root_cause(failure)`
- `generate_improvement_suggestions(failures)`
- `generate_improvement_log(failures, suggestions)`

Kiểm tra:

```bash
pytest tests/ -v
```

`rerank_by_overlap()` là TODO bonus của Exercise 3.5. Test tương ứng được skip
nếu bạn chưa làm bonus.

---

## Part 3 — Golden Dataset & Real Benchmark (10:40–11:35)

### Exercise 3.1 — Build the Golden Dataset

Thiết kế và validate dataset theo Mục 5–6 trong `guide_lab.md`. Nội dung 20 QA
được điền trực tiếp trong `golden_dataset.json`; phần dưới chỉ ghi lại kết quả
và quyết định thiết kế, không chép lại toàn bộ QA.

**Kết quả dataset**

| Hạng mục | Kết quả |
|---|---|
| Tổng số records | ____ / 20 |
| Easy | ____ / 5 |
| Medium | ____ / 7 |
| Hard | ____ / 5 |
| Adversarial | ____ / 3 |
| Source documents được sử dụng | ____ / 10 |
| Validator status | PASS / FAIL |

**Ba case đại diện cho quyết định thiết kế**

| ID | Difficulty | Source document(s) | Vì sao case phù hợp với difficulty/attack type? |
|---|---|---|---|
| | | | |
| | | | |
| | | | |

**Điểm khó nhất khi xây dựng expected answer hoặc evidence là gì?**

> *Câu trả lời:*

**Xác nhận:**

- [ ] Mọi claim trong expected answer đều có evidence hỗ trợ.
- [ ] Không có questions trùng ý và không dùng kiến thức ngoài corpus.
- [ ] `python validate_golden_dataset.py` báo `PASS`.

### Exercise 3.2 — Benchmark Run

Chạy:

```bash
python domain_assistant.py
python evaluate_answers.py
```

Copy bảng terminal vào đây hoặc điền từ `artifacts/benchmark_results.json`.

| ID | Question (short) | Ctx Recall | Ctx Precision | Faithfulness | Relevance | Completeness | Overall | Passed? | Failure Type |
|---|---|---:|---:|---:|---:|---:|---:|---|---|
| E01 | | | | | | | | | |
| E02 | | | | | | | | | |
| E03 | | | | | | | | | |
| E04 | | | | | | | | | |
| E05 | | | | | | | | | |
| M01 | | | | | | | | | |
| M02 | | | | | | | | | |
| M03 | | | | | | | | | |
| M04 | | | | | | | | | |
| M05 | | | | | | | | | |
| M06 | | | | | | | | | |
| M07 | | | | | | | | | |
| H01 | | | | | | | | | |
| H02 | | | | | | | | | |
| H03 | | | | | | | | | |
| H04 | | | | | | | | | |
| H05 | | | | | | | | | |
| A01 | | | | | | | | | |
| A02 | | | | | | | | | |
| A03 | | | | | | | | | |

**Aggregate Report**

- Overall pass rate: ____%
- Avg Context Recall: ____
- Avg Context Precision: ____
- Avg Faithfulness: ____
- Avg Relevance: ____
- Avg Completeness: ____
- Failure type distribution: ____

**Ba cases có Overall Score thấp nhất**

1. ID: ____ | Score: ____ | Failure type: ____
2. ID: ____ | Score: ____ | Failure type: ____
3. ID: ____ | Score: ____ | Failure type: ____

**Nhận xét ngắn:** Metric nào yếu nhất? Kết quả gợi ý vấn đề nằm ở retrieval
hay generation?

> *Câu trả lời:*

### Exercise 3.3 — LLM-as-a-Judge Rubric Design

Thiết kế rubric domain-specific cho OrbitTech Customer Support. Mỗi mức phải
đủ cụ thể để hai người chấm độc lập có thể hiểu giống nhau.

Chọn 3–5 dimensions:

- [ ] Correctness
- [ ] Completeness
- [ ] Relevance
- [ ] Evidence/citation
- [ ] Actionability
- [ ] Safety/privacy
- [ ] Tone/clarity
- [ ] Dimension khác: __________

| Score | Tiêu chí domain-specific | Ví dụ response |
|---:|---|---|
| 5 | | |
| 4 | | |
| 3 | | |
| 2 | | |
| 1 | | |

**Ba edge cases khó chấm**

| Edge Case | Tại sao khó chấm? | Rubric xử lý thế nào? |
|---|---|---|
| | | |
| | | |
| | | |

**Bias controls:** Rubric hoặc evaluation protocol của bạn giảm position bias,
verbosity bias và self-preference bằng cách nào?

> *Câu trả lời:*

### Exercise 3.4 — Framework Comparison (Bonus +5)

Chỉ làm sau khi hoàn thành 3.1–3.3. Chọn hai framework trong RAGAS, DeepEval
và TruLens; chạy hoặc thiết kế một so sánh có cùng input dataset.

| Tiêu chí | Framework 1: ____ | Framework 2: ____ |
|---|---|---|
| Setup complexity | | |
| Metrics available | | |
| CI/CD integration | | |
| Kết quả trên cùng dataset | | |
| Insight rút ra | | |

- Scores có nhất quán không?
- Framework nào strict hơn và vì sao?
- Hai framework có tìm ra cùng failure cases không?

> *Phân tích:*

### Exercise 3.5 — Retrieval Reranking (Bonus +5)

Mục tiêu: kiểm tra việc đổi thứ tự chunks có tăng Context Precision mà không
thay đổi Context Recall hay không.

1. Chọn ít nhất 5 cases từ `artifacts/actual_answers.json`.
2. Tính Context Recall và Context Precision trước rerank.
3. Implement `rerank_by_overlap()` hoặc một reranker khác.
4. Rerank cùng tập chunks, không thêm hoặc xóa chunk.
5. Tính lại hai metrics và giải thích kết quả.

| ID | Recall before | Recall after | Precision before | Precision after | Delta Precision |
|---|---:|---:|---:|---:|---:|
| | | | | | |
| | | | | | |
| | | | | | |
| | | | | | |
| | | | | | |
| **Avg** | | | | | |

**Tại sao Recall dự kiến không đổi?**

> *Câu trả lời:*

**Khi nào reranking không đủ và cần sửa retriever/query/chunking?**

> *Câu trả lời:*

---

## Part 4 — Reflection (11:35–11:50)

Hoàn thành `reflection.md` bằng kết quả thật từ Exercise 3.2.

---

## Completion Checklist

Hoàn thành kiểm tra cuối trong khoảng 11:50–12:00.

- [ ] Tất cả required tests pass.
- [ ] `golden_dataset.json` validate thành công.
- [ ] Exercise 3.1 hoàn thành trong file JSON và bảng kết quả phía trên.
- [ ] Exercise 3.2 có năm metrics, aggregate report và ba cases thấp nhất.
- [ ] Exercise 3.3 có rubric 1–5 và bias controls.
- [ ] `reflection.md` có ba failure analyses và regression strategy.
- [ ] Đã copy `template.py` thành `solution/solution.py`.
- [ ] Exercise 3.4 và 3.5 chỉ làm nếu chọn bonus.
