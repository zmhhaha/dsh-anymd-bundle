# dsh-anymd-bundle

一个**组合包（bundle）**，唯一作用是给 DSH 的 profile 插入一条 MCP 服务器配置：
把 [`@sylphx/anymd`](https://www.npmjs.com/package/@sylphx/anymd)（Any file → clean Markdown，
含 PDF）挂成 `mcp__anymd__*` 工具。

## 为什么需要这个壳

DSH 的插件管理器**只管理组合包**——没有 `dsh.bundle.patch` 的普通包在安装前就被拒绝
（见 `panghu_chat/dsh/docs/plugin-install-capabilities.md` §2）。`@sylphx/anymd` 本身是
CLI + stdio MCP server，不是 DSH 插件，所以需要一个壳来承载配置行。

## 装法

三条路都在 **dsh-web 容器**侧生效（插件管理器跑在 host plane）：

1. **Web UI → Plugins 面板**：把本仓库的 Git 地址填进去。
2. **Web UI → Creator 入口开新任务**：那个会话用 `cordis` preset，agent 有
   `tool-plugin-manager`，可以让它装。
   （默认 `standard` 会话里该工具是 `disabled: true`，是 DSH 的设计。）
3. **容器内 CLI**：`dsh plugin --profile web add <git-url>`

装完开**新会话**，工具才会出现。

## 这条配置的要点

- `command: npx` —— 用 npx 自举，不必把 anymd 烘进镜像；首次 spawn 时下载到 HOME 缓存。
- `transport: stdio` —— stdio 服务由 DSH 进程直接 spawn，**跑在 dsh 容器里**，看到的是
  那个容器的文件系统。`--allow-dir=/opt/data` 是它的目录白名单，按需改。
- `failOnStartupError: false` —— 网关/npx 起不来时不会拖垮整个 profile 加载。

## 已知限制

- **stdio 服务看不到项目卷**：agent 的 `bash`/`fs` 被 SSH 重定向到项目容器（`/workspace`），
  而这个 MCP 服务在 dsh 容器里跑。要读的 PDF 必须在 **dsh 容器能看到的路径**下。
  只为"读项目卷里的 PDF"，直接在项目容器里跑 `anymd` CLI 更简单。
- 扫描版/图片型 PDF 需要 `tesseract`，dsh 镜像里没有；有文本层的 PDF 不受影响。
- 开源协议：本壳为 MIT；anymd 自身的许可见其 npm 页面。
