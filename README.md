# netcheck

A terminal network-health monitor for flaky connections — plane WiFi, hotspots, cafes, captive portals. Single Python file, standard library only.

```
netcheck   ● UP   PlaneNet · 5 GHz · Good · 866 Mbps                 12m 04s

  Health         ████████████████░░░░ 82/100   Browsing OK
                 ▆▇▇▆▅▄▃▃▄▅▆▇▇▇▇▇▆▆▅▅▄▃▂▂▃▄▅▆▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▆▆▅▅

  Latency           143 ms  ▃▄▅▆▇▇▆▅▄▃▃▄▅   p50 132 · p95 410
  Jitter             24 ms  ▂▃▃▂▂▃▄▃▂▂▂▃▂
  Loss              0.0 %   ▁▁▁▁▁▁▁▁▁▁▁▁▁
  DNS                42 ms  ▂▂▂▃▂▂▂▃▂▂▂▂▂
  Throughput      287.4 KB/s ▇▇▆▅▄▃▄▅▆▇▇▇▇

  Session        98.4% uptime · 248/252 checks · portal clean
  Gateway        ● 3 ms via icmp · 172.20.10.1

  Site Probes  (next in 30s)
                  Code    DNS   TTFB   Total
        Google    200     38    198     412 ms
     Wikipedia    200     42    211     488 ms
        GitHub    200     51    284     622 ms
        Reddit    200     61    393    1024 ms
       NYTimes    200     55    348     891 ms

  Transitions    ● UP for 11m 04s    1 outage · longest 28s
                 ↑16m 23s ▸ ↓28s ▸ ↑11m 04s ▸ now

  ────────  ~/.netcheck/netcheck.csv  ·  ⌃C to quit         ██████████ 10s
```

## What it measures

Every `--interval` seconds (default 10s), in parallel:

- **Ping** — latency, jitter (rolling stdev), loss to 8.8.8.8 and 1.1.1.1
- **Gateway** — your router's reachability (ICMP first, TCP 80/443/53 fallback when ICMP is filtered)
- **DNS** — resolution time for `www.google.com`
- **Throughput** — small CDN download in KB/s
- **Captive portal** — detects unexpected responses from `detectportal.firefox.com`
- **Site probes** (every Nth check) — full `curl` timing breakdown for Google, Wikipedia, GitHub, Reddit, NYTimes
- **WiFi info** — SSID, signal, link rate, band (polled in background)

All of these roll into a weighted 0–100 **health score** with a plain-English quality label (`Video/calls OK`, `Browsing OK`, `Chat/text only`, `Barely usable`, `Unusable`) and a one-word **bottleneck** call-out so you know whether to blame DNS, loss, throughput, etc.

When the network is broken, a yellow `↳` diagnosis line under Health explains *why* (captive portal expired, gateway reachable but Internet unreachable, gateway unreachable, etc.).

## Quick start

```bash
git clone https://github.com/YOUR-ORG/netcheck.git
cd netcheck
./netcheck                      # live dashboard
./netcheck --once               # one check, plain output, exit
./netcheck --json --once        # one check as JSON
./netcheck --quiet              # no dashboard; CSV log only
```

No `pip install` needed. Drop the `netcheck` file anywhere on your `PATH` if you want to run it from any directory.

## Common flags

| Flag                      | Purpose                                                                  |
| ------------------------- | ------------------------------------------------------------------------ |
| `--interval N`            | Seconds between checks (default 10)                                      |
| `--once`                  | Run one check and exit; exit code reflects health threshold              |
| `--threshold N`           | Health score below this makes `--once` exit non-zero (default 50)        |
| `--json`                  | Emit one JSON record and exit (implies `--once`)                         |
| `--quiet`                 | Suppress the dashboard; keep logging                                     |
| `--no-color`              | Disable ANSI colors                                                      |
| `--log-file PATH`         | CSV log path (default `~/.netcheck/netcheck.csv`; pass `''` to disable)  |
| `--json-log PATH`         | Optional JSONL log path                                                  |
| `--targets a,b,c`         | Comma-separated ping targets (default `8.8.8.8,1.1.1.1`)                 |
| `--probe-site NAME=URL`   | Replace default site probes; repeatable                                  |
| `--no-probes`             | Skip site probes                                                         |
| `--probe-interval N`      | Run site probes every Nth check (default 6)                              |
| `--notify`                | macOS notification on UP/DOWN transitions                                |
| `--sound`                 | Terminal bell on UP/DOWN transitions                                     |
| `--version`               | Print version and exit                                                   |

## CSV log

By default, every check appends a row to `~/.netcheck/netcheck.csv`. Schema:

```
timestamp, state, health, bottleneck, latency_ms, latency_p50_ms, latency_p95_ms,
jitter_ms, loss_pct, dns_ms, throughput_kBps, gateway_ok, gateway_ms,
gateway_method, captive_portal, portal_status, portal_final_url,
ssid, rssi, noise, tx_rate_mbps, channel, ping_targets
```

`ping_targets` is a compact JSON blob of the per-target results. Good for grepping or feeding into a notebook later. Pass `--json-log path.jsonl` if you'd rather have full structured records.

## Use as a one-shot health check

```bash
./netcheck --once && echo "good" || echo "bad"
```

Exits 0 if the composite health is ≥ `--threshold` and the network is up; non-zero otherwise. Useful in shell loops or as an `Alfred`/`Raycast` button.

## Requirements

- **macOS** (uses `route get default`, BSD `ping -W ms`, optionally `airport`/`system_profiler` for WiFi info, `osascript` for notifications)
- **Python 3.8+**
- `ping` and `curl` in `PATH`

Linux and Windows are not currently supported. The biggest blocker is that BSD `ping -W` is milliseconds while Linux `ping -W` is seconds, so a naive port would silently misbehave. Patches welcome.

## License

MIT — see [LICENSE](LICENSE).
