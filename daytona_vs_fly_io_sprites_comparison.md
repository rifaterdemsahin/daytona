# ⚔️ Daytona vs. Fly.io Sprites: The Ultimate Comparison

Both **Daytona** and **Fly.io Sprites** are modern sandbox infrastructure platforms built for AI agents, code execution, and cloud development environments. However, they operate on fundamentally different philosophies:
* **Daytona** is built as an **ephemeral, high-velocity execution engine** (spin up in milliseconds, run tasks, tear down or archive).
* **Fly.io Sprites** is built as a **persistent, stateful Linux computer for agents** (stays alive with its own filesystem and URL, sleeps automatically when idle, and wakes up on demand).

---

## 📊 Head-to-Head Feature Comparison

| Feature / Category | ⚡ Daytona | 🧚 Fly.io Sprites |
| :--- | :--- | :--- |
| **Core Philosophy** | Ephemeral, high-speed task & code runner | Persistent, stateful "computer" for long-lived agents |
| **Isolation Architecture** | 🐳 Docker containers by default (VM sandboxes available as separate tier) | 🔥 Hardware-isolated **Firecracker microVMs** by default (dedicated guest kernel) |
| **Cold / Wake Latency** | ⚡ **Sub-90 ms** (~60 ms) from warm pools | ⏱️ **100–500 ms** (warm wake) / **1–2 seconds** (cold start) |
| **Idle & Lifecycle Model** | ⏳ Auto-stops after 15 mins (configurable) $\rightarrow$ Auto-archives to object storage after 7 days | 💤 Auto-suspends to **Warm** state after ~30s of inactivity (RAM preserved), then **Cold** (disk preserved) |
| **Billing Model** | 📏 **Allocated resources** (billed per second for reserved vCPU & RAM while running) | 🎯 **Actual usage** (billed only when active; measured by actual CPU cycles & resident RAM) |
| **Storage & Persistence** | 💾 Ephemeral by default; filesystem snapshots; VM tier supports memory snapshots & forks | 🗄️ **100 GB persistent ext4 disk** per Sprite; instant copy-on-write filesystem checkpoints (~1s) |
| **GPU Support** | 🎮 **Yes** (RTX 4090, H100, H200, B200, B300, AMD MI355X—on-demand & preemptible) | 🚫 **No** (CPU-only; GPU workloads require separate Fly Machines) |
| **OS Support** | 🐧 Linux & 🪟 Windows sandboxes | 🐧 Full Linux VM environment only |
| **Networking & URLs** | 🌐 Outbound internet + per-sandbox firewall (block all, CIDR allowlist, domain allowlist) | 🔗 **Public HTTPS URL with auto-TLS** per Sprite; inbound HTTP requests automatically wake sleeping Sprites |
| **Secrets & Auth** | 🔑 Environment variables & standard SDK secret injection | 🛡️ **Built-in Connectors / Identity Brokering** (credentials never enter the VM; gateway injects auth) |
| **Process Management** | 🛠️ Standard shell/process execution (no native process supervisor documented) | 🔄 **Built-in service supervisor** with dependency ordering, auto-restart backoff, and boot relaunch |
| **Docker-in-Docker / Nested** | 📦 Native OCI/Docker image ecosystem | 🐳 Full guest kernel allows running Docker/containers inside the microVM |
| **MCP (Model Context Protocol)** | 🤖 Supported via SDK/integrations | 🔌 Native MCP server with OAuth consent screens & dynamic tool generation |
| **Concurrency & Scale** | 🚀 Pool-based (up to 500 vCPUs / 600 creations/min on Tier 4; bare-metal burst scheduler) | 📈 Tier-based (20 active on Adventurer, 100 active on Hero; unlimited cold Sprites) |

---

## 💰 Pricing Breakdown

### ⚡ Daytona Pricing
Daytona uses a **Pay-As-You-Go** model billed **per second** based on the **allocated** size of the sandbox while it is running (stopped/archived sandboxes only pay for storage).
* **Free Trial:** **$200** in free compute credits (no credit card required).
* **Compute (vCPU):** `$0.0504 / vCPU-hour` (`$0.000014 / vCPU-second`)
* **Memory (RAM):** `$0.0162 / GiB-hour` (`$0.0000045 / GiB-second`)
* **Storage:** `$0.000108 / GiB-hour` (~`$0.079 / GiB-month`) — *First 5 GiB free*
* **Windows OS:** `$0.0858 / vCPU-hour`
* **GPUs (On-Demand / Preemptible):** Ranging from RTX 4090 up to Nvidia H100 (`~$2.27/hr` preemptible), H200 (`~$2.61/hr`), B200 (`~$3.59/hr`), and B300 (`~$4.08/hr` preemptible / `$6.25/hr` on-demand).
* **Tiers (Quota Limits):**
  * **Tier 1 (Email verified):** 10 vCPU / 20 GiB RAM / 30 GiB Storage pool
  * **Tier 2 ($25 top-up):** 100 vCPU / 200 GiB RAM / 300 GiB Storage pool
  * **Tier 3 ($500 top-up):** 250 vCPU / 500 GiB RAM / 2,000 GiB Storage pool
  * **Tier 4 ($2,000/mo top-up):** 500 vCPU / 1,000 GiB RAM / 5,000 GiB Storage pool

---

### 🧚 Fly.io Sprites Pricing
Fly.io Sprites uses **actual-utilization metering**. When a Sprite goes idle (~30 seconds without HTTP/TCP/stdout activity), it enters a **Warm** state and **compute billing drops to $0**.
* **Free Trial:** `$1`–`$30` trial credits depending on onboarding flow (invoices under $5/mo are waived).
* **CPU Time (Measured via `cpu.stat`):** `$0.0385 / CPU-hour`
* **Memory Time (Actual resident RAM):** `$0.021875 / GB-hour`
* **Hot Storage (while awake):** `$0.000683 / GB-hour` (~`$0.50 / GB-month`)
* **Cold Storage (while asleep/paused):** `$0.000027 / GB-hour` (~`$0.02 / GB-month`)
* **Network Egress:** Free / unmetered for Sprites.
* **Subscription Plans (Bundled Allowances + Concurrency):**
  * **Recruit (Free / Pay-as-you-go):** Low concurrency (10 creations/min), pay standard rates.
  * **Adventurer ($20/mo):** Includes **450 CPU-hrs**, **1,800 RAM GB-hrs**, **50 GB storage**; up to **20 concurrent active** & **20 warm** Sprites (unlimited cold). Overages billed at standard rates.
  * **Hero ($100/mo):** Includes **1,200 CPU-hrs**, **4,800 RAM GB-hrs**, **150 GB storage**; up to **100 concurrent active** & **100 warm** Sprites.
  * **Legend ($200/mo) & Mythic ($2,000/mo):** Higher concurrency, faster creation rates (up to 240/min), and priority support.

> 💡 **Cost Comparison Insight:** Because AI agents spend ~70–80% of their session waiting on LLM API responses, **Fly.io Sprites is significantly cheaper for interactive agent sessions** (often 50–70% less) since you aren't paying for idle CPU/RAM while waiting on tokens. Conversely, at **100% continuous CPU load**, both platforms cost nearly the exact same (`~$83` per 500 sandbox-hours at 2 vCPU / 4 GB).

---

## 🎯 Where to Use Which One?

### Choose ⚡ Daytona when:
* 🎮 **Your agents need GPUs:** You are running ML training, fine-tuning, local inference, or CUDA workloads inside the sandbox.
* ⏱️ **Sub-100ms cold starts are critical:** You are building high-frequency code interpreters, RL (Reinforcement Learning) rollout evaluators, or CI/CD runners that spin up thousands of short-lived tasks per minute.
* 🐳 **You rely on custom Docker images & Windows:** You already have complex OCI/Docker registries you want to boot directly, or your workflow requires Windows environments.
* 💥 **Massive burst parallelism:** You need to spin up hundreds of concurrent environments simultaneously (up to 600 creations/min) without per-sandbox concurrency caps.

### Choose 🧚 Fly.io Sprites when:
* 🧑‍💻 **Your agent "owns" a project or repository:** Ideal for coding agents (like Claude Code) that need a persistent 100 GB Linux workstation with dependencies, git history, and databases intact across days or weeks.
* 🌐 **You need webhooks, OAuth callbacks, or preview URLs:** Every Sprite gets a native HTTPS URL that automatically wakes the VM from sleep when a webhook or user hits it.
* 🔒 **You run untrusted code & want zero-trust security:** Every Sprite runs inside a hardware-isolated Firecracker microVM by default, and **Connectors** keep OAuth tokens/API keys outside the sandbox environment completely.
* 💸 **Your workload is bursty or LLM-bound:** You want the sandbox to automatically pause billing 30 seconds after the agent finishes a step, without losing in-memory process state.
* 🚀 **Seamless path to production:** Because Sprites run on the same Firecracker substrate as **Fly Machines**, an app built by an agent inside a Sprite can be promoted to a 24/7 production Fly Machine with zero rewrites.

---

## 🏆 Summary Verdict

* **Pick Daytona** for **stateless speed, GPU compute, and massive horizontal task execution**.
* **Pick Fly.io Sprites** for **stateful persistence, microVM security, wake-on-HTTP workflows, and cost-efficient interactive agents**.