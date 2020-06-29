---
layout: post
title:  "当pandas传入的多个参数均为column时"
date:   2020-06-29
excerpt: "apply函数的扩展用法"
author: Chi Shen
tags: [python, plotly]
feature: https://i1.wp.com/box5497.temp.domains/~betwefc1/wp-content/uploads/2017/06/Five_Mile_Point_Light_-_New_Haven_CT.jpg
comments: true
---

**Shen, Chi**

**June 29 2020, New Haven, CT**  

Python的pandas库中有三个比较好用的函数`apply`、`aggregate`和`transform`，可以很方便进行column和row的遍历运算，一定程度上可以替代for循环，并且运算速度更快。

但是在使用`apply`时总有一个问题困扰着一直没有解决：当需要为func传入多个参数，且这些参数都是dataframe的column时，是否可以实现？因为func的第一个参数默认是指定的dataframe的column，那么第二个参数如何处理？

> ps：尽管上述三个函数均可以通过axis=0或1来指定是向row的方向运算，还是向column的方向运算，但是在data analysis时通常向row的方向运算比较常用（就是对指定column进行函数操作），并且默认取值也为0.


先来看一个比较简单的传入两个参数的例子：

###  示例数据

```python
import pandas as pd
from tabulate import tabulate 
mtcars = pd.read_csv("https://gist.githubusercontent.com/seankross/a412dfbd88b3db70b74b/raw/5f23f993cd87c283ce766e7ac6b329ee7cc2e1d1/mtcars.csv")
df = mtcars.loc[:, ["model", "mpg", "cyl"]]
```

```python
print(tabulate(df.head(), headers=df.columns, tablefmt="github"))
```

|    | model             |   mpg |   cyl |
|----|-------------------|-------|-------|
|  0 | Mazda RX4         |  21   |     6 |
|  1 | Mazda RX4 Wag     |  21   |     6 |
|  2 | Datsun 710        |  22.8 |     4 |
|  3 | Hornet 4 Drive    |  21.4 |     6 |
|  4 | Hornet Sportabout |  18.7 |     8 |

​    

###  例1


这个例子比较简单，也就是传入的第二个参数为一个标量

```python
def fun(x, y):
    return x * y
df["mpg_new"] = df["mpg"].apply(fun, y=2)
print(tabulate(df.head(), headers=df.columns, tablefmt="github"))
```


|    | model             |   mpg |   cyl |   mpg_new |
|----|-------------------|-------|-------|-----------|
|  0 | Mazda RX4         |  21   |     6 |      42   |
|  1 | Mazda RX4 Wag     |  21   |     6 |      42   |
|  2 | Datsun 710        |  22.8 |     4 |      45.6 |
|  3 | Hornet 4 Drive    |  21.4 |     6 |      42.8 |
|  4 | Hornet Sportabout |  18.7 |     8 |      37.4 |

  

###  例2


但是在处理数据时，通常会有更复杂的需求，比如：

> 如果`cyl`为6，我们乘以3，如果`cyl`为4，我们乘以2，如果`cyl`为其他，我们乘以5

那么这时候需要传入两个列，即`mpg`和`cyl`，如果仍然按照上面的方式传入的话，就会报错.
原因为在fun1中，只有y是标量时才可以进行==判断


```python
def fun1(x, y):
    if y == 6:
        return x * 3
    elif y == 4:
        return x * 4
    else:
        return x * 5
df["mpg_new"] = df["mpg"].apply(fun1, y=df["cyl"])
```

  


> Traceback (most recent call last):
ValueError: The truth value of a Series is ambiguous. Use a.empty, a.bool(), a.item(), a.any() or a.all().

  

~~之前我直接想到的办法如下，通过for循环来遍历每一个row，执行效率比较低下：~~

```python
import numpy as np
def fun1(x, y):
    if y == 6:
        return x * 3
    elif y == 4:
        return x * 4
    else:
        return x * 5
df["mpg_new"] = np.NaN
for i, j in enumerate(df["cyl"]):
    df.loc[i, "mpg_new"] = fun1(df["mpg"][i], y=j)
    
print(tabulate(df.head(), headers=df.columns, tablefmt="github"))
```

  

|    | model             |   mpg |   cyl |   mpg_new |
|----|-------------------|-------|-------|-----------|
|  0 | Mazda RX4         |  21   |     6 |      63   |
|  1 | Mazda RX4 Wag     |  21   |     6 |      63   |
|  2 | Datsun 710        |  22.8 |     4 |      91.2 |
|  3 | Hornet 4 Drive    |  21.4 |     6 |      64.2 |
|  4 | Hornet Sportabout |  18.7 |     8 |      93.5 |

  


###  不过这两天，在stackoverflow上看到一个比较好的解决办法


参考地址为[stackoverflow](https://stackoverflow.com/questions/34279378/python-pandas-apply-function-with-two-arguments-to-columns/34279543 )

####  方法一


该方法是将向row方向进行运算，改为向column方向运算

```python
import numpy as np
def fun1(x, y):
    if y == 6:
        return x * 3
    elif y == 4:
        return x * 4
    else:
        return x * 5
df["mpg_new"] = df[["mpg", "cyl"]].apply(fun1, axis=1)
print(tabulate(df.head(), headers=df.columns, tablefmt="github"))
```


|      | model             | mpg  | cyl  | mpg_new |
| ---- | ----------------- | ---- | ---- | ------- |
| 0    | Mazda RX4         | 21   | 6    | 63      |
| 1    | Mazda RX4 Wag     | 21   | 6    | 63      |
| 2    | Datsun 710        | 22.8 | 4    | 91.2    |
| 3    | Hornet 4 Drive    | 21.4 | 6    | 64.2    |
| 4    | Hornet Sportabout | 18.7 | 8    | 93.5    |

---

向row和column的方向运算的区别，如下：

当axis=1时，运算的方向是横向的，也就是对row进行函数运算

这里的axis的0和1所指示的方向和`R`刚好是相反的

```python
import numpy as np
df["new"] = df[["mpg", "cyl"]].apply(np.sum, axis=1)
print(tabulate(df.head(), headers=df.columns, tablefmt="github"))
```


|    | model             |   mpg |   cyl |   new |
|----|-------------------|-------|-------|-------|
|  0 | Mazda RX4         |  21   |     6 |  27   |
|  1 | Mazda RX4 Wag     |  21   |     6 |  27   |
|  2 | Datsun 710        |  22.8 |     4 |  26.8 |
|  3 | Hornet 4 Drive    |  21.4 |     6 |  27.4 |
|  4 | Hornet Sportabout |  18.7 |     8 |  26.7 |

当axis=0时，运算的方向是纵向的，也就是对column进行函数运算


```python
import numpy as np
print(df[["mpg", "cyl"]].apply(np.sum, axis=0))
```


mpg    642.9
cyl    198.0
dtype: float64

 

---

####  方法二

```python
import numpy as np
def fun1(x, y):
    if y == 6:
        return x * 3
    elif y == 4:
        return x * 4
    else:
        return x * 5
df["mpg_new"] = df.apply(lambda x: fun1(x["mpg"], x["cyl"]), axis=1)
print(tabulate(df.head(), headers=df.columns, tablefmt="github")) 
```

|    | model             |   mpg |   cyl |   mpg_new |
|----|-------------------|-------|-------|-----------|
|  0 | Mazda RX4         |  21   |     6 |      63   |
|  1 | Mazda RX4 Wag     |  21   |     6 |      63   |
|  2 | Datsun 710        |  22.8 |     4 |      91.2 |
|  3 | Hornet 4 Drive    |  21.4 |     6 |      64.2 |
|  4 | Hornet Sportabout |  18.7 |     8 |      93.5 |

  

####  方法三


这个方法也是一个道理，这是把两个参数改成一个参数了
但是和*方法一*比起来有点多此一举的感觉


```python
def fun2(df):
    x = df[0]
    y = df[1]
    if y == 6:
        return x * 3
    elif y == 4:
        return x * 4
    else:
        return x * 5
df["mpg_new"] = df[["mpg", "cyl"]].apply(fun2, axis=1)
print(tabulate(df.head(), headers=df.columns, tablefmt="github"))
```


|    | model             |   mpg |   cyl |   mpg_new |
|----|-------------------|-------|-------|-----------|
|  0 | Mazda RX4         |  21   |     6 |      63   |
|  1 | Mazda RX4 Wag     |  21   |     6 |      63   |
|  2 | Datsun 710        |  22.8 |     4 |      91.2 |
|  3 | Hornet 4 Drive    |  21.4 |     6 |      64.2 |
|  4 | Hornet Sportabout |  18.7 |     8 |      93.5 |

另外一点，如果func返回的是两个值，那么最好返回成series，这样就可以同时在dataframe中生成多列，如下：

```python
def func(x):
    ...
    return pd.Series(m, n)
  
df[["var1", "var2"]] = df.apply(func)
```


####  End

  

