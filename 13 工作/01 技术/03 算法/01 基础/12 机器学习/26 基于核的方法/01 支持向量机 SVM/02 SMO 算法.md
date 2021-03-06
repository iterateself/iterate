


# SMO 算法

SMO Sequential Minimal Optimization

缘由：

- 对于 SVM 中，通过拉格朗日乘子法转化得到的对偶问题：

$$
\begin{aligned}
\max_{\boldsymbol{\alpha}} & \sum_{i=1}^m\alpha_i - \frac{1}{2}\sum_{i = 1}^m\sum_{j=1}^m\alpha_i \alpha_j y_iy_j\boldsymbol{x}_i^T\boldsymbol{x}_j \\
s.t. & \sum_{i=1}^m \alpha_i y_i =0 \\
& \alpha_i \geq 0 \quad i=1,2,\dots ,m
\end{aligned}\tag{6.11}
$$

- 如何进行求解呢？
  - 不难发现，这是一个二次规划问题，可使用通用的二次规划算法来求解。
  - 然而，该问题的规模正比于训练样本数，这会在实际任务中造成很大的开销。
  - 为了避开这个障碍，人们通过利用问题本身的特性，提出了很多高效算法。
    - SMO (Sequential Minimal Optimization)是其中一个著名的代表。


SMO 算法：

- 基本思路是先固定 $\alpha_i$ 之外的所有参数，然后求 $\alpha_i$ 上的极值。
- 由于存在约束 $\sum_{i=1}^{m} \alpha_{i} y_{i}=0$ ，若固定 $\alpha_i$ 之外的其他变量，则 $\alpha_i$ 可由其他变量导出。 于是，SMO 每次选择两个变量 $\alpha_i$ 和 $\alpha_j$ ，并固定其他参数。
- 这样，在参数初始化后，SMO不断执行如下两个步骤直至收敛：
    - 选取一对需更新的变量 $\alpha_i$ 和 $\alpha_j$ 。
    - 固定 $\alpha_i$ 和 $\alpha_j$ 以外的参数，求解上面的对偶问题，获得更新后的 $\alpha_i$ 和 $\alpha_j$。

变量选择：

- 注意到只需选取的 $\alpha_i$ 和 $\alpha_j$ 中有一个不满足 KKT 条件 $\left\{\begin{array}{l}{\alpha_{i} \geqslant 0 ;} \\ {y_{i} f\left(x_{i}\right)-1 \geqslant 0} \\ {\alpha_{i}\left(y_{i} f\left(x_{i}\right)-1\right)=0 }\end{array}\right.$，目标函数就会在迭代后减小。
  - 直观来看，KKT条件违背的程度越大，则变量更新后可能导致的目标函数值减幅越大。
  - 于是，SMO 先选取违背 KKT 条件程度最大的变量。
- 第二个变量应选择一个使目标函数值减小最快的变量，但由于比较各变量所对应的目标函数值减幅的复杂度过高，因此 SMO 采用了一个启发式：
  - 使选取的两变量所对应样本之间的间隔最大。
    - 一种直观的解释是，这样的两个变量有很大的差别，与对两个相似的变量进行更新相比，对它们进行更新会带给目标函数值更大的变化。

固定其他参数后的优化过程：

- SMO算法之所以高效，恰由于在固定其他参数后，仅优化两个参数的过程能做到非常高效。
- 具体来说，仅考虑 $\alpha_i$ 和 $\alpha_j$ 时，式(6.11)中的约束可重写为：

$$
\alpha_{i} y_{i}+\alpha_{j} y_{j}=c, \quad \alpha_{i} \geqslant 0, \quad \alpha_{j} \geqslant 0\tag{6.14}
$$

- 其中:$c=-\sum_{k \neq i, j} \alpha_{k} y_{k}$ 是使 $\sum_{i=1}^{m} \alpha_{i} y_{i}=0$ 成立的常数。
- 将 $\alpha_{i} y_{i}+\alpha_{j} y_{j}=c$ 带入 $\max_{\boldsymbol{\alpha}} \sum_{i=1}^m\alpha_i - \frac{1}{2}\sum_{i = 1}^m\sum_{j=1}^m\alpha_i \alpha_j y_iy_j\boldsymbol{x}_i^T\boldsymbol{x}_j$ 来消去 $\alpha_j$。
- 则得到一个关于 $\alpha_i$ 的单变量二次规划问题，仅有的约束是  $\alpha_i\geq 0$ 。不难发现，这样的二次规划问题具有闭式解，于是不必调用数值优化算法即可高效地计算出更新后的 $\alpha_i$ 和 $\alpha_j$ 。（补充）
- 如何确定偏移项 $b$ 呢？注意到对任意支持向量 $\left(\boldsymbol{x}_{s}, y_{s}\right)$ 都有 $y_{s} f\left(\boldsymbol{x}_{s}\right)=1$ ，即：

$$
y_{s}\left(\sum_{i \in S} \alpha_{i} y_{i} \boldsymbol{x}_{i}^{\mathrm{T}} \boldsymbol{x}_{s}+b\right)=1\tag{6.17}
$$

- 其中 $S=\left\{i | \alpha_{i}>0, i=1,2, \ldots, m\right\}$ 为所有支持向量的下标集。
- 理论上，可选取任意支持向量并通过求解上式获得 $b$ ，但现实任务中常采用一种更鲁棒的做法：使用所有支持向量求解的平均值

$$
b=\frac{1}{|S|} \sum_{s \in S}\left(y_{s}-\sum_{i \in S} \alpha_{i} y_{i} \boldsymbol{x}_{i}^{\mathrm{T}} \boldsymbol{x}_{s}\right)\tag{6.18}
$$

