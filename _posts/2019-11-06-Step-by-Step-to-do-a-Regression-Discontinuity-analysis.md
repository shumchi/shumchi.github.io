---
layout: post
title:  "Step by Step to do Regression Discontinuity Analysis"
date:   2019-11-06 19:10:00
excerpt: "断点回归（Regression
Discontinuity）适用于以下情形：人群是否接受干预（Treatment）是依据某一
数值变量（rating
variable）是否高于或低于某一确定的阈值（threshold）或者分割点（cut-point），
例如在研究是否上大学会影响收入时，数值变量（rating variable，也叫
assignment variable，score， running variable，forcing variable， or
index）就是高考分数，阈值或者分割点就是本科录取分数线。"
author: Chi Shen
tags: [R, Regression
Discontinuity]
feature: https://upload.wikimedia.org/wikipedia/commons/d/db/English_Bulldog_about_to_sleep.jpg
comments: true


---

**Chi Shen, Nov 06 2019**

**New Haven, CT**

## 1. 简介

断点回归（Regression
Discontinuity）适用于以下情形：人群是否接受干预（Treatment）是依据某一
数值变量（rating
variable）是否高于或低于某一确定的阈值（threshold）或者分割点（cut-point），
例如在研究是否上大学会影响收入时，数值变量（rating variable，也叫
assignment variable，score， running variable，forcing variable， or
index）就是高考分数，阈值或者分割点就是本科录取分数线。

## 2. 发展历史

Regression Discontinuity最早由社会学家Thistlethwaite and Campbell
在1960年提出的，
用于评估社会项目，但是他们的研究虽然引起一些影响但是没有得到广泛的注意，后来，在1972-2009年期间，
被一系列经济学家(Goldberger，van der Klaauw，Imbens and
Kalyanaraman等)在方法学方面进行了完善， 最终在2008年达到顶峰，
标志是在2008年Journal of Econometrics出了一 期RD分析的[Special
Issue](https://www.sciencedirect.com/journal/journal-of-econometrics/vol/142/issue/2)，
光看期刊名字就知道是 经济学顶刊了(**Journal of
Econometrics是公认的计量经济学顶尖期刊，是教育部认可的12本经济学国际顶级期刊之一**
)。

## 3. 特点与分类

**RD有两个特点：**

-   在Rating
    variable的一个明确点上，outcome出现了跳跃或者不连续(discontinuity at
    a cut-point)
-   可以认为在一个限定的rating
    variable区间上，个体是服从局部随机的(local randomization)

**RD有两种类型：**

-   精确断点(sharp design): 即所有个体(All
    subject)在明确的cut-point之后全部接受干预(treatment)

-   模糊断点(fuzzy design): 即在cut-point前后，存在 no-shows (treatment
    group members who do not receive the treatment) 或者crossovers
    (control group members who do receive the treatment) ，
    换句话说就无法找到一个明确的cut-point完全区分干预和对照组。
    更严格的分法是，fuzzy也可以分成两类：type
    I是no-shows，只存在处理组有未接受处理的个体， type
    II是同时存在no-shows和crossovers。实际就是RCT中常说的沾染问题。

## 4. 适用RD分析的先决条件(Conditions for Internal Validity)

由于RD仍然属于非实验方法，尽管也被成为类实验(quasi-experimental)，但本质还是非实验方法(nonexperimental)，
所以它必须满足一系列前期条件，才能提供无偏估计和更可能的接近RCT的严格情形

-   **一、** Rating variable
    (中文的翻译很多，但比较通用的翻译为驱动变量或者配置变量)不能被干预(treatment)所影响，
    Rating variable
    的值必须是在干预(treatment)**之前**就已经确定了，或者是不可再更改的变量。
    也就是只能是驱动变量的值决定是否接受干预，不能是干预决定驱动变量的值。

    比如：高考成绩(Rating
    variable)是在判断是否能够上大学(treatment)之前就已经确定了，并且分数不会再发生变化，
    如果存在严重的根据分数线修改高考成绩的情况，则高考分数就不能作为Rating
    variable。  
    **检验方法是：画Rating
    variable的频率分布直方图或者密度图，再进行McCrary 检验**

-   **二、**
    断点(cut-point)是完全外生的，并且干预与否完全取决于驱动变量和断点。即断点不受其他因素影响而改变干预的判断，
    即把本不该接受干预的对象划入了干预，或者把该接受干预的对象划入了对照。

    比如，如果高考录取线(cut-point)的确定是完全已经严格的录取指标划定的，
    而不存在为了使得某部分人群能够获得上大学的机会，而修改录取线。

-   **三、**
    在驱动变量的cut-point前后，除了干预与否(treatment)是不连续或者跳跃的(比如断点前为0，断点后为1)，
    其他任何因素都不能出现不连续或者跳跃(Nothing other than treatment
    status is discontinuous in the analysis interval)，
    因为要保证outcome的跳跃只能是干预的不连续或者跳跃导致的。

    比如，如果高考分数线除了决定是否能上大学外，还决定是否享有创业扶持机会，
    那么如果研究上大学对某种结局的影响，就无法分离出到底是大学教育导致，还是获得创业扶持机会导致的。

-   **四、** 驱动变量(rating
    variable)与结局变量(outcome)之间的函数关系应该是连续的，严格的说，
    应该是在进行RD分析的这段区间内(interval)是连续的。换句话说就是，如果不存在干预的情况，
    驱动变量与结局变量之间不会出现不连续或者跳跃，才能在结局变量出现不连续的情况下，反推出是干预引起的，
    因为只有干预是不连续的。**注意：此条件只需要在选择参数估计方法是要满足(applies
    only to parametric estimators)**

## 5.断点回归的图形分析(Graphical Presentations in the RD)

**图形分析是进行RD的第一步，也是非常重要的组成部分**

**通常，进行RD至少需要画4种图：**

-   驱动变量与干预之间的关系图，在这一步可以确定是应该采取Sharp还是Fuzzy断点回归设计
-   非结局变量与驱动变量的关系图，可以用来判断是否满足第3条内部有效性(Internal
    Validity)条件
-   驱动变量的密度分布图，是为了判断驱动变量是否连续，以及在cut-point附近是否存在被操控，可以用来判读石佛满足第1条内部有效性(Internal
    Validity)条件
-   结局变量与驱动变量的关系图，可以帮助预估干预效应的大小，以及判断结局变量与驱动变量之间的函数关系。而且必须是结局变量在纵轴，驱动变量在横轴。**注意：通常这一幅图是用来初步判断整个研究是否能够获得预期的干预效应的，如果在这幅图中无法观测到明显的jump，基本后续的分析也是徒劳。**

## 6. 进行RD分析的步骤及示例

**示列软件**: R version 3.5.3 (2019-03-11)  
**示列分析包**：在R语言中一共有三个package可以进行RD分析：

-   **rdd**:
    由Dimmery在2016年发布，最后一次更新是2016-3-14，貌似目前没有获得积极的更新
-   **rddtools**：由Stigler &
    Quast最早在2013年发布，最后一次更新是2015-07-27，貌似目前也没有获得积极的更新
-   **rdrobust**：由Sebastian Calonico, Matias D. Cattaneo, Max H.
    Farrell等在2016年发布的，最后一次更新是2018-09-26，
    是目前功能最完善的RD分析包，同时也开发了Stata版本。该包的作者同时也是RD方法学上的大佬，多种bandwidth选择方法的发明者，著名的CCT就是。

> rdrobust简介：目前该包的开发项目可在<a href="https://sites.google.com/site/rdpackages/home" class="uri">https://sites.google.com/site/rdpackages/home</a>主页上查看，该包共有三个函数：
> + rdplot：用来画图 + rdrobust：用来进行局部非参数估计 +
> rdbwselect：用来进行bandwidth选择
> 另外还有一个协同的包**rddensity**，需要从CRAN上单独安装，用来进行对驱动变量是否被操控进行检验，因为rdrobust包未包含McCrary检验

**示列数据集**：采用rdrobust包中自带的*rdrobust\_RDsenate*数据集：

-   数据集简介：该数据集包含了1914–2010年间美国参议院选举的数据，可以利用RD的方法分析民主党赢得参议院席位对于
    在下次选举中获得的相同席位的影响。该数据共包含两个变量：  
    &gt; +
    vote：记录了州级层面下的参议院议席中民主党所占的比例，值从0到100，是RD分析中结局变量 &gt; +
    margin：记录了民主党在上次选举中获得相同参议院席位的胜利边际，值从-100到100，当大于0时，说明民主党胜利，小于0则输了，是RD分析中的驱动变量

### 6.1 Step 1: 确定驱动变量(Rating variable)和断点(Cut-point)

---

根据研究设计，判断是否存在采用RD进行因果识别或者效应估计的可能，即是否可以找到合适的驱动变量和明确的断点来识别施加干预的状态。

通常情况下时间（具体到天）、年龄（如研究退休，代表性的文章:
[“退休影响会健康吗”](http://cpfd.cnki.com.cn/Article/CPFDTOTAL-BDGF201007002017.htm)）、地理距离（研究雾霾，代表性文章:
[“冬季供暖导致雾霾？来自华北
城市面板的证据”](http://www.cnki.com.cn/Article/CJFDTOTAL-NKJJ201704002.htm)）比较容易作为驱动变量。

### 6.2 Step 2: 内部有效性(Internal Validity)条件的检验

---

#### 条件一：通过画驱动变量的频率分布直方图或者密度分布图，以及McCrary检验

可以利用graphics包或者ggplot2包画histogram图，也可以利用**rddenstiy**包中的*rddenstiy*函数画密度图，
也可利用**rdd**包进行McCrary检验。

*rddenstiy*函数的其他参数可以用默认值，输出结果中，最下面一行robust的p值大于0.05，就说明驱动变量未受到操纵。

    library(rdrobust)
    library(rddensity)
    library(magrittr)
    data(rdrobust_RDsenate)
    
    # 方法一
    hist(x = rdrobust_RDsenate$margin, breaks = 30, col = 'red',
         xlab = 'margin', ylab = 'Frequance', main = 'Histogram of Rating Variable')
    abline(v = 0)

![](https://github.com/shumchi/shumchi.github.io/blob/master/_posts/2019-11-06-Step-by-Step-to-do-a-Regression-Discontinuity-analysis_files/figure-markdown_strict/Chunk-1-1.png?raw=true)

    # 方法二
    library(ggplot2)
    ggplot(rdrobust_RDsenate) +
        geom_histogram(aes(x = margin, fill = margin < 0), color = 'black', binwidth = 4, boundary = 4) +
        geom_vline(xintercept = 0, size = 1) +
        labs(title = 'Histogram of the Rating varibale', y = 'Number of Observation', x = 'Margin') + 
        scale_fill_manual(name = '', values = c('red', 'blue'),
                          labels = c('Control', 'Treat')) +
        theme_bw() +
            theme(legend.position = c(.9, .9),
                  legend.background = element_blank())

![](https://github.com/shumchi/shumchi.github.io/blob/master/_posts/2019-11-06-Step-by-Step-to-do-a-Regression-Discontinuity-analysis_files/figure-markdown_strict/Chunk-1-2.png?raw=true)

    # 方法三
    rdplotdensity(rddensity(rdrobust_RDsenate$margin), X = rdrobust_RDsenate$margin)

![](https://github.com/shumchi/shumchi.github.io/blob/master/_posts/2019-11-06-Step-by-Step-to-do-a-Regression-Discontinuity-analysis_files/figure-markdown_strict/Chunk-1-3.png?raw=true)

    $Estl
    Call: lpdensity
    
    Sample size                                      640
    Polynomial order for point estimation    (p=)    2
    Order of derivative estimated            (v=)    1
    Polynomial order for confidence interval (q=)    3
    Kernel function                                  triangular
    Scaling factor                                   0.460043196544276
    Bandwidth method                                 user provided
    
    Use summary(...) to show estimates.
    
    $Estr
    Call: lpdensity
    
    Sample size                                      750
    Polynomial order for point estimation    (p=)    2
    Order of derivative estimated            (v=)    1
    Polynomial order for confidence interval (q=)    3
    Kernel function                                  triangular
    Scaling factor                                   0.539236861051116
    Bandwidth method                                 user provided
    
    Use summary(...) to show estimates.
    
    $Estplot

#### 条件二三四根据研究设计及背景资料进行判断



### 6.3 Step 3: 画驱动变量与干预的关系图，判断sharp或者fuzzy类型

---

画驱动变量与干预的散点图，判断是否为sharp或者fuzzy类型。

可以看出本例中，干预在cut-point前后，有明确的界限，为sharp类型。

    rdrobust_RDsenate$treatment <- ifelse(rdrobust_RDsenate$margin < 0, 0, 1)
    rdrobust_RDsenate$color <- ifelse(rdrobust_RDsenate$margin < 0, 'blue', 'red')
    
    plot(x = rdrobust_RDsenate$margin, y = rdrobust_RDsenate$treatment, col = rdrobust_RDsenate$color,
         type = 'p', pch = 16, xlab = 'Margin', ylab = 'Treatment',
         main = 'Relationship between rating variable and treatment')
    abline(v = 0)

![](https://github.com/shumchi/shumchi.github.io/blob/master/_posts/2019-11-06-Step-by-Step-to-do-a-Regression-Discontinuity-analysis_files/figure-markdown_strict/Chunk-2-1.png?raw=true)

### 6.4 Step 4: 画驱动变量与结果变量的关系图，选择合适的bin数量

---

#### 一、横轴为驱动变量，纵轴为结果变量，画散点图，初步判断两者之间关系

但是散点图噪音(noise)太大，不利于发现跳跃点，需要对驱动变量分不同的段(intervals
or bin)将使得关系取线更平顺

    # 第一步：散点图
    
    plot(x = rdrobust_RDsenate$margin, y = rdrobust_RDsenate$vote, type = 'p',
         col = rdrobust_RDsenate$color, pch = 16, cex = 0.8, xlab = 'Margin', ylab = 'Vote')
    abline(v = 0)

![](https://github.com/shumchi/shumchi.github.io/blob/master/_posts/2019-11-06-Step-by-Step-to-do-a-Regression-Discontinuity-analysis_files/figure-markdown_strict/Chunk-3-1.png?raw=true)

#### 二、将驱动变量分段的步骤

将驱动变量分段后再做关系图，大致有四步：

-   将驱动变量分成若干宽度(width)相等的区间(bin)，但是需要注意的是不能存在骑跨cut-point的区间，
    所以最好从cut-point开始分别向左右两端进行划分，并不一定要求左右两边的bin数量相等。  
-   计算每个bin中的结局变量的平均值、驱动变量的中位数，以及个体的数量(num
    of observation)
-   将驱动变量的中位数作为横轴，将结局变量的平均值作为纵轴，同时用每个bin中个体的数量进行加权
-   为了更好的显示两变量之间的关系，最好加上局部加权平滑回归取线(如：lowess:
    locally weighted regression)

#### 三、确定bins数量的方法

如何设定bin的数量或者binwidth的方法比较复杂，bin的数量不宜过多或过少，过少起不到将噪音的作用，过多会导致jump不明显，
在Stata和R中有打包的好的方法去判断bin的数量，本例中采用**rdrobust**包中的*rdplot*函数进行判断。

***rdplot*函数的常用参数有**：

-   c为cut-point，默认为0
-   p为全局多项式的幂次方，默认为4
-   nbins用来设定断点左右的bin的数量，默认为NULL
-   kernel用来设定加权方法，默认为uniform，同时还有triangular和epanechnikov方法
-   binselect为判断方法，默认为’esmv’，
    判断方法常用一共4种：Evenly-spaced (es)，Quantile-spaced
    (qs)，以及结合mimicking-variance (MV)的esmv和qsmv， 推荐使用qs方法。

**p和nbin如果不确定也可缺失，让binselect方法自行设定**

从结果可以看出总的样本量，以及cut-point左右的样本量，以及设定的bin的数量。

    bins <- rdplot(rdrobust_RDsenate$vote, rdrobust_RDsenate$margin, c = 0, p = 4,
                   nbins = c(20, 20), binselect = 'esmv', kernel = 'uniform')
    summary(bins)
    
    Call: rdplot
    
    Number of Obs.                 1297
    Kernel                      Uniform
    
    Number of Obs.                 595            702
    Eff. Number of Obs.            595            702
    Order poly. fit (p)              4              4
    BW poly. fit (h)           100.000        100.000
    Number of bins scale             1              1
    
    Bins Selected                   20             20
    Average Bin Length           5.000          5.000
    Median Bin Length            5.000          5.000
    
    IMSE-optimal bins                8              9
    Mimicking Variance bins         15             35
    
    Relative to IMSE-optimal:
    Implied scale                2.500          2.222
    WIMSE variance weight        0.060          0.084
    WIMSE bias weight            0.940          0.916
    
    rdplot(rdrobust_RDsenate$vote, rdrobust_RDsenate$margin, c = 0, p = 4,
           nbins = c(20, 20), binselect = 'esmv', kernel = 'uniform')

![](https://github.com/shumchi/shumchi.github.io/blob/master/_posts/2019-11-06-Step-by-Step-to-do-a-Regression-Discontinuity-analysis_files/figure-markdown_strict/Chunk-4-1.png?raw=true)

### 6.5 Step 5: 模型估计效应大小

---

**效应大小的估计方法一共有两种：**

-   全局参数估计(Parametric/global strategy):
    即是利用全部个体的数据，对驱动变量和结局变量拟合线性、二次项、三次项、四次项回归模型，
    此方法利用的是RD设计的“断点处不连续(discontinuity at a
    cut-point)”的特性。模型设定如下：  
    *Y*<sub>*i*</sub> = *α* + *β*<sub>0</sub> ⋅ *T**r**e**a**t**m**e**n**t*<sub>*i*</sub> + *β*<sub>1</sub> ⋅ (*r**a**t**i**n**g*<sub>*i*</sub> − *c**u**t**p**o**i**n**t*)<sup>*k*</sup> + *β*<sub>3</sub> ⋅ *T**r**e**a**t**m**e**n**t*<sub>*i*</sub> ⋅ (*r**a**t**i**n**g*<sub>*i*</sub> − *c**u**t**p**o**i**n**t*)<sup>*k*</sup> + *β*<sub>4</sub> ⋅ *C**o**n**f**o**u**d**e**r**s*<sub>*i*</sub> + *ϵ*<sub>*i*</sub>
    
    **式中**：  
    &gt; 1. Y为结局变量，*α*
    为常数项，表示在控制驱动变量后干预组结局变量的平均值，k为多项式的幂次，通常为1到4次方，其他参数含义见字面意思， &gt; 2.
    *β*<sub>0</sub>就是核心关注的系数，表示干预措施在断点处的边际效应，也就是干预的处理效应。  
    &gt; 3.
    而如何选择合适的次项数，需要多次尝试，通过比较AIC，选择AIC最小的模型。 &gt; 4.
    驱动变量减去cutpoint的目的是为了中心化，为了使*α*反映的是断点处的平均值，因为差值在断点处为0 &gt; 5.
    Confouders为混杂因素，也可以加入模型中，但是中RD分析时，混杂因素的控制不是必须的。

-   局部非参数估计(Nonparametric/local strategy):
    即是在cut-point左右选择一段合适的带宽(bandwidth)，在这一段带宽局部范围内，
    拟合驱动变量和结局变量的线性或者多项式回归模型，此方法利用的是RD设计的“局部随机(local
    randomization)”的特性。此方法适用于样本量较大的情况，
    因为样本量太少，设定带宽之后样本量将会不足，增加估计误差。前面所提到的R包中均只提供局部非参数估计函数。  
    **局部非参数估计的难点在于如何确定最优带宽，常用的方法有交叉验证法(cross
    validation procedure， CV)、 IK 法和 CCT 法，后两种方法较为推荐**

<br>

**关于上述两种方法的比较或者权衡，主要是关于精度和误差，全局参数估计因为利用了全部样本，通常对效应的估计精度更高，然而其也存在一个问题，
就是驱动变量和结局变量的关系函数在越大的区间下越难以准确识别；而非参数估计可以通过局部拟合加权线性或者多项式回归模型而减少这种偏误。**

<br>

**以上方法主要针对sharp类型，fuzzy类型的估计方法见最后一节**

<br>

#### （1）全局参数估计

分为一次线性、一次线性加交互项、以及二次、三次线性及交互项，共6个模型，比较AIC或者回归残差，较小者模型较优。

    rdrobust_RDsenate$margin_del <- rdrobust_RDsenate$margin - 0
    
    fit_1 <- lm(vote ~ margin_del + treatment, data = rdrobust_RDsenate) # linear
    fit_2 <- lm(vote ~ margin_del * treatment, data = rdrobust_RDsenate) # linear interaction
    fit_3 <- lm(vote ~ margin_del + I(margin_del ^ 2) + treatment, data = rdrobust_RDsenate) # quadratic
    fit_4 <- lm(vote ~ (margin_del + I(margin_del ^ 2)) * treatment, data = rdrobust_RDsenate) # quadratic interaction
    fit_5 <- lm(vote ~ margin_del + I(margin_del ^ 2) + I(margin_del ^ 3) + treatment, data = rdrobust_RDsenate) # cubic
    fit_6 <- lm(vote ~ (margin_del + I(margin_del ^ 2) + I(margin_del ^ 3)) * treatment, data = rdrobust_RDsenate) # cubic interaction
    
    stargazer::stargazer(fit_1, fit_2, fit_3, fit_4, fit_5, fit_6, type = 'html', style = 'all')

<table style="text-align:center"><tr><td colspan="7" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"></td><td colspan="6"><em>Dependent variable:</em></td></tr>
    <tr><td></td><td colspan="6" style="border-bottom: 1px solid black"></td></tr>
    <tr><td style="text-align:left"></td><td colspan="6">vote</td></tr>
    <tr><td style="text-align:left"></td><td>(1)</td><td>(2)</td><td>(3)</td><td>(4)</td><td>(5)</td><td>(6)</td></tr>
    <tr><td colspan="7" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">margin_del</td><td>0.348<sup>***</sup></td><td>0.216<sup>***</sup></td><td>0.285<sup>***</sup></td><td>0.421<sup>***</sup></td><td>0.337<sup>***</sup></td><td>0.285<sup>*</sup></td></tr>
    <tr><td style="text-align:left"></td><td>(0.013)</td><td>(0.028)</td><td>(0.017)</td><td>(0.069)</td><td>(0.028)</td><td>(0.151)</td></tr>
    <tr><td style="text-align:left"></td><td>t = 26.078</td><td>t = 7.803</td><td>t = 16.873</td><td>t = 6.133</td><td>t = 12.175</td><td>t = 1.884</td></tr>
    <tr><td style="text-align:left"></td><td>p = 0.000</td><td>p = 0.000</td><td>p = 0.000</td><td>p = 0.000</td><td>p = 0.000</td><td>p = 0.060</td></tr>
    <tr><td style="text-align:left">I(margin_del2)</td><td></td><td></td><td>0.001<sup>***</sup></td><td>0.003<sup>***</sup></td><td>0.001<sup>***</sup></td><td>-0.002</td></tr>
    <tr><td style="text-align:left"></td><td></td><td></td><td>(0.0002)</td><td>(0.001)</td><td>(0.0002)</td><td>(0.005)</td></tr>
    <tr><td style="text-align:left"></td><td></td><td></td><td>t = 6.016</td><td>t = 3.254</td><td>t = 6.325</td><td>t = -0.349</td></tr>
    <tr><td style="text-align:left"></td><td></td><td></td><td>p = 0.000</td><td>p = 0.002</td><td>p = 0.000</td><td>p = 0.727</td></tr>
    <tr><td style="text-align:left">I(margin_del3)</td><td></td><td></td><td></td><td></td><td>-0.00001<sup>**</sup></td><td>-0.00004</td></tr>
    <tr><td style="text-align:left"></td><td></td><td></td><td></td><td></td><td>(0.00000)</td><td>(0.00004)</td></tr>
    <tr><td style="text-align:left"></td><td></td><td></td><td></td><td></td><td>t = -2.386</td><td>t = -1.011</td></tr>
    <tr><td style="text-align:left"></td><td></td><td></td><td></td><td></td><td>p = 0.018</td><td>p = 0.313</td></tr>
    <tr><td style="text-align:left">treatment</td><td>4.785<sup>***</sup></td><td>6.044<sup>***</sup></td><td>6.663<sup>***</sup></td><td>4.935<sup>***</sup></td><td>5.238<sup>***</sup></td><td>7.319<sup>***</sup></td></tr>
    <tr><td style="text-align:left"></td><td>(0.923)</td><td>(0.942)</td><td>(0.963)</td><td>(1.256)</td><td>(1.131)</td><td>(1.593)</td></tr>
    <tr><td style="text-align:left"></td><td>t = 5.184</td><td>t = 6.414</td><td>t = 6.922</td><td>t = 3.929</td><td>t = 4.630</td><td>t = 4.595</td></tr>
    <tr><td style="text-align:left"></td><td>p = 0.00000</td><td>p = 0.000</td><td>p = 0.000</td><td>p = 0.0001</td><td>p = 0.00001</td><td>p = 0.00001</td></tr>
    <tr><td style="text-align:left">margin_del:treatment</td><td></td><td>0.170<sup>***</sup></td><td></td><td>-0.095</td><td></td><td>-0.229</td></tr>
    <tr><td style="text-align:left"></td><td></td><td>(0.032)</td><td></td><td>(0.087)</td><td></td><td>(0.195)</td></tr>
    <tr><td style="text-align:left"></td><td></td><td>t = 5.406</td><td></td><td>t = -1.088</td><td></td><td>t = -1.175</td></tr>
    <tr><td style="text-align:left"></td><td></td><td>p = 0.00000</td><td></td><td>p = 0.277</td><td></td><td>p = 0.241</td></tr>
    <tr><td style="text-align:left">I(margin_del2):treatment</td><td></td><td></td><td></td><td>-0.002<sup>**</sup></td><td></td><td>0.010<sup>*</sup></td></tr>
    <tr><td style="text-align:left"></td><td></td><td></td><td></td><td>(0.001)</td><td></td><td>(0.006)</td></tr>
    <tr><td style="text-align:left"></td><td></td><td></td><td></td><td>t = -2.242</td><td></td><td>t = 1.770</td></tr>
    <tr><td style="text-align:left"></td><td></td><td></td><td></td><td>p = 0.026</td><td></td><td>p = 0.077</td></tr>
    <tr><td style="text-align:left">I(margin_del3):treatment</td><td></td><td></td><td></td><td></td><td></td><td>-0.00002</td></tr>
    <tr><td style="text-align:left"></td><td></td><td></td><td></td><td></td><td></td><td>(0.00004)</td></tr>
    <tr><td style="text-align:left"></td><td></td><td></td><td></td><td></td><td></td><td>t = -0.411</td></tr>
    <tr><td style="text-align:left"></td><td></td><td></td><td></td><td></td><td></td><td>p = 0.682</td></tr>
    <tr><td style="text-align:left">Constant</td><td>47.331<sup>***</sup></td><td>44.904<sup>***</sup></td><td>45.486<sup>***</sup></td><td>46.740<sup>***</sup></td><td>45.997<sup>***</sup></td><td>46.029<sup>***</sup></td></tr>
    <tr><td style="text-align:left"></td><td>(0.542)</td><td>(0.699)</td><td>(0.616)</td><td>(0.896)</td><td>(0.651)</td><td>(1.138)</td></tr>
    <tr><td style="text-align:left"></td><td>t = 87.340</td><td>t = 64.221</td><td>t = 73.798</td><td>t = 52.153</td><td>t = 70.614</td><td>t = 40.457</td></tr>
    <tr><td style="text-align:left"></td><td>p = 0.000</td><td>p = 0.000</td><td>p = 0.000</td><td>p = 0.000</td><td>p = 0.000</td><td>p = 0.000</td></tr>
    <tr><td colspan="7" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">Observations</td><td>1,297</td><td>1,297</td><td>1,297</td><td>1,297</td><td>1,297</td><td>1,297</td></tr>
    <tr><td style="text-align:left">R<sup>2</sup></td><td>0.578</td><td>0.587</td><td>0.590</td><td>0.591</td><td>0.591</td><td>0.593</td></tr>
    <tr><td style="text-align:left">Adjusted R<sup>2</sup></td><td>0.577</td><td>0.586</td><td>0.589</td><td>0.590</td><td>0.590</td><td>0.591</td></tr>
    <tr><td style="text-align:left">Residual Std. Error</td><td>11.781 (df = 1294)</td><td>11.654 (df = 1293)</td><td>11.624 (df = 1293)</td><td>11.609 (df = 1291)</td><td>11.603 (df = 1292)</td><td>11.587 (df = 1289)</td></tr>
    <tr><td style="text-align:left">F Statistic</td><td>886.433<sup>***</sup> (df = 2; 1294) (p = 0.000)</td><td>613.586<sup>***</sup> (df = 3; 1293) (p = 0.000)</td><td>619.091<sup>***</sup> (df = 3; 1293) (p = 0.000)</td><td>373.392<sup>***</sup> (df = 5; 1291) (p = 0.000)</td><td>467.426<sup>***</sup> (df = 4; 1292) (p = 0.000)</td><td>268.726<sup>***</sup> (df = 7; 1289) (p = 0.000)</td></tr>
    <tr><td colspan="7" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"><em>Note:</em></td><td colspan="6" style="text-align:right"><sup>*</sup>p<0.1; <sup>**</sup>p<0.05; <sup>***</sup>p<0.01</td></tr>
    </table>


    AIC(fit_1, fit_2, fit_3, fit_4, fit_5, fit_6)
    
          df      AIC
    fit_1  4 10083.70
    fit_2  5 10056.71
    fit_3  5 10049.89
    fit_4  7 10048.72
    fit_5  6 10046.19
    fit_6  9 10045.74

<br> 

#### （2）局部非参数估计

利用**rdrobust**包的*rdrobust*函数进行拟合。由于该包是由Sebastian
Calonico, Matias D. Cattaneo and Rocío Titiunik开发，
所以该包中所提供的带宽选择方法可认为就是CCT法，因为CCT本来就是Calonico,
Cattaneo, and Titiunik三人姓氏的缩写。

**有两种确定带宽的方法**：

-   一是先利用*rdbwselect*函数计算出最优带宽，然后用*rdrobust*函数中的h参数手动指定左右的带宽。  
-   二是直接利用*rdrobust*函数中的bdselect参数指定选择最优带宽的方法。

**rdrobust**包中一共提供了10种选择最优带宽的方法，如下:

-   其中，共包含MSE = Mean Square Error和CER = Coverage Error
    Rate两大类，MSE更适用于进行点估计的带宽选择，
    而CER更适合区间估计的带宽选择。  
-   以rd结尾是表示选择的带宽在cut-point左右相等，而以two结尾是表示选择的带宽在cut-point左右不相等。
-   h即为选择的左右最优带宽，而b给出的是用来进行敏感性分析时应该考虑的带宽。

<!-- -->

    rdbwselect(y = rdrobust_RDsenate$vote, x = rdrobust_RDsenate$margin, all = TRUE) %>% summary()
    
    Call: rdbwselect
    
    Number of Obs.                 1297
    BW type                         All
    Kernel                   Triangular
    VCE method                       NN
    
    Number of Obs.                 595         702
    Order est. (p)                   1           1
    Order bias  (q)                  2           2
    
    =======================================================
                      BW est. (h)    BW bias (b)
                Left of c Right of c  Left of c Right of c
    =======================================================
         mserd    17.708     17.708     27.984     27.984
        msetwo    16.154     18.009     27.096     29.205
        msesum    18.326     18.326     31.280     31.280
      msecomb1    17.708     17.708     27.984     27.984
      msecomb2    17.708     18.009     27.984     29.205
         cerrd    12.374     12.374     27.984     27.984
        certwo    11.288     12.585     27.096     29.205
        cersum    12.806     12.806     31.280     31.280
      cercomb1    12.374     12.374     27.984     27.984
      cercomb2    12.374     12.585     27.984     29.205
    =======================================================

<br>

虽然在此文中[rdrobust: An R Package for Robust Nonparametric Inference
in Regression-Discontinuity
Designs](https://journal.r-project.org/archive/2015/RJ-2015-004/RJ-2015-004.pdf)
提到也可以通过bdselect = ’CV’或者bdselect =
’IK’来采用CV和IK法，但是在写这篇笔记时，已无法使用这两种方法。

输出的结果的最后一张表的第一行Conventional即为干预的处理效应。

    loc_fit_1 <- rdrobust(rdrobust_RDsenate$vote, rdrobust_RDsenate$margin, c = 0, p = 1,
                          kernel = 'triangular', bwselect = 'msetwo') # 如果有混杂因素需要控制,可以用covs = c('var1', 'var2')
    loc_fit_2 <- rdrobust(rdrobust_RDsenate$vote, rdrobust_RDsenate$margin, c = 0, p = 2,
                          kernel = 'triangular', bwselect = 'msetwo') # c用来指定cut-point，p用来指定局部加权回归的多项式幂次
    loc_fit_3 <- rdrobust(rdrobust_RDsenate$vote, rdrobust_RDsenate$margin, c = 0, p = 1,
                          kernel = 'triangular', bwselect = 'cerrd')
    loc_fit_4 <- rdrobust(rdrobust_RDsenate$vote, rdrobust_RDsenate$margin, c = 0, p = 2,
                          kernel = 'triangular', bwselect = 'certwo')
    
    summary(loc_fit_1)
    
    Call: rdrobust
    
    Number of Obs.                 1297
    BW type                      msetwo
    Kernel                   Triangular
    VCE method                       NN
    
    Number of Obs.                 595         702
    Eff. Number of Obs.            336         325
    Order est. (p)                   1           1
    Order bias  (p)                  2           2
    BW est. (h)                 16.154      18.009
    BW bias (b)                 27.096      29.205
    rho (h/b)                    0.596       0.617
    
    =============================================================================
            Method     Coef. Std. Err.         z     P>|z|      [ 95% C.I. ]       
    =============================================================================
      Conventional     7.453     1.499     4.973     0.000     [4.516 , 10.390]    
            Robust         -         -     4.277     0.000     [4.081 , 10.983]    
    =============================================================================

### 6.6 Step 6: 敏感性分析

---

敏感性分析主要用来检验模型估计结果的稳健性，RD分析主要有四种敏感性分析方式;

-   1.  对前面所述的Internal
        Validity条件三的检验，即除了干预(Treatment)变量外，其他的任何非结局变量(包括混杂因素在内)都不能在cut-point处
        出现不连续或称为跳跃，如果存在，则无法推断结局变量的跳跃是由干预(Treatment)引起的，检验方式就是将所有的混杂因素作为结局，利用*rdrobust*函数进行检验，
        输出结果的coef应该很小，且p值应该统计学不显著 (Continuity-Based
        Analysis for Covariates)。

-   1.  对cut-point的敏感性分析，即是更换cut-point，检验是否左右还存在处理效应，如果更换断点后，仍然存在处理效应，
        则无法说明本研究的干预措施是有效的，因为在研究设定的断点处识别到的处理效应，有可能是由其他因素引起的。

-   1.  对cut-point附近个体的敏感性分析，在Internal
        Validity条件一中提到，驱动变量不可被操控，但是这个无法直接检验，因此，换个思路，如果驱动变量被超控，
        那自然是在cut-point左右离得最近的值被操纵的可能性最大，所以如果将这部分个体剔除，若仍然能观测到处理效应，则说明这种效应是真实由干预所导致。
        这种方法被称为甜甜圈(donut
        hole)法，同样也适用于个体过多的堆积与cut-point附近的RD分析。

-   1.  对带宽bandwidth的敏感性分析，即是cut-point不改变，而是更换选择的最优带宽，进行多次局部非参数检验，如果更换带宽之后，仍然能在断点处识别的处理效应，
        说明研究的干预措施是有效的，因为在研究设定的断点处识别到的处理效应不是由于某一特点的带宽下才观测到的，说明处理效应稳健。

#### 对cut-point的敏感性分析

选择一个虚拟的断点用来替换真正的断点，但是真实的干预情况不改变。  
关于虚拟断点如何选择以及选择多少个并没有明确的标准，通常在真实断点的附近，左右对称选取即可

    sen_cut_1 <- rdrobust(rdrobust_RDsenate$vote, rdrobust_RDsenate$margin, c = 1, p = 1,
                          kernel = 'triangular', bwselect = 'msetwo') # 除c意外的其他参数应该与前面分析保持一致
    sen_cut_2 <- rdrobust(rdrobust_RDsenate$vote, rdrobust_RDsenate$margin, c = -1, p = 1,
                          kernel = 'triangular', bwselect = 'msetwo') # 除c意外的其他参数应该与前面分析保持一致
    
    summary(sen_cut_1)
    
    Call: rdrobust
    
    Number of Obs.                 1297
    BW type                      msetwo
    Kernel                   Triangular
    VCE method                       NN
    
    Number of Obs.                 620         677
    Eff. Number of Obs.            370         346
    Order est. (p)                   1           1
    Order bias  (p)                  2           2
    BW est. (h)                 17.832      21.431
    BW bias (b)                 32.113      35.123
    rho (h/b)                    0.555       0.610
    
    =============================================================================
            Method     Coef. Std. Err.         z     P>|z|      [ 95% C.I. ]       
    =============================================================================
      Conventional     5.264     1.406     3.744     0.000     [2.508 , 8.020]     
            Robust         -         -     3.023     0.003     [1.744 , 8.174]     
    =============================================================================
    
    summary(sen_cut_2)
    
    Call: rdrobust
    
    Number of Obs.                 1297
    BW type                      msetwo
    Kernel                   Triangular
    VCE method                       NN
    
    Number of Obs.                 577         720
    Eff. Number of Obs.            377         200
    Order est. (p)                   1           1
    Order bias  (p)                  2           2
    BW est. (h)                 20.113       9.823
    BW bias (b)                 32.031      21.323
    rho (h/b)                    0.628       0.461
    
    =============================================================================
            Method     Coef. Std. Err.         z     P>|z|      [ 95% C.I. ]       
    =============================================================================
      Conventional     1.008     1.614     0.625     0.532    [-2.154 , 4.170]     
            Robust         -         -     0.217     0.828    [-3.249 , 4.059]     
    =============================================================================

#### 对cut-point附近个体的敏感性分析

对称剔除真实断点左右一定较小范围内的个体。  
关于范围的大小同样没有明确的标准，通常选取多个范围比较即可

    rdrobust_RD_hole_1 <- subset(rdrobust_RDsenate, abs(margin) > 0.3)
    rdrobust_RD_hole_2 <- subset(rdrobust_RDsenate, abs(margin) > 0.4)
    rdrobust_RD_hole_3 <- subset(rdrobust_RDsenate, abs(margin) > 0.5)
    
    sen_hole_1 <- rdrobust(rdrobust_RD_hole_1$vote, rdrobust_RD_hole_1$margin, c = 0, p = 1,
                          kernel = 'triangular', bwselect = 'msetwo') # 除c意外的其他参数应该与前面分析保持一致
    sen_hole_2 <- rdrobust(rdrobust_RD_hole_2$vote, rdrobust_RD_hole_2$margin, c = 0, p = 1,
                          kernel = 'triangular', bwselect = 'msetwo') # 除c意外的其他参数应该与前面分析保持一致
    sen_hole_3 <- rdrobust(rdrobust_RD_hole_3$vote, rdrobust_RD_hole_3$margin, c = 0, p = 1,
                          kernel = 'triangular', bwselect = 'msetwo') # 除c意外的其他参数应该与前面分析保持一致
    
    summary(sen_hole_1)
    
    Call: rdrobust
    
    Number of Obs.                 1288
    BW type                      msetwo
    Kernel                   Triangular
    VCE method                       NN
    
    Number of Obs.                 592         696
    Eff. Number of Obs.            334         300
    Order est. (p)                   1           1
    Order bias  (p)                  2           2
    BW est. (h)                 16.332      16.543
    BW bias (b)                 27.424      28.160
    rho (h/b)                    0.596       0.587
    
    =============================================================================
            Method     Coef. Std. Err.         z     P>|z|      [ 95% C.I. ]       
    =============================================================================
      Conventional     6.911     1.559     4.434     0.000     [3.856 , 9.966]     
            Robust         -         -     3.735     0.000     [3.265 , 10.478]    
    =============================================================================
    
    summary(sen_hole_2)
    
    Call: rdrobust
    
    Number of Obs.                 1282
    BW type                      msetwo
    Kernel                   Triangular
    VCE method                       NN
    
    Number of Obs.                 590         692
    Eff. Number of Obs.            354         313
    Order est. (p)                   1           1
    Order bias  (p)                  2           2
    BW est. (h)                 17.472      17.836
    BW bias (b)                 28.932      30.388
    rho (h/b)                    0.604       0.587
    
    =============================================================================
            Method     Coef. Std. Err.         z     P>|z|      [ 95% C.I. ]       
    =============================================================================
      Conventional     6.778     1.559     4.346     0.000     [3.721 , 9.834]     
            Robust         -         -     3.579     0.000     [3.014 , 10.310]    
    =============================================================================
    
    summary(sen_hole_3)
    
    Call: rdrobust
    
    Number of Obs.                 1274
    BW type                      msetwo
    Kernel                   Triangular
    VCE method                       NN
    
    Number of Obs.                 586         688
    Eff. Number of Obs.            350         335
    Order est. (p)                   1           1
    Order bias  (p)                  2           2
    BW est. (h)                 17.670      20.322
    BW bias (b)                 29.850      35.253
    rho (h/b)                    0.592       0.576
    
    =============================================================================
            Method     Coef. Std. Err.         z     P>|z|      [ 95% C.I. ]       
    =============================================================================
      Conventional     7.005     1.592     4.400     0.000     [3.885 , 10.125]    
            Robust         -         -     3.677     0.000     [3.294 , 10.815]    
    =============================================================================

#### 对带宽bandwidth的敏感性分析

即是更换不同的带宽即可，带宽的变化实际是cut-point区间范围尾部的个体在发生变化，
可以通过选取不同的带宽选择方式，真实存在的处理效应不会随着带宽的改变而发生变化。

7 关于Fuzzy类型的RD模型估计方法
-------------------------------

Fuzzy类型的RD可以根据驱动变量与干预的实际情况采取不同的处理方式:

-   如果只是no-shows的情况，可以结合研究目的，决定是否采取意向性分析法(intent-to-treat)

-   采用两阶段回归two-stage least squares (2SLS) ，两阶段模型设定如下:  
    第一阶段：  
    *T**r**e**a**t**m**e**n**t*<sub>*i*</sub> = *α*<sub>1</sub> + *γ*<sub>0</sub> ⋅ *t**r**e**a**t* − *b**y* − *c**u**t**p**o**i**n**t*<sub>*i*</sub> + *ϵ*<sub>*i*</sub>
    
    第二阶段：  
    *Y*<sub>*i*</sub> = *α*<sub>*i*</sub> + *β*<sub>0</sub> ⋅ *Ṫ**r**e**a**t**m**e**n**t* + *μ*<sub>*i*</sub>
    其中： &gt; 1.
    *T**r**e**a**t**m**e**n**t*<sub>*i*</sub>为真实的干预情况 &gt; 2.
    *t**r**e**a**t* − *b**y* − *c**u**t**p**o**i**n**t*<sub>*i*</sub>是指驱动变量根据cut-point判断的是否接受了干预，为虚拟变量 &gt; 3.
    第二阶段模型中的*Ṫ**r**e**a**t**m**e**n**t*将第一阶段模型的预测指yhat

通过两阶段回归，第二阶段中的Standard errors是经过了调整的.

**在*rdrobust*包的rdrobust函数中，有一个`fuzzy`的逻辑参数，用来指定RD类型是否为fuzzy，可以很方便的进行fuzzy类型的模型估计。**

8 结束
------

由于真实世界的复杂性，对因果推断的分析方法的要求也越来越高，没有一种方法可以适用所有的情形，因此RD的方法也衍生发展出很多类型，
包括多个断点(Multi-Cutoff RD Design)、多个驱动变量(Multi-Score RD
Design)、离散驱动变量(Multi-Score RD Design)等等。

Reference
---------

[1. Jacob, Robin,etc.(2012). A Practical Guide to Regression
Discontinuity](https://www.mdrc.org/sites/default/files/regression_discontinuity_full.pdf)

[2. Matias D. Cattaneo, Nicol´as Idroboy, Roc´ıo Titiunik. (2017). A
Practical Introduction to Regression Discontinuity Designs: Volume
I](https://www.collingwoodresearch.com/uploads/8/3/6/0/8360930/cattaneo-idrobo-titiunik_2017_cambridge-part1.pdf)

[3. Sebastian Calonico, Matias D. Cattaneo and Rocío Titiunik. ().
rdrobust: An R Package for Robust Nonparametric Inference in
Regression-Discontinuity
Designs](https://journal.r-project.org/archive/2015/RJ-2015-004/RJ-2015-004.pdf)

[4.谢谦,薛仙玲,付明卫.断点回归设计方法应用的研究综述.经济与管理评论,2019(02):69-79](https://doi.org/10.13962/j.cnki.37-1486/f.2019.02.006)

