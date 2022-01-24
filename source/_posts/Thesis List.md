---
title: Graduation Thesis Literature Review  
date: 2022-01-20
tags: 
categories: 总结
copyright: true
---

:pushpin:

**这里是为Heisenberg的毕业论文做准备，相关的文献列表，在此基础上完善形成文献综述:dash:**

- 题目：基于传播差异的虚假消息检测方法设计与实现
- 针对真假消息在传播模式、传播结构和传播速度等方面的差异，提出基于传播差异的虚假消息检测方法，通过实验验证提出虚假消息检测方法的有效性。
<!--more-->

----------

## Motivation ##

1. 虚假新闻严重危害了人们的生活，手动检测需要消耗大量的人力和物力，因此需要进行对新闻进行自动检测。
2. 现有的虚假新闻检测方法往往基于充足的数据，但是现实中大量的数据并没有被标注，标注数据也需要耗费大量时间和精力，因此无监督方式和弱监督的方法也非常常见。
3. 对现有的实验结果做了一定的分析，提出了对未来工作的展望，未来的研究方向。

> 社交网络的新闻往往包括新闻内容，社交上下文信息，以及外部知识。其中新闻内容指的是文章中所包含的文本信息以及图片视频等各种模态的信息。社交上下文信息指的是新闻的发布者，新闻的传播网络，以及其他用户对新闻的评论和转发。外部知识指客观事实知识，通常由知识图谱表示外部知识。

## Classification criteria ##

1. 有监督虚假新闻检测方法(Supervised learning)

> 指在完全标注数据的情况下，对虚假新闻进行检测。按照输入数据的不同，将监督学习分为基于文章内容的有监督方法（其中包括文本信息的检测，图片内容的检测，以及多模态融合检测）；基于社交上下文的有监督方法（其中包括用户可信度和传播差异）；基于外部知识的虚假新闻检测方法以及混合方法。

1. 弱监督虚假新闻检测方法(Weak supervised learning)

> 原有标注不精确，不完全的情况下，对虚假新闻检测。本文按照输入数据的不同，将弱监督学习分为基于文本的弱监督方法，以及基于社交上下文的弱监督方法。

3. 无监督虚假新闻检测方法(Unsupervised learning)

> 无监督虚假新闻检测方法是指在完全没有标注数据的情况下对虚假新闻进行检测。

## Supervised learning ##

### Basic infomation ###

#### Textual ####

1) [`Detecting rumors from microblogs with recurrent neural networks`][Textual 1] **IJCAI 2016**

> 香港中文大学的马晶博士发表，将深度学习技术应用到虚假新闻检测中。该方法将新闻的每个句子输入到循环神经网络RNN，LSTM或者GRU中，利用循环神经网络的隐层向量表示新闻信息，将隐藏层信息输入到分类器中，得到分类结果。

2) [`A Convolutional Approach for Misinformation Identification`][Textual 2] **IJCAI 2017**

> 首次利用卷积神经网络建模新闻文章。该工作将新闻事件的各个post映射到向量空间，之后将各个post向量拼接形成一个矩阵，之后利用卷积神经网络提取文本特征，将得到的嵌入向量输入到分类器中，得到最后的分类结果。

3) [`Detect rumor and stance jointly by neural multi-task learning`][Textual 3] **WWW 2018**

> 年香港中文大学马晶博士的文章第一次将multi-task的思想应用到虚假新闻检测中。该文章将虚假新闻检测任务和立场分类任务组合成一个多任务模型，利用RNN作为backbone，训练两个任务，取得了不错的结果。

4) [`Detect rumors on twitter by promoting information campaigns with generative adversarial learning`][Textual 4] **WWW 2019**

> 年香港中文大学马晶博士的文章,第一次将对抗训练的思想应用到虚假新闻检测中。该文章利用生成器将谣言转化为非谣言，将谣言转化为非谣言，扩展了训练数据。之后将生成器生成的新闻和原始新闻输入到判别器中进行虚假新闻检测。利用对抗学习，对抗训练生成器和判别器，提升模型的鲁棒性和分类准确率。

5) [`Do Sentence Interactions Matter? Leveraging Sentence Level Representations for Fake News Classification`][Textual 5] **EMNLP 2019**

> 将新闻文章建模为一张以句子为节点，以句子间相似度为边的图。将虚假新闻检测问题转化为图分类问题。利用GCN融合图中节点之间的信息，获得节点嵌入向量，将节点向量池化得到图嵌入，输入分类器中进行分类，取得了不错的效果

6) [`Vroc: Variational autoencoder-aided multi-task rumor classifier based on text`][Textual 6] **WWW 2020**

> 使用变分自动编码器VAE自编码文本信息的方式得到新闻文本的嵌入表示，并且将得到的新闻向量进行多任务学习，提升了模型的效果。

#### Visual ####

[`Exploiting Multi-domain Visual Information for Fake News Detection`][visual 1] **ICDM 2019**
计算机视觉方面的研究表明，经过修图软件伪造得到的图片与原始图片在频域的特征会有很大的不同。基于此，中科院曹娟老师团队提出虚假图片判别器MVNN，该工作发表在ICDM19上。MVNN提取图片的空域特征和频域特征，利用频领特征判别图片是否经过修图软件进行伪造，利用空域特征识别图片的语义信息，将得到的空域embedding和频域embedding拼接到一起，输入到分类器重，得到分类结果。该工作不但可以进行虚假图片的检测，还可以作为插件进行多模态虚假新闻的检测

新闻中不仅包含文本信息，还包含图片，视频等视觉信息。
传统的基于统计学的方法利用附加图片的数目，图片流行度以及图片类型检测虚假新闻。然而这些基于统计学的特征无法描述图片的语义特征。
随着深度学习的兴起，大量的工作使用卷积神经网络VGG或者ResNet对图片进行特征抽取，利用抽取到的特征进行虚假新闻检测。但现有的图片造假技术可以更改图像的语义信息，传统基于CNN的模型只可以提取图片像素级信息，无法识别图片是否经过伪造。

#### Multi ####

1) [`Spotfake: A multi-modal framework for fake news detection`][multi 1] **BIG MM 2019**

> 利用VGG19提取视觉信息，利用BERT提取文本信息，将视觉信息和文本信息拼接，输入到分类器中，对虚假新闻进行分类。

2) [`Spotfake+: A multimodal framework for fake news detection via transfer learning`][multi 2] **AAAI 2020**

> 利用VGG提取视觉特征，利用XLNET提取文本特征，将两者进行拼接输入到分类器中，对虚假新闻进行分类。

3) [`MSRD: Multi-Modal Web Rumor Detection Method`][multi 3] **计算机研究与发展 2020**

> 计算机研究与发展的文章 考虑了新闻图片中包含的文本信息。使用LSTM建模文本信息以及图片中的文本信息，使用VGG建模视觉信息，最后将视觉信息，图片中的文本信息，新闻本身的文本信息拼接，得到最终的新闻表示，送入到分类器中，得到最终的分类结果。

4) [`Eann: Event adversarial neural networks for multi-modal fake news detection`][multi 4] **KDD 2018**

> 利用VGG提取视觉特征，利用Text-CNN提取视觉特征，将视觉信息和文本信息拼接得到新闻的表示。为了让模型更好的利用多模态信息，EANN设计了一个辅助任务，事件鉴别。事件鉴别器将拼接的多模态新闻信息作为输入，输出事件的类别。通过辅助任务更好的理解多模态信息，从而帮助虚假新闻检测。

5) [`Mvae: Multimodal variational autoencoder for fake news detection`][multi 5] **WWW 2018**

> 利用VGG提取图像特征，利用双向的LSTM提取文本特征，将视觉特征和文本特征拼接得到新闻的表示。为了让模型更好的利用多模态信息，MVAE设计了一个辅助任务，新闻重构任务。通过encoder编码新闻的视觉信息和文本信息，通过decoder将视觉信息和文本信息进行重构，通过重构任务，更好的融合新闻的多模态信息。最后，将编码器得到的新闻embedding输入到分类器中，得到新闻的分类。

6) [`SAFE: Similarity-Aware Multi-modal Fake News Detection`][multi 6] **KDD 2020**

> 是检测新闻图文相符性的代表工作。该工作利用image2text模型将视觉信息转化为文本信息，并通过全>连接层将文本信息和视觉信息映射到同一向量空间中，之后对比视觉信息和文本信息之间的相似度。如果相似度较高，则图文相符，为真实新闻；如果相似度较低，则图文不符，为虚假新闻。

7) [`Detecting fake news by exploring the consistency of multimodal data`][multi 7] **IPM 2021**

> 利用BERT建模文本信息，利用ResNet建模视觉信息，计算两者之间的相似度，判别图文是否相符。

8) [`Multimodal fusion with recurrent neural networks for rumor detection on microblogs`][multi 8] **ACM MM 2017**

> 首次提出利用模态之间的注意力对模态之间的信息进行增强。该工作使用LSTM提取文本信息，使用VGG提取视觉信息，之后利用模态之间的注意力机制增强模态之间的信息理解，更好的对多模态信息进行理解，将融合的多模态信息输入到分类器中进行分类，取得不错的效果

9) [`Multimodal Fusion with Co-Attention Networks for Fake News Detection`][multi 9] **ACL 2021**

> 借鉴了人们阅读新闻时的习惯”人们通常是阅读一下文本，再看看图片，再阅读文本，再看看图片“，设计了双层的图片文本信息co-attention，从而更好的融合图片信息和文本信息。该工作认为图像的频域和空域信息对虚假新闻检测都是有必要的，因此作者使用VGG建模图片的空域信息，利用CNN建模图片的频域信息，使用co-attention将频域信息和空域信息进行融合，得到更好的图片表示

10) [`Improving Fake News Detection by Using an Entity-enhanced Framework to Fuse Diverse Multimodal Clues`][multi 10] **ACM MM 2021**

> 中科院曹娟老师团队的文章综合关注了多模态之间的互补信息，多模态的信息增强，以及多模态信息之间的对比。

### Social context feature ###

现有的虚假新闻作者往往仿照真实新闻的写法编写虚假新闻，因此，仅仅根据虚假新闻的内容去判别虚假新闻是不够的。大量研究表明，新闻作者的可信度可以帮助我们进行虚假新闻检测，可信度高的用户发表的新闻文章更有可能是真实新闻，可信度低的用户发表的新闻更有可能是虚假新闻。社会学研究表明，真实新闻和虚假新闻在社交网络的传播情况往往有所不同，因此，可以利用新闻的传播信息对虚假新闻进行检测。

#### User credibility ####

基于用户可信度的虚假新闻检测方法利用用户的profile以及用户的历史发文评估用户的可信度，之后对虚假新闻进行检测

1) [`GCAN: Graph-aware Co-Attention Networks for Explainable Fake News Detection on Social Media`][user 1] **ACL 2020** #TAG 1

> GCAN构建了一张用户图，利用用户的profile作为图中节点的初始化信息，利用GCN得到用户的embedding，利用该用户信息进行虚假新闻检测。

2) [`User-Characteristic enhanced model for fake news detection in social meida`][user 2] **NLPCC 2019**

> 将新闻的传播网络和用户的社交网络建模为一张异质图，通过异质图神经网络建模图中节点信息，并将新闻信息和用户信息拼接到一起进行虚假新闻检测。

3) [`User Preference-aware Fake News Detection`][user 3] **SIGIR 2021**

> 利用用户的发文历史识别用户可信度，将其作为内因。同时该工作将新闻的传播情况作为外因，利用内因和外因共同进行虚假新闻检测

#### Propagation ####

1) [`Early detection of fake news on social media through propagation path classification with recurrent and convolutional networks`][prop 1] **AAAI 2018**

> 将谣言的传播与评论信息视做一个时间序列，利用RNN和CNN建模该序列，将两个隐向量拼接在一起，输入到分类层，得到分类结果。

2) [`Interpretable rumor detection in microblogs by attending to user interactions`][prop 2] **AAAI 2020**

> 利用transformer建模时间序列，将源新闻以及其他转发句子作为transformer的输入，用time delay embedding替换掉原始transformer中的position embedding。将transformer输出的新闻嵌入向量输入到分类器中，得到分类结果。由于transformer优秀的表征能力，Plan取得了很好的效果。

3) [`Rumor detection on twitter with tree-structured recursive neural networks`][prop 3] **ACL 2018**

> 香港中文大学马晶博士将谣言的传播过程建模为树形结构，该工作构建了一个bottom-up传播树，又构建了一个top-down传播树，并使用递归神经网络对树中的节点进行建模，对虚假新闻进行分类。

4) [`Debunking rumors on Twitter with tree transformer`][prop 4] **ACI 2020**

> 香港中文大学马晶博士利用tree transformer对谣言传播树进行建模，由于transformer优秀的表征能力，tree transformer取得了很好的效果。

5) [`Rumor detection on social media with bi-directional graph convolutional networks`][prop 5] **AAAI 2020** # TAG 2

> 首次将新闻的传播过程建模为图结构，Bi-GCN利用top-down的图表示新闻的传播信息，利用bottom-up的图表示新闻的弥散信息，利用GCN去融合图中的节点信息，获得节点表征，Bi-GCN还提出了一种根节点强化机制，认为源新闻会对其他信息产生语义增强效果，因此在其余节点中融合了源新闻的信息。最后对top-down的图和bottom-up得到的节点信息进行池化得到两个图的图信息，再将两个图信息进行拼接，输入到分类器中，得到最终的分类结果。

6) [`Temporally evolving graph neural network for fake news detection`][prop 6] **IPM**

> 北京邮电大学吴斌老师团队将新闻的传播图建模为一张动态图，考虑了新闻传播过程的动态变化，利用动态图神经网络得到动态图嵌入向量，将其输入到分类器中得到分类结果，取得了不错的效果。

### External knowledge ###

#### Attention ####

1) [`Multi-modal knowledge-aware event memory network for social media rumor detection`][attention 1] **MM 2019**

> 利用注意力机制，将视觉信息和外部知识信息融合到文本表示中，帮助模型更好的理解新闻文本内容，对虚假新闻进行分类，取得了很好的效果。

2) [`KAN: Knowledge-aware Attention Network for Fake News Detection`][attention 2] **AAAI 2021**

> 利用命名实体识别的方法将文本中的实体与知识图谱中的实体进行对齐，寻找到知识图谱中对应的实体。为了具有更加丰富的语义信息，KAN模型利用了知识图谱中对应的实体上下文信息。利用设计的multi head attention的方式融合新闻文本信息，实体信息以及实体上下文信息，获得语义丰富的新闻文本建模，对虚假新闻分类取得了很好的效果。

#### Graph neural networks ####

1) [`Fake news detection via knowledge-driven multimodal graph convolutional networks`][gra-neural 1] **ICMR 2020**

> 构建了一张包含文本信息，图像信息，以及知识图谱中实体信息的异质量信息网络，使用GCN对各个模态信息进行信息融合，得到融合文本信息，外界知识与视觉信息的新闻表示的新闻表示，取得很好的判别效果。

2) [`meet the truth: Leverage Objective Facts and Subjective Views for Interpretable Rumor Detection`][gra-neural 2] **ACL 2021**

> 利用预训练的事实核查模型在外部知识语料重查找事实证据，将事实证据和新闻内容构造为星形图，利用GCN融合新闻内容和事实证据，对虚假新闻进行检测。

#### Compare ####

- [`Compare to The Knowledge: Graph Neural Fake News Detection with External Knowledge`][compare 1] **ACL 2021**

> 北京邮电大学胡琳梅团队考虑将新闻内容中的抽取实体信息与知识图谱中的实体信息进行对比，从而识别出虚假新闻。

## Weak supervised learning ##

### 基于新闻内容 ###

#### 图结构 ####

1) 首先建模新闻的文本信息作为新闻节点的初始化信息，之后利用新闻之间的相似性构图，将相似性较高的前n个新闻互相连边，之后根据图神经网络方法进行信息传递，获得新闻的嵌入表示，最后将嵌入表示输入到分类器中，得到新闻的分类结果。Weak supervised、Weak content supervised、Graph based methods

#### 伪标签 ####

1) Weak supervised、Weak content supervised、Semi supervised methods

### 基于社交网络信息 ###

i. 社交网络包含丰富的信息，可以根据社交网络中的信息为新闻打上标签，比如，可信度较低的用户相较于可信度较高的用户更有可能发表虚假新闻；引起用户较强情感波动的新闻更有可能是虚假新闻。
ii. ECML/PKDD的文章“. Early detection of fake news with multi-source weak social supervision“基于以下三个假设为新闻打上伪标签：1，可信度较低的用户更有可能发表虚假信息；2，新闻内容如果包含更大的政治偏见，则更有可能是虚假信息；3，新闻如果引起用户较大的情感极性变化，则更有可能是虚假新闻。该工作利用以上三个假设为无标注新闻标上伪标签，训练网络进行分类，取得了较好的效果。
iii. ECML/PKDD的文章“. Early detection of fake news with multi-source weak social supervision“基于以下三个假设为新闻打上伪标签：1，可信度较低的用户更有可能发表虚假信息；2，新闻内容如果包含更大的政治偏见，则更有可能是虚假信息；3，新闻如果引起用户较大的情感极性变化，则更有可能是虚假新闻。该工作利用以上三个假设为无标注新闻标上伪标签，训练网络进行分类，取得了较好的效果。
ICDM20的文章“Adversarial active learning based heterogeneous graph neural network for fake news detection“将新闻作者，主题以及新闻文本信息构建为一张异质图，并且根据异质图表示学习方法融合各个异质节点的信息，进而为无标注数据提供伪标签，作者又利用主动学习思想对伪标签进行挑选，挑选出可信度较高的，作为最终的分类。

## Unsupervised learning ##

### 转化为异常检测 ###

i. TEMSCON17的工作“Detecting rumors on online social networks using multi-layer autoencoder“将虚假新闻检测问题转化为异常检测问题。该工作基于一个假设”用户发表的虚假新闻是用户发文历史中的异常行为“。该工作选取待检新闻作者的历史发文信息，利用自编码器对用户发文历史做编码，映射到向量空间，选出向量空间的离群点，将其视为异常，识别虚假新闻。
ii. TEMSCON17的工作“Detecting rumors on online social networks using multi-layer autoencoder“将虚假新闻检测问题转化为异常检测问题。该工作基于一个假设”用户发表的虚假新闻是用户发文历史中的异常行为“。该工作选取待检新闻作者的历史发文信息，利用自编码器对用户发文历史做编码，映射到向量空间，选出向量空间的离群点，将其视为异常，识别虚假新闻。

### 利用图结构 ###

i. ACM Conference on Hypertext and Social Media2020的工作“Unsupervised fake news detection: A graph-based approach”利用新闻之间的相似性构建新闻图。之后作者利用新闻发布者可信度等方式为一些新闻打上伪标签，将其作为种子节点，之后利用标签平滑特性为其他虚假新闻进行分类，从而取得不错的效果。

### 迁移学习的方式 ###

i. Arxiv21的工作“Cross-lingual COVID-19 Fake News Detection”提出了一个中文新冠疫情虚假新闻检测数据集，并且在多语言的场景下对虚假新闻进行分类。英文数据集具有大量标注好的数据，中文数据集规模较小，同时大量数据暂无标注，将英文数据集下预训练好的虚假新闻检测模型迁移到中文虚假新闻检测任务中，进行微调。
ii. ECML/PLDD21年的工作“Rumour detection via zero-shot Cross-lingual Transfer Learning”同样在标注充足的英文语料上训练模型，将其迁移到无标注的中文语料库中。该工作使用标注充足的英文数据集微调一个teacher BERT模型，之后让teacher BERT在无标注的中文语料对中文语料标注伪标签，利用伪标签的数据训练student BERT，通过不断的往复，student BERT取得很好的分类效果。

## Datasets ##

## Future work ##

<!-- markdownlint-disable-file MD029 -->

[Textual 1]: https://www.ijcai.org/Proceedings/16/Papers/537.pdf
[Textual 2]: https://www.ijcai.org/Proceedings/2017/0545.pdf
[Textual 3]: https://dl.acm.org/doi/pdf/10.1145/3184558.3188729
[Textual 4]: https://ink.library.smu.edu.sg/cgi/viewcontent.cgi?article=5562&context=sis_research
[Textual 5]: https://aclanthology.org/D19-5316.pdf
[Textual 6]: https://arxiv.org/pdf/2102.00816.pdf

[visual 1]: https://arxiv.org/pdf/1908.04472.pdf

[multi 1]: https://precog.iiitd.edu.in/pubs/SpotFake-IEEE_BigMM.pdf
[multi 2]: https://www.researchgate.net/publication/341322867_SpotFake_A_Multimodal_Framework_for_Fake_News_Detection_via_Transfer_Learning_Student_Abstract
[multi 3]: https://crad.ict.ac.cn/EN/Y2020/V57/I11/2328
[multi 4]: https://dl.acm.org/doi/pdf/10.1145/3219819.3219903
[multi 5]: https://dl.acm.org/doi/10.1145/3308558.3313552
[multi 6]: https://arxiv.org/pdf/2003.04981.pdf
[multi 7]: https://www.sciencedirect.com/science/article/pii/S0306457321001060
[multi 8]: https://dl.acm.org/doi/abs/10.1145/3123266.3123454?casa_token=YezZ9B--TsAAAAAA:-tXSt-GU3owhdAj1Oaa5PEqwCdO0kfIXfL_VDQJavItIcAswt_rAyxBDucIJxEAXxj5pdbMTT5lYH7Q
[multi 9]: https://aclanthology.org/2021.findings-acl.226.pdf
[multi 10]: https://dl.acm.org/doi/abs/10.1145/3474085.3481548

[user 1]: https://aclanthology.org/2020.acl-main.48.pdf
[user 2]: http://tcci.ccf.org.cn/conference/2019/papers/182.pdf
[user 3]: https://dl.acm.org/doi/10.1145/3404835.3462990

[prop 1]: https://www.aaai.org/ocs/index.php/AAAI/AAAI18/paper/view/16826
[prop 2]: https://ojs.aaai.org/index.php/AAAI/article/view/6405
[prop 3]: https://aclanthology.org/P18-1184/
[prop 4]: https://aclanthology.org/2020.coling-main.476/
[prop 5]: https://ojs.aaai.org/index.php/AAAI/article/view/5393
[prop 6]: https://www.sciencedirect.com/science/article/pii/S0306457321001965

[attention 1]: https://dl.acm.org/doi/10.1145/3343031.3350850
[attention 2]: https://ojs.aaai.org/index.php/AAAI/article/view/16080

[gra-neural 1]: https://dl.acm.org/doi/10.1145/3372278.3390713
[gra-neural 2]: https://aclanthology.org/2021.findings-acl.63

[compare 1]: https://aclanthology.org/2021.acl-long.62