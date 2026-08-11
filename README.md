# OTA-Test

Public repo for HaLow OTA testing:

- **GitHub Releases** — firmware packages for the existing Wi‑Fi / GitHub OTA path
- **GitHub Pages** — WebSerial USB updater (no ESP32 Wi‑Fi required)

## WebSerial USB updater URL

After Pages is enabled (see below):

**https://musman5921.github.io/OTA-Test/**

Use **Chrome** or **Edge** on a desktop. WebSerial needs HTTPS (Pages) or localhost.

The page never uploads your `.bin` files to GitHub. You choose a local file; the browser sends it straight to the Remote over USB.

## Enable GitHub Pages (one time)

1. Commit and push `index.html` (and this README) to `main`.
2. Open https://github.com/musman5921/OTA-Test/settings/pages
3. Under **Build and deployment**:
   - **Source:** Deploy from a branch
   - **Branch:** `main`
   - **Folder:** `/ (root)`
4. Click **Save**.
5. Wait 1–2 minutes, then open https://musman5921.github.io/OTA-Test/

## Keep `ver.txt` / Releases as they are

Do not delete `ver.txt`. Tag/release workflow for Wi‑Fi OTA can stay unchanged. The updater page is only an extra file at the repo root.

---

## Full USB OTA test procedure

You need two things:

1. **Remote firmware that includes USB OTA** (built from private `esp32s3-morse-halow` branch `feature/webserial-usb-ota`)
2. **This Pages site** open in Chrome/Edge

### A. Build and flash Remote once (ESP-IDF)

1. Open ESP-IDF PowerShell.
2. Build Remote:

```powershell
cd "c:\Users\SPARTA LAPTOP\Documents\GitHub\esp32s3-morse-halow\mm-iot-esp32-2.10.4\examples\ap_mode"
idf.py build
```

3. Plug Remote USB-C into the PC. Note the COM port (Device Manager).
4. Flash:

```powershell
idf.py -p COMx flash
```

Replace `COMx` with your port (example: `COM5`).

5. Wait for boot (LCD / HaLow AP as usual). Leave Remote powered on.

Firmware file you will pick later in the browser:

`...\examples\ap_mode\build\ap_mode.bin`

### B. Publish this updater page (if not done yet)

```powershell
cd "c:\Users\SPARTA LAPTOP\Documents\GitHub\OTA-Test"
git status
git add index.html README.md
git commit -m "Add WebSerial USB firmware updater for GitHub Pages."
git push origin main
```

Then enable Pages as in **Enable GitHub Pages** above.

### C. Open the Pages site and connect USB

1. On the PC, open **Chrome** or **Edge**.
2. Go to: https://musman5921.github.io/OTA-Test/
3. Plug Remote in (app running — not stuck in bootloader/download mode).
4. Click **Connect USB**.
5. In the port list, choose **ESP32-S3 USB Serial/JTAG**.

Tip: the board may show two COM ports. Prefer **USB Serial/JTAG**, not the UART bridge used by `idf.py monitor`. If Ping fails, try the other port. Close `idf.py monitor` if it holds the same port.

6. Click **Ping Remote**.  
   Pass = log shows `HELLO_ACK` and a firmware version.

### D. Test Flash Remote (self update)

1. Click **Choose remote.bin** → select:

`...\ap_mode\build\ap_mode.bin`

2. Click **Flash Remote**.
3. Wait for progress → `COMMIT OK` → Remote reboots.
4. After reboot: **Connect USB** again → **Ping Remote**.  
   Pass = Ping works on the new image.

### E. Test Flash Jockeys (optional)

Only if at least one Jockey is paired and online on the Remote dashboard.

1. Build Jockey firmware:

```powershell
cd "c:\Users\SPARTA LAPTOP\Documents\GitHub\esp32s3-morse-halow\mm-iot-esp32-2.10.4\examples\sta_connect"
idf.py build
```

Binary: `...\sta_connect\build\sta_connect.bin`

2. Confirm Jockey is **online** on Remote.
3. On the Pages site: Connect → Ping.
4. **Choose jockey.bin** → `sta_connect.bin` → **Flash Jockeys**.
5. Pass = `COMMIT OK` and Jockey reboots / comes back online.

### Pass / fail cheat sheet

| Step | Pass |
|------|------|
| Pages URL loads | Updater UI visible over HTTPS |
| Ping Remote | `HELLO_ACK` + version |
| Flash Remote | `COMMIT OK`, reboot, Ping again works |
| Flash Jockeys | `COMMIT OK`, online Jockey updates |

### Common problems

| Symptom | Likely fix |
|---------|------------|
| No WebSerial / Connect fails | Use Chrome/Edge desktop; use the HTTPS Pages URL |
| Ping timeout | Wrong COM port, or Remote missing USB OTA firmware (re-flash Step A) |
| Port busy | Close `idf.py monitor` / another serial app |
| Jockey `NO_JOCKEY` | Pair and wait until Jockey shows online |

### Files involved

| Repo | Role |
|------|------|
| `musman5921/OTA-Test` (this repo) | Public Pages host + Releases |
| Private `esp32s3-morse-halow` | Real firmware source (`usb_ota` on Remote) |
