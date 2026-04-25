# Surface Laptop Graphics Issue — Context

## Device
- Surface Laptop with **Intel Arc 130V GPU (16GB)**
- Current driver: `32.0.101.8735` (dated April 20, 2026) — fixed the regression
- Previous driver `32.0.101.8629` regressed; 8735 installed 2026-04-25 and resolved the issue

## The Problem
Chrome and Microsoft Teams both show black screens / broken UI. Both are Chromium-based (Teams uses msedgewebview2), so this is one root cause: Intel Arc driver breaking GPU hardware acceleration in Chromium rendering.

Audio crackles when Chrome opens — indicates a GPU context reset is happening on launch, which momentarily interrupts Intel display audio (bundled with the graphics driver).

## History
- Issue existed **before** a previous driver install
- The driver install (forced via Intel app) **fixed** the issue
- Issue has **returned** as of April 25, 2026 — regression on driver `32.0.101.8629`

## Previous Actions
- Had a Claude session about this issue (conversation not saved to memory)
- Forced newer Intel driver via Intel's app (version newer than Surface OEM driver)
- Posted about this on Reddit (Surface Laptop subreddit), Intel Community, and Microsoft community — check Reddit profile submitted history to find the post

## Known Fix History
- Intel driver `31.0.101.4499` / `31.0.101.4502` introduced the WebView2 black screen fix
- Current driver `32.0.101.8629` should include that fix but is regressing

## chrome://gpu Snapshot (captured 2026-04-25 14:35 UTC, driver 32.0.101.8629 — the broken driver)
- All features hardware accelerated (Canvas, Compositing, Rasterization, Video, WebGL, WebGPU)
- `exit_on_context_lost` workaround active — Chrome exits instead of recovering on GPU context reset; explains hard crashes rather than graceful recovery
- GPU crash count: 0 at time of capture (issue hadn't fully manifested yet)
- Chrome 147.0.7727.117, OS build 26200.8246

## Diagnostics To Try
- `chrome://gpu` — look for "GPU process was unable to start" or "GPU Reset" in the log at the bottom
- `chrome://flags/#disable-accelerated-2d-canvas` — disable, relaunch, test
- Roll back to Surface OEM driver from Microsoft Update Catalog (search "Surface Laptop Intel Graphics")
- Check Intel download page for driver newer than `32.0.101.8629`

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
Launch Claude from `C:\Users\matth\Desktop\SurfaceGraphicsIssue` to resume this context.

## Permissions
Claude is authorized to run any command in this folder without asking for confirmation.
