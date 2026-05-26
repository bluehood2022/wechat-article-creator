# wechat-article-creator

> 一个让 AI 帮你「一句话出活」的微信公众号自动排版 + 配图 skill。

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![OpenClaw](https://img.shields.io/badge/OpenClaw-Skill-07C160)](https://github.com/openclaw/openclaw)
[![Claude](https://img.shields.io/badge/Claude-Compatible-8A2BE2)](https://www.anthropic.com/claude)

---

## 这是个啥

把一篇文章（正文或链接）丢给 AI，它会：

1. **自动抓取/清洗正文**
2. **跟你确认风格**（默认值都填好了，不指定就用默认）
3. **调即梦 CLI 生成配图**（封面 + 每个章节，风格自动统一）
4. **吐出一份可以直接复制粘贴进公众号编辑器的 HTML**

你只要 **Cmd+A → Cmd+C → 进编辑器 Cmd+V**。完事。

> 这个 skill 是我（[@bluehood2022](https://github.com/bluehood2022)）从一次次踩坑里调教出来的——中文乱码、base64 图挂掉、等宽字体、emoji 方块、`position: fixed` 失效……每一条规则背后都是一个翻车现场。详见 [《我是怎么调教出一个公众号排版 skill 的》](#相关阅读)。

---

## 适用场景

- ✅ 公众号原创作者，每天为排版手酸
- ✅ 一人公司 / 自媒体，需要批量产出风格统一的内容
- ✅ 同一选题要发公众号 + 小红书 + 口播稿，想统一调性
- ✅ 想训练 AI 替自己干重复活的"普通人创造者"

---

## 前置条件

| 依赖                | 说明                                       | 必需？ |
| ------------------- | ------------------------------------------ | ------ |
| **OpenClaw** 或兼容 skill 系统的 Agent 框架（Claude Code、Cursor 自定义、自建 AI Agent 均可） | 用于加载和执行 SKILL.md | ✅      |
| **即梦 CLI**（dreamina）        | 自动生成配图；不装也能用，只是配图要手动补 | ⚠️ 推荐 |
| **web_fetch / Playwright**       | 抓取链接正文                               | ⚠️ 推荐 |

> 即梦 CLI 安装见 [dreamina-cli](https://github.com/Bistutu/dreamina-cli)（社区版）。没装的话，把"步骤 3 配图"换成你常用的工具（Midjourney、文心一格、SD 都行），把生成好的图 URL 喂回给 AI 即可。

---

## 安装方式

### 方式一：OpenClaw 用户（推荐）

```bash
# 1. 克隆到你的 skills 目录
cd ~/.openclaw/workspace/skills
git clone https://github.com/bluehood2022/wechat-article-creator.git

# 2. 重启或在对话里说一句 "重新加载 skills"
```

OpenClaw 会自动扫描 `skills/` 目录下所有的 `SKILL.md` 并注册。

### 方式二：Claude Code / Cursor / 其他 Agent

把 `SKILL.md` 的内容**复制到你的 Agent 系统提示词**，或者放到对方约定的 skill / rule 目录下。例如：

- **Cursor**：放到 `.cursorrules`
- **Claude Code**：放到 `~/.claude/CLAUDE.md` 或项目 `CLAUDE.md`
- **自建 Agent**：作为长期记忆 / 工具说明的一部分注入

### 方式三：纯人工使用

把 `SKILL.md` 当作排版规则手册自己看着用——告诉你的 AI「严格按这份规范输出 HTML」，效果也不差。

---

## 怎么用（30 秒上手）

最朴素的用法，**一句话**：

```
帮我把这篇文章排成公众号 HTML：
<贴正文 / 或者贴一个链接>
```

进阶用法，**指定风格**：

```
用极客像素风做成公众号文章：
https://example.com/your-article
```

更进阶，**自定义参数**：

```
帮我做成公众号文章：
- 主色调：#FF6B35（橙色）
- 配图风格：扁平商务
- 不要封面图
- 输出到 ./drafts/

<正文>
```

### 默认值（你不说就用这个）

| 参数     | 默认值                                       |
| -------- | -------------------------------------------- |
| 主色调   | `#07C160`（微信绿）                          |
| 背景     | 白色（兼容公众号编辑器）                     |
| 字体     | 无衬线（PingFang SC / Microsoft YaHei 优先） |
| 行距     | 1.8                                          |
| 配图风格 | 商务示意图                                   |
| 配图数量 | 封面 1 + 每章节 1                            |

---

## 内置风格模板

不想自己想 prompt？直接报名字：

| 名称           | 调用方式              | 适合内容                       |
| -------------- | --------------------- | ------------------------------ |
| **商务示意图** | "用商务风" *（默认）* | 干货、科普、行业分析           |
| **极客像素风** | "用极客像素风"        | 技术、AI、独立开发、产品复盘   |
| **简约扁平风** | "用简约扁平风"        | 设计、生活方式、个人成长       |
| **科技未来风** | "用科技未来风"        | 趋势预测、深度技术、未来主义   |

样张可以看 `examples/` 目录。

---

## 输出结构

skill 默认把产物组织成这样：

```
articles/
├── <文章名>.html        ← 直接复制粘贴用这个
└── images/
    ├── cover.png
    ├── section1.png
    ├── section2.png
    └── ...
```

打开 HTML → **Cmd+A 全选 → Cmd+C 复制 → 公众号编辑器 Cmd+V 粘贴**。完事。

---

## 关键避坑（skill 已经替你处理了，但你应该知道）

公众号编辑器是个非常挑剔的环境。这个 skill 内置了以下规则，**全部都是真实翻车换来的**：

- ✅ HTML 头部必须有 `<meta charset="utf-8">`，否则中文乱码
- ✅ 图片**只能用 URL 引用**，绝对不能用 `data:image/png;base64,`（公众号不解析）
- ✅ 全文用**无衬线字体**，禁用等宽（否则像 GitHub README）
- ✅ **不用 emoji**，用 `▶` `■` `[!]` `[YES]` 等 ASCII 符号代替（避免方块）
- ✅ 不用 `!important` / `@keyframes` / `position:fixed`（公众号会吃掉）
- ✅ 所有 `<div>` 改成 `<section>`（兼容性更好）
- ✅ 样式**全部内联**（公众号编辑器会剥掉 `<style>` 标签）
- ⚠️ 即梦图片签名链接**有时效**，长期保存建议传到公众号素材库

---

## 实战样张

仓库的 `examples/` 目录放了几篇真实做出来的文章 HTML，你可以下载打开看看效果：

- `examples/一人公司100Agent账单.html`（商务风）
- `examples/Copilot低代码.html`（极客像素风）
- `examples/一人公司毒鸡汤.html`（深色科技风）

---

## SKILL.md 是怎么写的

打开 [`SKILL.md`](./SKILL.md) 自己看。简单说，结构是：

```
---
name: wechat-article-creator
description: 一句话告诉 AI 这个 skill 干嘛的，决定它什么时候被自动调用
---

# 标题

**触发条件**：……（什么场景下用）

## 工作流程
1. 提取正文
2. 确认风格
3. 生成配图
4. 生成 HTML
5. 保存文件
6. 告知用户

## 字体规范 / 关键规则 / 模板 / 注意事项
```

**核心心法**：把你脑子里那套"每次都这么干"的 SOP 原原本本写下来，把你踩过的坑也写下来。skill 的本质是 SOP，不是黑科技。

---

## 想自己改 / 二开？

最常见的修改场景：

| 我想……                  | 改哪儿                                                      |
| ----------------------- | ----------------------------------------------------------- |
| 换默认主色              | SKILL.md → 第 2 节"确认风格"表格里的 `主色调` 默认值         |
| 加一种新的配图风格      | SKILL.md → "Dreamina 配图提示词模板" → 新加一行             |
| 改默认输出路径          | SKILL.md → 第 5 节"保存文件"                                |
| 不用即梦，用 MJ / SD    | 改第 3 节"生成配图"的工具调用方式                           |
| 换字体栈                | SKILL.md → "字体规范" 那段的 `font-family`                  |

修改完保存即可，AI 下次会读到新规则。

---

## 相关阅读

- 📖 [《我是怎么调教出一个公众号排版 skill 的》](https://mp.weixin.qq.com/your-article-link) — 这个 skill 的完整调教故事
- 🔧 [OpenClaw 官方文档](https://docs.openclaw.ai) — skill 系统底层原理
- 🎨 [Dreamina CLI](https://github.com/Bistutu/dreamina-cli) — 配图工具

---

## License

[MIT](./LICENSE) — 随便用、随便改、随便商用，记得保留作者署名就行。

---

## 致谢 & 贡献

调教这玩意儿踩了一个多月坑，欢迎 PR 补充更多翻车案例和规则。

如果这个 skill 帮到了你，给个 ⭐ Star 是最大的鼓励。

如果你做出了好玩的风格模板、好用的 prompt，欢迎提 PR 到 `references/` 目录。

---

*Made with 🦞 by [@bluehood2022](https://github.com/bluehood2022) — 一个不写代码也能造 AI 工具的普通人。*
