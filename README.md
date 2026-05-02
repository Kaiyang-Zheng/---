# 两样本：引用跨度 logits / au·eu / 方差 / GPT 与候选句

## 阅读说明（量表与表格列）

- **答案分 `raw`**：来自 `scores_correct/*.json` 中 `gpt_usage.responses`，由 `eval_correct.py` 生成。 **multifieldqa_en** 走 `gpt_score_qa`：每条参考输出 **`[[1]]`–`[[3]]`**（问答正确性/覆盖面）。 **squality** 走 `gpt_score_summ`：每条参考输出 **`[[1]]`–`[[5]]`**（摘要：正确性、覆盖、**连贯性**、压缩效率等）。
  - 因此：SQuALITY 若出现 `[[3]]`、`[[2]]`，应读作 **3/5、2/5**，**不要**当成问答量表里的「满分 3 分」。
- **`normalized_0_1`**：与 `eval_correct` 写回的 `score` 一致；摘要侧单条参考为 `(rating−1)/4` 再与多参考聚合（见 `eval_correct.process`）。
- **候选句表「解析句号 / 该句内容」**：把 top 数字 token 当作句编号 sid，在 **bench 文档**经 `ensure_longcite_indexed_context` 后的 `<Csid>` 上查句。 若某 sid 下文字与 gold 引用段落不一致，多为**切句索引与当时生成引用不一致**，便于对照「模型若选该编号会指到哪句」，不等于 logits 错误。


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
答案 GPT（eval_correct / gpt_score_qa，1–3 per reference (QA)）: normalized_0_1=1.0, raw=['[[3]]']

### 陈述 0
Based on the information provided, the most active group of fans for FC Banants is the South West Ultras fan club.
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[28-28]`
- au=2.256385564804077, eu=0.06279434263706207, au′=3.042393445968628, eu′=0.06922319531440735
- top_logits 方差 (pvariance, 有限 logit): 8.1490625
- top_logits 数值: `[22.875, 16.25, 15.3125, 14.875, 14.0625, 13.5, 13.4375, 13.125, 12.9375, 12.875]`

| rank | token | 解析句号 | logit | 该句内容（截断） |
|---|---|---|---|---|
| 1 | 29 | 29 | 22.875 | Fans |
| 2 | 28 | 28 | 16.25 | The current technical director of the academy is the former Russian footballer Ilshat Faizulin. |
| 3 | 30 | 30 | 15.3125 | Members of the fan club benefit from events organized by the club and many facilities of the Banants training centre, such as the mini football pitch, the club store and other entertainments. |
| 4 | 27 | 27 | 14.875 | In addition to the main stadium, the centre houses 3 full-size training pitches, mini football pitches as well as an indoor facility. |
| 5 | 26 | 26 | 14.0625 | Training centre/academy |
| 6 | 1 | 1 | 13.5 | Football Club Urartu (, translated Futbolayin Akumb Urartu), commonly known as Urartu, is an Armenian professional football team based in the capital Yerevan that currently plays in the Armenian Premier League. |
| 7 | 31 | 31 | 13.4375 | Achievements |
| 8 | 25 | 25 | 13.125 | Further developments were implemented later in 2011, when the playing pitch was modernized and the capacity of the stadium was increased up to 4,860 seats (2,760 at the northern stand, 1,500 at the southern stand and 600 at the western stand). |
| 9 | 23 | 23 | 12.9375 | Domestic |
| 10 | 5 | 5 | 12.875 | The club was known as FC Banants until 1 August 2019, when it was officially renamed FC Urartu. |
- GPT 引用1: span=[28, 28], relevant=1

### 陈述 1
The passage states: "The most active group of fans is the South West Ultras fan club, mainly composed of residents from several neighbourhoods within the Malatia-Sebastia District of Yerevan, since the club is a de facto representer of the district."
- GPT support_score=1; Rating: [[Fully supported]]

**跨度** `[28-28]`
- au=2.2601747512817383, eu=0.05895356833934784, au′=3.408114194869995, eu′=0.07244941592216492
- top_logits 方差 (pvariance, 有限 logit): 7.92359375
- top_logits 数值: `[22.625, 19.625, 16.375, 16.0, 15.3125, 14.1875, 14.1875, 14.0, 13.8125, 13.5]`

| rank | token | 解析句号 | logit | 该句内容（截断） |
|---|---|---|---|---|
| 1 | 29 | 29 | 22.625 | Fans |
| 2 | 28 | 28 | 19.625 | The current technical director of the academy is the former Russian footballer Ilshat Faizulin. |
| 3 | 27 | 27 | 16.375 | In addition to the main stadium, the centre houses 3 full-size training pitches, mini football pitches as well as an indoor facility. |
| 4 | 30 | 30 | 16.0 | Members of the fan club benefit from events organized by the club and many facilities of the Banants training centre, such as the mini football pitch, the club store and other entertainments. |
| 5 | 26 | 26 | 15.3125 | Training centre/academy |
| 6 | 24 | 24 | 14.1875 | It was officially opened in 2008 with a capacity of 3,600 seats. |
| 7 | 25 | 25 | 14.1875 | Further developments were implemented later in 2011, when the playing pitch was modernized and the capacity of the stadium was increased up to 4,860 seats (2,760 at the northern stand, 1,500 at the southern stand and 600 at the western stand). |
| 8 | 31 | 31 | 14.0 | Achievements |
| 9 | 23 | 23 | 13.8125 | Domestic |
| 10 | 22 | 22 | 13.5 | The club headquarters are located on Jivani Street 2 of the Malatia-Sebastia District, Yerevan. |
- GPT 引用1: span=[28, 28], relevant=1

### 陈述 2
So the South West Ultras fan club is the most active group of fans for FC Banants.
- GPT support_score=0; [[Yes]]   Need Citation: [[Yes]]

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

| rank | token | 解析句号 | logit | 该句内容（截断） |
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
- top_logits 数值: `[23.68421173095703, 22.10526466369629, 22.10526466369629, 21.973684310913086, 21.842105865478516]`

| rank | token | 解析句号 | logit | 该句内容（截断） |
|---|---|---|---|---|
| 1 | 182 | 182 | 23.68421173095703 | Then I began testing, trying all the while to quiet the trembling of my hands. |
| 2 | 178 | 178 | 22.10526466369629 | It had a yellowish tinge, and it was heavier .... |
| 3 | 177 | 177 | 22.10526466369629 | I held the stuff in my bare hand. |
| 4 | 183 | 183 | 21.973684310913086 | I weighed it on a balance. |
| 5 | 190 | 190 | 21.842105865478516 | The unknown metal—the stuff of which that whole mammoth spaceship from the stars was built—was now.... |
- GPT 引用1: span=[177, 190], relevant=1

### 陈述 3
- The captain is horrified to discover that the metal has drawn energy from the crew of the Martian Maid, causing them to age and die.
- GPT support_score=0; Rating: [[No support]]

**跨度** `[358-359]`
- au=1.924755573272705, eu=0.06551723182201385, au′=1.924755573272705, eu′=0.04678156226873398
- top_logits 方差 (pvariance, 有限 logit): 0.2642881122898258
- top_logits 数值: `[21.052631378173828, 20.921052932739258, 20.789474487304688, 20.263158798217773, 20.131580352783203, 19.86842155456543, 19.60526466369629]`

| rank | token | 解析句号 | logit | 该句内容（截断） |
|---|---|---|---|---|
| 1 | 357 | 357 | 21.052631378173828 | I knew what my wrinkled aged hands meant. |
| 2 | 347 | 347 | 20.921052932739258 | But in those moments I think I was discovering what |
| 3 | 356 | 356 | 20.789474487304688 | I understood then. |
| 4 | 177 | 177 | 20.263158798217773 | I held the stuff in my bare hand. |
| 5 | 358 | 358 | 20.131580352783203 | That |
| 6 | 178 | 178 | 19.86842155456543 | It had a yellowish tinge, and it was heavier .... |
| 7 | 343 | 343 | 19.60526466369629 | Decks that had a yellowish |
- GPT 引用1: span=[358, 359], relevant=0

### 陈述 4
- The captain throws the gold into space to escape the curse of the treasure.
- GPT support_score=0; Rating: [[No support]]

**跨度** `[365-365]`
- au=1.084091305732727, eu=0.1275167465209961, au′=1.0840907096862793, eu′=0.04200441762804985
- top_logits 方差 (pvariance, 有限 logit): 0.05001539815647346
- top_logits 数值: `[23.026315689086914, 22.894737243652344, 22.5]`

| rank | token | 解析句号 | logit | 该句内容（截断） |
|---|---|---|---|---|
| 1 | 364 | 364 | 23.026315689086914 | My hands |
| 2 | 363 | 363 | 22.894737243652344 | I ran from that place still screaming with the horror of it. |
| 3 | 365 | 365 | 22.5 | Age was in them, creeping like molten lead through my |
- GPT 引用1: span=[365, 365], relevant=0

### 陈述 5
So in summary, the metal is significant because it is revealed to be gold, but the cost of obtaining it is the lives of the crew and the captain's own health and youth. The gold is cursed treasure that brings only pain and sorrow.
- GPT support_score=1; [[No]]   Need Citation: [[No]]
