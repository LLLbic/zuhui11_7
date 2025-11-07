# “One Model Fits All Nodes”: Neuron Activation  Pattern Analysis-Based Attack Traffic Detection  Framework for P2P Networks

## 本文解决的问题

## fig2里面的token construction是干什么的？
“Token Construction”（标记构造）是 tNeuron 框架训练阶段的第一个步骤，其主要目的是**构建用于训练 Transformer 影子模型（shadow model）的数据集 $\left\{X_T, y_T\right\}$** [1-3]。

tNeuron 设计 Token Construction 步骤是为了提取流量特征并生成可供影子模型学习的标签，从而提供用于后续神经元激活模式分析的丰富信息 [1, 4, 5]。

Token Construction 的具体工作内容和流程如下：

1.  **收集和组织数据包（Collect and Organize Packets）：**
    tNeuron 首先从用于训练（原始）检测模型的训练数据集中收集数据包 [1, 2]。然后，它根据数据包头中的五元组（five-tuples）将这些数据包组织成流（flows）[1, 2, 6]。
2.  **特征提取与标记化（Tokenization）：**
    tNeuron 随后对流中的每个数据包进行标记化（tokenization），方法是提取**每数据包的特征** [1]。具体提取的特征包括前 $L_{Max}$ 个数据包的**到达间隔（以微秒计）、IANA 协议号以及数据包长度（以字节计）** [2]。对于数据包数量少于 $L_{Max}$ 的流，tNeuron 会在剩余位置放置零向量进行表示 [2]。这个标记化的数据集被称为 $X_T$ [2, 3]。
3.  **掩盖与目标选择（Masking and Target Selection）：**
    tNeuron 会掩盖（mask）表示服务类型的端口号，因为这些字段是**将由影子模型预测的目标** [1]。
4.  **标签构建（Label Construction）：**
    为了构建标签 $y_T$，tNeuron 使用 **K-Means 算法**对源端口号和目标端口号（$y_p$），分别针对入站和出站流量进行聚类 [1, 3, 7]。
5.  **目标标签分配（Target Label Assignment）：**
    tNeuron 根据与端口号关联的**聚类中心**来为每个流构造标签 [1, 7]。这个步骤的目的是训练影子模型基于流量模式来预测这些端口号所属的聚类 [7]。

**为何选择端口号进行预测：**
tNeuron 训练影子模型来预测基于端口号的标签，是因为端口号可以为预测提供多样化的标签，并且这些标签可以直接获得，**无需大量人工工作** [7]。影子模型的准确性并不是重点，tNeuron 关注的是其神经元激活模式，用以分析流量知识 [8, 9]。

通过 Token Construction 步骤创建的 $X_T$ 和 $y_T$ 数据集将被用于后续的“Transformer Shadow Model Training”（Transformer 影子模型训练）阶段 [3, 10]。影子模型通过学习这些标记序列，提取流量模式的知识，并提供丰富的神经元激活模式，从而支撑 tNeuron 准确且鲁棒地检测误报（FPs）[4, 10, 11]。

形象地说，如果 tNeuron 的影子模型是一座旨在理解交通规则的 AI 大脑，那么“Token Construction”就像是**把原始的交通视频（数据包）剪辑成标准化的、带有明确标记（端口号类别）的短片（流特征 $X_T$ 和标签 $y_T$）**。这些标准化、易于学习的短片，帮助 AI 大脑理解正常的交通模式，即使它后来遇到未在训练中见过的交通场景（未见过的良性流量），也能通过比较神经元的反应（激活模式）来判断报警是否属于误报。
