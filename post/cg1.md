新年新气象，在1.1我开了个新坑，试图用最基本的2d绘图api完成一个web版的软渲染引擎，以此来践行学习的图形学知识。

项目名称为Mini3D，[地址戳这里](https://github.com/Easonzero/Mini3D)

![demo](https://github.com/Easonzero/Mini3D/raw/master/img/demo_3.png)

在这个项目开发的过程中，我也会同步跟进技术博客，记录所得。

# 从点、线、面说起

计算机图形学是一门应用广阔的计算机技术，科学与艺术的结合在这个领域上体现的淋漓尽致。但无论这个领域有多瑰丽，说到底终究是在一张四四方方的屏幕上做文章。

对于2d图形来说，api是极为简单也极易理解的，无非是在一个二维数组里填充rgb数据而已，点、线、面都是通过这样的方式呈现出来的。

对于3d图形来说，0.0其实也不难，相当于3维世界在2维屏幕上的投影，或者说3维坐标与屏幕上的2维坐标建立映射关系。

那么这种映射关系如何表达呢？

# 变换

我们假设3维世界中有这么两个要素，一个物体、一个观察物体的人。

想一想物体从3维世界中变换到2维屏幕上都需要什么变换？

标准答案是：物体空间->世界空间->视体空间->投影变换->屏幕

大白话就是：

* 把一个物体放在3维坐标系里(物体空间->世界空间)

* 旋转偏移观察物体的人以及所有物体，使人站在坐标系原点(世界空间->视体空间)

* 将物体投影在人的视网膜上，其实也就是屏幕上(视体空间->投影变换)

* 将图像进行偏移放缩使得坐标原点落在屏幕的中心(投影变换->屏幕)

## 物体空间->世界空间

以最简单的情形考虑，一个正方体，从自己的物体空间到世界空间只需要乘上旋转、放缩及偏移的变换矩阵即可。  
下面仅列出绕x，y，z轴旋转的矩阵。有的同学可能会有疑问，为什么要用4维矩阵？其实第4维并不具有实际意义，很多时候都与计算结果无关，但是有些时候，出于计算需要，比如偏移，则需要用到4维。

```js
//绕x轴
[1,             0,              0, 0],
[0, Math.cos(rad), -Math.sin(rad), 0],
[0, Math.sin(rad),  Math.cos(rad), 0],
[0,             0,              0, 1]
//绕y轴
[ Math.cos(rad), 0, Math.sin(rad), 0],
[             0, 1,             0, 0],
[-Math.sin(rad), 0, Math.cos(rad), 0],
[             0, 0,             0, 1]
//绕z轴
[Math.cos(rad), -Math.sin(rad), 0, 0],
[Math.sin(rad),  Math.cos(rad), 0, 0],
[            0,              0, 1, 0],
[            0,              0, 0, 1]
```

## 世界空间->视体空间

主要是对物体进行观察坐标系偏移到坐标轴所需的变换，即旋转和偏移即可。  
下面仅列出旋转部分的矩阵。u，v，w相当于观察坐标系的x，y，z轴，其中w一般是观察方向的反方向，u则通过任意向上向量与w叉乘求得，v则由w和u叉乘求得。

```js
//旋转矩阵
[u.x,u.y,u.z,0],
[v.x,v.y,v.z,0],
[w.x,w.y,w.z,0],
[  0,  0,  0,1]
```

## 视体空间->投影变换

投影有很多种，最直接的就是直接忽略z值，也就是所谓的正投影。

另外一种较常用的投影是透视投影，原理是使物体x，y坐标根据物体距离屏幕的远近进行放缩，其推导过程是由几个平移、放缩矩阵合并而成，具体过程不做详述，在此仅列出最后结果。

```js
//透视矩阵
[2*n/(t-b),        0,(l+r)/(l-r),          0],
[        0,2*n/(r-l),(t+b)/(t-b),          0],
[        0,        0,(f+n)/(n-f),f*n*2/(f-n)],
[        0,        0,          1,          0]
```

## 投影变换->屏幕

最简单，只是一步2维的偏移放缩变换而已。

```js
[ width/2,        0,0, (width-1)/2],
[       0,-height/2,0,(height-1)/2],
[       0,        0,1,           0],
[       0,        0,0,           1]
```