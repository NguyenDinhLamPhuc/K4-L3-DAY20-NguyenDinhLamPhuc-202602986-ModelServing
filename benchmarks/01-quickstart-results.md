# 01 - Measure: latency baseline

Model `Qwen3.5 0.8B` � host `Windows-AMD64` � llama.cpp `b10488`
Settings: `threads=14` `ngl=99` `ctx=2048`
`max_tokens=64` � warm-up discarded
Completed requests: `Q4_K_M` 10/10 � `UD-Q2_K_XL` 10/10

| Quantization | Size (GB) | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode (tok/s) |
|:--|--:|--:|--:|--:|--:|--:|
| Q4_K_M | 0.50 | 1854 | 186 / 202 | 5.9 / 6.2 | 552 / 585 / 585 | 168.8 |
| UD-Q2_K_XL | 0.39 | 1466 | 260 / 289 | 6.4 / 6.6 | 665 / 705 / 705 | 156.6 |

- **TTFT** = prefill. Short prompts keep it small; long-context RAG is where it explodes.
- **TPOT** = per-output-token decode cost, bounded by memory bandwidth. `decode tok/s = 1000 / TPOT_p50`.
- `UD-Q2_K_XL` decodes **1.08x SLOWER** than `Q4_K_M` here, despite being 0.11 GB smaller. That is a real result, not a mistake: fewer bits only buys speed when decode is limited by memory bandwidth. On a machine that is compute-limited instead � few cores, no GPU offload � the extra dequantization work of a heavily-quantized format can cost more than the bytes it saves. Say which case yours is.

## Your observation

The `UD-Q2_K_XL` model is smaller at 0.39 GB and loads faster at 1466 ms, compared with 0.50 GB and 1854 ms for `Q4_K_M`. However, the 4-bit model performs better during inference: its TTFT P50 is 186 ms versus 260 ms, and its decode speed is 168.8 tok/s versus 156.6 tok/s for the 2-bit model.

I also asked both servers the same question. The 4-bit response was generic and incomplete, but it at least discussed quantization trade-offs. The 2-bit response was also incomplete and less relevant because it discussed unrelated formats such as fixed-point, FP32/FP64, and semi-fixed precision instead of comparing `Q4_K_M` with `UD-Q2_K_XL`. Therefore, the answer-quality test did not show a clear advantage for the 2-bit model.

On my machine, the 2-bit quantization is worthwhile only when smaller model size and faster loading are more important. For normal use, I would choose `Q4_K_M` because it provides better latency, higher decode throughput, and more useful output.