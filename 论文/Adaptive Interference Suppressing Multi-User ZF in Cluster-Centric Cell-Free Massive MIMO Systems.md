
# Adaptive Interference Suppressing Multi-User ZF in Cluster-Centric Cell-Free Massive MIMO Systems

## **论文综合解析：《簇中心无蜂窝大规模MIMO系统中的自适应干扰抑制多用户迫零技术》**

### **摘要 (Abstract)**

本文研究了一种**簇中心**（Cluster-Centric, CC）架构的**无蜂窝大规模MIMO**（Cell-Free Massive MIMO, CF-mMIMO）系统。该系统通过将邻近用户和天线进行逻辑分组（形成“簇”）来实现在簇内的高效**多用户空分复用**。为了解决各簇之间的干扰问题，论文采用了**干扰抑制多用户迫零**（Interference Suppressing Multi-User Zero-Forcing, IS-MU-ZF）技术。然而，研究发现传统的IS-MU-ZF存在由**噪声增强**（Noise Enhancement）效应导致的严重**容量骤降**问题。为此，本文提出了一种**自适应干扰用户选择方法**，该方法能根据系统状态动态调整需要抑制的干扰用户数量。仿真结果表明，搭载了该自适应算法的CC系统，在性能上接近于更复杂的MMSE方案，并且在用户容量和计算效率方面，均优于当前流行的**用户中心**（User-Centric, UC）架构。

---

### **第一部分：系统模型与核心技术原理**

#### **1. 系统架构：簇中心无蜂窝大规模MIMO (CC-CF-mMIMO)**

*   **物理实体**：网络由 A 根分布式天线单元（或AP）和 U 个单天线用户构成。所有天线通过**前传网络**（Fronthaul）连接到一个或多个**处理单元**（Processing Unit）。
*   **逻辑划分**：
    1.  **用户聚类**：处理单元采用**受约束的K-means算法**将 U 个用户划分为 K 个逻辑上的**用户簇** $S_k$，每个簇的用户数不超过 $U_{\text{max}}$。
    2.  **天线关联**：为每个簇 k 动态关联一个**天线服务集** $A_k$。该服务集是簇内所有用户各自的“最优天线列表”的并集，最优的判断依据是**大尺度衰落**（路径损耗+阴影衰落）。不同簇的服务集**允许重叠**，这是保留**宏分集**（Macro-diversity）优势的关键。
*   **核心优势**：
    *   **宏分集**：一个用户由地理上分散的多根天线协同服务，有效对抗阴影衰落，保证了连接的**可靠性**。
    *   **多用户空分复用 (MU-MIMO)**：一个天线服务集 $A_k$ 在同一时频资源上同时服务簇内多个用户，极大地提升了系统的**容量**和**频谱效率**。

#### **2. 上下行信道传输过程（从单个天线和处理单元的视角）**

本文基于**TDD（时分双工）模式**，利用**信道互易性**。这意味着上行信道 $\mathbf{h}_{\text{UL}}$ 和下行信道 $\mathbf{h}_{\text{DL}}$ 满足 $\mathbf{h}_{\text{DL}} \approx \mathbf{h}_{\text{UL}}^*$，因此只需进行上行信道估计。

*   **上行链路 (Uplink)：从接收到恢复**
    1.  **用户发射**：在同一时刻，所有 U 个用户并行发射各自的独立数据符号 $s_u$。
    2.  **天线接收**：网络中的**每一根天线 a**，作为一个被动的传感器，接收到的是所有 U 个用户信号在空中物理叠加后的混合信号，以及自身的接收机噪声 $n_a$。天线 a 接收到的标量信号为：
        $$ y_a = \sum_{u=1}^{U} h_{a,u} \sqrt{P_u} s_u + n_a $$
        其中 $h_{a,u}$ 是用户 u 到天线 a 的信道系数，$P_u$ 是发射功率。
    3.  **数据采集**：负责簇 k 的**处理单元**通过前传网络，**只采集**其服务集 $A_k$ 内所有天线上的信号数据，形成一个混合信号向量 $\mathbf{y}_k \in \mathbb{C}^{|A_k| \times 1}$。
    4.  **并行恢复**：对于簇 k 内的**每一个用户 u**，处理单元都**独立地**、**并行地**执行以下操作：
        *   **计算权重**：根据信道估计结果，计算出专门用于恢复用户 u 信号的合并权重向量 $\mathbf{w}_u \in \mathbb{C}^{A \times 1}$（其中只有对应 $A_k$ 的元素非零）。
        *   **空间滤波（加权求和）**：处理单元执行向量内积运算，将服务集 $A_k$ 内每根天线 a 接收到的信号 $y_a$ 乘以对应的权重分量 $w_{a,u}^*$，然后将所有结果**叠加**起来：
            $$ \hat{s}_u \approx \sum_{a \in A_k} w_{a,u}^* y_a = \mathbf{w}_u^H \mathbf{y} $$
            这个数学运算通过**数字域的干涉消除**，从混合信号中提取出用户 u 的纯净信号估计值 $\hat{s}_u$。

*   **下行链路 (Downlink)：从准备到发射**
    1.  **数据准备**：**处理单元**为簇 k 内的**每一个用户 u** 准备好要发送的数据符号 $s_u$。
    2.  **独立加权 (预编码)**：处理单元利用信道互易性，为**每一个用户 u** 计算其专属的预编码权重 $\mathbf{w}_u^*$。然后，将用户 u 的数据 $s_u$ 与其权重向量相乘，生成一个加权信号向量 $\mathbf{x}_u = \mathbf{w}_u^* s_u \in \mathbb{C}^{A \times 1}$。这个向量的每个元素 $(\mathbf{x}_u)_a$ 代表了天线 a 需要为用户 u 发射的信号分量。
    3.  **信号叠加**：**处理单元**将所有这些为不同用户准备的加权信号向量**在数字域叠加**，得到一个总的、将要发射的信号向量：
        $$ \mathbf{x}_{\text{total}} = \sum_{u \in S_k} \mathbf{x}_u = \sum_{u \in S_k} \mathbf{w}_u^* s_u $$
    4.  **信号分发与发射**：处理单元通过前传网络，将总信号向量 $\mathbf{x}_{\text{total}}$ 的**第 a 个分量** $(\mathbf{x}_{\text{total}})_a$ 发送给**天线 a**。天线 a 作为一个主动的扬声器，将这个指令信号转换为电磁波发射出去。
    5.  **物理分离**：这些从服务集 $A_k$ 中所有天线发射出去的、经过精确调制的叠加信号，在**物理空间**中传播并干涉。由于波束赋形的设计，在每个目标用户 u 的位置，只有他自己的信号会发生**相长干涉**，而其他用户的信号则会发生**相消干涉**，从而实现了信号的物理分离。

#### **3. 信号处理核心：IS-MU-ZF 与波束赋形**

*   **波束赋形 (Beamforming)**：是实现上述过程的底层技术。
    *   **下行**：体现为**物理波束赋形 (Precoding)**，在空间中塑造能量分布。
    *   **上行**：体现为**数字波束赋形 (Combining/Spatial Filtering)**，在数字域通过数学运算分离信号。
*   **IS-MU-ZF 权重公式**：这是计算波束赋形权重的“剧本”。
    $$ \mathbf{w}_u = \left( \mathbf{D}_k \sum_{v \in (S_k \cup N_k)} \hat{\mathbf{h}}_v \hat{\mathbf{h}}_v^H \mathbf{D}_k \right)^\dagger \mathbf{D}_k \hat{\mathbf{h}}_u $$
    *   $\mathbf{D}_k$：天线选择矩阵，一个 $A \times A$ 的对角矩阵，其对角元素为1或0，用于在逻辑上只选择服务集 $A_k$ 内的天线。
    *   $\hat{\mathbf{h}}_v \in \mathbb{C}^{A \times 1}$：用户 v 的**估计信道向量**。
    *   $S_k \cup N_k$：需要被抑制的干扰用户集合，包括**簇内其他用户**和**部分主要的簇外用户**。
    *   **原理**：该公式通过矩阵伪逆 $(\cdot)^\dagger$ 运算，找到一个与所有指定干扰用户的信道向量都**正交**（即 $\mathbf{w}_u^H \hat{\mathbf{h}}_v = 0$ for $v \ne u$）的权重向量 $\mathbf{w}_u$，从而在数学上将干扰消除。

#### **4. 现实挑战：信道估计与导频污染**

*   **信道估计**：通过用户发送已知的**导频序列** $\boldsymbol{\phi}_u \in \mathbb{C}^{\tau \times 1}$ 来测量信道。
*   **导频污染 (Pilot Contamination)**：由于正交导频序列数量有限（最多 $\tau$ 个），不同用户必须复用导频。当多个用户使用相同导频时，基站无法区分他们的信道，估计出的信道是被污染的（所有同导频用户的信道之和）。
    $$ \hat{h}_{a,u} = h_{a,u} + \sum_{v \in \mathcal{P}_u, v \ne u} h_{a,v} + \frac{1}{\sqrt{\tau P_p}} \boldsymbol{\phi}_u^H \mathbf{n}_p $$
    其中 $\mathcal{P}_u$ 是与用户 u 使用相同导频的用户集合。
*   **解决方案**：本文采用**基于图着色的导频分配算法**，将用户簇视为图的顶点，强干扰关系视为边，正交导频组视为颜色。通过为相邻顶点涂上不同颜色，优先保证了地理邻近、干扰最强的簇之间使用正交导频，从而有效减轻了导频污染。

---

### **第二部分：核心问题与创新方法**

#### **1. 核心问题：ZF的容量骤降与噪声增强**

*   **现象**：传统的IS-MU-ZF在特定条件下，用户容量会急剧下降。
*   **根本原因：噪声增强 (Noise Enhancement)**。
*   **触发条件**（满足其一即可）：
    1.  **自由度耗尽**：当需要抑制的干扰用户数 $|N_k|$ 接近系统的剩余天线自由度 $R_k = A_k - |S_k|$ 时，ZF权重计算中的矩阵求逆会变得**病态（ill-conditioned）**，对信道估计误差等微小扰动极其敏感。
    2.  **信道几何恶劣**：当某个干扰用户的信道方向与目标用户的信道方向**非常接近**（高度相关）时，ZF为了维持正交性，被迫选择一个对目标信号接收效果极差的方向。
*   **后果**：在上述任一情况下，ZF算法为了补偿信号损失，会计算出一个**范数（长度）巨大**的权重向量 $||\mathbf{w}_u||$。这个巨大的范数在合并信号时，会不成比例地**放大背景噪声**（噪声功率为 $\sigma_n^2 ||\mathbf{w}_u||^2$），导致信干噪比（SINR）雪崩。

#### **2. 关键洞察：容量与干扰数呈“U型”关系**

*   通过仿真发现，用户容量与被抑制的干扰用户数 $|N_k|$ 的关系并非单调。
    *   **少量干扰 ($|N_k| < R_k$)**：容量随 $|N_k|$ 增加而**下降**。
    *   **极限点 ($|N_k| \approx R_k$)**：容量达到**谷底**，噪声增强最严重。
    *   **大量干扰 ($|N_k| \gg R_k$)**：由于**信道硬化（Channel Hardening）**效应，协方差矩阵 $\sum \mathbf{h}_v \mathbf{h}_v^H$ 趋近于单位矩阵 $\mathbf{I}$，ZF算法行为近似于简单的匹配滤波，噪声增强效应减弱，容量反而**回升**。

#### **3. 创新方法：自适应干扰用户选择**

基于上述洞察，本文提出了一种“趋利避害”的自适应算法。
*   **核心思想**：不再盲目抑制所有潜在干扰，而是根据当前系统状态，动态选择一个**最优数量** $M_k$ 的最强干扰源进行抑制。
*   **算法**：通过一个分段函数实现“分区施策”。
    $$ M_k = \begin{cases} \min(\lceil b_1 R_k \rceil, |N_k|), & \text{if } |N_k| \le b_2 R_k \\ |N_k|, & \text{if } b_2 R_k < |N_k| < b_3 R_k \\ \lceil b_3 R_k \rceil, & \text{if } b_3 R_k \le |N_k| \end{cases} $$
    *   **当 $|N_k|$ 较小（高速段）**：**保守抑制**，设置上限，主动远离“性能陷阱”。
    *   **当 $|N_k|$ 巨大（国道段）**：**限制上限**，在保证性能的同时，控制不必要的计算开销。
    *   **在过渡区域**：**保持默认**，随机应变。
*   **复杂度**：该算法增加的计算量主要是排序的 $O(|N_k| \log |N_k|)$，与ZF本身的 $O(A_k^3)$ 相比，**可以忽略不计**。

---

### **第三部分：性能评估与结论**

*   **有效性**：仿真证明，自适应IS-MU-ZF相比传统方法有巨大性能提升，性能曲线非常接近更复杂的IS-MU-MMSE。
*   **优越性**：与业界流行的UC架构（使用IS-MU-MMSE）相比，本文提出的CC架构（使用自适应IS-MU-ZF）在**用户容量、总容量和计算复杂度**三个方面展现了综合优势。
*   **结论**：本文提出的自适应算法成功解决了ZF的性能瓶颈，并证明了CC架构是一种极具潜力的下一代无线通信系统方案。

---
### **附录：参考文献解析**

本文的研究建立在大量经典和前沿工作的基础之上。这些参考文献可以大致分为两类：一类是直接涉及**波束赋形方法及其理论**的论文，另一类则是为本文提供**系统架构、核心概念和支持算法**的论文。

#### **一、 与波束赋形方法直接相关的论文**

这些文献是理解本文信号处理核心（ZF/MMSE权重计算、噪声增强等）的理论基石。

*   [5] Goldsmith, Andrea. (2005). *Wireless Communications*. Cambridge, United Kingdom: Cambridge University Press.
    *   **作用**：作为通信领域的权威**教科书**，它为本文提供了**迫零**（Zero-Forcing, ZF）和**最小均方误差**（Minimum Mean Square Error, MMSE）等基础线性波束赋形（或合并）算法的原始定义和数学推导。

*   [12] Peel, Chris B., Hochwald, Bertrand M., & Swindlehurst, A. Lee. (2005). A vector-perturbation technique for near-capacity multiantenna multiuser communication-Part I: Channel inversion and regularization. *IEEE Transactions on Communications*, 53(1), 195-202.
    *   **作用**：这篇论文是解释本文核心问题——**噪声增强**——的**关键理论依据**。它深入分析了“信道求逆”（ZF的本质）和“正则化”（MMSE的本质），从数学上阐明了为什么在特定条件下（如信道矩阵病态），简单的ZF波束赋形会导致噪声被急剧放大。

*   [3] Björnson, Emil, & Sanguinetti, Luca. (2020). Scalable cell-free massive MIMO systems. *IEEE Transactions on Communications*, 68(7), 4247-4261.
    *   **作用**：这篇论文提出了可扩展的用户中心（UC）架构，并详细阐述了在该架构下应用的**部分MMSE（Partial MMSE）**等先进波束赋形方法，是本文进行性能对比的**主要“标杆”**。

#### **二、 提供系统架构与支持算法的论文**

这些文献共同构建了本文研究的框架，并为具体实现提供了必要的算法工具。

*   [1] Zhang, Jiayi, Chen, Sheng, Lin, Ying-Chang, Zheng, Jing, Ai, Bo, & Hanzo, Lajos. (2019). Cell-free massive MIMO: A new next-generation paradigm. *IEEE Access*, 7, 99878-99888.
    *   **作用**：提供了“**无蜂窝大规模MIMO**”这一宏观概念的背景综述，确立了本文研究的领域和重要性。

*   [2] Chen, Zhaorui, & Björnson, Emil. (2018). Channel hardening and favorable propagation in cell-free massive MIMO with stochastic geometry. *IEEE Transactions on Communications*, 66(11), 5205-5219.
    *   **作用**：深入分析了无蜂窝系统中的“**信道硬化**”物理现象，这为理解本文中“大量干扰下ZF性能回升”的“U型曲线”行为提供了理论解释。

*   [4] Buzzi, Stefano, & D'Andrea, Carmen. (2017). Cell-free massive MIMO: User-centric approach. *IEEE Wireless Communications Letters*, 6(6), 706-709.
    *   **作用**：作为用户中心（UC）架构的早期关键论文，与[3]共同构成了本文进行架构对比的背景。

*   [6] Xia, Sijie, Ge, Chang, Takahashi, Ryo, Chen, Qiang, & Adachi, Fumiyuki. (2022, October). A study on cluster-centric cell-free massive MIMO system. In *Proceedings of the 27th Asia Pacific Conference on Communications (APCC)* (pp. 247-252).
*   [7] Takahashi, Ryo, Matsuo, Hiroki, Xia, Sijie, Chen, Qiang, & Adachi, Fumiyuki. (2022, September). Evaluation of uplink capacity of user-cluster-centric cell-free massive MIMO. In *Proceedings of the IEEE 96th Vehicular Technology Conference (VTC2022-Fall)* (pp. 1-5).
    *   **作用**：这两篇是**作者团队的前期工作**，本文提出的CC系统框架，包括具体的聚类和天线关联方法，都是在这两篇论文中首次建立的。

*   [8] Hao, Yijie, Xin, Jialin, Tao, Wei, Tao, Si, Yu-xiang, Li, & Hao, Wen. (2020, December). Pilot allocation algorithm based on K-means clustering in cell-free massive MIMO systems. In *Proceedings of the IEEE 6th International Conference on Computer and Communications (ICCC)* (pp. 608-611).
    *   **作用**：提供了将K-means聚类思想应用于**导频分配**的先例，为本文的导频管理方案提供了思路参考。

*   [9] Riera-Palou, Felip, Femenias, Guillem, Armada, Ana Garcia, & Pérez-Neira, Ana. (2018, December). Clustered cell-free massive MIMO. In *Proceedings of the IEEE Globecom Workshops (GC Wkshps)* (pp. 1-6).
    *   **作用**：提出了“**簇中心**（Clustered Cell-Free）”这一通用概念，是本文所研究架构的早期思想源头。

*   [10] Bradley, Paul S., Bennett, Kristin P., & Demiriz, Ayhan. (2000). Constrained K-means clustering. *Microsoft Research*.
    *   **作用**：提供了本文**用户聚类**步骤所使用的核心算法——**受约束K-means算法**。

*   [11] Welsh, Dominic J. A., & Powell, Michael B. (1967). An upper bound for the chromatic number of a graph and its application to timetabling problems. *The Computer Journal*, 10(1), 85-86.
    *   **作用**：作为图论领域的经典论文，为本文的**导频分配**方案提供了核心的**图着色理论**基础。

*   [13] Xiang, Wen. (2011, November). Analysis of the time complexity of quick sort algorithm. In *Proceedings of the International Conference on Information Management, Innovation Management and Industrial Engineering* (Vol. 1, pp. 408-410).
    *   **作用**：为本文的**计算复杂度分析**提供了依据，用于证明自适应算法中排序步骤的开销可以忽略不计。

### **附录B：本文研究在通信系统中的定位**

为了全面理解本文的贡献和适用范围，有必要将其研究内容置于一个完整的现代无线通信系统（如5G）的框架中，并阐明其与OFDM、NOMA及信道编码等关键技术的关系。


#### **1. 系统在MIMO-OFDMA框架中的定位与工作流程**

本文研究的核心，是为现代无线通信的基石——**MIMO-OFDMA系统**，提供一个在**空间维度**上进行高效资源管理和信号处理的先进算法。为了理解本文的贡献，我们首先需要构建一个完整的MIMO-OFDMA系统的工作图景。

**1.1. 宏观框架：三个维度的资源管理**

一个现代无线通信系统（如5G NR）在三个基本维度上管理其宝贵的频谱资源：
*   **频率维度**：由**OFDMA**管理。
*   **时间维度**：由**TDMA**（时分多址）管理。
*   **空间维度**：由**MIMO/SDMA**（空分多址）管理。

**1.2. 系统工作流程详解**

**步骤一：频率维度处理 (OFDM/OFDMA - “修路与交通管制”)**

1.  **“修路” (OFDM - 调制)**：系统首先将一个宽阔的物理频谱（例如100MHz）通过**OFDM**技术，在数学上划分为数千个相互正交的、非常窄的**子载波**（例如，每个15kHz）。这个过程将一个复杂的、随频率变化的**频率选择性衰落信道**，转化成了数千个并行的、易于处理的**平坦衰落子信道**。这是后续所有处理的简化基础。

2.  **“交通管制” (OFDMA - 多址)**：位于基站的**调度器**（Scheduler）扮演着交通警察的角色。它会根据所有用户的实时需求、信道质量和业务类型，做出宏观的资源分配决策。
    *   **调度决策**：调度器决定，在接下来的一个时间片（TTI, Transmission Time Interval）内，将哪些子载波（通常打包成**资源块, Resource Blocks**）分配给哪些用户。
    *   **决策结果**：例如，调度器决定：“将1-100号子载波组成的资源块，分配给**用户组 {A, B, C}**，并在这个资源块上启用**多用户MIMO**（MU-MIMO）模式。”

**步骤二：空间维度处理 (MU-MIMO - “立体交通”，本文研究的核心)**

**在OFDMA完成频率资源分配之后，本文的研究内容正式登场。** 它的工作发生在**每一个**被分配用于MU-MIMO的资源块（及其包含的子载波）上，即相同的频率段。每个子载波执行一次（但根据稀疏性，有些不用重复计算）

1.  **信道估计**：
    *   用户组 {A, B, C} 在被分配的子载波上发送已知的**导频序列**。
    *   网络中的天线接收这些导频。处理单元通过**稀疏导频+插值**等高效方法，估计出每个用户在这一组子载波上的信道向量 $\hat{\mathbf{h}}_A, \hat{\mathbf{h}}_B, \hat{\mathbf{h}}_C$。
    *   为了对抗**导频污染**，系统采用了**基于图着色的导频分配**策略，确保强干扰用户之间使用正交导频。

2.  **用户组织与天线关联 (CC架构)**：
    *   处理单元根据用户的地理位置，将他们逻辑地划分到**簇**中。
    *   然后，根据每个用户的**大尺度衰落**（所有子载波上的平均），为每个簇动态地关联一个**天线服务集** $A_k$。这个服务集是地理分散的，从而提供了**宏分集**增益，保证了连接的可靠性。

3.  **波束赋形权重计算 (本文的创新点)**：
    *   **目标**：为用户组 {A, B, C} 中的每一个用户，计算出一个专属的波束赋形权重向量 $\mathbf{w}_A, \mathbf{w}_B, \mathbf{w}_C$。
    *   **传统方法 (IS-MU-ZF)**：使用迫零算法，找到与所有干扰用户信道都正交的权重。
        $$ \mathbf{w}_u = \left( \dots \sum_{v \in \text{Interferers}} \hat{\mathbf{h}}_v \hat{\mathbf{h}}_v^H \dots \right)^\dagger \dots \hat{\mathbf{h}}_u $$
    *   **问题**：当干扰用户数接近天线自由度，或信道高度相关时，ZF会导致灾难性的**噪声增强**。
    *   **本文的解决方案 (自适应IS-MU-ZF)**：在计算ZF权重**之前**，先运行一个**自适应干扰用户选择算法**。该算法基于对“U型”性能曲线的洞察，智能地从所有潜在干扰者中，挑选出一个**最优数量** $M_k$ 的最强干扰源。
        $$ M_k = f(|N_k|, R_k, b_1, b_2, b_3) $$
        这个预处理步骤，通过主动规避ZF的性能陷阱，极大地提升了波束赋形的效果。

4.  **数据传输 (上行/下行)**：
    *   计算出优化后的权重后，系统便按照我们之前详细讨论的**上行空间滤波**或**下行预编码**流程进行数据传输，从而在**同一个**子载波资源上，实现了对用户A, B, C的**并行服务**，即**多用户空分复用**。

**步骤三：时间维度处理 (TDMA)**

*   上述的OFDMA和MIMO处理过程，都在一个极短的时间单位（如1毫秒的子帧）内完成。
*   在下一个时间片，调度器会根据网络状态的变化，重新进行资源分配，服务另一批用户。这种在时间上的轮转，就是**时分多址**（TDMA）思想的体现。

**结论**：
本文的工作，是现代MIMO-OFDMA通信系统中一个**至关重要的中间环节**。它位于**OFDMA频域调度之后**，**实际数据传输之前**，专注于解决**空间域多用户复用**的核心难题——**如何设计出既能有效抑制干扰，又能避免自身算法缺陷（如噪声增强）的高性能波束赋形权重**。通过提出自适应干扰选择这一创新点，本文显著提升了空间维度资源利用的效率和鲁棒性。

#### **2. 与NOMA（非正交多址接入）的关系：不同哲学的多址方案**

本文采用的**多用户MIMO**（MU-MIMO），本质上是一种**空分多址**（Space Division Multiple Access, SDMA）技术，它与NOMA是两种不同技术哲学下的多址接入方案。

*   **本文技术** (SDMA)：其核心是**正交化**。通过波束赋形，在空间上为不同用户创建**近似正交**的“虚拟信道”，目标是在接收端**消除**（Eliminate）用户间干扰。这依赖于多天线提供的空间自由度。
*   **NOMA技术**：其核心是**非正交**与**功率域区分**。它**主动地**让多个用户在完全相同的资源（时间、频率、空间）上叠加传输，并依赖接收端强大的**连续干扰消除**（Successive Interference Cancellation, SIC）能力，根据用户信道增益的差异来分离信号。
*   **关系**：
    *   **竞争性**：在提升频谱效率的目标上，两者是可相互替代的技术路线。
    *   **互补性**：两者可以结合形成**MIMO-NOMA**混合系统。例如，可以先用本文的波束赋形技术将用户分成几个空间上可区分的组，然后在每个波束内部，再利用NOMA技术服务多个用户，从而进一步压榨系统容量。

本文的研究本身并未采用NOMA，而是专注于SDMA路线，但其构建的高性能无蜂窝平台为未来融合NOMA技术提供了坚实的基础。

#### **3. 与信道编码（Channel Coding）的关系：上下游的协同**

本文的研究与信道编码（如LDPC码、Polar码）是通信链路中紧密协作的“上下游”关系。

*   **本文的角色（物理层）：“信道净化器”**
    *   本文的所有技术，从宏分集到自适应波束赋形，其最终的物理目标都是**最大化接收端的信干噪比（Signal-to-Interference-plus-Noise Ratio, SINR）**。一个高的SINR意味着信号清晰，干扰和噪声被有效抑制。

*   **信道编码的角色（数据链路层）：“数据守护者”**
    *   信道编码通过增加冗余比特，为数据提供纠错能力，以对抗信道中残余的噪声和干扰。其纠错能力的强弱，直接取决于物理层所能提供的SINR水平。

*   **协同关系**：
    *   本文提出的自适应算法通过有效提升SINR，相当于为信道编码模块创造了一个**更“干净”、更容易工作的信道环境**。
    *   根据**香农信道容量定理** $C = B \log_2(1 + \text{SINR})$，更高的SINR意味着更高的信道容量 $C$，即理论上无差错传输的速率上限更高。
    *   因此，本文的工作通过优化物理层，直接为上层的信道编码技术**创造了更大的发挥空间**，使得系统能够采用更高效的编码方案（码率更高），从而实现更高的数据传输速率。

综上所述，本文的研究聚焦于**物理层的空间信号处理**，它在OFDM提供的频率资源单元上，通过先进的波束赋形技术实现高效的多用户空分多址，其最终成果（高SINR）是上层信道编码能够高效工作、实现可靠高速通信的**关键前提**。