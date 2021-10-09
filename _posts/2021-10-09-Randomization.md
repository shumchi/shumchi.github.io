---
layout: post
title:  "实验研究中的随机化分组及R语言实现"
date:   2021-10-09
excerpt: "R"
author: Chi Shen
tags: [R, Randomization]
feature: ![Figure 1](https://github.com/shumchi/shumchi.github.io/blob/master/_posts/2020-09-21-M12-Final/figure-1.jpeg?raw=true)
comments: true
---

**Shen, Chi**

**Oct 21 2021, Xi‘an**  

# 摘要

------------------------------------------------------------------------

实验研究作为因果推断效能最高的一种研究设计，被广泛应用于医学及社会科学研究中。在实验研究中又以随机对照试验（Randomised
Controlled
Trials，RCT）最为常见。RCT研究一个最明显的特点就是随机化（Randomization）施加干预<sup>\[1\]</sup>，随机化和盲法是RCT研究的基石<sup>\[2\]</sup>。随机化干预的重要性在于可以有效避免选择或其他偏倚，提供统一的统计分析方法，以及为精确检验显著性和区间估计提供基础<sup>\[3,4\]</sup>。在具体的研究中，随机化干预的基础是对干预对象进行随机分组，然而，在现有针对本科生和研究生阶段学习的统计教材中，却缺少对随机分组方法以及其具体实现方法的详细介绍，一定程度上不利于青年研究者将RCT方法应用到具体的研究实践中。尽管已有文献报道了基于SAS和Stata的随机分组方法<sup>\[5–7\]</sup>，但是相较于SAS和Stata而言，R语言安装与使用更为简便，并且由于其开源免费的特性更易于获取。因此，本文基于已有文献，详细介绍三种随机分组的具体方法和操作过程，以及提供基于R语言的实现代码，供其他学者在具体研究中作为参考。

# 1. 随机分组方法简介

随机化的概念最早由Fisher于1926年提出<sup>\[8\]</sup>，其后被广泛应用于实验研究中，已成为控制干预组和对照组之间混杂偏倚的必要手段。随机化具体指将受试者按照相同的概率分配在干预组和非干预组的过程<sup>\[9\]</sup>，根据实验设计的样本大小以及受试者招募时间长短等因素的不同，随机化分组方法很多种，但较为常用的有三种[1]：简单随机分组（simple
randomization）、区组随机分组（block
randomization）和分层随机分组（stratified
randomization）<sup>\[10\]</sup>。具体如下：

1）简单随机分组是指按照一组随机序列将受试者以相同的概率分配至不同的实验组中，最典型的简单随机分组方法是抛硬币（flipping
a
coin）。简单随机分组操作容易，但是存在一个明显的不足，即当受试者样本量较小时，无法保证纳入不同实验组的受试者数量相等<sup>\[10,11\]</sup>。一般认为当受试者样本量小于200时不宜采用简单随机分组<sup>\[10\]</sup>。实际上，在具体的研究过程中（如临床药物试验），通常要求干预组和对照组所分配的受试者数量相等或达到一定的比例，因此即使样本量大于200简单随机分组也不适用。然而，在实际研究过程中，会对简单随机分组方法进行一定的变化以满足研究设计需要，具体见下节。

2）区组随机分组是指先根据受试者样本量大小及实验分组数量和组间比例设定好区组（block）的长度和数量，将特征相似或相同（如入组时间、地区来源）的受试者编入同一区组，然后在每个区组内部进行简单随机分组。区组随机分组可以有效保证不同的实验组中所分配的受试者数量严格满足研究设计要求。不仅如此，区组随机分组的另一个优势在于，当入组后的受试者出现某种不满足实验要求而需退出实验时，可以直接将该受试者所在的区组舍弃另外补充新的区组，而不破坏整个实验的随机化。

3）分层随机分组是指先按已知的混杂因素对受试者进行分层后，再在各个层内采用简单随机或者区组随机方法将受试者分配至不同的实验组。分层随机分组可以在实验设计阶段就对部分混杂因素进行控制，从而提高不同实验组中受试者的基线均衡性<sup>\[11\]</sup>。

需要补充的是，在实际研究过程中，很难一次性招募足够的受试者，然后再为受试者分配随机号进行随机分组工作。通常情况是会设定一段时间（一个月或三个月，视具体情况而定）的招募入组期，当有受试者满足入组条件后就需要进行随机分组，既而接受相应的干预，因此，这就需要在受试者入组之前准备好随机分组规则，受试者根据进入研究的顺序，依据预先准备好的随机分组表进入相应的实验组。

# 2. 随机分组的具体过程及R语言代码

## 2.1 简单随机分组（simple randomization）

简单随机分组的基本步骤一共有五点：1）按照实验设计的受试者样本量生成受试者编号；2）利用随机函数生成与受试者编号数量相等的随机数字；3）对生成的随机数字进行编秩；4）根据实验设计的组数及分组比例，对随机数字的秩序按照大小或者奇偶数进行分组；5）获得符合实验设计要求的随机分组表。本文提供了一段基于R语言的自编函数，能够满足两组及以上分组且不同组间比例的简单随机分组需求，自编函数代码及具体含义如下：

    simple_random <- function(size, grp = 2, T_2_C = "1:1"){
        set.seed(20210412)
        id_num <- seq(1, size, 1)
        random_seq <- runif(n = size, min = 0, max = 1)
        int_rank <- rank(random_seq)
        
        ratio_T <- as.numeric(substr(T_2_C, 1, 1))
        ratio_C <- as.numeric(substr(T_2_C, 3, 3))
        
        if (grp == 2) {
            group <- ifelse(int_rank <= size/(ratio_T + ratio_C), "T", "C")
        } else if (grp > 3) {
            group <- cut(int_rank, breaks = grp, labels = paste("Group", 1:grp))
        }
        
        df <- data.frame("ID" = id_num, "RandomNum" = random_seq, 
                            "Rank" = int_rank, "Group" = group)
        
        write.csv(df, "simple randomization table.csv", row.names = FALSE)
        
        return(df)
    }

1）函数名为simple\_random，函数共有3个参数，分别为size（样本量大小）、grp（分组数）和T\_2\_C（干预组与对照组比例），grp和T\_2\_C为默认参数，分别默认赋值为2和“1:1”；

2）set.seed()函数用于设定随机种子数[2]，以便下一步利用随机函数生成的随机数可再现，此步骤在利用计算机进行实验研究的随机化分组时十分关键，因为只有固定了随机种子数，才可以保证实验研究中生成的随机分组表可以复现，这一点对于研究论文的发表和临床药物试验质量稽查十分重要；

3）seq()函数用于生成从1开始的受试者编号；

4）runif()函数用于生成与受试者人数相等的服从均匀分布的0到1之间随机数字，其中min和max参数可以任意指定，此处也可采用R语言中自带的其他随机函数，如rnorm()等，也可以采用sample()函数直接生成非负随机整数；

5）rank()函数用于对生成的随机数字按照升序进行编秩；

6）ratio\_T和ratio\_C变量用于提取T\_2\_C参数的赋值，用于分组比例的计算；

7）if条件语句中的内容即是对随机数字的秩进行分组，其规则为：当为两组且组间比例为1:1时，随机数字的秩位于前50%的受试者编号设定为干预组（“T”），后50%对应的受试者编号设定为对照组（“C”），当为两组且组间比例为1:N时，随机数字的秩位于前1/(1+N)的受试者编号设定为干预组（“T”），剩余受试者编号设定为对照组（“C”），当为三组及以上时，本文提供的自编函数仅支持组间比例相等的情况，其分组规则为将随机数字的秩进行等分，与秩对应的受试者编号分别分入“Group
1”、“Group 2”…“Group N”；

8）data.frame()和write.csv()函数用于生成随机分组表并输出为csv格式，随机分组表中包含受试者编号、随机数字、随机数字的秩、分组。

实例一：为检验某一治疗高血压新药A的临床疗效，选择已上市B药作为对照组，开展单中心双臂临床试验，需要受试者240名，每组各120名受试者，请按简单随机分组方法制定随机分组表。利用本文提供的自编函数simple\_random()则可将参数size设定为240，
参数grp设定为2，参数T\_2\_C设定为“1:1”（注意此处为英文半角引号），具体如下：

    res <- simple_random(size = 240, grp = 2, T_2_C = "1:1")

函数输出的随机分组表见表1，其中分组为T和C的受试者分别为120名。

<table>
<caption>表1 简单随机分组表</caption>
<thead>
<tr class="header">
<th style="text-align: left;"></th>
<th style="text-align: right;">ID</th>
<th style="text-align: right;">RandomNum</th>
<th style="text-align: right;">Rank</th>
<th style="text-align: left;">Group</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">1</td>
<td style="text-align: right;">1</td>
<td style="text-align: right;">0.8323749</td>
<td style="text-align: right;">198</td>
<td style="text-align: left;">C</td>
</tr>
<tr class="even">
<td style="text-align: left;">2</td>
<td style="text-align: right;">2</td>
<td style="text-align: right;">0.9552218</td>
<td style="text-align: right;">228</td>
<td style="text-align: left;">C</td>
</tr>
<tr class="odd">
<td style="text-align: left;">3</td>
<td style="text-align: right;">3</td>
<td style="text-align: right;">0.5978788</td>
<td style="text-align: right;">134</td>
<td style="text-align: left;">C</td>
</tr>
<tr class="even">
<td style="text-align: left;">4</td>
<td style="text-align: right;">4</td>
<td style="text-align: right;">0.3507679</td>
<td style="text-align: right;">75</td>
<td style="text-align: left;">T</td>
</tr>
<tr class="odd">
<td style="text-align: left;">5</td>
<td style="text-align: right;">5</td>
<td style="text-align: right;">0.4315742</td>
<td style="text-align: right;">91</td>
<td style="text-align: left;">T</td>
</tr>
<tr class="even">
<td style="text-align: left;">6</td>
<td style="text-align: right;">6</td>
<td style="text-align: right;">0.6332632</td>
<td style="text-align: right;">147</td>
<td style="text-align: left;">C</td>
</tr>
<tr class="odd">
<td style="text-align: left;">7</td>
<td style="text-align: right;">7</td>
<td style="text-align: right;">0.7801558</td>
<td style="text-align: right;">189</td>
<td style="text-align: left;">C</td>
</tr>
<tr class="even">
<td style="text-align: left;">8</td>
<td style="text-align: right;">8</td>
<td style="text-align: right;">0.4699095</td>
<td style="text-align: right;">102</td>
<td style="text-align: left;">T</td>
</tr>
<tr class="odd">
<td style="text-align: left;">9</td>
<td style="text-align: right;">9</td>
<td style="text-align: right;">0.3853540</td>
<td style="text-align: right;">81</td>
<td style="text-align: left;">T</td>
</tr>
<tr class="even">
<td style="text-align: left;">10</td>
<td style="text-align: right;">10</td>
<td style="text-align: right;">0.6336118</td>
<td style="text-align: right;">148</td>
<td style="text-align: left;">C</td>
</tr>
<tr class="odd">
<td style="text-align: left;">11</td>
<td style="text-align: right;">11</td>
<td style="text-align: right;">0.7365508</td>
<td style="text-align: right;">178</td>
<td style="text-align: left;">C</td>
</tr>
<tr class="even">
<td style="text-align: left;">12</td>
<td style="text-align: right;">12</td>
<td style="text-align: right;">0.4967514</td>
<td style="text-align: right;">106</td>
<td style="text-align: left;">T</td>
</tr>
<tr class="odd">
<td style="text-align: left;">229</td>
<td style="text-align: right;">229</td>
<td style="text-align: right;">0.6637343</td>
<td style="text-align: right;">156</td>
<td style="text-align: left;">C</td>
</tr>
<tr class="even">
<td style="text-align: left;">230</td>
<td style="text-align: right;">230</td>
<td style="text-align: right;">0.9801347</td>
<td style="text-align: right;">235</td>
<td style="text-align: left;">C</td>
</tr>
<tr class="odd">
<td style="text-align: left;">231</td>
<td style="text-align: right;">231</td>
<td style="text-align: right;">0.1659937</td>
<td style="text-align: right;">36</td>
<td style="text-align: left;">T</td>
</tr>
<tr class="even">
<td style="text-align: left;">232</td>
<td style="text-align: right;">232</td>
<td style="text-align: right;">0.4225586</td>
<td style="text-align: right;">90</td>
<td style="text-align: left;">T</td>
</tr>
<tr class="odd">
<td style="text-align: left;">233</td>
<td style="text-align: right;">233</td>
<td style="text-align: right;">0.3599470</td>
<td style="text-align: right;">78</td>
<td style="text-align: left;">T</td>
</tr>
<tr class="even">
<td style="text-align: left;">234</td>
<td style="text-align: right;">234</td>
<td style="text-align: right;">0.6065274</td>
<td style="text-align: right;">136</td>
<td style="text-align: left;">C</td>
</tr>
<tr class="odd">
<td style="text-align: left;">235</td>
<td style="text-align: right;">235</td>
<td style="text-align: right;">0.5213893</td>
<td style="text-align: right;">111</td>
<td style="text-align: left;">T</td>
</tr>
<tr class="even">
<td style="text-align: left;">236</td>
<td style="text-align: right;">236</td>
<td style="text-align: right;">0.8585871</td>
<td style="text-align: right;">202</td>
<td style="text-align: left;">C</td>
</tr>
<tr class="odd">
<td style="text-align: left;">237</td>
<td style="text-align: right;">237</td>
<td style="text-align: right;">0.6808066</td>
<td style="text-align: right;">161</td>
<td style="text-align: left;">C</td>
</tr>
<tr class="even">
<td style="text-align: left;">238</td>
<td style="text-align: right;">238</td>
<td style="text-align: right;">0.2789617</td>
<td style="text-align: right;">63</td>
<td style="text-align: left;">T</td>
</tr>
<tr class="odd">
<td style="text-align: left;">239</td>
<td style="text-align: right;">239</td>
<td style="text-align: right;">0.9939375</td>
<td style="text-align: right;">236</td>
<td style="text-align: left;">C</td>
</tr>
<tr class="even">
<td style="text-align: left;">240</td>
<td style="text-align: right;">240</td>
<td style="text-align: right;">0.7866525</td>
<td style="text-align: right;">190</td>
<td style="text-align: left;">C</td>
</tr>
</tbody>
</table>

表1 简单随机分组表

## 2.2 区组随机分组（block randomization）

### 1）方法一

区组随机分组一般有两种办法，第一种是先根据实验分组情况设定区组长度和区组类型组合，再根据区组数量对区组类型进行随机抽样，从而获得每个区组内部的分组情况<sup>\[10\]</sup>，其基本步骤为：1）按照实验设计的受试者样本量生成受试者编号；2）设定区组长度和区组组合；3）根据样本量计算区组数；4）根据区组数从区组组合中随机抽取对应数量的区组类型；5）将随机抽取的区组类型顺序连接，从而获得符合实验设计要求的随机分组表。本文同样提供了一段基于R语言的自编函数，但仅能满足组间比例为1:1的区组随机分组需求，自编函数代码及具体含义如下：

    block_random_m1 <- function(size, block_len, block_category){
        id_num <- seq(1, size, 1)
        block <- size / block_len
        block_set <- c()
        block_num <- c()
        for (i in 1:block) {
            set.seed(20210412+i)
            block <- sample(block_category, 1, replace = FALSE)
            block_set[i] <- block
            block_num <- append(block_num, rep(i, block_len))
            
        }
        group <- as.vector(
                          unlist(
                                strsplit(
                                    paste0(block_set, collapse=";"), split=";")
                                )
                          )
        df <- data.frame("ID" = id_num, "Block" = block_num, "Group" = group)
        
        write.csv(df, "block radomization table 1.csv", row.names = FALSE)
        
        return(df)
    }

1）函数名为block\_random\_m1，函数共有3个参数，分别为size（样本量大小）、block\_len（区组长度）和block\_category（区组组合）；

2）seq()函数和set.seed()作用同上；

3）block变量为根据样本量和区组长度计算的区组数量；

4）两组实验中常用的区组长度为4，那么参数block\_ken即为4，对应的区组组合共有6种情况（“TTCC”,
“TCTC”, “TCCT”, “CCTT”, “CTCT”,
“CTTC”），其中T表示干预组，C表示对照组，用参数block\_category来指定；

5）for循环中即是依据计算所得的区组数量，利用sample()函数从全部区组组合中随机抽取相应数量的区组组合，其中sample()函数的第二个参数用于指定每次抽取的组合数，本文中采取每次抽取1个，第三个参数replace用于指定是否为有放回抽样，当每次抽取数量大于1时应采取无放回抽样，replace的值应设定为FALSE；

6）block\_set变量表示将for循环随机抽取的区组组合以向量的形式存储，以便于后续输出结构化的随机分组表；

7）block\_num变量用于记录区组的编号；

8）as.vector()函数是将block\_set变量中存储的区组组合连接成与样本量相同的长度，以便与受试者编号对应；

9）data.frame()和write.csv()函数用于生成随机分组表并输出为csv格式，随机分组表中包含受试者编号、区组编号、分组。

实例二：为检验某一治疗肿瘤新化疗方案的临床疗效，选择常规化疗方案作为对照组，开展单中心双臂临床试验，需要受试者120名，每组各60名受试者，受试者入组期为3个月，化疗期为6个月，请按区组随机分组方法制定随机分组表。利用本文提供的自编函数block\_random\_m1()则可将参数size设定为120，
参数block\_len设定为4，参数block\_category设定为c(“T;T;C;C”, “T;C;T;C”,
“T;C;C;T”, “C;C;T;T”, “C;T;C;T”, “C;T;T;C”)，具体如下：

    res_block1 <- block_random_m1(size = 120, block_len = 4, 
                                  block_category = c("T;T;C;C", "T;C;T;C", 
                                                     "T;C;C;T", "C;C;T;T", 
                                                     "C;T;C;T", "C;T;T;C"))

函数输出的随机分组表见表2，其中共产生30个区组，每个区组内干预组和对照组各2名受试者。在实际的研究中区组长度的设定可适当变化，具体可见本文讨论部分。

<table>
<caption>表2 方法一区组随机分组表</caption>
<thead>
<tr class="header">
<th style="text-align: left;"></th>
<th style="text-align: right;">ID</th>
<th style="text-align: right;">Block</th>
<th style="text-align: left;">Group</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">1</td>
<td style="text-align: right;">1</td>
<td style="text-align: right;">1</td>
<td style="text-align: left;">C</td>
</tr>
<tr class="even">
<td style="text-align: left;">2</td>
<td style="text-align: right;">2</td>
<td style="text-align: right;">1</td>
<td style="text-align: left;">T</td>
</tr>
<tr class="odd">
<td style="text-align: left;">3</td>
<td style="text-align: right;">3</td>
<td style="text-align: right;">1</td>
<td style="text-align: left;">T</td>
</tr>
<tr class="even">
<td style="text-align: left;">4</td>
<td style="text-align: right;">4</td>
<td style="text-align: right;">1</td>
<td style="text-align: left;">C</td>
</tr>
<tr class="odd">
<td style="text-align: left;">5</td>
<td style="text-align: right;">5</td>
<td style="text-align: right;">2</td>
<td style="text-align: left;">T</td>
</tr>
<tr class="even">
<td style="text-align: left;">6</td>
<td style="text-align: right;">6</td>
<td style="text-align: right;">2</td>
<td style="text-align: left;">T</td>
</tr>
<tr class="odd">
<td style="text-align: left;">7</td>
<td style="text-align: right;">7</td>
<td style="text-align: right;">2</td>
<td style="text-align: left;">C</td>
</tr>
<tr class="even">
<td style="text-align: left;">8</td>
<td style="text-align: right;">8</td>
<td style="text-align: right;">2</td>
<td style="text-align: left;">C</td>
</tr>
<tr class="odd">
<td style="text-align: left;">9</td>
<td style="text-align: right;">9</td>
<td style="text-align: right;">3</td>
<td style="text-align: left;">T</td>
</tr>
<tr class="even">
<td style="text-align: left;">10</td>
<td style="text-align: right;">10</td>
<td style="text-align: right;">3</td>
<td style="text-align: left;">C</td>
</tr>
<tr class="odd">
<td style="text-align: left;">11</td>
<td style="text-align: right;">11</td>
<td style="text-align: right;">3</td>
<td style="text-align: left;">C</td>
</tr>
<tr class="even">
<td style="text-align: left;">12</td>
<td style="text-align: right;">12</td>
<td style="text-align: right;">3</td>
<td style="text-align: left;">T</td>
</tr>
<tr class="odd">
<td style="text-align: left;">109</td>
<td style="text-align: right;">109</td>
<td style="text-align: right;">28</td>
<td style="text-align: left;">C</td>
</tr>
<tr class="even">
<td style="text-align: left;">110</td>
<td style="text-align: right;">110</td>
<td style="text-align: right;">28</td>
<td style="text-align: left;">T</td>
</tr>
<tr class="odd">
<td style="text-align: left;">111</td>
<td style="text-align: right;">111</td>
<td style="text-align: right;">28</td>
<td style="text-align: left;">T</td>
</tr>
<tr class="even">
<td style="text-align: left;">112</td>
<td style="text-align: right;">112</td>
<td style="text-align: right;">28</td>
<td style="text-align: left;">C</td>
</tr>
<tr class="odd">
<td style="text-align: left;">113</td>
<td style="text-align: right;">113</td>
<td style="text-align: right;">29</td>
<td style="text-align: left;">T</td>
</tr>
<tr class="even">
<td style="text-align: left;">114</td>
<td style="text-align: right;">114</td>
<td style="text-align: right;">29</td>
<td style="text-align: left;">T</td>
</tr>
<tr class="odd">
<td style="text-align: left;">115</td>
<td style="text-align: right;">115</td>
<td style="text-align: right;">29</td>
<td style="text-align: left;">C</td>
</tr>
<tr class="even">
<td style="text-align: left;">116</td>
<td style="text-align: right;">116</td>
<td style="text-align: right;">29</td>
<td style="text-align: left;">C</td>
</tr>
<tr class="odd">
<td style="text-align: left;">117</td>
<td style="text-align: right;">117</td>
<td style="text-align: right;">30</td>
<td style="text-align: left;">T</td>
</tr>
<tr class="even">
<td style="text-align: left;">118</td>
<td style="text-align: right;">118</td>
<td style="text-align: right;">30</td>
<td style="text-align: left;">C</td>
</tr>
<tr class="odd">
<td style="text-align: left;">119</td>
<td style="text-align: right;">119</td>
<td style="text-align: right;">30</td>
<td style="text-align: left;">C</td>
</tr>
<tr class="even">
<td style="text-align: left;">120</td>
<td style="text-align: right;">120</td>
<td style="text-align: right;">30</td>
<td style="text-align: left;">T</td>
</tr>
</tbody>
</table>

表2 方法一区组随机分组表

### 2）方法二

第二种是先根据受试者数量生成随机数字，然后根据实验分组情况设定区组长度并依次对区组编号，再按区组编号分组对生成的随机数字编秩，最后根据各区组内随机数字秩的大小关系进行分组，其基本步骤为：1）按照实验设计的受试者样本量生成受试者编号；2）设定区组长度并根据受试者数量计算区组数并依次编号；3）利用随机函数生成与受试者编号数量相等的随机数字；4）按区组编号分组计算随机数字的秩；5）在各区组内，按照实验设计的组数及分组比例，对随机数字的秩序按照大小或者奇偶数进行分组；6）获得符合实验设计要求的随机分组表。本文同样提供了一段基于R语言的自编函数，能满足两组及多组且组间比例不同的区组随机分组需求，自编函数代码及具体含义如下：

    library(dplyr, warn = FALSE)
    block_random_m2 <- function(size, block_len, grp = 2, T_2_C = "1:1"){
        id_num <- seq(1, size, 1)
        block <- size / block_len
        
        block_num <- c()
        for (i in 1:block) {
            block_num <- append(block_num, rep(i, block_len))
        }
        
        random_seq <- runif(n = size, min = 0, max = 1)
        
        df <- data.frame("ID" = id_num, "BlockNum" = block_num, "RandomNum" = random_seq)
        df_rank = df %>% group_by(BlockNum) %>% mutate(Rank = rank(RandomNum))
        
        ratio_T <- as.numeric(substr(T_2_C, 1, 1))
        ratio_C <- as.numeric(substr(T_2_C, 3, 3))
        
        if (grp == 2){
            df_rank$Group <- ifelse(df_rank$Rank <= block_len/(ratio_T + ratio_C), "T", "C")
        } else if (grp >2) {
            df_rank$Group <- cut(df_rank$Rank, breaks = grp, labels = paste("Group", 1:grp))
        }
        
        write.csv(df_rank, "block radomization table 2.csv", row.names = FALSE)   
        return(df_rank)
    }

1）函数名为block\_random\_m2，函数共有4个参数，分别为size（样本量大小）、block\_len（区组长度）、grp（分组数）和T\_2\_C（干预组与对照组比例），grp和T\_2\_C为默认参数，分别默认赋值为2和“1:1”；

2）seq()和runif()函数作用同上；

3）由于此方法需要进行分组编秩，考虑到R语言自带分组功能使用较为不便，本文在自编函数中引入了第三方程序包dplyr，可通过install.packages(“dplyr”)安装，其中group\_by()、mutate()及%&gt;%管道函数为dplyr包中引入的函数;

4）变量ratio\_T和ratio\_C，以及if条件语句中的功能与简单随机分组相同。

5）data.frame()和write.csv()函数用于生成随机分组表并输出为csv格式，随机分组表中包含受试者编号、区组编号、随机数字、分组。

同样以实例二为例，利用block\_random\_m2()函数可以获得同表2相似的结果，具体使用方式如下，输出结果见表3

    res_block2 <- block_random_m2(size = 120, block_len = 4, grp = 2, T_2_C = "1:1")

<table>
<caption>表3 方法二区组随机分组表</caption>
<thead>
<tr class="header">
<th style="text-align: right;">ID</th>
<th style="text-align: right;">BlockNum</th>
<th style="text-align: right;">RandomNum</th>
<th style="text-align: right;">Rank</th>
<th style="text-align: left;">Group</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: right;">1</td>
<td style="text-align: right;">1</td>
<td style="text-align: right;">0.2355342</td>
<td style="text-align: right;">3</td>
<td style="text-align: left;">C</td>
</tr>
<tr class="even">
<td style="text-align: right;">2</td>
<td style="text-align: right;">1</td>
<td style="text-align: right;">0.1950892</td>
<td style="text-align: right;">1</td>
<td style="text-align: left;">T</td>
</tr>
<tr class="odd">
<td style="text-align: right;">3</td>
<td style="text-align: right;">1</td>
<td style="text-align: right;">0.7458419</td>
<td style="text-align: right;">4</td>
<td style="text-align: left;">C</td>
</tr>
<tr class="even">
<td style="text-align: right;">4</td>
<td style="text-align: right;">1</td>
<td style="text-align: right;">0.2197261</td>
<td style="text-align: right;">2</td>
<td style="text-align: left;">T</td>
</tr>
<tr class="odd">
<td style="text-align: right;">5</td>
<td style="text-align: right;">2</td>
<td style="text-align: right;">0.0764369</td>
<td style="text-align: right;">1</td>
<td style="text-align: left;">T</td>
</tr>
<tr class="even">
<td style="text-align: right;">6</td>
<td style="text-align: right;">2</td>
<td style="text-align: right;">0.6039926</td>
<td style="text-align: right;">4</td>
<td style="text-align: left;">C</td>
</tr>
<tr class="odd">
<td style="text-align: right;">7</td>
<td style="text-align: right;">2</td>
<td style="text-align: right;">0.4317742</td>
<td style="text-align: right;">2</td>
<td style="text-align: left;">T</td>
</tr>
<tr class="even">
<td style="text-align: right;">8</td>
<td style="text-align: right;">2</td>
<td style="text-align: right;">0.4814425</td>
<td style="text-align: right;">3</td>
<td style="text-align: left;">C</td>
</tr>
<tr class="odd">
<td style="text-align: right;">9</td>
<td style="text-align: right;">3</td>
<td style="text-align: right;">0.7359029</td>
<td style="text-align: right;">2</td>
<td style="text-align: left;">T</td>
</tr>
<tr class="even">
<td style="text-align: right;">10</td>
<td style="text-align: right;">3</td>
<td style="text-align: right;">0.8289157</td>
<td style="text-align: right;">3</td>
<td style="text-align: left;">C</td>
</tr>
<tr class="odd">
<td style="text-align: right;">11</td>
<td style="text-align: right;">3</td>
<td style="text-align: right;">0.9952405</td>
<td style="text-align: right;">4</td>
<td style="text-align: left;">C</td>
</tr>
<tr class="even">
<td style="text-align: right;">12</td>
<td style="text-align: right;">3</td>
<td style="text-align: right;">0.5653650</td>
<td style="text-align: right;">1</td>
<td style="text-align: left;">T</td>
</tr>
<tr class="odd">
<td style="text-align: right;">109</td>
<td style="text-align: right;">28</td>
<td style="text-align: right;">0.7804675</td>
<td style="text-align: right;">4</td>
<td style="text-align: left;">C</td>
</tr>
<tr class="even">
<td style="text-align: right;">110</td>
<td style="text-align: right;">28</td>
<td style="text-align: right;">0.6807008</td>
<td style="text-align: right;">3</td>
<td style="text-align: left;">C</td>
</tr>
<tr class="odd">
<td style="text-align: right;">111</td>
<td style="text-align: right;">28</td>
<td style="text-align: right;">0.3534151</td>
<td style="text-align: right;">2</td>
<td style="text-align: left;">T</td>
</tr>
<tr class="even">
<td style="text-align: right;">112</td>
<td style="text-align: right;">28</td>
<td style="text-align: right;">0.3243371</td>
<td style="text-align: right;">1</td>
<td style="text-align: left;">T</td>
</tr>
<tr class="odd">
<td style="text-align: right;">113</td>
<td style="text-align: right;">29</td>
<td style="text-align: right;">0.3971316</td>
<td style="text-align: right;">2</td>
<td style="text-align: left;">T</td>
</tr>
<tr class="even">
<td style="text-align: right;">114</td>
<td style="text-align: right;">29</td>
<td style="text-align: right;">0.5407747</td>
<td style="text-align: right;">4</td>
<td style="text-align: left;">C</td>
</tr>
<tr class="odd">
<td style="text-align: right;">115</td>
<td style="text-align: right;">29</td>
<td style="text-align: right;">0.4979685</td>
<td style="text-align: right;">3</td>
<td style="text-align: left;">C</td>
</tr>
<tr class="even">
<td style="text-align: right;">116</td>
<td style="text-align: right;">29</td>
<td style="text-align: right;">0.1560650</td>
<td style="text-align: right;">1</td>
<td style="text-align: left;">T</td>
</tr>
<tr class="odd">
<td style="text-align: right;">117</td>
<td style="text-align: right;">30</td>
<td style="text-align: right;">0.6870941</td>
<td style="text-align: right;">1</td>
<td style="text-align: left;">T</td>
</tr>
<tr class="even">
<td style="text-align: right;">118</td>
<td style="text-align: right;">30</td>
<td style="text-align: right;">0.7521457</td>
<td style="text-align: right;">2</td>
<td style="text-align: left;">T</td>
</tr>
<tr class="odd">
<td style="text-align: right;">119</td>
<td style="text-align: right;">30</td>
<td style="text-align: right;">0.8746505</td>
<td style="text-align: right;">4</td>
<td style="text-align: left;">C</td>
</tr>
<tr class="even">
<td style="text-align: right;">120</td>
<td style="text-align: right;">30</td>
<td style="text-align: right;">0.7950460</td>
<td style="text-align: right;">3</td>
<td style="text-align: left;">C</td>
</tr>
</tbody>
</table>

表3 方法二区组随机分组表

## 2.3 分层随机分组（stratified randomization）

从操作层面而言，分层随机分组的本质是根据混杂因素对受试者进行分层后，再在各个层内采用简单随机或者区组随机的方法将受试者分配至不同的实验组。如在进行多中心肿瘤疾病的临床试验中，研究中心的情况和受试者肿瘤的分型分期会对治疗结局产生明显的影响，但是若采取简单随机或者区组随机分组的方法，很难确保不同研究中心入组的受试者特征相似，以及不同实验组间受试者肿瘤分型分期均衡，因此需要对此类明显可能造成实验结局偏倚的变量进行分层控制。假设存在3个研究中心，肿瘤共分为2型2期，那么就可以设置12个层，再在12个层内分别采用简单随机或者区组随机分组方法生成随机分组表，由于所采用方法同上，此处不再详细叙述。

# 3. 讨论

本文详细梳理了当前阶段实验研究中常用的随机分组方法，并以此为基础，结合现阶段研究领域常用的开源统计软件R语言，详细介绍了实验研究中进行随机分组的具体过程，并通过自编函数的形式提出了集成化解决方案，且通过实例验证了自编函数的可靠性，可为研究者在开展实验研究的随机分组时提供借鉴和参考。有几点需要讨论说明，如下：

随机种子数的设定及管理。在RCT研究中，随机化通常和盲法联合使用，而随机分组的结果和规则是盲法的重要内容之一，因此除了需要保证不将随机分组结果透露至研究人员或者受试者外，还需要保证随机分组规则不可轻易被猜出。在利用计算机平台进行随机数生成时，合理设定随机种子数是保证生成的随机数可再现的重要前提。那么随机种子数的设定就十分关键，一般情况下随机种子数不可设置过于简单，如1或者2，亦或者年份。随机种子数应由项目组的统计人员保管，在实验揭盲或者结束之前不可透露给其他研究相关人员。

区组随机中区组长度的设定。区组长度由研究者确定，对于实验组为两组时通常取4和6<sup>\[10\]</sup>。考虑到区组长度较短且单一时，研究人员容易总结发现其中规律，可能会破坏盲法，因此在设定区组长度时可以采用复合方案，即在同一份随机分组表中同时设置4和6的区组长度，从而增加区组的复杂性，减少可预见性。另一方面，由于在具体的实验研究中，受试者脱落或者中途退出的情况并不少见，因此在生成随机分组表时，一般情况下推荐多设置1至2个区组，以备进行补充。

除本文中详细介绍的三种常用随机分组方法外，为了应对受试者入组的复杂情况，还有协变量自适应随机分组（covariate
adaptive randomization）<sup>\[10\]</sup>和响应自适应随机分组（
response-adaptive
randomization）<sup>\[12\]</sup>方法，由于以上方法已有学者开发出了供R语言直接使用的软件包（carat），因此本文中未详细介绍，具体可见
[&lt;https://CRAN.R-project.org/package=carat&gt;](https://cran.r-project.org/package=carat)。

# 参考文献

<span class="csl-left-margin">\[1\] </span><span
class="csl-right-inline">李立明. 流行病学\[M\]. 北京: 人民卫生出版社,
2007.</span>

<span class="csl-left-margin">\[2\] </span><span
class="csl-right-inline">KENDALL J M. Designing a Research Project:
Randomised Controlled Trials and Their Principles\[J\]. Emergency
Medicine Journal, 2003, 20(2): 164–168.
DOI:[10.1136/emj.20.2.164](https://doi.org/10.1136/emj.20.2.164).</span>

<span class="csl-left-margin">\[3\] </span><span
class="csl-right-inline">ROBERTS C, TORGERSON D. Understanding
Controlled Trials: Randomisation Methods in Controlled Trials\[J\]. BMJ,
1998, 317(7168): 1301–1310.
DOI:[10.1136/bmj.317.7168.1301](https://doi.org/10.1136/bmj.317.7168.1301).</span>

<span class="csl-left-margin">\[4\] </span><span
class="csl-right-inline">COX D R. Randomization in the Design of
Experiments\[J\]. International Statistical Review, 2009, 77(3):
415–429.</span>

<span class="csl-left-margin">\[5\] </span><span
class="csl-right-inline">胡良平, 关雪, 毛玮, 等.
各种常见随机化的SAS实现\[J\]. 中华脑血管病杂志(电子版), 2011, 5(01):
68–76.</span>

<span class="csl-left-margin">\[6\] </span><span
class="csl-right-inline">刘玉秀, 姚晨, 杨友春, 等.
随机化临床试验及随机化的SAS实现\[J\]. 中国临床药理学与治疗学, 2001,
6(3): 193–195.
DOI:[10.3969/j.issn.1009-2501.2001.03.001](https://doi.org/10.3969/j.issn.1009-2501.2001.03.001).</span>

<span class="csl-left-margin">\[7\] </span><span
class="csl-right-inline">吴春霖, 王镭, 李卫兵.
临床试验随机化分组及其Stata的实现\[J\]. 中国循证医学杂志, 2013, 13(2):
242–244.
DOI:[10.7507/1672-2531.20130041](https://doi.org/10.7507/1672-2531.20130041).</span>

<span class="csl-left-margin">\[8\] </span><span
class="csl-right-inline">AYLMER FISHER R. The Arrangement of Field
Experiments\[J\]. Journal of the Ministry of Agriculture of Great
Britain, 1926, 33: 503–513.
DOI:[10.23637/ROTHAMSTED.8V61Q](https://doi.org/10.23637/ROTHAMSTED.8V61Q).</span>

<span class="csl-left-margin">\[9\] </span><span
class="csl-right-inline">FLEISS J L, LEVIN B A, PAIK M C. Chapter 5: How
to Randomize in Statistical Methods for Rates and Proportions.\[M\].
Hoboken, N.J.: Wiley-Interscience, 2003.</span>

<span class="csl-left-margin">\[10\] </span><span
class="csl-right-inline">KANG M, RAGAN B G, PARK J-H. Issues in Outcomes
Research: An Overview of Randomization Techniques for Clinical
Trials\[J\]. Journal of Athletic Training, 2008, 43(2): 215–221.
DOI:[10.4085/1062-6050-43.2.215](https://doi.org/10.4085/1062-6050-43.2.215).</span>

<span class="csl-left-margin">\[11\] </span><span
class="csl-right-inline">KERNAN W N, VISCOLI C M, MAKUCH R W, 等.
Stratified Randomization for Clinical Trials\[J\]. Journal of Clinical
Epidemiology, 1999, 52(1): 19–26.
DOI:[10.1016/S0895-4356(98)00138-3](https://doi.org/10.1016/S0895-4356(98)00138-3).</span>

<span class="csl-left-margin">\[12\] </span><span
class="csl-right-inline">刘晓燕, 陈峰, 魏永越, 等.
响应—自适应随机化分组方法\[J\]. 临床药理学, 2008, 13(6): 684–689.</span>

[1] 其他随机分组方法还有：协变量自适应随机分组（covariate adaptive
randomization）和响应自适应随机分组（ response-adaptive randomization）

[2] 利用计算机生成的随机数本质为伪随机数（Pseudorandomness），它是通过特定的算法计算所得，其满足随机数的统计特征，在相同的计算平台下可以重复出现。

