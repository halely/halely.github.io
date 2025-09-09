---
title: MCP模型上下文协议学习与配置
date: 2025-08-21 10:30:00
cover: "https://images.unsplash.com/photo-1743452548596-7095486ee7e9?q=80&w=1621&auto=format&fit=crop&ixlib=rb-4.1.0&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
tags:
  - MCP
  - AI
  - 开发工具
categories:
  - 前端学习
---

# 前言

在 AI 开发过程中，模型上下文协议（Model Context Protocol，简称 MCP）为我们提供了强大的工具集成能力。通过 MCP，我们可以让 AI 助手访问各种外部工具和服务，大大提升开发效率。本文记录 MCP 的配置和使用经验。

# 什么是 MCP

MCP（Model Context Protocol）是一种标准化协议，允许 AI 模型与外部工具和服务进行交互。通过 MCP 服务器，AI 可以执行文件操作、网络请求、数据库查询等复杂任务。

# MCP 服务配置

## 文本编辑器服务

```json
{
  "mcpServers": {
    "textEditor": {
      "command": "npx",
      "args": ["-y", "mcp-server-text-editor"]
    }
  }
}
```

**功能**：提供文件读写、编辑功能
**用途**：代码修改、文档编辑、配置文件更新

## DeepWiki 知识库

```json
{
  "mcpServers": {
    "DeepWiki": {
      "command": "npx",
      "args": ["mcp-remote", "https://mcp.deepwiki.com/sse"]
    }
  }
}
```

**功能**：访问技术文档和知识库
**用途**：技术查询、最佳实践获取、概念解释

## 桌面命令执行器

```json
{
  "mcpServers": {
    "desktop-commander": {
      "command": "npx",
      "args": [
        "-y",
        "@smithery/cli@latest",
        "run",
        "@sondotpin/desktopcommandermcp",
        "--key",
        "2a89e84d-c4d7-4fb0-82b5-4fb06449f09d"
      ]
    }
  }
}
```

**功能**：执行系统命令、文件操作
**用途**：项目构建、测试运行、文件管理

## 任务管理器

```json
{
  "mcpServers": {
    "mcp-shrimp-task-manager": {
      "command": "npx",
      "args": ["mcp-shrimp-task-manager"]
    }
  }
}
```

**功能**：任务拆解和依赖管理
**用途**：项目规划、进度跟踪、任务协调

## 用户交互工具

```json
{
  "mcpServers": {
    "cunzhi": {
      "command": "D:\\cunzhi\\寸止.exe",
      "args": [],
      "env": {}
    }
  }
}
```

# MCP 集合网站网址

**MCP Servers 市场**：[https://lobehub.com/zh/mcp](https://lobehub.com/zh/mcp)
**Open-Source MCP servers**：[https://glama.ai/mcp/servers ](https://glama.ai/mcp/servers)
**Smithery.ai**：[https://smithery.ai/](https://smithery.ai/)

# MCP 工作流程

**功能**：用户确认和反馈
**用途**：关键节点确认、用户交互

## 开发流程

1. **研究阶段**：使用 DeepWiki 获取技术背景
2. **构思阶段**：结合知识库设计方案
3. **计划阶段**：使用任务管理器拆解任务
4. **执行阶段**：通过文本编辑器和命令执行器实现
5. **评审阶段**：验证功能完整性

## 工具协同

- **代码编辑**：textEditor + desktop-commander
- **知识查询**：DeepWiki + 技术文档
- **任务管理**：mcp-shrimp-task-manager
- **用户交互**：cunzhi 确认关键节点

# 配置注意事项

## 路径配置

- 确保可执行文件路径正确
- 检查环境变量设置
- 验证网络连接（远程服务）

## 权限设置

- 文件读写权限
- 系统命令执行权限
- 网络访问权限

# 实际应用场景

## 项目开发

1. 需求分析 → DeepWiki 查询技术方案
2. 任务规划 → task-manager 拆解任务
3. 代码实现 → textEditor 编写代码
4. 测试验证 → desktop-commander 运行测试
5. 用户确认 → cunzhi 获取反馈

## 文档编写

- 使用 textEditor 创建和编辑文档
- 通过 DeepWiki 获取参考资料
- desktop-commander 进行文件管理

## 系统维护

- 批量文件操作
- 自动化脚本执行
- 配置文件更新

# 总结

MCP 为 AI 开发提供了强大的工具集成能力，通过合理配置和使用这些服务，可以大大提升开发效率。关键是要理解每个工具的特点，在合适的场景下使用合适的工具，形成高效的工作流程。

在实际使用中，建议：

1. 熟悉各工具的功能边界
2. 建立标准化的工作流程
3. 注意安全性和权限控制
4. 定期更新和维护配置

通过 MCP，我们可以让 AI 助手真正成为开发过程中的得力助手。
