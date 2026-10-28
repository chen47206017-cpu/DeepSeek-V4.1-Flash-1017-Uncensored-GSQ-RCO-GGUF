---
license: mit
base_model:
- ISTA-DASLab/DeepSeek-V4.1-Flash-1017-GSQ-RCO-GGUF
base_model_relation: finetune
quantized_by: AtomicChat
pipeline_tag: text-generation
library_name: gguf
tags:
- atomic-chat
- deepseek
- deepseek41
- gguf
- gsq
- rco
- strata
- abliteration
- refusal-direction
---

# DeepSeek-V4.1-Flash-1017 Without Refusals · GSQ-RCO
<p style="margin-top: 0; margin-bottom: 0;">
  <em>ISTA-DASLab's GSQ-RCO quantizations of DeepSeek-V4.1-Flash-1017 (763B, 384 experts) with DeepSeek's refusal direction projected
  out of the dense residual writers — the same edit as <a href="https://huggingface.co/AtomicChat/DeepSeek-V4.1-Flash-1017-Uncensored-GGUF">AtomicChat/DeepSeek-V4.1-Flash-1017-Uncensored-GGUF</a>,
  measured on every rung.</em>
</p>

- These are [ISTA-DASLab/DeepSeek-V4.1-Flash-1017-GSQ-RCO-GGUF](https://huggingface.co/ISTA-DASLab/DeepSeek-V4.1-Flash-1017-GSQ-RCO-GGUF)'s GSQ-RCO backbone shards (revision `cb02422c900d`)
  with **only the residual-writing dense tensors replaced**: attention output (`attn_output_b`), the shared-expert
  down projection (`ffn_down_shexp`) and the Engram value projection (`engram_wkv`). The **routed experts are left
  exactly as ISTA shipped them**, so GSQ's learned low-bit rounding of the experts is preserved byte for byte, and
  each file stays the same size and runs wherever the original runs.
- The replaced tensors carry the one-direction edit (EN + RU, strength 2.0) baked into DeepSeek's weights at full
  precision and quantized to ISTA's exact per-tensor type.
- Runs on a **single 12 GB GPU** with [Strata](https://github.com/Niko1221/Strata): the backbone sits in system
  memory and the shared **Engram table streams from an SSD**, so a 763B model fits a consumer machine.

## What changed and what did not

Both files served on Strata, same prompts and template; "→" reads the stock GSQ-RCO rung → this one.

| Rung | bpw | Total GB | JailbreakBench (EN) | XSTest (safe refused) | Thinking high (30) | MMLU (500) | KL to stock |
|---|---|---|---|---|---|---|---|
| `IQ2_XXS` | 2.0 | 249 | 85.2% → **0.0%** | 7.6% → 0.0% | 27/30 → **0/30** | 82.8% (p=0.27) | — |
| `IQ1_M` | 1.75 | 231 | 88.9% → **0.0%** | 4.8% → 0.0% | 27/30 → **1/30** | 80.6% (p=0.75) | — |

KL is the edit's distribution shift from the stock rung, from Strata's teacher-forced score mode: a top-64 lower
bound, so the true KL is a little higher. Measurements, prompts and gates:
[AtomicChat/DeepSeek-V4.1-Flash-1017-Uncensored-GGUF-metrics](https://huggingface.co/datasets/AtomicChat/DeepSeek-V4.1-Flash-1017-Uncensored-GGUF-metrics) (`ablit/gsq/`).

## Running it

Download one rung and ISTA-DASLab's shared Engram file, then serve with Strata:

```bash
hf download AtomicChat/DeepSeek-V4.1-Flash-1017-Uncensored-GSQ-RCO-GGUF --include "*IQ2_XXS*" --local-dir ds-unc
hf download ISTA-DASLab/DeepSeek-V4.1-Flash-1017-GSQ-RCO-GGUF --include "*Engram*" --local-dir ds-unc   # the 110 GB lookup table, once
strata launch --model-path ds-unc/DeepSeek-V4.1-Flash-1017-GSQ-RCO-IQ2_XXS-00001-of-00004.gguf
```

The Engram file is identical to ISTA-DASLab's and is **not** re-hosted here; keep it on a fast SSD next to the shards.
As of this writing these files run in Strata (DeepSeek-V4.1 with a separate Engram file is not yet in llama.cpp).

## Limitations

- The refusal classifier reads the start of an answer, so a soft refusal later in the text can pass as an answer.
- The edit removes refusals across topics; it does not keep a narrower set of them.
- With thinking on, the edit plans an answer to a request the original declines, and the plan can be long; give such
  requests a larger token budget or a lower effort.

## Responsible use

This model answers requests the original declines. You are responsible for how you use it and for following the laws
that apply to you.

## Credits and license

- Base model: [deepseek-ai/DeepSeek-V4.1-Flash-1017](https://huggingface.co/deepseek-ai/DeepSeek-V4.1-Flash-1017) — MIT (its LICENSE ships here).
- GSQ-RCO quantization and the shared Engram file: [ISTA-DASLab/DeepSeek-V4.1-Flash-1017-GSQ-RCO-GGUF](https://huggingface.co/ISTA-DASLab/DeepSeek-V4.1-Flash-1017-GSQ-RCO-GGUF) (DASLab, IST Austria).
- Runtime: [Strata](https://github.com/Niko1221/Strata).

MIT, as the base model and ISTA-DASLab's GGUFs. Built by AtomicChat.
