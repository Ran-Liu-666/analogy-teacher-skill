# Analogy Teacher — 论文精读 + 引文学习 + Idea 发现

[![Version](https://img.shields.io/badge/version-2.8.0-blue)](SKILL.md)
[![Platform](https://img.shields.io/badge/platform-Claude%20Code-orange)](https://claude.ai/code)
[![License](https://img.shields.io/badge/license-MIT-green)](LICENSE)

> **"如果你不能用生活例子解释一个概念，你就没有真正理解它。"**

Analogy Teacher 是一个 Claude Code 技能，将任意论文的任意页面转化为**零基础可独立阅读的完整教材**。它不是论文摘要器，不是翻译工具，而是一个真正的**教师**——用生活类比、手算例子、逐符号翻译和工程直觉，帮助读者从"完全不懂"到达"闭着眼睛也能复述"的理解深度。

---

## 🎯 三大核心能力

### 📖 论文精读（Paper Reading）
- **Two-Pass 架构**：先提取后教学，绝不混合，确保每个解释都有论文原文锚点
- **四层递进教学**：生活类比 → 数值算例 → 形式化翻译 → 工程直觉
- **🔴 17 条红线内部自审**：可测试的质量门（R1-R17），防止错误产出；红线为**内部执行**，不写入交付文档
- **7 种论文类型自适应**：methods / discovery / resource / clinical / materials / review / conceptual
- **WYSIATI 认知校准**：基于 Kahneman 双过程理论，防止"所见即全部"偏差
- **🔴 全量解读强制（v2.6）**：公式、图表、图片**全部**解读，一个不挑——论文有多少公式/图表，交付文档就有多少对应解读小节
- **🔴 图表视觉验证（v2.3/v2.7）**：pymupdf 200dpi 渲染 → 视觉模型确认 → `[VISUALLY CONFIRMED]` 标注；纯文本模型（DeepSeek 等）自动走 vision-skill 视觉桥接，绝不盲猜图表
- **🔴 强制公式内联化（v2.4）**：所有公式/符号输出为 `$...$` 内联 LaTeX（R17 红线）
- **🔴 论文优点发现（v2.5）**：第18节五维亮点——写作语言 / 建模 / 思考路径 / Idea生成过程 / 问题发现
- **🔴 科研技能协同调度（v2.8）**：按**工作流阶段 × 论文类型**双维度调度 11 个科研 skill（学术研究/arXiv日报/写作智库/引用审计/文献综述/Gap探测/科学图表/证据补全/毕设对接/写作引擎/医学证据链）
- **🔴 自我学习机制（v2.5）**：每篇精读后把可复用方法论沉淀进 skill 记忆，越用越聪明

### 🔗 引文学习（Reference Mining）
- 参考文献四类分类（基石/竞争/工具/背景文献）
- 7 级证据层级评估（系统综述→专家观点）
- 方法谱系追踪 + ASCII 谱系图
- 基石文献深度追踪（借了什么？做了什么改进？）

### 💡 Idea 发现（Idea Engine）
- **7 种生成策略**：假设松弛 / 组件替换 / 跨领域移植 / 极限测试 / 反向思考 / Future Work / 组合爆炸
- **4 道品质关卡**：源追溯 → 新颖性区分 → 可行性 → 非发明
- 每个 Idea 附带标准输出模板（可直接用于开题/立项）

---

## 🚀 快速开始

### 安装

```bash
# 1. 创建 skills 目录（如果不存在）
mkdir -p ~/.claude/skills

# 2. 克隆本仓库（替换为你的仓库地址）
git clone https://github.com/YOUR_USERNAME/analogy-teacher.git ~/.claude/skills/analogy-teacher

# 3. 安装核心依赖（v2.3 起，PDF 图表渲染需要）
pip install pymupdf
```

### （可选）让纯文本模型也能看图 — vision-skill

如果你使用的模型**不能直接读图片**（如 DeepSeek 等纯文本模型），Analogy Teacher v2.7+ 会自动调用 [vision-skill](https://github.com/LearningByDoingNow/vision-skill) 把图表交给视觉模型转成文字描述，再回来精读：

```bash
# 1. 安装 vision-skill 与依赖
git clone https://github.com/LearningByDoingNow/vision-skill.git ~/.claude/skills/vision-skill
pip install zai-sdk Pillow requests

# 2. 配置视觉模型 API Key（示例为智谱 GLM-4.6V，中国直连有免费额度）
#    写入 ~/.claude/settings.json：
#    { "env": { "ZHIPU_API_KEY": "你的key", "VISION_MODEL": "glm-4.6v" } }
```

配置后，图表精读自动升级为：**pymupdf 渲染 PDF 页 → vision-skill 视觉确认 → `[VISUALLY CONFIRMED via vision-skill]` 标注**。有视觉能力的模型（Claude/GPT-4o）无需此步，直接用 Read 工具看图。

### 触发

安装后，在新对话中说以下任意自然语句即可激活：

| 你想做什么 | 这样说 |
|-----------|--------|
| 精读论文 | `讲懂我` `打个比方` `零基础` `精读这篇论文` `读论文` `论文第X页` |
| 学习引用论文 | `学习引用论文` `追踪方法谱系` `这个方法从哪里来的` |
| 找研究 Idea | `找Idea` `有什么可做的` `研究空白` `未来方向` |

首次触发时，技能会先询问你的知识水平（🌱零基础 / 🌿初学者 / 🌳进阶者 / 🎓研究者）与阅读目标，然后根据你的水平自适应调整教学深度。

---

## 🏗️ 架构总览（v2.8）

```
Gate 0: 学习者水平评估 + 读者原型 → 4级自适应教学
    ↓
Pass 1: 结构提取（只读）
    ├── 论文类型分类（7种原型）
    ├── 术语账本构建
    ├── 证据清单 + 声明-证据矩阵 + 块ID锚定 [B###][E###][F###]
    └── 科研技能协同调度（工作流阶段 × 论文类型 → 11 skill 路由）
    ↓
[Gate: 用户确认]
    ↓
Pass 2: 教学写作
    ├── 🔴 全量解读（v2.6）：每个公式/图表/图片都解读，不挑选
    ├── 🔴 图表视觉验证：pymupdf 渲染 → 视觉确认（纯文本模型走 vision-skill）
    ├── 🔴 公式内联化（$...$ LaTeX）
    ├── 类型特定的教学模板 + 四层递进 + 八个协议
    ├── WYSIATI 认知校准检查点
    ├── 17条红线内部自审（不写入交付文档）
    └── 第18节 论文优点发现（五维）+ 第19节 Idea
    ↓
输出: 19节固定教学卡片 → 保存到工作目录
     ↓
自我学习: 可复用方法论沉淀进 skill 记忆
```

| 版本 | 行数 | 核心新增 |
|------|:---:|------|
| v2.8 | 2,249 | 🔴 科研技能协同调度全面升级：工作流阶段 × 论文类型 双维度调度 11 个科研 skill |
| v2.7 | 2,240 | 🔴 vision-skill 集成：纯文本模型看图（GLM-4.6V 视觉桥接 + `[VISUALLY CONFIRMED via vision-skill]` 标注） |
| v2.6 | 2,230 | 🔴 全量解读强制：公式/图表/图片全部解读，一个不挑（数量对不上即违规） |
| v2.5 | 2,226 | 🔴 第18节改为论文优点发现(五维) + 红线内移不写入交付文档 + 科研技能协同调度 + 自我学习机制 |
| v2.4 | 2,164 | 🔴 强制公式内联化(R17红线：所有公式/符号输出为 `$...$` 内联LaTeX) |
| v2.3 | 2,148 | 🔴 强制PDF图表渲染协议 + R16红线 + SciPilot视觉检查清单 |
| v2.2 | 2,078 | 学习者水平自适应（4级） |
| v2.1 | 2,001 | 读图协议（三层读图+6易错检查）+ 作图协议（Python/Mermaid） |
| v2.0 | 1,491 | Two-Pass 架构、15条红线、论文类型分类、WYSIATI |

---

## 📁 文件结构

```
analogy-teacher/
├── SKILL.md              # 主技能文件（2,249行）— Claude Code 自动加载
├── README.md             # 本文件
├── LICENSE               # MIT 许可
├── .gitignore            # 忽略生成产物（状态文件/教学输出/临时图）
└── examples/             # 示例输出
    └── EXAMPLE.md        # 一页论文的完整教学输出示例
```

---

## 🤝 贡献

欢迎提交 Issue 和 Pull Request。在提交 PR 之前，请确保：
- 修改不破坏现有的 17 条红线内部自审逻辑
- 新增内容与 19 节固定输出模式兼容
- 所有公式保持 `$...$` 内联 LaTeX 形式
- Markdown 语法正确，表格对齐

---

## 📄 许可

本技能基于 Claude Code Skill 开放标准构建。源代码以 [MIT 许可](LICENSE) 发布。

---

## 🙏 致谢

本技能的设计借鉴了以下 Claude Code 技能生态中的最佳实践：
- **vision-skill**（LearningByDoingNow）— 多厂商视觉桥接，让纯文本模型看图
- **nature-reader** — 术语账本、源锚定架构
- **nature-reviewer** — 15 条红线、洞察 ID 系统
- **nature-paper-card** — 16 节固定卡片模式（启发 19 节输出）
- **citation-check-skill** — Two-Pass 架构
- **survey** — WYSIATI 认知校准（Kahneman 双过程理论）
- **scipilot-figure-skill** — 图表反欺骗检查、色盲安全配色
- **scientific-figure-generator** — 论文类型→图表结构映射
- **paper-spine** — 贡献优先方法论、防跳过规则
- **academic-paper** — 反泄漏协议、故障路径目录
- **thesis-writing** — 状态文件 + 强制暂停关卡
