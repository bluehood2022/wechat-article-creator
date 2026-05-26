---
name: wechat-article-creator
description: "微信公众号文章自动排版+配图：从正文内容+风格要求出发，自动提取/生成配图，输出可直接粘贴到公众号编辑器的HTML。"
---

# 微信公众号文章排版 + 配图

**触发条件**：用户给一篇文章正文（或链接），要求自动排版/配图/生成公众号文章。

## 工作流程

### 1️⃣ 提取正文

- 用户给链接 → 用 `web_fetch` 尝试抓取；失败则用 Playwright 浏览器抓取
- 用户直接给文本 → 直接使用
- 清理广告、导航、水印等无关内容，只保留标题+正文

### 2️⃣ 确认风格

与用户确认以下风格参数（如未指定，用默认值）：

| 参数 | 默认值 | 说明 |
|------|--------|------|
| 主色调 | `#07C160`（微信绿） | 标题、标签、分隔线、高亮 |
| 背景 | 白色 | 公众号编辑器适配 |
| 配图风格 | 示意图/插画风 | 传给 Dreamina 的 prompt 关键词 |
| 字体 | 无衬线（全文统一） | 正文/标题/注释全部 sans-serif |
| 段落间距 | 1.8 line-height | 阅读舒适 |

### 3️⃣ 生成配图

用 Dreamina CLI 生成文章配图：

- **工具**：`dreamina text2image`
- **路径**：`/Users/jiejielu/.local/bin/dreamina`
- **参数**：`--ratio=16:9 --resolution_type=2k --poll=120`
- **数量**：封面图 1 张 + 每章节 1 张（通常 3-4 张）
- **prompt 设计**：根据文章主题和确认的风格生成，确保调性一致
- **下载**：保存到 workspace 目录，后续引用

### 4️⃣ 生成 HTML

输出微信公众号兼容的 HTML，关键规则：

- ✅ 必须包含 `<!DOCTYPE html>` + `<meta charset="utf-8">`（否则中文乱码）
- ✅ 使用内联 CSS（`style="..."`），无外部样式表
- ✅ 图片用 **纯 URL** 引用，绝对不能用 `data:image/png;base64,` 前缀（图片无法加载）
- ✅ 所有标签用 `<section>` 替代 `div`（公众号兼容性更好）
- ✅ 所有样式用 `margin/padding/font-size/color` 等基础属性
- ✅ 避免 `!important`、`@keyframes`、`position:fixed` 等不支持的属性
- ✅ 图片外层包 `border` + `border-radius` 做圆角效果

### 字体规范 ⚠️ 必须遵守

- **全文统一无衬线字体**，不要用等宽字体（monospace）
- 字体栈：`-apple-system, 'PingFang SC', 'Hiragino Sans GB', 'Microsoft YaHei', 'Helvetica Neue', Arial, sans-serif`
- `<body>` 和顶层 `<section>` 都要显式设置 `font-family`
- **禁止使用任何 emoji 表情字符**（如 🎮💊⚠️✓✗▎ 等），用纯文本符号代替：
  - 装饰符号用 `▶`、`■`、`■`
  - 警告用 `[!]`
  - 勾选用 `[YES]` / `[NO]`
  - 注释用 `//` 前缀

### 5️⃣ 保存文件

保存到 workspace：

```
/Users/jiejielu/.openclaw/workspace/articles/
  ├── <文章名>.html
  └── images/
      ├── cover.png
      ├── section1.png
      └── ...
```

### 6️⃣ 告知用户

告诉用户文件路径 + 使用方法：

> 1. 双击 HTML 文件用浏览器打开
> 2. Cmd+A 全选 → Cmd+C 复制
> 3. 打开公众号编辑器 → Cmd+V 粘贴

## Dreamina 配图提示词模板

根据章节内容生成 prompt，保持风格一致：

```
封面：[文章主题] + [风格关键词] + "16:9 composition, illustration style, professional"
章节：[章节核心概念] + [风格关键词] + "16:9, matching article theme"
```

常用风格关键词：
- 示意图/商务风：`modern business illustration, clean, professional, tech infographic`
- 像素/极客风：`8-bit pixel art style, retro game aesthetic, bright yellow accent, geeky`
- 简约/扁平风：`flat design, minimalist, clean vector illustration, white background`
- 科技/未来风：`futuristic tech illustration, holographic UI, dark theme, neon accents`

## 注意事项

- 公众号编辑器粘贴时，HTML 中的外部图片 URL 需要可访问（Dreamina 签名链接有有效期）
- 如果用户需要长期有效，建议把图片上传到公众号素材库
- 深色主题 HTML 在公众号白色编辑器里可能显示异常，优先做白底版
- 如果用户要改色，直接替换 HTML 中的主色值即可（grep -o 确认数量）
