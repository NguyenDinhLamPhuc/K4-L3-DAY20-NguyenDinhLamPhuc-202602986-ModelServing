# 01 - Tune: thread-count sweep

Model `Qwen3.5-0.8B-Q4_K_M.gguf` � host `Windows-AMD64` � llama.cpp `b10488`
CPU: **14 physical � 20 logical** cores � `ngl=99` � metric `tg128`

| threads (-t) | tg128 (tok/s) | vs best |
|:--|--:|--:|
| 1 | 177.3 | 100% |
| 7 | 178.0 | 100% |
| 14 | 177.7 | 100% |
| 20 | 177.3 | 100% |
| 40 | 177.5 | 100% |

**Best**: `-t 7` at 178.0 tok/s
**Slowest tested**: `-t 20` at 177.3 tok/s (1.00x spread)
**Against the physical-core default** (`-t 14`, 177.7 tok/s): 1.00x

Use this in your run:

```bash
LAB_N_THREADS=7 make bench
```

## Your explanation


The curve is effectively flat, so there is no clear thread-count knee in this sweep. The best result is `-t 7` at 178.0 tok/s, but it is only 1.00x the physical-core baseline of `-t 14` at 177.7 tok/s. The difference is about 0.2%, which is within normal benchmark noise rather than a meaningful speedup.

This is likely because `ngl=99` offloads the model to the NVIDIA GPU. For the `tg128` decode workload, the main bottleneck is therefore on the GPU, such as GPU memory bandwidth and dequantization, rather than CPU thread scheduling. Increasing CPU threads from 1 to 40 does not provide more GPU bandwidth or wider GPU parallelism, so the throughput remains around 177-178 tok/s. The small peak at 7 threads is not evidence of a real knee. In this configuration, changing GPU offload or quantization would likely have a larger effect than changing the CPU thread count.