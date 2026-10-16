
Test **local Spectrum/community gateway**.
```
ping -t 100.72.0.1
```

Default Gateway from
```
ipconfig
```

```
ping -t YOUR_DEFAULT_GATEWAY
```

Test the connection out to the **internet** via Cloudflare
```
ping -t 1.1.1.1
```



Induce Load
```bash
curl.exe -L "https://speed.cloudflare.com/__down?bytes=1000000000" -o NUL
```

That downloads about **1 GB** through your connection but discards the data instead of saving it.

For a more aggressive test, open **2–3 additional terminals** and run that same `curl` command simultaneously. That should push your connection much closer to saturation.


---

If `100.72.0.1` times out during the speed test while:

```
ping -t 1.1.1.1
```

continues responding normally, the most likely explanation is that the Spectrum device at `100.72.0.1` is **deprioritizing or rate-limiting ICMP replies to itself under load**. It can still forward your traffic normally while refusing to spend resources answering pings addressed directly to it.


| During load    | `100.72.0.1` | `1.1.1.1`   | Likely meaning                                       |
| -------------- | ------------ | ----------- | ---------------------------------------------------- |
| Healthy        | low          | low         | Connection behaving normally                         |
| Bad            | spikes/loss  | spikes/loss | Problem starts at/near your first Spectrum hop       |
| Odd            | timeouts     | normal      | Gateway likely deprioritizing ICMP replies           |
| Upstream issue | normal       | spikes/loss | Congestion/problem farther inside Spectrum or beyond |

the most likely explanation is that `100.72.0.1` was **deprioritizing or rate-limiting ICMP replies to itself while under load**, while it continued forwarding your traffic normally.

ICMP stands for **Internet Control Message Protocol**.

It’s a network protoco  jj      klkj;lkj;lkj;lkj;lk l used for diagnostics and error reporting. The `ping` command uses ICMP:

## Current home network

```
Your PC
  ↓ Ethernet
100.72.0.1   ← first routed hop / default gateway
  ↓
Spectrum network
  ↓
Internet
```

