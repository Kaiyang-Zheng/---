# Banzhaf-Cite：基于单通道支持度图的 LongCite 实时引用修正方案
---

## 0. 方法目标

LongCite 已经能生成带 `<statement>...</statement>` 和 `<cite>[a-b]</cite>` 的回答，但引用可能存在三类问题：

| 类型 | 问题 | 修正目标 |
|---|---|---|
| A 类 | 多个候选句协同才能完整支持 statement | 找到最小多句证据联盟 |
| B 类 | 候选集中已有关键句，但模型选错句或边界错 | 找到最关键单句或少数句 |
| C 类 | 候选中仍无足够证据，或 statement overclaim | 不强行修，删除/改写/补检索 |

Banzhaf-Cite 

# QA 750 — statement-level precision × AU′ × mean_logit

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

