# Claude Code Learning Skills

两个互相协作的 Claude Code Skill，构建AI驱动的个人学习系统。

## Skills

### karpathy-llm-wiki

基于 Andrej Karpathy 的 LLM Wiki 理念，AI 帮你维护一个持久化的个人知识库。

**核心操作：**
- **ingest** — 扔素材进去，AI 自动消化、归类、合并、建交叉引用
- **query** — 从你自己的笔记里找答案，不是网上瞎编
- **lint** — 自动检查断链、矛盾、过期内容

**特性：**
- Conversation Source Detection — 自动识别对话类素材，保留用户思考痕迹
- `## 💭 你的理解` 章节 — 专为学习场景设计的知识指纹格式

### learning-companion

自适应学习伴侣。每次学新东西时，先扫描你的 wiki 了解你已经会了什么，用你已有的概念做桥梁来讲解，最后自动归档。

**五阶段工作流：**
- Phase 0: 画像 — 扫描 wiki 了解你
- Phase 1: 桥梁 — 用已知概念类比
- Phase 2: 讲解 — 四层递进
- Phase 3: 确认 — 轻量检测
- Phase 3.5: 费曼时刻 — 引导你说出理解
- Phase 4: 归档 — 自动沉淀到 wiki

**回顾模式：** "昨天学了什么" → 从 profile.md 时间线 + log.md + index.md 三层查询

### 协作设计

两个 skill 通过 `## 💭 你的理解` 统一章节格式互相补位——无论走哪个 skill，你的思考都不会丢。

## 安装

```bash
# 克隆到 Claude Code skills 目录
git clone https://github.com/gxl109090/claude-learning-skills.git
cp -r claude-learning-skills/karpathy-llm-wiki ~/.claude/skills/
cp -r claude-learning-skills/learning-companion ~/.claude/skills/
```

## 使用方法

1. 创建一个 wiki 目录（如 `~/wiki/`）
2. 在该目录下启动 Claude Code
3. 说 "教我 [任何你想学的]" → learning-companion 激活
4. 说 "ingest 这篇文章" → karpathy-llm-wiki 激活
5. 说 "lint 一下 wiki" → 知识库健康检查
6. 说 "昨天学了什么" → 时间回顾
