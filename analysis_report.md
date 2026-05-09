# 1135 样本引用 logits / AU·EU / GPT 支持度分析报告

## 数据规模

- 样本数：1135
- statement 数：5987
- 解析到数字引用跨度数：7020
- GPT citation 评判条目数：6476

### 数据集分布与引用质量

| dataset         |   samples |   mean_recall |   mean_precision |   mean_f1 |   median_f1 |   samples_with_empty_need |   empty_need_statements |   samples_with_out_of_range |   out_of_range_spans |   mean_span_count |
|:----------------|----------:|--------------:|-----------------:|----------:|------------:|--------------------------:|------------------------:|----------------------------:|---------------------:|------------------:|
| dureader        |       200 |        0.7041 |           0.8907 |    0.7661 |      0.8271 |                       138 |                378.0000 |                           0 |               0.0000 |           10.3050 |
| hotpotqa        |       200 |        0.5900 |           0.7178 |    0.6010 |      0.6667 |                        86 |                128.0000 |                           0 |               0.0000 |            2.6500 |
| multifieldqa_en |       150 |        0.6767 |           0.8215 |    0.7148 |      0.8000 |                        82 |                122.0000 |                           0 |               0.0000 |            4.5333 |
| multifieldqa_zh |       200 |        0.7821 |           0.9453 |    0.8362 |      0.8844 |                       102 |                182.0000 |                           1 |              10.0000 |            3.6750 |
| squality_dev    |       125 |        0.4179 |           0.3485 |    0.3435 |      0.3750 |                        37 |                 53.0000 |                           0 |               0.0000 |            7.7360 |
| squality_test   |       260 |        0.4344 |           0.4059 |    0.3801 |      0.4000 |                        77 |                107.0000 |                           0 |               0.0000 |            7.8731 |


![dataset f1](chart_dataset_mean_f1.png)


## 1. 指标层面相关性

### Span-level：数字位指标 vs GPT support_score

| metric    |    n |   pearson_r |   spearman_rho |   pearson_p |   spearman_p |
|:----------|-----:|------------:|---------------:|------------:|-------------:|
| au_prime  | 7020 |     -0.4682 |        -0.4712 |      0.0000 |       0.0000 |
| eu_prime  | 7020 |     -0.4474 |        -0.4641 |      0.0000 |       0.0000 |
| logit_var | 7020 |     -0.3466 |        -0.4511 |      0.0000 |       0.0000 |
| max_logit | 7020 |      0.4220 |         0.4358 |      0.0000 |       0.0000 |
| top_n     | 7020 |     -0.4198 |        -0.4816 |      0.0000 |       0.0000 |


![span corr](chart_span_spearman_correlation.png)


### Sample-level：样本聚合指标 vs citation_f1

| metric         |    n |   pearson_r |   spearman_rho |   pearson_p |   spearman_p |
|:---------------|-----:|------------:|---------------:|------------:|-------------:|
| mean_au_prime  | 1119 |     -0.5540 |        -0.5636 |      0.0000 |       0.0000 |
| mean_eu_prime  | 1119 |     -0.4784 |        -0.4885 |      0.0000 |       0.0000 |
| mean_logit_var | 1119 |     -0.4813 |        -0.5172 |      0.0000 |       0.0000 |
| mean_max_logit | 1119 |      0.4319 |         0.4515 |      0.0000 |       0.0000 |
| span_count     | 1119 |      0.0113 |        -0.1559 |      0.7064 |       0.0000 |


### 按 GPT support_score 分组的均值/中位数

|   support_score |   au_prime_count |   au_prime_mean |   au_prime_median |   eu_prime_mean |   eu_prime_median |   logit_var_mean |   logit_var_median |   max_logit_mean |   max_logit_median |   top_n_mean |   top_n_median |
|----------------:|-----------------:|----------------:|------------------:|----------------:|------------------:|-----------------:|-------------------:|-----------------:|-------------------:|-------------:|---------------:|
|          0.0000 |        1272.0000 |          1.0166 |            1.0816 |          0.0449 |            0.0445 |           0.2880 |             0.2121 |          22.0573 |            22.1053 |       3.5778 |         3.0000 |
|          0.5000 |        2599.0000 |          0.7305 |            0.6819 |          0.0434 |            0.0430 |           0.2137 |             0.0962 |          22.6542 |            22.6316 |       2.6476 |         2.0000 |
|          1.0000 |        3149.0000 |          0.2175 |           -0.0000 |          0.0405 |            0.0401 |           0.0599 |             0.0000 |          23.9596 |            24.0789 |       1.3944 |         1.0000 |


![AU](chart_support_vs_au_prime_box.png)

![EU](chart_support_vs_eu_prime_box.png)

![var](chart_support_vs_logit_variance_box.png)

![logit](chart_support_vs_max_logit_box.png)


## 2. 错误模式

| error_type                                        |   count |   sample_count |
|:--------------------------------------------------|--------:|---------------:|
| empty citation where Need Citation=Yes            |     970 |            522 |
| high logit≥P90 (25.39) but irrelevant/unsupported |      74 |             60 |
| out-of-range citation span                        |      10 |              1 |


![error counts](chart_error_mode_counts.png)


### 高 logit 但引用不相关/statement 不支持：Top 20

| sample_id              |   stmt_no |   span_start |   span_end |   support_score |   overlap_relevant |   max_logit |   logit_var |   au_prime |   eu_prime |   sample_citation_f1 | statement                                                                                                                                                                                                                                                                                                    |
|:-----------------------|----------:|-------------:|-----------:|----------------:|-------------------:|------------:|------------:|-----------:|-----------:|---------------------:|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| squality_test_idx6     |         5 |          247 |        247 |           0.500 |              0.000 |      29.605 |       0.000 |     -0.000 |      0.033 |                0.652 | - Memories are what make Willard's experiences on the Earth ship so difficult. He is unable to adjust to being around other people and is frightened by their presence. His memories of life on Earth are hazy and confused, and he is unable to understand the strangeness of the people on the Earth ship. |
| squality_test_idx58    |         5 |          565 |        565 |           0.500 |              0.000 |      28.421 |       0.000 |     -0.000 |      0.034 |                0.583 | - She has electronic guards protecting her and the Travelers Aid Bureau. This suggests she is involved in something important or valuable that people want to harm her for.                                                                                                                                  |
| squality_dev_idx13     |         1 |          559 |        560 |           0.500 |              0.000 |      27.632 |       0.000 |     -0.000 |      0.035 |                0.308 | - It is a luxury item that the settlers on Adobe have been unable to produce due to the harsh conditions on the planet. The Flap-jacks are able to produce wine, and share some with the settlers as a peace offering.                                                                                       |
| multifieldqa_zh_idx378 |         6 |          105 |        106 |           1.000 |              0.000 |      27.500 |       0.000 |     -0.000 |      0.035 |                0.712 | 四、本条“利息”一语是指从各种债权取得的所得，不论其有无抵押担保或者是否有权分享债务人的利润；特别是从公债、债券或者信用债券取得的所得，包括其溢价和奖金。                                                                                                                                                     |
| squality_test_idx38    |         1 |          381 |        381 |           0.500 |              0.000 |      27.237 |       0.000 |     -0.000 |      0.035 |                0.000 | Throughout the story, Captain Linden is injured by a club thrown by Gravgak when Gravgak is trying to defend himself from the attacking warriors. He is unconscious for several weeks and is cared for by Vauna and Omosla. He regains his health gradually but is still weak when the Benzendellas decide t |
| dureader_idx767        |        15 |          149 |        151 |           0.000 |              0.000 |      27.105 |       0.000 |     -0.000 |      0.036 |                0.914 | 8. 定期产检                                                                                                                                                                                                                                                                                                  |
| squality_dev_idx118    |         0 |           72 |         72 |           0.500 |              0.000 |      26.842 |       0.000 |     -0.000 |      0.036 |                0.000 | Based on the story, the 27 women were stranded on the asteroid after their space ship crashed. They were originally intended to be wives for the colonists on Jupiter, but only 27 of them survived the crash. They were able to repair their ship somewhat and lived on the asteroid for three years, survi |
| multifieldqa_zh_idx279 |         4 |          255 |        255 |           0.000 |              1.000 |      26.842 |       0.000 |     -0.000 |      0.036 |                0.880 | 4. 以胡锦涛同志为总书记的党中央的指示和要求                                                                                                                                                                                                                                                                  |
| squality_dev_idx80     |         0 |            6 |          6 |           0.000 |              0.000 |      26.579 |       0.000 |     -0.000 |      0.036 |                0.000 | The plot of "Morgue Ship" by Ray Bradbury is about a spaceship called the Constellation that transports the bodies of space warriors back to Earth. The story follows Sam Burnett, a coroner on the ship, who is on his last trip. He is joined by Rice, a new crew member, who is excited about his first m |
| multifieldqa_en_idx67  |         3 |          106 |        106 |           0.500 |              0.000 |      26.579 |       0.000 |     -0.000 |      0.036 |                0.364 | 2. The test ball is considered with five test particles at rest relative to each other initially. Their motion is described by Taylor expansions around their initial positions.                                                                                                                             |
| multifieldqa_en_idx67  |         4 |          118 |        118 |           0.500 |              0.000 |      26.579 |       0.277 |      0.683 |      0.037 |                0.364 | 3. The distance between the particles is calculated as they move due to tidal forces. The radial distance changes due to both the linear and quadratic terms in the Taylor expansion, while the transverse distance changes only due to the quadratic term.                                                  |
| squality_dev_idx12     |         1 |           17 |         17 |           0.500 |              0.000 |      26.579 |       0.000 |     -0.000 |      0.036 |                0.375 | The story mentions that Adobe is in a backwater system and has eight inhabited planets.                                                                                                                                                                                                                      |
| squality_dev_idx102    |         3 |          417 |        417 |           0.500 |              0.000 |      26.579 |       0.000 |     -0.000 |      0.036 |                0.609 | He is radioactive but not much, and uses ordinary matter on an atomic level to create radioactivity.                                                                                                                                                                                                         |
| multifieldqa_en_idx122 |         2 |          124 |        124 |           0.500 |              0.000 |      26.579 |       0.000 |     -0.000 |      0.036 |                0.636 | - For problems with anisotropic scattering like HGK, NFPA and FPSA still outperform GMRES and DSA, although not as drastically as for forward-peaked problems. Both NFPA and FPSA require more time and iterations as the problem becomes more anisotropic, which is expected since HGK does not have a vali |
| squality_dev_idx13     |         4 |          559 |        560 |           0.500 |              0.000 |      26.447 |       0.000 |     -0.000 |      0.036 |                0.308 | - It is used as a plot device. The settlers are desperate for any alcohol, and the Flap-jacks' wine becomes a valuable bargaining chip in negotiations.                                                                                                                                                      |
| multifieldqa_en_idx75  |         2 |          127 |        127 |           0.000 |              1.000 |      26.447 |       0.000 |     -0.000 |      0.036 |                0.343 | - As the transition probability increases, the learning rate decreases. This is because with a higher transition probability, the agent experiences environmental changes more frequently and needs to learn faster to adapt to the changing environments.                                                   |
| squality_test_idx226   |         3 |          604 |        604 |           0.000 |              1.000 |      26.447 |       0.000 |     -0.000 |      0.036 |                0.429 | - Resourceful - He manages to escape from Pikeville and make his way to Oak Grove on foot despite being pursued.                                                                                                                                                                                             |
| squality_test_idx204   |         3 |          209 |        209 |           0.500 |              0.000 |      26.447 |       0.000 |     -0.000 |      0.036 |                0.522 | - The inventor is aware that the government is not interested in his machine, but that Arcivox is. He realizes that the government often gets things it doesn't know what to do with, and that his machine may fall into this category. This highlights the disconnect between the goals of government and t |
| multifieldqa_en_idx174 |         3 |          214 |        214 |           0.500 |              0.000 |      26.316 |       0.000 |     -0.000 |      0.037 |                0.741 | - It allows the robot to leverage the human's latest observations. The paper shows an example where the goal-only method becomes over-confident in an incorrect goal and fails to search the correct area. In contrast, the approach that accounts for path preference updates its belief more steadily and  |
| multifieldqa_en_idx174 |         3 |          216 |        216 |           0.500 |              0.000 |      26.316 |       0.069 |      0.684 |      0.037 |                0.741 | - It allows the robot to leverage the human's latest observations. The paper shows an example where the goal-only method becomes over-confident in an incorrect goal and fails to search the correct area. In contrast, the approach that accounts for path preference updates its belief more steadily and  |


### 引用空且 Need Citation=Yes：样本级 Top 20

| sample_id              |   idx | dataset         | question                                                                       |   empty_need_statements |   support0_empty |   citation_recall |   citation_precision |   citation_f1 |
|:-----------------------|------:|:----------------|:-------------------------------------------------------------------------------|------------------------:|-----------------:|------------------:|---------------------:|--------------:|
| dureader_idx782        |   782 | dureader        | lolfps突然变得很低                                                             |                      47 |               47 |             0.113 |                1.000 |         0.203 |
| dureader_idx655        |   655 | dureader        | 必赢彩票网打不开                                                               |                      39 |               39 |             0.037 |                1.000 |         0.071 |
| dureader_idx650        |   650 | dureader        | 如何做好公众号                                                                 |                      17 |               17 |             0.422 |                1.000 |         0.593 |
| multifieldqa_zh_idx264 |   264 | multifieldqa_zh | 该产品如何帮助企业掌握档案动态？                                               |                      16 |               16 |             0.105 |                1.000 |         0.190 |
| dureader_idx763        |   763 | dureader        | qq旋风 没速度                                                                  |                      14 |               14 |             0.273 |                1.000 |         0.429 |
| dureader_idx727        |   727 | dureader        | 人民币美元汇率                                                                 |                      11 |               11 |             0.083 |                0.000 |         0.000 |
| dureader_idx672        |   672 | dureader        | 数组成员引用下标必须大于等于1                                                  |                      11 |               11 |             0.227 |                0.727 |         0.346 |
| multifieldqa_zh_idx324 |   324 | multifieldqa_zh | VPN使用结束后如何断开连接？                                                    |                       9 |                9 |             0.250 |                1.000 |         0.400 |
| multifieldqa_zh_idx395 |   395 | multifieldqa_zh | 如何读取夹爪的夹持状态？                                                       |                       8 |                8 |             0.045 |                0.333 |         0.080 |
| dureader_idx658        |   658 | dureader        | 页游为什么那么火                                                               |                       8 |                8 |             0.227 |                1.000 |         0.370 |
| dureader_idx679        |   679 | dureader        | 社会什么意思                                                                   |                       7 |                7 |             0.222 |                1.000 |         0.364 |
| dureader_idx624        |   624 | dureader        | 奇怪的大冒险攻略                                                               |                       7 |                7 |             0.364 |                1.000 |         0.533 |
| dureader_idx744        |   744 | dureader        | 论文访谈记录怎么写                                                             |                       6 |                6 |             0.273 |                0.571 |         0.369 |
| multifieldqa_en_idx159 |   159 | multifieldqa_en | What is the electron correlation parameter, $\Gamma_e$?                        |                       6 |                6 |             0.333 |                1.000 |         0.500 |
| multifieldqa_zh_idx305 |   305 | multifieldqa_zh | 附卡持卡人的責任是什么？                                                       |                       6 |                6 |             0.400 |                1.000 |         0.571 |
| dureader_idx620        |   620 | dureader        | 鼻子周围红红的                                                                 |                       6 |                6 |             0.462 |                1.000 |         0.632 |
| multifieldqa_en_idx54  |    54 | multifieldqa_en | Why is it important for the sides of the fuselage to be sloped (tumbled home)? |                       5 |                5 |             0.286 |                0.000 |         0.000 |
| squality_test_idx189   |   189 | squality_test   | What is the significance of the dream of townspeople?                          |                       5 |                5 |             0.188 |                0.333 |         0.240 |
| dureader_idx640        |   640 | dureader        | 四季豆炒肉的做法                                                               |                       5 |                5 |             0.222 |                0.333 |         0.267 |
| dureader_idx741        |   741 | dureader        | 真菌感染用什么药膏                                                             |                       5 |                5 |             0.333 |                0.500 |         0.400 |


### 越界引用

| sample_id              |   idx | dataset         | question                           |   stmt_no | statement     |   span_start |   span_end |   sid_min |   sid_max |   support_score |   max_logit |   logit_var |   au_prime |   eu_prime |   sample_citation_f1 |
|:-----------------------|------:|:----------------|:-----------------------------------|----------:|:--------------|-------------:|-----------:|----------:|----------:|----------------:|------------:|------------:|-----------:|-----------:|---------------------:|
| multifieldqa_zh_idx210 |   210 | multifieldqa_zh | 主人公身边陆续出现了什么样的人物？ |         5 | 5. 美女钟倩倩 |           47 |         47 |         1 |        36 |           0.000 |      21.842 |       0.623 |      0.681 |      0.045 |                0.476 |
| multifieldqa_zh_idx210 |   210 | multifieldqa_zh | 主人公身边陆续出现了什么样的人物？ |         6 | 6. 妓女秦水瑶 |           99 |         99 |         1 |        36 |           0.000 |      22.105 |       0.000 |     -0.000 |      0.043 |                0.476 |
| multifieldqa_zh_idx210 |   210 | multifieldqa_zh | 主人公身边陆续出现了什么样的人物？ |         7 | 7. 警官       |           66 |         66 |         1 |        36 |           0.000 |      21.316 |       0.000 |     -0.000 |      0.045 |                0.476 |
| multifieldqa_zh_idx210 |   210 | multifieldqa_zh | 主人公身边陆续出现了什么样的人物？ |         8 | 8. 张星宇     |           69 |         69 |         1 |        36 |           0.000 |      23.553 |       0.000 |     -0.000 |      0.041 |                0.476 |
| multifieldqa_zh_idx210 |   210 | multifieldqa_zh | 主人公身边陆续出现了什么样的人物？ |         9 | 9. 薛虎       |          156 |        156 |         1 |        36 |           0.000 |      21.711 |       0.000 |     -0.000 |      0.044 |                0.476 |
| multifieldqa_zh_idx210 |   210 | multifieldqa_zh | 主人公身边陆续出现了什么样的人物？ |        10 | 10. 高熙      |          196 |        196 |         1 |        36 |           0.000 |      21.842 |       0.000 |     -0.000 |      0.044 |                0.476 |
| multifieldqa_zh_idx210 |   210 | multifieldqa_zh | 主人公身边陆续出现了什么样的人物？ |        11 | 11. 蒋丽      |          179 |        179 |         1 |        36 |           0.000 |      23.026 |       0.000 |     -0.000 |      0.042 |                0.476 |
| multifieldqa_zh_idx210 |   210 | multifieldqa_zh | 主人公身边陆续出现了什么样的人物？ |        12 | 12. 钟        |          163 |        163 |         1 |        36 |           0.000 |      19.211 |       1.117 |      1.078 |      0.053 |                0.476 |
| multifieldqa_zh_idx210 |   210 | multifieldqa_zh | 主人公身边陆续出现了什么样的人物？ |        13 | 13. 南山南    |          231 |        231 |         1 |        36 |           0.000 |      23.289 |       0.000 |     -0.000 |      0.041 |                0.476 |
| multifieldqa_zh_idx210 |   210 | multifieldqa_zh | 主人公身边陆续出现了什么样的人物？ |        14 | 14. 孙渺      |          287 |        287 |         1 |        36 |           0.000 |      22.105 |       0.000 |     -0.000 |      0.043 |                0.476 |


## 3. 方法诊断


- AU′/EU′ 与 support_score、citation_f1 均为负相关，方向符合“数字位不确定性越高，引用质量越差”。但注意：这是在“已经生成出可解析数字 span”的条件下统计，970 条 Need Citation=Yes 的空引用失败不会进入 AU/EU 相关性计算，所以相关性会高估可用性。
- logit 方差与质量也为负相关：support_score=1 的 span，logit_var 中位数为 0；support_score=0 的 span，logit_var 中位数约 0.212。这说明“多数字候选竞争”通常是坏信号。
- 但 max_logit 与质量是正相关，不等于“高 logit 一定正确”。本批仍有 74 个 span 满足 max_logit ≥ P90(25.39) 且 citation 不相关或 statement 不支持；典型原因是数字格式/局部位置被强烈记住，而不是证据语义被验证。
- teacher-force prefix / refit 的局限：冻结前缀后重算数字 logits，能测“在这个已生成 statement + cite 格式下模型最想填哪个编号”，但不能判断 statement 是否应该有引用、是否过度扩写、是否越界，也不能解决 `<cite></cite>` 的缺失问题。


## 4. 特定样本

### idx51 / ISR 论文

|   stmt_no |   span_start |   span_end |   support_score |   overlap_relevant |   max_logit |   logit_var |   au_prime |   eu_prime | statement                                                                                                                                            |
|----------:|-------------:|-----------:|----------------:|-------------------:|------------:|------------:|-----------:|-----------:|:-----------------------------------------------------------------------------------------------------------------------------------------------------|
|         1 |          218 |        219 |           1.000 |              1.000 |      23.816 |       0.069 |      0.683 |      0.041 | - The ISR is activated upon Salubrinal treatment, as shown by increased CHOP expression, but it does not induce OA1 transgene reactivation (Fig 5B). |
|         2 |          241 |        243 |           1.000 |              1.000 |      23.421 |       0.000 |     -0.000 |      0.041 | - Treatment with ISRIB, which inhibits the activation of the ISR, does not affect the reactivation of the OA1 transgene (Fig 5C).                    |
|         3 |          244 |        244 |           1.000 |              1.000 |      22.500 |       0.212 |      0.682 |      0.043 | - Knockdown of ATF4 by siRNAs does not interfere with the reactivation of the OA1 transgene by L-Histidinol (Fig 5D).                                |
|         4 |          251 |        251 |           1.000 |              1.000 |      22.632 |       0.185 |      1.083 |      0.043 | - GCN2 knockout does not abolish or reduce the reactivation of the OA1 transgene in response to EAA starvation (Fig 6-8).                            |
|         4 |          267 |        267 |           1.000 |              1.000 |      23.684 |       0.524 |      0.682 |      0.042 | - GCN2 knockout does not abolish or reduce the reactivation of the OA1 transgene in response to EAA starvation (Fig 6-8).                            |
|         4 |          268 |        268 |           1.000 |              1.000 |      24.605 |       0.000 |     -0.000 |      0.039 | - GCN2 knockout does not abolish or reduce the reactivation of the OA1 transgene in response to EAA starvation (Fig 6-8).                            |


诊断：idx51 的主体证据链是对的，P=1.0、F1=0.909；真正的损失来自两个无引用的总结性 statement。第一句直接回答“ISR neither sufficient nor necessary”，Need Citation=Yes 却 `<cite></cite>`，因此 support_score=0。后面的 Salubrinal、ISRIB、ATF4、GCN2 分别引用具体实验句，构成“非充分 + 非必要”的组合证据。这里不是选错句，而是“全局结论句没有把后续分项证据回链”。


### idx210 / 小说人物

|   stmt_no |   span_start |   span_end |   sid_min |   sid_max |   support_score | out_of_range   |   max_logit |   logit_var |   au_prime |   eu_prime | statement           |
|----------:|-------------:|-----------:|----------:|----------:|----------------:|:---------------|------------:|------------:|-----------:|-----------:|:--------------------|
|         1 |            3 |          3 |         1 |        36 |           1.000 | False          |      24.079 |       0.000 |     -0.000 |      0.040 | 1. 邻家可人的大姐姐 |
|         2 |            3 |          3 |         1 |        36 |           1.000 | False          |      25.132 |       0.000 |     -0.000 |      0.038 | 2. 冷艳高贵的女总裁 |
|         3 |            3 |          3 |         1 |        36 |           1.000 | False          |      25.132 |       0.000 |     -0.000 |      0.038 | 3. 妩媚动人的交际花 |
|         4 |            3 |          3 |         1 |        36 |           1.000 | False          |      25.526 |       0.000 |     -0.000 |      0.038 | 4. 火爆热辣的女警花 |
|         5 |           47 |         47 |         1 |        36 |           0.000 | True           |      21.842 |       0.623 |      0.681 |      0.045 | 5. 美女钟倩倩       |
|         6 |           99 |         99 |         1 |        36 |           0.000 | True           |      22.105 |       0.000 |     -0.000 |      0.043 | 6. 妓女秦水瑶       |
|         7 |           66 |         66 |         1 |        36 |           0.000 | True           |      21.316 |       0.000 |     -0.000 |      0.045 | 7. 警官             |
|         8 |           69 |         69 |         1 |        36 |           0.000 | True           |      23.553 |       0.000 |     -0.000 |      0.041 | 8. 张星宇           |
|         9 |          156 |        156 |         1 |        36 |           0.000 | True           |      21.711 |       0.000 |     -0.000 |      0.044 | 9. 薛虎             |
|        10 |          196 |        196 |         1 |        36 |           0.000 | True           |      21.842 |       0.000 |     -0.000 |      0.044 | 10. 高熙            |
|        11 |          179 |        179 |         1 |        36 |           0.000 | True           |      23.026 |       0.000 |     -0.000 |      0.042 | 11. 蒋丽            |
|        12 |          163 |        163 |         1 |        36 |           0.000 | True           |      19.211 |       1.117 |      1.078 |      0.053 | 12. 钟              |
|        13 |          231 |        231 |         1 |        36 |           0.000 | True           |      23.289 |       0.000 |     -0.000 |      0.041 | 13. 南山南          |
|        14 |          287 |        287 |         1 |        36 |           0.000 | True           |      22.105 |       0.000 |     -0.000 |      0.043 | 14. 孙渺            |


诊断：idx210 是越界失败的集中样本。文档只有 sid 1–36，但模型从第 5 个条目开始生成了 47、99、66、69、156、196、179、163、231、287 等编号。前四个答案复用了 [3-3]，能覆盖参考答案中的四类人物，因此 precision=1.0，但 recall 只有 0.3125；后续新增人名既越界又不支持。根因不是语义检索失败，而是 citation index 约束没有被硬性执行，数字生成退化为“可见人名/常见编号”的自由续写。
