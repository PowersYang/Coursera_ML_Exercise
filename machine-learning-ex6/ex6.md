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

### 1.1 样本数据集1
&#160;&#160;&#160;&#160;我们将从一个2D示例数据集开始，该数据集可以由线性边界分隔。 脚本ex6.m将绘制训练数据（图1）。 在该数据集中，正例（用+表示）和负例（用o表示）的位置暗示了由间隙表示的自然分离。 但是，请注意在最左边有一个异常值正例+大约（0.1,4.1）。 作为本练习的一部分，你还将了解此异常值如何影响SVM决策边界。

<center><img src="https://note.youdao.com/yws/api/personal/file/WEB49bd742d8b49bb6587a8632ed164c099?method=download&shareKey=10fc7067262d2b19947e9236fc9d2e76" width="80%" /></center>
<center><h6>Figure 1: Example Dataset 1</h6></center>

&#160;&#160;&#160;&#160;在练习的这一部分中，你将尝试使用SVMs中不同的C参数值。非正式地说，C参数是一个正值，它控制了对错误分类的训练示例的惩罚。一个大的C参数告诉SVM尝试正确地对所有示例进行分类。 C起着类似于`$\frac{1}{λ}$`的作用，其中λ是我们之前用于逻辑回归的正则化参数。

&#160;&#160;&#160;&#160;ex6.m的下一部分将使用我们已经给出的代码svmTrain.m运行SVM训练（C = 1）。
当C = 1时，你应该发现SVM将决策边界置于两个数据集之间的间隙中，并对最左侧的数据点进行错误分类（图2）。

<center><img src="https://note.youdao.com/yws/api/personal/file/WEBadf7d009fa7b128473fe868a54651292?method=download&shareKey=c85351059a7cd3ef3c488d91f6a3f4bf" width="80%" /></center>
<center><h6>Figure 2: SVM Decision Boundary with C = 1 (Example Dataset 1)</h6></center>

> **实现注意**：大多数SVM软件包（包括svmTrain.m）会自动为你添加额外的特征`$x_0 = 1$`，并自动负责学习截距项`$θ_0$`。 因此，在将训练数据传递给SVM软件时，无需自己添加此额外特征`$x_0 = 1$`。 特别是，在Octave / MATLAB中，你的代码应该使用训练样例`$x∈R^n$`（而不是`$x∈R^{n+1}$`）; 例如，在第一个示例中，数据集`$x∈R^2$`。

&#160;&#160;&#160;&#160;你的任务是在这个数据集中尝试不同的C值。具体来说，你应该将脚本中的C值更改为C = 100并再次运行SVM训练。当C = 100时，你应该会发现SVM现在正确地分类了每个示例，但是它的决策边界似乎不是很好地拟合数据(图3)。

<center><img src="https://note.youdao.com/yws/api/personal/file/WEB4f9b6c4c091a42a03a049e27c1e9c451?method=download&shareKey=c85351059a7cd3ef3c488d91f6a3f4bf" width="80%" /></center>
<center><h6>Figure 3: SVM Decision Boundary with C = 100 (Example Dataset 1)
</h6></center>

### 1.2 高斯核支持向量机
&#160;&#160;&#160;&#160;在本练习的这一部分中，你将使用SVM进行非线性分类。
特别是，你将在不可线性分离的数据集上使用具有高斯内核的SVM。

#### 1.2.1 高斯核
&#160;&#160;&#160;&#160;为了利用支持向量机找到非线性决策边界，首先需要实现高斯核函数。你可以把高斯核想象成一个相似函数，它度量两个样本`$(x^{(i)},x^{(j)})$`之间的“距离”。高斯内核也由带宽参数，参数化σ，决定相似性度量以多快的速度减少（到0）为例进一步分开。

&#160;&#160;&#160;&#160;你现在应该完成gaussianKernel.m中的代码来计算两个样本`$(x^{(i)},x^{(j)})$`之间的高斯核。高斯核的定义如下：
<center><img src="https://note.youdao.com/yws/api/personal/file/WEBde4f3556cb59d87c4bb31d0620e68989?method=download&shareKey=6bc28d994f57cb5fc8ef380528429839" /></center>


&#160;&#160;&#160;&#160;一旦你完成gaussianKernel.m函数之后，ex6.m脚本会在两个提供的样本上测试你的核函数，并且你看到的值应该是0.324652。

&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;*==你现在应该提交答案==*

#### 1.2.2 样本数据集2
&#160;&#160;&#160;&#160;ex6.m的下一部分将加载并绘制数据集2（图4）。 从图中可以看出，没有线性决策边界来区分此数据集的正负样本。 但是，通过将SVM与高斯核一起使用，你将能够学习非线性决策边界，该边界在数据及上会表现地很好。

<center><img src="https://note.youdao.com/yws/api/personal/file/WEBa2a10441c6cb30f282d813ca18db7376?method=download&shareKey=c85351059a7cd3ef3c488d91f6a3f4bf" width="80%" /></center>
<center><h6>Figure 4: Example Dataset 2</h6></center>

&#160;&#160;&#160;&#160;如果你正确地实现了高斯核函数，ex6.m将在此数据集中使用高斯核训练SVM。

&#160;&#160;&#160;&#160;图5为高斯核支持向量机的决策边界。该决策边界能够正确地分离出大部分正、负样本，并能很好地跟踪数据集的轮廓。

<center><img src="https://note.youdao.com/yws/api/personal/file/WEBd421f0ecb789444fc201ce5018b15e71?method=download&shareKey=c85351059a7cd3ef3c488d91f6a3f4bf" width="80%" /></center>
<center><h6>Figure 5: SVM (Gaussian Kernel) Decision Boundary (Example Dataset 2)</h6></center>

#### 1.2.3 样本数据集3
&#160;&#160;&#160;&#160;在本练习的这一部分中，你将获得有关如何使用具有高斯内核的SVM的更多实用技巧。ex6.m的下一部分将加载并显示第三个数据集（图6）。 你将使用带有此数据集的高斯内核的SVM。

<center><img src="https://note.youdao.com/yws/api/personal/file/WEB18d7e5ef1bb13723b649e142fa62c427?method=download&shareKey=c85351059a7cd3ef3c488d91f6a3f4bf" width="80%" /></center>
<center><h6>Figure 6: Example Dataset 3</h6></center>

&#160;&#160;&#160;&#160;在提供的数据集ex6data3.mat中，您将获得变量X，y，Xval，yval。ex6.m中提供的代码使用从dataset3Params.m加载的参数，使用训练集（X，y）训练SVM分类器。

&#160;&#160;&#160;&#160;你的任务是使用交叉验证集Xval，yval来确定要使用的最佳C和σ参数。你应该编写必要的其他代码来帮助你探索参数C和σ。 对于C和σ，我们建议在乘法步骤中尝试值（例如，0.01，0.03，0.1，0.3，1，3，10，30）。 请注意，您应该尝试C和σ的所有可能的值对（例如，C = 0.3和σ= 0.1）。 例如，如果您尝试上面列出的C和`$σ^2$`的8个值中的每一个，您最终将训练和评估（在交叉验证集上）总共`$8^2$` = 64个不同的模型。

&#160;&#160;&#160;&#160;确定要使用的最佳C和σ参数后，应修改dataset3Params.m中的代码，填写找到的最佳参数。 对于我们的最佳参数，SVM返回了如图7所示的决策边界。

<center><img src="https://note.youdao.com/yws/api/personal/file/WEB4dd71753981af9993a6224d66594fc90?method=download&shareKey=c85351059a7cd3ef3c488d91f6a3f4bf" width="80%" /></center>
<center><h6>Figure 7: SVM (Gaussian Kernel) Decision Boundary (Example Dataset 3)</h6></center>

> **实现注意**：在实现交叉验证以选择要使用的最佳C和σ参数时，需要评估交叉验证集上的误差。回想一下，对于分类问题，误差被定义为错误分类的交叉验证样本比例。在Octave / MATLAB中，您可以使用mean(double(predictions ~= yval))计算此误差，其中predictions是包含来自SVM的所有预测的向量，yval是来自交叉验证集的真实标签。您可以使用svmPredict函数生成交叉验证集的预测。

&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;*==你现在应该提交答案==*

## 2、垃圾邮件分类
&#160;&#160;&#160;&#160;如今，许多电子邮件服务提供垃圾邮件过滤器，能够将电子邮件精确地分类为垃圾邮件和非垃圾邮件。在本部分练习中，你将使用SVMs构建自己的垃圾邮件过滤器。

&#160;&#160;&#160;&#160;你将训练一个分类器以分类给定的电子邮件x是垃圾邮件（y = 1）还是非垃圾邮件（y = 0）。特别是，你需要将每封电子邮件转换为特征向量`$x∈R^n$`。练习的以下部分将引导你了解如何通过电子邮件构建此类特征向量。

&#160;&#160;&#160;&#160;在本练习的其余部分中，你将使用脚本ex6_spam.m。 本练习中包含的数据集基于SpamAssassin Public Corpus的一个子集。出于本练习的目的，你将只使用电子邮件的正文（不包括电子邮件标题）。

### 2.1 预处理邮件
&#160;&#160;&#160;&#160;在开始机器学习任务之前，从数据集中查看示例通常是很有洞察力的。图8显示了一个示例电子邮件，其中包含URL、电子邮件地址(末尾)、数字和金额。虽然许多电子邮件包含类似类型的实体(例如，数字、其他URL或其他电子邮件地址)，但几乎每封电子邮件中的特定实体(例如，特定URL或特定金额)都是不同的。因此，处理电子邮件时经常使用的一种方法是“规范化”这些值，以便所有url都被处理相同，所有数字都被处理相同，等等。例如，我们可以用惟一的字符串“httpaddr”替换电子邮件中的每个URL，以表示存在一个URL。这样做的效果是让垃圾邮件分类器根据是否存在某个URL而不是某个特定的URL来进行分类决策。这通常会提高垃圾邮件分类器的性能，因为垃圾邮件发送者经常随机分配URL，因此在新垃圾邮件中再次看到特定URL的几率非常小。

<center><img src="https://note.youdao.com/yws/api/personal/file/WEBc633ba434d7d449b4a1cb55a3761b4ed?method=download&shareKey=c85351059a7cd3ef3c488d91f6a3f4bf" /></center>
<center><h6>Figure 8: Sample Email</h6></center>

&#160;&#160;&#160;&#160;在processEmail.m中，我们实现了以下电子邮件预处理和规范化步骤：
> * **字母小写**：整个电子邮件被转换成小写字母，这样就忽略了大写（例如，IndIcaTE被当作Indicate对待）。
> * **删除HTML**：所有HTML标记都将从电子邮件中删除。许多电子邮件通常都带有HTML格式; 我们删除所有HTML标记，以便只保留内容。
> * **规范URL**：所有的URL会被替换成文本“httpaddr”。
> * **规范邮件地址**：所有的邮件地址都被替换成文本“emailaddr”。
> * **规范数字**：所有的数字都被替换成文本“number”。
> * **规范美元**：所有的美元符号（$）都被替换成文本“dollar”。
> * **词干提取**：单词被简化成词根形式。例如，“discount”，“discounts”，“discounted”和“discounting”都被“discount”所代替。有时候实际上是从末尾删除了额外的字符，所以“include”、“includes”、“included”和“including”都被替换为“includ”。
> * **删除非单词**：非文字和标点符号已被删除。所有空白(制表符、换行符、空格)都已被修剪为一个空格字符。

&#160;&#160;&#160;&#160;这些预处理步骤的结果如图9所示。虽然预处理留下了单词片段和非单词，但事实证明，使用这种形式执行特征提取要容易得多。

<center><img src="https://note.youdao.com/yws/api/personal/file/WEBed55dd8e6bf04a2dd2409fa2f8cd527e?method=download&shareKey=c85351059a7cd3ef3c488d91f6a3f4bf" /></center>
<center><h6>Figure 9: Preprocessed Sample Email</h6></center>

#### 2.1.1 词汇列表
&#160;&#160;&#160;&#160;在对电子邮件进行预处理之后，我们将为每个电子邮件创建一个单词列表(例如，图9)。下一步是选择我们想要在分类器中使用的单词和我们想要省略的单词。

&#160;&#160;&#160;&#160;对于本练习，我们仅选择最常出现的单词作为我们考虑的单词集（词汇表）。由于训练集中很少出现的单词仅在几封电子邮件中，因此可能会导致模型过度训练我们的训练集。完整的词汇表列在文件vocab.txt中，如图10所示。我们的词汇表是通过选择在垃圾邮件语料库中至少出现100次的所有单词来选择的，从而得到1899个单词的列表。 在实践中，经常使用具有大约10,000到50,000个单词的词汇表。

<center><img src="https://note.youdao.com/yws/api/personal/file/WEBc99b67e5a992cc188e499e45dd5b2ee2?method=download&shareKey=c85351059a7cd3ef3c488d91f6a3f4bf" width="20%" /></center>
<center><h6>Figure 10: Vocabulary List</h6></center>

&#160;&#160;&#160;&#160;给定词汇表，我们现在可以将预处理的电子邮件（例如，图9）中的每个单词映射到包含词汇表中单词的索引的单词索引列表。 图11显示了示例电子邮件的映射。具体而言，在示例电子邮件中，单词“everyone”首先被标准化为“anyon”，然后映射到词汇表列表中的索引86。

<center><img src="https://note.youdao.com/yws/api/personal/file/WEBab50cad52ada84206c62afc65422a73a?method=download&shareKey=c85351059a7cd3ef3c488d91f6a3f4bf" width="30%" /></center>
<center><h6>Figure 11: Word Indices for Sample Email</h6></center>

&#160;&#160;&#160;&#160;你现在的任务是完成processEmail.m中的代码以执行此映射。在代码中，你将获得一个字符串str，它是已处理电子邮件中的单个单词。你应该在词汇表vocabList中查找单词，并查找词汇表中是否存在该单词。如果单词存在，则应将单词的索引添加到单词indices变量中。 如果单词不存在，因此不在词汇表中，则可以跳过单词。

&#160;&#160;&#160;&#160;一旦你实现了processEmail.m，脚本ex6 spam.m将在电子邮件示例上运行你的代码，你应该看到类似于图9和11的输出。

> **Octave/MATLAB提示**：在Octave/MATLAB中，可以用strcmp函数比较两个字符串。例如，strcmp(str1,str2)仅当两个字符串相等时才返回1。在提供的启动代码中，vocabList是一个“单元格数组”，其中包含词汇表中的单词。在Octave/MATLAB中，单元格数组与普通数组(即，但它的元素也可以是字符串(在Octave/MATLAB的矩阵/向量中不能是字符串)，并且可以使用大括号而不是方括号对它们进行索引。具体来说，要获得索引i处的单词，可以使用vocabList{i}。你还可以使用length(vocabList)来获得词汇表中的单词数量。

&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;*==你现在应该提交答案==*

### 2.2 从邮件中提取特征 
&#160;&#160;&#160;&#160;现在你将实现特征提取，即将每个邮件转为向量`$R^n$`。在这个练习中，你的n等于词汇表中的单词总数。具体地，电子邮件的特征`$x_i∈\{0,1\}$`对应于字典中的第i个单词是否出现在电子邮件中。 也就是说，如果第i个单词在电子邮件中，则`$x_i = 1$`，如果第i个单词不在电子邮件中，则`$x_i = 0$`。

&#160;&#160;&#160;&#160;因此，对于一个典型的电子邮件，这个功能看起来像:

<center><img src="https://note.youdao.com/yws/api/personal/file/WEBb1bb4304c2ec01458b92457d94812f24?method=download&shareKey=f756e0bd609a524610333926094690b7" /></center>

&#160;&#160;&#160;&#160;你现在应该在emailFeatures.m中完成代码，以便在给定单词索引的情况下为电子邮件生成特征向量。

&#160;&#160;&#160;&#160;一旦你实现了emailFeatures.m，ex6_spam.m的下一部分将在电子邮件样本上运行你的代码。你应该看到特征向量的长度为1899，非零条目为45。

&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;*==你现在应该提交答案==*

### 2.3 为垃圾邮件分类训练SVM
&#160;&#160;&#160;&#160;完成特征提取功能后，ex6_spam.m的下一步将加载预处理的训练数据集，该数据集将用于训练SVM分类器。 spamTrain.mat包含4000个垃圾邮件和非垃圾邮件的训练样本，而spamTest.mat则包含1000个测试样本。使用processEmail和emailFeatures函数处理每个原始电子邮件并将其转换为向量`$x^{(i)}∈R^{1899}$`
。

&#160;&#160;&#160;&#160;加载数据集后，ex6_spam.m将继续训练SVM，以便在垃圾邮件（y = 1）和非垃圾邮件（y = 0）电子邮件之间进行分类。 训练完成后，你应该看到分类器的训练准确率约为99.8％，测试精度约为98.5％。

### 2.4 垃圾邮件的最佳预测指标
&#160;&#160;&#160;&#160;为了更好地理解垃圾邮件分类器的工作原理，我们可以检查参数，以查看分类器认为哪些字最能预测垃圾邮件。 ex6_spam.m的下一步是在分类器中找到具有最大正值的参数，并显示相应的单词（图12）。 因此，如果电子邮件包含诸如“guarantee”，“remove”，“dollar”，和“price”（图12中所示的最佳预测指标）之类的单词，则可能将其归类为垃圾邮件。

<center><img src="https://note.youdao.com/yws/api/personal/file/WEBa05d0a2275468327e4165559394fbee1?method=download&shareKey=c85351059a7cd3ef3c488d91f6a3f4bf" /></center>
<center><h6>Figure 12: Top predictors for spam email</h6></center>

### 2.5 可选练习（不评分）：试试你自己的邮件
&#160;&#160;&#160;&#160;现在你已经训练了垃圾邮件分类器，你可以开始在自己的电子邮件中进行尝试。在启动代码中，我们提供了两个电子邮件样本（emailSample1.txt和emailSample2.txt）和两个垃圾邮件样本（spamSample1.txt和spamSample2.txt）。ex6_spam.m的最后一部分在第一个垃圾邮件示例上运行垃圾邮件分类器，并使用学习的SVM对其进行分类。 你现在应该尝试我们提供的其他示例，看看分类器是否正确。 你也可以使用自己的电子邮件替换示例（纯文本文件）来尝试自己的电子邮件。

&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;*==这部分不需要提交作业==*

### 2.6 可选练习（不评分）：建立自己的数据集
&#160;&#160;&#160;&#160;在本练习中，我们提供了一个预处理的训练集和测试集。 这些数据集是使用你现在已完成的相同函数（processEmail.m和emailFeatures.m）创建的。对于此可选（未评级）练习，你将使用[SpamAssassin Public Corpus](http://spamassassin.apache.org/old/publiccorpus/)的原始电子邮件构建你自己的数据集。

&#160;&#160;&#160;&#160;在这个可选的（不评分）练习中，你的任务是从公共语料库下载原始文件并提取它们。在提取它们之后，你应该在每个电子邮件上运行processEmail和emailFeatures函数，以便从每个电子邮件中提取一个特征向量。这将允许你构建一个样本的数据集X，y。然后，你应该将数据集随机分为训练集、交叉验证集和测试集。

&#160;&#160;&#160;&#160;在构建自己的数据集时，我们还鼓励你尝试构建自己的词汇表（通过选择数据集中出现的高频词）并添加你认为可能有用的任何其他功能。

&#160;&#160;&#160;&#160;最后，我们还建议尝试使用高度优化的SVM工具箱，例如[LIBSVM](https://www.csie.ntu.edu.tw/~cjlin/libsvm/)。

&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;*==这部分不需要提交作业==*

## 提交作业和评分
&#160;&#160;&#160;&#160;完成各个部分之后，请使用提交功能将你的代码提交到我们的服务器。以下是本次作业评分的细则。

<center><img src="https://note.youdao.com/yws/api/personal/file/WEB5069725937c51593b620df8c4eb0d4bc?method=download&shareKey=c85351059a7cd3ef3c488d91f6a3f4bf" /></center>

&#160;&#160;&#160;&#160;你可以多次提交作业，但我们只考虑最高分。
