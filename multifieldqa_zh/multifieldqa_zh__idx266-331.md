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



> 本文件为 `two_samples_cite_span_report.md` 的 **multifieldqa_zh** 子集（共 200 条样本），其余数据集见同目录下其它 md。


> 本文件为 **multifieldqa_zh** 子集第 **2/3** 部分，样本 `idx` ∈ [266, 331]（共 66 条）。 同数据集另见本目录下其余分片（文件名形如：数据集名__idx起止范围.md）。

## multifieldqa_zh_idx266 (idx=266, multifieldqa_zh)
### 问题
古人是如何根据天象预测人事的？

### 参考答案（数据集）
- ref1: 古人常用天象来预测人事，把天象与人间的吉凶祸福联系起来，如：彗星的出现被认为是预兆兵灾的凶象；月亮的盈亏圆缺被比附于人事之聚散离合。古代占星家为了用天象变化来占卜人间的吉凶祸福，还将天上星空区域与地上的州国互相对应，称作“分野”。

### 模型全文输出
```
<statement>根据文中内容，古人是通过观测天象来预测人事的。主要有以下几点：<cite></cite></statement>

<statement>1. 彗星被认为是预兆兵灾的凶象。月亮是诸侯的象征，彗星袭月表示诸侯有灾。<cite>[5-7]</cite></statement>

<statement>2. 月亮的盈亏圆缺被比附于人事之聚散离合。望月抒怀，借月寄情。<cite>[8-9]</cite></statement>

<statement>3. 占星家将天上星空区域与地上的州国互相对应，称作“分野”。天地联为一体，天上某一区域的天象变化，就直接预示着地上相应区域的吉凶祸福了。<cite>[10-11]</cite></statement>

<statement>4. 通过观测天象，人们可以预测天气、季节、物候等，也可以预测人事的吉凶祸福。<cite>[2-2][4-4][8-8]</cite></statement>

<statement>所以古人通过观测天象，预测人事的方法是通过天象的变化来反映地上的吉凶祸福，天象是地事的反映。<cite></cite></statement>
```

文档句索引范围: sid ∈ [1, 95], 共 95 句。
引用指标: R=0.8333333333333334, P=1.0, F1=0.9090909090909091
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=4534, completion_tokens=87
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据文中内容，古人是通过观测天象来预测人事的。主要有以下几点：
- GPT support_score=1; [[No]]   Need Citation: [[No]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 1
1. 彗星被认为是预兆兵灾的凶象。月亮是诸侯的象征，彗星袭月表示诸侯有灾。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[5-7]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> 如彗星是绕太阳运行的天体，拖有长长的尾巴，它的出现就被古人认为是预兆兵灾的凶象。
> 《唐雎不辱使命》中说“夫专诸之刺王僚也，彗星袭月”，因为月亮是诸侯的象征，所以这种天象是诸侯有灾的凶兆。 僚是诸侯（吴王），他将被刺，所以上天才显示出这样的天象。
- au=1.6068129298218992e-06, eu=0.28464409708976746, au′=-0.0, eu′=0.038267869502305984
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.131580352783203]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 5 | 5 | 25.131580352783203 | 如彗星是绕太阳运行的天体，拖有长长的尾巴，它的出现就被古人认为是预兆兵灾的凶象。 |
- GPT 引用1: span=[5, 7], relevant=1

### 陈述 2
2. 月亮的盈亏圆缺被比附于人事之聚散离合。望月抒怀，借月寄情。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[8-9]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> 通过对月亮的观测，人们又将月亮的盈亏圆缺比附于人事之聚散离合。
> 无数文人墨客望月抒怀，借月寄情，请看“可怜楼上月徘徊，应照离人妆镜台”，“露从今夜白，月是故乡明”，“海上生明月，天涯共此时”，这些诗句里包含了多少人间的悲欢离合之情啊。
- au=1.6396279534092173e-06, eu=0.2911876440048218, au′=-0.0, eu′=0.039460018277168274
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.342105865478516]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 8 | 8 | 24.342105865478516 | 通过对月亮的观测，人们又将月亮的盈亏圆缺比附于人事之聚散离合。 |
- GPT 引用1: span=[8, 9], relevant=1

### 陈述 3
3. 占星家将天上星空区域与地上的州国互相对应，称作“分野”。天地联为一体，天上某一区域的天象变化，就直接预示着地上相应区域的吉凶祸福了。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[10-11]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> 古代占星家为了用天象变化来占卜人间的吉凶祸福，还将天上星空区域与地上的州国互相对应，称作“分野”。
> 这样，天地联为一体，天上某一区域的某些天象变化，就直接预示着地上相应区域的吉凶祸福了。
- au=1.6802155187178869e-06, eu=0.2992125153541565, au′=-0.0, eu′=0.04094827175140381
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.421052932739258]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 10 | 10 | 23.421052932739258 | 古代占星家为了用天象变化来占卜人间的吉凶祸福，还将天上星空区域与地上的州国互相对应，称作“分野”。 |
- GPT 引用1: span=[10, 11], relevant=1

### 陈述 4
4. 通过观测天象，人们可以预测天气、季节、物候等，也可以预测人事的吉凶祸福。
- GPT support_score=1; **Rating**: [[Fully supported]]   **

**跨度** `[2-2]`
- au=0.6820250749588013, eu=0.18313249945640564, au′=0.6820241808891296, eu′=0.04291360080242157
- top_logits 方差 (pvariance, 有限 logit): 0.038953996431700944
- top_logits 数值: `[22.5, 22.10526466369629]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 4 | 4 | 22.5 | 古人常用天象来预测人事，把天象与人间的吉凶祸福联系起来。 |
| 2 | 2 | 2 | 22.10526466369629 | 古人根据太阳一年内的位置变化以及由此引起的地面气候的演变次序，把一年又分成24段，分列在十二个月中，以反映四季、气温、物候等情况。 |

**跨度** `[4-4]`
- au=1.650961507948523e-06, eu=0.2934361696243286, au′=-0.0, eu′=0.03987407684326172
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.078948974609375]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 4 | 4 | 24.078948974609375 | 古人常用天象来预测人事，把天象与人间的吉凶祸福联系起来。 |

**跨度** `[8-8]`
- au=1.0835933685302734, eu=0.13080891966819763, au′=1.0835925340652466, eu′=0.043198175728321075
- top_logits 方差 (pvariance, 有限 logit): 0.11926778392403826
- top_logits 数值: `[22.631580352783203, 21.973684310913086, 21.842105865478516]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 5 | 5 | 22.631580352783203 | 如彗星是绕太阳运行的天体，拖有长长的尾巴，它的出现就被古人认为是预兆兵灾的凶象。 |
| 2 | 8 | 8 | 21.973684310913086 | 通过对月亮的观测，人们又将月亮的盈亏圆缺比附于人事之聚散离合。 |
| 3 | 10 | 10 | 21.842105865478516 | 古代占星家为了用天象变化来占卜人间的吉凶祸福，还将天上星空区域与地上的州国互相对应，称作“分野”。 |
- GPT 引用1: span=[2, 2], relevant=1
- GPT 引用2: span=[4, 4], relevant=1
- GPT 引用3: span=[8, 8], relevant=1

### 陈述 5
所以古人通过观测天象，预测人事的方法是通过天象的变化来反映地上的吉凶祸福，天象是地事的反映。
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

## multifieldqa_zh_idx267 (idx=267, multifieldqa_zh)
### 问题
如何购买耗材或清洁材料？

### 参考答案（数据集）
- ref1: 访问网站：https://imagescanner.fujitsu.com/r/ctl/21。

### 模型全文输出
```
<statement>根据操作指南中的内容，可以购买耗材或清洁材料的方式如下：<cite></cite></statement>

<statement>1. 访问以下网页，查看联系列表，联系富士通扫描仪经销商或经授权的富士通扫描仪服务提供商：<cite>[269-269]</cite></statement>

<statement>https://imagescanner.fujitsu.com/r/ctl/21<cite>[269-269]</cite></statement>

<statement>2. 在操作指南中也给出了购买耗材或清洁材料的联系方式，访问以下网页，查看联系方式信息：<cite>[269-269]</cite></statement>

<statement>https://imagescanner.fujitsu.com/r/ctl/21<cite>[269-269]</cite></statement>

<statement>3. 也可以联系您身边的富士通办事处，查看联系方式信息：<cite>[269-269]</cite></statement>

<statement>https://www.fujitsu.com/global/about/resources/shop/computing/peripheral/scanners/index.html<cite>[269-269]</cite></statement>

<statement>4. 操作指南中也给出了购买耗材或清洁材料的联系方式，查看操作指南中的"购买耗材或清洁材料的联系方式"部分。<cite>[269-269]</cite></statement>

<statement>所以可以通过以上几种方式来购买耗材或清洁材料。<cite></cite></statement>
```

文档句索引范围: sid ∈ [1, 272], 共 272 句。
引用指标: R=0.7222222222222222, P=1.0, F1=0.8387096774193548
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=7347, completion_tokens=116
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据操作指南中的内容，可以购买耗材或清洁材料的方式如下：
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 1
1. 访问以下网页，查看联系列表，联系富士通扫描仪经销商或经授权的富士通扫描仪服务提供商：
- GPT support_score=0.5; **Rating**: [[Partially supported]]

**跨度** `[269-269]`
- au=1.9430344764259644e-06, eu=0.3039999008178711, au′=-0.0, eu′=0.041850216686725616
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[22.894737243652344]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 269 | 269 | 22.894737243652344 | ） 20 联系方式 请参阅以下网页中的联系列表： https://imagescanner.fujitsu.com/r/ctl/ fi Series 图像扫描仪 要咨询扫描仪的其他相关信息，请参阅以下网页： https://www.fujitsu.com/global/support/products/ computing/peripheral/scanners/fi/ 如上述网页中的信息无法帮助您解决问题，请参 阅以下网站中您身边富士通办事处的联系方式信 息： https://www.fujitsu.com/global/about/resources/shop/ computing/peripheral/scanners/index.html 购买耗材或清洁材料的联系方式 https://imagescanner.fujitsu.com/r/ctl/21 fi-760PRB 背书器 操作指南 P3PC-6112-04ZH 发行日期 : 2022 年 2 月 发行机构 : PFU Limited 本指南的内容如有变动，恕不另行通知。 |
- GPT 引用1: span=[269, 269], relevant=1

### 陈述 2
https://imagescanner.fujitsu.com/r/ctl/21
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[269-269]`
- au=1.9430344764259644e-06, eu=0.3039999008178711, au′=-0.0, eu′=0.041850216686725616
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[22.894737243652344]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 269 | 269 | 22.894737243652344 | ） 20 联系方式 请参阅以下网页中的联系列表： https://imagescanner.fujitsu.com/r/ctl/ fi Series 图像扫描仪 要咨询扫描仪的其他相关信息，请参阅以下网页： https://www.fujitsu.com/global/support/products/ computing/peripheral/scanners/fi/ 如上述网页中的信息无法帮助您解决问题，请参 阅以下网站中您身边富士通办事处的联系方式信 息： https://www.fujitsu.com/global/about/resources/shop/ computing/peripheral/scanners/index.html 购买耗材或清洁材料的联系方式 https://imagescanner.fujitsu.com/r/ctl/21 fi-760PRB 背书器 操作指南 P3PC-6112-04ZH 发行日期 : 2022 年 2 月 发行机构 : PFU Limited 本指南的内容如有变动，恕不另行通知。 |
- GPT 引用1: span=[269, 269], relevant=1

### 陈述 3
2. 在操作指南中也给出了购买耗材或清洁材料的联系方式，访问以下网页，查看联系方式信息：
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[269-269]`
- au=1.6567056491112453e-06, eu=0.2945735454559326, au′=-0.0, eu′=0.04008438438177109
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.947368621826172]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 269 | 269 | 23.947368621826172 | ） 20 联系方式 请参阅以下网页中的联系列表： https://imagescanner.fujitsu.com/r/ctl/ fi Series 图像扫描仪 要咨询扫描仪的其他相关信息，请参阅以下网页： https://www.fujitsu.com/global/support/products/ computing/peripheral/scanners/fi/ 如上述网页中的信息无法帮助您解决问题，请参 阅以下网站中您身边富士通办事处的联系方式信 息： https://www.fujitsu.com/global/about/resources/shop/ computing/peripheral/scanners/index.html 购买耗材或清洁材料的联系方式 https://imagescanner.fujitsu.com/r/ctl/21 fi-760PRB 背书器 操作指南 P3PC-6112-04ZH 发行日期 : 2022 年 2 月 发行机构 : PFU Limited 本指南的内容如有变动，恕不另行通知。 |
- GPT 引用1: span=[269, 269], relevant=1

### 陈述 4
https://imagescanner.fujitsu.com/r/ctl/21
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[269-269]`
- au=1.6068129298218992e-06, eu=0.28464409708976746, au′=-0.0, eu′=0.038267869502305984
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.131580352783203]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 269 | 269 | 25.131580352783203 | ） 20 联系方式 请参阅以下网页中的联系列表： https://imagescanner.fujitsu.com/r/ctl/ fi Series 图像扫描仪 要咨询扫描仪的其他相关信息，请参阅以下网页： https://www.fujitsu.com/global/support/products/ computing/peripheral/scanners/fi/ 如上述网页中的信息无法帮助您解决问题，请参 阅以下网站中您身边富士通办事处的联系方式信 息： https://www.fujitsu.com/global/about/resources/shop/ computing/peripheral/scanners/index.html 购买耗材或清洁材料的联系方式 https://imagescanner.fujitsu.com/r/ctl/21 fi-760PRB 背书器 操作指南 P3PC-6112-04ZH 发行日期 : 2022 年 2 月 发行机构 : PFU Limited 本指南的内容如有变动，恕不另行通知。 |
- GPT 引用1: span=[269, 269], relevant=1

### 陈述 5
3. 也可以联系您身边的富士通办事处，查看联系方式信息：
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[269-269]`
- au=1.8040525446849642e-06, eu=0.2763635516166687, au′=-0.0, eu′=0.03678605705499649
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[26.18421173095703]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 269 | 269 | 26.18421173095703 | ） 20 联系方式 请参阅以下网页中的联系列表： https://imagescanner.fujitsu.com/r/ctl/ fi Series 图像扫描仪 要咨询扫描仪的其他相关信息，请参阅以下网页： https://www.fujitsu.com/global/support/products/ computing/peripheral/scanners/fi/ 如上述网页中的信息无法帮助您解决问题，请参 阅以下网站中您身边富士通办事处的联系方式信 息： https://www.fujitsu.com/global/about/resources/shop/ computing/peripheral/scanners/index.html 购买耗材或清洁材料的联系方式 https://imagescanner.fujitsu.com/r/ctl/21 fi-760PRB 背书器 操作指南 P3PC-6112-04ZH 发行日期 : 2022 年 2 月 发行机构 : PFU Limited 本指南的内容如有变动，恕不另行通知。 |
- GPT 引用1: span=[269, 269], relevant=1

### 陈述 6
https://www.fujitsu.com/global/about/resources/shop/computing/peripheral/scanners/index.html
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[269-269]`
- au=1.5318007626774488e-06, eu=0.26950347423553467, au′=-0.0, eu′=0.03558051958680153
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[27.10526466369629]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 269 | 269 | 27.10526466369629 | ） 20 联系方式 请参阅以下网页中的联系列表： https://imagescanner.fujitsu.com/r/ctl/ fi Series 图像扫描仪 要咨询扫描仪的其他相关信息，请参阅以下网页： https://www.fujitsu.com/global/support/products/ computing/peripheral/scanners/fi/ 如上述网页中的信息无法帮助您解决问题，请参 阅以下网站中您身边富士通办事处的联系方式信 息： https://www.fujitsu.com/global/about/resources/shop/ computing/peripheral/scanners/index.html 购买耗材或清洁材料的联系方式 https://imagescanner.fujitsu.com/r/ctl/21 fi-760PRB 背书器 操作指南 P3PC-6112-04ZH 发行日期 : 2022 年 2 月 发行机构 : PFU Limited 本指南的内容如有变动，恕不另行通知。 |
- GPT 引用1: span=[269, 269], relevant=1

### 陈述 7
4. 操作指南中也给出了购买耗材或清洁材料的联系方式，查看操作指南中的"购买耗材或清洁材料的联系方式"部分。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[269-269]`
- au=1.698429286989267e-06, eu=0.30278876423835754, au′=-0.0, eu′=0.041621025651693344
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.026315689086914]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 269 | 269 | 23.026315689086914 | ） 20 联系方式 请参阅以下网页中的联系列表： https://imagescanner.fujitsu.com/r/ctl/ fi Series 图像扫描仪 要咨询扫描仪的其他相关信息，请参阅以下网页： https://www.fujitsu.com/global/support/products/ computing/peripheral/scanners/fi/ 如上述网页中的信息无法帮助您解决问题，请参 阅以下网站中您身边富士通办事处的联系方式信 息： https://www.fujitsu.com/global/about/resources/shop/ computing/peripheral/scanners/index.html 购买耗材或清洁材料的联系方式 https://imagescanner.fujitsu.com/r/ctl/21 fi-760PRB 背书器 操作指南 P3PC-6112-04ZH 发行日期 : 2022 年 2 月 发行机构 : PFU Limited 本指南的内容如有变动，恕不另行通知。 |
- GPT 引用1: span=[269, 269], relevant=1

### 陈述 8
所以可以通过以上几种方式来购买耗材或清洁材料。
- GPT support_score=1; [[No]]   Need Citation: [[No]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

## multifieldqa_zh_idx268 (idx=268, multifieldqa_zh)
### 问题
2013年新增绿色食品基地的面积有多大？

### 参考答案（数据集）
- ref1: 2013年新增绿色食品基地的面积达到30万亩。

### 模型全文输出
```
<statement>根据报告内容，2013年新增绿色食品基地的面积是30万亩。<cite></cite></statement>

<statement>报告原文：<cite></cite></statement>

<statement>“新新增绿色食品基地30万亩，播种面积达到1280万亩。”<cite>[32-32]</cite></statement>

<statement>所以，2013年新增绿色食品基地的面积是30万亩。<cite></cite></statement>
```

文档句索引范围: sid ∈ [1, 301], 共 301 句。
引用指标: R=0.5, P=1.0, F1=0.6666666666666666
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=1349, completion_tokens=43
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据报告内容，2013年新增绿色食品基地的面积是30万亩。
- GPT support_score=0; [[Yes]]   Need Citation: [[Yes]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 1
报告原文：
- GPT support_score=1; [[No]]   Need Citation: [[No]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 2
“新新增绿色食品基地30万亩，播种面积达到1280万亩。”
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[32-32]`
- au=0.6820714473724365, eu=0.18095232546329498, au′=0.6820706725120544, eu′=0.0423162542283535
- top_logits 方差 (pvariance, 有限 logit): 0.1558167386256173
- top_logits 数值: `[23.026315689086914, 22.23684310913086]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 32 | 32 | 23.026315689086914 | 新增绿色食品基地30万亩，播种面积达到1280万亩。 |
| 2 | 31 | 31 | 22.23684310913086 | 粮豆薯总产270亿斤，实现“六连增”，荣获全国粮食生产先进市，五个县（市）荣获全国粮食生产先进县。 |
- GPT 引用1: span=[32, 32], relevant=1

### 陈述 3
所以，2013年新增绿色食品基地的面积是30万亩。
- GPT support_score=0; [[Yes]]   Need Citation: [[Yes]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

## multifieldqa_zh_idx269 (idx=269, multifieldqa_zh)
### 问题
赵长山是否同意按照劳务合同内容履行？

### 参考答案（数据集）
- ref1: 是的。

### 模型全文输出
```
<statement>根据判决书内容，赵长山同意按照劳务合同内容履行。<cite></cite></statement>

<statement>判决书中提到：赵长山不识字，合同是将其他专业户家里做成的文本复印完之后签订的，只是因为赵长山不识字而没有签字而已。合同内容是当场宣读给他的，我认为他无法拿回保证金。<cite>[12-12]</cite></statement>

<statement>这表明赵长山同意按照合同内容履行劳务，但由于他不识字没有签字。所以判决书认为他无法拿回保证金。<cite></cite></statement>
```

文档句索引范围: sid ∈ [1, 76], 共 76 句。
引用指标: R=0.16666666666666666, P=1.0, F1=0.2857142857142857
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=1387, completion_tokens=34
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据判决书内容，赵长山同意按照劳务合同内容履行。
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 1
判决书中提到：赵长山不识字，合同是将其他专业户家里做成的文本复印完之后签订的，只是因为赵长山不识字而没有签字而已。合同内容是当场宣读给他的，我认为他无法拿回保证金。
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[12-12]`
- au=1.6396279534092173e-06, eu=0.2911876440048218, au′=-0.0, eu′=0.039460018277168274
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.342105865478516]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 12 | 12 | 24.342105865478516 | 合同是将其他专业户家里做成的文本复印完之后签订的，只是因为赵长山不识字而没有签字而已。 |
- GPT 引用1: span=[12, 12], relevant=1

### 陈述 2
这表明赵长山同意按照合同内容履行劳务，但由于他不识字没有签字。所以判决书认为他无法拿回保证金。
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

## multifieldqa_zh_idx270 (idx=270, multifieldqa_zh)
### 问题
“营改增”改革取得了哪些具体成果？

### 参考答案（数据集）
- ref1: 全年为企业减少地方税费180亿元。

### 模型全文输出
```
<statement>根据报告内容，实施“营改增”改革取得的主要成果有：<cite></cite></statement>

<statement>1. 落实小微企业税收优惠，清理规范涉企收费和基金，全年为企业减少地方税费180亿元。<cite>[21-21]</cite></statement>

<statement>2. 规上工业增加值增长7.5%左右。<cite>[17-17]</cite></statement>

<statement>3. 服务业增加值增长8%左右，占生产总值比重达55%以上。<cite>[18-18]</cite></statement>

<statement>4. 粮食产量296.9万吨，肉蛋奶产量100万吨，蔬菜产量830万吨，水产品产量121万吨。<cite>[19-19]</cite></statement>

<statement>5. 新登记市场主体25.1万户，总量达到118.7万户。<cite>[26-26]</cite></statement>

<statement>6. 战略性新兴产业和高技术产业增加值分别增长10%、11%左右。<cite>[29-29]</cite></statement>

<statement>7. 金融业增加值增长8%左右，旅游消费增长15%左右。<cite>[32-32]</cite></statement>

<statement>8. 国家现代农业示范区建设水平在副省级以上城市中居第二位。<cite>[33-33]</cite></statement>

<statement>9. 新引进各类人才13.1万人，新增住青院士5人、聘任院士3人、千人计划专家16人、泰山系列人才工程专家66人，人才总量180万人。<cite>[45-45]</cite></statement>

<statement>10. 国际贸易总额达到1150亿美元，增长14%左右，其中货物进出口总额5007亿元，增长15%左右；服务进出口总额140亿美元，增长16%左右。<cite>[58-59]</cite></statement>

<statement>11. 实际利用外资77.3亿美元，增长10%左右，总量占全省45%左右。<cite>[56-56]</cite></statement>

<statement>12. 实际到位内资1887.2亿元，增长13.3%左右。<cite>[57-57]</cite></statement>

<statement>13. 国际贸易总额达到1150亿美元，增长14%左右，其中货物进出口总额5007亿元，增长15%左右；服务进出口总额140亿美元，增长16%左右。<cite>[58-59]</cite></statement>

<statement>14. 位列全国服务外包示范城市前10强。<cite>[60-60]</cite></statement>

<statement>15. 新开通4条空中洲际直达航线。<cite>[61-61]</cite></statement>

<statement>16. 港口货物和集装箱吞吐量分别达到5.1亿吨、1830万标箱，青岛港全自动化码头投入使用。<cite>[62-62]</cite></statement>

<statement>17. 深度融入“一带一路”建设，对沿线国家投资额、进出口额分别占全市的48.3%、49%左右。<cite>[63-63]</cite></statement>

<statement>18. 新增1个国际友好合作关系城市、17个国际经济合作伙伴城市。<cite>[64-64]</cite></statement>

<statement>19. 建成7个境外青岛工商中心。<cite>[65-65]</cite></statement>

<statement>20. 成功举办亚欧数字互联互通高级别论坛、亚洲媒体峰会、金砖国家协调人第二次会议、国际教育信息化大会、国际标准化论坛等重要国际活动。<cite>[66-66]</cite></statement>
```

文档句索引范围: sid ∈ [1, 309], 共 309 句。
引用指标: R=1.0, P=0.7, F1=0.8235294117647058
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=12861, completion_tokens=282
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据报告内容，实施“营改增”改革取得的主要成果有：
- GPT support_score=1; [[No]]  Need Citation: [[No]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 1
1. 落实小微企业税收优惠，清理规范涉企收费和基金，全年为企业减少地方税费180亿元。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[21-21]`
- au=1.7108611700678011e-06, eu=0.3052207827568054, au′=-0.0, eu′=0.04208194464445114
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[22.763158798217773]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 21 | 21 | 22.763158798217773 | 推进“营改增”改革，落实小微企业税收优惠，清理规范涉企收费和基金，全年为企业减少地方税费180亿元。 |
- GPT 引用1: span=[21, 21], relevant=1

### 陈述 2
2. 规上工业增加值增长7.5%左右。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[17-17]`
- au=1.6121624639708898e-06, eu=0.28571420907974243, au′=-0.0, eu′=0.038461536169052124
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.0]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 17 | 17 | 25.0 | 规上工业增加值增长7.5%左右。 |
- GPT 引用1: span=[17, 17], relevant=1

### 陈述 3
3. 服务业增加值增长8%左右，占生产总值比重达55%以上。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[18-18]`
- au=1.8040525446849642e-06, eu=0.2763635516166687, au′=-0.0, eu′=0.03678605705499649
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[26.18421173095703]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 18 | 18 | 26.18421173095703 | 服务业增加值增长8%左右，占生产总值比重达55%以上。 |
- GPT 引用1: span=[18, 18], relevant=1

### 陈述 4
4. 粮食产量296.9万吨，肉蛋奶产量100万吨，蔬菜产量830万吨，水产品产量121万吨。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[19-19]`
- au=1.8140860902349232e-06, eu=0.27838820219039917, au′=-0.0, eu′=0.037145648151636124
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.921052932739258]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 19 | 19 | 25.921052932739258 | 粮食产量296.9万吨，肉蛋奶产量100万吨，蔬菜产量830万吨，水产品产量121万吨。 |
- GPT 引用1: span=[19, 19], relevant=0

### 陈述 5
5. 新登记市场主体25.1万户，总量达到118.7万户。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[26-26]`
- au=1.9246485862822738e-06, eu=0.3003951609134674, au′=-0.0, eu′=0.04117009416222572
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.289474487304688]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 26 | 26 | 23.289474487304688 | 出台培育市场主体发展民营经济的政策措施，新登记市场主体25.1万户，总量达到118.7万户。 |
- GPT 引用1: span=[26, 26], relevant=1

### 陈述 6
6. 战略性新兴产业和高技术产业增加值分别增长10%、11%左右。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[29-29]`
- au=1.6396279534092173e-06, eu=0.2911876440048218, au′=-0.0, eu′=0.039460018277168274
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.342105865478516]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 29 | 29 | 24.342105865478516 | 战略性新兴产业和高技术产业增加值分别增长10%、11%左右。 |
- GPT 引用1: span=[29, 29], relevant=1

### 陈述 7
7. 金融业增加值增长8%左右，旅游消费增长15%左右。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[32-32]`
- au=1.650961507948523e-06, eu=0.2934361696243286, au′=-0.0, eu′=0.03987407684326172
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.078948974609375]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 32 | 32 | 24.078948974609375 | 金融业增加值增长8%左右，旅游消费增长15%左右。 |
- GPT 引用1: span=[32, 32], relevant=1

### 陈述 8
8. 国家现代农业示范区建设水平在副省级以上城市中居第二位。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[33-33]`
- au=1.6068129298218992e-06, eu=0.28464409708976746, au′=-0.0, eu′=0.038267869502305984
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.131580352783203]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 33 | 33 | 25.131580352783203 | 国家现代农业示范区建设水平在副省级以上城市中居第二位。 |
- GPT 引用1: span=[33, 33], relevant=1

### 陈述 9
9. 新引进各类人才13.1万人，新增住青院士5人、聘任院士3人、千人计划专家16人、泰山系列人才工程专家66人，人才总量180万人。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[45-45]`
- au=1.6284955108858412e-06, eu=0.28897327184677124, au′=-0.0, eu′=0.03905446454882622
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.60526466369629]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 45 | 45 | 24.60526466369629 | 新引进各类人才13.1万人，新增住青院士5人、聘任院士3人、千人计划专家16人、泰山系列人才工程专家66人，人才总量180万人。 |
- GPT 引用1: span=[45, 45], relevant=0

### 陈述 10
10. 国际贸易总额达到1150亿美元，增长14%左右，其中货物进出口总额5007亿元，增长15%左右；服务进出口总额140亿美元，增长16%左右。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[58-59]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> 国际贸易总额达到1150亿美元，增长14%左右，其中货物进出口总额5007亿元，增长15%左右； 服务进出口总额140亿美元，增长16%左右。
- au=1.9009210063813953e-06, eu=0.2957197427749634, au′=-0.0, eu′=0.04029691964387894
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.8157901763916]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 58 | 58 | 23.8157901763916 | 国际贸易总额达到1150亿美元，增长14%左右，其中货物进出口总额5007亿元，增长15%左右； |
- GPT 引用1: span=[58, 59], relevant=1

### 陈述 11
11. 实际利用外资77.3亿美元，增长10%左右，总量占全省45%左右。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[56-56]`
- au=1.6284955108858412e-06, eu=0.28897327184677124, au′=-0.0, eu′=0.03905446454882622
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.60526466369629]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 56 | 56 | 24.60526466369629 | 实际利用外资77.3亿美元，增长10%左右，总量占全省45%左右。 |
- GPT 引用1: span=[56, 56], relevant=0

### 陈述 12
12. 实际到位内资1887.2亿元，增长13.3%左右。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[57-57]`
- au=1.5807482895979774e-06, eu=0.2794116735458374, au′=-0.0, eu′=0.037328090518713
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.789474487304688]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 57 | 57 | 25.789474487304688 | 实际到位内资1887.2亿元，增长13.3%左右。 |
- GPT 引用1: span=[57, 57], relevant=1

### 陈述 13
13. 国际贸易总额达到1150亿美元，增长14%左右，其中货物进出口总额5007亿元，增长15%左右；服务进出口总额140亿美元，增长16%左右。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[58-59]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> 国际贸易总额达到1150亿美元，增长14%左右，其中货物进出口总额5007亿元，增长15%左右； 服务进出口总额140亿美元，增长16%左右。
- au=1.6284955108858412e-06, eu=0.28897327184677124, au′=-0.0, eu′=0.03905446454882622
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.60526466369629]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 58 | 58 | 24.60526466369629 | 国际贸易总额达到1150亿美元，增长14%左右，其中货物进出口总额5007亿元，增长15%左右； |
- GPT 引用1: span=[58, 59], relevant=0

### 陈述 14
14. 位列全国服务外包示范城市前10强。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[60-60]`
- au=1.6742565094318707e-06, eu=0.29803910851478577, au′=-0.0, eu′=0.04072882607579231
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.55263328552246]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 60 | 60 | 23.55263328552246 | 位列全国服务外包示范城市前10强。 |
- GPT 引用1: span=[60, 60], relevant=1

### 陈述 15
15. 新开通4条空中洲际直达航线。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[61-61]`
- au=1.8090482853949652e-06, eu=0.27737218141555786, au′=-0.0, eu′=0.03696497529745102
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[26.05263328552246]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 61 | 61 | 26.05263328552246 | 新开通4条空中洲际直达航线。 |
- GPT 引用1: span=[61, 61], relevant=1

### 陈述 16
16. 港口货物和集装箱吞吐量分别达到5.1亿吨、1830万标箱，青岛港全自动化码头投入使用。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[62-62]`
- au=1.8140860902349232e-06, eu=0.27838820219039917, au′=-0.0, eu′=0.037145648151636124
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.921052932739258]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 62 | 62 | 25.921052932739258 | 港口货物和集装箱吞吐量分别达到5.1亿吨、1830万标箱，青岛港全自动化码头投入使用。 |
- GPT 引用1: span=[62, 62], relevant=1

### 陈述 17
17. 深度融入“一带一路”建设，对沿线国家投资额、进出口额分别占全市的48.3%、49%左右。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[63-63]`
- au=1.9246485862822738e-06, eu=0.3003951609134674, au′=-0.0, eu′=0.04117009416222572
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.289474487304688]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 63 | 63 | 23.289474487304688 | 深度融入“一带一路”建设，对沿线国家投资额、进出口额分别占全市的48.3%、49%左右。 |
- GPT 引用1: span=[63, 63], relevant=1

### 陈述 18
18. 新增1个国际友好合作关系城市、17个国际经济合作伙伴城市。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[64-64]`
- au=1.9430344764259644e-06, eu=0.3039999008178711, au′=-0.0, eu′=0.041850216686725616
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[22.894737243652344]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 64 | 64 | 22.894737243652344 | 新增1个国际友好合作关系城市、17个国际经济合作伙伴城市。 |
- GPT 引用1: span=[64, 64], relevant=0

### 陈述 19
19. 建成7个境外青岛工商中心。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[65-65]`
- au=1.6452690942969639e-06, eu=0.29230761528015137, au′=-0.0, eu′=0.03966596722602844
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.210527420043945]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 65 | 65 | 24.210527420043945 | 建成7个境外青岛工商中心。 |
- GPT 引用1: span=[65, 65], relevant=1

### 陈述 20
20. 成功举办亚欧数字互联互通高级别论坛、亚洲媒体峰会、金砖国家协调人第二次会议、国际教育信息化大会、国际标准化论坛等重要国际活动。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[66-66]`
- au=1.6452690942969639e-06, eu=0.29230761528015137, au′=-0.0, eu′=0.03966596722602844
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.210527420043945]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 66 | 66 | 24.210527420043945 | 成功举办亚欧数字互联互通高级别论坛、亚洲媒体峰会、金砖国家协调人第二次会议、国际教育信息化大会、国际标准化论坛等重要国际活动。 |
- GPT 引用1: span=[66, 66], relevant=0

## multifieldqa_zh_idx271 (idx=271, multifieldqa_zh)
### 问题
原告刘长生受伤的原因是什么？

### 参考答案（数据集）
- ref1: 刘长生受伤的原因是在工作期间，因木料枝疤飞出击中左眼。

### 模型全文输出
```
<statement>根据判决书，原告刘长生在2015年3月4日上午，在车间开电锯时，因木料枝疤飞出击中左眼，致左眼受伤。<cite>[4-4]</cite></statement>
```

文档句索引范围: sid ∈ [1, 52], 共 52 句。
引用指标: R=0.5, P=1.0, F1=0.6666666666666666
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=710, completion_tokens=14
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据判决书，原告刘长生在2015年3月4日上午，在车间开电锯时，因木料枝疤飞出击中左眼，致左眼受伤。
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[4-4]`
- au=1.650961507948523e-06, eu=0.2934361696243286, au′=-0.0, eu′=0.03987407684326172
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.078948974609375]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 4 | 4 | 24.078948974609375 | 原告刘长生诉称：2013年原告经被告招进公司做木工，合同一年一定，2015年3月4日上午，原告在车间开电锯时，因木料枝疤飞出击中左眼。 |
- GPT 引用1: span=[4, 4], relevant=1

## multifieldqa_zh_idx272 (idx=272, multifieldqa_zh)
### 问题
本案中原告许茂忠住院了多少天？

### 参考答案（数据集）
- ref1: 27天。

### 模型全文输出
```
<statement>根据判决书内容，本案中原告许茂忠住院了27天。<cite></cite></statement>

<statement>相关证据：<cite></cite></statement>

<statement>判决书中提到：<cite></cite></statement>

<statement>原告许茂忠受伤当日即被送往临安市於潜人民医院治疗并住院，并于同年11月20日出院。<cite>[26-26]</cite></statement>

<statement>所以本案中原告许茂忠住院了27天。<cite></cite></statement>
```

文档句索引范围: sid ∈ [1, 73], 共 73 句。
引用指标: R=0.4, P=1.0, F1=0.5714285714285715
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=1783, completion_tokens=53
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据判决书内容，本案中原告许茂忠住院了27天。
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 1
相关证据：
- GPT support_score=1; [[No]]   Need Citation: [[No]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 2
判决书中提到：
- GPT support_score=0; [[Yes]]   Need Citation: [[Yes]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 3
原告许茂忠受伤当日即被送往临安市於潜人民医院治疗并住院，并于同年11月20日出院。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[26-26]`
- au=1.8140860902349232e-06, eu=0.27838820219039917, au′=-0.0, eu′=0.037145648151636124
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.921052932739258]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 26 | 26 | 25.921052932739258 | 原告许茂忠受伤当日即被送往临安市於潜人民医院治疗并住院，并于同年11月20日出院。 |
- GPT 引用1: span=[26, 26], relevant=1

### 陈述 4
所以本案中原告许茂忠住院了27天。
- GPT support_score=0; [[Yes]]   Need Citation: [[Yes]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

## multifieldqa_zh_idx273 (idx=273, multifieldqa_zh)
### 问题
论坛的主题是什么？

### 参考答案（数据集）
- ref1: 论坛的主题是区域名校长的成长。

### 模型全文输出
```
<statement>根据通知内容，论坛的主题是“区域名校长的成长”。<cite>[1-1]</cite></statement>
```

文档句索引范围: sid ∈ [1, 12], 共 12 句。
引用指标: R=0.0, P=0.0, F1=0.0
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=830, completion_tokens=15
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据通知内容，论坛的主题是“区域名校长的成长”。
- GPT support_score=0; Rating: [[No support]]

**跨度** `[1-1]`
- au=1.9246485862822738e-06, eu=0.3003951609134674, au′=-0.0, eu′=0.04117009416222572
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.289474487304688]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 1 | 1 | 23.289474487304688 | '皖教秘师〔2022〕100 号 安徽省教育厅关于举办 2022 年长三角 基础教育校长发展高端论坛的通知 各市、省直管县（市）教育局（教育体育局）： 为深入学习贯彻党的二十大精神,贯彻落实《新时代基础教 育强师计划》，推动我省加快融入长三角地区基础教育更高质量 一体化发展步伐，根据《安徽省人民政府办公厅关于印发省政府 四季度重点工作安排的通知》（皖政办秘〔2022〕53 号）有关 要求，决定举办 2022 年长三角基础教育校长发展高端论坛。 |
- GPT 引用1: span=[1, 1], relevant=0

## multifieldqa_zh_idx274 (idx=274, multifieldqa_zh)
### 问题
上诉人的上诉请求成功了吗？

### 参考答案（数据集）
- ref1: 没有成功。

### 模型全文输出
```
<statement>根据判决书内容，沈阳双兴建设集团有限公司的上诉请求没有成功，判决驳回了其上诉请求，维持了原审判决。<cite>[41-43]</cite></statement>
```

文档句索引范围: sid ∈ [1, 45], 共 45 句。
引用指标: R=1.0, P=1.0, F1=1.0
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=736, completion_tokens=13
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据判决书内容，沈阳双兴建设集团有限公司的上诉请求没有成功，判决驳回了其上诉请求，维持了原审判决。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[41-43]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> 综上所述，沈阳双兴建设集团有限公司的上诉请求不能成立，应予驳回； 一审判决认定事实清楚，适用法律正确，应予维持。
> 依照《中华人民共和国民事诉讼法》第一百七十条第一款第一项规定，判决如下： 驳回上诉，维持原判。
- au=1.930719463416608e-06, eu=0.30158722400665283, au′=-0.0, eu′=0.04139433056116104
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.157896041870117]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 41 | 41 | 23.157896041870117 | 综上所述，沈阳双兴建设集团有限公司的上诉请求不能成立，应予驳回； |
- GPT 引用1: span=[41, 43], relevant=1

## multifieldqa_zh_idx275 (idx=275, multifieldqa_zh)
### 问题
被告向原告支付了多少保证金？

### 参考答案（数据集）
- ref1: 50万元。

### 模型全文输出
```
<statement>根据判决书内容，被告永巍工贸公司向原告支付了50万元保证金。<cite>[50-50]</cite></statement>

<statement>相关段落摘录如下：<cite></cite></statement>

<statement>本院认为，原告起诉被告永巍工贸公司不仅是基于履行担保责任后对债务人行使追偿权，同时也是依据双方签订的委托担保合同要求其承担违约责任；<cite>[51-51]</cite></statement>

<statement>原告对其余被告则是基于反担保合同要求其承担反担保责任，故案由应为委托合同及保证合同纠纷。<cite>[52-52]</cite></statement>

<statement>原告与案外人北京银行济南分行签订的《最高额保证合同》,与被告永巍工贸公司签订的《最高额委托保证合同》以及与被告神鲁纳米材料公司、蓝盾服装公司、樊永江、刘美英签订的《最高额信用反担保合同》均系当事人的真实意思表示，除《最高额委托保证合同》约定的资金占用费率超过中国人民银行同期同类人民币贷款基准利率的四倍，超出部分不予保护外，其他部分均未违反法律和行政法规的强制性规定，合法有效。<cite>[53-53]</cite></statement>

<statement>各方当事人应按合同约定履行义务，违约则应承担违约责任。<cite>[54-54]</cite></statement>

<statement>原告与被告永巍工贸公司在《最高额委托保证合同》中约定，原告按照约定履行代偿义务后即取得债权人地位，因被告永巍工贸公司未按期偿还借款，导致原告为其代偿，被告永巍工贸公司除应将该款偿还给中小企业担保中心外，亦应承担违约责任。<cite>[55-55]</cite></statement>

<statement>故原告除有权要求被告永巍工贸公司归还垫付的全部款项及其他费用、损失等外(应扣除已向原告支付的保证金50万元),还有权主张支付违约金和自代偿之日起的资金占用费。<cite>[55-56]</cite></statement>
```

文档句索引范围: sid ∈ [1, 68], 共 68 句。
引用指标: R=0.875, P=0.2857142857142857, F1=0.4307692307692308
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=6165, completion_tokens=114
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据判决书内容，被告永巍工贸公司向原告支付了50万元保证金。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[50-50]`
- au=1.6396279534092173e-06, eu=0.2911876440048218, au′=-0.0, eu′=0.039460018277168274
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.342105865478516]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 50 | 50 | 24.342105865478516 | 另查明，最高额委托保证合同签订后，被告永巍工贸公司向原告支付保证金50万元。 |
- GPT 引用1: span=[50, 50], relevant=1

### 陈述 1
相关段落摘录如下：
- GPT support_score=1; [[No]]   Need Citation: [[No]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 2
本院认为，原告起诉被告永巍工贸公司不仅是基于履行担保责任后对债务人行使追偿权，同时也是依据双方签订的委托担保合同要求其承担违约责任；
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[51-51]`
- au=1.5807482895979774e-06, eu=0.2794116735458374, au′=-0.0, eu′=0.037328090518713
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.789474487304688]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 51 | 51 | 25.789474487304688 | 本院认为，原告起诉被告永巍工贸公司不仅是基于履行担保责任后对债务人行使追偿权，同时也是依据双方签订的委托担保合同要求其承担违约责任； |
- GPT 引用1: span=[51, 51], relevant=0

### 陈述 3
原告对其余被告则是基于反担保合同要求其承担反担保责任，故案由应为委托合同及保证合同纠纷。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[52-52]`
- au=1.6068129298218992e-06, eu=0.28464409708976746, au′=-0.0, eu′=0.038267869502305984
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.131580352783203]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 52 | 52 | 25.131580352783203 | 原告对其余被告则是基于反担保合同要求其承担反担保责任，故案由应为委托合同及保证合同纠纷。 |
- GPT 引用1: span=[52, 52], relevant=0

### 陈述 4
原告与案外人北京银行济南分行签订的《最高额保证合同》,与被告永巍工贸公司签订的《最高额委托保证合同》以及与被告神鲁纳米材料公司、蓝盾服装公司、樊永江、刘美英签订的《最高额信用反担保合同》均系当事人的真实意思表示，除《最高额委托保证合同》约定的资金占用费率超过中国人民银行同期同类人民币贷款基准利率的四倍，超出部分不予保护外，其他部分均未违反法律和行政法规的强制性规定，合法有效。
- GPT support_score=0; Rating: [[No support]]

**跨度** `[53-53]`
- au=1.6015096662158612e-06, eu=0.28358200192451477, au′=-0.0, eu′=0.03807614743709564
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.263158798217773]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 53 | 53 | 25.263158798217773 | 原告与案外人北京银行济南分行签订的《最高额保证合同》，与被告永巍工贸公司签订的《最高额委托保证合同》以及与被告神鲁纳米材料公司、蓝盾服装公司、樊永江、刘美英签订的《最高额信用反担保合同》均系当事人的真实意思表示，除《最高额委托保证合同》约定的资金占用费率超过中国人民银行同期同类人民币贷款基准利率的四倍，超出部分不予保护外，其他部分均未违反法律和行政法规的强制性规定，合法有效。 |
- GPT 引用1: span=[53, 53], relevant=0

### 陈述 5
各方当事人应按合同约定履行义务，违约则应承担违约责任。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[54-54]`
- au=1.5962523320922628e-06, eu=0.28252777457237244, au′=-0.0, eu′=0.03788633644580841
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.394737243652344]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 54 | 54 | 25.394737243652344 | 各方当事人应按合同约定履行义务，违约则应承担违约责任。 |
- GPT 引用1: span=[54, 54], relevant=0

### 陈述 6
原告与被告永巍工贸公司在《最高额委托保证合同》中约定，原告按照约定履行代偿义务后即取得债权人地位，因被告永巍工贸公司未按期偿还借款，导致原告为其代偿，被告永巍工贸公司除应将该款偿还给中小企业担保中心外，亦应承担违约责任。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[55-55]`
- au=1.6068129298218992e-06, eu=0.28464409708976746, au′=-0.0, eu′=0.038267869502305984
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.131580352783203]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 55 | 55 | 25.131580352783203 | 原告与被告永巍工贸公司在《最高额委托保证合同》中约定，原告按照约定履行代偿义务后即取得债权人地位，因被告永巍工贸公司未按期偿还借款，导致原告为其代偿，被告永巍工贸公司除应将该款偿还给中小企业担保中心外，亦应承担违约责任。 |
- GPT 引用1: span=[55, 55], relevant=0

### 陈述 7
故原告除有权要求被告永巍工贸公司归还垫付的全部款项及其他费用、损失等外(应扣除已向原告支付的保证金50万元),还有权主张支付违约金和自代偿之日起的资金占用费。
- GPT support_score=1; **Rating: [[Fully supported]]**   **

**跨度** `[55-56]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> 原告与被告永巍工贸公司在《最高额委托保证合同》中约定，原告按照约定履行代偿义务后即取得债权人地位，因被告永巍工贸公司未按期偿还借款，导致原告为其代偿，被告永巍工贸公司除应将该款
> 偿还给中小企业担保中心外，亦应承担违约责任。
> 故原告除有权要求被告永巍工贸公司归还垫付的全部款项及其他费用、损失等外（应扣除已向原告支付的保证金50万元），还有权主张支付违约金和自代偿之日起的资金占用费。
- au=0.6826463341712952, eu=0.175115168094635, au′=0.682645320892334, eu′=0.04072882607579231
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.55263328552246, 23.55263328552246]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 56 | 56 | 23.55263328552246 | 故原告除有权要求被告永巍工贸公司归还垫付的全部款项及其他费用、损失等外（应扣除已向原告支付的保证金50万元），还有权主张支付违约金和自代偿之日起的资金占用费。 |
| 2 | 55 | 55 | 23.55263328552246 | 原告与被告永巍工贸公司在《最高额委托保证合同》中约定，原告按照约定履行代偿义务后即取得债权人地位，因被告永巍工贸公司未按期偿还借款，导致原告为其代偿，被告永巍工贸公司除应将该款偿还给中小企业担保中心外，亦应承担违约责任。 |
- GPT 引用1: span=[55, 56], relevant=1

## multifieldqa_zh_idx276 (idx=276, multifieldqa_zh)
### 问题
如何进行讲义学习？

### 参考答案（数据集）
- ref1: 讲义学习可以在线阅读，或下载到本地阅读。

### 模型全文输出
```
<statement>根据操作指南，进行讲义学习的方法如下：<cite></cite></statement>

<statement>1. 在课程学习区，点击进入要学习的课程。<cite>[43-43]</cite></statement>

<statement>2. 在课程页面，点击进入“讲义学习”板块。<cite></cite></statement>

<statement>3. 在讲义学习页面，可以在线阅读讲义，也可以点击下载按钮下载讲义到本地。<cite>[50-50]</cite></statement>

<statement>4. 讲义学习完成后，可以点击“本课自测”或“下一课”继续学习其他章节。<cite>[49-49]</cite></statement>

<statement>所以，通过点击课程、讲义学习、下载等按钮，就可以完成讲义学习。注意，讲义学习是在线阅读或下载到本地阅读，不能在线观看。<cite></cite></statement>
```

文档句索引范围: sid ∈ [1, 81], 共 81 句。
引用指标: R=0.6666666666666666, P=1.0, F1=0.8
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=2885, completion_tokens=69
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据操作指南，进行讲义学习的方法如下：
- GPT support_score=1; [[No]]   Need Citation: [[No]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 1
1. 在课程学习区，点击进入要学习的课程。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[43-43]`
- au=1.9067709899900365e-06, eu=0.29687488079071045, au′=-0.0, eu′=0.04051172360777855
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.68421173095703]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 43 | 43 | 23.68421173095703 | 学员课程学习通过首页的课程中心按钮和课程学习区，可以点击进入课程学习，其中首页课程学习区会自 动记录你当前学习课程，方便学员进行连续课程学习。 |
- GPT 引用1: span=[43, 43], relevant=1

### 陈述 2
2. 在课程页面，点击进入“讲义学习”板块。
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 3
3. 在讲义学习页面，可以在线阅读讲义，也可以点击下载按钮下载讲义到本地。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[50-50]`
- au=1.6284955108858412e-06, eu=0.28897327184677124, au′=-0.0, eu′=0.03905446454882622
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.60526466369629]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 50 | 50 | 24.60526466369629 | 讲义学习讲义学习，可在线阅读， 学员也可下载下来在本地 阅读7 大学生入党积极分子学习与考试系统-学员使用操作指南2.3.3. |
- GPT 引用1: span=[50, 50], relevant=1

### 陈述 4
4. 讲义学习完成后，可以点击“本课自测”或“下一课”继续学习其他章节。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[49-49]`
- au=1.6068129298218992e-06, eu=0.28464409708976746, au′=-0.0, eu′=0.038267869502305984
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.131580352783203]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 49 | 49 | 25.131580352783203 | 课件学习6 大学生入党积极分子学习与考试系统-学员使用操作指南此页面为课件学习提供在线学习，学 员也可下载到本地学习本课学习完成，可点击本课自测或下 一课2.3.2. |
- GPT 引用1: span=[49, 49], relevant=1

### 陈述 5
所以，通过点击课程、讲义学习、下载等按钮，就可以完成讲义学习。注意，讲义学习是在线阅读或下载到本地阅读，不能在线观看。
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

## multifieldqa_zh_idx277 (idx=277, multifieldqa_zh)
### 问题
原审第三人郑新家对于上诉人意见的看法是什么？

### 参考答案（数据集）
- ref1: 原审第三人郑新家同意上诉人的意见。

### 模型全文输出
```
<statement>根据判决书中的内容，原审第三人郑新家同意上诉人的意见。<cite></cite></statement>

<statement>相关段落摘录如下：<cite></cite></statement>

<statement>原审第三人郑新家到庭参加诉讼，述称同意上诉人的意见。<cite>[3-3][17-17]</cite></statement>

<statement>所以原审第三人郑新家同意上诉人的意见。<cite></cite></statement>
```

文档句索引范围: sid ∈ [1, 45], 共 45 句。
引用指标: R=0.5, P=0.5, F1=0.5
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=1764, completion_tokens=51
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据判决书中的内容，原审第三人郑新家同意上诉人的意见。
- GPT support_score=0; [[Yes]]   Need Citation: [[Yes]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 1
相关段落摘录如下：
- GPT support_score=1; [[No]]   Need Citation: [[No]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 2
原审第三人郑新家到庭参加诉讼，述称同意上诉人的意见。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[3-3]`
- au=1.6396279534092173e-06, eu=0.2911876440048218, au′=-0.0, eu′=0.039460018277168274
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.342105865478516]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 3 | 3 | 24.342105865478516 | 上诉人沈阳双兴建设集团有限公司的委托诉讼代理人王嵩，被上诉人董士军，被上诉人曹德志的委托诉讼代理人崔善良，原审第三人郑新家到庭参加诉讼。 |

**跨度** `[17-17]`
- au=1.765542265275144e-06, eu=0.26855114102363586, au′=-0.0, eu′=0.03541472181677818
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[27.23684310913086]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 17 | 17 | 27.23684310913086 | 郑新家述称，同意上诉人的意见。 |
- GPT 引用1: span=[3, 3], relevant=0
- GPT 引用2: span=[17, 17], relevant=1

### 陈述 3
所以原审第三人郑新家同意上诉人的意见。
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

## multifieldqa_zh_idx278 (idx=278, multifieldqa_zh)
### 问题
龙门石窟有哪些景点？

### 参考答案（数据集）
- ref1: 龙门石窟包括西山石窟、东山石窟、香山寺和白园等景点。

### 模型全文输出
```
<statement>根据文档内容，龙门石窟景区包括以下景点：<cite></cite></statement>

<statement>1. 西山石窟<cite>[132-132]</cite></statement>

<statement>2. 东山石窟 <cite>[132-132]</cite></statement>

<statement>3. 香山寺<cite>[168-168]</cite></statement>

<statement>4. 白园<cite>[170-170]</cite></statement>

<statement>其中西山石窟和东山石窟是龙门石窟的核心景点，分别位于洛阳南郊的伊水河畔东侧和西侧，保存了大量唐代石窟佛像和壁画。香山寺和白园也是重要景点，香山寺是唐代洛阳著名的香山寺，白园是唐代诗人白居易的墓园。<cite></cite></statement>
```

文档句索引范围: sid ∈ [1, 186], 共 186 句。
引用指标: R=0.8333333333333334, P=1.0, F1=0.9090909090909091
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=3334, completion_tokens=72
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据文档内容，龙门石窟景区包括以下景点：
- GPT support_score=1; [[No]]   Need Citation: [[No]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 1
1. 西山石窟
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[132-132]`
- au=1.7108611700678011e-06, eu=0.3052207827568054, au′=-0.0, eu′=0.04208194464445114
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[22.763158798217773]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 132 | 132 | 22.763158798217773 | 龙门石窟景区(包括：西山石窟、东 山石窟、香山寺、白园) (1)参加全国统一普通高校招生考试被高校录取的在校研究生、本科生、专科生，在校中、小 学生凭有效学生证购买半价票。 |
- GPT 引用1: span=[132, 132], relevant=1

### 陈述 2
2. 东山石窟
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[132-132]`
- au=1.95558504856308e-06, eu=0.3064514994621277, au′=-0.0, eu′=0.0423162542283535
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[22.631580352783203]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 132 | 132 | 22.631580352783203 | 龙门石窟景区(包括：西山石窟、东 山石窟、香山寺、白园) (1)参加全国统一普通高校招生考试被高校录取的在校研究生、本科生、专科生，在校中、小 学生凭有效学生证购买半价票。 |
- GPT 引用1: span=[132, 132], relevant=1

### 陈述 3
3. 香山寺
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[168-168]`
- au=1.930719463416608e-06, eu=0.30158722400665283, au′=-0.0, eu′=0.04139433056116104
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.157896041870117]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 168 | 168 | 23.157896041870117 | 香山寺位于 十三朝古都洛阳城南13公里处的香山西坳，与世界文化遗产——龙门石窟西山窟区一衣带水，隔 河相望，与龙门石窟东山窟区和白园一脉相连，并肩邻立。 |
- GPT 引用1: span=[168, 168], relevant=1

### 陈述 4
4. 白园
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[170-170]`
- au=0.6815015077590942, eu=0.18581904470920563, au′=0.6815006732940674, eu′=0.04365306720137596
- top_logits 方差 (pvariance, 有限 logit): 0.35058822658265854
- top_logits 数值: `[22.5, 21.3157901763916]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 170 | 170 | 22.5 | 白园，位于 洛阳龙门风景名胜区东山琵琶峰上，是唐代诗人白居易的墓园，占地面积3万平方米，1961年被国务院公布 为国家重点文物保护单位。 |
| 2 | 168 | 168 | 21.3157901763916 | 香山寺位于 十三朝古都洛阳城南13公里处的香山西坳，与世界文化遗产——龙门石窟西山窟区一衣带水，隔 河相望，与龙门石窟东山窟区和白园一脉相连，并肩邻立。 |
- GPT 引用1: span=[170, 170], relevant=1

### 陈述 5
其中西山石窟和东山石窟是龙门石窟的核心景点，分别位于洛阳南郊的伊水河畔东侧和西侧，保存了大量唐代石窟佛像和壁画。香山寺和白园也是重要景点，香山寺是唐代洛阳著名的香山寺，白园是唐代诗人白居易的墓园。
- GPT support_score=0; [[Yes]]   Need Citation: [[Yes]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

## multifieldqa_zh_idx279 (idx=279, multifieldqa_zh)
### 问题
报告强调要坚持什么思想观点？

### 参考答案（数据集）
- ref1: 报告强调要坚持科学发展观。

### 模型全文输出
```
<statement>根据报告，要坚持以下思想观点：<cite></cite></statement>

<statement>1. 邓小平理论和“三个代表”重要思想<cite>[80-80][255-255]</cite></statement>

<statement>2. 科学发展观<cite>[80-80][255-255]</cite></statement>

<statement>3. 省第八次党代会和市第三次党代会精神<cite>[255-255]</cite></statement>

<statement>4. 以胡锦涛同志为总书记的党中央的指示和要求<cite>[255-255]</cite></statement>

<statement>5. 党的十七大胜利召开的要求<cite>[255-255]</cite></statement>

<statement>报告提出要牢固树立科学发展观，深入贯彻落实省第八次党代会和市第三次党代会精神，聚精会神搞建设，一心一意谋发展，确保各项目标圆满实现，以优异的成绩迎接党的十七大胜利召开。<cite>[255-255]</cite></statement>
```

文档句索引范围: sid ∈ [1, 256], 共 256 句。
引用指标: R=0.7857142857142857, P=1.0, F1=0.88
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=5518, completion_tokens=106
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据报告，要坚持以下思想观点：
- GPT support_score=1; [[No]]   Need Citation: [[No]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 1
1. 邓小平理论和“三个代表”重要思想
- GPT support_score=1; **Rating**: [[Fully supported]]   **

**跨度** `[80-80]`
- au=0.6806294918060303, eu=0.19289332628250122, au′=0.6806285381317139, eu′=0.045618243515491486
- top_logits 方差 (pvariance, 有限 logit): 0.6232684603028247
- top_logits 数值: `[21.710527420043945, 20.131580352783203]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 80 | 80 | 21.710527420043945 | （四）政府自身建设不断加强</p><p>高度重视思想建设，深入学习邓小平理论、“三个代表”重要思想，全面贯彻落实科学发展观，进一步坚定理想信念，执政为民的意识不断增强。 |
| 2 | 95 | 95 | 20.131580352783203 | </p><p>二、坚持科学发展，切实抓好2007年的各项工作</p><p>根据中央和省委、省政府的精神，按照市第三次党代会和市委三届二次全会的部署，2007年政府工作的总体要求是：以邓小平理论和“三个代表”重要思想为指导，全面落实科学发展观，继续坚持以改革开放和科技进步为动力的“生态立市、烟草兴市、工业强市、农业稳市”战略，完善“三优一特”经济发展思路，更加重视社会主义新农村建设、让农民增加实惠，更加重视生态市建设、为群众创造良好的生存环境和发展条件，更加重视社会事业的发展、推进经济社会协调发展，更加重视提高广大人民群众的生活水平、加快和谐社会建设，突出加快县区特色经济发展，高度关注民生，促进发展观念、增长方式和工作作风转变，努力实现全市经济社会又好又快发展。 |

**跨度** `[255-255]`
- au=0.681172251701355, eu=0.19487175345420837, au′=0.6811712384223938, eu′=0.04617253318428993
- top_logits 方差 (pvariance, 有限 logit): 0.0173131382698557
- top_logits 数值: `[20.789474487304688, 20.526315689086914]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 95 | 95 | 20.789474487304688 | </p><p>二、坚持科学发展，切实抓好2007年的各项工作</p><p>根据中央和省委、省政府的精神，按照市第三次党代会和市委三届二次全会的部署，2007年政府工作的总体要求是：以邓小平理论和“三个代表”重要思想为指导，全面落实科学发展观，继续坚持以改革开放和科技进步为动力的“生态立市、烟草兴市、工业强市、农业稳市”战略，完善“三优一特”经济发展思路，更加重视社会主义新农村建设、让农民增加实惠，更加重视生态市建设、为群众创造良好的生存环境和发展条件，更加重视社会事业的发展、推进经济社会协调发展，更加重视提高广大人民群众的生活水平、加快和谐社会建设，突出加快县区特色经济发展，高度关注民生，促进发展观念、增长方式和工作作风转变，努力实现全市经济社会又好又快发展。 |
| 2 | 255 | 255 | 20.526315689086914 | 让我们在市委的正确领导下，紧密团结在以胡锦涛同志为总书记的党中央周围，高举邓小平理论和“三个代表”重要思想伟大旗帜，牢固树立科学发展观，深入贯彻落实省第八次党代会和市第三次党代会精神，聚精会神搞建设，一心一意谋发展，确保各项目标圆满实现，以优异的成绩迎接党的十七大胜利召开！ |
- GPT 引用1: span=[80, 80], relevant=1
- GPT 引用2: span=[255, 255], relevant=1

### 陈述 2
2. 科学发展观
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[80-80]`
- au=1.930719463416608e-06, eu=0.30158722400665283, au′=-0.0, eu′=0.04139433056116104
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.157896041870117]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 80 | 80 | 23.157896041870117 | （四）政府自身建设不断加强</p><p>高度重视思想建设，深入学习邓小平理论、“三个代表”重要思想，全面贯彻落实科学发展观，进一步坚定理想信念，执政为民的意识不断增强。 |

**跨度** `[255-255]`
- au=0.6808781027793884, eu=0.19791662693023682, au′=0.6808770895004272, eu′=0.04702969640493393
- top_logits 方差 (pvariance, 有限 logit): 0.0692520511447583
- top_logits 数值: `[20.526315689086914, 20.0]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 255 | 255 | 20.526315689086914 | 让我们在市委的正确领导下，紧密团结在以胡锦涛同志为总书记的党中央周围，高举邓小平理论和“三个代表”重要思想伟大旗帜，牢固树立科学发展观，深入贯彻落实省第八次党代会和市第三次党代会精神，聚精会神搞建设，一心一意谋发展，确保各项目标圆满实现，以优异的成绩迎接党的十七大胜利召开！ |
| 2 | 95 | 95 | 20.0 | </p><p>二、坚持科学发展，切实抓好2007年的各项工作</p><p>根据中央和省委、省政府的精神，按照市第三次党代会和市委三届二次全会的部署，2007年政府工作的总体要求是：以邓小平理论和“三个代表”重要思想为指导，全面落实科学发展观，继续坚持以改革开放和科技进步为动力的“生态立市、烟草兴市、工业强市、农业稳市”战略，完善“三优一特”经济发展思路，更加重视社会主义新农村建设、让农民增加实惠，更加重视生态市建设、为群众创造良好的生存环境和发展条件，更加重视社会事业的发展、推进经济社会协调发展，更加重视提高广大人民群众的生活水平、加快和谐社会建设，突出加快县区特色经济发展，高度关注民生，促进发展观念、增长方式和工作作风转变，努力实现全市经济社会又好又快发展。 |
- GPT 引用1: span=[80, 80], relevant=1
- GPT 引用2: span=[255, 255], relevant=1

### 陈述 3
3. 省第八次党代会和市第三次党代会精神
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[255-255]`
- au=1.5858721553740907e-06, eu=0.2804427146911621, au′=-0.0, eu′=0.03751233592629433
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.657896041870117]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 255 | 255 | 25.657896041870117 | 让我们在市委的正确领导下，紧密团结在以胡锦涛同志为总书记的党中央周围，高举邓小平理论和“三个代表”重要思想伟大旗帜，牢固树立科学发展观，深入贯彻落实省第八次党代会和市第三次党代会精神，聚精会神搞建设，一心一意谋发展，确保各项目标圆满实现，以优异的成绩迎接党的十七大胜利召开！ |
- GPT 引用1: span=[255, 255], relevant=1

### 陈述 4
4. 以胡锦涛同志为总书记的党中央的指示和要求
- GPT support_score=0; Rating: [[No support]]

**跨度** `[255-255]`
- au=1.7796883184928447e-06, eu=0.27142849564552307, au′=-0.0, eu′=0.03591682016849518
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[26.842105865478516]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 255 | 255 | 26.842105865478516 | 让我们在市委的正确领导下，紧密团结在以胡锦涛同志为总书记的党中央周围，高举邓小平理论和“三个代表”重要思想伟大旗帜，牢固树立科学发展观，深入贯彻落实省第八次党代会和市第三次党代会精神，聚精会神搞建设，一心一意谋发展，确保各项目标圆满实现，以优异的成绩迎接党的十七大胜利召开！ |
- GPT 引用1: span=[255, 255], relevant=1

### 陈述 5
5. 党的十七大胜利召开的要求
- GPT support_score=0.5; **Rating: [[Partially supported]]**

**跨度** `[255-255]`
- au=1.5318007626774488e-06, eu=0.26950347423553467, au′=-0.0, eu′=0.03558051958680153
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[27.10526466369629]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 255 | 255 | 27.10526466369629 | 让我们在市委的正确领导下，紧密团结在以胡锦涛同志为总书记的党中央周围，高举邓小平理论和“三个代表”重要思想伟大旗帜，牢固树立科学发展观，深入贯彻落实省第八次党代会和市第三次党代会精神，聚精会神搞建设，一心一意谋发展，确保各项目标圆满实现，以优异的成绩迎接党的十七大胜利召开！ |
- GPT 引用1: span=[255, 255], relevant=1

### 陈述 6
报告提出要牢固树立科学发展观，深入贯彻落实省第八次党代会和市第三次党代会精神，聚精会神搞建设，一心一意谋发展，确保各项目标圆满实现，以优异的成绩迎接党的十七大胜利召开。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[255-255]`
- au=1.5807482895979774e-06, eu=0.2794116735458374, au′=-0.0, eu′=0.037328090518713
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.789474487304688]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 255 | 255 | 25.789474487304688 | 让我们在市委的正确领导下，紧密团结在以胡锦涛同志为总书记的党中央周围，高举邓小平理论和“三个代表”重要思想伟大旗帜，牢固树立科学发展观，深入贯彻落实省第八次党代会和市第三次党代会精神，聚精会神搞建设，一心一意谋发展，确保各项目标圆满实现，以优异的成绩迎接党的十七大胜利召开！ |
- GPT 引用1: span=[255, 255], relevant=1

## multifieldqa_zh_idx280 (idx=280, multifieldqa_zh)
### 问题
本报告从几个方面总结了工作？

### 参考答案（数据集）
- ref1: 6个方面。

### 模型全文输出
```
<statement>根据报告内容，可以总结出以下几个方面：<cite></cite></statement>

<statement>1. 改革开放创新重大举措有效落地。<cite>[7-18]</cite></statement>

<statement>2. 经济转型升级取得新进展。<cite>[20-31]</cite></statement>

<statement>3. 环境综合治理走在前列。<cite>[35-41]</cite></statement>

<statement>4. 都市区共建加快推进。<cite>[42-52]</cite></statement>

<statement>5. 民生福祉持续改善。<cite>[53-69]</cite></statement>

<statement>6. 政府自身建设不断加强。<cite>[70-84]</cite></statement>
```

文档句索引范围: sid ∈ [1, 84], 共 84 句。
引用指标: R=1.0, P=1.0, F1=1.0
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=8070, completion_tokens=94
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据报告内容，可以总结出以下几个方面：
- GPT support_score=1; [[No]]   Need Citation: [[No]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 1
1. 改革开放创新重大举措有效落地。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[7-18]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> （一）改革开放创新重大举措有效落地。 市县两级“四张清单一张网”
> 基本建成，权力清单事项平均削减36.8%，取消行政事业性收费97项，率先探索“互联网+政务服务”建设，“网上政府”建设三年行动计划全面实施。
> 供给侧结构性改革初见成效，40多项国家和省部级改革试点扎实推进。
> 义乌国际贸易综合改革试点取得重大突破，国家现代服务业综合试点绩效居全国前列，全国低丘缓坡综合开发利用试点进展顺利，浦江协调推进“四个全面”战略布局试点成效明显。
> 商事制度改革加快推进，新增市场主体数量居全省前列。 认真落实国地税征管体制改革及“营改增”试点，“互联网+财税”做法在全省推广。
> 科技创新力度加大，研发经费支出占生产总值比重超过2%。 义乌开发区升格为国家级开发区。 规划建设22个特色小镇，其中省级11个。
> 累计入选和引进“千人计划”专家129人、评选“双龙计划”人才项目154个。 获评中国“一带一路”最具活力城市，义甬舟开放大通道加快建设，“义新欧”班列开通线路8条、运送量突破1
> 万标箱，义乌国际邮件互换局业务量居全国第四，金义综合保税区通过国家验收，全球跨境电商大会永久落户，陆港、口岸加快建设，海外名校学子走进金华古村落活动产生良好影响。
> 外贸出口额年均增长25.2%，总量跃居全省第三。
- au=1.7108611700678011e-06, eu=0.3052207827568054, au′=-0.0, eu′=0.04208194464445114
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[22.763158798217773]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 7 | 7 | 22.763158798217773 | （一）改革开放创新重大举措有效落地。 |
- GPT 引用1: span=[7, 18], relevant=1

### 陈述 2
2. 经济转型升级取得新进展。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[20-31]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> （二）经济转型升级取得新进展。
> 三次产业结构实现“二三一”向“三二一”转变，三产增加值占生产总值比重达52.3%，规上工业总产值突破5000亿元，工业增加值突破1000亿元，农业全省排名实现晋位。 先进装备制
> 造业加快发展，新能源汽车产业规模居全省第三，新引进华科汽车、华灿光电等总投资超15亿元的浙（婺）商回归重大产业项目38个，新增上市公司10家、新三板挂牌企业46家，“金华制造上
> 京东”企业6277家，“机器换人、空间换地”经验在全省推广。 义乌进入生产总值千亿元俱乐部。 永康规上工业总产值突破千亿元，成为全国首个县级质量强市示范城市。
> 兰溪规上工业进入全省20强，华东铝业实施战略重组。 浦江水晶等行业治转结合成效明显。 武义获批国家级出口质量安全示范区。 东阳建筑业发展稳居全国县市榜首。
> 网络零售额居全省第二，7个县市全部入围中国电子商务百强县，青岩刘等一批中国淘宝村蓬勃发展。
> 旅游收入居全省第四，横店影视文化产业实验区成为国家级文化和科技融合示范基地，影视城年游客接待量居全国景区前三。 金融存贷款余额位居全省第四。
- au=1.6567056491112453e-06, eu=0.2945735454559326, au′=-0.0, eu′=0.04008438438177109
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.947368621826172]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 20 | 20 | 23.947368621826172 | （二）经济转型升级取得新进展。 |
- GPT 引用1: span=[20, 31], relevant=1

### 陈述 3
3. 环境综合治理走在前列。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[35-41]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> （三）环境综合治理走在前列。 全国水环境综合整治、违法建设治理、农村生活垃圾分类及资源化利用等现场会在我市召开，成功创建国家园林城市、国家森林城市、国家节水型城市，各县市创建为
> 国家卫生城市、省级森林城市，磐安、义乌成为国家生态县市。 全省“五水共治”第一枪在浦江打响，河长制、“治水十法”在全国推广，农村面源污染治理成效明显，沙金兰等水源涵养生态功能区
> 建设深入推进，兰江等全流域防洪能力进一步提升，兑现了“一年灭黑臭、两年提水质、三年可游泳”承诺，地表水Ⅲ类以上水质断面占比达到97.6%，连续三年荣获大禹鼎。
> 实施气尘合治三年行动，2016年金华市区PM2.5平均浓度比2013年下降34.3%，空气质量优良率提高到80.6%，改善幅度居全省前列。 全面推进“三改一拆”“整治环境、靓化
> 金华”等行动，婺城二七棚户区改造成全省典型，金东横塘沿城中村改造做法获百姓点赞，东阳红椿巷、兰溪排岭、浦江金狮湖、义乌仓后等区块大拆大整取得突破。
> 深化美丽乡村建设，加强历史文化村落保护利用，农村生活垃圾“两次四分法”在全国推广，垃圾分类处理和污水治理村庄覆盖率分别达98.1%和95%。
> 浦江成为全省美丽乡村示范县，浦阳江国家湿地公园获批。
- au=1.861421651483397e-06, eu=0.28787869215011597, au′=-0.0, eu′=0.038854800164699554
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.73684310913086]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 35 | 35 | 24.73684310913086 | （三）环境综合治理走在前列。 |
- GPT 引用1: span=[35, 41], relevant=1

### 陈述 4
4. 都市区共建加快推进。
- GPT support_score=1; **Rating**: [[Fully supported]]   **

**跨度** `[42-52]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> （四）都市区共建加快推进。 编制完成都市区规划纲要和综合交通规划、轨道交通线网规划、金义都市新区规划、基础设施空间一体化规划等8个专项规划，村庄规划覆盖率达95%。
> 实施都市区建设三年行动，基本形成“一小时通勤圈”，城市化率达到65.7%，被列为长三角26个中心城市、沪杭金发展带重要节点城市。
> 金华跨入高铁时代，杭长客专、金温铁路新线开通，金义东市域轨道交通、金甬铁路开工建设，金华铁路南站和新货场投入运营，铁路客站交通枢纽区改造加快推进。
> 东永高速、杭金衢高速改扩建一期和一批城际快速路建成通车，义乌疏港高速、杭绍金台高速、临金高速加快建设。
> 布局推动金义都市新区、金华山旅游经济区、多湖中央商务区开发建设，湖海塘等重点区块建设步伐加快。 开通BRT 线路6条，金华市区公交分担率提高到20.2%。
> 衢江金华段航运开发和义乌机场改扩建、横店通用机场建设加快推进。 治堵工作考核三年全省第一。 九峰水库验收运行，第二水厂建成使用。
> 我市成为特高压交流直流混联城市，全社会用电量突破300亿千瓦时。
- au=1.9067709899900365e-06, eu=0.29687488079071045, au′=-0.0, eu′=0.04051172360777855
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.68421173095703]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 42 | 42 | 23.68421173095703 | （四）都市区共建加快推进。 |
- GPT 引用1: span=[42, 52], relevant=1

### 陈述 5
5. 民生福祉持续改善。
- GPT support_score=1; **Rating**: [[Fully supported]]   **

**跨度** `[53-69]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> （五）民生福祉持续改善。 财政民生支出占比超过70%，公共服务均等化程度达90%以上。 每年办好十方面实事。 全面消除年人均收入4600元以下贫困现象，城乡居民收入差距缩小。
> 武义、磐安实现“摘帽快跑、绿色发展”。 新增城镇就业37万人，城镇登记失业率控制在2.9%以内。
> 最低工资标准提高69%，企业退休人员基本养老金标准提高57%，城乡居民基础养老金最低标准提高100%，农村最低生活保障标准提高18.5%。
> 建成城乡居家养老服务照料中心3020家，惠及20余万老年人。 金东居家养老模式获评全国民生示范工程。 城乡居民医疗保险实现并轨。
> 基本公共文化服务体系不断健全，建成农村文化礼堂784个、居全省第四，市体育中心、中国婺剧院、东阳木雕博物馆等投入使用。 开展最美系列创建评选，一城五人“感动中国”。
> 教育改革发展步伐加快，义务教育入学率和巩固率达100%，实现全国义务教育基本均衡县和省体育强县全覆盖。 “双下沉、两提升”
> 扎实推进，9家县级医院与上级医院开展紧密型合作办医，基本形成城乡20分钟医疗服务圈。 依法稳妥实施全面两孩政策，出生人口性别比实现六连降。
> 社会治安、安全生产、食品药品安全、信访、调解、社区矫正、应急管理等工作扎实有效，实现平安银鼎全覆盖，婺城、武义、磐安获平安金鼎。
> 创建8890便民服务平台，为群众解决各类诉求162万件，办结率和回访满意率99%以上。
- au=1.6121624639708898e-06, eu=0.28571420907974243, au′=-0.0, eu′=0.038461536169052124
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.0]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 53 | 53 | 25.0 | （五）民生福祉持续改善。 |
- GPT 引用1: span=[53, 69], relevant=1

### 陈述 6
6. 政府自身建设不断加强。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[70-84]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> （六）政府自身建设不断加强。
> 从严落实中央八项规定精神，扎实开展党的群众路线教育实践活动、“三严三实”专题教育、“两学一做”学习教育，认真抓好中央和省委巡视组反馈意见的整改落实。
> 建立健全政府常务会议学法、领导干部夜读学法、政府法律顾问等制度，行政诉讼案件败诉率明显低于全省平均水平。
> 坚决贯彻市委决策部署，主动接受人大及其常委会依法监督、政协民主监督和社会监督、舆论监督，邀请人大代表、政协委员列席政府常务会议。
> 改进办理方式，认真做好人大代表、政协委员建议提案办理工作。 加强廉政风险点防控，审计监督实现全覆盖，政务公开力度进一步加大。
> 建立政府投资项目超概算责任追究办法、财政性资金竞争性存放管理制度，强化公共资源交易领域信用建设，完成党政机关公车改革，“三公”经费支出持续下降。
> 深化行政执法体制改革，综合行政执法全面推进。 组建市场监管局、卫生计生委。 全面发展老龄、残疾人和关心下一代等事业，广泛开展慈善救助活动。
> 支持工会、共青团、妇联等人民团体开展工作。 加强民族、宗教、外侨、对台、统计、气象、民防、档案、地方志、仲裁等工作。
> 完成“六五”普法、第三次经济普查和第六次人口普查，启动“七五”普法。 支持国防建设和军队改革，实现全国双拥模范城“三连冠”。 金华在全省首批获得地方立法权。
- au=1.6340368347300682e-06, eu=0.29007625579833984, au′=-0.0, eu′=0.03925619646906853
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.473684310913086]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 70 | 70 | 24.473684310913086 | （六）政府自身建设不断加强。 |
- GPT 引用1: span=[70, 84], relevant=1

## multifieldqa_zh_idx281 (idx=281, multifieldqa_zh)
### 问题
作者的父亲刘英在中共浙江临时省委以及浙江正式省委任职期间有哪些具体工作？

### 参考答案（数据集）
- ref1: 父亲担任省委书记，统一领导浙江党的工作，贯彻党的统一战线政策，组织领导开展抗日斗争；大张旗鼓地发动群众开展抗日救亡运动，成立统战部，亲自兼任统战部长，推动与国民党浙江省政府合作，推动颁布了以中共《抗日救国十大纲领》为基础制定的《浙江省战时政治纲领》，促进了抗日进步力量的发展。

### 模型全文输出
```
<statement>根据文章内容，作者的父亲刘英在中共浙江临时省委以及浙江正式省委任职期间的主要工作有：<cite></cite></statement>

<statement>1. 统一领导浙江党的工作，贯彻党的统一战线政策，组织领导开展抗日斗争。<cite>[214-214][215-215]</cite></statement>

<statement>2. 大张旗鼓地发动群众开展抗日救亡运动，成立统战部，亲自兼任统战部长。<cite>[216-216]</cite></statement>

<statement>3. 与国民党浙江省政府建立上层统战关系，推动颁布《浙江省战时政治纲领》。<cite>[217-217]</cite></statement>

<statement>4. 与省长黄绍竑协商，设立新四军驻丽水、温州、平阳办事处，并得到子弹和军衣的供给。<cite>[218-218]</cite></statement>

<statement>5. 指示协助李友邦筹建台湾义勇队，并成立台湾义勇队少年团。<cite>[220-222]</cite></statement>

<statement>6. 在台湾义勇队中秘密建立中共党支部。<cite>[224-224]</cite></statement>

<statement>7. 组织领导浙西南和闽东两个特委，建立党的组织，政权组织，指导分配土地。<cite>[201-201]</cite></statement>

<statement>8. 配合中央红军主力部队的行动，调动和牵制住大批敌人，彻底粉碎敌人对中央红军的“清剿”计划。<cite>[197-197]</cite></statement>

<statement>9. 在两次反“清剿”战斗中，前后牵制国民党兵力三四十个团，历时各达八个多月之久。<cite>[203-203]</cite></statement>

<statement>10. 担任政委，领导浙西南人民全面建设革命根据地，开展武装斗争。<cite>[200-200]</cite></statement>

<statement>11. 在平阳集中，改番号为国民革命军闽浙边抗日游击总队，粟裕伯伯任总队长，父亲任政委。<cite>[210-210]</cite></statement>

<statement>12. 编入新四军，开赴皖南北上抗日。<cite>[212-212]</cite></statement>

<statement>所以刘英在浙江工作期间，主要负责领导抗日救亡运动，开展统战工作，领导建设革命根据地，组织领导武装斗争等工作。<cite></cite></statement>
```

文档句索引范围: sid ∈ [1, 225], 共 225 句。
引用指标: R=0.75, P=1.0, F1=0.8571428571428571
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=10024, completion_tokens=184
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据文章内容，作者的父亲刘英在中共浙江临时省委以及浙江正式省委任职期间的主要工作有：
- GPT support_score=1; [[No]]   Need Citation: [[No]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 1
1. 统一领导浙江党的工作，贯彻党的统一战线政策，组织领导开展抗日斗争。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[214-214]`
- au=0.6822757124900818, eu=0.17882347106933594, au′=0.6822749376296997, eu′=0.04173530638217926
- top_logits 方差 (pvariance, 有限 logit): 0.10820617305944324
- top_logits 数值: `[23.289474487304688, 22.631580352783203]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 215 | 215 | 23.289474487304688 | 父亲任省委书记，统一领导浙江党的工作，贯彻党的统一战线政策，组织领导开展抗日斗争。 |
| 2 | 214 | 214 | 22.631580352783203 | 1938年5月，根据中共中央东南分局的指示，并经中共中央批准，恢复成立了中共浙江临时省委，9月转为正式省委。 |

**跨度** `[215-215]`
- au=1.5962523320922628e-06, eu=0.28252777457237244, au′=-0.0, eu′=0.03788633644580841
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.394737243652344]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 215 | 215 | 25.394737243652344 | 父亲任省委书记，统一领导浙江党的工作，贯彻党的统一战线政策，组织领导开展抗日斗争。 |
- GPT 引用1: span=[214, 215], relevant=1

### 陈述 2
2. 大张旗鼓地发动群众开展抗日救亡运动，成立统战部，亲自兼任统战部长。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[216-216]`
- au=1.6340368347300682e-06, eu=0.29007625579833984, au′=-0.0, eu′=0.03925619646906853
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.473684310913086]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 216 | 216 | 24.473684310913086 | 父亲认真贯彻党的抗日民族统一战线政策，大张旗鼓地发动群众开展抗日救亡运动，专门成立了统战部，父亲亲自兼任统战部长。 |
- GPT 引用1: span=[216, 216], relevant=1

### 陈述 3
3. 与国民党浙江省政府建立上层统战关系，推动颁布《浙江省战时政治纲领》。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[217-217]`
- au=1.5962523320922628e-06, eu=0.28252777457237244, au′=-0.0, eu′=0.03788633644580841
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.394737243652344]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 217 | 217 | 25.394737243652344 | 父亲与省长黄绍竑为首的国民党浙江省政府建立了上层统战关系，推动黄绍竑颁布了以中共《抗日救国十大纲领》为基础制定的《浙江省战时政治纲领》，使抗战初期国共合作的浙江抗日救亡运动开展得轰轰烈烈，促进了抗日进步力量的发展。 |
- GPT 引用1: span=[217, 217], relevant=1

### 陈述 4
4. 与省长黄绍竑协商，设立新四军驻丽水、温州、平阳办事处，并得到子弹和军衣的供给。
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[218-218]`
- au=1.8090482853949652e-06, eu=0.27737218141555786, au′=-0.0, eu′=0.03696497529745102
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[26.05263328552246]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 218 | 218 | 26.05263328552246 | 父亲和黄绍竑就挺进师部队改编后有关事宜进行协商，黄绍竑同意在丽水设立“新四军驻丽水办事处”，在温州设立“新四军驻温州通讯处”，在平阳设立“新四军平阳后方留守处”，并答应给粟裕伯伯率领的即将开赴前线与日本侵略军作战的新四军配发子弹五万发、军衣一千套。 |
- GPT 引用1: span=[218, 218], relevant=1

### 陈述 5
5. 指示协助李友邦筹建台湾义勇队，并成立台湾义勇队少年团。
- GPT support_score=1; **Rating: [[Fully supported]]**   **

**跨度** `[220-222]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> 1938年夏，父亲指示省委文委书记骆耕漠和共产党员张毕来协助李友邦筹建台湾义勇队。 1939年2月，台湾义勇队成立，李友邦任队长，张毕来任秘书。
> 又组建了台湾义勇队少年团，其成员都是台湾义勇队队员的子女，团长由李友邦兼任，刚从新四军教导队学习归来的夏云任指导员。
- au=1.6175589507838595e-06, eu=0.2867923676967621, au′=-0.0, eu′=0.03865716978907585
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.86842155456543]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 220 | 220 | 24.86842155456543 | 1938年夏，父亲指示省委文委书记骆耕漠和共产党员张毕来协助李友邦筹建台湾义勇队。 |
- GPT 引用1: span=[220, 222], relevant=1

### 陈述 6
6. 在台湾义勇队中秘密建立中共党支部。
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[224-224]`
- au=1.6175589507838595e-06, eu=0.2867923676967621, au′=-0.0, eu′=0.03865716978907585
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.86842155456543]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 224 | 224 | 24.86842155456543 | 随着义勇队人员的不断增加和党员的发展，在队里秘密建立了中共党支部，直接受浙江省委统战部副部长吴毓领导。 |
- GPT 引用1: span=[224, 224], relevant=1

### 陈述 7
7. 组织领导浙西南和闽东两个特委，建立党的组织，政权组织，指导分配土地。
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[201-201]`
- au=1.9246485862822738e-06, eu=0.3003951609134674, au′=-0.0, eu′=0.04117009416222572
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.289474487304688]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 201 | 201 | 23.289474487304688 | 1935年11月，面对国民党大举进攻，挺进师和闽东红军独立师（师长叶飞）成立闽浙边临时省委，父亲任书记，粟裕任组织部长，叶飞任宣传部长兼少共临时省委书记，下辖浙西南和闽东两个特委。 |
- GPT 引用1: span=[201, 201], relevant=1

### 陈述 8
8. 配合中央红军主力部队的行动，调动和牵制住大批敌人，彻底粉碎敌人对中央红军的“清剿”计划。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[197-197]`
- au=1.9009210063813953e-06, eu=0.2957197427749634, au′=-0.0, eu′=0.04029691964387894
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.8157901763916]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 197 | 197 | 23.8157901763916 | 配合中央红军主力部队的行动，调动和牵制住大批敌人，彻底粉碎敌人对中央红军的“清剿”计划。 |
- GPT 引用1: span=[197, 197], relevant=1

### 陈述 9
9. 在两次反“清剿”战斗中，前后牵制国民党兵力三四十个团，历时各达八个多月之久。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[203-203]`
- au=1.5606800616296823e-06, eu=0.2753622531890869, au′=-0.0, eu′=0.03660885989665985
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[26.3157901763916]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 203 | 203 | 26.3157901763916 | 挺进师在两次反“清剿”战斗中，前后共牵制国民党兵力三四十个团，前后历时各达八个多月之久，吸引牵制和消灭了相当数量的敌军，一定程度上配合策应了中央主力红军的长征，配合掩护了邻近红军游击区和斗争。 |
- GPT 引用1: span=[203, 203], relevant=1

### 陈述 10
10. 担任政委，领导浙西南人民全面建设革命根据地，开展武装斗争。
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[200-200]`
- au=1.7108611700678011e-06, eu=0.3052207827568054, au′=-0.0, eu′=0.04208194464445114
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[22.763158798217773]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 200 | 200 | 22.763158798217773 | 1935年7月，挺进师师部进驻具有特殊的政治经济地理有利条件的浙江遂昌县王村口，领导浙西南人民全面建设革命根据地，开展武装斗争。 |
- GPT 引用1: span=[200, 200], relevant=1

### 陈述 11
11. 在平阳集中，改番号为国民革命军闽浙边抗日游击总队，粟裕伯伯任总队长，父亲任政委。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[210-210]`
- au=1.6396279534092173e-06, eu=0.2911876440048218, au′=-0.0, eu′=0.039460018277168274
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.342105865478516]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 210 | 210 | 24.342105865478516 | 谈判后，挺进师在平阳集中，改番号为国民革命军闽浙边抗日游击总队，粟裕伯伯任总队长，父亲任政委。 |
- GPT 引用1: span=[210, 210], relevant=1

### 陈述 12
12. 编入新四军，开赴皖南北上抗日。
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[212-212]`
- au=1.6284955108858412e-06, eu=0.28897327184677124, au′=-0.0, eu′=0.03905446454882622
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.60526466369629]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 212 | 212 | 24.60526466369629 | 1938年3月，红军挺进师主力奉命由粟裕伯伯率领编入新四军，从平阳山门开赴皖南北上抗日。 |
- GPT 引用1: span=[212, 212], relevant=1

### 陈述 13
所以刘英在浙江工作期间，主要负责领导抗日救亡运动，开展统战工作，领导建设革命根据地，组织领导武装斗争等工作。
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

## multifieldqa_zh_idx282 (idx=282, multifieldqa_zh)
### 问题
云台山的门票价格是多少？

### 参考答案（数据集）
- ref1: 旺季门票价格150元/人，淡季门票60元/人。

### 模型全文输出
```
<statement>根据文档内容，云台山景区门票价格如下：<cite></cite></statement>

<statement>- 旺季价格：150元/人(旺季：3月1日到11月30日)<cite>[184-184]</cite></statement>

<statement>- 淡季价格：60元/人 (淡季：12月1日到2月底)<cite>[184-184]</cite></statement>

<statement>所以云台山景区门票价格是旺季150元/人，淡季60元/人。<cite></cite></statement>
```

文档句索引范围: sid ∈ [1, 186], 共 186 句。
引用指标: R=0.75, P=1.0, F1=0.8571428571428571
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=2221, completion_tokens=46
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据文档内容，云台山景区门票价格如下：
- GPT support_score=1; [[No]]   Need Citation: [[No]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 1
- 旺季价格：150元/人(旺季：3月1日到11月30日)
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[184-184]`
- au=1.5508946944464697e-06, eu=0.27338120341300964, au′=-0.0, eu′=0.03625953570008278
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[26.578948974609375]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 184 | 184 | 26.578948974609375 | 旺季价格：150元/人（旺季：3月1 日到11月30日） 淡季价格：60元/人 （淡季：12月1日到2月底） 免票范围：现役军人凭军官证或士兵证，残疾人凭残疾证，离休干部凭离休证，65周岁以上( 含65周岁）老人凭身份证/老年人优待证，身高低于1.4米或6周岁（含6周岁）以下儿童，一级英模、全国道 德模范凭有效证件，中国摄影家协会会员凭有效证件。 |
- GPT 引用1: span=[184, 184], relevant=1

### 陈述 2
- 淡季价格：60元/人 (淡季：12月1日到2月底)
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[184-184]`
- au=1.5807482895979774e-06, eu=0.2794116735458374, au′=-0.0, eu′=0.037328090518713
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.789474487304688]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 184 | 184 | 25.789474487304688 | 旺季价格：150元/人（旺季：3月1 日到11月30日） 淡季价格：60元/人 （淡季：12月1日到2月底） 免票范围：现役军人凭军官证或士兵证，残疾人凭残疾证，离休干部凭离休证，65周岁以上( 含65周岁）老人凭身份证/老年人优待证，身高低于1.4米或6周岁（含6周岁）以下儿童，一级英模、全国道 德模范凭有效证件，中国摄影家协会会员凭有效证件。 |
- GPT 引用1: span=[184, 184], relevant=1

### 陈述 3
所以云台山景区门票价格是旺季150元/人，淡季60元/人。
- GPT support_score=0; [[Yes]]   Need Citation: [[Yes]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

## multifieldqa_zh_idx283 (idx=283, multifieldqa_zh)
### 问题
阿诺新常见的不良反应有哪些？

### 参考答案（数据集）
- ref1: 阿诺新的常见不良反应包括潮热、关节痛和疲乏等。

### 模型全文输出
```
<statement>根据阿诺新的说明书，常见的不良反应包括：<cite></cite></statement>

<statement>- 潮热(22%)<cite>[23-23]</cite></statement>

<statement>- 关节痛(18%)<cite>[23-23]</cite></statement>

<statement>- 疲乏(16%)<cite>[23-23]</cite></statement>

<statement>- 恶心(12%)<cite>[25-25]</cite></statement>

<statement>- 出汗增多(8%)<cite>[71-71]</cite></statement>

<statement>- 食欲增加(6%)<cite>[71-71]</cite></statement>

<statement>- 腹泻(4%)<cite>[52-52]</cite></statement>

<statement>- 皮肤瘙痒(4%)<cite>[95-95]</cite></statement>

<statement>- 发热(4%)<cite>[74-74]</cite></statement>

<statement>- 肿瘤局部疼痛(8%)<cite>[74-74]</cite></statement>

<statement>- 虚弱(6%)<cite>[74-74]</cite></statement>

<statement>- 胃溃疡(0.7%)<cite>[54-54]</cite></statement>

<statement>- 骨质疏松症(4.6%)<cite>[51-51]</cite></statement>

<statement>- 骨软骨病(0.3%)<cite>[51-51]</cite></statement>

<statement>- 感染(2%)<cite>[75-75]</cite></statement>

<statement>- 心肌梗塞(0.6%)<cite>[48-48]</cite></statement>

<statement>- 心衰(0.4%)<cite>[49-49]</cite></statement>

<statement>- 血栓栓塞(2.0%)<cite>[56-56]</cite></statement>

<statement>- 子宫内膜增生(1.7%)<cite>[56-56]</cite></statement>

<statement>- 子宫息肉(2.4%)<cite>[56-56]</cite></statement>

<statement>- 肝功能异常(5.3%-15.0%)<cite>[80-87]</cite></statement>

<statement>- 肾功能异常(罕见)<cite></cite></statement>

<statement>- 骨密度降低(中度)<cite>[173-173]</cite></statement>

<statement>- 胆红素升高(5.3%-6.9%)<cite>[83-84][85-85]</cite></statement>

<statement>- 碱性磷酸酶升高(15.0%-13.7%)<cite>[86-87]</cite></statement>

<statement>- 肌酐升高(5.8%-5.5%)<cite>[88-89]</cite></statement>

<statement>- 淋巴细胞减少(20%)<cite>[30-30]</cite></statement>

<statement>- 白细胞减少(罕见)<cite>[29-29]</cite></statement>

<statement>- 血小板减少(罕见)<cite>[29-29]</cite></statement>

<statement>这些不良反应通常为轻至中度，严重不良反应很少见。<cite></cite></statement>
```

文档句索引范围: sid ∈ [1, 197], 共 197 句。
引用指标: R=0.8225806451612904, P=1.0, F1=0.9026548672566371
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=20315, completion_tokens=406
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据阿诺新的说明书，常见的不良反应包括：
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 1
- 潮热(22%)
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[23-23]`
- au=1.6802155187178869e-06, eu=0.2992125153541565, au′=-0.0, eu′=0.04094827175140381
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.421052932739258]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 23 | 23 | 23.421052932739258 | 最常报告的不良反应为潮热（22%）、关节痛（18%）和疲乏（16%）。 |
- GPT 引用1: span=[23, 23], relevant=1

### 陈述 2
- 关节痛(18%)
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[23-23]`
- au=1.5606800616296823e-06, eu=0.2753622531890869, au′=-0.0, eu′=0.03660885989665985
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[26.3157901763916]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 23 | 23 | 26.3157901763916 | 最常报告的不良反应为潮热（22%）、关节痛（18%）和疲乏（16%）。 |
- GPT 引用1: span=[23, 23], relevant=1

### 陈述 3
- 疲乏(16%)
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[23-23]`
- au=1.8040525446849642e-06, eu=0.2763635516166687, au′=-0.0, eu′=0.03678605705499649
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[26.18421173095703]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 23 | 23 | 26.18421173095703 | 最常报告的不良反应为潮热（22%）、关节痛（18%）和疲乏（16%）。 |
- GPT 引用1: span=[23, 23], relevant=1

### 陈述 4
- 恶心(12%)
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[25-25]`
- au=1.6567056491112453e-06, eu=0.2945735454559326, au′=-0.0, eu′=0.04008438438177109
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.947368621826172]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 25 | 25 | 23.947368621826172 | 最常报告的不良反应为潮热（14%）和恶心（12%）。 |
- GPT 引用1: span=[25, 25], relevant=1

### 陈述 5
- 出汗增多(8%)
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[71-71]`
- au=0.6805775761604309, eu=0.19740253686904907, au′=0.6805766820907593, eu′=0.04688463732600212
- top_logits 方差 (pvariance, 有限 logit): 0.35058822658265854
- top_logits 数值: `[20.921052932739258, 19.73684310913086]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 71 | 71 | 20.921052932739258 | 与药物有关或不能确定原因的不良事件中依西美坦组和醋酸甲地孕酮组的发生率分别为 ：潮热（13%对5%）、恶心（9%对5%）、乏力（8%对10%）、出汗增多（4%对8%）和食欲增加(3%对6%）。 |
| 2 | 23 | 23 | 19.73684310913086 | 最常报告的不良反应为潮热（22%）、关节痛（18%）和疲乏（16%）。 |
- GPT 引用1: span=[71, 71], relevant=1

### 陈述 6
- 食欲增加(6%)
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[71-71]`
- au=1.6802155187178869e-06, eu=0.2992125153541565, au′=-0.0, eu′=0.04094827175140381
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.421052932739258]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 71 | 71 | 23.421052932739258 | 与药物有关或不能确定原因的不良事件中依西美坦组和醋酸甲地孕酮组的发生率分别为 ：潮热（13%对5%）、恶心（9%对5%）、乏力（8%对10%）、出汗增多（4%对8%）和食欲增加(3%对6%）。 |
- GPT 引用1: span=[71, 71], relevant=1

### 陈述 7
- 腹泻(4%)
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[52-52]`
- au=2.0362715531518916e-06, eu=0.3220337927341461, au′=-0.0, eu′=0.045346058905124664
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[21.052631378173828]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 52 | 52 | 21.052631378173828 | 腹泻在依西美坦组发生更频繁（4.2%对2.2%）。 |
- GPT 引用1: span=[52, 52], relevant=1

### 陈述 8
- 皮肤瘙痒(4%)
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[95-95]`
- au=1.0812523365020752, eu=0.14843745529651642, au′=1.081251859664917, eu′=0.04969484359025955
- top_logits 方差 (pvariance, 有限 logit): 0.09618326494172733
- top_logits 数值: `[19.342105865478516, 19.342105865478516, 18.68421173095703]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 71 | 71 | 19.342105865478516 | 与药物有关或不能确定原因的不良事件中依西美坦组和醋酸甲地孕酮组的发生率分别为 ：潮热（13%对5%）、恶心（9%对5%）、乏力（8%对10%）、出汗增多（4%对8%）和食欲增加(3%对6%）。 |
| 2 | 73 | 73 | 19.342105865478516 | 在该项对照研究中，接受依西美坦25 mg/天治疗的患者中，任何原因引起的较少发生的不良事件（发生率2%-5%）有发热、疲乏、感觉异常、病理性骨折、支气管炎、鼻窦炎、皮疹、瘙痒、尿路感染和淋巴水肿。 |
| 3 | 95 | 95 | 18.68421173095703 | [u]皮肤和皮下组织异常[/u] 常见：荨麻疹，皮肤瘙痒 禁忌 禁用于已知对药物活性成分或任何辅料过敏者，以及绝经前和妊娠或哺乳期妇女。 |
- GPT 引用1: span=[95, 95], relevant=1

### 陈述 9
- 发热(4%)
- GPT support_score=0; Rating: [[No support]]

**跨度** `[74-74]`
- au=0.6803268194198608, eu=0.20266659557819366, au′=0.6803257465362549, eu′=0.04837682470679283
- top_logits 方差 (pvariance, 有限 logit): 0.21208462622689694
- top_logits 数值: `[20.131580352783203, 19.210527420043945]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 74 | 74 | 20.131580352783203 | 在全部临床研究中（N=1058），接受依西美坦25 mg/天治疗的患者(除对照研究外)中观察到的任何原因引起的发生率≥5%的其它不良事件有肿瘤局部疼痛（8%）、虚弱（6%）和发热（5%）。 |
| 2 | 73 | 73 | 19.210527420043945 | 在该项对照研究中，接受依西美坦25 mg/天治疗的患者中，任何原因引起的较少发生的不良事件（发生率2%-5%）有发热、疲乏、感觉异常、病理性骨折、支气管炎、鼻窦炎、皮疹、瘙痒、尿路感染和淋巴水肿。 |
- GPT 引用1: span=[74, 74], relevant=1

### 陈述 10
- 肿瘤局部疼痛(8%)
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[74-74]`
- au=1.5962523320922628e-06, eu=0.28252777457237244, au′=-0.0, eu′=0.03788633644580841
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.394737243652344]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 74 | 74 | 25.394737243652344 | 在全部临床研究中（N=1058），接受依西美坦25 mg/天治疗的患者(除对照研究外)中观察到的任何原因引起的发生率≥5%的其它不良事件有肿瘤局部疼痛（8%）、虚弱（6%）和发热（5%）。 |
- GPT 引用1: span=[74, 74], relevant=1

### 陈述 11
- 虚弱(6%)
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[74-74]`
- au=1.5910399042695644e-06, eu=0.28148138523101807, au′=-0.0, eu′=0.0376984067261219
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.526317596435547]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 74 | 74 | 25.526317596435547 | 在全部临床研究中（N=1058），接受依西美坦25 mg/天治疗的患者(除对照研究外)中观察到的任何原因引起的发生率≥5%的其它不良事件有肿瘤局部疼痛（8%）、虚弱（6%）和发热（5%）。 |
- GPT 引用1: span=[74, 74], relevant=1

### 陈述 12
- 胃溃疡(0.7%)
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[54-54]`
- au=1.6742565094318707e-06, eu=0.29803910851478577, au′=-0.0, eu′=0.04072882607579231
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.55263328552246]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 54 | 54 | 23.55263328552246 | 中位治疗持续时间约30个月和中位随访时间约52个月后，依西美坦组胃溃疡的发生率比他莫昔芬组稍高（0.7%对[0.1%）。 |
- GPT 引用1: span=[54, 54], relevant=1

### 陈述 13
- 骨质疏松症(4.6%)
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[51-51]`
- au=1.7108611700678011e-06, eu=0.3052207827568054, au′=-0.0, eu′=0.04208194464445114
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[22.763158798217773]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 51 | 51 | 22.763158798217773 | 在IES研究中，与他莫昔芬组相比，依西美坦组在肌肉骨骼系统和神经系统的不良事件发生率较高，包括以下发生率低于5%的事件[骨质疏松症（ 4.6%对2.8%）、骨软骨病和扳机指（0.3%对0 ）、感觉异常（2.6%对 0.9%）、腕管综合征（2.4%对0.2%）和神经疾病（0.6%对0.1%）]。 |
- GPT 引用1: span=[51, 51], relevant=1

### 陈述 14
- 骨软骨病(0.3%)
- GPT support_score=1; **Rating**: [[Fully supported]]   **

**跨度** `[51-51]`
- au=1.6068129298218992e-06, eu=0.28464409708976746, au′=-0.0, eu′=0.038267869502305984
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.131580352783203]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 51 | 51 | 25.131580352783203 | 在IES研究中，与他莫昔芬组相比，依西美坦组在肌肉骨骼系统和神经系统的不良事件发生率较高，包括以下发生率低于5%的事件[骨质疏松症（ 4.6%对2.8%）、骨软骨病和扳机指（0.3%对0 ）、感觉异常（2.6%对 0.9%）、腕管综合征（2.4%对0.2%）和神经疾病（0.6%对0.1%）]。 |
- GPT 引用1: span=[51, 51], relevant=1

### 陈述 15
- 感染(2%)
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[75-75]`
- au=1.95558504856308e-06, eu=0.3064514994621277, au′=-0.0, eu′=0.0423162542283535
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[22.631580352783203]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 75 | 75 | 22.631580352783203 | 上述临床研究的患者中，任何原因引起的发生率为2%-5%的不良事件有胸痛、感觉迟钝、意识错乱、消化不良、关节痛、背痛、骨骼痛、感染、上呼吸道感染、咽炎、鼻炎和脱发。 |
- GPT 引用1: span=[75, 75], relevant=1

### 陈述 16
- 心肌梗塞(0.6%)
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[48-48]`
- au=1.7496245163783897e-06, eu=0.31275713443756104, au′=-0.0, eu′=0.04352806136012077
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[21.973684310913086]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 48 | 48 | 21.973684310913086 | 心脏缺血事件的发生率（心肌梗塞、心绞痛和心肌缺血）依西美坦组为1.6%，他莫昔芬组为0.6%。 |
- GPT 引用1: span=[48, 48], relevant=1

### 陈述 17
- 心衰(0.4%)
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[49-49]`
- au=1.6742565094318707e-06, eu=0.29803910851478577, au′=-0.0, eu′=0.04072882607579231
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.55263328552246]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 49 | 49 | 23.55263328552246 | 心衰的发生率依西美坦组为0.4%，他莫昔芬组为0.3%。 |
- GPT 引用1: span=[49, 49], relevant=1

### 陈述 18
- 血栓栓塞(2.0%)
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[56-56]`
- au=1.698429286989267e-06, eu=0.30278876423835754, au′=-0.0, eu′=0.041621025651693344
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.026315689086914]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 56 | 56 | 23.026315689086914 | 与他莫昔芬相关的具有较高发生率的不良事件有肌肉痉挛（3.1%对1.5%）、血栓栓塞（2.0%对0.9%）、子宫内膜增生（1.7% 对0.6%）和子宫息肉（2.4%对0.4%）。 |
- GPT 引用1: span=[56, 56], relevant=1

### 陈述 19
- 子宫内膜增生(1.7%)
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[56-56]`
- au=1.5910399042695644e-06, eu=0.28148138523101807, au′=-0.0, eu′=0.0376984067261219
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.526317596435547]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 56 | 56 | 25.526317596435547 | 与他莫昔芬相关的具有较高发生率的不良事件有肌肉痉挛（3.1%对1.5%）、血栓栓塞（2.0%对0.9%）、子宫内膜增生（1.7% 对0.6%）和子宫息肉（2.4%对0.4%）。 |
- GPT 引用1: span=[56, 56], relevant=1

### 陈述 20
- 子宫息肉(2.4%)
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[56-56]`
- au=1.6068129298218992e-06, eu=0.28464409708976746, au′=-0.0, eu′=0.038267869502305984
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.131580352783203]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 56 | 56 | 25.131580352783203 | 与他莫昔芬相关的具有较高发生率的不良事件有肌肉痉挛（3.1%对1.5%）、血栓栓塞（2.0%对0.9%）、子宫内膜增生（1.7% 对0.6%）和子宫息肉（2.4%对0.4%）。 |
- GPT 引用1: span=[56, 56], relevant=1

### 陈述 21
- 肝功能异常(5.3%-15.0%)
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[80-87]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> 在晚期乳腺癌患者的治疗中血浆AST、ALT、碱性磷酸酶和γ-谷氨酰转移酶水平大于正常范围上限的5倍（例如≥CTC
> 3级）的报告罕见，但是大部分出现于有潜在肝和/或骨转移的患者中。 在晚期乳腺癌患者的对照研究中无肝转移情况下发生CTC 3或4级γ-
> 谷氨酰转移酶水平升高的发生率在依西美坦组为2.7%，醋酸甲地孕酮组为1.8%。 在早期乳腺癌患者中，依西美坦组比他莫昔芬组或安慰剂组更易发生胆红素、碱性磷酸酶和肌酐水平的升高。
> 在IES研究中与治疗相关的胆红素水平的升高（任何CTC等级）的发生率，依西美坦组为5.3%，他莫昔芬组为0.8%； 在027研究中依西美坦组为6.9%，安慰剂组没有发生。
> CTC 3或4级的胆红素水平升高的发生率，依西美坦组为0.9%，他莫昔芬组为0.1%。
> 任何CTC等级的碱性磷酸酶水平升高的发生率，在IES研究中依西美坦组为15.0%，他莫昔芬组为2.6% ； 在027研究中依西美坦组为13.7%，安慰剂组为6.9%。
- au=1.0816240310668945, eu=0.14531543850898743, au′=1.0816236734390259, eu′=0.04853128641843796
- top_logits 方差 (pvariance, 有限 logit): 0.13850443691222203
- top_logits 数值: `[20.131580352783203, 19.342105865478516, 19.342105865478516]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 83 | 83 | 20.131580352783203 | 在IES研究中与治疗相关的胆红素水平的升高（任何CTC等级）的发生率，依西美坦组为5.3%，他莫昔芬组为0.8%； |
| 2 | 86 | 86 | 19.342105865478516 | 任何CTC等级的碱性磷酸酶水平升高的发生率，在IES研究中依西美坦组为15.0%，他莫昔芬组为2.6% ； |
| 3 | 80 | 80 | 19.342105865478516 | 在晚期乳腺癌患者的治疗中血浆AST、ALT、碱性磷酸酶和γ-谷氨酰转移酶水平大于正常范围上限的5倍（例如≥CTC 3级）的报告罕见，但是大部分出现于有潜在肝和/或骨转移的患者中。 |
- GPT 引用1: span=[80, 87], relevant=1

### 陈述 22
- 肾功能异常(罕见)
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 23
- 骨密度降低(中度)
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[173-173]`
- au=0.6819287538528442, eu=0.18491479754447937, au′=0.6819279193878174, eu′=0.04340376332402229
- top_logits 方差 (pvariance, 有限 logit): 0.004328347309638048
- top_logits 数值: `[22.10526466369629, 21.973684310913086]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 173 | 173 | 22.10526466369629 | 关于骨的亚组研究分析结果表明，采用2-3年的他莫昔芬辅助治疗后序贯应用依西美坦治疗，患者的骨密度出现中度降低。 |
| 2 | 90 | 90 | 21.973684310913086 | 依西美坦组观察到与治疗时间相关的骨密度下降。 |
- GPT 引用1: span=[173, 173], relevant=1

### 陈述 24
- 胆红素升高(5.3%-6.9%)
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[83-84]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> 在IES研究中与治疗相关的胆红素水平的升高（任何CTC等级）的发生率，依西美坦组为5.3%，他莫昔芬组为0.8%；
>
> 在027研究中依西美坦组为6.9%，安慰剂组没有发生。
- au=1.6567056491112453e-06, eu=0.2945735454559326, au′=-0.0, eu′=0.04008438438177109
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.947368621826172]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 83 | 83 | 23.947368621826172 | 在IES研究中与治疗相关的胆红素水平的升高（任何CTC等级）的发生率，依西美坦组为5.3%，他莫昔芬组为0.8%； |

**跨度** `[85-85]`
- au=1.6567056491112453e-06, eu=0.2945735454559326, au′=-0.0, eu′=0.04008438438177109
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.947368621826172]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 85 | 85 | 23.947368621826172 | CTC 3或4级的胆红素水平升高的发生率，依西美坦组为0.9%，他莫昔芬组为0.1%。 |
- GPT 引用1: span=[83, 85], relevant=1

### 陈述 25
- 碱性磷酸酶升高(15.0%-13.7%)
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[86-87]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> 任何CTC等级的碱性磷酸酶水平升高的发生率，在IES研究中依西美坦组为15.0%，他莫昔芬组为2.6% ； 在027研究中依西美坦组为13.7%，安慰剂组为6.9%。
- au=1.6284955108858412e-06, eu=0.28897327184677124, au′=-0.0, eu′=0.03905446454882622
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.60526466369629]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 86 | 86 | 24.60526466369629 | 任何CTC等级的碱性磷酸酶水平升高的发生率，在IES研究中依西美坦组为15.0%，他莫昔芬组为2.6% ； |
- GPT 引用1: span=[86, 87], relevant=1

### 陈述 26
- 肌酐升高(5.8%-5.5%)
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[88-89]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> 肌酐水平升高的发生率，在IES研究中，依西美坦组为5.8%，他莫昔芬组为4.3%； 在027研究中依西美坦组为5.5%，安慰剂组没有发生。
- au=1.6567056491112453e-06, eu=0.2945735454559326, au′=-0.0, eu′=0.04008438438177109
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.947368621826172]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 88 | 88 | 23.947368621826172 | 肌酐水平升高的发生率，在IES研究中，依西美坦组为5.8%，他莫昔芬组为4.3%； |
- GPT 引用1: span=[88, 89], relevant=1

### 陈述 27
- 淋巴细胞减少(20%)
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[30-30]`
- au=1.698429286989267e-06, eu=0.30278876423835754, au′=-0.0, eu′=0.041621025651693344
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.026315689086914]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 30 | 30 | 23.026315689086914 | 约有20%的患者接受本品后出现偶发的淋巴细胞减少，特别是既往有淋巴细胞减少症的患者； |
- GPT 引用1: span=[30, 30], relevant=1

### 陈述 28
- 白细胞减少(罕见)
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[29-29]`
- au=1.7108611700678011e-06, eu=0.3052207827568054, au′=-0.0, eu′=0.04208194464445114
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[22.763158798217773]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 29 | 29 | 22.763158798217773 | [u]血液和淋巴系统异常 [/u] 依西美坦治疗晚期乳腺癌患者罕见有血小板减少或白细胞减少的报告。 |
- GPT 引用1: span=[29, 29], relevant=1

### 陈述 29
- 血小板减少(罕见)
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[29-29]`
- au=1.6175589507838595e-06, eu=0.2867923676967621, au′=-0.0, eu′=0.03865716978907585
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.86842155456543]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 29 | 29 | 24.86842155456543 | [u]血液和淋巴系统异常 [/u] 依西美坦治疗晚期乳腺癌患者罕见有血小板减少或白细胞减少的报告。 |
- GPT 引用1: span=[29, 29], relevant=1

### 陈述 30
这些不良反应通常为轻至中度，严重不良反应很少见。
- GPT support_score=1; [[No]]   Need Citation: [[No]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

## multifieldqa_zh_idx284 (idx=284, multifieldqa_zh)
### 问题
娜塔莉·波特曼获得了哪届奥斯卡金像奖最佳女主角？

### 参考答案（数据集）
- ref1: 83届和89届。

### 模型全文输出
```
<statement>根据上文，娜塔莉·波特曼在2011年凭借电影《黑天鹅》获得了第83届奥斯卡金像奖最佳女主角。<cite>[12-12]</cite></statement>
```

文档句索引范围: sid ∈ [1, 82], 共 82 句。
引用指标: R=1.0, P=1.0, F1=1.0
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=686, completion_tokens=13
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据上文，娜塔莉·波特曼在2011年凭借电影《黑天鹅》获得了第83届奥斯卡金像奖最佳女主角。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[12-12]`
- au=1.0827006101608276, eu=0.13669061660766602, au′=1.0827001333236694, eu′=0.045346055179834366
- top_logits 方差 (pvariance, 有限 logit): 0.21929824561407762
- top_logits 数值: `[21.710527420043945, 20.789474487304688, 20.657896041870117]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 12 | 12 | 21.710527420043945 | 同年凭借电影《黑天鹅》，获得第83届奥斯卡金像奖最佳女主角奖 、第68届金球奖最佳剧情片女主角奖等奖项 。 |
| 2 | 61 | 61 | 20.789474487304688 | 同年，娜塔莉·波特曼凭借电影《黑天鹅》，获得第15届美国广播影评人协会奖最佳女主角的提名 [46] 。 |
| 3 | 11 | 11 | 20.657896041870117 | 2011年主演科幻电影《雷神》 ； |
- GPT 引用1: span=[12, 12], relevant=1

## multifieldqa_zh_idx285 (idx=285, multifieldqa_zh)
### 问题
国内销售的工业产品必须提供哪种语言的使用说明书？

### 参考答案（数据集）
- ref1: 中文。

### 模型全文输出
```
<statement>根据国家标准《工业产品使用说明书总则》,国内销售的工业产品必须提供中文使用说明书。<cite>[55-55]</cite></statement>

<statement>相关原文：<cite></cite></statement>

<statement>4.1.1国内销售的工业产品必须提供中文使用说明书。<cite>[55-55]</cite></statement>

<statement>所以国内销售的工业产品必须提供中文使用说明书。<cite></cite></statement>
```

文档句索引范围: sid ∈ [1, 173], 共 173 句。
引用指标: R=0.75, P=1.0, F1=0.8571428571428571
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=1823, completion_tokens=46
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据国家标准《工业产品使用说明书总则》,国内销售的工业产品必须提供中文使用说明书。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[55-55]`
- au=1.6567056491112453e-06, eu=0.2945735454559326, au′=-0.0, eu′=0.04008438438177109
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.947368621826172]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 55 | 55 | 23.947368621826172 | １犌犅／犜９９６９—２００８ ４\u3000编制要求４．１\u3000文字、语言４．１．１\u3000国内销售的工业产品必须提供中文使用说明书。 |
- GPT 引用1: span=[55, 55], relevant=1

### 陈述 1
相关原文：
- GPT support_score=1; [[No]]   Need Citation: [[No]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 2
4.1.1国内销售的工业产品必须提供中文使用说明书。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[55-55]`
- au=1.5606800616296823e-06, eu=0.2753622531890869, au′=-0.0, eu′=0.03660885989665985
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[26.3157901763916]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 55 | 55 | 26.3157901763916 | １犌犅／犜９９６９—２００８ ４\u3000编制要求４．１\u3000文字、语言４．１．１\u3000国内销售的工业产品必须提供中文使用说明书。 |
- GPT 引用1: span=[55, 55], relevant=1

### 陈述 3
所以国内销售的工业产品必须提供中文使用说明书。
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

## multifieldqa_zh_idx286 (idx=286, multifieldqa_zh)
### 问题
该产品在什么温度开始热分解？

### 参考答案（数据集）
- ref1: 220 °C。

### 模型全文输出
```
<statement>根据安全技术说明书中的信息，该产品在220°C开始热分解。<cite>[95-95]</cite></statement>

<statement>相关段落摘录如下：<cite></cite></statement>

<statement>"热分解:220°C, 650 kJ/kg (DSC (OECD 113))（起始温度）"<cite>[95-95]</cite></statement>

<statement>所以该产品在220°C开始热分解。<cite></cite></statement>
```

文档句索引范围: sid ∈ [1, 166], 共 166 句。
引用指标: R=0.75, P=1.0, F1=0.8571428571428571
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=1950, completion_tokens=46
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据安全技术说明书中的信息，该产品在220°C开始热分解。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[95-95]`
- au=0.6816002726554871, eu=0.18719208240509033, au′=0.6815991997718811, eu′=0.04403243958950043
- top_logits 方差 (pvariance, 有限 logit): 0.1558174915262498
- top_logits 数值: `[22.10526466369629, 21.3157901763916]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 95 | 95 | 22.10526466369629 | 稳定性和反应性需避免的情况:参见化学品安全技术说明书 第7节 - 操作处置与储存.热分解:热分解:220 °C, 650 kJ/kg (DSC (OECD 113))（起始温度）根据联合国运输规定之分类4.1，该物质不是自分解性物质。 |
| 2 | 91 | 91 | 21.3157901763916 | 大约 5 - 8(1 %(m), 20 °C)（作为乳化剂）大约 -20 °C溶剂的资料大约 178 - 209 °C溶剂的资料 巴斯夫 安全技术说明书日期 / 本次修订: 04.07.2023产品: Stomp New 330 EC版本: 4.0页: 8/14(30448922/SDS_CPA_CN/ZH)印刷日期 05.07.2023闪点:蒸发速率:可燃性 (固体/气体):爆炸下限:爆炸上限:燃烧温度:热分解:爆炸危险:促燃性:蒸气压:密度:相对蒸气密度（空气）:大约 61 °C(德国工业标准 51755)不适用不适用根据我们对该产品的经验和对其成分 的知识, 只要我们适当地并且在其应 用的范围内使用该产品, 不会有任何 的危险.根据我们对该产品的经验和对其成分 的知识, 只要我们适当地并且在其应 用的范围内使用该产品, 不会有任何 的危险.375 °C220 °C , 650 kJ/kg（起始温度） 根据联合国运输规定之分类4.1，该物 质不是自分解性物质。 |
- GPT 引用1: span=[95, 95], relevant=1

### 陈述 1
相关段落摘录如下：
- GPT support_score=1; [[No]]   Need Citation: [[No]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 2
"热分解:220°C, 650 kJ/kg (DSC (OECD 113))（起始温度）"
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[95-95]`
- au=1.6567056491112453e-06, eu=0.2945735454559326, au′=-0.0, eu′=0.04008438438177109
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.947368621826172]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 95 | 95 | 23.947368621826172 | 稳定性和反应性需避免的情况:参见化学品安全技术说明书 第7节 - 操作处置与储存.热分解:热分解:220 °C, 650 kJ/kg (DSC (OECD 113))（起始温度）根据联合国运输规定之分类4.1，该物质不是自分解性物质。 |
- GPT 引用1: span=[95, 95], relevant=1

### 陈述 3
所以该产品在220°C开始热分解。
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

## multifieldqa_zh_idx287 (idx=287, multifieldqa_zh)
### 问题
青岛2017年实际利用外资多少美元？

### 参考答案（数据集）
- ref1: 77.3亿美元。

### 模型全文输出
```
<statement>根据报告内容，青岛2017年实际利用外资77.3亿美元。<cite>[56-56]</cite></statement>

<statement>报告原文：<cite></cite></statement>

<statement>“实际利用外资77.3亿美元，增长10%左右，总量占全省45%左右。”<cite>[56-56]</cite></statement>
```

文档句索引范围: sid ∈ [1, 309], 共 309 句。
引用指标: R=1.0, P=1.0, F1=1.0
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=1396, completion_tokens=36
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据报告内容，青岛2017年实际利用外资77.3亿美元。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[56-56]`
- au=1.930719463416608e-06, eu=0.30158722400665283, au′=-0.0, eu′=0.04139433056116104
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.157896041870117]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 56 | 56 | 23.157896041870117 | 实际利用外资77.3亿美元，增长10%左右，总量占全省45%左右。 |
- GPT 引用1: span=[56, 56], relevant=1

### 陈述 1
报告原文：
- GPT support_score=1; [[No]]   Need Citation: [[No]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 2
“实际利用外资77.3亿美元，增长10%左右，总量占全省45%左右。”
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[56-56]`
- au=1.5910399042695644e-06, eu=0.28148138523101807, au′=-0.0, eu′=0.0376984067261219
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.526317596435547]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 56 | 56 | 25.526317596435547 | 实际利用外资77.3亿美元，增长10%左右，总量占全省45%左右。 |
- GPT 引用1: span=[56, 56], relevant=1

## multifieldqa_zh_idx288 (idx=288, multifieldqa_zh)
### 问题
成人应该如何服用阿诺新？

### 参考答案（数据集）
- ref1: 每日一次，每次一片，建议餐后服用。

### 模型全文输出
```
<statement>根据说明书，成人和老年患者推荐剂量为25mg，每日一次，每次一片，建议餐后服用。<cite>[9-9][14-14]</cite></statement>

<statement>早期乳腺癌患者应持续服用本品，直至完成5年的联合序贯辅助内分泌治疗(即他莫昔芬序贯依西美坦)。或服用本品直至出现肿瘤复发。<cite>[10-11][15-15]</cite></statement>

<statement>晚期乳腺癌患者应持续服用本品直至出现肿瘤进展。<cite>[12-12]</cite></statement>

<statement>肝功能或肾功能不全的患者无需进行调整剂量。<cite>[13-13]</cite></statement>

<statement>患者同时接受细胞色素P-450(CYP) 3A4诱导剂，如利福平、苯妥英时，本品的推荐剂量为50 mg，每日一次，餐后服用。<cite>[17-17]</cite></statement>
```

文档句索引范围: sid ∈ [1, 196], 共 196 句。
引用指标: R=1.0, P=1.0, F1=1.0
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=3879, completion_tokens=77
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据说明书，成人和老年患者推荐剂量为25mg，每日一次，每次一片，建议餐后服用。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[9-9]`
- au=1.6742565094318707e-06, eu=0.29803910851478577, au′=-0.0, eu′=0.04072882607579231
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.55263328552246]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 9 | 9 | 23.55263328552246 | 用法用量 成人和老年患者 ：推荐剂量为25 mg，每日一次，每次一片，建议餐后服用。 |

**跨度** `[14-14]`
- au=1.9619515114754904e-06, eu=0.30769219994544983, au′=-0.0, eu′=0.04255318641662598
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[22.5]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 14 | 14 | 22.5 | 治疗早期和晚期乳腺癌患者的推荐剂量为25 mg，每日一次，每次一片，建议餐后服用。 |
- GPT 引用1: span=[9, 9], relevant=1
- GPT 引用2: span=[14, 14], relevant=1

### 陈述 1
早期乳腺癌患者应持续服用本品，直至完成5年的联合序贯辅助内分泌治疗(即他莫昔芬序贯依西美坦)。或服用本品直至出现肿瘤复发。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[10-11]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> 早期乳腺癌患者应持续服用本品，直至完成5年的联合序贯辅助内分泌治疗（即他莫昔芬序贯依西美坦） 。 或服用本品直至出现肿瘤复发。
- au=1.6567056491112453e-06, eu=0.2945735454559326, au′=-0.0, eu′=0.04008438438177109
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.947368621826172]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 10 | 10 | 23.947368621826172 | 早期乳腺癌患者应持续服用本品，直至完成5年的联合序贯辅助内分泌治疗（即他莫昔芬序贯依西美坦） 。 |

**跨度** `[15-15]`
- au=1.5910399042695644e-06, eu=0.28148138523101807, au′=-0.0, eu′=0.0376984067261219
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.526317596435547]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 15 | 15 | 25.526317596435547 | 早期乳腺癌患者在接受2-3年的他莫昔芬治疗后，在未出现复发或对侧乳腺癌的情况下，应使用本品继续治疗，直至完成5年的联合序贯辅助内分泌治疗（即他莫昔芬序贯依西美坦）。 |
- GPT 引用1: span=[10, 11], relevant=1
- GPT 引用2: span=[15, 15], relevant=1

### 陈述 2
晚期乳腺癌患者应持续服用本品直至出现肿瘤进展。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[12-12]`
- au=1.9009210063813953e-06, eu=0.2957197427749634, au′=-0.0, eu′=0.04029691964387894
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.8157901763916]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 12 | 12 | 23.8157901763916 | 晚期乳腺癌患者应持续服用本品直至出现肿瘤进展。 |
- GPT 引用1: span=[12, 12], relevant=1

### 陈述 3
肝功能或肾功能不全的患者无需进行调整剂量。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[13-13]`
- au=1.861421651483397e-06, eu=0.28787869215011597, au′=-0.0, eu′=0.038854800164699554
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.73684310913086]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 13 | 13 | 24.73684310913086 | 肝功能或肾功能不全的患者无需进行调整剂量。 |
- GPT 引用1: span=[13, 13], relevant=1

### 陈述 4
患者同时接受细胞色素P-450(CYP) 3A4诱导剂，如利福平、苯妥英时，本品的推荐剂量为50 mg，每日一次，餐后服用。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[17-17]`
- au=1.5858721553740907e-06, eu=0.2804427146911621, au′=-0.0, eu′=0.03751233592629433
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.657896041870117]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 17 | 17 | 25.657896041870117 | 患者同时接受细胞色素P-450(CYP) 3A4诱导剂，如利福平、苯妥英时，本品的推荐剂量为50 mg，每日一次，餐后服用。 |
- GPT 引用1: span=[17, 17], relevant=1

## multifieldqa_zh_idx289 (idx=289, multifieldqa_zh)
### 问题
梧州市环城高速公路的工程复杂性如何？

### 参考答案（数据集）
- ref1: 桥梁、隧道在工程中所占比例较高，工程复杂。

### 模型全文输出
```
<statement>根据文档内容，可以看出梧州市环城高速公路的工程复杂性主要体现在以下几个方面：<cite></cite></statement>

<statement>1. 环城高速公路桥梁、隧道所占比例较高，工程复杂。<cite>[6-6]</cite></statement>

<statement>2. 全线特大桥4座、长隧道1座、中隧道2座、短隧道4座，工程数量较多，技术复杂度较高。<cite>[16-16][29-29][36-36]</cite></statement>

<statement>3. 主线采用高速公路标准，路基宽度26米，设计行车速度为每小时100公里，双向四车道；连接线采用一级公路标准，设计速度60公里/小时，路基宽23米。工程标准较高，技术复杂度较高。<cite>[2-2][4-4][21-21]</cite></statement>

<statement>4. 环境影响较大，需要采取多项环境保护措施。<cite>[76-80]</cite></statement>

<statement>5. 工程数量较多，包括桥梁、隧道、互通立交、分离式立交、人行天桥、通道、服务区、收费站等，管理复杂度较高。<cite>[16-16][24-24][29-31]</cite></statement>

<statement>6. 预计项目投资估算总金额(含连接线)为38.3641亿元，项目规模较大，管理复杂度较高。<cite>[24-24]</cite></statement>

<statement>综上所述，梧州市环城高速公路的工程复杂性主要体现在桥梁隧道数量多、工程标准高、环境影响大、工程数量多、项目规模大等方面。<cite></cite></statement>
```

文档句索引范围: sid ∈ [1, 81], 共 81 句。
引用指标: R=0.625, P=0.75, F1=0.6818181818181818
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=7860, completion_tokens=148
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据文档内容，可以看出梧州市环城高速公路的工程复杂性主要体现在以下几个方面：
- GPT support_score=1; [[No]]   Need Citation: [[No]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 1
1. 环城高速公路桥梁、隧道所占比例较高，工程复杂。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[6-6]`
- au=1.650961507948523e-06, eu=0.2934361696243286, au′=-0.0, eu′=0.03987407684326172
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.078948974609375]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 6 | 6 | 24.078948974609375 | 环城高速公路桥梁、隧道所占比例较高，工程复杂。 |
- GPT 引用1: span=[6, 6], relevant=1

### 陈述 2
2. 全线特大桥4座、长隧道1座、中隧道2座、短隧道4座，工程数量较多，技术复杂度较高。
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[16-16]`
- au=1.698429286989267e-06, eu=0.30278876423835754, au′=-0.0, eu′=0.041621025651693344
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.026315689086914]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 16 | 16 | 23.026315689086914 | 全线特大桥4座共5508.5米，大桥20座共6049米，中桥4座共221米，长隧道1座1265米，中隧道2座共1707米，短隧道4座共1167米，互通立交6处，分离式立交5座608米，人行天桥1座71米，通道55道，服务区1处，收费站5处。 |

**跨度** `[29-29]`
- au=0.6815296411514282, eu=0.18811875581741333, au′=0.6815286874771118, eu′=0.04428904131054878
- top_logits 方差 (pvariance, 有限 logit): 0.1558167386256173
- top_logits 数值: `[21.973684310913086, 21.18421173095703]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 29 | 29 | 21.973684310913086 | 里程及主要工程数量：路线长，21.7km，特大桥1122米/1座, 技术复杂大桥871米/1座，大桥2605米/8座，中桥121米/2座； |
| 2 | 36 | 36 | 21.18421173095703 | 里程及主要工程数量：路线长，主线20.314km，一级公路标准连接线6.457km，特大桥1452米/1座, 技术复杂大桥2063.5米/1座，大桥3444米/12座，中桥100米/1座； |

**跨度** `[36-36]`
- au=1.6452690942969639e-06, eu=0.29230761528015137, au′=-0.0, eu′=0.03966596722602844
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.210527420043945]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 36 | 36 | 24.210527420043945 | 里程及主要工程数量：路线长，主线20.314km，一级公路标准连接线6.457km，特大桥1452米/1座, 技术复杂大桥2063.5米/1座，大桥3444米/12座，中桥100米/1座； |
- GPT 引用1: span=[16, 16], relevant=1
- GPT 引用2: span=[29, 29], relevant=0
- GPT 引用3: span=[36, 36], relevant=0

### 陈述 3
3. 主线采用高速公路标准，路基宽度26米，设计行车速度为每小时100公里，双向四车道；连接线采用一级公路标准，设计速度60公里/小时，路基宽23米。工程标准较高，技术复杂度较高。
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[2-2]`
- au=0.6822757124900818, eu=0.17882347106933594, au′=0.6822749376296997, eu′=0.04173530638217926
- top_logits 方差 (pvariance, 有限 logit): 0.10820617305944324
- top_logits 数值: `[23.289474487304688, 22.631580352783203]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 4 | 4 | 23.289474487304688 | 主线采用高速公路标准，路基宽度26米，设计行车速度为每小时100公里，双向四车道（跨西江特大桥双向六车道，预留两车道）； |
| 2 | 2 | 2 | 22.631580352783203 | 主线、省际通道按双向四车道高速公路标准建设，设计速度100公里/小时。 |

**跨度** `[4-4]`
- au=1.650961507948523e-06, eu=0.2934361696243286, au′=-0.0, eu′=0.03987407684326172
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.078948974609375]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 4 | 4 | 24.078948974609375 | 主线采用高速公路标准，路基宽度26米，设计行车速度为每小时100公里，双向四车道（跨西江特大桥双向六车道，预留两车道）； |

**跨度** `[21-21]`
- au=0.6820899248123169, eu=0.1822541505098343, au′=0.68208909034729, eu′=0.04267264902591705
- top_logits 方差 (pvariance, 有限 logit): 0.03895437288156245
- top_logits 数值: `[22.631580352783203, 22.23684310913086]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 21 | 21 | 22.631580352783203 | 采用一级公路标准； |
| 2 | 17 | 17 | 22.23684310913086 | 主线方案 采用高速公路标准，建设里程42.014km，设计速度100km/h，路基宽26m； |
- GPT 引用1: span=[2, 2], relevant=1
- GPT 引用2: span=[4, 4], relevant=1
- GPT 引用3: span=[21, 21], relevant=0

### 陈述 4
4. 环境影响较大，需要采取多项环境保护措施。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[76-80]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> 工程可能造成的环境影响与防治措施 时期 环境影响 拟采取的防治措施 施工期 土地占用、植被破坏 保护表层耕作土、合理制定施工区、及时恢复 粉尘污染 物料堆放时加盖蓬布；
> 施工场地定期洒水； 敏感点300m范围内不设置拌合站 工程废水 对工程施工废水进行收集处理，不随意排放 施工噪声 采用低噪声机械；
> 敏感点附近尽量避免高噪声机械夜间（22:00-次日6:00）施工 固体废物 集中堆放，当地环卫部门定期清运或外运至当地固体废物处理场 运营期 交通噪声 加强公路交通管理；
> 采取声屏障、绿化、隔声窗等降噪措施 服务区污水 经处理达标后回用或排放 服务区生活垃圾 集中堆放，并联系当地环卫部门定期清运
> 本次公示本着公开、公正、客观、真实的基本原则，征求工程相关个人和单位的意见。
- au=1.6175589507838595e-06, eu=0.2867923676967621, au′=-0.0, eu′=0.03865716978907585
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.86842155456543]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 76 | 76 | 24.86842155456543 | 工程可能造成的环境影响与防治措施 时期 环境影响 拟采取的防治措施 施工期 土地占用、植被破坏 保护表层耕作土、合理制定施工区、及时恢复 粉尘污染 物料堆放时加盖蓬布； |
- GPT 引用1: span=[76, 80], relevant=1

### 陈述 5
5. 工程数量较多，包括桥梁、隧道、互通立交、分离式立交、人行天桥、通道、服务区、收费站等，管理复杂度较高。
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[16-16]`
- au=1.9067709899900365e-06, eu=0.29687488079071045, au′=-0.0, eu′=0.04051172360777855
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.68421173095703]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 16 | 16 | 23.68421173095703 | 全线特大桥4座共5508.5米，大桥20座共6049米，中桥4座共221米，长隧道1座1265米，中隧道2座共1707米，短隧道4座共1167米，互通立交6处，分离式立交5座608米，人行天桥1座71米，通道55道，服务区1处，收费站5处。 |

**跨度** `[24-24]`
- au=0.6816425323486328, eu=0.1840193271636963, au′=0.681641697883606, eu′=0.043157294392585754
- top_logits 方差 (pvariance, 有限 logit): 0.35058822658265854
- top_logits 数值: `[22.763158798217773, 21.578948974609375]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 30 | 30 | 22.763158798217773 | 隧道1052米/2座,涵洞52道，通道36道，互通立交2处，分离立交4处。 |
| 2 | 24 | 24 | 21.578948974609375 | 项目投资估算总金额（含连接线）为38.3641亿元，设互通立交6座，服务区1处，收费站5处，桥梁28座，隧道7座； |

**跨度** `[29-31]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> 里程及主要工程数量：路线长，21.7km，特大桥1122米/1座, 技术复杂大桥871米/1座，大桥2605米/8座，中桥121米/2座；
> 隧道1052米/2座,涵洞52道，通道36道，互通立交2处，分离立交4处。 人行天桥71米/1座。
- au=0.6814299821853638, eu=0.18673212826251984, au′=0.6814291477203369, eu′=0.04390525445342064
- top_logits 方差 (pvariance, 有限 logit): 0.35058822658265854
- top_logits 数值: `[22.36842155456543, 21.18421173095703]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 30 | 30 | 22.36842155456543 | 隧道1052米/2座,涵洞52道，通道36道，互通立交2处，分离立交4处。 |
| 2 | 29 | 29 | 21.18421173095703 | 里程及主要工程数量：路线长，21.7km，特大桥1122米/1座, 技术复杂大桥871米/1座，大桥2605米/8座，中桥121米/2座； |
- GPT 引用1: span=[16, 16], relevant=1
- GPT 引用2: span=[24, 24], relevant=1
- GPT 引用3: span=[29, 31], relevant=1

### 陈述 6
6. 预计项目投资估算总金额(含连接线)为38.3641亿元，项目规模较大，管理复杂度较高。
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[24-24]`
- au=1.9246485862822738e-06, eu=0.3003951609134674, au′=-0.0, eu′=0.04117009416222572
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.289474487304688]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 24 | 24 | 23.289474487304688 | 项目投资估算总金额（含连接线）为38.3641亿元，设互通立交6座，服务区1处，收费站5处，桥梁28座，隧道7座； |
- GPT 引用1: span=[24, 24], relevant=1

### 陈述 7
综上所述，梧州市环城高速公路的工程复杂性主要体现在桥梁隧道数量多、工程标准高、环境影响大、工程数量多、项目规模大等方面。
- GPT support_score=0; [[Yes]]   Need Citation: [[Yes]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

## multifieldqa_zh_idx290 (idx=290, multifieldqa_zh)
### 问题
原告共支付了多少房款？

### 参考答案（数据集）
- ref1: 419001元。

### 模型全文输出
```
<statement>根据判决书内容，原告闫晓霞共支付了419001元房款。<cite></cite></statement>

<statement>相关证据是：<cite></cite></statement>

<statement>1. 判决书第4页提到：“原告2015年7月17日交定金10000元，2015年7月27日交房款140000元，2015年8月27日交房款50000元，2015年11月9日交房款9001元，2016年2月5日交商业贷款210000元，共计交房款419001元。”<cite>[32-32]</cite></statement>

<statement>2. 判决书第12页提到：“原告于2015年7月17日交定金10000元，2015年7月27日交房款140000元，2015年8月27日交房款50000元，2015年11月9日交房款9001元，2016年2月5日交商业贷款210000元，共计交房款419001元。”<cite>[32-32]</cite></statement>

<statement>所以，原告闫晓霞共支付了419001元房款。<cite></cite></statement>
```

文档句索引范围: sid ∈ [1, 58], 共 58 句。
引用指标: R=0.6, P=1.0, F1=0.7499999999999999
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=2951, completion_tokens=56
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据判决书内容，原告闫晓霞共支付了419001元房款。
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 1
相关证据是：
- GPT support_score=1; [[No]]   Need Citation: [[No]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 2
1. 判决书第4页提到：“原告2015年7月17日交定金10000元，2015年7月27日交房款140000元，2015年8月27日交房款50000元，2015年11月9日交房款9001元，2016年2月5日交商业贷款210000元，共计交房款419001元。”
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[32-32]`
- au=0.6817582845687866, eu=0.18673212826251984, au′=0.6817576885223389, eu′=0.04390525445342064
- top_logits 方差 (pvariance, 有限 logit): 0.038953996431700944
- top_logits 数值: `[21.973684310913086, 21.578948974609375]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 32 | 32 | 21.973684310913086 | 原告2015年7月17日交定金10000元，2015年7月27日交房款140000元，2015年8月27日交房款50000元，2015年11月9日交房款9001元，2016年2月5日交商业贷款210000元，共计交房款419001元。 |
| 2 | 14 | 14 | 21.578948974609375 | 证明原告2015年7月17日交定金10000元，2015年7月27日交房款140000元，2015年8月27日交房款50000元，2015年11月9日交房款9001元，2016年2月5日交商业贷款210000元，共计交房款419001元。 |
- GPT 引用1: span=[32, 32], relevant=1

### 陈述 3
2. 判决书第12页提到：“原告于2015年7月17日交定金10000元，2015年7月27日交房款140000元，2015年8月27日交房款50000元，2015年11月9日交房款9001元，2016年2月5日交商业贷款210000元，共计交房款419001元。”
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[32-32]`
- au=1.7108611700678011e-06, eu=0.3052207827568054, au′=-0.0, eu′=0.04208194464445114
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[22.763158798217773]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 32 | 32 | 22.763158798217773 | 原告2015年7月17日交定金10000元，2015年7月27日交房款140000元，2015年8月27日交房款50000元，2015年11月9日交房款9001元，2016年2月5日交商业贷款210000元，共计交房款419001元。 |
- GPT 引用1: span=[32, 32], relevant=1

### 陈述 4
所以，原告闫晓霞共支付了419001元房款。
- GPT support_score=0; [[Yes]]   Need Citation: [[Yes]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

## multifieldqa_zh_idx291 (idx=291, multifieldqa_zh)
### 问题
博丽灵梦的职业是什么？

### 参考答案（数据集）
- ref1: 幻想乡境内博丽神社的的巫女，负责解决在幻想乡中发生的各种异变。

### 模型全文输出
```
<statement>根据上文，博丽灵梦的职业是博丽神社的现职巫女。<cite>[1-1][5-5]</cite></statement>
```

文档句索引范围: sid ∈ [1, 123], 共 123 句。
引用指标: R=1.0, P=1.0, F1=1.0
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=907, completion_tokens=19
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据上文，博丽灵梦的职业是博丽神社的现职巫女。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[1-1]`
- au=1.9619515114754904e-06, eu=0.30769219994544983, au′=-0.0, eu′=0.04255318641662598
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[22.5]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 1 | 1 | 22.5 | 博丽灵梦 博丽灵梦，系列作品《东方Project》中的主人公，幻想乡境内博丽神社的现有巫女，负责解决在幻想乡中发生的各种异变。 |

**跨度** `[5-5]`
- au=1.650961507948523e-06, eu=0.2934361696243286, au′=-0.0, eu′=0.03987407684326172
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.078948974609375]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 5 | 5 | 24.078948974609375 | 角色形象 身份背景 幻想乡境内博丽神社的现职巫女。 |
- GPT 引用1: span=[1, 1], relevant=1
- GPT 引用2: span=[5, 5], relevant=1

## multifieldqa_zh_idx292 (idx=292, multifieldqa_zh)
### 问题
MBLAQ的首次单独演唱会在哪里举行？

### 参考答案（数据集）
- ref1: 首尔蚕室室内体育馆。

### 模型全文输出
```
<statement>根据上文，MBLAQ的首次单独演唱会是在2011年8月20日，在韩国首尔蚕室室内体育馆举行的。<cite>[5-5][60-60]</cite></statement>
```

文档句索引范围: sid ∈ [1, 84], 共 84 句。
引用指标: R=1.0, P=1.0, F1=1.0
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=956, completion_tokens=19
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据上文，MBLAQ的首次单独演唱会是在2011年8月20日，在韩国首尔蚕室室内体育馆举行的。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[5-5]`
- au=0.6819555163383484, eu=0.18313249945640564, au′=0.6819546222686768, eu′=0.04291360080242157
- top_logits 方差 (pvariance, 有限 logit): 0.10820680047709175
- top_logits 数值: `[22.631580352783203, 21.973684310913086]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 60 | 60 | 22.631580352783203 | 8月20日，MBLAQ在首尔蚕室室内体育馆举行首次单独演唱会MBLAQ The 1st Asia Tour－Menin MBLAQ（首尔场） [4] 。 |
| 2 | 5 | 5 | 21.973684310913086 | 8月20日，在首尔举行首次单独演唱会 ； |

**跨度** `[60-60]`
- au=1.9009210063813953e-06, eu=0.2957197427749634, au′=-0.0, eu′=0.04029691964387894
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.8157901763916]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 60 | 60 | 23.8157901763916 | 8月20日，MBLAQ在首尔蚕室室内体育馆举行首次单独演唱会MBLAQ The 1st Asia Tour－Menin MBLAQ（首尔场） [4] 。 |
- GPT 引用1: span=[5, 5], relevant=1
- GPT 引用2: span=[60, 60], relevant=1

## multifieldqa_zh_idx293 (idx=293, multifieldqa_zh)
### 问题
Sun公司与TI公司合作开发了哪款RISC微处理器？

### 参考答案（数据集）
- ref1: SPARC。

### 模型全文输出
```
<statement>根据文中内容，Sun公司与TI公司合作开发的RISC微处理器是SPARC。<cite>[227-227]</cite></statement>
```

文档句索引范围: sid ∈ [1, 258], 共 258 句。
引用指标: R=1.0, P=1.0, F1=1.0
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=600, completion_tokens=13
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据文中内容，Sun公司与TI公司合作开发的RISC微处理器是SPARC。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[227-227]`
- au=1.5962523320922628e-06, eu=0.28252777457237244, au′=-0.0, eu′=0.03788633644580841
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.394737243652344]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 227 | 227 | 25.394737243652344 | SUN： 1987年，SUN和TI公司合作开发了RISC微处理器——SPARC。 |
- GPT 引用1: span=[227, 227], relevant=1

## multifieldqa_zh_idx294 (idx=294, multifieldqa_zh)
### 问题
2018年韶关居民人均可支配收入增长了多少？

### 参考答案（数据集）
- ref1: 8%。

### 模型全文输出
```
<statement>根据韶关市2018年政府工作报告，2018年韶关居民人均可支配收入增长了8%。<cite>[26-26]</cite></statement>
```

文档句索引范围: sid ∈ [1, 105], 共 105 句。
引用指标: R=1.0, P=1.0, F1=1.0
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=604, completion_tokens=13
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据韶关市2018年政府工作报告，2018年韶关居民人均可支配收入增长了8%。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[26-26]`
- au=1.6452690942969639e-06, eu=0.29230761528015137, au′=-0.0, eu′=0.03966596722602844
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.210527420043945]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 26 | 26 | 24.210527420043945 | 居民人均可支配收入2.36万元，增长8%。 |
- GPT 引用1: span=[26, 26], relevant=1

## multifieldqa_zh_idx295 (idx=295, multifieldqa_zh)
### 问题
二审法院认为利息应如何计算?

### 参考答案（数据集）
- ref1: 二审法院认为从郭海燕起诉之日2017年7月4日起，利息应按同期银行贷款利率计算。

### 模型全文输出
```
<statement>根据判决书，二审法院认为利息应如下计算：<cite></cite></statement>

<statement>1. 利息计算基数为转让款770000元。<cite>[45-45]</cite></statement>

<statement>2. 利息计算时间为从郭海燕起诉之日2017年7月4日起，至本判决确定的履行期限届满之日止。<cite>[45-45]</cite></statement>

<statement>3. 利率按照银行同期贷款利率计算。<cite>[45-45]</cite></statement>

<statement>4. 范舒炜应按照上述方法计算并支付利息给郭海燕。<cite>[45-45]</cite></statement>

<statement>所以二审法院认为利息应按照上述方法计算，由范舒炜支付给郭海燕。<cite></cite></statement>
```

文档句索引范围: sid ∈ [1, 48], 共 48 句。
引用指标: R=0.5833333333333334, P=1.0, F1=0.7368421052631579
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=3412, completion_tokens=76
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据判决书，二审法院认为利息应如下计算：
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 1
1. 利息计算基数为转让款770000元。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[45-45]`
- au=1.6340368347300682e-06, eu=0.29007625579833984, au′=-0.0, eu′=0.03925619646906853
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.473684310913086]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 45 | 45 | 24.473684310913086 | 二、范舒炜于本判决生效后十日内支付郭海燕转让款770000元及利息（利息以770000元为计算基数，从2017年7月4日计算至本判决确定的履行期限届满之日止，利率按照银行同期贷款利率计算。 |
- GPT 引用1: span=[45, 45], relevant=1

### 陈述 2
2. 利息计算时间为从郭海燕起诉之日2017年7月4日起，至本判决确定的履行期限届满之日止。
- GPT support_score=1; **Rating: [[Fully supported]]**   **

**跨度** `[45-45]`
- au=1.9067709899900365e-06, eu=0.29687488079071045, au′=-0.0, eu′=0.04051172360777855
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.68421173095703]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 45 | 45 | 23.68421173095703 | 二、范舒炜于本判决生效后十日内支付郭海燕转让款770000元及利息（利息以770000元为计算基数，从2017年7月4日计算至本判决确定的履行期限届满之日止，利率按照银行同期贷款利率计算。 |
- GPT 引用1: span=[45, 45], relevant=1

### 陈述 3
3. 利率按照银行同期贷款利率计算。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[45-45]`
- au=0.6822130680084229, eu=0.17966897785663605, au′=0.6822121143341064, eu′=0.04196576029062271
- top_logits 方差 (pvariance, 有限 logit): 0.10820680047709175
- top_logits 数值: `[23.157896041870117, 22.5]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 45 | 45 | 23.157896041870117 | 二、范舒炜于本判决生效后十日内支付郭海燕转让款770000元及利息（利息以770000元为计算基数，从2017年7月4日计算至本判决确定的履行期限届满之日止，利率按照银行同期贷款利率计算。 |
| 2 | 40 | 40 | 22.5 | 关于利息部分，因双方在合同中并未约定，故从郭海燕起诉之日2017年7月4日起计算，利率按同期银行贷款利率计算。 |
- GPT 引用1: span=[45, 45], relevant=1

### 陈述 4
4. 范舒炜应按照上述方法计算并支付利息给郭海燕。
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[45-45]`
- au=1.6452690942969639e-06, eu=0.29230761528015137, au′=-0.0, eu′=0.03966596722602844
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.210527420043945]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 45 | 45 | 24.210527420043945 | 二、范舒炜于本判决生效后十日内支付郭海燕转让款770000元及利息（利息以770000元为计算基数，从2017年7月4日计算至本判决确定的履行期限届满之日止，利率按照银行同期贷款利率计算。 |
- GPT 引用1: span=[45, 45], relevant=1

### 陈述 5
所以二审法院认为利息应按照上述方法计算，由范舒炜支付给郭海燕。
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

## multifieldqa_zh_idx296 (idx=296, multifieldqa_zh)
### 问题
全市"十二五"时期累计解决或改善多少户中低收入家庭的住房困难？

### 参考答案（数据集）
- ref1: 4.6万户。

### 模型全文输出
```
<statement>根据工作报告，黄山市“十二五”时期累计解决或改善4.6万户中低收入家庭的住房困难。<cite>[57-57]</cite></statement>
```

文档句索引范围: sid ∈ [1, 177], 共 177 句。
引用指标: R=1.0, P=1.0, F1=1.0
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=684, completion_tokens=13
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据工作报告，黄山市“十二五”时期累计解决或改善4.6万户中低收入家庭的住房困难。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[57-57]`
- au=1.9246485862822738e-06, eu=0.3003951609134674, au′=-0.0, eu′=0.04117009416222572
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.289474487304688]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 57 | 57 | 23.289474487304688 | 新开工建设各类保障性住房和棚户区改造安置房3.6万套，发放廉租住房租赁补贴5452万元，累计解决或改善4.6万户中低收入家庭的住房困难。 |
- GPT 引用1: span=[57, 57], relevant=1

## multifieldqa_zh_idx297 (idx=297, multifieldqa_zh)
### 问题
一审判决是哪个法院给出的？

### 参考答案（数据集）
- ref1: 北京市通州区人民法院。

### 模型全文输出
```
<statement>根据判决书内容，一审判决是由北京市通州区人民法院(2017)京0112民初20125号民事判决给出的。<cite>[1-1]</cite></statement>
```

文档句索引范围: sid ∈ [1, 55], 共 55 句。
引用指标: R=1.0, P=1.0, F1=1.0
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=778, completion_tokens=17
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据判决书内容，一审判决是由北京市通州区人民法院(2017)京0112民初20125号民事判决给出的。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[1-1]`
- au=1.650961507948523e-06, eu=0.2934361696243286, au′=-0.0, eu′=0.03987407684326172
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.078948974609375]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 1 | 1 | 24.078948974609375 | 孙毅等与闫某提供劳务者受害责任纠纷二审民事判决书 上诉人孙毅因与被上诉人闫某、原审被告北京民芳斋古典家具有限公司（以下简称民芳斋公司）提供劳务者受害责任纠纷一案，不服北京市通州区人民法院（2017）京0112民初20125号民事判决，向本院提起上诉。 |
- GPT 引用1: span=[1, 1], relevant=1

## multifieldqa_zh_idx298 (idx=298, multifieldqa_zh)
### 问题
什么是飞航？

### 参考答案（数据集）
- ref1: 航空器的起飞、航行、降落和滑行。

### 模型全文输出
```
<statement>根据民用航空法的定义，飞航是指航空器的起飞、航行、降落及起飞前降落後所需在航空站、飞行场之滑行。<cite>[18-18]</cite></statement>
```

文档句索引范围: sid ∈ [1, 29], 共 29 句。
引用指标: R=1.0, P=1.0, F1=1.0
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=648, completion_tokens=13
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据民用航空法的定义，飞航是指航空器的起飞、航行、降落及起飞前降落後所需在航空站、飞行场之滑行。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[18-18]`
- au=1.6742565094318707e-06, eu=0.29803910851478577, au′=-0.0, eu′=0.04072882607579231
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.55263328552246]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 18 | 18 | 23.55263328552246 | 三、飛航：指航空器之起飛、航行、降落及起飛前降落後所需在航空站、飛行場之滑行。 |
- GPT 引用1: span=[18, 18], relevant=1

## multifieldqa_zh_idx299 (idx=299, multifieldqa_zh)
### 问题
报废机动车在进行拆解作业之前可以侧放或倒放吗？

### 参考答案（数据集）
- ref1: 不可以。

### 模型全文输出
```
<statement>根据《报废机动车拆解环境保护技术规范》HJ 348-2007,报废机动车在进行拆解作业之前不得侧放、倒放。<cite>[51-51]</cite></statement>

<statement>相关原文：<cite></cite></statement>

<statement>6.4.4 报废机动车在进行拆解作业之前不得侧放、倒放。<cite>[51-51]</cite></statement>

<statement>所以，拆解作业之前不能侧放或倒放报废机动车。<cite></cite></statement>
```

文档句索引范围: sid ∈ [1, 94], 共 94 句。
引用指标: R=0.75, P=1.0, F1=0.8571428571428571
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=1809, completion_tokens=46
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据《报废机动车拆解环境保护技术规范》HJ 348-2007,报废机动车在进行拆解作业之前不得侧放、倒放。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[51-51]`
- au=1.7496245163783897e-06, eu=0.31275713443756104, au′=-0.0, eu′=0.04352806136012077
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[21.973684310913086]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 51 | 51 | 21.973684310913086 | ６ \ue010 ４ 报废机动车在进行拆解作业之前不得侧放、倒放。 |
- GPT 引用1: span=[51, 51], relevant=1

### 陈述 1
相关原文：
- GPT support_score=1; [[No]]   Need Citation: [[No]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 2
6.4.4 报废机动车在进行拆解作业之前不得侧放、倒放。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[51-51]`
- au=1.6396279534092173e-06, eu=0.2911876440048218, au′=-0.0, eu′=0.039460018277168274
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.342105865478516]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 51 | 51 | 24.342105865478516 | ６ \ue010 ４ 报废机动车在进行拆解作业之前不得侧放、倒放。 |
- GPT 引用1: span=[51, 51], relevant=1

### 陈述 3
所以，拆解作业之前不能侧放或倒放报废机动车。
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

## multifieldqa_zh_idx300 (idx=300, multifieldqa_zh)
### 问题
长沙市在交通方面采取了哪些措施来推动城乡融合？

### 参考答案（数据集）
- ref1: 启动南、北横线建设，基本实现乡镇30分钟上高速，行政村、自然村100%通水泥路。

### 模型全文输出
```
<statement>长沙市在交通方面采取了以下措施来推动城乡融合：<cite></cite></statement>

<statement>1. 启动南、北横线建设，基本实现乡镇30分钟上高速。<cite>[38-38]</cite></statement>

<statement>2. 行政村、自然村100%通水泥路。<cite>[38-38]</cite></statement>

<statement>3. 农村电网改造升级率、自来水普及率、乡镇污水处理厂覆盖率分别达90%、71%、75%以上。<cite>[39-39]</cite></statement>

<statement>4. 新增高速公路153.7公里。<cite>[29-29]</cite></statement>

<statement>5. 新增过江通道4条。<cite>[30-30]</cite></statement>

<statement>6. 湘江新区综合交通枢纽、万家丽快速干道、火车南站东广场等重大基础设施建成。<cite>[30-30]</cite></statement>

<statement>7. 芙蓉北大道、黄兴北路、洞株公路建成通车。<cite>[30-30]</cite></statement>

<statement>8. 打通断头路瓶颈路47条。<cite>[30-30]</cite></statement>

<statement>9. 公交都市加快建设，新增公交线路18条，新建公交站场24个。<cite>[31-31]</cite></statement>

<statement>通过这些措施，长沙市在交通基础设施建设方面加大投入，全面提升城乡交通网络水平，为城乡融合提供了有力支撑。<cite></cite></statement>
```

文档句索引范围: sid ∈ [1, 73], 共 73 句。
引用指标: R=1.0, P=1.0, F1=1.0
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=6702, completion_tokens=137
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
长沙市在交通方面采取了以下措施来推动城乡融合：
- GPT support_score=1; [[No]]   Need Citation: [[No]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 1
1. 启动南、北横线建设，基本实现乡镇30分钟上高速。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[38-38]`
- au=1.6015096662158612e-06, eu=0.28358200192451477, au′=-0.0, eu′=0.03807614743709564
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.263158798217773]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 38 | 38 | 25.263158798217773 | 启动南、北横线建设，基本实现乡镇30分钟上高速，行政村、自然村100%通水泥路。 |
- GPT 引用1: span=[38, 38], relevant=1

### 陈述 2
2. 行政村、自然村100%通水泥路。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[38-38]`
- au=1.7844809008238371e-06, eu=0.2724013328552246, au′=-0.0, eu′=0.036087363958358765
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[26.710527420043945]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 38 | 38 | 26.710527420043945 | 启动南、北横线建设，基本实现乡镇30分钟上高速，行政村、自然村100%通水泥路。 |
- GPT 引用1: span=[38, 38], relevant=1

### 陈述 3
3. 农村电网改造升级率、自来水普及率、乡镇污水处理厂覆盖率分别达90%、71%、75%以上。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[39-39]`
- au=1.5807482895979774e-06, eu=0.2794116735458374, au′=-0.0, eu′=0.037328090518713
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.789474487304688]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 39 | 39 | 25.789474487304688 | 农村电网改造升级率、自来水普及率、乡镇污水处理厂覆盖率分别达90%、71%、75%以上，建成国家卫生乡镇5个、垃圾分类示范村50个。 |
- GPT 引用1: span=[39, 39], relevant=1

### 陈述 4
4. 新增高速公路153.7公里。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[29-29]`
- au=1.650961507948523e-06, eu=0.2934361696243286, au′=-0.0, eu′=0.03987407684326172
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.078948974609375]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 29 | 29 | 24.078948974609375 | 长韶娄、长浏等高速建成通车，新增高速公路153.7公里。 |
- GPT 引用1: span=[29, 29], relevant=1

### 陈述 5
5. 新增过江通道4条。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[30-30]`
- au=1.6068129298218992e-06, eu=0.28464409708976746, au′=-0.0, eu′=0.038267869502305984
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.131580352783203]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 30 | 30 | 25.131580352783203 | 新增过江通道4条，湘江新区综合交通枢纽、万家丽快速干道、火车南站东广场等重大基础设施建成，芙蓉北大道、黄兴北路、洞株公路建成通车，打通断头路瓶颈路47条。 |
- GPT 引用1: span=[30, 30], relevant=1

### 陈述 6
6. 湘江新区综合交通枢纽、万家丽快速干道、火车南站东广场等重大基础设施建成。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[30-30]`
- au=1.6175589507838595e-06, eu=0.2867923676967621, au′=-0.0, eu′=0.03865716978907585
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.86842155456543]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 30 | 30 | 24.86842155456543 | 新增过江通道4条，湘江新区综合交通枢纽、万家丽快速干道、火车南站东广场等重大基础设施建成，芙蓉北大道、黄兴北路、洞株公路建成通车，打通断头路瓶颈路47条。 |
- GPT 引用1: span=[30, 30], relevant=1

### 陈述 7
7. 芙蓉北大道、黄兴北路、洞株公路建成通车。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[30-30]`
- au=1.861421651483397e-06, eu=0.28787869215011597, au′=-0.0, eu′=0.038854800164699554
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.73684310913086]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 30 | 30 | 24.73684310913086 | 新增过江通道4条，湘江新区综合交通枢纽、万家丽快速干道、火车南站东广场等重大基础设施建成，芙蓉北大道、黄兴北路、洞株公路建成通车，打通断头路瓶颈路47条。 |
- GPT 引用1: span=[30, 30], relevant=1

### 陈述 8
8. 打通断头路瓶颈路47条。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[30-30]`
- au=1.6015096662158612e-06, eu=0.28358200192451477, au′=-0.0, eu′=0.03807614743709564
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.263158798217773]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 30 | 30 | 25.263158798217773 | 新增过江通道4条，湘江新区综合交通枢纽、万家丽快速干道、火车南站东广场等重大基础设施建成，芙蓉北大道、黄兴北路、洞株公路建成通车，打通断头路瓶颈路47条。 |
- GPT 引用1: span=[30, 30], relevant=1

### 陈述 9
9. 公交都市加快建设，新增公交线路18条，新建公交站场24个。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[31-31]`
- au=1.6068129298218992e-06, eu=0.28464409708976746, au′=-0.0, eu′=0.038267869502305984
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.131580352783203]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 31 | 31 | 25.131580352783203 | 公交都市加快建设，新增公交线路18条，新建公交站场24个，公交出行分担率达43.1%。 |
- GPT 引用1: span=[31, 31], relevant=1

### 陈述 10
通过这些措施，长沙市在交通基础设施建设方面加大投入，全面提升城乡交通网络水平，为城乡融合提供了有力支撑。
- GPT support_score=1; [[No]]   Need Citation: [[No]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

## multifieldqa_zh_idx301 (idx=301, multifieldqa_zh)
### 问题
鹤壁市的经济社会发展还面临着哪些困难和挑战？

### 参考答案（数据集）
- ref1: 鹤壁市的经济社会发展还面临着经济下行压力加大、转型任务艰巨、风险隐患不容忽视、自身建设需要加强等困难和挑战。

### 模型全文输出
```
<statement>根据报告，鹤壁市经济社会发展还面临以下困难和挑战：<cite></cite></statement>

<statement>1. 经济下行压力加大。受外部环境影响、自身问题叠加、结构矛盾凸显等制约，部分重点企业生产经营困难，平台和要素支撑能力不强，发展内生动力不足，主要经济指标增速放缓。<cite>[56-57]</cite></statement>

<statement>2. 转型任务依然艰巨。三次产业结构不优，城乡建设和民生领域欠账依然较多，科技创新和成果转化能力相对偏弱，财税和居民增收后劲不足，环境质量有待进一步改善。<cite>[58-59]</cite></statement>

<statement>3. 风险隐患不容忽视。不确定因素明显增多，市场风险、债务风险、信用风险、企业风险、金融风险、社会风险、安全风险等交织叠加，维护安全稳定的任务依然艰巨。<cite>[60-61]</cite></statement>

<statement>4. 自身建设需要加强。政府工作人员能力素质难以适应高质量发展要求，观念落后、作风浮漂、推拖庸懒、不作为慢作为等现象不同程度存在，一些领域不正之风和腐败问题时有发生。<cite>[62-63]</cite></statement>

<statement>所以，鹤壁市经济社会发展还面临经济下行压力、转型任务艰巨、风险隐患不容忽视以及自身建设需要加强等多重困难和挑战。<cite></cite></statement>
```

文档句索引范围: sid ∈ [1, 96], 共 96 句。
引用指标: R=0.8333333333333334, P=1.0, F1=0.9090909090909091
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=4063, completion_tokens=72
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据报告，鹤壁市经济社会发展还面临以下困难和挑战：
- GPT support_score=1; [[No]]   Need Citation: [[No]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 1
1. 经济下行压力加大。受外部环境影响、自身问题叠加、结构矛盾凸显等制约，部分重点企业生产经营困难，平台和要素支撑能力不强，发展内生动力不足，主要经济指标增速放缓。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[56-57]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> 一是经济下行压力加大。 受外部环境影响、自身问题叠加、结构矛盾凸显等制约，部分重点企业生产经营困难，平台和要素支撑能力不强，发展内生动力不足，主要经济指标增速放缓。
- au=1.5962523320922628e-06, eu=0.28252777457237244, au′=-0.0, eu′=0.03788633644580841
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.394737243652344]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 56 | 56 | 25.394737243652344 | 一是经济下行压力加大。 |
- GPT 引用1: span=[56, 57], relevant=1

### 陈述 2
2. 转型任务依然艰巨。三次产业结构不优，城乡建设和民生领域欠账依然较多，科技创新和成果转化能力相对偏弱，财税和居民增收后劲不足，环境质量有待进一步改善。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[58-59]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> 二是转型任务依然艰巨。 三次产业结构不优，城乡建设和民生领域欠账依然较多，科技创新和成果转化能力相对偏弱，财税和居民增收后劲不足，环境质量有待进一步改善。
- au=1.6175589507838595e-06, eu=0.2867923676967621, au′=-0.0, eu′=0.03865716978907585
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.86842155456543]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 58 | 58 | 24.86842155456543 | 二是转型任务依然艰巨。 |
- GPT 引用1: span=[58, 59], relevant=1

### 陈述 3
3. 风险隐患不容忽视。不确定因素明显增多，市场风险、债务风险、信用风险、企业风险、金融风险、社会风险、安全风险等交织叠加，维护安全稳定的任务依然艰巨。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[60-61]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> 三是风险隐患不容忽视。 不确定因素明显增多，市场风险、债务风险、信用风险、企业风险、金融风险、社会风险、安全风险等交织叠加，维护安全稳定的任务依然艰巨。
- au=1.6015096662158612e-06, eu=0.28358200192451477, au′=-0.0, eu′=0.03807614743709564
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.263158798217773]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 60 | 60 | 25.263158798217773 | 三是风险隐患不容忽视。 |
- GPT 引用1: span=[60, 61], relevant=1

### 陈述 4
4. 自身建设需要加强。政府工作人员能力素质难以适应高质量发展要求，观念落后、作风浮漂、推拖庸懒、不作为慢作为等现象不同程度存在，一些领域不正之风和腐败问题时有发生。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[62-63]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> 四是自身建设需要加强。 政府工作人员能力素质难以适应高质量发展要求，观念落后、作风浮漂、推拖庸懒、不作为慢作为等现象不同程度存在，一些领域不正之风和腐败问题时有发生。
- au=1.6175589507838595e-06, eu=0.2867923676967621, au′=-0.0, eu′=0.03865716978907585
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.86842155456543]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 62 | 62 | 24.86842155456543 | 四是自身建设需要加强。 |
- GPT 引用1: span=[62, 63], relevant=1

### 陈述 5
所以，鹤壁市经济社会发展还面临经济下行压力、转型任务艰巨、风险隐患不容忽视以及自身建设需要加强等多重困难和挑战。
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

## multifieldqa_zh_idx302 (idx=302, multifieldqa_zh)
### 问题
《孔子世家谱》的编修工作历时多久？

### 参考答案（数据集）
- ref1: 7年。

### 模型全文输出
```
<statement>根据上文，《孔子世家谱》的编修工作历时10年。<cite>[1-1]</cite></statement>

<statement>上文提到：<cite></cite></statement>

<statement>“从2008年10月决定编修补遗卷到目前，最终有1.1万多人填报了后裔资料登记表。”<cite>[3-3]</cite></statement>

<statement>“编修补遗卷是《孔子世家谱》第五次大修的最后一步，因为不少后裔族人得到修谱消息较晚或身份考证过程较长，未能及时录入总谱，为避免这部分孔子后裔资料失传，孔子世家谱续修工作协会决定编修一部补遗卷。”<cite>[1-4]</cite></statement>

<statement>“2008年12月31日停止收集孔子后裔资料，历时10年的《孔子世家谱》第五次大修后裔资料收集工作全面完成。”<cite>[1-1]</cite></statement>

<statement>所以，《孔子世家谱》的编修工作历时10年。<cite></cite></statement>
```

文档句索引范围: sid ∈ [1, 136], 共 136 句。
引用指标: R=0.8333333333333334, P=1.0, F1=0.9090909090909091
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=3937, completion_tokens=72
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据上文，《孔子世家谱》的编修工作历时10年。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[1-1]`
- au=1.7430054413125617e-06, eu=0.31147530674934387, au′=-0.0, eu′=0.04328017681837082
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[22.10526466369629]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 1 | 1 | 22.10526466369629 | 孔子世家谱 《孔子世家谱》：随着新版《孔子世家谱》补遗卷在2008年12月31日停止收集孔子后裔资料，历时10年的《孔子世家谱》第五次大修后裔资料收集工作全面完成。 |
- GPT 引用1: span=[1, 1], relevant=1

### 陈述 1
上文提到：
- GPT support_score=1; [[No]]   Need Citation: [[No]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 2
“从2008年10月决定编修补遗卷到目前，最终有1.1万多人填报了后裔资料登记表。”
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[3-3]`
- au=1.6340368347300682e-06, eu=0.29007625579833984, au′=-0.0, eu′=0.03925619646906853
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.473684310913086]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 3 | 3 | 24.473684310913086 | 从2008年10月决定编修补遗卷到目前，最终有1.1万多人填报了后裔资料登记表。 |
- GPT 引用1: span=[3, 3], relevant=1

### 陈述 3
“编修补遗卷是《孔子世家谱》第五次大修的最后一步，因为不少后裔族人得到修谱消息较晚或身份考证过程较长，未能及时录入总谱，为避免这部分孔子后裔资料失传，孔子世家谱续修工作协会决定编修一部补遗卷。”
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[1-4]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> 孔子世家谱 《孔子世家谱》：随着新版《孔子世家谱》补遗卷在2008年12月31日停止收集孔子后裔资料，历时10年的《孔子世家谱》第五次大修后裔资料收集工作全面完成。 编修补遗卷
> 是《孔子世家谱》第五次大修的最后一步，因为不少后裔族人得到修谱消息较晚或身份考证过程较长，未能及时录入总谱，为避免这部分孔子后裔资料失传，孔子世家谱续修工作协会决定编修一部补遗
> 卷。 从2008年10月决定编修补遗卷到目前，最终有1.1万多人填报了后裔资料登记表。 编修 民国十七年，孔族中人倡修合族大谱，几经筹备，于民国十九年在孔庙开馆，正式进行。
- au=0.6825815439224243, eu=0.17551958560943604, au′=0.6825807094573975, eu′=0.04083825275301933
- top_logits 方差 (pvariance, 有限 logit): 0.03895437288156245
- top_logits 数值: `[23.68421173095703, 23.289474487304688]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 1 | 1 | 23.68421173095703 | 孔子世家谱 《孔子世家谱》：随着新版《孔子世家谱》补遗卷在2008年12月31日停止收集孔子后裔资料，历时10年的《孔子世家谱》第五次大修后裔资料收集工作全面完成。 |
| 2 | 2 | 2 | 23.289474487304688 | 编修补遗卷是《孔子世家谱》第五次大修的最后一步，因为不少后裔族人得到修谱消息较晚或身份考证过程较长，未能及时录入总谱，为避免这部分孔子后裔资料失传，孔子世家谱续修工作协会决定编修一部补遗卷。 |
- GPT 引用1: span=[1, 4], relevant=1

### 陈述 4
“2008年12月31日停止收集孔子后裔资料，历时10年的《孔子世家谱》第五次大修后裔资料收集工作全面完成。”
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[1-1]`
- au=1.9067709899900365e-06, eu=0.29687488079071045, au′=-0.0, eu′=0.04051172360777855
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.68421173095703]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 1 | 1 | 23.68421173095703 | 孔子世家谱 《孔子世家谱》：随着新版《孔子世家谱》补遗卷在2008年12月31日停止收集孔子后裔资料，历时10年的《孔子世家谱》第五次大修后裔资料收集工作全面完成。 |
- GPT 引用1: span=[1, 1], relevant=1

### 陈述 5
所以，《孔子世家谱》的编修工作历时10年。
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

## multifieldqa_zh_idx303 (idx=303, multifieldqa_zh)
### 问题
"十一五"时期我市的港口货物吞吐量增长了多少倍？

### 参考答案（数据集）
- ref1: 2.2倍。

### 模型全文输出
```
<statement>根据工作报告内容，“十一五”时期我市港口货物吞吐量由2377万吨增加到7650万吨，增长2.2倍。<cite>[48-48]</cite></statement>
```

文档句索引范围: sid ∈ [1, 60], 共 60 句。
引用指标: R=1.0, P=1.0, F1=1.0
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=666, completion_tokens=13
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据工作报告内容，“十一五”时期我市港口货物吞吐量由2377万吨增加到7650万吨，增长2.2倍。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[48-48]`
- au=1.7108611700678011e-06, eu=0.3052207827568054, au′=-0.0, eu′=0.04208194464445114
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[22.763158798217773]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 48 | 48 | 22.763158798217773 | 全市港口货物年吞吐量由2377万吨增加到7650万吨，增长2.2倍，占广西沿海港口货物吞吐量近7成。 |
- GPT 引用1: span=[48, 48], relevant=1

## multifieldqa_zh_idx304 (idx=304, multifieldqa_zh)
### 问题
极地区文件命名中的标准图号是什么字符？

### 参考答案（数据集）
- ref1: 南北半球均为"W"。

### 模型全文输出
```
<statement>根据CH/T XXXXX-202X规范，极地区文件命名中的标准图号是1位字符，南北半球均为“W”。<cite>[52-54]</cite></statement>
```

文档句索引范围: sid ∈ [1, 188], 共 188 句。
引用指标: R=1.0, P=1.0, F1=1.0
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=728, completion_tokens=13
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据CH/T XXXXX-202X规范，极地区文件命名中的标准图号是1位字符，南北半球均为“W”。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[52-54]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> 图 2 极地区文件命名结构图极地区文件命名结构说明如下：南北半球标识符：1 位字符，北半球为“N”，南半球为“S”。 标准图号：1 位字符，南北半球均为“W”。 产品标识符：3
> 位字符，数字表面模型数据为“DSM”。
- au=0.6820714473724365, eu=0.18095232546329498, au′=0.6820706725120544, eu′=0.0423162542283535
- top_logits 方差 (pvariance, 有限 logit): 0.1558167386256173
- top_logits 数值: `[23.026315689086914, 22.23684310913086]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 52 | 52 | 23.026315689086914 | 图 2 极地区文件命名结构图极地区文件命名结构说明如下：南北半球标识符：1 位字符，北半球为“N”，南半球为“S”。 |
| 2 | 53 | 53 | 22.23684310913086 | 标准图号：1 位字符，南北半球均为“W”。 |
- GPT 引用1: span=[52, 54], relevant=1

## multifieldqa_zh_idx305 (idx=305, multifieldqa_zh)
### 问题
附卡持卡人的責任是什么？

### 参考答案（数据集）
- ref1: 正卡持卡人未依規定清償時，附卡持卡人負責該附卡所生的應付帳款。

### 模型全文输出
```
<statement>根据约定条款，正卡持卡人对其本人和附卡持卡人使用信用卡所生应付账款的全部负清償责任。<cite>[16-16]</cite></statement>

<statement>如果正卡持卡人未清償时，附卡持卡人仅就该附卡所生应付账款负清償责任。<cite>[18-18]</cite></statement>

<statement>所以附卡持卡人的责任是：<cite></cite></statement>

<statement>1. 就其使用附卡的应付账款负清償责任。<cite></cite></statement>

<statement>2. 如果正卡持卡人未清償，附卡持卡人仅就其使用附卡的应付账款负清償责任。<cite></cite></statement>

<statement>3. 需要妥善保管并使用附卡。<cite></cite></statement>

<statement>4. 需要保密开卡密码等信息。<cite></cite></statement>

<statement>5. 不得进行虚假交易。<cite></cite></statement>

<statement>6. 需要妥善保管收执联等资料。<cite>[54-54][55-55]</cite></statement>

<statement>7. 需要遵守约定条款中的其他规定。<cite></cite></statement>
```

文档句索引范围: sid ∈ [1, 65], 共 65 句。
引用指标: R=0.4, P=1.0, F1=0.5714285714285715
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=4876, completion_tokens=113
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据约定条款，正卡持卡人对其本人和附卡持卡人使用信用卡所生应付账款的全部负清償责任。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[16-16]`
- au=1.6396279534092173e-06, eu=0.2911876440048218, au′=-0.0, eu′=0.039460018277168274
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.342105865478516]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 16 | 16 | 24.342105865478516 | 正卡持卡人就其本人與附卡持卡人使用信用卡所生應付帳款之全部負清償責任。 |
- GPT 引用1: span=[16, 16], relevant=1

### 陈述 1
如果正卡持卡人未清償时，附卡持卡人仅就该附卡所生应付账款负清償责任。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[18-18]`
- au=1.8040525446849642e-06, eu=0.2763635516166687, au′=-0.0, eu′=0.03678605705499649
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[26.18421173095703]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 18 | 18 | 26.18421173095703 | 如正卡持卡人未依前項規定清償時，附卡持卡人僅就該附卡所生應付帳款負清償責任。 |
- GPT 引用1: span=[18, 18], relevant=1

### 陈述 2
所以附卡持卡人的责任是：
- GPT support_score=1; [[No]]   Need Citation: [[No]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 3
1. 就其使用附卡的应付账款负清償责任。
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 4
2. 如果正卡持卡人未清償，附卡持卡人仅就其使用附卡的应付账款负清償责任。
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 5
3. 需要妥善保管并使用附卡。
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 6
4. 需要保密开卡密码等信息。
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 7
5. 不得进行虚假交易。
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 8
6. 需要妥善保管收执联等资料。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[54-54]`
- au=1.95558504856308e-06, eu=0.3064514994621277, au′=-0.0, eu′=0.0423162542283535
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[22.631580352783203]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 54 | 54 | 22.631580352783203 | 持卡人使用信用卡交易時，於出示信用卡刷卡後，經查對無誤，應於簽帳單上簽名確認，並自行妥善保管收執聯，以供查證之用。 |

**跨度** `[55-55]`
- au=1.9067709899900365e-06, eu=0.29687488079071045, au′=-0.0, eu′=0.04051172360777855
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.68421173095703]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 55 | 55 | 23.68421173095703 | 持卡人於特約商店同意持卡人就原使用信用卡交易辦理退貨、取消交易、終止服務、變更貨品或其價格時，應向特約商店索取退款單，經查對無誤後，應於退款單上簽名確認，並自行妥善保管退款單收執聯，以供查證之用。 |
- GPT 引用1: span=[54, 55], relevant=1

### 陈述 9
7. 需要遵守约定条款中的其他规定。
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

## multifieldqa_zh_idx306 (idx=306, multifieldqa_zh)
### 问题
被告人郑术文被指控犯了什么罪？

### 参考答案（数据集）
- ref1: 盗窃罪。

### 模型全文输出
```
<statement>根据判决书，被告人郑术文被指控犯了盗窃罪。<cite>[1-1][10-10][42-42]</cite></statement>

<statement>判决书提到，公诉机关指控2016年7月7日晚上，被告人郑术文趁被害人张某某不备，在该房屋卧室衣柜的女士包内盗得车钥匙一把。当晚10时许，被告人郑术文离开被害人张某某家中后，利用盗取的车钥匙将被害人张某某丈夫鲜某某停放在附近的一辆车牌号为“XXXXXX”北京现代牌途胜越野车打开后并开走。次日9时许，被害人鲜某某发现车辆被盗后随即向公安机关报警。经鉴定，被盗“XXXXXX”北京现代牌途胜越野车价值人民币156200.00元。<cite>[5-10]</cite></statement>

<statement>因此，被告人郑术文被指控犯了盗窃罪。<cite></cite></statement>
```

文档句索引范围: sid ∈ [1, 51], 共 51 句。
引用指标: R=0.6666666666666666, P=1.0, F1=0.8
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=3206, completion_tokens=48
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据判决书，被告人郑术文被指控犯了盗窃罪。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[1-1]`
- au=0.6809419393539429, eu=0.19047614932060242, au′=0.680941104888916, eu′=0.04494381695985794
- top_logits 方差 (pvariance, 有限 logit): 0.523718981861748
- top_logits 数值: `[21.973684310913086, 20.526315689086914]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 1 | 1 | 21.973684310913086 | 被告人郑术文盗窃案一审刑事判决书 青海省西宁市城东区人民检察院以东检公诉刑诉［2016］489号起诉书指控被告人郑术文犯盗窃罪，于2016年10月19日向本院提起公诉。 |
| 2 | 42 | 42 | 20.526315689086914 | 本院认为，被告人郑术文以非法占有为目的，盗取他人财物，且数额巨大，其行为已构成盗窃罪。 |

**跨度** `[10-10]`
- au=1.0828224420547485, eu=0.1354723572731018, au′=1.0828220844268799, eu′=0.04489956051111221
- top_logits 方差 (pvariance, 有限 logit): 0.25777144005001495
- top_logits 数值: `[21.973684310913086, 21.052631378173828, 20.789474487304688]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 42 | 42 | 21.973684310913086 | 本院认为，被告人郑术文以非法占有为目的，盗取他人财物，且数额巨大，其行为已构成盗窃罪。 |
| 2 | 10 | 10 | 21.052631378173828 | 针对以上指控，公诉机关当庭出示了相关的证据、并认为被告人郑术文的行为已构成盗窃罪，应依照《中华人民共和国刑法》第二百六十四条之规定，提请本院依法惩处。 |
| 3 | 5 | 5 | 20.789474487304688 | 公诉机关指控：2016年7月7日晚9时许，被告人郑术文前往被害人张某某位于本市某某号的家中做客，期间趁被害人张某某不备，在该房屋卧室衣柜的女士包内盗得车钥匙一把。 |

**跨度** `[42-42]`
- au=1.650961507948523e-06, eu=0.2934361696243286, au′=-0.0, eu′=0.03987407684326172
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.078948974609375]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 42 | 42 | 24.078948974609375 | 本院认为，被告人郑术文以非法占有为目的，盗取他人财物，且数额巨大，其行为已构成盗窃罪。 |
- GPT 引用1: span=[1, 1], relevant=1
- GPT 引用2: span=[10, 10], relevant=1
- GPT 引用3: span=[42, 42], relevant=1

### 陈述 1
判决书提到，公诉机关指控2016年7月7日晚上，被告人郑术文趁被害人张某某不备，在该房屋卧室衣柜的女士包内盗得车钥匙一把。当晚10时许，被告人郑术文离开被害人张某某家中后，利用盗取的车钥匙将被害人张某某丈夫鲜某某停放在附近的一辆车牌号为“XXXXXX”北京现代牌途胜越野车打开后并开走。次日9时许，被害人鲜某某发现车辆被盗后随即向公安机关报警。经鉴定，被盗“XXXXXX”北京现代牌途胜越野车价值人民币156200.00元。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[5-10]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> 公诉机关指控：2016年7月7日晚9时许，被告人郑术文前往被害人张某某位于本市某某号的家中做客，期间趁被害人张某某不备，在该房屋卧室衣柜的女士包内盗得车钥匙一把。
> 当晚10时许，被告人郑术文离开被害人张某某家中后，利用盗取的车钥匙将被害人张某某丈夫鲜某某停放在附近的一辆车牌号为“XXXXXX”北京现代牌途胜越野车打开后并开走。
> 次日9时许，被害人鲜某某发现车辆被盗后随即向公安机关报警。 2016年7月12日被告人郑术文在本市城东区博雅路附近抓获，被盗车辆被追回并发还被害人鲜某某。
> 经鉴定，被盗“XXXXXX”北京现代牌途胜越野车价值人民币156200.00元。
> 针对以上指控，公诉机关当庭出示了相关的证据、并认为被告人郑术文的行为已构成盗窃罪，应依照《中华人民共和国刑法》第二百六十四条之规定，提请本院依法惩处。
- au=1.6015096662158612e-06, eu=0.28358200192451477, au′=-0.0, eu′=0.03807614743709564
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.263158798217773]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 5 | 5 | 25.263158798217773 | 公诉机关指控：2016年7月7日晚9时许，被告人郑术文前往被害人张某某位于本市某某号的家中做客，期间趁被害人张某某不备，在该房屋卧室衣柜的女士包内盗得车钥匙一把。 |
- GPT 引用1: span=[5, 10], relevant=1

### 陈述 2
因此，被告人郑术文被指控犯了盗窃罪。
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

## multifieldqa_zh_idx307 (idx=307, multifieldqa_zh)
### 问题
Mac OS系统下载MotionPro客户端需要去哪里下载？

### 参考答案（数据集）
- ref1: 可以通过AppStore或文档提供的客户端链接下载MotionPro客户端。

### 模型全文输出
```
<statement>根据使用说明，Mac OS系统下载MotionPro客户端需要前往AppStore进行下载。<cite></cite></statement>

<statement>具体来说：<cite></cite></statement>

<statement>1. 打开AppStore<cite>[20-20]</cite></statement>

<statement>2. 在右上角搜索框中搜索MotionPro<cite>[20-20]</cite></statement>

<statement>3. 点击MotionPro进行下载安装<cite>[20-21]</cite></statement>

<statement>使用说明中明确指出：<cite></cite></statement>

<statement>"Mac OS系统(版本10.11.0以上的操作系统)安装vpn首先需前往AppStore中下载客户端MotionPro,打开AppStore后可在右上角搜索框中搜索MotionPro;"<cite>[20-20]</cite></statement>

<statement>所以Mac OS系统下载MotionPro客户端需要去AppStore进行下载。<cite></cite></statement>
```

文档句索引范围: sid ∈ [1, 27], 共 27 句。
引用指标: R=0.5625, P=1.0, F1=0.72
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=3937, completion_tokens=93
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据使用说明，Mac OS系统下载MotionPro客户端需要前往AppStore进行下载。
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 1
具体来说：
- GPT support_score=1; [[No]]   Need Citation: [[No]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 2
1. 打开AppStore
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[20-20]`
- au=1.6340368347300682e-06, eu=0.29007625579833984, au′=-0.0, eu′=0.03925619646906853
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.473684310913086]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 20 | 20 | 24.473684310913086 | 三、Mac OS 系统 VPN 使用方法1、Mac OS 系统（版本 10.11.0 以上的操作系统）安装 vpn 首先需前往 AppStore 中下载客户端 MotionPro，打开 AppStore 后可在右上角搜索框中搜索 MotionPro； |
- GPT 引用1: span=[20, 20], relevant=1

### 陈述 3
2. 在右上角搜索框中搜索MotionPro
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[20-20]`
- au=1.5606800616296823e-06, eu=0.2753622531890869, au′=-0.0, eu′=0.03660885989665985
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[26.3157901763916]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 20 | 20 | 26.3157901763916 | 三、Mac OS 系统 VPN 使用方法1、Mac OS 系统（版本 10.11.0 以上的操作系统）安装 vpn 首先需前往 AppStore 中下载客户端 MotionPro，打开 AppStore 后可在右上角搜索框中搜索 MotionPro； |
- GPT 引用1: span=[20, 20], relevant=1

### 陈述 4
3. 点击MotionPro进行下载安装
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[20-21]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> 三、Mac OS 系统 VPN 使用方法1、Mac OS 系统（版本 10.11.0 以上的操作系统）安装 vpn 首先需前往 AppStore 中下载客户端
> MotionPro，打开 AppStore 后可在右上角搜索框中搜索 MotionPro； 也可以点击此处下载客户端2、客户端 MotionPro 下载完成后点击安装；
- au=1.5910399042695644e-06, eu=0.28148138523101807, au′=-0.0, eu′=0.0376984067261219
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.526317596435547]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 20 | 20 | 25.526317596435547 | 三、Mac OS 系统 VPN 使用方法1、Mac OS 系统（版本 10.11.0 以上的操作系统）安装 vpn 首先需前往 AppStore 中下载客户端 MotionPro，打开 AppStore 后可在右上角搜索框中搜索 MotionPro； |
- GPT 引用1: span=[20, 21], relevant=1

### 陈述 5
使用说明中明确指出：
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 6
"Mac OS系统(版本10.11.0以上的操作系统)安装vpn首先需前往AppStore中下载客户端MotionPro,打开AppStore后可在右上角搜索框中搜索MotionPro;"
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[20-20]`
- au=1.6015096662158612e-06, eu=0.28358200192451477, au′=-0.0, eu′=0.03807614743709564
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.263158798217773]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 20 | 20 | 25.263158798217773 | 三、Mac OS 系统 VPN 使用方法1、Mac OS 系统（版本 10.11.0 以上的操作系统）安装 vpn 首先需前往 AppStore 中下载客户端 MotionPro，打开 AppStore 后可在右上角搜索框中搜索 MotionPro； |
- GPT 引用1: span=[20, 20], relevant=1

### 陈述 7
所以Mac OS系统下载MotionPro客户端需要去AppStore进行下载。
- GPT support_score=0; [[Yes]]   Need Citation: [[Yes]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

## multifieldqa_zh_idx308 (idx=308, multifieldqa_zh)
### 问题
青岛市2018年举办了哪个重大会议？

### 参考答案（数据集）
- ref1: 上海合作组织成员国元首理事会第十八次会议。

### 模型全文输出
```
<statement>根据报告内容，青岛市2018年举办的重大会议是上海合作组织成员国元首理事会第十八次会议。<cite>[5-5]</cite></statement>

<statement>报告中提到：做好服务保障工作，是习近平总书记和以习近平同志为核心的党中央交给青岛的一项重大政治任务。全市上下坚决贯彻落实习近平总书记重要指示精神，有力有序、精心细致做好场馆改造提升、会务礼宾服务、交通安全保卫工作，同步实施城市环境提升行动，圆满实现了“世界水准、中国气派、山东风格、青岛特色”的服务保障目标，习近平总书记作出重要指示给予充分肯定，与会各方和国内外给予广泛赞誉。<cite>[5-6]</cite></statement>

<statement>所以，青岛市2018年举办的重大会议是上海合作组织成员国元首理事会第十八次会议。<cite></cite></statement>
```

文档句索引范围: sid ∈ [1, 89], 共 89 句。
引用指标: R=0.6666666666666666, P=1.0, F1=0.8
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=2191, completion_tokens=36
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据报告内容，青岛市2018年举办的重大会议是上海合作组织成员国元首理事会第十八次会议。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[5-5]`
- au=1.650961507948523e-06, eu=0.2934361696243286, au′=-0.0, eu′=0.03987407684326172
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.078948974609375]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 5 | 5 | 24.078948974609375 | 主要做了以下工作：  (一)团结一心众志成城，峰会服务保障任务圆满完成  上海合作组织成员国元首理事会第十八次会议在我市召开，做好服务保障工作，是习近平总书记和以习近平同志为核心的党中央交给青岛的一项重大政治任务。 |
- GPT 引用1: span=[5, 5], relevant=1

### 陈述 1
报告中提到：做好服务保障工作，是习近平总书记和以习近平同志为核心的党中央交给青岛的一项重大政治任务。全市上下坚决贯彻落实习近平总书记重要指示精神，有力有序、精心细致做好场馆改造提升、会务礼宾服务、交通安全保卫工作，同步实施城市环境提升行动，圆满实现了“世界水准、中国气派、山东风格、青岛特色”的服务保障目标，习近平总书记作出重要指示给予充分肯定，与会各方和国内外给予广泛赞誉。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[5-6]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> 主要做了以下工作：
>
> (一)团结一心众志成城，峰会服务保障任务圆满完成
>
> 上海合作组织成员国元首理事会第十八次会议在我市召开，做好服务保障工作，是习近平总书记和以习近平同志为核心的党中央交给青岛的一项重大政治任务。 全市上下坚决贯彻落实习近平总书记重
> 要指示精神，有力有序、精心细致做好场馆改造提升、会务礼宾服务、交通安全保卫工作，同步实施城市环境提升行动，圆满实现了“世界水准、中国气派、山东风格、青岛特色”的服务保障目标，习
> 近平总书记作出重要指示给予充分肯定，与会各方和国内外给予广泛赞誉。
- au=1.5910399042695644e-06, eu=0.28148138523101807, au′=-0.0, eu′=0.0376984067261219
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.526317596435547]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 5 | 5 | 25.526317596435547 | 主要做了以下工作：  (一)团结一心众志成城，峰会服务保障任务圆满完成  上海合作组织成员国元首理事会第十八次会议在我市召开，做好服务保障工作，是习近平总书记和以习近平同志为核心的党中央交给青岛的一项重大政治任务。 |
- GPT 引用1: span=[5, 6], relevant=1

### 陈述 2
所以，青岛市2018年举办的重大会议是上海合作组织成员国元首理事会第十八次会议。
- GPT support_score=0; [[Yes]]   Need Citation: [[Yes]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

## multifieldqa_zh_idx309 (idx=309, multifieldqa_zh)
### 问题
该产品对环境是否有害？

### 参考答案（数据集）
- ref1: 是的，该产品对水生生物毒性极大并具有长期持续影响。

### 模型全文输出
```
<statement>根据安全技术说明书，该产品对环境有害，主要表现在以下几个方面：<cite></cite></statement>

<statement>1. 对水生生物有毒性极大，并具有长期持续影响。对鱼类、水生无脊椎动物和水生植物的毒性都很高。<cite>[135-136]</cite></statement>

<statement>2. 产品未经测试，但生物降解和消除评价显示该产品不易生物降解。<cite>[140-140][142-142]</cite></statement>

<statement>3. 产品未经测试，但生物积累潜势评定显示该产品不会有生物积累性。<cite>[142-144]</cite></statement>

<statement>4. 产品对环境有害的物质包括乳油、石脑油、二甲戊乐灵等。这些物质对环境有害。<cite>[39-39][40-40][43-43]</cite></statement>

<statement>5. 产品未经测试，但根据其成分的性质，可能对环境造成污染。<cite>[39-43][135-140][142-144]</cite></statement>

<statement>6. 产品未经测试，但根据其成分的性质，可能会被生物累积。<cite>[142-144]</cite></statement>

<statement>7. 产品未经测试，但根据其成分的性质，可能会被环境降解。<cite>[140-140]</cite></statement>

<statement>8. 产品未经测试，但根据其成分的性质，可能会被环境迁移。<cite>[138-138]</cite></statement>

<statement>综上，该产品对环境有害，主要表现在对水生生物有毒性极大，并可能对环境造成污染、被生物累积和迁移。但需要注意，该产品未经测试，实际情况可能与评估结果不同。<cite></cite></statement>
```

文档句索引范围: sid ∈ [1, 166], 共 166 句。
引用指标: R=0.45, P=0.8333333333333334, F1=0.5844155844155844
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=11113, completion_tokens=159
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据安全技术说明书，该产品对环境有害，主要表现在以下几个方面：
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 1
1. 对水生生物有毒性极大，并具有长期持续影响。对鱼类、水生无脊椎动物和水生植物的毒性都很高。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[135-136]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> 生态学信息生态毒性水生毒性评价:对水生生物毒性极大并具有长期持续影响。 对鱼类的毒性:半致死浓度 (96 h) 1.5 mg/l, 虹鳟（Oncorhynchus
> mykiss） (EPA 72-1, 静态的，静电的)水生无脊椎动物:半有效浓度 (48 h) 1.76 mg/l, 大型蚤 (经济合作开发组织方针 202 第1部分,
> 静态的，静电的)水生植物:半有效浓度 (72 h) 0.074 mg/l (生长率), Chlamydomonas reinhardii10％有效浓度 (72 h) 0.022
> mg/l (生长率), Chlamydomonas reinhardii迁移率对化学品在不同环境介质间转换的评估:产品未经测试。
- au=0.6820714473724365, eu=0.18095232546329498, au′=0.6820706725120544, eu′=0.0423162542283535
- top_logits 方差 (pvariance, 有限 logit): 0.1558167386256173
- top_logits 数值: `[23.026315689086914, 22.23684310913086]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 9 | 9 | 23.026315689086914 | 对水生生物毒性极大。 |
| 2 | 135 | 135 | 22.23684310913086 | 生态学信息生态毒性水生毒性评价:对水生生物毒性极大并具有长期持续影响。 |
- GPT 引用1: span=[135, 136], relevant=1

### 陈述 2
2. 产品未经测试，但生物降解和消除评价显示该产品不易生物降解。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[140-140]`
- au=1.6802155187178869e-06, eu=0.2992125153541565, au′=-0.0, eu′=0.04094827175140381
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.421052932739258]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 140 | 140 | 23.421052932739258 | ---------------------------------- 巴斯夫 安全技术说明书日期 / 本次修订: 04.07.2023产品: Stomp New 330 EC版本: 4.0页: 12/14(30448922/SDS_CPA_CN/ZH)印刷日期 05.07.2023持续性和可降解性生物降解和消除评价（H20):产品未经测试。 |

**跨度** `[142-142]`
- au=1.6175589507838595e-06, eu=0.2867923676967621, au′=-0.0, eu′=0.03865716978907585
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.86842155456543]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 142 | 142 | 24.86842155456543 | 物质信息： N-(1-乙基丙基)-3,4-二甲基-2,6-二硝基苯胺生物降解和消除评价（H20):不易生物降解（根据经济合作开发组织OECD标准）----------------------------------生物积累潜势潜在生物体内积累评定:产品未经测试。 |
- GPT 引用1: span=[140, 140], relevant=1
- GPT 引用2: span=[142, 142], relevant=1

### 陈述 3
3. 产品未经测试，但生物积累潜势评定显示该产品不会有生物积累性。
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[142-144]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> 物质信息： N-(1-乙基丙基)-3,4-二甲基-2,6-二硝基苯胺生物降解和消除评价（H20):不易生物降解（根据经济合作开发组织OECD标准）---------------
> -------------------生物积累潜势潜在生物体内积累评定:产品未经测试。 本声明基于单个组分的性质。 物质信息：
> N-(1-乙基丙基)-3,4-二甲基-2,6-二硝基苯胺生物积累潜势:生物浓缩因子: 3,300根据有力的证据，该化合物不会有生物积累性。
- au=1.729960786178708e-06, eu=0.3089430034160614, au′=-0.0, eu′=0.04279278963804245
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[22.36842155456543]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 142 | 142 | 22.36842155456543 | 物质信息： N-(1-乙基丙基)-3,4-二甲基-2,6-二硝基苯胺生物降解和消除评价（H20):不易生物降解（根据经济合作开发组织OECD标准）----------------------------------生物积累潜势潜在生物体内积累评定:产品未经测试。 |
- GPT 引用1: span=[142, 144], relevant=1

### 陈述 4
4. 产品对环境有害的物质包括乳油、石脑油、二甲戊乐灵等。这些物质对环境有害。
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[39-39]`
- au=1.698429286989267e-06, eu=0.30278876423835754, au′=-0.0, eu′=0.041621025651693344
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.026315689086914]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 39 | 39 | 23.026315689086914 | 成分/组成信息化学性质: 混合物植物保护产品, 除草剂, 乳油危险组分N-(1-乙基丙基)-3,4-二甲基-2,6-二硝基苯胺含量 (W/W): 31.55 %CAS No. |

**跨度** `[40-40]`
- au=1.08339524269104, eu=0.1319444179534912, au′=1.0833951234817505, eu′=0.0436113178730011
- top_logits 方差 (pvariance, 有限 logit): 0.16543498146428545
- top_logits 数值: `[22.5, 21.710527420043945, 21.578948974609375]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 40 | 40 | 22.5 | : 40487-42-1重芳烃溶剂石脑油(石油)急性毒性: 分类 5 (口服)对生殖有毒性: 分类 2 (unborn child)对水环境的急性危害: 分类 1对水环境的慢性危害: 分类 1M-系数 急性: 100M-系数 慢性: 10 巴斯夫 安全技术说明书日期 / 本次修订: 04.07.2023产品: Stomp New 330 EC版本: 4.0页: 4/14(30448922/SDS_CPA_CN/ZH)印刷日期 05.07.2023含量 (W/W): < 60 %CAS No. |
| 2 | 41 | 41 | 21.710527420043945 | : 64742-94-5吸入危害: 分类 1对水环境的急性危害: 分类 2对水环境的慢性危害: 分类 2四丙烯苯磺酸钙盐含量 (W/W): < 5 %CAS No. |
| 3 | 135 | 135 | 21.578948974609375 | 生态学信息生态毒性水生毒性评价:对水生生物毒性极大并具有长期持续影响。 |

**跨度** `[43-43]`
- au=0.6817837357521057, eu=0.1840193271636963, au′=0.6817826628684998, eu′=0.043157294392585754
- top_logits 方差 (pvariance, 有限 logit): 0.21208462622689694
- top_logits 数值: `[22.631580352783203, 21.710527420043945]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 41 | 41 | 22.631580352783203 | : 64742-94-5吸入危害: 分类 1对水环境的急性危害: 分类 2对水环境的慢性危害: 分类 2四丙烯苯磺酸钙盐含量 (W/W): < 5 %CAS No. |
| 2 | 43 | 43 | 21.710527420043945 | : 78-83-1易燃液体: 分类 3急性毒性: 分类 4 (皮肤接触)皮肤腐蚀/刺激: 分类 1B严重损伤/刺激眼睛: 分类 1对水环境的急性危害: 分类 3对水环境的慢性危害: 分类 3吸入危害: 分类 2易燃液体: 分类 3急性毒性: 分类 5 (口服)急性毒性: 分类 5 (皮肤接触)皮肤腐蚀/刺激: 分类 2严重损伤/刺激眼睛: 分类 1特异性靶器官毒性-一次接触: 分类 3 (嗜睡及眩晕 )特异性靶器官毒性-一次接触: 分类 3 (对呼吸道系 统有刺激性)4. |
- GPT 引用1: span=[39, 40], relevant=1
- GPT 引用2: span=[43, 43], relevant=0

### 陈述 5
5. 产品未经测试，但根据其成分的性质，可能对环境造成污染。
- GPT support_score=0.5; **Rating**: [[Partially supported]]

**跨度** `[39-43]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> 成分/组成信息化学性质: 混合物植物保护产品, 除草剂, 乳油危险组分N-(1-乙基丙基)-3,4-二甲基-2,6-二硝基苯胺含量 (W/W): 31.55 %CAS No.
> : 40487-42-1重芳烃溶剂石脑油(石油)急性毒性: 分类 5 (口服)对生殖有毒性: 分类 2 (unborn child)对水环境的急性危害: 分类
> 1对水环境的慢性危害: 分类 1M-系数 急性: 100M-系数 慢性: 10 巴斯夫 安全技术说明书日期 / 本次修订: 04.07.2023产品: Stomp New
> 330 EC版本: 4.0页: 4/14(30448922/SDS_CPA_CN/ZH)印刷日期 05.07.2023含量 (W/W): < 60 %CAS No. :
> 64742-94-5吸入危害: 分类 1对水环境的急性危害: 分类 2对水环境的慢性危害: 分类 2四丙烯苯磺酸钙盐含量 (W/W): < 5 %CAS No. :
> 11117-11-6异丁醇含量 (W/W): < 3 %CAS No. : 78-83-1易燃液体: 分类 3急性毒性: 分类 4 (皮肤接触)皮肤腐蚀/刺激: 分类
> 1B严重损伤/刺激眼睛: 分类 1对水环境的急性危害: 分类 3对水环境的慢性危害: 分类 3吸入危害: 分类 2易燃液体: 分类 3急性毒性: 分类 5 (口服)急性毒性:
> 分类 5 (皮肤接触)皮肤腐蚀/刺激: 分类 2严重损伤/刺激眼睛: 分类 1特异性靶器官毒性-一次接触: 分类 3 (嗜睡及眩晕 )特异性靶器官毒性-一次接触: 分类 3
> (对呼吸道系 统有刺激性)4.
- au=1.3661795854568481, eu=0.11394301056861877, au′=1.3661794662475586, eu′=0.04892178252339363
- top_logits 方差 (pvariance, 有限 logit): 0.7909880912866356
- top_logits 数值: `[20.921052932739258, 19.342105865478516, 18.8157901763916, 18.68421173095703]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 140 | 140 | 20.921052932739258 | ---------------------------------- 巴斯夫 安全技术说明书日期 / 本次修订: 04.07.2023产品: Stomp New 330 EC版本: 4.0页: 12/14(30448922/SDS_CPA_CN/ZH)印刷日期 05.07.2023持续性和可降解性生物降解和消除评价（H20):产品未经测试。 |
| 2 | 40 | 40 | 19.342105865478516 | : 40487-42-1重芳烃溶剂石脑油(石油)急性毒性: 分类 5 (口服)对生殖有毒性: 分类 2 (unborn child)对水环境的急性危害: 分类 1对水环境的慢性危害: 分类 1M-系数 急性: 100M-系数 慢性: 10 巴斯夫 安全技术说明书日期 / 本次修订: 04.07.2023产品: Stomp New 330 EC版本: 4.0页: 4/14(30448922/SDS_CPA_CN/ZH)印刷日期 05.07.2023含量 (W/W): < 60 %CAS No. |
| 3 | 142 | 142 | 18.8157901763916 | 物质信息： N-(1-乙基丙基)-3,4-二甲基-2,6-二硝基苯胺生物降解和消除评价（H20):不易生物降解（根据经济合作开发组织OECD标准）----------------------------------生物积累潜势潜在生物体内积累评定:产品未经测试。 |
| 4 | 39 | 39 | 18.68421173095703 | 成分/组成信息化学性质: 混合物植物保护产品, 除草剂, 乳油危险组分N-(1-乙基丙基)-3,4-二甲基-2,6-二硝基苯胺含量 (W/W): 31.55 %CAS No. |

**跨度** `[135-140]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> 生态学信息生态毒性水生毒性评价:对水生生物毒性极大并具有长期持续影响。 对鱼类的毒性:半致死浓度 (96 h) 1.5 mg/l, 虹鳟（Oncorhynchus
> mykiss） (EPA 72-1, 静态的，静电的)水生无脊椎动物:半有效浓度 (48 h) 1.76 mg/l, 大型蚤 (经济合作开发组织方针 202 第1部分,
> 静态的，静电的)水生植物:半有效浓度 (72 h) 0.074 mg/l (生长率), Chlamydomonas reinhardii10％有效浓度 (72 h) 0.022
> mg/l (生长率), Chlamydomonas reinhardii迁移率对化学品在不同环境介质间转换的评估:产品未经测试。 本声明基于单个组分的性质。 物质信息：
> N-(1-乙基丙基)-3,4-二甲基-2,6-二硝基苯胺对化学品在不同环境介质间转换的评估:物质将从水表缓慢蒸发到大气中。
> 接触土壤后，可能吸附在固态土壤颗粒上，因此预计不会对地下水造成污染。 ---------------------------------- 巴斯夫 安全技术说明书日期 /
> 本次修订: 04.07.2023产品: Stomp New 330 EC版本: 4.0页: 12/14(30448922/SDS_CPA_CN/ZH)印刷日期
> 05.07.2023持续性和可降解性生物降解和消除评价（H20):产品未经测试。
- au=1.0819092988967896, eu=0.13944950699806213, au′=1.0819083452224731, eu′=0.04636029899120331
- top_logits 方差 (pvariance, 有限 logit): 0.5694051961529314
- top_logits 数值: `[21.447368621826172, 20.657896041870117, 19.60526466369629]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 135 | 135 | 21.447368621826172 | 生态学信息生态毒性水生毒性评价:对水生生物毒性极大并具有长期持续影响。 |
| 2 | 140 | 140 | 20.657896041870117 | ---------------------------------- 巴斯夫 安全技术说明书日期 / 本次修订: 04.07.2023产品: Stomp New 330 EC版本: 4.0页: 12/14(30448922/SDS_CPA_CN/ZH)印刷日期 05.07.2023持续性和可降解性生物降解和消除评价（H20):产品未经测试。 |
| 3 | 40 | 40 | 19.60526466369629 | : 40487-42-1重芳烃溶剂石脑油(石油)急性毒性: 分类 5 (口服)对生殖有毒性: 分类 2 (unborn child)对水环境的急性危害: 分类 1对水环境的慢性危害: 分类 1M-系数 急性: 100M-系数 慢性: 10 巴斯夫 安全技术说明书日期 / 本次修订: 04.07.2023产品: Stomp New 330 EC版本: 4.0页: 4/14(30448922/SDS_CPA_CN/ZH)印刷日期 05.07.2023含量 (W/W): < 60 %CAS No. |

**跨度** `[142-144]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> 物质信息： N-(1-乙基丙基)-3,4-二甲基-2,6-二硝基苯胺生物降解和消除评价（H20):不易生物降解（根据经济合作开发组织OECD标准）---------------
> -------------------生物积累潜势潜在生物体内积累评定:产品未经测试。 本声明基于单个组分的性质。 物质信息：
> N-(1-乙基丙基)-3,4-二甲基-2,6-二硝基苯胺生物积累潜势:生物浓缩因子: 3,300根据有力的证据，该化合物不会有生物积累性。
- au=1.6742565094318707e-06, eu=0.29803910851478577, au′=-0.0, eu′=0.04072882607579231
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.55263328552246]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 142 | 142 | 23.55263328552246 | 物质信息： N-(1-乙基丙基)-3,4-二甲基-2,6-二硝基苯胺生物降解和消除评价（H20):不易生物降解（根据经济合作开发组织OECD标准）----------------------------------生物积累潜势潜在生物体内积累评定:产品未经测试。 |
- GPT 引用1: span=[39, 43], relevant=1
- GPT 引用2: span=[135, 140], relevant=1
- GPT 引用3: span=[142, 144], relevant=1

### 陈述 6
6. 产品未经测试，但根据其成分的性质，可能会被生物累积。
- GPT support_score=0; Rating: [[No support]]

**跨度** `[142-144]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> 物质信息： N-(1-乙基丙基)-3,4-二甲基-2,6-二硝基苯胺生物降解和消除评价（H20):不易生物降解（根据经济合作开发组织OECD标准）---------------
> -------------------生物积累潜势潜在生物体内积累评定:产品未经测试。 本声明基于单个组分的性质。 物质信息：
> N-(1-乙基丙基)-3,4-二甲基-2,6-二硝基苯胺生物积累潜势:生物浓缩因子: 3,300根据有力的证据，该化合物不会有生物积累性。
- au=1.0816408395767212, eu=0.14559385180473328, au′=1.0816398859024048, eu′=0.04863480478525162
- top_logits 方差 (pvariance, 有限 logit): 0.09618326494172733
- top_logits 数值: `[20.0, 19.342105865478516, 19.342105865478516]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 142 | 142 | 20.0 | 物质信息： N-(1-乙基丙基)-3,4-二甲基-2,6-二硝基苯胺生物降解和消除评价（H20):不易生物降解（根据经济合作开发组织OECD标准）----------------------------------生物积累潜势潜在生物体内积累评定:产品未经测试。 |
| 2 | 135 | 135 | 19.342105865478516 | 生态学信息生态毒性水生毒性评价:对水生生物毒性极大并具有长期持续影响。 |
| 3 | 39 | 39 | 19.342105865478516 | 成分/组成信息化学性质: 混合物植物保护产品, 除草剂, 乳油危险组分N-(1-乙基丙基)-3,4-二甲基-2,6-二硝基苯胺含量 (W/W): 31.55 %CAS No. |
- GPT 引用1: span=[142, 144], relevant=0

### 陈述 7
7. 产品未经测试，但根据其成分的性质，可能会被环境降解。
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[140-140]`
- au=1.974869974219473e-06, eu=0.3102039694786072, au′=-0.0, eu′=0.043035101145505905
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[22.23684310913086]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 140 | 140 | 22.23684310913086 | ---------------------------------- 巴斯夫 安全技术说明书日期 / 本次修订: 04.07.2023产品: Stomp New 330 EC版本: 4.0页: 12/14(30448922/SDS_CPA_CN/ZH)印刷日期 05.07.2023持续性和可降解性生物降解和消除评价（H20):产品未经测试。 |
- GPT 引用1: span=[140, 140], relevant=1

### 陈述 8
8. 产品未经测试，但根据其成分的性质，可能会被环境迁移。
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[138-138]`
- au=1.7630600268603303e-06, eu=0.3153526186943054, au′=-0.0, eu′=0.04403243958950043
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[21.710527420043945]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 138 | 138 | 21.710527420043945 | 物质信息： N-(1-乙基丙基)-3,4-二甲基-2,6-二硝基苯胺对化学品在不同环境介质间转换的评估:物质将从水表缓慢蒸发到大气中。 |
- GPT 引用1: span=[138, 138], relevant=1

### 陈述 9
综上，该产品对环境有害，主要表现在对水生生物有毒性极大，并可能对环境造成污染、被生物累积和迁移。但需要注意，该产品未经测试，实际情况可能与评估结果不同。
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

## multifieldqa_zh_idx310 (idx=310, multifieldqa_zh)
### 问题
市的“双百工程”有多少个项目正在建设中？

### 参考答案（数据集）
- ref1: 市的“双百工程”有246个项目正在建设中。

### 模型全文输出
```
<statement>根据政府工作报告中的相关内容，市的“双百工程”项目在建246个。<cite>[12-12]</cite></statement>

<statement>相关原文：<cite></cite></statement>

<statement>“双百工程”项目在建246个，竣工42个，完成投资197.4亿元。<cite>[12-12]</cite></statement>
```

文档句索引范围: sid ∈ [1, 101], 共 101 句。
引用指标: R=1.0, P=1.0, F1=1.0
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=1430, completion_tokens=36
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据政府工作报告中的相关内容，市的“双百工程”项目在建246个。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[12-12]`
- au=1.9246485862822738e-06, eu=0.3003951609134674, au′=-0.0, eu′=0.04117009416222572
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.289474487304688]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 12 | 12 | 23.289474487304688 | “双百工程”项目在建246个，竣工42个，完成投资197.4亿元。 |
- GPT 引用1: span=[12, 12], relevant=1

### 陈述 1
相关原文：
- GPT support_score=1; [[No]]   Need Citation: [[No]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 2
“双百工程”项目在建246个，竣工42个，完成投资197.4亿元。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[12-12]`
- au=1.6175589507838595e-06, eu=0.2867923676967621, au′=-0.0, eu′=0.03865716978907585
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.86842155456543]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 12 | 12 | 24.86842155456543 | “双百工程”项目在建246个，竣工42个，完成投资197.4亿元。 |
- GPT 引用1: span=[12, 12], relevant=1

## multifieldqa_zh_idx311 (idx=311, multifieldqa_zh)
### 问题
这段文字主要是关于什么产品手册的内容介绍？

### 参考答案（数据集）
- ref1: 这段文字主要是关于状元魔方K12产品手册的内容介绍。

### 模型全文输出
```
<statement>根据文档内容可以看出，这段文字是关于状元魔方K12产品手册的内容介绍。<cite>[1-1]</cite></statement>
```

文档句索引范围: sid ∈ [1, 18], 共 18 句。
引用指标: R=1.0, P=1.0, F1=1.0
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=788, completion_tokens=13
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据文档内容可以看出，这段文字是关于状元魔方K12产品手册的内容介绍。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[1-1]`
- au=1.7430054413125617e-06, eu=0.31147530674934387, au′=-0.0, eu′=0.04328017681837082
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[22.10526466369629]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 1 | 1 | 22.10526466369629 | '状元魔方K12产品手册 Ver:NF1.6  六、 腾讯教育应用平台 快速入门 平台介绍 软件设置 应用管理 七、 服务与支持 保修服务条例 三包服务说明 12 12 13 14 15 16 16 16 概述 售后服务流程 发回件注意事项 售后服务登记卡 维修记录 16 18 19 19 20  1.笔 记 本电脑*1 5.附件盒*1 2. |
- GPT 引用1: span=[1, 1], relevant=1

## multifieldqa_zh_idx312 (idx=312, multifieldqa_zh)
### 问题
富兰克林·罗斯福是怎么患残疾的？

### 参考答案（数据集）
- ref1: 富兰克林·罗斯福在扑灭了一场林火后，因跳进了冰冷的海水，而患上了脊髓灰质炎症。

### 模型全文输出
```
<statement>根据上文，富兰克林·罗斯福在1921年8月在坎波贝洛岛休假时，扑灭了一场林火后跳进了冰冷的海水，因此患上了脊髓灰质炎症。<cite>[53-53]</cite></statement>

<statement>所以他是通过在海水中过凉导致脊髓灰质炎的。<cite></cite></statement>
```

文档句索引范围: sid ∈ [1, 236], 共 236 句。
引用指标: R=0.5, P=1.0, F1=0.6666666666666666
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=1011, completion_tokens=23
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据上文，富兰克林·罗斯福在1921年8月在坎波贝洛岛休假时，扑灭了一场林火后跳进了冰冷的海水，因此患上了脊髓灰质炎症。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[53-53]`
- au=1.6284955108858412e-06, eu=0.28897327184677124, au′=-0.0, eu′=0.03905446454882622
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.60526466369629]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 53 | 53 | 24.60526466369629 | 1921年8月，富兰克林·罗斯福带全家在坎波贝洛岛休假，在扑灭了一场林火后，他跳进了冰冷的海水，因此患上了脊髓灰质炎症。 |
- GPT 引用1: span=[53, 53], relevant=1

### 陈述 1
所以他是通过在海水中过凉导致脊髓灰质炎的。
- GPT support_score=0; [[Yes]]   Need Citation: [[Yes]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

## multifieldqa_zh_idx313 (idx=313, multifieldqa_zh)
### 问题
纪念碑的后面介绍了什么？

### 参考答案（数据集）
- ref1: 介绍了承平宁抗日游击根据地的历史。

### 模型全文输出
```
<statement>根据文章内容，纪念碑的后面简单介绍了承平宁抗日游击根据地的历史。<cite></cite></statement>

<statement>具体来说，文章提到：<cite></cite></statement>

<statement>纪念碑的后面，简单介绍了承平宁抗日游击根据地的历史。这段文字言短意赅，表达了老区人民对抗战时期革命先烈的崇敬和缅怀之情。<cite>[10-11]</cite></statement>

<statement>所以纪念碑的后面介绍了承平宁抗日游击根据地的历史。<cite></cite></statement>
```

文档句索引范围: sid ∈ [1, 59], 共 59 句。
引用指标: R=0.5, P=1.0, F1=0.6666666666666666
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=1633, completion_tokens=43
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据文章内容，纪念碑的后面简单介绍了承平宁抗日游击根据地的历史。
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 1
具体来说，文章提到：
- GPT support_score=1; [[No]]   Need Citation: [[No]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 2
纪念碑的后面，简单介绍了承平宁抗日游击根据地的历史。这段文字言短意赅，表达了老区人民对抗战时期革命先烈的崇敬和缅怀之情。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[10-11]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> 纪念碑的后面，简单介绍了承平宁抗日游击根据地的历史。 这段文字言短意赅，表达了老区人民对抗战时期革命先烈的崇敬和缅怀之情 [1] 。
- au=1.9009210063813953e-06, eu=0.2957197427749634, au′=-0.0, eu′=0.04029691964387894
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.8157901763916]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 10 | 10 | 23.8157901763916 | 纪念碑的后面，简单介绍了承平宁抗日游击根据地的历史。 |
- GPT 引用1: span=[10, 11], relevant=1

### 陈述 3
所以纪念碑的后面介绍了承平宁抗日游击根据地的历史。
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

## multifieldqa_zh_idx314 (idx=314, multifieldqa_zh)
### 问题
2018年青岛市的城镇登记失业率是多少？

### 参考答案（数据集）
- ref1: 2.94%。

### 模型全文输出
```
<statement>根据报告内容，2018年青岛市的城镇登记失业率是2.94%。<cite>[4-4]</cite></statement>
```

文档句索引范围: sid ∈ [1, 89], 共 89 句。
引用指标: R=1.0, P=1.0, F1=1.0
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=710, completion_tokens=13
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据报告内容，2018年青岛市的城镇登记失业率是2.94%。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[4-4]`
- au=1.6742565094318707e-06, eu=0.29803910851478577, au′=-0.0, eu′=0.04072882607579231
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.55263328552246]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 4 | 4 | 23.55263328552246 | 预计,全市生产总值实现1.2万亿元左右，同比增长7.5%左右;一般公共预算收入达到1231.9亿元，增长6.5%;全体居民人均可支配收入增长8%以上;城镇登记失业率2.94%;居民消费价格上涨2.1%;节能减排降碳任务全面完成。 |
- GPT 引用1: span=[4, 4], relevant=1

## multifieldqa_zh_idx315 (idx=315, multifieldqa_zh)
### 问题
使用什么墨盒可以取得最佳效果？

### 参考答案（数据集）
- ref1: 原装Brother墨盒。

### 模型全文输出
```
<statement>根据产品手册，建议使用Brother原装墨盒以便取得最佳效果。<cite>[27-27]</cite></statement>

<statement>手册中提到：<cite></cite></statement>

<statement>"请使用Brother原装墨盒Brother多功能一体机使用特定墨水工作，建议使用原装Brother墨盒以便取得最佳效果。如果使用其他品牌墨水或墨盒，Brother将不保证取得最佳效果。不推荐使用Brother原装墨盒以外的其他墨盒，也不推荐使用其他来源的墨水来填充Brother墨盒。如果由于使用与本产品不兼容或非Brother原装墨盒而对打印头或其他部件造成损坏，都不在保修范围之内。"<cite>[27-30]</cite></statement>

<statement>所以建议使用Brother原装墨盒以取得最佳效果。<cite></cite></statement>
```

文档句索引范围: sid ∈ [1, 227], 共 227 句。
引用指标: R=0.75, P=1.0, F1=0.8571428571428571
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=2273, completion_tokens=46
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据产品手册，建议使用Brother原装墨盒以便取得最佳效果。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[27-27]`
- au=1.7108611700678011e-06, eu=0.3052207827568054, au′=-0.0, eu′=0.04208194464445114
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[22.763158798217773]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 27 | 27 | 22.763158798217773 | 请使用 Brother 原装墨盒 Brother 多功能一体机使用特定墨水工作，建议使用原装 Brother 墨盒以便取得最 佳效果。 |
- GPT 引用1: span=[27, 27], relevant=1

### 陈述 1
手册中提到：
- GPT support_score=1; [[No]]  Need Citation: [[No]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 2
"请使用Brother原装墨盒Brother多功能一体机使用特定墨水工作，建议使用原装Brother墨盒以便取得最佳效果。如果使用其他品牌墨水或墨盒，Brother将不保证取得最佳效果。不推荐使用Brother原装墨盒以外的其他墨盒，也不推荐使用其他来源的墨水来填充Brother墨盒。如果由于使用与本产品不兼容或非Brother原装墨盒而对打印头或其他部件造成损坏，都不在保修范围之内。"
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[27-30]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> 请使用 Brother 原装墨盒 Brother 多功能一体机使用特定墨水工作，建议使用原装 Brother 墨盒以便取得最 佳效果。 如果使用其他品牌墨水或墨盒，
> Brother 将不保证取得最佳效果。 不推 荐使用 Brother 原装墨盒以外的其他墨盒，也不推荐使用其他来源的墨水来填充Brother 墨盒。
> 如果由于使用与本产品不兼容或非 Brother 原装墨盒而对打印头或 其他部件造成损坏，都不在保修范围之内。
- au=1.5910399042695644e-06, eu=0.28148138523101807, au′=-0.0, eu′=0.0376984067261219
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.526317596435547]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 27 | 27 | 25.526317596435547 | 请使用 Brother 原装墨盒 Brother 多功能一体机使用特定墨水工作，建议使用原装 Brother 墨盒以便取得最 佳效果。 |
- GPT 引用1: span=[27, 30], relevant=1

### 陈述 3
所以建议使用Brother原装墨盒以取得最佳效果。
- GPT support_score=0; [[Yes]]   Need Citation: [[Yes]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

## multifieldqa_zh_idx316 (idx=316, multifieldqa_zh)
### 问题
鲁明公司成立多久了？

### 参考答案（数据集）
- ref1: 鲁明公司成立了15年。

### 模型全文输出
```
<statement>根据文章内容，鲁明公司成立15周年。<cite>[1-1]</cite></statement>
```

文档句索引范围: sid ∈ [1, 56], 共 56 句。
引用指标: R=0.5, P=1.0, F1=0.6666666666666666
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=654, completion_tokens=14
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据文章内容，鲁明公司成立15周年。
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[1-1]`
- au=0.6818511486053467, eu=0.18313249945640564, au′=0.6818503141403198, eu′=0.04291360080242157
- top_logits 方差 (pvariance, 有限 logit): 0.21208462622689694
- top_logits 数值: `[22.763158798217773, 21.842105865478516]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 1 | 1 | 22.763158798217773 | 'C04 重点 2023年7月5日 星期三 15年累计生产原油800万吨，鲁明公司初心如磐 勇做“百万鲁明”赛道上的追梦人 本 报7 月4 日 讯( 特 约 记 者 番、效益翻倍。 |
| 2 | 5 | 5 | 21.842105865478516 | 近日，胜利油田鲁明公司召 开成立十五周年座谈会，回顾15 周年风雨历程，聚焦“闯、敢、算、 智、和”五个字，凝练发布“五个好 传统”，即“闯天地南北，战难动极 限，探千米黑金”“敢走改革路，善 打创新牌，勇向潮头立”“先算后 干，边算边干，干完再算”“‘智’采 清洁能源，‘智’撑管理现代”“融 百家事、聚千方力、用天下才、和 万众心”。 |
- GPT 引用1: span=[1, 1], relevant=1

## multifieldqa_zh_idx317 (idx=317, multifieldqa_zh)
### 问题
鄂尔多斯市建成了多大的生态自然恢复区？

### 参考答案（数据集）
- ref1: 2.13万平方公里。

### 模型全文输出
```
<statement>根据鄂尔多斯市政府工作报告，鄂尔多斯市建成了生态自然恢复区2.13万平方公里。<cite>[39-39]</cite></statement>
```

文档句索引范围: sid ∈ [1, 99], 共 99 句。
引用指标: R=1.0, P=1.0, F1=1.0
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=626, completion_tokens=13
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据鄂尔多斯市政府工作报告，鄂尔多斯市建成了生态自然恢复区2.13万平方公里。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[39-39]`
- au=1.861421651483397e-06, eu=0.28787869215011597, au′=-0.0, eu′=0.038854800164699554
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.73684310913086]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 39 | 39 | 24.73684310913086 | 加大生态环境保护与建设，建成生态自然恢复区2.13万平方公里，生态环境持续好转。 |
- GPT 引用1: span=[39, 39], relevant=1

## multifieldqa_zh_idx318 (idx=318, multifieldqa_zh)
### 问题
嵩山少林寺武术馆每天有几个场次的武术表演？

### 参考答案（数据集）
- ref1: 7个场次。

### 模型全文输出
```
<statement>根据文档中的相关描述，嵩山少林寺武术馆每天有以下几个场次的武术表演：<cite></cite></statement>

<statement>上午：9:30、10:30、11:30<cite>[50-50]</cite></statement>

<statement>下午：3:00、4:00、5:00<cite>[3-3]</cite></statement>

<statement>每场表演30分钟，免费观看。<cite>[4-4][51-51]</cite></statement>
```

文档句索引范围: sid ∈ [1, 186], 共 186 句。
引用指标: R=0.875, P=0.75, F1=0.8076923076923077
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=2514, completion_tokens=61
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据文档中的相关描述，嵩山少林寺武术馆每天有以下几个场次的武术表演：
- GPT support_score=1; [[No]]   Need Citation: [[No]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 1
上午：9:30、10:30、11:30
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[50-50]`
- au=1.6121624639708898e-06, eu=0.28571420907974243, au′=-0.0, eu′=0.038461536169052124
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.0]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 50 | 50 | 25.0 | 武术馆 每天都有固定场次武术表演，分别为上午09：30、10:30、11:30； |
- GPT 引用1: span=[50, 50], relevant=1

### 陈述 2
下午：3:00、4:00、5:00
- GPT support_score=1; **Rating**: [[Fully supported]]   **

**跨度** `[3-3]`
- au=1.0835727453231812, eu=0.1299145221710205, au′=1.0835723876953125, eu′=0.04287325590848923
- top_logits 方差 (pvariance, 有限 logit): 0.257771161198131
- top_logits 数值: `[23.026315689086914, 22.10526466369629, 21.842105865478516]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 50 | 50 | 23.026315689086914 | 武术馆 每天都有固定场次武术表演，分别为上午09：30、10:30、11:30； |
| 2 | 4 | 4 | 22.10526466369629 | 每场表演30分钟。 |
| 3 | 3 | 3 | 21.842105865478516 | 郑州—少 林寺—龙门石窟—郑州 07：20左右导游会到您下踏的酒店接您 08：00乘车 出发赴嵩山少林寺景区，全程高速抵达景区（郑少高速全程1.5小时） 09：30-10：00左右抵达景区，参观少林寺武术表演（上午三场：9：30、10：30、11：30，下 午三场：3点、4点、5点； |
- GPT 引用1: span=[3, 3], relevant=1

### 陈述 3
每场表演30分钟，免费观看。
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[4-4]`
- au=1.6742565094318707e-06, eu=0.29803910851478577, au′=-0.0, eu′=0.04072882607579231
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.55263328552246]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 4 | 4 | 23.55263328552246 | 每场表演30分钟。 |

**跨度** `[51-51]`
- au=1.5807482895979774e-06, eu=0.2794116735458374, au′=-0.0, eu′=0.037328090518713
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.789474487304688]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 51 | 51 | 25.789474487304688 | 下午14:00、15:00、16:00、17:00，凭景 区门票免费观看。 |
- GPT 引用1: span=[4, 4], relevant=1
- GPT 引用2: span=[51, 51], relevant=0

## multifieldqa_zh_idx319 (idx=319, multifieldqa_zh)
### 问题
2005年全市新增了多少个农产品获得绿色食品标志使用权？

### 参考答案（数据集）
- ref1: 2005年全市新增了3个农产品获得绿色食品标志使用权。

### 模型全文输出
```
<statement>根据报告内容，2005年全市新增了3个农产品获绿色食品标志使用权。<cite>[20-20]</cite></statement>
```

文档句索引范围: sid ∈ [1, 197], 共 197 句。
引用指标: R=1.0, P=1.0, F1=1.0
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=614, completion_tokens=13
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据报告内容，2005年全市新增了3个农产品获绿色食品标志使用权。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[20-20]`
- au=1.9246485862822738e-06, eu=0.3003951609134674, au′=-0.0, eu′=0.04117009416222572
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.289474487304688]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 20 | 20 | 23.289474487304688 | 全市新增3个农产品获绿色食品标志使用权、12个农产品获无公害食品认证； |
- GPT 引用1: span=[20, 20], relevant=1

## multifieldqa_zh_idx320 (idx=320, multifieldqa_zh)
### 问题
故事发生在哪个时代和地点？

### 参考答案（数据集）
- ref1: 故事发生在二十一世纪末的日本。

### 模型全文输出
```
<statement>根据文中内容，《魔法科高中的劣等生》故事发生在：<cite></cite></statement>

<statement>- 二十一世纪末的日本。<cite>[4-4]</cite></statement>

<statement>- 故事发生在现代科学已经过一个世纪的日本。<cite>[5-6]</cite></statement>

<statement>- 魔法被应用至今已经过了一个世纪。<cite>[5-6]</cite></statement>

<statement>- 故事发生在国立魔法大学附属第一高校——通称“魔法科高校”。<cite>[9-9]</cite></statement>

<statement>- 故事发生在由成绩优秀的“一科生”,和作为一科生替补的“二科生”所构成的学校。<cite>[9-9]</cite></statement>

<statement>- 故事发生在一个魔法被应用至今已经过了一个世纪的现代日本。<cite>[5-7]</cite></statement>

<statement>- 故事发生在国立魔法大学附属第一高校——通称“魔法科高校”。<cite>[9-9]</cite></statement>

<statement>所以故事发生在二十一世纪末的日本。<cite></cite></statement>
```

文档句索引范围: sid ∈ [1, 156], 共 156 句。
引用指标: R=0.6666666666666666, P=0.8571428571428571, F1=0.75
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=5162, completion_tokens=117
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据文中内容，《魔法科高中的劣等生》故事发生在：
- GPT support_score=1; [[No]]   Need Citation: [[No]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 1
- 二十一世纪末的日本。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[4-4]`
- au=1.6452690942969639e-06, eu=0.29230761528015137, au′=-0.0, eu′=0.03966596722602844
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.210527420043945]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 4 | 4 | 24.210527420043945 | 故事内容 故事发生在二十一世纪末的日本。 |
- GPT 引用1: span=[4, 4], relevant=1

### 陈述 2
- 故事发生在现代科学已经过一个世纪的日本。
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[5-6]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> 魔法，这不是一个传说，而是现代科学的一项技术。 它被应用至今已经过了一个世纪。
- au=0.6825650334358215, eu=0.17431187629699707, au′=0.6825642585754395, eu′=0.04051172360777855
- top_logits 方差 (pvariance, 有限 logit): 0.1558174915262498
- top_logits 数值: `[24.078948974609375, 23.289474487304688]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 5 | 5 | 24.078948974609375 | 魔法，这不是一个传说，而是现代科学的一项技术。 |
| 2 | 6 | 6 | 23.289474487304688 | 它被应用至今已经过了一个世纪。 |
- GPT 引用1: span=[5, 6], relevant=1

### 陈述 3
- 魔法被应用至今已经过了一个世纪。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[5-6]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> 魔法，这不是一个传说，而是现代科学的一项技术。 它被应用至今已经过了一个世纪。
- au=0.6825781464576721, eu=0.17471259832382202, au′=0.68257737159729, eu′=0.04061998426914215
- top_logits 方差 (pvariance, 有限 logit): 0.10820617305944324
- top_logits 数值: `[23.947368621826172, 23.289474487304688]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 5 | 5 | 23.947368621826172 | 魔法，这不是一个传说，而是现代科学的一项技术。 |
| 2 | 6 | 6 | 23.289474487304688 | 它被应用至今已经过了一个世纪。 |
- GPT 引用1: span=[5, 6], relevant=1

### 陈述 4
- 故事发生在国立魔法大学附属第一高校——通称“魔法科高校”。
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[9-9]`
- au=1.6340368347300682e-06, eu=0.29007625579833984, au′=-0.0, eu′=0.03925619646906853
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.473684310913086]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 9 | 9 | 24.473684310913086 | 国立魔法大学附属第一高校——通称“魔法科高校”，是由成绩优秀的“一科生”，和作为一科生替补的“二科生”所构成，他们也各自被冠上了“花冠（bloom）”，以及“杂草（weed）”之名。 |
- GPT 引用1: span=[9, 9], relevant=1

### 陈述 5
- 故事发生在由成绩优秀的“一科生”,和作为一科生替补的“二科生”所构成的学校。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[9-9]`
- au=1.5962523320922628e-06, eu=0.28252777457237244, au′=-0.0, eu′=0.03788633644580841
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.394737243652344]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 9 | 9 | 25.394737243652344 | 国立魔法大学附属第一高校——通称“魔法科高校”，是由成绩优秀的“一科生”，和作为一科生替补的“二科生”所构成，他们也各自被冠上了“花冠（bloom）”，以及“杂草（weed）”之名。 |
- GPT 引用1: span=[9, 9], relevant=1

### 陈述 6
- 故事发生在一个魔法被应用至今已经过了一个世纪的现代日本。
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[5-7]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> 魔法，这不是一个传说，而是现代科学的一项技术。 它被应用至今已经过了一个世纪。
> 曾经被称为“超能力”的先天具有的能力被以“魔法”这个名字的体系替代，高超的魔法师被认为是国家的实力。
- au=1.9067709899900365e-06, eu=0.29687488079071045, au′=-0.0, eu′=0.04051172360777855
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.68421173095703]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 5 | 5 | 23.68421173095703 | 魔法，这不是一个传说，而是现代科学的一项技术。 |
- GPT 引用1: span=[5, 7], relevant=0

### 陈述 7
- 故事发生在国立魔法大学附属第一高校——通称“魔法科高校”。
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[9-9]`
- au=1.6452690942969639e-06, eu=0.29230761528015137, au′=-0.0, eu′=0.03966596722602844
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.210527420043945]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 9 | 9 | 24.210527420043945 | 国立魔法大学附属第一高校——通称“魔法科高校”，是由成绩优秀的“一科生”，和作为一科生替补的“二科生”所构成，他们也各自被冠上了“花冠（bloom）”，以及“杂草（weed）”之名。 |
- GPT 引用1: span=[9, 9], relevant=1

### 陈述 8
所以故事发生在二十一世纪末的日本。
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

## multifieldqa_zh_idx321 (idx=321, multifieldqa_zh)
### 问题
文婧是谁的女儿？

### 参考答案（数据集）
- ref1: 文婧是文太极与佰夷族族长女儿的独生女。

### 模型全文输出
```
<statement>根据角色介绍，文婧是文太极与佰夷族族长女儿的独生女。<cite>[20-20]</cite></statement>
```

文档句索引范围: sid ∈ [1, 193], 共 193 句。
引用指标: R=1.0, P=1.0, F1=1.0
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=646, completion_tokens=13
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据角色介绍，文婧是文太极与佰夷族族长女儿的独生女。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[20-20]`
- au=0.6824045777320862, eu=0.17757005989551544, au′=0.6824037432670593, eu′=0.04139433056116104
- top_logits 方差 (pvariance, 有限 logit): 0.0692520511447583
- top_logits 数值: `[23.421052932739258, 22.894737243652344]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 20 | 20 | 23.421052932739258 | 文太极与佰夷族族长女儿的独生女，外表长相甜美，为人善良，因宗帅保媒，跟完颜藏之子完颜政缔有婚约。 |
| 2 | 19 | 19 | 22.894737243652344 | 初登场：2 能力：拳法、剑法 武器：御龙剑、追血令 招式：凝龙诀、寒羽 文婧 女主角。 |
- GPT 引用1: span=[20, 20], relevant=1

## multifieldqa_zh_idx322 (idx=322, multifieldqa_zh)
### 问题
巴斯夫是由哪个公司生产的？

### 参考答案（数据集）
- ref1: 巴斯夫(中国)有限公司中国。

### 模型全文输出
```
<statement>根据安全技术说明书内容，巴斯夫是由巴斯夫(中国)有限公司生产的。<cite>[1-1]</cite></statement>
```

文档句索引范围: sid ∈ [1, 166], 共 166 句。
引用指标: R=0.0, P=0.0, F1=0.0
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=768, completion_tokens=15
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据安全技术说明书内容，巴斯夫是由巴斯夫(中国)有限公司生产的。
- GPT support_score=0; Rating: [[No support]]

**跨度** `[1-1]`
- au=0.6823925375938416, eu=0.17840370535850525, au′=0.6823915243148804, eu′=0.041621025651693344
- top_logits 方差 (pvariance, 有限 logit): 0.0173131382698557
- top_logits 数值: `[23.157896041870117, 22.894737243652344]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 2 | 2 | 23.157896041870117 | 化学品及企业标识Stomp New 330 EC推荐用途和限制用途: 植物保护产品, 除草剂公司:巴斯夫(中国)有限公司中国 上海浦东江心沙路300号邮政编码 200137电话: +86 21 20391000传真号: +86 21 20394800E-mail地址: china-psr-sds@basf.com紧急联络信息:巴斯夫紧急热线中心（中国）+86 21 5861-1199巴斯夫紧急热线中心（国际）:电话: +49 180 2273-112Company:BASF (China) Co., Ltd.300 Jiang Xin Sha RoadPu Dong Shanghai 200137, CHINATelephone: +86 21 20391000Telefax number: +86 21 20394800E-mail address: china-psr-sds@basf.comEmergency information:Emergency Call Center (China):+86 21 5861-1199International emergency number:Telephone: +49 180 2273-1122. |
| 2 | 1 | 1 | 22.894737243652344 | '安全技术说明书巴斯夫 安全技术说明书按照GB/T 16483编制日期 / 本次修订: 04.07.2023日期/上次修订: 07.08.2018日期 / 首次编制: 11.03.2010产品: Stomp New 330 EC版本: 4.0上次版本: 3.0页: 1/14(30448922/SDS_CPA_CN/ZH)印刷日期 05.07.20231. |
- GPT 引用1: span=[1, 1], relevant=0

## multifieldqa_zh_idx323 (idx=323, multifieldqa_zh)
### 问题
意识流文学的作品类型有哪些？

### 参考答案（数据集）
- ref1: 意识流文学主要的作品类型为小说，但也表现在戏剧和诗歌等作品类型中。

### 模型全文输出
```
<statement>根据上文，意识流文学的作品类型主要有：<cite></cite></statement>

<statement>1. 小说<cite>[108-108][110-110][111-111]</cite></statement>

<statement>意识流文学最主要的作品类型是小说。它以人物的意识活动为结构中心，将人物的观察、回忆、联想的全部场景与人物的感觉、思想、情绪、愿望等，交织叠合在一起加以展示。<cite>[49-50]</cite></statement>

<statement>2. 短篇小说<cite>[109-109]</cite></statement>

<statement>意识流文学也适用于短篇小说的写作。短篇意识流小说相比长篇更易于理解。<cite>[109-109]</cite></statement>

<statement>3. 长篇小说<cite>[108-108][109-109][111-111]</cite></statement>

<statement>意识流文学也可以写长篇小说，但长篇意识流小说过于复杂，会加剧作品内容的理解难度，让人感觉隐晦以及阴暗。<cite>[109-109]</cite></statement>

<statement>4. 古诗<cite>[110-110]</cite></statement>

<statement>中国古诗中有非常多的意识流写作手法的体现。<cite>[110-110]</cite></statement>

<statement>5. 宗教经典<cite>[110-110]</cite></statement>

<statement>很多宗教经典中也含有非常多的意识流写作手法的体现。<cite>[110-110]</cite></statement>

<statement>所以，意识流文学主要体现在小说领域，也可以应用于短篇小说、古诗和宗教经典等其他文学形式。<cite></cite></statement>
```

文档句索引范围: sid ∈ [1, 230], 共 230 句。
引用指标: R=0.7916666666666666, P=0.8333333333333334, F1=0.811965811965812
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=8228, completion_tokens=169
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据上文，意识流文学的作品类型主要有：
- GPT support_score=1; [[No]]   Need Citation: [[No]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 1
1. 小说
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[108-108]`
- au=1.0826759338378906, eu=0.13309979438781738, au′=1.0826756954193115, eu′=0.044032443314790726
- top_logits 方差 (pvariance, 有限 logit): 0.7040627709389405
- top_logits 数值: `[22.894737243652344, 21.18421173095703, 21.052631378173828]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 108 | 108 | 22.894737243652344 | 文体分类 意识流文字中所有的内容都是对主人公内心意识活动的忠实记叙，因此本质上可以认为一篇完整独立的意识流文章属于记叙文体裁，但由于其特殊的表述形式和较为高级的表述技巧，暗合形散神聚的散文特征，因此我们也可以把它当成散文来看待。 |
| 2 | 4 | 4 | 21.18421173095703 | 意识流文学是现代主义文学的重要分支，主要成就局限在小说领域，在戏剧、诗歌中也有表现。 |
| 3 | 51 | 51 | 21.052631378173828 | 西方现代小说史上，如马塞尔·普鲁斯特、詹姆斯·乔伊斯、弗吉尼亚·伍尔芙、威廉·福克纳等，都以成功地运用意识流而闻名于世。 |

**跨度** `[110-110]`
- au=0.682269275188446, eu=0.18009471893310547, au′=0.682268500328064, eu′=0.04208194464445114
- top_logits 方差 (pvariance, 有限 logit): 0.017312887302978197
- top_logits 数值: `[22.894737243652344, 22.631580352783203]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 110 | 110 | 22.894737243652344 | 目前的绝大部分意识流文学作品体裁都是小说，但实际上中国古诗中有非常多的意识流写作手法的体现，而且很多宗教经典中也含有非常多的意识流写作手法的体现。 |
| 2 | 109 | 109 | 22.631580352783203 | 意识流文字先天优势让它更适用于现代诗的写作，以及短篇小说，长篇的意识流小说过于复杂，会加剧作品内容的理解难度，让人感觉隐晦以及阴暗。 |

**跨度** `[111-111]`
- au=0.6813729405403137, eu=0.19240501523017883, au′=0.6813719272613525, eu′=0.04548174515366554
- top_logits 方差 (pvariance, 有限 logit): 0.004328221825744549
- top_logits 数值: `[21.052631378173828, 20.921052932739258]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 112 | 112 | 21.052631378173828 | 常见技法 内心独白 在假定没有其他人倾听的情况下，一个人物把自己的所感所思毫无顾忌的直接表露出来，就是“内心独白”。 |
| 2 | 111 | 111 | 20.921052932739258 | 意识流文学由于过于晦涩强调无秩序，难以被大众理解，所以无法成为通俗文学的主流。 |
- GPT 引用1: span=[108, 108], relevant=0
- GPT 引用2: span=[110, 111], relevant=1

### 陈述 2
意识流文学最主要的作品类型是小说。它以人物的意识活动为结构中心，将人物的观察、回忆、联想的全部场景与人物的感觉、思想、情绪、愿望等，交织叠合在一起加以展示。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[49-50]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> 小说中的意识流，是指小说叙事过程对于人物持续流动的意识过程的模仿。 具体说来，也就是以人物的意识活动为结构中心，围绕人物表面看来似乎是随机产生，且逻辑松散的意识中心，将人物的观
> 察、回忆、联想的全部场景与人物的感觉、思想、情绪、愿望等，交织叠合在一起加以展示，以"原样"准确地描摹人物的意识流动过程。
- au=0.6818957328796387, eu=0.18446597456932068, au′=0.6818947792053223, eu′=0.04328017681837082
- top_logits 方差 (pvariance, 有限 logit): 0.0692520511447583
- top_logits 数值: `[22.36842155456543, 21.842105865478516]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 49 | 49 | 22.36842155456543 | 小说中的意识流，是指小说叙事过程对于人物持续流动的意识过程的模仿。 |
| 2 | 50 | 50 | 21.842105865478516 | 具体说来，也就是以人物的意识活动为结构中心，围绕人物表面看来似乎是随机产生，且逻辑松散的意识中心，将人物的观察、回忆、联想的全部场景与人物的感觉、思想、情绪、愿望等，交织叠合在一起加以展示，以"原样"准确地描摹人物的意识流动过程。 |
- GPT 引用1: span=[49, 50], relevant=1

### 陈述 3
2. 短篇小说
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[109-109]`
- au=1.6015096662158612e-06, eu=0.28358200192451477, au′=-0.0, eu′=0.03807614743709564
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.263158798217773]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 109 | 109 | 25.263158798217773 | 意识流文字先天优势让它更适用于现代诗的写作，以及短篇小说，长篇的意识流小说过于复杂，会加剧作品内容的理解难度，让人感觉隐晦以及阴暗。 |
- GPT 引用1: span=[109, 109], relevant=1

### 陈述 4
意识流文学也适用于短篇小说的写作。短篇意识流小说相比长篇更易于理解。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[109-109]`
- au=1.7609027054277249e-06, eu=0.2676055431365967, au′=-0.0, eu′=0.0352504625916481
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[27.36842155456543]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 109 | 109 | 27.36842155456543 | 意识流文字先天优势让它更适用于现代诗的写作，以及短篇小说，长篇的意识流小说过于复杂，会加剧作品内容的理解难度，让人感觉隐晦以及阴暗。 |
- GPT 引用1: span=[109, 109], relevant=1

### 陈述 5
3. 长篇小说
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[108-108]`
- au=0.6820317506790161, eu=0.183574840426445, au′=0.6820309162139893, eu′=0.043035101145505905
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[22.23684310913086, 22.23684310913086]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 109 | 109 | 22.23684310913086 | 意识流文字先天优势让它更适用于现代诗的写作，以及短篇小说，长篇的意识流小说过于复杂，会加剧作品内容的理解难度，让人感觉隐晦以及阴暗。 |
| 2 | 108 | 108 | 22.23684310913086 | 文体分类 意识流文字中所有的内容都是对主人公内心意识活动的忠实记叙，因此本质上可以认为一篇完整独立的意识流文章属于记叙文体裁，但由于其特殊的表述形式和较为高级的表述技巧，暗合形散神聚的散文特征，因此我们也可以把它当成散文来看待。 |

**跨度** `[109-109]`
- au=1.6284955108858412e-06, eu=0.28897327184677124, au′=-0.0, eu′=0.03905446454882622
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.60526466369629]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 109 | 109 | 24.60526466369629 | 意识流文字先天优势让它更适用于现代诗的写作，以及短篇小说，长篇的意识流小说过于复杂，会加剧作品内容的理解难度，让人感觉隐晦以及阴暗。 |

**跨度** `[111-111]`
- au=0.6818891167640686, eu=0.1840193271636963, au′=0.681888222694397, eu′=0.043157294392585754
- top_logits 方差 (pvariance, 有限 logit): 0.10820617305944324
- top_logits 数值: `[22.5, 21.842105865478516]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 110 | 110 | 22.5 | 目前的绝大部分意识流文学作品体裁都是小说，但实际上中国古诗中有非常多的意识流写作手法的体现，而且很多宗教经典中也含有非常多的意识流写作手法的体现。 |
| 2 | 111 | 111 | 21.842105865478516 | 意识流文学由于过于晦涩强调无秩序，难以被大众理解，所以无法成为通俗文学的主流。 |
- GPT 引用1: span=[108, 109], relevant=1
- GPT 引用2: span=[111, 111], relevant=0

### 陈述 6
意识流文学也可以写长篇小说，但长篇意识流小说过于复杂，会加剧作品内容的理解难度，让人感觉隐晦以及阴暗。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[109-109]`
- au=1.7382523083142587e-06, eu=0.2629757225513458, au′=-0.0, eu′=0.03445149213075638
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[28.026317596435547]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 109 | 109 | 28.026317596435547 | 意识流文字先天优势让它更适用于现代诗的写作，以及短篇小说，长篇的意识流小说过于复杂，会加剧作品内容的理解难度，让人感觉隐晦以及阴暗。 |
- GPT 引用1: span=[109, 109], relevant=1

### 陈述 7
4. 古诗
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[110-110]`
- au=1.5807482895979774e-06, eu=0.2794116735458374, au′=-0.0, eu′=0.037328090518713
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.789474487304688]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 110 | 110 | 25.789474487304688 | 目前的绝大部分意识流文学作品体裁都是小说，但实际上中国古诗中有非常多的意识流写作手法的体现，而且很多宗教经典中也含有非常多的意识流写作手法的体现。 |
- GPT 引用1: span=[110, 110], relevant=1

### 陈述 8
中国古诗中有非常多的意识流写作手法的体现。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[110-110]`
- au=1.5178816283878405e-06, eu=0.26666659116744995, au′=-0.0, eu′=0.035087715834379196
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[27.500001907348633]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 110 | 110 | 27.500001907348633 | 目前的绝大部分意识流文学作品体裁都是小说，但实际上中国古诗中有非常多的意识流写作手法的体现，而且很多宗教经典中也含有非常多的意识流写作手法的体现。 |
- GPT 引用1: span=[110, 110], relevant=1

### 陈述 9
5. 宗教经典
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[110-110]`
- au=1.5606800616296823e-06, eu=0.2753622531890869, au′=-0.0, eu′=0.03660885989665985
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[26.3157901763916]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 110 | 110 | 26.3157901763916 | 目前的绝大部分意识流文学作品体裁都是小说，但实际上中国古诗中有非常多的意识流写作手法的体现，而且很多宗教经典中也含有非常多的意识流写作手法的体现。 |
- GPT 引用1: span=[110, 110], relevant=1

### 陈述 10
很多宗教经典中也含有非常多的意识流写作手法的体现。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[110-110]`
- au=1.7609027054277249e-06, eu=0.2676055431365967, au′=-0.0, eu′=0.0352504625916481
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[27.36842155456543]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 110 | 110 | 27.36842155456543 | 目前的绝大部分意识流文学作品体裁都是小说，但实际上中国古诗中有非常多的意识流写作手法的体现，而且很多宗教经典中也含有非常多的意识流写作手法的体现。 |
- GPT 引用1: span=[110, 110], relevant=1

### 陈述 11
所以，意识流文学主要体现在小说领域，也可以应用于短篇小说、古诗和宗教经典等其他文学形式。
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

## multifieldqa_zh_idx324 (idx=324, multifieldqa_zh)
### 问题
VPN使用结束后如何断开连接？

### 参考答案（数据集）
- ref1: 点击右上方的“注销”按钮，断开连接。

### 模型全文输出
```
<statement>根据VPN使用说明，VPN使用结束后断开连接的方法如下：<cite></cite></statement>

<statement>1. Web方式使用<cite></cite></statement>
<statement>在右上方点击“注销”按钮，断开连接。<cite>[9-9]</cite></statement>

<statement>2. 客户端方式使用<cite></cite></statement>
<statement>(1) 双击任务托盘中的红色字母A,选择“断开连接”选项。<cite></cite></statement>
<statement>(2) 或者在桌面上的VPN客户端程序中点击“断开连接”按钮。<cite></cite></statement>

<statement>3. Mac OS系统VPN使用<cite></cite></statement>
<statement>在系统菜单栏的VPN图标上点击，选择“断开连接”选项。<cite></cite></statement>

<statement>4. 移动智能终端VPN使用<cite></cite></statement>
<statement>(1) IOS系统：在手机屏幕右上角显示的VPN图标上长按，选择“断开连接”选项。<cite></cite></statement>
<statement>(2) Android系统：在手机屏幕右下角显示的VPN图标上长按，选择“断开连接”选项。<cite></cite></statement>

<statement>所以使用VPN结束后，可以通过以上方法断开连接。<cite></cite></statement>
```

文档句索引范围: sid ∈ [1, 27], 共 27 句。
引用指标: R=0.25, P=1.0, F1=0.4
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=4998, completion_tokens=123
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据VPN使用说明，VPN使用结束后断开连接的方法如下：
- GPT support_score=0; [[Yes]]   Need Citation: [[Yes]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 1
1. Web方式使用
- GPT support_score=0; [[Yes]]   Need Citation: [[Yes]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 2
在右上方点击“注销”按钮，断开连接。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[9-9]`
- au=1.6340368347300682e-06, eu=0.29007625579833984, au′=-0.0, eu′=0.03925619646906853
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.473684310913086]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 9 | 9 | 24.473684310913086 | 4、使用结束后请点击右上方的“注销”按钮，断开连接。 |
- GPT 引用1: span=[9, 9], relevant=1

### 陈述 3
2. 客户端方式使用
- GPT support_score=1; [[No]]   Need Citation: [[No]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 4
(1) 双击任务托盘中的红色字母A,选择“断开连接”选项。
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 5
(2) 或者在桌面上的VPN客户端程序中点击“断开连接”按钮。
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 6
3. Mac OS系统VPN使用
- GPT support_score=0; [[Yes]]   Need Citation: [[Yes]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 7
在系统菜单栏的VPN图标上点击，选择“断开连接”选项。
- GPT support_score=0; [[Yes]]   Need Citation: [[Yes]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 8
4. 移动智能终端VPN使用
- GPT support_score=0; [[Yes]]   Need Citation: [[Yes]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 9
(1) IOS系统：在手机屏幕右上角显示的VPN图标上长按，选择“断开连接”选项。
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 10
(2) Android系统：在手机屏幕右下角显示的VPN图标上长按，选择“断开连接”选项。
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 11
所以使用VPN结束后，可以通过以上方法断开连接。
- GPT support_score=1; [[No]]  Need Citation: [[No]] Analysis

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

## multifieldqa_zh_idx325 (idx=325, multifieldqa_zh)
### 问题
交通事故中谁负主要责任？

### 参考答案（数据集）
- ref1: 谢震宏。

### 模型全文输出
```
<statement>根据民事判决书，交通事故中谢震宏应承担主要责任，刘勇超应承担次要责任，徐岸灵不承担责任。<cite>[6-6]</cite></statement>
```

文档句索引范围: sid ∈ [1, 88], 共 88 句。
引用指标: R=1.0, P=1.0, F1=1.0
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=644, completion_tokens=13
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据民事判决书，交通事故中谢震宏应承担主要责任，刘勇超应承担次要责任，徐岸灵不承担责任。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[6-6]`
- au=1.698429286989267e-06, eu=0.30278876423835754, au′=-0.0, eu′=0.041621025651693344
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.026315689086914]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 6 | 6 | 23.026315689086914 | 经公安交警部门作出交通事故责任认定，谢震宏应承担事故的主要责任，刘勇超应承担事故的次要责任，徐岸灵不承担责任。 |
- GPT 引用1: span=[6, 6], relevant=1

## multifieldqa_zh_idx326 (idx=326, multifieldqa_zh)
### 问题
在基于立体卫星影像的DSM生产作业中，影像星下点的分辨率应该满足什么要求？

### 参考答案（数据集）
- ref1: 分辨率优于5m。

### 模型全文输出
```
<statement>根据CH/T XXXXX-202X标准，影像星下点的分辨率应优于5m。<cite>[69-69]</cite></statement>
```

文档句索引范围: sid ∈ [1, 188], 共 188 句。
引用指标: R=1.0, P=1.0, F1=1.0
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=638, completion_tokens=13
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据CH/T XXXXX-202X标准，影像星下点的分辨率应优于5m。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[69-69]`
- au=1.6452690942969639e-06, eu=0.29230761528015137, au′=-0.0, eu′=0.03966596722602844
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.210527420043945]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 69 | 69 | 24.210527420043945 | 选择影像 的要求如下：a） 影像星下点的分辨率应优于 5m； |
- GPT 引用1: span=[69, 69], relevant=1

## multifieldqa_zh_idx327 (idx=327, multifieldqa_zh)
### 问题
相较于上个五年，本市的固定资产投资增长了多少？

### 参考答案（数据集）
- ref1: 市的固定资产投资增长了4.4倍。

### 模型全文输出
```
<statement>根据报告内容，本市2004-2008年的固定资产投资累计完成602亿元，比上个五年增长4.4倍。<cite>[6-6]</cite></statement>

<statement>所以相较于上个五年，本市的固定资产投资增长了4.4倍。<cite></cite></statement>
```

文档句索引范围: sid ∈ [1, 120], 共 120 句。
引用指标: R=0.5, P=1.0, F1=0.6666666666666666
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=883, completion_tokens=23
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据报告内容，本市2004-2008年的固定资产投资累计完成602亿元，比上个五年增长4.4倍。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[6-6]`
- au=1.6284955108858412e-06, eu=0.28897327184677124, au′=-0.0, eu′=0.03905446454882622
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.60526466369629]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 6 | 6 | 24.60526466369629 | 全社会固定资产投资206亿元，五年累计完成602亿元，比上个五年增长4.4倍。 |
- GPT 引用1: span=[6, 6], relevant=1

### 陈述 1
所以相较于上个五年，本市的固定资产投资增长了4.4倍。
- GPT support_score=0; [[Yes]]   Need Citation: [[Yes]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

## multifieldqa_zh_idx328 (idx=328, multifieldqa_zh)
### 问题
法院认定了谁对事故负有责任？

### 参考答案（数据集）
- ref1: 根据法院的判决书，法院认定了聂红雨和于峰对事故负有同等责任。

### 模型全文输出
```
<statement>根据判决书，法院认定了以下人员对事故负有责任：<cite></cite></statement>

<statement>1. 聂红雨<cite>[7-7][50-50][51-51]</cite></statement>

<statement>2. 于峰<cite>[7-7][9-9][50-50]</cite></statement>

<statement>法院认为聂红雨和于峰各自负有事故的同等责任。<cite>[7-7][9-9][50-50]</cite></statement>
```

文档句索引范围: sid ∈ [1, 88], 共 88 句。
引用指标: R=0.625, P=0.625, F1=0.625
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=4534, completion_tokens=86
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据判决书，法院认定了以下人员对事故负有责任：
- GPT support_score=0; [[Yes]]   Need Citation: [[Yes]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 1
1. 聂红雨
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[7-7]`
- au=0.6820790767669678, eu=0.1826922595500946, au′=0.6820782423019409, eu′=0.04279278963804245
- top_logits 方差 (pvariance, 有限 logit): 0.017312887302978197
- top_logits 数值: `[22.5, 22.23684310913086]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 7 | 7 | 22.5 | 赵玉国、阜新洺伟物流有限公司诉称：2011年8月2日15时许，聂红雨驾驶的皖皖C×××**式箱式货车沿S307线自东向西行驶至26KM＋300M处时，与自西向东于峰驾驶的辽辽J×××**辽辽J×××**重型半挂牵引车相撞，造成人员受伤、车辆受损的道路交通事故。 |
| 2 | 50 | 50 | 22.23684310913086 | 根据上述认定的证据和庭审中当事人的陈述，查明以下事实：2011年8月2日15时许，聂红雨驾驶皖C××**皖C×××**沿S307线自东向西行驶至26KM＋300M处时，与于峰自西向东驾驶的辽J××**辽J×××**×辽J×××**车相撞，造成聂红雨、皖C×××**皖C×××**乘车人李响受伤及两车受损的道路交通事故。 |

**跨度** `[50-50]`
- au=1.0835089683532715, eu=0.13217388093471527, au′=1.0835084915161133, eu′=0.04369489848613739
- top_logits 方差 (pvariance, 有限 logit): 0.026931436878738977
- top_logits 数值: `[22.10526466369629, 21.842105865478516, 21.710527420043945]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 50 | 50 | 22.10526466369629 | 根据上述认定的证据和庭审中当事人的陈述，查明以下事实：2011年8月2日15时许，聂红雨驾驶皖C××**皖C×××**沿S307线自东向西行驶至26KM＋300M处时，与于峰自西向东驾驶的辽J××**辽J×××**×辽J×××**车相撞，造成聂红雨、皖C×××**皖C×××**乘车人李响受伤及两车受损的道路交通事故。 |
| 2 | 60 | 60 | 21.842105865478516 | 聂红雨驾驶皖C×××**重型箱皖C×××**驾驶的辽J×××**-辽J辽J×××**型辽J×××**造成聂红雨、皖C×××**重型箱式皖C×××**响受伤及两车受损的道路交通事故。 |
| 3 | 8 | 8 | 21.710527420043945 | 经凤阳县公安局交通管理大队认定，聂红雨负此次事故的同等责任； |

**跨度** `[51-51]`
- au=0.6818511486053467, eu=0.18313249945640564, au′=0.6818503141403198, eu′=0.04291360080242157
- top_logits 方差 (pvariance, 有限 logit): 0.21208462622689694
- top_logits 数值: `[22.763158798217773, 21.842105865478516]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 60 | 60 | 22.763158798217773 | 聂红雨驾驶皖C×××**重型箱皖C×××**驾驶的辽J×××**-辽J辽J×××**型辽J×××**造成聂红雨、皖C×××**重型箱式皖C×××**响受伤及两车受损的道路交通事故。 |
| 2 | 51 | 51 | 21.842105865478516 | 该起事故经凤阳县公安交通管理大队认定，聂红雨负此次事故的同等责任，于峰负此次事故的同等责任，李响无责任。 |
- GPT 引用1: span=[7, 7], relevant=1
- GPT 引用2: span=[50, 51], relevant=1

### 陈述 2
2. 于峰
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[7-7]`
- au=1.6742565094318707e-06, eu=0.29803910851478577, au′=-0.0, eu′=0.04072882607579231
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.55263328552246]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 7 | 7 | 23.55263328552246 | 赵玉国、阜新洺伟物流有限公司诉称：2011年8月2日15时许，聂红雨驾驶的皖皖C×××**式箱式货车沿S307线自东向西行驶至26KM＋300M处时，与自西向东于峰驾驶的辽辽J×××**辽辽J×××**重型半挂牵引车相撞，造成人员受伤、车辆受损的道路交通事故。 |

**跨度** `[9-9]`
- au=0.6822815537452698, eu=0.1792452484369278, au′=0.6822808980941772, eu′=0.04185021296143532
- top_logits 方差 (pvariance, 有限 logit): 0.0692520511447583
- top_logits 数值: `[23.157896041870117, 22.631580352783203]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 9 | 9 | 23.157896041870117 | 于峰负此次事故的同等责任。 |
| 2 | 50 | 50 | 22.631580352783203 | 根据上述认定的证据和庭审中当事人的陈述，查明以下事实：2011年8月2日15时许，聂红雨驾驶皖C××**皖C×××**沿S307线自东向西行驶至26KM＋300M处时，与于峰自西向东驾驶的辽J××**辽J×××**×辽J×××**车相撞，造成聂红雨、皖C×××**皖C×××**乘车人李响受伤及两车受损的道路交通事故。 |

**跨度** `[50-50]`
- au=0.6820210218429565, eu=0.1822541505098343, au′=0.6820201873779297, eu′=0.04267264902591705
- top_logits 方差 (pvariance, 有限 logit): 0.10820617305944324
- top_logits 数值: `[22.763158798217773, 22.10526466369629]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 50 | 50 | 22.763158798217773 | 根据上述认定的证据和庭审中当事人的陈述，查明以下事实：2011年8月2日15时许，聂红雨驾驶皖C××**皖C×××**沿S307线自东向西行驶至26KM＋300M处时，与于峰自西向东驾驶的辽J××**辽J×××**×辽J×××**车相撞，造成聂红雨、皖C×××**皖C×××**乘车人李响受伤及两车受损的道路交通事故。 |
| 2 | 51 | 51 | 22.10526466369629 | 该起事故经凤阳县公安交通管理大队认定，聂红雨负此次事故的同等责任，于峰负此次事故的同等责任，李响无责任。 |
- GPT 引用1: span=[7, 7], relevant=1
- GPT 引用2: span=[9, 9], relevant=1
- GPT 引用3: span=[50, 50], relevant=0

### 陈述 3
法院认为聂红雨和于峰各自负有事故的同等责任。
- GPT support_score=0.5; Rating: [[Partially supported]]

**跨度** `[7-7]`
- au=1.0832923650741577, eu=0.13217388093471527, au′=1.08329176902771, eu′=0.04369489848613739
- top_logits 方差 (pvariance, 有限 logit): 0.2346873114608267
- top_logits 数值: `[22.5, 21.842105865478516, 21.3157901763916]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 7 | 7 | 22.5 | 赵玉国、阜新洺伟物流有限公司诉称：2011年8月2日15时许，聂红雨驾驶的皖皖C×××**式箱式货车沿S307线自东向西行驶至26KM＋300M处时，与自西向东于峰驾驶的辽辽J×××**辽辽J×××**重型半挂牵引车相撞，造成人员受伤、车辆受损的道路交通事故。 |
| 2 | 8 | 8 | 21.842105865478516 | 经凤阳县公安局交通管理大队认定，聂红雨负此次事故的同等责任； |
| 3 | 51 | 51 | 21.3157901763916 | 该起事故经凤阳县公安交通管理大队认定，聂红雨负此次事故的同等责任，于峰负此次事故的同等责任，李响无责任。 |

**跨度** `[9-9]`
- au=1.6567056491112453e-06, eu=0.2945735454559326, au′=-0.0, eu′=0.04008438438177109
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.947368621826172]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 9 | 9 | 23.947368621826172 | 于峰负此次事故的同等责任。 |

**跨度** `[50-50]`
- au=0.6825187802314758, eu=0.17551958560943604, au′=0.6825180053710938, eu′=0.04083825275301933
- top_logits 方差 (pvariance, 有限 logit): 0.10820617305944324
- top_logits 数值: `[23.8157901763916, 23.157896041870117]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 50 | 50 | 23.8157901763916 | 根据上述认定的证据和庭审中当事人的陈述，查明以下事实：2011年8月2日15时许，聂红雨驾驶皖C××**皖C×××**沿S307线自东向西行驶至26KM＋300M处时，与于峰自西向东驾驶的辽J××**辽J×××**×辽J×××**车相撞，造成聂红雨、皖C×××**皖C×××**乘车人李响受伤及两车受损的道路交通事故。 |
| 2 | 61 | 61 | 23.157896041870117 | 该起事故经凤阳县公安交通管理大队认定，聂红雨负此次事故的同等责任，于峰负此次事故的同等责任，李响无责任。 |
- GPT 引用1: span=[7, 7], relevant=0
- GPT 引用2: span=[9, 9], relevant=1
- GPT 引用3: span=[50, 50], relevant=0

## multifieldqa_zh_idx329 (idx=329, multifieldqa_zh)
### 问题
阿诺新是否适用于儿童？

### 参考答案（数据集）
- ref1: 阿诺新不适用于儿童。

### 模型全文输出
```
<statement>根据该产品说明书，依西美坦目前尚未评估在儿童患者中的疗效和安全性，不推荐儿童使用。<cite>[134-135]</cite></statement>
```

文档句索引范围: sid ∈ [1, 196], 共 196 句。
引用指标: R=1.0, P=1.0, F1=1.0
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=600, completion_tokens=13
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据该产品说明书，依西美坦目前尚未评估在儿童患者中的疗效和安全性，不推荐儿童使用。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[134-135]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> 儿童用药 尚未评估本品在儿童患者中的疗效和安全性。 不推荐儿童使用。
- au=1.6452690942969639e-06, eu=0.29230761528015137, au′=-0.0, eu′=0.03966596722602844
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.210527420043945]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 134 | 134 | 24.210527420043945 | 儿童用药 尚未评估本品在儿童患者中的疗效和安全性。 |
- GPT 引用1: span=[134, 135], relevant=1

## multifieldqa_zh_idx330 (idx=330, multifieldqa_zh)
### 问题
如何清洁本产品？

### 参考答案（数据集）
- ref1: 使用柔软的无绒干抹布。

### 模型全文输出
```
<statement>根据产品安全手册中的说明，建议使用以下方法清洁本产品：<cite></cite></statement>

<statement>1. 清洁产品外部和平板扫描器前，请先从墙上插座中拔出本产品的电源插头。<cite>[127-127]</cite></statement>

<statement>2. 请勿使用液体或喷雾清洁剂清洁本产品。 <cite>[128-128]</cite></statement>

<statement>3. 请使用柔软的无绒干抹布擦拭本产品。<cite>[59-59]</cite></statement>

<statement>4. 如果需要清洁墨盒，请使用墨盒制造商提供的清洁剂。<cite></cite></statement>

<statement>5. 如果墨水沾到皮肤上，请立即用水和肥皂清洗。<cite>[71-71]</cite></statement>

<statement>6. 如果墨水误入眼中，请立即用清水冲洗。<cite>[72-72]</cite></statement>

<statement>7. 如果置之不理，可能会造成红眼或轻微发炎。如果发现任何异常，请立即就医。<cite>[73-74]</cite></statement>

<statement>8. 如果墨水误入口中，请立即吐出来并就医。<cite>[75-75]</cite></statement>

<statement>9. 更换墨盒时，请注意不要让墨水误入眼中。<cite>[76-76]</cite></statement>

<statement>10. 请将墨盒置于儿童不易触及的地方。严禁吞食墨水。<cite>[77-77][78-78]</cite></statement>

<statement>11. 切勿摇晃墨盒，否则可能会有墨水渗漏。切勿拆解墨盒，否则墨水可能会误入眼中或沾到皮肤上。<cite>[79-80]</cite></statement>

<statement>12. 切勿用手抚摸打印纸张边缘，否则纸张的锋利边缘可能会造成您受伤。<cite>[81-81]</cite></statement>

<statement>总之，请勿使用液体或喷雾清洁剂，使用柔软的无绒干抹布进行清洁。如果墨水沾到皮肤、眼部或入口中，请及时采取措施。<cite></cite></statement>
```

文档句索引范围: sid ∈ [1, 227], 共 227 句。
引用指标: R=0.7857142857142857, P=1.0, F1=0.88
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=8161, completion_tokens=173
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据产品安全手册中的说明，建议使用以下方法清洁本产品：
- GPT support_score=0; [[Yes]]   Need Citation: [[Yes]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 1
1. 清洁产品外部和平板扫描器前，请先从墙上插座中拔出本产品的电源插头。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[127-127]`
- au=1.6802155187178869e-06, eu=0.2992125153541565, au′=-0.0, eu′=0.04094827175140381
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.421052932739258]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 127 | 127 | 23.421052932739258 | 4 清洁产品外部和平板扫描器前，请先从墙上插座中拔出本产品的电源插头。 |
- GPT 引用1: span=[127, 127], relevant=1

### 陈述 2
2. 请勿使用液体或喷雾清洁剂清洁本产品。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[128-128]`
- au=1.729960786178708e-06, eu=0.3089430034160614, au′=-0.0, eu′=0.04279278963804245
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[22.36842155456543]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 128 | 128 | 22.36842155456543 | 请 勿使用液体或喷雾清洁剂清洁本产品。 |
- GPT 引用1: span=[128, 128], relevant=1

### 陈述 3
3. 请使用柔软的无绒干抹布擦拭本产品。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[59-59]`
- au=1.6742565094318707e-06, eu=0.29803910851478577, au′=-0.0, eu′=0.04072882607579231
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.55263328552246]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 59 | 59 | 23.55263328552246 | 请使用柔软的无绒干抹布清洁 产品。 |
- GPT 引用1: span=[59, 59], relevant=1

### 陈述 4
4. 如果需要清洁墨盒，请使用墨盒制造商提供的清洁剂。
- GPT support_score=0; [[Yes]]   Need Citation: [[Yes]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 5
5. 如果墨水沾到皮肤上，请立即用水和肥皂清洗。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[71-71]`
- au=1.5557670849375427e-06, eu=0.27436816692352295, au′=-0.0, eu′=0.03643336147069931
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[26.447368621826172]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 71 | 71 | 26.447368621826172 | 如果墨水沾到皮肤上或者误入眼中或口中，请立即采取以下步骤： • 如果墨水沾到皮肤上，请立即用水和肥皂清洗。 |
- GPT 引用1: span=[71, 71], relevant=1

### 陈述 6
6. 如果墨水误入眼中，请立即用清水冲洗。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[72-72]`
- au=1.6340368347300682e-06, eu=0.29007625579833984, au′=-0.0, eu′=0.03925619646906853
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.473684310913086]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 72 | 72 | 24.473684310913086 | • 如果墨水误入眼中，请立即用清水冲洗。 |
- GPT 引用1: span=[72, 72], relevant=1

### 陈述 7
7. 如果置之不理，可能会造成红眼或轻微发炎。如果发现任何异常，请立即就医。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[73-74]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> 如果置之不理，可能会造成红眼或 轻微发炎。 如发现任何异常，请立即就医。
- au=1.6802155187178869e-06, eu=0.2992125153541565, au′=-0.0, eu′=0.04094827175140381
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.421052932739258]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 73 | 73 | 23.421052932739258 | 如果置之不理，可能会造成红眼或 轻微发炎。 |
- GPT 引用1: span=[73, 74], relevant=1

### 陈述 8
8. 如果墨水误入口中，请立即吐出来并就医。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[75-75]`
- au=1.6742565094318707e-06, eu=0.29803910851478577, au′=-0.0, eu′=0.04072882607579231
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.55263328552246]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 75 | 75 | 23.55263328552246 | • 如果墨水误入口中，请立即吐出来并就医。 |
- GPT 引用1: span=[75, 75], relevant=1

### 陈述 9
9. 更换墨盒时，请注意不要让墨水误入眼中。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[76-76]`
- au=1.5910399042695644e-06, eu=0.28148138523101807, au′=-0.0, eu′=0.0376984067261219
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.526317596435547]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 76 | 76 | 25.526317596435547 | • 更换墨盒时，请注意不要让墨水误入眼中。 |
- GPT 引用1: span=[76, 76], relevant=1

### 陈述 10
10. 请将墨盒置于儿童不易触及的地方。严禁吞食墨水。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[77-77]`
- au=1.5910399042695644e-06, eu=0.28148138523101807, au′=-0.0, eu′=0.0376984067261219
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[25.526317596435547]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 77 | 77 | 25.526317596435547 | • 请将墨盒置于儿童不易触及的地方。 |

**跨度** `[78-78]`
- au=1.774934617060353e-06, eu=0.2704625725746155, au′=-0.0, eu′=0.03574788197875023
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[26.973684310913086]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 78 | 78 | 26.973684310913086 | • 严禁吞食墨水。 |
- GPT 引用1: span=[77, 78], relevant=1

### 陈述 11
11. 切勿摇晃墨盒，否则可能会有墨水渗漏。切勿拆解墨盒，否则墨水可能会误入眼中或沾到皮肤上。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[79-80]`

**本跨度合并正文**（`a–b` 句拼接，供人工核对）

> • 切勿摇晃墨盒，否则可能会有墨水渗漏。 • 切勿拆解墨盒，否则墨水可能会误入眼中或沾到皮肤上。
- au=1.5606800616296823e-06, eu=0.2753622531890869, au′=-0.0, eu′=0.03660885989665985
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[26.3157901763916]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 79 | 79 | 26.3157901763916 | • 切勿摇晃墨盒，否则可能会有墨水渗漏。 |
- GPT 引用1: span=[79, 80], relevant=1

### 陈述 12
12. 切勿用手抚摸打印纸张边缘，否则纸张的锋利边缘可能会造成您受伤。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[81-81]`
- au=1.8090482853949652e-06, eu=0.27737218141555786, au′=-0.0, eu′=0.03696497529745102
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[26.05263328552246]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 81 | 81 | 26.05263328552246 | 切勿用手抚摸打印纸张边缘， 否则纸张的锋利边缘可能会造成您受伤。 |
- GPT 引用1: span=[81, 81], relevant=1

### 陈述 13
总之，请勿使用液体或喷雾清洁剂，使用柔软的无绒干抹布进行清洁。如果墨水沾到皮肤、眼部或入口中，请及时采取措施。
- GPT support_score=0; [[Yes]]   Need Citation: [[Yes]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

## multifieldqa_zh_idx331 (idx=331, multifieldqa_zh)
### 问题
有哪些常见的错误消息？

### 参考答案（数据集）
- ref1: 常见的错误消息包括卡纸、无打印盒、背书器错误等。

### 模型全文输出
```
<statement>根据操作指南，常见的错误消息包括：<cite></cite></statement>

<statement>- J1:5A 错误详情 操作 探测到卡纸 (背书器)<cite>[227-227]</cite></statement>

<statement>- U5:4A (*1) 背书器盖打开<cite>[227-227]</cite></statement>

<statement>- U6:B4 U7:B9 A1:B3 A3:B6 A4:B8 无打印盒<cite>[227-227]</cite></statement>

<statement>- A2:B5 背书器错误 (打印头)<cite>[244-244]</cite></statement>

<statement>- H6:B1 背书器系统故障<cite>[245-245]</cite></statement>

<statement>- 背书器上发生错误<cite>[254-254]</cite></statement>

<statement>*1: 如在扫描仪处于待机状态时打开背书器的打印部位，将仅显示错误消息而不显示错误代码。<cite>[257-257]</cite></statement>

<statement>这些错误消息通常表示卡纸、打印盒未安装、打印头故障、系统故障等问题。操作指南中详细说明了针对每个错误代码的解决方法。<cite></cite></statement>
```

文档句索引范围: sid ∈ [1, 272], 共 272 句。
引用指标: R=0.8888888888888888, P=1.0, F1=0.9411764705882353
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=5282, completion_tokens=114
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
根据操作指南，常见的错误消息包括：
- GPT support_score=1; [[No]]   Need Citation: [[No]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

### 陈述 1
- J1:5A 错误详情 操作 探测到卡纸 (背书器)
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[227-227]`
- au=1.6396279534092173e-06, eu=0.2911876440048218, au′=-0.0, eu′=0.039460018277168274
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.342105865478516]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 227 | 227 | 24.342105865478516 | 错误代码 J1:5A 错误详情 操作 探测到卡纸 （背书器） 请尝试以下方法： U5:4A (*1) 背书器盖打开 U6:B4 U7:B9 A1:B3 A3:B6 A4:B8 无打印盒 背书器印字区域出错 背书器错误 （通讯超时） 背书器错误 （EEPROM） 背书器错误 （ROM） 1. |
- GPT 引用1: span=[227, 227], relevant=1

### 陈述 2
- U5:4A (*1) 背书器盖打开
- GPT support_score=1; **Rating: [[Fully supported]]**   **

**跨度** `[227-227]`
- au=1.7844809008238371e-06, eu=0.2724013328552246, au′=-0.0, eu′=0.036087363958358765
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[26.710527420043945]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 227 | 227 | 26.710527420043945 | 错误代码 J1:5A 错误详情 操作 探测到卡纸 （背书器） 请尝试以下方法： U5:4A (*1) 背书器盖打开 U6:B4 U7:B9 A1:B3 A3:B6 A4:B8 无打印盒 背书器印字区域出错 背书器错误 （通讯超时） 背书器错误 （EEPROM） 背书器错误 （ROM） 1. |
- GPT 引用1: span=[227, 227], relevant=1

### 陈述 3
- U6:B4 U7:B9 A1:B3 A3:B6 A4:B8 无打印盒
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[227-227]`
- au=1.7609027054277249e-06, eu=0.2676055431365967, au′=-0.0, eu′=0.0352504625916481
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[27.36842155456543]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 227 | 227 | 27.36842155456543 | 错误代码 J1:5A 错误详情 操作 探测到卡纸 （背书器） 请尝试以下方法： U5:4A (*1) 背书器盖打开 U6:B4 U7:B9 A1:B3 A3:B6 A4:B8 无打印盒 背书器印字区域出错 背书器错误 （通讯超时） 背书器错误 （EEPROM） 背书器错误 （ROM） 1. |
- GPT 引用1: span=[227, 227], relevant=1

### 陈述 4
- A2:B5 背书器错误 (打印头)
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[244-244]`
- au=1.930719463416608e-06, eu=0.30158722400665283, au′=-0.0, eu′=0.04139433056116104
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[23.157896041870117]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 244 | 244 | 23.157896041870117 | A2:B5 背书器错误 （打印头） 背书器上发生错误。 |
- GPT 引用1: span=[244, 244], relevant=1

### 陈述 5
- H6:B1 背书器系统故障
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[245-245]`
- au=1.6396279534092173e-06, eu=0.2911876440048218, au′=-0.0, eu′=0.039460018277168274
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[24.342105865478516]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 245 | 245 | 24.342105865478516 | 请尝试以下方法： H6:B1 背书器系统故障 1. |
- GPT 引用1: span=[245, 245], relevant=1

### 陈述 6
- 背书器上发生错误
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[254-254]`
- au=0.681657612323761, eu=0.1885855346918106, au′=0.6816568970680237, eu′=0.04441846162080765
- top_logits 方差 (pvariance, 有限 logit): 0.004328347309638048
- top_logits 数值: `[21.578948974609375, 21.447368621826172]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 236 | 236 | 21.578948974609375 | 背书器上发生错误。 |
| 2 | 254 | 254 | 21.447368621826172 | 背书器上发生错误。 |
- GPT 引用1: span=[254, 254], relevant=1

### 陈述 7
*1: 如在扫描仪处于待机状态时打开背书器的打印部位，将仅显示错误消息而不显示错误代码。
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[257-257]`
- au=1.8090482853949652e-06, eu=0.27737218141555786, au′=-0.0, eu′=0.03696497529745102
- top_logits 方差 (pvariance, 有限 logit): 0.0
- top_logits 数值: `[26.05263328552246]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 257 | 257 | 26.05263328552246 | *1: 如在扫描仪处于待机状态时打开背书器的打印部位，将仅显示错误消息而不显示错误代码。 |
- GPT 引用1: span=[257, 257], relevant=1

### 陈述 8
这些错误消息通常表示卡纸、打印盒未安装、打印头故障、系统故障等问题。操作指南中详细说明了针对每个错误代码的解决方法。
- GPT support_score=0; [[Yes]]   Need Citation: [[Yes]]

*（本陈述无引用跨度 logits：pred 中 `cite_spans_first_digit_top10_numeric_logits` 为空，常见为模型写 `<cite></cite>` 或 cite 内无 `[sid-sid]`。若无下列 `GPT 引用k`，则为 cite 里该条 `citation` 为空；与 `Need Citation: Yes` 无强制对应。）*

