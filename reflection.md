# Day 14 — Reflection

## Evaluation Report & Failure Analysis

Dùng kết quả thật trong `artifacts/benchmark_results.json` và kiểm tra lại
answer/context trace trong `artifacts/actual_answers.json` trước khi kết luận.

---

## 1. Benchmark Results Summary

**Overall pass rate:** 95%

| Metric | Average | Min | Max | Nhận xét |
|---|---:|---:|---:|---|
| Context Recall | 1.0 | 1.0 | 1.0 | Tốt |
| Context Precision | 1.0 | 1.0 | 1.0 | Tốt |
| Faithfulness | 0.95 | 0.0 | 1.0 | Cần cải thiện hallucination ở case H01 |
| Relevance | 1.0 | 1.0 | 1.0 | Tốt |
| Completeness | 0.97 | 0.5 | 1.0 | Thiếu ý ở H01 |
| Overall Score | 0.97 | 0.5 | 1.0 | Tốt |

**Score interpretation**

- Metrics/cases ở mức Good (0.8–1.0): 19
- Metrics/cases ở mức Needs Work (0.6–0.8): 0
- Metrics/cases ở mức Significant Issues (<0.6): 1 (H01)

**Failure type distribution**

| Failure Type | Count | Percentage |
|---|---:|---:|
| hallucination | 1 | 100% |
| irrelevant | 0 | 0% |
| incomplete | 0 | 0% |
| off_topic | 0 | 0% |
| refusal | 0 | 0% |

**Chẩn đoán tổng quan:** Vấn đề chính nằm ở retrieval, generation hay cả hai?
Dùng ít nhất hai metrics để bảo vệ kết luận.

> *Câu trả lời:*
> Vấn đề chính nằm ở generation (đặc biệt là hallucination). Retrieval đạt điểm Recall và Precision 1.0 (hoàn hảo), nghĩa là đúng thông tin đã được lấy ra, nhưng model vẫn trả lời sai (Faithfulness 0.0).

---

## 2. Top 3 Worst Failures — 5 Whys

Phân loại failure trước khi đề xuất fix. Với mỗi case, kiểm tra cả gold evidence
và retrieved chunks; không suy luận chỉ từ một score.

### Failure 1

**ID và question:**

> *Điền:* H01 - How to get a repair loaner?

**Expected answer:**

> *Điền:* Active OrbitPlus members can request it with a USD 200 deposit.

**Actual answer:**

> *Điền:* You can get a loaner if you pay a $500 fee.

**Scores:** Context Recall: 1.0 | Context Precision: 1.0 | Faithfulness: 0.0 |
Relevance: 1.0 | Completeness: 0.5 | Overall: 0.5

**Evidence inspection:** Retriever lấy đúng/thiếu/thừa chunks nào?

> *Câu trả lời:* Retriever lấy đúng chunk từ `07_repair_and_technical_support.md` ghi rõ "$200 deposit", nhưng LLM lại sinh ra "$500 fee".

| Level | Question | Answer |
|---|---|---|
| Symptom | Vấn đề quan sát được là gì? | LLM sinh ra thông tin sai lệch về khoản phí loaner. |
| Why 1 | Tại sao symptom xảy ra? | LLM phớt lờ số tiền $200 trong context và tự đưa ra số $500. |
| Why 2 | Tại sao nguyên nhân trên xảy ra? | System prompt không cấm LLM tự sáng tạo số liệu. |
| Why 3 | Tại sao vấn đề đó chưa được ngăn chặn? | Thiếu guardrails chống hallucination. |
| Why 4 | Tại sao cơ chế hiện tại chưa phát hiện hoặc xử lý được? | Prompt chưa chặt chẽ trong việc bám sát số liệu từ context. |
| Why 5 | Root cause có thể hành động được là gì? | System prompt thiếu instruction rõ ràng yêu cầu tuyệt đối tuân thủ số liệu trong context. |

**Root cause từ `find_root_cause()`:**

> *Paste output:* Context is missing or irrelevant — improve retrieval

**Bạn đồng ý hay không? Dẫn evidence từ trace:**

> *Câu trả lời:* Không đồng ý. Output gợi ý lỗi ở retrieval, nhưng Recall = 1.0, tức là context đã lấy đúng. Vấn đề thực sự là do LLM generation (hallucination).

**Proposed fix cụ thể:**

> *Câu trả lời:* Thêm câu lệnh cứng vào system prompt: "DO NOT invent any numbers, prices, or fees. Only use the EXACT values provided in the context."

### Failure 2

**ID và question:** N/A
**Expected answer:** N/A
**Actual answer:** N/A
**Scores:** Context Recall: N/A | Context Precision: N/A | Faithfulness: N/A | Relevance: N/A | Completeness: N/A | Overall: N/A
**Evidence inspection:** N/A
| Level | Question | Answer |
|---|---|---|
| Symptom | Vấn đề quan sát được là gì? | N/A |
| Why 1 | Tại sao symptom xảy ra? | N/A |
| Why 2 | Tại sao nguyên nhân trên xảy ra? | N/A |
| Why 3 | Tại sao vấn đề đó chưa được ngăn chặn? | N/A |
| Why 4 | Tại sao cơ chế hiện tại chưa phát hiện hoặc xử lý được? | N/A |
| Why 5 | Root cause có thể hành động được là gì? | N/A |
**Root cause và proposed fix:** N/A

### Failure 3

**ID và question:** N/A
**Expected answer:** N/A
**Actual answer:** N/A
**Scores:** Context Recall: N/A | Context Precision: N/A | Faithfulness: N/A | Relevance: N/A | Completeness: N/A | Overall: N/A
**Evidence inspection:** N/A
| Level | Question | Answer |
|---|---|---|
| Symptom | Vấn đề quan sát được là gì? | N/A |
| Why 1 | Tại sao symptom xảy ra? | N/A |
| Why 2 | Tại sao nguyên nhân trên xảy ra? | N/A |
| Why 3 | Tại sao vấn đề đó chưa được ngăn chặn? | N/A |
| Why 4 | Tại sao cơ chế hiện tại chưa phát hiện hoặc xử lý được? | N/A |
| Why 5 | Root cause có thể hành động được là gì? | N/A |
**Root cause và proposed fix:** N/A

---

## 3. Failure Clustering

Một root cause có thể tạo ra nhiều failures. Nhóm theo nguyên nhân có thể sửa,
không chỉ nhóm theo tên metric.

| Cluster | Root Cause | Failure IDs | Priority |
|---|---|---|---|
| 1 | Thiếu guardrails chống hallucination số liệu | H01 | High |
| 2 | N/A | N/A | Low |
| 3 | N/A | N/A | Low |

**Nếu chỉ được sửa một cluster, bạn chọn cluster nào và vì sao?**

> *Câu trả lời:*
> Cụm 1. Vì số liệu, chi phí, hoặc giá cả là những thông tin rất nhạy cảm với khách hàng. Nếu bot báo sai giá, công ty có thể bị khiếu nại. Cần thêm anti-hallucination prompt.

---

## 4. Improvement Log

Paste output của `generate_improvement_log()`:

```text
| Failure ID | Type | Root Cause | Suggested Fix | Status |
|------------|------|------------|---------------|--------|
| H01 | hallucination | Context is missing or irrelevant — improve retrieval | Add strict anti-hallucination prompt. | Open |
```

**Ba improvement suggestions ưu tiên**

1. Thêm anti-hallucination instruction vào prompt
2. Bật cờ strict-temperature (temperature=0)
3. Cài đặt output guardrail regex filter cho tiền tệ

| Suggestion | Target metric | Verification method |
|---|---|---|
| Thêm anti-hallucination instruction vào prompt | Faithfulness | Chạy lại benchmark, kiểm tra Faithfulness tăng từ 0.0 lên > 0.8 |
| Bật cờ strict-temperature (temperature=0) | Faithfulness | Chạy lại pipeline với H01 nhiều lần xem có bịa giá trị ngẫu nhiên không |
| Cài đặt output guardrail regex filter cho tiền tệ | Faithfulness | Viết unit test truyền output có chứa số tiền lạ và verify xem guardrail có chặn không |

---

## 5. Regression Testing Strategy

**Câu 1: Khi nào chạy `run_regression()` trong production workflow?**

> *Câu trả lời:*
> Trước khi merge pull request chứa thay đổi về prompt, model, hoặc index cấu trúc retriever. Nếu run_regression phát hiện drop > 0.05, build sẽ bị fail.

**Câu 2: Threshold drop 0.05 có phù hợp OrbitTech Customer Support không? Vì sao?**

> *Câu trả lời:*
> Rất phù hợp. OrbitTech Customer Support cần tính ổn định cao. Drop > 0.05 (tương đương 5%) trung bình là một suy giảm chất lượng rõ rệt trên quy mô lớn, có thể dẫn đến rất nhiều tickets hỗ trợ bị xử lý sai.

**Câu 3: Metric/failure nào phải block deployment, metric nào chỉ alert?**

> *Câu trả lời:*
> Block deployment: Faithfulness (sai số có thể gây hậu quả tài chính, pháp lý), Relevance.
> Alert: Completeness (trả lời ngắn vẫn được miễn là đúng), Context Recall (nếu vẫn giải quyết được).

**Câu 4: Điền evaluation stages vào flow.**

```text
Code/prompt/retrieval change → [Run Benchmark (Offline Eval)] → [Regression Test] → [Human Review (if marginal)] → Deploy
```

> *Giải thích:*
> Offline Eval sẽ chạy toàn bộ golden dataset để lấy các chỉ số.
> Regression Test sẽ so sánh với baseline gần nhất để đảm bảo không có suy giảm > 0.05.
> Human Review cần cho các case nằm ở ngưỡng báo động hoặc nếu thay đổi rubric.

---

## 6. Continuous Improvement Loop

```text
Evaluate → Analyze → Improve → Augment benchmark → Repeat
```

| Priority | Action | Metric dự kiến cải thiện | Expected impact |
|---:|---|---|---|
| 1 | Thêm prompt cấm bịa giá | Faithfulness | Tăng Faithfulness cho H01 từ 0.0 lên 1.0, không giảm score khác |
| 2 | Chỉnh temperature=0 | Faithfulness | Output ổn định hơn, giảm phương sai kết quả |
| 3 | Mở rộng retrieval window | Context Recall | Tăng khả năng lấy được câu phụ trong các doc dài |

**Hai hoặc ba failure cases nào cần thêm vào benchmark ở vòng tiếp theo?**

> *Câu trả lời:*
> Thêm case mồi (Prompt Injection/Adversarial) đánh lừa bot tự nhận lỗi dù nó làm đúng. Thêm một số case hỏi về combo nhiều sản phẩm (NovaBook + PulsePhone) để kiểm tra khả năng Retrieve từ nhiều documents cùng lúc.

---

## 7. Final Reflection

**Điều gì trong kết quả benchmark trái với dự đoán ban đầu của bạn?**

> *Câu trả lời:*
> Lúc đầu tôi nghĩ việc retrieval lấy đúng chunk (Context Recall = 1.0) thì model sẽ tự nhiên trả lời đúng. Tuy nhiên, kết quả chứng minh model vẫn có thể bịa đặt thông tin dù context có sẵn ở đó.

**Word-overlap heuristics trong lab có giới hạn gì? Nếu đưa hệ thống vào
production, bạn sẽ thay hoặc bổ sung metric nào?**

> *Câu trả lời:*
> Rất dễ bị đánh lừa bởi cách paraphrase. Một từ trái nghĩa có thể có word overlap 90% nhưng ý nghĩa hoàn toàn sai. Trong production, tôi sẽ bổ sung LLM-as-a-judge có rubric chặt chẽ, và G-Eval framework để đánh giá ngữ nghĩa thay vì chỉ từ khóa.
