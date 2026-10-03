# os-memory · 跨应用共享记忆系统的主作品

> **OctoSense 黑客松参赛项目 · 系统应用赛道**
> 这不是 demo，这是一个**内置到设备的系统应用**。

> ℹ️ **本仓库是被动 bundle**——`bundle/main.splash` 是纯脚本 + 数据；不安装 hook / 不触发浏览器跳转 / 不发起任何 HTTP 调用。你看到 `vscode.dev/github/...` 这类链接是被你本地 IDE / GitHub 扩展 / 浏览器插件打开的，不是本仓库干的。

---

## 一句话

`os.memory` 是一个 `os.*` 系统应用——**不通过商店分发**，而是 ship with the device；提供 pinned memory（个人偏好/事实）+ 导出能力，**作为跨应用记忆的汇聚点**。

> 📘 完整方案与三仓联合演示见 [`docs/JOINT-DEMO.md`](docs/JOINT-DEMO.md)

---

## 角色

在三仓架构里：

```
            ┌─────────────────────────────────────────┐
            │     os.memory （本仓 · 系统应用）          │
            │   pin · export · 跨 peer 读               │
            └────────▲────────────────────▲────────────┘
                     │ 写                 │ 读
   ┌────────────┐    │                    │    ┌────────────┐
   │memory-notes│────┘                    └─────│memory-digest│
   │  （写端）   │                              │  （读端）    │
   └────────────┘                              └────────────┘
```

两个 demo（notes / digest）通过 `octos.turn.start` 写入自己 peer；本应用通过 `octos.session.history` 读取多 peer 内容并**汇聚展示**——这是"跨应用共享"的物理基础。

---

## 本仓特异

| 字段 | 值 |
|---|---|
| 应用 id | `os.memory` |
| 版本 | `0.1.0` |
| 命名空间 | `os.*` 系统应用（保留命名空间） |
| 提交路径 | `card-host --system`（**不进商店**） |
| 资源上限 | `HostLimits::system()` · **64 MiB storage** · 67,108,864 bytes |
| Agent profile | `read-only` |
| Capabilities | `storage` + `octos.session.history` |
| Integrity | `bundle_blake3` 已 stamp（见 `bundle/manifest.json`） |
| Platforms | `windows`（其他平台未验证，**不假装**） |

### 验证证据（实测）

`bundle/manifest.json` 真值（2026-10-02 重算 stamp）：

```
octo check bundle --system-app
os.memory 0.1.0 — PASSED
  [warning] publisher-signature: unsigned
  grants: capabilities {"octos.session.history", "storage"},
          hosts {}, storage 67108864 bytes, agent read-only
```

64 MiB = 67,108,864 bytes = `HostLimits::system()` 的精确额度。

> `.local-state/card-host.log` 是更早一轮（cap 砍前）跑出来的，capabilities 数对不上当前 manifest；上面的 `hub check` 输出是当前真值。

---

## 关键决策（锚定）

完整决策清单与 jsonl 时间戳见 [`docs/JOINT-DEMO.md` § 3](docs/JOINT-DEMO.md)。本仓最相关：

- **A2.2 不做 `memory.*` 原生宿主服务**——技术上确实更强（`mail.*` 是样板），但本轮做这个是负分
- **A4.2 系统应用 gate gap**——`hub check/scan` 无条件拒 `os.*` id（store 上架语义），但 `card-host --system` 入口是对的；上游 PR #59 已补
- **A1.1 诚实边界**——`platforms: ["windows"]`，不假装全平台

---

## 演示与复现

### 截图

- `bundle/screenshots/01-main.png` — 主屏（4 条 demo pin memories）
- `build/first.png` / `build/demo-memory.png` — 历史快照

### 复现命令

```bash
git clone https://github.com/Thneoly/os-memory.git
cd os-memory

# 用 card-host --system 跑（64 MiB）
card-host bundle --system

# 或自检（用临时 store id 绕过 gate，验证完改回 os.memory）
octo check bundle
```

### 7 问审核答案

见 [`build/REVIEW-ANSWERS.md`](build/REVIEW-ANSWERS.md)（7 问逐项作答，含诚实自报：publisher 占位 / manifest 未签 / os.* 走 card-host --system 而非 hub check 提交流程）。

---

## 已知边界

- **Windows only** — 其他平台未验证不假装
- **未签名** — `integrity.signature: null`；系统应用首次可不上签
- **未转 public** — 当前是 fork 仓，赛前必转 public
- **gate gap** — `hub check/scan` 不认 `os.*`（PR #59 已提，本地实测通过）

---

## 项目结构

```
os-memory/
├── README.md            ← 你在这
├── BRIEF.md             ← 简报（screens/actions/data/states/hosts/capabilities）
├── AGENTS.md            ← 跨 agent 开发说明
├── CLAUDE.md / GEMINI.md ← @AGENTS.md 转发
├── docs/
│   └── JOINT-DEMO.md    ← 三仓联合演示文档
├── bundle/
│   ├── manifest.json    ← stamp manifest
│   ├── listing.json     ← store 元数据（publisher 占位待替换）
│   ├── main.splash      ← 主程序（142 行 · pin/export 两屏）
│   ├── screenshots/01-main.png
│   └── assets/icon.svg
├── build/
│   ├── review.json      ← 7 问审核包（hub scan 产物）
│   ├── REVIEW-ANSWERS.md ← 7 问人工答
│   ├── first.png / demo-memory.png
└── .local-state/        ← card-host 运行期数据
```

---

## 提交前 TODO

见 [`docs/JOINT-DEMO.md` § 6](docs/JOINT-DEMO.md)。

---

## 参考

- 比赛规则（覆盖性）：[App Hub docs/PUBLISHING.md](https://github.com/OctoSense-org/OctoSense-App-Hub/blob/main/docs/PUBLISHING.md)
- 参赛流程：[App-Design-Flow README](https://github.com/OctoSense-org/OctoScript-App-Design-Flow/blob/main/README.zh-CN.md)
- 联合演示：[`docs/JOINT-DEMO.md`](docs/JOINT-DEMO.md)

---

*生成于 2026-10-02 · 锚定大会话 `6d0c2850...`（2026-09-29 → 2026-10-02）· mcp memory #30*