# story-page

> 🌍 [English README](README.en.md)

一个 [Claude Code](https://claude.com/claude-code) skill，把你和 Claude Code 共创一个项目的过程，自动整理成一页**可以录短视频的故事页**。

不是 commit log，不是流水账——它从你的对话记录和代码里**提取关键决策时刻**，渲染成一段精挑细选的对话剧本。

![story page preview](preview.png)

<details>
<summary>📸 更多预览</summary>

![chapter view](preview-2.png)

![chapter with note](preview-3.png)

</details>

> 想看完整效果？打开 `example.html` 在浏览器里翻一遍就知道了。

---

## 它适合谁

- vibe Coding 做出了一些自己满意的小产品，想留下记录，但回顾全部沟通过程太长太繁琐
- 想给项目做一个介绍页，但又不想写长文档
- 想拍短视频/写文章介绍产品，需要一个**视觉上好看、暂停时能看懂**的页面

---

## 安装

把整个 `story-page/` 文件夹放到你的 Claude Code skills 目录下：

```bash
mkdir -p ~/.claude/skills
cd ~/.claude/skills
git clone https://github.com/jasminemo1110/story-page.git
```

完成后，Claude Code 会自动识别这个 skill。

---

## 使用

在任意项目目录下打开 Claude Code，输入：

```
/story
```

Claude 会自动：

1. 读取 `git log`、`README.md`、`CLAUDE.md` 和最近的对话记录
2. 提取 4–7 个关键决策时刻
3. 用对话气泡的形式渲染成一个 HTML 文件，输出到 `<项目根目录>/story.html`

### 在命令后附加方向，效果会好很多

直接 `/story` 也能跑，但**附几句话告诉 Claude 你最想突出什么**，结果会精准得多。常见用法：

```
/story 重点讲跑道概念是怎么诞生的
/story 我想突出的是父母真的在用这一点，技术细节都可以省略
/story 只讲后半段，从用户开始用起
/story 已经讲过的不要重复，只挑这几天的新进展
```

把你想突出的角度、想忽略的内容、目标读者是谁，都写出来——Claude 会照着你的意图筛选章节、调整语气。

---

## 续集：第二次、第三次生成

当你已经为某个项目生成过故事页之后，再次运行 `/story` 会**自动接着上一集讲**，而不是重新写一遍：

- 文件名：`story.html` → `story-2.html` → `story-3.html`，互不覆盖
- 大标题保持一致，封面右上角显示 `Vol.2` 标记
- 章节序号**接着上一集编**——上一集到 CHAPTER 07，下一集就从 CHAPTER 08 开始
- 副标题和时间标签会根据新的内容重写
- 已经讲过的章节不会重复出现，只挑新内容

适合：项目分阶段发布、产品有重大新版本、想做成系列内容。

---

## 自定义封面与署名

JSON 里有这几个字段控制封面和页脚：

| 字段 | 控制什么 | 示例 |
|---|---|---|
| `cover_tag` | 封面顶部小胶囊标签 | `"茉白Jasmine × Claude Code 共创"` |
| `title` | 大标题（用 `{accent: "..."}` 包裹想橘色高亮的词；用 `{br}` 强制换行） | `"{accent: \"0基础文科生\"}{br}一个月做出 5 个产品"` |
| `subtitle` | 副标题 | 一句话定调 |
| `meta` | 标题下方小字 | `"2026 年 5 月 · Skill · 第一个开源产品"` |
| `created_by` | 页脚个人署名 | `"茉白"` → 渲染成 `Created by 茉白` |
| `closing` | 页脚最后一行（含日期，与正文同语言） | `"故事讲到 2026 年 5 月 5 日 · 未完待续"` |

Claude 生成时会优先从 `git config user.name` 读你的名字，或直接问你一下。

生成后所有文字都可以手动改——HTML 文件顶部的 `<script id="story-data">` JSON 块里，**不用动 CSS、不用动 HTML 结构**。

> **手动改 JSON 的小提醒：** 想用引号引用一段话时，请用中文方角引号 `「」` / `『』`，不要用普通双引号 `"`——后者会破坏 JSON 语法导致页面变空白。

---

## 不用 Claude Code 也能用吗？

可以，**手动模式**：

Skill 这个机制本身只有 Claude Code 支持（`SKILL.md` 是 Claude Code 独有的格式），但 `template.html` 是纯 HTML，谁都能用：

1. 把 `template.html` 复制一份重命名为 `story.html`
2. 用任何 AI 工具（Cursor、ChatGPT、Gemini 等）让它根据你的项目内容，**只填 JSON 那部分**——把里面的 `cover_tag`、`title`、`chapters` 等字段写出来
3. 把生成的 JSON 替换 `<script id="story-data">` 里的内容
4. 浏览器打开就行

封面的 `cover_tag` 字段你可以填 `"我 × Cursor 共创"` / `"我 × ChatGPT 共创"` / 任何你想写的——这只是一行文本。

---

## 文件结构

```
story-page/
├── SKILL.md         # 给 Claude 的执行指令（生成规则、章节挑选、对话语气）
├── template.html    # 故事页模板，所有样式在这里
├── example.html     # 真实样例：作者从 0 基础到 5 个产品 + 这个 skill 自己的诞生故事
├── preview.png      # README 顶部预览图
├── README.md        # 中文文档
├── README.en.md     # English documentation
└── LICENSE          # MIT License
```

---

## 设计取舍

- **每章只讲一个决策点**——不是功能清单，不是提交记录
- **静止时能看懂，滚动时有节奏**——专门为短视频暂停画面而设计
- **用户是想法的来源**，Claude 是回应者——这个角色分工写在了 SKILL.md 里
- **橘色 `#D97757`** 是 Claude 品牌色，关键词高亮和章节序号都用它
- **JSON 与 CSS 完全分离**——以后想改字体、改间距、改颜色，只动一处

---

## 一键导出截图

故事页右上角有一个橘色按钮「📸 导出截图」，点一下会把整页拆成多张 PNG，**打包成一个 zip 下载**：

- `01-cover.png` —— 封面
- `02-chapter-01.png` ~ `0N-chapter-NN.png` —— 每章一张（含章节标题 + 旁白 + 对话 + 批注）
- 最后一章 PNG 会**把页脚（日期 + 署名 + 永久水印）一起带上**，作为整个故事的收尾

每张 PNG 都带一层非常浅的斜向水印（约 8% 透明度，肉眼几乎不可见，截图放大才看得到），双行文字：

```
github：Story-page（JasmineMobai）
全平台：茉白Jasmine
```

既不影响观感、又能在内容被分享时保留来源标记。

**保存路径**：Chrome / Edge 会弹出系统级"另存为"对话框让你选目录；Safari / Firefox 会按浏览器默认下载位置保存（在 Chrome 设置里勾选"下载前询问每个文件的保存位置"也能让默认浏览器弹出对话框）。

适合场景：录短视频时用每章图片做画面切换、写公众号/小红书时分章节穿插展示。

> 这个功能用到了 [`html2canvas`](https://github.com/niklasvh/html2canvas) 和 [`JSZip`](https://github.com/Stuk/jszip)（CDN 加载，仅在点击按钮时才会请求，不影响页面打开速度）。打印场景下按钮自动隐藏。

---

## 关于页脚

每个生成的故事页底部有三行小字：

1. **故事讲到 X 月 X 日 · 未完待续** —— 由 `closing` 字段控制，记录这次生成的日期
2. **Created by ...** —— 你的署名，由 `created_by` 字段控制，可以留空
3. **Made with story-page · by 茉白Jasmine** —— 工具名 + 作者名，链向本仓库，**永久显示**

第三行是开源工具的常见做法（参考 Obsidian Publish、Notion 等），让用过这个工具的人也能找到来源。字号很小，不影响内容阅读。

---

## License

MIT — 见 [LICENSE](LICENSE)。

随便用、随便改、随便分发。如果你做出了自己的故事页，欢迎告诉我，我会很开心。
