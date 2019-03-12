# 程序设计练习5：正则化线性回归和偏差vs方差

## 介绍
&#160;&#160;&#160;&#160;在这个练习中，你将实现正则化线性回归并用它来研究具有不同偏差方差属性的模型。在开始这个练习之前，我们强烈建议你观看视频讲座并完成相关主题的复习问题。

&#160;&#160;&#160;&#160;要开始练习，您需要下载起始代码并将其内容解压缩到您希望完成练习的目录中。如果需要，请在开始本练习之前使用octave/matlab中的cd命令更改到此目录。

&#160;&#160;&#160;&#160;你也可以在课程网站的“环境设置说明”中找到安装octave/matlab的说明。

### 本练习中包含的文件

    ex5.m - 指导你完成本次练习的Octave/MATLAB脚本
    ex5data1.mat - 数据集
    submit.m - 提交作业的脚本
    featureNormalize.m - 特征归一化函数
    fmincg.m - 函数最小化例程（类似于fminunc）
    plotFit.m - 绘制多项式拟合
    trainLinearReg.m - 用你的代价函数训练线性回归
    [*] linearRegCostFunction.m - 正则化线性回归代价函数
    [*] learningCurve.m - 生成学习曲线
    [*] polyFeatures.m - 将数据映射到多项式特征空间
    [*] validationCurve.m - 生成交叉验证曲线
    
    * 表示你要完成的文件
    
&#160;&#160;&#160;&#160;在整个练习中，您将使用ex2.m和ex2_reg.m脚本。这些脚本为后面的问题设置数据集，并调用你将要编写的函数。你无需修改它们两个，你只需按照本作业中的说明修改其他文件中的功能。

### 在哪里寻求帮助
&#160;&#160;&#160;&#160;本课程的练习使用非常适合数值计算的高级编程语言Octave或MATLAB。如果你没有安装Octave或MATLAB，请参阅课程网站上“环境设置说明”中的安装说明。

&#160;&#160;&#160;&#160;在Octave/MATLAB命令行中输入help紧跟函数名称会显示内建的函数说明。比如输入help plot会显示绘图函数的帮助信息。更多Octave和MATLAB的函数说明请在[Octave官网](https://octave.org/doc/interpreter/)和[MATLAB官网](https://www.mathworks.com/help/matlab/?refresh=true)查阅。

&#160;&#160;&#160;&#160;我们也非常鼓励使用在线讨论与其他学生讨论练习。但是，不要查看任何源代码或与他人共享源代码。

---

## 1、正则化线性回归
&#160;&#160;&#160;&#160;在练习的前半部分，你将使用水库水位的变化实现正则化线性回归来预测大坝的出水量。在下半部分中，你将对调试学习算法进行一些诊断，并检查偏差vs方差的影响。

&#160;&#160;&#160;&#160;提供的ex5.m脚本将指导你完成本次练习。

### 1.1 可视化数据集
&#160;&#160;&#160;&#160;我们将从可视化包含水位变化(x)和流出大坝的水量(y)的历史记录的数据集开始。

&#160;&#160;&#160;&#160;这个数据集分为三个部分：
> * 你的模型将要学习的训练集：X， y
> * 确定正则化参数的交叉验证集：Xval， yval
> * 用于评估性能的测试集。这些是你的模型在训练期间不知道的“不可见”样本：Xtest、ytest

&#160;&#160;&#160;&#160;然后ex5.m将绘制训练集（图1）。在接下来的部分中，你将实现线性回归，并使用它将直线拟合到数据中，并绘制学习曲线。在此之后，你将实现多项式回归，以便更好地匹配数据。
<center><img src="https://note.youdao.com/yws/api/personal/file/WEBae2d0f68169881a28fe435e3a662f1be?method=download&shareKey=fe253e17f5106ab2d7e50709e299a7ad" width="80%" /></center>
<center><h6>Figure 1: Data</h6></center>

### 1.2 正则化线性回归代价函数
&#160;&#160;&#160;&#160;回顾一下，正则化线性回归代价函数如下：
<center><img src="https://note.youdao.com/yws/api/personal/file/WEB9dfb6c729c9b92038a1c7383eeefac36?method=download&shareKey=4d22d9ed3a7518b18d92bdf5a24d9b16" width="60%"/></center>

&#160;&#160;&#160;&#160;其中λ是一个正则化参数，它控制正则化程度（因此，有助于防止过度拟合）。正则化项对总成本J进行了惩罚。随着模型参数`$θ_j$`的大小增加，惩罚也增加。 请注意，你不应该将`$θ_0$`项正则化。 （在Octave/MATLAB中，`$θ_0$`项表示为theta(1)，因为Octave/MATLAB中的索引从1开始）。

&#160;&#160;&#160;&#160;你现在应该完成linearRegCostFunction.m中的代码。你的任务是写一个函数来计算正则化线性回归代价函数。如果可能的话，试着向量化你的代码并且避免写循环。当你完成之后，ex5.m会使用初始化的theta（[1; 1]）调用你的函数。你应该期望输出值为303.993。

&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;*==你现在应该提交答案==*


### 1.3 正则化线性回归梯度
&#160;&#160;&#160;&#160;相应地，`$θ_j$`的正则化线性回归成本的偏导数定义为
<center><img src="https://note.youdao.com/yws/api/personal/file/WEB20adab4945f07159c894b26dcee69480?method=download&shareKey=a5560180022bd265f211a5ed17f937b3" width="70%"></center>

&#160;&#160;&#160;&#160;在linearRegCostFunction.m中添加代码来计算梯度，并在grad变量中返回它。当你完成后，ex5.m将会使用初始化的theta（[1; 1]）调用你的梯度函数。你应该期望看到[-15.30; 598.250]的梯度。

&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;*==你现在应该提交答案==*


### 1.4 拟合线性回归
&#160;&#160;&#160;&#160;一旦你的代价函数和梯度函数正确运行之后，ex5.m接下来会运行trainLinearReg.m中的代码来计算θ的最优值。该训练函数使用fmincg优化代价函数。

&#160;&#160;&#160;&#160;在这部分中，我们设置正则化参数λ为0。因为我们目前线性回归的实现是尝试拟合一个2维的θ，正则化对于如此低维度的θ将不会非常有用。在练习的后面部分，你将使用带正则化的多项式回归。

&#160;&#160;&#160;&#160;最后，ex5.m脚本应该绘制最拟合的直线，结果如图2所示。最佳拟合线告诉我们该模型不是很适合数据集，因为数据具有非线性模式。 虽然如图所示可视化最佳拟合是调试学习算法的一种可能方法，但是可视化数据和模型并不总是那么容易。 在下一节中，你将实现一个函数来生成学习曲线，这些曲线可以帮助你调试学习算法，即使数据不容易可视化。
<center><img src="https://note.youdao.com/yws/api/personal/file/WEB3e7a3b54ea0d2487fb79767fcb32b600?method=download&shareKey=e158ca9070e8f922ebba8dcae8a1a4d4" width="80%" /></center>
<center><h6>Figure 2: Linear Fit</h6></center>


## 2、偏差-方差
&#160;&#160;&#160;&#160;机器学习中的一个重要概念是偏差 - 方差权衡。 具有高偏差的模型对于数据来说不够复杂并且倾向于拟合，而具有高方差的模型过度拟合训练数据。

&#160;&#160;&#160;&#160;在本练习的这一部分中，你将在学习曲线上绘制训练和测试误差，以诊断偏差-方差问题。

### 2.1 学习曲线
&#160;&#160;&#160;&#160;你现在将完成生成用于调试学习算法曲线的代码。回想一下，学习曲线将训练和交叉验证错误绘制为训练集大小的函数。 你的工作是填写learningCurve.m，以便它返回训练集和交叉验证集的误差向量。

&#160;&#160;&#160;&#160;



&#160;&#160;&#160;&#160;

&#160;&#160;&#160;&#160;

&#160;&#160;&#160;&#160;

&#160;&#160;&#160;&#160;


&#160;&#160;&#160;&#160;

&#160;&#160;&#160;&#160;

&#160;&#160;&#160;&#160;

&#160;&#160;&#160;&#160;


&#160;&#160;&#160;&#160;

&#160;&#160;&#160;&#160;

&#160;&#160;&#160;&#160;

&#160;&#160;&#160;&#160;


&#160;&#160;&#160;&#160;

&#160;&#160;&#160;&#160;

&#160;&#160;&#160;&#160;

&#160;&#160;&#160;&#160;


&#160;&#160;&#160;&#160;

&#160;&#160;&#160;&#160;

&#160;&#160;&#160;&#160;

&#160;&#160;&#160;&#160;








&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;*==你现在应该提交答案==*


<center><img src="https://note.youdao.com/yws/api/personal/file/WEBf5bd61518df691ccc749cdd5917ee19e?method=download&shareKey=06e8e005d5e4722c972803a7cf827b64" width="80%" /></center>
<center><h6>Figure 3: Linear regression learning curve</h6></center>


<center><img src="https://note.youdao.com/yws/api/personal/file/WEBa79c2d8348c68e8f8ad318b4ab3e518d?method=download&shareKey=d4ab9dd98c7001bff86ce72256d266bc" width="80%" /></center>
<center><h6>Figure 4: Polynomial fit, λ = 0</h6></center>


<center><img src=https://note.youdao.com/yws/api/personal/file/WEB799f145646cb0a5fc2912a4ad18d8bf9?method=download&shareKey=2819c7f592a337541e8bd3de394e0c92" width="80%" /></center>
<center><h6>Figure 5: Polynomial learning curve, λ = 0</h6></center>


<center><img src="https://note.youdao.com/yws/api/personal/file/WEBa72cb377535bece3fc6efe4bc87cc59a?method=download&shareKey=29f5255fa2bc04d87c4271900da4a663" width="80%" /></center>
<center><h6>Figure 6: Polynomial fit, λ = 1</h6></center>


<center><img src="https://note.youdao.com/yws/api/personal/file/WEBaa107571c3e126004b110790bbf81f8b?method=download&shareKey=2495bb60183a11601e932df3601004f1" width="80%" /></center>
<center><h6>Figure 7: Polynomial learning curve, λ = 1</h6></center>


<center><img src="https://note.youdao.com/yws/api/personal/file/WEB60e4ceff0781f0ba2fa875e3355c23b4?method=download&shareKey=c1ca8c844a7ff0dcfd7d605f6d30e952" width="80%" /></center>
<center><h6>Figure 8: Polynomial fit, λ = 100</h6></center>


<center><img src="https://note.youdao.com/yws/api/personal/file/WEBf9a21ffce2b273f81dac3fc59aacfd6e?method=download&shareKey=385700a875d82d349467c22b0dbdbe57" width="80%" /></center>
<center><h6>Figure 9: Selecting λ using a cross validation set</h6></center>


<center><img src="https://note.youdao.com/yws/api/personal/file/WEB2a8cd9d81aecb91692efa5155c4ad848?method=download&shareKey=3c16b4e77129a79f4ebe57a11245d749" width="80%" /></center>
<center><h6>Figure 10: Optional (ungraded) exercise: Learning curve with randomly
selected examples</h6></center>


## 提交和评分

&#160;&#160;&#160;&#160;在完成任务的各个模块后，请务必使用提交系统向我们的服务器提交你的作业。以下是这个练习的每个部分如何得分的细则。

<center><img src="https://note.youdao.com/yws/api/personal/file/WEB1b85b40f9751976702e6d815b1fab6dc?method=download&shareKey=aa318433a88cc7b6df1be56722cca4b0" /></center>

&#160;&#160;&#160;&#160;你可以多次提交，但我们只考虑最高分。
