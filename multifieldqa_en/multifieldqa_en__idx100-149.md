# LongBench-Cite 与 SQuALITY（本批 preds + gpt4o cite）：引用跨度 logits / au·eu / 方差 / GPT 与候选句
> 数据来源：`/share/home/zhengky/knowledge-provenance-demo/LongCite-main/LongBench-Cite/scores_local/longbench_full_plus_squality_preds_with_cite_text.json` + `/share/home/zhengky/knowledge-provenance-demo/LongCite-main/LongBench-Cite/scores_local/merged_gpt4o_cite_for_report.json`。 共 **1135** 条（cite 中含 `citation_f1`；bench `context` 来自 LongBench-Cite.json 或 SQuALITY dev/test.jsonl）。


## 阅读说明（量表与表格列）

- **答案分 `raw`**：来自 `scores_correct/*.json` 中 `gpt_usage.responses`，由 `eval_correct.py` 生成。 **multifieldqa_en** 走 `gpt_score_qa`：每条参考输出 **`[[1]]`–`[[3]]`**（问答正确性/覆盖面）。 **squality** 走 `gpt_score_summ`：每条参考输出 **`[[1]]`–`[[5]]`**（摘要：正确性、覆盖、**连贯性**、压缩效率等）。
  - 因此：SQuALITY 若出现 `[[3]]`、`[[2]]`，应读作 **3/5、2/5**，**不要**当成问答量表里的「满分 3 分」。
- **`normalized_0_1`**：与 `eval_correct` 写回的 `score` 一致；摘要侧单条参考为 `(rating−1)/4` 再与多参考聚合（见 `eval_correct.process`）。
- **候选句表「解析句号 / 该句内容」**：把 top 数字 token 当作句编号 sid，在 **bench 文档**经 `ensure_longcite_indexed_context` 后的 `<Csid>` 上查句（默认**全文**，不截断）。 若某 sid 下文字与 gold 引用段落不一致，多为**切句索引与当时生成引用不一致**。
- **logits 高 ≠ 语义相关**：表内 logit 是模型在写 `<cite>[` 之后、对**下一个数字 token** 的偏好（句编号在词表里的相对强度），**不是**「该句与当前 statement 是否相关」的分数，因此排名靠前的 sid 完全可能与 statement 无关。
- **同一句跨多行**：CoF 分句里某段可含换行（如 `Fans` 独立成行 + 下一段正文）。`ensure_longcite_indexed_context` 打成 `<C29> Fans` 与续行后，旧版按「每行一条」解析会把续行丢掉，表里只剩「Fans」；现已用 `parse_indexed_context` 把续行并入同一 sid，该句内容与分句结果一致。
- **GPT 引用/答案分**：由 `eval_gpt_cite_json` / `eval_correct` 另路评判，与上表 logits **不是同一套量**；cite 支持度高也不代表数字 logits 在「选句」上合理。
- **无 logits 表 / 无「GPT 引用 k」**：logits 表只来自模型在该陈述的 `<cite>` 里是否写出 **`[sid-sid]`**；`<cite></cite>` 或无可解析跨度时 pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，故无跨度块。 **`GPT 引用 k`** 来自 cite JSON 的 `statement[].citation[]`；若列表为空则不会列出。 **`support_output` 里的 Need Citation: Yes** 是裁判文案，**不**保证已有结构化 `citation[]`，也**不**表示模型已写句编号。
- **`cite_spans_logits_refit`**：若 pred 行含该字段（如 `method=teacher_force_prefix`），则表中「首数字 top logits」为 **`refit_multi_span_cite_logits_teacher_force.py`** 在冻结首数字前整段上文后重算；否则仍为生成时逐步记录的 logits。
- **本跨度合并正文**：**仅**在真正跨多句的 `[a-b]`（**a < b**）下展示，供人工核对多句合并引用；`[a-a]` 单句跨度不展示（候选表里已有该句全文）。正文来自 `citation[].cite`（与 `merge_citations` 一致）或 bench 句表按 sid 一段一段用空行隔开；在 MD 里用**引用块 + 固定列宽折行**，避免整段压成一行左右拖动。与下方「候选 token → 单句」表不同。



> 本文件为 `two_samples_cite_span_report.md` 的 **multifieldqa_en** 子集（共 150 条样本），其余数据集见同目录下其它 md。


> 本文件为 **multifieldqa_en** 子集第 **2/3** 部分，样本 `idx` ∈ [100, 149]（共 50 条）。 同数据集另见本目录下其余分片（文件名形如：数据集名__idx起止范围.md）。

## multifieldqa_en_idx100 (idx=100, multifieldqa_en)
### 问题
When did Born resign as chairperson of the CFTC?

### 参考答案（数据集）
- ref1: June 1, 1999.

### 模型全文输出
```
<statement>Based on the information provided, Brooksley Born resigned as chairperson of the Commodity Futures Trading Commission (CFTC) on June 1, 1999.<cite>[3-3][62-62]</cite></statement>
```

文档句索引范围: sid ∈ [1, 88], 共 88 句。
引用指标: R=1.0, P=1.0, F1=1.0
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=1089, completion_tokens=19
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
Based on the information provided, Brooksley Born resigned as chairperson of the Commodity Futures Trading Commission (CFTC) on June 1, 1999.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[3-3]`
- au=0.6819406747817993, eu=0.1826922744512558, au′=0.6819398403167725, eu′=0.04279278963804245
- top_logits 方差 (pvariance, 有限 logit): 0.1558174915262498
- top_logits 数值: `[22.763158798217773, 21.973684310913086]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 3 | 3 | 22.763158798217773 | During her tenure on the CFTC, Born lobbied Congress and the President to give the CFTC oversight of off-exchange markets for derivatives, in addition to its role with respect to exchange-traded derivatives, but her warnings were ignored or dismissed, and her calls for reform resisted by other regulators.<ref name="nytimes">Goodman, Peter S. The Reckoning - Taking Hard New Look at a Greenspan Legacy, The New York Times, October 9, 2008.</ref> Born resigned as chairperson on June 1, 1999, shortly after Congress passed legislation prohibiting her agency from regulating derivatives. |
| 2 | 1 | 1 | 21.973684310913086 | Brooksley Elizabeth Born (born August 27, 1940) is an American attorney and former public official who, from August 26, 1996, to June 1, 1999, was chair of the Commodity Futures Trading Commission (CFTC), the federal agency which oversees the U.S. |

**跨度** `[62-62]`
- au=1.5910399042695644e-06, eu=0.28148138523101807, au′=-0.0, eu′=0.0376984067261219
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.526317596435547]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 62 | 62 | 25.526317596435547 | Born resigned on June 1, 1999. |
- GPT 引用1: span=[3, 3], relevant=1
- GPT 引用2: span=[62, 62], relevant=1

## multifieldqa_en_idx101 (idx=101, multifieldqa_en)
### 问题
Which orders did Mufti-e-Azam-e-Hind receive Khilafat from?

### 参考答案（数据集）
- ref1: Mufti-e-Azam-e-Hind received Khilafat in the Qaderi, Chishti, Nakshbandi, Suharwardi, and Madaari Orders.

### 模型全文输出
```
<statement>Based on the passage, Mufti-e-Azam-e-Hind received Khilafat from the following orders:<cite></cite></statement>

<statement>- Qaderi Silsila (Order)<cite>[20-20]</cite></statement>

<statement>- Chishti Silsila (Order)<cite>[20-20]</cite></statement>

<statement>- Nakshbandi Silsila (Order)<cite>[20-20]</cite></statement>

<statement>- Suharwardi Silsila (Order)<cite>[20-20]</cite></statement>

<statement>- Madaari Silsila (Order)<cite>[20-20]</cite></statement>

<statement>So in total, he received Khilafat from 5 different Sufi orders. The passage states: "Mufti-e-Azam-e-Hind (radi Allahu anhu) also received Khilafat from his blessed father, A'la Hazrat, Ash Shah Imam Ahmed Raza Khan Al Qaderi (radi Allahu anhu) and not only did he receive Khilafat in the Qaderi Silsila (Order), but also in the Chishti, Nakshbandi, Suharwardi, and Madaari Orders."<cite>[20-21]</cite></statement>
```

文档句索引范围: sid ∈ [1, 423], 共 423 句。
引用指标: R=1.0, P=1.0, F1=1.0
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=4405, completion_tokens=88
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
Based on the passage, Mufti-e-Azam-e-Hind received Khilafat from the following orders:
- GPT support_score=1; [[No]]   Need Citation: [[No]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 1
- Qaderi Silsila (Order)
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[20-20]`
- au=1.9009210063813953e-06, eu=0.2957197427749634, au′=-0.0, eu′=0.04029691964387894
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.8157901763916]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 20 | 20 | 23.8157901763916 | Not only did he receive Khilafat in the Qaderi Silsila (Order), but also in the Chishti, Nakshbandi, Suharwardi, and Madaari Orders. |
- GPT 引用1: span=[20, 20], relevant=1

### 陈述 2
- Chishti Silsila (Order)
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[20-20]`
- au=1.5557670849375427e-06, eu=0.27436816692352295, au′=-0.0, eu′=0.03643336147069931
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[26.447368621826172]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 20 | 20 | 26.447368621826172 | Not only did he receive Khilafat in the Qaderi Silsila (Order), but also in the Chishti, Nakshbandi, Suharwardi, and Madaari Orders. |
- GPT 引用1: span=[20, 20], relevant=1

### 陈述 3
- Nakshbandi Silsila (Order)
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[20-20]`
- au=1.5318007626774488e-06, eu=0.26950347423553467, au′=-0.0, eu′=0.03558051958680153
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[27.10526466369629]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 20 | 20 | 27.10526466369629 | Not only did he receive Khilafat in the Qaderi Silsila (Order), but also in the Chishti, Nakshbandi, Suharwardi, and Madaari Orders. |
- GPT 引用1: span=[20, 20], relevant=1

### 陈述 4
- Suharwardi Silsila (Order)
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[20-20]`
- au=1.7844809008238371e-06, eu=0.2724013328552246, au′=-0.0, eu′=0.036087363958358765
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[26.710527420043945]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 20 | 20 | 26.710527420043945 | Not only did he receive Khilafat in the Qaderi Silsila (Order), but also in the Chishti, Nakshbandi, Suharwardi, and Madaari Orders. |
- GPT 引用1: span=[20, 20], relevant=1

### 陈述 5
- Madaari Silsila (Order)
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[20-20]`
- au=1.5606800616296823e-06, eu=0.2753622531890869, au′=-0.0, eu′=0.03660885989665985
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[26.3157901763916]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 20 | 20 | 26.3157901763916 | Not only did he receive Khilafat in the Qaderi Silsila (Order), but also in the Chishti, Nakshbandi, Suharwardi, and Madaari Orders. |
- GPT 引用1: span=[20, 20], relevant=1

### 陈述 6
So in total, he received Khilafat from 5 different Sufi orders. The passage states: "Mufti-e-Azam-e-Hind (radi Allahu anhu) also received Khilafat from his blessed father, A'la Hazrat, Ash Shah Imam Ahmed Raza Khan Al Qaderi (radi Allahu anhu) and not only did he receive Khilafat in the Qaderi Silsila (Order), but also in the Chishti, Nakshbandi, Suharwardi, and Madaari Orders."
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[20-21]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> Not only did he receive Khilafat in the Qaderi Silsila (Order), but also in the Chishti,
> Nakshbandi, Suharwardi, and Madaari Orders. Mufti-e-Azam-e-Hind (radi Allahu anhu) also
> received Khilafat from his blessed father, A'la Hazrat, Ash Shah Imam Ahmed Raza Khan Al
> Qaderi (radi Allahu anhu).
- au=1.650961507948523e-06, eu=0.2934361696243286, au′=-0.0, eu′=0.03987407684326172
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.078948974609375]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 20 | 20 | 24.078948974609375 | Not only did he receive Khilafat in the Qaderi Silsila (Order), but also in the Chishti, Nakshbandi, Suharwardi, and Madaari Orders. |
- GPT 引用1: span=[20, 21], relevant=1

## multifieldqa_en_idx102 (idx=102, multifieldqa_en)
### 问题
What was the reason given by Governor Rick Scott for not implementing a prescription drug monitoring database in Florida?

### 参考答案（数据集）
- ref1: Privacy concerns and skepticism about its effectiveness.

### 模型全文输出
```
<statement>Based on the article, Governor Rick Scott gave the following reasons for not implementing a prescription drug monitoring database in Florida:<cite></cite></statement>

<statement>- Privacy concerns - Scott cited privacy concerns as a reason for not implementing the database. He felt it would infringe on the privacy of patients and doctors.<cite>[291-291]</cite></statement>

<statement>- Skepticism about effectiveness - Scott expressed skepticism that the monitoring program would be effective in curbing prescription drug abuse. He felt it would be too costly and burdensome to operate.<cite>[291-291]</cite></statement>

<statement>- Cost concerns - Scott raised the possibility that taxpayers would be left with a $500,000-a-year bill to operate the database, which he felt was too expensive.<cite>[291-291]</cite></statement>

<statement>- Abolishing Office of Drug Control - Scott abolished Florida's Office of Drug Control, which included the state drug czar position, soon after taking office in 2011. This eliminated the state's top drug policy position and further hindered efforts to implement a prescription drug monitoring database.<cite>[290-290]</cite></statement>

<statement>So in summary, Scott cited privacy, effectiveness, cost, and abolishing the Office of Drug Control as reasons for not implementing the database. His reasons were largely political and focused on cost and privacy concerns rather than the need to combat prescription drug abuse.<cite></cite></statement>
```

文档句索引范围: sid ∈ [1, 313], 共 313 句。
引用指标: R=0.5, P=1.0, F1=0.6666666666666666
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=3653, completion_tokens=76
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
Based on the article, Governor Rick Scott gave the following reasons for not implementing a prescription drug monitoring database in Florida:
- GPT support_score=1; [[No]]   Need Citation: [[No]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 1
- Privacy concerns - Scott cited privacy concerns as a reason for not implementing the database. He felt it would infringe on the privacy of patients and doctors.
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[291-291]`
- au=1.5962523320922628e-06, eu=0.28252777457237244, au′=-0.0, eu′=0.03788633644580841
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.394737243652344]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 291 | 291 | 25.394737243652344 | Scott, a tea party conservative, cited privacy concerns, expressed skepticism the monitoring program would work and raised the possibility taxpayers would be left with a $500,000-a-year bill to operate it. |
- GPT 引用1: span=[291, 291], relevant=1

### 陈述 2
- Skepticism about effectiveness - Scott expressed skepticism that the monitoring program would be effective in curbing prescription drug abuse. He felt it would be too costly and burdensome to operate.
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[291-291]`
- au=1.4823438050370896e-06, eu=0.2593856155872345, au′=-0.0, eu′=0.033837929368019104
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[28.55263328552246]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 291 | 291 | 28.55263328552246 | Scott, a tea party conservative, cited privacy concerns, expressed skepticism the monitoring program would work and raised the possibility taxpayers would be left with a $500,000-a-year bill to operate it. |
- GPT 引用1: span=[291, 291], relevant=1

### 陈述 3
- Cost concerns - Scott raised the possibility that taxpayers would be left with a $500,000-a-year bill to operate the database, which he felt was too expensive.
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[291-291]`
- au=1.707995238575677e-06, eu=0.2567567229270935, au′=-0.0, eu′=0.033391911536455154
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[28.947368621826172]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 291 | 291 | 28.947368621826172 | Scott, a tea party conservative, cited privacy concerns, expressed skepticism the monitoring program would work and raised the possibility taxpayers would be left with a $500,000-a-year bill to operate it. |
- GPT 引用1: span=[291, 291], relevant=1

### 陈述 4
- Abolishing Office of Drug Control - Scott abolished Florida's Office of Drug Control, which included the state drug czar position, soon after taking office in 2011. This eliminated the state's top drug policy position and further hindered efforts to implement a prescription drug monitoring database.
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[290-290]`
- au=1.6452690942969639e-06, eu=0.29230761528015137, au′=-0.0, eu′=0.03966596722602844
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.210527420043945]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 290 | 290 | 24.210527420043945 | Within weeks of being sworn in, Scott abolished Florida’s Office of Drug Control, eliminating the state drug czar position, announced plans to drive a final stake in the heart of the database and rebuffed Purdue Pharma’s renewed offer to help pay for it. |
- GPT 引用1: span=[290, 290], relevant=1

### 陈述 5
So in summary, Scott cited privacy, effectiveness, cost, and abolishing the Office of Drug Control as reasons for not implementing the database. His reasons were largely political and focused on cost and privacy concerns rather than the need to combat prescription drug abuse.
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

## multifieldqa_en_idx103 (idx=103, multifieldqa_en)
### 问题
In which electorate was Simon English elected to the New Zealand Parliament?

### 参考答案（数据集）
- ref1: The Wallace electorate.

### 模型全文输出
```
<statement>Based on the information provided, Simon English was elected to the New Zealand Parliament in 1990 as the National Party's candidate in the Wallace electorate.<cite>[3-3]</cite></statement>
```

文档句索引范围: sid ∈ [1, 156], 共 156 句。
引用指标: R=0.5, P=1.0, F1=0.6666666666666666
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=624, completion_tokens=14
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
Based on the information provided, Simon English was elected to the New Zealand Parliament in 1990 as the National Party's candidate in the Wallace electorate.
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[3-3]`
- au=1.9430344764259644e-06, eu=0.3039999008178711, au′=-0.0, eu′=0.041850216686725616
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[22.894737243652344]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 3 | 3 | 22.894737243652344 | A farmer and public servant before entering politics, English was elected to the New Zealand Parliament in  as the National Party's candidate in the Wallace electorate. |
- GPT 引用1: span=[3, 3], relevant=1

## multifieldqa_en_idx104 (idx=104, multifieldqa_en)
### 问题
What was the population of McPherson County according to the 2020 census?

### 参考答案（数据集）
- ref1: 30,223.

### 模型全文输出
```
<statement>According to the 2020 census, the population of McPherson County was 30,223.<cite>[3-3]</cite></statement>
```

文档句索引范围: sid ∈ [1, 95], 共 95 句。
引用指标: R=1.0, P=1.0, F1=1.0
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=588, completion_tokens=13
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
According to the 2020 census, the population of McPherson County was 30,223.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[3-3]`
- au=1.6015096662158612e-06, eu=0.28358200192451477, au′=-0.0, eu′=0.03807614743709564
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.263158798217773]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 3 | 3 | 25.263158798217773 | As of the 2020 census, the county population was 30,223. |
- GPT 引用1: span=[3, 3], relevant=1

## multifieldqa_en_idx105 (idx=105, multifieldqa_en)
### 问题
When did Margaret Way start self-publishing her books as e-books?

### 参考答案（数据集）
- ref1: Margaret Way started self-publishing her books as e-books in 2013.

### 模型全文输出
```
<statement>Based on the information provided, Margaret Way started self-publishing her books as e-books in mid-July 2013.<cite>[8-8]</cite></statement>
```

文档句索引范围: sid ∈ [1, 18], 共 18 句。
引用指标: R=1.0, P=1.0, F1=1.0
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=604, completion_tokens=13
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
Based on the information provided, Margaret Way started self-publishing her books as e-books in mid-July 2013.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[8-8]`
- au=1.6452690942969639e-06, eu=0.29230761528015137, au′=-0.0, eu′=0.03966596722602844
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.210527420043945]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 8 | 8 | 24.210527420043945 | Beginning in 2013, Margaret began to self-publish, releasing her first "e-book" mid-July. |
- GPT 引用1: span=[8, 8], relevant=1

## multifieldqa_en_idx106 (idx=106, multifieldqa_en)
### 问题
When did the 2017 general election be held?

### 参考答案（数据集）
- ref1: 23 September.

### 模型全文输出
```
<statement>Based on the information provided, the 2017 general election in New Zealand was held on 23 September 2017.<cite>[113-113]</cite></statement>
```

文档句索引范围: sid ∈ [1, 154], 共 154 句。
引用指标: R=1.0, P=1.0, F1=1.0
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=618, completion_tokens=13
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
Based on the information provided, the 2017 general election in New Zealand was held on 23 September 2017.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[113-113]`
- au=1.6068129298218992e-06, eu=0.28464409708976746, au′=-0.0, eu′=0.038267869502305984
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.131580352783203]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 113 | 113 | 25.131580352783203 | At a press conference at the Beehive on 1 February 2017, English announced that the 2017 general election would be held on 23 September. |
- GPT 引用1: span=[113, 113], relevant=1

## multifieldqa_en_idx107 (idx=107, multifieldqa_en)
### 问题
What hedge fund's collapse in 1998 highlighted the need for regulation of derivatives?

### 参考答案（数据集）
- ref1: Long Term Capital Management (LTCM).

### 模型全文输出
```
<statement>Based on the information provided, the collapse of Long Term Capital Management (LTCM) in 1998 highlighted the need for regulation of derivatives.<cite></cite></statement>

<statement>- LTCM was a trillion-dollar hedge fund that used derivatives to leverage $5 billion into more than $1 trillion.<cite>[49-49]</cite></statement>

<statement>- The derivative transactions were not regulated, nor were investors able to evaluate LTCM's exposures.<cite>[50-50]</cite></statement>

<statement>- Brooksley Born, chair of the Commodity Futures Trading Commission (CFTC), stated "I thought that LTCM was exactly what I had been worried about".<cite>[51-51]</cite></statement>

<statement>- In congressional hearings into the crisis, Federal Reserve chairman Alan Greenspan acknowledged that language had been introduced into an agriculture bill that would prevent the CFTC from regulating the derivatives which were at the center of the crisis.<cite>[54-54]</cite></statement>

<statement>- Brooksley Born had been warning about the lack of regulation of over-the-counter derivatives for years, but her warnings were ignored or dismissed by other regulators and financial industry leaders.<cite>[41-47]</cite></statement>

<statement>So in summary, LTCM's collapse in 1998 was a key moment that highlighted the need for regulation of derivatives, as the lack of regulation had allowed a highly leveraged and risky investment vehicle to nearly collapse and threaten the entire financial system. Brooksley Born's warnings about the need for regulation were vindicated by the crisis.<cite></cite></statement>
```

文档句索引范围: sid ∈ [1, 88], 共 88 句。
引用指标: R=0.6428571428571429, P=1.0, F1=0.782608695652174
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=4734, completion_tokens=86
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
Based on the information provided, the collapse of Long Term Capital Management (LTCM) in 1998 highlighted the need for regulation of derivatives.
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 1
- LTCM was a trillion-dollar hedge fund that used derivatives to leverage $5 billion into more than $1 trillion.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[49-49]`
- au=0.6823434829711914, eu=0.17840370535850525, au′=0.682342529296875, eu′=0.041621025651693344
- top_logits 方差 (pvariance, 有限 logit): 0.0692520511447583
- top_logits 数值: `[23.289474487304688, 22.763158798217773]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 49 | 49 | 23.289474487304688 | Using mathematical models to calculate debt risk, LTCM used derivatives to leverage $5 billion into more than $1 trillion, doing business with fifteen of Wall Street's largest financial institutions. |
| 2 | 48 | 48 | 22.763158798217773 | In 1998, a trillion-dollar hedge fund called Long Term Capital Management (LTCM) was near collapse. |
- GPT 引用1: span=[49, 49], relevant=1

### 陈述 2
- The derivative transactions were not regulated, nor were investors able to evaluate LTCM's exposures.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[50-50]`
- au=1.6452690942969639e-06, eu=0.29230761528015137, au′=-0.0, eu′=0.03966596722602844
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.210527420043945]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 50 | 50 | 24.210527420043945 | The derivative transactions were not regulated, nor were investors able to evaluate LTCM's exposures. |
- GPT 引用1: span=[50, 50], relevant=1

### 陈述 3
- Brooksley Born, chair of the Commodity Futures Trading Commission (CFTC), stated "I thought that LTCM was exactly what I had been worried about".
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[51-51]`
- au=1.6068129298218992e-06, eu=0.28464409708976746, au′=-0.0, eu′=0.038267869502305984
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.131580352783203]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 51 | 51 | 25.131580352783203 | Born stated, "I thought that LTCM was exactly what I had been worried about". |
- GPT 引用1: span=[51, 51], relevant=1

### 陈述 4
- In congressional hearings into the crisis, Federal Reserve chairman Alan Greenspan acknowledged that language had been introduced into an agriculture bill that would prevent the CFTC from regulating the derivatives which were at the center of the crisis.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[54-54]`
- au=1.9067709899900365e-06, eu=0.29687488079071045, au′=-0.0, eu′=0.04051172360777855
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.68421173095703]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 54 | 54 | 23.68421173095703 | In congressional hearings into the crisis, Greenspan acknowledged that language had been introduced into an agriculture bill that would prevent CFTC from regulating the derivatives which were at the center of the crisis that threatened the US economy. |
- GPT 引用1: span=[54, 54], relevant=1

### 陈述 5
- Brooksley Born had been warning about the lack of regulation of over-the-counter derivatives for years, but her warnings were ignored or dismissed by other regulators and financial industry leaders.
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[41-47]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> Due to litigation against Bankers Trust Company by Procter and Gamble and other
> corporate clients, Born and her team at the CFTC sought comments on the regulation of
> over-the-counter derivatives, a first step in the process of writing CFTC regulations to
> supplement the existing regulations of the Federal Reserve System,  the Options Clearing
> Corporation, and the National Association of Insurance Commissioners. Born was
> particularly concerned about swaps, financial instruments that are traded over the
> counter between banks, insurance companies or other funds or companies, and thus have no
> transparency except to the two counterparties and the counterparties' regulators, if
> any. CFTC regulation was strenuously opposed by Federal Reserve chairman Alan Greenspan,
> and by Treasury Secretaries Robert Rubin and Lawrence Summers. On May 7, 1998, former
> SEC Chairman Arthur Levitt joined Rubin and Greenspan in objecting to the issuance of
> the CFTC's concept release. Their response dismissed Born's analysis and focused on the
> hypothetical possibility that CFTC regulation of swaps and other OTC derivative
> instruments could create a "legal uncertainty" regarding such financial instruments,
> hypothetically reducing the value of the instruments. They argued that the imposition of
> regulatory costs would "stifle financial innovation" and encourage financial capital to
> transfer its  transactions offshore. The disagreement between Born and the Executive
> Office's top economic policy advisors has been described not only as a classic
> Washington turf war, but also a war of ideologies,  insofar as it is possible to argue
> that Born's actions were consistent with Keynesian and neoclassical economics while
> Greenspan, Rubin, Levitt, and Summers consistently espoused neoliberal, and
> neoconservative policies.
- au=1.083716869354248, eu=0.12925167381763458, au′=1.083716630935669, eu′=0.042632754892110825
- top_logits 方差 (pvariance, 有限 logit): 0.20006159262589385
- top_logits 数值: `[22.894737243652344, 22.631580352783203, 21.842105865478516]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 41 | 41 | 22.894737243652344 | Due to litigation against Bankers Trust Company by Procter and Gamble and other corporate clients, Born and her team at the CFTC sought comments on the regulation of over-the-counter derivatives, a first step in the process of writing CFTC regulations to supplement the existing regulations of the Federal Reserve System,  the Options Clearing Corporation, and the National Association of Insurance Commissioners. |
| 2 | 3 | 3 | 22.631580352783203 | During her tenure on the CFTC, Born lobbied Congress and the President to give the CFTC oversight of off-exchange markets for derivatives, in addition to its role with respect to exchange-traded derivatives, but her warnings were ignored or dismissed, and her calls for reform resisted by other regulators.<ref name="nytimes">Goodman, Peter S. The Reckoning - Taking Hard New Look at a Greenspan Legacy, The New York Times, October 9, 2008.</ref> Born resigned as chairperson on June 1, 1999, shortly after Congress passed legislation prohibiting her agency from regulating derivatives. |
| 3 | 43 | 43 | 21.842105865478516 | CFTC regulation was strenuously opposed by Federal Reserve chairman Alan Greenspan, and by Treasury Secretaries Robert Rubin and Lawrence Summers. |
- GPT 引用1: span=[41, 47], relevant=1

### 陈述 6
So in summary, LTCM's collapse in 1998 was a key moment that highlighted the need for regulation of derivatives, as the lack of regulation had allowed a highly leveraged and risky investment vehicle to nearly collapse and threaten the entire financial system. Brooksley Born's warnings about the need for regulation were vindicated by the crisis.
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

## multifieldqa_en_idx108 (idx=108, multifieldqa_en)
### 问题
What is the dynamical behavior of the anisotropic order parameter following a quench to the critical point?

### 参考答案（数据集）
- ref1: It is well described by the Gaussian theory.

### 模型全文输出
```
<statement>Based on the paper, the dynamical behavior of the anisotropic order parameter $m$ following a quench to the critical point is well described by the Gaussian theory for all three lattice gas models studied.<cite>[1-2]</cite></statement>

<statement>Specifically:<cite></cite></statement>

<statement>- In the short-time regime, $m \sim t^{1/2}$.<cite>[2-2]</cite></statement>

<statement>- The Binder cumulant $g$ of the lowest transverse mode is zero in this regime.<cite>[3-4]</cite></statement>

<statement>The alternative order parameter $O$ distinguishes between the driven and equilibrium lattice gases. At short times, the dependence of $O$ on system size $L_\parallel$ is $O \sim L_\parallel^{-1/2}$. The generic behavior of $O$ is $O \sim t^{\alpha} L_\parallel^{-1/2}$, where $\alpha$ is a phenomenological exponent determined by the scaling form of $O$. The prediction $\alpha = 1/8$ of the Gaussian theory can only be obtained when $\eta=0$, which is the case for the driven lattice gases but not the equilibrium lattice gas. The non-Gaussian nature of higher fluctuating modes in the equilibrium lattice gas leads to the emergence of a new value $\alpha = 1/10$ for the exponent.<cite>[5-5][8-8][10-10]</cite></statement>
```

文档句索引范围: sid ∈ [1, 26], 共 26 句。
引用指标: R=0.9, P=0.8333333333333334, F1=0.8653846153846153
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=4384, completion_tokens=77
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
Based on the paper, the dynamical behavior of the anisotropic order parameter $m$ following a quench to the critical point is well described by the Gaussian theory for all three lattice gas models studied.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[1-2]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> \section*{Dynamical Behaviour of $O$ in Lattice Gases}
>
> The dynamical behaviour of the anisotropic order parameter  $m$ [see
> Eq.~\eqref{eq:def-m} in the Letter] following a quench to the critical point is well
> described by the Gaussian theory for all the three lattice gas models studied, $i.e.,$
> driven lattice gas with either constant (IDLG) or random (RDLG) infinite drive and
> equilibrium lattice gas (LG). In other words, in the short-time regime, $m \sim t^{1/2}$
> [see Eq.
- au=1.650961507948523e-06, eu=0.2934361696243286, au′=-0.0, eu′=0.03987407684326172
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.078948974609375]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 1 | 1 | 24.078948974609375 | \section*{Dynamical Behaviour of $O$ in Lattice Gases}  The dynamical behaviour of the anisotropic order parameter  $m$ [see Eq.~\eqref{eq:def-m} in the Letter] following a quench to the critical point is well described by the Gaussian theory for all the three lattice gas models studied, $i.e.,$ driven lattice gas with either constant (IDLG) or random (RDLG) infinite drive and equilibrium lattice gas (LG). |
- GPT 引用1: span=[1, 2], relevant=1

### 陈述 1
Specifically:
- GPT support_score=1; [[No]]  Need Citation: [[No]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 2
- In the short-time regime, $m \sim t^{1/2}$.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[2-2]`
- au=1.6068129298218992e-06, eu=0.28464409708976746, au′=-0.0, eu′=0.038267869502305984
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.131580352783203]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 2 | 2 | 25.131580352783203 | In other words, in the short-time regime, $m \sim t^{1/2}$ [see Eq. |
- GPT 引用1: span=[2, 2], relevant=1

### 陈述 3
- The Binder cumulant $g$ of the lowest transverse mode is zero in this regime.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[3-4]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> \eqref{eq:mt}] and the Binder cumulant $g$ of the lowest transverse mode [defined in Eq.
> \eqref{eq:binder}] is zero in this regime.
- au=0.6826401948928833, eu=0.17471261322498322, au′=0.6826393604278564, eu′=0.04061998426914215
- top_logits 方差 (pvariance, 有限 logit): 0.03895437288156245
- top_logits 数值: `[23.8157901763916, 23.421052932739258]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 3 | 3 | 23.8157901763916 | \eqref{eq:mt}] and the Binder cumulant $g$ of the lowest transverse mode [defined in Eq. |
| 2 | 4 | 4 | 23.421052932739258 | \eqref{eq:binder}] is zero in this regime. |
- GPT 引用1: span=[3, 4], relevant=1

### 陈述 4
The alternative order parameter $O$ distinguishes between the driven and equilibrium lattice gases. At short times, the dependence of $O$ on system size $L_\parallel$ is $O \sim L_\parallel^{-1/2}$. The generic behavior of $O$ is $O \sim t^{\alpha} L_\parallel^{-1/2}$, where $\alpha$ is a phenomenological exponent determined by the scaling form of $O$. The prediction $\alpha = 1/8$ of the Gaussian theory can only be obtained when $\eta=0$, which is the case for the driven lattice gases but not the equilibrium lattice gas. The non-Gaussian nature of higher fluctuating modes in the equilibrium lattice gas leads to the emergence of a new value $\alpha = 1/10$ for the exponent.
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[5-5]`
- au=1.861421651483397e-06, eu=0.28787869215011597, au′=-0.0, eu′=0.038854800164699554
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.73684310913086]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 5 | 5 | 24.73684310913086 | The alternative order parameter $O,$ however, distinguishes between the driven (IDLG, RDLG) and the equilibrium (LG) lattice gases. |

**跨度** `[8-8]`
- au=1.6284955108858412e-06, eu=0.28897327184677124, au′=-0.0, eu′=0.03905446454882622
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.60526466369629]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 8 | 8 | 24.60526466369629 | However, following Ref. |

**跨度** `[10-10]`
- au=1.6175589507838595e-06, eu=0.2867923676967621, au′=-0.0, eu′=0.03865716978907585
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.86842155456543]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 10 | 10 | 24.86842155456543 | Accordingly,  the generic behaviour of $O$ can be assumed to be \begin{eqnarray} O \sim t^{\alpha} L_\parallel^{-1/2}, \label{eq:O} \end{eqnarray} where $\alpha$ is a phenomenological exponent to be determined. |
- GPT 引用1: span=[5, 5], relevant=1
- GPT 引用2: span=[8, 8], relevant=0
- GPT 引用3: span=[10, 10], relevant=1

## multifieldqa_en_idx109 (idx=109, multifieldqa_en)
### 问题
What is the recommended space for using the VR headset?

### 参考答案（数据集）
- ref1: It is recommended to have at least a 2x2 meter space for using the VR headset.

### 模型全文输出
```
<statement>Based on the user guide, the recommended space for using the VR headset is at least 2 meters by 2 meters (6.56 feet by 6.56 feet).<cite>[18-18]</cite></statement>

<statement>It is important to have enough space around your head and body to stretch your arms without damaging yourself, others, or your surroundings.<cite>[50-50]</cite></statement>
```

文档句索引范围: sid ∈ [1, 130], 共 130 句。
引用指标: R=1.0, P=1.0, F1=1.0
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=1242, completion_tokens=26
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
Based on the user guide, the recommended space for using the VR headset is at least 2 meters by 2 meters (6.56 feet by 6.56 feet).
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[18-18]`
- au=0.6809309720993042, eu=0.18491479754447937, au′=0.6809301376342773, eu′=0.04340376704931259
- top_logits 方差 (pvariance, 有限 logit): 0.9738574397861157
- top_logits 数值: `[23.026315689086914, 21.052631378173828]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 18 | 18 | 23.026315689086914 | 建议准备一个整洁安全的体验空间：至少 2×2 米； |
| 2 | 4 | 4 | 21.052631378173828 | ““显示” ► ► ► 包装内含： VR 头盔 / 手柄 × 2 / 1.5V AA 碱性干电池 × 4/ 眼镜支架 / 遮光鼻托 / 手柄挂绳 × 2 / USB-C 电源适配器 / USB-Cto C 2.0 数据线 / 快速指南 / 用户指南 / 安全与质保指南使用前注意事项 •本产品在开阔的室內环境使用体验最佳，建议至少预留 2×2 米 的空间。 |
- GPT 引用1: span=[18, 18], relevant=1

### 陈述 1
It is important to have enough space around your head and body to stretch your arms without damaging yourself, others, or your surroundings.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[50-50]`
- au=1.6452690942969639e-06, eu=0.29230761528015137, au′=-0.0, eu′=0.03966596722602844
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.210527420043945]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 50 | 50 | 24.210527420043945 | Make sure that there is enough space around yourhead and body (at least 2 meters by 2 meters) to stretch your arms to avoid damage or injury toyourself, others, and your surroundings. |
- GPT 引用1: span=[50, 50], relevant=1

## multifieldqa_en_idx110 (idx=110, multifieldqa_en)
### 问题
How does the performance of the PLM with decimation compare to other methods?

### 参考答案（数据集）
- ref1: It outperforms mean-field methods and the PLM with $l_2$ regularization in terms of reconstruction error and true positive rate.

### 模型全文输出
```
<statement>Based on the results presented in the paper, the PLM with decimation outperforms other methods in several ways:<cite></cite></statement>

<statement>- It is able to accurately infer sparse networks with low sample sizes. The PLM with decimation is able to correctly infer the majority of couplings present in the system even when the number of samples is very low (e.g. M=256). In contrast, the PLM with regularization and the mean-field method perform poorly when sample sizes are low, often failing to find more than half of the couplings.<cite>[275-276][277-277]</cite></statement>

<statement>- It is able to infer the correct network structure even when sample sizes are equal to the number of variables. The PLM with decimation is able to correctly infer the network structure even when the number of samples is equal to the number of variables (M=N), indicating it could be considered in the under-sampling regime M<N. In contrast, the other methods do not perform well in this regime.<cite>[278-278]</cite></statement>

<statement>- It has lower reconstruction error than other methods. The PLM with decimation has a lower reconstruction error than the PLM with regularization and the mean-field method across all temperatures and sample sizes tested. The error is several times smaller than the error estimated by the other methods.<cite>[275-275][277-277][281-281]</cite></statement>

<statement>- It is able to clearly infer the network of interaction. The PLM with decimation leads to cleaner results than the other methods. The inferred couplings are more clearly separated into sets corresponding to non-zero and zero couplings.<cite>[265-265][266-266][267-267]</cite></statement>

<statement>- It has a precise estimate of the total number of bonds. The PLM with decimation allows for a precise estimate of the total number of bonds by identifying the number of decimated bonds at which the tilted pseudo-likelihood is maximum. This number is close to the true number of bonds in the system.<cite>[289-289]</cite></statement>

<statement>In summary, the PLM with decimation outperforms other methods in accurately inferring sparse networks with low sample sizes, correctly inferring network structure even in under-sampling regimes, having lower reconstruction error, and providing a precise estimate of the total number of bonds. Its superior performance makes it a promising method for statistical inference in systems with sparse interactions.<cite></cite></statement>
```

文档句索引范围: sid ∈ [1, 298], 共 298 句。
引用指标: R=0.6428571428571429, P=0.7142857142857143, F1=0.6766917293233083
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=5797, completion_tokens=104
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
Based on the results presented in the paper, the PLM with decimation outperforms other methods in several ways:
- GPT support_score=1; [[No]]   Need Citation: [[No]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 1
- It is able to accurately infer sparse networks with low sample sizes. The PLM with decimation is able to correctly infer the majority of couplings present in the system even when the number of samples is very low (e.g. M=256). In contrast, the PLM with regularization and the mean-field method perform poorly when sample sizes are low, often failing to find more than half of the couplings.
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[275-276]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> However, for lower values of $M$ the performances of the PLM with $l_2$ regularization
> and with decimation drastically overcome those ones of the previous mean field
> technique.
>
> In particular, for $M=256$ the PLM techniques still reproduce the original network while
> the mean-field method fails to find more than half of the couplings.
- au=1.0821540355682373, eu=0.13893964886665344, au′=1.0821532011032104, eu′=0.04617253318428993
- top_logits 方差 (pvariance, 有限 logit): 0.4270547336709569
- top_logits 数值: `[21.578948974609375, 20.263158798217773, 20.131580352783203]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 275 | 275 | 21.578948974609375 | However, for lower values of $M$ the performances of the PLM with $l_2$ regularization and with decimation drastically overcome those ones of the previous mean field technique. |
| 2 | 276 | 276 | 20.263158798217773 | In particular, for $M=256$ the PLM techniques still reproduce the original network while the mean-field method fails to find more than half of the couplings. |
| 3 | 274 | 274 | 20.131580352783203 | When $M$ is high enough, all techniques reproduce the true network. |

**跨度** `[277-277]`
- au=1.5962523320922628e-06, eu=0.28252777457237244, au′=-0.0, eu′=0.03788633644580841
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.394737243652344]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 277 | 277 | 25.394737243652344 | When $M=128$, the network is clearly reconstructed only through the PLM with decimation while the PLM with $l_2$ regularization underestimates the couplings. |
- GPT 引用1: span=[275, 277], relevant=1

### 陈述 2
- It is able to infer the correct network structure even when sample sizes are equal to the number of variables. The PLM with decimation is able to correctly infer the network structure even when the number of samples is equal to the number of variables (M=N), indicating it could be considered in the under-sampling regime M<N. In contrast, the other methods do not perform well in this regime.
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[278-278]`
- au=0.6813287138938904, eu=0.18446597456932068, au′=0.6813278198242188, eu′=0.04328017681837082
- top_logits 方差 (pvariance, 有限 logit): 0.6232684603028247
- top_logits 数值: `[22.894737243652344, 21.3157901763916]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 278 | 278 | 22.894737243652344 | Furthermore, we notice that the PLM method with decimation is able to clearly infer the network of interaction even when $M=N$ signalling that it could be considered also in the under-sampling regime $M<N$. |
| 2 | 277 | 277 | 21.3157901763916 | When $M=128$, the network is clearly reconstructed only through the PLM with decimation while the PLM with $l_2$ regularization underestimates the couplings. |
- GPT 引用1: span=[278, 278], relevant=1

### 陈述 3
- It has lower reconstruction error than other methods. The PLM with decimation has a lower reconstruction error than the PLM with regularization and the mean-field method across all temperatures and sample sizes tested. The error is several times smaller than the error estimated by the other methods.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[275-275]`
- au=0.6820008158683777, eu=0.17840370535850525, au′=0.6819998025894165, eu′=0.041621025651693344
- top_logits 方差 (pvariance, 有限 logit): 0.4328259470721605
- top_logits 数值: `[23.68421173095703, 22.36842155456543]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 281 | 281 | 23.68421173095703 | In can be observed that for all temperatures and for all sample sizes  the reconstruction error, ${\rm err_J}$, (plotted here in log-scale) obtained with the PLM+decimation is always smaller than that one obtained with the other techniques. |
| 2 | 275 | 275 | 22.36842155456543 | However, for lower values of $M$ the performances of the PLM with $l_2$ regularization and with decimation drastically overcome those ones of the previous mean field technique. |

**跨度** `[277-277]`
- au=0.6827458739280701, eu=0.1735159456729889, au′=0.6827449798583984, eu′=0.04029691964387894
- top_logits 方差 (pvariance, 有限 logit): 0.017312887302978197
- top_logits 数值: `[23.947368621826172, 23.68421173095703]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 277 | 277 | 23.947368621826172 | When $M=128$, the network is clearly reconstructed only through the PLM with decimation while the PLM with $l_2$ regularization underestimates the couplings. |
| 2 | 281 | 281 | 23.68421173095703 | In can be observed that for all temperatures and for all sample sizes  the reconstruction error, ${\rm err_J}$, (plotted here in log-scale) obtained with the PLM+decimation is always smaller than that one obtained with the other techniques. |

**跨度** `[281-281]`
- au=1.5858721553740907e-06, eu=0.2804427146911621, au′=-0.0, eu′=0.03751233592629433
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.657896041870117]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 281 | 281 | 25.657896041870117 | In can be observed that for all temperatures and for all sample sizes  the reconstruction error, ${\rm err_J}$, (plotted here in log-scale) obtained with the PLM+decimation is always smaller than that one obtained with the other techniques. |
- GPT 引用1: span=[275, 275], relevant=0
- GPT 引用2: span=[277, 277], relevant=0
- GPT 引用3: span=[281, 281], relevant=1

### 陈述 4
- It is able to clearly infer the network of interaction. The PLM with decimation leads to cleaner results than the other methods. The inferred couplings are more clearly separated into sets corresponding to non-zero and zero couplings.
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[265-265]`
- au=2.0151851458649617e-06, eu=0.3179915249347687, au′=-0.0, eu′=0.044548649340867996
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[21.447368621826172]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 265 | 265 | 21.447368621826172 | In comparison to the results shown in Sec. |

**跨度** `[266-266]`
- au=0.6801754832267761, eu=0.20105816423892975, au′=0.6801745891571045, eu′=0.04791928827762604
- top_logits 方差 (pvariance, 有限 logit): 0.432824692237773
- top_logits 数值: `[20.526315689086914, 19.210527420043945]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 266 | 266 | 20.526315689086914 | \ref{sec:res_reg}, the PLM with decimation leads to rather cleaner results. |
| 2 | 267 | 267 | 19.210527420043945 | In Figs. |

**跨度** `[267-267]`
- au=2.0550742149353027, eu=0.06440676748752594, au′=2.0550739765167236, eu′=0.05219779908657074
- top_logits 方差 (pvariance, 有限 logit): 0.35058879125830344
- top_logits 数值: `[19.60526466369629, 18.289474487304688, 18.157896041870117, 18.157896041870117, 17.894737243652344, 17.894737243652344, 17.631580352783203, 17.631580352783203]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 267 | 267 | 19.60526466369629 | In Figs. |
| 2 | 277 | 277 | 18.289474487304688 | When $M=128$, the network is clearly reconstructed only through the PLM with decimation while the PLM with $l_2$ regularization underestimates the couplings. |
| 3 | 268 | 268 | 18.157896041870117 | \ref{MF_PL_err} and \ref{MF_PL_TP} we compare the performances of the PLM with decimation in respect to ones of the PLM with $l_2$-regularization. |
| 4 | 274 | 274 | 18.157896041870117 | When $M$ is high enough, all techniques reproduce the true network. |
| 5 | 288 | 288 | 17.894737243652344 | The behavior of the quality of the network reconstruction is analyzed by looking at the overall sorted couplings and at the single site couplings, comparing them with the real network, and at the true positive curves in all three approaches. |
| 6 | 275 | 275 | 17.894737243652344 | However, for lower values of $M$ the performances of the PLM with $l_2$ regularization and with decimation drastically overcome those ones of the previous mean field technique. |
| 7 | 276 | 276 | 17.631580352783203 | In particular, for $M=256$ the PLM techniques still reproduce the original network while the mean-field method fails to find more than half of the couplings. |
| 8 | 272 | 272 | 17.631580352783203 | The results refer to the case of ordered and real valued couplings, but similar behaviours were obtained for the other cases analysed. |
- GPT 引用1: span=[265, 267], relevant=1

### 陈述 5
- It has a precise estimate of the total number of bonds. The PLM with decimation allows for a precise estimate of the total number of bonds by identifying the number of decimated bonds at which the tilted pseudo-likelihood is maximum. This number is close to the true number of bonds in the system.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[289-289]`
- au=1.7630600268603303e-06, eu=0.3153526186943054, au′=-0.0, eu′=0.04403243958950043
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[21.710527420043945]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 289 | 289 | 21.710527420043945 | In the PLM +decimation method, moreover, the identification of the number of decimated bonds at which the tilted pseudo-likelihood is maximum allows for a precise estimate of the total number of bonds. |
- GPT 引用1: span=[289, 289], relevant=1

### 陈述 6
In summary, the PLM with decimation outperforms other methods in accurately inferring sparse networks with low sample sizes, correctly inferring network structure even in under-sampling regimes, having lower reconstruction error, and providing a precise estimate of the total number of bonds. Its superior performance makes it a promising method for statistical inference in systems with sparse interactions.
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

## multifieldqa_en_idx111 (idx=111, multifieldqa_en)
### 问题
What are the three phases of the author's preaching process?

### 参考答案（数据集）
- ref1: The three phases are exegetical, theological, and homiletical.

### 模型全文输出
```
<statement>Based on the blog post, the author's preaching process involves three main phases:<cite></cite></statement>

<statement>1. Exegetical Phase - This is the initial phase where the author studies the passage itself. He looks at the immediate context, Greek language, and researches the meaning of key concepts and relationships between ideas. He tries to understand the passage on its own terms before applying theological ideas.<cite>[16-36]</cite></statement>

<statement>2. Theological Phase - This phase involves asking questions and answering them to identify key theological ideas and timeless truths communicated by the passage. The author tries to determine what theology the author of the passage was trying to communicate, rather than just imposing his own ideas. He tries to bracket out some doctrines while holding onto core ones.<cite>[36-36][38-38][39-39]</cite></statement>

<statement>3. Homiletical Phase - This is the phase where the author starts crafting the sermon. He looks for interesting ideas and phrases to incorporate, tries to relate the passage back to a main thesis, and focuses on application. He tries to find one main application emphasized by the text rather than making up his own. He also tries to show how Christ has already fulfilled the passage's commands and how believers can imitate Christ in applying it.<cite>[61-61][62-62][63-63]</cite></statement>
```

文档句索引范围: sid ∈ [1, 593], 共 593 句。
引用指标: R=0.75, P=0.5, F1=0.6
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=3589, completion_tokens=64
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
Based on the blog post, the author's preaching process involves three main phases:
- GPT support_score=1; [[No]]  Need Citation: [[No]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 1
1. Exegetical Phase - This is the initial phase where the author studies the passage itself. He looks at the immediate context, Greek language, and researches the meaning of key concepts and relationships between ideas. He tries to understand the passage on its own terms before applying theological ideas.
- GPT support_score=1; **Rating: [[Fully supported]]**   **

**跨度** `[16-36]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> Phase 1: Exegetical The exegetical phase is just getting into the passage itself. First,
> I want to know the immediate context for the passage. Why is this here? What ideas are
> continuing, and which are new? How the author frames the passage is probably the most
> important factor in choosing how I will introduce it in the sermon. If it’s an epistle,
> I will also play with a structural outline to try and identify rhetorical choices.
> Second, I want to try and surface issues in the Greek. I do a very rough translation,
> and if a word jumps out at me I take note. I’m looking for ambiguous meanings,
> untranslated concepts, repeated words, related words, etc. I don’t expect to come up
> with a better translation than the professionals; I just want to have some idea of why
> they made the choices they did and what might be getting lost in translation. (Note: I
> don’t talk about Greek and Hebrew words from the pulpit; I only explain the concepts
> because that’s what I expect people to remember.) Third, I take the list of questions
> I’ve been building and I start to do research. Who’s the referent in this verse? What
> does this metaphor mean? Is it used elsewhere? What’s the relationship between these two
> ideas? Does this command really imply that? My research is mostly based on Scripture
> alone, although there are times when I have to turn to historical background information
> to really get a reference. I see the Bible as one whole text even though it has many
> authors, and I’m very interested at drawing legitimate connections across books. I’ve
> also found Carson’s Exegetical Fallacies is a great help at avoiding common errors in
> biblical studies. Phase 2: Theological The boundary between exegesis and theology is
> thin and messy.
- au=0.6821538805961609, eu=0.18138422071933746, au′=0.6821529865264893, eu′=0.042434390634298325
- top_logits 方差 (pvariance, 有限 logit): 0.03895437288156245
- top_logits 数值: `[22.763158798217773, 22.36842155456543]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 15 | 15 | 22.763158798217773 | Once I have my topic, I start running the passage through a system in the three phases we used in seminary: exegetical, theological, and homiletical. |
| 2 | 16 | 16 | 22.36842155456543 | Phase 1: Exegetical The exegetical phase is just getting into the passage itself. |
- GPT 引用1: span=[16, 36], relevant=1

### 陈述 2
2. Theological Phase - This phase involves asking questions and answering them to identify key theological ideas and timeless truths communicated by the passage. The author tries to determine what theology the author of the passage was trying to communicate, rather than just imposing his own ideas. He tries to bracket out some doctrines while holding onto core ones.
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[36-36]`
- au=0.6813099384307861, eu=0.18052251636981964, au′=0.6813089847564697, eu′=0.04219876974821091
- top_logits 方差 (pvariance, 有限 logit): 0.9738574397861157
- top_logits 数值: `[23.68421173095703, 21.710527420043945]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 36 | 36 | 23.68421173095703 | Phase 2: Theological The boundary between exegesis and theology is thin and messy. |
| 2 | 43 | 43 | 21.710527420043945 | In short, the question raising and answering process is really the beginning of the theological phase for me. |

**跨度** `[38-38]`
- au=0.6832991242408752, eu=0.1659388244152069, au′=0.6832983493804932, eu′=0.038267869502305984
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.131580352783203, 25.131580352783203]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 39 | 39 | 25.131580352783203 | Not all doctrines are equal. |
| 2 | 38 | 38 | 25.131580352783203 | I try to do both if I can. |

**跨度** `[39-39]`
- au=0.6831138134002686, eu=0.16851437091827393, au′=0.6831128597259521, eu′=0.03895437717437744
- top_logits 方差 (pvariance, 有限 logit): 0.004328221825744549
- top_logits 数值: `[24.73684310913086, 24.60526466369629]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 40 | 40 | 24.73684310913086 | I refuse to bracket out core doctrines like the Trinity or salvation by grace alone through faith alone. |
| 2 | 39 | 39 | 24.60526466369629 | Not all doctrines are equal. |
- GPT 引用1: span=[36, 36], relevant=0
- GPT 引用2: span=[38, 39], relevant=0

### 陈述 3
3. Homiletical Phase - This is the phase where the author starts crafting the sermon. He looks for interesting ideas and phrases to incorporate, tries to relate the passage back to a main thesis, and focuses on application. He tries to find one main application emphasized by the text rather than making up his own. He also tries to show how Christ has already fulfilled the passage's commands and how believers can imitate Christ in applying it.
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[61-61]`
- au=1.6068129298218992e-06, eu=0.28464409708976746, au′=-0.0, eu′=0.038267869502305984
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.131580352783203]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 61 | 61 | 25.131580352783203 | Phase 3: Homiletical Throughout the whole process thus far, I’m keeping my eyes open for anything interesting, catchy, or eloquent. |

**跨度** `[62-62]`
- au=0.6825760006904602, eu=0.17272724211215973, au′=0.6825752258300781, eu′=0.04008438438177109
- top_logits 方差 (pvariance, 有限 logit): 0.2770082045790332
- top_logits 数值: `[24.473684310913086, 23.421052932739258]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 63 | 63 | 24.473684310913086 | If a crucial idea jumps out, I want to note it so I can craft a phrase around it. |
| 2 | 62 | 62 | 23.421052932739258 | In some ways I’m having a conversation with the text and cross-references, and I note the parts of the conversation I like. |

**跨度** `[63-63]`
- au=1.5858721553740907e-06, eu=0.2804427146911621, au′=-0.0, eu′=0.03751233592629433
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.657896041870117]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 63 | 63 | 25.657896041870117 | If a crucial idea jumps out, I want to note it so I can craft a phrase around it. |
- GPT 引用1: span=[61, 63], relevant=1

## multifieldqa_en_idx112 (idx=112, multifieldqa_en)
### 问题
When did KSTP switch to a sports radio format?

### 参考答案（数据集）
- ref1: KSTP switched to a sports radio format on February 15, 2010.

### 模型全文输出
```
<statement>Based on the information provided, KSTP switched to a sports radio format on February 15, 2010.<cite>[66-66]</cite></statement>
```

文档句索引范围: sid ∈ [1, 96], 共 96 句。
引用指标: R=1.0, P=1.0, F1=1.0
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=588, completion_tokens=13
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
Based on the information provided, KSTP switched to a sports radio format on February 15, 2010.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[66-66]`
- au=1.5858721553740907e-06, eu=0.2804427146911621, au′=-0.0, eu′=0.03751233592629433
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.657896041870117]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 66 | 66 | 25.657896041870117 | Sports Radio KSTP switched to Sports Radio on February 15, 2010. |
- GPT 引用1: span=[66, 66], relevant=1

## multifieldqa_en_idx113 (idx=113, multifieldqa_en)
### 问题
What was the best performing model for the Spanish language in Track-1?

### 参考答案（数据集）
- ref1: The best performing model for the Spanish language in Track-1 was Spanish BERT.

### 模型全文输出
```
<statement>Based on the paper, the best performing model for the Spanish language in Track-1 was RoBERTa.<cite>[71-71]</cite></statement>

<statement>Specifically, the paper states: "The best performing models for the English language were RoBERTa and BERT whereas GPT-2 was the worst performing. Similarly the language specific versions of RoBERTa and BERT performed well for the Spanish and Portuguese respectively."<cite>[71-72]</cite></statement>

<statement>So RoBERTa was the best performing model for Spanish in Track-1.<cite></cite></statement>
```

文档句索引范围: sid ∈ [1, 130], 共 130 句。
引用指标: R=0.3333333333333333, P=0.5, F1=0.4
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=1618, completion_tokens=38
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
Based on the paper, the best performing model for the Spanish language in Track-1 was RoBERTa.
- GPT support_score=0; Rating: [[No support]]

**跨度** `[71-71]`
- au=0.6814788579940796, eu=0.19047614932060242, au′=0.6814780235290527, eu′=0.04494381695985794
- top_logits 方差 (pvariance, 有限 logit): 0.03895437288156245
- top_logits 数值: `[21.447368621826172, 21.052631378173828]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 71 | 71 | 21.447368621826172 | The best performing models for the English language were RoBERTa and BERT whereas GPT-2 was the worst performing. |
| 2 | 70 | 70 | 21.052631378173828 | We used XLM-RoBERTa as the baseline for all 3 languages. |
- GPT 引用1: span=[71, 71], relevant=0

### 陈述 1
Specifically, the paper states: "The best performing models for the English language were RoBERTa and BERT whereas GPT-2 was the worst performing. Similarly the language specific versions of RoBERTa and BERT performed well for the Spanish and Portuguese respectively."
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[71-72]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> The best performing models for the English language were RoBERTa and BERT whereas GPT-2
> was the worst performing. Similarly the language specific versions of RoBERTa and BERT
> performed well for the Spanish and Portuguese respectively.
- au=1.5318007626774488e-06, eu=0.26950347423553467, au′=-0.0, eu′=0.03558051958680153
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[27.10526466369629]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 71 | 71 | 27.10526466369629 | The best performing models for the English language were RoBERTa and BERT whereas GPT-2 was the worst performing. |
- GPT 引用1: span=[71, 72], relevant=1

### 陈述 2
So RoBERTa was the best performing model for Spanish in Track-1.
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

## multifieldqa_en_idx114 (idx=114, multifieldqa_en)
### 问题
How does the infall rate and gas density in the magnetized model compare to non-magnetized accretion?

### 参考答案（数据集）
- ref1: Infall rate is 2-5 times smaller and gas density is 2-5 times smaller.

### 模型全文输出
```
<statement>Based on the paper, the infall rate and gas density in the magnetized model are both lower compared to non-magnetized accretion.<cite></cite></statement>

<statement>Specifically:<cite></cite></statement>

<statement>- The infall rate in the magnetized model is 2-5 times smaller than in non-magnetized accretion, depending on the outer magnetization.<cite>[41-41]</cite></statement>

<statement>- The gas density in the magnetized model is 2-5 times smaller in the region close to the black hole compared to non-magnetized accretion.<cite>[41-42]</cite></statement>

<statement>- The lower infall rate and gas density in the magnetized model are attributed to the inhibiting effect of the magnetic field on accretion.<cite></cite></statement>

<statement>- The lower gas density in the magnetized model is in qualitative agreement with models of the spectrum of Sgr A*, which produces relatively weak synchrotron radiation.<cite>[43-45]</cite></statement>

<statement>So in summary, the presence of a magnetic field leads to a lower infall rate and gas density compared to non-magnetized accretion, which is consistent with observations of Sgr A*. The magnetic field inhibits accretion and leads to lower densities in the inner flow region where synchrotron radiation emerges.<cite></cite></statement>
```

文档句索引范围: sid ∈ [1, 73], 共 73 句。
引用指标: R=0.5714285714285714, P=1.0, F1=0.7272727272727273
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=3949, completion_tokens=79
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
Based on the paper, the infall rate and gas density in the magnetized model are both lower compared to non-magnetized accretion.
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 1
Specifically:
- GPT support_score=1; [[No]]   Need Citation: [[No]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 2
- The infall rate in the magnetized model is 2-5 times smaller than in non-magnetized accretion, depending on the outer magnetization.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[41-41]`
- au=1.774934617060353e-06, eu=0.2704625725746155, au′=-0.0, eu′=0.03574788197875023
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[26.973684310913086]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 41 | 41 | 26.973684310913086 | Compared to non-magnetized accretion, infall rate is 2-5 times smaller depending on outer magnetization. |
- GPT 引用1: span=[41, 41], relevant=1

### 陈述 3
- The gas density in the magnetized model is 2-5 times smaller in the region close to the black hole compared to non-magnetized accretion.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[41-42]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> Compared to non-magnetized accretion, infall rate is 2-5 times smaller depending on
> outer magnetization. In turn, gas density is 2-5 times smaller in the region close to
> the black hole, where synchrotron radiation emerges \citep{narayan}.
- au=1.7294394183409167e-06, eu=0.26116830110549927, au′=-0.0, eu′=0.03414195403456688
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[28.289474487304688]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 41 | 41 | 28.289474487304688 | Compared to non-magnetized accretion, infall rate is 2-5 times smaller depending on outer magnetization. |
- GPT 引用1: span=[41, 42], relevant=1

### 陈述 4
- The lower infall rate and gas density in the magnetized model are attributed to the inhibiting effect of the magnetic field on accretion.
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 5
- The lower gas density in the magnetized model is in qualitative agreement with models of the spectrum of Sgr A*, which produces relatively weak synchrotron radiation.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[43-45]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> Sgr A* produces relatively weak synchrotron \citep{narayan}. So, either gas density $n$
> or electron temperature $T_e$ or magnetic field $B$ are small in the inner flow or
> combination of factors works. Thus low gas density in magnetized model is in qualitative
> agreement with the results of modelling the spectrum.
- au=1.6015096662158612e-06, eu=0.28358200192451477, au′=-0.0, eu′=0.03807614743709564
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.263158798217773]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 43 | 43 | 25.263158798217773 | Sgr A* produces relatively weak synchrotron \citep{narayan}. |
- GPT 引用1: span=[43, 45], relevant=1

### 陈述 6
So in summary, the presence of a magnetic field leads to a lower infall rate and gas density compared to non-magnetized accretion, which is consistent with observations of Sgr A*. The magnetic field inhibits accretion and leads to lower densities in the inner flow region where synchrotron radiation emerges.
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

## multifieldqa_en_idx115 (idx=115, multifieldqa_en)
### 问题
According to the text, what is Toby Schindelbeck's observation about the police?

### 参考答案（数据集）
- ref1: Toby Schindelbeck's observation is that the police say they aren't paid enough to enforce the laws in the streets.

### 模型全文输出
```
<statement>Based on the text, Toby Schindelbeck's observation about the police is that they say they aren't paid enough to enforce the laws in the streets.<cite>[15-15]</cite></statement>
```

文档句索引范围: sid ∈ [1, 314], 共 314 句。
引用指标: R=0.0, P=0.0, F1=0.0
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=600, completion_tokens=15
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
Based on the text, Toby Schindelbeck's observation about the police is that they say they aren't paid enough to enforce the laws in the streets.
- GPT support_score=0; Rating: [[No support]]

**跨度** `[15-15]`
- au=0.6819406747817993, eu=0.1826922744512558, au′=0.6819398403167725, eu′=0.04279278963804245
- top_logits 方差 (pvariance, 有限 logit): 0.1558174915262498
- top_logits 数值: `[22.763158798217773, 21.973684310913086]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 16 | 16 | 22.763158798217773 | The police say they aren’t paid enough to enforce the laws in the streets, and now Dave Burkland says, he just doesn’t have to. |
| 2 | 15 | 15 | 21.973684310913086 | We’ve got a serious law enforcement problem in our town. |
- GPT 引用1: span=[15, 15], relevant=0

## multifieldqa_en_idx116 (idx=116, multifieldqa_en)
### 问题
Who is the program chair of this conference?

### 参考答案（数据集）
- ref1: Peter Denning.

### 模型全文输出
```
<statement>Based on the transcript, the program chair of this conference is Peter Denning.<cite>[12-12]</cite></statement>
```

文档句索引范围: sid ∈ [1, 431], 共 431 句。
引用指标: R=0.0, P=0.0, F1=0.0
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=584, completion_tokens=15
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
Based on the transcript, the program chair of this conference is Peter Denning.
- GPT support_score=0; Rating: [[No support]]

**跨度** `[12-12]`
- au=1.861421651483397e-06, eu=0.28787869215011597, au′=-0.0, eu′=0.038854800164699554
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.73684310913086]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 12 | 12 | 24.73684310913086 | To my left is Peter Denning, who is Chairman of the Computer Science Department at George Mason University and also the associate dean for computing. |
- GPT 引用1: span=[12, 12], relevant=0

## multifieldqa_en_idx117 (idx=117, multifieldqa_en)
### 问题
How does the conduction gap depend on the strain direction?

### 参考答案（数据集）
- ref1: Peaks occur at certain strain directions, while the gap is zero at others.

### 模型全文输出
```
<statement>Based on the tight binding calculations, the conduction gap in graphene strained junctions depends strongly on the direction of applied strain:<cite></cite></statement>

<statement>- The conduction gap is largest when the strain is applied along directions that cause the largest shift of Dirac points along the k_y axis. For tensile strain, this is the zigzag direction and for compressive strain, it is the armchair direction.<cite>[87-87][125-125][126-126]</cite></statement>

<statement>- The conduction gap is smallest when the strain is applied along directions that cause the smallest shift of Dirac points along the k_y axis. For tensile strain, this is the armchair direction and for compressive strain, it is the zigzag direction.<cite>[87-87][125-125][126-126]</cite></statement>

<statement>- There is a complete reversal of the preferred strain directions for opening a gap between tensile and compressive strain, depending on whether transport is along armchair or zigzag directions.<cite>[125-127][142-144]</cite></statement>

<statement>- The dependence of the conduction gap on strain direction is a result of the strain-induced deformation of the graphene lattice and bandstructure. The strain causes the Dirac points to shift in k space, and the conduction gap opens when the shifts in the two graphene sections are different.<cite>[103-103][106-106][109-109]</cite></statement>

<statement>- The calculations show that large conduction gaps of over 500 meV can be achieved with relatively small strains of 6% by carefully choosing the strain direction and transport direction. The properties of the conduction gap are very sensitive to the strain direction and transport direction, and a full picture of these dependencies has been presented.<cite>[126-127][142-144][154-157]</cite></statement>
```

文档句索引范围: sid ∈ [1, 186], 共 186 句。
引用指标: R=0.25, P=0.4166666666666667, F1=0.3125
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=7032, completion_tokens=142
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
Based on the tight binding calculations, the conduction gap in graphene strained junctions depends strongly on the direction of applied strain:
- GPT support_score=0; [[Yes]]   Need Citation: [[Yes]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 1
- The conduction gap is largest when the strain is applied along directions that cause the largest shift of Dirac points along the k_y axis. For tensile strain, this is the zigzag direction and for compressive strain, it is the armchair direction.
- GPT support_score=0; Rating: [[No support]]

**跨度** `[87-87]`
- au=1.7698992490768433, eu=0.07941482961177826, au′=1.7698991298675537, eu′=0.04921215772628784
- top_logits 方差 (pvariance, 有限 logit): 0.3832913509113496
- top_logits 数值: `[20.394737243652344, 19.60526466369629, 19.60526466369629, 18.947368621826172, 18.8157901763916, 18.552631378173828]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 118 | 118 | 20.394737243652344 | For instance, the conduction gap takes the values of $\sim$ 295, 172 and 323 meV for $\theta = 0$, $30^\circ$ and $90^\circ$, respectively. |
| 2 | 143 | 143 | 19.60526466369629 | In fact, the conduction gap in this case can reach a value as high as that of the case of $\phi = 0$ but has different $\theta$-dependence. |
| 3 | 117 | 117 | 19.60526466369629 | 3 shows that besides the strength of strain, the strain effect is also strongly dependent on the applied direction. |
| 4 | 142 | 142 | 18.947368621826172 | 5(c,d) where the transport is along the zigzag direction $\phi = 30^\circ$. |
| 5 | 125 | 125 | 18.8157901763916 | Let us first discuss the results obtained in the armchair case. |
| 6 | 87 | 87 | 18.552631378173828 | We extend our investigation to the case of compressive strain and find  (see in Fig. |

**跨度** `[125-125]`
- au=1.7701984643936157, eu=0.07941482961177826, au′=1.7701982259750366, eu′=0.049212172627449036
- top_logits 方差 (pvariance, 有限 logit): 0.15822172458471565
- top_logits 数值: `[19.86842155456543, 19.73684310913086, 19.473684310913086, 19.078948974609375, 18.947368621826172, 18.8157901763916]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 143 | 143 | 19.86842155456543 | In fact, the conduction gap in this case can reach a value as high as that of the case of $\phi = 0$ but has different $\theta$-dependence. |
| 2 | 88 | 88 | 19.73684310913086 | 2(b)) that (i) the same gap threshold of $\sigma \simeq 23 \%$ is observed but (ii) the preferred direction to open the gap under a compressive strain is the armchair, not the zigzag as the case of tensile strain. |
| 3 | 144 | 144 | 19.473684310913086 | In particular, the conduction gap has peaks at $\theta \approx 47^\circ$ and $133^\circ$ for tensile strain and at $\theta \approx 43^\circ$ and $137^\circ$ for compressive strain, where it is zero in the case of $\phi = 0$. |
| 4 | 173 | 173 | 19.078948974609375 | In particular, our calculations show that the preferred transport directions to achieve a large conduction gap are close to the armchair one. |
| 5 | 125 | 125 | 18.947368621826172 | Let us first discuss the results obtained in the armchair case. |
| 6 | 142 | 142 | 18.8157901763916 | 5(c,d) where the transport is along the zigzag direction $\phi = 30^\circ$. |

**跨度** `[126-126]`
- au=1.0823065042495728, eu=0.13944950699806213, au′=1.082305669784546, eu′=0.04636030271649361
- top_logits 方差 (pvariance, 有限 logit): 0.23468809224707204
- top_logits 数值: `[21.18421173095703, 20.526315689086914, 20.0]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 126 | 126 | 21.18421173095703 | Figs. |
| 2 | 143 | 143 | 20.526315689086914 | In fact, the conduction gap in this case can reach a value as high as that of the case of $\phi = 0$ but has different $\theta$-dependence. |
| 3 | 144 | 144 | 20.0 | In particular, the conduction gap has peaks at $\theta \approx 47^\circ$ and $133^\circ$ for tensile strain and at $\theta \approx 43^\circ$ and $137^\circ$ for compressive strain, where it is zero in the case of $\phi = 0$. |
- GPT 引用1: span=[87, 87], relevant=0
- GPT 引用2: span=[125, 126], relevant=0

### 陈述 2
- The conduction gap is smallest when the strain is applied along directions that cause the smallest shift of Dirac points along the k_y axis. For tensile strain, this is the armchair direction and for compressive strain, it is the zigzag direction.
- GPT support_score=0; Rating: [[No support]]

**跨度** `[87-87]`
- au=0.6807417273521423, eu=0.19537270069122314, au′=0.6807407736778259, eu′=0.046313218772411346
- top_logits 方差 (pvariance, 有限 logit): 0.35058935593406204
- top_logits 数值: `[21.18421173095703, 20.0]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 87 | 87 | 21.18421173095703 | We extend our investigation to the case of compressive strain and find  (see in Fig. |
| 2 | 125 | 125 | 20.0 | Let us first discuss the results obtained in the armchair case. |

**跨度** `[125-125]`
- au=1.95558504856308e-06, eu=0.3064514994621277, au′=-0.0, eu′=0.0423162542283535
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[22.631580352783203]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 125 | 125 | 22.631580352783203 | Let us first discuss the results obtained in the armchair case. |

**跨度** `[126-126]`
- au=1.9009210063813953e-06, eu=0.2957197427749634, au′=-0.0, eu′=0.04029691964387894
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.8157901763916]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 126 | 126 | 23.8157901763916 | Figs. |
- GPT 引用1: span=[87, 87], relevant=0
- GPT 引用2: span=[125, 126], relevant=0

### 陈述 3
- There is a complete reversal of the preferred strain directions for opening a gap between tensile and compressive strain, depending on whether transport is along armchair or zigzag directions.
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[125-127]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> Let us first discuss the results obtained in the armchair case. Figs. 5(a,b) show that
> (i) a large conduction gap up to about 500 meV can open with a strain of 6 $\%$ and (ii)
> again the conduction gap is strongly $\theta$-dependent, in particular, its peaks occur
> at $\theta = 0$ or $90^\circ$ while the gap is zero at $\theta \approx 47^\circ$ and
> $133^\circ$ for tensile strain and at $\theta \approx 43^\circ$ and $137^\circ$ for
> compressive strain.
- au=0.6811630129814148, eu=0.1928933709859848, au′=0.6811621785163879, eu′=0.045618243515491486
- top_logits 方差 (pvariance, 有限 logit): 0.1558174915262498
- top_logits 数值: `[21.3157901763916, 20.526315689086914]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 87 | 87 | 21.3157901763916 | We extend our investigation to the case of compressive strain and find  (see in Fig. |
| 2 | 125 | 125 | 20.526315689086914 | Let us first discuss the results obtained in the armchair case. |

**跨度** `[142-144]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> 5(c,d) where the transport is along the zigzag direction $\phi = 30^\circ$. In fact, the
> conduction gap in this case can reach a value as high as that of the case of $\phi = 0$
> but has different $\theta$-dependence. In particular, the conduction gap has peaks at
> $\theta \approx 47^\circ$ and $133^\circ$ for tensile strain and at $\theta \approx
> 43^\circ$ and $137^\circ$ for compressive strain, where it is zero in the case of $\phi
> = 0$.
- au=1.0824486017227173, eu=0.14022138714790344, au′=1.0824482440948486, eu′=0.04664483666419983
- top_logits 方差 (pvariance, 有限 logit): 0.026931436878738977
- top_logits 数值: `[20.657896041870117, 20.394737243652344, 20.263158798217773]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 142 | 142 | 20.657896041870117 | 5(c,d) where the transport is along the zigzag direction $\phi = 30^\circ$. |
| 2 | 139 | 139 | 20.394737243652344 | The transport along the armchair/zigzag directions is considered in (a,b)/(c,d), respectively. |
| 3 | 143 | 143 | 20.263158798217773 | In fact, the conduction gap in this case can reach a value as high as that of the case of $\phi = 0$ but has different $\theta$-dependence. |
- GPT 引用1: span=[125, 127], relevant=1
- GPT 引用2: span=[142, 144], relevant=0

### 陈述 4
- The dependence of the conduction gap on strain direction is a result of the strain-induced deformation of the graphene lattice and bandstructure. The strain causes the Dirac points to shift in k space, and the conduction gap opens when the shifts in the two graphene sections are different.
- GPT support_score=0.5; **Rating**: [[Partially supported]]

**跨度** `[103-103]`
- au=1.366729736328125, eu=0.11225994676351547, au′=1.366729497909546, eu′=0.04814697057008743
- top_logits 方差 (pvariance, 有限 logit): 0.6178584653589496
- top_logits 数值: `[20.526315689086914, 20.394737243652344, 19.60526466369629, 18.552631378173828]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 103 | 103 | 20.526315689086914 | Actually, the strain causes the lattice deformation and can result in the deformation of graphene bandstructure. |
| 2 | 102 | 102 | 20.394737243652344 | The appearance of this conduction gap, as mentioned previously, is due to the strain-induced shift of Dirac points and is explained as follows. |
| 3 | 11 | 11 | 19.60526466369629 | Indeed, strain engineering has been suggested to be an alternative approach to modulating efficiently the electronic properties of graphene nanomaterials. |
| 4 | 12 | 12 | 18.552631378173828 | In particular, the bandgap has periodic oscillations in the armchair GNRs \cite{ylu210} while the spin polarization at the ribbon edges (and also the bandgap) can be modulated by the strain in the zigzag cases. |

**跨度** `[106-106]`
- au=0.6815448999404907, eu=0.1818181425333023, au′=0.6815442442893982, eu′=0.04255318641662598
- top_logits 方差 (pvariance, 有限 logit): 0.6232684603028247
- top_logits 数值: `[23.289474487304688, 21.710527420043945]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 106 | 106 | 23.289474487304688 | As one can see, the shift of Dirac points leads to the situation where there is no value of $\kappa_y$, for which the energy gaps $E_{unstrain}^{gap}\left( {{\kappa_y}} \right)$ and $E_{strain}^{gap}\left( {{\kappa_y}} \right)$ are simultaneously equal to zero. |
| 2 | 105 | 105 | 21.710527420043945 | 4. |

**跨度** `[109-109]`
- au=1.3674870729446411, eu=0.10951007902622223, au′=1.3674864768981934, eu′=0.04688463732600212
- top_logits 方差 (pvariance, 有限 logit): 0.45446661700839286
- top_logits 数值: `[21.052631378173828, 20.921052932739258, 19.86842155456543, 19.473684310913086]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 109 | 109 | 21.052631378173828 | strained) graphene section but finite in the strained (resp. |
| 2 | 108 | 108 | 20.921052932739258 | For instance, the energy gap is zero (or small) in the unstrained (resp. |
| 3 | 128 | 128 | 19.86842155456543 | In principle, the conduction gap is larger if the shift of Dirac points in the $\kappa_y$-axis is larger, as discussed above about Figs. |
| 4 | 107 | 107 | 19.473684310913086 | This means that the transmission probability always shows a finite gap for any $\kappa_y$. |
- GPT 引用1: span=[103, 103], relevant=1
- GPT 引用2: span=[106, 106], relevant=1
- GPT 引用3: span=[109, 109], relevant=0

### 陈述 5
- The calculations show that large conduction gaps of over 500 meV can be achieved with relatively small strains of 6% by carefully choosing the strain direction and transport direction. The properties of the conduction gap are very sensitive to the strain direction and transport direction, and a full picture of these dependencies has been presented.
- GPT support_score=0.5; **Rating**: [[Partially supported]]

**跨度** `[126-127]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> Figs. 5(a,b) show that (i) a large conduction gap up to about 500 meV can open with a
> strain of 6 $\%$ and (ii) again the conduction gap is strongly $\theta$-dependent, in
> particular, its peaks occur at $\theta = 0$ or $90^\circ$ while the gap is zero at
> $\theta \approx 47^\circ$ and $133^\circ$ for tensile strain and at $\theta \approx
> 43^\circ$ and $137^\circ$ for compressive strain.
- au=1.7714741230010986, eu=0.07487683743238449, au′=1.7714742422103882, eu′=0.046313218772411346
- top_logits 方差 (pvariance, 有限 logit): 0.2063134964815213
- top_logits 数值: `[21.447368621826172, 20.789474487304688, 20.526315689086914, 20.526315689086914, 20.263158798217773, 20.0]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 125 | 125 | 21.447368621826172 | Let us first discuss the results obtained in the armchair case. |
| 2 | 126 | 126 | 20.789474487304688 | Figs. |
| 3 | 117 | 117 | 20.526315689086914 | 3 shows that besides the strength of strain, the strain effect is also strongly dependent on the applied direction. |
| 4 | 101 | 101 | 20.526315689086914 | However, similar to that reported in \cite{hung14}, a significant conduction-gap of a few hundreds meV can open in the unstrained/strained graphene junctions. |
| 5 | 178 | 178 | 20.263158798217773 | It has been shown that due to the strain-induced deformation of graphene lattice and hence of graphene bandstructure, a finite conduction gap higher than 500 meV can be achieved for a strain of only 6 $\%$. |
| 6 | 118 | 118 | 20.0 | For instance, the conduction gap takes the values of $\sim$ 295, 172 and 323 meV for $\theta = 0$, $30^\circ$ and $90^\circ$, respectively. |

**跨度** `[142-144]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> 5(c,d) where the transport is along the zigzag direction $\phi = 30^\circ$. In fact, the
> conduction gap in this case can reach a value as high as that of the case of $\phi = 0$
> but has different $\theta$-dependence. In particular, the conduction gap has peaks at
> $\theta \approx 47^\circ$ and $133^\circ$ for tensile strain and at $\theta \approx
> 43^\circ$ and $137^\circ$ for compressive strain, where it is zero in the case of $\phi
> = 0$.
- au=1.769271731376648, eu=0.07966455817222595, au′=1.7692716121673584, eu′=0.04937201738357544
- top_logits 方差 (pvariance, 有限 logit): 0.7887034975373253
- top_logits 数值: `[20.394737243652344, 20.0, 19.86842155456543, 18.947368621826172, 18.157896041870117, 18.157896041870117]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 178 | 178 | 20.394737243652344 | It has been shown that due to the strain-induced deformation of graphene lattice and hence of graphene bandstructure, a finite conduction gap higher than 500 meV can be achieved for a strain of only 6 $\%$. |
| 2 | 143 | 143 | 20.0 | In fact, the conduction gap in this case can reach a value as high as that of the case of $\phi = 0$ but has different $\theta$-dependence. |
| 3 | 142 | 142 | 19.86842155456543 | 5(c,d) where the transport is along the zigzag direction $\phi = 30^\circ$. |
| 4 | 128 | 128 | 18.947368621826172 | In principle, the conduction gap is larger if the shift of Dirac points in the $\kappa_y$-axis is larger, as discussed above about Figs. |
| 5 | 133 | 133 | 18.157896041870117 | Hence, the zero conduction gap is obtained if \begin{equation*} \frac{{t_3^2 - {{\left( {{t_1} - {t_2}} \right)}^2}}}{{4{t_1}{t_2}}} = \frac{1}{4} \end{equation*} Additionally, it is observed that the effects of a strain $\{\sigma,\theta\}$ are qualitatively similar to those of a strain $\{-\sigma,\theta+90^\circ\}$, i.e., the peaks and zero values of conduction gap are obtained at the same $\theta$ in these two situations. |
| 6 | 177 | 177 | 18.157896041870117 | \section{Conclusion}  Based on the tight binding calculations, we have investigated the effects of uniaxial strain on the transport properties of graphene strained junctions and discuss systematically the possibilities of achieving a large conduction gap with respect to the strain, its applied direction and the transport direction. |

**跨度** `[154-157]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> Next, we investigate the conduction gap with respect to different transport directions
> $\phi$. We display a ($\theta,\phi$)-map of conduction gap for $\sigma = 4 \%$ in Fig. 6
> and, in the top, an additional diagram illustrating the rotation of Dirac points in the
> $k-$space with the change in the transport direction. It is clearly shown that (i) a
> similar scale of conduction gap is obtained for all different transport directions, (ii)
> there is a smooth and continuous shift of $E_{cond.gap}-\theta$ behavior when varying
> $\phi$, and (iii) the same behavior of $E_{cond.gap}$ is also observed when comparing
> the two transport directions of $\phi$ and $\phi+30^\circ$, similarly to the comparison
> above between $\phi = 0^\circ$ and $30^\circ$.
- au=1.7711198329925537, eu=0.07509881258010864, au′=1.7711200714111328, eu′=0.046454768627882004
- top_logits 方差 (pvariance, 有限 logit): 0.4559094621865572
- top_logits 数值: `[21.973684310913086, 20.526315689086914, 20.394737243652344, 20.263158798217773, 20.0, 20.0]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 154 | 154 | 21.973684310913086 | Next, we investigate the conduction gap with respect to different transport directions $\phi$. |
| 2 | 173 | 173 | 20.526315689086914 | In particular, our calculations show that the preferred transport directions to achieve a large conduction gap are close to the armchair one. |
| 3 | 178 | 178 | 20.394737243652344 | It has been shown that due to the strain-induced deformation of graphene lattice and hence of graphene bandstructure, a finite conduction gap higher than 500 meV can be achieved for a strain of only 6 $\%$. |
| 4 | 147 | 147 | 20.263158798217773 | On the one hand, based on the analyses above for $\phi = 0$, we find that for a given strength of strain, a maximum shift of Dirac points along the $k_y$-axis corresponds to a minimum along the $k_x$-one and vice versa when varying the strain direction $\theta$. |
| 5 | 155 | 155 | 20.0 | We display a ($\theta,\phi$)-map of conduction gap for $\sigma = 4 \%$ in Fig. |
| 6 | 157 | 157 | 20.0 | It is clearly shown that (i) a similar scale of conduction gap is obtained for all different transport directions, (ii) there is a smooth and continuous shift of $E_{cond.gap}-\theta$ behavior when varying $\phi$, and (iii) the same behavior of $E_{cond.gap}$ is also observed when comparing the two transport directions of $\phi$ and $\phi+30^\circ$, similarly to the comparison above between $\phi = 0^\circ$ and $30^\circ$. |
- GPT 引用1: span=[126, 127], relevant=1
- GPT 引用2: span=[142, 144], relevant=1
- GPT 引用3: span=[154, 157], relevant=0

## multifieldqa_en_idx118 (idx=118, multifieldqa_en)
### 问题
When was the paper published?

### 参考答案（数据集）
- ref1: The paper was published on 7 March 2023.

### 模型全文输出
```
<statement>Based on the information provided, the paper was published on 7 March 2023.<cite>[1-1]</cite></statement>
```

文档句索引范围: sid ∈ [1, 129], 共 129 句。
引用指标: R=1.0, P=1.0, F1=1.0
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=774, completion_tokens=13
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
Based on the information provided, the paper was published on 7 March 2023.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[1-1]`
- au=1.6121624639708898e-06, eu=0.28571420907974243, au′=-0.0, eu′=0.038461536169052124
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.0]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 1 | 1 | 25.0 | Paper Info  Title: Interpretable reduced-order modeling with time-scale separation Interpretable reduced-order modeling with time-scale separation Publish Date: 7 March 2023 Author List: Sebastian Kaltenbach (from CSE-Lab, ETH Zurich, Harvard SEAS), Phaedon-Stelios Koutsourelakis (from CSE-Lab, ETH Zurich, Harvard SEAS), Petros Koumoutsakos (from CSE-Lab, ETH Zurich, Harvard SEAS), Harvard Seas (from CSE-Lab, ETH Zurich, Harvard SEAS)  Figure  FIG. |
- GPT 引用1: span=[1, 1], relevant=1

## multifieldqa_en_idx119 (idx=119, multifieldqa_en)
### 问题
What are the three subsets into which the parameter space V is divided?

### 参考答案（数据集）
- ref1: The three subsets are V+, V0, and V-, determined by the Kullback-Leibler information distance.

### 模型全文输出
```
<statement>Based on the paper, the parameter space V is divided into three subsets as follows:<cite></cite></statement>

<statement>V+ : This subset contains parameters v where the Kullback-Leibler information distance between the post-change distribution P_v and the pre-change distribution P_0 is less than the distance between P_v and the reference distribution P_1. In other words, P_v is closer to P_0 than to P_1 according to the Kullback-Leibler information distance.<cite>[65-67]</cite></statement>

<statement>V0 : This subset contains parameters v where the Kullback-Leibler information distance between P_v and P_0 is equal to the distance between P_v and P_1. In other words, P_v is equally close to P_0 and P_1 according to the Kullback-Leibler information distance.<cite>[65-65]</cite></statement>

<statement>V- : This subset contains parameters v where the Kullback-Leibler information distance between P_v and P_0 is greater than the distance between P_v and P_1. In other words, P_v is closer to P_1 than to P_0 according to the Kullback-Leibler information distance.<cite>[65-67]</cite></statement>

<statement>So in summary, V+ contains parameters v where P_v is "smaller" changes from P_0, V0 contains parameters v where P_v is "medium" changes from P_0, and V- contains parameters v where P_v is "larger" changes from P_0 according to the Kullback-Leibler information distance.<cite></cite></statement>
```

文档句索引范围: sid ∈ [1, 98], 共 98 句。
引用指标: R=0.1, P=0.3333333333333333, F1=0.15384615384615383
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=3565, completion_tokens=64
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
Based on the paper, the parameter space V is divided into three subsets as follows:
- GPT support_score=0; [[Yes]]   Need Citation: [[Yes]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 1
V+ : This subset contains parameters v where the Kullback-Leibler information distance between the post-change distribution P_v and the pre-change distribution P_0 is less than the distance between P_v and the reference distribution P_1. In other words, P_v is closer to P_0 than to P_1 according to the Kullback-Leibler information distance.
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[65-67]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> Next we shall divide the parameter space V into three subsets V + , V 0 and V − by the
> Kullback-Leibler information distance. Let and are two Kullblak-Leibler information
> distances between P v , P v 0 and P v , P v 1 . Since I(p|q) = 0 if and only if p = q,
> where p and q are two probability measures, it follows that , it means that P v is
> closer to P v 0 than to P v 1 according to the Kullblak-Leibler information distance.
- au=0.6822620034217834, eu=0.17840370535850525, au′=0.682261049747467, eu′=0.041621025651693344
- top_logits 方差 (pvariance, 有限 logit): 0.1558167386256173
- top_logits 数值: `[23.421052932739258, 22.631580352783203]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 65 | 65 | 23.421052932739258 | Next we shall divide the parameter space V into three subsets V + , V 0 and V − by the Kullback-Leibler information distance. |
| 2 | 67 | 67 | 22.631580352783203 | Since I(p\|q) = 0 if and only if p = q, where p and q are two probability measures, it follows that , it means that P v is closer to P v 0 than to P v 1 according to the Kullblak-Leibler information distance. |
- GPT 引用1: span=[65, 67], relevant=1

### 陈述 2
V0 : This subset contains parameters v where the Kullback-Leibler information distance between P_v and P_0 is equal to the distance between P_v and P_1. In other words, P_v is equally close to P_0 and P_1 according to the Kullback-Leibler information distance.
- GPT support_score=0; Rating: [[No support]]

**跨度** `[65-65]`
- au=1.6396279534092173e-06, eu=0.2911876440048218, au′=-0.0, eu′=0.039460018277168274
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.342105865478516]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 65 | 65 | 24.342105865478516 | Next we shall divide the parameter space V into three subsets V + , V 0 and V − by the Kullback-Leibler information distance. |
- GPT 引用1: span=[65, 65], relevant=0

### 陈述 3
V- : This subset contains parameters v where the Kullback-Leibler information distance between P_v and P_0 is greater than the distance between P_v and P_1. In other words, P_v is closer to P_1 than to P_0 according to the Kullback-Leibler information distance.
- GPT support_score=0; Rating: [[No support]]

**跨度** `[65-67]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> Next we shall divide the parameter space V into three subsets V + , V 0 and V − by the
> Kullback-Leibler information distance. Let and are two Kullblak-Leibler information
> distances between P v , P v 0 and P v , P v 1 . Since I(p|q) = 0 if and only if p = q,
> where p and q are two probability measures, it follows that , it means that P v is
> closer to P v 0 than to P v 1 according to the Kullblak-Leibler information distance.
- au=0.6821103692054749, eu=0.17798590660095215, au′=0.6821095943450928, eu′=0.04150736704468727
- top_logits 方差 (pvariance, 有限 logit): 0.35058935593406204
- top_logits 数值: `[23.68421173095703, 22.5]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 65 | 65 | 23.68421173095703 | Next we shall divide the parameter space V into three subsets V + , V 0 and V − by the Kullback-Leibler information distance. |
| 2 | 67 | 67 | 22.5 | Since I(p\|q) = 0 if and only if p = q, where p and q are two probability measures, it follows that , it means that P v is closer to P v 0 than to P v 1 according to the Kullblak-Leibler information distance. |
- GPT 引用1: span=[65, 67], relevant=0

### 陈述 4
So in summary, V+ contains parameters v where P_v is "smaller" changes from P_0, V0 contains parameters v where P_v is "medium" changes from P_0, and V- contains parameters v where P_v is "larger" changes from P_0 according to the Kullback-Leibler information distance.
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

## multifieldqa_en_idx120 (idx=120, multifieldqa_en)
### 问题
What happens to Ngotho after he attacks Jacobo at a workers' strike?

### 参考答案（数据集）
- ref1: After attacking Jacobo at a workers' strike, Ngotho loses his job and Njoroge's family is forced to move.

### 模型全文输出
```
<statement>After Ngotho attacks Jacobo at a workers' strike, he loses his job and the family is forced to move.<cite>[21-23][25-25]</cite></statement>

<statement>Jacobo survives but swears revenge.<cite>[24-24]</cite></statement>

<statement>Ngotho's position in the family weakens and he becomes depressed, realizing he spent his life waiting for a prophecy to come true rather than fighting for independence.<cite>[61-61]</cite></statement>
```

文档句索引范围: sid ∈ [1, 93], 共 93 句。
引用指标: R=1.0, P=0.75, F1=0.8571428571428571
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=2185, completion_tokens=47
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
After Ngotho attacks Jacobo at a workers' strike, he loses his job and the family is forced to move.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[21-23]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> Suddenly, the white police inspector brings Jacobo to the gathering to pacify the native
> people. Jacobo tries to put an end to the strike. Ngotho attacks Jacobo, and the result
> is a riot where two people are killed.
- au=1.3686766624450684, eu=0.10468317568302155, au′=1.368676781654358, eu′=0.04467959702014923
- top_logits 方差 (pvariance, 有限 logit): 0.22939770175503327
- top_logits 数值: `[22.10526466369629, 21.447368621826172, 21.18421173095703, 20.789474487304688]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 23 | 23 | 22.10526466369629 | Ngotho attacks Jacobo, and the result is a riot where two people are killed. |
| 2 | 18 | 18 | 21.447368621826172 | Ngotho is ambivalent about participating in the strike because he fears he will lose his job. |
| 3 | 21 | 21 | 21.18421173095703 | Suddenly, the white police inspector brings Jacobo to the gathering to pacify the native people. |
| 4 | 22 | 22 | 20.789474487304688 | Jacobo tries to put an end to the strike. |

**跨度** `[25-25]`
- au=1.9246485862822738e-06, eu=0.3003951609134674, au′=-0.0, eu′=0.04117009416222572
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.289474487304688]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 25 | 25 | 23.289474487304688 | Ngotho loses his job and Njoroge’s family is forced to move. |
- GPT 引用1: span=[21, 23], relevant=0
- GPT 引用2: span=[25, 25], relevant=1

### 陈述 1
Jacobo survives but swears revenge.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[24-24]`
- au=0.6826422214508057, eu=0.17431187629699707, au′=0.6826412677764893, eu′=0.04051172360777855
- top_logits 方差 (pvariance, 有限 logit): 0.0692520511447583
- top_logits 数值: `[23.947368621826172, 23.421052932739258]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 24 | 24 | 23.947368621826172 | Jacobo survives and swears revenge. |
| 2 | 23 | 23 | 23.421052932739258 | Ngotho attacks Jacobo, and the result is a riot where two people are killed. |
- GPT 引用1: span=[24, 24], relevant=1

### 陈述 2
Ngotho's position in the family weakens and he becomes depressed, realizing he spent his life waiting for a prophecy to come true rather than fighting for independence.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[61-61]`
- au=0.6820657253265381, eu=0.17757005989551544, au′=0.6820648908615112, eu′=0.04139433056116104
- top_logits 方差 (pvariance, 有限 logit): 0.4328259470721605
- top_logits 数值: `[23.8157901763916, 22.5]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 61 | 61 | 23.8157901763916 | Over the course of the book his position as the central power of the family weakened, to the point where his self-realization that he has spent his whole life waiting for the prophecy (that proclaims the blacks will be returned their land) to come true rather than fighting for Kenyan independence, leads to his depression. |
| 2 | 60 | 60 | 22.5 | He is fired and the family is forced to move to another section of the country. |
- GPT 引用1: span=[61, 61], relevant=1

## multifieldqa_en_idx121 (idx=121, multifieldqa_en)
### 问题
Where is the club's headquarters located?

### 参考答案（数据集）
- ref1: The club's headquarters are located on Jivani Street 2 of the Malatia-Sebastia District, Yerevan.

### 模型全文输出
```
<statement>Based on the information provided, the club's headquarters are located on Jivani Street 2 of the Malatia-Sebastia District, Yerevan.<cite>[22-22]</cite></statement>
```

文档句索引范围: sid ∈ [1, 38], 共 38 句。
引用指标: R=1.0, P=1.0, F1=1.0
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=608, completion_tokens=13
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
Based on the information provided, the club's headquarters are located on Jivani Street 2 of the Malatia-Sebastia District, Yerevan.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[22-22]`
- au=1.6742565094318707e-06, eu=0.29803910851478577, au′=-0.0, eu′=0.04072882607579231
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.55263328552246]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 22 | 22 | 23.55263328552246 | The club headquarters are located on Jivani Street 2 of the Malatia-Sebastia District, Yerevan. |
- GPT 引用1: span=[22, 22], relevant=1

## multifieldqa_en_idx122 (idx=122, multifieldqa_en)
### 问题
How do the runtimes and iteration counts of NFPA and FPSA compare to GMRES and DSA in the numerical experiments?

### 参考答案（数据集）
- ref1: NFPA and FPSA greatly outperform GMRES and DSA.

### 模型全文输出
```
<statement>Based on the numerical experiments, NFPA and FPSA have much faster runtimes and require fewer iterations compared to GMRES and DSA for the tested problems:<cite></cite></statement>

<statement>- For problems with highly forward-peaked scattering kernels like SRK and EK, NFPA and FPSA show a significant reduction in runtime and iteration count as the problem becomes more forward-peaked. In contrast, GMRES and DSA require many more iterations as the FP limit is approached.<cite>[92-94][111-112]</cite></statement>

<statement>- For problems with anisotropic scattering like HGK, NFPA and FPSA still outperform GMRES and DSA, although not as drastically as for forward-peaked problems. Both NFPA and FPSA require more time and iterations as the problem becomes more anisotropic, which is expected since HGK does not have a valid FP limit.<cite>[122-122][124-124][125-125]</cite></statement>

<statement>- NFPA and FPSA have similar runtimes and iteration counts for the tested problems. FPSA is slightly faster due to fewer calculations per iteration, but NFPA has the advantage of preserving angular moments in both the HO and LO equations.<cite>[131-131]</cite></statement>

<statement>- Overall, NFPA and FPSA show a significant speedup compared to GMRES and DSA for the tested problems, especially for forward-peaked scattering. Their performance suggests that using a modified FP equation that accounts for higher-order moments and preserves angular moments can be an effective way to accelerate convergence for transport problems with highly forward-peaked scattering.<cite>[92-97][111-113][122-125]</cite></statement>
```

文档句索引范围: sid ∈ [1, 144], 共 144 句。
引用指标: R=0.5, P=0.875, F1=0.6363636363636364
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=6628, completion_tokens=96
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
Based on the numerical experiments, NFPA and FPSA have much faster runtimes and require fewer iterations compared to GMRES and DSA for the tested problems:
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 1
- For problems with highly forward-peaked scattering kernels like SRK and EK, NFPA and FPSA show a significant reduction in runtime and iteration count as the problem becomes more forward-peaked. In contrast, GMRES and DSA require many more iterations as the FP limit is approached.
- GPT support_score=1; **Rating: [[Fully supported]]**   **

**跨度** `[92-94]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> We see that NFPA and FPSA tremendously outperform GMRES and DSA in runtime for all
> cases. FPSA is a simpler method than NFPA, requiring less calculations per iteration;
> therefore, it is expected that it outperforms NFPA in runtime. We see a reduction in
> runtime and iterations for FPSA and NFPA as the FP limit is approached, with DSA and
> GMRES requiring many more iterations by comparison as $\eta$ approaches 0.
- au=0.6814520359039307, eu=0.18536581099033356, au′=0.6814513206481934, eu′=0.04352806136012077
- top_logits 方差 (pvariance, 有限 logit): 0.4328259470721605
- top_logits 数值: `[22.631580352783203, 21.3157901763916]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 92 | 92 | 22.631580352783203 | We see that NFPA and FPSA tremendously outperform GMRES and DSA in runtime for all cases. |
| 2 | 94 | 94 | 21.3157901763916 | We see a reduction in runtime and iterations for FPSA and NFPA as the FP limit is approached, with DSA and GMRES requiring many more iterations by comparison as $\eta$ approaches 0. |

**跨度** `[111-112]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> We see a similar trend with the EK as seen with SRK. Smaller $\Delta$ values lead to a
> reduction in runtime and iterations for NFPA and FPSA, which greatly outperform DSA and
> GMRES in both categories.
- au=1.6742565094318707e-06, eu=0.29803910851478577, au′=-0.0, eu′=0.04072882607579231
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.55263328552246]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 111 | 111 | 23.55263328552246 | We see a similar trend with the EK as seen with SRK. |
- GPT 引用1: span=[92, 94], relevant=1
- GPT 引用2: span=[111, 112], relevant=1

### 陈述 2
- For problems with anisotropic scattering like HGK, NFPA and FPSA still outperform GMRES and DSA, although not as drastically as for forward-peaked problems. Both NFPA and FPSA require more time and iterations as the problem becomes more anisotropic, which is expected since HGK does not have a valid FP limit.
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[122-122]`
- au=1.6567056491112453e-06, eu=0.2945735454559326, au′=-0.0, eu′=0.04008438438177109
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.947368621826172]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 122 | 122 | 23.947368621826172 | Contrary to what happened in those cases, both solvers require more time and iterations as the problem becomes more anisotropic. |

**跨度** `[124-124]`
- au=1.5508946944464697e-06, eu=0.27338120341300964, au′=-0.0, eu′=0.03625953570008278
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[26.578948974609375]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 124 | 124 | 26.578948974609375 | However, both NFPA and FPSA continue to greatly outperform GMRES and DSA. |

**跨度** `[125-125]`
- au=1.7430054413125617e-06, eu=0.31147530674934387, au′=-0.0, eu′=0.04328017681837082
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[22.10526466369629]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 125 | 125 | 22.10526466369629 | Moreover, NFPA outperforms FPSA in iteration count for problem 1. |
- GPT 引用1: span=[122, 122], relevant=1
- GPT 引用2: span=[124, 125], relevant=0

### 陈述 3
- NFPA and FPSA have similar runtimes and iteration counts for the tested problems. FPSA is slightly faster due to fewer calculations per iteration, but NFPA has the advantage of preserving angular moments in both the HO and LO equations.
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[131-131]`
- au=0.6821879744529724, eu=0.18138420581817627, au′=0.6821870803833008, eu′=0.042434390634298325
- top_logits 方差 (pvariance, 有限 logit): 0.004328347309638048
- top_logits 数值: `[22.631580352783203, 22.5]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 131 | 131 | 22.631580352783203 | The runtime and iterations of NFPA and FPSA were shown to be similar. |
| 2 | 92 | 92 | 22.5 | We see that NFPA and FPSA tremendously outperform GMRES and DSA in runtime for all cases. |
- GPT 引用1: span=[131, 131], relevant=1

### 陈述 4
- Overall, NFPA and FPSA show a significant speedup compared to GMRES and DSA for the tested problems, especially for forward-peaked scattering. Their performance suggests that using a modified FP equation that accounts for higher-order moments and preserves angular moments can be an effective way to accelerate convergence for transport problems with highly forward-peaked scattering.
- GPT support_score=0.5; **Rating: [[Partially supported]]**  **

**跨度** `[92-97]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> We see that NFPA and FPSA tremendously outperform GMRES and DSA in runtime for all
> cases. FPSA is a simpler method than NFPA, requiring less calculations per iteration;
> therefore, it is expected that it outperforms NFPA in runtime. We see a reduction in
> runtime and iterations for FPSA and NFPA as the FP limit is approached, with DSA and
> GMRES requiring many more iterations by comparison as $\eta$ approaches 0. An advantage
> that NFPA offers is that the angular moments of the flux in the LO equation will remain
> consistent with those of the transport equation even as a problem becomes less forward-
> peaked. On the other hand, the moments found using only the FP equation and source
> iteration lose accuracy. To illustrate this, Problem 1 was tested using different
> Screened Rutherford Kernels with increasing $\eta$ parameters.
- au=1.729960786178708e-06, eu=0.3089430034160614, au′=-0.0, eu′=0.04279278963804245
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[22.36842155456543]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 92 | 92 | 22.36842155456543 | We see that NFPA and FPSA tremendously outperform GMRES and DSA in runtime for all cases. |

**跨度** `[111-113]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> We see a similar trend with the EK as seen with SRK. Smaller $\Delta$ values lead to a
> reduction in runtime and iterations for NFPA and FPSA, which greatly outperform DSA and
> GMRES in both categories. \begin{table}[h] \begin{center} \scalebox{0.8}{
> \begin{tabular}{c || c || c || c} \hline Parameter & Solver & Runtime (s) & Iterations
> \\ \hline \hline \multirow{4}{*}{$\Delta = 10^{-5}$} & GMRES & 196 & 142 \\ & DSA & 3110
> & 70140 \\ & FPSA & 0.514 & 11 \\ & NFPA & 0.630 & 11 \\\hline \multirow{4}{*}{$\Delta =
> 10^{-6}$} & GMRES & 156 & 132 \\ & DSA & 3120 & 70758 \\ & FPSA & 0.388 & 7 \\ & NFPA &
> 0.393 & 7 \\ \hline \multirow{4}{*}{$\Delta = 10^{-7}$} & GMRES & 81 & 127 \\ & DSA &
> 3120 & 70851  \\ & FPSA & 0.292 & 6 \\ & NFPA & 0.318 & 6 \\ \hline \end{tabular}}
> \end{center} \caption{Runtime and Iteration Counts for Problem 1 with EK}
> \label{Expresults1} \end{table} \begin{table}[h] \begin{center} \scalebox{0.8}{
> \begin{tabular}{c || c || c || c} \hline Parameter & Solver & Runtime (s) & Iterations
> \\ \hline \hline \multirow{4}{*}{$\Delta = 10^{-5}$} & GMRES & 110 & 73 \\ & DSA & 1455
> & 33033 \\ & FPSA & 0.492 & 10 \\ & NFPA & 0.613 & 10 \\ \hline \multirow{4}{*}{$\Delta
> = 10^{-6}$} & GMRES & 82.7 & 79 \\ & DSA & 1470 & 33309 \\ & FPSA & 0.358 & 7 \\ & NFPA
> & 0.431 & 7 \\ \hline \multirow{4}{*}{$\Delta = 10^{-7}$} & GMRES & 56.8 & 90 \\ & DSA &
> 1470 & 33339 \\ & FPSA & 0.273 & 5 \\ & NFPA & 0.319 & 5 \\ \hline \end{tabular}}
> \end{center} \caption{Runtime and Iteration Counts for Problem 2 with EK}
> \label{Expresults2} \end{table}
>
> \subsubsection{HGK: Henyey-Greenstein Kernel}
>
> The Henyey-Greenstein Kernel \cite{HGK,JapanFPSA} is most commonly used in light
> transport in clouds.
- au=1.6742565094318707e-06, eu=0.29803910851478577, au′=-0.0, eu′=0.04072882607579231
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.55263328552246]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 111 | 111 | 23.55263328552246 | We see a similar trend with the EK as seen with SRK. |

**跨度** `[122-125]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> Contrary to what happened in those cases, both solvers require more time and iterations
> as the problem becomes more anisotropic. This is somewhat expected, due to HGK not
> having a valid Fokker-Planck limit. However, both NFPA and FPSA continue to greatly
> outperform GMRES and DSA. Moreover, NFPA outperforms FPSA in iteration count for problem
> 1.
- au=0.6825187802314758, eu=0.17551958560943604, au′=0.6825180053710938, eu′=0.04083825275301933
- top_logits 方差 (pvariance, 有限 logit): 0.10820617305944324
- top_logits 数值: `[23.8157901763916, 23.157896041870117]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 124 | 124 | 23.8157901763916 | However, both NFPA and FPSA continue to greatly outperform GMRES and DSA. |
| 2 | 122 | 122 | 23.157896041870117 | Contrary to what happened in those cases, both solvers require more time and iterations as the problem becomes more anisotropic. |
- GPT 引用1: span=[92, 97], relevant=1
- GPT 引用2: span=[111, 113], relevant=1
- GPT 引用3: span=[122, 125], relevant=1

## multifieldqa_en_idx123 (idx=123, multifieldqa_en)
### 问题
What are some reasons for the lack of data sharing in archaeobotany?

### 参考答案（数据集）
- ref1: Technological limitations, resistance to exposing data to scrutiny, and desire to hold onto data for personal use.

### 模型全文输出
```
<statement>Based on the review, some key reasons for the lack of data sharing in archaeobotany include:<cite></cite></statement>

<statement>- Technological limitations - Some archaeologists may not have the technical knowledge or skills to make their data available in digital formats that facilitate reuse.<cite>[189-189]</cite></statement>

<statement>- Resistance to sharing data - Some archaeologists may be hesitant to share data due to concerns about exposing it to scrutiny, losing opportunities for analysis before others use it, or losing control over how it is presented.<cite>[189-189]</cite></statement>

<statement>- Emphasis on creating new data - In archaeobotany, creating new datasets through primary analysis is seen as an important aspect of training and career progression. The perceived value of reusing previously published datasets may be low.<cite>[191-192]</cite></statement>

<statement>- Data quality concerns - Some archaeologists may be concerned about the quality or completeness of their data, and feel it is not ready for sharing.<cite></cite></statement>

<statement>- Commercial confidentiality - In the case of developer-funded data, issues of commercial confidentiality may limit the sharing of data.<cite>[235-235]</cite></statement>

<statement>- Access to digital archiving resources - Some archaeologists may lack access to funds, training, or knowledge to deposit data in digital repositories.<cite>[239-239]</cite></statement>

<statement>- Career pressures - There may be pressure to publish results in excavation monographs and journal articles for career progression, which can take time and may compete with data sharing.<cite>[237-237]</cite></statement>

<statement>- Equity issues - Differences in access to resources and training may lead to disparities in data sharing and reuse across the discipline.<cite>[239-239]</cite></statement>

<statement>In summary, a complex mix of technical, cultural, and structural factors contribute to the low levels of data sharing in archaeobotany. Addressing these issues will require improvements in training, infrastructure, and cultural norms around data sharing.<cite></cite></statement>
```

文档句索引范围: sid ∈ [1, 250], 共 250 句。
引用指标: R=0.8, P=1.0, F1=0.888888888888889
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=6332, completion_tokens=126
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
Based on the review, some key reasons for the lack of data sharing in archaeobotany include:
- GPT support_score=1; [[No]]   Need Citation: [[No]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 1
- Technological limitations - Some archaeologists may not have the technical knowledge or skills to make their data available in digital formats that facilitate reuse.
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[189-189]`
- au=1.7108611700678011e-06, eu=0.3052207827568054, au′=-0.0, eu′=0.04208194464445114
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[22.763158798217773]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 189 | 189 | 22.763158798217773 | Reasons suggested for a lack of data sharing within archaeology include technological limitations, and resistance amongst some archaeologists to making their data available due to cautions of exposing data to scrutiny, lost opportunities of analysis before others use it and loss of ‘capital’ of data (Moore & Richards 2015: 34–35). |
- GPT 引用1: span=[189, 189], relevant=1

### 陈述 2
- Resistance to sharing data - Some archaeologists may be hesitant to share data due to concerns about exposing it to scrutiny, losing opportunities for analysis before others use it, or losing control over how it is presented.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[189-189]`
- au=1.8040525446849642e-06, eu=0.2763635516166687, au′=-0.0, eu′=0.03678605705499649
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[26.18421173095703]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 189 | 189 | 26.18421173095703 | Reasons suggested for a lack of data sharing within archaeology include technological limitations, and resistance amongst some archaeologists to making their data available due to cautions of exposing data to scrutiny, lost opportunities of analysis before others use it and loss of ‘capital’ of data (Moore & Richards 2015: 34–35). |
- GPT 引用1: span=[189, 189], relevant=1

### 陈述 3
- Emphasis on creating new data - In archaeobotany, creating new datasets through primary analysis is seen as an important aspect of training and career progression. The perceived value of reusing previously published datasets may be low.
- GPT support_score=1; **Rating: [[Fully supported]]**   **

**跨度** `[191-192]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> Another factor to consider is the emphasis on the creation of new data through
> archaeological research (Huvila 2016). The creation of a new archaeobotanical dataset
> through primary analysis is a key form of training in archaeobotany, and the perception
> of the value of the reuse of other previously published archaeobotanical journals may be
> low, hence not encouraging the sharing of well-documented datasets.
- au=0.6825781464576721, eu=0.17471259832382202, au′=0.68257737159729, eu′=0.04061998426914215
- top_logits 方差 (pvariance, 有限 logit): 0.10820617305944324
- top_logits 数值: `[23.947368621826172, 23.289474487304688]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 191 | 191 | 23.947368621826172 | Another factor to consider is the emphasis on the creation of new data through archaeological research (Huvila 2016). |
| 2 | 192 | 192 | 23.289474487304688 | The creation of a new archaeobotanical dataset through primary analysis is a key form of training in archaeobotany, and the perception of the value of the reuse of other previously published archaeobotanical journals may be low, hence not encouraging the sharing of well-documented datasets. |
- GPT 引用1: span=[191, 192], relevant=1

### 陈述 4
- Data quality concerns - Some archaeologists may be concerned about the quality or completeness of their data, and feel it is not ready for sharing.
- GPT support_score=0; [[Yes]]   Need Citation: [[Yes]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 5
- Commercial confidentiality - In the case of developer-funded data, issues of commercial confidentiality may limit the sharing of data.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[235-235]`
- au=1.8090482853949652e-06, eu=0.27737218141555786, au′=-0.0, eu′=0.03696497529745102
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[26.05263328552246]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 235 | 235 | 26.05263328552246 | In the case of developer-funded derived data, issues of commercial confidentiality could limit the sharing of data. |
- GPT 引用1: span=[235, 235], relevant=1

### 陈述 6
- Access to digital archiving resources - Some archaeologists may lack access to funds, training, or knowledge to deposit data in digital repositories.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[239-239]`
- au=1.5557670849375427e-06, eu=0.27436816692352295, au′=-0.0, eu′=0.03643336147069931
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[26.447368621826172]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 239 | 239 | 26.447368621826172 | More broadly, one important aspect is issues of equity in access to digital archiving resources (Wright & Richards 2018), such as differential access to funds, training and knowledge. |
- GPT 引用1: span=[239, 239], relevant=1

### 陈述 7
- Career pressures - There may be pressure to publish results in excavation monographs and journal articles for career progression, which can take time and may compete with data sharing.
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[237-237]`
- au=1.5606800616296823e-06, eu=0.2753622531890869, au′=-0.0, eu′=0.03660885989665985
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[26.3157901763916]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 237 | 237 | 26.3157901763916 | Issues within archaeobotany may include the conflict between having to publish results within excavation monographs, which may take some time to be published, and have limited visibility due to high purchase costs and no digital access, and the need to publish journal articles for career progression within academia. |
- GPT 引用1: span=[237, 237], relevant=1

### 陈述 8
- Equity issues - Differences in access to resources and training may lead to disparities in data sharing and reuse across the discipline.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[239-239]`
- au=1.7609027054277249e-06, eu=0.2676055431365967, au′=-0.0, eu′=0.0352504625916481
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[27.36842155456543]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 239 | 239 | 27.36842155456543 | More broadly, one important aspect is issues of equity in access to digital archiving resources (Wright & Richards 2018), such as differential access to funds, training and knowledge. |
- GPT 引用1: span=[239, 239], relevant=1

### 陈述 9
In summary, a complex mix of technical, cultural, and structural factors contribute to the low levels of data sharing in archaeobotany. Addressing these issues will require improvements in training, infrastructure, and cultural norms around data sharing.
- GPT support_score=1; [[No]]   Need Citation: [[No]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

## multifieldqa_en_idx124 (idx=124, multifieldqa_en)
### 问题
What are some fields in which the inverse problem is encountered?

### 参考答案（数据集）
- ref1: Physics, biology, social sciences, finance, and neuroscience are some fields in which the inverse problem is encountered.

### 模型全文输出
```
<statement>Based on the introduction, some fields in which the inverse problem is encountered include:<cite></cite></statement>

<statement>- Physics - inverse problems arise in areas like quantum mechanics, condensed matter physics, and statistical physics. Examples mentioned include reconstructing images from light propagated through random media, and modeling the behavior of optical systems like lasers.<cite>[17-17][18-18][38-39]</cite></statement>

<statement>- Biology - inverse problems are used in areas like systems biology, neuroscience, and genomics to infer underlying interactions and networks from observed data. Examples mentioned include inferring gene regulatory networks and understanding synchronization in biological systems like starling flocks.<cite>[3-3][36-36]</cite></statement>

<statement>- Social sciences and finance - inverse problems are used in areas like economics, sociology, and finance to infer underlying structures and relationships from data. Examples mentioned include modeling financial markets and social networks.<cite>[3-3]</cite></statement>

<statement>- Neuroscience - inverse problems are used in neuroscience to infer connectivity and interactions in brain networks from observed data. Examples mentioned include inferring connectivity from functional MRI data and understanding neural coding.<cite>[3-3]</cite></statement>

<statement>- Other fields - inverse problems are encountered in many other areas including geophysics, medical imaging, signal processing, and engineering. The introduction states that with the increase in data availability across fields, inverse problems are becoming more important.<cite>[3-3]</cite></statement>
```

文档句索引范围: sid ∈ [1, 298], 共 298 句。
引用指标: R=0.5833333333333334, P=0.5714285714285714, F1=0.577319587628866
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=5540, completion_tokens=104
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
Based on the introduction, some fields in which the inverse problem is encountered include:
- GPT support_score=1; [[No]]   Need Citation: [[No]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 1
- Physics - inverse problems arise in areas like quantum mechanics, condensed matter physics, and statistical physics. Examples mentioned include reconstructing images from light propagated through random media, and modeling the behavior of optical systems like lasers.
- GPT support_score=0.5; **Rating**: [[Partially supported]]

**跨度** `[17-17]`
- au=0.6802244782447815, eu=0.18952614068984985, au′=0.6802234649658203, eu′=0.04467959702014923
- top_logits 方差 (pvariance, 有限 logit): 1.250864640497639
- top_logits 数值: `[22.5, 20.263158798217773]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 17 | 17 | 22.5 | Our main motivation to study statistical inference is that these kind of models have recently turned out to be rather useful in describing the behavior of optical systems, including standard mode-locking lasers \cite{Gordon02,Gat04,Angelani07,Marruzzo15} and random lasers \cite{Angelani06a,Leuzzi09a,Antenucci15a,Antenucci15b,Marruzzo16}. |
| 2 | 1 | 1 | 20.263158798217773 | \section{Introduction} Given a data set and a model with some unknown parameters, the inverse problem aims to find the values of the model parameters that best fit the data. |

**跨度** `[18-18]`
- au=0.68033766746521, eu=0.18999996781349182, au′=0.6803368330001831, eu′=0.04481131583452225
- top_logits 方差 (pvariance, 有限 logit): 1.1080328183161328
- top_logits 数值: `[22.36842155456543, 20.263158798217773]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 18 | 18 | 22.36842155456543 | In particular, the inverse problem on the pairwise XY model analyzed here might be of help in recovering images from light propagated through random media. |
| 2 | 39 | 39 | 20.263158798217773 | As couplings are strongly frustrated, these models turn out to be especially useful to the study of optical properties in random media \cite{Antenucci15a,Antenucci15b}, as in the noticeable case of random lasers \cite{Wiersma08,Andreasen11,Antenucci15e}  and they might as well be applied to linear scattering problems, e.g., propagation of waves in opaque systems or disordered fibers. |

**跨度** `[38-39]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> Phasor and phase models with pairwise and  multi-body interaction terms can, indeed,
> describe the behavior of electromagnetic modes in both linear and nonlinear optical
> systems in the analysis of problems such as light propagation and lasing \cite{Gordon02,
> Antenucci15c, Antenucci15d}. As couplings are strongly frustrated, these models turn out
> to be especially useful to the study of optical properties in random media
> \cite{Antenucci15a,Antenucci15b}, as in the noticeable case of random lasers
> \cite{Wiersma08,Andreasen11,Antenucci15e}  and they might as well be applied to linear
> scattering problems, e.g., propagation of waves in opaque systems or disordered fibers.
- au=1.3673090934753418, eu=0.11192928999662399, au′=1.3673088550567627, eu′=0.0479949414730072
- top_logits 方差 (pvariance, 有限 logit): 0.2196585124881949
- top_logits 数值: `[20.526315689086914, 19.86842155456543, 19.73684310913086, 19.210527420043945]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 38 | 38 | 20.526315689086914 | Phasor and phase models with pairwise and  multi-body interaction terms can, indeed, describe the behavior of electromagnetic modes in both linear and nonlinear optical systems in the analysis of problems such as light propagation and lasing \cite{Gordon02, Antenucci15c, Antenucci15d}. |
| 2 | 294 | 294 | 19.86842155456543 | The techniques displayed and the results obtained in this work can be of use in any of the many systems whose theoretical representation is given by Eq. |
| 3 | 37 | 37 | 19.73684310913086 | Our interest on the $XY$ model resides, though, in optics. |
| 4 | 33 | 33 | 19.210527420043945 | In presence of disorder and frustration \cite{Villain77,Fradkin78} the model has been adopted to describe synchronization problems as the Kuramoto model \cite{Kuramoto75} and in the theoretical modeling of Josephson junction arrays \cite{Teitel83a,Teitel83b} and arrays of coupled lasers \cite{Nixon13}. |
- GPT 引用1: span=[17, 18], relevant=1
- GPT 引用2: span=[38, 39], relevant=1

### 陈述 2
- Biology - inverse problems are used in areas like systems biology, neuroscience, and genomics to infer underlying interactions and networks from observed data. Examples mentioned include inferring gene regulatory networks and understanding synchronization in biological systems like starling flocks.
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[3-3]`
- au=1.0829722881317139, eu=0.13523128628730774, au′=1.0829721689224243, eu′=0.04481131583452225
- top_logits 方差 (pvariance, 有限 logit): 0.15004619446942039
- top_logits 数值: `[21.710527420043945, 21.447368621826172, 20.789474487304688]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 2 | 2 | 21.710527420043945 | In this work, in which we focus on systems of interacting elements, the inverse problem concerns  the statistical inference of the underling interaction network and of its coupling coefficients from observed data on the dynamics  of the system. |
| 2 | 3 | 3 | 21.447368621826172 | Versions of this problem are encountered in physics, biology (e.g., \cite{Balakrishnan11,Ekeberg13,Christoph14}), social sciences and finance (e.g.,\cite{Mastromatteo12,yamanaka_15}), neuroscience (e.g., \cite{Schneidman06,Roudi09a,tyrcha_13}), just to cite a few, and are becoming more and more important due to the increase in the amount of data available from these fields.\\ \indent A standard approach used in statistical inference is to predict the interaction couplings by maximizing the likelihood function. |
| 3 | 1 | 1 | 20.789474487304688 | \section{Introduction} Given a data set and a model with some unknown parameters, the inverse problem aims to find the values of the model parameters that best fit the data. |

**跨度** `[36-36]`
- au=1.95558504856308e-06, eu=0.3064514994621277, au′=-0.0, eu′=0.0423162542283535
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[22.631580352783203]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 36 | 36 | 22.631580352783203 | A rather fascinating example being the behavior of starlings flocks  \cite{Reynolds87,Deneubourg89,Huth90,Vicsek95, Cavagna13}. |
- GPT 引用1: span=[3, 3], relevant=1
- GPT 引用2: span=[36, 36], relevant=0

### 陈述 3
- Social sciences and finance - inverse problems are used in areas like economics, sociology, and finance to infer underlying structures and relationships from data. Examples mentioned include modeling financial markets and social networks.
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[3-3]`
- au=1.6452690942969639e-06, eu=0.29230761528015137, au′=-0.0, eu′=0.03966596722602844
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.210527420043945]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 3 | 3 | 24.210527420043945 | Versions of this problem are encountered in physics, biology (e.g., \cite{Balakrishnan11,Ekeberg13,Christoph14}), social sciences and finance (e.g.,\cite{Mastromatteo12,yamanaka_15}), neuroscience (e.g., \cite{Schneidman06,Roudi09a,tyrcha_13}), just to cite a few, and are becoming more and more important due to the increase in the amount of data available from these fields.\\ \indent A standard approach used in statistical inference is to predict the interaction couplings by maximizing the likelihood function. |
- GPT 引用1: span=[3, 3], relevant=1

### 陈述 4
- Neuroscience - inverse problems are used in neuroscience to infer connectivity and interactions in brain networks from observed data. Examples mentioned include inferring connectivity from functional MRI data and understanding neural coding.
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[3-3]`
- au=1.930719463416608e-06, eu=0.30158722400665283, au′=-0.0, eu′=0.04139433056116104
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.157896041870117]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 3 | 3 | 23.157896041870117 | Versions of this problem are encountered in physics, biology (e.g., \cite{Balakrishnan11,Ekeberg13,Christoph14}), social sciences and finance (e.g.,\cite{Mastromatteo12,yamanaka_15}), neuroscience (e.g., \cite{Schneidman06,Roudi09a,tyrcha_13}), just to cite a few, and are becoming more and more important due to the increase in the amount of data available from these fields.\\ \indent A standard approach used in statistical inference is to predict the interaction couplings by maximizing the likelihood function. |
- GPT 引用1: span=[3, 3], relevant=0

### 陈述 5
- Other fields - inverse problems are encountered in many other areas including geophysics, medical imaging, signal processing, and engineering. The introduction states that with the increase in data availability across fields, inverse problems are becoming more important.
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[3-3]`
- au=1.6068129298218992e-06, eu=0.28464409708976746, au′=-0.0, eu′=0.038267869502305984
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.131580352783203]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 3 | 3 | 25.131580352783203 | Versions of this problem are encountered in physics, biology (e.g., \cite{Balakrishnan11,Ekeberg13,Christoph14}), social sciences and finance (e.g.,\cite{Mastromatteo12,yamanaka_15}), neuroscience (e.g., \cite{Schneidman06,Roudi09a,tyrcha_13}), just to cite a few, and are becoming more and more important due to the increase in the amount of data available from these fields.\\ \indent A standard approach used in statistical inference is to predict the interaction couplings by maximizing the likelihood function. |
- GPT 引用1: span=[3, 3], relevant=0

## multifieldqa_en_idx125 (idx=125, multifieldqa_en)
### 问题
What is the maximum radius of the emitting region for the black body model?

### 参考答案（数据集）
- ref1: 4.5x10^8 cm.

### 模型全文输出
```
<statement>Based on the analysis of the XMM-Newton slew and follow-up observations, the maximum radius of the emitting region for the black body model is estimated to be:<cite></cite></statement>

<statement>- For the XMM-Newton slew discovery data, the maximum radius is 20^+31_{-10} kpc.<cite>[75-75]</cite></statement>

<statement>- For the Swift-XRT observation data, the maximum radius is not well constrained but is estimated to be around 4.5 x 10^8 cm at a distance of 50 kpc.<cite>[112-112]</cite></statement>

<statement>- For the XMM-Newton ToO observation data, the maximum radius is estimated to be around 1.3 x 10^8 cm at a distance of 50 kpc.<cite>[140-140]</cite></statement>

<statement>The maximum radius estimates for the black body model are all consistent with being at the high end of the mass range for white dwarfs (>1.2 solar masses) and possibly even approaching the white dwarf radius. However, caution is needed as the black body model may be overestimating the luminosity and therefore underestimating the distance and radius. The white dwarf atmosphere model yields smaller radius estimates, but still within the range of possible white dwarf sizes.<cite>[113-113][144-144][145-145]</cite></statement>
```

文档句索引范围: sid ∈ [1, 287], 共 287 句。
引用指标: R=0.5, P=1.0, F1=0.6666666666666666
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=3879, completion_tokens=69
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
Based on the analysis of the XMM-Newton slew and follow-up observations, the maximum radius of the emitting region for the black body model is estimated to be:
- GPT support_score=0; [[Yes]]   Need Citation: [[Yes]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 1
- For the XMM-Newton slew discovery data, the maximum radius is 20^+31_{-10} kpc.
- GPT support_score=0; Rating: [[No support]]

**跨度** `[75-75]`
- au=1.930719463416608e-06, eu=0.30158722400665283, au′=-0.0, eu′=0.04139433056116104
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.157896041870117]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 75 | 75 | 23.157896041870117 | If we assume a typical emitting region for the white dwarf atmosphere to be of spherical radius 10$^{9}$\,cm, then, for the black body model, this distance turns out to be 20$^{+31}_{-10}$\,kpc. |
- GPT 引用1: span=[75, 75], relevant=1

### 陈述 2
- For the Swift-XRT observation data, the maximum radius is not well constrained but is estimated to be around 4.5 x 10^8 cm at a distance of 50 kpc.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[112-112]`
- au=1.7108611700678011e-06, eu=0.3052207827568054, au′=-0.0, eu′=0.04208194464445114
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[22.763158798217773]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 112 | 112 | 22.763158798217773 | \label{xrtspec} \end{figure}  A cautious estimate of the size of the emitting region can be obtained from the model normalization; the assumed distance of 50\,kpc yields a maximum radius of 4.5$\times$10$^{8}$\,cm (the fit normalization is essentially unconstrained at the lower bound). |
- GPT 引用1: span=[112, 112], relevant=1

### 陈述 3
- For the XMM-Newton ToO observation data, the maximum radius is estimated to be around 1.3 x 10^8 cm at a distance of 50 kpc.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[140-140]`
- au=1.9009210063813953e-06, eu=0.2957197427749634, au′=-0.0, eu′=0.04029691964387894
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.8157901763916]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 140 | 140 | 23.8157901763916 | For the assumed distance of 50\,kpc, then the black body model returns an emitting region radius of only 1.3$\pm$0.2$\times$10$^{8}$\,cm. |
- GPT 引用1: span=[140, 140], relevant=1

### 陈述 4
The maximum radius estimates for the black body model are all consistent with being at the high end of the mass range for white dwarfs (>1.2 solar masses) and possibly even approaching the white dwarf radius. However, caution is needed as the black body model may be overestimating the luminosity and therefore underestimating the distance and radius. The white dwarf atmosphere model yields smaller radius estimates, but still within the range of possible white dwarf sizes.
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[113-113]`
- au=1.0816854238510132, eu=0.1428571194410324, au′=1.0816850662231445, eu′=0.047619037330150604
- top_logits 方差 (pvariance, 有限 logit): 0.3578025152148459
- top_logits 数值: `[20.789474487304688, 19.86842155456543, 19.342105865478516]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 113 | 113 | 20.789474487304688 | Though great care should be taken in interpreting this result, as the black body model is possibly overestimating the luminosity, this obtained radius is still consistent with that of moderately massive ($>$1.1$M_{\odot}$) white dwarfs (Hamada \& Salpeter 1961), i.e.\,the whole white dwarf surface may still be emitting at 59\,eV. |
| 2 | 140 | 140 | 19.86842155456543 | For the assumed distance of 50\,kpc, then the black body model returns an emitting region radius of only 1.3$\pm$0.2$\times$10$^{8}$\,cm. |
| 3 | 144 | 144 | 19.342105865478516 | The range in allowed radius therefore is quite large, and it is not impossible for for the whole of the white dwarf surface to be emitting at 70\,eV. |

**跨度** `[144-144]`
- au=1.0826431512832642, eu=0.1359570473432541, au′=1.0826423168182373, eu′=0.04507710412144661
- top_logits 方差 (pvariance, 有限 logit): 0.3578025152148459
- top_logits 数值: `[21.842105865478516, 21.3157901763916, 20.394737243652344]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 140 | 140 | 21.842105865478516 | For the assumed distance of 50\,kpc, then the black body model returns an emitting region radius of only 1.3$\pm$0.2$\times$10$^{8}$\,cm. |
| 2 | 144 | 144 | 21.3157901763916 | The range in allowed radius therefore is quite large, and it is not impossible for for the whole of the white dwarf surface to be emitting at 70\,eV. |
| 3 | 139 | 139 | 20.394737243652344 | As before, the model normalization can be used to obtain a cautious estimate of the size of the emitting region. |

**跨度** `[145-145]`
- au=2.2675894797430374e-06, eu=0.32067495584487915, au′=-0.0, eu′=0.04507710039615631
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[21.18421173095703]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 145 | 145 | 21.18421173095703 | If this is the case, then the white dwarf would have to be at the high end of the mass range ($>$1.2$M_{\odot}$; Hamada \& Salpeter 1961). |
- GPT 引用1: span=[113, 113], relevant=1
- GPT 引用2: span=[144, 145], relevant=1

## multifieldqa_en_idx126 (idx=126, multifieldqa_en)
### 问题
Which air unit did Goodwin command during the initial landings of Marines on Saipan?

### 参考答案（数据集）
- ref1: VC-10 Squadron.

### 模型全文输出
```
<statement>Based on the information provided, Goodwin commanded VC-10 Squadron during the initial landings of Marines on Saipan.<cite></cite></statement>

<statement>Specifically, the passage states: "Goodwin departed on May 1, 1944, to join Rear admiral Harold B. Sallada's Carrier Support Group 2, staging in the Marshalls for the invasion of the Marianas. The air unit, VC-10 Squadron, under Goodwin's command gave close air support to the initial landings of Marines on Saipan on June 15, 1944, destroying enemy gun emplacements, troops, tanks, and trucks."<cite>[35-36]</cite></statement>

<statement>So VC-10 Squadron under Goodwin provided close air support for the initial landings on Saipan.<cite></cite></statement>
```

文档句索引范围: sid ∈ [1, 79], 共 79 句。
引用指标: R=0.3333333333333333, P=1.0, F1=0.5
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=1683, completion_tokens=33
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
Based on the information provided, Goodwin commanded VC-10 Squadron during the initial landings of Marines on Saipan.
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 1
Specifically, the passage states: "Goodwin departed on May 1, 1944, to join Rear admiral Harold B. Sallada's Carrier Support Group 2, staging in the Marshalls for the invasion of the Marianas. The air unit, VC-10 Squadron, under Goodwin's command gave close air support to the initial landings of Marines on Saipan on June 15, 1944, destroying enemy gun emplacements, troops, tanks, and trucks."
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[35-36]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> During the first half of 1944, Gambier Bay was tasked with ferrying aircraft for repairs
> and qualified carrier pilots from San Diego to Pearl Harbor, Hawaii, before departed on
> May 1, 1944, to join Rear admiral Harold B. Sallada's Carrier Support Group 2, staging
> in the Marshalls for the invasion of the Marianas. The air unit, VC-10 Squadron, under
> Goodwin's command gave close air support to the initial landings of Marines on Saipan on
> June 15, 1944, destroying enemy gun emplacements, troops, tanks, and trucks.
- au=1.6396279534092173e-06, eu=0.2911876440048218, au′=-0.0, eu′=0.039460018277168274
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.342105865478516]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 35 | 35 | 24.342105865478516 | During the first half of 1944, Gambier Bay was tasked with ferrying aircraft for repairs and qualified carrier pilots from San Diego to Pearl Harbor, Hawaii, before departed on May 1, 1944, to join Rear admiral Harold B. Sallada's Carrier Support Group 2, staging in the Marshalls for the invasion of the Marianas. |
- GPT 引用1: span=[35, 36], relevant=1

### 陈述 2
So VC-10 Squadron under Goodwin provided close air support for the initial landings on Saipan.
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

## multifieldqa_en_idx127 (idx=127, multifieldqa_en)
### 问题
How does the receptive field size affect the completion of shapes?

### 参考答案（数据集）
- ref1: Bigger receptive field size leads to more successful shape completion.

### 模型全文输出
```
<statement>The receptive field size in the model affects the completion of shapes as follows:<cite></cite></statement>

<statement>- Bigger receptive field size allows the model to capture and process larger context information from the input image. This helps the model to better connect fragmented contours and fill in larger gaps between them.<cite>[188-191]</cite></statement>

<statement>- The table in the paper shows that as the receptive field size increases, the model becomes more successful in completing shapes with larger gaps. For example, with a receptive field size of 5, the model succeeds in completing 50% of shapes with gaps of size 3, while with a receptive field size of 9, it succeeds in completing 75% of shapes with gaps of size 5.<cite>[8-8][190-190]</cite></statement>

<statement>- So in summary, a larger receptive field size allows the model to process more context information and fill in larger gaps between fragmented contours, leading to better contour completion overall. The experiments in the paper demonstrate this improvement in performance as the receptive field size is increased.<cite></cite></statement>
```

文档句索引范围: sid ∈ [1, 207], 共 207 句。
引用指标: R=0.375, P=1.0, F1=0.5454545454545454
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=2762, completion_tokens=55
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
The receptive field size in the model affects the completion of shapes as follows:
- GPT support_score=1; [[No]]   Need Citation: [[No]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 1
- Bigger receptive field size allows the model to capture and process larger context information from the input image. This helps the model to better connect fragmented contours and fill in larger gaps between them.
- GPT support_score=0.5; **Rating**: [[Partially supported]]

**跨度** `[188-191]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> Effect of Receptive Field Size
>
> To better understand the effect of receptive field size on our algorithm, we test the
> following hypothesis: can models with bigger receptive field size complete shapes with
> bigger gaps? In Table , we report showing the results of this experiment. As we can see,
> the bigger the receptive field size, the more complete shapes we can reconstruct using
> DSP. Implementation Details
>
> As shown in Figure , we use a model with 5 layers and 128 channels for downsampling and
> upsampling convolutions, and 64 channels for skip convolutions.
- au=0.6826401948928833, eu=0.17471261322498322, au′=0.6826393604278564, eu′=0.04061998426914215
- top_logits 方差 (pvariance, 有限 logit): 0.03895437288156245
- top_logits 数值: `[23.8157901763916, 23.421052932739258]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 188 | 188 | 23.8157901763916 | Effect of Receptive Field Size  To better understand the effect of receptive field size on our algorithm, we test the following hypothesis: can models with bigger receptive field size complete shapes with bigger gaps? |
| 2 | 8 | 8 | 23.421052932739258 | In this table, we show the effect of the receptive filter size on our algorithm's capability to fill in bigger gap sizes.The numbers in this table are showing the percentage of the time that DIP was successful to complete shapes with each gap size and corresponding receptive field size.As predicted, the bigger the filter size, the more successful the algorithm is in filling in the gaps. |
- GPT 引用1: span=[188, 191], relevant=1

### 陈述 2
- The table in the paper shows that as the receptive field size increases, the model becomes more successful in completing shapes with larger gaps. For example, with a receptive field size of 5, the model succeeds in completing 50% of shapes with gaps of size 3, while with a receptive field size of 9, it succeeds in completing 75% of shapes with gaps of size 5.
- GPT support_score=0; Rating: [[No support]]

**跨度** `[8-8]`
- au=1.8090482853949652e-06, eu=0.27737218141555786, au′=-0.0, eu′=0.03696497529745102
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[26.05263328552246]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 8 | 8 | 26.05263328552246 | In this table, we show the effect of the receptive filter size on our algorithm's capability to fill in bigger gap sizes.The numbers in this table are showing the percentage of the time that DIP was successful to complete shapes with each gap size and corresponding receptive field size.As predicted, the bigger the filter size, the more successful the algorithm is in filling in the gaps. |

**跨度** `[190-190]`
- au=1.8140860902349232e-06, eu=0.27838820219039917, au′=-0.0, eu′=0.037145648151636124
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.921052932739258]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 190 | 190 | 25.921052932739258 | As we can see, the bigger the receptive field size, the more complete shapes we can reconstruct using DSP. |
- GPT 引用1: span=[8, 8], relevant=1
- GPT 引用2: span=[190, 190], relevant=1

### 陈述 3
- So in summary, a larger receptive field size allows the model to process more context information and fill in larger gaps between fragmented contours, leading to better contour completion overall. The experiments in the paper demonstrate this improvement in performance as the receptive field size is increased.
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

## multifieldqa_en_idx128 (idx=128, multifieldqa_en)
### 问题
What is the significance of the interlayer Berry connection polarizability?

### 参考答案（数据集）
- ref1: The momentum space curl of the interlayer Berry connection polarizability generates the crossed nonlinear dynamical Hall effect.

### 模型全文输出
```
<statement>The interlayer Berry connection polarizability (BCP) plays a crucial role in the crossed nonlinear dynamical Hall effect uncovered in twisted bilayers. Some key points about its significance are:<cite></cite></statement>

<statement>- It is a band geometric quantity that measures the k-space curl of the interlayer BCP over occupied states. This curl is a characteristic of layer-hybridized electronic states.<cite>[61-61]</cite></statement>

<statement>- It is invariant under rotation but flips sign under space inversion, mirror reflection, and rotoreflection symmetries. This means it is allowed only in systems with chiral crystal structure, like twisted bilayers.<cite>[63-64]</cite></statement>

<statement>- It controls the strength and direction of the Hall current in the crossed nonlinear dynamical Hall effect. Regions with large [∂k × G]z contribute most to the Hall conductivity.<cite>[61-61][86-86][88-88]</cite></statement>

<statement>- It is related to the interlayer tunneling and hybridization of electronic states in twisted bilayers. It is suppressed for states fully polarized in one layer, indicating its link to interlayer coupling.<cite>[57-57][58-58][61-61]</cite></statement>

<statement>- It offers a new way to probe chiral symmetry and interlayer coupling in bilayer systems through nonlinear Hall response. Its tunability by phase difference of driving fields provides a unique rectification functionality.<cite>[17-17][33-33][34-34]</cite></statement>

<statement>In summary, the interlayer BCP is a key band geometric quantity that underlies the crossed nonlinear dynamical Hall effect in twisted bilayers. It provides a new tool to probe chiral symmetry and interlayer coupling in these systems. Its tunability and connection to layer hybridization make it a significant concept in the study of twisted bilayer materials.<cite></cite></statement>
```

文档句索引范围: sid ∈ [1, 148], 共 148 句。
引用指标: R=0.7857142857142857, P=0.7777777777777778, F1=0.7817258883248731
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=5858, completion_tokens=120
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
The interlayer Berry connection polarizability (BCP) plays a crucial role in the crossed nonlinear dynamical Hall effect uncovered in twisted bilayers. Some key points about its significance are:
- GPT support_score=1; [[No]]   Need Citation: [[No]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 1
- It is a band geometric quantity that measures the k-space curl of the interlayer BCP over occupied states. This curl is a characteristic of layer-hybridized electronic states.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[61-61]`
- au=0.6826944947242737, eu=0.1731206774711609, au′=0.6826937198638916, eu′=0.04019036889076233
- top_logits 方差 (pvariance, 有限 logit): 0.10820617305944324
- top_logits 数值: `[24.210527420043945, 23.55263328552246]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 61 | 61 | 24.210527420043945 | This band geometric quantity measures the k-space curl of the interlayer BCP over the occupied states, and hence is also a characteristic of layer-hybridized electronic states. |
| 2 | 60 | 60 | 23.55263328552246 | The band geometric contributions to ṙ lead to a Hall current where is intrinsic to the band structure. |
- GPT 引用1: span=[61, 61], relevant=1

### 陈述 2
- It is invariant under rotation but flips sign under space inversion, mirror reflection, and rotoreflection symmetries. This means it is allowed only in systems with chiral crystal structure, like twisted bilayers.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[63-64]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> Since χ int is a time-reversal even pseudoscalar, it is invariant under rotation, but
> flips sign under space inversion, mirror reflection and rotoreflection symmetries. As
> such, χ int is allowed if and only if the system possesses a chiral crystal structure,
> which is the very case of twisted bilayers .
- au=1.6015096662158612e-06, eu=0.28358200192451477, au′=-0.0, eu′=0.03807614743709564
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.263158798217773]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 63 | 63 | 25.263158798217773 | Since χ int is a time-reversal even pseudoscalar, it is invariant under rotation, but flips sign under space inversion, mirror reflection and rotoreflection symmetries. |
- GPT 引用1: span=[63, 64], relevant=1

### 陈述 3
- It controls the strength and direction of the Hall current in the crossed nonlinear dynamical Hall effect. Regions with large [∂k × G]z contribute most to the Hall conductivity.
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[61-61]`
- au=0.6815940141677856, eu=0.183574840426445, au′=0.6815932989120483, eu′=0.043035101145505905
- top_logits 方差 (pvariance, 有限 logit): 0.432824692237773
- top_logits 数值: `[22.894737243652344, 21.578948974609375]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 60 | 60 | 22.894737243652344 | The band geometric contributions to ṙ lead to a Hall current where is intrinsic to the band structure. |
| 2 | 61 | 61 | 21.578948974609375 | This band geometric quantity measures the k-space curl of the interlayer BCP over the occupied states, and hence is also a characteristic of layer-hybridized electronic states. |

**跨度** `[86-86]`
- au=1.082545518875122, eu=0.13868612051010132, au′=1.0825450420379639, eu′=0.04607921838760376
- top_logits 方差 (pvariance, 有限 logit): 0.11926778392403826
- top_logits 数值: `[21.18421173095703, 20.526315689086914, 20.394737243652344]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 86 | 86 | 21.18421173095703 | It favors band near-degeneracy regions in k -space made up of strongly layer hybridized electronic states. |
| 2 | 109 | 109 | 20.526315689086914 | The peaks are centered around band degeneracies, and their profiles can be understood from the distribution of [∂ k × G] z . |
| 3 | 85 | 85 | 20.394737243652344 | ( ). |

**跨度** `[88-88]`
- au=0.6818867921829224, eu=0.1818181425333023, au′=0.6818859577178955, eu′=0.04255318641662598
- top_logits 方差 (pvariance, 有限 logit): 0.2770082045790332
- top_logits 数值: `[23.026315689086914, 21.973684310913086]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 88 | 88 | 23.026315689086914 | that [∂ k × G] z is negligible at lower energies, and it is dominated by positive values as the doping increases, thus the conductivity rises initially. |
| 2 | 109 | 109 | 21.973684310913086 | The peaks are centered around band degeneracies, and their profiles can be understood from the distribution of [∂ k × G] z . |
- GPT 引用1: span=[61, 61], relevant=0
- GPT 引用2: span=[86, 86], relevant=0
- GPT 引用3: span=[88, 88], relevant=1

### 陈述 4
- It is related to the interlayer tunneling and hybridization of electronic states in twisted bilayers. It is suppressed for states fully polarized in one layer, indicating its link to interlayer coupling.
- GPT support_score=0.5; **Rating**: [[Partially supported]]

**跨度** `[57-57]`
- au=1.9067709899900365e-06, eu=0.29687488079071045, au′=-0.0, eu′=0.04051172360777855
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.68421173095703]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 57 | 57 | 23.68421173095703 | This picture is further augmented by the connotation that the interlayer BCP is featured exclusively by interlayer-hybridized electronic states: According to Eq. |

**跨度** `[58-58]`
- au=1.650961507948523e-06, eu=0.2934361696243286, au′=-0.0, eu′=0.03987407684326172
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.078948974609375]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 58 | 58 | 24.078948974609375 | ( ), if the state \|u n is fully polarized in a specific layer around some momentum k, then G (k) is suppressed. |

**跨度** `[61-61]`
- au=1.9009210063813953e-06, eu=0.2957197427749634, au′=-0.0, eu′=0.04029691964387894
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.8157901763916]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 61 | 61 | 23.8157901763916 | This band geometric quantity measures the k-space curl of the interlayer BCP over the occupied states, and hence is also a characteristic of layer-hybridized electronic states. |
- GPT 引用1: span=[57, 58], relevant=1
- GPT 引用2: span=[61, 61], relevant=1

### 陈述 5
- It offers a new way to probe chiral symmetry and interlayer coupling in bilayer systems through nonlinear Hall response. Its tunability by phase difference of driving fields provides a unique rectification functionality.
- GPT support_score=0.5; **Rating**: [[Partially supported]]

**跨度** `[17-17]`
- au=0.6818683743476868, eu=0.18009476363658905, au′=0.6818675994873047, eu′=0.04208194464445114
- top_logits 方差 (pvariance, 有限 logit): 0.432824692237773
- top_logits 数值: `[23.421052932739258, 22.10526466369629]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 17 | 17 | 23.421052932739258 | This novel intrinsic Hall response has a band geometric origin in the momentum space curl of interlayer Berry connection polarizability, arising from layer hybridization of electrons by the twisted interlayer coupling. |
| 2 | 33 | 33 | 22.10526466369629 | The effect arises from the interlayer hybridization of electronic states under the chiral crystal symmetry characteristic of twisted bilayers, and has a novel band geometric origin in the momentum space curl of interlayer Berry connection polarizability (BCP). |

**跨度** `[33-33]`
- au=0.6826401948928833, eu=0.17471261322498322, au′=0.6826393604278564, eu′=0.04061998426914215
- top_logits 方差 (pvariance, 有限 logit): 0.03895437288156245
- top_logits 数值: `[23.8157901763916, 23.421052932739258]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 33 | 33 | 23.8157901763916 | The effect arises from the interlayer hybridization of electronic states under the chiral crystal symmetry characteristic of twisted bilayers, and has a novel band geometric origin in the momentum space curl of interlayer Berry connection polarizability (BCP). |
| 2 | 18 | 18 | 23.421052932739258 | The effect allows a unique rectification functionality and a transport probe of chiral symmetry in bilayer systems. |

**跨度** `[34-34]`
- au=0.6826032996177673, eu=0.1731206774711609, au′=0.6826024055480957, eu′=0.04019037261605263
- top_logits 方差 (pvariance, 有限 logit): 0.21208462622689694
- top_logits 数值: `[24.342105865478516, 23.421052932739258]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 34 | 34 | 24.342105865478516 | Having two driving fields of the same frequency, a dc Hall current develops, whose on/off, direction and magnitude can all be controlled by the phase difference of the two fields, which does not affect the magnitude of the double-frequency component. |
| 2 | 35 | 35 | 23.421052932739258 | Such a characteristic tunability renders this effect a unique approach to rectification and transport probe of chiral bilayers. |
- GPT 引用1: span=[17, 17], relevant=1
- GPT 引用2: span=[33, 34], relevant=1

### 陈述 6
In summary, the interlayer BCP is a key band geometric quantity that underlies the crossed nonlinear dynamical Hall effect in twisted bilayers. It provides a new tool to probe chiral symmetry and interlayer coupling in these systems. Its tunability and connection to layer hybridization make it a significant concept in the study of twisted bilayer materials.
- GPT support_score=1; [[No]]   Need Citation: [[No]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

## multifieldqa_en_idx129 (idx=129, multifieldqa_en)
### 问题
Can the denoiser be applied to circuits with non-Clifford noise?

### 参考答案（数据集）
- ref1: Yes, the denoiser works for non-Clifford local noise channels.

### 模型全文输出
```
<statement>Yes, the denoiser can be applied to circuits with non-Clifford noise.<cite></cite></statement>

<statement>- The denoiser is designed to invert a global noise channel, which can be non-Clifford.<cite>[45-45][48-48]</cite></statement>

<statement>- The paper explicitly states that the approach works for non-Clifford local noise channels.<cite>[45-45]</cite></statement>

<statement>- The denoiser is a sum of CPTP maps, which can be unitary or non-unitary. Non-unitary maps are essential for cancelling the non-unitary accumulated noise.<cite>[72-76]</cite></statement>

<statement>- The paper considers uniform depolarizing noise as an example, but the approach can be used for any non-Clifford noise. The denoiser can be optimized to mitigate errors for specific types of noise.<cite>[112-112]</cite></statement>

<statement>- The paper shows the effectiveness of the denoiser for non-Clifford noise by considering eigenvalues of the noisy and denoised circuits. The denoiser pushes the eigenvalues back to the unit circle, showing it cancels the noise.<cite>[4-4][7-7][128-132]</cite></statement>

<statement>So in summary, the denoiser can be applied to circuits with non-Clifford noise, and it works by inverting the global noise channel. The denoiser can be optimized for specific types of noise and has been shown to effectively mitigate errors.<cite></cite></statement>
```

文档句索引范围: sid ∈ [1, 215], 共 215 句。
引用指标: R=0.5, P=1.0, F1=0.6666666666666666
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=6313, completion_tokens=106
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
Yes, the denoiser can be applied to circuits with non-Clifford noise.
- GPT support_score=0; [[Yes]]   Need Citation: [[Yes]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 1
- The denoiser is designed to invert a global noise channel, which can be non-Clifford.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[45-45]`
- au=1.9246485862822738e-06, eu=0.3003951609134674, au′=-0.0, eu′=0.04117009416222572
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.289474487304688]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 45 | 45 | 23.289474487304688 | Here we introduce a mitigation technique that takes into account the propagation of errors, can be performed with a tunable number of extra gates, and works for non-Clifford local noise channels since the inversion of the accumulated global noise channel is implicit. |

**跨度** `[48-48]`
- au=0.6818891167640686, eu=0.1840193271636963, au′=0.681888222694397, eu′=0.043157294392585754
- top_logits 方差 (pvariance, 有限 logit): 0.10820617305944324
- top_logits 数值: `[22.5, 21.842105865478516]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 48 | 48 | 22.5 | Essentially, the denoiser inverts a global noise channel. |
| 2 | 47 | 47 | 21.842105865478516 | We call the corresponding ensemble of random circuits a denoiser, and we construct it such that upon averaging the accumulated errors cancel. |
- GPT 引用1: span=[45, 45], relevant=1
- GPT 引用2: span=[48, 48], relevant=1

### 陈述 2
- The paper explicitly states that the approach works for non-Clifford local noise channels.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[45-45]`
- au=1.5962523320922628e-06, eu=0.28252777457237244, au′=-0.0, eu′=0.03788633644580841
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.394737243652344]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 45 | 45 | 25.394737243652344 | Here we introduce a mitigation technique that takes into account the propagation of errors, can be performed with a tunable number of extra gates, and works for non-Clifford local noise channels since the inversion of the accumulated global noise channel is implicit. |
- GPT 引用1: span=[45, 45], relevant=1

### 陈述 3
- The denoiser is a sum of CPTP maps, which can be unitary or non-unitary. Non-unitary maps are essential for cancelling the non-unitary accumulated noise.
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[72-76]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> Concretely, we use a trace preserv-ing sum of a unitary and a non-unitary channel. For
> the unitary part we take a two-qubit unitary channel U( φ i ) = U ( φ i ) ⊗ U * ( φ i ),
> with U ( φ i ) a two-qubit unitary gate parameterized by φ i . For this we take the two-
> qubit ZZ rotation exp(−iα(σ z ⊗ σ z )) with angle α, which can be obtained from native
> gates on current hardware , and dress it with four general one-qubit unitaries, only two
> of which are independent if we want a circuit that is space inversion symmetric around
> every bond. The resulting gate has 7 real parameters φ i . For the non-unitary part,
> which is essential because D has to cancel the non-unitary accumulated noise to obtain
> the noiseless unitary circuit, we use a general onequbit measurement followed by
> conditional preparation channel M( , with V a general one-qubit unitary and each κ i a
> 3-dimensional vector, resulting in a real 9-dimensional ζ i .
- au=0.6821558475494385, eu=0.18095232546329498, au′=0.6821550726890564, eu′=0.0423162542283535
- top_logits 方差 (pvariance, 有限 logit): 0.0692520511447583
- top_logits 数值: `[22.894737243652344, 22.36842155456543]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 71 | 71 | 22.894737243652344 | To facilitate the stochastic interpretation we parameterize each two-qubit denoiser channel G i as a sum of CPTP maps, such that we can sample the terms in this sum and execute the sampled gate on the quantum processor. |
| 2 | 72 | 72 | 22.36842155456543 | Concretely, we use a trace preserv-ing sum of a unitary and a non-unitary channel. |
- GPT 引用1: span=[72, 76], relevant=1

### 陈述 4
- The paper considers uniform depolarizing noise as an example, but the approach can be used for any non-Clifford noise. The denoiser can be optimized to mitigate errors for specific types of noise.
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[112-112]`
- au=1.698429286989267e-06, eu=0.30278876423835754, au′=-0.0, eu′=0.041621025651693344
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.026315689086914]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 112 | 112 | 23.026315689086914 | We consider circuits that are affected by uniform depolarizing noise with probability p for simplicity, but our approach can be used for any non-Clifford noise. |
- GPT 引用1: span=[112, 112], relevant=1

### 陈述 5
- The paper shows the effectiveness of the denoiser for non-Clifford noise by considering eigenvalues of the noisy and denoised circuits. The denoiser pushes the eigenvalues back to the unit circle, showing it cancels the noise.
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[4-4]`
- au=1.0828534364700317, eu=0.13571427762508392, au′=1.0828527212142944, eu′=0.044988155364990234
- top_logits 方差 (pvariance, 有限 logit): 0.20006159262589385
- top_logits 数值: `[21.842105865478516, 21.052631378173828, 20.789474487304688]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 128 | 128 | 21.842105865478516 | In Fig. |
| 2 | 4 | 4 | 21.052631378173828 | The complex eigenvalues λ of the noisy second-order Trotter supercircuit with Mtrot = 16 at time t = 1 (left), the corresponding optimized denoiser with M = 4 (center), and the denoised Trotter supercircuit (right).The Trotter circuit is for a L = 6 Heisenberg model with PBC, and all twoqubit channels are affected by depolarizing noise with p = 0.0046.The unit circle, on which unitary eigenvalues must lie, is shown in black, and the noiseless eigenvalues are shown as blue bars.It is evident that the denoiser recovers all the noiseless eigenvalues from the noisy circuit. |
| 3 | 130 | 130 | 20.789474487304688 | The eigenvalues λ of a unitary supercircuit lie on the unit circle, and in the presence of dissipation they are pushed to the center. |

**跨度** `[7-7]`
- au=1.6396279534092173e-06, eu=0.2911876440048218, au′=-0.0, eu′=0.039460018277168274
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.342105865478516]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 7 | 7 | 24.342105865478516 | The complex eigenvalues λ of the noisy second-order Trotter supercircuit with Mtrot = 16 at time t = 1 (left), the corresponding optimized denoiser with M = 4 (center), and the denoised Trotter supercircuit (right).The Trotter circuit is for a L = 6 Heisenberg model with PBC, and all twoqubit channels are affected by depolarizing noise with p = 0.036.The unit circle, on which unitary eigenvalues must lie, is shown in black, and the noiseless eigenvalues are shown as blue bars.It is clear that the denoiser recovers with high accuracy the noiseless eigenvalues from the noisy circuit. |

**跨度** `[128-132]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> In Fig. we show the eigenvalues of the noisy supercircuits for a noisy second-order
> Trotter supercircuit with M trot = 16 at t = 1 (left), the corresponding optimized
> denoiser with M = 4 (center), and the denoised supercircuit (right). The eigenvalues λ
> of a unitary supercircuit lie on the unit circle, and in the presence of dissipation
> they are pushed to the center. We see that the spectrum of the denoiser lies outside the
> unit circle, making it an unphysical channel which cures the effect of the noise on the
> circuit, such that the spectrum of the denoised circuit is pushed back to the unit
> circle. The noiseless eigenvalues are shown as blue bars, making it clear that the
> denoiser is able to recover the noiseless eigenvalues from the noisy circuit.
- au=1.0813100337982178, eu=0.13919411599636078, au′=1.08130943775177, eu′=0.04626622423529625
- top_logits 方差 (pvariance, 有限 logit): 1.1234235015055876
- top_logits 数值: `[22.10526466369629, 20.0, 19.73684310913086]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 128 | 128 | 22.10526466369629 | In Fig. |
| 2 | 129 | 129 | 20.0 | we show the eigenvalues of the noisy supercircuits for a noisy second-order Trotter supercircuit with M trot = 16 at t = 1 (left), the corresponding optimized denoiser with M = 4 (center), and the denoised supercircuit (right). |
| 3 | 130 | 130 | 19.73684310913086 | The eigenvalues λ of a unitary supercircuit lie on the unit circle, and in the presence of dissipation they are pushed to the center. |
- GPT 引用1: span=[4, 4], relevant=1
- GPT 引用2: span=[7, 7], relevant=1
- GPT 引用3: span=[128, 132], relevant=1

### 陈述 6
So in summary, the denoiser can be applied to circuits with non-Clifford noise, and it works by inverting the global noise channel. The denoiser can be optimized for specific types of noise and has been shown to effectively mitigate errors.
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

## multifieldqa_en_idx130 (idx=130, multifieldqa_en)
### 问题
What is Professor Tulis's forthcoming book?

### 参考答案（数据集）
- ref1: Legacies of Losing in American Politics and an expanded edition of The Rhetorical Presidency in the Princeton Classics series.

### 模型全文输出
```
<statement>Based on the information provided, Professor Tulis has two forthcoming books:<cite></cite></statement>

<statement>1. Legacies of Losing in American Politics, with Nicole Mellow (University of Chicago Press, Fall 2017)<cite>[11-11]</cite></statement>

<statement>2. An expanded edition of The Rhetorical Presidency in the Princeton Classics series (Princeton, Fall 2017)<cite>[11-11]</cite></statement>
```

文档句索引范围: sid ∈ [1, 225], 共 225 句。
引用指标: R=0.6666666666666666, P=1.0, F1=0.8
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=1562, completion_tokens=36
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
Based on the information provided, Professor Tulis has two forthcoming books:
- GPT support_score=0; [[Yes]]   Need Citation: [[Yes]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 1
1. Legacies of Losing in American Politics, with Nicole Mellow (University of Chicago Press, Fall 2017)
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[11-11]`
- au=1.7844809008238371e-06, eu=0.2724013328552246, au′=-0.0, eu′=0.036087363958358765
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[26.710527420043945]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 11 | 11 | 26.710527420043945 | His forthcoming books include: Legacies of Losing in American Politics, with Nicole Mellow (University of Chicago Press, Fall 2017), and an expanded edition of The Rhetorical Presidency in the Princeton Classics series (Princeton, Fall 2017). |
- GPT 引用1: span=[11, 11], relevant=1

### 陈述 2
2. An expanded edition of The Rhetorical Presidency in the Princeton Classics series (Princeton, Fall 2017)
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[11-11]`
- au=1.7294394183409167e-06, eu=0.26116830110549927, au′=-0.0, eu′=0.03414195403456688
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[28.289474487304688]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 11 | 11 | 28.289474487304688 | His forthcoming books include: Legacies of Losing in American Politics, with Nicole Mellow (University of Chicago Press, Fall 2017), and an expanded edition of The Rhetorical Presidency in the Princeton Classics series (Princeton, Fall 2017). |
- GPT 引用1: span=[11, 11], relevant=1

## multifieldqa_en_idx131 (idx=131, multifieldqa_en)
### 问题
How does a media application determine the context of an event?

### 参考答案（数据集）
- ref1: It uses a content-recognition module or algorithm.

### 模型全文输出
```
<statement>Based on the patent, a media application determines the context of an event in a media asset using a content-recognition module or algorithm.<cite>[80-80]</cite></statement>

<statement>The content-recognition module uses various techniques to analyze data associated with the event, such as:<cite></cite></statement>

<statement>- Object recognition techniques like edge detection, pattern recognition, and optical character recognition to identify objects and characteristics in the media asset.<cite>[81-81]</cite></statement>

<statement>- Speech recognition techniques like Hidden Markov Models, dynamic time warping, and neural networks to translate spoken words into text.<cite>[85-85]</cite></statement>

<statement>- Analysis of volume and tone of statements to determine the tone of the statement.<cite>[87-87]</cite></statement>

<statement>- Use of multiple types of optical character recognition and fuzzy logic to match keywords and cross-reference data with databases indicating different contexts.<cite>[88-89]</cite></statement>

<statement>- Analysis of relationships between words and other information obtained by the content-recognition module to understand the meaning of the event.<cite>[128-128]</cite></statement>

<statement>- Cross-referencing the information obtained with a database to determine the context of the event. For example, determining who said a statement, what the statement refers to, the relationships between words, and the meaning of keywords.<cite>[123-123][128-128][129-129]</cite></statement>

<statement>So in summary, the content-recognition module uses various techniques to analyze and understand the data associated with an event in a media asset to determine the context of the event. The context is the set of circumstances or facts that surround an event that influence or affect the meaning of the event.<cite></cite></statement>
```

文档句索引范围: sid ∈ [1, 235], 共 235 句。
引用指标: R=0.7777777777777778, P=1.0, F1=0.8750000000000001
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=5937, completion_tokens=119
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
Based on the patent, a media application determines the context of an event in a media asset using a content-recognition module or algorithm.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[80-80]`
- au=2.0506940927589312e-06, eu=0.3247861862182617, au′=-0.0, eu′=0.04589371383190155
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[20.789474487304688]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 80 | 80 | 20.789474487304688 | In some embodiments, a media application may use a content-recognition module or algorithm to determine the context of an event and distribute itemized tasks to multiple users in order to generate the supplemental information about the event. |
- GPT 引用1: span=[80, 80], relevant=1

### 陈述 1
The content-recognition module uses various techniques to analyze data associated with the event, such as:
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 2
- Object recognition techniques like edge detection, pattern recognition, and optical character recognition to identify objects and characteristics in the media asset.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[81-81]`
- au=1.8090482853949652e-06, eu=0.27737218141555786, au′=-0.0, eu′=0.03696497529745102
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[26.05263328552246]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 81 | 81 | 26.05263328552246 | The content-recognition module may use object recognition techniques such as edge detection, pattern recognition, including, but not limited to, self-learning systems (e.g., neural networks), optical character recognition, on-line character recognition (including but not limited to, dynamic character recognition, real-time character recognition, intelligent character recognition), and/or any other suitable technique or method to determine the objects and/or characteristics in media assets. |
- GPT 引用1: span=[81, 81], relevant=1

### 陈述 3
- Speech recognition techniques like Hidden Markov Models, dynamic time warping, and neural networks to translate spoken words into text.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[85-85]`
- au=1.5910399042695644e-06, eu=0.28148138523101807, au′=-0.0, eu′=0.0376984067261219
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.526317596435547]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 85 | 85 | 25.526317596435547 | In some embodiments, the content-recognition module or algorithm may also include speech recognition techniques, including but not limited to Hidden Markov Models, dynamic time warping, and/or neural networks (as described above) to translate spoken words into text. |
- GPT 引用1: span=[85, 85], relevant=1

### 陈述 4
- Analysis of volume and tone of statements to determine the tone of the statement.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[87-87]`
- au=1.774934617060353e-06, eu=0.2704625725746155, au′=-0.0, eu′=0.03574788197875023
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[26.973684310913086]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 87 | 87 | 26.973684310913086 | For example, the media application may monitor the volume of a statement in a media asset to determine the tone of the statement (e.g., a high volume may indicate an angry tone). |
- GPT 引用1: span=[87, 87], relevant=1

### 陈述 5
- Use of multiple types of optical character recognition and fuzzy logic to match keywords and cross-reference data with databases indicating different contexts.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[88-89]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> In addition, the media application may use multiple types of optical character
> recognition and/or fuzzy logic, for example, when determining the context of a
> keyword(s) retrieved from data (e.g., media data, translated audio data, subtitle data,
> user-generated data, etc.) associated with the media asset (or when cross-referencing
> various types of data with databases indicating the different contexts of events as
> described below).
- au=1.7844809008238371e-06, eu=0.2724013328552246, au′=-0.0, eu′=0.036087363958358765
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[26.710527420043945]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 88 | 88 | 26.710527420043945 | In addition, the media application may use multiple types of optical character recognition and/or fuzzy logic, for example, when determining the context of a keyword(s) retrieved from data (e.g., media data, translated audio data, subtitle data, user-generated data, etc.) |
- GPT 引用1: span=[88, 89], relevant=1

### 陈述 6
- Analysis of relationships between words and other information obtained by the content-recognition module to understand the meaning of the event.
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[128-128]`
- au=1.9067709899900365e-06, eu=0.29687488079071045, au′=-0.0, eu′=0.04051172360777855
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.68421173095703]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 128 | 128 | 23.68421173095703 | The content-recognition module or algorithm may also determine the relationships between words and/or other information obtained by the content-recognition module. |
- GPT 引用1: span=[128, 128], relevant=1

### 陈述 7
- Cross-referencing the information obtained with a database to determine the context of the event. For example, determining who said a statement, what the statement refers to, the relationships between words, and the meaning of keywords.
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[123-123]`
- au=1.7496245163783897e-06, eu=0.31275713443756104, au′=-0.0, eu′=0.04352806136012077
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[21.973684310913086]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 123 | 123 | 21.973684310913086 | Furthermore, by cross-referencing the words and/or other information obtained by the content-recognition module (e.g., as discussed below in relation to FIG. |

**跨度** `[128-128]`
- au=0.6819146871566772, eu=0.18052251636981964, au′=0.6819138526916504, eu′=0.04219876974821091
- top_logits 方差 (pvariance, 有限 logit): 0.35058822658265854
- top_logits 数值: `[23.289474487304688, 22.10526466369629]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 128 | 128 | 23.289474487304688 | The content-recognition module or algorithm may also determine the relationships between words and/or other information obtained by the content-recognition module. |
| 2 | 125 | 125 | 22.10526466369629 | The content-recognition module or algorithm may also determine that the term “export” refers to shipping goods out of a country. |

**跨度** `[129-129]`
- au=1.7630600268603303e-06, eu=0.3153526186943054, au′=-0.0, eu′=0.04403243958950043
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[21.710527420043945]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 129 | 129 | 21.710527420043945 | For example, by processing the relationship between the words, the media application determines that event 106 is a statement regarding a particular amount of a particular substance shipped out of a particular country. |
- GPT 引用1: span=[123, 123], relevant=1
- GPT 引用2: span=[128, 129], relevant=1

### 陈述 8
So in summary, the content-recognition module uses various techniques to analyze and understand the data associated with an event in a media asset to determine the context of the event. The context is the set of circumstances or facts that surround an event that influence or affect the meaning of the event.
- GPT support_score=1; [[No]]  Need Citation: [[No]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

## multifieldqa_en_idx132 (idx=132, multifieldqa_en)
### 问题
What are the titles of one of Kam W. Leong's publications in Journal of Controlled Release?

### 参考答案（数据集）
- ref1: Sustained viral gene delivery through core-shell fibers and Gene transfer to hemophilia A mice via oral delivery of FVIII-chitosan nanoparticles.

### 模型全文输出
```
<statement>Based on the chronological alphabetical combined bibtex listing, one of Kam W. Leong's publications in Journal of Controlled Release is:<cite></cite></statement>

<statement>Chen, Beiyi and Dang, Jiyoung and Tan, Tuan Lin and Fang, Ning and Chen, Wei Ning and Leong, Kam W. and Chan, Vincent, Dynamics of smooth muscle cell deadhesion from thermosensitive hydroxybutyl chitosan, Biomaterials, vol. 28 no. 8 (2007), pp. 1503 - 1514<cite>[96-102]</cite></statement>

<statement>The title of this publication is "Dynamics of smooth muscle cell deadhesion from thermosensitive hydroxybutyl chitosan".<cite></cite></statement>
```

文档句索引范围: sid ∈ [1, 174], 共 174 句。
引用指标: R=0.3333333333333333, P=0.0, F1=0.0
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=1899, completion_tokens=35
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
Based on the chronological alphabetical combined bibtex listing, one of Kam W. Leong's publications in Journal of Controlled Release is:
- GPT support_score=1; [[No]]  Need Citation: [[No]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 1
Chen, Beiyi and Dang, Jiyoung and Tan, Tuan Lin and Fang, Ning and Chen, Wei Ning and Leong, Kam W. and Chan, Vincent, Dynamics of smooth muscle cell deadhesion from thermosensitive hydroxybutyl chitosan, Biomaterials, vol. 28 no. 8 (2007), pp. 1503 - 1514
- GPT support_score=0; Rating: [[No support]]

**跨度** `[96-102]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> 847 -- 853, ISSN 0006-291X [abs] Chen, Beiyi and Dang, Jiyoung and Tan, Tuan Lin and
> Fang, Ning and Chen, Wei Ning and Leong, Kam W. and Chan, Vincent, Dynamics of smooth
> muscle cell deadhesion from thermosensitive hydroxybutyl chitosan, Biomaterials, vol. 28
> no. 8 (2007), pp. 1503 - 1514 [027] [abs] Chen, B. and Dang, J. and Tan, T. L. and Fang,
> N. and Chen, W. N. and Leong, K. W. and Chan, V., Dynamics of smooth muscle cell
> deadhesion from thermosensitive hydroxybutyl chitosan, Biomaterials, vol. 28 no. 8
> (2007), pp. 1503-14 [abs] Park, D. J. and Choi, J. H. and Leong, K. W. and Kwon, J. W.
> and Eun, H. S., Tissue-engineered bone formation with gene transfer and mesenchymal stem
> cells in a minimally invasive technique, Laryngoscope, vol.
- au=1.6567056491112453e-06, eu=0.2945735454559326, au′=-0.0, eu′=0.04008438438177109
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.947368621826172]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 96 | 96 | 23.947368621826172 | 847 -- 853, ISSN 0006-291X [abs] Chen, Beiyi and Dang, Jiyoung and Tan, Tuan Lin and Fang, Ning and Chen, Wei Ning and Leong, Kam W. and Chan, Vincent, Dynamics of smooth muscle cell deadhesion from thermosensitive hydroxybutyl chitosan, Biomaterials, vol. |
- GPT 引用1: span=[96, 102], relevant=0

### 陈述 2
The title of this publication is "Dynamics of smooth muscle cell deadhesion from thermosensitive hydroxybutyl chitosan".
- GPT support_score=0; [[Yes]]   Need Citation: [[Yes]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

## multifieldqa_en_idx133 (idx=133, multifieldqa_en)
### 问题
What was the club known as before being officially renamed FC Urartu?

### 参考答案（数据集）
- ref1: FC Banants.

### 模型全文输出
```
<statement>Before being officially renamed FC Urartu, the club was known as FC Banants.<cite>[5-5]</cite></statement>

<statement>The name change occurred on 1 August 2019.<cite>[5-5]</cite></statement>
```

文档句索引范围: sid ∈ [1, 38], 共 38 句。
引用指标: R=1.0, P=1.0, F1=1.0
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=1170, completion_tokens=26
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
Before being officially renamed FC Urartu, the club was known as FC Banants.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[5-5]`
- au=0.6820250749588013, eu=0.18313249945640564, au′=0.6820241808891296, eu′=0.04291360080242157
- top_logits 方差 (pvariance, 有限 logit): 0.038953996431700944
- top_logits 数值: `[22.5, 22.10526466369629]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 5 | 5 | 22.5 | The club was known as FC Banants until 1 August 2019, when it was officially renamed FC Urartu. |
| 2 | 1 | 1 | 22.10526466369629 | Football Club Urartu (, translated Futbolayin Akumb Urartu), commonly known as Urartu, is an Armenian professional football team based in the capital Yerevan that currently plays in the Armenian Premier League. |
- GPT 引用1: span=[5, 5], relevant=1

### 陈述 1
The name change occurred on 1 August 2019.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[5-5]`
- au=1.5858721553740907e-06, eu=0.2804427146911621, au′=-0.0, eu′=0.03751233592629433
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.657896041870117]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 5 | 5 | 25.657896041870117 | The club was known as FC Banants until 1 August 2019, when it was officially renamed FC Urartu. |
- GPT 引用1: span=[5, 5], relevant=1

## multifieldqa_en_idx134 (idx=134, multifieldqa_en)
### 问题
What was the conclusion of the study?

### 参考答案（数据集）
- ref1: The conclusion was that fruit consumption may provide a protective effect for mercury exposure in Amazonian riparians.

### 模型全文输出
```
<statement>Based on the abstract, the conclusion of the study was:<cite></cite></statement>

<statement>"Findings suggest that fatigue may be associated with eating high mercury fish but sample size is small. Larger studies are needed to confirm these associations."<cite>[137-138]</cite></statement>
```

文档句索引范围: sid ∈ [1, 138], 共 138 句。
引用指标: R=0.75, P=1.0, F1=0.8571428571428571
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=832, completion_tokens=24
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
Based on the abstract, the conclusion of the study was:
- GPT support_score=1; [[No]]   Need Citation: [[No]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 1
"Findings suggest that fatigue may be associated with eating high mercury fish but sample size is small. Larger studies are needed to confirm these associations."
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[137-138]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> Findings suggest that fatigue may be associated with eating high mercury fish but sample
> size is small. Larg
- au=1.650961507948523e-06, eu=0.2934361696243286, au′=-0.0, eu′=0.03987407684326172
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.078948974609375]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 137 | 137 | 24.078948974609375 | Findings suggest that fatigue may be associated with eating high mercury fish but sample size is small. |
- GPT 引用1: span=[137, 138], relevant=1

## multifieldqa_en_idx135 (idx=135, multifieldqa_en)
### 问题
How does the scoring engine generate a stream of content for the channel?

### 参考答案（数据集）
- ref1: By comparing candidate content items to a model and scoring them.

### 模型全文输出
```
<statement>Based on the patent, the scoring engine generates a stream of content for the channel by:<cite></cite></statement>

<statement>- Receiving a request from the channel engine for a stream of content for a channel. The request includes a channel category and channel attributes.<cite>[320-322]</cite></statement>

<statement>- Querying new content items stored in memory or a data storage server based on the channel category and channel attributes. The query generator generates a query based on the channel attributes.<cite>[320-322][321-321]</cite></statement>

<statement>- Receiving candidate content items that include the channel category and channel attributes. The content stream generator receives the candidate content items.<cite>[324-325]</cite></statement>

<statement>- Scoring each candidate content item by comparing it to a model generated for the user. The content stream generator determines an interestingness of each candidate content item to the user by comparing it to the model and scoring it.<cite>[375-377]</cite></statement>

<statement>- Generating the stream of content based on the scores for each candidate content item. The content stream generator generates the stream of content based on the scores.<cite>[334-334][377-377]</cite></statement>

<statement>- Optionally filtering the stream of content to include only candidate content items that exceed a certain threshold score.<cite>[335-335]</cite></statement>

<statement>- Transmitting the stream of content to the channel engine.<cite></cite></statement>

<statement>So in summary, the scoring engine uses a model of the user's interests and activities, along with channel attributes and historical trends, to score and select relevant content items to generate a personalized stream of content for the channel.<cite></cite></statement>
```

文档句索引范围: sid ∈ [1, 441], 共 441 句。
引用指标: R=0.5555555555555556, P=1.0, F1=0.7142857142857143
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=5703, completion_tokens=123
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
Based on the patent, the scoring engine generates a stream of content for the channel by:
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 1
- Receiving a request from the channel engine for a stream of content for a channel. The request includes a channel category and channel attributes.
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[320-322]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> The query generator 301 receives a request for a stream of content for a channel from
> the channel engine 240. The query generator 301 generates a query based on the channel
> attributes that are included in the request. The query generator 301 queries the data
> storage server 265 or memory 237 depending upon the embodiment.
- au=1.365067720413208, eu=0.1215999647974968, au′=1.3650673627853394, eu′=0.052468061447143555
- top_logits 方差 (pvariance, 有限 logit): 0.4533850634859391
- top_logits 数值: `[18.8157901763916, 18.289474487304688, 18.157896041870117, 16.973684310913086]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 256 | 256 | 18.8157901763916 | In one embodiment, the channel engine 240 transmits a query based on the channel category to the scoring server 262. |
| 2 | 195 | 195 | 18.289474487304688 | The channel engine 240 submits a request for a stream of content that includes the channel category and channel attributes to the scoring engine 211. |
| 3 | 320 | 320 | 18.157896041870117 | The query generator 301 receives a request for a stream of content for a channel from the channel engine 240. |
| 4 | 297 | 297 | 16.973684310913086 | The channel generator 378 submits a request for a stream of content for a channel to the scoring engine 211. |
- GPT 引用1: span=[320, 322], relevant=1

### 陈述 2
- Querying new content items stored in memory or a data storage server based on the channel category and channel attributes. The query generator generates a query based on the channel attributes.
- GPT support_score=1; **Rating**: [[Fully supported]]   **

**跨度** `[320-322]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> The query generator 301 receives a request for a stream of content for a channel from
> the channel engine 240. The query generator 301 generates a query based on the channel
> attributes that are included in the request. The query generator 301 queries the data
> storage server 265 or memory 237 depending upon the embodiment.
- au=1.081560492515564, eu=0.14205606281757355, au′=1.0815600156784058, eu′=0.047322534024715424
- top_logits 方差 (pvariance, 有限 logit): 0.5540164091580664
- top_logits 数值: `[21.18421173095703, 19.60526466369629, 19.60526466369629]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 320 | 320 | 21.18421173095703 | The query generator 301 receives a request for a stream of content for a channel from the channel engine 240. |
| 2 | 372 | 372 | 19.60526466369629 | The query generator 301 generates a query based on the channel category and the channel attributes and queries 710 the new content items stored on the data storage server 265. |
| 3 | 321 | 321 | 19.60526466369629 | The query generator 301 generates a query based on the channel attributes that are included in the request. |

**跨度** `[321-321]`
- au=0.681096613407135, eu=0.1885855346918106, au′=0.6810957789421082, eu′=0.04441846162080765
- top_logits 方差 (pvariance, 有限 logit): 0.523718981861748
- top_logits 数值: `[22.23684310913086, 20.789474487304688]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 372 | 372 | 22.23684310913086 | The query generator 301 generates a query based on the channel category and the channel attributes and queries 710 the new content items stored on the data storage server 265. |
| 2 | 321 | 321 | 20.789474487304688 | The query generator 301 generates a query based on the channel attributes that are included in the request. |
- GPT 引用1: span=[320, 322], relevant=1

### 陈述 3
- Receiving candidate content items that include the channel category and channel attributes. The content stream generator receives the candidate content items.
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[324-325]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> The content stream generator 304 receives candidate content items that include the
> channel attributes. The content stream generator 304, for the above mentioned query,
> receives text based articles that include the channel category politics and have a
> global score greater than 80.
- au=0.6814087629318237, eu=0.19095472991466522, au′=0.6814078092575073, eu′=0.04507710412144661
- top_logits 方差 (pvariance, 有限 logit): 0.0692520511447583
- top_logits 数值: `[21.447368621826172, 20.921052932739258]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 324 | 324 | 21.447368621826172 | The content stream generator 304 receives candidate content items that include the channel attributes. |
| 2 | 373 | 373 | 20.921052932739258 | The content stream generator 304 receives 712 candidate content items that include the channel category and channel attributes. |
- GPT 引用1: span=[324, 325], relevant=1

### 陈述 4
- Scoring each candidate content item by comparing it to a model generated for the user. The content stream generator determines an interestingness of each candidate content item to the user by comparing it to the model and scoring it.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[375-377]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> The content stream generator 304 scores 714 each candidate content item by comparing it
> to a model generated by the model generation engine 207. The score is calculated by
> determining an interestingness of the candidate content item to the user. The content
> stream generator 304 then generates 716 the stream of content based on the scores for
> each candidate content item.
- au=0.6817265748977661, eu=0.1876542717218399, au′=0.6817258596420288, eu′=0.04416036233305931
- top_logits 方差 (pvariance, 有限 logit): 0.004328221825744549
- top_logits 数值: `[21.710527420043945, 21.578948974609375]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 375 | 375 | 21.710527420043945 | The content stream generator 304 scores 714 each candidate content item by comparing it to a model generated by the model generation engine 207. |
| 2 | 207 | 207 | 21.578948974609375 | The scoring engine 211 then compares the candidate content items to the model to determine whether the user would find the candidate content items interesting. |
- GPT 引用1: span=[375, 377], relevant=1

### 陈述 5
- Generating the stream of content based on the scores for each candidate content item. The content stream generator generates the stream of content based on the scores.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[334-334]`
- au=0.682465136051178, eu=0.17674414813518524, au′=0.6824643015861511, eu′=0.04117009416222572
- top_logits 方差 (pvariance, 有限 logit): 0.0692525530794228
- top_logits 数值: `[23.55263328552246, 23.026315689086914]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 334 | 334 | 23.55263328552246 | Once the scores are calculated, the content stream generator 304 generates a stream of content for the channel that is ordered according to the candidate content item scores. |
| 2 | 377 | 377 | 23.026315689086914 | The content stream generator 304 then generates 716 the stream of content based on the scores for each candidate content item. |

**跨度** `[377-377]`
- au=1.9009210063813953e-06, eu=0.2957197427749634, au′=-0.0, eu′=0.04029691964387894
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.8157901763916]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 377 | 377 | 23.8157901763916 | The content stream generator 304 then generates 716 the stream of content based on the scores for each candidate content item. |
- GPT 引用1: span=[334, 334], relevant=1
- GPT 引用2: span=[377, 377], relevant=1

### 陈述 6
- Optionally filtering the stream of content to include only candidate content items that exceed a certain threshold score.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[335-335]`
- au=1.8140860902349232e-06, eu=0.27838820219039917, au′=-0.0, eu′=0.037145648151636124
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.921052932739258]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 335 | 335 | 25.921052932739258 | In one embodiment, only the candidate content items that exceed a certain threshold are included in the stream of content for the channel. |
- GPT 引用1: span=[335, 335], relevant=1

### 陈述 7
- Transmitting the stream of content to the channel engine.
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 8
So in summary, the scoring engine uses a model of the user's interests and activities, along with channel attributes and historical trends, to score and select relevant content items to generate a personalized stream of content for the channel.
- GPT support_score=0; [[Yes]]   Need Citation: [[Yes]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

## multifieldqa_en_idx136 (idx=136, multifieldqa_en)
### 问题
When did the club win the Armenian Premier League for the first time?

### 参考答案（数据集）
- ref1: In the 2013-2014 season.

### 模型全文输出
```
<statement>Based on the information provided, the club won the Armenian Premier League for the first time in their history during the 2013-2014 season.<cite>[3-3]</cite></statement>
```

文档句索引范围: sid ∈ [1, 38], 共 38 句。
引用指标: R=1.0, P=1.0, F1=1.0
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=610, completion_tokens=13
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
Based on the information provided, the club won the Armenian Premier League for the first time in their history during the 2013-2014 season.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[3-3]`
- au=1.6567056491112453e-06, eu=0.2945735454559326, au′=-0.0, eu′=0.04008438438177109
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.947368621826172]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 3 | 3 | 23.947368621826172 | In 2013–2014, they won the Armenian Premier League for the first time in their history. |
- GPT 引用1: span=[3, 3], relevant=1

## multifieldqa_en_idx137 (idx=137, multifieldqa_en)
### 问题
How many novels did Margaret Way write?

### 参考答案（数据集）
- ref1: Margaret Way wrote more than 120 novels.

### 模型全文输出
```
<statement>Based on the information provided, Margaret Way wrote over 120 novels since 1970.<cite>[2-2]</cite></statement>
```

文档句索引范围: sid ∈ [1, 18], 共 18 句。
引用指标: R=1.0, P=1.0, F1=1.0
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=614, completion_tokens=13
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
Based on the information provided, Margaret Way wrote over 120 novels since 1970.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[2-2]`
- au=0.682612955570221, eu=0.17551958560943604, au′=0.6826121807098389, eu′=0.04083825275301933
- top_logits 方差 (pvariance, 有限 logit): 0.004328347309638048
- top_logits 数值: `[23.55263328552246, 23.421052932739258]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 1 | 1 | 23.55263328552246 | Margaret Way (b. Brisbane d. Cleveland, Queensland, Australia ) was an Australian writer of romance novels and women's fiction. |
| 2 | 2 | 2 | 23.421052932739258 | A prolific author, Way wrote more than 120 novels since 1970, many through Mills & Boon, a romance imprint of British publisher Harlequin UK Ltd., owned by Harlequin Enterprises. |
- GPT 引用1: span=[2, 2], relevant=1

## multifieldqa_en_idx138 (idx=138, multifieldqa_en)
### 问题
What is the proposed approach in this research paper?

### 参考答案（数据集）
- ref1: This research paper proposed an approach based on approximating the posterior distribution with an isotropic Gaussian distribution.

### 模型全文输出
```
<statement>The proposed approach in this research paper is to provide a probabilistic interpretation of the least-mean-square (LMS) filter.<cite>[14-14]</cite></statement>

<statement>The paper approximates the posterior distribution of the parameter vector in a probabilistic state-space model with an isotropic Gaussian distribution.<cite>[14-16][37-37]</cite></statement>

<statement>This approximation leads to an LMS-like adaptive filter with an adaptable step size, which performs well in both stationary and non-stationary environments.<cite>[20-20][86-86]</cite></statement>

<statement>The proposed algorithm has fewer free parameters than previous LMS algorithms with variable step size and its parameters are easier to tune.<cite>[21-22]</cite></statement>

<statement>Moreover, the use of a probabilistic model provides an estimate of the error variance, which is useful in many applications.<cite>[23-23]</cite></statement>
```

文档句索引范围: sid ∈ [1, 107], 共 107 句。
引用指标: R=0.9, P=1.0, F1=0.9473684210526316
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=3913, completion_tokens=81
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
The proposed approach in this research paper is to provide a probabilistic interpretation of the least-mean-square (LMS) filter.
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[14-14]`
- au=0.6814142465591431, eu=0.19191914796829224, au′=0.6814134120941162, eu′=0.045346058905124664
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[21.052631378173828, 21.052631378173828]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 85 | 85 | 21.052631378173828 | \section{Conclusions and Opened Extensions} \label{sec:conclusions}  {We have presented a probabilistic interpretation of the least-mean-square filter. |
| 2 | 14 | 14 | 21.052631378173828 | In this work, we provide a similar connection between state-space models and least-mean-squares (LMS). |
- GPT 引用1: span=[14, 14], relevant=1

### 陈述 1
The paper approximates the posterior distribution of the parameter vector in a probabilistic state-space model with an isotropic Gaussian distribution.
- GPT support_score=1; **Rating**: [[Fully supported]]    **

**跨度** `[14-16]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> In this work, we provide a similar connection between state-space models and least-mean-
> squares (LMS). Our approach is based on approximating the posterior distribution with an
> isotropic Gaussian distribution. We show how the computation of this approximated
> posterior leads to a linear-complexity algorithm, comparable to the standard LMS.
- au=0.6820963621139526, eu=0.1826922744512558, au′=0.6820955276489258, eu′=0.04279278963804245
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[22.36842155456543, 22.36842155456543]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 14 | 14 | 22.36842155456543 | In this work, we provide a similar connection between state-space models and least-mean-squares (LMS). |
| 2 | 15 | 15 | 22.36842155456543 | Our approach is based on approximating the posterior distribution with an isotropic Gaussian distribution. |

**跨度** `[37-37]`
- au=0.6815455555915833, eu=0.1885855346918106, au′=0.6815447211265564, eu′=0.04441846162080765
- top_logits 方差 (pvariance, 有限 logit): 0.10820617305944324
- top_logits 数值: `[21.842105865478516, 21.18421173095703]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 37 | 37 | 21.842105865478516 | \section{Approximating the posterior distribution: LMS filter }  The proposed approach consists in approximating the posterior distribution $p({\bf w}_k\|y_{1:k})$, in general a multivariate Gaussian distribution with a full covariance matrix, by an isotropic spherical Gaussian distribution  \begin{equation} \label{eq:aprox_post} \hat{p}({\bf w}_{k}\|y_{1:k})=\mathcal{N}({\bf w}_{k};{\bf \hat{\boldsymbol\mu}}_{k}, \hat{\sigma}_{k}^2 {\bf I} ). |
| 2 | 35 | 35 | 21.18421173095703 | In the next section, we show how such a scalar is obtained naturally when $p({\bf w}_k\|y_{1:k})$ is approximated with an isotropic Gaussian distribution. |
- GPT 引用1: span=[14, 16], relevant=1
- GPT 引用2: span=[37, 37], relevant=1

### 陈述 2
This approximation leads to an LMS-like adaptive filter with an adaptable step size, which performs well in both stationary and non-stationary environments.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[20-20]`
- au=0.6818286180496216, eu=0.18536581099033356, au′=0.6818277835845947, eu′=0.04352806136012077
- top_logits 方差 (pvariance, 有限 logit): 0.0692520511447583
- top_logits 数值: `[22.23684310913086, 21.710527420043945]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 14 | 14 | 22.23684310913086 | In this work, we provide a similar connection between state-space models and least-mean-squares (LMS). |
| 2 | 20 | 20 | 21.710527420043945 | Firstly, a novel LMS algorithm with adaptable step size emerges naturally with this approach, making it suitable for both stationary and non-stationary environments. |

**跨度** `[86-86]`
- au=1.080199122428894, eu=0.14643540978431702, au′=1.080198049545288, eu′=0.04894804209470749
- top_logits 方差 (pvariance, 有限 logit): 1.1118796804423456
- top_logits 数值: `[20.921052932739258, 18.68421173095703, 18.68421173095703]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 86 | 86 | 20.921052932739258 | The resulting algorithm is an adaptable step-size LMS that performs well both in stationary and tracking scenarios. |
| 2 | 46 | 46 | 18.68421173095703 | \label{eq:prob_lms} \end{eqnarray} Note that, instead of a gain matrix ${\bf K}_k$ as in Eq.~\eqref{eq:prob_rls}, we now have a scalar gain $\eta_k$ that operates as a variable step size. |
| 3 | 36 | 36 | 18.68421173095703 | We also show that this approximation leads to an LMS-like estimation. |
- GPT 引用1: span=[20, 20], relevant=1
- GPT 引用2: span=[86, 86], relevant=1

### 陈述 3
The proposed algorithm has fewer free parameters than previous LMS algorithms with variable step size and its parameters are easier to tune.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[21-22]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> The proposed algorithm has less free parameters than previous LMS algorithms with
> variable step size \cite{kwong1992variable,aboulnasr1997robust,shin2004variable}, and
> its parameters are easier to be tuned w.r.t. these algorithms and standard LMS.
- au=1.9009210063813953e-06, eu=0.2957197427749634, au′=-0.0, eu′=0.04029691964387894
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.8157901763916]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 21 | 21 | 23.8157901763916 | The proposed algorithm has less free parameters than previous LMS algorithms with variable step size \cite{kwong1992variable,aboulnasr1997robust,shin2004variable}, and its parameters are easier to be tuned w.r.t. |
- GPT 引用1: span=[21, 22], relevant=1

### 陈述 4
Moreover, the use of a probabilistic model provides an estimate of the error variance, which is useful in many applications.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[23-23]`
- au=1.6452690942969639e-06, eu=0.29230761528015137, au′=-0.0, eu′=0.03966596722602844
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.210527420043945]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 23 | 23 | 24.210527420043945 | Secondly, the use of a probabilistic model provides us with an estimate of the error variance, which is useful in many applications. |
- GPT 引用1: span=[23, 23], relevant=1

## multifieldqa_en_idx139 (idx=139, multifieldqa_en)
### 问题
Does DUO contain more instances per image than COCO?

### 参考答案（数据集）
- ref1: Yes, DUO has 9.57 instances per image while COCO contains 7.7.

### 模型全文输出
```
<statement>Based on the information provided in the paper, DUO contains more instances per image than COCO.<cite></cite></statement>

<statement>- DUO has an average of 9.57 instances per image, with most images containing between 5 and 15 instances.<cite>[85-85]</cite></statement>

<statement>- COCO has an average of 7.7 instances per image.<cite>[88-88]</cite></statement>

<statement>So DUO has a higher average number of instances per image compared to COCO. The paper also notes that DUO has a long-tail distribution of instances sizes, with most instances being small, which is different from COCO where objects are more evenly distributed in size.<cite>[89-90][87-87]</cite></statement>
```

文档句索引范围: sid ∈ [1, 135], 共 135 句。
引用指标: R=0.625, P=0.75, F1=0.6818181818181818
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=2660, completion_tokens=58
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
Based on the information provided in the paper, DUO contains more instances per image than COCO.
- GPT support_score=0; [[Yes]]   Need Citation: [[Yes]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 1
- DUO has an average of 9.57 instances per image, with most images containing between 5 and 15 instances.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[85-85]`
- au=1.5807482895979774e-06, eu=0.2794116735458374, au′=-0.0, eu′=0.037328090518713
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.789474487304688]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 85 | 85 | 25.789474487304688 | Most images contain between 5 and 15 instances, with an average of 9.57 instances per image. |
- GPT 引用1: span=[85, 85], relevant=1

### 陈述 2
- COCO has an average of 7.7 instances per image.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[88-88]`
- au=1.7844809008238371e-06, eu=0.2724013328552246, au′=-0.0, eu′=0.036087363958358765
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[26.710527420043945]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 88 | 88 | 26.710527420043945 | In the aspect of instances number per image, COCO contains 7.7 instances per image and VOC contains 3. |
- GPT 引用1: span=[88, 88], relevant=1

### 陈述 3
So DUO has a higher average number of instances per image compared to COCO. The paper also notes that DUO has a long-tail distribution of instances sizes, with most instances being small, which is different from COCO where objects are more evenly distributed in size.
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[89-90]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> In comparison, DUO has 9.57 instances per image and most instances less than 1.5\% of
> the image size. Therefore, DUO contains almost exclusively massive small instances and
> has the long-tail distribution at the same time, which means it is promising to design a
> detector to deal with massive small objects and stay high efficiency at the same time
> for underwater robot picking.
- au=1.590732216835022, eu=0.08379270136356354, au′=1.590732216835022, eu′=0.04372841864824295
- top_logits 方差 (pvariance, 有限 logit): 0.5637118067438132
- top_logits 数值: `[23.289474487304688, 21.842105865478516, 21.710527420043945, 21.3157901763916, 21.18421173095703]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 89 | 89 | 23.289474487304688 | In comparison, DUO has 9.57 instances per image and most instances less than 1.5\% of the image size. |
| 2 | 85 | 85 | 21.842105865478516 | Most images contain between 5 and 15 instances, with an average of 9.57 instances per image. |
| 3 | 76 | 76 | 21.710527420043945 | \subsection{Dataset Statistics} {\bf The proportion of classes}: The total number of objects is 74,515. |
| 4 | 80 | 80 | 21.3157901763916 | {\bf The distribution of instance sizes}: Figure \ref{sum}(a) shows an instance size distribution of DUO. |
| 5 | 86 | 86 | 21.18421173095703 | {\bf Summary}: In general, smaller objects are harder to detect. |

**跨度** `[87-87]`
- au=1.0808675289154053, eu=0.1458732932806015, au′=1.0808672904968262, eu′=0.04873877018690109
- top_logits 方差 (pvariance, 有限 logit): 0.6617422682134525
- top_logits 数值: `[20.657896041870117, 19.078948974609375, 18.8157901763916]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 87 | 87 | 20.657896041870117 | For PASCAL VOC \cite{Everingham2007The} or COCO \cite{Belongie2014}, roughly 50\% of all objects occupy no more than 10\% of the image itself, and others evenly occupy from 10\% to 100\%. |
| 2 | 86 | 86 | 19.078948974609375 | {\bf Summary}: In general, smaller objects are harder to detect. |
| 3 | 127 | 127 | 18.8157901763916 | For class AP, AP$_{Sc}$ lags significantly behind the other three classes because it has the smallest number of instances. |
- GPT 引用1: span=[87, 87], relevant=0
- GPT 引用2: span=[89, 90], relevant=1

## multifieldqa_en_idx140 (idx=140, multifieldqa_en)
### 问题
What are the symptoms of vitamin K deficiency?

### 参考答案（数据集）
- ref1: Symptoms of vitamin K deficiency include anemia, bruising, nosebleeds, bleeding of the gums, and heavy menstrual bleeding in women.

### 模型全文输出
```
<statement>Based on the article, some symptoms of vitamin K deficiency include:<cite></cite></statement>

<statement>- Anemia <cite>[99-99]</cite></statement>

<statement>- Bruising <cite>[99-99]</cite></statement>

<statement>- Nosebleeds <cite>[99-99]</cite></statement>

<statement>- Bleeding of the gums <cite>[99-99]</cite></statement>

<statement>- Heavy menstrual bleeding in women <cite>[99-99]</cite></statement>

<statement>The article states that deficiency can result in coagulopathy, a bleeding disorder. It also notes that low levels of vitamin K can weaken bones and promote calcification of arteries and other soft tissues, although this is not explicitly listed as a symptom.<cite>[10-11]</cite></statement>
```

文档句索引范围: sid ∈ [1, 712], 共 712 句。
引用指标: R=1.0, P=1.0, F1=1.0
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=3820, completion_tokens=88
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
Based on the article, some symptoms of vitamin K deficiency include:
- GPT support_score=1; [[No]]   Need Citation: [[No]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 1
- Anemia
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[99-99]`
- au=0.6822757124900818, eu=0.17882347106933594, au′=0.6822749376296997, eu′=0.04173530638217926
- top_logits 方差 (pvariance, 有限 logit): 0.10820617305944324
- top_logits 数值: `[23.289474487304688, 22.631580352783203]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 99 | 99 | 23.289474487304688 | [50]Symptoms of K1 deficiency include anemia, bruising, nosebleeds and bleeding of the gums in both sexes, and heavy menstrual bleeding in women. |
| 2 | 98 | 98 | 22.631580352783203 | Vitamin K1 deficiency can result in coagulopathy, a bleeding disorder. |
- GPT 引用1: span=[99, 99], relevant=1

### 陈述 2
- Bruising
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[99-99]`
- au=1.6068129298218992e-06, eu=0.28464409708976746, au′=-0.0, eu′=0.038267869502305984
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.131580352783203]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 99 | 99 | 25.131580352783203 | [50]Symptoms of K1 deficiency include anemia, bruising, nosebleeds and bleeding of the gums in both sexes, and heavy menstrual bleeding in women. |
- GPT 引用1: span=[99, 99], relevant=1

### 陈述 3
- Nosebleeds
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[99-99]`
- au=1.8140860902349232e-06, eu=0.27838820219039917, au′=-0.0, eu′=0.037145648151636124
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.921052932739258]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 99 | 99 | 25.921052932739258 | [50]Symptoms of K1 deficiency include anemia, bruising, nosebleeds and bleeding of the gums in both sexes, and heavy menstrual bleeding in women. |
- GPT 引用1: span=[99, 99], relevant=1

### 陈述 4
- Bleeding of the gums
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[99-99]`
- au=1.7844809008238371e-06, eu=0.2724013328552246, au′=-0.0, eu′=0.036087363958358765
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[26.710527420043945]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 99 | 99 | 26.710527420043945 | [50]Symptoms of K1 deficiency include anemia, bruising, nosebleeds and bleeding of the gums in both sexes, and heavy menstrual bleeding in women. |
- GPT 引用1: span=[99, 99], relevant=1

### 陈述 5
- Heavy menstrual bleeding in women
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[99-99]`
- au=1.7844809008238371e-06, eu=0.2724013328552246, au′=-0.0, eu′=0.036087363958358765
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[26.710527420043945]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 99 | 99 | 26.710527420043945 | [50]Symptoms of K1 deficiency include anemia, bruising, nosebleeds and bleeding of the gums in both sexes, and heavy menstrual bleeding in women. |
- GPT 引用1: span=[99, 99], relevant=1

### 陈述 6
The article states that deficiency can result in coagulopathy, a bleeding disorder. It also notes that low levels of vitamin K can weaken bones and promote calcification of arteries and other soft tissues, although this is not explicitly listed as a symptom.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[10-11]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> Without vitamin K, blood coagulation is seriously impaired, and uncontrolled bleeding
> occurs. Low levels of vitamin K also weaken bones and promote calcification of arteries
> and other soft tissues[citation needed].
- au=0.6824589371681213, eu=0.17633406817913055, au′=0.6824581027030945, eu′=0.04105888307094574
- top_logits 方差 (pvariance, 有限 logit): 0.10820680047709175
- top_logits 数值: `[23.68421173095703, 23.026315689086914]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 11 | 11 | 23.68421173095703 | Low levels of vitamin K also weaken bones and promote calcification of arteries and other soft tissues[citation needed]. |
| 2 | 10 | 10 | 23.026315689086914 | Without vitamin K, blood coagulation is seriously impaired, and uncontrolled bleeding occurs. |
- GPT 引用1: span=[10, 11], relevant=1

## multifieldqa_en_idx141 (idx=141, multifieldqa_en)
### 问题
What is the security parameter for the AES-256 block cipher?

### 参考答案（数据集）
- ref1: 172.

### 模型全文输出
```
<statement>Based on the analysis in the paper, the quantum security parameter for the AES-256 block cipher is 172 bits.<cite>[351-351]</cite></statement>

<statement>This is calculated by fitting the space/time tradeoff data with a third degree polynomial and evaluating the polynomial at the point corresponding to breaking the cipher in 24 hours with a conservative physical error rate per gate of 10^-4. The resulting value of the polynomial at this point is 172 bits.<cite></cite></statement>

<statement>The paper also notes that the number of qubits required to break AES-256 in one day with a more optimistic physical error rate of 10^-5 is approximately 4.64 x 10^6 physical qubits.<cite></cite></statement>
```

文档句索引范围: sid ∈ [1, 371], 共 371 句。
引用指标: R=0.3333333333333333, P=1.0, F1=0.5
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=1558, completion_tokens=33
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
Based on the analysis in the paper, the quantum security parameter for the AES-256 block cipher is 172 bits.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[351-351]`
- au=1.974869974219473e-06, eu=0.3102039694786072, au′=-0.0, eu′=0.043035101145505905
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[22.23684310913086]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 351 | 351 | 22.23684310913086 | \begin{tabular}{ll} \hline Name    & qs  \\ \hline AES-128 & 106 \\ AES-192 & 139 \\ AES-256 & 172 \\ \hline SHA-256 & 166 \\ SHA3-256	 &167 \\ Bitcoin's PoW & 75\\ \hline \end{tabular} \caption{Quantum security parameter ($qs$) for the AES family of ciphers, SHA family of hash functions, and Bitcoin, assuming a conservative physical error rate per gate $p_g=10^{-4}$.} |
- GPT 引用1: span=[351, 351], relevant=1

### 陈述 1
This is calculated by fitting the space/time tradeoff data with a third degree polynomial and evaluating the polynomial at the point corresponding to breaking the cipher in 24 hours with a conservative physical error rate per gate of 10^-4. The resulting value of the polynomial at this point is 172 bits.
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 2
The paper also notes that the number of qubits required to break AES-256 in one day with a more optimistic physical error rate of 10^-5 is approximately 4.64 x 10^6 physical qubits.
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

## multifieldqa_en_idx142 (idx=142, multifieldqa_en)
### 问题
What is the definition of mobile device management (MDM)?

### 参考答案（数据集）
- ref1: Centralized control of mobile devices and applications.

### 模型全文输出
```
<statement>Based on the paper, mobile device management (MDM) is defined as follows:<cite></cite></statement>

<statement>- "Mobile device management (MDM) systems are usually referred to “support centralized control of an entire fleet of mobile devices (smartphones and tablets) and mobile applications by applying and ensuring pre-defined configuration settings”" <cite>[22-22]</cite></statement>

<statement>- "MDM systems are today a very common tool to manage users’ devices. With MDM, all mobile device types, tablets and PCs with typical operating systems can be controlled centrally. It is often thought that MDM can manage only mobile phones, but actually the MDM framework includes also users’ identities and profiles. This makes MDM a viable tool for organizations to manage their employees identities, user profiles, all devices, all applications and security controls under same system."<cite>[59-62]</cite></statement>

<statement>So in summary, MDM is a system that allows organizations to centrally manage and control mobile devices (smartphones, tablets, etc) and mobile applications by applying pre-defined configuration settings. It includes managing user identities and profiles as well. The goal is to ensure security and compliance across a fleet of mobile devices.<cite></cite></statement>
```

文档句索引范围: sid ∈ [1, 435], 共 435 句。
引用指标: R=0.75, P=1.0, F1=0.8571428571428571
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=2453, completion_tokens=46
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
Based on the paper, mobile device management (MDM) is defined as follows:
- GPT support_score=1; [[No]]   Need Citation: [[No]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 1
- "Mobile device management (MDM) systems are usually referred to “support centralized control of an entire fleet of mobile devices (smartphones and tablets) and mobile applications by applying and ensuring pre-defined configuration settings”"
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[22-22]`
- au=1.6175589507838595e-06, eu=0.2867923676967621, au′=-0.0, eu′=0.03865716978907585
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.86842155456543]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 22 | 22 | 24.86842155456543 | Mobile device management (MDM) systems are usually referred to “support centralized control of an entire fleet of mobile devices (smartphones and tablets) and mobile applications by applying and ensuring pre-defined configuration settings” [8, 9]. |
- GPT 引用1: span=[22, 22], relevant=1

### 陈述 2
- "MDM systems are today a very common tool to manage users’ devices. With MDM, all mobile device types, tablets and PCs with typical operating systems can be controlled centrally. It is often thought that MDM can manage only mobile phones, but actually the MDM framework includes also users’ identities and profiles. This makes MDM a viable tool for organizations to manage their employees identities, user profiles, all devices, all applications and security controls under same system."
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[59-62]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> MDM systems are today a very common tool to manage users’ devices. With MDM, all mobile
> device types, tablets and PCs with typical operating systems can be controlled centrally
> . It is often thought that MDM can manage only mobile phones, but actually the MDM
> framework includes also users’ identities and profiles. This makes MDM a viable tool for
> organizations to manage their employees identities, user profiles, all devices, all
> applications and security controls under same system.
- au=1.6175589507838595e-06, eu=0.2867923676967621, au′=-0.0, eu′=0.03865716978907585
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.86842155456543]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 59 | 59 | 24.86842155456543 | MDM systems are today a very common tool to manage users’ devices. |
- GPT 引用1: span=[59, 62], relevant=1

### 陈述 3
So in summary, MDM is a system that allows organizations to centrally manage and control mobile devices (smartphones, tablets, etc) and mobile applications by applying pre-defined configuration settings. It includes managing user identities and profiles as well. The goal is to ensure security and compliance across a fleet of mobile devices.
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

## multifieldqa_en_idx143 (idx=143, multifieldqa_en)
### 问题
What models were used for dialect identification?

### 参考答案（数据集）
- ref1: BERT, RoBERTa, ELECTRA, GPT-2, and XLM-RoBERTa.

### 模型全文输出
```
<statement>Based on the paper, the following models were used for dialect identification:<cite></cite></statement>

<statement>- BERT <cite>[60-60][107-107]</cite></statement>

<statement>- RoBERTa<cite>[60-60][65-65][107-107]</cite></statement>

<statement>- ELECTRA<cite>[65-65][107-107]</cite></statement>

<statement>- GPT-2<cite>[65-65][71-71][73-73]</cite></statement>

<statement>- XLM-RoBERTa<cite>[65-65][70-70][72-72]</cite></statement>

<statement>The best performing models for each language were chosen for the final pipeline. For English, RoBERTa and BERT performed well. For Spanish, the language specific version of RoBERTa performed well. For Portuguese, the language specific version of BERT performed well.<cite>[70-72]</cite></statement>

<statement>So in summary, BERT, RoBERTa, ELECTRA, GPT-2 and XLM-RoBERTa were all experimented with for dialect identification, but the best performing models for each language were ultimately chosen for the final system.<cite></cite></statement>
```

文档句索引范围: sid ∈ [1, 130], 共 130 句。
引用指标: R=0.75, P=0.8571428571428571, F1=0.7999999999999999
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=6584, completion_tokens=150
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
Based on the paper, the following models were used for dialect identification:
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 1
- BERT
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[60-60]`
- au=1.729960786178708e-06, eu=0.3089430034160614, au′=-0.0, eu′=0.04279278963804245
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[22.36842155456543]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 60 | 60 | 22.36842155456543 | For dialect identification we have used models like BERT and RoBERTa with a linear layer connected to the pooler output of the models. |

**跨度** `[107-107]`
- au=0.6809439659118652, eu=0.1862744688987732, au′=0.6809432506561279, eu′=0.04377879574894905
- top_logits 方差 (pvariance, 有限 logit): 0.8483385049075878
- top_logits 数值: `[22.763158798217773, 20.921052932739258]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 107 | 107 | 22.763158798217773 | BERT-based models outperform other models across all languages: We observe that BERT-based models outperform ELECTRA-based and GPT-2-based models, as shown in Table . |
| 2 | 65 | 65 | 20.921052932739258 | 5 Experiments and Results  Experiments using Large Language Models  For the task of Dialect Identification we have tried various language specific models like XLM-RoBERTa, BERT, ELECTRA, RoBERTa and GPT- 2. |
- GPT 引用1: span=[60, 60], relevant=1
- GPT 引用2: span=[107, 107], relevant=1

### 陈述 2
- RoBERTa
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[60-60]`
- au=1.6802155187178869e-06, eu=0.2992125153541565, au′=-0.0, eu′=0.04094827175140381
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.421052932739258]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 60 | 60 | 23.421052932739258 | For dialect identification we have used models like BERT and RoBERTa with a linear layer connected to the pooler output of the models. |

**跨度** `[65-65]`
- au=1.367969274520874, eu=0.10951007902622223, au′=1.3679687976837158, eu′=0.04688463732600212
- top_logits 方差 (pvariance, 有限 logit): 0.05626751115141815
- top_logits 数值: `[20.657896041870117, 20.394737243652344, 20.263158798217773, 20.0]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 107 | 107 | 20.657896041870117 | BERT-based models outperform other models across all languages: We observe that BERT-based models outperform ELECTRA-based and GPT-2-based models, as shown in Table . |
| 2 | 71 | 71 | 20.394737243652344 | The best performing models for the English language were RoBERTa and BERT whereas GPT-2 was the worst performing. |
| 3 | 62 | 62 | 20.263158798217773 | For the task of dialect identification we experimented with several pretrained models like XLM-RoBERTa, BERT, ELECTRA, GPT-2 and RoBERTa. |
| 4 | 65 | 65 | 20.0 | 5 Experiments and Results  Experiments using Large Language Models  For the task of Dialect Identification we have tried various language specific models like XLM-RoBERTa, BERT, ELECTRA, RoBERTa and GPT- 2. |

**跨度** `[107-107]`
- au=1.0826994180679321, eu=0.1379310041666031, au′=1.0826990604400635, eu′=0.045801520347595215
- top_logits 方差 (pvariance, 有限 logit): 0.07309969405711551
- top_logits 数值: `[21.18421173095703, 20.789474487304688, 20.526315689086914]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 71 | 71 | 21.18421173095703 | The best performing models for the English language were RoBERTa and BERT whereas GPT-2 was the worst performing. |
| 2 | 70 | 70 | 20.789474487304688 | We used XLM-RoBERTa as the baseline for all 3 languages. |
| 3 | 107 | 107 | 20.526315689086914 | BERT-based models outperform other models across all languages: We observe that BERT-based models outperform ELECTRA-based and GPT-2-based models, as shown in Table . |
- GPT 引用1: span=[60, 60], relevant=1
- GPT 引用2: span=[65, 65], relevant=1
- GPT 引用3: span=[107, 107], relevant=0

### 陈述 3
- ELECTRA
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[65-65]`
- au=1.9430344764259644e-06, eu=0.3039999008178711, au′=-0.0, eu′=0.041850216686725616
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[22.894737243652344]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 65 | 65 | 22.894737243652344 | 5 Experiments and Results  Experiments using Large Language Models  For the task of Dialect Identification we have tried various language specific models like XLM-RoBERTa, BERT, ELECTRA, RoBERTa and GPT- 2. |

**跨度** `[107-107]`
- au=1.6175589507838595e-06, eu=0.2867923676967621, au′=-0.0, eu′=0.03865716978907585
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.86842155456543]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 107 | 107 | 24.86842155456543 | BERT-based models outperform other models across all languages: We observe that BERT-based models outperform ELECTRA-based and GPT-2-based models, as shown in Table . |
- GPT 引用1: span=[65, 65], relevant=1
- GPT 引用2: span=[107, 107], relevant=1

### 陈述 4
- GPT-2
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[65-65]`
- au=1.6396279534092173e-06, eu=0.2911876440048218, au′=-0.0, eu′=0.039460018277168274
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.342105865478516]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 65 | 65 | 24.342105865478516 | 5 Experiments and Results  Experiments using Large Language Models  For the task of Dialect Identification we have tried various language specific models like XLM-RoBERTa, BERT, ELECTRA, RoBERTa and GPT- 2. |

**跨度** `[71-71]`
- au=0.6816202402114868, eu=0.1885855793952942, au′=0.68161940574646, eu′=0.04441846162080765
- top_logits 方差 (pvariance, 有限 logit): 0.03895437288156245
- top_logits 数值: `[21.710527420043945, 21.3157901763916]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 107 | 107 | 21.710527420043945 | BERT-based models outperform other models across all languages: We observe that BERT-based models outperform ELECTRA-based and GPT-2-based models, as shown in Table . |
| 2 | 71 | 71 | 21.3157901763916 | The best performing models for the English language were RoBERTa and BERT whereas GPT-2 was the worst performing. |

**跨度** `[73-73]`
- au=0.6825650334358215, eu=0.17431187629699707, au′=0.6825642585754395, eu′=0.04051172360777855
- top_logits 方差 (pvariance, 有限 logit): 0.1558174915262498
- top_logits 数值: `[24.078948974609375, 23.289474487304688]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 73 | 73 | 24.078948974609375 | Overall the worst performing model was GPT-2 across all 3 languages. |
| 2 | 107 | 107 | 23.289474487304688 | BERT-based models outperform other models across all languages: We observe that BERT-based models outperform ELECTRA-based and GPT-2-based models, as shown in Table . |
- GPT 引用1: span=[65, 65], relevant=1
- GPT 引用2: span=[71, 71], relevant=1
- GPT 引用3: span=[73, 73], relevant=1

### 陈述 5
- XLM-RoBERTa
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[65-65]`
- au=1.3667162656784058, eu=0.11411409825086594, au′=1.3667161464691162, eu′=0.049000635743141174
- top_logits 方差 (pvariance, 有限 logit): 0.35058847754953604
- top_logits 数值: `[20.263158798217773, 19.60526466369629, 19.078948974609375, 18.68421173095703]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 41 | 41 | 20.263158798217773 | For our experiments, we use a version of XLM-RoBERTa finetuned on a language identification dataset 2 . |
| 2 | 65 | 65 | 19.60526466369629 | 5 Experiments and Results  Experiments using Large Language Models  For the task of Dialect Identification we have tried various language specific models like XLM-RoBERTa, BERT, ELECTRA, RoBERTa and GPT- 2. |
| 3 | 55 | 55 | 19.078948974609375 | The Language Identification (LID) model which is the first stage classifies the sentence into 3 languages: English (EN), Spanish (ES) and Portuguese (PT). |
| 4 | 56 | 56 | 18.68421173095703 | The LID is a pretrained XLM-RoBERTa that is fine-tuned for the task of language identification. |

**跨度** `[70-70]`
- au=0.6803783178329468, eu=0.19437333941459656, au′=0.6803776621818542, eu′=0.046032704412937164
- top_logits 方差 (pvariance, 有限 logit): 0.7314760136805489
- top_logits 数值: `[21.578948974609375, 19.86842155456543]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 70 | 70 | 21.578948974609375 | We used XLM-RoBERTa as the baseline for all 3 languages. |
| 2 | 67 | 67 | 19.86842155456543 | The pooler output of these models was passed through a linear layer and the models were fine-tuned. |

**跨度** `[72-72]`
- au=1.0817255973815918, eu=0.14205606281757355, au′=1.0817251205444336, eu′=0.047322534024715424
- top_logits 方差 (pvariance, 有限 logit): 0.4155123068687519
- top_logits 数值: `[20.921052932739258, 20.131580352783203, 19.342105865478516]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 72 | 72 | 20.921052932739258 | Similarly the language specific versions of RoBERTa and BERT performed well for the Spanish and Portuguese respectively. |
| 2 | 71 | 71 | 20.131580352783203 | The best performing models for the English language were RoBERTa and BERT whereas GPT-2 was the worst performing. |
| 3 | 73 | 73 | 19.342105865478516 | Overall the worst performing model was GPT-2 across all 3 languages. |
- GPT 引用1: span=[65, 65], relevant=1
- GPT 引用2: span=[70, 70], relevant=1
- GPT 引用3: span=[72, 72], relevant=0

### 陈述 6
The best performing models for each language were chosen for the final pipeline. For English, RoBERTa and BERT performed well. For Spanish, the language specific version of RoBERTa performed well. For Portuguese, the language specific version of BERT performed well.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[70-72]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> We used XLM-RoBERTa as the baseline for all 3 languages. The best performing models for
> the English language were RoBERTa and BERT whereas GPT-2 was the worst performing.
> Similarly the language specific versions of RoBERTa and BERT performed well for the
> Spanish and Portuguese respectively.
- au=0.6825187802314758, eu=0.17551958560943604, au′=0.6825180053710938, eu′=0.04083825275301933
- top_logits 方差 (pvariance, 有限 logit): 0.10820617305944324
- top_logits 数值: `[23.8157901763916, 23.157896041870117]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 70 | 70 | 23.8157901763916 | We used XLM-RoBERTa as the baseline for all 3 languages. |
| 2 | 71 | 71 | 23.157896041870117 | The best performing models for the English language were RoBERTa and BERT whereas GPT-2 was the worst performing. |
- GPT 引用1: span=[70, 72], relevant=1

### 陈述 7
So in summary, BERT, RoBERTa, ELECTRA, GPT-2 and XLM-RoBERTa were all experimented with for dialect identification, but the best performing models for each language were ultimately chosen for the final system.
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

## multifieldqa_en_idx144 (idx=144, multifieldqa_en)
### 问题
What are the restrictions on the use of Broadjam's servers?

### 参考答案（数据集）
- ref1: No excessive overloading and no use for illegal activity.

### 模型全文输出
```
<statement>Based on the Broadjam agreement, some key restrictions on the use of Broadjam's servers include:<cite></cite></statement>

<statement>- Hosting Subscriber's Website will not be used in connection with any illegal activity.<cite>[277-277]</cite></statement>

<statement>- Hosting Subscriber is responsible for ensuring there is no excessive overloading on Broadjam's DNS or servers.<cite>[278-278]</cite></statement>

<statement>- Broadjam prohibits the use of software or scripts run on its servers that cause the server to load beyond a reasonable level.<cite>[279-279]</cite></statement>

<statement>- Hosting Subscriber agrees Broadjam reserves the right to remove Hosting Subscriber's Website temporarily or permanently if it threatens the stability of Broadjam's network.<cite>[280-280]</cite></statement>

<statement>- Hosting Subscriber may not use Broadjam's servers or Hosting Subscriber's Website as a source, intermediary, reply to address, or destination address for mail bombs, Internet packet flooding, packet corruption, denial of service, or any other abusive activities.<cite>[281-281]</cite></statement>

<statement>- Server hacking or other perpetration of security breaches is strictly prohibited.<cite>[282-282]</cite></statement>

<statement>- Use of Hosting Subscriber's Website as an anonymous gateway is prohibited.<cite>[283-283]</cite></statement>

<statement>- Hosting Subscriber agrees to comply with all applicable laws and regulations, including but not limited to the Can-Spam Act of 2003 and the Telephone Consumer Protection Act, as well as Broadjam's no-spam policies.<cite>[302-302]</cite></statement>

<statement>- Commercial advertising and/or bulk emails or faxes may only be sent to recipients who have already "opted-in" to receive messages from the sender specifically.<cite>[303-303]</cite></statement>

<statement>- If Broadjam determines Hosting Services are being used in association with spam, Broadjam will re-direct, suspend, or cancel such Hosting Service for a period of no less than 2 days.<cite>[306-306]</cite></statement>

<statement>So in summary, Broadjam's servers are restricted to legal uses, may not be overloaded, and may not be used for abusive or spam activities. Compliance with laws and Broadjam policies is required.<cite></cite></statement>
```

文档句索引范围: sid ∈ [1, 358], 共 358 句。
引用指标: R=1.0, P=1.0, F1=1.0
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=7378, completion_tokens=150
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
Based on the Broadjam agreement, some key restrictions on the use of Broadjam's servers include:
- GPT support_score=1; [[No]]   Need Citation: [[No]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 1
- Hosting Subscriber's Website will not be used in connection with any illegal activity.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[277-277]`
- au=1.6121624639708898e-06, eu=0.28571420907974243, au′=-0.0, eu′=0.038461536169052124
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.0]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 277 | 277 | 25.0 | Hosting Subscriber's Website will not be used in connection with any illegal activity. |
- GPT 引用1: span=[277, 277], relevant=1

### 陈述 2
- Hosting Subscriber is responsible for ensuring there is no excessive overloading on Broadjam's DNS or servers.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[278-278]`
- au=1.8090482853949652e-06, eu=0.27737218141555786, au′=-0.0, eu′=0.03696497529745102
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[26.05263328552246]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 278 | 278 | 26.05263328552246 | (b) Hosting Subscriber is responsible for ensuring that there is no excessive overloading on Broadjam's DNS or servers. |
- GPT 引用1: span=[278, 278], relevant=1

### 陈述 3
- Broadjam prohibits the use of software or scripts run on its servers that cause the server to load beyond a reasonable level.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[279-279]`
- au=1.6175589507838595e-06, eu=0.2867923676967621, au′=-0.0, eu′=0.03865716978907585
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.86842155456543]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 279 | 279 | 24.86842155456543 | Broadjam prohibits the use of software or scripts run on its servers that cause the server to load beyond a reasonable level, as determined by Broadjam. |
- GPT 引用1: span=[279, 279], relevant=1

### 陈述 4
- Hosting Subscriber agrees Broadjam reserves the right to remove Hosting Subscriber's Website temporarily or permanently if it threatens the stability of Broadjam's network.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[280-280]`
- au=1.5858721553740907e-06, eu=0.2804427146911621, au′=-0.0, eu′=0.03751233592629433
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.657896041870117]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 280 | 280 | 25.657896041870117 | Hosting Subscriber agrees that Broadjam reserves the right to remove Hosting Subscriber's Website temporarily or permanently from its hosting servers if Hosting Subscriber's Website threatens the stability of Broadjam's network. |
- GPT 引用1: span=[280, 280], relevant=1

### 陈述 5
- Hosting Subscriber may not use Broadjam's servers or Hosting Subscriber's Website as a source, intermediary, reply to address, or destination address for mail bombs, Internet packet flooding, packet corruption, denial of service, or any other abusive activities.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[281-281]`
- au=1.7844809008238371e-06, eu=0.2724013328552246, au′=-0.0, eu′=0.036087363958358765
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[26.710527420043945]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 281 | 281 | 26.710527420043945 | (c) Hosting Subscriber may not use Broadjam's servers or Hosting Subscriber's Website as a source, intermediary, reply to address, or destination address for mail bombs, Internet packet flooding, packet corruption, denial of service, or any other abusive activities. |
- GPT 引用1: span=[281, 281], relevant=1

### 陈述 6
- Server hacking or other perpetration of security breaches is strictly prohibited.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[282-282]`
- au=1.6121624639708898e-06, eu=0.28571420907974243, au′=-0.0, eu′=0.038461536169052124
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.0]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 282 | 282 | 25.0 | Server hacking or other perpetration of security breaches is strictly prohibited and Broadjam reserves the right to remove websites that contain information about hacking or links to such information. |
- GPT 引用1: span=[282, 282], relevant=1

### 陈述 7
- Use of Hosting Subscriber's Website as an anonymous gateway is prohibited.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[283-283]`
- au=1.6284955108858412e-06, eu=0.28897327184677124, au′=-0.0, eu′=0.03905446454882622
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.60526466369629]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 283 | 283 | 24.60526466369629 | Use of Hosting Subscriber's Website as an anonymous gateway is prohibited. |
- GPT 引用1: span=[283, 283], relevant=1

### 陈述 8
- Hosting Subscriber agrees to comply with all applicable laws and regulations, including but not limited to the Can-Spam Act of 2003 and the Telephone Consumer Protection Act, as well as Broadjam's no-spam policies.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[302-302]`
- au=1.9246485862822738e-06, eu=0.3003951609134674, au′=-0.0, eu′=0.04117009416222572
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.289474487304688]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 302 | 302 | 23.289474487304688 | In order to use our products and services, Hosting Subscriber shall abide by all applicable laws and regulations, including but not limited to the Can-Spam Act of 2003 and the Telephone Consumer Protection Act, as well as Broadjam's no-spam policies. |
- GPT 引用1: span=[302, 302], relevant=1

### 陈述 9
- Commercial advertising and/or bulk emails or faxes may only be sent to recipients who have already "opted-in" to receive messages from the sender specifically.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[303-303]`
- au=1.5807482895979774e-06, eu=0.2794116735458374, au′=-0.0, eu′=0.037328090518713
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.789474487304688]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 303 | 303 | 25.789474487304688 | Commercial advertising and/or bulk emails or faxes may only be sent to recipients who have already "opted-in" to receive messages from the sender specifically. |
- GPT 引用1: span=[303, 303], relevant=1

### 陈述 10
- If Broadjam determines Hosting Services are being used in association with spam, Broadjam will re-direct, suspend, or cancel such Hosting Service for a period of no less than 2 days.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[306-306]`
- au=1.5557670849375427e-06, eu=0.27436816692352295, au′=-0.0, eu′=0.03643336147069931
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[26.447368621826172]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 306 | 306 | 26.447368621826172 | (d) If Broadjam determines that Hosting Services are being used in association with spam, Broadjam will re-direct, suspend, or cancel such Hosting Service for a period of no less than 2 days. |
- GPT 引用1: span=[306, 306], relevant=1

### 陈述 11
So in summary, Broadjam's servers are restricted to legal uses, may not be overloaded, and may not be used for abusive or spam activities. Compliance with laws and Broadjam policies is required.
- GPT support_score=1; [[No]]   Need Citation: [[No]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

## multifieldqa_en_idx145 (idx=145, multifieldqa_en)
### 问题
How is the vacuum processing system configured in terms of the arrangement of the vacuum processing apparatus?

### 参考答案（数据集）
- ref1: Multiple vacuum processing apparatuses are arranged in parallel.

### 模型全文输出
```
<statement>Based on the patent, the vacuum processing system is configured with multiple vacuum processing apparatus arranged in parallel.<cite>[64-64]</cite></statement>

<statement>Each vacuum processing apparatus includes:<cite></cite></statement>

<statement>- An air loader with cassette stands and a carrier for transporting wafers<cite>[2-2]</cite></statement>

<statement>- A vacuum loader with etching treatment chambers and a vacuum carriage chamber connected through gate valves<cite>[2-2]</cite></statement>

<statement>- A lock device consisting of load and unload lock chambers with gate valves for connecting the carrier with the vacuum carriage chamber<cite>[2-2]</cite></statement>

<statement>The etching treatment chambers are arranged symmetrically about an axis line passing through the center of the vacuum carriage chamber. They are positioned at an acute angle on the opposite side of the vacuum carriage chamber from the lock device.<cite>[3-3][58-58][60-60]</cite></statement>

<statement>The multiple vacuum processing apparatus are arranged in parallel with their vacuum processing chambers aligned in a line. The air loader, vacuum loader, and lock device are arranged in a T-shape configuration around the vacuum carriage chamber.<cite>[64-64][62-62][59-59]</cite></statement>

<statement>So in summary, the vacuum processing system is configured with multiple vacuum processing apparatus arranged in parallel, with their vacuum processing chambers aligned and their air loader, vacuum loader, and lock device arranged in a T-shape around the vacuum carriage chamber.<cite></cite></statement>
```

文档句索引范围: sid ∈ [1, 124], 共 124 句。
引用指标: R=0.8125, P=0.8, F1=0.8062015503875969
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=9432, completion_tokens=145
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
Based on the patent, the vacuum processing system is configured with multiple vacuum processing apparatus arranged in parallel.
- GPT support_score=1; **Rating: [[Fully supported]]**   **

**跨度** `[64-64]`
- au=2.0151851458649617e-06, eu=0.3179915249347687, au′=-0.0, eu′=0.044548649340867996
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[21.447368621826172]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 64 | 64 | 21.447368621826172 | 【００１５】本発明は、並設した複数のカセット台およびカセット台から、あるいはカセット台へウエハを搬送するための搬送装置を備えた大気ローダと、ウエハを処理するための真空処理室およびこれにゲート弁を介して連接された真空搬送室を備えた真空ローダと、前記搬送装置と前記真空搬送室とを連接するためのゲート弁を備えたロードロック室およびアンロードロック室からなるロック装置とを含んで構成される真空処理装置が平行に複数台並設された真空処理システムにおいて、ウエハを処理するための真空処理室は、ＵＨＦ−ＥＣＲリアクタによって形成される真空処理室であり、該真空処理室は、真空搬送室およびロック装置の中央を通る軸線に対して対称にして、かつ真空搬送室を中心にしてロック装置の反対側のみに２ The present invention, a plurality of cassette tables and cassette stand juxtaposed, or the atmosphere loader having a conveying device for conveying the wafer to the cassette base, the vacuum processing chamber for processing the wafer and to a vacuum loader having a vacuum transfer chamber which is connected via a gate valve, the locking consisting of the conveying device and the load lock chamber and the unload lock chamber with gate valves for connecting the said vacuum transfer chamber apparatus and in the vacuum processing system vacuum processing apparatus is constituted with a plurality Tainami set in parallel include vacuum processing chamber for processing a wafer is vacuum processing chamber formed by the UHF-ECR reactor, vacuum processing chamber, and symmetrically with respect to the axis passing through the center of the vacuum transfer chamber and the locking device, and 2 only on the opposite side of the locking device around the vacuum transfer chamber 設けられ、かつ真空搬送室に対して前記２つの真空処理室の配置位置は鋭角をなしており、並設された複数の真空処理装置のすべての真空処理室に一直線上に配列される真空処理システムを提供する。 |
- GPT 引用1: span=[64, 64], relevant=1

### 陈述 1
Each vacuum processing apparatus includes:
- GPT support_score=1; [[No]]  Need Citation: [[No]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 2
- An air loader with cassette stands and a carrier for transporting wafers
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[2-2]`
- au=0.6823741793632507, eu=0.17882348597049713, au′=0.6823733448982239, eu′=0.04173530638217926
- top_logits 方差 (pvariance, 有限 logit): 0.004328221825744549
- top_logits 数值: `[23.026315689086914, 22.894737243652344]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 1 | 1 | 23.026315689086914 | PROBLEM TO BE SOLVED: To provide a vacuum processor which can suppress the rise of manufacture cost while coping with the enlargement of diameter of a sample, and besides is excellent in maintainability. |
| 2 | 2 | 2 | 22.894737243652344 | SOLUTION: This vacuum processor comprises an air loader 1 which is equipped with a plurality of juxtaposed cassette stands 2a and 2b and a carrier 13 for carrying a wafer 20 from or to the cassette stands, a vacuum loader 7 which is equipped with etching treatment chambers 11 and 11b for processing the wafer and a vacuum carriage chamber 16 connected to them through gate valves 15a and 15b, and a lock device 6 consisting of a load lock chamber 6b and an unload lock chamber 6b equipped with gate valves 12a, 12b, 12c, and 12d for connecting the said carrier 13 with the vacuum carriage chamber 16. |
- GPT 引用1: span=[2, 2], relevant=1

### 陈述 3
- A vacuum loader with etching treatment chambers and a vacuum carriage chamber connected through gate valves
- GPT support_score=1; **Rating: [[Fully supported]]**   **

**跨度** `[2-2]`
- au=1.5962523320922628e-06, eu=0.28252777457237244, au′=-0.0, eu′=0.03788633644580841
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.394737243652344]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 2 | 2 | 25.394737243652344 | SOLUTION: This vacuum processor comprises an air loader 1 which is equipped with a plurality of juxtaposed cassette stands 2a and 2b and a carrier 13 for carrying a wafer 20 from or to the cassette stands, a vacuum loader 7 which is equipped with etching treatment chambers 11 and 11b for processing the wafer and a vacuum carriage chamber 16 connected to them through gate valves 15a and 15b, and a lock device 6 consisting of a load lock chamber 6b and an unload lock chamber 6b equipped with gate valves 12a, 12b, 12c, and 12d for connecting the said carrier 13 with the vacuum carriage chamber 16. |
- GPT 引用1: span=[2, 2], relevant=1

### 陈述 4
- A lock device consisting of load and unload lock chambers with gate valves for connecting the carrier with the vacuum carriage chamber
- GPT support_score=1; **Rating: [[Fully supported]]**   **

**跨度** `[2-2]`
- au=1.5807482895979774e-06, eu=0.2794116735458374, au′=-0.0, eu′=0.037328090518713
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.789474487304688]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 2 | 2 | 25.789474487304688 | SOLUTION: This vacuum processor comprises an air loader 1 which is equipped with a plurality of juxtaposed cassette stands 2a and 2b and a carrier 13 for carrying a wafer 20 from or to the cassette stands, a vacuum loader 7 which is equipped with etching treatment chambers 11 and 11b for processing the wafer and a vacuum carriage chamber 16 connected to them through gate valves 15a and 15b, and a lock device 6 consisting of a load lock chamber 6b and an unload lock chamber 6b equipped with gate valves 12a, 12b, 12c, and 12d for connecting the said carrier 13 with the vacuum carriage chamber 16. |
- GPT 引用1: span=[2, 2], relevant=1

### 陈述 5
The etching treatment chambers are arranged symmetrically about an axis line passing through the center of the vacuum carriage chamber. They are positioned at an acute angle on the opposite side of the vacuum carriage chamber from the lock device.
- GPT support_score=1; **Rating: [[Fully supported]]**   **

**跨度** `[3-3]`
- au=1.861421651483397e-06, eu=0.28787869215011597, au′=-0.0, eu′=0.038854800164699554
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.73684310913086]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 3 | 3 | 24.73684310913086 | For the etching treatment chambers, two are provided only on the opposite side of the vacuum carriage chamber symmetrically about the axis line A passing the center, and besides the arrangement positions of the two etching treatment chambers are at an acute angle on the opposite side of the vacuum carriage chamber. |

**跨度** `[58-58]`
- au=1.9009210063813953e-06, eu=0.2957197427749634, au′=-0.0, eu′=0.04029691964387894
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.8157901763916]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 58 | 58 | 23.8157901763916 | referred UHF-ECR reactor) that a vacuum processing chamber, the vacuum processing chamber, degradable sidewall inner unit and an antenna are provided, the vacuum processing chamber, a vacuum transportable 室およびロック装置の中央を通る軸線に対して対称にして、かつ真空搬送室を中心にしてロック装置の反対側のみに２つ設けられ、かつ真空搬送室に対して前記２つの真空処理室の配置位置は鋭角をなしている真空処理装置を提供する。 |

**跨度** `[60-60]`
- au=1.5910399042695644e-06, eu=0.28148138523101807, au′=-0.0, eu′=0.0376984067261219
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.526317596435547]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 60 | 60 | 25.526317596435547 | ＣＲリアクタによって形成される真空処理室であり、該真空処理室は、真空搬送室およびロック装置の中央を通る軸線に対して対称にして、かつ真空搬送室を中心にしてロック装置の反対側のみに２つ設けられ、かつ真空搬送室に対して前記２つの真空処理室の配置位置は鋭角をなしており、ＵＨＦ−ＥＣＲのアンテナは、前記軸線に対して平行で、かつ前記真空搬送室とは反対側に開放される真空処理装置を提供する。 |
- GPT 引用1: span=[3, 3], relevant=1
- GPT 引用2: span=[58, 58], relevant=1
- GPT 引用3: span=[60, 60], relevant=1

### 陈述 6
The multiple vacuum processing apparatus are arranged in parallel with their vacuum processing chambers aligned in a line. The air loader, vacuum loader, and lock device are arranged in a T-shape configuration around the vacuum carriage chamber.
- GPT support_score=0.5; **Rating**: [[Partially supported]]

**跨度** `[64-64]`
- au=0.6817387342453003, eu=0.18536581099033356, au′=0.6817380785942078, eu′=0.04352806136012077
- top_logits 方差 (pvariance, 有限 logit): 0.1558167386256173
- top_logits 数值: `[22.36842155456543, 21.578948974609375]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 62 | 62 | 22.36842155456543 | かつ真空搬送室を中心にしてロック装置の反対側のみに２つ設けられ、かつ真空搬送室に対して前記２つの真空処理室の配置位置は鋭角をなしており、大気ローダ，真空ローダおよびロック装置はＴ字配置とされた真空処理方法を提供する。 |
| 2 | 64 | 64 | 21.578948974609375 | 【００１５】本発明は、並設した複数のカセット台およびカセット台から、あるいはカセット台へウエハを搬送するための搬送装置を備えた大気ローダと、ウエハを処理するための真空処理室およびこれにゲート弁を介して連接された真空搬送室を備えた真空ローダと、前記搬送装置と前記真空搬送室とを連接するためのゲート弁を備えたロードロック室およびアンロードロック室からなるロック装置とを含んで構成される真空処理装置が平行に複数台並設された真空処理システムにおいて、ウエハを処理するための真空処理室は、ＵＨＦ−ＥＣＲリアクタによって形成される真空処理室であり、該真空処理室は、真空搬送室およびロック装置の中央を通る軸線に対して対称にして、かつ真空搬送室を中心にしてロック装置の反対側のみに２ The present invention, a plurality of cassette tables and cassette stand juxtaposed, or the atmosphere loader having a conveying device for conveying the wafer to the cassette base, the vacuum processing chamber for processing the wafer and to a vacuum loader having a vacuum transfer chamber which is connected via a gate valve, the locking consisting of the conveying device and the load lock chamber and the unload lock chamber with gate valves for connecting the said vacuum transfer chamber apparatus and in the vacuum processing system vacuum processing apparatus is constituted with a plurality Tainami set in parallel include vacuum processing chamber for processing a wafer is vacuum processing chamber formed by the UHF-ECR reactor, vacuum processing chamber, and symmetrically with respect to the axis passing through the center of the vacuum transfer chamber and the locking device, and 2 only on the opposite side of the locking device around the vacuum transfer chamber 設けられ、かつ真空搬送室に対して前記２つの真空処理室の配置位置は鋭角をなしており、並設された複数の真空処理装置のすべての真空処理室に一直線上に配列される真空処理システムを提供する。 |

**跨度** `[62-62]`
- au=0.6810101270675659, eu=0.19487175345420837, au′=0.6810091137886047, eu′=0.04617253318428993
- top_logits 方差 (pvariance, 有限 logit): 0.1558167386256173
- top_logits 数值: `[21.052631378173828, 20.263158798217773]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 62 | 62 | 21.052631378173828 | かつ真空搬送室を中心にしてロック装置の反対側のみに２つ設けられ、かつ真空搬送室に対して前記２つの真空処理室の配置位置は鋭角をなしており、大気ローダ，真空ローダおよびロック装置はＴ字配置とされた真空処理方法を提供する。 |
| 2 | 65 | 65 | 20.263158798217773 | Provided, and positions of the two vacuum processing chamber to the vacuum transfer chamber is an acute angle, a vacuum process that is arranged in alignment with all of the vacuum processing chamber of a plurality of vacuum processing apparatus are arranged in parallel to provide a system. |

**跨度** `[59-59]`
- au=0.6809500455856323, eu=0.1890546828508377, au′=0.680949330329895, eu′=0.0445486456155777
- top_logits 方差 (pvariance, 有限 logit): 0.6232684603028247
- top_logits 数值: `[22.23684310913086, 20.657896041870117]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 63 | 63 | 22.23684310913086 | And around the vacuum transfer chamber opposite only two provided a locking device, and positions of the two vacuum processing chamber to the vacuum transfer chamber is an acute angle, atmospheric loader, vacuum loader and locking apparatus to provide a vacuum processing method which is a T-arrangement. |
| 2 | 59 | 59 | 20.657896041870117 | And symmetrically with respect to the axis passing through the center of the chamber and the locking device, and the opposite side only two provided for to lock device around the vacuum transfer chamber, and the two vacuum processing chamber to the vacuum transfer chamber position is to provide a vacuum processing apparatus which forms an acute angle. |
- GPT 引用1: span=[59, 59], relevant=0
- GPT 引用2: span=[62, 62], relevant=0
- GPT 引用3: span=[64, 64], relevant=1

### 陈述 7
So in summary, the vacuum processing system is configured with multiple vacuum processing apparatus arranged in parallel, with their vacuum processing chambers aligned and their air loader, vacuum loader, and lock device arranged in a T-shape around the vacuum carriage chamber.
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

## multifieldqa_en_idx146 (idx=146, multifieldqa_en)
### 问题
How many massive star-forming regions were studied?

### 参考答案（数据集）
- ref1: 21.

### 模型全文输出
```
<statement>Based on the information provided in the paper, 21 massive star-forming regions were studied.<cite></cite></statement>

<statement>Specifically, the paper states: "The 21 massive star-forming regions were observed with the Atacama Pathfinder Experiment (APEX) in the 875\,$\mu$m window in fall 2006."<cite>[11-11]</cite></statement>

<statement>So a total of 21 regions were observed with APEX.<cite></cite></statement>
```

文档句索引范围: sid ∈ [1, 88], 共 88 句。
引用指标: R=0.3333333333333333, P=1.0, F1=0.5
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=1244, completion_tokens=33
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
Based on the information provided in the paper, 21 massive star-forming regions were studied.
- GPT support_score=0; [[Yes]]   Need Citation: [[Yes]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 1
Specifically, the paper states: "The 21 massive star-forming regions were observed with the Atacama Pathfinder Experiment (APEX) in the 875\,$\mu$m window in fall 2006."
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[11-11]`
- au=1.8140860902349232e-06, eu=0.27838820219039917, au′=-0.0, eu′=0.037145648151636124
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.921052932739258]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 11 | 11 | 25.921052932739258 | \section{Observations} \label{obs}  The 21 massive star-forming regions were observed with the Atacama Pathfinder Experiment (APEX) in the 875\,$\mu$m window in fall 2006. |
- GPT 引用1: span=[11, 11], relevant=1

### 陈述 2
So a total of 21 regions were observed with APEX.
- GPT support_score=0; [[Yes]]   Need Citation: [[Yes]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

## multifieldqa_en_idx147 (idx=147, multifieldqa_en)
### 问题
What is the average magnetic moment per column in Ge$_{1-x}$Mn$_{x}$ films?

### 参考答案（数据集）
- ref1: 1425 $\mu_{B}$.

### 模型全文输出
```
<statement>Based on the analysis of the susceptibility data, the average magnetic moment per column in Ge$_{1-x}$Mn$_{x}$ films is estimated to be around 1425 Bohr magnetons (µB). This value is derived from fitting the inverse normalized in-plane and out-of-plane susceptibilities of the samples assuming isotropic nanostructures or cubic anisotropy.<cite>[240-241]</cite></statement>

<statement>The fitting leads to similar values of magnetic moment for in-plane and out-of-plane susceptibilities, indicating isotropic magnetic behavior of the nanocolumns. The average magnetic moment per column is calculated by dividing the magnetic moment by the volume of a single nanocolumn, assuming a cylindrical shape with diameter and height equal to the average diameter and height of the nanocolumns observed in TEM.<cite>[240-241][247-247]</cite></statement>

<statement>The value of 1425 µB per column is consistent with the susceptibility data and provides a reasonable estimate of the average magnetic moment in the Ge$_{1-x}$Mn$_{x}$ nanocolumns. However, it is important to note that the exact value may depend on the growth conditions, Mn concentration, and the precise size and shape of the nanocolumns.<cite></cite></statement>
```

文档句索引范围: sid ∈ [1, 326], 共 326 句。
引用指标: R=0.5, P=0.6666666666666666, F1=0.5714285714285715
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=2625, completion_tokens=45
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
Based on the analysis of the susceptibility data, the average magnetic moment per column in Ge$_{1-x}$Mn$_{x}$ films is estimated to be around 1425 Bohr magnetons (µB). This value is derived from fitting the inverse normalized in-plane and out-of-plane susceptibilities of the samples assuming isotropic nanostructures or cubic anisotropy.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[240-241]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> 11, the best fits lead to $M\approx$1250 $\mu_{B}$ and $\mu_{0}H_{eff}\approx$102 mT for
> in-plane susceptibility and $M\approx$1600 $\mu_{B}$ and $\mu_{0}H_{eff}\approx$98 mT
> for out-of-plane susceptibility. It gives an average magnetic moment of 1425 $\mu_{B}$
> per column and an effective interaction field of 100 mT.
- au=1.861421651483397e-06, eu=0.28787869215011597, au′=-0.0, eu′=0.038854800164699554
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.73684310913086]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 240 | 240 | 24.73684310913086 | 11, the best fits lead to $M\approx$1250 $\mu_{B}$ and $\mu_{0}H_{eff}\approx$102 mT for in-plane susceptibility and $M\approx$1600 $\mu_{B}$ and $\mu_{0}H_{eff}\approx$98 mT for out-of-plane susceptibility. |
- GPT 引用1: span=[240, 241], relevant=1

### 陈述 1
The fitting leads to similar values of magnetic moment for in-plane and out-of-plane susceptibilities, indicating isotropic magnetic behavior of the nanocolumns. The average magnetic moment per column is calculated by dividing the magnetic moment by the volume of a single nanocolumn, assuming a cylindrical shape with diameter and height equal to the average diameter and height of the nanocolumns observed in TEM.
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[240-241]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> 11, the best fits lead to $M\approx$1250 $\mu_{B}$ and $\mu_{0}H_{eff}\approx$102 mT for
> in-plane susceptibility and $M\approx$1600 $\mu_{B}$ and $\mu_{0}H_{eff}\approx$98 mT
> for out-of-plane susceptibility. It gives an average magnetic moment of 1425 $\mu_{B}$
> per column and an effective interaction field of 100 mT.
- au=1.729960786178708e-06, eu=0.3089430034160614, au′=-0.0, eu′=0.04279278963804245
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[22.36842155456543]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 240 | 240 | 22.36842155456543 | 11, the best fits lead to $M\approx$1250 $\mu_{B}$ and $\mu_{0}H_{eff}\approx$102 mT for in-plane susceptibility and $M\approx$1600 $\mu_{B}$ and $\mu_{0}H_{eff}\approx$98 mT for out-of-plane susceptibility. |

**跨度** `[247-247]`
- au=1.3684501647949219, eu=0.10674156248569489, au′=1.3684494495391846, eu′=0.045618243515491486
- top_logits 方差 (pvariance, 有限 logit): 0.086565189414614
- top_logits 数值: `[21.3157901763916, 21.052631378173828, 20.789474487304688, 20.526315689086914]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 246 | 246 | 21.3157901763916 | As for magnetic anisotropy, the nanostructures behave as isotropic magnetic systems or exhibit a cubic magnetic anisotropy. |
| 2 | 242 | 242 | 21.052631378173828 | Using this magnetic moment and its temperature dependence, magnetization curves could be fitted using a Langevin function and $M(H/T)$ curves superimpose for $T<$100 K. However, from the saturated magnetic moment of the columns and their density (35000 $\rm{\mu m}^{-2}$), we find almost 6000 $\mu_{B}$ per column. |
| 3 | 247 | 247 | 20.789474487304688 | First we can confirm that nanostructures are not amorphous otherwise shape anisotropy would dominate leading to out-of-plane anisotropy. |
| 4 | 243 | 243 | 20.526315689086914 | Therefore, for low growth temperatures, we need to assume that nanocolumns are actually made of almost four independent elongated magnetic nanostructures. |
- GPT 引用1: span=[240, 241], relevant=1
- GPT 引用2: span=[247, 247], relevant=0

### 陈述 2
The value of 1425 µB per column is consistent with the susceptibility data and provides a reasonable estimate of the average magnetic moment in the Ge$_{1-x}$Mn$_{x}$ nanocolumns. However, it is important to note that the exact value may depend on the growth conditions, Mn concentration, and the precise size and shape of the nanocolumns.
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

## multifieldqa_en_idx148 (idx=148, multifieldqa_en)
### 问题
Is there any evidence of heaven and hell?

### 参考答案（数据集）
- ref1: Unknown.

### 模型全文输出
```
<statement>Based on the conversation, there is no concrete evidence of heaven and hell. The possibility of their existence is considered a matter of faith and cannot be proven or disproven through logic or science. The conversation suggests that faith in heaven and hell is a personal choice and that each individual must decide for themselves whether to believe in them or not.<cite>[63-63][65-65][67-67]</cite></statement>
```

文档句索引范围: sid ∈ [1, 154], 共 154 句。
引用指标: R=0.5, P=0.6666666666666666, F1=0.5714285714285715
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=1266, completion_tokens=28
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
Based on the conversation, there is no concrete evidence of heaven and hell. The possibility of their existence is considered a matter of faith and cannot be proven or disproven through logic or science. The conversation suggests that faith in heaven and hell is a personal choice and that each individual must decide for themselves whether to believe in them or not.
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[63-63]`
- au=1.9246485862822738e-06, eu=0.3003951609134674, au′=-0.0, eu′=0.04117009416222572
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.289474487304688]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 63 | 63 | 23.289474487304688 | Nothing can give a solid proof of the existence of heaven and hell, yet, nothing can disprove it either. |

**跨度** `[65-65]`
- au=0.683157742023468, eu=0.16557730734348297, au′=0.6831569671630859, eu′=0.03817176818847656
- top_logits 方差 (pvariance, 有限 logit): 0.21208462622689694
- top_logits 数值: `[25.657896041870117, 24.73684310913086]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 65 | 65 | 25.657896041870117 | So the chance remain fifty-fifty. |
| 2 | 64 | 64 | 24.73684310913086 | Same goes to God. |

**跨度** `[67-67]`
- au=0.6831138134002686, eu=0.16851437091827393, au′=0.6831128597259521, eu′=0.03895437717437744
- top_logits 方差 (pvariance, 有限 logit): 0.004328221825744549
- top_logits 数值: `[24.73684310913086, 24.60526466369629]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 66 | 66 | 24.73684310913086 | You either believe in it for the sake of taking a lesser risk if you don’t believe in blind faith, or discard it all together. |
| 2 | 67 | 67 | 24.60526466369629 | Yet, the possibility is there until proved or disproved. |
- GPT 引用1: span=[63, 63], relevant=1
- GPT 引用2: span=[65, 65], relevant=0
- GPT 引用3: span=[67, 67], relevant=1

## multifieldqa_en_idx149 (idx=149, multifieldqa_en)
### 问题
When will BC leave Boston?

### 参考答案（数据集）
- ref1: August 25.

### 模型全文输出
```
<statement>Based on the information provided, there is no indication that BC plans to leave Boston in the near future. The main focus of the article is on the current state of BC football and the upcoming season, with a few mentions of past events and potential future developments like hosting the Olympics or moving to a new stadium. The article does not contain any information about BC considering a move from Boston or any plans to do so.<cite></cite></statement>

<statement>- The article mentions the potential for BC to host the Olympics and suggests that Fenway Sports Group could get involved, but does not discuss any plans for BC to leave Boston.<cite>[76-76]</cite></statement>

<statement>- There is a brief mention of the possibility of BC moving to a new stadium, but this is just discussed as a hypothetical and not presented as a definite plan.<cite>[74-74]</cite></statement>

<statement>- The article focuses on the current state of BC football and the upcoming season, with no mention of BC considering a move from Boston.<cite></cite></statement>

<statement>- The article does not contain any information about BC considering a move from Boston or any plans to do so.<cite></cite></statement>

<statement>So in summary, the information provided in the article does not indicate that BC plans to leave Boston in the near future. The article is focused on the current state of BC football and the upcoming season, with a few mentions of past events and potential future developments that do not involve a move from Boston.<cite></cite></statement>
```

文档句索引范围: sid ∈ [1, 700], 共 700 句。
引用指标: R=0.16666666666666666, P=0.0, F1=0.0
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=3174, completion_tokens=70
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
Based on the information provided, there is no indication that BC plans to leave Boston in the near future. The main focus of the article is on the current state of BC football and the upcoming season, with a few mentions of past events and potential future developments like hosting the Olympics or moving to a new stadium. The article does not contain any information about BC considering a move from Boston or any plans to do so.
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 1
- The article mentions the potential for BC to host the Olympics and suggests that Fenway Sports Group could get involved, but does not discuss any plans for BC to leave Boston.
- GPT support_score=0; Rating: [[No support]]

**跨度** `[76-76]`
- au=1.650961507948523e-06, eu=0.2934361696243286, au′=-0.0, eu′=0.03987407684326172
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.078948974609375]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 76 | 76 | 24.078948974609375 | I would still love to see Boston host the games and wish our partner Fenway Sports Group would get behind it. |
- GPT 引用1: span=[76, 76], relevant=0

### 陈述 2
- There is a brief mention of the possibility of BC moving to a new stadium, but this is just discussed as a hypothetical and not presented as a definite plan.
- GPT support_score=0; Rating: [[No support]]

**跨度** `[74-74]`
- au=1.3679821491241455, eu=0.10872673988342285, au′=1.3679819107055664, eu′=0.04652585834264755
- top_logits 方差 (pvariance, 有限 logit): 0.16771988195068843
- top_logits 数值: `[20.921052932739258, 20.789474487304688, 20.394737243652344, 19.86842155456543]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 73 | 73 | 20.921052932739258 | Boston would probably have similar suitors depending on the location. |
| 2 | 66 | 66 | 20.789474487304688 | The big obstacle would always be the main stadium for Track & Field and the opening and closing ceremonies. |
| 3 | 74 | 74 | 20.394737243652344 | I would guess that UMasss Boston would want a small stadium or maybe the Revolution would want to convert it to a soccer only facility. |
| 4 | 65 | 65 | 19.86842155456543 | My point then remains that the same: Boston is ideal because the local universities could already supply many of the venues. |
- GPT 引用1: span=[74, 74], relevant=0

### 陈述 3
- The article focuses on the current state of BC football and the upcoming season, with no mention of BC considering a move from Boston.
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 4
- The article does not contain any information about BC considering a move from Boston or any plans to do so.
- GPT support_score=0; [[Yes]]   Need Citation: [[Yes]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 5
So in summary, the information provided in the article does not indicate that BC plans to leave Boston in the near future. The article is focused on the current state of BC football and the upcoming season, with a few mentions of past events and potential future developments that do not involve a move from Boston.
- GPT support_score=1; [[No]]   Need Citation: [[No]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

