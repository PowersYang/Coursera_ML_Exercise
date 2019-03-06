# 程序设计练习3：多类分类和神经网络
## 介绍
&#160;&#160;&#160;&#160;在这个练习中，你将实现一对多逻辑回归和神经网络来识别手写数字。在开始编程之前，我们强烈建议观看视频课程并完成相关主题问题的复习。

&#160;&#160;&#160;&#160;要开始练习，您需要下载起始代码并将其内容解压缩到您希望完成练习的目录中。如果需要，请在开始本练习之前使用octave/matlab中的cd命令更改到此目录。

&#160;&#160;&#160;&#160;你也可以在课程网站的“环境设置说明”中找到安装octave/matlab的说明。

### 本练习包含的文件
    ex3.m - 指导你完成第一部分练习的Octave/MATLAB脚本
    ex3_nn.m - 指导你完成第二部分练习的Octave/MATLAB脚本
    ex3data1.mat - 手写数字的训练集
    ex3weights.mat - I神经网络练习的初始权值
    submit.m - 提交作业的脚本
    displayData.m - 帮助可视化数据集的函数
    fmincg.m - 最小化函数程序（类似于fminunc）
    sigmoid.m - S函数
    [*] lrCostFunction.m - 逻辑回归代价函数
    [*] oneVsAll.m - 训练一个one-vs-all多类分类器
    [*] predictOneVsAll.m - 使用one-vs-all多类分类器预测
    [*] predict.m - 神经网络预测函数
    * 表示你要完成的文件

&#160;&#160;&#160;&#160;在本练习中，你将使用ex3.m和ex3_nn.m脚本。这些脚本为题目设置数据集并且调用你编写的函数。你不需要修改这些脚本，只需要按照作业说明定义其他函数。

### 在哪里寻求帮助
&#160;&#160;&#160;&#160;本课程的练习使用非常适合数值计算的高级编程语言Octave或MATLAB。如果你没有安装Octave或MATLAB，请参阅课程网站上“环境设置说明”中的安装说明。

&#160;&#160;&#160;&#160;在Octave/MATLAB命令行中输入help紧跟函数名称会显示内建的函数说明。比如输入help plot会显示绘图函数的帮助信息。更多Octave和MATLAB的函数说明请在[Octave官网](https://octave.org/doc/interpreter/)和[MATLAB官网](https://www.mathworks.com/help/matlab/?refresh=true)查阅。

&#160;&#160;&#160;&#160;我们也非常鼓励使用在线讨论与其他学生讨论练习。但是，不要查看任何源代码或与他人共享源代码。

---

## 1、多分类
&#160;&#160;&#160;&#160;在这个练习中，你将使用逻辑回归和神经网络识别手写数字（0到9）。自动手写数字识别在今天得到了广泛的应用——从识别信封上的邮政编码到识别银行支票上的金额。这个练习将向你展示如何将你所学的方法用于这个分类任务。

&#160;&#160;&#160;&#160;在本练习的第一部分中，你将扩展之前对逻辑回归的实现，并将其应用于one-vs-all分类。

### 1.1 数据集
&#160;&#160;&#160;&#160;你会在ex3data1.mat中得到一个包含5000个手写数字识别样本的训练集。mat格式意味着该数据集保存为原生的Octave/MATLAB矩阵格式，而不是像csv那样的文本格式。这些矩阵可以通过load命令直接加载到你的程序中。加载之后，维度和值都无误的矩阵已经被加载到内存中了，而且该矩阵已经被命名了，你不需要对其重命名。

```
    % Load saved matrices from file
    load('ex3data1.mat');
    % The matrices X and y will now be in your Octave environment
```
&#160;&#160;&#160;&#160;ex3data1.mat中有5000个训练样本，其中每个训练样本都是一个20px * 20px的灰度数字图像。每个像素由一个浮点数表示，该浮点数表示该位置的灰度强度。20×20像素的网格被“展示”成一个400维的向量。这些训练示例中的每一个都变成了数据矩阵X中的一行。这样就给了我们一个5000×400矩阵X，其中每一行都是手写数字图像的训练示例。

<center><img src="https://note.youdao.com/yws/api/personal/file/WEB315e507ecc41d536d2b97d50eaa38c2c?method=download&shareKey=ac587a9edd8d9a05f04a6265dce03374" /></center>

&#160;&#160;&#160;&#160;训练集的第二部分是包含训练集标签的5000维向量y，为了使它与Octave/MATLAB索引更加兼容，在没有零索引的情况下，我们将数字0映射到值10。因此，数字“0”被标记为“10”，而“1”到“9”的数字按其自然顺序被标记为“1”到“9”。

### 1.2 数据可视化
&#160;&#160;&#160;&#160;你将从可视化训练集的一个子集开始。在ex3.m的Part 1中，代码随机从X中选择100行并将其传递给函数displayData。该函数将每一行映射到20px * 20px的灰度图像，并将图像一起显示。我们提供了displayData函数，同时也鼓励你观察该函数是如何工作的。
运行代码之后，你应该会看到一张图如图1所示：

<center><img src="https://note.youdao.com/yws/api/personal/file/WEB9e0c3338d4d8ea0a945c382c9d872a74?method=download&shareKey=e8cdec3fcfb45e912f85867aaf7876d7" width="50%" /></center>
<center><h6>Figure 1: Examples from the dataset</h6></center>

### 1.3 向量化逻辑回归
&#160;&#160;&#160;&#160;你将使用多个one-vs-all逻辑回归模型来构建一个多类分类器。由于有10个类（1-10个数字），你将需要训练10个单独的逻辑回归分类器。为了使训练有效，确保代码良好地支持向量化非常重要。在本节中，你将实现逻辑回归的向量化版本，它不使用任何for循环。你可以使用上一个练习中的代码作为这个练习的开始。

#### 1.3.1 向量化代价函数
&#160;&#160;&#160;&#160;我们将从编写成本函数的向量化版本开始。回想一下，在非正则化逻辑回归中，成本函数是：
<center><img src="https://note.youdao.com/yws/api/personal/file/WEB72f1222ea162d4dcf5da96d84a06728d?method=download&shareKey=a2ebe17dae0454884bc951a1f6f19bf6" width="50%" /></center>

&#160;&#160;&#160;&#160;为了对每个元素求和，我们必须计算hθ(x(i))，其中：
<center><img src="https://note.youdao.com/yws/api/personal/file/WEBbd388371fa613a903899348485e89a80?method=download&shareKey=9d8d1e0234c9cad72f6df3042bd1bb33" width="40%" /></center>
是S函数。这样一来我们就可以用矩阵乘法快速计算所有样本。其中X和他和θ的定义为：

<center><img src="https://note.youdao.com/yws/api/personal/file/WEBed18c69bc6f50581aa18d44d4a722b05?method=download&shareKey=ba9dba971b0f03976aa91c7e40088d13" width="50%" /></center>
然后通过计算得到矩阵Xθ，我们有：
<center><img src="https://note.youdao.com/yws/api/personal/file/WEB9ed339aed239d5698615023dbff6fab3?method=download&shareKey=f7f83c299083530f621040df8279cc59" width="50%" /></center>
在上个等式中，假设a和b都是向量，我们使a.T * b = b.T * a。这样我们就可以在一行代码中计算所有样本i的θ.T*x(i)（i为上标，表示第i个样本）。
&#160;&#160;&#160;&#160;你的任务就是在lrCostFunction.m中写非正则化代价函数的代码。你的实现应该使用我们上面的策略来计算的θ.T*x(i)。你还应该对代价函数的其余部分使用向量化的方法，一个完全向量化的lrCostFunction.m应该是没有任何循环的。

#### 1.3.2 向量化梯度
&#160;&#160;&#160;&#160;回想一下（）非正则化）逻辑回归成本的梯度是一个向量，其中第j个元素定义为：
<center><img src="https://note.youdao.com/yws/api/personal/file/WEB53b55e89044bb70cd7fe9cf6d28f186d?method=download&shareKey=72b6828d07fb4f1bdafc3441dc507970" width="70%" /></center>

&#160;&#160;&#160;&#160;为了在数据集上对该操作进行向量化，我们首先明确地为所有θj写出所有偏导数，
<center><img src="https://note.youdao.com/yws/api/personal/file/WEB1efa7d7beee3100f68265e8d1ae86c38?method=download&shareKey=7b40f5fc43a054c68fa61a510870fdfd" /></center>

&#160;&#160;&#160;&#160;注意，当(hθ(x(i))−y(i)是一个标量（单个数字）时，x(i)是一个向量。为了最后一步求导，我们让βi = (hθ(x(i)) − y(i)并观察：
<center><img src="https://note.youdao.com/yws/api/personal/file/WEB9b1dfe04b474b121fe7d467b850f75e5?method=download&shareKey=f1f37efb25b1804d751849602592d1a3" /></center>

&#160;&#160;&#160;&#160;上面的表达式允许我们在不适用任何循环的情况下计算所有偏导数。 如果你对线性代数比较熟悉，我们建议你通过上面的矩阵乘法，因为矢量化版本执行相同的计算。 你现在应该实现等式1来计算正确的矢量化梯度。 一旦结束后，就完成了通过实现梯度来完成函数lrCostFunction.m。

>**调试技巧**：向量化代码有时很棘手。调试的一个常见策略是使用size函数打印出正在处理的矩阵的大小。例如,给定一个大小100×20（100个样本,20特征）的数据矩阵的和一个维度维度是20×1θ的向量，你可以观察到Xθ是一个有效的乘法操作，虽然θX不是。此外，如果你有代码的非向量化版本，你可以比较向量化代码和非向量化代码的输出，以确保它们产生相同的输出。


#### 1.3.3 向量化正则化逻辑回归
&#160;&#160;&#160;&#160;在实现了逻辑回归向量化之后，现在你将向成本函数添加正则化。回想一下，对于正则化逻辑回归，成本函数的定义是
<center><img src="https://note.youdao.com/yws/api/personal/file/WEB6b20003cb9e8df116603b5bc3429e16b?method=download&shareKey=9ec878aea2f88c1c4983d2ceb117e8bd" width="80%" /></center>

&#160;&#160;&#160;&#160;注意，你不能正则化偏差项θ0。

&#160;&#160;&#160;&#160;相应地，正规化的逻辑回归代价函数对于θj的偏导数被定义为
<center><img src="https://note.youdao.com/yws/api/personal/file/WEBc97d6124facf475a2494ad6e7105acff?method=download&shareKey=5fcb8bbb8e9d9e798b3a9649646dcda8" /></center>

&#160;&#160;&#160;&#160;现在在lrCostFunction中修改代码以实现正则化。
同样，代码中不应该出现任何循环。

&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;*==你现在应该提交答案==*

### 1.4  One-vs-all分类器
&#160;&#160;&#160;&#160;在这部分练习中，你将通过训练多个规范化逻辑回归分类器来实现one-vs-all分类，每个规范化逻辑回归分类器对应于我们数据集中的K个类（图1）。在手写数字识别训练集中，K=10，但是你的代码应该接受任意值的K。

&#160;&#160;&#160;&#160;你现在应该完成oneVsAll.m中的代码为每个类训练一个分类器。特别是，你的代码应该在一个矩阵θ中返回所有分类器的参数，每一行Θ对应于学习到的逻辑回归参数的一个类。你可以使用一个从1到K的for循环来实现这一点，独立地训练每个分类器。

&#160;&#160;&#160;&#160;注意，这个函数的参数y是从1到10的标签向量，其中我们将数字“0”映射到标签10(以避免与索引混淆)。

&#160;&#160;&#160;&#160;当训练的分类器类k∈{1,…K}，会想要一个标签y的m维向量（yj∈0,1表）示第j训练实例是否属于类K(yj = 1)，或者如果它属于一个不同的类(yj = 0)。你可能会发现逻辑阵列有助于这项任务。

> **Ocvate/MATLAB技巧**：在Octave/MATLAB中的逻辑数组是包含二进制（0或1）的数组。在Octave/MATLAB中对于向量a（m * 1）和标量b计算a==b时，会返回一个和a大小相同的向量。该向量的元素值在a中的元素等于b的位置为1，否则为0。为了看清楚他们是如何工作的，在你的Octave/MATLAB环境中执行以下代码：
>```
>    a = 1:10; % Create a and b
>    b = 3;
>    a == b % You should try different values of b here
>```

&#160;&#160;&#160;&#160;此外，你将在本练习中使用fmincg（而不是fminunc）。
fmincg的工作原理与fminunc类似，但在处理大量参数时效率更高。

&#160;&#160;&#160;&#160;在你正确完成oneVsAll.m中的代码之后，ex3.m将调用oneVsAll函数来训练多类分类器。

&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;*==你现在应该提交答案==*

#### 1.4.1 One-vs-all预测
&#160;&#160;&#160;&#160;在训练了你的one-vs-all分类器之后，你可以使用它来预测给定图像中包含的数字。对于每个输入，你应该使用训练过的逻辑回归分类器计算它属于每个类的“概率”。您的one-vs-all预测函数将选择相应的逻辑回归分类器输出的概率最大的类，并返回类标签（1,2，…，或K）作为输入示例的预测。

&#160;&#160;&#160;&#160;你现在应该完成predictOneVsAll.m中的代码来做预测。
