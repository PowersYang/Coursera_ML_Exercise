# 程序设计练习2：逻辑回归

## 介绍
&#160;&#160;&#160;&#160;在这个练习中，你将实现逻辑回归并将其运用在两个不同的数据集上。在开始这个练习之前，我们强烈建议你观看视频讲座并完成相关主题的复习题。

&#160;&#160;&#160;&#160;要开始练习，您需要下载起始代码并将其内容解压缩到您希望完成练习的目录中。如果需要，请在开始本练习之前使用octave/matlab中的cd命令更改到此目录。

&#160;&#160;&#160;&#160;你也可以在课程网站的“环境设置说明”中找到安装octave/matlab的说明。

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
&#160;&#160;&#160;&#160;在整个练习中，您将使用ex2.m和ex2_reg.m脚本。这些脚本为后面的问题设置数据集，并调用你将要编写的函数。你无需修改它们两个，你只需按照本作业中的说明修改其他文件中的功能。

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
<center><img src="https://note.youdao.com/yws/api/personal/file/WEB807bb3ba0a9c9e798f7f01b64e76bd01?method=download&shareKey=d61f9296b8eab981df790c22d6eb213b" width="80%" /></center>
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

&#160;&#160;&#160;&#160;首先你需要在sigmoid.m中实现S函数，这样你才能在其他程序中调用它。完成之后在Octave/MATLAB命令行中用一些值通过调用sigmoid(x)来测试一下