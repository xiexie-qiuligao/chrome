# Chrome DevTools MCP Continuous

`Chrome DevTools MCP Continuous` 是基于 [ChromeDevTools/chrome-devtools-mcp](https://github.com/ChromeDevTools/chrome-devtools-mcp) 的增强版 fork，重点针对 **长生命周期浏览器会话** 做了改进，适合需要持续接管真实 Chrome 会话的 AI agent 场景。

本仓库地址：

- [xiexie-qiuligao/chrome](https://github.com/xiexie-qiuligao/chrome)

## 项目定位

这个版本的目标不是替代上游官方项目，而是在保留上游工具面和工程基础的前提下，优化以下场景：

- 接管已经打开、已经登录的 Chrome
- 多标签页持续操作
- 降低每次工具调用时的全量页面/target 重扫
- 提高 agent 长时间运行时的会话连续性和稳定性

一句话概括：

**这个版本更强调“持续会话”，而不是“每次命令重新认识浏览器”。**

## 与上游相比的主要改动

相比上游版，本 fork 重点做了以下增强：

- 将浏览器状态刷新从“每次工具调用无条件执行”改为 **按需刷新**
- 引入 **增量 page/target registry**
- 已知标签页的变化优先走 **增量更新**
- DevTools 状态改为 **按脏页更新**
- `newPage()` / `closePage()` 不再强制触发整套页面世界重建
- 为页面增加更稳定的 `targetId` 身份跟踪

这些改动的直接收益是：

- 已登录会话更容易保持
- 多标签页时更稳定
- 长时间 agent 操作时状态更连续

## 适用场景

推荐在以下场景使用：

- 需要复用现有浏览器登录态
- 需要在大量标签页之间持续切换和操作
- 需要让 AI agent 长时间接管浏览器而不是偶尔执行一次自动化
- 希望浏览器控制过程更接近真实会话，而不是短连接式操作

如果你只需要标准官方 DevTools MCP，上游版本已经足够。  
如果你更重视 **持续会话、多标签页、长时间运行稳定性**，这个版本更合适。

## 环境要求

- Node.js `20.19+`
- 推荐 Node.js `22`
- Google Chrome 最新稳定版或兼容版本
- npm

## npm 包名

本项目对应的 npm 包名为：

```bash
chrome-devtools-mcp-continuous
```

## 安装方式

### 方式一：通过 npm 使用

已发布到 npm，可直接通过 `npx` 启动：

```bash
npx -y chrome-devtools-mcp-continuous
```

### 方式二：从源码构建

#### 1. 克隆仓库

```bash
git clone https://github.com/xiexie-qiuligao/chrome.git
cd chrome
```

#### 2. 安装依赖

```bash
npm install --ignore-scripts
```

#### 3. 构建

推荐使用 Node 22：

```bash
npx -y node@22 node_modules/typescript/bin/tsc
npx -y node@22 --experimental-strip-types scripts/post-build.ts
```

构建完成后，MCP 服务入口为：

```text
build/src/bin/chrome-devtools-mcp.js
```

## 推荐使用方式：连接已经打开的 Chrome

如果希望保留：

- 登录态
- 当前标签页状态
- 多标签页上下文
- 长时间运行时的连续会话体验

最推荐的方式是：**先启动 Chrome，再让 MCP 连接到该实例。**

### Windows 示例

```powershell
& "C:\Program Files\Google\Chrome\Application\chrome.exe" --remote-debugging-port=9222 --user-data-dir=D:\chrome-mcp-profile
```

说明：

- `--remote-debugging-port=9222` 用于开放调试端口
- `--user-data-dir` 建议使用单独目录，不要直接使用日常默认浏览数据目录

## MCP 配置示例

### 使用 npm 包

```json
{
  "mcpServers": {
    "chrome-devtools-continuous": {
      "command": "npx",
      "args": [
        "-y",
        "chrome-devtools-mcp-continuous",
        "--browser-url=http://127.0.0.1:9222",
        "--no-usage-statistics"
      ]
    }
  }
}
```

### 使用本地构建版本

适用于本地直接使用源码构建结果。

#### TOML 示例

```toml
[mcp_servers.chrome-devtools-continuous]
command = "cmd"
args = [
  "/c",
  "npx",
  "-y",
  "node@22",
  "C:\\path\\to\\chrome\\build\\src\\bin\\chrome-devtools-mcp.js",
  "--browser-url=http://127.0.0.1:9222",
  "--no-usage-statistics"
]
env = { SystemRoot = "C:\\Windows", PROGRAMFILES = "C:\\Program Files" }
startup_timeout_ms = 30000
```

请将 `C:\\path\\to\\chrome\\...` 替换为本地实际路径。

## 维护者：发布到 npm

### 本地打包验证

```bash
npm pack
```

### 正式发布

```bash
npm publish --access public
```

## 关键文件

- [package.json](./package.json)
- [server.json](./server.json)
- [.mcp.json](./.mcp.json)
- [gemini-extension.json](./gemini-extension.json)

## 上游文档

本项目仍然基于上游 Chrome DevTools MCP，下列文档依然具有参考价值：

- [工具文档](./docs/tool-reference.md)
- [变更记录](./CHANGELOG.md)
- [故障排查](./docs/troubleshooting.md)
- [设计原则](./docs/design-principles.md)

## 许可协议

沿用上游许可协议：

- `Apache-2.0`
