---
layout: post
title:  "[#3] - 下雪了，不止一场"
date:   2019-12-15 15:10:00
excerpt: "有没有哪个哲人说过，开始下雪的时候就该回家了"
author: Chi Shen
tags: [Home, December, Snow]
feature: https:https://news.yale.edu/sites/default/files/d6_files/P4.jpg
comments: true
---

**Chi Shen, Dec 15 2019**

**New Haven, CT**

> 距离圣诞节还有10天，距离新年还有15天，距离春节还有40天，离家的人们都开始计划着回家的行程，纽村也适时的下了三两场雪，不知道有没有哪个哲人说过，开始下雪的时候就该回家了，现在的你离家还有多少个hundred mile呢！



## 出门是山

出门确实是山，称为东山（East Rock），但是在北边，山是在门后。来了三个月，却也一次都没爬过。感恩节假期的某一天，风和日丽，做完午饭，本计划着爬上山顶，俯瞰一下整个纽黑文。临出门前，Google map一下路线，却发现有个街景功能，于是就顺着路线查看了街景，发现与预想的风景不太一致，瞬间失去了兴趣，爬山的计划自然也就作罢了，但是，也算**“Cloud Visiting”**了一把山上的风景了。

来了纽黑文之后，发现年龄越大反而越“宅”了。之前要不是陶大夫有事没事要出去看看，估计到现在西安城都还没转明白。到了这没有车就寸步难行的美帝，自然是更加不愿出门了，若不是小兰博士还没有回国的时候，带着出去逛了逛，那真的是会三个月没出过城。再加之前几个月要缴访问费，经费有些困难，实力不允许出去瞎浪。所以，**归纳起来就是地理可及性和经济可及性着实会影响行为**。



## 除了自习，就是做饭

为了保重身体（主要是为了头发），适当的放慢了些节奏，缩短了些工作时长。除了学习，大部分的时间都是做饭了。三个月的时间，得有两个半月的时间都是在煮面吃了，对于一个南方人来说实属不易了。

这一个月，因为临近年末，参加的seminar不多，受到的启发有限，当然主要是自己懒了些。但是为了博士论文储备知识点，花了一个星期粗略的啃了`沈体雁`主编的**《空间计量经济学》**和`王劲峰`的**《空间数据分析教程》**，有种豁然开朗的感觉。

之前只有零星的几点想法，不确定是不是主流的研究方向，一直担心万一选了偏冷门的方向就风险很大了。但是，看完了空间计量，似乎有点头绪了，选题方法可以和**区域科学**和**新经济地理**结合起来，并且对**“模型--方法--数据”**的研究路线有了更深刻的体会。



## 累了码个砖

休息时突然想到，就做出来看了看，本来应该是动图。

    std_name <- LETTERS[1:20]
    lucky <- sample(std_name, size = 9, replace = F)
    
    plot(0:10,0:10, type="n")
    for(i in 1:9){  
        text(x = i, y = i, lucky[i])
        title(main = 'who is the lucky guy:')
        Sys.sleep(1) 
    }

![letter](https://github.com/shumchi/shumchi.github.io/blob/master/_posts/2019-12-15-M3%20First%20Snow/letter-1.png?raw=true)

    z=seq(0, 2*pi,length=60)
    plot(x = sin(z), y = cos(z), type="l", main = 'Next decade is on the way', xlab = ' ', ylab = ' ')
    arrows(x0 = 0, y0 = 0, x1 = sin(z)[59], y1 = cos(z)[59] - 0.1, col = 'blue', lwd = 2.5)
    for(i in 2:61){  
        text(x = 0, y = 0, '*', cex = 2, font = 2)
        arrows(x0 = 0, y0 = 0, x1 = sin(z)[i], y1 = cos(z)[i], length = 0.1)
        if (i >= 3 & i <= 60) {
            arrows(x0 = 0, y0 = 0, x1 = sin(z)[i-1], y1 = cos(z)[i-1], length = 0.1, col = 'white')
        }   
    
        Sys.sleep(1) 
    
        if (i == 60) {
            arrows(x0 = 0, y0 = 0, x1 = sin(z)[59], y1 = cos(z)[59] - 0.1, col = 'white', lwd = 2.5)
            arrows(x0 = 0, y0 = 0, x1 = sin(z)[i], y1 = cos(z)[i] - 0.1, col = 'blue', lwd = 2.5)
        }
    
        if (i == 61) {
        text(x = 0, y = 0, 'Happy New Year!', col = 'red', cex = 2, font = 15)
        }
    }

![clock](https://github.com/shumchi/shumchi.github.io/blob/master/_posts/2019-12-15-M3%20First%20Snow/clock-1.png?raw=true)

## 半晚的雪景有点暗
![snow-1](https://github.com/shumchi/shumchi.github.io/blob/master/_posts/2019-12-15-M3%20First%20Snow/s-1.jpeg?raw=true)

![snow-2](https://github.com/shumchi/shumchi.github.io/blob/master/_posts/2019-12-15-M3%20First%20Snow/s-2.jpeg?raw=true)

![snow-3](https://github.com/shumchi/shumchi.github.io/blob/master/_posts/2019-12-15-M3%20First%20Snow/s-3.jpeg?raw=true)
