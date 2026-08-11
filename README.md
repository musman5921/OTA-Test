# OTA-Test

Public repo for HaLow OTA testing:

- **GitHub Releases** — `remote.bin` + `jockey.bin` assets (Wi‑Fi OTA and USB updater)
- **GitHub Pages** — WebSerial USB updater

## WebSerial USB updater

**https://musman5921.github.io/OTA-Test/**

The page:

1. Loads the **latest Release** via GitHub API  
2. Downloads `remote.bin` / `jockey.bin` in the browser  
3. Sends them to the Remote over **USB Serial/JTAG** (WebSerial)

There is no local file picker. To ship a new USB-flashable build, attach those assets to a new Release (same as Wi‑Fi OTA).

Use **Chrome** or **Edge** on desktop (HTTPS required — Pages provides that).

## Enable GitHub Pages (one time)

1. Push `index.html` to `main`.
2. Open https://github.com/musman5921/OTA-Test/settings/pages  
3. **Source:** Deploy from a branch → **Branch:** `main` → **Folder:** `/ (root)` → Save  
4. Open https://musman5921.github.io/OTA-Test/

## Keep Releases / `ver.txt`

Do not remove `ver.txt`. Continue publishing releases with assets named exactly:

| Asset | Target |
|-------|--------|
| `remote.bin` | Remote (ap_mode) |
| `jockey.bin` | Jockey (sta_connect) |

---

## Test procedure (USB via Pages)

### A. One-time: Flash Remote with USB-OTA firmware

Remote must already run the private-repo firmware that includes `usb_ota` (branch `feature/webserial-usb-ota`).

```powershell
cd "c:\Users\SPARTA LAPTOP\Documents\GitHub\esp32s3-morse-halow\mm-iot-esp32-2.10.4\examples\ap_mode"
idf.py build
idf.py -p COMx flash
```

Leave Remote powered and booted.

### B. Publish this page (if you changed it)

```powershell
cd "c:\Users\SPARTA LAPTOP\Documents\GitHub\OTA-Test"
git add index.html README.md
git commit -m "Fetch firmware from latest OTA-Test Release (no local picker)."
git push origin main
```

Hard-refresh Pages (`Ctrl+Shift+R`) after ~1 minute.

### C. Ensure a Release has the bins

Latest Release must include `remote.bin` and (for Jockey test) `jockey.bin`.  
Example: https://github.com/musman5921/OTA-Test/releases/latest

### D. Run the updater

1. Chrome/Edge → https://musman5921.github.io/OTA-Test/  
2. Wait for log: release loaded + downloads finished (or click **Load latest release**)  
3. **Connect USB** → pick **ESP32-S3 USB Serial/JTAG**  
4. **Ping Remote** → `HELLO_ACK` + device version  
5. **Flash Remote** → progress → `COMMIT OK` → device reboots  
6. Optional: with an online paired Jockey → **Flash Jockeys**

### Pass / fail

| Check | Pass |
|-------|------|
| Release load | Log shows tag + `remote.bin` / `jockey.bin` sizes |
| Ping | `HELLO_ACK` |
| Flash Remote | `COMMIT OK`, reboot, Ping works again |
| Flash Jockeys | `COMMIT OK`, Jockey reboots / returns online |

### Common problems

| Symptom | Fix |
|---------|-----|
| Release load fails | Check network / API rate limit; open Releases page manually |
| Missing asset | Attach `remote.bin` / `jockey.bin` to the latest Release |
| Ping timeout | Wrong COM port, or Remote missing USB OTA app |
| Same version | USB path still reflash; Wi‑Fi menu may skip same tag |

### Repos

| Repo | Role |
|------|------|
| `musman5921/OTA-Test` | Public Releases + Pages updater |
| Private `esp32s3-morse-halow` | Firmware source (`usb_ota`) |
