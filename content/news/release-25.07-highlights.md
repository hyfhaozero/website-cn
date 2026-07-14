+++
title = "Release 25.07 版本亮点"
date = 2025-07-15T00:01:00Z
type = "post"
description = "25.07 版本亮点。"
in_search_index = true
+++

期待已久的 25.07 版本终于来了。这个版本替换了 Helix 的一个核心组件，并在此之外添加了许多炫酷的功能。这个版本包含了来自 195 位贡献者的更改。**衷心感谢**所有让这个版本成为可能的人。

刚接触 Helix？Helix 是一个模式文本编辑器，内置支持多重选择、语言服务器协议（LSP）、tree-sitter，以及实验性的调试适配器协议（DAP）支持。

系好安全带；这些发布说明会有点技术性，因为我们将讨论新的 Tree-sitter 绑定 `tree-house`。在深入细节之前，让我们先看看更炫酷的功能。

## 文件浏览器

{{ asciinema(id="file-explorer", width="94", height="25") }}

25.07 在 `<space>e` 下添加了一个文件浏览器。文件浏览器是一个_选择器_，是 Helix 核心的类似 [telescope](https://github.com/nvim-telescope/telescope.nvim) 的 UI 组件。和大多数其他选择器一样，您可以在选项中模糊搜索。用 Enter 选择目录会打开该目录下的新文件浏览器，选择文件则会打开该文件。这对于以层级方式查看目录很有用。相比之下，常规的文件选择器（`<space>f`）会递归地打开一个包含目录内容的选择器。对于庞大的项目，文件浏览器可以是一个更精确的工具。

## LSP documentColors

语言服务器协议（LSP）规范中比较炫酷的功能之一是 [Document Color Request](https://microsoft.github.io/language-server-protocol/specifications/lsp/3.17/specification/#textDocument_documentColor)。这个请求允许客户端（Helix）向语言服务器（如 `tailwindcss-language-server` 或 `vscode-css-language-server`）询问文档中哪些范围对应 RGB 颜色。

在 25.07 中，Helix 现在向语言服务器请求文档颜色，并内联显示颜色的色块（一个小方块）。这就像 LSP 内联提示功能——显示类型——但这次是显示颜色。

{{ asciinema(id="lsp-document-colors", width="94", height="25") }}

## 新的命令模式功能

命令模式（`:`）用于执行_可输入的_命令。例如，`:write` 是一个可输入命令，它接受一个可选参数。`:quit` 也是，但它不接受参数。

命令模式的语法很精妙。它应该足够简单，以便像 `:write path/to/doc.md` 这样的常见操作易于输入。但它也需要转义空格的工具，如 `:write 'a b.txt'`。对于某些命令，拥有自定义或可扩展的语法也很有用，如 `:run-shell-command <complex shell-specific command>`。

25.07 包含了对所有用于解析和表示参数以及为命令行提供补全的代码的完全重写。这修复了解析和补全方面的许多错误，比如尝试补全名称中包含空格的文件，并引入了两个新功能：_标志_和_展开_。

### 标志

标志的工作方式就像您在 shell 命令中传递的标志一样。它们用于处理您想要以轻微修改行为执行命令的情况。

到目前为止，标志仅用于一小部分命令：`:write` 系列命令（以 `:write` 开头的任何命令）和 `:sort`。

25.07 移除了 `:rsort` 命令，并用 `:sort --reverse` 替代，简写为 `:sort -r`。而 `:write` 命令现在都接受一个 `--no-format` 标志。通常您希望在配置了自动格式化时格式化当前文档，但有时按原样写入文件也很有用。这些是标志的绝佳用例：您不应该仅仅为了调整小细节而需要额外的可输入命令。

标志会显示在可输入命令的信息框中，长版本（如 `--reverse`）可以自动补全。

### 展开

展开引入了一种特殊的语法来插值值。这些主要遵循 [Kakoune](https://github.com/mawww/kakoune) 的展开概念，并做了一些小的调整。

基于当前编辑器状态的变量可以写成 `%{variable_name}`。`%{buffer_name}` 打印当前焦点文档在状态栏中显示的名称，`%{cursor_line}` 打印主光标的从 1 开始计数的行号。

Shell 命令可以使用 `%sh{..}` 展开来执行。与上面的变量展开和新的、简单的 `:echo` 命令（在状态栏打印信息）一起，像这样的命令会在状态栏上打印当前行的 git blame：

```
:echo %sh{git blame -L %{cursor_line},+1 %{buffer_name}}
```

变量名称和展开类型（如 shell 命令的 `sh` 或 Unicode 的 `u`）都可以自动补全。

### 可扩展解析

探索这次重写的最初原因是重新审视命令行的解析方式。通过 25.07 的更改，命令模式可以更好地解析和补全文件名，允许使用标志和展开，并且还可以在行中切换到其他解析方法。

可输入命令 `:set-option` 和 `:toggle-option` 现在使用 [`serde_json` 的流式反序列化器](https://docs.rs/serde_json/latest/serde_json/struct.StreamDeserializer.html)来解析复杂的配置值，如列表。像 `:run-shell-command` 和 `:pipe` 这样的 shell 命令不再尝试解析命令名之后的部分。因此，您无需猜测如何转义 Helix 的解析规则，然后再转义 shell 的解析规则——也就是终极的引号地狱。

## Tree-house

在这个发布周期中，我们更换了用于与 Tree-sitter 交互的 crate，添加了从头构建的新 crate，并移除了官方绑定以及 Helix 中的大量旧代码。

本文的其余部分将讨论 Tree-sitter 和 Tree-house 的细节。想了解 Helix 25.01.1 之后更改的更多详情？查看[更新日志]以获取完整的代码更改集合。

#### Tree-sitter

不熟悉 [Tree-sitter](https://github.com/tree-sitter/tree-sitter)？从高层次来看，它是一个用于生成和使用快速、容错解析器的框架。在 `grammar.js` 文件中，您可以通过 [Grammar DSL](https://tree-sitter.github.io/tree-sitter/creating-parsers/2-the-grammar-dsl.html) 编写解析器规则，然后使用 `tree-sitter` CLI 工具来生成和测试解析器。

编辑器等工具可以使用您通过 Tree-sitter C 库或特定语言绑定定义的解析器来解析语法树并对其进行操作。您如何利用语法树取决于您的想象力！语言服务器可以使用 Tree-sitter 作为其解析器，像 [Difftastic](https://github.com/Wilfred/difftastic) 这样的差异比较工具可以生成语法感知的差异，像 [Codebook](https://github.com/blopker/codebook) 这样的语言服务器可以进行语法感知的拼写检查扫描。甚至 [GitHub 也使用 tree-sitter](https://docs.github.com/en/repositories/working-with-files/using-files/navigating-code-on-github) 进行代码导航和某些语言的语法高亮。

处理已解析树的一个非常强大的工具是 _Tree-sitter 查询_。查询是一种对子树进行模式匹配并_捕获_节点以供后续使用的方法。对于编辑器，您可能会使用一个通常称为 `highlights.scm` 的查询来捕获树节点（如 Rust 关键字），以便根据当前主题高亮节点的文本。

和语法树一样，查询的应用只受限于您的想象力。我们目前在 Helix 中为高亮、缩进和文本对象（识别函数、参数等）使用查询。未来，代码折叠、拼写检查和代码导航也可能使用 tree-sitter 查询。

#### Helix 中的历史

Helix 甚至在首次公开发布之前就通过官方 Rust 绑定（[`tree-sitter`](https://crates.io/crates/tree-sitter) crate）依赖 Tree-sitter 进行语法高亮。`tree-sitter` crate 包装了 C 库，相当底层。我们还需要一个高亮器，这由另一个 crate 提供：`tree-sitter-highlight`。

[`tree-sitter-highlight`](https://crates.io/crates/tree-sitter-highlight) 提供了一个语法高亮器，它接受语言查询和文档文本，可以迭代产生高亮事件。Helix 然后在渲染可见文档时使用高亮迭代器。这在 `tree-sitter-highlight` 中开箱即用，对于像一次性高亮文档这样的简单用例，`tree-sitter-highlight` 就足够了。

`tree-sitter-highlight` 的问题是它不支持增量工作。创建新的高亮迭代器意味着完全重新解析文档以及重新分析查询。这是浪费的，因为 Tree-sitter 可以重用查询。而且 Tree-sitter 中的解析可以增量工作：您可以将旧的语法树交给 Tree-sitter，它会更快地解析新版本的文档。

因此，Helix 早期的高亮器是 `tree-sitter-highlight` 的一个分支，受 Tree-sitter 在 [Atom 编辑器](https://github.com/atom/atom) 中的使用启发，它将已解析的树（`Syntax` 类型）和 `tree_sitter::Query` 从高亮器中分离出来。理想情况下，我们希望有一天能将这个高亮器提取到自己的 crate 中，以便轻松地与其他工具共享。

然而，这个高亮器变得越来越难以维护。对长期存在问题的修复要么太大无法实施，要么完全与这个高亮器的设计不兼容，而且代码难以理解。

#### Tree-house 登场

在这个版本中，我们用一个新的 crate 替换了高亮器：[`tree-house`](https://github.com/helix-editor/tree-house)。我们基于早期高亮器的经验从头编写了 Tree-house。

Tree-house 倾向于那些运行良好的方面，比如将解析与查询分离以及在解析期间确定注入。它避开了那些运行不佳的方面，比如将高亮暴露为 `Iterator`。Tree-house 的代码被分解为更小、更容易理解的组件，它修复了我们以前无力解决的长期问题。Tree-house 也为未来的改进（如并行解析）打开了大门。

Tree-house 的主要优势在于它对称为_注入_的功能的稳健处理。

#### 注入，一棵树的树

_注入_是 `tree-sitter-highlight` 中的一个概念。注入查询捕获应该"切换"到另一种语言的节点。例如，在 Markdown 中，您可以使用如下代码围栏：

    ```rust
    println!("Hello, world!")
    ```

您会期望该代码围栏的内容被高亮为 Rust 代码。其工作原理是，整个 Markdown 文档的完整文本使用 Markdown Tree-sitter 解析器进行解析。Markdown 的 `injections.scm` 查询告诉 Helix，这个代码围栏的内容应该被视为 Rust 代码。然后 Helix 为该文档范围运行 Rust Tree-sitter 解析器并创建语法树。

然后，在需要高亮此文档时，Markdown 的 `highlights.scm` 查询为 Markdown 部分定义高亮。而当我们到达文档的 Rust _层_时，Rust 的 `highlights.scm` 会接管。

虽然这是一个简单的例子，但即使在复杂情况下，注入也能稳健地工作。例如，这个版本增加了对 Rust 文档注释中 Markdown 注入的支持，导致像这样的深度嵌套注入：

```rust
/// 一个解析 **内容** 的类型
///
/// 这是一个文档注释，所以它应该在常规注释高亮之上
/// 具有 _Markdown_ 高亮。
///
/// # 一级标题
///
/// 知道我们可以用 Markdown 做什么吗？注入 Rust！
///
///     println!("Hello, world!");
pub struct Parser(/* ... */);
```

在这样一个 Rust 文件中，覆盖整个文档的_根_层自然是 Rust。然后每个文档行注释（`///`）将其之后的内容解析为 Markdown 文档，_组合_——意味着这些范围被整体视为一个 Markdown 层。而在其中，缩进块应该像代码围栏一样，这是另一个 Rust 层。

在内部，Tree-house 将这种_层_概念表示为一棵树。整体的 `Syntax` 类型为其文件类型有一个根层，下面有子层用于其所有注入。而子层本身也可以注入其他层，依此类推。因此，这些层形成了一棵树。并且每个层都被解析，使其拥有自己的语法树，形成一种_树的树_。

#### 增量注入

注入之前早在 [22.03 发布说明](@/news/2022-03-28-release-22.03-highlights.md)中讨论过，当时添加了对_组合_注入的支持，就像那些 Markdown 注释一样。那年晚些时候，[22.12](@/news/2022-12-06-release-22.12-highlights.md) 带来了_增量注入_。这一变化减少了为具有许多注入的文档重新解析和重新运行注入查询所做的不必要工作。切换到 Tree-house 进一步改进了增量注入，使得注入层只在实际因任何编辑集而发生变化的层进行重新解析和重新运行注入查询。

要更直观地理解其工作原理，想象一个大型 Markdown 列表。Markdown Tree-sitter 解析器实际上分为两个：一个用于块级语法（如代码围栏），另一个用于"内联"语法（如粗体、斜体和内联代码）。Markdown 解析器为列表项等情况注入"内联 Markdown"解析器，因此 Markdown 中一个非常大的列表意味着为每个列表项注入数千个小的"内联"解析器。

切换到 Tree-house 后，在大型列表中编辑一个列表项只会导致根层和被编辑的"内联"层重新解析和重新运行注入查询——这是所需的最小工作量。

#### Locals

`tree-sitter-highlight` 中另一个有用的概念是 _locals_。`locals.scm` 是一个查询，用于标记那些应将其高亮应用于同一_作用域_内任何后续_引用_的节点。

想象一个简单的 Rust 函数：

```rust
fn add(a: usize, b: usize) -> usize {
    a + b
}
```

这个函数的 `a` 和 `b` 参数应该被高亮为参数——根据主题可能使用不同的高亮。为了跟踪这些信息，locals 查询捕获函数参数（如 `a` 和 `b`）的节点，以及_作用域_：本例中是函数体。作用域内的任何_引用_——同样由 locals 查询捕获——应继承定义的高亮。

Tree-house 对 locals 采用了不同的方法，解决了 Helix 中长期存在的问题。由于高亮器只对您屏幕上可见的小范围运行，每当定义移出视野时，locals 就会消失。

{{ asciinema(id="viewport-locals-before", width="94", height="25") }}

注意当参数 `slice`、`char_idx` 和 `n` 移出视野时，它们是如何失去参数高亮（下划线）的。

使用 Tree-house，定义在解析时被跟踪，并以树格式存储，像注入一样，用于快速查找。因此，代码的当前视图并不影响。无论定义是否在视野内，参数都会被正确高亮。

{{ asciinema(id="viewport-locals-after", width="94", height="25") }}

现在，无论您深入到函数多远，`slice`、`char_idx` 和 `n` 参数都能保持其高亮。

#### 适用于所有功能的注入

Tree-house 带来的一个不错的可用性提升是，Tree-house 的 `Syntax` 类型——对应于已解析的文档——具有在注入层之间平滑工作的函数。`Syntax` 被组织为一棵树的树，因此查找注入层的时间复杂度为对数级别，而不是完全扫描所有层。

在此基础上，一个镜像 Tree-sitter C 库中 `TreeCursor` 的 `TreeCursor` 类型可以跨注入层移动，其 API 与 Tree-sitter 的 `TreeCursor` API 几乎相同。新的 `QueryIter` 类型提供了在文档中所有注入层运行任何查询的能力——不仅仅是高亮。

将基于 Tree-sitter 的功能都利用注入来处理，将带来跨语言边界的一致体验。HTML `<script>` 标签中的注释标记和文本对象应遵循 JavaScript 规则而不是 HTML。Markdown 代码围栏中的缩进应遵循您正在编写的语言，而不是 Markdown。这些功能尚未合并或发布，但最终所有基于 Tree-sitter 的 Helix 功能都应像高亮一样表现一致。

## 总结

以上就是 25.07 版本的亮点，加上对我们 Tree-sitter 集成的深入探讨。查看完整的[更新日志]了解详情。

在 [Matrix 空间][matrix] 中讨论使用和开发问题，并在 [GitHub 仓库][helix-git] 中关注 Helix 的开发进展。

[更新日志]: https://github.com/helix-editor/helix/blob/master/CHANGELOG.md#2507-2025-07-15
[helix-git]: https://github.com/helix-editor/helix/
[matrix]: https://matrix.to/#/#helix-community:matrix.org