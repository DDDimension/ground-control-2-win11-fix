# 通用化改造指南 + 提示词

> **用途**：让**任何**遇到同类问题的玩家，用自己的硬件配置和文件路径，
> 生成一份适配自己电脑的 DXVK 修复方案。
>
> 分两部分：
> - **第一部分**：给人看的操作指南（自己手动改要改哪几处）
> - **第二部分**：给 AI 看的提示词（复制进 ChatGPT / Claude / WorkBuddy 等，让它帮你定制）

---

# 第一部分 · 通用化改造指南（人工版）

## 本方案为什么需要"改造"？

原始配置是针对**特定硬件**调优的。换一台电脑，只有 **2 处**必须改，
其他都可以沿用。

| 配置项 | 原值 | 为什么要改 | 不改的后果 |
|---|---|---|---|
| `d3d9.forceRefreshRate` | `300` | 必须等于**你自己显示器的最高刷新率** | 花屏，或掉到低分辨率 |
| `dxvk.deviceFilter` | `"NVIDIA"` | 必须等于**你自己显卡的厂商** | 有核显的机器可能选错 GPU |

**其余 17 项全部通用，不用动。**

---

## 第 1 步：查出你显示器的最高刷新率

### 方法 A：系统设置（最简单）

1. 桌面右键 → **显示设置**
2. 拉到最下面 → **高级显示**
3. 看「**选择刷新率**」下拉框里的**最大值**

例如显示 `300 Hz` → 就把配置改成 `d3d9.forceRefreshRate = 300`

### 方法 B：命令行

PowerShell 里跑：

```powershell
Get-CimInstance -ClassName Win32_VideoController |
  Select-Object Name, CurrentHorizontalResolution, CurrentVerticalResolution, CurrentRefreshRate
```

### 方法 C：听显卡控制面板的

- NVIDIA：右键桌面 → NVIDIA 控制面板 → 更改分辨率 → 刷新率下拉框看最大值
- AMD：Adrenalin → 显示 → 看当前刷新率
- Intel：Intel 显卡命令中心 → 显示

> ⚠️ **关键**：填**显示器实际支持的最高值**，不要填一个它不支持的值。
> 填错会导致花屏或黑屏。

---

## 第 2 步：查出你显卡的厂商

设备管理器 → **显示适配器**，看你用的是哪个：

| 显示的名字 | `dxvk.deviceFilter` 填 |
|---|---|
| `NVIDIA GeForce ...` | `"NVIDIA"` |
| `AMD Radeon ...` / `Radeon RX ...` | `"AMD"` |
| `Intel ... Graphics` / `Intel Arc ...` | `"Intel"` |

### 特殊情况：只有独显，没有核显

如果你的显示适配器里**只有一项**，可以**整行删掉**，DXVK 会自动用唯一那块：

```ini
# dxvk.deviceFilter = "NVIDIA"      ← 整行删掉，或注释掉
```

### 特殊情况：有核显 + 独显（笔记本 / 带核显的台式机）

**必须保留这一行**，否则游戏可能选到没有 3D 能力的核显 → 黑屏。

---

## 第 3 步：改配置

打开 `dxvk.conf`，找到这两行改掉即可：

```ini
d3d9.forceRefreshRate = 300        ← 改成你的最高刷新率，如 144 / 165 / 240
dxvk.deviceFilter = "NVIDIA"       ← 改成 "AMD" 或 "Intel"
```

**改完保存，就完成了。**

---

## 第 4 步（可选）：处理虚拟显示器

**如果你的机器装过远程桌面 / 投屏 / 虚拟显示软件**（各种"远控""投屏""虚拟屏"工具），
它们会注册"虚拟显示器"适配器。老游戏枚举适配器时可能选到这些**没有 3D 能力**的虚拟设备 → 黑屏。

**检查方法**：设备管理器 → 显示适配器，看有没有带这些关键词的条目：

```
Virtual Display ...
<某个远控软件名> Device
<某投屏软件名> Display Adapter
```

**不用卸载它们**，只要 `dxvk.deviceFilter` 填对了（指向真显卡），DXVK 就会跳过它们。

如果填对了还黑屏，再考虑临时禁用虚拟显示器测试。

---

## 第 5 步（可选）：如果 `forceRefreshRate` 怎么调都花屏

说明你的显示器在游戏请求的那个刷新率档位上有 DP 链路问题。试试：

1. **换成显示器上的另一个 DP 接口**（有些接口带宽不同）
2. **换一根质量好的 DP 线**（老线跑高刷容易花屏）
3. **把刷新率往下调一档**：

```ini
d3d9.forceRefreshRate = 144     ; 如果 240 花屏
d3d9.forceRefreshRate = 120     ; 如果 144 也花屏
```

4. **绝对不要留 60** —— 60Hz 是最常见的花屏档位

---

## 改造检查清单

- [ ] 查到自己显示器的最高刷新率
- [ ] 查到自己的显卡厂商
- [ ] 改 `d3d9.forceRefreshRate` 为自己的刷新率
- [ ] 改 `dxvk.deviceFilter` 为自己的显卡厂商（或整行删掉如果只有独显）
- [ ] 确认没有装虚拟显示器干扰（装了也没关系，填对 filter 即可）
- [ ] 从 DXVK 官方下载 **x32** 的 `d3d9.dll` + `dxgi.dll`（不是 x64！）
- [ ] 三个文件放进游戏目录（与 `gcii.exe` 同级）
- [ ] 从 Steam 正常启动
- [ ] 启动弹窗点「否」

---

# 第二部分 · 提示词（复制给 AI 用）

> 把下面整段复制给 ChatGPT / Claude / WorkBuddy / 任何 AI 助手。
> AI 会引导你查出自己的配置并生成专属的 `dxvk.conf`。

---

## 提示词 A：标准版（推荐）

```
我要修复一个老游戏的兼容性问题，请帮我生成适配我电脑的 DXVK 配置文件。

【背景】
- 游戏：Ground Control II（Steam AppID 254840），2004 年的 32 位 DirectX 9 RTS
- 系统：Windows 11
- 症状：黑屏卡死 / 竖条花屏 / 鼠标不能动 / 画面缩在左上角一块
- 方案：用 DXVK 转译层接管 D3D9，把 d3d9.dll + dxgi.dll + dxvk.conf 放进游戏目录
- 硬约束：绝对不能修改游戏本体文件

【我需要你做的】
1. 先问我几个问题，把我电脑的关键信息问清楚：
   - 显示器最高刷新率是多少 Hz（比如 60 / 144 / 165 / 240 / 300）
   - 显示器分辨率是多少（比如 1920x1080 / 2560x1440 / 3840x2160）
   - 显卡是 NVIDIA / AMD / Intel 的哪一家，型号是什么
   - 有没有核显（即"显示适配器"里是不是有两项）
2. 根据我的回答，生成一份完整的 dxvk.conf
3. 逐项告诉我每个配置是干什么的，哪些是必需项
4. 如果我的显示器刷新率不是 300Hz（示例值），务必把 d3d9.forceRefreshRate 改成正确的值
5. 如果我有核显，务必保留 dxvk.deviceFilter 并填对我的显卡厂商

【已知的参考配置（来自一个 300Hz 高刷 + NVIDIA 的参考环境，你需要按自己的硬件改前两项）】
d3d9.forceRefreshRate = 300
d3d9.modeCountCompatibility = False
d3d9.enumerateByDisplays = True
dxvk.enablePresentTiming = False
d3d9.presentInterval = -1
d3d9.maxFrameRate = 60
dxvk.tearFree = False
dxvk.allowFse = True
d3d9.deferSurfaceCreation = False
d3d9.lenientClear = True
d3d9.maxFrameLatency = 0
dxvk.deviceFilter = "NVIDIA"
dxvk.hideIntegratedGraphics = False
d3d9.supportX4R4G4B4 = True
d3d9.supportDFFormats = True
d3d9.floatEmulation = Auto
d3d9.shaderModel = 3
d3d9.dpiAware = True
d3d9.textureMemory = 100
d3d9.deviceLocalConstantBuffers = Auto

【重要提醒】
- 只改必须改的（forceRefreshRate 和 deviceFilter），其他别乱动
- 不要建议我修改游戏本体文件
- 不要建议我改 Steam 配置或系统环境变量，我习惯直接从 Steam 点开始游戏
- 如果我要的效果做不到，直接告诉我，不要编造

开始吧，先问我问题。
```

---

## 提示词 B：给别人分享用（生成通用化文档）

```
我要把我做的一个老游戏修复方案分享到 GitHub 和 Steam 社区。
但我的方案是针对我自己的硬件调优的，别人直接用可能不合适。

请帮我写一份「通用化改造指南」，让任何玩家都能改成适配自己电脑的版本。

【我的原始方案】
- 游戏：Ground Control II（Steam AppID 254840），2004 年的 32 位 DirectX 9 游戏
- 修复方式：DXVK 转译层（把 d3d9.dll + dxgi.dll + dxvk.conf 放进游戏目录）
- 硬约束：不修改游戏本体任何文件
- 新增文件：d3d9.dll、dxgi.dll（从 DXVK 官方下载）、dxvk.conf（我写的）

【我针对的硬件】（示例，请替换成你自己的）
- 显示器：<填你的分辨率和刷新率，如 1920x1080 @144Hz>
- 显卡：<填你的显卡，如 NVIDIA RTX 3060>
- 系统：Windows 11

【配置文件里的关键项】
d3d9.forceRefreshRate = 300      ← 针对刷新率，别人必须改
dxvk.deviceFilter = "NVIDIA"     ← 针对显卡厂商，别人可能必须改
（其余 18 项通用）

【我要的文档内容】
1. 明确列出「哪些配置项是通用的、哪些是必须按自己硬件改的」
2. 教读者怎么查自己显示器的最高刷新率（给 2~3 种方法）
3. 教读者怎么查自己显卡厂商，以及"只有独显"和"有核显+独显"两种情况分别怎么写
4. 给出常见硬件组合的配置示例（NVIDIA 144Hz / AMD 165Hz / Intel 核显 60Hz 等）
5. 如果怎么调都花屏，给出排查步骤
6. 最后给一个自检清单

【格式要求】
- 用中文写
- 用表格和步骤，不要大段文字
- 所有路径用通用写法（如 "你的游戏目录"），不要出现我的个人路径
- 不要出现我的 Steam AppID 以外的个人标识信息

开始写。
```

---

## 提示词 C：极简版（只想快点拿到配置）

```
老游戏 Ground Control II 在我电脑上黑屏/花屏/鼠标不能用，
我想用 DXVK 修复。我的配置是：

- 显示器刷新率：___ Hz
- 显示器分辨率：___ x ___
- 显卡：___（NVIDIA / AMD / Intel）
- 有没有核显：有 / 没有
- 系统：Windows 11

请给我一份 dxvk.conf，只改必须改的项，
不要建议我改游戏本体文件或 Steam 设置。
```

---

## 提示词 D：让 AI 帮我做审计报告

```
我按 DXVK 方案修好了一个老游戏，现在要分享到 GitHub 和 Steam 社区。
别人可能会怀疑"这算不算修改游戏本体"。请帮我说清楚。

【我的情况】
- 游戏：Ground Control II，我通过 Steam 安装
- 游戏目录：<填你的路径，如 D:\Steam\steamapps\common\Ground Control II>
- 我新增的文件：d3d9.dll、dxgi.dll（从 DXVK 官方下载）、dxvk.conf（我自己写的）
- 我没有修改、删除、替换任何游戏原装文件

【请你做】
1. 解释清楚「为什么"DLL 放游戏目录"不算修改游戏本体」
   —— 从 Windows DLL 加载顺序的角度讲
2. 给我 3~5 种「向别人证明游戏本体没被改」的方法，要具体可操作
3. 教我怎么生成一份完整的审计报告：
   - 需要记录哪些数据（时间戳、文件大小、MD5）
   - 用什么命令获取这些数据（Windows PowerShell 和 Git Bash 两种）
   - 报告应该包含哪些章节
4. 顺便给我一段「在 Steam 社区被质疑时的标准回复」

【格式要求】
- 中文
- 命令要能直接复制运行
- 不要在文档里写入我的真实路径，用占位符代替

开始。
```

---

## 提示词使用提示

| 你的情况 | 用哪个 |
|---|---|
| 自己要用，想生成配置 | 提示词 A（标准版） |
| 要分享给别人，需要通用文档 | 提示词 B |
| 只想快点拿到配置 | 提示词 C（极简版） |
| 被人质疑"改游戏了" | 提示词 D |

**用完记得**：AI 生成的配置，**先备份原文件再替换**，改完实际跑一遍确认有效。

---

## 常见硬件配置示例

下面是几种典型组合的配置片段，可以直接参考：

### NVIDIA 独显 + 144Hz

```ini
d3d9.forceRefreshRate = 144
dxvk.deviceFilter = "NVIDIA"
```

### NVIDIA 独显 + 240Hz

```ini
d3d9.forceRefreshRate = 240
dxvk.deviceFilter = "NVIDIA"
```

### AMD 独显 + 165Hz

```ini
d3d9.forceRefreshRate = 165
dxvk.deviceFilter = "AMD"
```

### Intel 核显（轻薄本）+ 60Hz

```ini
d3d9.forceRefreshRate = 60
dxvk.deviceFilter = "Intel"
```

> ⚠️ 注意：如果 60Hz 下出现花屏（这是本作在部分硬件上的已知问题），
> 试试把显示器切到更高的刷新率，或换一根视频线。

### 只有一块独显（没有核显）+ 60Hz 显示器

```ini
d3d9.forceRefreshRate = 60
# dxvk.deviceFilter 整行删掉
```

### 笔记本（Intel 核显 + NVIDIA 独显）+ 165Hz

```ini
d3d9.forceRefreshRate = 165
dxvk.deviceFilter = "NVIDIA"      ← 必须保留，否则可能选到核显
```
