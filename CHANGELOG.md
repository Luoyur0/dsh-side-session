# 变更记录

## 0.3.2 — 2026-11-16

**修复**

- 侧栏 footer 入口与同槽插件（如 dsh-session-navigator）横向重叠：官方内核把 `sidebar.footer.action` 渲染成横排 flex 行，本插件按整行绘制的按钮（`flex:none` + `width:calc(100% + 8px)`）会撑满并压住前一项。宽态下把该槽容器标成竖排（`data-dss-footer-host`；宿主按 `[data-slot='sidebar.footer.action']` 契约定位，兜底取类名含 `footer` 的 flex row 容器），折叠态维持圆钮；按钮宽态样式收为 `width:100%; margin:4px 0`，避免列内横向溢出。

## 0.3.1 — 2026-11-15

首个在 DSH 官方内核（0.2.2-rc.1）上验证可用的版本。上游 0.3.0 的功能全部保留。

**修复**

- 客户端 `inject` 收敛为 `["slots", "commandUi"]`。`settingsScope` 是 EAC 分叉内核专有的 client 服务，官方内核不提供；声明为必需会让客户端 entry 永久 pending，web boot 失败（`Error: web boot: 1 entry did not activate`），桌面端无法启动。
- 上下文卡改为三级会话 id 解析：`uiSession.adapter.current.getSnapshot().key` → `sessions.list` 快照的 `current` → `byId` 中 `retainedBy.mainView > 0` 且 `updatedAt` 最新者。官方 `SessionListState` 没有 `current` 字段，原实现只能拿到空 id，浮窗停在「未检测到会话」。
- 会话日志文件名兼容 `session.vN.jsonl.zstd`（官方内核实际命名），不再只认 `session.jsonl.zstd`。
- 工具事件兼容 `tool/ptc-dispatch*`（`data.name` + 结构化 `data.arguments`）；路径提取覆盖 `file_path/filePath/path`、`files[].path` 与 `*path` 形态；工具名覆盖 `ops_read_files/ops_inspect_local_file/ops_write_file/ops_edit_file/ops_grep/ops_glob`。
- `schemastery` 改从 `@deepseek-ai/schemastery` 导入，`dependencies` 置空。设置 schema 必须与宿主使用同一个 `Schema` 类，包内不能留第二份副本。

**性能**

- 会话文件改为直探 `<sessions>/<工作区>/<sessionId>/`，命中即返回（实测 2ms，全树扫描约 120ms）。
- 会话文件查找的空结果不再写缓存，避免一次查询失败长期返回空。

**诊断**

- 会话 id 解析失败时把 `uiSession` / `snapshotKeys` / `mainViewCandidates` 写入 `window.__dssSessionDiag`，空态标题的 `title` 属性悬停可看。

## 0.3.0 — 2026-08-18（上游）

模式 1 改为「当前会话供应商感知」，新增实时模型显示。

## 0.2.3 — 2026-08-16（上游）

上下文长度切换改为选择暂存 + 确定按钮；新增 `?meta=1` 轻量轮询端点。

## 0.2.2 — 2026-08-16（上游）

上下文长度三档；默认不自动弹出；事件驱动缓存失效；Stop 按钮；文件读取 `stat` + 二进制跳过。

## 0.2.0 — 2026-08-15（上游）

独立浮窗形态。

## 0.1.0（上游）

初版：右侧自绘面板 + 撕出浮窗 + 三模式引擎。
