

# Channel Estimation and Passive Beamforming for Intelligent Reflecting Surface: Discrete Phase Shift and Progressive Refinement

Changsheng You, Beixiong Zheng, and Rui Zhang

## 智能反射面中的信道估计与无源波束赋形：离散相移和渐进式优化

### **摘要 (Abstract)**

本文旨在解决智能反射面 (IRS) 在实际应用中面临的两大挑战：在**离散相移 (Discrete Phase Shift)** 约束下进行**信道估计 (Channel Estimation)** 和**无源波束赋形 (Passive Beamforming)**。论文提出了一种新颖的**分层渐进式 (Hierarchical and Progressive)** 框架。首先，设计了一种分层的训练反射矩阵，用于在有限的导频符号下，逐步地、精细化地估计IRS的级联信道。该设计将信道估计分解为**组级 (per-group)** 和**组内 (intra-group)** 两个层次，并通过**子组划分 (subgroup partition)** 策略在多个时间块上不断提升估计精度。其次，基于渐进式获得的、包含相关估计误差的信道状态信息 (CSI)，设计了相应的渐进式无源波束赋形算法，以最大化每个时间块的数据传输速率。仿真结果表明，该框架相比传统方案在性能上有显著提升。

---

### **1. 引言 (Introduction)**

*   **背景与动机**: 传统无线通信技术面临硬件成本与能耗瓶颈。IRS作为一种低成本、低功耗的无源技术，能智能重构无线环境，前景广阔。
*   **研究缺口 (Problem Statement)**:
    1.  **完美CSI假设**: 现有研究多假设CSI完美可知，但这在实践中难以实现，因为IRS是无源设备，无法主动收发信号。
    2.  **高昂的信道估计开销**: 传统的“一次性”估计所有 $N$ 个单元的级联信道，需要至少 $N$ 个导频符号，导致巨大时延。虽有分组方法减少开销，但会牺牲波束赋形性能。
    3.  **连续相移假设**: 多数工作假设相移连续可调，而实际硬件只能提供有限的离散相移值，使优化问题变为NP-hard。
*   **本文贡献**: 针对上述挑战，本文提出一个统一的渐进式框架，在离散相移和有限导频的实际约束下，联合设计信道估计和波束赋形。核心思想是在多个时间块上，逐步精化CSI并相应优化波束赋形，从而动态提升系统性能。

---

### **2. 系统模型 (System Model)**

#### **A. 信道模型 (Channel Model)**

*   **场景**: 一个单天线用户 (User)，一个单天线基站 (AP)，以及一个含 $N$ 个单元的IRS。用户到AP的直接链路被阻塞。
*   **IRS反射模型**: IRS的反射行为由对角矩阵 $\mathbf{\Omega} = \text{diag}(\theta_1, \dots, \theta_N)$ 描述，其中 $\theta_n = e^{j\omega_n}$ 是第 $n$ 个单元的反射系数。
*   **离散相移约束**: 相移 $\omega_n$ 只能从一个 $K=2^b$ 个值的离散集合 $\mathcal{F}$ 中选取。
*   **端到端等效信道**: 经过数学简化，从用户到AP的等效信道 $q$ 可以表示为：
    $$ q(\boldsymbol{\theta}) = \boldsymbol{\theta}^H \mathbf{h} $$
    其中 $\boldsymbol{\theta} = [\theta_1, \dots, \theta_N]^T$ 是IRS的反射向量，$\mathbf{h} \in \mathbb{C}^{N \times 1}$ 是需要被估计的级联信道向量。

#### **B. 渐进式框架概述 (Proposed Progressive Designs)**

整个系统在一个传输帧内，按**时间块 (time block)** $i=1, 2, \dots, I_o$ 循环工作。每个时间块包含**信道估计**和**数据传输**两个阶段。

*   **核心思想**: 将 $N$ 个IRS单元分为 $M$ 组，每组 $L$ 个单元 ($N=ML$) 。通过 $L$ 个时间块，逐步将每组内的信道从一个整体解析到每个独立的单元。
*   **分层训练反射向量**: 在时间块 $i$ 的第 $m$ 次测量中，总反射向量 $\boldsymbol{\theta}^{(i)}[m]$ 被设计为两个向量的克罗内克积：
    $$ (\boldsymbol{\theta}^{(i)}[m])^H = (\boldsymbol{\theta}_{s}[m])^H \otimes (\boldsymbol{\psi}_{a}^{(i)})^H $$
    *   $\boldsymbol{\theta}_{s}[m] \in \mathcal{F}^{M \times 1}$: **组级基础训练向量**，用于区分 $M$ 个不同的组。这套向量在所有时间块中**固定不变**。
    *   $\boldsymbol{\psi}_{a}^{(i)} \in \mathcal{F}^{L \times 1}$: **组内训练向量 (探针)**，用于探测组内信道细节。它在时间块 $i$ 内固定，但随 $i$ **演进**。

---

### **3. 渐进式信道估计 (Progressive Channel Estimation)**

#### **阶段一: 组级有效信道估计 (Per-group Estimation)**

*   **目标**: 在每个时间块 $i$ 内，利用 $M$ 个导频符号，估计出 $M$ 个组的**等效信道** $\mathbf{h}^{(i)} \in \mathbb{C}^{M \times 1}$。
*   **等效信道定义**: 第 $k$ 组的等效信道 $h_k^{(i)}$ 是其真实信道 $\mathbf{h}_k \in \mathbb{C}^{L \times 1}$ 在当前探针 $\boldsymbol{\psi}_{a}^{(i)}$ 下的“投影”：
    $$ h_k^{(i)} \triangleq (\boldsymbol{\psi}_{a}^{(i)})^H \mathbf{h}_k $$
*   **线性模型**: 经过 $M$ 次测量，AP接收到的信号向量 $\mathbf{y}^{(i)}$ 与未知的 $\mathbf{h}^{(i)}$ 满足线性关系：
    $$ \mathbf{y}^{(i)} = \mathbf{X}_t \mathbf{\Theta}_s \mathbf{h}^{(i)} + \mathbf{z}^{(i)} $$
    其中 $\mathbf{\Theta}_s \in \mathcal{F}^{M \times M}$ 是由 $\{\boldsymbol{\theta}_s[m]\}_{m=1}^M$ 构成的**基础训练矩阵**。
*   **求解**: 只要 $\mathbf{\Theta}_s$ 可逆，就可以通过最小二乘法解出 $\hat{\mathbf{h}}^{(i)}$。论文后续讨论了如何基于DFT/Hadamard矩阵设计一个“好”的 $\mathbf{\Theta}_s$。

#### **阶段二: 组内信道恢复 (Intra-group Recovery)**

*   **目标**: 跨越多个时间块，利用积累的测量数据 $\{\hat{\mathbf{h}}^{(1)}, \dots, \hat{\mathbf{h}}^{(i)}\}$，恢复出更精细的**子组聚合信道** $\hat{\mathbf{g}}^{(i)}$。
*   **子组划分**: 预先设定一个划分策略（如对称或非对称），在时间块 $i$，每个组被划分为 $i$ 个子组。
*   **线性模型**: 对于每个组 $m$，其历史测量数据向量 $\hat{\boldsymbol{\eta}}_m^{(i)} = [\hat{h}_m^{(1)}, \dots, \hat{h}_m^{(i)}]^T$ 与其内部 $i$ 个子组的聚合信道 $\mathbf{g}_m^{(i)}$ 满足线性关系：
    $$ \hat{\boldsymbol{\eta}}_m^{(i)} \approx \tilde{\mathbf{\Psi}}_a^{(i)} \mathbf{g}_m^{(i)} $$
    其中 $\tilde{\mathbf{\Psi}}_a^{(i)}$ 是一个 $i \times i$ 的矩阵，由探针序列 $\{\boldsymbol{\psi}_a^{(k)}\}_{k=1}^i$ 和子组划分策略共同决定。
*   **求解**: 通过求解上述 $M$ 个并行的 $i$ 维线性方程组，得到 $\hat{\mathbf{g}}_m^{(i)}$，拼接后得到总的 $\hat{\mathbf{g}}^{(i)} \in \mathbb{C}^{iM \times 1}$。

---

### **4. 渐进式无源波束赋形 (Progressive Passive Beamforming)**

在每个时间块 $i$ 的数据传输阶段，根据最新得到的信道地图 $\hat{\mathbf{g}}^{(i)}$ 进行优化。

*   **问题建模**: 目标是最大化考虑了信道估计误差的SINR。优化问题 (P2) 如下：
    $$ \begin{aligned} \max_{\boldsymbol{\phi}^{(i)}} & \quad \frac{(\boldsymbol{\phi}^{(i)})^H \hat{\mathbf{G}}^{(i)} \boldsymbol{\phi}^{(i)}}{(\boldsymbol{\phi}^{(i)})^H \mathbf{R}^{(i)} \boldsymbol{\phi}^{(i)} + \sigma^2/P} \\ \text{s.t.} & \quad |\phi_l^{(i)}| = 1, \quad \forall l \\ & \quad \angle \phi_l^{(i)} \in \mathcal{F}, \quad \forall l \end{aligned} $$
    *   $\hat{\mathbf{G}}^{(i)} = \hat{\mathbf{g}}^{(i)} (\hat{\mathbf{g}}^{(i)})^H$ 代表信号功率项。
    *   $\mathbf{R}^{(i)} = \mathbb{E}[\mathbf{g}_e^{(i)} (\mathbf{g}_e^{(i)})^H]$ 是误差协方差矩阵，代表干扰功率项。
    *   $\boldsymbol{\phi}^{(i)}$ 是维度为 $iM \times 1$ 的波束赋形向量。

*   **求解算法**: 由于问题是NP-hard的，采用“初始化+迭代优化”的框架。
    1.  **初始化方法**:
        *   **SDR (Semidefinite Relaxation)**: 通过将问题“提升”到矩阵空间并“松弛”掉棘手的秩一约束，将其转化为一个可以高效求解的**半定规划 (SDP)** 问题。然后通过**高斯随机化**从SDP的解中恢复出一个高质量的初始向量。
        *   **Replication-based**: 利用前一个时间块的解作为当前块的初始解，是一种低复杂度的自适应方法。
        *   **Channel-gain-maximization**: 一种更简单的贪心策略，仅最大化信号功率。
    2.  **迭代优化 (Successive Refinement)**: 采用**交替优化**思想，固定其他变量，依次对每一个子组的相移进行一维搜索，在离散集合 $\mathcal{F}$ 中找到最优值，循环迭代直至收敛。

---

### **附录 A: 主要参考文献及其作用**

#### **I. IRS基础概念与框架**

1.  **[2] Wu, Qingqing, and Rui Zhang. "Towards smart and reconfigurable environment: Intelligent reflecting surface aided wireless network." *IEEE Communications Magazine* 60.1 (2020): 106-112.**
    *   **作用**: **奠基性的综述文章**。它全面介绍了IRS的概念、物理原理、潜在优势（如低功耗、易部署）和核心研究挑战（如信道估计、无源波束赋形），为本文的研究动机提供了宏观背景和技术蓝图。

2.  **[4] Basar, Ertugrul, et al. "Wireless communications through reconfigurable intelligent surfaces." *IEEE Access* 7 (2019): 116753-116773.**
    *   **作用**: **全面的教程性论文**。由多位该领域的顶尖学者撰写，系统地介绍了RIS的信号模型、信道特性、性能分析方法和硬件约束，为后续研究者提供了坚实的理论基础和建模规范。

#### **II. IRS信道估计**

3.  **[29] Zheng, Beixiong, and Rui Zhang. "Intelligent reflecting surface-enhanced OFDM: Channel estimation and reflection optimization." *IEEE Wireless Communications Letters* 9.4 (2020): 518-522.**
    *   **作用**: **本文工作的直接前期基础**。它率先提出了通过**IRS单元分组 (grouping)** 来减少OFDM系统中信道估计导频开销的核心思想。本文的“渐进式”框架正是对该分组思想的深化和性能改进，旨在解决分组带来的波束赋形性能损失问题。

4.  **[28] He, Zhen-Qing, and Xiaodi Yuan. "Cascaded channel estimation for large intelligent metasurface assisted massive MIMO." *IEEE Wireless Communications Letters* 9.2 (2020): 210-214.**
    *   **作用**: **级联信道估计的代表性工作**。它专注于估计“用户-IRS-基站”这一级联信道，并提出了相应的估计算法框架。本文的信道估计部分也遵循这一主流的级联信道模型。

#### **III. IRS波束赋形**

5.  **[6] Wu, Qingqing, and Rui Zhang. "Intelligent reflecting surface enhanced wireless network via joint active and passive beamforming." *IEEE Transactions on Wireless Communications* 18.11 (2019): 5394-5409.**
    *   **作用**: **IRS波束赋形领域的开创性工作之一**。它建立了在**连续相移**和**完美CSI**假设下，联合优化基站主动波束赋形和IRS无源波束赋形的优化框架，其提出的交替优化算法被后续大量研究参考和扩展。

6.  **[11] Wu, Qingqing, and Rui Zhang. "Beamforming optimization for wireless network aided by intelligent reflecting surface with discrete phase shifts." *IEEE Transactions on Communications* 68.3 (2020): 1838-1851.**
    *   **作用**: **离散相移波束赋形的里程碑工作**。它首次系统地研究了实际硬件的**离散相移**约束，指出了其NP-hard特性，并提出了高效的次优算法（如基于SDR的松弛和基于交替优化的精化）。本文的波束赋形算法是在此基础上，进一步考虑了信道估计误差的影响。

#### **IV. 波束赋形优化方法论**

7.  **Luo, Zhi-Quan, et al. "Semidefinite relaxation of quadratic optimization problems." *IEEE Signal Processing Magazine* 27.3 (2010): 20-34.**
    *   **作用**: **SDR方法的经典教程**。这篇综述详细解释了半定松弛 (SDR) 技术的原理、应用和理论保证，特别是在解决二次约束二次规划 (QCQP) 问题中的应用。本文中使用的SDR方法正是基于此文所阐述的理论框架。

8.  **Gershman, Alex B., et al. "Convex optimization-based beamforming." *IEEE Signal Processing Magazine* 27.3 (2010): 35-49.**
    *   **作用**: **凸优化在波束赋形中应用的综述**。它系统地介绍了如何将各种传统的波束赋形问题（如最小化输出功率、最大化信噪比）建模为凸优化问题（如二阶锥规划 SOCP 或半定规划 SDP），并进行高效求解。为本文将波束赋形问题转化为SDP提供了方法论背景。

9.  **[35] Charnes, Abraham, and William W. Cooper. "Programming with linear fractional functionals." *Naval Research Logistics Quarterly* 9.3-4 (1962): 181-186.**
    *   **作用**: **分式规划的经典文献**。提供了经典的**Charnes-Cooper变换**，这是一种将分式规划问题（目标函数为分式形式）转化为等价的线性规划问题的数学工具。本文在SDR方法中用它来处理分式形式的SINR目标函数，是问题能够成功转化为标准SDP的关键一步。

---

### **附录 B: 本文与OFDMA-MIMO系统及波束赋形的关系**

本文研究的是一个基础的单用户单天线 (SISO) 系统，但这套“渐进式”框架的思想可以自然地扩展到更复杂的**OFDMA-MIMO**系统中，并与传统的波束赋形概念产生深刻的联系。

*   **与OFDMA的关系**:
    *   在**OFDMA (Orthogonal Frequency Division Multiple Access)** 系统中，信道在频域上是变化的，每个子载波都有不同的信道响应。本文的框架可以直接应用于每个子载波上。
    *   更高效的做法是利用信道的频率相关性。可以只在部分导频子载波上执行本文的渐进式估计，然后通过插值得到其他子载波的信道。
    *   对于多用户OFDMA，基站可以为不同用户分配不同的子载波资源。本文的框架可以并行地应用于服务不同用户的子载波组上，实现多用户间的资源调度和干扰协调。

*   **与MIMO的关系**:
    *   当基站和/或用户配备**多根天线 (MIMO)** 时，信道不再是向量，而是**矩阵**。例如，对于一个 $M_{AP}$-天线的基站，级联信道 $\mathbf{h}$ 会变成一个 $N \times M_{AP}$ 的矩阵 $\mathbf{H}$。
    *   本文的信道估计算法可以**并行地**应用于估计矩阵 $\mathbf{H}$ 的每一列。即，基站的每根天线都可以独立地执行本文提出的渐进式估计流程。
    *   在数据传输阶段，问题就变成了联合设计基站的**主动数字波束赋形**（在基带处理）和IRS的**无源模拟波束赋形**（调整反射相位）。这是一个更复杂的联合优化问题，但本文提出的渐进式思想依然适用：随着IRS信道矩阵 $\mathbf{H}$ 的估计越来越精细，基站的主动波束赋形和IRS的无源波束赋形都可以进行更精细的联合优化，从而获得更高的空间复用和波束赋形增益。

*   **与传统波束赋形的关系**:
    *   **传统波束赋形**（如在基站端）是**主动的**，通过消耗能量的射频链路对信号进行加权处理，以在特定方向上形成高增益波束。
    *   **IRS波束赋形**是**无源的**，它不产生新信号，而是通过改变信道本身，将入射信号能量反射到期望的方向。它更像是在“重塑传播环境”，而不是“产生定向信号”。
    *   在MIMO-IRS系统中，这两种波束赋形需要**协同工作**。基站的主动波束赋形负责将信号能量有效地“馈送”给IRS，而IRS的无源波束赋形则负责将这些能量高效地“转发”给用户。本文提出的渐进式框架，为这种复杂的协同优化提供了动态演进的CSI基础，使得联合波束赋形可以从一个粗略的初始配置，逐步自适应地调整到更精细、性能更优的状态。