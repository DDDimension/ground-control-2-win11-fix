# dxvk.conf 精简版（抄写用）

> 如果你的平台不方便上传附件，可以新建一个 `dxvk.conf`，
> 把这个代码块**整段**粘贴进去，放在游戏目录（与 `gcii.exe` 同级）即可。
>
> 编码用 **UTF-8**，文件名**全小写**。

---

## 精简版（19 项生效配置）

```ini
# Ground Control II — DXVK config (stable)
# Place next to gcii.exe. Game files are not modified.

# ---- refresh rate / mode list ----
d3d9.forceRefreshRate = 300
d3d9.modeCountCompatibility = False
d3d9.enumerateByDisplays = True

# ---- present / mouse fix ----
dxvk.enablePresentTiming = False
d3d9.presentInterval = -1
d3d9.maxFrameRate = 60
dxvk.tearFree = False

# ---- swapchain / fullscreen ----
dxvk.allowFse = False
d3d9.deferSurfaceCreation = False
d3d9.lenientClear = True
d3d9.maxFrameLatency = 0

# ---- GPU selection ----
dxvk.deviceFilter = "NVIDIA"
dxvk.hideIntegratedGraphics = False

# ---- legacy format support ----
d3d9.supportX4R4G4B4 = True
d3d9.supportDFFormats = True
d3d9.floatEmulation = Auto
d3d9.shaderModel = 3
d3d9.dpiAware = True

# ---- memory ----
d3d9.textureMemory = 100
d3d9.deviceLocalConstantBuffers = Auto
```

---

## 每项的作用

| 配置 | 作用 | 能不能删 |
|---|---|---|
| `d3d9.forceRefreshRate = 300` | 强制最高刷新率。**改这里适配你的显示器**（144 / 165 / 240…） | ❌ 关键 |
| `d3d9.modeCountCompatibility = False` | 保持完整模式列表 | ❌ 关键 |
| `d3d9.enumerateByDisplays = True` | 按显示器枚举适配器 | 建议保留 |
| `dxvk.enablePresentTiming = False` | ★ **修"鼠标不能动"** | ❌ 关键 |
| `d3d9.presentInterval = -1` | 立即呈现，避免帧率限制卡顿 | ❌ 关键 |
| `d3d9.maxFrameRate = 60` | 限制 60 FPS（引擎不适配高帧） | 可调 |
| `dxvk.tearFree = False` | 关闭防撕裂（本作会卡） | 建议保留 |
| `dxvk.allowFse = False` | 用无边框全屏窗口模拟全屏（本作实测最稳的模式） | 建议保留 |
| `d3d9.deferSurfaceCreation = False` | 立即创建表面 | 建议保留 |
| `d3d9.lenientClear = True` | 宽松清除（老引擎兼容） | 建议保留 |
| `d3d9.maxFrameLatency = 0` | 最多预渲染帧 | 建议保留 |
| `dxvk.deviceFilter = "NVIDIA"` | ★ 确保用独显（有核显的机器必留） | 建议保留 |
| `dxvk.hideIntegratedGraphics = False` | 不隐藏核显 | 可删 |
| `d3d9.supportX4R4G4B4 = True` | 支持老式 4-4-4-4 格式 | 建议保留 |
| `d3d9.supportDFFormats = True` | 支持老式深度/模板格式 | 建议保留 |
| `d3d9.floatEmulation = Auto` | 浮点仿真 | 可删 |
| `d3d9.shaderModel = 3` | 着色器模型上限 | 建议保留 |
| `d3d9.dpiAware = True` | ★ DPI 感知（缩放屏上窗口尺寸正确） | 建议保留 |
| `d3d9.textureMemory = 100` | 纹理内存配额 | 可删 |
| `d3d9.deviceLocalConstantBuffers = Auto` | 常量缓冲策略 | 可删 |

**最小可用集**（如果只想要能跑）：

```ini
d3d9.forceRefreshRate = 300
d3d9.modeCountCompatibility = False
dxvk.enablePresentTiming = False
d3d9.presentInterval = -1
d3d9.maxFrameRate = 60
dxvk.deviceFilter = "NVIDIA"
d3d9.dpiAware = True
```

---

## 常见改动

### 我是 144Hz 显示器

```ini
d3d9.forceRefreshRate = 144
```

### 我是 AMD 显卡

```ini
dxvk.deviceFilter = "AMD"
```

（如果只有独显，这一行可以删掉）

### 我是 Intel 核显

```ini
dxvk.deviceFilter = "Intel"
```

### 我想试试能不能出更高分辨率选项（不推荐）

```ini
d3d9.forceAspectRatio = "16:9"
```

⚠️ **必须配合 `DXVK_FORCE_WINDOWED=1` 环境变量使用，否则画面会缩在左上角一块、鼠标错乱。**
从 Steam 直接启动时环境变量注入不进去，所以**不建议普通用户尝试**。

---

## 验证配置是否被读取

启动游戏后，打开游戏目录里的 `gcii_d3d9.log`，搜索：

```
Found config file: dxvk.conf
```

下面应该列出你写进去的每一项。例如：

```
info:  Found config file: dxvk.conf
info:    d3d9.forceRefreshRate = 300
info:    dxvk.enablePresentTiming = False
```

**如果显示 `Found config file:` 但没有列出你的项**，说明文件名或路径不对
（检查是否变成了 `dxvk.conf.txt`）。
