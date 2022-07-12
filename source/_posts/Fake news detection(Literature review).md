---
title: Fake news detection(Literature review)
date: 2022-07-06
tags:
   - Fake news
categories: Literature review
---

**摘要**：随着移动互联网的快速发展，人们的社交方式发生持续变革，微博和Twitter等逐渐成为人们获取信息和发表意见的便捷在线平台。然而，便捷在线平台同样导致了虚假消息的大量产生和传播，将会给舆论和社会稳定带来不利影响。本文调研了虚假消息检测领域的研究现状，主要介绍了基于有大量数据标注的监督学习方法，强调了基于传播模式的研究前景。之后简单介绍了一些评估方法和常用的虚假消息检测领域的数据集。最后进行总结，为基于传播差异的虚假消息检测方法设计与实现做好基础。

**关键词**：信息传播、深度学习、虚假消息、传播差异

<!--more-->

<style>
.center {
  width: auto;
  display: table;
  margin-left: auto;
  margin-right: auto;
}
</style>

## 1  引言 ##

近年来，得益于移动互联网的发展，微博和Twitter等主流社交媒体已经成为人们获取信息和发布信息的重要平台^[1]^，信息的自由获取和传播达到了空前的便利，给虚假消息创造了有利的生存环境。虚假消息有几个相近的定义，如谣言、虚假新闻等，都是通过对图像、文本、音频视频进行处理以歪曲或捏造事实，企图误导或引诱大众从而达到经济或政治目的。
虚假消息主要在互联网发布和广泛传播。用户对消息和文章的评论、点赞与转发构成了最基本的在线社交活动^[2]^，在线社交网络中的用户通过评论、点赞等基本操作将消息传播出去，人的情绪、观点和行为容易受到他人的影响，虚假消息利用这一特点吸引注意力，通过用户间的社交活动进行有效传播^[3]^。传统纸媒发版的内容需要经编辑严格审核，可信度得到大众的认可，然而用户在线上平台发布的文本、图片和视频很少经过内容真实性验证，经评论与转发使得虚假消息得到进一步传播。传统的解决方案是通过人工识别的方式对发布在社交网络的消息进行甄别，但存在效率低下和具有时滞性等问题^[4]^，难以在第一时间对海量数据进行甄别，无法在传播初期阻断虚假消息的散布。随着在线社交的普及，这种现象变得更为普遍，依靠传统的人工识别已不再适用，因此迫切需要一种高效的方法自动识别、检测出虚假消息进而阻断影响范围的持续扩大。
已有的检测方法主要通过提取消息内容、用户可信度和传播模式等关键特征进行训练得到真假消息二分类器以实现虚假消息的自动检测，也有研究采用多特征融合的方式进行检测以提高性能。其中常用的分类器包括：支持向量机（Support Vector Machine，SVM）^[5]^、随机森林（Random Forest，RF）^[6]^和决策树（Decision Tree，DT）^[7]^等。然而这些依赖于特征工程的方法非常耗时费力，并且需要制作特定的数据，缺乏高层的特征表征，因此往往针对性较强，缺乏泛化性能。另外也有一系列研究基于深度学习技术对真实消息与虚假消息在传播模式上的结构特征进行挖掘，利用图卷积网络提取消息传播过程中的拓扑结构特征，对局部的结构信息进行融合以得到更丰富的特征表示。
本文主要开展虚假消息检测基于有完全数据标注的监督学习领域研究，针对基于消息内容和消息传播结构等检测方法进行综述，从研究现状、应用、优缺点等方面进行分析，为进一步研究基于传播差异的虚假消息检测方法设计与实现打下基础。

## 2  虚假消息检测研究现状 ##

虚假消息具有传播快、影响广、危害大的特点，微博和Twitter上每天以数亿计增长的消息中有大量是虚假的，而虚假消息的泛滥对群众的影响非常大，造成的舆论影响会影响社会治安并使网络暴力更为频繁^[8,9]^。国内外均展开了一系列针对虚假消息自动检测的研究，以下主要从基于消息内容的方法、基于社交背景的方法和基于传播结构的方法进行综述。其中，消息内容的方法可以分为文本和图像单模态及多模态融合的方法，社交背景的方法可以分为融合用户背景的方法和融合外部知识的方法，传播结构的方法可以分为基于图结构和基于树结构以及基于时间序列的方法。

### 2.1  基于消息内容的方法 ###

首先，由于文本是消息内容最基本的组成部分，早期的虚假消息检测研究采用最直接的方法，从消息内容本身入手，对消息的文本内容特征进行提取和建模以鉴定消息内容的真假。Ma等人^[10]^通过建立基于循环神经网络（Rurrent Neural Network，RNN）及其变体即长短期记忆（Long Short Term Memory，LSTM）和门控循环单元（Gated Recurrent Unit，GRU）共三种不同结构模型如图1所示，利用深度学习的方式提取消息的文本特征并使用多个隐藏层，相比于依赖特征工程的方法有了更强的泛化能力。然而经过训练的RNN模型在两个连续的文本输入间产生的循环转移矩阵是固定不变的，在动态和复杂的场景下并不适用，Feng等人^[11]^利用卷积神经网络（Convolutional Neural Network, CNN）的卷积结构和池化操作灵活提取分散在文本序列中的关键特征。
 
![图1 采用三种不同结构RNN模型的检测框架][29]
<center><font size=2>图1 采用三种不同结构RNN模型的检测框架</font></center>

其次，消息内容除了文本，还包括图片等视觉信息。正如我们平时所见到的，大部分虚假消息会附有经过伪造处理的图片。CNN通常只能提取图片的像素级特征，然而现有的造假技术已经可以更改图像的语义信息，从而无法进行图像真假的判断。经过修图处理的图像经过频域分析后可以发现与原始图像有显著差别^[12]^，具有误导性的图像通常比真实的图像具有更重的压缩伪影。

![图2 多模态虚假消息检测方法总体框架][30]
<center><font size=2>图2 多模态虚假消息检测方法总体框架</font></center>

另一方面，依靠单一的文本或图像进行检测并不完善，因为大量的消息以文和、图片混合的形式存在，将文本特征和视觉特征混合建模的方法能够进一步提高分类器的性能。基于将文本信息和视觉信息拼接的方法，Singhal等人^[13,14]^提出分别利用预训练的模型对文本信息和视觉信息进行特征提取，然后拼接融合得到消息的向量表示。显式提取消息图片中的视觉实体和嵌入文字^[15]^，可以得到不同语义层次的视觉特征，能够有效增强多模态融合的虚假消息检测性能。刘金硕等人^[16]^设计了如图2所示的多模态虚假消息检测方法，在简单融合文本和图像特征的基础上还提取了图像内嵌的文本特征，进行串接后输入全连接层进行多模态的检测。图像和文本的匹配程度能够反映消息是否经过篡改，可以将文本信息和视觉信息同通过全连接层映射到同一向量空间^[17]^，使用余弦相似度算法度量文本和图像之间的相似性。

### 2.2  基于社交背景的方法 ###
虚假消息的作者仿照真实消息进行编写，基于消息内容的检测方法会受到制约，分析用户的可信度、发文历史、偏好等用户资料信息可以作为虚假消息检测的辅助评价指标，可信度高的用户更有可能发表真实的消息^[18]^。此外，丰富的外部知识和客观事实可以帮助模型更准确地识别虚假消息。

#### 2.2.1  融合用户背景 ####
社会学研究表明可信度高的用户发出的消息更可能是真实的^[19]^，基于用户可信度的研究对用户的基本资料、历史发文等信息进行建模作为辅助评价标准。发布虚假消息的可疑用户具有未实名认证，创建时间短，用户介绍缺失等显著特征^[20]^。当一条假消息证实了用户现有的观念和偏好时，用户更有可能传播这条假消息^[21]^，在基于消息内容的基础上，添加用户的偏好表示，同时根据消息的传播情况共同进行虚假消息检测能够有效提高虚假消息检测性能。

#### 2.2.2  融合外部知识 ####
客观事实作为虚假消息检测的重要线索在基于消息内容的检测方法中经常被被忽略，而大量外部知识包含丰富的语义信息，将注意力和图神经网络运用到虚假消息检测领域以融合外部知识可以有效提高模型学习消息内容的能力。人脑中已存在的知识含有大量语义信息，能帮助人们在查看消息时更好的理解消息内容，融合外部知识输入模型同样也能更好的理解消息内容从而提高虚假消息检测模型的性能。Li^[22]^等人将虚假消息检测任务分成取证和检测两部分，利用维基百科的固有知识预训练模型作为事实证据，融合消息内容和事实依据进行取证，找出对应的事实依据，然后利用事实依据对消息内容的真实性进行判断。

### 2.3  基于消息传播结构的方法 ###
消息传播主要通过用户的转发来实现，用户对收到的消息进行层级转发构成了消息传播的复杂结构，根据消息在层级转发过程中的相互关系来判断消息的真实性。基于消息内容的检测方法忽略了消息在层级传播过程中的树形结构这一重要特征。然而树结构无法表示相同层级转发的消息之间的联系，图结构使用顶点和边的表示更好的建模了消息在广度和深度上的传播，图结构比树结构拥有更复杂的拓扑结构。下面介绍分别从基于树结构和图结构的虚假消息检测方法。

#### 2.3.1  树结构 ####

![图3 自底向上和自顶向下两种树形结构模型][31]
<center><font size=2>图3 自底向上和自顶向下两种树形结构模型</font></center>

基于递归神经网络（Recursive Neural networks，RvNN），Ma等人^[23]^使用如图2所示的自顶向下和自底向上两种变体对消息的树形传播模式进行建模，将源消息作为根节点，对父节点消息的包含支持或反对倾向的转发与评论作为子节点。对真实消息的反对评论会引起其他人的反对，而对虚假消息的反对评论则会引起其他人的支持。从这两种不同变体的不同角度表示消息传播的树形结构，自顶向下的模型在池化层中可以有效地选择嵌入叶子节点的重要特征，信息损失更小，从而拥有更好的性能。
将一棵树的所有节点消息全连接过度简化了用户之间的交互从而带来一定的误差，部分节点之间并没有直接的上下文关联，应当根据语境来增强表征能力^[24]^。在虚假消息的传播结构中，回复消息能够进一步增强被回复消息的立场，通过对比相关消息可以检测出一些不准确信息。选择性地关注相应消息增强指示性特征的表征学习，从而达到深入挖掘用户的观点的效果。

#### 2.3.2  图结构 ####
消息的传播通常是从某一用户发文作为起点，在点对点的传播中较为简单，但在现实情况下存在消息经过层层转发，部分还会修改语义信息，树的结构无法完全表示彼此之间的联系，图结构能更好的反映消息在传播过程中彼此的联系。
传播和散布是虚假消息的两个关键特征，原有的例如RvNN只考虑了消息传播速度而忽视了消息的散布结构，具有一定的局限性^[25]^。双向图卷积神经网络（Bi-Directional Graph Convolutional Networks，Bi-GCN）能够从自顶向下和自底向上两个方向发掘虚假消息的传播和散布特征，即从虚假消息传播树中的父节点到子节点学习消息的传播模式，从子节点反向到父节点学习消息的弥散模式。基于图卷积神经网络（Graph Convolutional Networks，GCN）提取子节点特征，子节点与父节点之间的融合与对比也能够增强语义信息^[21]^。将原始消息融合进传播过程中的其他消息，对自顶向下图和自底向上图进行池化得到两个图的信息。
微博、推文和新闻文章在社交上下文中存在多种节点类型和关系类型，例如用户发发布微博，用户之间相互点赞关注等，使用同质图神经网络忽视了一部分关键的用户关系，不同文章领域、转发关系等也形成异质的信息网络。Shu等人^[26]^将发布、传播和关注等社交关系构建成异质信息网络，使用矩阵分解的方式获得各个消息节点的嵌入表示，进行虚假消息检测。

## 3  常用数据集 ##
虚假消息检测领域关注消息本身，用户画像、关注列表等社交上下文内容以及消息在传播过程中的结构即关联信息，采用收集自不同领域不同平台具有显著差异性的数据集能够更好地构建模型。常用的数据集信息如表1所示，搜集整理自Twitter和微博平台^[9,27]^。除此之外，FakeNewsNet提供了Buzzfeed和PolitFact的两个数据集^[28]^，包含了消息的来源以及标题等内容属性和关注列表、粉丝列表等社会背景属性。

<div class="center">
<center><font size=2>表1  常用数据集信息统计</font></center>

| 数据统计 | Weibo | Twitter15 | Twitter16 |
|-------|-------|-------|-------|
|原始推文数量|4664|1,490|818|
|用户数量|2,746,818|276,663|173,487|
|转发数量|3,805,656|331,612|204,820|
|真实消息数量|2351|372|207|
|虚假消息数量|2313|370|205|
|单篇推文平均传播时长|2,460小时|1,337小时|848小时|
|单篇推文平均转发次数|816|223|251|
|单篇推文最多转发次数|59,318|1,768|2,765|
|单篇推文最少转发次数|10|55|81|
</div>

## 4  评价指标 ##
虚假消息检测作为二分类任务，虚假消息检测通常使用多个指标同时进行评价，常用的评价指标有准确率（Accuracy）、精准率（Precision）、召回率（Recall）和F1-score等。Accuracy较为直观，被广泛应用在各种分类任务中，对应到在虚假消息检测领域即正确识别出虚假消息或真实消息的总比例。但由于Accuracy的本身缺陷，例如正负类数量差距过大将导致准确度的判断不能正确反映事实。Precision反映了是否有大量假消息被判断成真消息，体现了模型对负样本的区分能力；Recall反映了是否有真消息被误判为假消息，体现模型对正样本的识别能力。Accuracy表示预测为正的结果占总样本的比例。

## 5  总结 ##
本文主要对基于大量有标注数据的虚假消息检测进行了回顾，对现有的工作进行客观分析，从消息内容和传播差异等角度建模进行虚假消息检测具有各自的优点和局限性。用户在社交网络上发布消息具有数据量大、结构关系复杂等特点，如何从传播速度、传播结构等方面多层次剖析消息传播特征，进一步促进虚假消息的自动化检测识别发展是今后的一个研究热点。

## :bell: 参考文献 ##
[1]	谢柏林, 蒋盛益, 周咏梅, 等. 基于把关人行为的微博虚假信息及早检测方法[J]. 计算机学报, 2016, 39(04): 730-744.
[2]	胡长军, 许文文, 胡颖, 等. 在线社交网络信息传播研究综述[J]. 电子与信息学报, 2017, 39(04): 794-804.
[3]	杨善林, 王佳佳, 代宝, 等. 在线社交网络用户行为研究现状与展望[J]. 中国科学院院刊, 2015, 30(02): 200-215.
[4]	何韩森, 孙国梓. 基于特征聚合的假新闻内容检测模型[J]. 计算机应用, 2020, 40(08): 2189-2193.
[5]	Cortes C, Vapnik V. Support-vector networks [J]. Machine learning, 1995, 20(3): 273-297.
[6]	李欣海. 随机森林模型在分类与回归分析种的应用[J]. 应用昆虫学报, 2013, 50(04): 1190-1197.
[7]	杨学斌, 张俊. 决策树算法及其核心技术[J]. 计算机技术与发展, 2007, 17(01): 43-46．
[8]	白月明. 新媒体时代虚假新闻的成因及规避探析[J]. 新闻研究导刊, 2021, 12(16): 157-159.
[9]	段大高, 盖新新, 韩忠明, 等. 基于梯度提升决策树的微博虚假消息检测[J]. 计算机应用, 2018, 38(02): 410-414+420.
[10]	Ma J, Gao W, Mitra P, et al. Detecting rumors from microblogs with recurrent neural networks [A]. Proceedings of the Twenty-Fifth International Joint Conference on Artificial Intelligence [C]. Palo Alto: AAAI Press, 2016, 3818-3824.
[11]	Yu F, Liu Q, Wu S, et al. A convolutional approach for misinformation identification [A]. Proceedings of the Twenty-Sixth International Joint Conference on Artificial Intelligence [C]. Palo Alto: AAAI Press, 2017, 3901-3907.
[12]	Qi P, Cao J, Yang T, et al. Exploiting multi-domain visual information for fake news detection [A]. 2019 IEEE International Conference on Data Mining(ICDM 2019) [C]. Piscataway: IEEE, 2019, 518-527.
[13]	Singhal S, Shah R R, Chakraborty T, et al. SpotFake: A multi-modal framework for fake news detection shivangi [A]. 2019 IEEE Fifth International Conference on Multimedia Big Data (BigMM) [C]. Piscataway: IEEE, 2019, 39-47.
[14]	Singhal S, Kabra A, Sharma M, et al. SpotFake+: A multimodal framework for fake news detection via transfer learning [A]. Proceedings of the AAAI Conference on Artificial Intelligence [C]. Palo Alto: AAAI Press, 2020, 13915-13916.
[15]	亓鹏, 曹娟, 盛强. 语义增强的多模态虚假新闻检测[J]. 计算机研究与发展, 2021, 58(07): 1456-1465.
[16]	刘金硕, 冯阔, Pan J, 等. MSRD: 多模态网络谣言检测方法[J]. 计算机研究与发展, 2020, 57(11): 2328-2336.
[17]	Xue J, Wang Y, Tian Y, et al. Detecting fake news by exploring the consistency of multimodal data [J]. Information Processing and Management, 2021, 58(5): 102610.
[18]	Dou Y, Shu K, Xia C, et al. User preference-aware fake news detection [A]. Proceedings of the 44th International ACM SIGIR Conference on Research and Development in Information Retrieval [C]. New York: Association for Computing Machinery, 2021, 2051-2055.
[19]	刘波, 李洋, 孟青, 等. 社交媒体内容可信性分析与评价[J]. 计算机研究与发展, 2019, 56(09): 1939-1952.
[20]	Lu Y-J, Li C-T. GCAN: Graph-aware co-attention networks for explainable fake news detection on social media [A]. Proceedings of the 58th Annual Meeting of the Association for Computational Linguistics [C]. Stroudsburg: Association for Computational Linguistics, 2020, 505–514.
[21]	He Z, Li C, Zhou F, et al. Rumor detection on social media with event augmentations [A]. Proceedings of the 44th International ACM SIGIR Conference on Research and Development in Information Retrieval [C]. New York: Association for Computing Machinery, 2021, 2020-2024.
[22]	Li J, Ni S, Kao H-Y. Meet the truth: Leverage objective facts and subjective views for interpretable rumor detection [A]. Findings of the Association for Computational Linguistics [C]. Stroudsburg: Association for Computational Linguistics, 2021, 705–715.
[23]	Ma J, Gao W, Wong K-F. Rumor detection on twitter with tree-structured recursive neural networks [A]. Proceedings of the 56th Annual Meeting of the Association for Computational Linguistics [C]. Stroudsburg: Association for Computational Linguistics, 2018, 1980–1989.
[24]	Ma J, Gao W. Debunking Rumors on Twitter with Tree Transformer [A]. Proceedings of the 28th International Conference on Computational Linguistics [C]. Barcelona: International Committee on Computational Linguistics, 2020, 5455–5466.
[25]	Bian T, Xiao X, Xu T, et al. Rumor detection on social media with bi-directional graph convolutional networks [A]. Proceedings of the 34th AAAI Conference on Artificial Intelligence [C]. Palo Alto: AAAI Press, 2020, 549-556.
[26]	Shu K, Wang S, Liu H. Beyond news contents: The role of social context for fake news detection [A]. Proceedings of the Twelfth ACM International Conference on Web Search and Data Mining [C]. Stroudsburg: Association for Computing Machinery, 2019, 312-320.
[27]	Ma J, Gao W, Wong K-F. Detect rumors in microblog posts using propagation structure via kernel learning [A]. Proceedings of the 55th Annual Meeting of the Association for Computational Linguistics [C]. Stroudsburg: Association for Computational Linguistics, 2017, 708-717.
[28]	Shu K, Mahudeswaran D, Wang S, et al. Fakenewsnet: A data repository with news content, social context, and spatiotemporal information for studying fake news on social media [J]. Big data, 2020, 8(3): 171-188.

<!-- markdownlint-disable-file MD025 MD033 -->
[1]: http://cjc.ict.ac.cn/online/onlinepaper/xbl-2016327154823.pdf
[2]: http://edit.jeit.ac.cn/EN/article/downloadArticleFile.do?attachType=PDF&id=18265
[3]: http://www.bulletin.cas.cn/zgkxyyk/ch/reader/view_abstract.aspx?file_no=20150208&flag=1
[4]: http://www.joca.cn/CN/abstract/abstract23957.shtml
[5]: https://link.springer.com/article/10.1007/BF00994018
[6]: http://library.ioz.ac.cn/bitstream/000000/10883/1/%E9%9A%8F%E6%9C%BA%E6%A3%AE%E6%9E%97%E6%A8%A1%E5%9E%8B%E5%9C%A8%E5%88%86%E7%B1%BB%E4%B8%8E%E5%9B%9E%E5%BD%92%E5%88%86%E6%9E%90%E4%B8%AD%E7%9A%84%E5%BA%94%E7%94%A8.pdf
[7]: http://www.cqvip.com/
[9]: http://www.cqvip.com/qk/94832x/201802/674500313.html
[10]: https://ink.library.smu.edu.sg/sis_research/4630/
[11]: https://www.ijcai.org/proceedings/2017/0545.pdf
[12]: https://ieeexplore.ieee.org/abstract/document/8970940/
[13]: https://ieeexplore.ieee.org/abstract/document/8919302/
[14]: https://ojs.aaai.org/index.php/AAAI/article/view/7230
[15]: https://www.cnki.com.cn/Article/CJFDTotal-JFYZ202107011.htm
[16]: https://www.cnki.com.cn/Article/CJFDTotal-JFYZ202011007.htm
[17]: https://www.sciencedirect.com/science/article/pii/S0306457321001060
[18]: https://dl.acm.org/doi/abs/10.1145/3404835.3462990
[19]: https://www.cnki.com.cn/Article/CJFDTotal-JFYZ201909015.htm
[20]: https://aclanthology.org/2020.acl-main.48.pdf
[21]: https://dl.acm.org/doi/abs/10.1145/3404835.3463001
[22]: https://aclanthology.org/2021.findings-acl.63.pdf
[23]: https://aclanthology.org/P18-1184/
[24]: https://aclanthology.org/2020.coling-main.476/
[25]: https://ojs.aaai.org/index.php/AAAI/article/view/5393
[26]: https://dl.acm.org/doi/abs/10.1145/3289600.3290994
[27]: https://aclanthology.org/P17-1066/?ref=https://githubhelp.com
[28]: https://www.liebertpub.com/doi/abs/10.1089/big.2020.0062
[29]: https://www.lingzhicheng.cn/usr/file/picture/fakenews/图片1.jpg
[30]: https://www.lingzhicheng.cn/usr/file/picture/fakenews/图片2.png
[31]: https://www.lingzhicheng.cn/usr/file/picture/fakenews/图片3.png