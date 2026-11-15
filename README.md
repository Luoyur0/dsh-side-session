# dsh-side-session

DSH（DeepSeek Harness）**临时会话**插件：基于**当前主对话上下文 + agent 触及的文件**，在**独立悬浮窗**里发起**不污染主会话**的临时追问。

> 本仓库是 [hzhz314159/dsh-side-session](https://github.com/hzhz314159/dsh-side-session) 的派生版本，在 **DSH 0.2.2-rc.1 官方内核**上做了兼容性修复与性能优化，上游 0.3.0 的全部功能保留。功能规格见 [SPEC.md](SPEC.md)，变更明细见 [CHANGELOG.md](CHANGELOG.md)。

## 功能

- **独立悬浮窗**（无侧栏停靠形态）：`position:fixed` 可拖拽移动、右下角缩放手柄；启动不自动弹出，由左下角 💬 图标 / `Ctrl+Shift+S` / `/side-session` 斜杠命令唤起。
- **自动导入主对话上下文**：打开即加载当前会话的对话记录 + 触及文件（解析会话日志），主对话变化时事件驱动缓存失效 + 2s 轮询刷新。
- **实时显示当前会话模型**：浮窗头部与上下文卡展示当前会话实际使用的 provider/model，并标注 DSH 当前默认。
- **上下文长度三档**：标准 / 加长（默认）/ 完整，仅当数值变化时才拉取全量上下文。
- **流式回答 + Stop 按钮**：回答可中止；浮窗隐藏时暂停轮询。
- **三种回答引擎**：复用 DSH 全局 Key（按供应商感知）/ 插件自带 Key / 走宿主 `ctx.llm.stream`。
- 主界面同款 UI（`--dsw-alias-*` 设计令牌，深浅主题自动跟随）。

## 安装

从 git 安装（`dsh` 版本需自带 `@deepseek-ai/schemastery`，官方 profile 顶层已 hoisted）：

```bash
dsh plugin --profile <name> add https://github.com/Luoyur0/dsh-side-session
```

手工安装：

1. 复制本仓库到 `~/.dsh/profiles/<name>/node_modules/@dsh-external/dsh-side-session/`
2. 在 profile `package.json` 的 `dsh.profile.bundles` 末尾登记 `@dsh-external/dsh-side-session`
3. `pnpm-lock.yaml` 同步该依赖条目（手工装时不要直接跑 `pnpm install`，会 prune 锁外的包）
4. 重启 DSH

不需要往 profile `cordis.patch.yml` 另加条目：包内 `cordis.patch.yml` 已由 `dsh.bundle.patch` 注入，重复登记会撞 duplicate loader id。

## 兼容性

- 验证环境：DSH 0.2.2-rc.1（Electron 44.7.0 / Node 26.13.0），Windows。
- 会话日志文件名与目录结构属于平台实现细节，内核升级后可能需要再适配。

## 设置

官方内核不提供 `ctx.settings.register`，插件注册的设置节不会出现在设置面板，客户端配置走内存默认值（重启回默认）。需要固定配置时，写 profile `cordis.patch.yml` 中该插件 entry 的 `config`，`apply(ctx, config)` 会读取。

## 文件结构

```
package.json        # 含 dsh.bundle / dsh.client.inject 声明
dsh.plugin.json     # 插件描述
cordis.patch.yml    # 服务端加载（insert 块）
lib/index.js        # 服务端：设置节 + /context + /ask 流式代理 + 日志解析 + 事件缓存失效
lib/client.js       # 浏览器 bundle：Store + 悬浮窗 + 拖拽/缩放 + 三模式 + 流式 UI（零构建）
SPEC.md             # 功能规格
```

## 安全与限制

- 路由仅限回环（`isLoopback`）；API Key 走 settings secret role。
- 大文件注入前先 `stat`，二进制跳过、超大文件只读前 N KB。
- 「浮窗」为页内可拖拽自由浮层（`position:fixed`），非独立 OS 窗口。

## 相对上游 0.3.0 的改动

| 问题 | 上游 0.3.0 | 本版 |
| --- | --- | --- |
| 客户端启动 | `inject` 声明 `settingsScope`（EAC 分叉内核专有服务），官方内核无此服务 → 客户端 entry 永久 pending，web boot 抛 `1 entry did not activate`，桌面端起不来 | `inject` 为 `["slots", "commandUi"]`；配置回落到内存默认值 |
| 上下文卡空态 | 读 `sessions.list` 快照的 `current` 字段，官方 `SessionListState` 只有 `ids/byId/phase/projectionsBySession` → 恒为空，显示「未检测到会话」 | 三级解析：`uiSession.adapter.current` → 快照 `current` → `byId` 中 `retainedBy.mainView > 0` 者（按 `updatedAt` 取最新） |
| 会话日志识别 | 只认 `session.jsonl.zstd` | 兼容 `session.jsonl.zstd` 与 `session.vN.jsonl.zstd` |
| 工具事件与文件统计 | 只认 `tool/code-dispatch*`、只取 `args.file_path` | 兼容 `tool/ptc-dispatch*`；路径取 `file_path/filePath/path`、`files[].path`、`*path`；工具名覆盖 `ops_read_files/ops_inspect_local_file/ops_write_file/ops_edit_file/ops_grep/ops_glob` |
| 依赖 | `import` 裸 `schemastery` | `@deepseek-ai/schemastery`（必须与宿主同一个 Schema 类；`dependencies` 置空，复用 profile 顶层 hoisted 的那份） |
| 会话文件定位 | 每次全树扫描 | 直探 `<sessions>/<工作区>/<sessionId>/`，未命中再全树扫描；空结果不写缓存 |

## 许可

MIT，见 [LICENSE](LICENSE)。原始版权归 [hzhz314159](https://github.com/hzhz314159)，派生修改部分 © Luoyur0。