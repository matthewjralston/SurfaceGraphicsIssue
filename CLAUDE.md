# Surface Laptop Graphics Issue — Context

## Device
- Surface Laptop with **Intel Arc 130V GPU (16GB)**
- External monitor connection: switched from USB-C→HDMI to USB-C→DisplayPort cable on 2026-04-26
- Current driver: `32.0.101.8735` (dated April 20, 2026) — latest available as of 2026-04-26
- Previous driver `32.0.101.8629` regressed; 8735 installed 2026-04-25 and resolved the issue
- 8735 is NOT a complete fix — issue recurs after every sleep/wake cycle AND after hibernate (see Sleep/Wake Bug below)

## The Problem
Chrome and Microsoft Teams both show black screens / broken UI. Both are Chromium-based (Teams uses msedgewebview2), so this is one root cause: Intel Arc driver breaking GPU hardware acceleration in Chromium rendering.

Audio crackles when Chrome opens — indicates a GPU context reset is happening on launch, which momentarily interrupts Intel display audio (bundled with the graphics driver).

## History
- Issue existed **before** a previous driver install
- The driver install (forced via Intel app) **fixed** the issue
- Issue returned 2026-04-25 — regression on driver `32.0.101.8629`
- Driver `32.0.101.8735` installed 2026-04-25, resolved the issue
- Issue returned again 2026-04-26 — triggered by sleep/wake cycle (machine slept at 1:46 AM, issue present on resume)
- 2026-04-26: switched external monitor cable from USB-C→HDMI to USB-C→DisplayPort; rebooted from failed state to test
- 2026-04-28: issue recurred after hibernate resume (slept 3:07 AM UTC, woke 8:28 AM via Surface Button) — USB-C→DisplayPort cable confirmed NOT the cause; cable swap ruled out

## Previous Actions
- Had a Claude session about this issue (conversation not saved to memory)
- Forced newer Intel driver via Intel's app (version newer than Surface OEM driver)
- Posted about this on Reddit (Surface Laptop subreddit), Intel Community, and Microsoft community — check Reddit profile submitted history to find the post

## Known Fix History
- Intel driver `31.0.101.4499` / `31.0.101.4502` introduced the WebView2 black screen fix
- Driver `32.0.101.8629` regressed that fix
- Driver `32.0.101.8735` restored it, but only until the next sleep/wake cycle

## Sleep/Wake + Hibernate Bug (confirmed 2026-04-26, hibernate confirmed 2026-04-28)
Intel Arc driver fails to properly reinitialize Chromium GPU contexts after sleep/resume. DWM (desktop rendering) recovers fine; Chrome and Teams (both Chromium-based) do not — their GPU process ends up with a dead D3D context, producing full black screens.

**Diagnosis confirmed via:**
- No TDR events or GPU crash entries in Windows Event Log
- Chrome GPU process running (PID visible) but rendering black
- Fresh Chrome launch after killing all processes still black — rules out stale process, confirms driver-level stuck state
- No Windows Update applied overnight — machine simply suspended at 1:46 AM (CBS log: "computer is suspending", WindowsUpdateAgent active but no driver changes)
- `32.0.101.8735` is the latest available driver as of 2026-04-26; no update to apply

**Immediate workaround:** Launch Chrome with `--disable-gpu` flag (software rendering, functional but slower)
```
"C:\Program Files\Google\Chrome\Application\chrome.exe" --disable-gpu
```
*(Launched on 2026-04-26 before reboot but result unconfirmed — machine rebooted while in failed state)*

**Faster workaround (confirmed 2026-04-28):** Sign out of Windows and sign back in — restarts the desktop session and reinitializes the GPU context without a full reboot. Faster than rebooting.

**Proper fix:** Full shutdown + cold boot (not sleep/resume) — forces complete driver reinit, clears stuck GPU context

**Recurring prevention options:**
- ~~Disable sleep, use hibernate instead~~ — **WRONG: hibernate resume also triggers the bug (confirmed 2026-04-28)**
- Reboot instead of sleeping/hibernating until Intel patches this — only reliable prevention

**GPU disable/re-enable** (would reinit driver without reboot) requires elevated pnputil — access denied in normal session

## chrome://gpu Snapshot (captured 2026-04-25 14:35 UTC, driver 32.0.101.8629 — the broken driver)
- All features hardware accelerated (Canvas, Compositing, Rasterization, Video, WebGL, WebGPU)
- `exit_on_context_lost` workaround active — Chrome exits instead of recovering on GPU context reset; explains hard crashes rather than graceful recovery
- GPU crash count: 0 at time of capture (issue hadn't fully manifested yet)
- Chrome 147.0.7727.117, OS build 26200.8246

## Diagnostics To Try
- `chrome://gpu` — look for "GPU process was unable to start" or "GPU Reset" in the log at the bottom
- `chrome://flags/#disable-accelerated-2d-canvas` — disable, relaunch, test
- Roll back to Surface OEM driver from Microsoft Update Catalog (search "Surface Laptop Intel Graphics")
- Check Intel download page for driver newer than current: https://www.intel.com/content/www/us/en/download/785597/intel-arc-graphics-windows.html

## Diagnostic Commands (PowerShell)
```powershell
# Check current driver version
Get-WmiObject Win32_VideoController | Select-Object Name, DriverVersion, DriverDate

# Check for GPU TDR events
Get-WinEvent -FilterHashtable @{LogName='System'; StartTime=(Get-Date).AddDays(-2)} | Where-Object { $_.Message -match 'display|GPU|TDR|dxgkrnl' } | Select-Object TimeCreated, Id, Message

# List Chrome processes (check if GPU process is running)
Get-CimInstance Win32_Process -Filter "name='chrome.exe'" | Select-Object ProcessId, @{N='Type';E={if($_.CommandLine -match 'gpu-process'){'GPU'}elseif($_.CommandLine -match 'renderer'){'Renderer'}else{'Main/Other'}}}

# Kill Chrome and relaunch with GPU disabled (workaround)
Stop-Process -Name chrome -Force -ErrorAction SilentlyContinue
Start-Process "C:\Program Files\Google\Chrome\Application\chrome.exe" "--disable-gpu"
```

## Original Reddit Post (r/Surface)
https://www.reddit.com/r/Surface/comments/1srxokf/surface_laptop_7_arc_130v_10_bsods_in_24_hours/

> Sharing this in case anyone else is hitting unexplained crashes with a Surface Laptop 7.
>
> **Device:** Surface Laptop for Business 7th Edition — Core Ultra 258V / Arc 130V (16GB)
> **OS:** Windows 11 Pro 25H2
>
> **What happened:**
> 10 CRITICAL_PROCESS_DIED BSODs in under 24 hours, all identical GPU TDR crashes. Triggered by running two 4K monitors over USB-C. When the driver crashes, DWM loses its rendering surface and takes everything down with it — Chrome, Teams, Outlook, Adobe, taskbar, all simultaneously.
>
> **Root cause:**
> Intel Arc driver 32.0.101.6737 (April 2025). Intel has officially deprecated all drivers older than 32.0.101.6987 — mine was below that threshold. The Surface app reported the device fully up to date. Windows Update never offered the newer driver.
>
> **Fix:**
> Intel DSA (Driver & Support Assistant) correctly identified the outdated driver. DSA install failed with a live-driver conflict, but downloading the installer directly and rebooting worked. Now on 32.0.101.8629 — no crashes since.
>
> **The problem:**
> Intel's current Arc driver is ~12 months ahead of what Microsoft certifies for Surface. If you're on a Surface Laptop 7 with external monitors and hitting random full-desktop crashes, check your Arc driver version before anything else.

*(No comments on the post as of 2026-04-25)*

## Launch Instructions
Launch Claude from `C:\Users\matth\source\repos\matthewjralston\SurfaceGraphicsIssue` to resume this context.

## Permissions
Claude is authorized to run any command in this folder without asking for confirmation.
