# better-deepseek-harness

面向 **Coding 用户**的 DeepSeek Harness 整合包，向 Codex 的风格与功能看齐。
基座 **DSH 0.1.7-rc.2**。

> 社区整合包，非 DeepSeek 官方产品。

## 装什么

一条命令拿到完整的 Codex 式编码工作台：

- **Codex 风格界面** —— 工作区会话树、全局搜索、轮次跳转、会话重命名/归档/分叉
- **代码审查与简化** —— `/review` 调独立子 Agent 审代码，`/simplify` 按 Git 变更范围简化
- **只读旁问** —— 基于当前上下文单独提问，不打断主任务
- **思考强度滑块** —— 推理等级换成 Codex 风格连续滑块，带动效
- **费用与余额** —— 本会话/当日费用、90+ 模型价格、11 家 Coding Plan 额度
- **桌宠** —— DeepSeek 娘鲸鱼女仆，报开工收工与单轮花费
- **上下文守卫** —— AST 压缩、测试日志过滤、token 预算
- **Git 图** —— 分支选择与提交图
- **消息回改** —— 中断后把最后一条消息拉回输入框
- **Computer Use** —— 操控 Windows 原生桌面：UIA 无障碍树观察、截图、鼠标键盘、窗口管理（22 个工具）

## 安装

需要 **DSH 0.1.7-rc.2**（本包 `dshVersion` 精确锁定该版本）。

```bash
dsh --profile better-deepseek-harness
```

或由 DSH 启动器导入 `.dspack`。

## 插件清单（11 个，全部钉死精确版本）

| 插件 | 版本 | 作用 |
|---|---|---|
| `@michengai/dsh-codex-ui` | 1.1.18 | Codex 风格侧栏、工作区会话树、全局搜索、轮次导航 |
| `@michengai/dsh-code-review` | 0.1.7 | `/review` 独立子 Agent 代码审查 |
| `@michengai/dsh-simplify` | 0.1.10 | `/simplify` Git 范围代码简化 |
| `@michengai/dsh-btw` | 0.1.13 | 只读旁问，不打断主任务 |
| `dsh-effort-slider` | 1.2.0 | Codex 风格思考强度连续滑块 |
| `dsh-cost-meter` | 1.7.37 | 费用统计、模型价格、Coding Plan 额度、余额 |
| `dsh-whale-girl-pet` | 0.3.4 | DeepSeek 娘鲸鱼女仆桌宠 |
| `@goodandready/dsh-context-lens` | 0.1.24 | AST 上下文压缩、token 预算守卫 |
| `dsh-plugin-edit-message` | 0.1.5 | 消息回改 |
| `@linxin666/dsh-client-ui-git-graph` | 0.4.2 | Git 分支图 |
| `dsh-computer-use-win` | 0.1.2 | Windows Computer Use 桌面操控（22 工具） |

### Computer Use 说明

`dsh-computer-use-win` 通过 DSH 内置的 `@deepseek-ai/dsh-mcp-client` 桥接一个
MCP stdio 服务器，工具以 `mcp__wincu__windows_computer_use_*` 出现：

- **看**：UIA 无障碍树（`control`/`content`/`raw` 三视图）、窗口裁剪截图
  （PrintWindow → WGC → 屏幕区域三级回退）、OCR 词框
- **做**：鼠标（标准/双击/拖拽/滚动）、键盘、UIA 语义动作、窗口管理
- **安全**：急停 failsafe（鼠标停屏幕角落 500ms 拒所有输入）、前台校验
  fail-closed、identity guard（HWND/PID 变化即拒）、Win 键组合黑名单

零运行时依赖（纯 Node 内置模块 + PowerShell/C# UIA 后端）。

> **本包已修正该插件的 MCP 路径 bug**（见下），否则它在本整合包内无法启动。

## 已知上游 bug 与修正

### `dsh-computer-use-win@0.1.2` 的 MCP 路径解析

该插件自带的 `cordis.patch.yml` 用 `new URL('mcp/server.mjs', baseUrl)` 定位自己的
MCP 服务器，注释里假设 `baseUrl` 是**该 patch 文件所在目录**。但 dsh 实际把
`baseUrl` 设为 **profile 根目录**：

```js
// dsh-app-boot: ctx.baseUrl = pathToFileURL(dirname(absoluteConfigPath)).href + "/"
```

**后果**：单包 profile 下恰好能跑（包目录 ≈ profile 根），但在多插件 profile 里
解析成 `<profile>/mcp/server.mjs` → `MODULE_NOT_FOUND`，MCP 起不来，模型看不到
`mcp__wincu__*` 工具。实测 boot 日志报错即为此。

**本包修法**：在 profile patch 层覆盖该行的 `args`，改用相对 profile 根
（即 `node_modules` 所在处）的路径：

```yaml
- id: mcp-dsh-computer-use-win
  name: "@deepseek-ai/dsh-mcp-client"
  config:
    serverName: wincu
    transport: stdio
    command: !!js process.execPath
    args:
      - !!js "process.getBuiltinModule('node:url').fileURLToPath(new URL('node_modules/dsh-computer-use-win/mcp/server.mjs', baseUrl))"
    toolCallTimeoutMs: 60000
    failOnStartupError: false
```

修复后 boot 日志出现 `windows-computer-use MCP server 0.1.2 ready`。
若上游修好此 bug，可以删掉这条补丁。

## 选型说明

### 兼容性判定

DSH 用 `semver.satisfies(版本, peer范围, { includePrerelease: true })` 校验，所以：

- `>=0.1.0-rc.5 <0.2.0` 这类宽范围 **匹配** rc.2；
- 钉死精确版本（如 `0.1.7-rc.1`）的插件在 rc.2 上会**硬失败**，本包一律不选。

本包 10 个插件的 peerDependencies 全部通过 rc.2 自带的
`evaluatePluginCompatibility()` 实测，10/10 无阻断。

### 冲突排除

- `@michengai/dsh-codex-ui` 会接管官方 `ui-sidebar` 与 `ui-settings-general`
  并自行重建；它同时**重新提供** `sidebar.footer.action` 与 `settings.section`，
  因此 `dsh-cost-meter` 的余额卡片和桌宠的设置分区仍有落点。
- 桌宠走浮动浮层，不争抢侧栏插槽。
- **没有**装 `dsh-better-sidebar` / `dsh-coding-sidebar`——与 codex-ui 抢同一侧栏位置。
- **没有**装 `dsh-prompt-history`——↑↓ 召回历史输入已由 codex-ui 自带。
- 桌宠只选 1 个（候选池 10+ 个全部抢同一浮层）。
- 层栈中 codex-ui 排在**最后**，确保它的插槽接管生效于其他插件注册之后。

## 验证

| 测试 | 结果 |
|---|---|
| 规格校验（pack-structure v3 + manifest v5 硬约束） | 30/30 PASS |
| `evaluatePluginCompatibility()` 实测 | 11/11 无阻断 |
| `pnpm install` 全量解析 | 成功，11 插件就位 |
| 模拟导入 → `dsh --dump-config` | exit 0，1303 行，零 stderr |
| 真实启动 web 服务 | 成功监听，插件正常初始化 |
| Computer Use MCP 服务器启动 | `windows-computer-use MCP server 0.1.2 ready` |
| 插件 MCP self-test（独立验证） | UIA 树 + 截图均 OK（2560×1600） |

## 自定义

profile patch 层 `overrides/cordis.patch.yml` 为空数组 `[]`——挂载关系已由各插件
自身的 bundle patch 完整表达。改动后请跑：

```bash
dsh --profile better-deepseek-harness --dump-config
```

> patch 对已存在条目是**逐键覆盖**而非深合并：只带 `config:` 不会清掉 `disabled`，
> 但写了 `disabled:` 会整体替换。

## 许可与致谢

本整合包仅为插件组合与配置，不含上述插件的源码。各插件版权归其作者所有：

- [MichengAI](https://github.com/MichengAI)（Codex UI / Code Review / Simplify / BTW）
- [goodandready](https://www.npmjs.com/~goodandready)（Context Lens）
- [linxin666](https://www.npmjs.com/~linxin666)（Git Graph）
- `dsh-effort-slider`、`dsh-cost-meter`、`dsh-whale-girl-pet`、`dsh-plugin-edit-message` 各作者

格式规范：[DSH-PackForge](https://github.com/DSH-PackForge/DSH-PackForge)。
