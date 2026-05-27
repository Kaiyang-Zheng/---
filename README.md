
# QA remaining 750 — 四数据集 × 训练/测试 × Base / Prelim LoRA / FA-φ e1

**评测**：GPT `gpt-4o`，macro citation F1（doc 级平均）  
**推理**：纯 LoRA 一次性 full-gen（temperature=0.95, top_p=0.7）

## 划分说明

| 标签 | 含义 |
|------|------|
| **训练 (prelim)** | `bi_role_cohort.jsonl` 中出现过的 doc（**347** 唯一 doc） |
| **训练 (FA-φ)** | `phi_anchor_cohort.jsonl` 中出现过的 doc（**472** 唯一 doc） |
| **测试** | 相对对应训练池未出现的 doc |

下表「训练/测试」列默认按 **prelim 池**；FA-φ e1 模型另注 phi 池规模。

## 750 总览（prelim 训练池划分）

| 划分 | n | Base F1 | Prelim LoRA (bi e3) F1 | FA-φ e1 (holdout~0.77) F1 |
|------|---|---:|---:|---:|
| 全集 | 750 | **0.709** | **0.7459** | **0.7457** |
| 训练 (prelim) | 100 | **0.5751** | **0.6772** | **0.6636** |
| 测试 (prelim) | 650 | **0.7296** | **0.7564** | **0.7583** |

### FA-φ e1 按 phi_anchor 训练池

| 划分 | n | FA-φ e1 F1 |
|------|--:|-----:|
| 全集 | 750 | 0.7457 |
| 训练 (phi) | 214 | 0.6997 |
| 测试 (phi) | 536 | 0.7641 |

## 分数据集（全集 / prelim 训练 / prelim 测试）

### hotpotqa（n=200，prelim 训练 27 / 测试 173）

| 划分 | n | Base F1 | Prelim LoRA (bi e3) F1 | FA-φ e1 (holdout~0.77) F1 |
|------|---|-----:|-----:|-----:|
| 全集 | 200 | 0.5828 | 0.6203 | 0.6267 |
| 训练 | 27 | 0.3793 | 0.5078 | 0.4865 |
| 测试 | 173 | 0.6145 | 0.6379 | 0.6485 |

### dureader（n=200，prelim 训练 36 / 测试 164）

| 划分 | n | Base F1 | Prelim LoRA (bi e3) F1 | FA-φ e1 (holdout~0.77) F1 |
|------|---|-----:|-----:|-----:|
| 全集 | 200 | 0.7498 | 0.7813 | 0.771 |
| 训练 | 36 | 0.6945 | 0.7687 | 0.741 |
| 测试 | 164 | 0.7619 | 0.7841 | 0.7776 |

### multifieldqa_en（n=150，prelim 训练 25 / 测试 125）

| 划分 | n | Base F1 | Prelim LoRA (bi e3) F1 | FA-φ e1 (holdout~0.77) F1 |
|------|---|-----:|-----:|-----:|
| 全集 | 150 | 0.7281 | 0.7411 | 0.769 |
| 训练 | 25 | 0.5784 | 0.6673 | 0.6781 |
| 测试 | 125 | 0.7581 | 0.7559 | 0.7872 |

### multifieldqa_zh（n=200，prelim 训练 12 / 测试 188）

| 划分 | n | Base F1 | Prelim LoRA (bi e3) F1 | FA-φ e1 (holdout~0.77) F1 |
|------|---|-----:|-----:|-----:|
| 全集 | 200 | 0.7802 | 0.8395 | 0.8219 |
| 训练 | 12 | 0.6502 | 0.8047 | 0.7994 |
| 测试 | 188 | 0.7885 | 0.8417 | 0.8234 |

## 产物

- `cite_scores_longcite_base_full_only_qa_remaining.json` — Base
- `cite_scores_longcite_lora_full_only_qa_remaining.json` — Prelim LoRA (bi e3)
- `cite_scores_fa_phi_e1_full_only_qa_remaining.json` — FA-φ e1 (holdout~0.77)



# QA 750经过Banzhaf指导的Lora微调后 — statement-level precision × AU′ × mean_logit

Each cell: **mean GPT relevant precision** over statements with ≥1 cite span; AU/logit from teacher-force on generated cite (T=0.95, top-p=0.7).

**Fixed bins**: tertiles from full cohort (1119, no squality), same as baseline figure.

### Prelim LoRA (bi e3) — fixed cohort bins

| AU_bin | logit_low | logit_mid | logit_high |
|--------|-----------|-----------|------------|
| AU_zero | 0.830 (n=44) | 0.863 (n=155) | 0.951 (n=2326) |
| AU_nonzero_low | 0.667 (n=3) | 0.814 (n=17) | 0.869 (n=66) |
| AU_nonzero_mid | 0.739 (n=58) | 0.768 (n=64) | 0.896 (n=74) |
| AU_nonzero_high | 0.393 (n=14) | 0.500 (n=2) | — (0) |

### Prelim LoRA (bi e3) — QA750 recomputed tertiles

| AU_bin | logit_low | logit_mid | logit_high |
|--------|-----------|-----------|------------|
| AU_zero | 0.909 (n=724) | 0.951 (n=889) | 0.963 (n=912) |
| AU_nonzero_low | 0.808 (n=86) | 0.962 (n=13) | 1.000 (n=1) |
| AU_nonzero_mid | 0.753 (n=99) | — (0) | — (0) |
| AU_nonzero_high | 0.799 (n=82) | 0.857 (n=14) | 1.000 (n=3) |

- n_statements=2823, mean_precision=0.9283


### FA-φ e1 — fixed cohort bins

| AU_bin | logit_low | logit_mid | logit_high |
|--------|-----------|-----------|------------|
| AU_zero | 0.924 (n=251) | 0.927 (n=967) | 0.938 (n=938) |
| AU_nonzero_low | 0.927 (n=32) | 0.894 (n=63) | 1.000 (n=13) |
| AU_nonzero_mid | 0.868 (n=345) | 0.923 (n=93) | 0.900 (n=10) |
| AU_nonzero_high | 0.758 (n=266) | 0.583 (n=4) | — (0) |

### FA-φ e1 — QA750 recomputed tertiles

| AU_bin | logit_low | logit_mid | logit_high |
|--------|-----------|-----------|------------|
| AU_zero | 0.923 (n=326) | 0.928 (n=892) | 0.938 (n=938) |
| AU_nonzero_low | 0.854 (n=191) | 0.876 (n=71) | 1.000 (n=14) |
| AU_nonzero_mid | 0.901 (n=207) | 0.932 (n=59) | 0.889 (n=9) |
| AU_nonzero_high | 0.759 (n=273) | 0.500 (n=2) | — (0) |

- n_statements=2982, mean_precision=0.9071

