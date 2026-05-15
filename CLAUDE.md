# 维护说明（story-page skill）

本文件面向**维护这个 skill 的人**，记录改动时容易踩的坑。
它不是运行时规则——执行 skill 时该怎么做，看 `SKILL.md`。

## 1. 架构与职责边界

- `template.html` —— HTML / CSS / 渲染 JS 的**唯一真源**。生成的故事页必须与它结构一致，只替换 `<script id="story-data">` 里的 JSON。
- `SKILL.md` —— 只管 JSON 内容（选哪些章节、怎么写对话）。
- `example.html` —— 参考产物，不是真源。
- 改外观 / 结构 → 改 `template.html`；改"讲故事的规则" → 改 `SKILL.md`。

## 2. ⚠️ schema 三处同步（最重要的隐藏依赖）

故事页的数据结构散在三个地方，改一处必须同步另外两处：

1. `SKILL.md` 的 "5. Fill the JSON" 小节（schema 块 + Notes）
2. `template.html` 顶部注释块（约 L100–116）
3. `template.html` 的渲染 JS —— **这才是真正的契约**

特别注意：

- `who` 字段只能是 `"user"` 或 `"claude"`。渲染 JS 写死了 `t.who === 'claude'`（约 L196）——改了这个字面量而不改 JS，所有气泡会渲染到同一侧。
- 新增内容类型（`narrate` / `turns` / `note` / `closing` / `volume` / `chapter_start`）必须三处一起改。
- 新增 token 语法（`{accent:"..."}`、`{br}`）由渲染 JS 里的正则解析（约 L155 / L158），同样三处同步。

## 3. 改了 template.html 之后要重新生成的资源

`template.html` 的 UI 或渲染 JS 一变，下面这些就过期了：

- `example.html`
- `preview.png` / `preview-2.png` / `preview-3.png`

`SKILL.md` 的 RULE 0 已经警告过历史 UI 漂移（旧版的渐变线、气泡名字标签等）。改完模板记得重新生成示例页和预览截图，否则 README 和 RULE 0 的"真源"说法就对不上了。

## 4. 双 README 同步

`README.md`（中文）和 `README.en.md`（英文）是两份独立文件。改其中一份，记得同步另一份。

## 5. 移植到其他 agent（如 Codex）的指引

适配其他 agent 时**不能只做文案 sed 替换**。必须同时改的点：

- `template.html` 渲染 JS 里的 `t.who === 'claude'` 判断——否则气泡渲染错位（见第 2 节）。
- `SKILL.md` 里硬编码的 `~/.claude/skills/story-page/` 路径（见第 6 节）。
- transcript 读取路径与格式：`~/.claude/projects/<project-path-hash>/*.jsonl` 是 Claude Code 专有的，其他 agent 的位置和格式都不同。
- 作为"项目意图来源"读取的文件名假设：`CLAUDE.md` / `README.md`（其他 agent 可能是 `AGENTS.md` 等）。
- 品牌相关文案：橘色 `#D97757`、`cover_tag` 里的 "Claude Code 共创" 等。

> 反面例子：曾有一份本地副本做 Claude→Codex 适配时只 sed 了文案，把 schema 里的 `who` 改成了 `"Codex"`，却漏了渲染 JS 的 `t.who === 'claude'` 判断——结果所有对话气泡都挤到同一侧。这就是第 2 节那个耦合没被注意到的后果。

## 6. 硬编码路径清单

`SKILL.md` 多处硬编码了绝对路径，例如 `~/.claude/skills/story-page/template.html`。skill 被安装到别处、或被 fork 时需要留意这些路径不会自动适配。
