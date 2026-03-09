---
title: "Native OpenMetrics in Slurm 25.11: Monitor Your Cluster with Prometheus Without a Separate Exporter"
date: 2026-03-06T12:00:00+09:00
draft: true
featured: false
authors:
  - admin
tags:
  - slurm
  - prometheus
  - grafana
  - monitoring
categories:
  - HPC
---

## TL;DR

Starting with Slurm 25.11, slurmctld directly exposes metrics in OpenMetrics format (the Prometheus metric standard). You no longer need to install a separate binary like `prometheus-slurm-exporter` that parses squeue/sinfo output. Just add one line to `slurm.conf` and write a Prometheus scrape config.

In this post, I share how I set up this feature on a real multi-GPU cluster. I also published the Grafana dashboard I built to the [marketplace](https://grafana.com/grafana/dashboards/24979-slurm-native-openmetrics/).

---

## The Problem with the Old Approach

Until now, adding Prometheus monitoring to a Slurm cluster required this architecture:

```
slurmctld ← squeue/sinfo/sdiag ← prometheus-slurm-exporter(:9341) ← Prometheus ← Grafana
```

A separate exporter binary like [vpenso/prometheus-slurm-exporter](https://github.com/vpenso/prometheus-slurm-exporter) would periodically run CLI commands such as `squeue`, `sinfo`, and `sdiag`, parse their output, and convert it into Prometheus metrics. It worked, but came with several pain points:

- **Managing an extra binary**: You had to build or download the exporter separately and register it as a systemd service.
- **CLI parsing fragility**: If the squeue output format changed, the exporter could break.
- **Metric accuracy**: Because the exporter periodically calls CLI commands, there is a time lag between the exporter and slurmctld.
- **Maintenance burden**: Every time Slurm was upgraded, you had to verify exporter compatibility.

In practice, the canonical exporter [prometheus-slurm-exporter](https://github.com/vpenso/prometheus-slurm-exporter) went unmaintained for a long time, leading to a proliferation of forks such as [rivosinc/prometheus-slurm-exporter](https://github.com/rivosinc/prometheus-slurm-exporter) and [SckyzO/slurm_exporter](https://github.com/SckyzO/slurm_exporter).

## What Changed in 25.11

Starting with Slurm 25.11, slurmctld exposes metrics conforming to the OpenMetrics 1.0 spec directly via an HTTP endpoint:

```
slurmctld(:6817) /metrics/* ← Prometheus ← Grafana
```

The exporter is gone. slurmctld exposes its own internal state directly. Because it exports internal data rather than parsing CLI output, the metrics are accurate and fast.

With official native metrics now available, the need for community exporters will naturally diminish. ([Slurm 25.11 Release Notes](https://github.com/SchedMD/slurm/releases/tag/slurm-25-11-0-0rc1), [Metrics Guide](https://slurm.schedmd.com/metrics.html))

## Configuration

### 1. slurm.conf

Only one line needs to be added:

```ini
MetricsType=metrics/openmetrics
```

Restart slurmctld after the change:

```bash
systemctl restart slurmctld
```

slurmctld will now serve metrics over HTTP on its default port (6817).

### Caveats

- **Metrics are disabled if `PrivateData` is set.** If your `slurm.conf` contains a `PrivateData` parameter, the metrics feature will not work.
- **There is no authentication.** Anyone who can send an HTTP request to the slurmctld port can access the metrics, so firewall or network access controls are required.
- **Be mindful of scrape frequency.** Metric requests acquire an internal lock inside slurmctld, so scraping too frequently can impact scheduler performance. The official documentation recommends a 60–120 second interval ([Metrics Guide](https://slurm.schedmd.com/metrics.html)).

### 2. Verifying the Endpoints

Once configured, you can verify immediately with curl:

```bash
# List available endpoints
curl http://localhost:6817/metrics

# Job status
curl http://localhost:6817/metrics/jobs

# Node resources
curl http://localhost:6817/metrics/nodes

# Partition status
curl http://localhost:6817/metrics/partitions

# Scheduler performance
curl http://localhost:6817/metrics/scheduler

# Per-user/account jobs
curl http://localhost:6817/metrics/jobs-users-accts
```

Example response (`/metrics/nodes`):

```
# HELP slurm_node_cpus Total number of cpus in the node
# TYPE slurm_node_cpus gauge
slurm_node_cpus{node="gpu-node-01"} 64
slurm_node_cpus{node="gpu-node-02"} 128
slurm_node_cpus{node="cpu-node-01"} 48
# HELP slurm_node_cpus_alloc Allocated cpus in the node
# TYPE slurm_node_cpus_alloc gauge
slurm_node_cpus_alloc{node="gpu-node-01"} 0
slurm_node_cpus_alloc{node="gpu-node-02"} 32
# EOF
```

### 3. Health Check Endpoints (Bonus)

25.11 also adds HTTP health check endpoints in addition to metrics:

```bash
# slurmctld
curl http://localhost:6817/livez    # Process liveness check
curl http://localhost:6817/readyz   # Ready to handle requests
curl http://localhost:6817/healthz  # Overall health status

# slurmd (port 6818 on each node)
curl http://<node-ip>:6818/livez
```

Combined with the Blackbox Exporter, you can monitor the status of Slurm daemons directly from Prometheus.

### 4. Prometheus Scrape Config

Add a scrape job for each endpoint in `prometheus.yml`. Separating by endpoint lets you tune the interval and timeout independently as needed:

```yaml
scrape_configs:
  # Job status (running, pending, completed, etc.)
  - job_name: "slurm-native-jobs"
    scrape_interval: 60s
    metrics_path: "/metrics/jobs"
    static_configs:
      - targets: ["<slurmctld-host>:6817"]

  # Node resources (CPU, memory allocation)
  - job_name: "slurm-native-nodes"
    scrape_interval: 60s
    metrics_path: "/metrics/nodes"
    static_configs:
      - targets: ["<slurmctld-host>:6817"]

  # Per-partition status
  - job_name: "slurm-native-partitions"
    scrape_interval: 60s
    metrics_path: "/metrics/partitions"
    static_configs:
      - targets: ["<slurmctld-host>:6817"]

  # Scheduler performance (backfill, cycle time, etc.)
  - job_name: "slurm-native-scheduler"
    scrape_interval: 60s
    metrics_path: "/metrics/scheduler"
    static_configs:
      - targets: ["<slurmctld-host>:6817"]

  # Per-user/account jobs
  - job_name: "slurm-native-users-accts"
    scrape_interval: 60s
    metrics_path: "/metrics/jobs-users-accts"
    static_configs:
      - targets: ["<slurmctld-host>:6817"]
```

> **Note**: The `/metrics/jobs-users-accts` endpoint generates time series proportional to the number of users. On clusters with many users, use a generous scrape interval.

Reload the Prometheus configuration:

```bash
# Method 1: HTTP API
curl -X POST http://localhost:9090/-/reload

# Method 2: Signal
kill -HUP $(pidof prometheus)
```

Verify that all targets show `UP` status in the Prometheus UI (`http://localhost:9090/targets`).

## Complete Metrics Reference

### /metrics/jobs — Cluster-wide Job Status

| Metric | Description |
|---|---|
| `slurm_jobs_running` | Number of running jobs |
| `slurm_jobs_pending` | Number of pending jobs |
| `slurm_jobs_completed` | Number of completed jobs |
| `slurm_jobs_failed` | Number of failed jobs |
| `slurm_jobs_cpus_alloc` | Total allocated CPUs |
| `slurm_jobs_memory_alloc` | Total allocated memory |
| `slurm_jobs_nodes_alloc` | Total allocated nodes |
| `slurm_jobs_timeout` | Number of timed-out jobs |
| `slurm_jobs_outofmemory` | Number of jobs failed due to OOM |

Beyond these, all Slurm job states are exposed as metrics, including `cancelled`, `completing`, `configuring`, `suspended`, and `preempted`.

### /metrics/nodes — Per-node Resources

| Metric | Label | Description |
|---|---|---|
| `slurm_node_cpus{node="..."}` | node | Total node CPUs |
| `slurm_node_cpus_alloc{node="..."}` | node | Allocated CPUs |
| `slurm_node_cpus_idle{node="..."}` | node | Idle CPUs |
| `slurm_node_memory_bytes{node="..."}` | node | Total memory |
| `slurm_node_memory_alloc_bytes{node="..."}` | node | Allocated memory |
| `slurm_node_memory_free_bytes{node="..."}` | node | Free memory |
| `slurm_nodes_idle` | — | Number of idle nodes |
| `slurm_nodes_mixed` | — | Number of mixed nodes |
| `slurm_nodes_alloc` | — | Number of allocated nodes |
| `slurm_nodes_down` | — | Number of down nodes |
| `slurm_nodes_drain` | — | Number of draining nodes |

### /metrics/partitions — Per-partition Status

Node and job metrics are provided at partition granularity with a `{partition="..."}` label. For example:

- `slurm_partition_jobs_running{partition="batch"}` — running jobs in the batch partition
- `slurm_partition_nodes_cpus_alloc{partition="batch"}` — allocated CPUs in the batch partition
- `slurm_partition_nodes_mem_alloc{partition="batch"}` — allocated memory in the batch partition

### /metrics/scheduler — Internal Scheduler Performance

| Metric | Description |
|---|---|
| `slurm_sched_mean_cycle` | Main scheduler average cycle time (µs) |
| `slurm_bf_mean_cycle` | Backfill scheduler average cycle time (µs) |
| `slurm_bf_cycle_last` | Last backfill cycle time |
| `slurm_bf_depth_mean` | Average backfill search depth |
| `slurm_bf_queue_len` | Backfill queue length |
| `slurm_schedule_queue_len` | Scheduler queue length |
| `slurm_slurmdbd_queue_size` | slurmdbd queue size |
| `slurm_backfilled_jobs` | Total backfilled jobs (cumulative) |
| `slurm_sdiag_latency` | RPC response latency |
| `slurm_server_thread_cnt` | Active slurmctld thread count |

### /metrics/jobs-users-accts — Per-user/Account

Job metrics broken down by user (`{user="..."}`) and account (`{account="..."}`):

- `slurm_user_jobs_running{user="alice"}` — alice's running jobs
- `slurm_user_jobs_cpus_alloc{user="alice"}` — CPUs in use by alice
- `slurm_account_jobs_pending{account="default"}` — pending jobs for the default account

This endpoint is useful for tracking per-user resource usage or monitoring fairshare distribution.

## Metric Name Comparison with the Legacy Exporter

If you were using `prometheus-slurm-exporter` before, note that metric names are completely different. Existing Grafana dashboards cannot be reused as-is; queries must be rewritten.

| Legacy Exporter | 25.11 Native | Description |
|---|---|---|
| `slurm_cpus_alloc` | `slurm_jobs_cpus_alloc` | Total allocated CPUs |
| `slurm_cpus_idle` | `sum(slurm_node_cpus_idle)` | Total idle CPUs |
| `slurm_cpus_total` | `sum(slurm_node_cpus)` | Total CPUs |
| `slurm_nodes_alloc` | `slurm_nodes_alloc` | Allocated nodes (same name) |
| `slurm_nodes_idle` | `slurm_nodes_idle` | Idle nodes (same name) |
| `slurm_node_cpu_alloc` | `slurm_node_cpus_alloc` | Per-node CPU |
| `slurm_node_mem_alloc` | `slurm_node_memory_alloc_bytes` | Per-node memory |
| `slurm_scheduler_mean_cycle` | `slurm_sched_mean_cycle` | Scheduler cycle time |
| `slurm_scheduler_backfilled_jobs_since_start` | `slurm_backfilled_jobs` | Backfilled jobs count |
| `slurm_account_fairshare` | — | Not available in native |

> Per-node metrics (`slurm_node_*`) carry a `{node="..."}` label, and fairshare metrics carry `{account="...", user="..."}` labels.

Some metrics differ only in name, while others are structured entirely differently. In particular, per-partition and per-user metrics are far richer in the native implementation. Fairshare metrics, however, are not yet included in the native endpoint.

## Grafana Dashboard

All existing Slurm dashboards on the Grafana Labs marketplace are based on the exporter:

- [SLURM Dashboard (#4323)](https://grafana.com/grafana/dashboards/4323-slurm-dashboard/)
- [Slurm DashboardV2 (#19835)](https://grafana.com/grafana/dashboards/19835-slurm-dashboardv2/)
- [Slurm Cgroups (#14587)](https://grafana.com/grafana/dashboards/14587-slurm-cgroups/)

I built a dashboard based on the 25.11 native metrics and published it to the marketplace:

- [Slurm Native OpenMetrics (#24979)](https://grafana.com/grafana/dashboards/24979-slurm-native-openmetrics/)

### Dashboard Layout

The dashboard is organized into five sections:

**1. Cluster Summary** — Stat panels showing running/pending jobs, CPU/memory utilization, and node state at a glance.

Key queries:
```yaml
# CPU utilization
sum(slurm_node_cpus_alloc) / sum(slurm_node_cpus)

# Memory utilization
sum(slurm_node_memory_alloc_bytes) / sum(slurm_node_memory_bytes)
```

**2. Job Trends** — Timeseries panels showing job state changes over time and completion/start/submission rates.

```yaml
# Job completion rate per minute
rate(slurm_sdiag_jobs_completed[5m]) * 60
```

**3. Per-node Resources** — Tracks CPU/memory allocation and utilization for each node. Useful for identifying nodes under concentrated load.

```yaml
# Per-node CPU utilization
slurm_node_cpus_alloc / slurm_node_cpus
```

**4. Per-user Jobs** — Stacked timeseries showing how much resources each user is consuming. Useful for detecting resource concentration.

```yaml
# Show only non-zero values (reduce noise)
slurm_user_jobs_running > 0
slurm_user_jobs_cpus_alloc > 0
```

**5. Scheduler Performance** — Internal performance metrics for slurmctld. Abnormally long scheduler cycle times can indicate throughput issues.

```yaml
# Scheduler cycle time (µs)
slurm_sched_mean_cycle
slurm_bf_mean_cycle

# slurmdbd queue size (growing = problem)
slurm_slurmdbd_queue_size
```

![Slurm Native OpenMetrics Dashboard](grafana_dashboard.png)

The dashboard can be imported from the [Grafana marketplace](https://grafana.com/grafana/dashboards/24979-slurm-native-openmetrics/).

## Summary

| Item | Legacy (Exporter) | 25.11 (Native) |
|---|---|---|
| Extra binary | Required | Not required |
| Data collection | CLI parsing (squeue, sinfo) | Direct internal data from slurmctld |
| Prometheus config | Required | Required (same) |
| Metric accuracy | Depends on CLI poll interval | Real-time internal state |
| Metric coverage | Limited | Rich (per-partition, per-user, detailed scheduler) |
| Maintenance | Manage exporter separately | Included in Slurm package |
| Grafana dashboard | Many on marketplace | [Listed on marketplace (#24979)](https://grafana.com/grafana/dashboards/24979-slurm-native-openmetrics/) |

Slurm 25.11's native OpenMetrics simplifies the architecture while providing richer metrics. The biggest advantages are eliminating the need to manage a separate exporter and being able to observe slurmctld's internal state directly.

That said, this is still an early-stage feature — there is no authentication yet, and some metrics like fairshare are not yet included. I look forward to seeing these gaps addressed in future releases.

---

*This post was written in a Slurm 25.11.2 + Prometheus 2.51 + Grafana 10.4 environment.*
