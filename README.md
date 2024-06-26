# 大模型 RAG 实战教程 之 RAG潘多拉宝盒

## 一、LLMs 已经具备了较强能力了，为什么还需要 RAG(检索增强生成)?

尽管 LLM 已展现出显著的能力，但以下几个挑战依然值得关注：

- **幻觉问题**：LLM 采用基于统计的概率方法逐词生成文本，这一机制内在地导致其可能出现看似逻辑严谨实则缺乏事实依据的输出，即所谓的“郑重其事的虚构陈述”；
- **时效性问题**：随着 LLM 规模扩大，训练成本与周期相应增加。鉴于此，包含最新信息的数据难以融入模型训练过程，导致 LLM 在应对诸如“请推荐当前热门影片”等时间敏感性问题时力有未逮；
- **数据安全问题**：通用的 LLM 没有企业内部数据和用户数据，那么企业想要在保证安全的前提下使用 LLM，最好的方式就是把数据全部放在本地，企业数据的业务计算全部在本地完成。而在线的大模型仅仅完成一个归纳的功能；

## 二、介绍一下 RAG?

RAG（Retrieval Augmented Generation, 检索增强生成）是一种技术框架，其核心在于当 LLM 面对解答问题或创作文本任务时，首先会在大规模文档库中搜索并筛选出与任务紧密相关的素材，继而依据这些素材精准指导后续的回答生成或文本构造过程，旨在通过此种方式提升模型输出的准确性和可靠性。

![](技术架构图.png)
> RAG 技术架构图

## 三、RAG 主要包含哪些模块?

- 模块一：版面分析
  - 本地知识文件读取（pdf、txt、html、doc、excel、png、jpg、语音等）
  - 知识文件复原
- 模块二：知识库构建
  - 知识文本分割，并构建Doc文本
  - Doc文本 embedding
  - Doc文本 构建索引
- 模块三：大模型微调
- 模块四：基于RAG的知识问答
  - 用户query embedding
  - query 召回
  - query 排序
  - 将 Top K 个相关的 Doc 进行拼接，构建 context
  - 基于 query 和 context 构建 Prompt
  - 将 prompt 喂给大模型生成答案

## 四、RAG 相较于直接使用 LLMs进行问答 有哪些优点?

RAG（检索增强生成）方法赋予了开发者无需为每个特定任务重新训练大型模型的能力，仅需连接外部知识库，即可为模型注入额外的信息资源，从而显著提升其回答的精确度。这一方法尤其适用于那些高度依赖专业知识的任务。

以下是 RAG 模型的主要优势：

1. **可扩展性**：减小模型规模及训练开销，同时简化知识库的扩容更新过程。
2. **准确性**：通过引用信息源，用户能够核查答案的可信度，进而增强对模型输出结果的信任感。
3. **可控性**：支持知识内容的灵活更新与个性化配置。
4. **可解释性**：展示模型预测所依赖的检索条目，增进理解与透明度。
5. **多功能性**：RAG 能够适应多种应用场景的微调与定制，涵盖问答、文本摘要、对话系统等领域。
6. **时效性**：运用检索技术捕捉最新信息动态，确保回答既即时又准确，相比仅依赖固有训练数据的语言模型具有明显优势。
7. **领域定制性**：通过对接特定行业或领域的文本数据集，RAG 能够提供针对性的专业知识支持。
8. **安全性**：通过在数据库层面实施角色划分与安全管控，RAG 有效强化了对数据使用的管理，相较于微调模型在数据权限管理上的潜在模糊性，展现出更高的安全性。

## 五、对比一下 RAG 和 SFT，说一下两者有哪些区别？

实际上，对于 LLM 存在的上述问题，SFT 是一个最常见最基本的解决办法，也是 LLM 实现应用的基础步骤。那么有必要在多个维度上比较一下两种方法：

![](rag_sft_1.png)

当然这两种方法并非非此即彼的，合理且必要的方式是结合业务需要与两种方法的优点，合理使用两种方法。

![](rag_sft_2.png)

## 模块一：版面分析

### 为什么 需要 版面分析？

尽管RAG（检索增强生成）技术的核心价值在于其结合检索与生成手段以提升文本内容的精确度与连贯性，然而在一些具体应用领域，如文档解析、智能化写作及对话系统构建中，特别是在面对结构化或半结构化信息的处理需求时，其功能边界可能拓展至版面分析。这是由于此类信息往往嵌于特定的布局结构中，需要对页面元素及其相互关系进行深入理解。

此外，当RAG模型面对包含丰富多媒体或多模态成分的数据源，诸如网页、PDF文件、富文本记录、Word文档、图像资料、语音片段、表格数据等复杂内容时，为了能高效地摄取并利用这些非文本信息，具备基本的版面分析能力变得至关重要。这种能力有助于模型精准解析各类信息单元，并成功将它们融合成有意义的整体解读。

### step 1：本地知识文件获取

#### q1：如何进行 本地知识文件获取？

本地知识文件获取涉及从多种数据源（如.txt、.pdf、.html、.doc、.xlsx、.png、.jpg、音频文件等）提取信息的过程。针对不同类型的文件，需要采取特定的访问与解析策略来有效获取其中蕴含的知识。下面我们将介绍对于不同数据源数据的获取方式和难点。

#### q2：如何获取 富文本txt 中内容？

- 介绍：富文本 主要存储于 txt 文件中，因为排版比较整洁，所以获取方式比较简单
- 实战技巧：
  - [【版面分析——富文本txt读取】](https://articles.zsxq.com/id_x01sdalig305.html)

#### q3：如何获取 PDF文档 中内容？

- 介绍：PDF文档中数据比较复杂，包含文本、图片、表格等不同样式的数据，所以解析过程中会比较复杂
- 实战技巧：
  - [【版面分析——PDF 解析神器 pdfplumber】](https://articles.zsxq.com/id_q02kt8h2cdn9.html)
  - [【版面分析——PDF 解析神器 PyMuPDF】](https://articles.zsxq.com/id_qmjw5wgbhs1s.html)

#### q4：如何获取 HTML文档 中内容？

- 介绍：PDF文档中数据比较复杂，包含文本、图片、表格等不同样式的数据，所以解析过程中会比较复杂
- 实战技巧：
  - [【版面分析——网页HTML解析 BeautifulSoup】](https://articles.zsxq.com/id_ptqivfoinkp8.html)

#### q5：如何获取 Doc文档 中内容？

- 介绍：Doc文档中数据比较复杂，包含文本、图片、表格等不同样式的数据，所以解析过程中会比较复杂
- 实战技巧：
  - [【版面分析——Docx 解析神器 python-docx】](https://articles.zsxq.com/id_wshfpiwfn4t4.html)

#### q6：如何使用 OCR 获取图片内容？

- 介绍：光学字符识别（Optical Character Recognition, OCR）是指对文本资料的图像文件进行分析识别处理，获取文字及版面信息的过程。亦即将图像中的文字进行识别，并以文本的形式返回。
- 思路：
  - 1) 文字检测：解决的问题是哪里有文字，文字的范围有多少；
  - 2) 文字识别：对定位好的文字区域进行识别，主要解决的问题是每个文字是什么，将图像中的文字区域进转化为字符信息。
- 目前开源的OCR项目
  - [Tesseract](https://github.com/tesseract-ocr/tesseract)
  - [PaddleOCR](https://gitee.com/paddlepaddle/PaddleOCR)
  - [EasyOCR]( https://github.com/JaidedAI/EasyOCR)
  - [chineseocr](https://github.com/chineseocr/chineseocr)
  - [chineseocr_lite](https://github.com/DayBreak-u/chineseocr_lite)
  - [TrWebOCR](https://github.com/alisen39/TrWebOCR)
  - [cnocr](https://github.com/breezedeus/cnocr)
  - [hn_ocr](https://github.com/cnhnkj/hn_ocr)
- 理论学习：
  - [【版面分析——图片解析神器 OCR】](https://articles.zsxq.com/id_u8f1xl8v9hvd.html)
- 实战技巧：
  - [【版面分析——OCR神器 tesseract】](https://articles.zsxq.com/id_40jnz2gj6ttk.html)
  - [【版面分析——OCR神器 PaddleOCR】](https://articles.zsxq.com/id_zdut9rwjrh64.html)
  - [【版面分析——OCR神器 hn_ocr】](https://articles.zsxq.com/id_ciu0at0acror.html)

#### q7：如何使用 ASR 获取语音内容？

- 别称：自动语音识别AutomaTlc Speech RecogniTlon，(ASR)
- 介绍：将一段语音信号转换成相对应的文本信息，好比"机器的听觉系统”，它让机器通过识别和理解，把语音信号转变为相应的文本或命令。
- 目标：将人类的语音中的词汇内容转换为计算机可读的输入（eg:按键、二进制编码或者字符序列）
- 思路：
  - 声学信号预处理：为了更有效地提取特征往往还需要对所采集到的声音信号进行滤波、分帧等预处理工作，把要分析的信号从原始信号中提取出来；
  - 特征提取：将声音信号从时域转换到频域，为声学模型提供合适的特征向量;
  - 声学模型：根据声学特性计算每一个特征向量在声学特征上的得分;
  - 语言模型：根据语言学相关的理论，计算该声音信号对应可能词组序列的概率;
  - 字典与解码：根据已有的字典，对词组序列进行解码，得到最后可能的文本表示
- 理论教程：
  - [【版面分析 之 语音识别】](https://articles.zsxq.com/id_v6wuwr064b9j.html)
- 实战技巧：
  - [【版面分析 之 Speech-to-Text】](https://articles.zsxq.com/id_evjplrfsjmci.html)
  - [【版面分析 之 WeTextProcessing】](https://articles.zsxq.com/id_o8ur4u6eilbq.html)
  - [【版面分析——ASR神器 Wenet】](https://articles.zsxq.com/id_mw0bod4kdbre.html)
  -  [【版面分析 之 ASR神器训练】](https://articles.zsxq.com/id_rhvivt6wmuwh.html)

### step 2：知识文件复原

#### q1：为什么需要进行 知识文件复原？

本地知识文件获取包含对多源化数据（txt、pdf、html、doc、excel、png、jpg、语音等）进行读取之后，容易将一个多行段落分割成多个段落，从而导致段落遇到被分割，所以需要根据内容逻辑重新组织段落。

#### q2：如何对 知识文件进行复原？

1. 方法一：基于规则的知识文件复原
2. 方法二：基于 Bert NSP 进行上下句拼接

### step 3：版面分析———优化策略篇

- 理论学习：
  - [【版面分析———优化策略篇】](https://articles.zsxq.com/id_4hhb8tu6wtbm.html)

### step 4：Homework 

- 任务描述：使用上述方法对 【[SMP 2023 ChatGLM金融大模型挑战赛](https://tianchi.aliyun.com/specials/promotion/SMP2023ChatGLMChallenge)】的 【[ChatGLM评估挑战赛-金融赛道数据集](https://modelscope.cn/datasets/modelscope/chatglm_llm_fintech_raw_dataset/summary?spm=a2c22.12281978.0.0.13472420ZI1gfg)】进行版面分析
- 任务效果：分析各种方法效果和性能

## 模块二：知识库构建

### 为什么 需要 知识库构建？

在RAG（Retrieval-Augmented Generation）中构建知识库是至关重要的，原因包括但不限于以下几点：

1. **扩展模型能力**：大规模语言模型如GPT系列虽然具有强大的语言生成和理解能力，但受限于训练数据集的覆盖范围，它们可能无法准确回答一些基于特定事实或详细背景信息的问题。通过构建知识库，RAG可以补充模型自身的知识局限性，允许模型检索到最新、最准确的信息来生成答案。
2. **实时更新信息**：知识库可以实时更新和扩充，确保模型能够获取最新的知识内容，这对于处理时效性强的信息尤为关键，比如新闻事件、科技进展等。
3. **提高准确性**：RAG结合了检索与生成两个过程，在生成回答前先检索相关文档，从而提高了回答问题时的准确性。这样，模型生成的答案不仅基于其内部参数化的知识，还基于外部可靠来源的知识库。
4. **减少过拟合与hallucination（幻觉生成）**：大模型有时会因为过度依赖内在模式而出现hallucination现象，即生成看似合理实则无依据的答案。通过引用知识库中的确切证据，RAG可以降低此类错误产生的可能性。
5. **增强可解释性**：RAG不仅能提供答案，还能指出答案的来源，增强了模型生成结果的透明度和可信度。
6. **支持个性化及私有化需求**： 对于企业或个人用户，可以通过构建专属知识库满足特定领域或私人定制的需求，使得大模型能更好地服务于特定场景和业务。

综上所述，构建知识库对于RAG模型来说，是实现高效准确地检索并生成答案的核心机制之一，它极大地提升了模型在实际应用中的性能和可靠性。

### step 1：知识文本分块

- 为什么需要对文本分块？
  - **信息丢失的风险**：试图一次性提取整个文档的嵌入向量，虽然可以捕捉到整体的上下文，但也可能会忽略掉许多针对特定主题的重要信息，这可能会导致生成的信息不够精确或者有所缺失。
  - **分块大小的限制**：在使用如OpenAI这样的模型时，分块大小是一个关键的限制因素。例如，GPT-4模型有一个32K的窗口大小限制。尽管这个限制在大多数情况下不是问题，但从一开始就考虑到分块大小是很重要的。
- 主要考虑两个因素：
  - embedding模型的Tokens限制情况；
  - 语义完整性对整体的检索效果的影响；
- 实战技巧：
  - [【知识库构建——知识文本分块】](https://articles.zsxq.com/id_rkscmgr99w7v.html)
  - [【知识库构建——文档切分优化策略篇】](https://articles.zsxq.com/id_tj26lav19t2l.html)

### step 2：Docs 向量化（embdeeing） 

#### q1：什么是Docs 向量化（embdeeing）？

Embedding 也是文本语义含义的信息密集表示，每个嵌入都是一个浮点数向量，使得向量空间中两个嵌入之间的距离与原始格式中两个输入之间的语义相似性相关联。

> 例如，如果两个文本相似，则它们的向量表示也应该相似，这一组向量空间内的数组表示描述了文本之间的细微特征差异。

简单来说，Embedding 帮助计算机来理解如人类信息所代表的“含义”，Embedding 可以用来获取文本、图像、视频、或其他信息的特征“相关性”，这种相关性在应用层面常用于搜索、推荐、分类、聚类。

![](emb1.png)

#### q2：Embedding 是如何工作的？

举例来讲，这里有三句话：

1. “The cat chases the mouse” “猫追逐老鼠”
2. “The kitten hunts rodents” 小猫捕猎老鼠。
3. “I like ham sandwiches” 我喜欢火腿三明治。

如果是人类来将这三个句子来分类，句子 1 和句子 2 几乎是同样的含义，而句子 3 却完全不同。但我们看到在英文原文句子中，句子 1 和句子 2 只有“The”是相同的，没有其他相同词汇。计算机该如何理解前两个句子的相关性？

Embedding 将离散信息（单词和符号）压缩为分布式连续值数据（向量）。如果我们将之前的短语绘制在图表上，它可能看起来像这样：

![](emb2.png)

在文本被 Embedding 压缩到计算机可以理解的多维向量化空间之后，由于句子 1 和 2 的含义相似，它们会被绘制在彼此附近。句子 3 却距离较远，因为它与它们没有关联。如果我们有第四个短语 “Sally 吃了瑞士奶酪”，它可能存在于句子 3（奶酪可以放在三明治上）和句子 1（老鼠喜欢瑞士奶酪）之间的某个地方。

#### q3：Embedding 的语义检索方式对比关键词检索的优势？

1. **语义理解**： 基于 Embedding 的检索方法通过词向量来表示文本，这使得模型能够捕捉到词汇之间的语义联关系，相比之下，基于关键词的检索往往关注字面匹配，可能忽略了词语之间的语义联系。
2. **容错性**： 由于基于 Embedding 的方法能够理解词汇之间的关系，所以在处理拼写错误、同义词、近义词等情况时更具优势。而基于关键词的检索方法对这些情况的处理相对较弱。
3. **多语言支持**： 许多 Embedding 方法可以支持多种语言，有助于实现跨语言的文本检索。比如你可以用中文输入来查询英文文本内容，而基于关键词的检索方法很难做到这一点。
4. **语境理解**： 基于 Embedding 的方法在处理一词多义的情况时更具优势，因为它能够根据上下文为词语赋予不同的向量表示。而基于关键词的检索方法可能无法很好地区分同一个词在不同语境下的含义。

#### q4：Embedding检索存在哪些限制?

1. **输入词数限制**： 即便借助Embedding技术选取与查询最为匹配的文本片段供大型模型参考，词汇数量的约束依然存在。当检索覆盖的文本范围广泛时，为了控制注入模型的上下文词汇量，通常会对检索结果设定TopK的阈值K，但这不可避免地引发了信息遗漏的问题。
2. **仅支持文本数据**：  现阶段的GPT-3.5及诸多大型语言模型尚不具备图像识别功能，然而，在知识检索过程中，许多关键信息往往依赖于图文结合来充分理解。例如，学术论文中的示意图、财务报告中的数据图表，仅凭文本难以精准把握其内涵。
3. **大模型的胡编乱造**： 当检索到的相关文献资料不足以支撑大型模型准确回答问题时，为尽力完成响应，模型可能会出现一定程度的“即兴创作”，即在有限信息基础上进行推测与补充。

- 理论学习：
  - [【知识库构建—Doc 向量化】](https://articles.zsxq.com/id_b042uca0dpn4.html)
- 实战技巧：
  - [【Docs向量化——腾讯词向量】](https://articles.zsxq.com/id_kupfkifz6phb.html)
  - [【Docs向量化——sbert】](https://articles.zsxq.com/id_l5ew9h3iemt9.html)
  - [【Docs向量化——SimCSE】](https://articles.zsxq.com/id_03zew57t2zdk.html)
  - [【Docs向量化——text2vec】](https://articles.zsxq.com/id_9wtoqvta1cw9.html)
  - [【Docs向量化——SGPT】](https://articles.zsxq.com/id_nhmdc54h1j4q.html)
  - [【Docs向量化——BGE —— 智源开源最强语义向量模型】](https://articles.zsxq.com/id_woj99hecibdd.html)
  - [【Docs向量化——M3E：一种大规模混合embedding】](https://articles.zsxq.com/id_ltr45nojb0wo.html)

### step 3：Docs 构建索引

- 介绍
- 实战技巧：
  - **[【Docs构建索引——Faiss】](https://articles.zsxq.com/id_9adfmnhcppou.html)**
  - **[【Docs构建索引——milvus】](https://articles.zsxq.com/id_67uolswwomqx.html)**
  - **[【Docs构建索引—— Elasticsearch】](https://articles.zsxq.com/id_7evv3mtfdxbw.html)**

## 模块三：大模型微调

### 为什么 需要 大模型微调？

通常，要对大模型进行微调，有以下一些原因：

- 第一个原因是，因为大模型的参数量非常大，**训练成本非常高**，每家公司都去从头训练一个自己的大模型，这个事情的性价比非常低；
- 第二个原因是，**Prompt Engineering的方式是一种相对来说容易上手的使用大模型的方式，但是它的缺点也非常明显**。因为通常大模型的实现原理，都会对输入序列的长度有限制，Prompt Engineering 的方式会把Prompt搞得很长。

越长的Prompt，大模型的推理成本越高，因为推理成本是跟Prompt长度的平方正向相关的。

另外，Prompt太长会因超过限制而被截断，进而导致大模型的输出质量打折口，这也是一个非常严重的问题。

对于个人使用者而言，如果是解决自己日常生活、工作中的一些问题，直接用Prompt Engineering的方式，通常问题不大。

但对于对外提供服务的企业来说，要想在自己的服务中接入大模型的能力，推理成本是不得不要考虑的一个因素，微调相对来说就是一个更优的方案。

- 第三个原因是，Prompt Engineering的效果达不到要求，企业又有比较好的自有数据，能够**通过自有数据，更好的提升大模型在特定领域的能力**。这时候微调就非常适用。
- 第四个原因是，**要在个性化的服务中使用大模型的能力**，这时候针对每个用户的数据，训练一个轻量级的微调模型，就是一个不错的方案。
- 第五个原因是，**数据安全的问题**。如果数据是不能传递给第三方大模型服务的，那么搭建自己的大模型就非常必要。通常这些开源的大模型都是需要用自有数据进行微调，才能够满足业务的需求，这时候也需要对大模型进行微调。

### 如何对大模型进行微调？

#### q1：大模型的微调技术路线问题

从参数规模的角度，大模型的微调分成两条技术路线：

- 技术路线一：对全量的参数，进行全量的训练，这条路径叫全量微调FFT(Full Fine Tuning)。
- 技术路线二：只对部分的参数进行训练，这条路径叫PEFT(Parameter-Efficient Fine Tuning)。

#### q2：大模型的全量微调FFT 技术存在哪些问题

FFT也会带来一些问题，影响比较大的问题，主要有以下两个：

- 问题一：**训练的成本会比较高**，因为微调的参数量跟预训练的是一样的多的；
- 问题二：**灾难性遗忘(Catastrophic Forgetting)**，用特定训练数据去微调可能会把这个领域的表现变好，但也可能会把原来表现好的别的领域的能力变差。

#### q3：大模型的 PEFT(Parameter-Efficient Fine Tuning) 解决哪些问题

PEFT主要想解决的问题，就是FFT存在的上述两个问题，PEFT也是目前比较主流的微调方案。

从训练数据的来源、以及训练的方法的角度，大模型的微调有以下几条技术路线：

- 技术路线一：**监督式微调SFT(Supervised Fine Tuning)**，这个方案主要是用人工标注的数据，用传统机器学习中监督学习的方法，对大模型进行微调；
- 技术路线二：**基于人类反馈的强化学习微调RLHF(Reinforcement Learning with Human Feedback)**，这个方案的主要特点是把人类的反馈，通过强化学习的方式，引入到对大模型的微调中去，让大模型生成的结果，更加符合人类的一些期望；
- 技术路线三：**基于AI反馈的强化学习微调RLAIF(Reinforcement Learning with AI Feedback)**，这个原理大致跟RLHF类似，但是反馈的来源是AI。这里是想解决反馈系统的效率问题，因为收集人类反馈，相对来说成本会比较高、效率比较低。

不同的分类角度，只是侧重点不一样，对同一个大模型的微调，也不局限于某一个方案，可以多个方案一起。

微调的最终目的，是能够在可控成本的前提下，尽可能地提升大模型在特定领域的能力。

### 大模型LLM进行SFT操作的时候在学习什么？

- 预训练->在大量无监督数据上进行预训练，得到基础模型-->将预训练模型作为SFT和RLHF的起点。
- SFT-->在有监督的数据集上进行SFT训练，利用上下文信息等监督信号进一步优化模型-->将SFT训练后的模型作为RLHF的起点。
-  RLHF-->利用人类反馈进行强化学习，优化模型以更好地适应人类意图和偏好-->将RLHF训练后的模型进行评估和验证，并进行必要的调整。

### step 1：大模型微调训练数据构建

- 介绍：如何构建 训练数据？
- 实战技巧：
  - **[【大模型（LLMs）LLM生成SFT数据方法篇】](https://articles.zsxq.com/id_nvvujwg77sn8.html)**

### step 2：大模型指令微调篇

- 介绍：如何构建 训练数据？
- 实战技巧：
  - **[【大模型（LLMs）继续预训练篇】](https://articles.zsxq.com/id_ddtrij3zbcic.html)**
  - **[【大模型（LLMs）指令微调篇】](https://articles.zsxq.com/id_fuxxu83p423m.html)**
  - **[【大模型（LLMs）奖励模型训练篇】](https://articles.zsxq.com/id_1ax2wmszimz2.html)**
  - **[【大模型（LLMs）强化学习——PPO训练篇】](https://articles.zsxq.com/id_3ap59jndj6fa.html)**
  - **[【大模型（LLMs）强化学习——DPO训练篇】](https://articles.zsxq.com/id_89ivqmt4lw2j.html)**

## 模块四：文档检索

### 为什么 需要 文档检索？

文档检索 作为 RAG 核心工作，其效果对于下游工作至关重要。

虽然可以通过向量召回的方式从文档库里召回和用户问题相关的文档片段，同时输入到LLM中，增强模型回答质量。

常用的方式直接用用户的问题进行文档召回。但是很多时候，**用户的问题是十分口语化的，描述的也比较模糊，这样会影响向量召回的质量，进而影响模型回答效果**。

本章主要介绍 文档检索 过程中 存在的一些问题和对应的解决方法。

### step 1：文档检索负样本样本挖掘

- 介绍：在各类检索任务中，为训练好一个高质量的检索模型，往往需要从大量的候选样本集合中采样高质量的负例，配合正例一起进行训练。
- 实战技巧：
  - **[【文档检索——负样本样本挖掘篇】](https://articles.zsxq.com/id_cdtoftb6lfbb.html)**

### step 2：文档检索优化策略

- 介绍：文档检索优化策略
- 实战技巧：
  - **[【文档检索——文档检索优化策略篇】](https://articles.zsxq.com/id_3x6c1sog5fi2.html)**

## 模块五：Reranker

### 为什么 需要 Reranker？

基本的 RAG 应用包括四个关键技术组成部分：

- Embedding 模型：用于将外部文档和用户查询转换成 Embedding 向量
- 向量数据库：用于存储 Embedding 向量和执行向量相似性检索（检索出最相关的 Top-K 个信息）
- 提示词工程（Prompt engineering）：用于将用户的问题和检索到的上下文组合成大模型的输入
- 大语言模型（LLM）：用于生成回答

上述的基础 RAG 架构可以有效解决 LLM 产生“幻觉”、生成内容不可靠的问题。但是，**一些企业用户对上下文相关性和问答准确度提出了更高要求，需要更为复杂的架构。一个行之有效且较为流行的做法就是在 RAG 应用中集成 Reranker**。

### 什么是 Reranker？

**Reranker 是信息检索（IR）生态系统中的一个重要组成部分，用于评估搜索结果，并进行重新排序，从而提升查询结果相关性**。

在 RAG 应用中，主要在拿到向量查询（ANN）的结果后使用 Reranker，能够更有效地确定文档和查询之间的语义相关性，更精细地对结果重排，最终提高搜索质量。

### step 1：Reranker 篇

- 理论学习：
  - **[【RAG文档检索——Reranker 篇】](https://articles.zsxq.com/id_hrdg0abngzww.html)**
- 实战技巧：
  - **[【Reranker——bge-reranker篇】](https://articles.zsxq.com/id_ed7i2t32wdrz.html)**

## 模块六：RAG 评测面 

### 为什么需要 对 RAG 进行评测？

在探索和优化 RAG（检索增强生成器）的过程中，如何有效评估其性能已经成为关键问题。

### step 1：RAG 评测 篇

- 理论学习：
  - **[【RAG评测篇】](https://articles.zsxq.com/id_bs4s6x213o8i.html)**

## 模块七：RAG 开源项目推荐学习

### 为什么 需要 RAG 开源项目推荐学习？

前面已经带你走完了 RAG 的各个流程，下面将推荐一些 RAG 开源项目，帮助大佬们进行消化学习。

#### RAG 开源项目推荐 —— RAGFlow 篇

- 介绍：RAGFlow 是一款基于深度文档理解构建的开源 RAG（Retrieval-Augmented Generation）引擎。RAGFlow 可以为各种规模的企业及个人提供一套精简的 RAG 工作流程，结合大语言模型（LLM）针对用户各类不同的复杂格式数据提供可靠的问答以及有理有据的引用。
- 项目学习：
  - **[【RAG 项目推荐——RagFlow 篇(一)——RagFlow docker 部署】](https://articles.zsxq.com/id_oota38wboidk.html)**
  - **[【RAG 项目推荐——RagFlow 篇(二)——RagFlow 知识库构建】](https://articles.zsxq.com/id_lfsekvfvqrs3.html)**
  - **[【RAG 项目推荐——RagFlow 篇(三)——RagFlow 模型供应商选择】](https://articles.zsxq.com/id_ac0xn785c6mo.html)**
  - **[【RAG 项目推荐——RagFlow 篇(四)——RagFlow 对话】](https://articles.zsxq.com/id_aamqhv6h4lsn.html)**
  - **[【RAG 项目推荐——RagFlow 篇(五)——RAGFlow Api 接入（以 ollama 为例）】](https://articles.zsxq.com/id_ol0qu2w2eraf.html)**
  - **[【RAG 项目推荐——RagFlow 篇(六)——RAGFlow 源码学习】](https://articles.zsxq.com/id_c0br2l9vkwi3.html)**

#### RAG 开源项目推荐 —— QAnything 篇

- 介绍：QAnything（Question and Answer based on Anything）是一个本地知识库问答系统，旨在支持多种文件格式和数据库，允许离线安装和使用。使用QAnything，您可以简单地删除本地存储的任何格式的文件，并获得准确、快速和可靠的答案。QAnything目前支持的知识库文件格式包括：PDF(pdf) , Word(docx) , PPT(pptx) , XLS(xlsx) , Markdown(md) , Email(eml) , TXT(txt) , Image(jpg，jpeg，png) , CSV (csv)、网页链接(html)等。
- 项目学习：
  - **[【RAG 开源项目推荐 —— QAnything 篇】](https://articles.zsxq.com/id_ead246k33bxk.html)**

#### RAG 开源项目推荐 —— ElasticSearch-Langchain 篇

- 介绍：受langchain-ChatGLM项目启发，由于Elasticsearch可实现文本和向量两种方式混合查询，且在业务场景中使用更广泛，因此本项目用Elasticsearch代替Faiss作为知识存储库，利用Langchain+Chatglm2实现基于自有知识库的智能问答。
- 项目学习：
  - **[【【LLMs 入门实战】基于 本地知识库 的高效 🤖ElasticSearch-Langchain-Chatglm2】](https://articles.zsxq.com/id_0259did6jf16.html)**

#### RAG 开源项目推荐 —— Langchain-Chatchat 篇

- 介绍：Langchain-Chatchat（原Langchain-ChatGLM）基于 Langchain 与 ChatGLM 等语言模型的本地知识库问答 | Langchain-Chatchat (formerly langchain-ChatGLM), local knowledge based LLM (like ChatGLM) QA app with langchain
- 项目学习：
  - **[【【LLMs 入门实战】基于 本地知识库 的高效 🤖Langchain-Chatchat】](https://github.com/chatchat-space/Langchain-Chatchat)**











