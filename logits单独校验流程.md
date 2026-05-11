引文生成本质上是概率寻址，通过Teacher-forcing技术，可以回溯模型在生成每一个引文数字瞬间的心理状态（Logits/概率分布）。
先把评测用的长文档（按句子编号后的版本）和问题拼成模型在正式推理时看到的那一整段用户输入，并得到对应的 token序列；再拿模型当时已经生成好的整段回答文本，一直截到某个引用里句首编号第一个数字出现之前（相当于引用方括号已经写好、数字还没写出来），把这一截作为已生成的前缀直接接在用户输入后面，并在序列末尾读出下一个 token的打分，再从中挑出十个数字相关 token的原始得分，作为该引用位置处模型当时的数字偏好。
```mermaid
flowchart TD
    A["build_chat_input：只编码用户轮（含长文+问题）"] --> B["记用户段长度 p0"]
    B --> C{"遍历助手前缀的几种分词变体（如是否多一个换行）"}
    C --> D["把当前变体编成 token 序列 suf"]
    D --> E{"suf 非空？"}
    E -->|否| C
    E -->|是| F["full = 用户 input_ids + suf；attention 全 1"]
    F --> G["模型前向（推理模式、无梯度）"]
    G --> H["取位置 next_pos = p0 + len(suf) - 1（做边界裁剪）"]
    H --> I["读出该位置的下一步 logits（一维）"]
    I --> J{"是否已知真实下一个 token？"}
    J -->|否| K["直接采用当前变体并结束循环"]
    J -->|是| L["用真实下一 token 的 logit 打分，选最高的变体"]
    L --> C
    K --> M{"所有变体都不可用？"}
    M -->|是| N["报错 RuntimeError"]
    M -->|否| O["softmax；非数字 top-k；数字前缀；再取数字 token top-k"]
    O --> P["返回 logits、变体、对比指标、数字 top10 等"]
```

# Teacher-force 增量 KV：按 `digit_pos` 全局排序

同一篇 `prediction` 内，所有 cite 首数字位先按字符位置 `digit_pos` 升序遍历；在首轮锁定 assistant 的换行变体后，后续步尽量只把**相对上一步多出来的** assistant token 前向进模型，并复用 `past_key_values`。在**最后一个新 token** 处取 `logits[0, -1]` 作为「下一 token」分布。

```mermaid
flowchart TD
  A[收集 cite 首数字位目标] --> B[按 digit_pos 升序排序]
  B --> C[user 段 build_chat_input 一次]
  C --> D[frozen0 = pred 到首位点前]
  D --> E[各 assistant 变体: cat user + tokenize variant]
  E --> F[model 全序列 use_cache=True]
  F --> G[用 teacher 首 token id 选 logit 最大的变体]
  G --> H[锁定 variant_idx, past, prev_asst_ids]
  H --> I{下一位点}
  I --> J[frozen = pred 到当前位点前]
  J --> K[new_ids = tokenize 该变体下 frozen]
  K --> L{LCP 与 prev 比较}
  L -->|LCP 小于 len prev| M[全量 user+new_ids 重算 KV]
  L -->|否则| N{有新 token 后缀?}
  N -->|否| O[KV 与 logits 不变]
  N -->|是| P[只 forward 后缀 + past_key_values]
  M --> Q[logits 最后一位置]
  P --> Q
  O --> R[沿用 last logits]
  Q --> S[top-K 数字 + AU EU 等]
  R --> S
  S --> T{还有位点?}
  T -->|是| I
  T -->|否| U[按 stmt / span_index 排序输出]
```

要点：

- **单调 `digit_pos`** 保证 `frozen` 字符串只变长，多数步下 `new_ids` 是 `prev_asst_ids` 的**延长**，只需增量一段 `chunk`。
- **`LCP < len(prev)`**：新 token 序列无法接在旧 cache 后（常见于跨 token 边界的字符增长），必须 **user+assistant 全量** 重算 cache。
- **首轮变体**：与原先 `_next_token_logits_after_frozen` 一致，用 teacher 的 `target_next_token_id` 在变体间打分，避免 chat template 下「是否 leading `\n`」歧义。
