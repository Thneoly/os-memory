# 跨应用共享记忆系统 · 联合演示

> OctoSense 黑客松参赛项目（系统应用赛道）
> 主作品：**os-memory** · 写端 demo：**memory-notes** · 读端 demo：**memory-digest**

---

## 1. 架构叙事

三仓**不是**三个孤立 demo 的拼盘，而是一个**跨应用共享记忆系统**的端到端实现：

```
              ┌─────────────────────────────────────────────┐
              │            os.memory (系统应用)              │
              │   pin · focused · export · 跨 peer 读取      │
              └────────▲────────────────────────▲───────────┘
                          │ 写入                  │ 读取
   ┌──────────────┐       │                       │       ┌──────────────┐
   │ memory-notes │───────┘                       └───────│memory-digest │
   │  (写端)      │                                       │  (读端)      │
   └──────────────┘                                       └──────────────┘
       ↑ 每条便签的 Remember 按钮                              ↑ Refresh + chips
       ↓ host.request("octos.turn.start", ...)                ↓ host.request(...)
   失败 → "kept locally: <text>"                          失败 → 本地 prefs 拼句
```

### 1.2 关键决策（锚定）

- **不做跨应用 API**（隔离红线 + 原生禁令）。走"各应用 Agent 写自己 peer + 系统 Agent 编排同步 + os.memory 汇聚"
- **不做 `memory.*` 原生宿主服务（Rust + os.\*）**。`mail.*` 是更优样板，但本轮做这个是负分
- **os.memory 是 `os.*` 系统应用**（ship with the device，不进商店）——走 `card-host --system`，`HostLimits::system()` 给到 64 MiB
- **诚实降级是核心 UX**：每个 `host.request` 失败必须有本地降级路径——不能因为 AI 不可用就坏 UI

---

## 2. 三仓角色

| | **os-memory**（主作品） | **memory-notes**（写端） | **memory-digest**（读端） |
|---|---|---|---|
| 命名空间 | `os.*` 系统应用 | 商店应用 | 商店应用 |
| 提交路径 | `card-host --system` · 64 MiB | `octo check` · 16 MiB | `octo check` · 16 MiB |
| 关键交互 | pin / focused / export | 每条便签的 **Remember** | **Refresh** + chips |
| Agent profile | `read-only` | `read-only` | `read-only` |
| 持久化 | `memories.json`（pin/focused/export） | `notes.json` | `prefs.json` |
| AI 失败降级 | 仍是 pin/focused | "kept locally: \<text\>" | "No assistant on this device — from your local interests" |
| README.md | ❌ | ❌ | ❌ |
| Packet | ✅ `build/review.json` + `REVIEW-ANSWERS.md` | ❌ | ❌ |

---

## 3. 关键决策（赛前固化 · 锚定 jsonl 2026-09-29 → 2026-10-02）

### 3.1 · A2.2 不做 memory.\* 原生宿主服务
技术上确实更强——`mail.*` Rust 宿主 + Mail Splash 是这套的样板。但本轮做这个是负分。
**Why**：架构红线 + 原生禁令 + 黑客松时窗 + "更值的去处"。
**How to apply**：任何"做一个 memory.\* 宿主服务"的冲动，先停下。

### 3.2 · A1.1 诚实边界
README 明确"已验证平台是 Apple silicon macOS，Windows/Linux 未验证"。我们在 Windows 上跑，碰到问题如实报。
**Why**：跨平台兼容问题是红线，不是潜规则。
**How to apply**：所有 listing 必须 `platforms: ["windows"]` 起步；不假装全平台。

### 3.3 · A3.1 诚实降级是核心 UX
每个 `host.request` 失败必须有本地降级路径——不能因为 AI 不可用就坏 UI。
**Why**：AI 是渐进增强层，不是核心路径。
**How to apply**：notes 失败时便签"kept locally"；digest 失败时显示本地 prefs；os-memory 不依赖 AI（base, no AI needed）。

### 3.4 · A4.2 系统应用 gate gap
`hub check/scan` 无条件拒 `os.*` id（store 上架语义），但 `card-host --system` 入口是对的——`HostLimits::system()` 已 64 MiB 实证通过。
**Why**：上游 hub 设计不同步（PR #59 已补 `card-host --system` 路径，本地实测通过）。
**How to apply**：os-memory 用临时 `memory-hub` store id 验证 bundle 完整性，验证完改回 `os.*` id。

### 3.5 · A1.3 fork workflow
origin = Thneoly fork（自己仓），upstream = OctoSense-org（无 push 权限）。各仓 local 配 noreply 邮箱。
**Why**：org 权限隔离。
**How to apply**：每个 PR 走 fork → push → upstream。

### 3.6 · A1.4 可见窗口演示
用户屏幕看三个 app 并排跑，远程同步驱动；不盲跑。
**Why**：协作透明、可调试、可截屏。
**How to apply**：评审现场三个窗口同开。

---

## 4. 演示脚本（评审现场）

1. **启动 os-memory**（在 card-host --system 下，64 MiB）
   - 浏览器打开 host port → 看 main.splash
   - 展示 4 条 demo pin memories；可点 grouped → 倒序

2. **启动 memory-notes**（独立运行，16 MiB）
   - 添加一条新便签 → 点 **Remember**
   - 成功：状态行 "Remembered ✅ \<text\>" + AI 摘要
   - 失败：关闭 hub → 再点 → "kept locally: \<text\>"

3. **启动 memory-digest**（独立运行，16 MiB）
   - 添加一个兴趣 → 点 **Refresh**
   - 成功：digest 文本 + "From your assistant"
   - 失败："No assistant on this device — from your local interests"

4. **三个窗口并排**：用户在屏幕直接看，三个交互并跑。

---

## 5. 复现入口

```bash
# 三个仓
git clone https://github.com/Thneoly/os-memory.git
git clone https://github.com/Thneoly/memory-notes.git
git clone https://github.com/Thneoly/memory-digest.git

# 每个仓自带 BRIEF.md + AGENTS.md + bundle/main.splash
# 检查：bundle

bundle
```

每个仓都自带 `BRIEF.md` + `AGENTS.md`。

---

## 6. 提交前必做（赛前 TODO）

- [ ] **替换 listing.json publisher 占位**（3 仓各 3 处：`publisher.name` / `support` / `privacy_policy_url`）
- [ ] **建 README.md**（3 仓根目录）
- [ ] **生成 Packet（review.json）+ 7 问 REVIEW-ANSWERS.md**（notes、digest 两仓）
- [ ] **补 screenshots**（每仓 ≥3 张：空/成功/降级）
- [ ] **3 仓转 public**（赛前必做，否则评审看不到）
- [ ] **memory-notes 的 bundle/manifest.json 新 stamp 提交**（内容已一致，安全 add）
- [ ] **生成 publisher key（ed25519）+ sign-manifest**（首次可 unsigned）
- [ ] **跑 `octo check` 实测 blake3 + bundle size**（memory-digest 当前未实测）

---

## 7. 已知风险与边界

- **Windows only**：3 仓 `platforms: ["windows"]`——其他平台未验证不假装
- **PR #59 待上游合**：hub check/scan 补 system 应用分支（本地 fork `feat/check-system-app` 已实测）
- **App Hub docs/PUBLISHING.md 资源限制**：系统应用 64 MiB storage；default 16 MB；bundle ≤ 8 MB
- **Submit 模板差异**：App Hub docs/PUBLISHING.md 标题 `Submit <App <app id> <version>`（大写 App）；Design-Flow README 是小写 `app`——提交前 double-check 上游最新版

---

## 8. 参考资料

| 文档 | 链接 |
|---|---|
| App Hub 比赛规则（覆盖性） | https://github.com/OctoSense-org/OctoSense-App-Hub/blob/main/docs/PUBLISHING.md |
| 参赛流程 README | https://github.com/OctoSense-org/OctoScript-App-Design-Flow/blob/main/README.zh-CN.md |
| Design-Flow docs/PUBLISHING | https://github.com/OctoSense-org/OctoScript-App-Design-Flow/blob/main/docs/PUBLISHING.md |
| 系统应用赛道首页（UX 概念 demo） | https://octosense.org/cn/apps/system/ |

> 注：`octosense.org/cn/apps/system/` 是 UX 概念 demo，不是规则页。规则在 App Hub docs/PUBLISHING.md（覆盖性）+ Design-Flow docs/PUBLISHING.md（辅助）。

---

*生成于 2026-10-02，黑客松参赛前。锚定大会话 jsonl `6d0c2850...`（2026-09-29 → 2026-10-02）+ mcp memory #30。*