# 02 - Continuous batching under load (u50)

Host `Windows-AMD64` � `--parallel 4` � 15 samples over
60s at 2.0s intervals � raw CSV: `02-server-metrics-u50.csv`

| Gauge | Peak observed |
|:--|--:|
| `n_busy_slots_per_decode` (avg/decode) | 3.87 of 4 slots (97%) |
| `requests_processing` | 4 |
| `requests_deferred` | 45 |
| `kv_cache_usage_ratio` | n/a � not exported by llama.cpp `b10488` |
| `tokens_predicted_total` (final) | 30791 |

Highest sampled value was **3.87 of 4** slots. Note this gauge is llama.cpp's *average* busy slots per decode step, so the number below is the highest average we sampled, not an instantaneous maximum batch width. A peak near 1 means
requests were served one at a time -- either the load was too light to overlap, or
they arrived too far apart. A peak approaching `--parallel` means the scheduler was
genuinely packing concurrent requests into shared decode steps.
`requests_deferred` went above zero: more requests arrived than there were slots, so some waited. That wait is the queue time in your P95.

## Your observation

The peak `n_busy_slots_per_decode` was **3.87 of 4 slots**, or about **97%**
utilization. This is very close to the configured `--parallel 4` limit and shows
that llama.cpp was actively using continuous batching to decode several requests
at the same time.

The `requests_processing` gauge reached **4**, matching the effective
concurrency reported by the server results. `requests_deferred` peaked at
**45**, which means the 50-user load generated more concurrent work than the
four available slots could handle. Those extra requests had to wait in the
queue, contributing to the higher P95 latency under load.

The batching metrics are the most direct evidence of scheduler behavior because
they measure slot activity during the run. The load-test results complement
this by showing the user-visible latency and throughput impact.