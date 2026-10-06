# 03 - Integrate: RAG pipeline run

Host `Windows-AMD64` � llama.cpp `b10488` �
retrieval backend: **keyword overlap** � 3 queries

| Query | Contexts retrieved | embed (ms) | retrieve (ms) | llm (ms) | total (ms) |
|:--|--:|--:|--:|--:|--:|
| Why is goodput more useful than raw throughp... | goodput, paged, radix | 0.0 | 0.0 | 3447.0 | 3447.0 |
| What problem does PagedAttention actually so... | paged, radix, disagg | 0.0 | 0.0 | 2919.5 | 2919.5 |
| When does splitting prefill and decode help?... | disagg, radix, batching | 0.0 | 0.0 | 3622.1 | 3622.2 |

Mean per stage (ms): embed **0.0** � retrieve **0.0** �
llm **3329.5** � total **3329.6**
Dominant stage: **llm** (100% of total)

## Answers returned

**Why is goodput more useful than raw throughput?**

> Based on the provided context, **Goodput** is more useful than raw throughput because it focuses on **slops (Service Level Objectives)** rather than ignoring them.

While raw throughput measures the total requests per second (TPS) that pass through, Goodput specifically counts only the requests that met the **TTFT** (Total Time to First Failure) and **TPOT** (Total Time to Peak Occupancy) targets.

**What problem does PagedAttention actually solve?**

> PagedAttention solves the problem of **internal fragmentation in GPU memory** caused by storing key-value pairs (KV cache) in non-contiguous pages.

By organizing the cache into specific pages rather than a single contiguous block, it allows the GPU to utilize unused memory space more efficiently. This optimization is particularly beneficial for large-scale inference tasks where memory bandwidth i

**When does splitting prefill and decode help?**

> Based on the provided context, splitting prefill and decode helps during **continuous batching**.

The reasoning is as follows:
1.  **Prefill vs. Decode Characteristics**: The context states that prefill is compute-bound (requires significant processing power) and decode is memory-bandwidth-bound (requires significant data transfer).
2.  **The Benefit of Splitting**: When continuous batching allow


## Which N16-N19 pieces are real

- **N16 Cloud/IaC:** Stubbed. The pipeline runs locally against localhost; no
  Kubernetes cluster or Compose deployment is connected.
- **N17 Data pipelines:** Stubbed. The example documents are stored in an
  in-memory Python list rather than being produced by a real pipeline.
- **N18 Lakehouse:** Stubbed. `TOY_DOCS` is a toy dictionary and is not backed by
  Delta, Iceberg, or another lakehouse table.
- **N19 Vector + features:** Stubbed. Retrieval uses keyword overlap by default
  rather than a production vector index or Feast feature view.
- **N20 Serving:** Real. The pipeline sends requests to the running local
  llama-server through its OpenAI-compatible API.

The dominant stage was llm, taking approximately 100% of the total
latency. This was expected because generation and decode are substantially
slower than the toy keyword retrieval stage.

If I had to reduce end-to-end latency by 2x, I would attack the `llm` stage
first by reducing the output-token budget and retrieved context size, then
measure the effect on answer quality. I would also keep the 4-bit model and
GPU offload enabled because the earlier measurements showed that they provide
better decode speed on this machine. Optimizing the toy retrieval code would
not produce a meaningful improvement while it contributes only a small fraction
of total latency.