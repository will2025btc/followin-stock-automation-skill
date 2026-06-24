# Followin Stock Automation Skill

这是一个给 Codex Agent 使用的美股自选股自动化 skill。它会要求 Agent 使用 Followin MCP 作为主要数据源，围绕你的自选股和持仓状态，生成盘前跟踪、异动提醒、重大新闻同步和操作计划。

适合这些场景：

- 每天盘前跟踪自选股
- 监控美股新闻、研报、Twitter/社媒热度和异动
- 根据当前持仓给出加仓、减仓、等待、止损或观察建议
- 空仓时给出条件化的入场计划

## 安装 Skill

把这个仓库克隆到 Codex skills 目录：

```bash
mkdir -p ~/.codex/skills
git clone https://github.com/will2025btc/followin-stock-automation-skill.git \
  ~/.codex/skills/followin-stock-automation
```

安装后重启 Codex，让新 skill 生效。

## 安装 Followin MCP

这个 skill 依赖 Followin MCP。没有 Followin MCP 时，Agent 不应该假装拿到了 Followin 数据。

Followin 官方 MCP 页面：[https://followin.io/zh-Hans/mcp](https://followin.io/zh-Hans/mcp)

下面使用推荐的 Streamable HTTP 端点：

```text
https://mcp.followin.io/v2/mcp
```

### Codex

在 `~/.codex/config.toml` 里加入：

```toml
[mcp_servers.followin]
url = "https://mcp.followin.io/v2/mcp"
env_http_headers = { "x-api-key" = "FOLLOWIN_MCP_TOKEN" }
```

然后在启动 Codex 前设置环境变量：

```bash
export FOLLOWIN_MCP_TOKEN="YOUR_API_KEY_HERE"
```

如果你接受把 API Key 直接写进配置，也可以用：

```toml
[mcp_servers.followin]
url = "https://mcp.followin.io/v2/mcp"
http_headers = { "x-api-key" = "YOUR_API_KEY_HERE" }
```

### Claude Code

可以直接运行：

```bash
claude mcp add followin https://mcp.followin.io/v2/mcp \
  --scope user \
  --transport http \
  --header "x-api-key: YOUR_API_KEY_HERE"
```

## 使用方式

告诉 Agent 三件事即可：

1. 你的自选股
2. 当前持仓状态
3. 需要的自动化频率或时间

示例：

```text
请用 followin-stock-automation 帮我创建一个盘前自动化。
自选股：DRAM, SNDK, MU, NOK, MRVL
持仓：目前全部空仓
时间：美股交易日盘前，每天北京时间 20:30
```

也可以只说：

```text
请用 Followin MCP 跟踪我的自选：DRAM, SNDK, MU, NOK, MRVL。我现在空仓。
```

Agent 会默认按美股盘前节奏创建或更新自动化。

## 输出内容

每次运行时，报告应包括：

- 市场背景：指数期货、板块情绪、风险偏好
- 单票跟踪：价格、涨跌幅、成交量、技术位、催化、新闻、研报、社媒热度
- 持仓建议：持有、加仓、减仓、止损、等待或观察
- 空仓计划：入场触发价、无效价、止损、优先级和初始仓位建议
- 组合视角：当天最值得关注的 1-2 个机会，以及需要回避的风险
- 来源说明：明确哪些信息来自 Followin MCP

## 注意

- 这不是投资顾问工具，输出只能作为交易研究和决策辅助。
- 如果 Followin MCP 不可用，Agent 应明确说明，而不是编造数据。
- 操作建议应是条件化的，例如“突破某价位并放量后考虑试仓”，而不是无条件买入或卖出。
