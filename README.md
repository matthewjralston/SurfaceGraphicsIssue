# Surface Laptop 7 — Intel Arc 130V Graphics Issue

Tracking a recurring Intel Arc driver regression that breaks Chrome, Microsoft Teams, and other Chromium-based apps on Surface Laptop 7.

## Device

- **Model:** Surface Laptop for Business 7th Edition
- **CPU/GPU:** Intel Core Ultra 258V / Arc 130V (16GB)
- **OS:** Windows 11 Pro 25H2 (build 26200.x)

## Symptoms

- Chrome shows a black screen or crashes on launch
- Microsoft Teams shows broken/black UI (Teams uses msedgewebview2, which is Chromium-based)
- Audio crackles briefly when Chrome opens — caused by a GPU context reset interrupting Intel display audio (bundled with the graphics driver)
- Both apps share the same root cause: Intel Arc driver breaking GPU hardware acceleration in Chromium rendering

## Driver History

| Driver Version | Status |
|---|---|
| `31.0.101.4499` / `31.0.101.4502` | First drivers to include the WebView2 black screen fix |
| `32.0.101.6737` | Caused 10 CRITICAL_PROCESS_DIED BSODs in 24 hours under dual 4K monitors (GPU TDR crashes) |
| `32.0.101.6987` | Intel deprecation cutoff — drivers older than this are officially unsupported |
| `32.0.101.8629` | Regressed — Chrome/Teams black screen returned (as of 2026-04-25) |
| `32.0.101.8735` | Fixed the regression (installed 2026-04-25) |

## Root Cause

Intel's Arc driver ships roughly 12 months ahead of what Microsoft certifies for Surface. Windows Update and the Surface app both report the device as fully up to date while leaving a significantly older driver installed.

When Chrome or Teams launches with a broken driver, the GPU context resets. The `exit_on_context_lost` workaround in Chrome causes it to hard-crash rather than recover gracefully. The audio glitch on launch is a side effect of the same reset hitting the Intel display audio device.

## Workaround / Fix

**Do not rely on Windows Update or the Surface app for Intel Arc drivers.** Use Intel DSA (Driver & Support Assistant) or download directly from Intel.

1. Check your current driver: Device Manager → Display Adapters → Intel Arc → Driver tab
2. Compare against the latest at [intel.com/arc-drivers](https://www.intel.com/content/www/us/en/products/sku/237832/intel-arc-graphics-130v/downloads.html)
3. If DSA install fails with a live-driver conflict, download the standalone installer and reboot before installing

## Diagnostics

- `chrome://gpu` — check the bottom log for "GPU process was unable to start" or "GPU Reset" entries
- `chrome://flags/#disable-accelerated-2d-canvas` — disable hardware canvas acceleration as a temporary workaround
- Roll back to the Surface OEM driver from Microsoft Update Catalog (search "Surface Laptop Intel Graphics") if the newest Intel driver is the regression

## Community Posts

- [r/Surface — 10 BSODs in 24 hours (GPU TDR / dual 4K)](https://www.reddit.com/r/Surface/comments/1srxokf/surface_laptop_7_arc_130v_10_bsods_in_24_hours/)
