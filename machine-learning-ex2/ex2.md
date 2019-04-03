# 程序设计练习2：逻辑回归

## 介绍
&#160;&#160;&#160;&#160;在这个练习中，你将实现逻辑回归并将其运用在两个不同的数据集上。在开始这个练习之前，我们强烈建议你观看视频讲座并完成相关主题的复习题。

&#160;&#160;&#160;&#160;要开始练习，你需要下载起始代码并将其内容解压缩到你希望完成练习的目录中。如果需要，请在开始本练习之前使用Octave/MATLAB中的cd命令更改到此目录。

&#160;&#160;&#160;&#160;你也可以在课程网站的“环境设置说明”中找到安装Octave/MATLAB的说明。

### 本练习中包含的文件

    ex2.m - 指导你完成本次练习的Octave/MATLAB脚本
    ex2_reg.m - 练习后面部分的Octave/MATLAB脚本
    ex2data1.txt - 第一部分的数据集
    ex2data2.txt - 第二部分的数据集
    submit.m - 将你的答案发送到我们服务器的提交脚本
    mapFeature.m - 用于生成多项式特征的函数
    plotDecisionBoundary.m - 绘制分类器决策边界的函数
    [*] plotData.m - 绘制2D分类数据的函数
    [*] sigmoid.m - S形函数
    [*] costFunction.m - 逻辑回归代价函数
    [*] predict.m - 逻辑回归预测函数
    [*] costFunctionReg.m - 正规化逻辑回归函数
    
    * 表示你要完成的文件
&#160;&#160;&#160;&#160;在整个练习中，你将使用ex2.m和ex2_reg.m脚本。这些脚本为后面的问题设置数据集，并调用你将要编写的函数。你无需修改它们两个，你只需按照本作业中的说明修改其他文件中的功能。

### 在哪里寻求帮助
&#160;&#160;&#160;&#160;本课程的练习使用非常适合数值计算的高级编程语言Octave或MATLAB。如果你没有安装Octave或MATLAB，请参阅课程网站上“环境设置说明”中的安装说明。

&#160;&#160;&#160;&#160;在Octave/MATLAB命令行中输入help紧跟函数名称会显示内建的函数说明。比如输入help plot会显示绘图函数的帮助信息。更多Octave和MATLAB的函数说明请在[Octave官网](https://octave.org/doc/interpreter/)和[MATLAB官网](https://www.mathworks.com/help/matlab/?refresh=true)查阅。

&#160;&#160;&#160;&#160;我们也非常鼓励使用在线讨论与其他学生讨论练习。但是，不要查看任何源代码或与他人共享源代码。

---



## 1、逻辑回归
&#160;&#160;&#160;&#160;在这部分练习中，你将构建一个逻辑回归模型预测一个学生是否会被大学录取。假设你是某个大学院系的管理人员，你打算根据申请人两次考试的成绩来判断他们被录取的可能性。你有可以用作逻辑回归训练集的以前申请者的历史数据。每个训练样本都包含历史申请人两次考试的成绩和最终的录取结果。

&#160;&#160;&#160;&#160;你的任务是建立一个根据申请者两次考试的分数预测他们被录取可能性的分类模型。本大纲和ex2.m中的框架代码将指导你完成这项练习。

### 1.1 可视化数据
&#160;&#160;&#160;&#160;如果可能的话，在实现任何学习算法之前可视化数据总是好的。在ex2.m的第一部分，该代码会加载数据并调用函数plotData将其显示在二维图中。

&#160;&#160;&#160;&#160;现在你将完成plotData中的代码，最终结果如图1所示，其中两个轴代表两次考试成绩，正负样本用不同标记表示。
<center><img src="https://note.youdao.com/yws/api/personal/file/WEB807bb3ba0a9c9e798f7f01b64e76bd01?method=download&shareKey=d61f9296b8eab981df790c22d6eb213b" /></center>
<center><h6>Figure 1: Scatter plot of training data</h6></center>

&#160;&#160;&#160;&#160;为了让你更熟悉绘图，我们没有实现plotData.m中的代码，所以你可以尝试自己实现它。当然，是否自己实现是可选的，我们在下面给出了我们已经实现的代码，你可以直接复制或者参考。如果你要复制我们的代码，请参阅Octave/MATLAB的说明文档确保你完全理解了每行命令的含义。


```
    % Find Indices of Positive and Negative Examples
    pos = find(y==1); neg = find(y == 0);
    % Plot Examples
    plot(X(pos, 1), X(pos, 2), 'k+','LineWidth', 2, ...
    'MarkerSize', 7);
    plot(X(neg, 1), X(neg, 2), 'ko', 'MarkerFaceColor', 'y', ...
    'MarkerSize', 7);
```

### 1.2 实现
#### 1.2.1 热身练习：S函数
&#160;&#160;&#160;&#160;在开始实际成本函数之前，请回想一下逻辑回归假设定义为：
<center><img src="https://note.youdao.com/yws/api/personal/file/WEB11342b804da7fd3c3e94591ba1b2312a?method=download&shareKey=164908c5b56f39633d29cfd83080decf" /></center>

&#160;&#160;&#160;&#160;其中g为S函数，它的定义为：
<center><img src="https://note.youdao.com/yws/api/personal/file/WEBcd3497cf54ee7df767346d7a6f3d23c6?method=download&shareKey=65e91bcdfdffc89306e787a4d1feebb4" /></center>

&#160;&#160;&#160;&#160;首先你需要在sigmoid.m中实现S函数，这样你才能在其他程序中调用它。完成之后在Octave/MATLAB命令行中用一些值通过调用sigmoid(x)来测试一下。当x为正数且很大的时候，S的值应该接近1；当x为负数且很小的时候，S的值应该接近0；S(0)的值应该是0.5。你的代码还要可以处理向量和矩阵，对于矩阵，S函数将作用在矩阵的每个元素上。

&#160;&#160;&#160;&#160;你可以在Octave/MATLAB命令行中键入submit提交答案进行评分。提交脚本会提示你输入登录的e-mail和token，你可以网页中获取本次作业的token。

&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;*==你现在应该提交答案==*


#### 1.2.2 代价函数和梯度
&#160;&#160;&#160;&#160;现在你将实现逻辑回归的代价函数和梯度下降。完成costFunction.m的代码以返回代价值和梯度值。

&#160;&#160;&#160;&#160;回顾一下逻辑回归中的代价函数：
<center><img src="https://note.youdao.com/yws/api/personal/file/WEB84500ba0c71573c249f28f0adb426e7b?method=download&shareKey=99a6a4b632a3753e9600a4b74be49bbe" /></center>

&#160;&#160;&#160;&#160;并且该代价函数的梯度函数是一个和θ长度相同的向量，且第j个元素（for j = 0, 1, . . . , n）的定义如下：
<center><img src="https://note.youdao.com/yws/api/personal/file/WEBdfb589b5fa098c87bc4fc6e60461687a?method=download&shareKey=c89d344fb6f455aa5bddd24f0b00cffb" /></center>

&#160;&#160;&#160;&#160;注意，虽然这个梯度下降函数看起来与线性回归的梯度下降函数相同，但实际上是不同的，因为线性和逻辑回归的代价函数`$h_θ(x)$`的定义是不同的。

&#160;&#160;&#160;&#160;当你完成后，ex2.m会使用θ的初始值调用你的costFunction函数，你应该看到代价值大约是0.693。

&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;*==你现在应该提交答案==*


#### 1.2.3使用fminunc学习参数
&#160;&#160;&#160;&#160;在前面的作业中，你通过实现梯度下降找到了线性回归模型的最优参数。你写了一个代价函数并计算它的梯度函数，然后做了一个梯度下降的步骤。这一次，不用梯度下降，而是使用Octave/MATLAB内置的一个名为fminunc的函数。

&#160;&#160;&#160;&#160;Octave/MATLAB的fminunc是一个寻找无约束函数最小值的优化求解器。对于逻辑回归，我们想优化代价函数J(θ)的参数θ。

&#160;&#160;&#160;&#160;具体而言，在给定固定数据集（X和y值）的情况下，你将使用fminunc查找逻辑回归代价函数的最佳参数θ。 你将传递给fminunc以下输入：
> * 我们要优化参数的初始值
> * 当给定训练集和特定θ时，计算逻辑回归代价值和相对于数据集（X，y）的θ的梯度的函数

&#160;&#160;&#160;&#160;在ex2.m中我们已经写好了用正确参数调用fminunc的代码。

```
    % Set options for fminunc
    options = optimset('GradObj', 'on', 'MaxIter', 400);
    % Run fminunc to obtain the optimal theta
    % This function will return theta and the cost
    [theta, cost] = ...
    fminunc(@(t)(costFunction(t, X, y)), initial theta, options);
```
&#160;&#160;&#160;&#160;在这段代码中，我们首先定义了与fminunc一起使用的一些选项。具体来说，我们将GradObj选项为on，它告诉fminunc，我们的函数返回代价值和梯度。这允许fminunc在最小化函数时使用梯度。此外，我们将MaxIter选项设置为400，这样fminunc在终止之前最多可以运行400次。

&#160;&#160;&#160;&#160;为了指定我们最小化的实际函数，我们使用简写来指定带有@(t)（costFunction(t，X，y)）的函数。这样会创建一个带有参数t的函数，该函数调用costFunction。 这允许我们包装costFunction以用于fminunc。

&#160;&#160;&#160;&#160;如果你已经正确完成了costFunction，fminunc将收敛于正确的优化参数并返回最终的代价值和θ值。注意，通过使用fminunc，你不必自己编写任何循环，也不必像梯度下降那样设置学习率。这都是由fminunc完成的：你只需要提供一个计算代价和梯度的函数。

&#160;&#160;&#160;&#160;一旦fminunc运行结束，ex2.m将使用最优的θ调用costFunction函数，你会看到代价值大约是0.203。

&#160;&#160;&#160;&#160;最终的θ值将用于训练数据的决策边界，执行结果类似于图2。我们还鼓励你查看plotDecisionBoundary中的代码以了解如何使用θ绘制这样的决策边界。
<center><img src="https://note.youdao.com/yws/api/personal/file/WEB1ca4253703fa208b87fe28464c0d7e47?method=download&shareKey=831b35ac17e92b91cb283c0c9a84dd7d" /></center>
<center><h6>Figure 2: Training data with decision boundary</h6></center>


#### 1.2.4 评估逻辑回归
&#160;&#160;&#160;&#160;在训练到了参数后，你应该使用模型来预测某个特定的学生是否会被录取。对于一个第一次考试成绩为45，第二次考试成绩为85的学生，最后被录取的概率应该大约是0.776。

&#160;&#160;&#160;&#160;另一种评估训练到的参数质量的方法是看模型对我们的训练集的预测情况。这部分你的任务是完成predict.m中的代码。对于给定的训练集和参数θ，预测函数的预测值为1或0。

&#160;&#160;&#160;&#160;完成predict.m中的代码后，ex2.m脚本将通过计算正确的示例百分比来继续报告分类器的训练准确性。

&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;*==你现在应该提交答案==*


## 2、正则化逻辑回归

&#160;&#160;&#160;&#160;这部分练习中，你将实现正则化逻辑回归来预测工厂生产的芯片质量是否合格。在质检过程中，芯片会经过各种测试以确保其能正常工作。

&#160;&#160;&#160;&#160;假设你是某工厂的生产经理，并且你有一些芯片在两次不同测试中的测试结果。从这两个测试中，你想知道这些芯片是否合格。为了帮你做决定，你有一个以前芯片测试结果的数据集，从中你可以构建逻辑回归模型。

&#160;&#160;&#160;&#160;你会ex2_reg.m脚本来完成这部分练习。


### 2.1 数据可视化
&#160;&#160;&#160;&#160;与本练习的前几部分类似，plotData用于生成如图3所示的图，其中坐标轴是两次测试分数，使用不同的标记显示合格(y = 1，接受)和不合格(y = 0，拒绝)示例。
<center><img src="https://note.youdao.com/yws/api/personal/file/WEB80d075b69719c3c3eed996b8f04d8ea4?method=download&shareKey=3bac0695a559d9b9488e4aa3c76449d4" /></center>
<center><h6>Figure 3: Plot of training data</h6></center>

&#160;&#160;&#160;&#160;图3显示了我们的数据集不能通过一条直线将其分为正示例和负示例。因此，逻辑回归的直接应用在这个数据集上不会有很好的效果，因为逻辑回归只能找到一个线性的决策边界。


### 2.2 特征映射

&#160;&#160;&#160;&#160;更好地拟合数据的一种方法是从每个数据点创建更多的特性。在提供的函数mapFeature.m，我们将把特征映射到x1和x2的所有多项式项直到特征的6次方。
<center><img src="https://note.youdao.com/yws/api/personal/file/WEBf6311031791f74a6298e29a329e12589?method=download&shareKey=a713af1dfe8b7a8eac9b1617a712a4b1" /></center>

&#160;&#160;&#160;&#160;这个映射的结果是，我们的两个特征向量(两个QA测试的分数)被转换成一个28维的向量。在这个高维特征向量上训练逻辑回归分类器将具有更复杂的决策边界，并且在二维图中呈现非线性。虽然特征映射允许我们构建一个更具表现力的分类器，但它也更容易过度拟合。在练习的下一部分中，你将实现正则化逻辑回归来拟合数据，并亲自查看正则化如何帮助解决过拟合问题。


### 2.3 代价函数和梯度
&#160;&#160;&#160;&#160;现在你将编写代码来计算正则化逻辑回归的代价函数和梯度。请完成costFunctionReg.m中的代码并返回代价值和梯度。

&#160;&#160;&#160;&#160;回顾一下正则化逻辑回归的代价函数是：
<center><img src="https://note.youdao.com/yws/api/personal/file/WEB90b89c606237e29bfb018ccb4e259288?method=download&shareKey=53fd5765495b2cbc303d61e4de22b57b" /></center>

&#160;&#160;&#160;&#160;请注意,你不能正则化参数θ0。在Octave/MATLAB中，回想一下，索引从1开始，因此，你不应该在代码中正则化θ(1)参数(对应于θ0)。代价函数的梯度是一个向量，其中第j个元素的定义如下:
<center><img src="https://note.youdao.com/yws/api/personal/file/WEB9775019e1c50a4d75a3f60afb32498a9?method=download&shareKey=5be6ee8123534f0ed3807b75211ff0ec" /></center>
<center><img src="https://note.youdao.com/yws/api/personal/file/WEBf6ef1c1c9259c361d9d3bf1279c73608?method=download&shareKey=fbdb9ff9bc77455232b944b355b2c542" /></center>

&#160;&#160;&#160;&#160;当你完成后，ex2_reg.m会用θ的初始值（初始值全为0）调用costFunctionReg函数，你应该会看到代价值大约是0.693。

&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;*==你现在应该提交答案==*

#### 2.3.1 使用fminunc学习参数
&#160;&#160;&#160;&#160;和前面部分相似，你将使用fminunc学习参数θ。如果你已经正确完成了正则化逻辑回归的代价函数和梯度（在costFunctionReg.m中），你应该可以使用fminunc逐步完成ex2_reg.m的下一部分以学习参数θ。


### 2.4 绘制决策边界 
&#160;&#160;&#160;&#160;为了帮你可视化通过此分类器学习到的模型，我们已经提供了可以绘制非线性决策边界的函数plotDecisionBoundary.m以区分积极和消极样本。在plotDecisionBoundary.m中，我们通过计算分类器在等间距网格上的预测来绘制非线性决策边界，然后绘制预测从y = 0到y = 1变化的等高线图。

&#160;&#160;&#160;&#160;在学习到参数θ后，下一步就是在ex_reg.m中绘制如图4所示的决策边界：
<center><img src="https://note.youdao.com/yws/api/personal/file/WEBf1fb168679e14ddd383e29872c928c03?method=download&shareKey=e7e5ab8b9cc85a57150854a5a1907356" /></center>
<center><h6>Figure 4: Training data with decision boundary (λ = 1)</h6></center>


### 2.5 可选练习
&#160;&#160;&#160;&#160;在这部分练习中，你将尝试为数据集使用不同的正则化参数，以了解正则化是如何防止过拟合的。

&#160;&#160;&#160;&#160;注意到随着λ的变化决定边界的变化。λ较小时，你应该发现，对于每个训练样本分类器得到几乎都是正确的结果，但是它做了一个非常复杂的边界，因此过度拟合数据(如图5)。这不是一个好的决策边界：例如,它预测一个点x =(−0.25,1.5)被接受(y = 1)，这在训练样本中似乎是一个错误的决策。

&#160;&#160;&#160;&#160;λ较大时，你应该看到一个图显示了一个简单的决定边界仍然把正样本和负样本分的相当好。然而，如果λ值太高，你就不会得到一个不错的选择，决定边界不会很好地跟踪数据，因此欠拟合(图6)。

&#160;&#160;&#160;&#160;你不需要为这些可选的(未评分的)练习提交任何解决方案。
<center><img src="https://note.youdao.com/yws/api/personal/file/WEB9685ddf3d1484db16d344d01ecd8aeff?method=download&shareKey=7242e30d55fe9459f1379491f942a8aa" /></center>
<center><h6>Figure 5: No regularization (Overfitting) (λ = 0)</h6></center>

<center><img src="https://note.youdao.com/yws/api/personal/file/WEBaddbf7f9645c19d8b9b4bad138142e04?method=download&shareKey=349b0b954dbe2ff28704f4ea0570b494" /></center>
<center><h6>Figure 6: Too much regularization (Underfitting) (λ = 100)</h6></center>

## 提交和评分
&#160;&#160;&#160;&#160;完成作业的各个部分后，请务必使用提交系统将你的作业提交给我们的服务器。以下是对此练习的每个部分进行评分的细则。

<center><img src="https://note.youdao.com/yws/api/personal/file/WEBc816c5a4b03a658f4af4cd05082f2a8d?method=download&shareKey=66f0f5656f8b301cb122c3544f3d43d6" /></center>

&#160;&#160;&#160;&#160;你可以多次提交作业，但我们只考虑最高分。
