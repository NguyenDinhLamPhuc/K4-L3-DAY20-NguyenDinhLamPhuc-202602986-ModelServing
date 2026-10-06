# Reflection — Day 20 Lab (Personal Report)

> **Đây là báo cáo cá nhân.** Số liệu của bạn **không** so sánh được với bạn cùng lớp
> — chỉ so **before vs after trên chính máy bạn**. Rubric chấm độ rõ ràng của setup,
> đo lường và **lập luận**, không chấm tốc độ tuyệt đối.
>
> `make verify` sẽ fail nếu còn placeholder chưa điền. Đó là cố ý.

**Họ Tên:** Nguyễn Đình Lâm Phúc
**MSSV:** 2A202602986
**Cohort:** A20-K4
**Ngày submit:** 2026-10-06

---

## 1. Hardware & runtime  *(rubric 1, 2 — 10 điểm)*

> Từ `make probe`. Paste output hoặc điền tay.

- **OS:** Windows 11
- **CPU:** 12th Gen Intel(R) Core(TM) i7-12700H
- **Cores:** 14 physical · 20 logical cores
- **CPU extensions:** 
- **RAM:** 15.6 GB
- **Accelerator:** NVIDIA GeForce RTX 3050 Laptop GPU, 4096 MiB, CUDA offload active
- **llama.cpp asset đã tải:** `llama-b10488-bin-win-cuda-12.4-x64.zip`
- **Model đã dùng:** Qwen3.5 0.8B (`LAB_MODEL=qwen35-0.8b`)
- **Quantization:** `Q4_K_M` + `UD-Q2_K_XL`

**Chạy ở đâu:** laptop của tôi

**Setup story** (≤ 80 chữ): Windows không có lệnh `make`, nên tôi dùng `.\lab.ps1` và Python trong virtual environment `.venv\Scripts\python.exe` để chạy các target của lab. Runtime CUDA prebuilt b10488 được tải và nhận diện GPU RTX 3050 thành công. Tôi chọn Qwen3.5 0.8B vì nhẹ hơn, dù máy có đủ RAM cho model lớn hơn.

## 2. Đo lường  *(rubric 3, 4, 5 — 20 điểm)*

> Paste bảng từ `benchmarks/01-quickstart-results.md` (`make bench` tự sinh).

| Quantization | Size (GB) | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode (tok/s) |
|:--|--:|--:|--:|--:|--:|--:|
| Q4_K_M | 0.50 | 1854 | 186 / 202 | 5.9 / 6.2 | 552 / 585 / 585 | 168.8 |
| UD-Q2_K_XL | 0.39 | 1466 | 260 / 289 | 6.4 / 6.6 | 665 / 705 / 705 | 156.6 |

**Quan sát** (≤ 60 chữ): 2-bit nhanh hơn bao nhiêu, và **có đáng không**? Bạn đã thử
hỏi cùng một câu trên cả hai (`make serve` vs `.venv/bin/python labs/02-serve/serve.py --compare`)
chưa? Chất lượng khác nhau thế nào?

UD-Q2_K_XL nhỏ hơn 0.11 GB và load nhanh hơn 20.9%, nhưng decode chậm hơn 7.2%, TTFT cao hơn 39.8% và E2E cao hơn 20.5%. Tôi đã hỏi cùng một câu trên cả hai server; câu trả lời 2-bit chung chung và lệch chủ đề hơn. Vì vậy, trên máy tôi, 4-bit đáng dùng hơn.
---

## 3. Serving under load  *(rubric 8, 9, 10 — 20 điểm)*

> Từ `benchmarks/02-server-results.md` (`make load-report`).

| Users | Reqs | RPS | P50 (ms) | P95 (ms) | P99 (ms) | Eff. concurrency | Failures |
|:--|--:|--:|--:|--:|--:|--:|--:|
| 10 | 232 | 3.97 | 1400 | 2800 | 4400 | 6.4 | 0.0% |
| 50 | 248 | 4.16 | 11000 | 12000 | 13000 | 41.8 | 0.0% |

- **Offered load tăng 5x, throughput thực tăng:** **1.05x**
- **P95 tăng:** **4.29x**
- **Effective concurrency ở 50 users:** 41.8 so với `--parallel` = 4 slots

**Peak `llamacpp:n_busy_slots_per_decode`** (từ `make metrics` khi `make load-50` đang
chạy): **3.87 / 4 slots**

**Saturation reading** (≤ 80 chữ): server của bạn bão hoà ở đâu, và **bằng chứng nào**
thuyết phục bạn? Nếu P95 tăng nhanh hơn RPS thì phần latency thêm đó là queue time hay
compute time — bạn biết bằng cách nào? Nếu bạn phải nâng goodput@SLO, bạn sẽ đổi knob
nào **trước**, và vì sao knob đó?

_Server bão hòa ở mức từ 10 đến 50 users. Offered load tăng 5× nhưng throughput chỉ tăng 1.05×, trong khi P95 tăng 4.29×. Effective concurrency đạt 41.8 so với 4 slots, cùng với 3.87/4 busy slots và 45 request deferred, chứng minh phần latency tăng chủ yếu là queue time. Tôi sẽ tăng `--parallel` trước, nếu GPU memory và KV cache còn đủ.._

---

## 4. Integration  *(rubric 12, 13 — 15 điểm)*

> Từ `make pipeline`. Nói thật cái nào real, cái nào stub — stub **không** mất điểm.

| Day | Piece | Real hay stub? |
| N16 Cloud/IaC | stub |
| N17 Data pipeline | stub |
| N18 Lakehouse | stub |
| N19 Vector + features | stub |
| N20 Serving | `llama-server` | real |

**Latency split** (mean của 3 query, từ output của `pipeline.py`):

- embed: **0.0 ms**
- retrieve: **0.0 ms**
- llm: **3329.5 ms**
- **stage chiếm nhiều nhất:** **llm** (**100%** của total)


**Reflection** (≤ 60 chữ): bottleneck ở đâu? Có khớp với kỳ vọng của bạn không? Nếu
phải giảm latency của pipeline này 2×, bạn sẽ tấn công vào đâu?

_Pipeline hiện dùng dữ liệu toy, keyword overlap và không có embedding thật, nên N16-N19 đều là stub. LLM chiếm 100% latency, đúng như kỳ vọng vì generation đắt hơn retrieval. Nếu cần giảm latency 2 lần, tôi sẽ giảm output-token budget và context size, rồi đo lại chất lượng._

---

## 5. The single change that mattered most  *(rubric 11 — 10 điểm)*

> **Phần quan trọng nhất của report.** Không cần bonus track: `make tune` đã cho bạn
> một before/after thật (`benchmarks/01-tuning-tg128.md`). Đổi quantization,
> `LAB_N_CTX`, hay `--parallel` rồi đo lại cũng được.

**Change:** _giảm số thread từ `-t 14` xuống `-t 7`_

```
before: 177.7 tok/s
after: 178.0 tok/s
speedup: 1.00×
```

**Tại sao nó work** (1–2 đoạn — đây là phần grader đọc kỹ nhất):

_Giải thích như đang nói với bạn ngồi cạnh. Bám vào **cơ chế**, không phải "vibes":
memory bandwidth? vector width? cache residency? scheduling? queueing? Nếu kết quả
**khác** với kỳ vọng từ deck — nói rõ, và giải thích vì sao. Grader thưởng điểm cho
lập luận đúng về một kết quả bất ngờ, hơn là một con số đẹp không được giải thích._

_Kết quả này không tạo ra speedup đáng kể: chỉ khoảng 0.2%. Curve gần như phẳng từ 1 đến 40 threads vì `ngl=99` đã offload model lên GPU. Decode bị giới hạn chủ yếu bởi GPU memory bandwidth và dequantization, không phải CPU scheduling. Vì vậy, thêm CPU threads không làm tăng năng lực xử lý của GPU; đỉnh nhỏ ở 7 threads có thể chỉ là benchmark noise.._

---

## 6. Bonus  *(optional — tối đa 10 điểm)*

> Bỏ trống nếu không làm. Xem `docs/bonus/README.md`. Đừng làm hết — **một** finding sâu
> ăn điểm hơn năm bảng nông.

**Đã làm:** _<B1 build-compare / B2 sweep nào / B4 challenge nào / B5 lựa chọn nào>_

**Numbers:**

```
before:  <số>
after:   <số>
speedup: <X.Y>×
```

**Điều này nói lên gì mà deck chưa nói:**

_(để trống nếu bạn không làm phần này)_

---

## 7. Điều làm bạn ngạc nhiên nhất  *(optional)*

_(1–2 câu. Không bắt buộc, nhưng grader đọc hết.)_

_(để trống nếu bạn không làm phần này)_

---

## 8. Self-check trước khi push

- [x] `hardware.json` committed
- [x] `models/active.json` committed
- [x] `benchmarks/01-quickstart-results.md` committed (`make bench`)
- [x] `benchmarks/01-tuning-tg128.md` committed (`make tune`)
- [x] `benchmarks/02-server-results.md` committed (`make load-report`)
- [x] `benchmarks/02-server-batching-u50.md` hoặc `-metrics-u50.csv` committed (`make metrics`)
- [x] `benchmarks/locust-10_stats.csv` + `locust-50_stats.csv` committed (`make load-10` / `load-50`)
- [x] `benchmarks/03-integration-results.md` committed (`make pipeline`)
- [x] Mọi section **"required — replace this line"** trong các file `benchmarks/*.md`
      đã được thay bằng nhận xét của bạn
- [x] 5 screenshots trong `submission/screenshots/`
- [x] `make verify` → **exit 0**
- [x] Repo tên đúng mẫu `K4-L3-DAY20-HoVaTen-MSSV-ModelServing` (xem `docs/SUBMISSION.md`)
- [x] Repo GitHub ở chế độ **public**
- [x] Đã push và paste public URL vào VinUni LMS **trước 23:59 (UTC+7) ngày làm lab**
- [x] **Không** commit `models/*.gguf`, `runtime/` hay `.env` (đã có trong `.gitignore`)

**Quan trọng:** repo phải **public** đến khi điểm được công bố. Private → grader không
xem được → 0 điểm.

---

## 9. Khai báo sử dụng AI  *(xem `docs/RULES.md` §3)*

_(Tôi đã sử dụng GitHub Copilot để giải thích lỗi, hướng dẫn lệnh tương đương trên
Windows. Toàn bộ lệnh, số liệu, kết quả benchmark và screenshot đều được chạy và kiểm tra trên laptop của tôi; tôi không dùng AI để tạo hoặc bịa số liệu.)_
