# Project-to-Resume AI Skill

<p align="center">
  <b>Generate professional, interview-ready HTML resumes directly from your project codebase.</b><br>
  <b>直接从您的项目源码生成专业、可直接面试的 HTML 简历。</b>
</p>

[English](#english) | [中文](#中文)

---

## 中文

### 🌟 简介

**Project-to-Resume** 是一个 AI Agent 技能。它可以自动分析您的代码仓库，提取技术亮点，并将其整合生成排版专业的 HTML 格式简历。该技能深度运用了 PSR（Problem-Solution-Result）方法论，确保您的简历侧重于技术深度和业务成果，而不仅仅是罗列功能。

### ✨ 核心功能

- 🔍 **深度代码扫描**：自动分析目录结构、识别架构模式，统计 API/数据表数量，精准提取技术栈。
- 💡 **PSR 方法论**：严格按照“问题(P) - 方案(S) - 结果(R)”的模型提炼技术亮点，直接满足面试官的阅读期望。
- 📄 **精美排版**：生成适合 A4 纸打印的单页 HTML 简历，包含一键复制等贴心交互，可直接使用浏览器导出为 PDF。
- 👤 **信息持久化**：首次使用自动采集并保存您的基本信息（联系方式、教育背景等），后续无缝复用。
- 🎯 **智能岗位适配**：根据前端、后端、全栈等目标岗位，智能调整简历描述的侧重点。

### 🚀 如何使用

本项目专为 AI Agent（如 Gemini / Claude / 各种 AI 辅助编程环境）设计。

**触发方式**：
直接在支持的 Agent 聊天窗口中输入：
- “帮我根据当前项目生成简历”
- “生成HTML简历”
- 或显式调用：`/skill project-to-resume`

**使用要求**：
请确保 Agent 当前的工作区/上下文环境为您需要提取的项目代码库。

### 📊 示例预览

您可以在以下链接中查看技能的最终产出和完整执行流程：
- [📄 完整生成的 HTML 简历示例 (点击查看源码)](./examples/sample-resume.html) _(注：建议下载该文件并在浏览器中打开以预览确切排版效果)_
- [📋 完整的 Agent 对话与执行流程示例](./examples/sample-output.md)

### 📁 输出说明

默认情况下，Agent 会在您的项目中生成以下文件：
- `docs/resume.html` （默认主产物：带排版的完整 HTML 简历）
- `docs/resume.md` （兼容的纯文本 Markdown 版本）

---

### 📝 License

[MIT License](./LICENSE)


## English

### 🌟 Overview

**Project-to-Resume** is an AI Agent Skill that automatically analyzes your code repository, extracts technical highlights, and synthesizes them into a professionally formatted HTML resume. It uses the PSR (Problem-Solution-Result) framework to ensure your resume focuses on technical depth and impact rather than just listing features.

### ✨ Features

- 🔍 **Deep Code Analysis**: Scans directories, identifies architecture patterns, counts APIs/Tables, and extracts core tech stacks.
- 💡 **PSR Methodology**: Automatically formulates technical achievements using the Problem-Solution-Result framework.
- 📄 **Print-Ready HTML**: Outputs a beautiful, single-page A4 HTML resume that you can easily save as a PDF.
- 👤 **Persistent Profile**: Asks for and saves your personal information (education, contact) for future reuse.
- 🎯 **Role Adaptation**: Tailors the resume content towards specific roles (Frontend, Backend, Fullstack).

### 🚀 Usage

This skill is designed to be run within an AI Agent environment (like Gemni / Claude / CodeArts).

**Trigger Phrases**:
Just tell your Agent:
- "Generate a resume for this project"
- "Create an HTML resume"
- `/skill project-to-resume`

**Requirements**:
The agent should be in a workspace containing your project code.

### 📊 Examples

You can view the final output and complete execution workflow of the skill in the following links:
- [📄 Sample generated HTML Resume (Click to view source)](./examples/sample-resume.html) _(Note: Recommend downloading this file and opening it in a web browser for the exact layout preview)_
- [📋 Complete Agent Execution Workflow Example](./examples/sample-output.md)

### 📁 Output Formats

- `docs/resume.html` (Default: Complete, stylized web resume)
- `docs/resume-zh.md` (Markdown format)

---

