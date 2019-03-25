# 程序设计练习7：k均值聚类与主成分分析

## 介绍

&#160;&#160;&#160;&#160;在本练习中，你将实现K均值聚类算法并将其应用于压缩图像。在第二部分中，你将使用主成分分析来寻找人脸图像的低维表示。在开始编程练习之前，我们强烈建议观看视频课程并完成相关主题的复习问题。

&#160;&#160;&#160;&#160;要开始练习，您需要下载起始代码并将其内容解压缩到您希望完成练习的目录中。如果需要，请在开始本练习之前使用Octave/MATLAB中的cd命令更改到此目录。

&#160;&#160;&#160;&#160;你也可以在课程网站的“环境设置说明”中找到安装Octave/MATLAB的说明。

### 本练习中包含的文件

```
ex7.m - Octave/MATLAB script for the first exercise on K-means
ex7 pca.m - Octave/MATLAB script for the second exercise on PCA
ex7data1.mat - Example Dataset for PCA
ex7data2.mat - Example Dataset for K-means
ex7faces.mat - Faces Dataset
bird small.png - Example Image
displayData.m - Displays 2D data stored in a matrix
drawLine.m - Draws a line over an exsiting figure
plotDataPoints.m - Initialization for K-means centroids
plotProgresskMeans.m - Plots each step of K-means as it proceeds
runkMeans.m - Runs the K-means algorithm
submit.m - Submission script that sends your solutions to our servers
[*] pca.m - Perform principal component analysis
[*] projectData.m - Projects a data set into a lower dimensional space
[*] recoverData.m - Recovers the original data from the projection
[*] findClosestCentroids.m - Find closest centroids (used in K-means)
[*] computeCentroids.m - Compute centroid means (used in K-means)
[*] kMeansInitCentroids.m - Initialization for K-means centroids

* 表示必须完成的文件
```

&#160;&#160;&#160;&#160;在第一部分练习中，你将使用脚本ex7.m，第二部分你将使用ex7_pca.m脚本。这些脚本为题目设置数据集并调用你将要编写的函数。你不需要修改这两个脚本其中任何一个，只需要按照本作业中的说明修改其他文件中的函数。

&#160;&#160;&#160;&#160;对于这个编程练习你只需要完成第一部分练习实现单变量线性回归，第二部分的多变量线性回归是可选的。

### 在哪里寻求帮助

&#160;&#160;&#160;&#160;本课程的练习使用非常适合数值计算的高级编程语言Octave或MATLAB。如果你没有安装Octave或MATLAB，请参阅课程网站上“环境设置说明”中的安装说明。

&#160;&#160;&#160;&#160;在Octave/MATLAB命令行中输入help紧跟函数名称会显示内建的函数说明。比如输入help plot会显示绘图函数的帮助信息。更多Octave和MATLAB的函数说明请在[Octave官网](https://octave.org/doc/interpreter/)和[MATLAB官网](https://www.mathworks.com/help/matlab/?refresh=true)查阅。

&#160;&#160;&#160;&#160;我们也非常鼓励使用在线讨论与其他学生讨论练习。但是，不要查看任何源代码或与他人共享源代码。

------


## 1、K均值聚类
&#160;&#160;&#160;&#160;在这个练习中，你将实现K均值算法并将它用于图像压缩。你将首先从一个2D样本数据集开始，它将帮助你直观了解K-means算法的工作原理。之后，你将使用K-means算法进行图像压缩，方法是将图像中出现的颜色数量减少到该图像中最常见的颜色数量。 你将使用ex7.m进行这部分练习。

### 1.1 实现K-means
&#160;&#160;&#160;&#160;K-means算法是一种自动将相似的数据样本聚在一起的方法。具体地说，你有一个训练集`$\{x^{(1)},...,x^{(m)}\}$`，（其中`$x^{(i)} ∈R^n$`），并想将数据分为几个内聚的“簇”。它背后的知识是猜测一个初始化中心点开始，然后反复将样本分配到最接近的聚类中心，然后根据已经分配的结果重新计算聚类中心来改进这个猜测。

&#160;&#160;&#160;&#160;K-means算法如下：
```
    % Initialize centroids
    centroids = kMeansInitCentroids(X, K);
    for iter = 1:iterations
        % Cluster assignment step: Assign each data point to the
        % closest centroid. idx(i) corresponds to cˆ(i), the index
        % of the centroid assigned to example i
        idx = findClosestCentroids(X, centroids);
        % Move centroid step: Compute means based on centroid
        % assignments
        centroids = computeMeans(X, idx, K);
    end
```

&#160;&#160;&#160;&#160;该算法的内环重复执行两个步骤:(i)将每个训练实例`$x^{(i)}$`分配到其最近的聚类中心，(ii)使用分配给它的点重新计算每个聚类中心的均值。K-means算法总是收敛到中心点的最终均值集。注意，收敛解可能并不总是理想的，它依赖于聚类中心的初始设置。因此，在实际操作中，K-means算法通常会运行几次，并具有不同的随机初始化。从不同的随机初始化中选择这些不同解的一种方法是选择成本函数值（失真）最低的解。

&#160;&#160;&#160;&#160;在下一节中，你将分别实现K-means算法的两个阶段。

#### 1.1.1 寻找最近的聚类中心
&#160;&#160;&#160;&#160;在K-means算法的“集群分配”阶段，给定当前聚类中心的位置，该算法将每个训练样本`$x^{(i)}$`分配到其最近的中心。具体来说，对于我们设置的每个样本i设置`$c^{(i)}:= j$`，然后最小化`$||x^{(i)} − µ_j||^2$`，其中`$c^{(i)}$`是最靠近`$x^{(i)}$`的中心下标，`$µ_j$`是第j个聚类中心的坐标值。注意，`$c^{(i)}$`对应于启动代码中的idx(i)。

&#160;&#160;&#160;&#160;你的任务是完成findClosestCentroids.m中的代码。该函数接受数据矩阵X和聚类中心内所有中心的位置，并应输出一个一维数组idx，其中包含索引（值在{1，…， K}，其中K为距离每个训练样本最近中心的总数）。

&#160;&#160;&#160;&#160;你可以通过每个训练样本和每个中心的循环来达到此目的。

&#160;&#160;&#160;&#160;一旦你完成了findClosestCentroids.m中的代码，脚本ex7.m将调用你的代码，应该看到与前3个样本的中心分配对应的输出[1 3 2]。

&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;*==你现在应该提交答案==*
 
#### 1.1.2 计算聚类中心均值
&#160;&#160;&#160;&#160;给定每个中心点的赋值，算法的第二阶段对每个中心点重新计算赋值点的均值。具体地说，对于每个中心点k我们设置
```math
µ_k := \frac{1}{|C_k|}\sum_{i∈C_k}x^{(i)}
```
其中`$C_k$`是分配给中心k的一组样本。具体地说，如果两个样本`$x^{(3)}$`和`$x^{(5)}$`被分配给中心 k = 2，那么你应该更新 `$µ_2 := \frac{1}{2}(x^{(3)} + x^{(5)})$`。

&#160;&#160;&#160;&#160;现在你应该在computeCentroids.m完成代码。你可以使用中心上的循环来实现这个函数。你还可以对样本使用循环;但是，如果你可以通过不使用这种循环的向量化实现，你的代码可能运行得更快。

&#160;&#160;&#160;&#160;完成computeCentroids.m中的代码后，脚本ex7.m将运行你的代码并在K-means的第一步之后输出中心。

&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;*==你现在应该提交答案==*
 
### 1.2 样本集上的K-means
&#160;&#160;&#160;&#160;在完成两个函数（findClosestCentroids和computeCentroids）之后，ex7.m中的下一步将在2D数据集上运行K-means算法，以帮助你了解K-means的工作原理。 你的函数是从runKmeans.m脚本中调用的。我们鼓励你查看函数以了解其工作原理。请注意，代码调用你在循环中实现的两个函数。

&#160;&#160;&#160;&#160;当你运行下一个步骤时，K-means代码将生成一个可视化图像，在每次迭代中引导你了解算法的过程。多次按enter键，查看K-means算法的每个步骤如何更改中心和集群分配。最后，你的图形应该如图1所示。

<center><img src="https://note.youdao.com/yws/api/personal/file/WEB64bbf251fc27bb9ed5fb74e798250b7b?method=download&shareKey=52016790c8af4ecc02ee1fe02a9e764d" /></center>
<center><h6>Figure 1: The expected output.</h6></center>

### 1.3 随机初始化
&#160;&#160;&#160;&#160;设计ex7.m样本数据集的聚类中心的初始分配，以便你将看到与图1中相同的图。实际上，初始化质心的一个好策略是从训练集中选择随机样本。

&#160;&#160;&#160;&#160;在本练习的这一部分中，你应该使用以下代码完成函数kMeansInitCentroids.m：

```
    % Initialize the centroids to be random examples
    % Randomly reorder the indices of examples
    randidx = randperm(size(X, 1));
    % Take the first K examples as centroids
    centroids = X(randidx(1:K), :);
```

&#160;&#160;&#160;&#160;上面的代码首先随机遍历示例的索引(使用randperm)。然后，根据指标的随机排列选择前K个样本。这允许随机选择示例，而不用冒两次选择相同示例的风险。

&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;*==这部分练习不需要做任何提交==*
 
### 1.4 用K-means进行图像压缩

<center><img src="https://note.youdao.com/yws/api/personal/file/WEBb7a4ca7f3364420fd69f7bab60c0e38f?method=download&shareKey=ab22fb0291dd09ab7f32241acd26dbfb" /></center>
<center><h6>Figure 2: The original 128x128 image.</h6></center>

&#160;&#160;&#160;&#160;在这个练习中，你会将K-means应用到图像压缩上。在直观的24位彩色图像表示中，每个像素表示为三个8位无符号整数(范围从0到255)，指定红、绿和蓝强度值。这种编码通常称为RGB编码。我们的图像包含成千上万种颜色，在这部分练习中，你将把颜色的数量减少到16种颜色。

&#160;&#160;&#160;&#160;通过进行这种缩小，可以以有效的方式表示（压缩）照片。具体来说，你只需要存储16种所选颜色的RGB值，并且对于图像中的每个像素，你现在只需要在该位置存储颜色的索引（其中只需要4位来表示16种可能性）。
通过进行这种缩小，可以以有效的方式表示（压缩）照片。 具体来说，你只需要存储16种所选颜色的RGB值，并且对于图像中的每个像素，你现在只需要在该位置存储颜色的索引（其中只需要4位来表示16种可能性）。

&#160;&#160;&#160;&#160;在本练习中，你将使用K-means算法来选择将用于表示压缩图像的16种颜色。具体来说，你将把原始图像中的每个像素作为一个数据样本，并使用K-means算法找到在三维RGB空间中对像素进行最佳分组（集群）的16种颜色。一旦你计算了图像上的集群中心，你将使用16种颜色替换原始图像中的像素。


#### 1.4.1 像素上的K-means
&#160;&#160;&#160;&#160;在Octave/MATLAB中，图片可以用下面的方式读取：

```
    % Load 128x128 color image (bird small.png)
    A = imread('bird small.png');
    % You will need to have installed the image package to used
    % imread. If you do not have the image package installed, you
    % should instead change the following line to
    %
    % load('bird small.mat'); % Loads the image into the variable A
```

&#160;&#160;&#160;&#160;这将创建一个三维矩阵A，其前两个索引标识一个像素位置，其最后一个索引代表红色，绿色或蓝色。 例如，A(50,33,3)给出第50行第33列的像素的蓝色强度。

&#160;&#160;&#160;&#160;ex7.m中的代码首先加载图像，然后将其再改造以创建m×3像素颜色矩阵（其中m=16384=128×128），并在其上调用K-means函数。

&#160;&#160;&#160;&#160;

&#160;&#160;&#160;&#160;

&#160;&#160;&#160;&#160;

&#160;&#160;&#160;&#160;

&#160;&#160;&#160;&#160;

&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;*==你现在应该提交答案==*
 
