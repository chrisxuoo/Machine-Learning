# [线性支持向量机](http://www.cnblogs.com/pinard/p/6097604.html)

支持向量机\(Support Vecor Machine,以下简称SVM\)虽然诞生只有短短的二十多年，但是自一诞生便由于它良好的分类性能席卷了机器学习领域，并牢牢压制了神经网络领域好多年。如果不考虑集成学习的算法，不考虑特定的训练数据集，在分类算法中的表现SVM说是排第一估计是没有什么异议的。

SVM是一个二元分类算法，线性分类和非线性分类都支持。经过演进，现在也可以支持多元分类，同时经过扩展，也能应用于回归问题。本系列文章就对SVM的原理做一个总结。本篇的重点是SVM用于线性分类时模型和损失函数优化的一个总结。

# 1. 回顾感知机模型

在[感知机原理小结](/ml/svm/gan-zhi-ji-mo-xing.md)中，我们讲到了感知机的分类原理，感知机的模型就是尝试找到一条直线，能够把二元数据隔离开。放到三维空间或者更高维的空间，感知机的模型就是尝试找到一个超平面，能够把所有的二元类别隔离开。对于这个分离的超平面，我们定义为$$w^Tx + b = 0$$，如下图。在超平面$$w^Tx + b = 0$$，上方的我们定义为y=1,在超平面$$w^Tx + b = 0$$下方的我们定义为y=−1。可以看出满足这个条件的超平面并不止一个。那么我们可能会尝试思考，这么多的可以分类的超平面，哪个是最好的呢？或者说哪个是泛化能力最强的呢?

![](http://images2015.cnblogs.com/blog/1042406/201611/1042406-20161124135616081-623185925.jpg)

接着我们看感知机模型的损失函数优化，它的思想是让所有误分类的点\(定义为M\)到超平面的距离和最小，即最小化下式：$$- \sum\limits_{x_i \in M}- y^{(i)}(w^Tx^{(i)} +b)\big / ||w||_2$$

当w和b成比例的增加，比如,当分子的w和b扩大N倍时，分母的L2范数也会扩大N倍。也就是说，分子和分母有固定的倍数关系。那么我们可以固定分子或者分母为1，然后求另一个即分子自己或者分母的倒数的最小化作为损失函数，这样可以简化我们的损失函数。在感知机模型中，我们采用的是保留分子，固定分母$$||w||_2 = 1$$,即最终感知机模型的损失函数为：$$- \sum\limits_{x_i \in M}- y^{(i)}(w^Tx^{(i)} +b)$$

如果我们不是固定分母，改为固定分子，作为分类模型有没有改进呢？

这些问题在我们引入SVM后会详细解释。

# 2. 函数间隔与几何间隔

在正式介绍SVM的模型和损失函数之前，我们还需要先了解下函数间隔和几何间隔的知识。

在分离超平面固定为$$w^Tx + b = 0$$的时候，$$|w^Tx + b |$$表示点x到超平面的距离。通过观察$$w^Tx + b$$和y是否同号，我们判断分类是否正确，这些知识我们在感知机模型里都有讲到。这里我们引入函数间隔的概念，定义函数间隔$$\gamma^{'}$$为：$$\gamma^{'} = y(w^Tx + b)$$

可以看到，它就是感知机模型里面的误分类点到超平面距离的分子。对于训练集中m个样本点对应的m个函数间隔的最小值，就是整个训练集的函数间隔。

函数间隔并不能正常反应点到超平面的距离，在感知机模型里我们也提到，当分子成比例的增长时，分母也是成倍增长。为了统一度量，我们需要对法向量w加上约束条件，这样我们就得到了几何间隔γ,定义为：$$\gamma = \frac{y(w^Tx + b)}{||w||_2} =  \frac{\gamma^{'}}{||w||_2}$$

几何间隔才是点到超平面的真正距离，感知机模型里用到的距离就是几何距离。

# 3. 支持向量

在感知机模型中，我们可以找到多个可以分类的超平面将数据分开，并且优化时希望所有的点都离超平面远。但是实际上离超平面很远的点已经被正确分类，我们让它离超平面更远并没有意义。反而我们最关心是那些离超平面很近的点，这些点很容易被误分类。如果我们可以让离超平面比较近的点尽可能的远离超平面，那么我们的分类效果会好有一些。SVM的思想起源正起于此。

如下图所示，分离超平面为$$w^Tx + b = 0$$，如果所有的样本不光可以被超平面分开，还和超平面保持一定的函数距离（下图函数距离为1），那么这样的分类超平面是比感知机的分类超平面优的。可以证明，这样的超平面只有一个。和超平面平行的保持一定的函数距离的这两个超平面对应的向量，我们定义为支持向量，如下图虚线所示。

![](http://images2015.cnblogs.com/blog/1042406/201611/1042406-20161124144326487-1331861308.jpg)

支持向量到超平面的距离为$$1/||w||_2$$,两个支持向量之间的距离为$$2/||w||_2$$。

# 4. SVM模型目标函数与优化

SVM的模型是让所有点到超平面的距离大于一定的距离，也就是所有的分类点要在各自类别的支持向量两边。用数学式子表示为：$$max \;\; \gamma = \frac{y(w^Tx + b)}{||w||_2}  \;\; s.t \;\; y_i(w^Tx_i + b) = \gamma^{'(i)} \geq \gamma^{'} (i =1,2,...m)$$

一般我们都取函数间隔γ′为1，这样我们的优化函数定义为：$$max \;\; \frac{1}{||w||_2}  \;\; s.t \;\; y_i(w^Tx_i + b)  \geq 1 (i =1,2,...m)$$

也就是说，我们要在约束条件$$y_i(w^Tx_i + b)  \geq 1 (i =1,2,...m)$$下，最大化$$\frac{1}{||w||_2}$$。可以看出，这个感知机的优化方式不同，感知机是固定分母优化分子，而SVM是固定分子优化分母，同时加上了支持向量的限制。

由于$$\frac{1}{||w||_2}$$的最大化等同于$$\frac{1}{2}||w||_2^2$$的最小化。这样SVM的优化函数等价于：

$$min \;\; \frac{1}{2}||w||_2^2  \;\; s.t \;\; y_i(w^Tx_i + b)  \geq 1 (i =1,2,...m)$$

由于目标函数$$\frac{1}{2}||w||_2^2$$是凸函数，同时约束条件不等式是仿射的，根据凸优化理论，我们可以通过拉格朗日函数将我们的优化目标转化为无约束的优化函数，这和[最大熵模型原理小结](/ml/regression/max-entropy.md)中讲到了目标函数的优化方法一样。具体的，优化函数转化为：$$L(w,b,\alpha) = \frac{1}{2}||w||_2^2 - \sum\limits_{i=1}^{m}\alpha_i[y_i(w^Tx_i + b) - 1] \;$$满足$$\alpha_i \geq 0$$

由于引入了朗格朗日乘子，我们的优化目标变成：$$min(w,b)( max(\alpha_i \geq 0)(L(w,b,\alpha)))$$

和最大熵模型一样的，我们的这个优化函数满足KKT条件，也就是说，我们可以通过拉格朗日对偶将我们的优化问题转化为等价的对偶问题来求解。如果对凸优化和拉格朗日对偶不熟悉，建议阅读鲍德的《凸优化》。

也就是说，现在我们要求的是：$$max(\alpha_i \geq 0)( \;min(w,b)(\;  L(w,b,\alpha)))$$

从上式中，我们可以先求优化函数对于w和b的极小值。接着再求拉格朗日乘子α的极大值。

首先我们来求w和b的极小值，即$$min(w,b)(\;  L(w,b,\alpha))$$。这个极值我们可以通过对w和b分别求偏导数得到：


$$
\frac{\partial L}{\partial w} = 0 \;\Rightarrow w = \sum\limits_{i=1}^{m}\alpha_iy_ix_i
$$



$$
\frac{\partial L}{\partial b} = 0 \;\Rightarrow 0 = \sum\limits_{i=1}^{m}\alpha_iy_i
$$


从上两式子可以看出，我们已经求得了w和α的关系，只要我们后面接着能够求出优化函数极大化对应的α，就可以求出我们的w了，至于b，由于上两式已经没有b，所以最后的b可以有多个。

好了，既然我们已经求出w和α的关系，就可以带入优化函数L\(w,b,α\)消去w了。我们定义:$$\psi(\alpha) = min(w,b)\;  L(w,b,\alpha)$$

现在我们来看将w替换为α的表达式以后的优化函数ψ\(α\)的表达式：  
$$\begin{aligned} \psi(\alpha) & =  \frac{1}{2}||w||_2^2 - \sum\limits_{i=1}^{m}\alpha_i[y_i(w^Tx_i + b) - 1] \\& = \frac{1}{2}w^Tw-\sum\limits_{i=1}^{m}\alpha_iy_iw^Tx_i - \sum\limits_{i=1}^{m}\alpha_iy_ib + \sum\limits_{i=1}^{m}\alpha_i \\& = \frac{1}{2}w^T\sum\limits_{i=1}^{m}\alpha_iy_ix_i -\sum\limits_{i=1}^{m}\alpha_iy_iw^Tx_i - \sum\limits_{i=1}^{m}\alpha_iy_ib + \sum\limits_{i=1}^{m}\alpha_i \\& = \frac{1}{2}w^T\sum\limits_{i=1}^{m}\alpha_iy_ix_i - w^T\sum\limits_{i=1}^{m}\alpha_iy_ix_i - \sum\limits_{i=1}^{m}\alpha_iy_ib + \sum\limits_{i=1}^{m}\alpha_i  \\& = - \frac{1}{2}w^T\sum\limits_{i=1}^{m}\alpha_iy_ix_i - \sum\limits_{i=1}^{m}\alpha_iy_ib + \sum\limits_{i=1}^{m}\alpha_i  \\& = - \frac{1}{2}w^T\sum\limits_{i=1}^{m}\alpha_iy_ix_i - b\sum\limits_{i=1}^{m}\alpha_iy_i + \sum\limits_{i=1}^{m}\alpha_i \\& = -\frac{1}{2}(\sum\limits_{i=1}^{m}\alpha_iy_ix_i)^T(\sum\limits_{i=1}^{m}\alpha_iy_ix_i) - b\sum\limits_{i=1}^{m}\alpha_iy_i + \sum\limits_{i=1}^{m}\alpha_i  \\& = -\frac{1}{2}\sum\limits_{i=1}^{m}\alpha_iy_ix_i^T\sum\limits_{i=1}^{m}\alpha_iy_ix_i - b\sum\limits_{i=1}^{m}\alpha_iy_i + \sum\limits_{i=1}^{m}\alpha_i \\& = -\frac{1}{2}\sum\limits_{i=1}^{m}\alpha_iy_ix_i^T\sum\limits_{i=1}^{m}\alpha_iy_ix_i + \sum\limits_{i=1}^{m}\alpha_i \\& = -\frac{1}{2}\sum\limits_{i=1,j=1}^{m}\alpha_iy_ix_i^T\alpha_jy_jx_j + \sum\limits_{i=1}^{m}\alpha_i \\& = \sum\limits_{i=1}^{m}\alpha_i  - \frac{1}{2}\sum\limits_{i=1,j=1}^{m}\alpha_i\alpha_jy_iy_jx_i^Tx_j  \end{aligned}$$  
其中，\(1\)式到\(2\)式用到了范数的定义$$||w||_2^2 =w^Tw$$, \(2\)式到\(3\)式用到了上面的$$w = \sum\limits_{i=1}^{m}\alpha_iy_ix_i$$， \(3\)式到\(4\)式把和样本无关的$$w^T$$提前，\(4\)式到\(5\)式合并了同类项，\(5\)式到\(6\)式把和样本无关的b提前，\(6\)式到\(7\)式继续用到$$w = \sum\limits_{i=1}^{m}\alpha_iy_ix_i$$，（7）式到\(8\)式用到了向量的转置。由于常量的转置是其本身，所有只有向量$$x_i$$被转置，（8）式到\(9\)式用到了上面的$$\sum\limits_{i=1}^{m}\alpha_iy_i = 0$$，（9）式到\(10\)式使用了$$(a+b+c+...)(a+b+c+...)=aa+ab+ac+ba+bb+bc+...$$的乘法运算法则，（10）式到\(11\)式仅仅是位置的调整。

从上面可以看出，通过对w,b极小化以后，我们的优化函数ψ\(α\)仅仅只有α向量做参数。只要我们能够极大化ψ\(α\)，就可以求出此时对应的α，进而求出w,b.

对ψ\(α\)求极大化的数学表达式如下:


$$
max(\alpha)\; -\frac{1}{2}\sum\limits_{i=1}^{m}\sum\limits_{j=1}^{m}\alpha_i\alpha_jy_iy_j(x_i \bullet x_j) + \sum\limits_{i=1}^{m} \alpha_i
$$



$$
s.t. \; \sum\limits_{i=1}^{m}\alpha_iy_i = 0
$$



$$
\alpha_i \geq 0  \; i=1,2,...m
$$


可以去掉负号，即为等价的极小化问题如下：


$$
min(\alpha) \;\;\frac{1}{2}\sum\limits_{i=1}^{m}\sum\limits_{j=1}^{m}\alpha_i\alpha_jy_iy_j(x_i \bullet x_j) -  \sum\limits_{i=1}^{m} \alpha_i
$$



$$
s.t. \; \sum\limits_{i=1}^{m}\alpha_iy_i = 0
$$



$$
s.t. \; \sum\limits_{i=1}^{m}\alpha_iy_i = 0
$$


只要我们可以求出上式极小化时对应的α向量就可以求出w和b了。具体怎么极小化上式得到对应的α，一般需要用到SMO算法，这个算法比较复杂，我们后面会专门来讲。在这里，我们假设通过SMO算法，我们得到了对应的α的值$$\alpha^{*}$$。

那么我们根据$$w = \sum\limits_{i=1}^{m}\alpha_iy_ix_i$$，可以求出对应的w的值$$w^{*} = \sum\limits_{i=1}^{m}\alpha_i^{*}y_ix_i$$

求b则稍微麻烦一点。注意到，对于任意支持向量$$(x_x, y_s)$$都有$$y_s(w^Tx_s+b) = y_s(\sum\limits_{i=1}^{S}\alpha_iy_ix_i^Tx_s+b) = 1$$

假设我们有S个支持向量，则对应我们求出S个$$b^*$$,理论上这些$$b^*$$都可以作为最终的结果， 但是我们一般采用一种更健壮的办法，即求出所有支持向量所对应的$$b_s^{*}$$，然后将其平均值作为最后的结果。

怎么得到支持向量呢？根据KKT条件中的对偶互补条件$$\alpha_{i}^{*}(y_i(w^Tx_i + b) - 1) = 0$$，如果$$\alpha_i>0$$则有$$y_i(w^Tx_i + b) =1$$即点在支持向量上，否则如果$$\alpha_i=0$$则有$$y_i(w^Tx_i + b) \geq 1$$，即样本在支持向量上或者已经被正确分类。

# 5. 线性可分SVM的算法过程

这里我们对线性可分SVM的算法过程做一个总结。

输入是线性可分的m个样本$${(x_1,y_1), (x_2,y_2), ..., (x_m,y_m),}$$,,其中x为n维特征向量。y为二元输出，值为1，或者-1.

输出是分离超平面的参数$$w^{*}$$和$$b^{*}$$和分类决策函数。

算法过程如下：

1）构造约束优化问题


$$
min(\alpha)\;\; \frac{1}{2}\sum\limits_{i=1}^{m}\sum\limits_{j=1}^{m}\alpha_i\alpha_jy_iy_j(x_i \bullet x_j) -  \sum\limits_{i=1}^{m} \alpha_i
$$



$$
s.t. \; \sum\limits_{i=1}^{m}\alpha_iy_i = 0
$$



$$
\alpha_i \geq 0  \; i=1,2,...m
$$


2）用SMO算法求出上式最小时对应的α向量的值$$\alpha^{*}$$向量.

3\) 计算$$w^{*} = \sum\limits_{i=1}^{m}\alpha_i^{*}y_ix_i$$

4\) 找出所有的S个支持向量,即满足$$\alpha_s > 0$$对应的样本$$(x_s,y_s)$$，通过 $$y_s(\sum\limits_{i=1}^{S}\alpha_iy_ix_i^Tx_s+b) = 1$$，计算出每个支持向量$$(x_x, y_s)$$对应的$$b_s^{*}$$,计算出这些$$b_s^{*} = y_s - \sum\limits_{i=1}^{S}\alpha_iy_ix_i^Tx_s$$. 所有的$$b_s^{*}$$对应的平均值即为最终的$$b^{*} = \frac{1}{S}\sum\limits_{i=1}^{S}b_s^{*}$$

这样最终的分类超平面为：$$w^{*} \bullet x + b^{*} = 0$$，最终的分类决策函数为：$$f(x) = sign(w^{*} \bullet x + b^{*})$$

线性可分SVM的学习方法对于非线性的数据集是没有办法使用的， 有时候不能线性可分的原因是线性数据集里面多了少量的异常点，由于这些异常点导致了数据集不能线性可分， 那么怎么可以处理这些异常点使数据集依然可以用线性可分的思想呢？ 我们在下一节的线性SVM的软间隔最大化里继续讲。

