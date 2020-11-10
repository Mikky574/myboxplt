<font size="8">myplt箱线图代码解释</font><br />

---

myplt箱线图作业代码解释文件兼使用说明

简介：箱线图myplt类大体分为三个主要代码块，info_boxplot（），histobox_plot（）和creative_boxplot（）。介绍我会分四部分介绍。

import的包

```python
#!/usr/bin/python3
# -*- coding: UTF-8 -*-
import pandas as pd
import matplotlib.pyplot as plt
import numpy as np
```

---

# Table of Contents

> 1. [功能性代码解释](#功能性代码解释)
> 2. [info_boxplot()解释以及绘图逻辑结构图](#info_boxplot解释以及绘图逻辑结构图)
> 3. [histobox_plot()解释以及绘图逻辑结构图](#histobox_plot解释以及绘图逻辑结构图)
> 4. [creative_boxplot()解释以及绘图逻辑结构图](#creative_boxplot解释以及绘图逻辑结构图)

~~正文开始~~

---

## **功能性代码解释**

myplt类中的cal_Quantile(),cal_max_min(),find_ex_point(),cul_all_max_min(),cul_step()函数解释。

### 1. cal_Quantile()

代码源码

```python
#!/usr/bin/python3
# -*- coding: UTF-8 -*-
def cal_Quantile(da):
    if da.count()%2==0:
        p_50=(da[int(da.count()/2-0.5)]+da[int(da.count()/2)])/2
        da1=da[:int(da.count()/2)]
    else:
        p_50=da[int(da.count()/2)]
        da1=da[:int((da.count()+1)/2)]
    if da1.count()%2==0:
        p_75=(da1[int(da1.count()/2-0.5)]+da1[int(da1.count()/2)])/2
        da2=da[int(da.count()/2):]
    else:
        p_75=da1[int(da1.count()/2)]
        da2=da[int((da.count()+1)/2):]
    da2.reset_index(drop=True, inplace=True)
    if da2.count()%2==0:
        p_25=(da2[int(da2.count()/2-0.5)]+da2[int(da2.count()/2)])/2
    else:
        p_25=da2[int(da2.count()/2)]
    da3=da[int(da1.count()/2):(int(da2.count()/2)+int(da.count()/2))]
    da3.reset_index(drop=True, inplace=True)
    return (p_25,p_50,p_75,da3)
```

输入的数据为单列按大小排序后的Series数据da，第一个if按中位数规则得到p_50，即中位数（奇数取第((da个数+1)/2)项，偶数取第（da个数/2)项）。然后二等分da得到前半部分da1。以da1为总数据集按上述规则再取中位数得到p_75。然后然后二等分da得到后半部分da2。按同样规则以da2为总数据集得到p_25。da3为p_25到p_50之间的数据集，这会在后面计算5%(cal_per5)以画复杂图中会用到。
函数返回值为三个数字p_25,p_50和p_75，即四分位数,以及一个Series数据列da3。

### 2. cal_max_min()

```python
def cal_max_min(da,p_25,p_75):
    ma=da[da<(p_25+(p_75-p_25)*1.5)].max()
    mi=da[da>(p_25-(p_75-p_25)*1.5)].min()
    return (ma,mi)
```

输入的数据为单列按大小排序后的Series数据da，和四分位数中的25%和75%。返回值为箱线图合理范围内的最大值和最小值。用于绘制箱线图的上边和下边，以及计算异常值。

### 3. find_ex_point()

```python
def find_ex_point(data):
    da=pd.Series(data).astype(int)
    da=da.sort_values(ascending=False)
    da.reset_index(drop=True, inplace=True)
    p_25,p_50,p_75=myplt.cal_Quantile(da)[:3]
    da_ex=da[(da<(p_25-(p_75-p_25)*1.5)) | (da>(p_25+(p_75-p_25)*1.5))]
    return da_ex
```

输入的数据为单层list数据data，在函数体内会调用之前的cal_Quantile()函数。计算后返回单列Series数据，内容为data中的异常值。

### 4. cul_all_max_min()

```python
def cul_all_max_min(data_l):
    data_l=[[int(i) for i in k] for k in data_l]
    return (max([max(i) for i in data_l]),min([min(i) for i in data_l]))
```

输入的数据为多层list数据data_l,第一步先将数据全部转换为int型数据，之后计算所有数据中的最大值和最小值。返回两个int型的数据，即，所有数据中的最大值和最小值，用于设定creative_boxplot()中公共y轴坐标。

### 5. cul_step()

```python
def cul_step(da_ma,da_mi):
    n=(da_ma-da_mi)/4
    count=0
    while n>10:
        count+=1
        n/=10
    step=int(n)*10**count
    return step
```

输入的数据为当个数据列da中的最大值da_ma和最小值da_mi。返回值为让y轴区间合适的步长。用于设定creative_boxplot()中的y轴坐标。

---

# **info_boxplot()解释以及绘图逻辑结构图**

## info_boxplot代码源码

```python
def info_boxplot(ax,data_l,multiplebox=True,linecolor='black',pointcolor='black'):
    ax.cla()
    lenth=len(data_l)
    ax.set_xlim((0,lenth+1))
    tick_x=range(1,lenth+1)
    ax.set_xticks(tick_x)
    ax.set_xticklabels([str(i) for i in tick_x])
    def cal_per5(da3):
        per_5=[]
        add=da3.count()*0.1
        count=0
        for i in range(9):
            count+=add
            per_5.append(da3[int(count-1)])
        return per_5
    def draw_boxplot(ax,mi,ma,p_25,p_50,p_75,multiplebox,da_ex,per_5,k,linecolor='black',pointcolor='black'):
        ax.vlines(k,mi,p_25,color=linecolor,linewidth=1)
        ax.vlines(k,p_75,ma,color=linecolor,linewidth=1)
        ax.hlines(mi,k-0.15,k+0.15,color=linecolor,linewidth=1)
        ax.hlines(ma,k-0.15,k+0.15,color=linecolor,linewidth=1)
        rect = plt.Rectangle((k-0.25,p_25),0.5,p_75-p_25,linewidth=1, edgecolor=linecolor, facecolor='none')
        ax.add_patch(rect)
        #异常点
        for i in da_ex.values:
            ax.text(k,i , "o",color=pointcolor, fontsize=5, verticalalignment="center",horizontalalignment='center')
            #ax.scatter(k,i,color='white', marker='o', edgecolors='black')
        ax.vlines(k,da_ex.min(),da_ex.max(),color='white',linewidth=1,zorder=0)
        if (not multiplebox):
            ax.hlines(p_50,k-0.25,k+0.25,color=linecolor,linewidth=1)
        else:
            #5%
            for i in per_5:
                ax.hlines(i,k-0.25,k+0.25,color=linecolor,linewidth=1)
            ax.hlines(p_50,k-0.25,k+0.25,color=linecolor,linewidth=3)
    def main_box(ax,data,multiplebox,k,linecolor,pointcolor):
        da=pd.Series(data).astype(int)
        da=da.sort_values(ascending=False)
        da.reset_index(drop=True, inplace=True)
        p_25,p_50,p_75,da3=myplt.cal_Quantile(da)
        per_5=cal_per5(da3)
        per_5[4]=p_50
        ma,mi=myplt.cal_max_min(da,p_25,p_75)
        da_ex=da[(da<(p_25-(p_75-p_25)*1.5)) | (da>(p_25+(p_75-p_25)*1.5))]
        draw_boxplot(ax,mi,ma,p_25,p_50,p_75,multiplebox,da_ex,per_5,k,linecolor,pointcolor)
    for k in range(lenth):
        main_box(ax,data_l[k],multiplebox,k+1,linecolor,pointcolor)
```

`before cal_per5`：清空画布，设置x轴标尺和标签

`cal_per5`：输入为pandas区间da3,按个数将da3区间平分为十份，返回list类型的per_5记录的是中间的9条分界线，也可以理解为应该被绘制的y轴刻度。用于绘制复杂图的5%分界线部分。

`draw_boxplot`：输入为绘制当个箱型图所需各种参数，作用为绘制单个箱型图。当`multiplebox`为`True`时多绘制9条5%为界的分界线，绘制复杂箱型图。

`main_box`：用于总领调用各类功能函数。其中输入参数k+1为绘制图形中线的x轴坐标。

## info_boxplot逻辑结构图

```flow
st=>start: 清空画布,设置x轴
e=>end: 数据输入draw_boxplot内进行画图
op1=>operation: for循环使每个进入main_box()的数据都为单层list数据,k+1为x轴标度
op2=>operation: main_box()先把list数据转换为单列Series数据，排序后调用相应函数
op3=>operation: 计算得到四分位int数据p_25,p_50,p_75,Series数据da3,
5%list数据per_5,
合理区间内的最大、最小值int数据ma,mi,
异常值Series数据da_ex

st->op1->op2->op3->e
```

## draw_boxplot逻辑结构图

```flow
st=>start: 绘制mi到p_25和p_75到ma的竖线
e=>end: 最后根据multiplebox参数决定矩形框内只绘制一条中位线还是按5%比例绘制多条线
op1=>operation: 绘制y值等于mi的水平线和y=ma的横线
op2=>operation: 绘制箱型图中部的方形框
op3=>operation: 根据list类型参数da_ex提供的y轴坐标用text标注的方法绘制异常值
绘制一条从异常值的最低点到最高点的白色竖线，z轴参数设为0，以便y轴能覆盖标注形式显示的异常值

st->op1->op2->op3->e
```

## 图片注释

![show picture,p1](img\md_pic1.jpg)

---

# **histobox_plot()解释以及绘图逻辑结构图**

## histobox_plot代码源码

```python
def histobox_plot(ax, data_l,linecolor='black',pointcolor='black',boxcolor='lavender'):
    ax.cla()
    lenth=len(data_l)
    ax.set_xlim((0,lenth+1))
    tick_x=range(1,lenth+1)
    ax.set_xticks(tick_x)
    ax.set_xticklabels([str(i) for i in tick_x])
    def normal_his(da,Rec_begin):
        his_l=[]
        for i in range(len(Rec_begin)-1):
            his_l.append(da[(da<Rec_begin[i+1]) & (da>=Rec_begin[i])].count())
        his_l.append(da[da>=Rec_begin[len(Rec_begin)-1]].count())
        ma_l=max(his_l)
        mi_l=min(his_l)
        his_nor=[((i-mi_l)/(ma_l-mi_l))*0.55 for i in his_l]
        #标准化 [0,1]*0.55
        return his_nor
    def draw_histobox(ax,mi,ma,p_25,p_50,p_75,da_ex,Rec_begin,his_nor,da_m,da_n,k,linecolor='black',pointcolor='black',boxcolor='lavender'):
        ax.vlines(k,mi,ma,color=linecolor,linewidth=1)
        ax.hlines(mi,k-0.15,k,color=linecolor,linewidth=1)
        ax.hlines(ma,k-0.15,k,color=linecolor,linewidth=1)
        ax.hlines(p_50,k-0.25,k,color=linecolor,linewidth=1)
        rect = plt.Rectangle((k-0.25,p_25),0.25,p_75-p_25,linewidth=1, edgecolor=linecolor, facecolor='none')
        ax.add_patch(rect)
        for i in da_ex.values:
            ax.text(k,i , "o", fontsize=5,color=pointcolor, verticalalignment="center",horizontalalignment='center')
            #ax.scatter(k,i,color='white', marker='o', edgecolors='black')
        ax.vlines(k,da_ex.min(),da_ex.max(),color='white',linewidth=1,zorder=0)
        for i in range(len(Rec_begin)):
            rect = plt.Rectangle((k,Rec_begin[i]),0.15+his_nor[i],(da_m-da_n)/10,linewidth=1,edgecolor=linecolor, facecolor=boxcolor)
            ax.add_patch(rect)
    def main_box(ax,data,k,linecolor,pointcolor,boxcolor):
        da=pd.Series(data).astype(int)
        da=da.sort_values(ascending=False)
        da.reset_index(drop=True, inplace=True)
        p_25,p_50,p_75=myplt.cal_Quantile(da)[:3]
        ma,mi=myplt.cal_max_min(da,p_25,p_75)
        da_ex=da[(da<(p_25-(p_75-p_25)*1.5)) | (da>(p_25+(p_75-p_25)*1.5))]
        #这里改
        Rec_begin=np.arange(da.min(),da.max(),(da.max()-da.min())/10)
        his_nor=normal_his(da,Rec_begin)
        draw_histobox(ax,mi,ma,p_25,p_50,p_75,da_ex,Rec_begin,his_nor,da.max(),da.min(),k,linecolor,pointcolor,boxcolor)
    for k in range(lenth):
        main_box(ax,data_l[k],k+1,linecolor,pointcolor,boxcolor)
```

`Rec_begin`：为list数据。内容为单列全部数据da按数值十等分后的结果，从最小值开始记录。用于绘制直方图区间的起始y轴坐标。

`normal_his`：输入为全部数据da和list类型数据Rec_begin。作用为按Rec_begin分的十个区间进行个数统计。得到`his_l`再进行标准化，转换成[0,0.55]大小。得到标准化后的数据list`his_nor`,并返回。

`draw_histobox`：根据输入的参数绘图。

`main_box`：用于总领调用各类功能函数。

## histobox_plot逻辑结构图

```flow
st=>start: 清空画布,设置x轴
e=>end: 数据输入draw_histobox内进行画图
op1=>operation: for循环使每个进入main_box()的数据都为单层list数据,k+1为x轴标度
op2=>operation: main_box()先把list数据转换为单列Series数据，排序后调用相应函数
op3=>operation: 计算得到四分位int数据p_25,p_50,p_75,Series数据da3,
合理区间内的最大、最小值int数据ma,mi,
异常值Series数据da_ex,
直方图区间的起始y轴坐标Rec_begin和直方图方格长度his_nor

st->op1->op2->op3->e
```

## draw_histobox逻辑结构图

```flow
st=>start: 绘制mi到ma的竖线
e=>end: his_nor整体+0.15,使得数据集处于[0.15,0.7]之间。以此绘图
op1=>operation: 绘制y值等于mi的水平线和y=ma的横线
op2=>operation: 绘制左半边箱线图方框
op3=>operation: 根据list类型参数da_ex提供的y轴坐标用text标注的方法绘制异常值
绘制一条从异常值的最低点到最高点的白色竖线，z轴参数设为0，以便y轴能覆盖标注形式显示的异常值
op4=>operation: 根据Rec_begin给的底边坐标和his_nor给的宽度坐标绘制直方图的矩形。

st->op1->op2->op3->op4->e
```

---

# **creative_boxplot()解释以及绘图逻辑结构图**

## creative_boxplot代码源码

```python
def creative_boxplot(ax, data_l,linecolor='black',pointcolor='black',boxcolor='lavender'):
    ax.cla()
    lenth=len(data_l)
    ax.set_xlim((0,lenth*3+1))#lenth+lenth*2+1
    tick_x=range(1,lenth*3+1)
    his_x=list(tick_x)[:lenth]
    sim_x=[j for j in list(tick_x)[lenth:] if j%2==0]
    ax.set_xticks(his_x+sim_x)
    ax.set_xticklabels([str(i) for i in his_x]+["zoom"+str(i) for i in his_x])
    da_ma,da_mi=myplt.cul_all_max_min(data_l)
    step=myplt.cul_step(da_ma,da_mi)
    tick_y=list(range(da_mi-(da_mi%step),da_ma+step*2,step))
    ax.set_ylim((tick_y[0]-step/6,tick_y[-1]+step/6))
    ax.set_yticks(tick_y)
    ax.set_yticklabels([str(i) for i in tick_y])
    def normal_his(da,Rec_begin):
        his_l=[]
        for i in range(len(Rec_begin)-1):
            his_l.append(da[(da<Rec_begin[i+1]) & (da>=Rec_begin[i])].count())
        his_l.append(da[da>=Rec_begin[len(Rec_begin)-1]].count())
        ma_l=max(his_l)
        mi_l=min(his_l)
        his_nor=[((i-mi_l)/(ma_l-mi_l))*0.55 for i in his_l]
        #标准化 [0,1]*0.55
        return his_nor
    def draw_histobox(ax,mi,ma,p_25,p_50,p_75,da_ex,Rec_begin,his_nor,da_m,da_n,k,linecolor='black',pointcolor='black',boxcolor='lavender'):
        ax.vlines(k,mi,ma,color=linecolor,linewidth=1)
        ax.hlines(mi,k-0.15,k,color=linecolor,linewidth=1)
        ax.hlines(ma,k-0.15,k,color=linecolor,linewidth=1)
        ax.hlines(p_50,k-0.25,k,color=linecolor,linewidth=1)
        rect = plt.Rectangle((k-0.25,p_25),0.25,p_75-p_25,linewidth=1, edgecolor=linecolor, facecolor='none')
        ax.add_patch(rect)
        for i in da_ex.values:
            ax.text(k,i , "o", fontsize=5,color=pointcolor, verticalalignment="center",horizontalalignment='center')
            #ax.scatter(k,i,color='white', marker='o', edgecolors='black')
        ax.vlines(k,da_ex.min(),da_ex.max(),color='white',linewidth=1,zorder=0)
        for i in range(len(Rec_begin)):
            rect = plt.Rectangle((k,Rec_begin[i]),0.15+his_nor[i],(da_m-da_n)/10,linewidth=1,edgecolor=linecolor, facecolor=boxcolor)
            ax.add_patch(rect)
    def draw_y_axis(mi,ma,step,step1,lenth,tick_y,ratio,k):
        tick_y1=list(range(mi-(mi%step1),ma+step1*2,step1))
        while len(tick_y1)!=len(tick_y):
            if (len(tick_y1)<len(tick_y)):
                tick_y1.append(tick_y1[-1]+step1)
            else:
                tick_y1.pop()
        ax.vlines(lenth+2*k-1,(tick_y1[0]-step/6)*ratio,(tick_y1[-1]+step/6)*ratio,color='black',linewidth=1)
        for i in tick_y1:
            ax.hlines(i*ratio,lenth+2*k-1-0.05,lenth+2*k-1,color='black',linewidth=1)
            ax.text(lenth+2*k-1-0.05, i*ratio , str(i), fontsize=10, verticalalignment="center",horizontalalignment='right')
    def draw_sim_box(ax,mi,ma,p_25,p_50,p_75,ratio,k,lenth,linecolor='black'):
        ax.vlines(lenth+2*k,mi*ratio,p_25*ratio,color=linecolor,linewidth=1)
        ax.vlines(lenth+2*k,p_75*ratio,ma*ratio,color=linecolor,linewidth=1)
        ax.hlines(mi*ratio,lenth+2*k-0.15,lenth+2*k+0.15,color=linecolor,linewidth=1)
        ax.hlines(ma*ratio,lenth+2*k-0.15,lenth+2*k+0.15,color=linecolor,linewidth=1)
        rect = plt.Rectangle((lenth+2*k-0.25,p_25*ratio),0.5,(p_75-p_25)*ratio,linewidth=1, edgecolor=linecolor, facecolor='none')
        ax.add_patch(rect)
        ax.hlines(p_50*ratio,lenth+2*k-0.25,lenth+2*k+0.25,color=linecolor,linewidth=3)
    def main_box(ax,data,k,linecolor,pointcolor,boxcolor):
        da=pd.Series(data).astype(int)
        da=da.sort_values(ascending=False)
        da.reset_index(drop=True, inplace=True)
        p_25,p_50,p_75=myplt.cal_Quantile(da)[:3]
        ma,mi=myplt.cal_max_min(da,p_25,p_75)
        step1=myplt.cul_step(ma,mi)
        ratio=step/step1
        da_ex=da[(da<(p_25-(p_75-p_25)*1.5)) | (da>(p_25+(p_75-p_25)*1.5))]
        Rec_begin=np.arange(da.min(),da.max(),(da.max()-da.min())/10)
        his_nor=normal_his(da,Rec_begin)
        draw_histobox(ax,mi,ma,p_25,p_50,p_75,da_ex,Rec_begin,his_nor,da.max(),da.min(),k,linecolor,pointcolor,boxcolor)
        draw_y_axis(mi,ma,step,step1,lenth,tick_y,ratio,k)
        draw_sim_box(ax,mi,ma,p_25,p_50,p_75,ratio,k,lenth,linecolor)
    for k in range(lenth):
        main_box(ax,data_l[k],k+1,linecolor,pointcolor,boxcolor)
```

`normal_his`、`draw_histobox`：参照hisbox代码解释相同位置的内容。

`draw_y_axis`：按照比例绘制放大图的y轴

`draw_sim_box`：绘制放大后的简单箱型图。

`main_box`：用于总领调用各类功能函数。

## creative_boxplot逻辑结构图

```flow
st=>start: 清空画布,设置x轴
e=>end: 数据输入draw_histobox内进行画原图,draw_y_axis绘制放大图y轴坐标,draw_sim_box绘制放大的箱型图
op1=>operation: 计算所有层数的最大值da_ma,最小值da_mi以此计算合适的步长step。设置所有图像公共的y轴
op2=>operation: for循环使每个进入main_box()的数据都为单层list数据,k+1为x轴标度
op3=>operation: main_box()先把list数据转换为单列Series数据，排序后调用相应函数
op4=>operation: 计算得到四分位int数据p_25,p_50,p_75,Series数据da3,
合理区间内的最大、最小值int数据ma,mi,
放大图的y轴步长step1,放大图与原图的放大比例ratio
异常值Series数据da_ex,
直方图区间的起始y轴坐标Rec_begin和直方图方格长度his_nor

st->op1->op2->op3->op4->e
```

# 使用示例

data enter
![show picture,data enter](img\md_pic2.png)
info_boxplot,True
![show picture,info_boxplot,True](img\md_pic3.png)
info_boxplot,False
![show picture,info_boxplot,False](img\md_pic4.png)
histobox_plot
![show picture,histobox_plot](img\md_pic5.png)
creative_boxplot
![show picture,creative_boxplot](img\md_pic6.png)