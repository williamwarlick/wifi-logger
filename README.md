# wifi-logger

A small background logger for macOS that checks your Wi-Fi and internet once a minute and writes the results to a spreadsheet-friendly CSV file. Leave it running for a week or two and you'll have timestamped evidence of outages and slow periods, plus whether the problem was the local network (Wi-Fi, router, extenders) or the internet provider.

## What it checks every minute

- **Wi-Fi link:** connected or not, signal and noise (dBm), channel, and link rate. A strong signal shows the problem isn't on your end.
- **Router:** 5 pings to your router (the "gateway"). If these fail, the problem is the local network: Wi-Fi, router, mesh or extender.
- **Internet:** 5 pings each to Cloudflare (1.1.1.1) and Google (8.8.8.8), plus a web request to Apple's connectivity check, which also measures DNS time. If the router answers but these fail, the problem is upstream at the internet provider.
- **Speed test:** every 30 minutes, using Apple's built-in `networkQuality` (download, upload, responsiveness). It isn't run every minute because each test uses the connection as hard as it can for about 20 seconds. That would use a lot of data, slow everyone else down, and cause the very slowdowns you're trying to measure.

Each check gets one of these labels:

| status | meaning |
| --- | --- |
| `ok` | everything answered, no packet loss |
| `degraded` | online, but packet loss to the router or either internet server, a failed web check, or a web response slower than 3 s |
| `router_unreachable` | offline, and the router didn't answer (local network problem) |
| `internet_down` | offline, but the router answered (internet provider problem) |
| `wifi_disconnected` | the Mac wasn't connected to any Wi-Fi network |
| `wifi_off` | Wi-Fi was turned off on the Mac (not counted) |
| `waking` | the Mac woke up less than 2 minutes ago and was still reconnecting (not counted) |

## Install

You need macOS 15 or newer. Open Terminal and run:

```bash
git clone https://github.com/williamwarlick/wifi-logger.git
cd wifi-logger
./wifi-logger install
```

No git? Download the ZIP from GitHub, unzip it, and run `bash wifi-logger install` from that folder.

macOS may show a "Background Items Added" notification for `wifi-logger`. That's expected. Leave it allowed under System Settings → General → Login Items & Extensions.

To run the speed test at a different interval, set it at install time. `0` turns it off:

```bash
SPEEDTEST_EVERY_MIN=60 ./wifi-logger install
```

## See the results

```bash
./wifi-logger status
```

shows whether it's running and the last 5 checks.

```bash
./wifi-logger summary
```

shows the evidence: % of time working, degraded and offline, every outage with its start time, length and likely cause, average signal, and speed test results. It ends with a breakdown by network (identified by the router's hardware address), which is useful for comparing two Wi-Fi networks or leaving out checks taken somewhere else.

The raw data is in `~/Library/Application Support/wifi-logger/wifi-log.csv`. You can open it in Numbers, Excel or Google Sheets. If an update changes the columns, the old file is kept next to it as `wifi-log-before-<date>.csv`.

## Uninstall

```bash
./wifi-logger uninstall
```

stops the background check. Your CSV log is kept.

## Limitations

- **It only records while the Mac is awake.** Gaps while it sleeps show up as gaps, not outages. For round-the-clock coverage, use a Mac that stays plugged in, and turn on "Prevent automatic sleeping when the display is off" in System Settings → Battery → Options (or Energy on desktops).
- **"Router" means the router your Mac uses as its gateway.** On a mesh system (eero, Orbi, etc.), that's the main mesh unit, so router pings also measure the wireless link between mesh units or extenders.
- **Network names don't appear in the log,** because macOS hides them from scripts. The `router_mac` column identifies which network each row came from, so you can filter out checks taken elsewhere.
