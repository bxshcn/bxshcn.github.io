---
layout: post
title:  "线性代数"
date:   2021-06-10 15:42:29 +0800
comments: true
categories: 数学
---
这是先前学习3bllue1brown的笔记。

## 一、vectors, what even are they? 什么是向量
从数学的角度来说，向量（a vector）可以是任何东西，只要两个向量相加（向量加法），以及数字与向量相乘（向量数乘）是有意义的即可。**这两种运算贯穿线性代数始终**。

数学中的向量由原点开始，指向终点的坐标，因此向量终点的坐标唯一表征了向量，但为了将向量和点坐标的写法区分开，我们将向量终点坐标竖着写来表示向量本身，比如$\begin{bmatrix} -2 \\\\ 3 \end{bmatrix}$。

向量加法的定义：$\overrightarrow v + \overrightarrow w$, move the second vector so that its tail sits at the tip of the first one(让第2个向量的起点落在第一个的终点上)，则draw a new vector from the tail of the first one to where the tips of the second one now sits, 这个新的向量就是它们的和。

从物理机械运动的角度看，我们先按照第一个向量的方式移动，然后再按照第二个向量的方式移动，其效果与直接从原点按照和向量的方式移动完全一样。

向量a与向量b相加（vector addition），就是将两个向量中的对应分量分别相加。

向量的数乘我们称为scaling（缩放）。用于缩放原始向量的数，我们称其为scalar（标量），**在线性代数中，数字的主要作用就是缩放向量**。

向量与标量相乘（verctor multiplication），就是将向量中的每个分量（each one of those components）与标量相乘。

## 二、linear combinations, span and bases 线性组合、张成的空间与基
在xy-coordinate system, there are two very special vectors: the unit vector in the x-direction and y-direction. 我们称其为x或者y方向的**单位向量**

这样xy-coordinate system中的任意向量，实际上是上述两个单位向量的线性组合：也就是缩放单位向量后再相加。我们也称这两个单位向量为**基向量（basis vectors）**， together, they're called **the basis of a coordinate system. 它们一起倍称为坐标系的基**。

如果选择不同的基向量，那我们就可以得到一个完全合理的新坐标系统（a completely reasonalbe, new coordinate system）.我们总是使用基向量的缩放标量组成的列表来表示一个向量，我们不那么精确的称其为向量的坐标。那么在不同的基向量下，同一个向量将具有不同的坐标。显然这两类坐标系统之间存在一定的关系。

简言之，当我们用数字描述向量时，它都依赖于我们正在使用的基basis。

linearly independent， if at least one vector can be expressed as a linear combination of others, since it is already in **the span of the others**.

on the other hand, if each vector really does add another dimension to the span, they are said to be linearly independent.

**technical definition of a basis of a space: a set of linearly independent vectors that span the spaces.**

基的定义：可张成整个空间的一组线性无关向量集合。

## 三、matrices as linear transformations
linear transformation is another fancy word of **function**: given a vector input, we get a vector output, 但变换有movement的隐含意味。设想一个变换是将空间中的所有点（向量）移动到其他点（向量）。

3Blue1Brown在这里以复平面上的function的几何图案来解释“transformation”。复平面即复数平面complex plane。

我们知道复数表示为$z=a+bi$，而the horizontal axis of the complex plane is identified with the real number line and the vertical axis with the imaginary number line, 因此Each complex number can be thought of as a point in the plane (or as a vector extending from the origin to that point). 也就是说，**任意一个复数可以表示为该平面上的一个点，或者该平面上的一个向量**。

简言之，complex plane is a representation of the space of complex numbers. In algebra and calculus, the complex plane is used to help visualize complex numbers. 

如果我们以该复平面上的一个点（也可理解为向量）作为输入，经过函数变换后，所得依然是这个平面上的一个点（或者说是一个向量）。这个变换过程可能非常复杂，如果我们将这个变换限定为linear，也就是说如果这个变换具备如下两个性值：
1. all **lines remain lines**, without getting curved
2. the **origin remain fixed** in place
我们可以将初略的这种变换视为“保持网格线平行且等距分布”的变换，因为如果不等距，那么网格的对角线在变换后必然变为曲线（因为该对角线的各个部分的斜率在不同网格会有所不同）

how would you describe one of these numerically?

如果$\vec v = -1\hat i + 2 \hat j$，那么经过变换后，依然有$Transformed \vec v = -1(Transformed\ \hat i) + 2(Transformed\ \hat j)$，也就是说，$\vec v$是基向量$\hat i$和$\hat j$的一个特定线性组合，无论经历何种变换，这个关系都保持不变。这意味着你只需要掌握变换后的$\hat i$和$\hat j$，就能推断出变换后的$\vec v$。

为了更好的理解这一点，我们看一张图：
![linear_transform](/assets/2021-06-10-linear-algebra/linear_transform.png)

上图中的$-1\begin{bmatrix} 1 \\\\ -2 \end{bmatrix} + 2\begin{bmatrix} 3 \\\\ 0 \end{bmatrix}$中的两个向量，正是原来的基变换后的新基。

我们将其写成矩阵的形式，则为：$\begin{bmatrix} 1 & 3\\\\ -2 & 0 \end{bmatrix} \begin{bmatrix} -1 \\\\ 2 \end{bmatrix}$。

其中左侧矩阵的列即为新基，而右侧的向量，则表征了在变换前后都保持一致的特定线性组合关系。

更确切的说，**左侧的矩阵唯一的定义了一个变换，它将原向量空间（由原基span张成）的基通过列直接映射为新的基**。

再换句话说，矩阵只是一个记号，它含有描述线性变换的信息。matrix is just a way of packaging the information needed to describe a linear transformation.

一些特别的线性变换。将整个复数平面逆时针旋转90&deg;C,基变量分别变为矩阵$\begin{bmatrix} 0 & -1\\\\ 1 & 0 \end{bmatrix}$中的两列。而shear（剪切），则意味着基变量分别变为矩阵$\begin{bmatrix} 0 & 1\\\\ 1 & 1 \end{bmatrix}$中的两列。

如果上述决定线性变换的矩阵的两列是linear dependent，那么这个线性变换会将整个平面上的所有向量都squish into the line where the two vector sits. 也就是这两个向量张成的一维空间。

linear transformation can be thought as smooshing around space. 线性变换可被视为对空间的挤压伸展。

to sum up, linear transformations是一种操纵空间的手段，这种变换can be described using only a handful of numbers. the coordinates of where each basis vector lands(这些数字也就是变换后基向量的坐标)， 以这些坐标为列所构成的矩阵give us a language to describe these transformation,而矩阵向量乘法is just a way to compute what that transformation does to a given vector.

**each time you see a matrix, you can interpret it as a certain transformation of space!**

## 四、matrix multiplication as composition将矩阵相乘理解为变换的组合

always remember, multiplying two matrices具有几何意义，也就是applying one transformation then another. 但要记住，这个作用过程是从右向左，即先应用右侧的变换，然后再应用左侧的变换。它起源于函数的记号，因为我们总是write the functions on the left of variables,比如：$f(g(x))$

矩阵相乘时，它们的先后顺序影响结果吗？当然，所以$M_1M_2 \ne M_2M_1$

那么$(AB)C$和$A(BC)$是否相等呢？当然相等，因为不论是$(AB)C$还是$A(BC)$，它们都等同于按照C，B，A的顺序进行线性变换。所以$(AB)C = ABC = A(BC)$

上例非常清晰的说明了：good explanation > symbolic proof
so Emil Artin said, It is my experience that proofs involving matrices can be shortened by 50% if one throws the matrices out.换句话说，很多矩阵相关的证明只需要应用物理或者几何学上的概念进行解释性说明，而无需使用繁杂的符号推导。

### 四.1 三维空间中的线性变换
同样，线性变换在三维空间中，也是保持：
1. 网格线平行且等距分布
2. 原点不动

同样，三维线性变换由基向量的去向完全决定：these transformations are completely described by where the basis vectors go.

三维矩阵相乘在某些领域有特别重要的应用，比如computer graphics and robotics.计算机图形学和机器人学。

## 五、determinant
"the purpose of computation is insight, not numbers" by Richard Hamming

对于线性变换，有些seems generally stretch space out, while others squish it in. 那么一个非常有意思的问题是how to measure exactly how much it streches or squishes things. 如何精确测量transfomation对空间拉伸或者压缩的程度？

对于二维平面来说，就是测量一个给定区域面积增大或者减小的比例；对于三维空间来说，则是测量一个物体的体积增大或者减小的比例。这个特殊的缩放比例，被称为**the determinant of the transformation**。显然，由于初始给定区域为1个单位，因此，对2d空间来说，determinant实际给出了变换后的面积，对于3d空间来说，determinant实际给出了变换后的体积。

$det\left(\begin{bmatrix} -1 & 1\\\\ -1 & -1 \end{bmatrix}\right)=2$

你当然可以埋头计算，但务必记住，understanding what it represents is much more important than the computation.

对于二维平面来说，如果一个变换的determinant等于0， 意味着它将整个平面压缩到一条直线，甚至一个点。推而广之，只要检验一个矩阵的determinant为0，我们就可以推断出，该矩阵代表的变换会将空间压缩到更小的维度上。

显然determinant可能为负值，那么what would the idea of scaling an area by a negative amount mean?

**the idea of orientation**, 设想一张纸形成的平面存在正反两个面，我们称其为这个平面具有orientation。 在数学上，orientation是一个几何概念，使得可以在二维平面上描述某种旋转运动是正时针还是逆时针，或者在三维空间中区分左和右。在linear algebra中，orientation在任意有限维空间都具有意义：the orientation of an ordered basis is a kind of asymmetry that makes a reflection impossible to replicate by means of a simple rotation. 一组有序基所决定的空间的orientation，是某种非对称性，使得反射不可能通过单纯的旋转得到。比如在现实三维空间中，单纯的旋转并不能使人的左右两侧重合，但通过镜子反射却可以。因此在三维的欧几里得空间中，两个可能的basis orientation（注意orientation总是和basis相关的，basis决定了空间的主要核心特性，orientation决定了空间的另外一种核心特性）被称为right-handed和left-handed，也称为right-chiral和left-chiral。A vector space with an orientation selected is called **an oriented vector space**, while one not having an orientation selected, is called unoriented.
> chiral, |ˈkaɪrəl|，化学术语，具有不对称的形式，比如分子结构，无法在镜像上叠加。

换句话说，the sign of determinant表示了变换是否改变了空间的orientation。而the absolute value of the determinant仍然给出了对象的缩放比例。

再给出一个结论$det(M_1M_2) = det(M_1)det(M_2)$，同样无需使用数值证明，只进行解释：即两个矩阵积的determinant表征了先后进行两次变换后对象大小改变的程度，这与先实施$M_2$变换改变其大小，再实施$M_1$改变其大小是一样的。由于$det(M_x)$是scalar，因此它们的乘法不存在先后顺序，满足交换律，所以我们可以继续推导得到$det(M_1M_2) = det(M_2M_1)$

## 六、inverse matrices逆矩阵、列空间与null space
> 将null space翻译为零空间并不完美，null应为空和无的意思，即这个空间上的向量经过线性变化后将归结为null，消失在原点中。

"To ask the right question is harder than to answer it" by Georg Cantor

column space(列空间)，rank（秩），null space（零空间）

Gaussian-elimination

Let the computers do the computing

线性代数的一个重要作用是能帮我们求解system of equations。很多情况下，这些equations会非常复杂，但同样存在这样一类情形，即未知数只具有常系数（constant coefficients)，且这些未知量之间只进行加操作，换句话说，只有scaling & add（这正好是向量支持的两种运算！），再换句话说，no exponents($x^2$) or fancy functions($sin(x)$), or multiplying two variable together。我们称其为linear system of equations(线性方程组).

线性方程组求解，可理解成寻找某个向量$\vec X$，使得其经过变换$A$后，与结果向量$\vec V$重合: $A \vec X = \vec V$

$A^{-1}A = I$，显然先正向变换，然后再逆向变换，最后得到的变换相当于什么也没做，我们称其为identity transformation

有了逆的概念后，线性方程组的求解过程类似于:

$A \vec X = \vec V \to A^{-1} A \vec X = A^{-1}\vec V \to X = A^{-1}\vec V$

对于A来说，只要$det(A)\ne 0$,那么就存在$A^{-1}$，从而X就有解。

但如果$det(A) = 0$，意味着变换A将原向量压缩到了更低维度的空间中，而我们无法将低维度的空间展开unsquish成更高维度的空间，没有什么线性变换能做到这一点，因为任何线性变换最多只能到同维度或者更低维度的空间。设想一下，将一条直线上的向量映射为一个平面上的多个向量，但线性变换只能将一个向量映射为另外一个向量。实际上，根本不存在将一个输入映射为多个输出的函数。

但即便$det(A) = 0$，solutions can still exist. 当且仅当结果向量（即$\vec V$）正好位于对应的低维度空间中。

显然，当某个$det(A) = 0$时，变换A将原始向量空间压缩到了低维度的空间中，比如将一个三维空间压缩成一个平面，更进一步的压缩成一条直线，再进一步的压缩成一个点。

**为了描述变换对空间维度的压缩程度，或者更严格的说，为了描述变换后空间的维度，我们引入new terminology: rank**

rank, the number of dimensions in the output of linear transforamtion.而代表线性变换的矩阵的各个列，实际上就是新的基，新基所张成span的空间，就是column space,显然，**当$\vec V$正好在column space中时，这个方程有解**。

关于rank，更精确的定义：it's the number of dimensions of column space. 当rank等于列数时，rank最大，我们称其为full rank。

变换的秩，满秩变换，矩阵的秩，描述的对象都是**变换transformation**的某种特性。

零向量（向量的所有分量均为0）一定被包含在column space中，对于一个full rank transformation，唯一能在变换后落在原点的向量就是零向量自身。但对于非满秩的矩阵来说，它所代表的变换将空间压缩到更低维度上，也就是说，它会将一系列向量变换成零向量。比如一个线性变换将一个三维空间压缩到一个平面上，意味着有一整条线上的向量在变换后落在原点（而其他向量在变换后都落在目标平面上）；如果一个线性变换将一个三维看空间压缩到一条直线上，意味着有一整个面上的向量在变换后落在原点（而其他向量在变换后都落在目标直线上）。

**变换后，落在原点的向量的集合，被称为矩阵的null space或者核kernel。** 换句话说，**$A \vec X = \vec 0$的解所构成的空间，就是null space**。而对于满秩矩阵来说，满足该方程的唯一解就是零向量自身。

显然，**变换矩阵的column space的维度，与其null space的维度，两者相加必然等于矩阵的列的数目**。

### 六.1 nonsquare matrices
Is it reasonable to talk about transformations between different dimensions? 比如输入一个二维向量，得到一个三维向量？或者输入一个三维向量，得到一个二维向量？

假设input space为一个二维平面，basis为$\begin{bmatrix} 1 & 0\\\\ 0 & 1 \end{bmatrix}$，output in 3D，其基为$\begin{bmatrix} 2 & 0\\\\ -1 & 1 \\\\ -2 & 1 \end{bmatrix}$，这是一个`3*2`的矩阵，其column space是由其basis，即该矩阵的两个列，span成的经过原点的一个平面。所以这仍然是一个$平面 \to 平面$的映射。

实际上，我们可以将初始input space的基视为$\begin{bmatrix} 1 & 0 & \|0\| \\\\ 0 & 1 & \|0\| \end{bmatrix}$，它是3D space中经过原点的水平平面，而这个`3*2`矩阵所代表的变换，将3D space中的一个**水平**平面，映射为另外一个经过原点的**倾斜**平面。

换句话说，`3*2`矩阵所代表的变换，其几何意义，就是将一个二维平面，映射到三维空间（的平面）上，**因为该矩阵有两个列，表示输入空间有两个基向量，所以是二维的；有三行表示变换后变换后每个基向量都用三个分量表示**。

类似的，`2*3`矩阵代表的变换的几何意义，因为有三列，表示输入空间有三个基向量，因此是三维的。变换后每个基向量用2个分量表示，因此是一个二维平面。简言之，一个`2*3`矩阵将一个3D空间压缩成经过原点的**水平**平面。

不失一般性，一个`m*n`矩阵，表示将一个n-D空间变换成一个m-D空间。

特别的，一个`1*2`矩阵接收一个二维向量，然后产生一个数，这是一类非常有意义的变换，with close tie to the pot products

## 七、Dot products and duality 点积与duality
only with transformation can we understand dot products.

传统的观点认为：两个同维度向量的dot products的计算，就是对它们对应分量的积求和。

投影的观点认为：$\vec v \cdot \vec w$ 等于$\vec v$在$\vec w$上的投影的长度与$\vec w$的长度的积。如果投影向量（the projected $\vec v$的向量方向与$\vec w$相反，则将积取negative。反之，也可以将其视为$\vec w$在$\vec v$上的投影的长度与$\vec v$的长度的积。

特别得到，当两个向量perpendicular（垂直）,一个向量在另一个向量上的projection为zero vector，其长度为0，所以点积为0。

我们对于上面对两个向量点击的两种几何处理过程，表面看顺序不同，存在不对称性，但为什么会得到相同结果呢？如果我们将$\vec v$视为$k\vec w^{'}$，其中$\vec w^{'}$的方向与$\vec v$一致，且长度与$\vec w$相同，则$\vec v \cdot \vec w = (k\vec w^{'}) \cdot \vec w = k(\vec w^{'} \cdot \vec w)$, 由于$\vec w$和$\vec w^{'}$长度相等，只是方向不同，那么在空间上具有对称性，所以无论是让谁在谁上投影，都没有区别。

![dot_product_symmetry](/assets/2021-06-10-linear-algebra/dot_product_symmetry.png)
![dot_product_symmetry1](/assets/2021-06-10-linear-algebra/dot_product_symmetry1.png)

简言之，因为投影保留了向量的拉伸倍数特性，所以无论谁向谁投影，我们都可以将它们的点积视为两个对称向量点积的同样倍数。

那么上述两种观点是如何联系起来的呢？why does multiplying pairs of matching coordinates and adding them togethor have anything to do with projection?

the most satisfactory answer comes from duality. so **what is duality**?

formal linearity properties 严格的线性特性
1. $L(\vec v + \vec w) = L(\vec v) + L(\vec w)$  两个向量和的线性变换，等于两个向量线性变换结果的和
2. $L(c\vec v) = cL(\vec v)$ 

which we will ignore now, so as to not distract from the end goal。 直观的看，多维空间中任意一条直线上的等距点在经过线性变换后，在输出空间中依然保持等距。假设输出空间为一个number line，那么这些点就均匀的分布在该number line上。

另外,`1*2`矩阵与2d向量之间有一种nice association。我们可以将一个2d向量看作是一个`1*2`矩阵，反之亦然，那么what does this association mean geometrically? **将向量转化为数的线性变换，和这个向量本身has some kind of connection.**

**so投影本身就是一个线性变换**，with this projection, we just define a linear transformation from 2-D vectors to numbers. 所以我们就能找到描述这个变换的`1*2`矩阵，我们称其为**Projection matrix**。

那么这个`1*2`的线性变换的Projection matrix的两列的值是什么呢？如果我们定义单位长度向量$\hat u$正好落在上面的number line上，那么它的两个分量正好就是这个矩阵的两列：我们始终跟踪$\hat i$和$\hat j$，根据对称性它们在经历投影变换后都落在number line上，其值正好就是$\hat u$的两个坐标，而这两个分量自然就是变换矩阵的两个分量：

[!projection_transform](/assets/2021-06-10-linear-algebra/projection_transform.png)

所以，空间中任意向量$\begin{pmatrix} x \\\\ y \end{pmatrix}$，在number line上投影的坐标，即为$\begin{bmatrix} u_x & u_y \end{bmatrix} \begin{bmatrix} x \\\\ y \end{bmatrix} = u_xx+u_yy$

这就是为什么向量与某个单位向量的点积，可以理解为该向量在单位向量所在number line上的投影变换。that's why we can interpret taking a dot product with a unit vector as projecting onto the span of that unit vector.换句话说，将向量$\vec v$投影到单位向量$\hat u$所在number line上的长度，等于该向量与单位向量的点积$\vec v \cdot \hat u$。

注意这里我们有一个从2D空间到1D直线的线性变换，it is not defined in terms of numerical vectors or numerical dot products, but by projecting space onto a diagonal copy of the number line.

不失一般性，一个1 by 2的矩阵$\begin{bmatrix} w_x & w_y \end{bmatrix}$，对任意向量$\vec v$进行线性变换，就有：

$\begin{bmatrix} w_x & w_y \end{bmatrix} \vec v = w \begin{bmatrix} u_x & u_y \end{bmatrix} \vec v = w \left(\begin{bmatrix} u_x & u_y \end{bmatrix} \vec v \right) = w(\hat u \cdot \vec v)$

换句话说，使用矩阵$\begin{bmatrix} w_x & w_y \end{bmatrix}$对$\vec v$进行变换，相当于：
1. $\vec v$在$\vec w$所在的直线上投影
2. 缩放该投影值$length(\vec w)$倍数

在这里要记住两个事实：
1. $\vec v$在某条number line上的projection，是该向量与number line上的**单位向量**$\hat u$的点积, 即$\hat u \cdot \vec v$,而一般意义上的点积，即任意两个向量的点积，意味着projection and scaling。
2. linear transformation, 实际历经两个阶段，即先projection，再scaling，这两个阶段都与$\vec v$的特性（方向:$\hat u$；长度$length(\vec v)$）相关，

所以当你看到一个nd-to-1d的linear transformaton，it's associated with some vector. 反过来，但你看到某个向量，你就知道它意味着某个线性变换。简言之，**任意一个多维空间到number line的线性变换，都与空间中的唯一一个向量对应，也就是说，应用这个线性变换和与这个向量点乘等价。It's an example of something in math called "duality"。**Duality shows up in many different ways and forms throughout math. and it's super tricky to actually define. 粗略的说，duality是两种数学对象之间某种自然的 correspondence. 因此the vector is called **the dual vector of the transformation** and performing the linear transformation is the same as taking a dot product with that vector.

**you can say that dual of a vector is the linear transformation that it encodes. 向量和它所蕴涵的线性变换（project first, then scaling）是某种实质的一体两面。换句话说，向量并不只是空间中的箭头，也是the physical embodiment of a linear transformation线性变换的物质载体，它仿佛是一个特定变换的conceptual shorthand(概念速记号)**

小结一下。点积是理解投影的有利几何工具，通过点积，我们可以判断检验两个向量具有simiar direction，or perpendicular（垂直），or opposite direction, 这也许是关于点积你需要记住的一个重要部分。当然，线性变换和向量的紧密联系，是数学中duality的一个绝佳例子。

## 八、cross products叉积 in the light of linear transformation
### 八.1 standard introduction
以二维平面上的向量为例，$\vec V \times \vec W$所得是一个标量，为area of parallelogram, if $\vec V$ to $\vec W$ is counter-clockwise，otherwise negative area of parallelogram. 注意这里提到了clockwise or counter-clockwise，这实际是由平面的orientation决定的，orientation是空间的一个属性。

当你按顺序求基向量的叉积，比如$\hat i \times \hat j$,其结果为(+1)，即为正。实际上，**the order of basis vectors is what defines orientation. 有序基决定了orientation**，这个在第五章已经讲过。

那么如何计算两个向量$\vec v$和$\vec w$的cross products呢？以二维空间中的情况为例。
1. 我们知道determinant的含义是线性变换对原空间对象进行缩放的比例。
2. 对于一个二维空间中的两个向量的叉积，就是它们形成的平行四边形面积（当然还有正负符号）。
3. 我们可以使用$\vec v$和$\vec w$作为列定义一个矩阵，这也意味着我们定义了一个linear transformation，即将原来的basis$\hat i$变换到新的basis$\vec v$，将原来的basis$\hat j$变换到新的basis$\vec w$。这个矩阵的determinant即该变换后的缩放比例，由于变换前原始面积为1（因为$\hat i \times \hat j = 1$），所以这个缩放比例即为变换后$\vec v$和$\vec w$形成的parallelogram的面积，故：

$det \left( \begin{bmatrix} v_x & w_x \\\\ v_y & w_y \end{bmatrix} \right) = \vec v \times \vec w$

简言之，$\vec v$和$\vec w$的叉乘，等于以$\vec v$和$\vec w$作为基所决定的线性变换矩阵的determinant。

对于一个`2*2`矩阵的determinant，计算方法是: 

$det \left( \begin{bmatrix} v_x & w_x \\\\ v_y & w_y \end{bmatrix} \right) = v_x \times w_y-w_x \times v_y$

基于上述结论的推论：
1. 当两个向量的方向$\Rightarrow$perpendicular时，叉积最大，而如果$\Rightarrow$same direction，则叉积接近于0。
2. $k\vec v \times \vec w = k (\vec v \times \vec w)$

但上面并没有给出严格的叉积定义，因为我们一直在二维平面讨论它。如果延展到三维平面，两个向量的叉积具有什么含义？

the true cross product is something that combines two diffent 3-D vectors to get a new 3-D vector. 而且这个新向量并非原来向量的linear composition。假设$\vec v \times \vec w = \vec p$, 则$\vec p$具有这样的特性：
1. $length(\vec p)$等于$\vec v$和$\vec w$形成parallelogram的面积。
2. $\vec p$的方向，与$\vec v$和$\vec w$决定的平面垂直（perpendicular to the parallelogram）。注意这个平面还具有orientation，$\vec p$的方向实际上与这个平面的orientation一致。
> 3Brown1Blue给出了一个方向判断法则：right hand rule，即用forefingher（食指）指向$\vec v$的方向，用middlefinger（中指）指向$\vec w$，则thumb指向的方向，就是叉积$\vec p$的方向。

那么两个三维向量的叉积如何计算呢？这里直接给出公式：

$$\begin{pmatrix} v_1 \\\\ v_2 \\\\v_3 \end{pmatrix} \times
\begin{pmatrix} w_1 \\\\ w_2 \\\\w_3 \end{pmatrix} = 
det\left( \begin{bmatrix} 
\hat i & v_1 & w_1 \\\\
\hat j & v_2 & w_2 \\\\
\hat k & v_3 & w_3 \\\\
\end{bmatrix} \right) =
(v_2*w_3-w_2*v_3)\hat i + (v_3*w_1-v_1*w_3)\hat j + (v_1*w_2-w_1*v_2)\hat k
$$

what on earth does it mean to put in **a vector as the entry** of a matrix?

$\vec v \times \vec w$的结果，似乎是原基（$\hat i\ \hat j\ \hat k$）的一个线性组合，对应一个限量，其指向与$\vec v$和$\vec w$决定的面垂直且满足右手法则，长度为$\vec v$和$\vec w$决定的面积。但为什么我们要这么做？这样做的理由是什么？

### 八.2 deeper understanding with linear transformations
take glory in the difficulty of proof is stupid. difficult means we have not understood. the ideal is to be able to paint a landscape in which the proof is obvious.  --by Pierre Deligne

我们先给出大纲the plan：
1. 使用$\vec v$和$\vec w$定义一个3d-to-1d的linear transformation
2. 找到它的dual vector
3. 证明这个dual就是$\vec v \times \vec w$

不要混淆，务必记住：
1. 虽然dot production也是nd-to-1d的linear transformation，并且它就是对应$\vec v$这个dual vector，但这些linear transformation的本质是projection
2. $\vec v \times \vec w$也是nd-to-1d的linear transformation，但它具有另外的几何意义。
![cross_product_geometry](/assets/2021-06-10-linear-algebra/cross_product_geometry.png)

我们已经知道：3d空间中的任意三个向量组成矩阵的determinant为一个数，这本质上就是一种3d-to-1d的transformation，所以假设：
$f\left( \begin{bmatrix} x \\\\ y \\\\ z \end{bmatrix} \right) = 
det\left( \begin{bmatrix} x & v_1 & w_1 \\\\ y & v_2 & w_2 \\\\ z & v_3 & w_3 \\\\ \end{bmatrix} \right)$

实际上，这个函数还是linear，我们只要证明这个函数符合线性变换的两个性值即可：
1. $L(\vec a + \vec b) = L(\vec a) + L(\vec b)$
2. $L(k\vec a) = kL(\vec a)$ 

而$det\left( \begin{bmatrix} a_1 & v_1 & w_1 \\\\ a_2 & v_2 & w_2 \\\\a_3 & v_3 & w_3 \\\\ \end{bmatrix} \right) = (v_2 \times w_3-w_2 \times v_3)a_1 + (v_3 \times w_1-v_1 \times w_3)a_2 + (v_1 \times w_2-w_1 \times v_2)a_3$

故：
$L(\vec a) + L(\vec b) = det\left( \begin{bmatrix} a_1 & v_1 & w_1 \\\\ a_2 & v_2 & w_2 \\\\a_3 & v_3 & w_3 \\\\ \end{bmatrix} \right) + det\left( \begin{bmatrix} b_1 & v_1 & w_1 \\\\ b_2 & v_2 & w_2 \\\\b_3 & v_3 & w_3 \\\\ \end{bmatrix} \right) = $ 
$(v_2 \times\ w_3-w_2 \times\ v_3)a_1 + (v_3 \times\ w_1-v_1 \times\ w_3)a_2 + (v_1 \times\ w_2-w_1 \times\ v_2)a_3 + \\\\
(v_2 \times\ w_3-w_2 \times\ v_3)b_1 + (v_3 \times\ w_1-v_1 \times\ w_3)b_2 + (v_1 \times\ w_2-w_1 \times\ v_2)b_3 = \\\\
(v_2 \times\ w_3-w_2 \times\ v_3)(a_1+b_1)+(v_3 \times\ w_1-v_1 \times\ w_3)(a_2+b_2) + (v_1 \times\ w_2-w_1 \times\ v_2)(a_3+b_3) = $
$det\left( \begin{bmatrix} a_1+b_1 & v_1 & w_1 \\\\ a_2+b_2 & v_2 & w_2 \\\\a_3+b_3 & v_3 & w_3 \\\\ \end{bmatrix} \right) = L(\vec a + \vec b)$

即线性变换将3d空间中的某个向量$\begin{bmatrix} x \\\\ y \\\\ z \end{bmatrix}$转换成1d number line上的一个数。这个数就是该向量与$\vec v$,以及$\vec w$组成的parallelepiped（平行六面体）的体积。

回到前面，因为：

$$f\left(\begin{bmatrix} x \\\\ y \\\\ z \end{bmatrix}\right) = 
det\left( \begin{bmatrix} 
x & v_1 & w_1 \\\\
y & v_2 & w_2 \\\\
z & v_3 & w_3 \\\\
\end{bmatrix} \right)$$

所以存在一个`1*3`矩阵，使得：

$$\begin{bmatrix} p_1 & p_2 & p_3 \end{bmatrix} \begin{bmatrix} x \\\\ y \\\\ z \end{bmatrix}= 
det\left( \begin{bmatrix} 
x & v_1 & w_1 \\\\
y & v_2 & w_2 \\\\
z & v_3 & w_3 \\\\
\end{bmatrix} \right)$$

and the whole idea of duality is that the special thing about transformation from several dimensions to one dimension is that you can turn that matrix on its side, and instead, interpret the whole transformation as the dot product with a certain vector. 也就是说有：

$$\begin{bmatrix} p_1 \\\\ p_2 \\\\ p_3 \end{bmatrix} \cdot \begin{bmatrix} x \\\\ y \\\\ z \end{bmatrix}= 
det\left( \begin{bmatrix} 
x & v_1 & w_1 \\\\
y & v_2 & w_2 \\\\
z & v_3 & w_3 \\\\
\end{bmatrix} \right)$$

所以，我们最终要找的就是这样一个向量$\vec p$, 它与任意向量(x, y, z)的点积，与将该向量，$\vec v$和$\vec w$分作3列组成的一个`3*3`矩阵的determinant相等。这个过程具有什么样的几何意义呢？

当你将$\vec p$与某个向量(x, y, z)进行dot product时，意味着将该向量在$\vec p$上投影的长度，然后scale $length(\vec p)$倍。从几何上看，向量在$\vec p$上投影的长度等于平行六面体的高，如果$length(\vec p)$等于平行六面体的底面积，那么点积就正好等于平行六面体的体积。

换句话说，**这个线性函数对于给定向量的作用，is to project that vector onto the line which perpendicular to both $\vec v$ and $\vec w$, then to multiply the length of that projection by the area of parallelogram spanned by $\vec v$ and $\vec w$**，这和**垂直于$\vec v$和$\vec w$，且长度为平行四边形面积的向量与该向量点乘**是一回事。

小结：两个三维向量$\vec v$和$\vec w$的叉积给出一个新的向量$\vec p$，根据duality，$\vec p$对应一个线性变换，使得任意向量$\vec x$被映射为一个数（点积），且其值为以$\vec p$、$\vec v$和$\vec w$为列组成的`3*3`矩阵的determinant，也即这三者所构成的parallelepiped的“体积”(带符号)。

简而言之，叉乘是在三维空间中，利用两个已知向量找到一个新的向量，根据duality，这个新的向量对应一个新的线性变换，满足**特定的投影特性（该向量与三维空间中的任意向量的点积是一个特定值）**。所以点积更具一般性，是将任意n-维空压缩为一维的一种线性变换。

一句话，cross product是dot product的一种应用。它们并不是一回事。**dot product实施空间压缩，而cross product是在3-D空间中找到某个特性的向量（线性变换）：基于该线性变换来实施空间压缩（点乘）并使结果满足特定要求**。

向矩阵中插入$\hat i$, $\hat j$, $\hat k$, is a way of signaling that we should interpret those coefficients as the coordinates of a vector. 

## 九、change of basis
mathematics is the art of giving the same name to different things. -by Henri Poincare

anyway to translate between vectors and sets of numbers is called a coordinate system.我们可以将一个向量理解为一组基于basis的运动的和，而在各个basis方向上运动的距离，记为该向量在对应维度上的坐标，由此形成一个coordinate system。

**向量$\vec v = \begin{bmatrix} -1 \\\\ 2 \end{bmatrix}$本质上是线性组合**，是相对于基向量（比如$\vec i$, $\vec j$，当然也可以是其他基)的线性组合。

**向量可以理解为一种机械装置，用于驱动基的运动**

**变换可以理解为保持向量的线性组合这一特性不变，但使用新的基。换句话说，变换的本质是基的变换**。显然由于被驱动的基发生变化，所得结果肯定不同于原来所得。

变换对应矩阵，则左乘的变换矩阵的各列为新的基向量，而右边的向量的各分量即为**保持不变**的线性组合的系数。

上面表示特定线性组合的向量，是Jennifer的语言，使用变换矩阵左乘该向量，相当于将Jennifer的语言转化为我们能理解的语言。

那么反过来，如何将我们的语言，转化乘Jennifer能理解的语言呢？换句话说，已知空间中的一个向量，如何求得它在新基下的线性表示？

这里因此而引入了inverse的概念，变换的逆，the inverse of a transformation is a new transformation that corresponds to playing the first one backwards.

我们来看如何用Jennifer's language（即由新基组成的坐标系）来看，将一个向量左旋90&deg;C后的结果（基转换后向量在新基下的坐标）。
1. 假设在Jennifer看来，初始向量为$\begin{bmatrix} -1 \\\\ 2 \end{bmatrix}$
2. 先用**基变换**转换为我们的视角
3. 然后在我们的视角下进行变换
4. 最后再用**基变换的逆**转化为Jennifer视角（下的向量）
![basis_transfermation](/assets/2021-06-10-linear-algebra/basis_transfermation.png)

类似于$A^{-1}MA$的表达式，正表达了上面的过程，它是某个其他基下的某个变换，而我们可以在标准基的视角下来描述它，其中**M表示变换本身（它与基的选取无关）**，而两侧的A和$A^{-1}$则表示the shift in perspective(所选基的视角上的转化)

我们为什么要关注坐标系的变换呢？中间的这个M是否为某种特征，某种不变性呢？

## 十、eigenvectors and eigenvalues
eigen-，荷兰语，本意是own，引申为固有的，特征的意思
eigenvector ['aɪgən]

数学仅仅意味着the manipulation of numbers, the manipulation of structures吗？难道音乐仅仅意味着manipulation of notes（音符）? —— by Serge Lang

线性变换后，一般情况下向量离开get knocked off它span的一维空间，但特殊情况下，新的向量依然remains on its own span.意味着该变换仅仅只是strech或squish它而已，just like a scalar.

对于变换矩阵$\begin{bmatrix} 3 & 1 \\\\ 0 & 2\end{bmatrix}$，基向量$\hat i$，与向量$\begin{bmatrix} -1 \\\\1  \end{bmatrix}$在变换后，依然留在它们span的直线上，这就是该矩阵对应的所有拥有这一性质（留在它们张成空间）的向量，我们称这些向量为eigenvectors of the transformation, and each eigenvector has a value with it, which is just the factor by which it streched or squished during the transformation.

我们设想一下，当我们想将3D空间中的一个物体围绕一根轴进行旋转（3D rotation）一定角度。比如我们想将一个立方体围绕向量$\begin{bmatrix} 2 \\\\ 3 \\\\ 1 \end{bmatrix}$所在轴旋转30&deg;。这个变换对应矩阵的eigenvector就是$\begin{bmatrix} 2 \\\\ 3 \\\\ 1 \end{bmatrix}$，eigenvalue为1。显然，用eigenvector和eigenvalue来表示变换，比用矩阵来表示要容易得多。

$$\begin{bmatrix} 
\cos (\theta)\cos (\phi) & -\sin (\phi) & \cos (\theta)\sin (\phi) \\\\
\sin (\theta)\cos (\phi) & \cos (\theta) & \sin (\theta)\sin (\phi) \\\\
-\sin (\phi) & 0 & \cos (\phi) \\\\
\end{bmatrix}
$$

进一步的，围绕eigenvector旋转不同的度数，意味着不同的变换，也即对应不同的变换矩阵，所以变换矩阵是旋转度数的函数。换句话说，当我们在3D空间中旋转一个物体时，我们实质上是在应用不同的矩阵对其实施变换。

在先前的内容中，我们用basis来理解transformation，basis的不同，意味着我们实际在不同的坐标系中转换，即对同一个对象（向量）用不同的语言（basis）表述。**but often a better way to get the heart of what the linear transformation actually does is less dependent on your particular coordinate system. a common pattern in linear algebra is to find the eigenvectors and eigenvalues**. 

这里不会详述计算矩阵eigenvector和eigenvalue的具体方法，但会简述计算的基本思想those are most important for conceptual understanding.

根据前述eigenvector的定义，用公式表示：

$A\vec v = \lambda \vec v$

其中A为变换矩阵，$\vec v$为eigenvector，$\lambda$为eigenvalue。换句话说，matrix-vector multiplication gives the same result as just scaling the eigenvector $\vec v$ by some value $\lambda$.

对上述公式进行简单的变换：
$(A - \lambda I)\vec v = \vec 0$

即我们要找到这样一个向量$\vec v$，使得它与$(A - \lambda I)$的乘积为零向量。如果$\vec v$本身就是零向量，当然满足，但这并没有任何意义，what we want is a nonzero vector。记住A已知，假设为$\begin{bmatrix} 3 & 1 & 4 \\\\ 1 & 5 & 9 \\\\ 2 & 6 & 5\\\\ \end{bmatrix}$，则我们有：

$$\begin{bmatrix} 
3-\lambda  & 1 & 4 \\\\
1 & 5-\lambda & 9 \\\\
2 & 6 & 5-\lambda \\\\
\end{bmatrix}
\vec v = \vec 0
$$

我们知道，当且仅当矩阵代表的变换将空间压缩为更低维度的空间时，才会存在非零向量，使得矩阵与它的乘积为零向量。换句话说，**空间压缩意味着矩阵的determinant为0**，所以必然有:

$$det(A-\lambda I) = det\left(\begin{bmatrix} 
3-\lambda  & 1 & 4 \\\\
1 & 5-\lambda & 9 \\\\
2 & 6 & 5-\lambda \\\\
\end{bmatrix}\right)=0$$

上面左侧实际上是cubic（*involving terms of the third degree at most*） polynomial（*a mathematical expression of one or more algebraic terms each of which consists of a constant multiplied by one or more variables raised to a nonnegative integral power (such as $a + bx + cx^2$)*） in $\lambda$, 

显然，对于一个二阶矩阵，det(A)=0对应一个quatratic polynomial，会有两个解，即有两个eigenvalue，对于三维空间中的三阶变换矩阵，det(A)会有三个eigenvalue。一旦求得了特征值，则可求出对应的eigenvector。

那么根据A求解其eigenvector和eigenvalue有什么意义呢？参照上面3D rotation的例子，我们关注的实际上是已知eigenvector和eigenvalue，然后根据这个对物体进行变换，而且我们知道，同一个eigenvector和eigenvalue，对应无数个A。正因为如此，为了方便描述变换，我们才会使用eigenvecter和eigenvalue。

我们把这个问题放一放。

再看几个典型的特殊案例
1. 2D下的旋转。什么情况下a transformation does't have eigenvectors?当$det(A-\lambda I) = 0$没有实数解时，即说明它没有eigenvector。特别的，对于2D transformation，虽然$det(A-\lambda I) = 0$没有实数解，但可能存在复数解，比如2D平面内逆时针旋转90&deg;的变换，A对应$\begin{bmatrix} 0  & -1 \\\\ 1 & 0 \end{bmatrix}$，则$det(A-\lambda I) = \lambda^2+1 = 0$, 故$\lambda = i, or\ \lambda = -i$。这种情况下，虽然没有eigenvector，但eigenvalue为imaginary number，换句话说，**当eigenvalue为虚数时，意味着2D平面的某种旋转变换**。
2. 2D下的剪切sheer变换$\begin{bmatrix} 1  & 0 \\\\ 1 & 1 \end{bmatrix}$，有$(A-\lambda I) = (1-\lambda)^2 = 0$，故**只有一个解$\lambda = 1$，对应特征向量为all the vectors on the x-axis即在一条直线上**。
3. 2D下的放大操作$\begin{bmatrix} 2  & 0 \\\\ 0 & 2 \end{bmatrix}$，$(A-\lambda I)=0$有唯一解$\lambda = 2$，对应2D平面上的所有vector均为eigenvector！**此时具有相同特征值的多个eigenvectors不在一条直线上**。

**实际上，上面的几个典型例子即我们日常生活中的主要需求，在2D或者3D空间中，放大vs缩小，沿一个或者多个方向拉伸或压缩对象，沿特定轴旋转。区别于基于basis和coordinate system来描述变换的模式，我们可以采用eigenbasis来描述变换**。

anytime a matrix has 0's everywhere other than the diagonal is called a diagonal matrix(对角矩阵), **a way to interpret this is that all the basis vectors are eigenvectors with the diagonal entries of this matrix being their eigenvalues**. 对角阵具有很多很好的特性，因此操作更为容易，比如矩阵自乘，每次变换都相当于用eigenvalue来scale基，所以总有$\left(\begin{bmatrix} 2  & 0 \\\\ 0 & 3 \end{bmatrix}\right)^n=\begin{bmatrix} 2^n  & 0 \\\\ 0 & 3^n \end{bmatrix}$.

如果一个变换A有多个eigenvector，而且这几个eigenvector线性无关且足以span整个空间(假设为$\vec v_i$，则我们可以选择这些eigenvector作为新的basis，然后在新basis下描述同一变换：

$$\left(\begin{bmatrix} \vec v_1 & \vec v_2 & ... \end{bmatrix}\right)^{-1}A\begin{bmatrix} \vec v_1 & \vec v_2 & ... \end{bmatrix}$$

由于基即是特征向量，因此所得矩阵必然是diagonal matrix且对角线的元素就是对应基的特征值，这是因为在该新基坐标系下，这个变换只对基向量进行缩放（而不改变方向）。a set of  basis vectors, which are also eigenvectors, is called eigenbasis.

但并非所有矩阵都能对角化，比如sheer，doesn't have enough eigenvectors to span the full space. 

## 十一、abstract vector spaces
such axioms, together with other unmotivated definitions, make it difficult to master mathematics. 这些公理，以及其他一些不明动机的定义，使得数学很难掌握。 -Vladmir Arnold

vector是一个箭头，为方便起见我们用坐标来描述它？
vector是一个实数pairs，我们只是形象的将其理解为一个箭头？

实际上，直觉上我们都很确定，所处的空间独立于坐标的存在，空间无需坐标即存在，只是为了描述（静态大小、动态伸缩运动）的方便我们引入了坐标，而坐标取决于basis的选择。因此，linear algebra中的一些core concepts，比如determinant，eigenvector等 don't care about the coordinate system. 也就是说，它们的定义与坐标无关，而只是一个相对的概念。比如determinant指变换前后对象伸缩的比例，而eigenvector也是指变换前后只存在伸缩特性的方向（在3Blue1Brown说eigenvector是变换中留在其span空间中的向量，依然引入了向量的定义，因此是一个循环论证）。换句话说，both of these two properties are inheritly spatial, 你可以自由的改变坐标系（新的基和变换矩阵，新的向量坐标和表示），而不会改变the underlying values of either one(e.g. determinants, eigenvectors)。

所以，vectors are not fundermentally lists of real numbers, their underlying essence is something more spatial. so what exactly does "space" or "spatial" mean? 为此我们要谈论一些其他vectorish的东西，它们既不是箭头，也不是一组数，但同样has vector-ish qualities。

比如functions，从某种意义上来说，functions is just another type of vector. 秉承good explanation > symbolic proof的精神, 两个函数的和所构造的新函数，当然等于f(x)+g(x)；而函数的倍乘所得到的新函数，当然等于kf(x)，换句话说，函数天然满足线性运算特性。

不失一般性，向量是对满足线性运算规则对象的抽象。vector is the abstract object which satisfying the rule of linear operations. 因此函数就是一种向量：function is a type of vector.

空间是向量的集合，空间中线性无关向量的最大数量即为空间的维度。如果我们以n维函数空间中的任意函数$f_n$（可表示为n个基函数的线性组合）为输入，并定义一个新的函数$f_{n+1}$，其值不同于先前n维函数空间中的任意物理量，则$f_{n+1}(f_n)$实质上拓展了原函数空间的维度，形成了n+1维函数空间。这个过程递推，从而得到任意维度的空间。比如物理中的field场概念，即是3D物理空间（具有长、宽、高三种物理量）中，增加新的物理量的量度，形成的更高维的空间。比如电场，磁场。

向量支持且仅支持相加和数乘两种线性运算，由于空间中的箭头满足向量的特性，因此我们以之为背景来考虑linear algebra。显然，基于空间坐标表示的箭头所得到的有用结构（useful constructs）和一些解决问题的方法（problem-solving techniques）具有一般性。比如linear transformation, null space（压缩）, dot products（投影变换，压缩）, cross products（求垂直线）, eigen-everything（求缩放轴）。

a perfectly reasonable notion of a linear transformation for function. 对于求导过程它接收一个函数（原函数，向量），并输出另外一个函数（导函数，向量）（注意这里并没有引入新的物理量度，因此这个过程实际上都发生在二维函数空间中），由于函数是一种向量，因此求导过程自然可理解为一种发生在同一空间中的transformation。我们可以记为：

$$L(f(x))=d(f(x))=df(x)$$

不难证明，derivative is linear(求导是线性的），我们常常称求导函数为operator（算子）而不是transformation，因为这个术语实际上更为algebraic（transformation更为geormetric），但两者本质相同。那么类似于求导这种operator满足linear，what it means for a transformation of functions to be linear? 如果我们能用一个矩阵来描述求导过程？

linear transformations preserve addition and scalar multiplication. 相关的一个重要推论是，一个linear transformation可以由它对原basis向量的作用来描述。

all polynomial组成的空间，是arbitrary large degree。则初始情况下，basis函数分别为$b_0(x)=1, b_1(x)=x, ... b_n(x)=x^n, ...$，求导后的新基函数为$b’_0(x)=0, b_1(x)=1, ... b_n(x)=nx^{n-1}, ...$，将新的基作为矩阵的列记为：

$$\frac d {dx} = \begin{bmatrix} 
0 & 1 & 0 & 0 & ... \\\\
0 & 0 & 2 & 0 & ... \\\\
0 & 0 & 0 & 3 & ... \\\\
0 & 0 & 0 & 0 & ... \\\\
... & ... & ... & ... & ... \\\\
\end{bmatrix}$$

实际上，我们在线性代数中的概念，在函数世界都有直接的类比：
$$\begin{array}{c|c}
Linear\ algebra\ concepts & alternate\ names\ when\ applied\ to\ functions \\\\
\hline
Linear\ transformations & Linear\ operators \\\\
Dot\ products & Inner\ products \\\\
Eigenvectors & Eigenfunctions \\\\
\end{array}$$

so what are vectors? the form they take doesn't really matter! arrow, lists of numbers, functions, $\pi$ creatures，it can be anything as long as there is some notion of adding and scaling that follows these rules:
1. $\vec u + (\vec v + \vec w) = (\vec u+\vec v) + \vec w$
2. $\vec v + \vec w = \vec w + \vec v$
3. there is a vector $\vec 0$ that $\vec 0 + \vec v = \vec v\ for\ all\ \vec v$
4. for every vector $\vec v$ there is a vector $-\vec v$ so that $\vec v + (-\vec v) = 0$
5. $a(b\vec v) = (ab)\vec v$
6. $1\vec v = \vec v$
7. $a(\vec v + \vec w) = a\vec v + a\vec w$
8. $(a+b)\vec v = a\vec v+b\vec v$

我们称上面的8条规则为axioms公理。

当我们问向量是什么？类似于我们问3是什么？3是所有三样东西的集合，它是一个抽象的概念for all possible triplets of things and let you reason about all possible triplets using a single idea, so vector is analogous, it has many embodiments, but maths abstracts them all into a single, intangible notion of vector space. 

Abstractness is the price of generality. 普适的代价是抽象。

## 十二、the geometric explanation of Cramer's rule
if $T(\vec v) \cdot T(\vec w) = \vec v \cdot \vec w,\ for\ all\ \vec v\ and\ \vec w$, 则我们称变换T is Orthonormal. 即我们定义，不改变点积的变换，称为Orthonormal transformation。点积意味着投影和伸缩，如果变换后的投影和伸缩结果始终与之前一致，说明这个变换并不改变向量的相对位置，我们可以将其想象成刚体运动，no streching， or squishing, or mophing（变形）。

但Orthonormal transformation只是一种特殊情况，更一般的情况是变换的过程都会历经拉伸，且拉伸的比例为变换矩阵的determinant。Cramer's rule的本质，即是先给出每一个未知量在变换A前的表示（由该向量与基比如$\hat i$或者$\hat j$围成的空间的度量值：在二维下是面积，在三维下是体积），以及该对象在变换后的表示（由新向量与基围成的空间的度量值），两者相差为det(A)。

举例来说，假设$A = \begin{bmatrix} 2 & -1 \\\\ 0 & 1\\\\ \end{bmatrix}$
![cramer_rule](/assets/2021-06-10-linear-algebra/cramer_rule.png)

变换前面积为$y$，变换后新的面积为$det(A)y$。

又已知变换后的面积为向量$\begin{pmatrix} 2 \\\\ 0 \end{pmatrix}$和$\begin{pmatrix} 4 \\\\ 2 \end{pmatrix}$围成，因此其面积为$det\left(\begin{bmatrix} 2 & 4 \\\\ 0 & 2\\\\ \end{bmatrix}\right)$，因此$y=\frac {det\left(\begin{bmatrix} 2 & \|4\| \\\\ 0 & \|2\| \end{bmatrix}\right)} {\det\left(\begin{bmatrix} 2 & -1 \\\\ 0 & 1 \end{bmatrix}\right)}$








































