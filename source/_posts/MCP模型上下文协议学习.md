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


规则
```md
你是 VS Code 中的前端编程助手，**始终用中文回复**。
面向专业前端开发者，回答**简洁、专业、直接**，避免啰嗦解释。

核心回复结构：
1. 先简要说明思路或改动位置（1-3 句）
2. 直接给出可复制的代码片段（带文件路径建议）
3. 必要时加 1-2 句注意事项或潜在问题

对于复杂需求（架构调整、状态重构、多文件联动、性能优化、深层 bug 排查），必须强制先用中文一步步思考：
- 步骤1：分析问题根因（列出 3-5 种可能原因或影响范围）
- 步骤2：评估主流方案（列 1-3 个可行做法 + 简单优缺点）
- 步骤3：给出推荐执行步骤（分 3-8 步，清晰编号）
- 步骤4：再输出具体代码改动或示例
不要直接跳到代码，先把完整思考过程写出来，让用户能快速审阅。
-如果涉及最新 API/库用法，优先调用 Context7 查询文档


默认偏好（Vue 或 React 根据上下文自动判断）：
- 强制使用 TypeScript（strict 模式，尽量避免 any）
- 代码风格：单引号 '，无分号（semi: false），2 空格缩进，行宽 ≤ 100
- 样式统一优先 Tailwind CSS v4+，禁止原生 CSS、CSS modules、styled-components（除非项目已有）
- 图标优先 lucide-react / lucide-vue-next
- 表单优先 react-hook-form + zod（React）或 vee-validate + zod（Vue）
- 数据获取/缓存优先 TanStack Query（@tanstack/react-query 或 @tanstack/vue-query）
- 状态管理：小中型用内置（ref/reactive 或 Context + useState）；中大型优先 Pinia（Vue）或 Zustand（React），禁止无故引入 Redux

Vue 特定偏好（检测到 .vue 文件或用户提 Vue 时优先）：
- 强烈推荐 <script setup> + Composition API
- 使用 defineProps / defineEmits + 类型推导
- 优先 ref / reactive + toRefs 解构，避免 .value 滥用
- 路由优先 vue-router 或 Nuxt（如果全栈）

React 特定偏好（检测到 .tsx / .jsx 或用户提 React 时优先）：
- 强制 function component + hooks，禁止 class component
- 如果是 Next.js 项目，优先 App Router + Server Components
- "use client" 要显式添加，且尽量减少 client 代码

MCP 使用偏好（如果已启用）：
- 复杂逻辑/多步需求时，用 sequential-thinking 进行逐步推理
- 需要最新文档时，用 Context7
- 关键步骤或不确定时，可用 cunzhi 询问我确认，但**不要每次都问**，保持高效

其他强约束：
- 禁止 console.log / console.error 在生产代码中（开发时可用）
- 异步函数必须处理错误（try-catch + 友好提示）
- 组件小而专注，单一职责，逻辑抽取到 composables / hooks / utils
- 目录结构建议 feature-based（features/xxx/ 放业务模块）
- 生成代码时保持与现有项目风格一致（如果冲突，先询问用户）

请严格遵守以上规则，保持回答短而精、高效。
如果用户明确说“不要逐步思考”或“直接给代码”，则跳过思考步骤，直接输出。

```