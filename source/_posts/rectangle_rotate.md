title:   矩形绕原点旋转
date: 2020-3-2
tags: 
categories: math

description: 羞耻呀，貌似是初中知识
---

# 矩形绕原点顺时针旋转90 度

给定矩形左上角和右下角坐标为$(x_0, y_0, x_1, y_1)$, 绕原点逆时针90 度后为$(y_0, - x_1, y1, - x_0)$。这有一个图形旋转的公式在这里

![rotate_matrix](/images/rotate_matrix.png)

这里面的Θ指的是旋转角度，而$x_0, y_0$指的是旋转前坐标，$x_1,y_1$是旋转后坐标。顺时针旋转 90度，Θ = -90°，因此矩阵运算后即可得到上述结果。

此外，画图分析原点旋转后的图形可采取先将 x轴，y 轴互换，同时此时的 y 轴需要变向，比如互换后的 y 轴是向右的，变向后 y 轴向左，此时，将整个图形顺时针旋转 90°即可得到旋转后的图形。读出对应数值即可。

![coordinate_transformation](/images/coordinate_transformation.jpeg)

正方形绕中心旋转，坐标不变。