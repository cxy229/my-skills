---
name: second-brain
description: 个人第二大脑系统 - 快速捕捉想法、自动分类归档到 Obsidian。当用户要求记录想法、保存稍后阅读、创建笔记、处理 Inbox、搜索笔记、或提到"第二大脑"、"知识管理"时激活。
---

# 第二大脑 Skill

基于 Obsidian 的个人知识管理系统，支持捕捉、整理、检索。

## Vault 路径
`/Users/czx229/Documents/BaiduSyncDisk/Notes`

## 核心命令

### `/capture` - 快速捕捉
用法：
- `记录想法：[内容]`

写入位置：`00_Inbox/闪念-YYYYMMDD-HHmm.md`

### `/read-later` - 稍后阅读
用法：
- `稍后阅读：[URL] [备注]`

写入位置：`00_Inbox/稍后阅读-YYYYMMDD-HHmm.md`
如果 URL 可访问，抓取内容摘要

### `/note` - 创建笔记
用法：
```
创建笔记：[标题]
[正文内容]
```

写入位置：根据内容智能判断目标文件夹，不确定则放 Inbox

### `/process-inbox` - 处理收件箱
处理 `00_Inbox/` 中所有笔记：

1. **提炼摘要** - 在顶部添加 50 字以内的【内容摘要】
2. **智能分类** - 移动到对应文件夹：
   - `10_Tech_AI` - AI、大模型、智能体、代码
   - `20_Finance_Invest` - 股市、研报、投资
   - `30_Life_Family` - 育儿、旅行、游戏、影视
   - `01_Buffer` - 不符合以上分类的新类别
3. **格式化** - 添加 YAML 头部：
   ```yaml
   ---
   title: [精准标题]
   date: YYYY-MM-DD HH:mm
   tags: [标签1, 标签2]
   category: [文件夹名]
   status: processed
   ---
   ```
4. **打标签** - 提取 1-2 个核心名词作为标签，**标签不能包含空格**

### `/note-search` - 搜索笔记
用法：
- `搜索笔记: [关键词]`
- `/note-search [关键词]`

支持多关键词搜索，用空格分隔。

**搜索范围：**
- 文件名（标题）
- YAML tags 字段
- 【内容摘要】
- 正文内容

**返回格式：**
```
🔍 找到 N 条相关笔记：

📄 [标题]
   📁 [分类] | 🏷️ [标签]
   📝 [摘要或内容片段]
   📍 [相对路径]
```

## 搜索实现

### 步骤 1：关键词搜索
使用 grep 搜索 Vault 目录：
```bash
grep -r -i -l "[关键词]" "/Users/czx229/Documents/BaiduSyncDisk/Notes" --include="*.md"
```

对于多关键词，分别搜索后取交集（同时包含所有关键词的文件）。

### 步骤 2：读取匹配文件
对每个匹配文件，读取：
- YAML front matter（提取 title、tags、date）
- 【内容摘要】（如果有）
- 匹配上下文（关键词前后 50 字）

### 步骤 3：排序优先级
搜索结果按以下优先级排序：
1. **标题匹配** - 标题包含关键词，优先级最高
2. **标签匹配** - tags 字段包含关键词
3. **摘要匹配** - 【内容摘要】包含关键词
4. **正文匹配** - 其他位置匹配

### 步骤 4：格式化输出
```
🔍 找到 N 条相关笔记：

📄 [标题]
   📁 [分类] | 🏷️ [标签]
   📝 [摘要或匹配片段]
   📍 [相对路径]
```

### 高级搜索

**按标签过滤：**
```
标签：[标签名]
找 #AI编程 的笔记
```

**按分类过滤：**
```
分类：[分类名]
在 Tech_AI 里找 AI 相关
```

**组合搜索：**
```
在 10_Tech_AI 分类里找 Claude Code
标签：AI编程 关键词：Claude
```

### 示例

**用户输入：**
```
帮我找 Claude Code 相关的笔记
```

**执行搜索：**
```bash
grep -r -i -l "Claude Code\|ClaudeCode\|Claude-Code" "/Users/czx229/Documents/BaiduSyncDisk/Notes" --include="*.md"
```

**输出：**
```
🔍 找到 3 条相关笔记：

📄 从零手搓迷你 Claude Code 教程
   📁 10_Tech_AI | 🏷️ Claude-Code, AI编程
   📝 GitHub 热门项目教程，4 天斩获 2 万 Star，教你从零开始构建迷你版 Claude Code。
   📍 10_Tech_AI/从零手搓迷你Claude Code教程.md

📄 当天学习总结
   📁 10_Tech_AI | 🏷️ Claude-Code, Skill开发
   📝 完成吴恩达 Claude Code 教程前 4 章学习，掌握 skill 创建技巧并实践。
   📍 10_Tech_AI/当天学习总结-20260227.md

📄 基于 OpenClaw+Obsidian 构建"第二大脑"
   📁 10_Tech_AI | 🏷️ 无
   📝 ...包含 Claude Code 教程学习记录...
   📍 10_Tech_AI/基于OpenClaw+Obsidian构建个人"第二大脑".md
```

## 处理规则

### 分类判断优先级
1. 内容主题 → 主要话题属于哪个领域
2. 关键词匹配 → 包含特定领域术语
3. 不确定 → 放入 Buffer，添加提示

### Buffer 处理
如果内容放入 `01_Buffer`，在文件顶部添加：
```
> 🤖 缓冲提示：该内容超出现有分类体系，暂存待分类。
```

### 标签规则
- 只提取内容中的核心名词，**标签不能包含空格**
- 使用中文标签，如 `#具身智能` `#INT4量化`
- 最多 2 个标签
- 可以创建新标签，但文件夹只能用预设的

## 示例

**输入：**
```
记录想法：AI Agent 的记忆系统应该分层，短期记忆用向量检索，长期记忆用知识图谱
```

**输出文件：** `00_Inbox/闪念-20260227-1500.md`
```markdown
---
title: AI Agent 记忆系统分层设计
date: 2026-02-27 15:00
tags: [AI-Agent, 记忆系统]
status: inbox
---

AI Agent 的记忆系统应该分层，短期记忆用向量检索，长期记忆用知识图谱
```

**执行 `/process-inbox` 后移动到：** `10_Tech_AI/AI Agent 记忆系统分层设计.md`