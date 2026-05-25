# Beamforming Optimization for Intelligent Reflecting Surfaces without CSI

## 无需CSI的智能反射面波束赋形优化

**by V. D. P. Souto, R. D. Souza, B. F. Uchôa-Filho, A. Li, and Y. Li**

---

### **第一部分：摘要 (Abstract)**

本文研究了一个由多天线基站（BS）、单个智能反射面（IRS）和单天线用户（UE）组成的下行通信系统。核心挑战在于，获取IRS相关的信道状态信息（CSI）需要巨大的导频开销和复杂的硬件，这在实际中难以实现。为解决此问题，本文旨在**不依赖CSI**的情况下，联合优化BS的主动波束赋形和IRS的被动波束赋形。优化目标是在满足用户最低信噪比（SNR）需求的前提下，**最小化基站的发射功率**。作者提出了一种基于**粒子群优化（PSO）**的全新黑盒优化方法。仿真结果表明，该无CSI方案的性能**非常接近**于拥有完美CSI的理论最优方案，且在开销方面更具优势。

---

### **第二部分：详细内容解析**

#### **I. 引言 (Introduction)**

1.  **背景与动机**: 未来无线网络（5G及以后）对容量和连接性有极高要求。现有技术如**毫米波**（mmWave）、**大规模MIMO**（Massive MIMO）和**超密集网络**（UDN）存在成本高、能耗大的问题。**智能反射面（IRS）**作为一种**新兴的、高能效、低成本**的技术应运而生，它通过无源反射单元智能调控信号相位，使信号在UE处相干叠加，从而增强网络性能。
    *   **深度讨论：为何IRS高能效、低成本？** IRS的核心元件是**无源的（passive）**，不包含昂贵且高功耗的射频（RF）链（如功率放大器、数模转换器），仅通过低功耗电路（如PIN二极管）调整相位。这与需要完整收发链路的有源中继（Relay）形成鲜明对比，从而实现了根本性的成本与能耗降低。
  
2.  **相关工作与研究空白**:
    *   已有大量研究（[4]–[9]）探讨了IRS系统的波束赋形优化，目标涵盖了能量效率最大化、速率最大化和功率最小化等，并已扩展到毫米波和离散相位等实际场景。
    *   然而，这些工作都依赖于一个共同的、不切实际的假设：**拥有完美的CSI**。
        *   **深度讨论：参考文献[4]-[9]的逻辑作用？** 作者通过引用这组文献，构建了一个“State-of-the-Art”的图景：首先通过[4][5][6]展示了**IRS波束赋形的核心优化目标**（能效/速率最大化、功率最小化）；然后通过[7]扩展到**前沿应用**（毫米波）；最后通过[8][9]展示了**向实际约束（离散相位）的演进**。这一系列铺垫的最终目的，是引出所有这些工作的共同“软肋”——**对完美CSI的依赖**。

3.  **获取CSI的现实挑战**:
    *   **核心难题**: IRS的**无源特性**使其**无法主动收发导频**，导致级联信道中BS-IRS信道 $\mathbf{G}$ 和IRS-UE信道 $\mathbf{h}_{\mathrm{r}}$ **耦合**在一起，难以分离。
        *   **深度讨论：为何无源是核心难题？** 传统信道估计依赖于链路两端的设备都能主动“说”（发送导频）和“听”（接收并处理导频）。IRS既不能“说”也无法“听”，打破了传统“分段击破”式的信道估计模式。我们只能在UE端观测到所有路径信号的**最终叠加结果**，从中反解出巨量的、纠缠在一起的信道参数极其困难。
    *   **现有CSI估计方案的四大弊端**:
        i.  **需要射频链 [10], [11]**: 部分方案要求在IRS上配备有源射频链，违背了其无源、低成本的初衷。
        ii. **需要振幅控制 [12], [13]**: 部分方案要求IRS能独立控制振幅和相位，增加了硬件复杂性。
        iii. **信道稀疏性假设 [14], [15]**: 高效的压缩感知类算法依赖于信道稀疏性，但这仅在毫米波或LoS主导的场景下成立，通用性差。
        iv. **巨大的训练开销 [10]–[13], [16]–[18]**: 估计完整的级联信道（参数数量级为 $M \times N$）所需的导频开销巨大，在信道时变的场景下不切实际。

4.  **本文贡献**:
    *   提出一种**完全无需CSI**的BS和IRS联合波束赋形优化方法。
    *   该方法基于**粒子群优化（PSO）**，将无线信道视为黑盒，通过UE的SNR反馈进行迭代搜索。
    *   证明了该方法能够在**不牺牲性能**（接近最优）的情况下，**避免CSI估计**，从而降低系统成本、能耗和信令开销。

#### **II. 系统模型 (System Model)**

1.  **系统组成**: 一个下行链路系统，包含：
    *   一个配备 $N$ 根天线的均匀线性阵列（ULA）的BS。
    *   一个配备 $M$ 个反射单元的均匀平面阵列（UPA）的IRS。
    *   一个单天线UE。

2.  **信号模型**: UE接收到的信号 $y$ 是直连路径和反射路径信号的叠加：
    $$ y = (\mathbf{h}_{\mathrm{r}}^H \mathbf{\Theta} \mathbf{G} + \mathbf{h}_{\mathrm{d}}^H) \mathbf{w} s + n $$
    *   $\mathbf{w} \in \mathbb{C}^{N \times 1}$: BS的主动波束赋形向量。
    *   $\mathbf{G} \in \mathbb{C}^{M \times N}$: BS到IRS的信道矩阵。
    *   $\mathbf{h}_{\mathrm{r}} \in \mathbb{C}^{M \times 1}$: IRS到UE的信道向量。
    *   $\mathbf{h}_{\mathrm{d}} \in \mathbb{C}^{N \times 1}$: BS到UE的直连信道向量。
    *   $\mathbf{\Theta} = \mathrm{diag}(e^{j\theta_1}, \ldots, e^{j\theta_M})$: IRS的被动波束赋形（反射相位）矩阵，由相位向量 $\boldsymbol{\theta} = [\theta_1, \ldots, \theta_M]^T$ 决定。
    *   $s$: 发送符号，$\mathbb{E}[|s|^2] = 1$。
    *   $n$: 噪声，$n \sim \mathcal{CN}(0, \sigma^2)$。
        *   **深度讨论：为何信道项相加？** 这是电磁波在接收天线处**物理叠加**（superposition）的直接数学体现。来自直连路径和反射路径的信号波在空间中的同一点（UE天线）相遇，其复振幅在此处进行向量求和。
        *   **深度讨论：级联信道矩阵乘法解析**: 乘法链 $\mathbf{h}_{\mathrm{r}}^H \mathbf{\Theta} \mathbf{G} \mathbf{w}$ 完美模拟了信号的物理旅程：
            1.  $\mathbf{Gw}$: 经BS波束赋形后的信号到达IRS的 $M$ 个单元。
            2.  $\mathbf{\Theta}(\mathbf{Gw})$: $M$ 路信号在IRS处被独立调相。
            3.  $\mathbf{h}_{\mathrm{r}}^H(\mathbf{\Theta G w})$: $M$ 路调相后的信号经各自路径传播并最终在UE天线处叠加成一个标量信号。
   
3.  **信道模型**: 采用包含路径损耗的莱斯衰落模型。
    $$ \mathbf{G} = \sqrt{PL(d)} \left( \sqrt{\frac{\beta}{1+\beta}} \mathbf{G}^{\mathrm{LoS}} + \sqrt{\frac{1}{1+\beta}} \mathbf{G}^{\mathrm{NLoS}} \right) $$
    *   **深度讨论：是否忽略频率选择性衰落？** **是的。** 本文采用的是**窄带/平坦衰落信道模型**，其中信道响应不随频率变化。这是为了**简化问题**、突出核心贡献（无CSI优化）的常见做法。若要考虑频率选择性衰落，需引入**宽带OFDM模型**，届时每个子载波都将有自己的信道矩阵，而IRS的相位 $\mathbf{\Theta}$ 通常对所有子载波是相同的，这会引出**波束斜视**（beam squint）等更复杂的问题。

4.  **优化问题**:
    *   **性能指标**: 信噪比 SNR (Signal-to-Noise Ratio)。
        $$ \mathrm{SNR} = \frac{|(\mathbf{h}_{\mathrm{r}}^H \mathbf{\Theta} \mathbf{G} + \mathbf{h}_{\mathrm{d}}^H) \mathbf{w}|^2}{\sigma^2} $$
        *   **深度讨论：为何是SNR而非SINR？** 本文研究的是一个**单用户、单小区**的理想化模型，其中**不存在任何干扰源**（如其他用户或其他小区的信号）。因此，分母中只有噪声功率，指标为SNR。在多用户或多小区场景下，该指标会变为SINR（信干噪比），分母中会增加干扰功率项。

    *   **目标**: 最小化发射功率 $P_t = ||\mathbf{w}||^2$，约束条件为SNR不低于门槛 $\gamma$。
$$
\begin{aligned}
\min_{\mathbf{w}, \boldsymbol{\theta}} \quad & ||\mathbf{w}||^2 \\
\mathrm{s.t.} \quad & |(\mathbf{h}_{\mathrm{r}}^H \mathbf{\Theta} \mathbf{G} + \mathbf{h}_{\mathrm{d}}^H) \mathbf{w}|^2 \ge \gamma \sigma^2 \\
& 0 \le \theta_m \le 2\pi, \quad \forall m
\end{aligned}
$$
        
        *   **深度讨论：若目标是最大化SNR会怎样？** 优化问题会变为在最大发射功率 $P_{\max}$ 约束下最大化SNR。这两个问题（功率最小化 vs. SNR最大化）互为对偶，但求解策略不同。SNR最大化问题的核心在于找到最佳的波束方向 $\bar{\mathbf{w}}$ ($||\bar{\mathbf{w}}||^2=1$)，并用尽功率预算 $P_{\max}$。

*   **问题性质**: 这是一个**非凸 (non-convex)** 优化问题。
    *   **深度讨论：非凸性的根源？** 尽管目标函数 $||\mathbf{w}||^2$ 是凸的，但SNR约束是**非凸的**。根本原因在于优化变量 $\mathbf{w}$ 和 $\boldsymbol{\theta}$ 在SNR约束表达式中以**乘法形式耦合**。此外，形如 $f(\mathbf{x}) \ge C$ (其中 $f(\mathbf{x})$ 是凸函数) 的约束本身定义的区域也是非凸的。

#### **III. 基于PSO的解决方法 (Proposed Method based on PSO)**

1.  **核心思想**: 采用PSO算法将整个无线信道视为一个黑盒。BS生成一系列候选解（粒子），在真实环境中测试，并根据UE反馈的SNR值来迭代地寻找最优解。

2.  **粒子定义**: 一个粒子 $i$ 的**位置向量 $\mathbf{x}_i$** 代表一个完整的波束赋形方案，即 $(\mathbf{w}_i, \boldsymbol{\theta}_i)$。在实现中，它是一个 $(2N+M)$ 维的实数向量，粒子的**速度向量 $\mathbf{v}_i$** 维度相同。
    *   **深度讨论：粒子位置向量的具体构成？** 一个粒子的位置向量 $\mathbf{x}_i$ 是一个 $(2N+M)$ 维的**实数长向量**，由 $\mathbf{w}_i$ 的 $N$ 个实部/虚部和 $\boldsymbol{\theta}_i$ 的 $M$ 个相位**拼接**而成。PSO的更新是对这个长向量进行**统一的向量运算**，而非分步、独立地更新各个部分。采用实部/虚部表示法比幅/相表示法在实现上更简单，因为它避免了相位的周期性和幅度的非负性等非线性问题。

3.  **改进的PSO更新机制**: 为平衡**探索（Exploration）**与**利用（Exploitation）**，避免早熟收敛，作者提出了一种混合更新策略：
    *   **前 L/2 的粒子 (利用者)**: 按照经典PSO规则更新速度和位置，负责在当前最优解附近进行精细搜索。
        $$ \mathbf{v}_i[t+1] = \omega \mathbf{v}_i[t] + c_1 r_1 (\mathbf{pbest}_i - \mathbf{x}_i[t]) + c_2 r_2 (\mathbf{gbest} - \mathbf{x}_i[t]) $$
        $$ \mathbf{x}_i[t+1] = \mathbf{x}_i[t] + \mathbf{v}_i[t+1] $$
    *   **后 L/2 的粒子 (探索者)**:
        *   以概率 $p_{\mathrm{mut1}}$: 基于全局最优解 $\mathbf{gbest}$ 进行**变异**。
        *   以概率 $1-p_{\mathrm{mut1}}$: 在整个搜索空间内**完全随机生成**新粒子。
        这部分粒子负责维持种群多样性，探索新的可能区域。
    *   **深度讨论：为何采用混合更新策略？** 这是为了在**探索**（Exploration）和**利用**（Exploitation）之间取得平衡。
        *   **利用**: 前 $L/2$ 个按经典规则更新的粒子，负责在当前最优解附近进行精细搜索，以**加快收敛**。
        *   **探索**: 后 $L/2$ 个变异/随机生成的粒子，负责跳出当前搜索区域，以**避免陷入局部最优**。
    *   **深度讨论：另一种可能的混合策略？** 比如可以让所有粒子先按经典PSO规则更新，再随机选取一半进行二次扰动。这种“串行增强”策略可能会更快收敛，但其探索能力和鲁棒性可能与原文的“并行分工”策略有所不同，具体优劣需实验验证。
  
#### **IV. 仿真结果 (Simulation Results)**

1.  **收敛性 (Fig. 3)**: 算法收敛速度快（约100次迭代），且收敛所需的迭代次数不随IRS单元数 $M$ 的增加而显著增加。

2.  **性能对比 (Fig. 4 & 5)**:
    *   **接近最优**: 所提PSO方法的性能曲线**非常接近**拥有完美CSI的理论下界，远优于无IRS或随机相位方案。
    *   **平方增益**: 在IRS起主导作用的场景（UE靠近IRS），发射功率随 $M$ 的增加呈平方级下降（$P_t \propto 1/M^2$），验证了IRS的巨大潜力。

3.  **开销分析 (Fig. 6)**:
    *   **反馈开销 vs. 导频开销**:
        *   **导频开销**: 传统方法为估计CSI所需的通信资源，与 $M \times N$ 成正比。
        *   **反馈开销**: 本文方法所需的总反馈次数，等于 $N_{it} \times L$。
    *   **关键结论**: 在大规模系统中（$M, N$ 较大），本文方法可以用**远少于**传统CSI估计的开销（例如只需50%-70%），达到几乎相同的性能。这证明了本文方法在**效率**上的巨大优势。
        *   **深度讨论：“反馈开销”的通俗理解？** 把它想象成一个蒙着眼睛的音响师调音。**导频开销**是为了绘制一幅完整的“旋钮-效果”地图（估计CSI）而进行的系统性测试的总次数。**反馈开销**是本文方法中，为了通过“智能试错”找到最佳设置，而向听众（UE）询问“现在效果如何？”的总次数。本文证明，“智能试错”所需的询问次数远少于“绘制地图”。

#### **V. 结论 (Conclusion)**

*   **总结**: 本文成功提出并验证了一种无需CSI的IRS波束赋形优化方法。该方法性能接近最优，且反馈开销合理，有效解决了传统方案面临的CSI获取难题，为IRS的实际部署提供了低成本、低能耗的可行路径。
*   **未来工作**:
    1.  研究**离散相位**约束的影响。
    2.  将**振幅控制**纳入联合优化。
    3.  利用**统计CSI**来辅助和加速搜索过程，进一步降低开销。

---

### **第三部分：附录**

#### **附录A：主要参考文献解析**

*   **[1] H. Tullberg et al., "The METIS 5G system concept: Meeting the 5G requirements," *IEEE Commun. Mag.*, 2016.**
    *   **作用**: 支撑引言，阐述5G及未来网络的需求和概念，为引入新技术提供宏观背景。
*   **[3] Q. Wu and R. Zhang, "Towards smart and reconfigurable environment: Intelligent reflecting surface aided wireless network," *IEEE Commun. Mag.*, 2020.**
    *   **作用**: 提供了IRS作为一个可重构智能环境的核心概念和愿景，是领域内的综述性强文。
*   **[6] Q. Wu and R. Zhang, "Intelligent reflecting surface enhanced wireless network via joint active and passive beamforming," *IEEE Trans. Wireless Commun.*, 2019.**
    *   **作用**: **（核心基准）** 提供了本文旨在解决的核心优化问题模型，并作为拥有完美CSI的性能下界（Lower Bound）基准。
*   **[8] Q. Wu and R. Zhang, "Beamforming optimization for wireless network aided by intelligent reflecting surface with discrete phase shifts," *IEEE Trans. Commun.*, 2020.**
    *   **作用**: 支撑引言中对“研究已扩展到实际约束”的论述，具体指出了离散相位这一重要研究方向。
*   **[10] A. Taha et al., "Enabling large intelligent surfaces with compressive sensing and deep learning," 2019.**
    *   **作用**: 作为引言中批判CSI估计方案的论据，证明了某些方案需要依赖于在IRS上安装**有源射频链**。
*   **[14] J. Chen et al., "Channel estimation for reconfigurable intelligent surface aided multi-user MIMO systems," 2019.**
    *   **作用**: 作为论据，证明了某些高效CSI估计算法依赖于**信道稀疏性**这一受限的假设。
*   **[17] B. Zheng and R. Zhang, "Intelligent reflecting surface-enhanced OFDM: Channel estimation and reflection optimization," *IEEE Wireless Commun. Lett.*, 2020.**
    *   **作用**: 作为论据，证明了即使是前沿的CSI估计算法，其**训练开销**依然是限制性能的关键瓶颈。
*   **[19] J. Kennedy and R. Eberhart, "Particle swarm optimization," *in Proc. Int. Conf. Neural Netw. (ICNN)*, 1995.**
    *   **作用**: **（算法基础）** 提供了本文所采用的PSO算法的原始出处和经典理论基础。
*   **[21], [22], [23] (Q. Wu, R. Zhang et al.)**
    *   **作用**: 在结论部分被引用，为未来工作的三个方向（离散相位、振幅控制、统计CSI）提供了权威的研究参考。

#### **附录B：本文技术与OFDMA-MIMO系统的关系**

1.  **MIMO 与 IRS 的关系**:
    *   **大规模MIMO** 和 **IRS** 都是实现高级波束赋形的两种不同技术路径。
    *   **大规模MIMO** 在**基站端**通过大量的**有源天线**实现**主动波束赋形 (Active Beamforming)**。它能自己产生、放大并精准地“塑造”发射出去的信号波束。
    *   **IRS** 在**信道中**通过大量的**无源单元**实现**被动波束赋形 (Passive Beamforming)**。它不产生新信号，而是像一面智能镜子一样，对经过它的信号波束进行“反射和重定向”。
    *   在本文所研究的系统中，这两种波束赋形是**协同工作**的：BS进行主动波束赋形将信号高效地发送到IRS，IRS再进行被动波束赋形将信号高效地反射给UE。

2.  **与 OFDMA-MIMO 系统的整合**:
    *   **OFDMA (Orthogonal Frequency-Division Multiple Access)** 是一种**多址接入和资源管理**技术。它将系统总带宽分割成许多正交的子载波，并将这些子载波资源（在时间和频率维度上）灵活地分配给不同的用户。OFDMA主要解决的是“**谁在什么时候，用哪部分频率资源进行通信**”的问题。
    *   一个完整的现代通信系统（如5G NR）是一个**OFDMA-MIMO系统**。其工作流程可以理解为：
        i.  **资源分配层 (OFDMA)**: 系统调度器决定，在当前时刻，将子载波集合 $K_A$ 分配给用户A，将子载波集合 $K_B$ 分配给用户B。
        ii. **物理传输层 (MIMO & IRS Beamforming)**:
            *   在分配给用户A的子载波 $K_A$ 上，BS利用其大规模MIMO天线阵列，形成一个指向用户A的**主动波束**。
            *   如果信道中部署了IRS，IRS会同时调整其相位，形成一个**被动波束**，将BS的信号进一步聚焦到用户A。
            *   这两者协同作用，为用户A在**空间维度**上创造了一个高质量的通信链路。
    *   **总结**: **OFDMA负责在时频域上“切分蛋糕”，而MIMO和IRS波束赋形则负责在空间域上将每一块“蛋糕”精准地“喂”给指定的用户。** 本文研究的技术，正是这个复杂系统中至关重要的“空间精准投喂”环节，并提出了一种更高效、更低成本的实现方法。