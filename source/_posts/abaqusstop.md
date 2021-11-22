---
title: Abaqus job monitor
toc: true
tags: [Python, Abaqus]
categories: Python
keywords: Python
date: 2021-07-30 15:30:16
---
![abaqus]

# 运用python对abaqus进行监控
 在通过abaqus进行计算时，很多时候通过监视器或者查看.sta文件是太清楚解析算到哪一步的。
 **运用python对abaqus CAE内置的监视器进行监控管理，特别对于非线性的分析能可以最大限度计算得到自己想要的数据，减少计算成本。**
<!--more-->

## 监控原理
利用abaqus CAE里的监视器，对模型指定的集合进行监视，分析过程中如果长变量达到某个指定的值就中止分析或者跳过这个分析步之类的指令。
这个命令可以用过monitorManager对象中写回调函数（callback function）来完成。
```python
def printMessages(jobname,messagetype,data,userdata):
 print 'job name：%s, messagetype:%s'%(jobname, messagetype)
 print 'data members:'
 format = '%-18s %s'
 print format%('member','value')
 members = dir(data)
 for member in members:
 	membervalue = getattr(data,member)
 	print format%(member, membervalue)
 
monitorManager.addMessageCallback(ANY_job,ANY_message_type,printMessages,None)
```
## 实例
现在我拉伸一块被焊接的钢板，想让他在位移达到1mm的时候停下。

首先对想要监控的区域做一个集合，这里设了点节点52作为监视集合，可以看到.inp文件里会出现如下关键字句。
```
**********
*Nset，nset=monitor, instance=mainplate-1 52,
**********
```
**这一句是设置集合的时候自动生成，当然自己添加关键句也是可以的。*

其次，在想要监控输出的分析步里，输入参数，这里想要测量的是y方向的位移（U2），关键句如下：
```
**********
*Step,name=Step-1, nlgeom, inc=100
compress dome
*Static, riks
0.2, 1., 1e-05,1.0, , monitor, 2, -3.7
.....
*Monitor, dof=2, node=monitor, frequncy=1	#（这个需要自己添加）
*End Step
**********
```
参数：
dof=2 	指U2位移
node=monitor		是指监视名为monitor的集合
frequecy=1	监视的频率，可以根据自己的分析步调整，一般为1.

---------
python 思路
 

 1.  定义函数 monitorDataValue(), 来输出分析的时间（data.time）和数值（data.value）

```python
def monitorDataValue(jobName, messageType, data, userData):
    print "%-8s  %s"%(data.time, data.value)
```

 2. 修改函数monitorDataValue()，通过监视集合monitor的U2，使其位移超过1mm的时候就停止计算。

```python
    if data.value < -1 :               
        mdb.jobs[jobName].kill()
```
3. 调整addMessageCallback()函数中的参数
* 将参数ANY_job赋值’123‘。（这里job的文件名为123）
* 将信息类型ANY_Message_type调整为 MONITOR_DATA。
* 将回调函数名字由printMessages调整为monitorDataValue
```python
 monitorManager.addMessageCallback('123', MONITOR_DATA,
    monitorDataValue, None)        
```
4. 通过CAE或者abaqus command里的’abaqus CAE script = xx.py‘来运行。

__完整代码如下__
```python
# -* - coding:UTF-8 -*-
from abaqus import * 
from abaqusConstants import * 
import job
from jobMessage import *

def monitorDataValue(jobName, messageType, data, userData):
    print "%-8s  %s"%(data.time, data.value)
    if data.value < -1 :               
        mdb.jobs[jobName].kill()

monitorManager.addMessageCallback('123', MONITOR_DATA,
    monitorDataValue, None)        

job = mdb.JobFromInputFile('123', '123.inp')
job.submit()  
```
---
# 运行结果
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210215174127684.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2thZWRlMHYw,size_16,color_FFFFFF,t_70#pic_center)

---
__拓展：__
监视器的功能很强大，可以结合python对很多参数进行监控和控制，比如还可以监控载荷，监控应变，来控制分析步的跳过，中止，以及输出变量之类的。


---

参考：
曹金凤，王旭春，孔亮：Python语言在Abaqus中的应用，机械工业出版社，2011.7


[abaqus]: https://img.shields.io/badge/Abaqus-V.2020-blue?logo=Dassault%20Syst%C3%A8mes
