# Eden SD8 Gen 1 performance lab — experiment 01

**Status: experimental, not validated on-device.** This branch is independent of
the public `eden-game-mode` release. It is a controlled compiler-only A/B test,
NOT a claim of improved FPS.

## What changes
- New opt-in `YUZU_BUILD_PRESET=sd8gen1` in root CMake configuration.
- AArch64 ISA: `-march=armv8.2-a+lse+rcpc`, including atomic LSE/RCpc.
- Code scheduling: `-mtune=cortex-x2`. The tune flag does **not** enable
  additional CPU instructions.
- The reference build uses `YUZU_BUILD_PRESET=custom` (same source revision).
- No changes to Vulkan, shaders, NCE logic, Game Booster integration,
  Lossless Scaling, or ReShade post-processing.
- LTO stays as in the existing Standard build; no LTO flag is enabled here.

The GitHub Actions job produces **two independent APK artifacts from the same
Git commit**: `baseline` and `sd8gen1`. Both are built with the existing
Samsung Game Booster integration and with LSFG and ReShade enabled. No DLL or
proprietary game content is included.

## Android test procedure (Galaxy S22 Ultra / SM8450 / 8 GB)
1. **Back up saves and Eden settings.** Do not uninstall the working app
   just to change variants. Preserve the public v2.1.0 APK as rollback.
2. Use the same Turnip version (initially 26.3.0-R6), resolution, game update,
   mods, battery mode, Game Booster setting, and external cooler for both.
3. For real game FPS, disable Lossless Scaling frame generation in *both*
   variants. Disable post-processing for the first comparison as well.
   We can retest cosmetics/FG after finding a base-FPS difference.
4. Same BOTW save, same spot and camera/route (e.g. outside Kakariko), same
   shader cache state where possible. Let shaders warm up; note whether a run
   was a cold or warm cache. Do NOT casually delete shaders between trials.
5. After a thermal reset/cooldown, warm up 2 minutes then run the same
   5-minute route, noting average FPS, lowest sustained FPS, stutters,
   temperature and crashes. Repeat at least three times per variant;
   alternate baseline/tuned order when possible.
6. Optional secondary test: Sparking! ZERO, recording crash/no-crash across
   repeated matches, not just instantaneous FPS.
7. If the tuned build crashes, generates artifacts, triggers higher heat, or
   underperforms, revert to baseline or the public release. Do not publish
   tuning as a performance improvement without repeated A/B evidence.

### Measurement template
| Build | Trial | FPS avg | 1% low (if measured) | Min sustained FPS |
|---|---:|---:|---:|---:|
| Baseline | 1 | | | |
| Baseline | 2 | | | |
| Baseline | 3 | | | |
| SD8G1 tuned | 1 | | | |
| SD8G1 tuned | 2 | | | |
| SD8G1 tuned | 3 | | | |

If only the in-game FPS counter is available, record a consistent series
of samples or a screen recording and avoid claiming accurate 1% lows.

## Why this first?
The Snapdragon 8 Gen 1 uses ARMv9-derived Cortex-X2, A710 and A510 cores.
This experiment aims to improve the **emulator's native C/C++ code generation**;
it does not accelerate already-native Switch instructions executed via NCE,
and it does not change GPU shader compilation or a game's inherent limits.
Performance gains may be zero or negative; only on-device tests can tell.

**Source:** internal `CMakeLists.txt` ARM64 preset block and
`.ci/android/build.sh`. This is a compiler experiment, not an upstream
optimisation patch.
