---
layout: post
title: "Perceptron"
subtitle: "感知机"
author: "Joy"
header-img: "img/post-bg-infinity.jpg"
header-mask: 0.2
tags:
  - 机器学习
---

## Perceptron

- 适用于线性可分的二分类问题

- 梯度下降法对损失函数极小化

  

## 学习策略

- 数据集	$\{(x_1,y_1),(x_2,y_2)...(x_N,y_N)\}$，其中$y_i$为label
- 求出超平面$w*x +b=0$ 将正样本和负样本分割到该超平面的两侧 



## 如何求得超平面？

### 损失函数

$L(w, b)=-\sum \limits_{x_i \in M } y_i(w*x_i+b)$  ,M为误分类点的集合

### Stochastic gradient descent(随机梯度下降)

#### 损失函数的梯度

$\partial L/ \partial w=-\sum\limits_{x_i \in M } y_ix_i$

$\partial L/ \partial b=-\sum\limits_{x_i \in M }x_i$

#### 感知机算法

- 选取任意$w_0,b_0$

- 训练集数据$(x_i,y_i)$

- if $y_i(w*x_i+b)<=0$ 误分类数据

  ​		$w=w+ny_ix_i$

  ​		$b=b+ny_i$

  $n \in(0,1]$,称为学习率



## 算法收敛性

待填写



## 解的不唯一性

感知机算法的解既依赖于初值的选择，也依赖于误分类点的选择顺序



## Python实现

```python
import numpy as np
import matplotlib.pyplot as plt
from mpl_toolkits.mplot3d import Axes3D

#数据集
training_set = [[(3,4,2),1],[(3,3,5),1],[(4,3,8),1],[(1, 2,1), 1], [(2, 3,4), 1],[(0,1.5,3),-1], [(3, 1,-1), -1], [(4, 2,2), -1],[(3,2,10),-1]]

#数据获取
data,label=[],[]

for i in training_set:
    data.append(i[0])
    label.append(i[1])

dataMat=np.mat(data)
#获得矩阵大小
m, n = np.shape(dataMat)
#参数初始化
w = np.zeros((1, np.shape(dataMat)[1]))
b=0
n=0.01#学习率

#data为1xn的矩阵，w为1xn的矩阵
#算法核心部分
def preceptron(dataset,labels,times):
    global w,b,n
    a=0
    print('start to train')
    #梯度下降
    for k in range(times):
        flag=False
        for i in range(m):
            xi=dataset[i]
            yi=labels[i]
            #误分类点
            if yi*(w*xi.T+b)[0,0]<=0:
                #更新w,b
                w=w+n*yi*xi
                b=b+n*yi
                flag=True
                
                a=a+1
                print('第 %d 次迭代'%a)
                print('w=',w,'b=',b)
        #如果没有误分类点，结束循环
        if not flag:
            break;   

    return w,b

#可视化
def show(w,b):
    x,x_,y,y_,z,z_=[],[],[],[],[],[]
    for p in training_set:
        if p[1] > 0:
            x.append(p[0][0])  # 存放yi=1的点的x1坐标
            y.append(p[0][1])  # 存放yi=1的点的x2坐标
            z.append(p[0][2])
        else:
            x_.append(p[0][0])  # 存放yi=-1的点的x1坐标
            y_.append(p[0][1])  # 存放yi=-1的点的x2坐标
            z_.append(p[0][2])
        
    fig = plt.figure()
    ax = Axes3D(fig)
    ax.set_alpha(.4)
    X = np.arange(-4, 4, 0.25)
    Y = np.arange(-4, 4, 0.25)
    X, Y = np.meshgrid(X, Y)
    Z=  -(b + w[0,0] * X+w[0,1]*Y) / w[0,2]
    ax.plot_surface(X,Y,Z,rstride = 1, cstride = 1,alpha=0.5)
    
    ax.scatter(x, y, z,c='r',marker='o', label='正样本')
    ax.scatter(x_, y_, z_,c='b',marker='x', label='负样本')
    ax.set_zlabel('Z', fontdict={'size': 15, 'color': 'red'})
    ax.set_ylabel('Y', fontdict={'size': 15, 'color': 'red'})
    ax.set_xlabel('X', fontdict={'size': 15, 'color': 'red'})
    plt.show()


if __name__ == '__main__':
    preceptron(dataMat,label,200)
    show(w,b)
```
