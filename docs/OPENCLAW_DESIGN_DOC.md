# OpenClaw 技术设计文档

## 1. 概述 (Overview)

OpenClaw 是一个个人 AI 助手平台，旨在让用户在自己控制的设备上运行 AI 助手。它通过一个统一的 **Gateway (网关)** 连接各种消息通道（如 WhatsApp, Telegram, Slack, Discord 等），并集成了强大的 **Pi Agent** 运行时，支持工具调用（如浏览器控制、终端执行等）和多模态交互。

## 2. 系统架构 (System Architecture)

OpenClaw 采用分层架构，核心是基于 WebSocket 的 Gateway 控制平面。

![OpenClaw 架构图](openclaw_architecture.svg)

### 核心组件

- **Gateway (网关)**: 系统的中心节点，负责管理所有通道连接、维护会话状态、处理客户端请求和分发事件。
- **Channels (通道)**: 各种消息平台的接入层。
- **Clients (客户端)**: 用户交互界面，包括 CLI、Web UI 和桌面应用。
- **Nodes (节点)**: 运行在不同设备上的执行单元，提供硬件级功能（如摄像头、屏幕录制、系统通知）。
- **Pi Agent Runtime**: 核心 agent 逻辑，负责上下文组装、LLM 推理、工具编排和结果持久化。

## 3. 关键子系统 (Key Subsystems)

### 3.1 Gateway WebSocket 网络

Gateway 提供一个统一的 WebSocket 接口，所有客户端和节点都通过该接口进行通信。它使用 JSON Schema 进行消息校验，并支持基于设备身份的配对 (Pairing) 机制以确保安全。

### 3.2 Pi Agent 运行时

Agent 循环 (Agent Loop) 是系统的核心工作流：

1. **输入处理**: 接收消息并解析意图。
2. **上下文组装**: 结合历史消息、系统提示词、工作区文件 (AGENTS.md) 和技能 (Skills)。
3. **推理与执行**: 调用 LLM 进行决策，并根据需要执行工具。
4. **流式响应**: 实时向客户端推送中间思考过程和最终回复。

### 3.3 存储与会话管理

使用本地文件系统存储会话历史、凭据和工作区配置。每个会话都有独立的 lane，确保并发处理时的顺序性和一致性。

## 4. 关键调用流程 (Key Call Flows)

典型的端到端消息处理流程如下：

![OpenClaw 调用流程图](openclaw_call_flow.svg)

1. **消息进入**: 用户通过 Telegram 发送消息。
2. **网关处理**: Telegram Provider 接收消息并转发给 Gateway。
3. **启动 Agent**: Gateway 调用 Pi Agent 的 `agent` 方法。
4. **模型推理**: Pi Agent 组装提示词并发送给 LLM。
5. **工具执行**: 如果 LLM 决定调用工具（例如搜索网页），Agent 会在本地或沙箱中执行工具并获取结果。
6. **流式反馈**: Agent 边推理边通过 WebSocket 推送 `assistant` 流。
7. **最终交付**: Agent 完成后发送 `lifecycle: end`，网关将完整回复发送回 Telegram。

## 5. 安全模型 (Security)

- **沙箱化**: 默认情况下，工具在本地执行。对于非主会话，可以配置 Docker 沙箱运行。
- **配对机制**: 新设备连接 WebSocket 需要显式批准。
- **DM 策略**: 针对陌生人的私聊，支持配对码校验。

---
*文档生成日期: 2025年1月*
