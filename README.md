# Pi Pixso Skill

<p>
  <img src="assets/pixso-full-color.svg" alt="Pixso" width="120">
</p>

面向 [pi](https://github.com/badlogic/pi-mono) 的 Pixso AI 集成 skill 包。安装后，pi 可以读取 Pixso 设计稿生成代码、把网页导入 Pixso、编辑设计稿、检查节点 DSL，以及生成 Design-to-Code 组件解析配置。

本包由 [PixsoLtd/pixso-ai-integration](https://github.com/PixsoLtd/pixso-ai-integration) 适配而来：保留并复用了其中的全部 skill 内容，去掉 Claude Code / Cursor / Gemini 专属的插件脚手架，改为符合 [pi skills 规范](https://github.com/badlogic/pi-mono) 的 package 清单 + `skills/` 目录，并附上 pi 适用的 MCP 配置。

## 前置要求

1. 安装并打开 **Pixso Desktop**，打开需要操作的 Pixso 文件。
2. 启动 Pixso MCP 服务，默认地址：

   ```text
   http://localhost:3667/mcp
   ```

3. 安装 pi（[pi quickstart](https://github.com/badlogic/pi-mono)）。
4. 安装 MCP 桥接扩展 `pi-mcp-adapter`（pi 通过它加载 `.mcp.json`）：

   ```bash
   pi install npm:pi-mcp-adapter
   ```

   安装后重启 pi。

如果 Pixso MCP 地址不是默认值，请同步修改你使用的 MCP 配置文件（见下文）。

## 一、安装本 skill 包

```bash
# 全局安装（写入 ~/.pi/agent/settings.json）
pi install git:github.com/EvenToss/pi-pixso-skill

# 或安装到当前项目（写入 .pi/settings.json）
pi install -l git:github.com/EvenToss/pi-pixso-skill
```

也可以先克隆再用本地路径安装，便于查看源码 / 更新：

```bash
git clone git@github.com:EvenToss/pi-pixso-skill.git
pi install ./pi-pixso-skill
```

更新：

```bash
pi update --extensions          # 更新所有扩展包
pi update git:github.com/EvenToss/pi-pixso-skill   # 只更新本包
```

## 二、配置 Pixso MCP

本包根目录的 `.mcp.json` 就是 Pixso MCP 服务配置：

```json
{
  "mcpServers": {
    "pixso": {
      "type": "http",
      "url": "http://localhost:3667/mcp"
    }
  }
}
```

pi 通过 `pi-mcp-adapter` 读取 MCP 配置，按需选择其中一种方式：

- **项目级（推荐随项目共享）**：在项目根目录放一份 `.mcp.json`（内容同上）。直接克隆本仓库并在其中启动 pi 时，pi 会自动读取根目录的 `.mcp.json`。
- **用户级（全局共享）**：写入 `~/.config/mcp/mcp.json`，所有 pi 会话都能用。
- **pi 专属覆盖**：`~/.pi/agent/mcp.json`（全局）或 `.pi/mcp.json`（项目）。

修改端口或地址后，执行 `/reload` 或重启 pi 生效。可用 `/mcp` 查看 pixso 服务连接状态与可用工具：

```text
/mcp
/mcp reconnect pixso
```

> 提示：pi 默认通过单一 `mcp` 代理工具访问所有 MCP 工具（约 200 token 开销）。如果希望把 Pixso 的常用工具（如 `design_to_code`、`get_node_dsl`、`apply_design`）直接暴露成一等工具，可在配置里给该服务加 `"directTools": true` 或指定工具列表，详见 [pi-mcp-adapter 文档](https://github.com/badlogic/pi-mcp-adapter#direct-tools)。

## 包含的 Skills

安装后，以下 skill 会自动被 pi 发现，并以 `/skill:<name>` 命令形式注册：

| Skill | 命令 | 用途 |
| --- | --- | --- |
| `pixso-design-to-code` | `/skill:pixso-design-to-code` | 把 Pixso 设计节点 / 屏幕 / 组件 / Pixso URL 转成目标框架代码（React、Vue、HTML、Flutter、ArkUI），保存文件、本地化资源、安全清理。 |
| `pixso-code-to-design` | `/skill:pixso-code-to-design` | 把网页 URL、HTML、ZIP、静态产物导入 Pixso 为可编辑设计节点。 |
| `pixso-design-editing` | `/skill:pixso-design-editing` | 在 Pixso 中创建、修改、检查、优化 UI 设计（DSL / apply_design 工作流）。 |
| `pixso-read-dsl` | `/skill:pixso-read-dsl` | 读取紧凑 DSL，递归解析变量 / 样式 / 组件 / 资源引用；或在 `design_to_code` 失败时作为回退。 |
| `pixso-component-config` | `/skill:pixso-component-config` | 生成或更新 Design-to-Code 组件解析 JSON（`componentParsers`）。 |

skill 之间按名称互相引用（例如设计转代码失败时回退到 `pixso-read-dsl`），pi 会按需自动加载，无需手动切换。

## 可以怎么用

直接用自然语言让 pi 调用 Pixso。

### 设计转代码

```text
Use Pixso to convert this selected frame to React code and save all generated assets locally.
```

也可以提供 Pixso URL：

```text
Convert this Pixso URL to Vue code: <Pixso node URL>
```

### 网页或 HTML 导入 Pixso

```text
Import this page into Pixso as editable design nodes: https://example.com
```

如果页面需要登录、权限、区域访问或资源不可抓取，pi 会先说明阻塞点，不会自动降级成近似重建设计。

### Pixso 设计编辑

```text
Create a dashboard screen in the current Pixso file using the existing design tokens and components.
```

```text
Refine the selected Pixso frame: improve spacing, hierarchy, and visual consistency.
```

### 组件解析配置

```text
/skill:pixso-component-config

或直接描述需求：
Generate componentParsers for the current Pixso component library and Element Plus.
```

## 常见问题

### Skill 已安装，但 Pixso 工具不可用

先确认 Pixso MCP 是否启动：

```bash
curl -sS -I http://localhost:3667/mcp
```

请求失败时，先启动 Pixso Desktop 并确认 MCP 服务已开启；然后在 pi 中执行 `/mcp` 查看 `pixso` 状态，必要时 `/mcp reconnect pixso`。

### 修改 MCP 端口后没生效

确认改的是 pi 实际读取的配置文件，并执行 `/reload` 或重启 pi。配置文件优先级：

1. `~/.config/mcp/mcp.json`
2. `~/.agents/mcp.json`、`~/.agents/mcp/mcp.json`
3. `~/.pi/agent/mcp.json`（pi 全局）
4. `.mcp.json`（项目共享）
5. `.pi/mcp.json`（pi 项目覆盖）

### `/skill:pixso-component-config` 看不到

通常是 skill 未启用或旧会话缓存。执行 `pi config` 确认对应 skill 已启用，或重启 pi 会话。

## 与原项目的关系

- **保留**：`skills/` 下的全部内容（5 个 skill 及其 references / scripts）、`LICENSE`、Pixso 图标。
- **移除**：Claude Code（`.claude-plugin/`、`commands/`）、Cursor（`.cursor-plugin/`）、Gemini（`gemini-extension.json`）专属的插件脚手架——pi skill 通过 `/skill:<name>` 自动注册命令，无需这些文件。
- **适配**：`package.json` 改为 pi package 清单（`pi.skills`）；新增 pi 适用的 `.mcp.json`（`pi-mcp-adapter` 直接读取）与本说明。

原项目采用 Apache-2.0 许可，本适配包沿用同一许可证。

## 参考

- [pi skills 文档](https://github.com/badlogic/pi-mono)
- [pi-mcp-adapter](https://github.com/badlogic/pi-mcp-adapter)
- [原项目 PixsoLtd/pixso-ai-integration](https://github.com/PixsoLtd/pixso-ai-integration)
