---
layout:     post
title:      "二维凸包检测"
subtitle:   "Graham扫描法与Jarvis步进法"
category : basictheory
date:       2017-06-22
author:     "Max"
header-img: "img/post-bg-algorithms.jpg"
catalog:    true
tags:
    - algorithms
---

## 1. 定义


在一个实数向量空间 V 中，对于给定集合 X，所有包含X的凸集的交集 S 被称为 X 的**凸包**（**Convex Hull**）。

## 2. 算法

### 2.1 蛮力法

##### 时间复杂度 ：O(n³）

##### 思路


	1. 将点集里面的所有点两两配对，组成 n(n-1)/2 条直线。

	2. 对于每条直线，再检查剩余的 (n-2) 个点，若都在直线的同一侧，则这两个点是凸包上的点


给定 $P_1(x1,y1)，P_2(x2,y2)，P_3(x3,y3)$，依据如下方法判断点 $P_3$ 是在直线 $P_1P_2$ 的位置关系：

result>0，$P_3$ 在 $P_1P_2$ 的左侧；result<0，$P_3$ 在 $P_1P_2$ 的右侧。另外 result 绝对值越大，$P_3$ 距 $P_1P_2$ 越远。

### 2.2 分治法

在蛮力思维的基础上，可以使用“分治”思维进行优化。

##### 时间复杂度：O(nlogn)

##### 思路

1. 把所有的点都放在二维坐标系里面。那么横坐标最小和最大的两个点 $P_1$ 和 $P_n$ 一定是凸包上的点。直线 $P_1P_n$ 把点集分成了两部分，即 X 轴上面和下面两部分，分别叫做上包和下包。
2. 对上包：求距离直线 $P_1P_n$ 最远的点，记为点 $P_{max}$ 。
3. 作直线 $P_1P_{max}$ 、 $P_nP_{max}$ 左侧的点当成是上包，把直线 $P_nP_{max}$ 右侧的点也当成是上包。
4. 重复步骤 2、3，直到上包为空集。
5. 对下包作类似操作。

### 2.3 Graham扫描法

##### 时间复杂度：O(nlogn)

##### 思路


1. 取y值最小的点（y值相同时，取x值最小的点），记为 $P_0$
2. 计算各点相对于 $P_0$x轴 的幅角 α ，按从小到大的顺序对各个点排序。当 α 相同时，距离 $P_0$ 比较近的排在前面。记为 $P_2 , P_3 , . . . , P_n$
3. 将 $P_0、P_1、P_2$ 入栈
4. 以栈顶两个点做直线L，判断下一个点在直线L的左侧或右侧。若在右侧，则将栈顶的点出栈，重新进入步骤4
5. 将下一个点入栈
6. 重复步骤4-5，直到遍历完成所有点。


（注：通常称在凸包顶点处的点为**极点**，是否保留除极点外的边点在排序处理上略有差别。当只需要保留极点时，α 相同的点距离 $P_0$ 远的排在前面；当需要边界上的所有点时， α 相同的点距离 $P_0$ 近的排在前面，然后对结束边反向重排，即距离 $P_0$ 远的排在前面。 ）

##### 例题

https://leetcode.com/problems/erect-the-fence/#/description

```golang
/**
 * Definition for a point.
 * type Point struct {
 *     X int
 *     Y int
 * }
 */
func findBottom(points []Point) Point {
    bottom := Point {
        X : math.MaxInt32,
        Y : math.MaxInt32,
    }
    for _, p := range points {
        if p.Y < bottom.Y || (p.Y == bottom.Y && p.X < bottom.X) {
            bottom = p
        }
    }
    return bottom
}

func sortByPolar(points []Point, bottom Point) {
    angle := func(p Point) float64 {
        v2 := Point{
            X : p.X - bottom.X,
            Y : p.Y - bottom.Y,
        }
        if v2.X == 0 && v2.Y == 0 {
            return float64(1)
        }
        sig := float64(1)
        if v2.X < 0 {
            sig = float64(-1)
        }
        return sig*float64(v2.X*v2.X)/float64(v2.X*v2.X + v2.Y*v2.Y)
    }
    dist := func(p Point) int {
        v1 := Point{
            X : p.X - bottom.X,
            Y : p.Y - bottom.Y,
        }
        return v1.X*v1.X + v1.Y*v1.Y
    }
    PointLess := func(i, j int) bool {
        ang1, ang2 := angle(points[i]), angle(points[j])
        if ang1 == ang2 {
            return dist(points[i]) < dist(points[j])
        }
        return ang1 > ang2
    }
    sort.Slice(points, PointLess)

    j, e := len(points)-1, len(points)-1
    for angle(points[j]) == angle(points[j-1]) {
        j--
    }
    for j < e {
        points[j], points[e] = points[e], points[j]
        j++
        e--
    }
    return
}

func isRight(p1, p2, p3 Point) bool {
    return p1.X*p2.Y + p3.X*p1.Y + p2.X*p3.Y - p3.X*p2.Y - p2.X*p1.Y - p1.X*p3.Y < 0
}

func outerTrees(points []Point) []Point {
    if len(points) < 4 {
        return points
    }

    bottom := findBottom(points)
    sortByPolar(points, bottom)

    result := make([]Point, 0)
    result = append(result, points[0:3]...)
    for i:=3; i<len(points); i++ {
        for isRight(result[len(result)-2], result[len(result)-1], points[i]) {
            result = result[:len(result)-1]
        }
        result = append(result, points[i])
    }
    return result
}
```

### 2.4 Jarvis步进法

##### 时间复杂度：O(nH)

（其中 n 是点的总个数，H 是凸包上的点的个数。）

##### 思路


1. 取x值最小的点（x值相同时，取y值最小的点） ，记为 $P_0$ ，加入队列
2. 取相对于 $P_0$ 逆时针方向最靠外的点（以 $P_0、P_1、P_2$ 点为例，若向量 $\vec{P_0P_1}$ 、$\vec{P_0P_2}$ 的[叉积](http://mathworld.wolfram.com/CrossProduct.html)为负，则说明向量 $\vec{P_0P_1}$ 顺时针旋转至 $\vec{P_0P_2}$ 方向，即相对于 $P_0$ ，点 $P_2$ 比 $P_1$ 在逆时针方向上更加靠外；若叉积为0，则 以 $P_0、P_1、P_2$ 共线），记为 $P_1$ ，加入队列（若多点共线，取距 $P_0$ 最远的点）
3. 重复步骤2，直到队列最后一个点是 $P_0$


（注：同样，当只需要保留极点时，共线的点只需要取最远的点加入队列；当需要边界上的所有点时， 将共线的点全部加入队列，但保证距离最远的点在队列末尾 。 ）

##### 例题

同一题按Jarvis步进法编码如下：

```golang
func findBottom(points []Point) Point {
    for i := len(points)-1; i > 0; i-- {
        if points[i].X < points[i-1].X || (points[i].X == points[i-1].X && points[i].Y < points[i-1].Y) {
            points[i-1], points[i] = points[i], points[i-1]
        }
    }
    return points[0]
}

func CrossProduct(p1, p2, p3 Point) int {
    return (p2.X - p1.X)*(p3.Y - p1.Y) - (p2.Y - p1.Y)*(p3.X - p1.X)
}

func dist(p1, p2 Point) int {
    v := Point {
        X : p2.X - p1.X,
        Y : p2.Y - p1.Y,
    }
    return v.X*v.X + v.Y*v.Y
}

func outerTrees(points []Point) []Point {
    if len(points) < 4 {
        return points
    }

    bottom := findBottom(points)
    result := make([]Point, 0)
    result = append(result, bottom)

    lenR := len(result)
    for result[lenR-1] != bottom  || lenR == 1 {
        minPoints := make([]Point, 0)
        for _, p := range points {
            if p == result[lenR-1] {
                continue
            }
            if len(minPoints) == 0 {
                minPoints =  append(minPoints, p)
                continue
            }
            cp := CrossProduct(result[lenR-1], minPoints[len(minPoints)-1], p)
            if cp < 0 {
                minPoints = []Point{p}
            } else if cp == 0 {
                minPoints = append(minPoints, p)
                l := len(minPoints)
                if dist(result[lenR-1], minPoints[l-2]) > dist(result[lenR-1], minPoints[l-1]) {
                    minPoints[l-2], minPoints[l-1] = minPoints[l-1], minPoints[l-2]
                }
            }
        }
        result = append(result, minPoints...)
        lenR = len(result)
    }
    return result[:lenR-1]
}
```
