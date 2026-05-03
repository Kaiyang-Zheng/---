# LongBench idx50 / idx51 与 SQuALITY：引用跨度 logits / au·eu / 方差 / GPT 与候选句

## 阅读说明（量表与表格列）

- **答案分 `raw`**：来自 `scores_correct/*.json` 中 `gpt_usage.responses`，由 `eval_correct.py` 生成。 **multifieldqa_en** 走 `gpt_score_qa`：每条参考输出 **`[[1]]`–`[[3]]`**（问答正确性/覆盖面）。 **squality** 走 `gpt_score_summ`：每条参考输出 **`[[1]]`–`[[5]]`**（摘要：正确性、覆盖、**连贯性**、压缩效率等）。
  - 因此：SQuALITY 若出现 `[[3]]`、`[[2]]`，应读作 **3/5、2/5**，**不要**当成问答量表里的「满分 3 分」。
- **`normalized_0_1`**：与 `eval_correct` 写回的 `score` 一致；摘要侧单条参考为 `(rating−1)/4` 再与多参考聚合（见 `eval_correct.process`）。
- **候选句表「解析句号 / 该句内容」**：把 top 数字 token 当作句编号 sid，在 **bench 文档**经 `ensure_longcite_indexed_context` 后的 `<Csid>` 上查句（默认**全文**，不截断）。 若某 sid 下文字与 gold 引用段落不一致，多为**切句索引与当时生成引用不一致**。
- **logits 高 ≠ 语义相关**：表内 logit 是模型在写 `<cite>[` 之后、对**下一个数字 token** 的偏好（句编号在词表里的相对强度），**不是**「该句与当前 statement 是否相关」的分数，因此排名靠前的 sid 完全可能与 statement 无关。
- **同一句跨多行**：CoF 分句里某段可含换行（如 `Fans` 独立成行 + 下一段正文）。`ensure_longcite_indexed_context` 打成 `<C29> Fans` 与续行后，旧版按「每行一条」解析会把续行丢掉，表里只剩「Fans」；现已用 `parse_indexed_context` 把续行并入同一 sid，该句内容与分句结果一致。
- **GPT 引用/答案分**：由 `eval_gpt_cite_json` / `eval_correct` 另路评判，与上表 logits **不是同一套量**；cite 支持度高也不代表数字 logits 在「选句」上合理。


## multifieldqa_en_idx50 (idx=50, multifieldqa_en)
### 问题
What is the name of the most active fan club?

### 参考答案（数据集）
- ref1: South West Ultras fan club.

### 模型全文输出
```
<statement>Based on the information provided, the most active group of fans for FC Banants is the South West Ultras fan club.<cite>[28-28]</cite></statement>

<statement>The passage states: "The most active group of fans is the South West Ultras fan club, mainly composed of residents from several neighbourhoods within the Malatia-Sebastia District of Yerevan, since the club is a de facto representer of the district."<cite>[28-28]</cite></statement>

<statement>So the South West Ultras fan club is the most active group of fans for FC Banants.<cite></cite></statement>
```

文档句索引范围: sid ∈ [1, 38], 共 38 句。
引用指标: R=0.6666666666666666, P=1.0, F1=0.8
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=1655, completion_tokens=36
答案 GPT（eval_correct / gpt_score_qa，1–3 per reference (QA)）: normalized_0_1=1.0, raw=['[[3]]']

### 陈述 0
Based on the information provided, the most active group of fans for FC Banants is the South West Ultras fan club.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[28-28]`
- au=2.256385564804077, eu=0.06279434263706207, au′=3.042393445968628, eu′=0.06922319531440735
- top_logits 方差 (pvariance, 有限 logit): 8.1490625
- top_logits 数值: `[22.875, 16.25, 15.3125, 14.875, 14.0625, 13.5, 13.4375, 13.125, 12.9375, 12.875]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 29 | 29 | 22.875 | Fans The most active group of fans is the South West Ultras fan club, mainly composed of residents from several neighbourhoods within the Malatia-Sebastia District of Yerevan, since the club is a de facto representer of the district. |
| 2 | 28 | 28 | 16.25 | The current technical director of the academy is the former Russian footballer Ilshat Faizulin. |
| 3 | 30 | 30 | 15.3125 | Members of the fan club benefit from events organized by the club and many facilities of the Banants training centre, such as the mini football pitch, the club store and other entertainments. |
| 4 | 27 | 27 | 14.875 | In addition to the main stadium, the centre houses 3 full-size training pitches, mini football pitches as well as an indoor facility. |
| 5 | 26 | 26 | 14.0625 | Training centre/academy Banants Training Centre is the club's academy base located in the Malatia-Sebastia District of Yerevan. |
| 6 | 1 | 1 | 13.5 | Football Club Urartu (, translated Futbolayin Akumb Urartu), commonly known as Urartu, is an Armenian professional football team based in the capital Yerevan that currently plays in the Armenian Premier League. |
| 7 | 31 | 31 | 13.4375 | Achievements Armenian Premier League Winner (1): 2013–14. |
| 8 | 25 | 25 | 13.125 | Further developments were implemented later in 2011, when the playing pitch was modernized and the capacity of the stadium was increased up to 4,860 seats (2,760 at the northern stand, 1,500 at the southern stand and 600 at the western stand). |
| 9 | 23 | 23 | 12.9375 | Domestic  European  Stadium  The construction of the Banants Stadium was launched in 2006 in the Malatia-Sebastia District of Yerevan, with the assistance of the FIFA goal programme. |
| 10 | 5 | 5 | 12.875 | The club was known as FC Banants until 1 August 2019, when it was officially renamed FC Urartu. |
- GPT 引用1: span=[28, 28], relevant=1

### 陈述 1
The passage states: "The most active group of fans is the South West Ultras fan club, mainly composed of residents from several neighbourhoods within the Malatia-Sebastia District of Yerevan, since the club is a de facto representer of the district."
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[28-28]`
- au=2.2601747512817383, eu=0.05895356833934784, au′=3.408114194869995, eu′=0.07244941592216492
- top_logits 方差 (pvariance, 有限 logit): 7.92359375
- top_logits 数值: `[22.625, 19.625, 16.375, 16.0, 15.3125, 14.1875, 14.1875, 14.0, 13.8125, 13.5]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 29 | 29 | 22.625 | Fans The most active group of fans is the South West Ultras fan club, mainly composed of residents from several neighbourhoods within the Malatia-Sebastia District of Yerevan, since the club is a de facto representer of the district. |
| 2 | 28 | 28 | 19.625 | The current technical director of the academy is the former Russian footballer Ilshat Faizulin. |
| 3 | 27 | 27 | 16.375 | In addition to the main stadium, the centre houses 3 full-size training pitches, mini football pitches as well as an indoor facility. |
| 4 | 30 | 30 | 16.0 | Members of the fan club benefit from events organized by the club and many facilities of the Banants training centre, such as the mini football pitch, the club store and other entertainments. |
| 5 | 26 | 26 | 15.3125 | Training centre/academy Banants Training Centre is the club's academy base located in the Malatia-Sebastia District of Yerevan. |
| 6 | 24 | 24 | 14.1875 | It was officially opened in 2008 with a capacity of 3,600 seats. |
| 7 | 25 | 25 | 14.1875 | Further developments were implemented later in 2011, when the playing pitch was modernized and the capacity of the stadium was increased up to 4,860 seats (2,760 at the northern stand, 1,500 at the southern stand and 600 at the western stand). |
| 8 | 31 | 31 | 14.0 | Achievements Armenian Premier League Winner (1): 2013–14. |
| 9 | 23 | 23 | 13.8125 | Domestic  European  Stadium  The construction of the Banants Stadium was launched in 2006 in the Malatia-Sebastia District of Yerevan, with the assistance of the FIFA goal programme. |
| 10 | 22 | 22 | 13.5 | The club headquarters are located on Jivani Street 2 of the Malatia-Sebastia District, Yerevan. |
- GPT 引用1: span=[28, 28], relevant=1

### 陈述 2
So the South West Ultras fan club is the most active group of fans for FC Banants.
- GPT support_score=0; [[Yes]]   Need Citation: [[Yes]]

## multifieldqa_en_idx51 (idx=51, multifieldqa_en)
### 问题
Is the ISR necessary for transgene reactivation?

### 参考答案（数据集）
- ref1: No, it is not necessary.

### 模型全文输出
```
<statement>Based on the results presented in the paper, the ISR does not appear to be necessary for transgene reactivation in response to essential amino acid starvation.<cite></cite></statement>

<statement>- The paper shows that activation of the ISR, as measured by increased CHOP expression, occurs upon essential amino acid starvation in various cell types. However, the upregulation of CHOP does not parallel quantitatively with transgene reactivation and may not be sufficient to induce it.<cite>[187-191]</cite></statement>

<statement>- Pharmacological activation of GCN2 by L-Histidinol is sufficient to reactivate the transgenes in some cell types, but the ISR inhibitor ISRIB does not block this reactivation. This suggests the ISR is not necessary for transgene reactivation.<cite>[196-196][240-242]</cite></statement>

<statement>- Knockdown of ATF4, a downstream effector of the ISR, does not interfere with the reactivation of the transgene by L-Histidinol.<cite>[243-243]</cite></statement>

<statement>- CRISPR/Cas9-mediated knockout of GCN2 does not abolish or reduce transgene reactivation in response to essential amino acid starvation in HeLa or HepG2 cells. This excludes GCN2 and the ISR as necessary for this response.<cite>[245-250][262-268]</cite></statement>

<statement>Therefore, while the ISR is activated upon essential amino acid starvation, it does not appear to be necessary for the reactivation of silenced transgenes in cultured cells. The mechanism by which essential amino acid starvation leads to transgene reactivation remains unclear, but it does not involve GCN2 and the ISR as previously suggested.<cite></cite></statement>
```

文档句索引范围: sid ∈ [1, 278], 共 278 句。
引用指标: R=0.6666666666666666, P=0.8333333333333334, F1=0.7407407407407408
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=6465, completion_tokens=92
答案 GPT（eval_correct）：*本样本 manifest 未配置 `correct` / 无 eval_correct 分数行，故无 normalized/raw。*

### 陈述 0
Based on the results presented in the paper, the ISR does not appear to be necessary for transgene reactivation in response to essential amino acid starvation.
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

### 陈述 1
- The paper shows that activation of the ISR, as measured by increased CHOP expression, occurs upon essential amino acid starvation in various cell types. However, the upregulation of CHOP does not parallel quantitatively with transgene reactivation and may not be sufficient to induce it.
- GPT support_score=1; **Rating**: [[Fully supported]]   **

**跨度** `[187-191]`
- au=2.2740859985351562, eu=0.05089057609438896, au′=4.336624622344971, eu′=0.06670140475034714
- top_logits 方差 (pvariance, 有限 logit): 3.215
- top_logits 数值: `[22.125, 20.25, 20.0, 19.125, 18.75, 18.5, 18.375, 17.25, 16.125, 16.0]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 188 | 188 | 22.125 | Thus, we tested whether CHOP expression was upregulated upon incubation of HeLa-OA1, HepG2-OA1 and C2C12-GFP cells in media deprived of different EAAs for 24–48 h. As shown in Fig 3 and S4B Fig, we found that CHOP expression is increased in all EAA-starvation conditions, but not in the absence of Ala, in all tested cell lines. |
| 2 | 190 | 190 | 20.25 | However, the upregulation of CHOP does not parallel quantitatively that of the transgene, neither appears sufficient to induce it. |
| 3 | 214 | 214 | 20.0 | Consistent with stimulation of the ISR, CHOP and to a minor extent ATF4 are upregulated by L-Histidinol in both cell lines, yet their expression levels show only an incomplete correlation with those of the transgene (Fig 4B, S5B Fig, and not shown). |
| 4 | 187 | 187 | 19.125 | In addition, this pathway has been typically assessed in transient assays, lasting for a few hours, which may not be comparable with the prolonged starvation conditions necessary to reactivate the transgene expression (at least 15–24 h). |
| 5 | 192 | 192 | 18.75 | Thus, while the ISR appears widely activated upon EAA starvation, the upregulation of its downstream effector CHOP only partly correlates with transgene reactivation and may not be sufficient to induce it. |
| 6 | 191 | 191 | 18.5 | In fact, CHOP is highly upregulated even upon Trp starvation, which consistently results in no or minimal reactivation of the transgenes (compare CHOP with OA1 or GFP expression; Fig 3 and S4B Fig). |
| 7 | 189 | 189 | 18.375 | Similar, yet less pronounced, results were obtained with ATF4, consistent with the notion that activation of this transcription factor is mainly mediated by translational upregulation (not shown) [15, 26]. |
| 8 | 186 | 186 | 17.25 | By contrast, the involvement of GCN2 and the ISR, including the downstream effectors ATF4 and CHOP, has never been tested. |
| 9 | 213 | 213 | 16.125 | However, differently from HepG2-OA1 cells, simultaneous treatment of C2C12-GFP cells with L-Histidinol and PP242 does not lead to synergistic effects. |
| 10 | 184 | 184 | 16.0 | mTORC1 inhibition and GCN2 activation trigger the best-known signaling pathways responding to AA starvation . |
- GPT 引用1: span=[187, 191], relevant=1

### 陈述 2
- Pharmacological activation of GCN2 by L-Histidinol is sufficient to reactivate the transgenes in some cell types, but the ISR inhibitor ISRIB does not block this reactivation. This suggests the ISR is not necessary for transgene reactivation.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[196-196]`
- au=2.278165340423584, eu=0.05031445994973183, au′=4.235926151275635, eu′=0.062062278389930725
- top_logits 方差 (pvariance, 有限 logit): 0.58125
- top_logits 数值: `[20.5, 19.625, 19.125, 19.125, 18.875, 18.875, 18.625, 18.25, 18.0, 17.75]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 197 | 197 | 20.5 | As shown in Fig 4 and S5 Fig, while inhibition of mTORC1 consistently leads to minor or no effects, in agreement with previous findings , treatment with L-Histidinol results in efficient reactivation of the transgene in HepG2-OA1 and C2C12-GFP cells, but not in HeLa cells. |
| 2 | 241 | 241 | 19.625 | To test whether the ISR is necessary to trigger the transgene response to L-Histidinol, we used the chemical compound ISRIB, which inhibits the activation of the ISR, even in the presence of phosphorylated eIF2α, likely by boosting the activity of the guanine-nucleotide exchange factor (GEF) for eIF2α, namely eIF2B [53, 54]. |
| 3 | 196 | 196 | 19.125 | To this aim, HeLa-OA1 or GFP, HepG2-OA1 and C2C12-GFP cells were cultured in the presence of different concentrations of PP242 (mTOR inhibitor) or L-Histidinol (GCN2 activator, inhibiting tRNAHis charging by histidyl-tRNA synthetase), either alone or in combination for 24 h, compared to Met/Cys-deprived and full medium. |
| 4 | 214 | 214 | 19.125 | Consistent with stimulation of the ISR, CHOP and to a minor extent ATF4 are upregulated by L-Histidinol in both cell lines, yet their expression levels show only an incomplete correlation with those of the transgene (Fig 4B, S5B Fig, and not shown). |
| 5 | 211 | 211 | 18.875 | Consistently, in HepG2-OA1 cells, L-Histidinol is sufficient to elicit extremely high levels of transgene reactivation, and its combination with PP242 results in additive or even synergistic effects, possibly due to an indirect effect of mTOR inhibition on GCN2 activity (Fig 4B) [50, 51]. |
| 6 | 215 | 215 | 18.875 | The finding that GCN2 activation by L-Histidinol is sufficient to reactivate the transgenes in both HepG2-OA1 and C2C12-GFP cells pointed to this kinase, and to the downstream ISR, as the pathway possibly involved in the EAA starvation response. |
| 7 | 212 | 212 | 18.625 | Similarly, C2C12-GFP cells efficiently reactivate the transgene upon treatment with L-Histidinol, but not PP242 (S5B Fig). |
| 8 | 194 | 194 | 18.25 | Therefore, we tested whether direct pharmacological activation of this kinase is sufficient to trigger the transgene reactivation similarly to starvation. |
| 9 | 70 | 70 | 18.0 | Treatments with drugs to be tested for their ability to inhibit transgene reactivation (ISRIB, SP600125 and U0126) were initiated 1h before the subsequent addition of L-Histidinol (ISRIB) or the subsequent depletion of Met/Cys (SP600125 and U0126). |
| 10 | 206 | 206 | 17.75 | Specifically, L-Histidinol is not effective in HeLa-OA1 and HeLa-GFP cells, either alone or in combination with PP242 (Fig 4A and S5A Fig), or by using different concentrations of the drug, with or without serum (not shown). |

**跨度** `[240-242]`
- au=2.278165340423584, eu=0.05031445994973183, au′=4.235926151275635, eu′=0.062062278389930725
- top_logits 方差 (pvariance, 有限 logit): 0.58125
- top_logits 数值: `[20.5, 19.625, 19.125, 19.125, 18.875, 18.875, 18.625, 18.25, 18.0, 17.75]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 197 | 197 | 20.5 | As shown in Fig 4 and S5 Fig, while inhibition of mTORC1 consistently leads to minor or no effects, in agreement with previous findings , treatment with L-Histidinol results in efficient reactivation of the transgene in HepG2-OA1 and C2C12-GFP cells, but not in HeLa cells. |
| 2 | 241 | 241 | 19.625 | To test whether the ISR is necessary to trigger the transgene response to L-Histidinol, we used the chemical compound ISRIB, which inhibits the activation of the ISR, even in the presence of phosphorylated eIF2α, likely by boosting the activity of the guanine-nucleotide exchange factor (GEF) for eIF2α, namely eIF2B [53, 54]. |
| 3 | 196 | 196 | 19.125 | To this aim, HeLa-OA1 or GFP, HepG2-OA1 and C2C12-GFP cells were cultured in the presence of different concentrations of PP242 (mTOR inhibitor) or L-Histidinol (GCN2 activator, inhibiting tRNAHis charging by histidyl-tRNA synthetase), either alone or in combination for 24 h, compared to Met/Cys-deprived and full medium. |
| 4 | 214 | 214 | 19.125 | Consistent with stimulation of the ISR, CHOP and to a minor extent ATF4 are upregulated by L-Histidinol in both cell lines, yet their expression levels show only an incomplete correlation with those of the transgene (Fig 4B, S5B Fig, and not shown). |
| 5 | 211 | 211 | 18.875 | Consistently, in HepG2-OA1 cells, L-Histidinol is sufficient to elicit extremely high levels of transgene reactivation, and its combination with PP242 results in additive or even synergistic effects, possibly due to an indirect effect of mTOR inhibition on GCN2 activity (Fig 4B) [50, 51]. |
| 6 | 215 | 215 | 18.875 | The finding that GCN2 activation by L-Histidinol is sufficient to reactivate the transgenes in both HepG2-OA1 and C2C12-GFP cells pointed to this kinase, and to the downstream ISR, as the pathway possibly involved in the EAA starvation response. |
| 7 | 212 | 212 | 18.625 | Similarly, C2C12-GFP cells efficiently reactivate the transgene upon treatment with L-Histidinol, but not PP242 (S5B Fig). |
| 8 | 194 | 194 | 18.25 | Therefore, we tested whether direct pharmacological activation of this kinase is sufficient to trigger the transgene reactivation similarly to starvation. |
| 9 | 70 | 70 | 18.0 | Treatments with drugs to be tested for their ability to inhibit transgene reactivation (ISRIB, SP600125 and U0126) were initiated 1h before the subsequent addition of L-Histidinol (ISRIB) or the subsequent depletion of Met/Cys (SP600125 and U0126). |
| 10 | 206 | 206 | 17.75 | Specifically, L-Histidinol is not effective in HeLa-OA1 and HeLa-GFP cells, either alone or in combination with PP242 (Fig 4A and S5A Fig), or by using different concentrations of the drug, with or without serum (not shown). |
- GPT 引用1: span=[196, 196], relevant=0
- GPT 引用2: span=[240, 242], relevant=1

### 陈述 3
- Knockdown of ATF4, a downstream effector of the ISR, does not interfere with the reactivation of the transgene by L-Histidinol.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[243-243]`
- au=2.2726454734802246, eu=0.05372733250260353, au′=4.095000267028809, eu′=0.06921650469303131
- top_logits 方差 (pvariance, 有限 logit): 2.96
- top_logits 数值: `[21.25, 19.625, 19.0, 17.875, 17.0, 17.0, 16.5, 16.25, 15.8125, 15.8125]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 244 | 244 | 21.25 | In addition, knockdown of the closest eIF2α downstream effector ATF4 by siRNAs does not interfere with the reactivation of the OA1 transgene by L-Histidinol (Fig 5D). |
| 2 | 236 | 236 | 19.625 | (D) Relative transgene (OA1) and ATF4 mRNA abundance in HepG2-OA1 cells transfected with control (CTRL) or anti-ATF4 siRNAs, and incubated in the presence or absence of L-Histidinol (HisOH, GCN2 activator; 4 mM) for 6 h. Mean ± range of two experiments. |
| 3 | 243 | 243 | 19.0 | As shown in Fig 5C, while the expression of CHOP is inhibited by ISRIB, as expected, the reactivation of the OA1 transgene is not affected. |
| 4 | 241 | 241 | 17.875 | To test whether the ISR is necessary to trigger the transgene response to L-Histidinol, we used the chemical compound ISRIB, which inhibits the activation of the ISR, even in the presence of phosphorylated eIF2α, likely by boosting the activity of the guanine-nucleotide exchange factor (GEF) for eIF2α, namely eIF2B [53, 54]. |
| 5 | 245 | 245 | 17.0 | Together, these data suggest that eIF2α phosphorylation and the downstream ISR pathway are neither sufficient nor necessary to induce transgene reactivation. |
| 6 | 242 | 242 | 17.0 | HepG2-OA1 cells were stimulated with L-Histidinol, either in the presence or absence of ISRIB. |
| 7 | 238 | 238 | 16.5 | control (w/o HisOH = 1, top; control siRNA = 1, bottom). |
| 8 | 214 | 214 | 16.25 | Consistent with stimulation of the ISR, CHOP and to a minor extent ATF4 are upregulated by L-Histidinol in both cell lines, yet their expression levels show only an incomplete correlation with those of the transgene (Fig 4B, S5B Fig, and not shown). |
| 9 | 240 | 240 | 15.8125 | control, unless otherwise indicated). |
| 10 | 237 | 237 | 15.8125 | Data are expressed as fold change vs. |
- GPT 引用1: span=[243, 243], relevant=1

### 陈述 4
- CRISPR/Cas9-mediated knockout of GCN2 does not abolish or reduce transgene reactivation in response to essential amino acid starvation in HeLa or HepG2 cells. This excludes GCN2 and the ISR as necessary for this response.
- GPT support_score=1; **Rating: [[Fully supported]]**   **

**跨度** `[245-250]`
- au=2.2770915031433105, eu=0.04842614382505417, au′=4.646625518798828, eu′=0.06433050334453583
- top_logits 方差 (pvariance, 有限 logit): 2.224375
- top_logits 数值: `[23.0, 21.625, 20.125, 19.75, 19.125, 19.125, 19.0, 18.375, 18.25, 18.125]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 246 | 246 | 23.0 | To definitively establish if GCN2 is necessary to trigger the transgene reactivation response to EAA starvation, we directly suppressed its expression by CRISPR/Cas9-mediated knock-out (KO). |
| 2 | 247 | 247 | 21.625 | We generated three independent GCN2-KO clones from the parental HeLa-OA1 cell line, by using three different guide RNAs, two against exon 1 (clones 183#11 and 185#5), and one against exon 6 (clone 239#1) of the GCN2 gene. |
| 3 | 251 | 251 | 20.125 | We found that the reactivation of the OA1 transgene is neither abolished, nor reduced by KO of GCN2, thus excluding that this kinase is necessary for the response to EAA starvation in HeLa-OA1 cells (Fig 6B and 6C). |
| 4 | 245 | 245 | 19.75 | Together, these data suggest that eIF2α phosphorylation and the downstream ISR pathway are neither sufficient nor necessary to induce transgene reactivation. |
| 5 | 248 | 248 | 19.125 | Genomic characterization confirmed the presence of mutations on both alleles of exon 1 of the GCN2 gene in clone 183#11, and on both alleles of exon 6 in clone 239#1; by contrast, clone 185#5 showed multiple alleles in exon 1, consistent with the presence of two cell populations, and was not characterized further at the genomic level (S6 Fig). |
| 6 | 250 | 250 | 19.125 | To test the GCN2-KO cells for their ability to respond to EAA starvation, parental HeLa-OA1 cells and the three GCN2-KO clones were cultured in media deprived of Met/Cys or Thr (corresponding to the most effective treatments in this cell line; see Fig 3A) for 24–48 h and transgene expression was assessed by RT-qPCR. |
| 7 | 249 | 249 | 19.0 | None of these clones express GCN2 at the protein level, as shown by immunoblotting (Fig 6A). |
| 8 | 244 | 244 | 18.375 | In addition, knockdown of the closest eIF2α downstream effector ATF4 by siRNAs does not interfere with the reactivation of the OA1 transgene by L-Histidinol (Fig 5D). |
| 9 | 265 | 265 | 18.25 | Genomic characterization confirmed the presence of mutations on both alleles of exon 1 of the GCN2 gene in clone F27 (S7 Fig) and all three clones showed a very low amount—if any—of residual GCN2 protein, compared to the original HepG2-OA1 cell line (Fig 7A). |
| 10 | 264 | 264 | 18.125 | By using a guide RNA against exon 1 of the GCN2 gene, we obtained three independent GCN2-KO clones, namely E23, F22 and F27. |

**跨度** `[262-268]`
- au=2.2770915031433105, eu=0.04842614382505417, au′=4.646625518798828, eu′=0.06433050334453583
- top_logits 方差 (pvariance, 有限 logit): 2.224375
- top_logits 数值: `[23.0, 21.625, 20.125, 19.75, 19.125, 19.125, 19.0, 18.375, 18.25, 18.125]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 246 | 246 | 23.0 | To definitively establish if GCN2 is necessary to trigger the transgene reactivation response to EAA starvation, we directly suppressed its expression by CRISPR/Cas9-mediated knock-out (KO). |
| 2 | 247 | 247 | 21.625 | We generated three independent GCN2-KO clones from the parental HeLa-OA1 cell line, by using three different guide RNAs, two against exon 1 (clones 183#11 and 185#5), and one against exon 6 (clone 239#1) of the GCN2 gene. |
| 3 | 251 | 251 | 20.125 | We found that the reactivation of the OA1 transgene is neither abolished, nor reduced by KO of GCN2, thus excluding that this kinase is necessary for the response to EAA starvation in HeLa-OA1 cells (Fig 6B and 6C). |
| 4 | 245 | 245 | 19.75 | Together, these data suggest that eIF2α phosphorylation and the downstream ISR pathway are neither sufficient nor necessary to induce transgene reactivation. |
| 5 | 248 | 248 | 19.125 | Genomic characterization confirmed the presence of mutations on both alleles of exon 1 of the GCN2 gene in clone 183#11, and on both alleles of exon 6 in clone 239#1; by contrast, clone 185#5 showed multiple alleles in exon 1, consistent with the presence of two cell populations, and was not characterized further at the genomic level (S6 Fig). |
| 6 | 250 | 250 | 19.125 | To test the GCN2-KO cells for their ability to respond to EAA starvation, parental HeLa-OA1 cells and the three GCN2-KO clones were cultured in media deprived of Met/Cys or Thr (corresponding to the most effective treatments in this cell line; see Fig 3A) for 24–48 h and transgene expression was assessed by RT-qPCR. |
| 7 | 249 | 249 | 19.0 | None of these clones express GCN2 at the protein level, as shown by immunoblotting (Fig 6A). |
| 8 | 244 | 244 | 18.375 | In addition, knockdown of the closest eIF2α downstream effector ATF4 by siRNAs does not interfere with the reactivation of the OA1 transgene by L-Histidinol (Fig 5D). |
| 9 | 265 | 265 | 18.25 | Genomic characterization confirmed the presence of mutations on both alleles of exon 1 of the GCN2 gene in clone F27 (S7 Fig) and all three clones showed a very low amount—if any—of residual GCN2 protein, compared to the original HepG2-OA1 cell line (Fig 7A). |
| 10 | 264 | 264 | 18.125 | By using a guide RNA against exon 1 of the GCN2 gene, we obtained three independent GCN2-KO clones, namely E23, F22 and F27. |
- GPT 引用1: span=[245, 250], relevant=1
- GPT 引用2: span=[262, 268], relevant=1

### 陈述 5
Therefore, while the ISR is activated upon essential amino acid starvation, it does not appear to be necessary for the reactivation of silenced transgenes in cultured cells. The mechanism by which essential amino acid starvation leads to transgene reactivation remains unclear, but it does not involve GCN2 and the ISR as previously suggested.
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

## squality_idx384_last (idx=384, squality)
### 问题
What is the significance of the mystery metal from the starship?

### 参考答案（数据集）
- ref1: The mystery metal is significant because it initially attracted the crew’s interest due to their greed - they had hoped to tear about the derelict starship and sell its pieces for millions. When the Captain tested out the mysterious metal and saw that it turned out to be gold, his greed increased so much that he became suspicious of his crew members that were sent out to investigate the ship.   Although the Captain and his crew thought they could take advantage of this metal and benefit from it, it turns out that the opposite is true. Instead, it is this mystery metal that gains its yellow-tint and subsequent gold composition through drawing its energy from them and draining the crew of their youth and strength. The latter named ‘devil-metal’ demonstrates the hastiness of the greed of man,…
- ref2: All the spacemen in this story are greedy and materialistic. When the Maid’s crew finds the derelict, they think of selling its parts. This thought cheers them up. Then Midas finds out the starship’s metal can turn into gold, and Spinelli learns that, too. Eventually, this chunk of gold makes Midas’ hands slowly decay while he sleeps with it every night. Spinelli becomes more and more suspicious and suspects everyone to be a traitor. Eventually, Midas realizes the starship’s metal has some evil in it. When he finds his team almost dead and still trying to climb the golden walls of the room, he understands that the metal takes the energy required for its stability from humans. This gold quite literally kills. Greed ends the lives of almost all crewmates of the Maid and leaves Midas to slowl…
- ref3: The mystery metal from the derelict can draw energy from life and turn itself into gold through the transmutation of the energy. In the story, the crews on Maid, a spaceship, find a large derelict constituted of this mysterious metal and bring it with them. Soon after they find out its property of becoming gold, they start to fight with each other and caress the metal unstoppably, but they do not know where the energy that makes the metal change comes from. After the captain kills one crewmember, finding the lost signals and weirdness of his hand and the metal, he realizes the metal draws the needed energy from humans. The mysterious metal plays a significant role in that it triggers the greed of the crewmembers to cause them to fight, symbolizing the cursed treasure. It is also the leadin…
- ref4: The mystery metal, later revealed to be ‘gold’, is significant because it is what drives the crew to want to become rich and leads to their downfall. When the captain first discovers it is gold, he thinks about how wealthy the entire crew will become once they reach Callisto and sell it off. The mystery metal is worth a lot of money, and it is what makes them decide to take the entire ship with them. However, this metal is also deadly because it sucks the energy from the crew. Out of their greed, they fail to realize that the gold drains their lifespan away to continue functioning. The crew has to pay the ultimate price with their lives, and only Captain Midas survives the incident. Even as they are old and skeletal, the rest of the crew do not want to give up the possibility of gold. When…

### 模型全文输出
```
<statement>Based on the story, the significance of the mystery metal from the starship is that it is revealed to be gold.<cite></cite></statement>

<statement>- The metal is taken from samples of the pressure hull of the starship.<cite>[164-164]</cite></statement>

<statement>- The captain tests the metal and discovers it has changed into gold.<cite>[177-190]</cite></statement>

<statement>- The captain is horrified to discover that the metal has drawn energy from the crew of the Martian Maid, causing them to age and die.<cite>[358-359]</cite></statement>

<statement>- The captain throws the gold into space to escape the curse of the treasure.<cite>[365-365]</cite></statement>

<statement>So in summary, the metal is significant because it is revealed to be gold, but the cost of obtaining it is the lives of the crew and the captain's own health and youth. The gold is cursed treasure that brings only pain and sorrow.<cite></cite></statement>
```

文档句索引范围: sid ∈ [1, 382], 共 382 句。
引用指标: R=0.25, P=0.25, F1=0.25
GPT 引用评判 token（`eval_gpt_cite_json`）: prompt_tokens=3339, completion_tokens=81
答案 GPT（eval_correct / gpt_score_summ，1–5 per reference (summary: coherence, compression, …)）: normalized_0_1=1.0, raw=['[[3]]', '[[3]]', '[[3]]', '[[2]]']

### 陈述 0
Based on the story, the significance of the mystery metal from the starship is that it is revealed to be gold.
- GPT support_score=0; [[Yes]]  Need Citation: [[Yes]] Analysis

### 陈述 1
- The metal is taken from samples of the pressure hull of the starship.
- GPT support_score=0; Rating: [[No support]]

**跨度** `[164-164]`
- au=1.0829732418060303, eu=0.13523128628730774, au′=1.0829730033874512, eu′=0.04481131583452225
- top_logits 方差 (pvariance, 有限 logit): 0.15004619446942039
- top_logits 数值: `[21.842105865478516, 21.18421173095703, 20.921052932739258]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 165 | 165 | 21.842105865478516 | He dropped two chunks of metal on my desk. |
| 2 | 164 | 164 | 21.18421173095703 | The hull alone was left. |
| 3 | 163 | 163 | 20.921052932739258 | She was never built to carry humanoids he told us, and there was nothing that could give us a hint of where she had come from. |
- GPT 引用1: span=[164, 164], relevant=0

### 陈述 2
- The captain tests the metal and discovers it has changed into gold.
- GPT support_score=0.5; **Rating: [[Partially supported]]**

**跨度** `[177-190]`
- au=1.5912408828735352, eu=0.08216214179992676, au′=1.591240644454956, eu′=0.04284102842211723
- top_logits 方差 (pvariance, 有限 logit): 0.4598340211485629


# 三样本：每句 top-logit 方差、eu、GPT 引用分

方差与 eu 为 `cite_spans_model` 中每个 `<cite>[a-b]` 跨度；多段数值用 `;` 对齐。TSV 中 cite_spans 多段用 `|`；Markdown 表内将 `|` 换成 `·` 以免破坏 pipe 表格。

| sample | idx | stmt | cite_spans | var / span | mean var | eu / span | mean eu | support | rel / cite | mean rel | ans norm | ans raw |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| multifieldqa_en_idx50 | 50 | 0 | [28-28] | 8.149062 | 8.149062 | 0.062794 | 0.062794 | 1 | 1 | 1.000000 | 1.0 | [[3]] |
| multifieldqa_en_idx50 | 50 | 1 | [28-28] | 7.923594 | 7.923594 | 0.058954 | 0.058954 | 1 | 1 | 1.000000 | 1.0 | [[3]] |
| multifieldqa_en_idx50 | 50 | 2 |  |  |  |  |  | 0 |  |  | 1.0 | [[3]] |
| multifieldqa_en_idx51 | 51 | 0 |  |  |  |  |  | 0 |  |  |  |  |
| multifieldqa_en_idx51 | 51 | 1 | [187-191] | 3.215000 | 3.215000 | 0.050891 | 0.050891 | 1 | 1 | 1.000000 |  |  |
| multifieldqa_en_idx51 | 51 | 2 | [196-196]·[240-242] | 0.581250;0.581250 | 0.581250 | 0.050314;0.050314 | 0.050314 | 1 | 0;1 | 0.500000 |  |  |
| multifieldqa_en_idx51 | 51 | 3 | [243-243] | 2.960000 | 2.960000 | 0.053727 | 0.053727 | 1 | 1 | 1.000000 |  |  |
| multifieldqa_en_idx51 | 51 | 4 | [245-250]·[262-268] | 2.224375;2.224375 | 2.224375 | 0.048426;0.048426 | 0.048426 | 1 | 1;1 | 1.000000 |  |  |
| multifieldqa_en_idx51 | 51 | 5 |  |  |  |  |  | 0 |  |  |  |  |
| squality_idx384_last | 384 | 0 |  |  |  |  |  | 0 |  |  | 1.0 | [[3]];[[3]];[[3]];[[2]] |
| squality_idx384_last | 384 | 1 | [164-164] | 0.150046 | 0.150046 | 0.135231 | 0.135231 | 0 | 0 | 0.000000 | 1.0 | [[3]];[[3]];[[3]];[[2]] |
| squality_idx384_last | 384 | 2 | [177-190] | 0.459834 | 0.459834 | 0.082162 | 0.082162 | 0.5 | 1 | 1.000000 | 1.0 | [[3]];[[3]];[[3]];[[2]] |
| squality_idx384_last | 384 | 3 | [358-359] | 0.264288 | 0.264288 | 0.065517 | 0.065517 | 0 | 0 | 0.000000 | 1.0 | [[3]];[[3]];[[3]];[[2]] |
| squality_idx384_last | 384 | 4 | [365-365] | 0.050015 | 0.050015 | 0.127517 | 0.127517 | 0 | 0 | 0.000000 | 1.0 | [[3]];[[3]];[[3]];[[2]] |
| squality_idx384_last | 384 | 5 |  |  |  |  |  | 1 |  |  | 1.0 | [[3]];[[3]];[[3]];[[2]] |

- top_logits 数值: `[23.68421173095703, 22.10526466369629, 22.10526466369629, 21.973684310913086, 21.842105865478516]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 182 | 182 | 23.68421173095703 | Then I began testing, trying all the while to quiet the trembling of my hands. |
| 2 | 178 | 178 | 22.10526466369629 | It had a yellowish tinge, and it was heavier ....  Even as I watched, the metal grew yellower, and the hand that held it grew bone weary, little tongues of fatigue licking up my forearm. |
| 3 | 177 | 177 | 22.10526466369629 | I held the stuff in my bare hand. |
| 4 | 183 | 183 | 21.973684310913086 | I weighed it on a balance. |
| 5 | 190 | 190 | 21.842105865478516 | The unknown metal—the stuff of which that whole mammoth spaceship from the stars was built—was now.... Gold! |
- GPT 引用1: span=[177, 190], relevant=1

### 陈述 3
- The captain is horrified to discover that the metal has drawn energy from the crew of the Martian Maid, causing them to age and die.
- GPT support_score=0; Rating: [[No support]]

**跨度** `[358-359]`
- au=1.924755573272705, eu=0.06551723182201385, au′=1.924755573272705, eu′=0.04678156226873398
- top_logits 方差 (pvariance, 有限 logit): 0.2642881122898258
- top_logits 数值: `[21.052631378173828, 20.921052932739258, 20.789474487304688, 20.263158798217773, 20.131580352783203, 19.86842155456543, 19.60526466369629]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 357 | 357 | 21.052631378173828 | I knew what my wrinkled aged hands meant. |
| 2 | 347 | 347 | 20.921052932739258 | But in those moments I think I was discovering what had happened to my crew and why the walls were turning into yellow metal. |
| 3 | 356 | 356 | 20.789474487304688 | I understood then. |
| 4 | 177 | 177 | 20.263158798217773 | I held the stuff in my bare hand. |
| 5 | 358 | 358 | 20.131580352783203 | That devil-metal from beyond the stars had drawn the energy it needed from ... us ! |
| 6 | 178 | 178 | 19.86842155456543 | It had a yellowish tinge, and it was heavier ....  Even as I watched, the metal grew yellower, and the hand that held it grew bone weary, little tongues of fatigue licking up my forearm. |
| 7 | 343 | 343 | 19.60526466369629 | Decks that had a yellowish cast ... decks that no longer danced with tiny questing force-whorls....  As I approached the airlock of the compartment set aside as living quarters for the prize crew, the saffron of the walls deepened. |
- GPT 引用1: span=[358, 359], relevant=0

### 陈述 4
- The captain throws the gold into space to escape the curse of the treasure.
- GPT support_score=0; Rating: [[No support]]

**跨度** `[365-365]`
- au=1.084091305732727, eu=0.1275167465209961, au′=1.0840907096862793, eu′=0.04200441762804985
- top_logits 方差 (pvariance, 有限 logit): 0.05001539815647346
- top_logits 数值: `[23.026315689086914, 22.894737243652344, 22.5]`

| rank | token | 解析句号 | logit | 该句内容（全文） |
|---|---|---|---|---|
| 1 | 364 | 364 | 23.026315689086914 | My hands burned like fire! |
| 2 | 363 | 363 | 22.894737243652344 | I ran from that place still screaming with the horror of it. |
| 3 | 365 | 365 | 22.5 | Age was in them, creeping like molten lead through my veins, ghastly and sure....  I reached the Maid and threw every scrap of that alien metal into space as I streaked madly away from that golden terror in the sky and its load of ancient evil....     On Callisto I was relieved of my command. |
- GPT 引用1: span=[365, 365], relevant=0

### 陈述 5
So in summary, the metal is significant because it is revealed to be gold, but the cost of obtaining it is the lives of the crew and the captain's own health and youth. The gold is cursed treasure that brings only pain and sorrow.
- GPT support_score=1; [[No]]   Need Citation: [[No]]
