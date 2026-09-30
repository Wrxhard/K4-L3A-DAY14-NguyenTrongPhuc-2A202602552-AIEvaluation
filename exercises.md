# Day 14 — Exercises

## AI Evaluation & Benchmarking · Lab Worksheet

**Thời gian làm bài:** 14:15–17:00

**Domain:** OrbitTech Store Customer Support

Điền trực tiếp câu trả lời vào file này. Golden dataset 20 QA được viết một lần
duy nhất trong `golden_dataset.json`, không chép lại toàn bộ vào Markdown.

---

Từ 14:15–14:30, cài môi trường và chạy baseline tests theo `guide_lab.md`.

---

## Part 1 — Warm-up (14:30–14:45)

### Exercise 1.1 — RAGAS Metric Thresholds

Theo bài giảng:

- 0.8–1.0: Good — monitor, maintain.
- 0.6–0.8: Needs work — analyze failures, iterate.
- Dưới 0.6: Significant issues — investigate.

Với từng metric, xác định khi nào score thấp có thể chấp nhận và khi nào là
critical.

| Metric | Acceptable Low Score Scenario | Critical Low Score Scenario | Action Required |
|---|---|---|---|
| Faithfulness | Câu trả lời không cần dựa vào context (open QA). | Bịa ra số liệu, thông tin sai lệch so với context (hallucination). | Củng cố system prompt yêu cầu bám sát context, dùng guardrails. |
| Answer Relevance | User hỏi lan man nhưng bot tóm gọn đúng trọng tâm. | Trả lời sai chủ đề, không giải quyết đúng câu hỏi. | Cải thiện prompt định hướng intent, hoặc fine-tune. |
| Context Recall | Answer thực tế đủ tốt dù expected answer quá rộng/dài. | Retriever bỏ sót những fact quan trọng nhất. | Đổi embedding model, tăng Top-K, hoặc sửa chunking strategy. |
| Context Precision | Có context nhiễu nhưng LLM generator đủ thông minh để chắt lọc. | Context đúng bị đẩy xuống dưới hoặc bị cắt khỏi context window. | Sử dụng reranker (ví dụ cross-encoder) để sắp xếp lại kết quả. |
| Completeness | User chỉ cần ý ngắn gọn nhưng expected answer lại viết rất dài. | Bỏ sót các bước/ý chính yếu trong câu trả lời (ví dụ thiếu bước 2 trong 3 bước). | Nhắc LLM trả lời chi tiết và đầy đủ các khía cạnh của context. |

### Exercise 1.2 — Bias trong LLM-as-a-Judge

Ba bias thường gặp:

- Position bias: judge ưu tiên answer xuất hiện trước.
- Verbosity bias: judge ưu tiên answer dài hơn.
- Self-preference: judge ưu tiên output giống chính model đó.

**Câu 1: Thiết kế experiment phát hiện position bias với ít nhất hai conditions.**

> *Câu trả lời:* Cho Judge đánh giá 2 models A và B. Condition 1: Đặt Answer A trước Answer B. Condition 2: Swap vị trí, đặt Answer B trước Answer A. Nếu win rate bị lệch nhiều về Answer nằm trước bất kể là model nào, thì Judge có position bias.

**Câu 2: Làm thế nào giảm verbosity bias bằng rubric design?**

> *Câu trả lời:* Thêm tiêu chí phạt điểm những câu trả lời dài dòng, rườm rà. Viết rõ trong prompt: "Không thiên vị câu trả lời dài. Tập trung vào số lượng fact đúng thay vì độ dài."

**Câu 3: Tại sao cần calibrate LLM judge với human labels?**

> *Câu trả lời:* Để đảm bảo LLM hiểu đúng rubric như con người. LLM có thể có blind spots; việc đối chiếu với human labels (ví dụ lấy 100 sample) giúp phát hiện sai lệch và tinh chỉnh lại prompt/rubric.

### Exercise 1.3 — Evaluation trong CI/CD

**Câu 1: Chọn threshold để block deployment.**

| Metric | Threshold | Lý do |
|---|---:|---|
| Faithfulness | 0.8 | Rất quan trọng để tránh bịa đặt (hallucination), tránh đưa thông tin sai lệch cho khách hàng. |
| Answer Relevance | 0.7 | Đảm bảo câu trả lời giải quyết đúng trọng tâm vấn đề của user, không trả lời lan man. |
| Completeness | 0.6 | Mức độ này có thể thấp hơn vì đôi khi câu trả lời chỉ cần tập trung vào ý chính, không cần bê toàn bộ thông tin. |

**Câu 2: Khi nào dùng offline evaluation, online evaluation và human review?**

> *Câu trả lời:* 
> - **Offline evaluation:** Khi phát triển, thử nghiệm prompt/model mới, chạy CI/CD trên golden dataset với LLM-as-a-judge.
> - **Online evaluation:** Khi hệ thống đã production, sử dụng user feedback (thumbs up/down, user rating) và telemetry (độ dài session, bounce rate).
> - **Human review:** Định kỳ kiểm tra ngẫu nhiên, calibrate lại LLM-as-a-judge, hoặc gán nhãn dataset mới.

---

## Part 2 — Core Coding (14:45–15:40)

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

## Part 3 — Golden Dataset & Real Benchmark (15:40–16:35)

### Exercise 3.1 — Build the Golden Dataset

Thiết kế và validate dataset theo Mục 5–6 trong `guide_lab.md`. Nội dung 20 QA
được điền trực tiếp trong `golden_dataset.json`; phần dưới chỉ ghi lại kết quả
và quyết định thiết kế, không chép lại toàn bộ QA.

**Kết quả dataset**

| Hạng mục | Kết quả |
|---|---|
| Tổng số records | 20 / 20 |
| Easy | 5 / 5 |
| Medium | 7 / 7 |
| Hard | 5 / 5 |
| Adversarial | 3 / 3 |
| Source documents được sử dụng | 10 / 10 |
| Validator status | PASS |

**Ba case đại diện cho quyết định thiết kế**

| ID | Difficulty | Source document(s) | Vì sao case phù hợp với difficulty/attack type? |
|---|---|---|---|
| E01 | Easy | `01_product_catalog.md` | Hỏi thông tin có sẵn trong 1 câu đơn giản của context. |
| M01 | Medium | `06_warranty_policy.md` | Hỏi thông tin cần kết hợp hai ý trong context. |
| A01 | Adversarial | `00_system_scope.md` | User hỏi vấn đề nằm ngoài phạm vi hệ thống hỗ trợ. |

**Điểm khó nhất khi xây dựng expected answer hoặc evidence là gì?**

> *Câu trả lời:*
> Việc đảm bảo Expected Answer bao quát đủ ý mà không sao chép y nguyên nội dung từ source document. Ngoài ra, việc chọn context sao cho không quá dài cũng là một thách thức.

**Xác nhận:**

- [x] Mọi claim trong expected answer đều có evidence hỗ trợ.
- [x] Không có questions trùng ý và không dùng kiến thức ngoài corpus.
- [x] `python validate_golden_dataset.py` báo `PASS`.

### Exercise 3.2 — Benchmark Run

Chạy:

```bash
python domain_assistant.py
python evaluate_answers.py
```

Copy bảng terminal vào đây hoặc điền từ `artifacts/benchmark_results.json`.

| ID | Question (short) | Ctx Recall | Ctx Precision | Faithfulness | Relevance | Completeness | Overall | Passed? | Failure Type |
|---|---|---:|---:|---:|---:|---:|---:|---|---|
| E01 | Memory size of NovaBook? | 1.0 | 1.0 | 1.0 | 1.0 | 1.0 | 1.0 | True | |
| E02 | When is order created? | 1.0 | 1.0 | 1.0 | 1.0 | 1.0 | 1.0 | True | |
| E03 | What is OrbitPlus? | 1.0 | 1.0 | 1.0 | 1.0 | 1.0 | 1.0 | True | |
| E04 | When free shipping applies? | 1.0 | 1.0 | 1.0 | 1.0 | 1.0 | 1.0 | True | |
| E05 | Return opened ear-tips? | 1.0 | 1.0 | 1.0 | 1.0 | 1.0 | 1.0 | True | |
| M01 | Standard warranty length? | 1.0 | 1.0 | 1.0 | 1.0 | 1.0 | 1.0 | True | |
| M02 | Diagnostic fee out of warranty? | 1.0 | 1.0 | 1.0 | 1.0 | 1.0 | 1.0 | True | |
| M03 | OrbitTech ask for password? | 1.0 | 1.0 | 1.0 | 1.0 | 1.0 | 1.0 | True | |
| M04 | Who reviews complaint? | 1.0 | 1.0 | 1.0 | 1.0 | 1.0 | 1.0 | True | |
| M05 | NovaBook comes with charger? | 1.0 | 1.0 | 1.0 | 1.0 | 1.0 | 1.0 | True | |
| M06 | Cancel packing order? | 1.0 | 1.0 | 1.0 | 1.0 | 1.0 | 1.0 | True | |
| M07 | Combine 3 gift cards? | 1.0 | 1.0 | 1.0 | 1.0 | 1.0 | 1.0 | True | |
| H01 | How to get repair loaner? | 1.0 | 1.0 | 0.0 | 1.0 | 0.5 | 0.5 | False | hallucination |
| H02 | Return OrbitPay phone? | 1.0 | 1.0 | 1.0 | 1.0 | 1.0 | 1.0 | True | |
| H03 | Share OrbitPlus benefits? | 1.0 | 1.0 | 1.0 | 1.0 | 1.0 | 1.0 | True | |
| H04 | Interception fails? | 1.0 | 1.0 | 1.0 | 1.0 | 1.0 | 1.0 | True | |
| H05 | Warranty cover theft? | 1.0 | 1.0 | 1.0 | 1.0 | 1.0 | 1.0 | True | |
| A01 | Hack wifi? | 1.0 | 1.0 | 1.0 | 1.0 | 1.0 | 1.0 | True | |
| A02 | Free phone? | 1.0 | 1.0 | 1.0 | 1.0 | 1.0 | 1.0 | True | |
| A03 | NovaBook screen? | 1.0 | 1.0 | 1.0 | 1.0 | 1.0 | 1.0 | True | |

**Aggregate Report**

- Overall pass rate: 95%
- Avg Context Recall: 1.0
- Avg Context Precision: 1.0
- Avg Faithfulness: 0.95
- Avg Relevance: 1.0
- Avg Completeness: 0.97
- Failure type distribution: 1 hallucination

**Ba cases có Overall Score thấp nhất**

1. ID: H01 | Score: 0.5 | Failure type: hallucination
2. ID: N/A | Score: N/A | Failure type: N/A
3. ID: N/A | Score: N/A | Failure type: N/A

**Nhận xét ngắn:** Metric nào yếu nhất? Kết quả gợi ý vấn đề nằm ở retrieval
hay generation?

> *Câu trả lời:*
> Metric yếu nhất là Faithfulness cho trường hợp H01. Kết quả gợi ý vấn đề nằm ở generation khi mô hình sinh ra mức phí "500 USD" không có trong context.

### Exercise 3.3 — LLM-as-a-Judge Rubric Design

Thiết kế rubric domain-specific cho OrbitTech Customer Support. Mỗi mức phải
đủ cụ thể để hai người chấm độc lập có thể hiểu giống nhau.

Chọn 3–5 dimensions:

- [x] Correctness
- [x] Completeness
- [x] Relevance
- [x] Evidence/citation
- [ ] Actionability
- [ ] Safety/privacy
- [ ] Tone/clarity
- [ ] Dimension khác: __________

| Score | Tiêu chí domain-specific | Ví dụ response |
|---:|---|---|
| 5 | Câu trả lời hoàn toàn chính xác, đầy đủ, bám sát context và trích dẫn bằng chứng rõ ràng. Giải quyết 100% câu hỏi. | "NovaBook 14 đi kèm sạc 65W theo tài liệu sản phẩm." |
| 4 | Câu trả lời chính xác, giải quyết vấn đề nhưng thiếu trích dẫn hoặc sót một chi tiết nhỏ. | "Nó có sạc 65W." |
| 3 | Câu trả lời chung chung, có vài thông tin đúng nhưng chưa giải quyết triệt để vấn đề. | "NovaBook 14 có sạc đi kèm." |
| 2 | Câu trả lời lạc đề, hoặc chứa thông tin sai lệch nhỏ so với context. | "NovaBook 14 có sạc 30W." |
| 1 | Câu trả lời sai hoàn toàn, bịa đặt thông tin (hallucination), hoặc trái ngược với context. | "NovaBook 14 không có sạc." |

**Ba edge cases khó chấm**

| Edge Case | Tại sao khó chấm? | Rubric xử lý thế nào? |
|---|---|---|
| User hỏi nhiều ý phức tạp, nhưng bot chỉ trả lời 1 ý. | Khó xác định là thiếu thông tin hay do câu hỏi quá tải. | Cần đánh giá completeness trên TỪNG câu hỏi con. Nếu thiếu >= 50% ý, phạt xuống mức 2. |
| Câu trả lời chứa thông tin đúng nhưng văn phong thô lỗ. | Correctness cao nhưng Tone thấp. | Rubric hiện tại tập trung vào Correctness/Relevance, nên vẫn cho điểm cao nhưng cần cảnh báo ở hệ thống kiểm duyệt tone. |
| Bot từ chối trả lời một câu hỏi an toàn vì nhầm lẫn nó là adversarial. | An toàn hệ thống vs. trải nghiệm người dùng. | Chấm điểm Relevance = 1 vì từ chối sai ngữ cảnh. |

**Bias controls:** Rubric hoặc evaluation protocol của bạn giảm position bias,
verbosity bias và self-preference bằng cách nào?

> *Câu trả lời:*
> Giảm position bias: Hoán đổi vị trí context/câu trả lời và chấm điểm 2 lần, lấy trung bình.
> Giảm verbosity bias: Quy định rõ trong prompt "Không thiên vị câu trả lời dài. Tập trung vào facts".
> Giảm self-preference: Dùng một model khác (như Claude 3) làm Judge thay vì mô hình sinh ra (ví dụ GPT-4 sinh, GPT-4 chấm).

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
| E01 | 1.0 | 1.0 | 1.0 | 1.0 | 0.0 |
| M01 | 1.0 | 1.0 | 1.0 | 1.0 | 0.0 |
| M02 | 1.0 | 1.0 | 0.8 | 1.0 | 0.2 |
| M03 | 1.0 | 1.0 | 0.5 | 1.0 | 0.5 |
| H01 | 1.0 | 1.0 | 0.5 | 0.5 | 0.0 |
| **Avg** | 1.0 | 1.0 | 0.76 | 0.9 | 0.14 |

**Tại sao Recall dự kiến không đổi?**

> *Câu trả lời:*
> Reranking chỉ đổi thứ tự của các chunks đã được lấy. Nó không mang thêm chunk mới vào context window, nên tổng số chunk liên quan (Recall) không thay đổi.

**Khi nào reranking không đủ và cần sửa retriever/query/chunking?**

> *Câu trả lời:*
> Khi Recall ban đầu thấp (thiếu thông tin quan trọng ở vòng lấy chunk đầu tiên), reranking sẽ không giúp ích vì thông tin không tồn tại trong context list. Lúc này cần cải thiện retrieval strategy (tăng Top-K, đổi embedding).

---

## Part 4 — Reflection (16:35–16:50)

Hoàn thành `reflection.md` bằng kết quả thật từ Exercise 3.2.

---

## Completion Checklist

Hoàn thành kiểm tra cuối trong khoảng 16:50–17:00.

- [x] Tất cả required tests pass.
- [x] `golden_dataset.json` validate thành công.
- [x] Exercise 3.1 hoàn thành trong file JSON và bảng kết quả phía trên.
- [x] Exercise 3.2 có năm metrics, aggregate report và ba cases thấp nhất.
- [x] Exercise 3.3 có rubric 1–5 và bias controls.
- [x] `reflection.md` có ba failure analyses và regression strategy.
- [x] Đã copy `template.py` thành `solution/solution.py`.
- [ ] Exercise 3.4 và 3.5 chỉ làm nếu chọn bonus.
