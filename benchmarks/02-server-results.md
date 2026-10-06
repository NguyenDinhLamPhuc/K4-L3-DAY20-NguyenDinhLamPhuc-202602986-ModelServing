# 02 - Serve: load test + saturation reading

Host `Windows-AMD64` � llama.cpp `b10488` �
`--parallel 4` � `ctx=2048` � `threads=14` �
`ngl=99`

| Users | Reqs | RPS | P50 (ms) | P95 (ms) | P99 (ms) | Eff. concurrency | Failures |
|:--|--:|--:|--:|--:|--:|--:|--:|
| 10 | 232 | 3.97 | 1400 | 2800 | 4400 | 6.4 | 0.0% |
| 50 | 248 | 4.16 | 11000 | 12000 | 13000 | 41.8 | 0.0% |

*Effective concurrency = RPS x average latency (Little's Law) -- how many requests were
really in flight, regardless of how many users locust simulated. It counts queued requests
too, so the occupancy/slot ratio can legitimately exceed 1.0; it is occupancy, not
utilisation. For true slot utilisation use the server's own gauges (`make metrics`).*

## What these two runs say

| Going from 10 to 50 users | |
|:--|--:|
| Offered load | 5x |
| Throughput actually delivered | **1.05x** (21% of linear) |
| P95 latency | **4.29x** |
| Effective concurrency at 50 users | 41.8 vs `--parallel 4` slots (occupancy/slot ratio 10.46) |

**Saturated.** Throughput delivered only 1.05x for 5x the offered load, and effective concurrency (41.8) is at or above all 4 decode slots. Saturation sets in somewhere at or below 50 users; the load you added beyond that point became queue time rather than throughput.

Throughput moved 1.05x while P95 moved 4.29x. That gap is the goodput argument: past saturation you buy throughput by spending latency, and if your SLO is a P95 target then the requests you added are no longer being served within it. (This lab does not fix an SLO number for you -- pick one in your write-up and state how much goodput you keep at it.)

## Your reading

The server saturates between the 10-user and 50-user tests. The strongest
evidence is that increasing the offered load by 5x produced only 1.05x, while P95 latency increased by 4.29x.
The effective concurrency at 50 users was close to or above the
`--parallel 4` slot limit, matching the batching result of 3.87 busy slots out
of 4 and 45 deferred requests.

This indicates that additional load became queue time rather than useful
throughput. The first knob I would change is `--parallel`, increasing it
carefully if GPU memory and KV-cache capacity allow. This may let more requests
decode concurrently and raise goodput at the SLO. I would then rerun both the
load test and metrics check, because increasing parallelism can also increase
memory pressure and P95 latency. I would not increase CPU threads first because
the earlier thread sweep was effectively flat with GPU offload enabled.