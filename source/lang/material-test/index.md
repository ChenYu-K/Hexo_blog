---
title: Material_test-data-processing
date: 2021-08-24 18:23:25
tags: [Python, Matlab, Data_processing]
categories: Python
toc: true
recommend: 0.6
---

@[toc]
## 材料試験数据处理
---
[日本語](/public/2021/08/24/2021-08-24-Material-test/index.html)

钢材的材料试验自动数据处理python脚本<br>
可以实现大量数据的一键处理，算出自动算出材料试验的屈服强度，最大强度，弹性模量，泊松比，最后自动出图，并输出结果在excel表李
<div class="img-x">
<img  height =" 20 " width =" 20 " src =" https://cdn.jsdelivr.net/npm/simple-icons@v5/icons/googlecolab.svg "/> <img  height =" 20 " width =" 20 " src =" https://cdn.jsdelivr.net/npm/simple-icons@v5/icons/jupyter.svg "/></div>

> <i class="fab fa-google-drive"></i>   Google colab　ソースコード[source_code](https://colab.research.google.com/drive/1ED1jSNf7LIF0wjTKKv-pL2neszNev9Gs?usp=sharing)

<!--more-->

<script src="https://gist.github.com/ChenYu-K/1e1d40352924237fd8f503b637983d3b.js"></script>

### 導入

---
最开始从csv的初始数据里，读取必要的数据然后根据截面积算出载荷对应的应力，应变。然后算各应变的平均值。

>在这里，原始数据得来的ch. 为了更好实现一键化处理，增加了横向应变和纵向应变的判定。通过判断中间段数据应变的正负来判断拉应变还是压应变，以此区分横向应变和纵向应变
```python
from glob import glob
from google.colab import drive
import csv
import numpy as np
from openpyxl import load_workbook
import matplotlib.pyplot as plt
 
###google drive 链接google云盘
drive.mount('/content/drive')
%cd '/content/drive/MyDrive/Colab_Notebooks/material_test'
###

filenames = glob('*.csv')
print(filenames)
 
#for filename in filenames:
with open('材料試験_H300W_No.1_データリスト.csv',encoding="utf8",errors='ignore') as f:
  f_csv = csv.reader(f)
  next(f_csv)
  data = list(f_csv)
  a = np.array(data)
  load = a[1:,2]
  a = a[1:,:]         #文字列の次から
  x = range(np.shape(a)[0])
  y = len(data) - 1     #行数を数える
  case1 = np.empty([y,8],dtype = float)
 
#########################应变的判定，csv数据位置不会变的可以忽略这段
for i in x:
  case1[i,0] = a[i,2]  #load
  case1[i,1] = a[i,3]  #disp
  if float(a[int(y/2),4]) > 0 :   #应变的判定
    case1[i,2] = a[i,4]       #縦ひずみ1を定義する
    if float(a[int(y/2),5]) > 0 : #縦ひずみを判定する
      case1[i,3] = a[i,5]     #縦ひずみ2を定義する
      case1[i,4] = a[i,6]     #横ひずみ1を定義する
      case1[i,5] = a[i,7]     #横ひずみ2を定義する
    else :
      case1[i,4] = a[i,5]      #横ひずみ1を定義する
      case1[i,5] = a[i,7]    #横ひずみ2を定義する
      case1[i,3] = a[i,6]    #縦ひずみ2を定義する
  if i == y :
    break
  striantate = (case1[:,2]+case1[:,3])/2        #縦ひずみ平均値
  strianyoko = (case1[:,4]+case1[:,5])/2        #横ひずみ平均値
############################
#　エクセルシート（テンプレート）を読み込む
wb = load_workbook("template_mt.xlsx", data_only=True)  #エクセルシートを導入する
wb1 = wb.active           #シートを選択する

#################
 
A = wb1['F9'].value   #获取材料试验片的断面积
case1[:,6] = case1[:,0] *1000/ A     #算出应力
 
#################
```

### 屈服强度的计算

---

对于标准的材料试验，根据弹塑性力学原理，一般来说，钢材的应变在4000左右是处在屈服点之后，应变强化之前。所以对于屈服强度的判定，如以下：
　
> 应变在4000之前的最大应力则为屈服强度
 
 　拉升强度则为整个实验的最大应力值

```python
sigma_y = 0
i = 0
while striantate[i] < 4000:
  sigma_y= max(sigma_y,case1[i,6])
  i = i+1
sigma_u = max(case1[:,6])
print('屈服强度：',sigma_y)
print('最大强度：',sigma_u)
```

### 通过最小二乘法计算弹性模量E
---

- 为了去除实验初期载荷的不规律变化，以及屈服点附近的塑性变形的影响。试验数据一般采用$[0.2\sigma_y - 0.7\sigma_y]$的范围作为弹性变形的范围，并截取这一段的数据来计算弹性模量以及泊松比。
- 通过最小二乘法计算，应力应变曲线的斜率$k_1$，来获得弹性模量：


$$[k_1 =\frac{n \sum_{i=1}^{n} x_{i} y_{i}-\sum_{i=1}^{n} x_{i} \sum_{i=1}^{n} y_{i}}{n \sum_{i=1}^{n} x_{i}^{2}-\left(\sum_{i=1}^{n} x_{i}\right)^{2}}$$

<br>

 - 为了更直观的表示以上计算式，在代码里改写为以下．
$$ k_1 = \frac {na1 - a21\times a22} {nb1 - a21^2}$$

- 另外，对于決定係数$R^2$的计算，通过1减去离差平方和除以偏差减去数据的平均值$\overline{y}$的平方和来获得，如以下计算式：


$$R^{2} = 1-\frac{\sum_{i=1}^{N}\left(y_{i}-f_{i}\right)^{2}}{\sum_{j=1}^{N}\left(y_{j}-\bar{y}\right)^{2}}$$

```python
i = 0; a1 = 0; a21 = 0; a22 = 0; k1 = 0; b1 = 0; n=0;pc = 0
for j in case1[:,6]:
  if j > 0.2*sigma_y and j < 0.7*sigma_y:
    a1 += (striantate[i] * case1[i,6])
    a21 += striantate[i]
    a22 += case1[i,6]
    b1 += striantate[i] ** 2
    pc += strianyoko[i]/striantate[i]
    n += 1
  i = i+1

k1 = (n*a1 - a21*a22)/(n*b1-a21**2)     #yong's modulus
b0 = a22/n - k1*a21/n            #intercept　切片
pc = -pc/n                   #Poisson coefficient ポアソン比
####### R決定係数の求め
i = 0; ssr = 0; sst = 0
ymean = a22/n
for j in case1[:,6]:
  if j > 0.2*sigma_y and j < 0.7*sigma_y:
    ssr += (case1[i,6] - (striantate[i]*k1+b0))**2
    sst += (case1[i,6] - ymean)**2
  i += 1
R2 = 1 - ssr/sst        #R2 Coefficient of determination，決定係数
######
print('yong\'s modulus E = ',k1*10**6)
print('Poisson coefficient = ',pc)
print('coefficient of determinationR^2 = ',R2)
```

### 出图

---

通过 matplotlib自动画出应力应变图

 ```python
##############
#グラフを書く
plt.plot(striantate,case1[:,6], label="stress-strain",color='black',linewidth=0.75)
plt.xlabel("Strain")
plt.ylabel("Stress [$N/mm^2$]")
plt.ylim(0, 500)
plt.xlim(0, 70000)
plt.grid(color="k", linestyle=":") # メッシュ背景を点線に設定する
plt.legend(loc=4,numpoints=1)            #凡例
plt.savefig('stress-strain.svg')    #svgとして保存
plt.show()
###############
### 数据的写入
最后处理玩得数据写入excel之中．
wb1.cell(6,10,sigma_y)
wb1.cell(6,11,sigma_u)
wb1.cell(6,12,k1)
wb1.cell(6,13,pc)
for i in range(3,y) :
  for j in range(2,8) :
    wb1.cell(i,j,case1[i-3,j-2]) #シートにi行j列にデータを書き込む
wb.save("111.xlsx")     #保存する
```

## Full Code
完整代码如以下，大致的结构流程为
1. 对文件夹内的`.csv`文件进行查找，取得所有csv的文件名。
2. 读取原始数据，并开始数据处理
3. 从原始数据中算出应变的平均值以及计算出应力。
4. 算出屈服强度以及最大强度
5. 通过最小二乘法计算出弹性模量以及泊松比
6. 自动画出应力应变图，并输出保存为`.svg`
7. 计算结果以及原始数据保存在Excel文档里输出报告


```python material-test.py >folded
from glob import glob
from google.colab import drive
import csv
import numpy as np
from openpyxl import load_workbook
import matplotlib.pyplot as plt
 
###
drive.mount('/content/drive')
%cd '/content/drive/MyDrive/Colab_Notebooks/material_test'
filenames = glob('*.csv')
###
m=1;  #生データ書き込むためcellの列数，
row1 = 6  #計算結果を書き込むための行数
############################
#　エクセルシート（テンプレート）を読み込む
wb = load_workbook("template_mt.xlsx", data_only=True)  #エクセルシートを導入する
wb1 = wb.active           #シートを選択する

######
for filename in filenames:
  fname = filename.split('_')[-2]
  with open(filename,encoding="utf8",errors='ignore') as f:
    f_csv = csv.reader(f)
    next(f_csv)
    data = list(f_csv)
    a = np.array(data)
    load = a[1:,2]
    a = a[1:,:]         #文字列の次から
    x = range(np.shape(a)[0])
    y = len(data) - 1     #行数を数える
    case1 = np.empty([y,8],dtype = float)
  
  #########################
  for i in x:
    case1[i,0] = a[i,2]  #load
    case1[i,1] = a[i,3]  #disp
    if float(a[int(y/2),4]) > 0 :   #縦ひずみを判定する
      case1[i,2] = a[i,4]       #縦ひずみ1を定義する
      if float(a[int(y/2),5]) > 0 : #縦ひずみを判定する
        case1[i,3] = a[i,5]     #縦ひずみ2を定義する
        case1[i,4] = a[i,6]     #横ひずみ1を定義する
        case1[i,5] = a[i,7]     #横ひずみ2を定義する
      else :
        case1[i,4] = a[i,5]      #横ひずみ1を定義する
        case1[i,5] = a[i,7]    #横ひずみ2を定義する
        case1[i,3] = a[i,6]    #縦ひずみ2を定義する
    if i == y :
      break
    striantate = (case1[:,2]+case1[:,3])/2        #縦ひずみ平均値
    strianyoko = (case1[:,4]+case1[:,5])/2        #横ひずみ平均値

  #################
  
  A = wb1.cell(row1,6).value   #断面積を取得する
  case1[:,6] = case1[:,0] *1000/ A     #応力の算出
  
  #################降伏点および引張強度の算出
  sigma_y = 0
  i = 0
  while striantate[i] < 4000:
    sigma_y= max(sigma_y,case1[i,6])
    i = i+1
  sigma_u = max(case1[:,6])
  ##############ヤング率の算出
  i = 0; a1 = 0; a21 = 0; a22 = 0; k1 = 0; b1 = 0; n=0;pc = 0;
  for j in case1[:,6]:
    if j > 0.2*sigma_y and j < 0.7*sigma_y:
      a1 += (striantate[i] * case1[i,6])
      a21 += striantate[i]
      a22 += case1[i,6]
      b1 += striantate[i] ** 2
      pc += strianyoko[i]/striantate[i]
      n += 1
    i = i+1

  k1 = (n*a1 - a21*a22)/(n*b1-a21**2)     #yong's modulus
  b0 = a22/n - k1*a21/n            #intercept　切片
  pc = -pc/n                   #Poisson coefficient ポアソン比
  ####### R決定係数の求め
  i = 0; ssr = 0; sst = 0
  ymean = a22/n
  for j in case1[:,6]:
    if j > 0.2*sigma_y and j < 0.7*sigma_y:
      ssr += (case1[i,6] - (striantate[i]*k1+b0))**2
      sst += (case1[i,6] - ymean)**2
    i += 1
  R2 = 1 - ssr/sst        #R2 Coefficient of determination，決定係数
  ##########グラフを書く
  plt.plot(striantate,case1[:,6], label="stress-strain",color='black',linewidth=0.75)
  plt.xlabel("Strain")
  plt.ylabel("Stress [$N/mm^2$]")
  plt.ylim(0, 500)
  plt.xlim(0, 70000)
  plt.grid(color="k", linestyle=":") # メッシュ背景を点線に設定する
  plt.legend(loc=4,numpoints=1)            #凡例
  plt.savefig(fname+'-stress-strain.svg')    #svgとして保存
  plt.show()
  ###############データの書き込む
  wb1.cell(row1,10,sigma_y)
  wb1.cell(row1,11,sigma_u)
  wb1.cell(row1,12,k1)
  wb1.cell(row1,13,pc)

  for i in range(15,y) :
    j1 = 0
    for j in range(m,m+6) :
      wb1.cell(i,j,case1[i-15,j1]) #シートにi行j列にデータを書き込む
      j1 +=1
  
  m += 6
  row1 += 1

  wb.save("test.xlsx")     #保存する
```

### Matlab ver.

---

>Matlabのバージョンも作っています．考え方は基本一緒．

```atlab material-test.m >folded
%%
clc                                 %コマンドウインドウをクリアする
clear                               %ワークスペースのクリア
file = dir(fullfile('*.csv'));      %csvファイルの情報を全部読み込む
filenames = {file.name};            %csvファイル名を取得
[~,n] = size(filenames);            %csvファイルの個数を数える
A = readmatrix('template_mt.xlsx','sheet',1,'range','F6:F8');   %断面積を取得する

%変数の定義
stressmax = 0; avx = 0; avy = 0; avx2 = 0;
Sxy = 0; Sx2 = 0; M = 0; Sxy1 = 0; Sxy2 = 0;

%%
%全てのデータを行列変換
%%%%figure
 figure1 =figure;
 axes1 = axes('Parent',figure1);
hold(axes1,'on');
ylabel({'Stress [N/mm^2]'});
xlabel({'Strain'});
%%%%%%%
for i = 1 : n
     stress_y(i) = 0;
    k = strcat(filenames(i));           %文字列に変換する
    data{i,1} = k{1,1};                 %名前を付けて
    data{i,2} = readmatrix(k{1,1});     %データを入れる
    f = data{i,2};                      %data中のi行2列のデータを取り出す
    f = f(:,3:8);                       %計測データの3～8列(荷重，変位，ひずみ*4)を取り出す
    data{i,2} = f;
    [j,~]=size(data{i,2});      %データ数の列数を数える
    if data{i,2}(j/2,4)>0          %縦ひずみ2を判定する
        next;
    else 
            data{i,2}(:,[4,5]) =data{i,2}(:,[5,4]);         %横ひずみと縦ひずみを並び替える
    end
    
    %応力・平均ひずみの出力
    for m=1:j
        stress(m) = data{i,2}(m,1)/A(i)*1000;           %`応力の出力
        strain(m,1) = 0.5*(data{i,2}(m,3) + data{i,2}(m,4));
        strain(m,2) = -0.5*(data{i,2}(m,5) + data{i,2}(m,6));
        if strain(m,1) < 4000
            stress_y(i)=max(stress(m),stress_y(i));
        end
    end
    stressmax(i) = max(stress);        %応力最大値を取得する

    %平均値を取得する
    for m=1:j
        if (stress(m) > (0.1*stress_y(i))) && (stress(m) < (0.7*stress_y(i)))
             avx = avx + strain(m,1);
             avy = avy + stress(m);
             avx2 = avx2 + strain(m,2);
             M = M+1;
        end
    end 
    avx = avx/M ; avy = avy/M; avx2 = avx2/M;
    %最小二乗法
    for m=1:j
        if (stress(m) > (0.1*stress_y(i))) && (stress(m) < (1.7*stress_y(i)))
            Sxy = Sxy+(strain(m,1)-avx)*(stress(m)-avy);
            Sx2 = Sx2+(strain(m,1)-avx)^2;
            Sxy1 = Sxy1+(strain(m,2)-avx2)*(strain(m,1)-avx);
            Sxy2 = Sxy2+(strain(m,2)-avx2)^2;
        end
    end 
    Yongs(i) = Sxy/Sx2*10^6;
    b = avy - Yongs * avx;
    Poissons(i) = Sxy2/Sxy1;
    
    plot(strain(:,1),stress);       %図の追加
end

writematrix(data{1,2},'template_mt.xlsx','sheet',1,'range','A15')
writematrix(data{2,2},'template_mt.xlsx','sheet',1,'range','G15')
writematrix(data{3,2},'template_mt.xlsx','sheet',1,'range','M15')
writematrix(stress_y','template_mt.xlsx','sheet',1,'range','J6')
writematrix(stressmax','template_mt.xlsx','sheet',1,'range','K6')
writematrix(Yongs','template_mt.xlsx','sheet',1,'range','L6')
writematrix(Poissons','template_mt.xlsx','sheet',1,'range','M6')

%%%Figure
xlim(axes1,[0 70000]);
ylim(axes1,[0 500]);
box(axes1,'on');
hold(axes1,'off');
legend1 = legend(axes1,'show');
legend('No.1','No.2','No.3','location','southeast');

```

