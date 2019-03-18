# 程序设计练习6：支持向量机

## 介绍

&#160;&#160;&#160;&#160;在本练习中，你将实现支持向量机来构建一个垃圾邮件分类器。在开始这个练习之前，我们强烈建议你观看视频讲座并完成相关主题的复习问题。

&#160;&#160;&#160;&#160;要开始练习，您需要下载起始代码并将其内容解压缩到您希望完成练习的目录中。如果需要，请在开始本练习之前使用Octave/MATLAB中的cd命令更改到此目录。

&#160;&#160;&#160;&#160;你也可以在课程网站的“环境设置说明”中找到安装Octave/MATLAB的说明。

### 本练习中包含的文件

```
ex6.m - 前半部分练习的Octave/MATLAB脚本
ex6data1.mat - 样本集1
ex6data2.mat - 样本集2
ex6data3.mat - 样本集3
svmTrain.m - SVM训练函数
svmPredict.m - SVM预测函数
plotData.m - 绘制2维图
visualizeBoundaryLinear.m - 绘制线性边界
visualizeBoundary.m - 绘制非线性边界
linearKernel.m - SVM的线性内核
[*] gaussianKernel.m - 用于SVM的高斯核
[*] dataset3Params.m - 用于数据集3的参数

ex6 spam.m - 第二部分练习的Octave/MATLAB脚本
spamTrain.mat - 垃圾邮件训练集
spamTest.mat - 垃圾邮件测试集
emailSample1.txt - 样本邮件1
emailSample2.txt - 样本邮件2
spamSample1.txt - 垃圾邮件样本1
spamSample2.txt - 垃圾邮件样本2
vocab.txt - 词汇表
getVocabList.m - 加载词汇表
porterStemmer.m - 干扰函数
readFile.m - 将文件读入字符串
submit.m - 提交你的作业至我们的服务器
[*] processEmail.m - 邮件预处理
[*] emailFeatures.m - 从电子邮件中提取特征

* 表示你需要完成的文件
```

&#160;&#160;&#160;&#160;在整个练习中，你将使用脚本ex6.m。
这些脚本为题目设置数据集并调用你将要编写的函数。你不需要修改这两个脚本其中任何一个，只需要按照本作业中的说明修改其他文件中的函数。

### 在哪里寻求帮助

&#160;&#160;&#160;&#160;本课程的练习使用非常适合数值计算的高级编程语言Octave或MATLAB。如果你没有安装Octave或MATLAB，请参阅课程网站上“环境设置说明”中的安装说明。

&#160;&#160;&#160;&#160;在Octave/MATLAB命令行中输入help紧跟函数名称会显示内建的函数说明。比如输入help plot会显示绘图函数的帮助信息。更多Octave和MATLAB的函数说明请在[Octave官网](https://octave.org/doc/interpreter/)和[MATLAB官网](https://www.mathworks.com/help/matlab/?refresh=true)查阅。

&#160;&#160;&#160;&#160;我们也非常鼓励使用在线讨论与其他学生讨论练习。但是，不要查看任何源代码或与他人共享源代码。

------

## 1、支持向量机
&#160;&#160;&#160;&#160;在前半部分练习中，你将使用支持向量机（SVM）与各种2D样本数据集。使用这些数据集进行实验将帮助你直观地了解支持向量机如何工作，以及如何使用支持向量机的高斯内核。在接下来的练习中，你将使用支持向量机来构建垃圾邮件分类器。

&#160;&#160;&#160;&#160;提供的ex6.m脚本将会指导你完成前半部分练习。

&#160;&#160;&#160;&#160;

&#160;&#160;&#160;&#160;

&#160;&#160;&#160;&#160;

&#160;&#160;&#160;&#160;

&#160;&#160;&#160;&#160;

&#160;&#160;&#160;&#160;


<center><img src="https://note.youdao.com/yws/api/personal/file/WEB49bd742d8b49bb6587a8632ed164c099?method=download&shareKey=c85351059a7cd3ef3c488d91f6a3f4bf" width="80%" /></center>
<center><h6>Figure 1: Example Dataset 1</h6></center>

<center><img src="https://note.youdao.com/yws/api/personal/file/WEBadf7d009fa7b128473fe868a54651292?method=download&shareKey=c85351059a7cd3ef3c488d91f6a3f4bf" width="80%" /></center>
<center><h6>Figure 2: SVM Decision Boundary with C = 1 (Example Dataset 1)</h6></center>

<center><img src="https://note.youdao.com/yws/api/personal/file/WEB4f9b6c4c091a42a03a049e27c1e9c451?method=download&shareKey=c85351059a7cd3ef3c488d91f6a3f4bf" width="80%" /></center>
<center><h6>Figure 3: SVM Decision Boundary with C = 100 (Example Dataset 1)
</h6></center>

<center><img src="https://note.youdao.com/yws/api/personal/file/WEBa2a10441c6cb30f282d813ca18db7376?method=download&shareKey=c85351059a7cd3ef3c488d91f6a3f4bf" width="80%" /></center>
<center><h6>Figure 4: Example Dataset 2</h6></center>

<center><img src="https://note.youdao.com/yws/api/personal/file/WEBd421f0ecb789444fc201ce5018b15e71?method=download&shareKey=c85351059a7cd3ef3c488d91f6a3f4bf" width="80%" /></center>
<center><h6>Figure 5: SVM (Gaussian Kernel) Decision Boundary (Example Dataset 2)</h6></center>

<center><img src="https://note.youdao.com/yws/api/personal/file/WEB18d7e5ef1bb13723b649e142fa62c427?method=download&shareKey=c85351059a7cd3ef3c488d91f6a3f4bf" width="80%" /></center>
<center><h6>Figure 6: Example Dataset 3</h6></center>

<center><img src="https://note.youdao.com/yws/api/personal/file/WEB4dd71753981af9993a6224d66594fc90?method=download&shareKey=c85351059a7cd3ef3c488d91f6a3f4bf" width="80%" /></center>
<center><h6>Figure 7: SVM (Gaussian Kernel) Decision Boundary (Example Dataset 3)</h6></center>

<center><img src="https://note.youdao.com/yws/api/personal/file/WEBc633ba434d7d449b4a1cb55a3761b4ed?method=download&shareKey=c85351059a7cd3ef3c488d91f6a3f4bf" /></center>
<center><h6>Figure 8: Sample Email</h6></center>

<center><img src="https://note.youdao.com/yws/api/personal/file/WEBed55dd8e6bf04a2dd2409fa2f8cd527e?method=download&shareKey=c85351059a7cd3ef3c488d91f6a3f4bf" /></center>
<center><h6>Figure 9: Preprocessed Sample Email</h6></center>

<center><img src="https://note.youdao.com/yws/api/personal/file/WEBc99b67e5a992cc188e499e45dd5b2ee2?method=download&shareKey=c85351059a7cd3ef3c488d91f6a3f4bf" width="20%" /></center>
<center><h6>Figure 10: Vocabulary List</h6></center>

<center><img src="https://note.youdao.com/yws/api/personal/file/WEBab50cad52ada84206c62afc65422a73a?method=download&shareKey=c85351059a7cd3ef3c488d91f6a3f4bf" width="30%" /></center>
<center><h6>Figure 11: Word Indices for Sample Email</h6></center>

<center><img src="https://note.youdao.com/yws/api/personal/file/WEBa05d0a2275468327e4165559394fbee1?method=download&shareKey=c85351059a7cd3ef3c488d91f6a3f4bf" /></center>
<center><h6>Figure 12: Top predictors for spam email</h6></center>

<center><img src="https://note.youdao.com/yws/api/personal/file/WEB5069725937c51593b620df8c4eb0d4bc?method=download&shareKey=c85351059a7cd3ef3c488d91f6a3f4bf" /></center>

&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;*==你现在应该提交答案==*
