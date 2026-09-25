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

## 安装

需要 **DSH 0.1.7-rc.2**（本包 `dshVersion` 精确锁定该版本）。

```bash
dsh --profile better-deepseek-harness
```

或由 DSH 启动器导入 `.dspack`。

## 插件清单（10 个，全部钉死精确版本）

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
| `evaluatePluginCompatibility()` 实测 | 10/10 无阻断 |
| `pnpm install` 全量解析 | 成功，10 插件就位 |
| 模拟导入 → `dsh --dump-config` | exit 0，1290 行，零 stderr |
| 真实启动 web 服务 | 成功监听，插件正常初始化 |

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
