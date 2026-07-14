+++
title = "Release 25.01 Highlights"
date = 2025-01-03T00:01:00Z
type = "post"
description = "25.01 版本亮点。"
in_search_index = true
+++

新的一年带来了新的 Helix 版本！向 25.01 问好。这是一个特别大的版本，包含了来自 171 位贡献者的更改。**感谢**所有让这个版本成为可能的人！

刚接触 Helix？
Helix 是一个模式文本编辑器，内置支持多重选择、语言服务器协议（LSP）、tree-sitter，以及实验性的调试适配器协议（DAP）支持。

这个大版本有很多重大改进，让我们直接进入亮点。

## 补全

25.01 中的补全功能有两个重大更新。首先，补全现在支持路径：

{{ asciinema(id="path-completion", width="94", height="25") }}

Helix 现在在插入文本时会检测可能的路径，并根据当前工作目录建议绝对路径或相对路径。这对于某些语言（如 Nix）特别有用，因为路径在这些语言中很常见。

路径补全特别令人兴奋，因为它是第一个不由 LSP 驱动的补全功能。添加路径补全的重构了代码库中假定补全只能来自语言服务器的部分，这应该会使将来添加新的补全源变得更加容易。

另一个重大变化来自代码片段补全的行为——目前仅由语言服务器发送——因为 25.01 增加了对代码片段制表位的支持。

{{ asciinema(id="snippet-tabstops", width="94", height="25") }}

在上面的录制中，rust-analyzer 为 `add` 建议了一个代码片段补全，其中包含函数参数的一些占位符。过去，Helix 在接受补全（按 Enter 键）时会清除这些占位符。现在，Enter 键会将光标跳转到第一个制表位。Tab 键可用于切换到下一个制表位。输入任何其他字符都会替换占位符文本。

## 诊断信息

传统上，LSP 诊断信息显示在编辑器右上角的右对齐位置。虽然这种显示方式很直接，但在小终端尺寸或语言服务器发送长诊断消息时可能难以阅读。25.01 增加了一种新的方式来_内联_渲染诊断信息。

{{ asciinema(id="inline-diagnostics", width="94", height="25") }}

内联诊断利用内部虚拟文本系统在文档中诊断范围的对应位置渲染诊断信息。诊断消息出现在行尾或相应代码行之间。

内联诊断目前默认禁用，因为我们还在调整显示和修复错误，可以通过如下配置启用：

```toml
# ~/.config/helix/config.toml
[editor]
# 在行尾显示诊断信息的最低严重级别：
end-of-line-diagnostics = "hint"

[editor.inline-diagnostics]
# 在主光标所在行显示诊断信息的最低严重级别。
# 注意：`cursor-line` 诊断在插入模式下隐藏。
cursor-line = "error"
# 在其他行显示诊断信息的最低严重级别：
# other-lines = "error"
```

将 `end-of-line-diagnostics`、`cursor-line` 或 `other-lines` 设置为 `"disable"` 以外的任何值都会启用内联诊断。

## 表格化选择器

选择器 UI 组件是 Helix 的核心：它是一种高效且可读的方式，用于在文件和感兴趣的位置（如 LSP 诊断和符号）之间跳转。在 25.01 中，选择器组件经过了重大改造，项目以表格形式排列。

{{ asciinema(id="tabular-pickers", width="94", height="25") }}

简单选择器（如文件选择器（`<space>f`））只需要一列，保持不变。但具有多个信息字段的选择器现在会在结果窗格顶部显示列名。行仍然对应要选择的单个项目，但现在有列来水平对齐项目之间相同类型的内容。例如，诊断选择器现在有三列——"severity"（严重级别）、"code"（代码）和"message"（消息）——而 LSP 符号选择器有两列——"kind"（类型）和"name"（名称）。

每列都可以单独筛选：默认筛选一列，其他列可以使用查询语法进行筛选：`%<column name> filter text`。例如，诊断选择器现在可以通过搜索 `%severity error` 筛选为仅显示错误。筛选文本会进行模糊匹配，列名可以通过前缀指定，因此在诊断选择器中搜索 `%s e` 的行为与搜索 `%severity error` 相同。

{{ asciinema(id="interactive-global-search", width="94", height="25") }}

伴随着这一变化，`global_search` 命令（`<space>/`）已被重构，以动态更新选择器的查询。这允许以交互方式在代码库中搜索正则表达式。然后可以使用类似 `%path filename` 的查询单独筛选文件名。

## 宏键绑定

对编写为宏的键绑定的初步支持已在 25.01 中落地。宏键绑定可以通过在输入字符串前加 `@` 来编写。

```toml
# 将 `<space>y` 键绑定改为拖拽到 `a` 寄存器而不是系统剪贴板：
[keys.normal]
space.y = "@\"ay"
```

宏键绑定使用与用 `q` 录制和用 `Q` 执行的宏相同的基础设施，因此上面的行为就像您键入了：

1. `"`：打开寄存器选择弹窗
2. `a`：选择 'a' 寄存器
3. `y`：将选择内容复制到寄存器

宏弥补了键绑定的一个空白：像寄存器选择弹窗这样的组件输入不会暴露为命令，因此像选择寄存器这样的操作本来无法绑定。

对宏键绑定的初步支持限制了绑定不能在命令序列中使用：您还不能将宏与常规命令组合。例如，您可以在像 `["@\"a", "yank"]` 这样的序列中将上面的 `y` 替换为其命令 `yank`，但这目前还不支持。

## 注释

25.01 包含对注释的改进，在编写行注释（如内联文档）时提供了更流畅的体验。

{{ asciinema(id="25.01-comment-improvements", width="94", height="25") }}

行注释现在默认延续：在插入模式下按 Enter 键，或在正常模式下光标位于行注释上时按 `o`/`O`，现在会延续该注释，插入当前注释令牌前缀和一个尾随空格字符。现在按 Enter 时会一致地去除尾随空格，因此 `<ret><ret>` 可以用于在内联 Markdown 文档等场景中在两个段落之间插入分隔。

`join_selections` 和 `join_selections_space` 命令（`J` 和 `A-J`）也得到了改进，现在在合并行注释时会去除注释令牌。这使得编辑现有注释或修复段落之间的间距更加容易。

## 总结

一如既往，这只是 25.01 版本的亮点。查看完整的[更新日志]了解详情。

在 [Matrix 空间][matrix] 中讨论使用和开发问题，并在 [GitHub 仓库][helix-git] 中关注 Helix 的开发进展。

[更新日志]: https://github.com/helix-editor/helix/blob/master/CHANGELOG.md#2501-2025-01-03
[helix-git]: https://github.com/helix-editor/helix/
[matrix]: https://matrix.to/#/#helix-community:matrix.org