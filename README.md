# VSCode Internal Command MCP Server

🚀 一个基于 [FastMCP](https://github.com/punkpeye/fastmcp) 框架的 VSCode 扩展，将 VSCode 转换为 MCP (Model Context Protocol) 服务器，支持外部客户端通过 HTTP Streaming 和 Server-Sent Events (SSE) 执行 VSCode 内部命令。

## ✨ 功能特性

-   🌐 **HTTP Streaming 支持**: 使用 `text/event-stream` 协议，支持实时通信
-   🔧 **VSCode 命令执行**: 远程执行任意 VSCode 内部命令
-   📊 **工作区信息查询**: 获取当前工作区状态和文件信息
-   🛡️ **安全控制**: 可配置的命令白名单机制
-   📡 **实时状态监控**: 状态栏显示服务器运行状态
-   🔗 **标准 MCP 协议**: 完全兼容 Model Context Protocol 规范
-   ⚡ **高性能**: 基于 FastMCP 框架，支持并发请求和会话管理
-   🩺 **健康检查**: 内置健康检查端点

## 📦 安装

### 1. 克隆项目

```bash
git clone https://github.com/bestk/vscode-internal-command-mcp-server
cd vscode-internal-command-mcp-server
```

### 2. 安装依赖

```bash
npm install
```

### 3. 编译项目

```bash
npm run compile
```

### 4. 在 VSCode 中安装

-   按 `F5` 启动扩展开发主机
-   或者打包为 `.vsix` 文件进行安装

## ⚙️ 配置

在 VSCode 设置中配置服务器参数：

```json
{
    "vscodeICommandMcpServer.port": 8080,
    "vscodeICommandMcpServer.host": "localhost",
    "vscodeICommandMcpServer.autoStart": true,
    "vscodeICommandMcpServer.enableWebSocket": true,
    "vscodeICommandMcpServer.allowedCommands": [
        "editor.action.formatDocument",
        "workbench.action.files.save",
        "editor.action.clipboardCopyAction"
    ]
}
```

### 配置说明

| 配置项            | 类型     | 默认值      | 说明                                         |
| ----------------- | -------- | ----------- | -------------------------------------------- |
| `port`            | number   | 8080        | MCP 服务器端口                               |
| `host`            | string   | "localhost" | MCP 服务器主机地址                           |
| `autoStart`       | boolean  | true        | 扩展激活时自动启动服务器                     |
| `enableWebSocket` | boolean  | true        | 启用 WebSocket 支持（未来功能）              |
| `allowedCommands` | string[] | []          | 允许执行的命令列表（空数组表示允许所有命令） |

## 🚀 使用方法

### 启动服务器

1. **自动启动**: 扩展激活时自动启动（如果 `autoStart` 为 true）
2. **手动启动**:
    - 命令面板: `VSCode MCP Server: Start Server`
    - 或点击状态栏中的 🚀 FastMCP 按钮

### 服务器地址

-   **MCP 端点**: `http://localhost:8080/mcp`
-   **健康检查**: `http://localhost:8080/health`

### 状态监控

-   状态栏显示: 🚀 FastMCP 🟢 (运行中) / 🚀 FastMCP 🔴 (已停止)
-   命令面板: `VSCode MCP Server: Show Status` 查看详细状态

## 🛠️ 可用工具 (MCP Tools)

### 1. execute_vscode_command

执行 VSCode 内部命令

**参数**:

```typescript
{
  command: string;      // VSCode 命令 ID
  arguments?: string[]; // 命令参数（可选）
}
```

**示例**:

```json
{
    "name": "execute_vscode_command",
    "arguments": {
        "command": "editor.action.formatDocument",
        "arguments": []
    }
}
```

### 2. list_vscode_commands

列出所有可用的 VSCode 命令

**参数**: 无

**返回**: 命令列表（前20个，如果超过会显示省略提示）

### 3. get_workspace_info

获取当前工作区信息

**参数**: 无

**返回**:

```typescript
{
    name: string; // 工作区名称
    folders: Array<{
        // 工作区文件夹
        name: string;
        uri: string;
    }>;
    activeEditor: string; // 当前活动编辑器文件路径
}
```

## 🔌 客户端连接

### 使用官方 MCP SDK

```typescript
import { StreamableHTTPClientTransport } from '@modelcontextprotocol/sdk/client/streamableHttp.js';
import { Client } from '@modelcontextprotocol/sdk/client/index.js';

const transport = new StreamableHTTPClientTransport(new URL('http://localhost:8080/mcp'), {
    requestInit: {
        headers: {
            'Content-Type': 'application/json',
            Accept: 'application/json, text/event-stream',
        },
    },
});

const client = new Client({
    name: 'vscode-mcp-client',
    version: '1.0.0',
});

// 连接并使用
await client.connect(transport);

// 调用工具
const result = await client.callTool({
    name: 'execute_vscode_command',
    arguments: {
        command: 'editor.action.formatDocument',
    },
});

console.log('Command result:', result);
```

### 使用 Cursor

在 Cursor 中配置 MCP 服务器：

```json
{
    "mcpServers": {
        "vscode-internal-commands": {
            "url": "http://localhost:8080/mcp",
            "transport": "http"
        }
    }
}
```

### 使用 curl 测试

```bash
# 健康检查
curl http://localhost:8080/health

# 列出工具
curl -X POST http://localhost:8080/mcp \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "tools/list"
  }'

# 执行命令
curl -X POST http://localhost:8080/mcp \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 2,
    "method": "tools/call",
    "params": {
      "name": "execute_vscode_command",
      "arguments": {
        "command": "workbench.action.files.save"
      }
    }
  }'
```

## 🏗️ 技术架构

### 核心组件

```
┌─────────────────────┐
│   VSCode Extension  │
├─────────────────────┤
│   FastMcpServer     │ ← 基于 FastMCP 框架
├─────────────────────┤
│   ServerManager     │ ← 服务器管理和状态
├─────────────────────┤
│   CommandExecutor   │ ← VSCode 命令执行器
└─────────────────────┘
```

### 技术栈

-   **框架**: [FastMCP](https://github.com/punkpeye/fastmcp) - TypeScript MCP 服务器框架
-   **协议**: Model Context Protocol (MCP)
-   **传输**: HTTP Streaming with Server-Sent Events (SSE)
-   **验证**: Zod Schema 验证
-   **平台**: VSCode Extension API

### 网络协议

-   **传输类型**: `httpStream`
-   **内容类型**: `text/event-stream`
-   **支持协议**: HTTP/1.1, WebSocket (计划中)
-   **CORS**: 默认启用

## 🔧 开发

### 项目结构

```
cursor-command-http/
├── src/
│   ├── extension.ts        # 扩展入口点
│   ├── fastMcpServer.ts    # FastMCP 服务器实现
│   ├── serverManager.ts    # 服务器管理器
│   ├── commandExecutor.ts  # VSCode 命令执行器
│   └── mcpServer.ts        # 原始 MCP 服务器（备用）
├── out/                    # 编译输出
├── package.json           # 扩展配置和依赖
├── tsconfig.json          # TypeScript 配置
└── README.md              # 项目文档
```

### 开发命令

```bash
# 开发模式编译
npm run compile

# 监视模式编译
npm run watch

# 启动开发
code . # 打开 VSCode，按 F5 启动调试
```

### 调试

1. 在 VSCode 中打开项目
2. 按 `F5` 启动扩展开发主机
3. 在新窗口中测试扩展功能
4. 查看调试控制台输出

## 🧪 测试

### 使用内置测试工具

1. 启动服务器后，使用命令: `VSCode MCP Server: Test MCP Tools`
2. 选择要测试的工具
3. 输入必要的参数
4. 查看执行结果

### 使用 FastMCP CLI

```bash
# 使用 FastMCP 开发工具测试
npx fastmcp dev src/fastMcpServer.ts

# 使用 MCP Inspector 检查
npx fastmcp inspect src/fastMcpServer.ts
```

## 🛡️ 安全考虑

### 命令白名单

为了安全起见，建议配置 `allowedCommands` 白名单：

```json
{
    "vscodeICommandMcpServer.allowedCommands": [
        "editor.action.formatDocument",
        "workbench.action.files.save",
        "workbench.action.files.saveAll",
        "editor.action.clipboardCopyAction",
        "editor.action.clipboardPasteAction"
    ]
}
```

### 网络安全

-   默认只监听 `localhost`，避免外部访问
-   支持 CORS，但建议在生产环境中配置适当的源限制
-   所有命令执行都在 VSCode 安全上下文中进行

## 📝 更新日志

### v0.0.1 (当前版本)

-   ✅ 基于 FastMCP 框架实现 MCP 服务器
-   ✅ 支持 HTTP Streaming 和 SSE
-   ✅ 实现三个核心工具：命令执行、命令列表、工作区信息
-   ✅ 状态栏集成和实时监控
-   ✅ 健康检查端点
-   ✅ Zod Schema 参数验证
-   ✅ TypeScript 5.9+ 支持

## 🤝 贡献

欢迎提交 Issue 和 Pull Request！

### 开发指南

1. Fork 项目
2. 创建功能分支: `git checkout -b feature/amazing-feature`
3. 提交更改: `git commit -m 'Add amazing feature'`
4. 推送分支: `git push origin feature/amazing-feature`
5. 创建 Pull Request

## 📄 许可证

MIT License - 详见 [LICENSE](LICENSE) 文件

## 🙏 致谢

-   [FastMCP](https://github.com/punkpeye/fastmcp) - 优秀的 TypeScript MCP 框架
-   [Model Context Protocol](https://modelcontextprotocol.io/) - 标准协议规范
-   VSCode Extension API - 强大的扩展平台

## 📞 支持

如果您遇到问题或有疑问：

1. 查看 [Issues](https://github.com/bestk/vscode-internal-command-mcp-server/issues)
2. 创建新的 Issue
3. 查看 FastMCP 文档: https://github.com/punkpeye/fastmcp

---

**🚀 让 VSCode 成为您的 MCP 服务器，释放无限可能！**
