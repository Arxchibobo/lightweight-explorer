---
name: lightweight-explorer
description: 轻量探索模式 — 用最少 token 完成代码搜索和调研。核心理念：只读搜索不需要加载全部工具和项目规则。适用于：(1) 搜索代码库找文件/函数 (2) 调研项目架构 (3) 多文件分析 (4) 回答"代码里哪里有 X"类问题。灵感源自 Claude Code Explore Agent 设计。
---

# Lightweight Explorer

## 核心理念

> 探索任务 = 只读 + 快速 + 低成本

**不修改任何文件**。探索的目标是收集信息并报告，不是做改动。

## 搜索策略

### 不知道位置时 → 广搜
```bash
# 文件模式搜索
find . -name "*.ts" -path "*/auth/*" | head -20

# 内容搜索
grep -rn "pattern" src/ --include="*.ts" | head -30

# 结构概览
find src/ -type f -name "*.ts" | head -50
```

### 知道大概位置时 → 精确读
直接 `read` 目标文件，不要先搜索。

### 第一次搜不到 → 换策略
- 换命名方式（camelCase vs snake_case vs kebab-case）
- 换搜索范围（src/ → lib/ → packages/）
- 用更宽泛的 pattern

## 并行执行

**独立的搜索操作必须并行发起**，不要串行：
```
✅ 同时搜 3 个不同的 grep pattern
❌ 搜完第一个再搜第二个
```

## 输出格式

探索完成后输出简洁报告：
```
## 发现

### [主题1]
- 文件: path/to/file.ts (line X-Y)
- 内容: 简短描述

### [主题2]
...
```

**不要**：
- 创建文件（包括临时文件）
- 修改任何文件
- 运行写操作命令（git add/commit, npm install, mkdir）
- 输出冗长的分析 — 保持精简

## 搜索深度控制

调用者可指定深度：
- `quick` — 基础搜索，找到主要文件即可
- `medium` — 中等探索，多角度验证
- `thorough` — 全面分析，检查多个位置和命名约定

默认 `medium`。

## 适用场景

| 场景 | 动作 |
|---|---|
| "XXX 在哪里？" | grep + find |
| "这个模块怎么工作的？" | 读入口文件 → 追踪调用链 |
| "有没有类似的实现？" | grep pattern + 并行搜多处 |
| "项目结构是什么样的？" | find + tree + 读关键文件 |
