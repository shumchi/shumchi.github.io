---
layout: post
title:  "Advanced methods of imputation missing value"
date:   2019-04-11 12:00:00
excerpt: ""
author: Chi Shen
tags: [R, Missing Value, Imputation]
feature: https://en.wikipedia.org/wiki/Lion#/media/File:Lion_waiting_in_Namibia.jpg
comments: true
---




<br> <br>
前段时间师妹有问到怎么处理数据库中的缺失值，之前的项目中自己也遇到过，这段时间正好在学习Data
Scientice的课程，刚好又遇到了这一部分，
正好就趁机会把去年查过的资料又复习了一遍，顺手做份笔记，分享给有需要的同学，网络有记忆，便于随时查看。  
<br> <br>

**参考文献：**

Robert Kabacoff (2013). R in Action, Data Analysis and Graphics with R  
高涛, 肖楠, 陈钢 (译).R语言实战. 人民邮电出版社, 2013  
Stef van Buuren, Karin Groothuis-Oudshoorn. mice: Multivariate
Imputation by Chained Equations in R. Journal of Statistical software,
2009

<br> <br>

这份笔记主要从四个方面记录：

**目录**

-   处理缺失数据的重要性
-   识别缺失数据
-   探索缺失原因或模式
-   缺失值的处理

<br>

一、处理缺失数据的重要性
------------------------

一项常见的定量研究或者分析工作，通常分为四个阶段：

Collect -- **Data cleaning** -- Analysis -- Report

当然，在这四个阶段之前还有Design的过程，这是决定一项研究或者分析任务成败的关键，一项好的研究应该是在设计阶段就
已经明确和定义了处理缺失值的原则了。这四个阶段中无疑Data
cleaning是最重要的，就好比做一桌菜，准备好原料很重要一样，不然再好的厨师也很难做
一桌好菜。如果你经常处理数据，应该能体会到一份好的数据对于一段愉悦的分析过程是多么重要，不然这个过程会十分煎熬。  
<br>

在Data
cleaning过程中除了常见的规范variable、label、value以及标准的codebook外，missing
value的处理也是非常重要的一个环节，因此规范的
缺失数据的处理直接影响着研究结果的稳健性和真实性，也是研究能够reproductive的重要保障。

通常，处理缺失值的步骤如下：

-   识别缺失数据
-   检查导致数据缺失的原因
-   删除包含缺失值的实例或是合理的数值代替（插补）缺失值

<br>

二、识别缺失数据
----------------

### 1.缺失数据分类

在识别缺失数据之前，先来了解一下缺失数据的分类，它直接关系到处理缺失值方式的选择。

-   1.  完全随机缺失。若某变量的缺失数据与其他任何观测或未观测变量都不相关，则数据为完全随机缺失（MCAR）。若12个动物的做梦时长值缺失不是由于系统原因，那么可认为数据是MCAR。注意，如果每个有缺失值的变量都是MCAR，那么可以将数据完整的实例看做是对更大数据集的一个简单随机抽样。

-   1.  随机缺失
        若某变量上的缺失数据与其他观测变量相关，与它自己的未观测值不相关，则数据为随机缺失（MAR）。

-   1.  非随机缺失
        若缺失数据不属于MCAR或MAR，则数据为非随机缺失（NMAR）。

例如：

-   （1）在最近一个问卷调查中，发现一些项常常一同缺失。很明显这些项聚集在一起，因为调查对象没有意识到问卷的第三页的背面也包含了这些项目。此时，可以认为这些数据是MCAR。  
-   （2）在一个关于全球领导风格的调查中，学历变量经常性地缺失。调查显示欧洲的调查对象更可能在此项目上留白，这说明某些特定国家的调查对象没有理解变量的分类。此时，这种数据最可能是MAR。

大部分处理缺失数据的方法都假定数据是MCAR或MAR。此时，你可以忽略缺失数据的生成机制。  
当数据是NMAR时，想对它进行恰当地分析比较困难，你既要对感兴趣的关系进行建模，还要对缺失值的生成机制进行建模。  
<br>

### 2.识别缺失值

此次只用**R语言**来演示软件操作过程，Stata及SAS等其他软件可自行查阅相关文档。

以下研究主要使用R的**VIM**包，以及该包中自带的sleep数据集进行演示。  
<br> <br>

**sleep数据集简介**

睡眠变量包含睡眠中做梦时长（Dream）、不做梦的时长（NonD）以及它们的和（Sleep）。体
质变量包含体重（BodyWgt，单位为千克）、脑重（BrainWgt，单位为克）、寿命（Span，单位为
年）和妊娠期（Gest，单位为天）。生态学变量包含物种被捕食的程度（Pred）、睡眠时暴露的程
度（Exp）和面临的总危险度（Danger）。

<br>

#### （1）以表格形式进行探索

主要使用的*is.na()*和*complete.cases()*函数

    # 加载VIM包
    library(VIM)

    # 加载数据集
    data(sleep, package = "VIM")

    # 查看数据集结构
    str(sleep)

    # 列出没有缺失值的行
    print("不包含缺失值的行")
    sleep[complete.cases(sleep),]

    #列出有一个或多个缺失值的行
    print("包含一个或多个缺失值的行")
    sleep[!complete.cases(sleep),]

    'data.frame':   62 obs. of  10 variables:
     $ BodyWgt : num  6654 1 3.38 0.92 2547 ...
     $ BrainWgt: num  5712 6.6 44.5 5.7 4603 ...
     $ NonD    : num  NA 6.3 NA NA 2.1 9.1 15.8 5.2 10.9 8.3 ...
     $ Dream   : num  NA 2 NA NA 1.8 0.7 3.9 1 3.6 1.4 ...
     $ Sleep   : num  3.3 8.3 12.5 16.5 3.9 9.8 19.7 6.2 14.5 9.7 ...
     $ Span    : num  38.6 4.5 14 NA 69 27 19 30.4 28 50 ...
     $ Gest    : num  645 42 60 25 624 180 35 392 63 230 ...
     $ Pred    : int  3 3 1 5 3 4 1 4 1 1 ...
     $ Exp     : int  5 1 1 2 5 4 1 5 2 1 ...
     $ Danger  : int  3 3 1 3 4 4 1 4 1 1 ...
    [1] "不包含缺失值的行"
        BodyWgt BrainWgt NonD Dream Sleep  Span  Gest Pred Exp Danger
    2     1.000     6.60  6.3   2.0   8.3   4.5  42.0    3   1      3
    5  2547.000  4603.00  2.1   1.8   3.9  69.0 624.0    3   5      4
    6    10.550   179.50  9.1   0.7   9.8  27.0 180.0    4   4      4
    7     0.023     0.30 15.8   3.9  19.7  19.0  35.0    1   1      1
    8   160.000   169.00  5.2   1.0   6.2  30.4 392.0    4   5      4
    9     3.300    25.60 10.9   3.6  14.5  28.0  63.0    1   2      1
    10   52.160   440.00  8.3   1.4   9.7  50.0 230.0    1   1      1
    11    0.425     6.40 11.0   1.5  12.5   7.0 112.0    5   4      4
    12  465.000   423.00  3.2   0.7   3.9  30.0 281.0    5   5      5
    15    0.075     1.20  6.3   2.1   8.4   3.5  42.0    1   1      1
    16    3.000    25.00  8.6   0.0   8.6  50.0  28.0    2   2      2
    17    0.785     3.50  6.6   4.1  10.7   6.0  42.0    2   2      2
    18    0.200     5.00  9.5   1.2  10.7  10.4 120.0    2   2      2
    22   27.660   115.00  3.3   0.5   3.8  20.0 148.0    5   5      5
    23    0.120     1.00 11.0   3.4  14.4   3.9  16.0    3   1      2
    25   85.000   325.00  4.7   1.5   6.2  41.0 310.0    1   3      1
    27    0.101     4.00 10.4   3.4  13.8   9.0  28.0    5   1      3
    28    1.040     5.50  7.4   0.8   8.2   7.6  68.0    5   3      4
    29  521.000   655.00  2.1   0.8   2.9  46.0 336.0    5   5      5
    32    0.005     0.14  7.7   1.4   9.1   2.6  21.5    5   2      4
    33    0.010     0.25 17.9   2.0  19.9  24.0  50.0    1   1      1
    34   62.000  1320.00  6.1   1.9   8.0 100.0 267.0    1   1      1
    37    0.023     0.40 11.9   1.3  13.2   3.2  19.0    4   1      3
    38    0.048     0.33 10.8   2.0  12.8   2.0  30.0    4   1      3
    39    1.700     6.30 13.8   5.6  19.4   5.0  12.0    2   1      1
    40    3.500    10.80 14.3   3.1  17.4   6.5 120.0    2   1      1
    42    0.480    15.50 15.2   1.8  17.0  12.0 140.0    2   2      2
    43   10.000   115.00 10.0   0.9  10.9  20.2 170.0    4   4      4
    44    1.620    11.40 11.9   1.8  13.7  13.0  17.0    2   1      2
    45  192.000   180.00  6.5   1.9   8.4  27.0 115.0    4   4      4
    46    2.500    12.10  7.5   0.9   8.4  18.0  31.0    5   5      5
    48    0.280     1.90 10.6   2.6  13.2   4.7  21.0    3   1      3
    49    4.235    50.40  7.4   2.4   9.8   9.8  52.0    1   1      1
    50    6.800   179.00  8.4   1.2   9.6  29.0 164.0    2   3      2
    51    0.750    12.30  5.7   0.9   6.6   7.0 225.0    2   2      2
    52    3.600    21.00  4.9   0.5   5.4   6.0 225.0    3   2      3
    54   55.500   175.00  3.2   0.6   3.8  20.0 151.0    5   5      5
    57    0.900     2.60 11.0   2.3  13.3   4.5  60.0    2   1      2
    58    2.000    12.30  4.9   0.5   5.4   7.5 200.0    3   1      3
    59    0.104     2.50 13.2   2.6  15.8   2.3  46.0    3   2      2
    60    4.190    58.00  9.7   0.6  10.3  24.0 210.0    4   3      4
    61    3.500     3.90 12.8   6.6  19.4   3.0  14.0    2   1      1
    [1] "包含一个或多个缺失值的行"
        BodyWgt BrainWgt NonD Dream Sleep Span Gest Pred Exp Danger
    1  6654.000   5712.0   NA    NA   3.3 38.6  645    3   5      3
    3     3.385     44.5   NA    NA  12.5 14.0   60    1   1      1
    4     0.920      5.7   NA    NA  16.5   NA   25    5   2      3
    13    0.550      2.4  7.6   2.7  10.3   NA   NA    2   1      2
    14  187.100    419.0   NA    NA   3.1 40.0  365    5   5      5
    19    1.410     17.5  4.8   1.3   6.1 34.0   NA    1   2      1
    20   60.000     81.0 12.0   6.1  18.1  7.0   NA    1   1      1
    21  529.000    680.0   NA   0.3    NA 28.0  400    5   5      5
    24  207.000    406.0   NA    NA  12.0 39.3  252    1   4      1
    26   36.330    119.5   NA    NA  13.0 16.2   63    1   1      1
    30  100.000    157.0   NA    NA  10.8 22.4  100    1   1      1
    31   35.000     56.0   NA    NA    NA 16.3   33    3   5      4
    35    0.122      3.0  8.2   2.4  10.6   NA   30    2   1      1
    36    1.350      8.1  8.4   2.8  11.2   NA   45    3   1      3
    41  250.000    490.0   NA   1.0    NA 23.6  440    5   5      5
    47    4.288     39.2   NA    NA  12.5 13.7   63    2   2      2
    53   14.830     98.2   NA    NA   2.6 17.0  150    5   5      5
    55    1.400     12.5   NA    NA  11.0 12.7   90    2   2      2
    56    0.060      1.0  8.1   2.2  10.3  3.5   NA    3   1      2
    62    4.050     17.0   NA    NA    NA 13.0   38    3   1      1

从结果可以看出，
完整数据集中有62条observation，其中只有42条不含缺失值，20条含一个或多个缺失值。

由于逻辑值TRUE和FALSE分别等价于数值1和0，可用sum()和mean()函数来获取关于缺失
数据的有用信息。如：

    sum(is.na(sleep$Dream))
    mean(is.na(sleep$Dream))
    mean(!complete.cases(sleep))

    [1] 12
    [1] 0.1935484
    [1] 0.3225806

结果表明变量Dream有12个缺失值，19%的实例在此变量上有缺失值。另外，数据集中32%的实
例包含一个或多个缺失值。

另外一种方法是，采用**mice**包中的*md.pattern()*函数来生成缺失值矩阵，如下：

    library(mice)
    md.pattern(sleep, plot = FALSE)

       BodyWgt BrainWgt Pred Exp Danger Sleep Span Gest Dream NonD   
    42       1        1    1   1      1     1    1    1     1    1  0
    9        1        1    1   1      1     1    1    1     0    0  2
    3        1        1    1   1      1     1    1    0     1    1  1
    2        1        1    1   1      1     1    0    1     1    1  1
    1        1        1    1   1      1     1    0    1     0    0  3
    1        1        1    1   1      1     1    0    0     1    1  2
    2        1        1    1   1      1     0    1    1     1    0  2
    2        1        1    1   1      1     0    1    1     0    0  3
             0        0    0   0      0     4    4    4    12   14 38

结果中，最右侧列显示的是缺失的variable数量，0表示无缺失；最左侧列显示对应的observation数量，如第一行显示，有42条observation在BodyWgt等10个variable中
均无缺失；第二行显示，Dream和NonD共2个变量同时有缺失的observation有9条。

那么，数据集的20条缺失observation中，共包含缺失值(42 × 0) + (2 × 1) + …
+ (1 × 3) = 38个。

<br> <br>

#### （2）以图形探索缺失数据

虽然表格的输出也很简洁，但是图形的输出会更直观，此时可以采用VIM包中的*aggr()*和*matrixplot()*函数。

    aggr(sleep, prop = FALSE, numbers = TRUE, labels = TRUE, )

![](https://github.com/shumchi/shumchi.github.io/blob/master/_posts/2019-02-11-Imputation-missing-value/figure-markdown_strict/Example-4-1.png?raw=true)

**Fig.1 aggr()生成的sleep数据集的缺失值模式图形**

输出的图中，左侧的红色条图分别显示了所有variable的缺失值数量，可以清楚的看出Gest变量有4个缺失值等。  
右侧的图中，红色代表缺失，可以看出无任何缺失的observation共有42条，同时了缺失NonD和Dream的observation共有9条。

    matrixplot(sleep, main = "Fig.2 按实例（行）展示真实值和缺失值的矩阵图")

![](https://github.com/shumchi/shumchi.github.io/blob/master/_posts/2019-02-11-Imputation-missing-value/figure-markdown_strict/Example-5-1.png?raw=true)

matrixplot()函数可生成展示每个实例数据的图形。
此处，数值型数据被重新转换到\[0,
1\]区间，并用灰度来表示大小：浅色表示值小，深色表示值大。默认缺失值为红色。

图中，横轴展示了全部的variable，纵轴显示了全部observation，
从图中可以清楚的了解缺失值的分布情况，默认是按照第一个变量BodyWgt从小到大排序过。

<br>

三、探索缺失原因或模式
----------------------

### 1.通过图形捕捉缺失原因或模式

通过Fig.2的矩阵图中，可以看出某些变量的缺失值模式是否与其他变量的真实值有关联。
此图中可以看到，无缺失值的睡眠变量（Dream、NonD和Sleep）对应着较小的体重（BodyWgt）或脑重（BrainWgt）。
此时，可以采用VIM包中的*marginplot()*函数可生成一幅散点图，在图形边界展示两个变量的缺失值信息，并观察变量之间的关系。

    marginplot(sleep[c("Gest", "Dream")], col = c("darkgrey", "red", "blue"), pch = c(20), main = "Fig.3 变量Gest与Deam的散点图及缺失信息")

![](https://github.com/shumchi/shumchi.github.io/blob/master/_posts/2019-02-11-Imputation-missing-value/figure-markdown_strict/Example-6-1.png?raw=true)

图中，共显示了两部分信息，（a）散点图（b）边界显示缺失信息，具体为：

横轴为Gest变量，边界的红色boxplot显示的是Dream变量中缺失值对应的Gest值的箱式图，灰色为Dream变量中非缺失值对应的Gest值的箱式图；  
纵轴为Dream变量，边界的红色boxplot显示的是Gest变量中缺失值对应的Dream值的箱式图，灰色为Gest变量中非缺失值对应的Dream值的箱式图；  
灰色散点为Dream与Gest的散点图；  
左下角的蓝色数字表示Dream与Gest共同缺失的observation数量。

从图中可以看出，Dream与Gest成负相关关系，并且，Gest的缺失值对应更大的Dream值，即缺失妊娠期数据时动物的做梦时长一般更长。

### 2.通过相关性探索缺失值

图形展示了缺失变量之间可能的相关关系，进一步可用指示变量替代数据集中的数据（1表示缺失，0表示存在），这样生成的矩阵有时称作影子矩阵。
求这些指示变量间和它们与初始（可观测）变量间的相关性，有助于观察哪些变量常一起缺失，以及分析变量“缺失”与其他变量间的关系。具体如下：

    a <- abs(is.na(sleep)) # 将是否缺失的逻辑变量转换为0和1变量的数据框形式
    as.data.frame(a)
    b <- apply(a, 2, sum) %>% .[which(. != 0)] %>% names()
    c <- subset(a, select = b ) # 提取出缺失数据
    cor(c, y = NULL) # 对缺失数据进行相关分析

       BodyWgt BrainWgt NonD Dream Sleep Span Gest Pred Exp Danger
    1        0        0    1     1     0    0    0    0   0      0
    2        0        0    0     0     0    0    0    0   0      0
    3        0        0    1     1     0    0    0    0   0      0
    4        0        0    1     1     0    1    0    0   0      0
    5        0        0    0     0     0    0    0    0   0      0
    6        0        0    0     0     0    0    0    0   0      0
    7        0        0    0     0     0    0    0    0   0      0
    8        0        0    0     0     0    0    0    0   0      0
    9        0        0    0     0     0    0    0    0   0      0
    10       0        0    0     0     0    0    0    0   0      0
    11       0        0    0     0     0    0    0    0   0      0
    12       0        0    0     0     0    0    0    0   0      0
    13       0        0    0     0     0    1    1    0   0      0
    14       0        0    1     1     0    0    0    0   0      0
    15       0        0    0     0     0    0    0    0   0      0
    16       0        0    0     0     0    0    0    0   0      0
    17       0        0    0     0     0    0    0    0   0      0
    18       0        0    0     0     0    0    0    0   0      0
    19       0        0    0     0     0    0    1    0   0      0
    20       0        0    0     0     0    0    1    0   0      0
    21       0        0    1     0     1    0    0    0   0      0
    22       0        0    0     0     0    0    0    0   0      0
    23       0        0    0     0     0    0    0    0   0      0
    24       0        0    1     1     0    0    0    0   0      0
    25       0        0    0     0     0    0    0    0   0      0
    26       0        0    1     1     0    0    0    0   0      0
    27       0        0    0     0     0    0    0    0   0      0
    28       0        0    0     0     0    0    0    0   0      0
    29       0        0    0     0     0    0    0    0   0      0
    30       0        0    1     1     0    0    0    0   0      0
    31       0        0    1     1     1    0    0    0   0      0
    32       0        0    0     0     0    0    0    0   0      0
    33       0        0    0     0     0    0    0    0   0      0
    34       0        0    0     0     0    0    0    0   0      0
    35       0        0    0     0     0    1    0    0   0      0
    36       0        0    0     0     0    1    0    0   0      0
    37       0        0    0     0     0    0    0    0   0      0
    38       0        0    0     0     0    0    0    0   0      0
    39       0        0    0     0     0    0    0    0   0      0
    40       0        0    0     0     0    0    0    0   0      0
    41       0        0    1     0     1    0    0    0   0      0
    42       0        0    0     0     0    0    0    0   0      0
    43       0        0    0     0     0    0    0    0   0      0
    44       0        0    0     0     0    0    0    0   0      0
    45       0        0    0     0     0    0    0    0   0      0
    46       0        0    0     0     0    0    0    0   0      0
    47       0        0    1     1     0    0    0    0   0      0
    48       0        0    0     0     0    0    0    0   0      0
    49       0        0    0     0     0    0    0    0   0      0
    50       0        0    0     0     0    0    0    0   0      0
    51       0        0    0     0     0    0    0    0   0      0
    52       0        0    0     0     0    0    0    0   0      0
    53       0        0    1     1     0    0    0    0   0      0
    54       0        0    0     0     0    0    0    0   0      0
    55       0        0    1     1     0    0    0    0   0      0
    56       0        0    0     0     0    0    1    0   0      0
    57       0        0    0     0     0    0    0    0   0      0
    58       0        0    0     0     0    0    0    0   0      0
    59       0        0    0     0     0    0    0    0   0      0
    60       0        0    0     0     0    0    0    0   0      0
    61       0        0    0     0     0    0    0    0   0      0
    62       0        0    1     1     1    0    0    0   0      0
                 NonD       Dream       Sleep        Span        Gest
    NonD   1.00000000  0.90711474  0.48626454  0.01519577 -0.14182716
    Dream  0.90711474  1.00000000  0.20370138  0.03752394 -0.12865350
    Sleep  0.48626454  0.20370138  1.00000000 -0.06896552 -0.06896552
    Span   0.01519577  0.03752394 -0.06896552  1.00000000  0.19827586
    Gest  -0.14182716 -0.12865350 -0.06896552  0.19827586  1.00000000

此时，你可以看到Dream和NonD常常一起缺失（r =
0.91）。相对可能性较小的是Sleep和NonD一起缺失（r =
0.49），以及Sleep和Dream（r = 0.20）

<br>

四、缺失值的处理
----------------

在对缺失数据进行识别和探索后，下一步就是对缺失数据进行填补，但是在进行下一步之前，需要前面的工作弄清楚几个问题，以便选择合理的缺失值处理方式：

-   缺失数据的比例为多少
-   哪些变量上存在缺失，是否存在某种明显的分布趋势
-   缺失的类型，完全随机（MCAR）、随机(MAR)或非随机(NMAR)

<br>

### 1.缺失值处理方式

#### （一）不予处理

如果有一小部分数据（如小于10%）随机分布在整个数据集中（MCAR），那么你可以分析数据完整的实例，这样仍可以得到可靠且有效的结果。

#### （二）删除变量（列）

如果缺失数据集中在几个相对不太重要的变量上，那么你可以删除这些变量，然后再进行正常的数据分析

#### （三）删除观测（行）

在完整的数据分析中，只有每个变量都包含了有效数据值的观测才会保留下来做进一步的分析。
实际上，这样会导致包含一个或多个缺失值的任意一行都会被删除，因此常称作行删除法（listwise）、个案删除（case-wise）或剔除。
大部分流行的统计软件包都默认采用行删除法来处理缺失值。

如果你平时留心观察过，不管是在Stata或者SAS中，进行crostable、t-test、anova或者regression分析的时候，默认都是剔除了包含缺失值的记录的。
但是，行删除法是有一个前提假定的，即数据是完全随机缺失（MCAR）（即完整的观测只是全数据集的一个随机子样本）。
如果不满足此条件，那么删除行是会产生有偏的结果。

#### （四）数据填补

根据研究设计不同，填补的方法有多种，如clinical
trial中对于重复观察结果采用的末次观测结转(LOCF:Last observation carried
forward)，
以及平时常用的简单插补法（即用某个值（如均值、中位数或众数）来替换变量中的缺失值）。

这里只介绍在面对复杂的缺失值问题时，常用的多重插补法(MI,Multiple
imputation)，此处使用前面提到的R语言的**mise**包，其他软件也有类似的函数或者包。

<br>

#### *多重插补法(MI,Multiple imputation)*

**原理：**

MI是将从一个包含缺失值的数据集中生成一组完整的数据集（通常是3到10个）。每个模拟数据集中，缺失数据将用蒙特卡洛方法来填补。

缺失值的插补通过Gibbs抽样完成。每个包含缺失值的变量都默认可通过数据集中的其他变量预测得来，于是这些预测方程便可用来预测缺失数据的有效值。
该过程不断迭代直到所有的缺失值都收敛为止。对于每个变量，用户可以选择预测模型的形式（称为基本插补法）和待选入的变量。

默认地，预测的均值用来替换连续型变量中的缺失数据，而Logistic或多元Logistic回归则分别用来替换二值目标变量（两水平因子）或多值变量（多于两水平的因子）。
其他基本插补法包括贝叶斯线性回归、判别分析、两水平正态插补和从观测值中随机抽样。用户也可以选择自己独有的方法。

**语法：**

函数mice()首先从一个包含缺失数据的数据框开始，然后返回一个包含多个（默认为5个）完整数据集的对象。
每个完整数据集都是通过对原始数据框中的缺失数据进行插补而生成的。由于插补有随机的成分，因此每个完整数据集都略有不同。
然后，with()函数可依次对每个完整数据集应用统计模型（如线性模型或广义线性模型），最后，pool()函数将这些单独的分析结果整合为一组结果。

    ```r
    library(mice) # 加载mice包
    imp <- mice(df, m, seed = 1234, method = "pmm") # m为生成的数据集个数，默认为5个, seed是设定种子数，保证填补结果的可重复性，值可任意设定   
    sleep_im <- complete(pooled, action = 3) # 导出需要的填补完整数据集，action表示选择哪一个
    ```

method为默认插补方式，pmm为默认方式预测均值匹配（Predictive mean
matching ）, 还有一些其他methods插补方法:

-   贝叶斯线性回归（norm）
-   基于bootstrap的线性回归（norm.boot）
-   线性回归预测值（norm.predict）
-   分类回归树（cart）
-   随机森林（rf）

使用这些插补方法对数据有严格的要求，比如贝叶斯线性回归等前三个模型都需要数据符合numeric格式，而pmm、cart、rf任意格式都行。

**使用以上模型遇见的问题有：**

1、pmm相当于某一指标的平均值作为插补，会出现插补值重复的问题；

2、cart以及rf是挑选某指标中最大分类的那个数字，是指标中的某一个数字，未按照规律；

3、要使用norm.predict，必须先对数据进行格式转换，这个过程中会出现一些错误；

**实例：**

    library(mice)
    library(magrittr)

    imp <- mice(sleep, seed = 1234, method = "pmm")
    sleep_im <- complete(imp, action = 1)

    #查看填补情况
    imp$imp$Dream


     iter imp variable
      1   1  NonD  Dream  Sleep  Span  Gest
      1   2  NonD  Dream  Sleep  Span  Gest
      1   3  NonD  Dream  Sleep  Span  Gest
      1   4  NonD  Dream  Sleep  Span  Gest
      1   5  NonD  Dream  Sleep  Span  Gest
      2   1  NonD  Dream  Sleep  Span  Gest
      2   2  NonD  Dream  Sleep  Span  Gest
      2   3  NonD  Dream  Sleep  Span  Gest
      2   4  NonD  Dream  Sleep  Span  Gest
      2   5  NonD  Dream  Sleep  Span  Gest
      3   1  NonD  Dream  Sleep  Span  Gest
      3   2  NonD  Dream  Sleep  Span  Gest
      3   3  NonD  Dream  Sleep  Span  Gest
      3   4  NonD  Dream  Sleep  Span  Gest
      3   5  NonD  Dream  Sleep  Span  Gest
      4   1  NonD  Dream  Sleep  Span  Gest
      4   2  NonD  Dream  Sleep  Span  Gest
      4   3  NonD  Dream  Sleep  Span  Gest
      4   4  NonD  Dream  Sleep  Span  Gest
      4   5  NonD  Dream  Sleep  Span  Gest
      5   1  NonD  Dream  Sleep  Span  Gest
      5   2  NonD  Dream  Sleep  Span  Gest
      5   3  NonD  Dream  Sleep  Span  Gest
      5   4  NonD  Dream  Sleep  Span  Gest
      5   5  NonD  Dream  Sleep  Span  Gest
         1   2   3   4   5
    1  0.5 0.3 0.5 1.3 1.4
    3  1.4 4.1 2.3 1.4 1.5
    4  3.4 3.4 2.2 3.6 3.6
    14 0.5 0.3 0.5 0.3 0.3
    24 1.2 1.0 3.4 1.3 6.1
    26 2.1 5.6 2.4 2.8 2.2
    30 2.3 1.2 2.3 2.4 2.4
    31 3.4 0.6 0.5 0.5 2.3
    47 1.4 2.0 1.4 1.5 3.6
    53 0.5 0.5 0.0 0.6 0.5
    55 0.5 0.3 1.4 2.7 3.4
    62 2.3 3.1 6.6 5.6 3.4

**检查插补效果：**

通过散点图和分布图，检查填补数据的效果，以下各图中，红色点表示填补值，蓝色表示非填补值

    library(lattice)
    xyplot(imp, Dream ~ Gest + Span, pch = 18, cex = 1)

![](https://github.com/shumchi/shumchi.github.io/blob/master/_posts/2019-02-11-Imputation-missing-value/figure-markdown_strict/Example-9-1.png?raw=true)

    densityplot(imp)

![](https://github.com/shumchi/shumchi.github.io/blob/master/_posts/2019-02-11-Imputation-missing-value/figure-markdown_strict/Example-9-2.png?raw=true)

    stripplot(imp, pch = 20, cex = 1.2)

![](https://github.com/shumchi/shumchi.github.io/blob/master/_posts/2019-02-11-Imputation-missing-value/figure-markdown_strict/Example-9-3.png?raw=true)

**填补后数据集应用：**

通常情况下，填补完成后，我们会进行回归分析，如前所示，MI填补生成了5个填补数据集，此时就会有一个疑问，到底选择哪一个作为最后的完整数据集。  
**mice**包提供了一个函数可以很容易的将5个填补数据集分别进行回归，然后将结果合并后返回一个结果。

    sleep_im_fit <- with(imp, lm(Sleep ~ Gest + Span))
    summary(pool(sleep_im_fit))

                   estimate   std.error  statistic       df     p.value
    (Intercept) 13.21124331 0.744488807 17.7453888 54.52341 0.000000000
    Gest        -0.01715258 0.005056239 -3.3923598 24.80308 0.001296459
    Span        -0.01742712 0.036796405 -0.4736092 46.32288 0.637669540

处理一般线性回归的lm()函数，
其他方法包括广义线性模型glm()函数、广义可加模型gam()
，负二项模型nbrm()函数均可实现

<br> <br> <br>

The End
-------
