# 能分享什么 · 不能分享什么

> 这是发布前**最需要看的一份**。搞错会带来许可风险和社区争议。

---

## 一、一张表看懂

| 内容 | 大小 | 能不能分享 | 许可 / 理由 |
|---|---|---|---|
| `dxvk.conf` | 8,160 B | ✅ **能，这是你的核心成果** | 你自己写的配置，无版权争议 |
| 所有说明文档（README / 教程 / 审计报告） | — | ✅ 能 | 你自己写的 |
| DXVK 官方下载链接 | — | ✅ 能 | 只是超链接 |
| `d3d9.dll`（DXVK） | 7,856,142 B | ❌ **不能** | DXVK 二进制，zlib 许可（允许再分发，但见下） |
| `dxgi.dll`（DXVK） | 5,877,774 B | ❌ **不能** | 同上 |
| 游戏本体任何文件（`gcii.exe` / `*.sdf` / `binkw32.dll` / `mss32.dll` / `goggame.dll`） | — | ❌ **绝对不能** | 受版权保护，属 Massive Entertainment / Ubisoft |
| `dinput.dll` + `dinput_inertia.ini` | 86,528 + 772 B | ✅ **能（MIT，需附署名）** | 来源已确认：[Vegasq/Ground-Control-2-wireless-fix](https://github.com/Vegasq/Ground-Control-2-wireless-fix) v1.1，MIT 协议，见第四节 |

---

## 二、为什么不能打包 DXVK 二进制

严格来说，**DXVK 是 zlib 许可，zlib 允许再分发**（包括二进制），前提是保留版权声明。
所以法律上不是"绝对禁止"。

**但实际发布时不应该打包，理由有三条：**

### 理由 1：版本会过期

DXVK 迭代很快。你今天打包 v3.1.1，三个月后用户装到过时版本，
出问题会回来找你，而你已经在用 v3.2.x 了。**让用户从官方渠道拿最新版**更省事。

### 理由 2：用户容易拿错架构

DXVK 压缩包里同时有 `x32` 和 `x64`。老游戏是 32 位的，必须用 `x32`。
如果你打包，用户无脑解压可能拿错；让他自己去官方页面看，反而会更小心。

### 理由 3：社区观感

Steam 社区和 GitHub 上，**"只发配置、让人去官方下二进制"** 是成熟做法，
会被视为专业。反过来打包一堆第三方 DLL，容易被质疑"这包里塞了什么"。
你的项目最大卖点就是"**干净、不改本体、删掉就还原**"——这个卖点要守住。

---

## 三、正确的做法（照抄）

### ❌ 不要这样做

```
gc2-fix.zip
├── d3d9.dll        ← 不要
├── dxgi.dll        ← 不要
├── dxvk.conf
└── README.md
```

### ✅ 要这样做

```
GitHub 仓库 ground-control-2-win11-fix/
├── README.md                      ← 说明文档
├── dxvk.conf                      ← 你的核心成果
├── 游戏本体完整性审计报告.md        ← 证明不改本体
├── docs/
│   ├── install-guide.md           ← 分步教程
│   └── known-issues.md            ← 已知限制
└── LICENSE                        ← MIT（你的文档 + 配置）
```

README 里写清楚：

```markdown
## Step 1 — Download DXVK

Get it from the official repository:
https://github.com/doitsujin/dxvk/releases

Download `dxvk-3.1.1.tar.gz` (or newer) and extract it.
Take `d3d9.dll` and `dxgi.dll` from the **x32** folder.

> This repository does not redistribute DXVK binaries.
```

---

## 四、关于 `dinput.dll`（鼠标惯性补丁）

**有些玩家的游戏目录里会有这两个文件**（不是本方案的一部分）：

```
dinput.dll              约 86 KB
dinput_inertia.ini      约 1 KB    [Inertia] Enabled=1 DecayRate=80 ...
```

### 来源（已确认，2026-09-27）

- 项目：**[Vegasq/Ground-Control-2-wireless-fix](https://github.com/Vegasq/Ground-Control-2-wireless-fix)**，release **v1.1**（2026-01-04）
- 作者：Mykola "Nick" Yakovliev（Vegasq），开发过程有 Claude（Anthropic）协助
- 协议：**MIT**（保留版权与许可声明即可自由再分发）
- 功能（两个，都与我们无关但有用）：
  1. 修复**无线 USB 设备导致的启动崩溃**（拦截 `EnumDevicesA` / `GetDeviceInfo`）
  2. 滚轮惯性平滑缩放（把滚轮转成 PageUp/PageDown 注入，星际 2 手感）
- 真伪校验：release 官方 SHA-256
  `dinput.dll` = `b870618c...adf86c7`、`dinput_inertia.ini` = `3789d889...18bece`，
  与本地文件比对**完全一致**（未被人篡改过）。

### 分发建议（相比旧版结论已更新）

**可以分发**（MIT 允许），前提：随包附上作者 LICENSE 全文并注明来源仓库。
它仍然与本 DXVK 修复**无关**——是否打包取决于你的发布策略：

1. **不打包**（保守做法）：文档里链接过去，让用户自己下载 → 包更小、责任更清
2. **打包**（省事做法）：附 `LICENSE-Vegasq-inertia-fix.txt` + 在 README 署名来源

### 在文档里"提到"它

在 Known Issues / 可选章节里写一句：

```markdown
### Optional: mouse wheel inertia & wireless crash fix

[Vegasq/Ground-Control-2-wireless-fix](https://github.com/Vegasq/Ground-Control-2-wireless-fix)
(MIT) fixes both the wireless-device startup crash and adds smooth wheel zoom.
It is independent of this DXVK fix; grab it from the link above.
```

---

## 五、关于录屏 / 截图

| 内容 | 能不能发 |
|---|---|
| 你自己截的游戏画面 | ✅ 能（属合理使用，用于说明兼容性问题） |
| 游戏官方宣传图 / 封面 | ⚠️ 尽量避免，用你自己的截图 |
| 短游戏录屏（说明修复效果） | ✅ 能，作为兼容性演示 |

Steam 社区发帖时，**优先用自己截的图**——最安全，也最有说服力。

---

## 六、许可证怎么标

### 你的部分（配置 + 文档）

建议 **MIT License**，最宽松，社区接受度最高：

```
MIT License

Copyright (c) 2026 <你的 GitHub 用户名>

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

### 别人部分（致谢）

在 README 里明确写：

```markdown
## Credits

- **DXVK** by doitsujin — https://github.com/doitsujin/dxvk (zlib License)
  The entire translation layer is their work. This project only provides
  configuration and documentation.
- **Ground Control II** © Massive Entertainment / Ubisoft.
  Unofficial compatibility fix, not affiliated with or endorsed by them.
```

---

## 七、发布前自检清单

发之前逐条打勾：

- [ ] 仓库里**没有** `d3d9.dll`
- [ ] 仓库里**没有** `dxgi.dll`
- [ ] 仓库里**没有**任何游戏本体文件（`gcii.exe` / `*.sdf` / `binkw32.dll` …）
- [ ] 仓库里**没有** `dinput.dll` / `dinput_inertia.ini`（若决定打包：已附 Vegasq 的 MIT LICENSE 全文并署名来源，此条可跳过）
- [ ] README 里**有** DXVK 官方下载链接
- [ ] README 里**有** `x32` 而非 `x64` 的明确提示
- [ ] README 里**有** 致谢段（DXVK + Massive Entertainment）
- [ ] 仓库根目录**有** LICENSE
- [ ] 已知限制**如实写明**（分辨率仍 5 项、启动弹窗）
- [ ] 卸载方法**写明**（删 3 个文件）

**全部打勾 = 可以发布。**
