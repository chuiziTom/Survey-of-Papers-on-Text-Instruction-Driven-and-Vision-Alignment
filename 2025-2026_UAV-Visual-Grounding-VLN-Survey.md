# 2025–2026 年 UAV 视觉定位、文本—视觉对齐与 VLN 论文调研报告

> 调研截止：2026-11-25。范围：2025–2026 年。主题：自然语言指令解析、细粒度图文对齐、空间关系推理、视觉定位、UAV-VLN。  
> 阅读对象：已了解基础 LLM/VLM，但刚开始系统学习 VLA/VLN 的研究者。

## 0. 执行摘要

本报告关注的不是“哪个 VLM 参数更多、榜单更高”，而是一个更具体的问题：

**当自然语言描述包含多个实体、属性、空间关系和长程目标时，模型如何保留文本的组合语义，并把每项语义约束落实到图像中的具体区域、物体及其关系上？**

可以拆成五个环节：

1. 文本解析：从句子中识别目标、参照物、属性、关系、顺序和约束。
2. 文本表示：保留“谁具有哪个属性”“谁相对谁处于什么位置”等组合结构，而不是把整句压成一个语义向量。
3. 视觉证据提取：找到候选区域、物体、属性和空间关系。
4. 跨模态匹配与推理：让文本实体和视觉实体对应，并联合检查多条关系约束。
5. 验证与行动：判断目标是否真的存在、匹配是否可信；在 VLN 中进一步把目标语义转化为导航阶段和动作。

| 层次 | 相关论文 | 关键问题 |
|---|---|---|
| 文本语义结构化 | HTVG、NAVER、DisCoCLIP、FILD-Nav | 如何从句子中提取或编码组合语义？ |
| 细粒度图文对齐 | S²CLNet、AerialVG | 如何将文本语义落实到图像区域？ |
| 多实体关系推理 | SceneProp、NAVER、GeoNav | 如何同时满足多个实体及其空间约束？ |
| 验证与拒识 | VIRO、NAVER、GeoRefer-Bench | 如何避免模型“总要猜一个目标”？ |
| UAV 导航应用 | GeoNav、FILD-Nav、AerialVG、GeoRefer-Bench | 如何在航拍视角、长距离和复杂环境中使用这些能力？ |

**核心判断：** 对 UAV 视觉定位而言，最值得关注的不是单纯增加一个 LLM 文本解析器，而是建立从“文本中的每项语义约束”到“图像中的可验证证据”的明确对应关系。

---

## 1. Hierarchical Textual-Visual Guidance for Referring Remote Sensing Segmentation（HTVG）

**发表信息**

- 作者：Shiyu Zhang, Qing Zhou, Qi Wang, Yuan Yuan, Jun-Yu Gao
- 期刊：Pattern Recognition，2026
- DOI：[10.1016/j.patcog.2026.113579](https://doi.org/10.1016/j.patcog.2026.113579)
- 状态：已发表期刊论文
- 证据边界：本次检索获得论文元数据与摘要记录，但未能取得可供逐页核对的全文。因此，以下对模块的讨论仅采用可确认的题录/摘要信息，不把未核实的实现细节写成确定事实。

### 1.1 研究问题

遥感指代分割要求模型根据自然语言描述，在遥感图像中分割目标区域。例如：

> “Find the small red vehicle near the long white building.”

模型不仅要识别 vehicle，还要理解 small、red、near、long、white 分别描述什么，以及哪个对象是目标、哪个对象是参照物。

遥感图像的困难包括目标尺度跨度大、背景复杂、同类物体相似、文本中经常同时出现类别/属性/关系等。整句语义特征容易被背景或高频类别主导，无法充分保留局部约束。

### 1.2 方法主线

论文提出层次化的文本—视觉引导框架。根据可检索到的论文摘要，核心设计包括：

- **TSPM（Textual Semantic Parsing Module）：** 对指代表达进行层次化语义处理，强调将不同语义信息拆分并组织起来，而不是把整句话作为不可分解的整体。
- **AVMM（Adaptive Visual Modulation Module）：** 将文本语义用于调制视觉表示，使视觉特征更关注与描述相关的内容。

可以把它的思想理解为：自然语言描述 → 提取目标相关的多层语义 → 用语义线索引导不同层级的视觉特征 → 输出目标分割结果。

这里最值得关注的是“层次化”：目标类别、属性、关系和空间线索可能对应不同粒度的视觉证据。粗粒度语义适合缩小搜索范围，细粒度属性和关系则用于区分相似候选。

### 1.3 启发、局限与阅读结论

文本解析的输出应当能够影响视觉特征的选择、增强或匹配，而不只是提供一段解释性文字。应进一步追问：哪些词描述目标类别？哪些词描述目标自身属性？哪些词描述目标与其他物体的关系？哪些语义线索应该作用于高层语义特征，哪些应作用于细节特征？

由于未能取得全文，本报告不对其具体解析器、损失函数、训练流程、消融表和精确性能数字作进一步断言。尤其不应仅凭摘要就声称它已经解决了多跳关系推理、开放词汇 grounding 或 UAV-VLN。

**阅读结论：** 适合作为“层次化文本语义如何引导遥感视觉特征”的参考；若要作为核心方法基线，必须进一步取得全文，核实 TSPM/AVMM 的真实输入输出和训练目标。

---

## 2. NAVER — A Neuro-Symbolic Compositional Automaton for Visual Grounding with Explicit Logic Reasoning

**发表信息**

- 作者：Zhixi Cai 等
- 会议：ICCV 2025
- DOI：[10.1109/ICCV51701.2025.02232](https://doi.org/10.1109/ICCV51701.2025.02232)
- 论文：[ICCV 官方论文页](https://openaccess.thecvf.com/content/ICCV2025/html/Cai_Naver_A_Neuro-Symbolic_Compositional_Automaton_for_Visual_Grounding_with_Explicit_ICCV_2025_paper.html)
- 代码：[ControlNet/NAVER](https://github.com/ControlNet/NAVER)

### 2.1 研究问题

例如：“Find the person wearing a blue shirt to the left of the car.” 至少有三个约束：目标类别 person、属性 wearing a blue shirt、目标相对 car 位于 left of。只找到一个 person 或只识别 blue shirt 都不足以完成定位。

NAVER 的核心想法是：**将自然语言转换成显式逻辑条件，再用视觉模型提供逻辑推理所需的证据。**

### 2.2 文本处理：从自然语言到 ProbLog 规则

NAVER 使用 LLM 将原始查询转化为类似 ProbLog/Prolog 的逻辑程序。示意如下：

    target(ID) :-
        entity(ID, "person", _, _, _, _),
        relation(ID, _, "left of"),
        attribute(ID, "wearing_blue_shirt").
    query(target(ID)).

它表达的是：找一个类别为 person 的实体，该实体具有 wearing_blue_shirt 属性，并且与某个参照实体满足 left of 关系，最终返回符合条件的实体 ID。

论文展示的主要事实谓词包括：

| 谓词 | 表示内容 |
|---|---|
| entity | 实体 ID、类别和边界框 |
| attribute | 实体具有某项属性 |
| relation | 两个实体之间的有向关系 |
| target/query | 组合条件和最终查询 |

论文没有给出完整的 ProbLog 程序形式化语法。因此应将它理解为“LLM 按提示生成受约束的逻辑代码”，而不是一个已经完整定义的自然语言编译器。

### 2.3 视觉侧如何提供逻辑证据？

1. 图像 caption/类别提取：确定需要关注哪些实体类别；
2. Grounding 模型：找到候选实体和边界框；
3. 属性识别器：判断候选实体是否满足颜色、服饰等属性；
4. 关系识别模块：判断实体对之间是否满足 left of、in front of 等关系；
5. 概率逻辑推理：将实体、属性和关系的置信度作为概率事实，推导满足查询的目标候选。

工程实现还将 ProbLog 程序转换为 Scallop，以执行概率逻辑推理。

### 2.4 自动机与验证

NAVER 设计五状态有限状态控制器：

- Perception：视觉感知；
- Logic Generation：生成逻辑程序；
- Logic Reasoning：执行逻辑推理；
- Answering：验证候选；
- Return Target：返回最终目标。

若逻辑程序生成失败、推理没有结果，或最终候选被视觉验证器否定，系统可以返回前面的阶段重试。

这点很重要：它把“模型给出一个结果”改成了“模型提出候选—验证候选—必要时重新感知”。

### 2.5 实验结果

论文报告了多个视觉定位基准的结果。不同实验配置使用的视觉基础模型不同，不能把不同配置的数值混成同一个模型结果。

较强配置下，检测准确率报告为：

| 数据集 | NAVER 报告准确率 |
|---|---:|
| RefCOCO test A | 96.2 |
| RefCOCO+ test A | 92.8 |
| RefCOCOg test | 91.6 |
| Ref-Adv test | 75.4 |

论文还报告 RefCOCOg segmentation test 的运行失败率约为 0.3%，并通过消融实验分析自动机、逻辑推理和 Answerer 的作用。这些数值是论文所报告的特定配置结果，不意味着 NAVER 在所有数据集、所有视觉模型配置下都能达到相同性能。

### 2.6 优点、不足与 UAV 启发

**优点：** 将实体、属性和关系显式化；逻辑程序便于检查和解释；视觉证据与语言规则相对解耦；验证和重试机制可以减少错误候选直接成为最终结果的情况。

**不足：** LLM 生成的逻辑程序可能语法正确但语义错误；视觉检测、属性识别和关系识别仍然可能出错；概率事实的可靠性和校准非常关键；多阶段调用增加推理复杂度与时延；论文没有证明这种机制可以无修改地迁移到 UAV 俯视图和长程导航。

**UAV 启发：** 可将导航指令转成“目标：红色汽车；参照物：白色建筑；约束：汽车位于建筑东侧；附加约束：汽车属于停车区域”。再将视觉检测和地图信息转化为候选事实，用逻辑程序判断候选是否满足全部条件。

**阅读结论：** 十篇中最值得借鉴的“文本到显式逻辑 + 视觉证据 + 验证控制”范式之一。

---

## 3. S²CLNet — Structure-Constrained Semantic Contrastive Learning for Referring Remote Sensing Image Segmentation

**发表信息**

- 作者：Sen Lei, Shuai Li, Xin-Yu Xiao, Heng-Chao Li, Zhen-Wei Shi, Qing Zhu
- 期刊：IEEE Transactions on Geoscience and Remote Sensing（TGRS），2026
- DOI：[10.1109/TGRS.2026.3724005](https://doi.org/10.1109/TGRS.2026.3724005)
- 官方代码：[45degreesl/S2CLNet](https://github.com/45degreesl/S2CLNet)

### 3.1 研究问题

遥感指代分割的传统训练通常以像素级分割损失为核心：给定图像和文本，输出目标 mask，再与标注 mask 计算交叉熵、Dice 等损失。

但只依靠最终 mask 监督，未必能保证文本语义和视觉区域在特征空间中形成可靠对应。

例如，文本要求“the red vehicle near the white building”，模型可能通过数据集偏差或背景相关性得到一个看似合理的 mask，却没有真正学会 red vehicle 与 white building 之间的语义关系。

S²CLNet 的主要切入点是：**在分割监督之外，引入区域级结构约束的语义对比学习，显式强化视觉区域与语言表示之间的对应关系。**

### 3.2 核心方法

论文提出 S²CLNet，摘要强调两个关键设计：

1. Structure-Constrained Semantic Contrastive Learning；
2. Dynamic Modality Balancing Module（DMBM），其中包含双向跨模态注意力和动态模态平衡。

根据论文摘要，结构约束对比学习的目标是让语义对应的视觉区域与语言表示更加接近，让无关背景区域与目标语义表示分离，减少背景占主导时对齐不充分的问题。

### 3.3 官方代码核查：重要实现细节

我检查了官方仓库中的 CLIP 实现。其 contrastive loss 代码可以看到：

- 将文本特征投影到与视觉特征相同的通道维度；
- 使用目标 mask 将视觉特征划分为局部网格区域；
- 对每个局部网格内的目标区域特征做 masked average pooling；
- 对视觉区域特征和文本特征做归一化；
- 计算区域—文本相似度矩阵；
- 使用温度参数进行缩放；
- 利用文本间的语义相似度对困难负样本进行加权。

代码中可见的默认设置包括：

| 参数 | 代码默认值 |
|---|---:|
| temperature | 0.15 |
| grid_division | 5 |
| hard_weight_factor | 2.0 |
| semantic_threshold | 0.6 |

代码训练循环还将对比损失以 0.35 的系数加入分割损失：

\[
L = L_{\mathrm{seg}} + 0.35 L_{\mathrm{contrast}}
\]

其中分割损失在实现中由 Cross Entropy 和 Dice Loss 组成。

**注意：** 上述是对公开仓库当前实现的代码观察，不应与论文正文中所有实验配置的超参数等同。论文的完整训练配置、各数据集结果表和 DMBM 的全部细节仍应以论文正文为准。

### 3.4 数据集与实验

官方仓库 README 确认实现支持 RefSegRS、RRSIS-D、RISBench。代码提供 CLIP 与 Swin-BERT 两种实现路径。

论文摘要报告在上述三个基准上取得优于已有方法的结果。但本次无法从可访问的论文全文中核验全部结果表，因此不在这里补写未经核实的 mIoU 数字。

### 3.5 优点、不足与 UAV 启发

**优点：** 不只监督最终 mask，还直接约束区域—文本特征的对应关系；通过 mask 引导的区域特征池化，减少无关背景的干扰；语义相似的文本负样本可以构成更难的对比样本；有公开代码，适合复现和进一步拆解。

**不足：** 当前对比实现仍然依赖目标 mask 作为区域监督；网格区域划分比较规则，复杂空间关系不一定能被完整表达；局部区域与文本的相似度不等于已经理解多个实体之间的关系；分割任务的监督形式不能直接等同于开放词汇 UAV 目标检测或 VLN。

**UAV 启发：** 同一张航拍图有五辆外观相似的汽车时，可构造正样本（红色汽车且位于白色建筑东侧）、困难负样本（另一辆红色汽车但位于建筑西侧）和普通负样本（蓝色卡车）。这样的对比训练更能检验模型是否真的利用了关系，而不是只匹配颜色和类别。

**阅读结论：** 十篇中最值得研究“如何训练细粒度区域—文本对齐”的一篇，且代码可供复现。

---

## 4. SceneProp — Combining Neural Network and Markov Random Field for Scene-Graph Grounding

**发表信息**

- 作者：Keita Otani, Tatsuya Harada
- 会议：WACV 2026
- DOI：[10.1109/WACV61042.2026.00595](https://doi.org/10.1109/WACV61042.2026.00595)
- 论文：[WACV 官方论文页](https://openaccess.thecvf.com/content/WACV2026/html/Otani_SceneProp_Combining_Neural_Network_and_Markov_Random_Field_for_Scene-Graph_Grounding_WACV_2026_paper.html)
- 代码：[KeitaOtani/SceneProp](https://github.com/KeitaOtani/SceneProp)

### 4.1 研究问题

假设文本查询是：“Find the woman holding a pen next to a tablet.” 单独判断候选框是不是 woman、是不是 pen、是不是 tablet，并不足以完成任务。真正的目标是找到一组候选框，使 woman、pen、tablet 的类别和 holding、next to 关系同时成立。

如果每个节点独立匹配，可能出现三个局部上看似正确、整体关系却不一致的组合。

SceneProp 研究的是：**如何让整个查询图的所有节点和关系联合参与定位。**

### 4.2 非常重要：SceneProp 不负责从自由文本生成图

它的输入是已经结构化的 Scene Graph：

\[
G=(O,R)
\]

其中 O 是物体节点，R 是关系边。每个节点有物体类别，每条边有关系类别。

论文没有提出完整的自由文本解析器。作者把利用 LVLM 将自然语言转换为查询图列为未来方向。因此，不能把 SceneProp 描述成“LLM 自动解析文本、自动生成拓扑图、再定位”的完整系统。

### 4.3 图像候选与关系能量

SceneProp 使用 ATSS 生成 512 个候选框，使用 Swin Transformer + FPN 提取视觉特征，并编码候选框的位置范围。

随后构建 Scene-MRF：

- Unary Potential：候选框是否匹配某个物体节点；
- Pairwise Potential：两个候选框是否满足某条关系边。

其联合概率形式为：

\[
P(A|G,I)\propto
\prod_i P(a_i|o_i)
\prod_{(j,k)}P(a_j,a_k|r_{jk})
\]

其中 I 是图像，G 是查询图，a_i 是分配给第 i 个查询节点的候选框，o_i 是节点类别，r_jk 是节点间关系。

一个候选框不仅要满足自身类别，还必须与其他候选框共同满足查询图中的关系约束。

### 4.4 全局推理：MAP、MPLP 和 Belief Propagation

SceneProp 区分两种推理目标：

1. Joint MAP：寻找整个图最可能的候选框分配组合；
2. Marginal：计算某个查询节点对应各候选框的边缘概率。

论文使用 MPLP 近似求解一般图上的 MAP 问题；对于边缘概率和训练，则使用 Belief Propagation 等消息传递方法。训练时会采样树结构子图，使 BP 可以进行精确计算。

这不是简单的 GNN message passing。MRF 的关键在于显式定义了联合概率分布，并在全局分配层面联合考虑节点和边的约束。

### 4.5 实验结果

论文在 VG-FO、VG150、COCO-Stuff、GQA 等数据集上评估。完整模型报告的 Recall@1 / Recall@5：

| 数据集 | R@1 | R@5 |
|---|---:|---:|
| VG-FO | 46.6 | 67.1 |
| VG150 | 43.7 | 68.4 |
| COCO-Stuff | 68.2 | 88.9 |
| GQA | 53.6 | 68.2 |

论文还做了关系数量变化实验，观察到 SceneProp 能利用更丰富的关系约束，而部分对比方法在关系数量增加时性能下降。

### 4.6 优点、不足与 UAV 启发

**优点：** 多实体和多关系联合匹配；全局推理能避免局部匹配彼此矛盾；可以解释每个节点和关系对最终分配的贡献；对复杂组合查询比单纯句子—区域相似度更有结构归纳偏置。

**不足：** 输入查询图需要预先提供；自由文本到图的解析不在本文方法范围内；闭集类别词表仍有限制；依赖候选框质量，漏检的目标无法通过图推理凭空恢复；原始评测不是 UAV 航拍定位。

**UAV 启发：** 若研究“指令生成图—图匹配图像”，SceneProp 是必须认真比较的基线。研究空间不在于“我们也用了图”，而在于查询图如何自动生成、不确定解析如何表示、航拍空间关系如何计算，以及如何扩展到跨帧和导航地图。

---

## 5. VIRO — Robust and Efficient Neuro-Symbolic Reasoning with Verification for Referring Expression Comprehension

**发表信息**

- 作者：Hyejin Park 等
- 会议：CVPR 2026
- 论文：[CVPR 官方论文页](https://openaccess.thecvf.com/content/CVPR2026/html/Park_VIRO_Robust_and_Efficient_Neuro-Symbolic_Reasoning_with_Verification_for_Referring_CVPR_2026_paper.html)
- 预印本：[arXiv:2601.12781](https://arxiv.org/abs/2601.12781)

### 5.1 研究问题

很多视觉定位方法默认输入文本一定能在图像中找到目标。但现实中可能出现图片里没有大象、图片里有人但没有人穿蓝色衣服、汽车不在要求的建筑左侧等情况。

如果模型总要从候选框中挑一个，就会产生错误定位。

VIRO 的核心是：**把定位任务变成可执行的程序，并在每一步检查视觉证据是否满足条件。**

### 5.2 文本处理：自然语言编译为操作序列

VIRO 使用 LLM 将自然语言转换成由有限操作符组成的程序。例如：

“the person left of the elephant”可以转换成：

    FIND("person")
    FIND("elephant")
    FIND_DIRECTION(person, elephant, "left")
    RESULT(person)

论文中定义了多类操作：

| 操作类别 | 典型操作 | 作用 |
|---|---|---|
| 目标识别 | FIND | 找到指定类别 |
| 属性过滤 | PROPERTY | 按颜色、衣着、状态等过滤 |
| 绝对位置 | LOCATE、ORDER | 选择中间、右侧、排序位置等 |
| 大小/深度 | SIZE、ABSOLUTE_DEPTH | 根据相对大小、前后深度筛选 |
| 相对空间关系 | FIND_DIRECTION、FIND_NEAR、FIND_INSIDE、RELATIVE_DEPTH | 检查目标与参照物的关系 |
| 输出 | RESULT | 返回最终候选 |

程序生成后，先通过语法/参数验证器检查格式、变量定义和操作参数是否合法。

### 5.3 执行与视觉验证

每个操作不仅执行，还必须验证：

- FIND：检测器生成候选框，CLIP 进一步检查候选是否真的属于目标类别；
- PROPERTY：利用 CLIP 和检测器分数筛选属性；
- FIND_DIRECTION：通过候选框几何关系检查方向；
- RELATIVE_DEPTH：使用深度信息验证前后关系；
- FIND_INSIDE：检查目标框与参照框的包含关系。

如果某一步得到空集合，程序立即终止，返回 no-target，而不是把错误候选继续传给后续模块。

### 5.4 一个关键设计：程序可复用

VIRO 将“生成程序”和“执行程序”分开。同一条文本查询生成的程序，可以在多张图像上重复执行，不必每张图都重新让 LLM 解析一遍。这对于批量图像定位有价值。

### 5.5 实验结果与边界

论文在 RefCOCO、RefCOCO+、RefCOCOg、RefAdv、gRefCOCO no-target 等基准上评估。

论文报告：

- gRefCOCO no-target/target-present 的 TestA balanced accuracy 为 61.1%；
- TestB balanced accuracy 为 56.9%；
- 标准 REC 测试中 RefCOCO TestA 为 71.9%，RefCOCO+ TestA 为 63.3%，RefCOCOg test 为 66.3%。

这些结果主要体现其组合推理与拒识能力，不应误读为在所有监督视觉定位方法中都达到最高精度。

### 5.6 优点、不足与 UAV 启发

**优点：** 文本被转成可检查的操作序列；关系约束逐步执行；空集合可以触发拒识；程序可跨图像复用；可以定位错误发生在哪个操作阶段。

**不足：** 操作符集合有限；程序语法正确不代表语义解析正确；检测器漏检、CLIP 误判、深度估计错误仍会影响结果；阈值校准影响 precision-recall；多阶段操作会带来推理时延。

**UAV 启发：** “Locate the red car between the two white trucks.” 系统应找红色汽车、找两辆白色卡车、验证汽车是否位于二者之间；若不满足，拒绝输出，而不是选择一个最像的汽车。

**阅读结论：** VIRO 的价值在于操作级验证和拒识，不是提出了更强的视觉编码器。

---

## 6. AerialVG — A Challenging Benchmark for Aerial Visual Grounding by Exploring Positional Relations

**发表信息**

- 作者：Jun-Li Liu 等
- 会议：ICCV 2025
- DOI：[10.1109/ICCV51701.2025.00492](https://doi.org/10.1109/ICCV51701.2025.00492)
- 论文：[ICCV 官方论文页](https://openaccess.thecvf.com/content/ICCV2025/html/Liu_AerialVG_A_Challenging_Benchmark_for_Aerial_Visual_Grounding_by_Exploring_ICCV_2025_paper.html)
- 代码：[Ideal-ljl/AerialVG](https://github.com/Ideal-ljl/AerialVG)

### 6.1 研究问题

自然图像中的视觉定位方法往往面对相对明显的目标。但航拍图像一张图里可能有大量相似车辆；小目标纹理和形状信息很少；俯视视角使外观线索不充分；参照物和目标之间的位置关系可能比目标自身外观更重要。

例如：“the white sedan with a blue SUV at its bottom-right”。如果只看 white sedan，图像里可能有多个白色轿车；加入蓝色 SUV 位于右下方的关系后，目标才可能唯一确定。

### 6.2 数据集与文本标注

论文构建 AerialVG：

- 5000 张高分辨率 UAV 图像；
- 约 50000 条人工描述；
- 约 103000 个物体实例。

描述包含目标属性、参照物和目标与参照物之间的八方向空间关系：above/below、left/right、top-left/top-right、bottom-left/bottom-right。

标注采用观察者视角，通常优先选取显著且相对近的辅助物体，避免冗余参照。

### 6.3 模型：HCA + RAG

**Hierarchical Cross-Attention（HCA）** 利用文本信息在不同尺度的图像特征之间传递注意力，帮助模型兼顾小目标与大范围场景语义。

**Relation-Aware Grounding（RAG）** 显式建模候选物体之间的两两关系，并将关系表示与文本特征进行交互，形成关系分数，再与物体分类/定位分数结合。

它不是将自然语言先转换成独立的符号图，再进行 SceneProp 式 MRF 全局推理；它主要通过神经网络内部的关系表示进行关系感知定位。

### 6.4 实验结果

AerialVG 的核心 Top-1 / Top-5 结果：

| 方法 | Top-1 | Top-5 |
|---|---:|---:|
| Grounding DINO | 29.36 | 78.87 |
| AerialVG 完整模型 | 50.01 | 87.00 |

模块消融：

| 配置 | Top-1 | Top-5 |
|---|---:|---:|
| Grounding DINO baseline | 29.36 | 78.87 |
| + HCA | 29.78 | 82.32 |
| + RAG | 45.63 | 79.63 |
| + HCA + RAG | 50.01 | 87.00 |

HCA 对 Top-5 候选召回更有帮助；RAG 对 Top-1 的最终目标消歧贡献更大；两者结合效果最好。

### 6.5 优点、不足与 UAV 启发

**优点：** 专门面向航拍视觉定位；目标属性与参照物关系共同参与定位；对小目标和相似目标具有针对性；有公开代码和数据集方向，适合做 UAV 定位基线。

**不足：** 论文没有证明它能处理任意长自然语言导航指令；关系建模主要围绕目标和辅助物体；背景环境语义（如道路、建筑区域）仍然较难充分利用；Top-1/Top-5 反映候选检索与选择，但不等于复杂空间逻辑完全正确；需要进一步确认多目标、无目标、跨帧关系等能力。

**UAV 启发：** 在无人机视角中，空间关系不是可有可无的辅助信息，而是消除同类目标歧义的重要线索。AerialVG 可以作为航拍图像视觉定位的强相关基线。

---

## 7. FILD-Nav — Vision-and-Language Navigation with Instruction Landmark Features in Continuous Environments

**发表信息**

- 作者：Chuan-Ye Hu, Lu-Lu Liu, Huai-Wei Si, Ya-Wen Zhao, Nan Ding
- 会议：IJCAI/ECAI 2026 proceedings
- DOI：[10.24963/ijcai.2026/841](https://doi.org/10.24963/ijcai.2026/841)
- 任务：连续环境 Vision-and-Language Navigation
- 注意：论文实验环境为 R2R-CE，不是 UAV 航拍环境。

### 7.1 研究问题

长导航指令通常包含多个地标。如果模型只将整条指令编码成一个全局向量，当前时刻可能无法知道现在应该寻找哪个地标、哪些地标已经经过、下一步应当关注哪个语义线索，以及当指令中的地标不可见时能否利用其他相关物体。

FILD-Nav 从指令中抽取 Landmark Features，并把它们显式送入局部路径点预测和全局拓扑规划。

### 7.2 文本处理：LLM 提取有序地标

论文使用 DeepSeek 在离线预处理阶段抽取：

- 3–8 个有序 primary landmarks；
- 每个 primary landmark 对应 5 个 co-occurring landmarks。

primary landmark 是指令中需要按顺序寻找的主要地标；co-occurring landmark 是与主要地标相关、可能在场景中共现的辅助物体。

例如：

    primary:
    bedroom -> table -> chair -> floor

    co-occurring:
    bedroom -> bed / pillow / wardrobe ...

共现不等于空间关系：pillow 可能与 bed 共现，却不表示 pillow 必然在 bed 的某个固定方向。

### 7.3 视觉侧修正地标先验

作者没有直接相信 LLM 输出。在每个导航步骤，CLIP 会把候选地标文本与当前视觉观测进行相似度匹配，并对相邻主地标和共现地标进行视觉加权。

因此，语言模型提供先验，视觉观测提供当前证据，最后形成修正后的 landmark features。

### 7.4 两个注入位置

地标特征进入两个模块：

1. Waypoint Prediction：帮助局部路径点预测，提升路径点与任务语义的相关性；
2. Topological Planning：将地标语义融入拓扑节点表示，再与完整指令特征进行跨模态规划。

这使地标语义不仅影响“当前往哪走”，也影响“全局图中哪个节点更接近当前任务”。

### 7.5 实验结果

论文在 VLN-CE 上报告：

- Success Rate 提升约 2 个百分点；
- SPL 提升约 3 个百分点；
- Oracle Success Rate 提升约 7 个百分点。

这些是相对论文所用基线的报告增益，不应解释为任意 VLN 数据集或 UAV 场景均能获得相同提升。

### 7.6 优点、不足与 UAV 启发

**优点：** 将长指令转为有序地标序列；引入共现地标，增加视觉线索；视觉观测可以修正语言先验；地标特征同时进入局部 waypoint 和全局拓扑规划。

**不足：** 地标提取和共现关系依赖 LLM 先验；共现不等于空间关系，可能引入错误关联；论文没有给出完整可复现的提示词和地标排序算法；主要实验为室内连续 VLN，不能直接证明 UAV 场景有效。

**UAV 启发：** 长距离无人机导航可以把自然语言转为地标序列，并记录每个地标的语义类别、属性、共现对象、与前后地标的空间关系和当前是否已观测/已通过。必须区分“共现关系”和“几何关系”。

**阅读结论：** 值得借鉴有序地标与视觉修正，但迁移到 UAV 前必须重新定义地标、方向参考系和跨视角记忆。

---

## 8. GeoNav — Empowering MLLMs with Dual-Scale Geospatial Reasoning for Language-Goal Aerial Navigation

**发表信息**

- 作者：Hao-Tian Xu, Yue Hu, Chen Gao, Zheng-Qiu Zhu, Yong Zhao, Quan-Jun Yin
- 期刊：Pattern Recognition，2026
- DOI：[10.1016/j.patcog.2026.113365](https://doi.org/10.1016/j.patcog.2026.113365)
- 数据集：CityNav
- 任务：Language-Goal Aerial Navigation

### 8.1 研究问题

无人机在城市中按照自然语言指令导航，与室内 VLN 有明显不同：目标可能距离无人机数百米；当前相机视野有限；高空图像中物体很小；城市环境中存在大量相似建筑、车辆和道路；只根据当前图像做局部反应，很容易丢失全局空间结构。

GeoNav 将导航拆成粗到细的阶段，并维护全局和局部两种空间表示。

### 8.2 三阶段导航

1. Landmark Navigation：先飞向指令中的地标区域；
2. Target Search：在地标附近搜索目标；
3. Precise Localization：利用局部空间关系确定目标并精确接近。

例如：“Find the dark red car in the parking row near the library.” 可以被规划为先导航到 library 附近、在停车区域内搜索、根据颜色/停车行/相邻车辆关系确定具体目标。

### 8.3 双尺度空间表示

**Schematic Cognitive Map（SCM）**：全局示意认知地图，融合先验地理信息、无人机位置和观测到的物体，形成俯视的空间表示。它解决无人机现在在哪里、目标地标在哪、应该朝哪个区域移动。

**Hierarchical Scene Graph（HSG）**：局部层次化场景图，节点包括 block、landmark/POI、observed object。边包含 contains、adjacent_to、near_corner、north/south/east/west 及对角方向等。它解决在目标区域内部，哪个物体满足指令中的属性和空间关系。

### 8.4 视觉感知与地图更新

GeoNav 使用 GroundingDINO 检测对象，SAM 获得分割区域，再结合无人机位姿和朝向将视觉对象投影到世界坐标，更新全局空间表示。

HSG 在搜索过程中持续更新，并通过空间相似度与语义相似度合并重复观测到的节点。

因此，GeoNav 的关键不是单次图像中的对象匹配，而是把多个时刻的视觉证据逐步积累到空间记忆中。

### 8.5 MLLM 空间 CoT 与操作链

不同阶段的 MLLM 输入不同：

- Navigate 阶段：全局 SCM、地标目标、无人机相对地标的位置；
- Search 阶段：更新后的 SCM、当前视野和待寻找目标；
- Localize 阶段：对 HSG 执行查询，例如先找到地理地标，再按空间关系筛选对象类别和属性。

如果原始关系查询没有结果，系统会放宽约束后再次查询。

### 8.6 实验结果

GeoNav 在 CityNav 上评估，平均飞行距离约 545 m。Test Unseen 上：

| 指标 | GeoNav |
|---|---:|
| Navigation Error | 73.5 m |
| Success Rate | 25.9% |
| Oracle Success Rate | 41.6% |
| SPL | 16.0% |

论文报告，相较第二名方法，SR 在三个 split 上分别提高约 18.4、8.7、14.6 个百分点。

不过，GeoNav 与人类表现仍有明显差距。论文报告 Test Unseen 人类 SR 为 87.9%，GeoNav 为 25.9%。

### 8.7 优点、不足与 UAV 启发

**优点：** 直接面向 UAV 语言目标导航；全局地图与局部场景图分工明确；先验地理知识和在线视觉观测融合；目标搜索与精确定位分阶段进行；空间记忆可跨多个观测时刻积累证据。

**不足：** MLLM 搜索动作仍可能错误，导致重复探索；图节点去重错误可能合并相似目标；高空图像分辨率下降会影响小目标识别；仍存在明显的人机性能差距；地图构建、投影和位姿误差会影响后续推理。

**UAV 启发：** 全局地图回答“去哪一片区域”，局部场景图回答“区域里哪个对象”，文本结构回答“目标是什么、与谁有什么关系”。这比让一个 VLM 直接从全局指令输出飞行动作更容易解释和调试。

**阅读结论：** 若研究目标是 UAV-VLN，而不只是单张航拍图像定位，GeoNav 应作为核心系统级参考。

---

## 9. DisCoCLIP — A Distributional Compositional Tensor Network Encoder for Vision-Language Understanding

**发表信息**

- 作者：Kin Ian Lo 等
- 会议：*SEM 2025
- 论文：[ACL Anthology](https://aclanthology.org/2025.starsem-1.25/)
- 预印本 DOI：[10.48550/arXiv.2509.21287](https://doi.org/10.48550/arXiv.2509.21287)

### 9.1 研究问题

CLIP 将文本压缩为一个全局向量，擅长学习整体图文相似度。但全局向量可能弱化句法和组合结构：

- “A chases B” 与 “B chases A”；
- “red car beside blue bus” 与 “blue car beside red bus”；
- “person holding cup” 与 “cup holding person”。

这些句子包含相似词汇，但语义关系不同。

DisCoCLIP 的问题意识是：**能否从文本编码器本身入手，让模型更明确地保留句法组合结构？**

### 9.2 文本处理：CCG 句法解析

论文使用 Combinatory Categorial Grammar（CCG）解析句子。CCG 为词语分配语法类型：

- 名词短语 NP；
- 句子 S；
- 形容词可看作 NP/NP；
- 及物动词可看作 (S\NP)/NP。

例如 “Alice loves Bob”：

- Alice：NP；
- loves：(S\NP)/NP；
- Bob：NP。

CCG 组合规则将 loves 与 Bob 组合，再与 Alice 组合，最终得到句子 S。

### 9.3 分布式张量组合

DisCoCLIP 不只是将所有词向量平均或拼接。它为词语构造与语法类型相对应的张量：名词是向量、形容词是矩阵、及物动词是更高阶张量。

随后按照 CCG 推导树执行张量收缩，将词语组合成句子向量。

直观理解：

- 传统 CLIP：词语进入 Transformer，最终汇总成一个句子向量；
- DisCoCLIP：先明确句法组合关系，再按语法结构组合词语表示。

论文还比较 Compact、Tree、Cups、Spider 等不同张量网络结构，其中 Spider 是更接近 bag-of-words 的对照。

### 9.4 视觉编码器与训练

DisCoCLIP 保留冻结的 CLIP Vision Transformer，主要替换/学习文本侧张量网络。训练使用图文对比学习 InfoNCE：真实图文对作为正样本，batch 内其他不匹配图文组合作为负样本。

因此，它主要检验文本组合结构是否能改善全局图文匹配，而不是直接训练目标框或 mask。

### 9.5 实验结果

论文报告：

| 任务 | CLIP | DisCoCLIP Compact |
|---|---:|---:|
| SVO-Probes verb accuracy | 77.60 | 82.42 |
| SVO-Swap | 57.89 | 93.68 |
| ARO Attribution | 61.00 | 70.01 |
| ARO Relation | 51.53 | 55.81 |

注意：SVO-Swap 测试只有 95 对样本，统计规模较小；BLIP 在 SVO overall 等部分指标上仍然更高；DisCoCLIP 并不是所有组合语义指标都全面超过所有模型。

### 9.6 优点、不足与 UAV 启发

**优点：** 从文本编码器结构上显式保留语法信息；对谓词、主宾语顺序敏感；张量网络可以进行参数分解；视觉编码器冻结，便于分析文本侧变化的作用。

**不足：** 依赖 CCG 解析质量；高阶张量组合仍有计算和表示能力权衡；评测主要是全局图文匹配；没有证明它可以直接把词语定位到图像区域；小规模 SVO-Swap 结果不能作为充分的泛化证据。

**UAV 启发：** 可将 DisCoCLIP 作为文本编码器侧的思路，将 AerialVG/S²CLNet 的区域对齐作为视觉定位侧，研究语法结构表示是否能改善目标—参照物关系匹配。但不能简单把 DisCoCLIP 的全局图文匹配提升等同于视觉定位能力提升，需要额外 phrase-region grounding 和关系定位实验。

---

## 10. GeoRefer-Bench — A Benchmark from Referring Pixels to Verifiable Geospatial Reasoning

**发表信息**

- 作者：Shuai Cao 等
- 状态：arXiv 2026 预印本，未确认正式发表
- 论文：[arXiv:2609.29541](https://arxiv.org/abs/2609.29541)
- 类型：数据集与评测基准，不是新的定位模型。

### 10.1 研究问题

传统视觉定位通常使用 IoU、mIoU、Acc@0.5 等指标。但一个模型可能获得较高的 mask IoU，却没有真正理解文本中的空间关系。

例如：“Select all cars east of the building and north of the road.” 模型即使输出一个大致重叠的区域，也可能没有正确执行 east of 和 north of 两个条件。

GeoRefer-Bench 试图将遥感/无人机指代定位提升为可验证的空间查询问题。

### 10.2 数据和查询表示

论文报告：

- 700 个完整 UAV 场景；
- 2048×2048 图像；
- 约 2.94 Gpx 总像素；
- 26,217 个实例；
- 142,796 条空间关系；
- 20,916 条可执行查询。

查询覆盖五个推理难度级别，并包含多种自然语言改写、无法回答的查询、反事实查询对、多跳关系和多目标集合查询。

文本查询被转化为可执行逻辑形式，在度量场景图上执行。

### 10.3 Exact Query Success

GeoRefer-Bench 的重要指标是 Exact Query Success（EQS）。

与 IoU 不同，EQS 要求模型返回的实例集合与逻辑查询所定义的实例集合完全一致。

例如查询定义“所有位于建筑东侧且在道路北侧的汽车”。如果模型只找到其中一部分，即使 mask 重叠度很高，也不算完全成功。

这能揭示模型是否真正使用了关系约束。

### 10.4 论文报告的主要观察

论文报告：

- 在 15 个现有模型上进行评估；
- 最强模型总体 EQS 为 74.1；
- 从 Level 1 到 Level 5，准确率明显下降；
- 多个模型在两跳空间关系查询上表现很差；
- 忽略关系的策略仍可能获得非零 mIoU，但 EQS 很低。

这些结果说明，常规像素重叠指标不足以衡量空间关系理解能力。

### 10.5 优点、不足与 UAV 启发

**优点：** 面向 UAV/遥感空间关系推理；查询可执行、结果可验证；有无目标、反事实、多跳关系等困难场景；EQS 能补充 IoU/mIoU 的不足；适合评估模型是否真正执行了文本空间约束。

**不足：** 是基准，不是新的模型方法；预印本状态，尚需关注后续审稿和版本变化；数据规模和查询生成方式仍需要独立审计；EQS 严格，但也可能对边界标注、几何阈值和实例分割质量敏感。

**UAV 启发：** 新方法不应只报告 mIoU 或 Acc@0.5。可以加入 relation accuracy、multi-hop query success、target set precision/recall、no-target rejection、counterfactual consistency、exact query success。

**阅读结论：** 十篇中最值得借鉴评测思想的一篇。它能帮助区分“模型碰巧找到一个重叠区域”和“模型真正理解并执行了空间关系”。

---

## 11. 十篇论文横向比较

| 论文 | 文本表示 | 图像侧证据 | 是否显式关系推理 | 是否验证/拒识 | UAV 直接相关性 |
|---|---|---|---|---|---|
| HTVG | 层次化语义 | 多层视觉特征 | 依赖文本—视觉模块设计 | 未确认 | 遥感分割 |
| NAVER | ProbLog 规则 | 检测、属性、关系概率 | 是，概率逻辑 | 是，自动机+Answerer | 可迁移 |
| S²CLNet | 文本特征+区域对齐 | mask 引导局部视觉特征 | 不是完整符号关系图 | 未强调 | 遥感分割 |
| SceneProp | 已给定 Scene Graph | 候选框和关系特征 | 是，MRF 全局推理 | 主要是概率推理 | 可迁移 |
| VIRO | 可执行操作程序 | 检测、CLIP、深度、几何 | 是，操作级约束 | 是，空集合拒识 | 可迁移 |
| AerialVG | BERT 编码自然语言描述 | 多尺度航拍候选和关系特征 | 神经关系建模 | 不是主要贡献 | 直接相关 |
| FILD-Nav | 有序地标+共现地标 | CLIP 观测匹配 | 地标序列和导航图 | 视觉修正 | 间接相关 |
| GeoNav | 指令分阶段子目标 | SCM + HSG + 在线感知 | 是，局部空间图查询 | 查询失败可放宽 | 直接相关 |
| DisCoCLIP | CCG+张量组合 | 冻结 CLIP 全局图像向量 | 语法组合，不是显式视觉场景图 | 否 | 基础方法可迁移 |
| GeoRefer-Bench | 可执行空间逻辑查询 | 度量场景图和实例 | 是，逻辑查询执行 | 包含不可回答查询 | 直接相关基准 |

### 11.1 三个重要区别

**A. 文本解析不等于文本编码**

- NAVER：将文本转成逻辑程序；
- VIRO：将文本转成操作序列；
- FILD-Nav：将文本转成有序地标；
- DisCoCLIP：将文本句法结构融入连续向量表示；
- HTVG：从摘要层面可确认是层次化文本语义引导。

它们的输出完全不同，不能统称为“LLM 解析文本”。

**B. 关系建模不等于全局关系约束**

- AerialVG：在神经网络中学习候选物体之间的关系特征；
- SceneProp：将关系作为 MRF 的 pairwise potential，联合求解候选框分配；
- NAVER：将关系变成逻辑事实并进行概率推理；
- GeoNav：将空间关系存入在线维护的场景图，再进行查询。

四者的建模层次不同。

**C. 视觉验证不等于模型解释**

- VIRO：每个操作都有明确的验证结果；
- NAVER：候选经过 Answerer 检查，不满足时可重试；
- GeoRefer-Bench：通过查询执行结果验证模型是否正确；
- SceneProp：输出全局概率匹配，但并不等于对原始自然语言做独立验证。

---

## 12. 面向当前研究问题：哪些技术最值得借鉴？

当前的核心问题是：**当前场景中，文本和图像对齐能力不足。**

我建议把这个问题进一步细分成三个可检验的假设。

### 12.1 假设一：模型丢失了文本中的组合结构

典型错误：属性绑定错对象、left/right 关系反转、主体和参照物混淆、多条关系没有按正确顺序组合。

可借鉴 DisCoCLIP（文本编码器结构）、NAVER（显式逻辑）、VIRO（可执行操作）。

实验：固定视觉 Backbone，只改变文本表示，比较原始整句、目标/属性/关系拆分、SPO 三元组、逻辑程序、结构化图。如果结构化表示在关系反转、属性交换和未见组合上明显更好，才有证据说明文本处理产生了贡献。

### 12.2 假设二：模型知道文本含义，但无法将它对齐到正确图像区域

典型错误：识别出红色汽车，却定位到另一辆红色汽车；目标类别正确，参照物错误；全局相似度高，但局部区域不对应；背景语义压过目标语义。

可借鉴 S²CLNet（区域级对比学习）、AerialVG（关系感知候选匹配）、HTVG（层次化文本—视觉引导）。

实验：固定候选框，比较全局文本特征、目标短语、目标+属性、目标+关系、结构约束对比损失；同时记录目标区域与困难负样本的相似度差距。

### 12.3 假设三：局部匹配正确，但多实体组合不一致

典型错误：找到了正确类别的两个物体，但它们之间的关系不成立；多条关系中只有一条成立；多跳关系中某个中间参照物选错；目标实际不存在，但模型仍然输出一个候选。

可借鉴 SceneProp（全局关系约束）、NAVER（概率逻辑组合）、VIRO（操作级验证）、GeoRefer-Bench（严格集合级评测）。

实验：构造同一图像的反事实指令：A 在 B 左侧、A 在 B 右侧、B 在 A 左侧、A 位于 B 和 C 之间。统计关系正确率、组合成功率和无目标拒识率。

---

## 13. 推荐的最小研究实验

不要一开始就把十篇论文的模块全部拼到一个系统里。那样即使结果提升，也很难说明到底哪个模块起作用。

### 13.1 固定视觉侧

固定同一图像 Backbone、同一候选框、同一候选区域特征、同一检测器、同一训练数据和数据划分。这样才能隔离文本表示的贡献。

### 13.2 文本侧做四个版本

| 实验 | 文本表示 |
|---|---|
| Baseline | 原始完整指令 |
| Exp-A | 目标实体 + 属性 |
| Exp-B | 目标实体 + 参照物 + 有向关系 |
| Exp-C | 结构化查询图/逻辑约束 |

### 13.3 匹配侧做三个版本

| 实验 | 匹配方式 |
|---|---|
| Match-1 | 全局文本—图像相似度 |
| Match-2 | 实体/短语—区域匹配 |
| Match-3 | 节点匹配 + 关系约束联合推理 |

### 13.4 验证侧做两个版本

- Verify-0：直接返回最高分候选；
- Verify-1：使用原始完整指令验证候选，允许 no-target。

这样至少可以区分文本结构、区域对齐、全局关系推理和验证分别产生了什么作用。

### 13.5 评测指标

| 指标 | 目的 |
|---|---|
| Acc@0.5 / mIoU | 常规定位质量 |
| Target-Anchor Accuracy | 目标与参照物是否都定位正确 |
| Relation Accuracy | 空间关系是否正确 |
| Compositional Success | 多个条件是否同时满足 |
| Counterfactual Consistency | 关系反转后预测是否随之改变 |
| No-target TNR | 无目标场景是否能拒识 |
| Multi-target F1 | 多目标集合是否正确 |
| EQS | 整条可执行空间查询是否完全正确 |

---

## 14. 最终研究判断

综合十篇论文，我认为最有价值的研究路线不是简单地做：

> LLM 解析文本 + Scene Graph + VLM 定位。

因为这些组件分别已经有较成熟的先例：

- 文本结构解析：DisCoCLIP、NAVER、VIRO；
- 区域级对齐：S²CLNet、AerialVG；
- 全局图推理：SceneProp；
- UAV 空间记忆和导航：GeoNav；
- 严格空间关系评测：GeoRefer-Bench。

更值得研究的方向是：

### 方向一：带不确定性的语言查询图

LLM 生成的图不一定完全正确。与其强制输出一个确定图，不如保留实体候选及其置信度、属性候选及其置信度、关系候选及其置信度和多种可能的解析结构。然后让视觉证据参与图结构消歧。

这比“先生成一个确定图，后面无条件相信它”更鲁棒。

### 方向二：视觉证据反向修正文本解析

现有方法经常是文本先解析、视觉后匹配。但实际上，视觉场景也能帮助判断文本结构。

例如：“the car near the building behind the tree”。如果图像中存在多种可能的关系附着结构，视觉证据可以帮助判断 behind 修饰的是 building 还是 car。

可以形成一个迭代过程：

文本解析 → 初始查询图 → 图像候选匹配 → 关系一致性检查 → 修正查询图或保留多个解析 → 最终定位。

这是值得进一步验证的研究方向，但需要设计严格实验，证明视觉反馈确实减少了语义解析错误，而不是只靠额外调用更强的 VLM。

### 方向三：跨视角、跨时刻的空间关系一致性

UAV 不只看单张图像。同一目标可能在多个视角中出现：视角 A 汽车在建筑右侧；视角 B 汽车在建筑下方；视角 C 汽车与另一辆白车相邻。

如果系统能够将这些观察映射到统一空间坐标系，就能检查不同视角下的关系是否一致。

GeoNav 的 SCM/HSG 提供了系统级参考；GeoRefer-Bench 提供了更严格的关系评测思路。

---

## 15. 论文阅读优先级

| 优先级 | 论文 | 为什么先读 |
|---|---|---|
| 1 | AerialVG | 最贴近航拍视觉定位，先理解任务本身 |
| 2 | SceneProp | 理解多实体关系如何联合匹配 |
| 3 | NAVER | 理解自然语言到逻辑约束的转换 |
| 4 | S²CLNet | 理解如何训练区域—文本细粒度对齐 |
| 5 | GeoNav | 理解完整 UAV-VLN 系统如何维护空间记忆 |
| 6 | VIRO | 理解操作级验证与拒识 |
| 7 | DisCoCLIP | 理解如何改造文本编码器以保留句法结构 |
| 8 | FILD-Nav | 理解有序地标如何进入连续 VLN |
| 9 | GeoRefer-Bench | 理解如何设计真正考验空间关系的评测 |
| 10 | HTVG | 获取层次化遥感文本—视觉引导思路；全文需进一步核验 |

## 结语

这十篇论文共同说明：

1. 只依赖整句—整图相似度，很难保证复杂组合语义正确；
2. 文本中的实体、属性和关系需要被保留，并且需要获得视觉证据；
3. 多实体定位需要联合考虑节点和关系，不能只做独立候选打分；
4. 复杂定位系统应允许验证失败和拒绝输出；
5. UAV-VLN 还需要全局空间记忆、地标进度和跨视角一致性。

**对当前研究最重要的一句话：**

> 不要只问模型能不能理解这句话，而要问这句话中的每一项语义约束，是否都能在图像或空间地图中找到对应证据，并且这些证据组合起来是否仍然成立。

这应该成为后续方法设计和实验评测的核心标准。

---

## 参考链接与证据说明

- HTVG：[DOI](https://doi.org/10.1016/j.patcog.2026.113579)
- NAVER：[ICCV 官方论文](https://openaccess.thecvf.com/content/ICCV2025/html/Cai_Naver_A_Neuro-Symbolic_Compositional_Automaton_for_Visual_Grounding_with_Explicit_ICCV_2025_paper.html) · [官方代码](https://github.com/ControlNet/NAVER)
- S²CLNet：[DOI](https://doi.org/10.1109/TGRS.2026.3724005) · [官方代码](https://github.com/45degreesl/S2CLNet)
- SceneProp：[DOI](https://doi.org/10.1109/WACV61042.2026.00595) · [官方代码](https://github.com/KeitaOtani/SceneProp)
- VIRO：[CVPR 官方论文](https://openaccess.thecvf.com/content/CVPR2026/html/Park_VIRO_Robust_and_Efficient_Neuro-Symbolic_Reasoning_with_Verification_for_Referring_CVPR_2026_paper.html) · [arXiv](https://arxiv.org/abs/2601.12781)
- AerialVG：[ICCV 官方论文](https://openaccess.thecvf.com/content/ICCV2025/html/Liu_AerialVG_A_Challenging_Benchmark_for_Aerial_Visual_Grounding_by_Exploring_ICCV_2025_paper.html) · [官方代码](https://github.com/Ideal-ljl/AerialVG)
- FILD-Nav：[IJCAI 论文](https://ijcai.org/proceedings/2026/841)
- GeoNav：[DOI](https://doi.org/10.1016/j.patcog.2026.113365)
- DisCoCLIP：[ACL Anthology](https://aclanthology.org/2025.starsem-1.25/) · [arXiv](https://doi.org/10.48550/arXiv.2509.21287)
- GeoRefer-Bench：[arXiv](https://arxiv.org/abs/2609.29541)

### 证据等级

- 已发表论文：以会议/期刊正式论文页和 DOI 为准。
- 预印本：明确标注 arXiv，不将其写成已正式发表。
- 实验数字：仅在能够从论文记录/全文中核对时引用；不同模型配置、不同数据 split 不混合比较。
- 代码观察：仅描述官方公开仓库中可直接检查到的实现，不将代码默认值误写成论文所有实验的统一设置。
- 全文不可得：HTVG 的部分实现细节和实验数字未在本报告中补写，需取得全文后再完善。
