---
layout: post
title:  "双重差分模型进行平行趋势检验(Parallel Test)的方法"
date:   2020-02-23 20:00:00
excerpt: "网络上关于双重差分模型平行趋势检验的方法有很多，但是略有差异，因此总结一份，留作学习。"
author: Chi Shen
tags: [Diff-in-Diff, Parallel test, Coefplot]
feature: https://images2.minutemediacdn.com/image/upload/c_crop,h_1196,w_2121,x_0,y_218/f_auto,q_auto,w_1100/v1554930708/shape/mentalfloss/55573-istock-891978706.jpg
comments: true
---

**Shen, Chi**

**Feb 23 2020, New Haven, CT**

> 网络上关于双重差分模型平行趋势检验的方法有很多，但是略有差异，因此总结一份，留作学习。



##  平行趋势检验的方法

需要说明的是两期数据是无法进行平行趋势检验的，因为时间长度不够，如果对于数据不自信，可以采用匹配与双重差分相结合的方法。以下方法主要适用于多年的面板数据。

### 1）示例数据说明

示例数据来源于普林斯顿大学网站上的数据集[Panel101](http://dss.princeton.edu/training/Panel101.dta)。

**数据详情如下：**

```python
import pandas as pd 
import numpy as np
panel = pd.read_stata("http://dss.princeton.edu/training/Panel101.dta")
panel.head()
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>country</th>
      <th>year</th>
      <th>y</th>
      <th>y_bin</th>
      <th>x1</th>
      <th>x2</th>
      <th>x3</th>
      <th>opinion</th>
      <th>op</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>A</td>
      <td>1990</td>
      <td>1.342788e+09</td>
      <td>1.0</td>
      <td>0.277904</td>
      <td>-1.107956</td>
      <td>0.282554</td>
      <td>Str agree</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>A</td>
      <td>1991</td>
      <td>-1.899661e+09</td>
      <td>0.0</td>
      <td>0.320685</td>
      <td>-0.948720</td>
      <td>0.492538</td>
      <td>Disag</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>A</td>
      <td>1992</td>
      <td>-1.123436e+07</td>
      <td>0.0</td>
      <td>0.363466</td>
      <td>-0.789484</td>
      <td>0.702523</td>
      <td>Disag</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>A</td>
      <td>1993</td>
      <td>2.645775e+09</td>
      <td>1.0</td>
      <td>0.246144</td>
      <td>-0.885533</td>
      <td>-0.094391</td>
      <td>Disag</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>A</td>
      <td>1994</td>
      <td>3.008335e+09</td>
      <td>1.0</td>
      <td>0.424623</td>
      <td>-0.729768</td>
      <td>0.946131</td>
      <td>Disag</td>
      <td>0.0</td>
    </tr>
  </tbody>
</table>



**数据预处理：**

```python
panel["time"] = np.where(panel["year"] >= 1994, 1, 0)
panel["treated"] = np.where(panel["country"].isin(["E", "F", "G"]), 1, 0)
panel["did"] = panel["time"] * panel["treated"]
```



### 2）作图法

即画出被解释变量每一年均值的时序图，通过观察在干预前是否趋势一致来判断是否满足平行假设，当然这是一种比较粗略的方式，通过画图发现干预前不一致，也不一定说明不满足平行假设。

如下：

```python
import seaborn as sns
import matplotlib.pyplot as plt 

fig, ax = plt.subplots(figsize=[10, 6])
sns.lineplot(x="year", y="y", hue="treated", estimator="mean", marker="s", ci=None, data=panel, ax=ax)
ax.set_xticks(np.arange(1990, 2000, 1))
ax.axvline(x=1994, linestyle='--', color='black', linewidth=1)
sns.despine()
```

![](https://github.com/shumchi/shumchi.github.io/blob/master/_posts/2020-02-23-DID-Parallel-Test/output_1_0.png?raw=true)



### 3）回归法

基本的模型设定如下[^1]： 

$$ y_{it} = \lambda_i + \delta_i + \beta_1 \cdot Treat_{it}*Pre_2 + \beta_2 \cdot Treat_{it}*Pre_1 + \beta_3 \cdot Treat_{it}*Current + \beta_4 \cdot Treat_{it}*Post_1 + \beta_5 \cdot Treat_{it}*Post_2 + \epsilon_{it} $$  

式中：

$\lambda_i$ 表示个体固定效应，$\delta_i$ 表示时间固定效应，Pre、Current和Post表示指示相对于干预年份的dummy变量，$Treat_{it}$ 表示干预的指示变量。**这里需要考虑是否要放入控制变量**[^2]

原理很简单，就是将每一年与干预年的关系做成虚拟变量，然后与干预变量做交互，放入回归中，看干预前年份的回归系数是否有统计学意义，如果无统计学意义，则说明满足平行趋势假设。

当然最理想的情况是干预前的年份全都无统计学意义，如果有个别年份有差异，也可认为满足平行趋势假设。



**首先生成年份的虚拟变量与treated交互项**

```python
panel["period"] = panel["year"] - 1994
vars_pre = ["pre_" + str(i) for i in np.arange(4, 0, -1)]
vars_post = ["post_" + str(i) for i in np.arange(1, 6, 1)]

for i, vars in enumerate(vars_pre):
    panel[vars] = np.where(panel["period"] == (i-4), 1, 0) * panel["treated"]

for i, vars in enumerate(vars_post):
    panel[vars] = np.where(panel["period"] == (i+1), 1, 0) * panel["treated"]

panel["current"] = np.where(panel["period"] == 0, 1, 0) * panel["treated"]
```

**然后进行回归**

主要是双向固定效应模型，可以采用ols，通过lsdv的估计方式加入个体和时间固定效应，也可以采用如`stata`中的*xtreg*函数和`R`中的*plm*函数直接进行面板回归，但是需要注意的是，最好计算稳健聚类的std.err，否则容易低估std.err，导致p值显著，出现假阳性。

```python
import statsmodels.formula.api as smf

formula = "y ~ " + " + ".join(vars_pre) + " + current + " + " + ".join(vars_post) + "+ C(year) + C(country)"
did = smf.ols(formula, data=panel).fit(cov_type="cluster", cov_kwds={"groups" : panel["country"]})
did.summary()
```

**需要特别说明的是**：最好将`pre_1`作为对照组（基期），即在模型中不放入`pre_1`，以便更好的观察干预前的组间差异。若像本例中，不特别指定基期，那么模型会自动drop一期，不同软件中规则不一样，比如本例中是将最后一期`post_6`给drop掉了，也就是`post_6`的系数为0.


<table class="simpletable">
<caption>OLS Regression Results</caption>
<tr>
  <th>Dep. Variable:</th>            <td>y</td>        <th>  R-squared:         </th> <td>   0.538</td>
</tr>
<tr>
  <th>Model:</th>                   <td>OLS</td>       <th>  Adj. R-squared:    </th> <td>   0.292</td>
</tr>
<tr>
  <th>Method:</th>             <td>Least Squares</td>  <th>  F-statistic:       </th> <td>   2.117</td>
</tr>
<tr>
  <th>Date:</th>             <td>Sun, 23 Feb 2020</td> <th>  Prob (F-statistic):</th>  <td> 0.194</td> 
</tr>
<tr>
  <th>Time:</th>                 <td>22:17:18</td>     <th>  Log-Likelihood:    </th> <td> -1599.7</td>
</tr>
<tr>
  <th>No. Observations:</th>      <td>    70</td>      <th>  AIC:               </th> <td>   3249.</td>
</tr>
<tr>
  <th>Df Residuals:</th>          <td>    45</td>      <th>  BIC:               </th> <td>   3306.</td>
</tr>
<tr>
  <th>Df Model:</th>              <td>    24</td>      <th>                     </th>     <td> </td>   
</tr>
<tr>
  <th>Covariance Type:</th>       <td>cluster</td>     <th>                     </th>     <td> </td>   
</tr>
</table>
<table class="simpletable">
<tr>
         <td></td>            <th>coef</th>     <th>std err</th>      <th>z</th>      <th>P>|z|</th>  <th>[0.025</th>    <th>0.975]</th>  
</tr>
<tr>
  <th>Intercept</th>       <td>-1.004e+09</td> <td> 1.44e+09</td> <td>   -0.697</td> <td> 0.486</td> <td>-3.83e+09</td> <td> 1.82e+09</td>
</tr>
<tr>
  <th>C(year)[T.1991]</th> <td> 1.003e+09</td> <td> 2.52e+09</td> <td>    0.398</td> <td> 0.691</td> <td>-3.94e+09</td> <td> 5.94e+09</td>
</tr>
<tr>
  <th>C(year)[T.1992]</th> <td> 4.278e+08</td> <td> 1.57e+09</td> <td>    0.273</td> <td> 0.785</td> <td>-2.65e+09</td> <td>  3.5e+09</td>
</tr>
<tr>
  <th>C(year)[T.1993]</th> <td> 4.003e+09</td> <td> 2.01e+09</td> <td>    1.995</td> <td> 0.046</td> <td> 7.03e+07</td> <td> 7.94e+09</td>
</tr>
<tr>
  <th>C(year)[T.1994]</th> <td> 4.616e+09</td> <td> 2.18e+09</td> <td>    2.115</td> <td> 0.034</td> <td> 3.38e+08</td> <td> 8.89e+09</td>
</tr>
<tr>
  <th>C(year)[T.1995]</th> <td> 5.214e+09</td> <td> 1.87e+09</td> <td>    2.788</td> <td> 0.005</td> <td> 1.55e+09</td> <td> 8.88e+09</td>
</tr>
<tr>
  <th>C(year)[T.1996]</th> <td> 3.412e+09</td> <td> 1.37e+09</td> <td>    2.494</td> <td> 0.013</td> <td> 7.31e+08</td> <td> 6.09e+09</td>
</tr>
<tr>
  <th>C(year)[T.1997]</th> <td> 4.195e+09</td> <td> 1.55e+09</td> <td>    2.700</td> <td> 0.007</td> <td> 1.15e+09</td> <td> 7.24e+09</td>
</tr>
<tr>
  <th>C(year)[T.1998]</th> <td> 3.075e+09</td> <td> 1.51e+09</td> <td>    2.031</td> <td> 0.042</td> <td> 1.07e+08</td> <td> 6.04e+09</td>
</tr>
<tr>
  <th>C(year)[T.1999]</th> <td> 1.376e+09</td> <td> 2.78e+09</td> <td>    0.495</td> <td> 0.620</td> <td>-4.07e+09</td> <td> 6.82e+09</td>
</tr>
<tr>
  <th>C(country)[T.B]</th> <td>-1.514e+09</td> <td>    2.635</td> <td>-5.75e+08</td> <td> 0.000</td> <td>-1.51e+09</td> <td>-1.51e+09</td>
</tr>
<tr>
  <th>C(country)[T.C]</th> <td>-3.835e+08</td> <td>      nan</td> <td>      nan</td> <td>   nan</td> <td>      nan</td> <td>      nan</td>
</tr>
<tr>
  <th>C(country)[T.D]</th> <td> 1.912e+09</td> <td>      nan</td> <td>      nan</td> <td>   nan</td> <td>      nan</td> <td>      nan</td>
</tr>
<tr>
  <th>C(country)[T.E]</th> <td>-1.067e+09</td> <td>      nan</td> <td>      nan</td> <td>   nan</td> <td>      nan</td> <td>      nan</td>
</tr>
<tr>
  <th>C(country)[T.F]</th> <td> 1.769e+09</td> <td>      nan</td> <td>      nan</td> <td>   nan</td> <td>      nan</td> <td>      nan</td>
</tr>
<tr>
  <th>C(country)[T.G]</th> <td>-8.405e+07</td> <td>    2.583</td> <td>-3.25e+07</td> <td> 0.000</td> <td> -8.4e+07</td> <td> -8.4e+07</td>
</tr>
<tr>
  <th>pre_4</th>           <td> 2.141e+09</td> <td> 1.71e+09</td> <td>    1.253</td> <td> 0.210</td> <td>-1.21e+09</td> <td> 5.49e+09</td>
</tr>
<tr>
  <th>pre_3</th>           <td> 1.241e+09</td> <td> 2.43e+09</td> <td>    0.510</td> <td> 0.610</td> <td>-3.53e+09</td> <td> 6.01e+09</td>
</tr>
<tr>
  <th>pre_2</th>           <td> 2.651e+09</td> <td> 6.91e+08</td> <td>    3.835</td> <td> 0.000</td> <td>  1.3e+09</td> <td> 4.01e+09</td>
</tr>
<tr>
  <th>pre_1</th>           <td> 2.618e+08</td> <td> 2.28e+09</td> <td>    0.115</td> <td> 0.909</td> <td>-4.21e+09</td> <td> 4.73e+09</td>
</tr>
<tr>
  <th>current</th>         <td>-9.168e+07</td> <td> 1.84e+09</td> <td>   -0.050</td> <td> 0.960</td> <td>-3.71e+09</td> <td> 3.52e+09</td>
</tr>
<tr>
  <th>post_1</th>          <td>-6.432e+09</td> <td> 2.91e+09</td> <td>   -2.211</td> <td> 0.027</td> <td>-1.21e+10</td> <td>-7.31e+08</td>
</tr>
<tr>
  <th>post_2</th>          <td>  -1.9e+08</td> <td> 1.56e+09</td> <td>   -0.122</td> <td> 0.903</td> <td>-3.25e+09</td> <td> 2.87e+09</td>
</tr>
<tr>
  <th>post_3</th>          <td> 1.035e+09</td> <td> 1.17e+09</td> <td>    0.887</td> <td> 0.375</td> <td>-1.25e+09</td> <td> 3.32e+09</td>
</tr>
<tr>
  <th>post_4</th>          <td>-2.716e+09</td> <td>  3.4e+09</td> <td>   -0.798</td> <td> 0.425</td> <td>-9.38e+09</td> <td> 3.95e+09</td>
</tr>
<tr>
  <th>post_5</th>          <td> 2.718e+09</td> <td>  2.4e+09</td> <td>    1.134</td> <td> 0.257</td> <td>-1.98e+09</td> <td> 7.42e+09</td>
</tr>
</table>
<table class="simpletable">
<tr>
  <th>Omnibus:</th>       <td> 1.113</td> <th>  Durbin-Watson:     </th> <td>   1.836</td>
</tr>
<tr>
  <th>Prob(Omnibus):</th> <td> 0.573</td> <th>  Jarque-Bera (JB):  </th> <td>   0.541</td>
</tr>
<tr>
  <th>Skew:</th>          <td>-0.144</td> <th>  Prob(JB):          </th> <td>   0.763</td>
</tr>
<tr>
  <th>Kurtosis:</th>      <td> 3.320</td> <th>  Cond. No.          </th> <td>9.40e+15</td>
</tr>
</table><br/><br/>Warnings:<br/>[1] Standard Errors are robust to cluster correlation (cluster)<br/>[2] The smallest eigenvalue is 9.79e-31. This might indicate that there are<br/>strong multicollinearity problems or that the design matrix is singular.



**画coef plot**

为了更为直观的观察各年份虚拟变量交互项是否具有统计学意义（即是否包含0），可以将上述回归模型的系数作图，通常称为coef plot，`stata`和`R`中均有相应的package，都名为*coefplot*，操作十分简便。

以下为通过python自定义的coef plot函数[^3]。

图中不仅可以看出干预前年份的平行趋势，同样可以看出干预后的效应，即did的效应。

```python
def coef_plot(fit, vars, label, ax):
    '''ploting the coef of a fit'''
    # extract coef and ci from a model
    coef_ci = pd.DataFrame({"vars" : fit.params.index.values[1:],
                            "coef" : fit.params.values[1:], 
                            "lower" : fit.conf_int().iloc[1:, 0], 
                            "upper" : fit.conf_int().iloc[1:, 1]})
    coef_ci["err"] = coef_ci["coef"] - coef_ci["upper"]
    df = coef_ci[coef_ci["vars"].isin(vars)]

    # plot
    df.plot(x="vars", y="coef", kind="scatter", color="black", yerr="err", marker="s", s=80, legend=False, ax=ax)
    ax.set_xlabel("")
    ax.set_ylabel("Coefficient", fontsize=16)

    ax.axhline(y=0, linestyle='--', color='black', linewidth=2)
    ax.xaxis.set_ticks_position('none')
    ax.set_xticklabels(label, rotation=0, fontsize=16)
```

```python
vars = vars_pre + vars_post
vars.insert(4, "current")

fig, ax = plt.subplots(figsize=[12, 8])
coef_plot(did, vars, vars, ax)
```

![](https://github.com/shumchi/shumchi.github.io/blob/master/_posts/2020-02-23-DID-Parallel-Test/output_6_0.png?raw=true)





[^1]:https://stats.stackexchange.com/questions/160359/difference-in-difference-method-how-to-test-for-assumption-of-common-trend-betw/160361#160361
[^2]: 关于是否需要加入控制变量，没有找到统一的说法，个人理解为基准模型中不需要加入控制变量，如果基准模型显示干预前年份的交互项有统计学意义，则可考虑加入控制变量
[^3]: 自编函数参考并修改自https://zhiyzuo.github.io/Python-Plot-Regression-Coefficient/



