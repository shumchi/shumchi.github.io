---
layout: post
title:  "[#9] - 一别三月，春暖花开"
date:   2020-06-17 20:00:00
excerpt: "纽村已经开始有了夏天的气息，可是依旧回程路漫漫"
author: Chi Shen
tags: [python, plotly]
feature: https://i1.wp.com/box5497.temp.domains/~betwefc1/wp-content/uploads/2017/06/Five_Mile_Point_Light_-_New_Haven_CT.jpg
comments: true
---

**Shen, Chi**

**June 17 2020, New Haven, CT**

> 因为新冠肺炎在美帝肆掠，自三月初到现在，已经居家隔离了三个月了，记得当时是在春假前后，纽村的天还有些冷，不时还飘点小雪，街边的树还未长出新叶。
>
> 转眼一望已是六月，纽村已经开始有了夏天的气息，后院和街边已经郁郁葱葱了，哪天出门再来补几张图。数一下时日，来纽村已经10个月了，但是对纽村的感受却是一下子从三月到了六月，偶尔有些恍惚，越是离回程的日子近了，就越是不自觉的回想这十个月的生活，百年不遇的状况基本打乱了所有的计划，还没有去看大西洋，还没有考下驾照游美国。。。
>
> 不过以后再回想，这也许没准儿是段不错的人生经历吧。回程的日子是近了，可是回程的路依据漫漫，只能祈祷世界和平，peace and love吧。
>
> 最近在不想写论文的间歇琢磨了下`Dash`、`Plotly`和`Streamlit`，一琢磨就耽搁了两三天进度，不过也许后面能用上，写不出论文的时候就储备些技术吧。
>
> 

![想去还没去的纽黑文的大西洋边的灯塔](https://cdn.thecrazytourist.com/wp-content/uploads/2019/11/ccimage-shutterstock_1139288306.jpg)

![这辈子应该只有一次机会去看的大西洋吧](https://cdn.pixabay.com/photo/2015/09/22/00/27/newhaven-950967_1280.jpg)

# 前言

今天分享一个做交互式图形（interactive plot）的方法，至于交互式图形对科研有什么帮助，我也还没想太明白，反正流行“一图胜千言”，试试再说。我记得有看过一些Journal的Scope里写到愿意接收方法创新、数据创新、结果展现形式创新的研究。对科研没有帮助的话，对工作有点帮助也还行，实在不行对生活有点帮助也行。



# 工具

做交互式图形的工具还是很多的，有一些网页也支持上传数据就能画出很漂亮的图。R语言和Python里相关的包和库也很多，比较好用的就是Plotly和Bokeh，下面用的就是Plotly。Plotly是一个十分强大的商业图形平台，原本是要付费的，但是给R和Python开发了免费使用的库。



关于Python下的Plotly如何安装，按照官方网站介绍就可以了，十分简单【https://plotly.com/python/】



# 实例

因为Black lives matter的缘故，美帝人民似乎不那么关心病毒了，好像就这么过去了似的，其实部分州都开始有出现第二波的迹象了，所以就打算来看看美帝的发病情况。



数据取自约翰霍普金斯那个很出名的dashboard平台的数据，就是这里【https://github.com/CSSEGISandData】，下面就直接读取：

**分为两部分：**

一部分是全球的，一部分是美国各地的

```python
import pandas as pdimport numpy as np
def data_wrangle(url):    
  df = pd.read_csv(url)    
  df_t = pd.melt(df,                     
                 id_vars=["Province/State", "Country/Region", "Lat", "Long"],                     
                 value_vars=df.columns[4: df.shape[1]],                     
                 value_name="case_num",                     
                 var_name="date")    
  df_t = df_t.rename(columns={"Lat": "lat", "Long": "lon", "Province/State": "Province", "Country/Region": "Country"})    
  return df_t

df = data_wrangle("https://raw.githubusercontent.com/CSSEGISandData/COVID-19/master/csse_covid_19_data/csse_covid_19_time_series/time_series_covid19_confirmed_global.csv")
# 因为有美国分地区的数据，所以就剔除其中全美的数据
df0 = df[df["Country"] != "US"].reset_index(drop=True)
```

但是数据是wide的形式，也就是把日期放在了列上，不便于分析，因此转成long数据，其实上面的function里面已经完成了这一步，但是美国的数据变量和全球数据库中不太一致，所以单独再来一次

```python
df_us = pd.read_csv("https://raw.githubusercontent.com/CSSEGISandData/COVID-19/master/csse_covid_19_data/csse_covid_19_time_series/time_series_covid19_confirmed_US.csv")

df_us_long = pd.melt(df_us, id_vars=["Combined_Key", "Lat", "Long_"],                     
                     value_vars=df_us.columns[11: df_us.shape[1]],                     
                     value_name="case_num",                     
                     var_name="date")
```

设定唯一的id，并且统一变量名

```python
df0["id"] =  np.where(df0["Province"].isna(), df0["Country"], df0["Province"] + " " + df0["Country"])

df_us_long = df_us_long.rename(columns={"Lat": "lat", "Long_": "lon", "Combined_Key": "id"})
```

因为这个数据库中提供的是累计发病人数，但是我们想看的是每天新发病例数，所以就得自己动手算一个，其实也很简单，就是用后一天的患者总数减去前一天的总数，关键函数就是shift，其实就是lag。**需要注意的一点是，要先排序，然后再按地区分组计算**

```python
from datetime import datetime
def preparing(data):    
  # 之前日期排序有点不对，改进一下    
  data["date_t"] = data["date"].apply(lambda x: datetime.strptime(x, "%m/%d/%y").strftime("%Y-%m-%d"))    
  data1 = data.sort_values(by = ["id", "date_t"])    
  data1["case_new"] = data1.groupby("id")["case_num"].apply(lambda x: abs(x - x.shift(1, axis=0)))    
  return data1[["id", "case_new", "lat", "lon", "date"]]

df1 = preparing(df0)
df_us_long1 = preparing(df_us_long)
```

然后就是将全球的数据和美国的数据纵向append在一起，便于画图

```python
df_all = pd.concat([df1, df_us_long1], axis=0)
```

最后就是画图了，直接调用plotly库中的express模块，这是一个封装好了的高级模块，语法不那么底层，和一般的可视化库的使用方式一样。其中让图动起来的关键就是animation_frame参数，也就是指定按照那个变量设定帧数，这也是为什么要转换成long数据的原因，其他的参数就是字面意思，很容易理解

```python
import plotly.express as px
# 因为lag减了一次后会出现NA，需要剔除
figure = px.scatter_geo(df_all[df_all["case_new"].notna()], 
                        lat="lat", lon="lon", 
                        size="case_new", color="case_new",                         
                        title="Daily New Case of Covid-19 in the World",                         
                        animation_frame="date_t",                         
                        template="plotly_dark",                         
                        text="id",                        
                        size_max=20,                        
                        opacity=1,                         
                        labels={"date_t": "Date", "case_new": "New Case"})
# 保存成本地的html文件
plotly.offline.plot(figure)
```

最后就是成果了。



<video id="video" controls="" preload="none">
    <source id="mp4" src="https://github.com/shumchi/shumchi.github.io/blob/master/_posts/2020-06-19-M9%20Interactive%20plot/mov.png?raw=true" type="video/mp4">
</video>

**除了mapping的作图函数外，其他常用的scatter、bar、line、violin、area、pie都可以很方便做出交互式的，详见[官方API](https://plotly.com/python-api-reference/)**



