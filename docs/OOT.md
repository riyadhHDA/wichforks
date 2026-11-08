# Zelda: Ocarina of Time Remake notes

Reports and setup guides from public community posts and the experimental AMD channel, November 2026. This page documents the experimental setup, shader cache procedures, and performance reports for the OoT Remake (`0400486024dec000`).

> [!WARNING]
> This guide includes experimental builds and community-maintained shader packages. Use with care. Configurations can vary depending on GPU architecture, driver version, and system memory.

---

## What matters

- **Shader cache is critical**: OoT Remake generates tens of thousands of pipeline shaders. On a cold shader cache, gameplay experiences heavy micro-stuttering, asset hitching, or immediate Vulkan driver crashes. Installing a precompiled community shader cache (such as 76K–90K pipelines) and configuring pipeline preloading resolves most stutter.
- **AMD vs NVIDIA emulator pick**:
  - The shader cache files themselves are the same for both vendors (Vulkan pipeline descriptions for Title ID `0400486024dec000`).
  - What differs is the emulator binary:
    - **AMD GPUs**: Upstream Lichii and Eglantine builds frequently suffer from black 3D worlds, NaN blend crashes, and GPU driver timeouts on RDNA cards. Use the experimental AMD build: [CheezerWheezer/Lichii](https://github.com/CheezerWheezer/Lichii/releases) (`amd-superior` branch, build 6+), or [MythosOfs/Orchid](https://github.com/MythosOfs/Orchid/releases) (v0.7.4).
    - **NVIDIA GPUs**: Use upstream [zortos293/Lichii](https://github.com/zortos293/Lichii/releases) (build 88 or build 94/95), or [Fritchii](https://github.com/Frizti/Fritchii/releases) (v4.1, formerly Frizti v1-v3, for Windows visual, water and lighting fixes).
- **Shader sync & preloading (`EDEN_NX2_PRELOAD_ALL_PIPELINES`)**:
  - You can install shaders manually or automatically using [Lichii Shader Sync](https://github.com/oceanpacificpeace/lichii-shader-sync/).
  - Setting `EDEN_NX2_PRELOAD_ALL_PIPELINES=1` (without `#`) in `nx2_switches` / `nx2_switches.txt` forces the emulator to parse and compile all cached Vulkan pipelines on boot.
  - *Build 117+ note*: Builds 117 and newer evict unused pipelines after ~2.5 minutes, which can cause a ~20s stall if preloaded continuously. Use preload once as a warm-up to populate your GPU driver cache, then disable it for normal gameplay (see below).
- **Dynamic FPS & Frame Pacing**:
  - Out of the box, the title is capped at 30 FPS.
  - Dark's Dynamic FPS mod is bundled starting with Lichii build 88 to allow 60 FPS playback. On low-end systems, keeping 30 FPS gives more consistent frame pacing.
- **RAM and VRAM**:
  - 16 GB of system RAM is the practical minimum. 8 GB VRAM or higher prevents texture thrashing during Kokiri Forest and Deku Tree transitions.

---

## Step-by-step setup guide

### 1. Download the appropriate emulator build

- **AMD GPUs**: [CheezerWheezer/Lichii Releases](https://github.com/CheezerWheezer/Lichii/releases) (build 6 or newer).
- **NVIDIA GPUs**: [zortos293/Lichii Releases](https://github.com/zortos293/Lichii/releases) (build 88+) or [Fritchii Releases](https://github.com/Frizti/Fritchii/releases) (v4.1).

---

### 2. Install / sync the shader cache

You can use an automated tool (recommended) or download and extract manually:

#### Option A: Automated via Lichii Shader Sync (Recommended)

[Lichii Shader Sync](https://github.com/oceanpacificpeace/lichii-shader-sync) is an open-source utility (Windows and Linux) that automatically downloads, validates, and installs the latest verified shader cache packs (over 90,000 shaders for OoT `0400486024DEC000`) and backs up previous cache files to `.bak`:

1. Download the latest release: [Lichii Shader Sync Releases](https://github.com/oceanpacificpeace/lichii-shader-sync/releases/latest).
2. Boot the game once in Lichii, then close Lichii completely.
3. Run `Lichii-Shader-Sync.exe` (on Windows, click *More info $\rightarrow$ Run anyway* if SmartScreen prompts) and click **Update shaders**.
   - On Linux CLI: run `./lichii-sync sync`.
4. The tool downloads checksum-verified chunks, merges them into the shader cache, and creates a backup.

#### Option B: Manual installation

1. Press `Win + R`, type `%appdata%` and hit Enter.
2. Navigate to:
   ```text
   AppData\Roaming\eden\cache\shader\0400486024dec000\
   ```
   *(On Linux: `~/.local/share/eden/cache/shader/0400486024dec000/`)*
   *(Alternatively, in Lichii: right-click the game $\rightarrow$ **Open Transferable Pipeline Cache**).*
3. Back up any existing files, then clear the folder contents.
4. Download the community shader archive referenced in the [Reddit guide: How to install OoT Remake shadercache files](https://www.reddit.com/r/SwitchPirates2/comments/1xpm39k/how_to_install_oot_remake_shadercache_files_for/) (or use Option A above to pull validated packs directly from GitHub releases).
5. Extract the `.7z` contents directly into `0400486024dec000/`.

> [!NOTE]
> The shader files are identical whether you run NVIDIA or AMD hardware; only the emulator executable varies.

---

### 3. Configure `nx2_switches` and pipeline preloading

1. Open your emulator folder (or `%APPDATA%\eden\`).
2. Open `nx2_switches` (or `nx2_switches.txt`) in a text editor (e.g., Notepad).
3. Check the end of the file and ensure the following line is present without a `#` comment:
   ```ini
   EDEN_NX2_PRELOAD_ALL_PIPELINES=1
   ```
4. If it has a `#` at the beginning, remove the `#`. If the line does not exist, append it at the bottom.
5. Save and close the file.

> [!TIP]
> **Warm-up strategy for Build 117+:**
> On Lichii build 117+, the emulator evicts unused pipelines after ~2.5 minutes, which can cause a ~20s freeze when preloading massive packs.
> The recommended workflow:
> 1. In your GPU control panel (NVIDIA Control Panel or Linux environment variables), set the shader disk cache size to **10 GB** or **Unlimited** (`export __GL_SHADER_DISK_CACHE_SIZE=10737418240`).
> 2. Leave `EDEN_NX2_PRELOAD_ALL_PIPELINES=1` on.
> 3. Launch the game, wait at the title screen until pipeline compiling finishes, and quit. The compiled shaders are now persisted in the host GPU driver's cache.
> 4. Comment out preload (`# EDEN_NX2_PRELOAD_ALL_PIPELINES=1`) for normal play so you get instant driver-level cache hits without eviction stalls.

---

### 4. Emulator graphics settings

1. Launch Lichii.
2. Open **Configure** $\rightarrow$ **Graphics**.
3. Set **Graphics API** to **Vulkan**.
4. Set **Shader Backend** / compilation to **Asynchronous** (if available).
5. Launch the game. On the first warm-up launch, wait patiently for the pipeline loading bar to finish completely before playing.

---

## Known visual fixes & workarounds

- **Water flicker and lighting**: Fritchii v4.1 fixes water flickering across all zones (waterfalls, streams) by unbinding spurious light buffer texture references, and resolves overexposure jumps in Hyrule Field.
- **Foliage, grass and graveyard fixes**: Fritchii v4.1 fixes grass disappearing when swinging the sword, restores near foliage, and eliminates the dark slab/giant bright triangle glitch in Kakariko graveyard.
- **One-frame white flashes**: Fritchii (formerly Frizti) and Lichii build 94/95 include patches to eliminate white frame flashes on Link and terrain.
- **Overexposure and fog**: Orchid v0.7.3/v0.7.4 incorporates exposure, fog, and foliage reconstruction. To disable the visual fog thinning in interiors, set `EDEN_NX2_OOT_VISUAL_WORKAROUND=0` in `nx2_switches.txt`.
- **AMD rejected hash / driver timeout**: If the emulator crashes on AMD during shader compilation, add `EDEN_NX2_REJECT_HASH=e40561a7a0f7a0b3` to `nx2_switches.txt` and restart.
- **Dynamic resolution floor & DLSS**: Fritchii embeds a Max Resolution patch that prevents the 3D renderer from dropping to the 854x480 resolution floor, and adds an experimental option to inject NVIDIA DLSS.

---

## Community performance configs

| Date | Build | CPU | GPU | RAM | Performance | Notes |
|---|---|---|---|---|---|---|
| 7 Nov | Lichii build 93 | i5-12400F | RTX 2060 | 16 GB | 15-60 FPS | Playable, cutscenes and boss fights verified; 26 FPS on GPU accurate |
| 7 Nov | Lichii build 93 | i9-12900H | RTX 3070 Ti Laptop | 16 GB | 20-40 FPS | Runs, occasional freezes during ocarina freestyle |
| 6 Nov | Frizti v1 | not stated | RTX 4070 | not stated | ~20 FPS | Clean visuals, no graphical glitches |
| 6 Nov | unspecified | Ryzen 5 5500 | RTX 3060 12 GB | not stated | 15-22 FPS | Playable in opening areas |
| 6 Nov | Lichii build 88 | not stated | RTX 4080 / 5080 | 32 GB | 60 FPS | Opening areas, Dynamic FPS mod enabled |
| 6 Nov | Frizti v1 / Lichii 80 | 7800X3D | RX 9070 XT | not stated | Crashes on load | Fixed on CheezerWheezer amd-superior build 6 |
| 5 Nov | Lichii build 80 | 9800X3D | RTX 2070 | not stated | 60 FPS | Kokiri Forest |
| 5 Nov | Lichii build 80 | not stated | RTX 3080 10 GB | not stated | 40-50 FPS | Kokiri Forest, 40-60 Deku Tree |
| 5 Nov | Lichii build 80 | i3-10100 | RTX 3060 12 GB | 24 GB | 20-24 FPS | Kokiri Forest |
| 5 Nov | Lichii build 80 | i5-11400H | RTX 2050 | 16 GB DDR4 | 20 FPS | |
| 5 Nov | Lichii build 80 | 7800X3D | RX 580 | 32 GB | 15 FPS | |
| 5 Nov | Lichii build 80 | 13400F | RX 7900 XTX | not stated | 20-24 FPS | Stutters without precompiled shader cache |
| 5 Nov | Eglantine rc1 | 7700X | RX 7900 XTX | not stated | Infinite load | Fixed with pipeline preloading and AMD build |

---

## Sources

- [CheezerWheezer/Lichii Releases](https://github.com/CheezerWheezer/Lichii/releases) (AMD experimental channel, `amd-superior`)
- [zortos293/Lichii Releases](https://github.com/zortos293/Lichii/releases) (NVIDIA builds, build 88+)
- [Fritchii Releases](https://github.com/Frizti/Fritchii/releases) (v4.1 visual, water and lighting fixes, formerly Frizti/Lichii)
- [Lichii Shader Sync](https://github.com/oceanpacificpeace/lichii-shader-sync) (Automated shader cache downloader & manager)
- [Reddit Shader Cache Guide](https://www.reddit.com/r/SwitchPirates2/comments/1xpm39k/how_to_install_oot_remake_shadercache_files_for/)
- [Lichii Reports dataset](LICHII-REPORTS.md)
