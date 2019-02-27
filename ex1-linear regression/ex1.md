# 程序设计练习1: 线性回归

机器学习

## 介绍
&#160;&#160;&#160;&#160;在本练习中，你将实现线性回归，并明确它对数据的作用。在开始这个练习之前，我们强烈建议你观看视频讲座并完成相关主题的复习问题。

&#160;&#160;&#160;&#160;要开始练习，您需要下载起始代码并将其内容解压缩到您希望完成练习的目录中。如果需要，请在开始本练习之前使用octave/matlab中的cd命令更改到此目录。

&#160;&#160;&#160;&#160;你也可以在课程网站的“环境设置说明”中找到安装octave/matlab的说明。

### 本练习中包含的文件
    ex1.m - 指导你完成练习的 Octave/MATLAB 脚本
    ex1_multi.m - 练习的后面部分 Octave/MATLAB 脚本
    ex1data1.txt - 单变量线性回归数据集
    ex1data2.txt - 多变量线性回归数据集
    submit.m - 将你的答案发送到我们服务器的提交脚本
    
    [*] warmUpExercise.m -  Octave/MATLAB 中的简单示例函数
    [*] plotData.m - 展示数据集的函数
    [*] computeCost.m - 线性回归代价函数
    [*] gradientDescent.m - 梯度下降函数
    [†] computeCostMulti.m - 多变量代价函数
    [†] gradientDescentMulti.m - 多变量梯度下降
    [†] featureNormalize.m - 用于规范化功能的函数
    [†] normalEqn.m - 计算正规方程的函数
    
    * 标示必须完成的文件
    † 标示可选练习

&#160;&#160;&#160;&#160;在整个练习中，你将使用脚本ex1.m和ex1_multi.m。
这些脚本为题目设置数据集并调用你将要编写的函数。你不需要修改这两个脚本其中任何一个，只需要按照本作业中的说明修改其他文件中的函数。

&#160;&#160;&#160;&#160;对于这个编程练习你只需要完成第一部分练习实现单变量线性回归，第二部分的多变量线性回归是可选的。

### 在哪里寻求帮助
&#160;&#160;&#160;&#160;本课程的练习使用非常适合数值计算的高级编程语言Octave或MATLAB。如果你没有安装Octave或MATLAB，请参阅课程网站上“环境设置说明”中的安装说明。

&#160;&#160;&#160;&#160;在Octave/MATLAB命令行中输入help紧跟函数名称会显示内建的函数说明。比如输入help plot会显示绘图函数的帮助信息。更多Octave和MATLAB的函数说明请在[Octave官网](https://octave.org/doc/interpreter/)和[MATLAB官网](https://www.mathworks.com/help/matlab/?refresh=true)查阅。

&#160;&#160;&#160;&#160;我们也非常鼓励使用在线讨论与其他学生讨论练习。但是，不要查看任何源代码或与他人共享源代码。

---



## 1、简单的Octave/MATLAB函数
&#160;&#160;&#160;&#160;ex1.m的第一部分给出了使用Octave/matlab的语法和作业提交流程。在warmupexercise.m文件中，有一个Octave/matlab函数的waixing。通过填充以下代码修将其修改为返回5 x 5单位矩阵的函数：

```
    A = eye(5);
``` 
&#160;&#160;&#160;&#160;当你完成后，运行（假设你在正确的目录中，
在Octave/matlab的提示下键入“ex1”）ex1.m文件，应该会看到类似下面的输出：

```
    ans =
    Diagonal Matrix
        1 0 0 0 0
        0 1 0 0 0
        0 0 1 0 0
        0 0 0 1 0
        0 0 0 0 1
```
&#160;&#160;&#160;&#160;现在，ex1.m将暂停，直你按下其他键，然后运行下一任务的代码。如果要退出，键入ctrl-c将在运行过程中终止程序。


### 1.1 提交答案
&#160;&#160;&#160;&#160;完成部分练习后，你可以提交答案并在Octave/matlab命令行中键入submit进行评分。提交脚本将提示你输入登录的e-mail和提交的token，并询问你要提交哪些文件。你可以网页中获得本次作业的token。


&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;*==你现在应该提交答案==*
    
&#160;&#160;&#160;&#160;你可以多次提交答案，我们只考虑最高分数。


---

## 2、单变量线性回归
&#160;&#160;&#160;&#160;在这部分练习中，你将实现单变量线性回归来预测快餐车的利润问题。假设你是一家餐厅的CEO并且正在考虑在不同城市开设分店的问题。该连锁店在早已经其他城市有快餐车并且你有各个城市的利润数据和人口数据。
你希望使用此数据帮助选择下一步要扩展的城市。

&#160;&#160;&#160;&#160;文件ex1data1.txt包含线性回归问题的数据集。第一列是一个城市的人口，第二列是该城市快餐车的利润。利润负值表示损失。

&#160;&#160;&#160;&#160;已经设置了ex1.m来为你加载数据。

### 2.1 绘制数据
&#160;&#160;&#160;&#160;在开始任何任务之前，通过可视化来理解数据通常是很有用的。对于此数据集，你可以使用散点图来可视化数据，因为它只有两个属性需要绘制(利润和人口)。(许多在现实生活中遇到的其他问题是多维的，不能在二维平面上绘制)
在ex1.m中，将数据集从数据文件加载到变量 *X* 和 *y* 中:

```
    data = load('ex1data1.txt'); % read comma separated data
    X = data(:, 1); y = data(:, 2);
    m = length(y); % number of training examples
```
&#160;&#160;&#160;&#160;接下来，脚本调用plotData函数来创建这些数据的散点图。你的任务是完成plotDat.m来绘图;修改文件并填写以下代码:
```
    plot(x, y, 'rx', 'MarkerSize', 10); % Plot the data
    ylabel('Profit in $10,000s'); % Set the y−axis label
    xlabel('Population of City in 10,000s'); % Set the x−axis label
```
&#160;&#160;&#160;&#160;现在，当你继续运行ex1.m，我们的最终结果应该如图1所示，有红色“x”标记和轴标记。
要了解plot命令的更多信息，你可以在Octave/MATLAB命令提示符中键入help plot，或者在线搜索绘图文档。
(为了将标记改为红色的“x”，我们在plot命令中使用了选项“rx”);

<center><img src="https://note.youdao.com/yws/api/personal/file/WEB461db5dd63fdd146e8b8a79d814e9f74?method=download&shareKey=51aa011fd75b26282d6d39cc61266093" height="70%" width="70%" /></center>
<center><h6>Figure 1: Scatter plot of training data</h6></center>


### 2.2 梯度下降
&#160;&#160;&#160;&#160;在这部分中，你将使用梯度下降法拟合线性回归中的参数θ到我们的数据集。

#### 2.2.1 更新方程
&#160;&#160;&#160;&#160;线性回归的目标是使代价函数最小化
<center><img src="https://note.youdao.com/yws/api/personal/file/WEB5f64736a9b0e0a9847ec62d23fa8f454?method=download&shareKey=826705049b876cf55392692b504ac8ea" /></center>

&#160;&#160;&#160;&#160;该假设函数的线性模型如下
<center><img src="https://note.youdao.com/yws/api/personal/file/WEBbaee3a75b7a54cac6fb3e9fd332e1405?method=download&shareKey=efb91f6f9eb4cfea6a2b04a48b0ec1db" /></center>

&#160;&#160;&#160;&#160;回想一下，模型的参数是θj，你将通过调整它来最小化代价函数J(θ)。一种方法是使用批梯度下降算法，批量梯度下降每次都迭代更新下面的式子：
<center><img src="https://note.youdao.com/yws/api/personal/file/WEB48c7754e3c7006472f5b615b013092e5?method=download&shareKey=1988d7dc7f508040c97c659bd775b97b" /></center>
&#160;&#160;&#160;&#160;每一步的梯度下降,参数θj都会更加接近使代价函数J(θ)最低的最优值。

###
>**注意**:我们将每个样本作为一行存储在Octave/MATLAB中的X矩阵中。考虑到截距项(θ0),我们在X上增加了额外的第一列，并把它的所有值设为1。这样我们就可以把θ0当做另一个简单“特征”。


#### 2.2.2 实现
在ex1.m中我们已经建立了线性回归的数据。接下来，我们向数据中添加第一列（全部为1）以适应θ0截距项。我们还将初始化参数，使为θ为0，学习率为0.01。

```
    X = [ones(m, 1), data(:,1)]; % Add a column of ones to x
    theta = zeros(2, 1); % initialize fitting parameters
    iterations = 1500;
    alpha = 0.01;
```


#### 2.2.3 计算代价函数
