<div align="center">

[English](README.md) · **简体中文**

# TARSBot

**一个拥有自己电脑的常驻 AI 队友。**

它有自己的 Linux 桌面、浏览器和 shell，对话永不中断，危险操作先问你。
你可以随时围观、接管，或者让它按时自己干活。

![Node ≥ 22](https://img.shields.io/badge/node-%E2%89%A522-339933)
![Pi Durable 1.4.0](https://img.shields.io/badge/Pi%20Durable-1.4.0-6c5ce7)
![Self-hosted](https://img.shields.io/badge/self--hosted-127.0.0.1-informational)

<img src="docs/images/ui-hero.png" alt="TARSBot：左边是 Bot 的对话与结论，右边是它正在操作的 Linux 桌面" width="100%">

</div>

---

大多数 AI 助手是一个对话框：关掉页面，它就忘了你；让它查个资料，它只能凭训练数据回答。

TARSBot 换了个思路——**给 AI 一台自己的电脑，再给它一条不会断的对话**。你把活交给它，它在自己的桌面里开浏览器、跑命令、写文件，做完告诉你结果；你可以看着它做，也可以随时插手。

## 它能做什么

### 🖥️ 它有自己的电脑
每个 Bot 都能用一台容器化的 Linux 桌面（XFCE）：浏览器、终端、文件系统、登录态都在里面。它能点按钮、填表单、读网页，也能直接跑 shell 和读写文件。多个 Bot 共享同一台电脑，登录一次，大家都能用。

### 💬 一条永不断的对话
每个 Bot 只有一条持久对话。服务重启、进程被 `kill -9`、容器重启，它都会从中断的地方接着干，而不是从头再来。对话太长时可以压缩；也可以随时搜索它的完整历史。

### 👀 你始终在场
- **围观**：网页右侧实时显示它的桌面，你的光标和它的光标互相独立。
- **接管**：需要你登录、过验证码的时候，直接在桌面上动手，做完交还给它。
- **纠偏**：它忙的时候，你可以插话改方向（steer）、排一条后续任务（queue），或者直接中止。

### 🛡️ 危险操作先问你
`rm -rf`、`git push`、会改动远端的 `curl`、格式化磁盘、关机，以及 `send_*` / `deploy*` 这类工具，都会先停在审批条上等你点头。批准的结果会跟着任务一起存下来，重启后不会重复问。

### ⏰ 按时自己干活
创建定时例程（routines），让 Bot 每隔一段时间自己执行一条指令。例程跨重启存活，错过的时间点只补一次，不会一下子连发。

### 🌿 从任意一句话分叉
对话里任何一条消息都能 **Fork**：新 Bot 继承分叉点之前的全部历史，可以换模型，也可以设成只读（只保留读取类工具）。适合“这个方向再试一次”，又不想弄脏原来的对话。

### 🔎 能上网查资料
内置网页搜索（Exa），查最新信息、文档和事实，不用靠模型记忆。

## 看它工作

每个动作在聊天里只占一行。回合结束后，中间过程自动折叠成一行 **Worked for 2m 45s**，只留下结论；展开能看到每一步，改过的文件会汇总成一张带 diff 的卡片。

<img src="docs/images/ui-details.png" alt="展开后的工具调用：命令、输出和文件 diff 各按类型渲染" width="100%">

<sub>上图为组件画廊（`web/gallery.html`）里的示例数据，用来集中展示各类工具的渲染效果，不是一次真实运行。</sub>

不同工具有各自的展示：Shell 显示命令与输出，Edit / Write 显示彩色 diff，网页搜索显示可点击的结果列表，桌面截图显示缩略图，点开可放大。

## 我们实际验证过什么

这些都有可重复的脚本和记录，不是口号：

| 验证项 | 结果 | 怎么跑 |
|---|---|---|
| 长任务进行中 `kill -9` 服务，重启后任务继续完成 | ✅ 通过 | `scripts/kill-test.sh` |
| 重启 Space 容器后，Bot 继续执行命令 | ✅ 通过 | 同上 |
| 服务和容器同时被杀，恢复后状态一致、能继续接任务 | ✅ 通过 | 同上 |
| 重启容器后，文件和 Chromium 登录态（cookie）保持不变 | ✅ 通过 | 同上 |
| 危险命令停在审批、批准后只执行一次 | ✅ 有测试 | `scripts/approval-test.sh` |
| 定时例程跨重启仍然触发 | ✅ 有测试 | `scripts/routine-test.sh` |

**桌面操控成功率**（3 个难度：读网页口令 / 表单登录 / 搜索商品并下单；66 次运行，`scripts/gui-bench.mjs`）：

| 模型 | 截图 + 无障碍树（默认） | 仅截图 |
|---|---|---|
| claude-sonnet-5-5 | 11/11 | 10/11 |
| glm-5.3 | 11/11 | 2/11 |
| gpt-5.6-terra | 11/11 | 3/11 |

结论：**给模型无障碍树（`desktop_elements`）是桌面操控稳定的关键**，TARSBot 默认就开着，并让模型优先用元素标识而不是像素坐标。

> 说明：每格只有 3–5 次运行，只能说明量级差异，不是统计结论；任务是我们自己搭的测试页，不代表任意真实网站。完整明细见 [`docs/gui-success.md`](docs/gui-success.md) 和 [`bench/conclusion.md`](bench/conclusion.md)。默认模型 `glm-5.3-flash` 没有单独跑过这组基准。

## 快速开始

前置：Node ≥ 22，以及一个 Docker 运行时（macOS 上用 [OrbStack](https://orbstack.dev)）。

```bash
orb start                                    # 启动 Docker
npm install && (cd web && npm install && npm run build)
scripts/space-up.sh                          # 拉起 Bot 的电脑 tarsbot-space（127.0.0.1:3211）
npm start                                    # 打开 http://127.0.0.1:8080
```

在项目根目录创建 `.env`（不要提交）：

| 变量 | 作用 |
|---|---|
| `TOKEN_ROUTER_BASE_URL` | OpenAI 兼容的模型网关地址 |
| `TOKEN_ROUTER_API_KEY` | 网关密钥 |
| `EXA_API_KEY` | 网页搜索 |

打开页面，点 **New bot**，给它一个任务就行。比如：

> 在你的桌面上用 Chromium 打开 nodejs.org，查一下当前的 LTS 版本和发布日期，把结论写进 `/home/cua/work/node-lts.md`，再告诉我。

### 常用配置

| 变量 | 默认 | 说明 |
|---|---|---|
| `TARSBOT_MODEL` | `glm-5.3-flash` | 新建 Bot 的默认模型；`glm-5.3`、`claude-sonnet-5-5`、`gpt-5.6-terra` 也可选 |
| `TARSBOT_PORT` | `8080` | Web UI / API 端口 |
| `TARSBOT_DATA_DIR` | `data/` | 对话、Bot 配置的存放位置 |
| `TARSBOT_SPACE_RESOLUTION` | `1280x800` | 桌面分辨率；由 `scripts/space-up.sh` 创建容器时读取 |

给已有 Bot 换模型：先停掉服务，再运行 `node scripts/set-model.mjs <bot> <modelId>`。

命令行替代：`npm run repl`。它和服务共用同一个数据库，**两者不能同时运行**。

## 它是怎么搭起来的

- **[Pi Durable](https://github.com/earendil-works/pi/tree/main/packages/durable)**（`@earendil-works/pi-durable`）：持久对话、任务恢复和定时例程的底座。TARSBot 之所以能“断了接着干”，靠的就是它。
- **[Cua](https://github.com/trycua/cua)**（`@trycua/cua`）：桌面与沙箱技术栈。Bot 的电脑是一个跑在本机 Docker 里的 Cua Space（XFCE + 浏览器 + 无障碍树驱动）。
- **TARSBot 自己**：多 Bot 管理、Web 界面、审批、例程、分叉，以及把上面两者接起来的胶水。

后端是纯 ESM 的 Node.js，前端是 React 19 + Vite + Tailwind 4。聊天界面是一套可复用的组件库，位于 `web/src/components/Chat/`（原子组件 / 消息组件 / 按工具分类的渲染器），新增工具的展示只需注册一个渲染器。

## 安全（用之前请先读）

TARSBot 默认是一个**只在本机使用**的工具：

- Web UI / API（8080）和桌面代理（3299）**没有鉴权**，只绑定 `127.0.0.1`。要放到云主机上，必须先加鉴权，并且只通过 Tailscale 或私网访问。
- **不要把 Space 的 3211 端口暴露到公网。**
- 模型密钥只存在于宿主机的服务进程环境里，不会写进 Bot 的电脑。
- Bot 的桌面里登录过的账号，对 Bot 是完全开放的：能用 `bash` 的主体，就拥有所有已登录应用的权限。审批只能拦住规则里列出的操作，不是万能的沙箱，规则见 `src/ext/approval.mjs`。

## 数据在哪里

| 位置 | 内容 |
|---|---|
| `data/alice.sqlite` | 全部对话与状态，**不要删除** |
| `data/bots.json` | Bot 配置 |
| `data/space.json` | Space 地址与访问令牌（权限 600） |
| Docker 卷 `tarsbot-space-home` | Bot 电脑的 `/home/cua`，包含浏览器登录态 |

备份：停服务 → 备份 `data/` → 需要保留登录态时再备份这个卷。

## 开发

```bash
TARSBOT_NO_DOTENV=1 npm test     # 后端单元测试，离线
npm run test:env                 # 执行环境一致性测试，需要 Space 在跑
cd web && npm test               # 前端测试
cd web && npm run dev            # 前端开发；访问 /gallery.html 可一次看遍所有消息与工具的渲染
```

## 文档

- 产品技术规范：[`DurableBot 产品技术规范（ASD-STE100） .md`](DurableBot%20产品技术规范（ASD-STE100）%20.md)
- 系统设计：[`DurableBot 系统设计文档（Pi Durable + Cua Spaces） .md`](DurableBot%20系统设计文档（Pi%20Durable%20+%20Cua%20Spaces）%20.md)
- 开发计划：[`docs/PLAN.md`](docs/PLAN.md)　接口契约：[`docs/CONTRACTS.md`](docs/CONTRACTS.md)

## 已知限制

- Pi Durable 目前处于实验阶段，API 可能变动（依赖已收敛在 `src/env.mjs`、`src/ext/`、`src/server.mjs` 三处）。
- 桌面操控的成功率取决于模型；没有无障碍树的页面（如 canvas、远程桌面）要靠截图，成功率明显更低。
- 存储只允许一个写进程，服务和命令行不能同时开。
- 目前只在 macOS + OrbStack 上完整验证过；其他环境理论上只要能跑 Docker 即可，但没有验证。

## 许可

本仓库暂未附带许可证。依赖的 Cua Spaces 采用 FSL-1.1-MIT：自用与内部部署不受限，**禁止用本产品构建与 Cua 竞争的商业产品**。
