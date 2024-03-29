---
title: UAV Track Planning Algorithms
date: 2022-11-24
tags:
   - UAV
   - Track Planning
categories: Literature review
---

**摘要**：无人机(Unmanned Aerial Vehicle, UAV)是指能够在脱离人员的主动干预下，依据自身对周围环境进行感知、理解并进行自主飞行控制的飞行器。无人机因其具备低成本、适用场景多、无人员伤亡等优点，已成为世界各国为适应未来战场需要而重点研究的内容。随着航空技术的发展，越来越多的无人机替代飞行员执行高危险和高强度的军事侦察打击、民用巡检搜救等任务，无人机的航迹规划问题日益受到重视。航迹规划问题起源于机器人的运动规划，航迹规划就算综合考虑环境和目标任务规划最优的飞行路径，本质为多约束条件下的最优化问题，核心思想就是建立目标函数模型，有效处理各项约束，而无人机飞行涉及的约束条件众多，目前大多数学者首先通过求解全局最优路径，在对局部约束进行航迹动态优化。实际环境瞬息万变，复杂的电磁干扰等问题随时都会影响无人机的飞行，无人机航迹规划需要智能、实时性高、可靠性强的优秀算法作为支撑。

**关键词**：无人机、航迹规划

<!--more-->

## 1  绪论

无人机(Unmanned Aerial Vehicle, UAV)是指能够在脱离人员的主动干预下，依据自身对周围环境进行感知、理解并进行自主飞行控制的飞行器。无人机因其具备低成本、适用场景多、无人员伤亡等优点，已成为世界各国为适应未来战场需要而重点研究的内容。

随着航空技术的发展，越来越多的无人机替代飞行员执行高危险和高强度的军事侦察打击、民用巡检搜救等任务，无人机的航迹规划问题日益受到重视。航迹规划问题起源于机器人的运动规划，航迹规划就算综合考虑环境和目标任务规划最优的飞行路径，本质为多约束条件下的最优化问题[1]，核心思想就是建立目标函数模型，有效处理各项约束，而无人机飞行涉及的约束条件众多，目前大多数学者首先通过求解全局最优路径，在对局部约束进行航迹动态优化[2]。实际环境瞬息万变，复杂的电磁干扰等问题随时都会影响无人机的飞行，无人机航迹规划需要智能、实时性高、可靠性强的优秀算法作为支撑。

现实空间是三维的，国内外针对无人机航迹规划的研究，主要集中在现实三维空间，三维航迹规划问题应用范围广，实用价值高。在不考虑高度的情况下，将三维空间中的物体投影到二维平面，可以将三维航迹规划问题降维到二维，从而可以用栅格法规划空间以简化问题求解，但在实际应用过程中，复杂的地形等在高度上都会极大的影响无人机的飞行路径，因此若无特定说明，下文所指的无人机航迹规划均指三维空间下的无人机航迹规划。

针对无人机航迹规划问题，主要可以分为传统航迹规划算法和现代智能航迹规划算法[3]以及新出现的机器学习算法。传统航迹规划算法主要有人工势场法(Artificial Potential Field, APF)、迪杰斯特拉算法(Dijkstra Algorithm)、模拟退火法(Simulated Annealing Algorithm, SAA)和$A\ast$算法(A Star Algorithm)等，现代智能航迹规划算法主要有遗传算法(Genetic Algorithm, GA)、蚁群优化(Ant Colony Optimization, ACO)算法和粒子群优化(Particle Swarm Optimization, PSO)算法等。

## 2  问题描述

无人机航迹规划问题是指规划无人机在综合考虑环境、时间消耗、安全性、障碍物等因素条件下，完成指定飞行任务，其中主要的因素是飞行航迹短、消耗时间少。无人机航迹规划问题可以从空间规划、约束条件和算法规划等几个方面进行研究。

空间规划是指如何将实际环境优化建模，降低求解难度，如对不同类型的障碍物进行近似的优化计算，在特定环境下将三维空间投影至二维空间减少约束条件等。在无人机航迹规划的约束条件中，既有来自无人机本身的内在约束条件，即无人机性能，涉及到最小转弯半径、最大俯仰角、最大飞行时长和飞行高度等限制无人机自身飞行动作的因素，同时也包括来在环境的约束条件，如复杂地形、电磁场干扰、无人机集群干扰碰撞、军事威胁、障碍物干扰等约束条件。无人机航迹规划的核心问题是规划算法，规划算法可以追溯到机器人的运动规划。在特定的环境和无人机种类乃至不同的任务要求下，对规划算法的要求也不尽相同。现实环境时刻都在发生变化，新的环境干扰因素在不断增加，现有的规划算法迟早会有难以满足飞行任务的时候，需要对现有算法进行改进或者在此基础上进行创新，更具实际的规划问题采用适合规划算法，才能够使无人机高效而安全地完成飞行任务。

## 3  发展现状

### 3.1  传统经典算法

传统航迹规划算法主要有人工势场法、Dijkstra算法、模拟退火法和$A\ast$算法等。

### 3.1.1  人工势场法

人工势场法是一种模拟电势场分布的方法，将空间假设为人工势场，势场由引力场和斥力场组成，目标点对物体表现为引力，威胁源对物体表现为斥力，物体的移动是由引力和斥力的合力来控制。传统人工势场法定义如下。

航迹点X的引力势函数和斥力势函数分别为：

​$${U_{att}}\left( X \right) = {1 \over 2}{K_{att}}{\rho ^2}\left( {X,{X_G}} \right) \tag{3.1.1}$$

$${U_{rep}}\left( X \right) = \left\{
\begin{array}{lrl}
   {{1 \over 2}{K_{rep}}{{\left[ {{1 \over {\rho \left( {X,{X_O}} \right)}} - {1 \over {{\rho _O}}}} \right]}^2}} & & {\rho \left( {X,{X_O}} \right) \le {\rho _O}} \\
   0 & & {\rho \left( {X,{X_O}} \right) > {\rho _O}}
\end{array}
\right. \tag{3.1.2}$$


其中${K_{att}}$和${K_{rep}}$分别为引力和斥力增益系数且均为正常数；$\rho \left( {X,{X_G}} \right)$为航迹点$X$与目标点${X_G}$之间的距离；$\rho \left( {X,{X_O}} \right)$为航迹点$X$与威胁源${X_O}$之间的距离；${\rho _O}$为威胁源影响的最大距离，超过${\rho _O}$表示该威胁源不能对飞行器产生影响。

无人机在航迹点 收到的引力和斥力分别为相应势函数的负梯度：

$${F_{att}} =  - \nabla {U_{att}}\left( X \right) =  - {K_{att}}\rho \left( {X,{X_G}} \right) \tag{3.1.3}$$

$${F_{rep}} = \left\{
\begin{array}{lrl}
   {{K_{rep}}\left[ {{1 \over {\rho \left( {X,{X_O}} \right)}} - {1 \over {{\rho _O}}}} \right]{1 \over {{\rho ^2}\left( {X,{X_O}} \right)}}} & & {\rho \left( {X,{X_O}} \right) \le {\rho _O}} \\ 
   0 & & {\rho \left( {X,{X_O}} \right) > {\rho _O}} \\ 
\end{array}
\right. \tag{3.1.4}$$

总势场力为目标点对航迹点$X$产生的引力和所有威胁源产生的斥力的矢量和：

$${F_{total}} = {F_{att}} + \sum {{F_{rep}}} \tag{3.1.5}$$

人工势场法有着算法简明、实时性好、规划速度快的优点，适用于局部的实时航迹规划应用。但其缺点是当环境比较复杂时，由于人工势场会依赖局部势场，会使问题陷入局部最优解，使算法出现停滞，会在危险源附近反复抖动：当无人机离目标点较远时，合力方向更趋近于目标点，容易使得无人机进入威胁源区域；当目标点附近有危险源时，斥力非常大而引力较小，这样会使无人机远离目标点从而无法到达。

针对上述缺陷，众多学者结合无人机航迹规划特点对人工势场法进行了改进，提出了许多优秀的算法。Kathib在缩小威胁搜索空间的基础上对航迹点预规划寻找起始点和目标点之间的路径，实现了全局规划，弥补了人工势场容易陷入局部最优的缺陷；并改进了人工势场中的引力算法，有效解决了人工势场法容易陷入局部最优的问题，提高了算法的效率和适应性[4]。Chen Xia等人[5]在斥力势函数中加入无人机与目标点的距离，减小斥力，改善障碍物在目标附近时目标不可达的问题。设置引导点为无人机提供方向随机的势场力，解决局部极小值和震荡问题。王强等人[6]在人工势场法搜索陷入威胁区时，构造惩罚势函数替代斥力势函数，并使用模拟退火算法取代虚拟力引导的方法搜索逃离位置，效避免了局部极小值和抖动现象。姚远等人[7]使用相对速度斥力势场和斥力增益模糊控制器实现人工势场法的动态避障，避免了局部极小值。

### 3.1.2  Dijkstra算法

在图论中，求解最短路径常用Dijkstra算法来实现，基于贪心算法、广度优先搜索和动态规划来实现。在无人机航迹规划问题中，使用Dijkstra算法构建带有权值的图，其顶点代表航迹点，边代表所有可行航迹，求解得到这些航迹中的最优航迹。使用Dijkstra算法进行航迹规划实现比较简单，但算法的时空复杂度均为$O\left( {{n^2}} \right)$，与空间中的点数平方成正比，即在高维空间中的搜索时间更长，对内存的要求也更高，因此在二维和三维航迹规划中有着较大的性能差异，常用于二维空间的航迹规划[8,9]。因此使用Dijkstra 算法进行航迹规划的关键问题为如何在大范围空间内合理选取航迹点，尽可能减低时空复杂度。采用Voronoi图的方式[8]，选取各个威胁源连线的中垂线的交点为航迹点，能够最大化避免威胁，但代价是航迹较长，并且受限于无人机实际的转弯角，不一定可行。

### 3.1.3  模拟退火法

模拟退火法是一种启发式随即搜索算法，基于蒙特卡洛迭代，实际上是模拟固体物质的退火过程：在某一初始温度下，伴随温度控制参数按照降温函数不断下降，结合概率特性在解空间中寻找目标函数的全局最优解，也就是能有机会跳出局部最优解得到全局最优解。其优点是算法求得的解与初始状态无关，具有渐进收敛性，局部搜索能力强，但其缺点也比较明显，解的质量非常依赖与参数，全局搜索能力较差。在航迹规划中，模拟退火法的每一个解都代表一条无人机航迹，目标函数则是代价函数，但常常因为没有考虑无人机本身的性能约束，导致得到的航迹可行性不高。当然，如果允许适当增加运算量，模拟退火算法也可与易陷入局部最优解的算法相结合帮助其跳出局部最优找到全局最优解，如遗传模拟退火算法[10]，在交叉和变异过程中使用Metropolis准则判断是否接新解。

### 3.1.4  $A\ast$算法

在无人机航迹规划中，$A\ast$算法将搜索空间表示为网格的形式，网格的顶点作为航迹点，通过搜索邻域内代价函数最小的航迹点，逐步搜索至目标点，然后通过回溯得到最优航迹。采用OPEN表和CLOSE表分别存储待扩展航迹点和已扩展航迹点，其中代价函数为：

$$f\left( x \right) = g\left( x \right) + h\left( x \right) \tag{3.1.6}$$

其中$g\left( x \right)$为起始点到当前航迹点的实际代价，$h\left( x \right)$为启发函数，代表当前航迹点到目标点的估算代价，常用欧氏距离、曼哈顿距离等，而$A\ast$算法的核心就是启发函数，选取合适的启发函数，平衡计算时间和最优解的选取，能有效提高航迹规划性能，若$h\left( x \right)$始终等于实际代价，此时$A\ast$算法严格遵守最优路径搜索，搜索效率也最高。

$A\ast$作为直接搜索算法，搜索性能好、准确度高；但随着搜索空间的增大，航迹点不断增多，计算时间呈指数上升，规划时间大幅增加，不适用于实时规划。在航迹规划问题中，如何提高$A\ast$算法的计算速度，是研究的重点内容，出现众多基于$A\ast$算法的改进方法运用于无人机航迹规划。使用2.5维网格模型描述三维规划空间[11]，简化三维空间的坐标表示，改进后的效率要远大于三维网格划分方式。同样也可将三维航迹规划分解为二维规划和高度规划[12]，使用动态时间规整(Dynamic Time Warping, DTW)距离作为$A\ast$算法的启发函数进行二维规划，再由二维航迹结合高程数字地图，利用粒子沉降法赋予每个航迹点高度信息，生成三维航迹，能够在不改变搜索空间的条件下，大大减少搜索的航迹点数量，缩短规划时间。但航迹规划问题的复杂性，虽然学者们通过各种改进方法提高了$A\ast$算法的搜索效率，仍没有找到值恒等于从当前节点到目标点真实代价的启发函数，实现$A\ast$算法的高效最优搜索。

## 3.2  现代智能算法

受到自然界的生物规律启发，出现的许多智能算法也被广泛应用于无人机航迹规划问题。现代智能航迹规划算法主要有、遗传算法、蚁群优化算法和粒子群优化算法等。

### 3.2.1  蚁群优化算法

蚂蚁在觅食过程中会释放信息素来标记自己的路径，随着时间的推移，信息素回越来越多，同样也会存在着挥发使其减少，其他蚂蚁利用剩余的信息素来寻找食物，这就是蚁群算法的生物原理。蚁群算法正是使用这种原理，采取正反馈机制使搜索过程不断逼近最优解。

下面是关于蚁群算法的相关定义，其中$T$时刻蚂蚁$n$从节点$a$转移到节点$b$的概率可以表示为

$$P_{ab}^n\left( t \right) = \left\{
\begin{array}{lrl}
   {{{\left( {{\tau _{ab}}} \right)}^\alpha }{{\left( {{\eta _{ab}}} \right)}^\beta }} \over {\sum\limits_{b \in S_a^n} {{{\left( {{\tau _{ab}}} \right)}^\alpha }{{\left( {{\eta _{ab}}} \right)}^\beta }}} & & {b \in S_a^n}  \\
   0 &  & {Otherwise} 
\end{array}
\right. \tag{3.2.1}$$

其中${{\tau _{ab}}}$为节点$b$上的信息素浓度；${{\eta _{ab}}}$为节点$a$与节点$b$之间的启发函数，可以说距离开销或者其他组合如高度、安全度等；$\alpha $、$\beta $为${{\tau _{ab}}}$和${{\eta _{ab}}}$的相对重要性权重；$S_a^n$为节点$a$的所有邻居节点的集合。

针对信息素随着时间会挥发的现象，采用局部更新和全局更新的方式更新信息素的浓度。

局部更新是指当蚂蚁完成一个航迹点的选择后，降低次点对其他蚂蚁的吸引程度，以避免陷入局部最优解，更新公式为

$${\tau _{ab}}\left( {t + 1} \right) = \left( {1 - \xi } \right){\tau _{ab}}\left( t \right) \tag{3.2.2}$$

式中$\xi $为局部信息素挥发因子，用于减少此航迹点的吸引程度。

全局更新是指经过$m$时刻后，对蚂蚁完成一段航迹上所有的航迹点进行信息素挥发，更新公式为

$${\tau _{ab}}\left( {t + m} \right) = \left( {1 - \rho } \right){\tau _{ab}}\left( t \right) + \rho \Delta {\tau _{ab}} \tag{3.2.3}$$

$$\Delta {\tau _{ab}} = Q/J \tag{3.2.4}$$

式中$\rho $为全局信息素挥发因子；$J$为此条航迹的性能指标，$Q$为性能指标对于信息素更新的贡献比例。

蚁群优化算法只要对模型稍加修改就能应用于不同的规划场景，具有较强的鲁棒性，同样也是基于种群的算法，具有良好的并行性。但其初始信息素匮乏，初始的前进方向具有盲目性，搜索时间较长，并且容易出现停滞状况，在局部最优解停留而无求得全局最优解。基于无人机航迹规划在不同环境下的特点，学者通常从以上缺陷入手对蚁群优化算法进行改进。陈侠等人[13]采用几何优化算法消除了传统蚁群算法搜索随机性导致最后结果并非最优解问题，采用指标函数和自适应参数分别解决了航迹不平滑和容易陷入局部最优问题。使用差分进化算法对信息素浓度的更新进行修改，合理分配信息素以免某一路径上的信息素浓度过高而导致陷入在局部最优解[14]，此种改进在不影响蚁群算法优点的同时又能够加快算法收敛，但会导致航迹长度明显增加。同样也是基于信息素浓度过大或过小会产生影响，在信息素调整规则中引入航迹评价指标，并且将起始点和目标点之间的连线这一最短航迹信息反馈到系统中作为搜索的指导信号[15]，有效降低了算法寻找航迹的盲目性，加快了搜索效率。

### 3.2.2  遗传算法

“适者生存”的原则让生物在自然界的选择下逐渐向更适合生存的方向进化和变异，遗传算法也是基于这种原理诞生的启发式搜索算法。其思想借鉴生物的进化，根据染色体对应个体适应度越高越容易被选择的原则，从整个种群中选择一个父方和一个母方，对父方和母方抽取染色体交叉产生的后代染色体进行变异，然后按照之前步骤选择新的父方和母方并对新产生的子代染色体进行变异，直到产生新的种群。在无人机航迹规划问题中，遗传算法的每条染色体代表无人机的一条航迹，基因的编码方式也就是航迹节点的编码方式。基于概率的遗传算法有着优秀的性能，有着较强的鲁棒性，是一种高效、并行且全局的搜索方法，适用于三维的全局航迹规划，但收敛速度慢，算法效率低，容易过早收敛。

针对以上问题，部分学者对遗传算法进行了算法的改进。针对遗传算法初始种群的单一性问题，引入按照概率划分的小生存环境协同进化策略[16]并对种群进行动态调整，采用精英机制保留各代最优个体进行更新，有效提高了遗传算法的稳定性和精度。病毒遗传算法[17]采用定长实值和变长实值两种编码方式分别为染色体和病毒个体编码，通过采用反转录和转导这两种病毒感染操作实现两个种群间模块的信息交换，使进化信息在种群内迅速、定向地传播，消除了寻优过程的盲目性，改善了遗传算法早熟和局部收敛慢的问题，提高了收敛速度。

### 3.2.3  粒子群优化算法

将无人机航迹规划比作鸟群在森林中搜索食物，目标点就是要找到食物最多的地方，所有的鸟都不知道食物具体在哪个位置，只能感受到食物大概在哪个方向。每只鸟沿着自己判定的方向进行搜索，并在搜索的过程中记录自己曾经找到过食物且量最多的位置，同时所有的鸟都共享自己每一次发现食物的位置以及食物的量，这样鸟群就知道当前在哪个位置食物的量最多。在搜索的过程中每只鸟都会根据自己记忆中食物量最多的位置和当前鸟群记录的食物量最多的位置调整自己接下来搜索的方向。经过一段时间后鸟群就可以确定森林中食物最多的地方也就是找到了全局的最优解。

粒子群优化算法就是模拟鸟群飞行捕食行为，把每个粒子看作优化问题的一个可行解，每个粒子通过跟踪本身找到的局部最优解和种群中其他例子找到的全局最优解来确定下一步搜索区域。

粒子群算法的速度和位置更新方式分别为

$${v_{ij}}\left( {k + 1} \right) = w{v_{ij}}\left( k \right) + {c_1}{r_1}\left( {{p_{ij}}\left( k \right) - {x_{ij}}\left( k \right)} \right) + {c_2}{r_2}\left( {{g_{ij}}\left( k \right) - {x_{ij}}\left( k \right)} \right) \tag{3.2.5}$$

$${x_{ij}}\left( {k + 1} \right) = {x_{ij}}\left( k \right) + {v_{ij}}\left( k \right) \tag{3.2.6}$$

式中下标$i$、$j$、$k$分别代表第$i$个粒子、第$j$维空间、第$k$代粒子；$w$为惯性权重，描述了粒子对之前速度的“继承”；${c_1}$、${c_2}$为学习因子，提现粒子向全局最优粒子学习的特性；${r_1}$、${r_1}$为$\left( {0,1} \right)$的随机数。

粒子群优化算法与其他智能算法相比，最显著的特点就是没有“优胜劣汰”的机制，所有粒子在迭代过程中都会作为种群的成员保留。粒子群算法的优点就是具有较强的鲁棒性，对种群的大小敏感度不高，参数少，收敛速度快，但缺点也很明显，后起收敛速度慢，容易陷入局部最优解，通常学者对粒子群算法的改进集中在提高种群多样性来避免局部最优问题。在基本PSO算法中引入病毒种群[18]可以增强主群体粒子的多样性，提高局部搜索能力，解决基本PSO算法容易陷入局部最优、收敛速度慢的问题。

## 3.3  机器学习算法

随着人工智能技术的发展，在无人机航迹规划与跟踪领域都出现了一系列基于机器学习的模型。基于深度强化学习理论出现了大量的协同航迹规划方法，但有部分存在求解时间过长，不适用于在线应用的问题，而无人机飞行的实时性也是一个重要问题。基于数据驱动的非参数模型具有较强的非线性建模能力，在对不确定性目标的跟踪中有着较好的预测性能。

### 3.3.1  基于Q学习的无人机航迹规划算法

Q学习（Q-learning）算法是一种与模型无关的强化学习算法，以马尔科夫决策过程（Markov Decision Processes, MDPs）为理论基础。标准的马尔可夫模型决策过程可用一个五元组$\left\langle {S,A,P,R,\gamma } \right\rangle $来表示，其中：$S$代表离散有界的状态空间，$A$代表离散的动作空间，$P$为状态转移概率函数，$R$为回报函数，$\gamma $是折扣因子，用于确定延迟回报和立即回报的相对比例，越大表示延迟回报的重要程度越高。协同航迹规划问题就是基于马尔可夫模型，采取动作能够获得收益，当前环境根据飞行器的动作反馈相应的回报，算法的主要思想就是选取能够获得长期积累值${V^\pi }\left( {{s_t},{a_t}} \right) = E\left[ {\sum\limits_{i = 0}^\infty  {{\gamma ^i}r\left( {{s_{i + t}},{a_{i + t}}} \right)} } \right]$数学期望最大值，其中策略$\pi $只和状态有关而与事件无关，${s_t}$是$t$时刻的环境状态，$a_{t}$是$t$时刻选择的动作。

根据贝尔曼Bellman方程，得到最优策略指标：${V^ * }\left( x \right) = \mathop {\max }\limits_{a \in A} \left[ {R\left( {s,a} \right) + \gamma \sum\limits_{{S^{'}} \in S} {{P_{s,a}}\left( {s^{'}} \right)V\left( {s^{'}} \right)} } \right]$，其中$R\left( {s,a} \right)$是$r\left( {{s_t},{a_t}} \right)$的数学期望${P_{s,a}}\left( {s^{'}} \right)$是状态$s$下选取动作$a$后转移到下一状态$s^{'}$的概率，而有些环境中状态之间的转移概率$P$较难获得，而Q学习不需要获取转移概率$P$，因而可用来解决具有马尔可夫性质的问题。

Q学习是一种基于数值迭代的动态规划算法，与环境无关。定义Q函数作为评估函数：$Q\left( {s,a} \right) = R\left( {s,a} \right) + \gamma {V^ \ast }$，即从状态$s$开始选择第一个动作$a$执行后获得的最大累积回报的折算值，也就是立即回报值$R\left( {s,a} \right)$加上遵循最优策略的折算值，此时最优策略可以写成${\pi ^ {\ast} }\left( s \right) = \arg \mathop {\max }\limits_A Q\left( {{s_{t}},a} \right)$，也就是说如果agent用Q函数替代${V^\pi }$就可以不考虑转移概率$P$，而只考虑当前状态$s$的所有可供选择状态$a$，从中选出使$Q\left( {{s_{t}},a} \right)$最大的动作，即agent对当前状态的部分Q值做出多次反应，就能够选出动作序列使得全局最优。

Q学习通过对环境的不断探索，积累历史经验，agent通过不断试错来强化自身，最终可以达到自主选择最优动作的目标，即不论出于何种状态，都可给出到达目标状态的最优选择路径，该算法中环境和动作相互影响，动作的选择影响环境状态，环境也可以通过强化回报函数来反馈动作的优劣性，影响动作的选择。

### 3.3.2  基于循环神经网络的无人机航迹跟踪算法

传统的神经网络中，输入和输出彼此独立，循环神经网络（Recurrent Neural Network, RNN）可以有效处理序列数据，适合处理与时序相关的问题，无人机目标跟踪问题也是其位置随着时间动态变化的问题，可以简化为坐标、速度、加速对随时间动态变化，采用循环神经网络对其下一时刻的各参量进行预测，结合传统的目标跟踪模型，能够有效提高跟踪准确率。

在训练模型时将目标的绝对坐标输入，将期望输出目标的绝对坐标作为标签实现预测，但这样的方法应用到实际场景中难以到达较高的预测精度，因为模拟场景的坐标集为有限集，而实际场景的坐标集为无限集，无法训练所有实际场景中的坐标。将基于循环神经网络的算法用于传统的基于KF滤波算法的运动模型预测，利用循环神经网络对$k$时刻和$k-1$时刻的平均速度和 时刻瞬时速度进行计算，获取目标的绝对坐标，可以有效避免该问题。

记目标运动轨迹长度为$N$，令$ L_{1:N}^i = \left\{ {{c_{r,k}},k = 1:N} \right\} $

由于期望输出的目标$k-1$时刻和$k$时刻之间平均速度${\bar v_{r,k|k - 1}}$与坐标数据没有时序关联，使用下式对坐标数据进行差分处理，将输入坐标转化为平均速度：

$${\bar v_{r,k|k - 1}} = {c_{r,k}} - {c_{r,k - 1}} \tag{3.3.1}$$

由于目标的瞬时速度与期望输出的平均速度存在时序联系，将瞬时速度作为输入增加一维信息以提高预测精度，此时预测平均速度的LSTM第$j$个输入向量为：

$${\bar I_{L,j}} = \left\{ {{{\bar v}_{r,k|k - 1}},{v_{r,k}},k = j:{W_L} + j - 1} \right\} \tag{3.3.2}$$

对应标签为${\bar v_{r,{W_L} + j|{W_L} + j - 1}}$

相似的，使用滑动窗口从速度集$V_{1:N}^i$提取长度为${W_V}$的速度数据，得预测瞬时速度的LSTM模型第$j$个输入分量为：

$${I_{V,j}} = \left\{ {{v_{r,k}},k = j:{W_V} + j - 1} \right\} \tag{3.3.3}$$

对应标签为${v_{r,{W_V} + j|{W_V} + j - 1}}$。考虑到$k$时刻以前的几个时刻的瞬时速度存在与期望输出的$k$时刻瞬时预测速度存在时序关联性，无需额外处理直接使用LSTM建立映射关系。

经过LSTM模型的训练，对目标的平均速度和瞬时速度进行预测，将得到的预测值输入传统的KF滤波模型得到最终的最终估计和速度估计，重复每个坐标点的步骤最终得到对目标的连续跟踪。

## 4  未来展望

在无人机航迹规划研究领域，主要的研究包括新算法的提出，已有算法的改进，多算法融合以及大规模多无人机协同等方面。经过多年的发展，对现有算法的改进已经较为成熟，新出现的算法多集中于受生物规律启发的智能算法已经人工智能领域算法的应用。

### 4.1  多无人机协同航迹规划

多架无人机之间协同进行的航迹规划不同于单无人机体系，约束条件大为增加，群体之间的时空协同性尤为关键。除了单无人机体系需要考虑的威胁源、环境障碍，多无人机体系还增加了机群碰撞处理、多机协同通信等棘手问题。相较于单无人机，多无人机协同执行飞行任务效率更高，能更好地处理突发事件，多无人机集群在其中部分无人机无法继续执行任务需要返航时时，仍然能够继续完成任务。尽管多无人机协同航迹规划将问题带到了一个新的高度，但多机协同能够带来众多优势，此必将是未来发展的大趋势。

### 4.2  多算法融合

各种经典算法和智能算法都有其优势和不足，多算法融合能够将现有算法进行融合使用，有效弥补所融合算法的缺陷。例如使用遗传算法和稀疏$A\ast$算法融合的方法在线规划航迹，改进的遗传算法用于全局规划，稀疏$A\ast$算法用于应对突发情况，，结合两者的优势，能够很好的做到系统任务的兼容。又例如使用$A\ast$算法弥补模拟退火算法耗时太长的问题，进一步结合改善求解质量。多算法融合的目的就是为了将多种算法相结合，取长补短弥补不足，往往比提出新算法来的更便捷且有效，从而更好地规划无人机航迹。

 

## :bell:参考文献

[1] 路晶, 史宇, 张书畅, 邓劲礼. 无人机航迹规划算法综述[J]. 航空计算技术, 2022, 52(04): 131-134.

[2] 韩攀, 陈谋, 陈哨东等. 基于改进蚁群算法的无人机航迹规划[J]. 吉林大学学报: 信息科学版, 2013, 31(1): 66-72.

[3] 胡中华, 赵敏, 姚敏等. 无人机航迹规划技术研究及发展趋势[J]. 航空电子技术, 2009, 40(2): 24-29.

[4] Kathib O. Real-time Obstacle Avoidance for Manipulators and Mobile Robots[M]. New York: Springer New York, 1986.

[5] CHEN X, ZHANG J. The Three-Dimension Path Planning of UAV Based on Improved Artificial Potential Field in Dynamic Environment[C] International Conference on Intelligent Human-Machine Systems and Cybernetics. Tlemcen, Algeria: IEEE, 2013: 144-147.

[6] 王强, 张安, 吴忠杰. 改进人工势场法与模拟退火算法的无人机航路规划[J]. 火力与指挥控制, 2014(8): 70-73.

[7] 姚远, 周兴社, 张凯龙等. 基于稀疏A∗搜索和改进人工势场的无人机动态航迹规划[J]. 控制理论与应用, 2010, 27(7): 953-959.

[8] DONG S, ZHU X, LONG G. Cooperative Planning Method For Swarm UAVs Based on Hierarchical Strategy[C] International Conference on System Science, Engineering Design and Manufacturing Informatization. Pacific Grove, CA, USA: IEEE, 2012: 304-307.

[9] MAINI P, SUJIT P B. Path Planning for a UAV with Kinematic Constraints in the Presence of Polygonal Obstacles[C] International Conference on Unmanned Aircraft Systems. Arlington, VA, USA: IEEE, 2016: 62-67.

[10]邱福生, 杨建平, 邵绪威. 基于遗传模拟退火算法的无人机航迹规划[J]. 沈阳航空航天大学学报, 2014, 31(1): 16-19.

[11]占伟伟, 王伟, 陈能成等. 一种利用改进A∗算法的无人机航迹规划[J]. 武汉大学学报: 信息科学版, 2015, 40(3): 315-320.

[12]杨润洲, 丁勇, 张承果. 基于DTW 的改进A∗算法在航迹规划中的应用[J]. 电光与控制, 2016, 23(6): 5-10.

[13]陈侠，艾宇迪，梁红利． 基于改进蚁群算法的无人机三维航迹规划研究［Ｊ］． 战术导弹技术， 2019(2): 59-66. 

[14]DUAN H, YU Y, ZHANG X, et al. Three-Dimension Path Planning for UCAV Using Hybrid Meta-Heuristic ACO-DE Algorithm[J]. Simulation Modelling Practice & Theory, 2010, 18(8): 1104-1115.

[15]TAO J, WANG Y, YANG H, et al. Three-Dimensional Path Planning of Unmanned Aerial Vehicle under Complicated Environment[C] Control and Decision Conference. Big Sky, MT, USA: IEEE, 2016: 6377-6382.

[16]鱼佳欣, 李刚, 李东涛等. 改进量子遗传算法在无人机航迹规划中的应用[J]. 计算机仿真, 2015, 32(5): 106-109.

[17]俞琪, 刘新, 周成平等. 基于病毒遗传算法的快速航迹规划方法[J]. 宇航学报, 2011, 32(4): 756-761.

[18]吴天爱, 吴云玉, 别晓峰. 采用病毒粒子群优化算法的飞行器航迹规划[J]. 电光与控制, 2014, 21(8): 102-105.