在构建 LLM 相关应用程序时，分块是将大块文本分解成更小的片段的过程。这是一项重要的技术，一旦我们使用LLM进行向量化内容，它有助于优化我们从矢量数据库返回的内容的相关性。在这篇博文中，我们将探讨它是否以及如何帮助提高LLM相关应用的效率和准确性。

众所周知，我们在向量数据库中索引的任何内容都需要首先向量化。分块的主要原因是为了确保我们向量化的内容尽可能少，但在语义上仍然相关。

例如，在语义搜索中，我们对文档语料库进行索引，每个文档都包含有关特定主题的有价值的信息。通过应用有效的分块策略，我们可以确保我们的搜索结果准确地捕捉用户查询的本质。如果我们的块太小或太大，可能会导致搜索结果不精确或错过显示相关内容的机会。根据经验，如果文本块在没有周围上下文的情况下对人类有意义，那么它对语言模型也有意义。因此，找到语料库中文档的最佳块大小对于确保搜索结果的准确性和相关性至关重要。

在这篇文章中，我们将探讨几种分块方法，并讨论在选择分块大小和方法时应考虑的权衡。最后，我们将提供一些建议，以确定适合应用程序的最佳块大小和方法。

## 向量化短内容和长内容
当我们向量化内容时，我们可以根据内容是短（如句子）还是长（如段落或整个文档）来预测不同的行为。

当向量化一个句子时，所得向量集中于该句子的特定含义。与其他句子嵌入相比，比较自然会在该级别上进行。这也意味着向量化可能会错过段落或文档中更广泛的上下文信息。

当向量化完整的段落或文档时，向量化过程会考虑整体上下文以及文本中句子和短语之间的关系。这可以产生更全面的向量表示，捕获文本的更广泛的含义和主题。另一方面，较大的输入文本大小可能会引入噪音或削弱单个句子或短语的重要性，从而使得在查询索引时找到精确匹配变得更加困难。

查询的长度也会影响向量之间的相互关系。较短的查询（例如单个句子或短语）将专注于细节，并且可能更适合与句子级嵌入进行匹配。跨越多个句子或段落的较长查询可能更适合段落或文档级别的嵌入，因为它可能正在寻找更广泛的上下文或主题。

索引也可以是非同质的，并且包含不同大小的块的嵌入。这可能会在查询结果相关性方面带来挑战，但也可能会产生一些积极的后果。一方面，由于长内容和短内容的语义表示之间的差异，查询结果的相关性可能会波动。另一方面，非同质索引可能会捕获更广泛的上下文和信息，因为不同的块大小代表文本中不同的粒度级别。这可以更灵活地适应不同类型的查询。

## 分块注意事项
有几个变量在确定最佳分块策略方面发挥着作用，这些变量根据用例而变化。以下是需要牢记的一些关键方面：

+ 被索引的内容的性质是什么？是否正在处理长文档（例如文章或书籍）或较短的内容（例如推文或即时消息）？答案将决定哪种模型更适合目标，以及应用哪种分块策略。
+ 使用哪种向量化模型，它在什么块大小上表现最佳？例如，sentence-transformer模型在单个句子上效果很好，但像text-embedding-ada-002这样的模型在包含 256 或 512 的块上表现更好。
+ 对用户查询的长度和复杂性有何期望？它们会简短而具体，还是长而复杂？这也可能会告诉你选择对内容进行分块的方式，以便嵌入式查询和嵌入式块之间有更紧密的相关性。
+ 检索到的结果将如何在特定应用程序中使用？例如，它们会用于语义搜索、问答、摘要或其他目的吗？例如，如果结果需要输入到具有令牌限制的另一个 LLM 中，必须考虑到这一点，并根据想要适应请求的块数量来限制块的大小。

回答这些问题将能够开发一种平衡性能和准确性的分块策略，这反过来又将确保查询结果更相关。

## 分块方法
分块的方法有多种，每种方法可能适合不同的情况。通过检查每种方法的优点和缺点，我们的目标是确定应用它们的正确场景。

### 固定大小分块
这是最常见、最直接的分块方法：我们只需决定块中的标记数量，以及可选地确定它们之间是否应该有重叠。一般来说，我们希望在块之间保留一些重叠，以确保语义上下文不会在块之间丢失。在大多数常见情况下，固定大小的分块将是最佳路径。与其他形式的分块相比，固定大小的分块计算成本低且易于使用，因为它不需要使用任何 NLP 库。

以下是使用LangChain执行固定大小分块的示例：

> text = "..." # your text
>
> from langchain.text_splitter import CharacterTextSplitter
>
> text_splitter = CharacterTextSplitter(
>
>     separator = "\n\n",
>
>     chunk_size = 256,
>
>     chunk_overlap  = 20
>
> )
>
> docs = text_splitter.create_documents([text])
>

### “内容感知”分块
这些是利用我们正在分块的内容的性质并对其应用更复杂的分块的一组方法。这里有些例子：

#### 分句
正如我们之前提到的，许多模型都针对嵌入句子级内容进行了优化。当然，我们会使用句子分块，并且有多种方法和工具可用于执行此操作，包括：

+ 幼稚的分割：最幼稚的方法是按句点（“.”）和换行符分割句子。虽然这可能快速且简单，但这种方法不会考虑所有可能的边缘情况。这是一个非常简单的例子：

> text = "..." # your text
>
> docs = text.split(".")
>

+ NLTK：自然语言工具包 (NLTK) 是一个流行的 Python 库，用于处理人类语言数据。它提供了一个句子标记器，可以将文本分割成句子，帮助创建更有意义的块。例如，要将 NLTK 与 LangChain 结合使用，可以执行以下操作：

> text = "..." # your text
>
> from langchain.text_splitter import NLTKTextSplitter
>
> text_splitter = NLTKTextSplitter()
>
> docs = text_splitter.split_text(text)
>

+ spaCy：spaCy 是另一个用于 NLP 任务的强大 Python 库。它提供了复杂的句子分割功能，可以有效地将文本分割成单独的句子，从而在生成的块中更好地保留上下文。例如，要将 spaCy 与 LangChain 结合使用，可以执行以下操作：

> text = "..." # your text
>
> from langchain.text_splitter import SpacyTextSplitter
>
> text_splitter = SpaCyTextSplitter()
>
> docs = text_splitter.split_text(text)
>

#### 递归分块
递归分块使用一组分隔符以分层和迭代的方式将输入文本划分为更小的块。如果分割文本的初始尝试没有生成所需大小或结构的块，则该方法会使用不同的分隔符或标准在生成的块上递归调用自身，直到达到所需的块大小或结构。这意味着虽然块的大小不会完全相同，但它们仍然“渴望”具有相似的大小。

以下是如何将递归分块与LangChain结合使用的示例：

> text = "..." # your text
>
> from langchain.text_splitter import RecursiveCharacterTextSplitter
>
> text_splitter = RecursiveCharacterTextSplitter(
>
>     # Set a really small chunk size, just to show.
>
>     chunk_size = 256,
>
>     chunk_overlap  = 20
>
> )
>
> docs = text_splitter.create_documents([text])
>

#### 找出最适合应用程序的块大小
如果常见的分块方法（例如固定分块）无法应用于任务，那么这里有一些提示可以帮助找到最佳的分块大小。

+ 预处理数据：在确定应用程序的最佳块大小之前，需要首先预处理数据以确保质量。例如，如果数据是从网络检索的，可能需要删除 HTML 标签或增加噪音的特定元素。
+ 选择块大小范围：数据经过预处理后，下一步是选择要测试的潜在块大小范围。如前所述，选择时应考虑内容的性质（例如，短消息或冗长的文档）、使用的向量化模型及其功能（例如，长度限制）。目标是在保留上下文和保持准确性之间找到平衡。首先探索各种块大小，包括用于捕获更细粒度语义信息的较小块（例如，128 或 256 个令牌）和用于保留更多上下文的较大块（例如，512 或 1024 个令牌）。
+ 评估每个块大小的性能：为了测试各种块大小，可以使用多个索引或具有多个命名空间的单个索引。使用代表性数据集，为要测试的块大小创建嵌入并将它们保存在索引中。然后，可以运行一系列查询，可以评估其质量，并比较不同块大小的性能。这很可能是一个迭代过程，可以针对不同的查询测试不同的块大小，直到可以确定内容和预期查询的最佳性能块大小。

## 结论
在大多数情况下，对内容进行分块非常简单，但当开始偏离常规时，它可能会带来一些挑战。没有一种万能的分块解决方案，因此适用于一种用例的方法可能不适用于另一种用例。

