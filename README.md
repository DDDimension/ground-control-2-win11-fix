# Ground Control II — Windows 11 Fix

**Fix Ground Control II (Steam AppID 254840) black screen / hang / artifacts / broken mouse on Windows 11 with high-refresh-rate displays.**

> ⚡ **Game files are 100% untouched.** This fix works by placing a proxy `d3d9.dll` in the game folder — Windows loads DLLs from the executable's own directory first, so the game's own `Direct3DCreate9()` call gets answered by DXVK instead of the obsolete Microsoft DLL. Delete the added files and the game is byte-for-byte back to original.

---

## Problem

Ground Control II (2004, 32-bit, DirectX 9) fails on modern Windows 11 systems, especially with high-refresh-rate monitors:

| Symptom | Detail |
|---|---|
| **Black screen / hang** | Process alive, window never appears, 1 second of intro audio then nothing |
| **AppHangB1 + APPCRASH** | `c0000005` access violation in `ntdll.dll` |
| **Vertical-stripe artifacts** | Rainbow vertical bars covering the screen |
| **Mouse doesn't respond** | Cursor frozen in game |
| **Rendering squeezed into a corner** | Only the top-left block renders, rest of the screen is garbage, mouse coordinates offset |

Root cause: the game initializes D3D9 and enumerates every display adapter. On a system with a high-refresh-rate monitor, extra virtual display adapters, and modern TSF input-method DLLs injected into the process, the 2004-era D3D9 initialization path crashes or mis-negotiates the swapchain.

Steam's "compatibility mode", lowering the refresh rate, disabling virtual adapters, and community `dinput` patches (e.g. [Vegasq/Ground-Control-2-wireless-fix](https://github.com/Vegasq/Ground-Control-2-wireless-fix), MIT) **do not fix it** — they all treat peripheral issues, not the D3D9 initialization path itself.

---

## Solution

Use **DXVK** (Direct3D 9 → Vulkan translation layer) to take over D3D9, plus a tuned `dxvk.conf`.

```
Without fix:  gcii.exe → C:\Windows\SysWOW64\d3d9.dll   (Microsoft, 2004-era → hang)
With fix:     gcii.exe → .\d3d9.dll  (DXVK → Vulkan → your GPU)
```

The game still calls the exact same `Direct3DCreate9()` API. It never knows it was intercepted.

---

## Results

| Item | Before | After |
|---|---|---|
| Launch | ❌ Black screen, hang, crash | ✅ Enters game |
| Picture | ❌ Vertical-stripe artifacts | ✅ Clean, fills the screen |
| Mouse | ❌ Frozen | ✅ Works normally |
| Rendering area | ❌ Squeezed into top-left corner | ✅ Correct full-screen |
| In-game resolution list | 5 low-res options | ⚠️ Still 5 low-res options — **game engine limitation**, see below |

---

## Requirements

- Windows 11 (likely also helps on Windows 10)
- A GPU with working **Vulkan** support (NVIDIA / AMD / Intel from ~2016 onward)
- Ground Control II installed via Steam (AppID **254840**)

---

## Installation

### Step 1 — Download DXVK

Go to **https://github.com/doitsujin/dxvk/releases** and download the latest release, e.g. `dxvk-3.1.1.tar.gz`.

> This repository deliberately **does not** redistribute DXVK's binaries (zlib-licensed, maintained by doitsujin). Always grab them from the official source.

### Step 2 — Extract the 32-bit DLLs

Extract the archive. Inside you'll find `x32/` and `x64/` folders.

**You need the `x32` folder** — the game is a 32-bit executable.

Copy these **two** files out of `x32/`:

```
x32\d3d9.dll
x32\dxgi.dll
```

> Do **not** copy from `x64/`. Do **not** copy `d3d10core.dll` / `d3d11.dll` / `d3d8.dll` — only `d3d9.dll` and `dxgi.dll` are required.

### Step 3 — Find your game folder

Default Steam location:

```
C:\Program Files (x86)\Steam\steamapps\common\Ground Control II
```

Not sure? In Steam: right-click **Ground Control II** → **Manage** → **Browse local files**.

You should see `gcii.exe` in there. That's the right folder.

### Step 4 — Copy the files in

Put the two DLLs **next to `gcii.exe`**:

```
Ground Control II\
├── gcii.exe
├── d3d9.dll          ← newly added (DXVK x32)
├── dxgi.dll          ← newly added (DXVK x32)
├── dxvk.conf         ← newly added (this project's config)
├── gc2data.sdf       (original, untouched)
└── ...               (all other original files untouched)
```

Then copy **`dxvk.conf`** from this repository into the same folder.

### Step 5 — Launch from Steam as usual

Just click **Play** in Steam. No launcher script, no environment variables, no Steam launch options needed.

### Step 6 — Dismiss the hardware-check dialog

On launch the game shows a dialog complaining about your GPU / driver.

**Click "No" / "Cancel" and continue.** It's harmless — see the FAQ below.

---

## Known Limitations

### In-game resolution list still only shows 5 low-res options

The game reads only the **first 5 entries** of the enumerated resolution list, and Windows enumerates resolutions in ascending width order. The first 5 are therefore always:

```
640x480 · 720x480 · 720x576 · 800x600 · 1024x768
```

Your native resolution sits far beyond that — the game can never reach it.

This is a **hardcoded game-engine limitation**, not a DXVK setting. A `d3d9.forceAspectRatio` filter could compress the list so that high resolutions fall within the first 5 entries, but that approach requires the `DXVK_FORCE_WINDOWED` environment variable — and environment variables are **not** injected when the game is launched directly from Steam. So it isn't applicable to the normal launch path.

**Important nuance:** the rendering *area* is correct and full-screen regardless — the picture is scaled up to fill the display. Only the in-game dropdown is limited.

### Hardware-check dialog on startup

The game reads legacy hardware info from the registry (`Gfx Vendor`, `Driver version`, DX9 driver presence). DXVK doesn't write those legacy registry values, so the game reports e.g. `Gfx Vendor: nVidia, 0 Mb` / `DX9 driver: no` and warns.

**This does not affect gameplay.** Graphics are handled by DXVK → Vulkan. Click "No"/"Cancel" and play normally. There is no clean way to suppress it without modifying game files or system configuration — both of which this fix deliberately avoids.

### Refresh-rate-specific config

`dxvk.conf` contains `d3d9.forceRefreshRate = 300` — **this is an example value tuned for a 300 Hz display, and you must change it to match your own monitor.** It serves two purposes:

1. It makes the game request the highest resolution (the highest refresh rate only exists on the monitor's native mode, so after filtering, that's the mode the game picks).
2. It prevents a fallback to a refresh-rate tier that triggers vertical-stripe artifacts on some hardware.

**Set it to your display's maximum refresh rate**, e.g.:

```ini
d3d9.forceRefreshRate = 144     ; for a 144 Hz monitor
d3d9.forceRefreshRate = 165     ; for a 165 Hz monitor
```

If you don't know it, run the game once and check the generated log — or just use `dxdiag`. Leaving it at a value your monitor doesn't support may cause artifacts or a fallback to a low resolution.

---

## Uninstall

Delete these files from the game folder:

```
d3d9.dll
dxgi.dll
dxvk.conf
```

The game instantly returns to its original state (which means: black screen hang on modern hardware).

To be extra sure nothing was changed, use Steam's built-in check:

**Steam Library → right-click Ground Control II → Properties → Installed Files → Verify integrity of game files**

All game files will pass — because none of them were ever modified.

---

## Proof that the game files are untouched

See **[游戏本体完整性审计报告.md](游戏本体完整性审计报告.md)** (Chinese) for the full audit, including:

- File modification timestamps: every original game file is stamped at the Steam install moment, and every fix action happened strictly later — the two sets are completely time-separated.
- MD5 fingerprints of the original executables and DLLs.
- A whitelist of every file written during the fix session.

### Verification methods you can run yourself

**Method 1 — Steam verification (most authoritative)**

Steam Library → right-click Ground Control II → Properties → Installed Files → **Verify integrity of game files**. Expect: all files validated.

**Method 2 — Timestamp comparison**

```bash
cd "/path/to/Ground Control II"
find . -type f -printf "%T+  %p\n" | sort
```

All original files share the install timestamp; only the newly added files have later timestamps.

**Method 3 — Delete test**

Remove `d3d9.dll` / `dxgi.dll` / `dxvk.conf` and launch. The original black-screen hang returns immediately — proving the entire fix lives in these three added files and that the game itself was never altered.

---

## How the interception works

Windows resolves DLL imports in this order:

```
executable's own directory  →  system directory  →  PATH
```

When `gcii.exe` imports `d3d9.dll`, Windows finds the copy **in the game folder first**. That's the entire mechanism — no patching, no hex editing, no binary modification.

```
gcii.exe calls Direct3DCreate9()
        ↓
Windows loads .\d3d9.dll  (DXVK)
        ↓
DXVK translates D3D9 calls to Vulkan
        ↓
Vulkan driver → your GPU
```

---

## Configuration reference

The shipped `dxvk.conf` is fully commented. Key settings:

| Setting | Value | Why |
|---|---|---|
| `d3d9.forceRefreshRate` | `300` | Forces the highest mode; avoids the 60 Hz artifact path |
| `d3d9.modeCountCompatibility` | `False` | Keeps the full mode list available to the engine |
| `dxvk.enablePresentTiming` | `False` | ✅ Fixes the frozen mouse |
| `d3d9.presentInterval` | `-1` | Immediate present; avoids frame pacing stalls |
| `d3d9.maxFrameRate` | `60` | The engine is not designed for high frame rates |
| `dxvk.deviceFilter` | `"NVIDIA"` | Ensures the discrete GPU is used (systems with iGPU + dGPU) |
| `d3d9.dpiAware` | `True` | Correct window sizing on scaled displays |
| `dxvk.allowFse` | `True` | Allow fullscreen-exclusive-style presentation |
| `dxvk.tearFree` | `False` | Tearing-free presentation caused stalls on this title |

---

## FAQ

**Q: Steam says the game is "running" but nothing appears.**
A: That's the original symptom. Confirm `d3d9.dll` and `dxgi.dll` are next to `gcii.exe` (not in a subfolder) and that you copied them from `x32/`, not `x64/`.

**Q: Artifacts (vertical stripes) are back.**
A: Adjust `d3d9.forceRefreshRate` to a refresh rate your monitor actually supports. On some hardware, certain refresh-rate tiers (notably 60 Hz) trigger the artifacts.

**Q: The mouse is frozen again.**
A: Check that `dxvk.enablePresentTiming = False` is present and not commented out. Also make sure the config file is named exactly `dxvk.conf` (lowercase), next to `gcii.exe`.

**Q: Can I get my native resolution in the in-game menu?**
A: Not through this approach — see Known Limitations. The engine only reads the first 5 enumerated modes.

**Q: Do I need to change Steam launch options or environment variables?**
A: No. Launch normally from Steam. Everything is file-based.

**Q: Does this modify my Steam configuration?**
A: No. Nothing outside the game folder is touched — no registry writes, no Steam settings, no system environment variables.

**Q: Can I use this with the GOG version?**
A: Yes — the same DLL-interception mechanism applies. Point the files at the folder containing the game's executable.

---

## Credits & License

- **DXVK** by doitsujin — https://github.com/doitsujin/dxvk (zlib License). All credit for the translation layer belongs to its authors. This repository only contains **configuration and documentation**.
- **Ground-Control-2-wireless-fix** by Mykola "Nick" Yakovliev (Vegasq) — https://github.com/Vegasq/Ground-Control-2-wireless-fix (**MIT License**, see `docs/LICENSE-Vegasq-inertia-fix.txt`). A community `dinput.dll` patch that fixes the wireless-USB startup crash and adds smooth wheel-zoom inertia. Not required by this fix and not distributed here; see `docs/licensing-notes.md` §4 for SHA-256 verification details.
- **Ground Control II** © Massive Entertainment / Ubisoft. This project is an unofficial compatibility fix and is not affiliated with or endorsed by them.

The configuration files and documentation in this repository are released under **MIT License** — use, modify, and redistribute freely.

---

## 中文说明

**问题**：Ground Control II（2004 年的老 RTS）在 Windows 11 + 高刷新率显示器上黑屏卡死、
出现竖条花屏、鼠标无法操作、画面缩在左上角一块。

**方案**：用 **DXVK** 转译层接管游戏的 Direct3D 9 调用，配合一份调优的 `dxvk.conf`。
Windows 会优先加载程序目录内的同名 DLL，所以只需把 `d3d9.dll` / `dxgi.dll` / `dxvk.conf`
放进游戏目录即可，**游戏本体一个字节都不改**。

**安装**（详见 `03-操作步骤-读者版.md`）：

1. 从 https://github.com/doitsujin/dxvk/releases 下载 DXVK（如 `dxvk-3.1.1.tar.gz`）
2. 解压，取 **`x32`** 目录里的 `d3d9.dll` 和 `dxgi.dll`
3. 复制到游戏目录（与 `gcii.exe` 同级），再加上本仓库的 `dxvk.conf`
4. 直接从 Steam 点「开始游戏」
5. 启动时弹出的硬件检测对话框点「否 / 取消」，正常进入游戏

**效果**：进游戏 ✅ / 不花屏 ✅ / 画面铺满 ✅ / 鼠标正常 ✅

**已知限制**：游戏内分辨率下拉框仍然只有 5 项低分辨率（`640x480` ~ `1024x768`），
这是游戏自身只读列表前 5 项导致的引擎硬限制，与 DXVK 配置无关。
但**渲染区域是正确的全屏**，画面会被缩放到铺满显示器。

**卸载**：删掉游戏目录里的 `d3d9.dll`、`dxgi.dll`、`dxvk.conf` 三个文件即可完全还原。

**⚠ 必须按自己的硬件改两处**（详见 `07-通用化改造指南-含提示词.md`）：

```ini
d3d9.forceRefreshRate = 300     ← 改成你显示器的最高刷新率
dxvk.deviceFilter = "NVIDIA"    ← 改成你的显卡厂商（AMD / Intel）
```

**注意**：DXVK 的二进制文件受 zlib 许可，本仓库不重新分发，请从官方 releases 下载。

**致谢补充**：社区 `dinput` 补丁（修无线 USB 启动崩溃 + 滚轮惯性缩放）出自
[Vegasq/Ground-Control-2-wireless-fix](https://github.com/Vegasq/Ground-Control-2-wireless-fix)（**MIT 协议**，
作者 Mykola "Nick" Yakovliev），与本修复无关、本仓库不分发，详见 `docs/licensing-notes.md` 第四节
（含 SHA-256 校验记录）与 `docs/LICENSE-Vegasq-inertia-fix.txt`。
