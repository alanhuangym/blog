---
title: Duality of Linear Programming
date: 2017/3/15
tags: 
- linear_programming
categories:
- CUHK_Notes
---



Resource:

**S´ebastien Lahaie, How to take the Dual of a Linear Program http://www.cs.columbia.edu/coms6998-3/lpprimer.pdf**

翻译：

Alan Huang



# 如何求线性规划的二元性

1. 线性规划是最优解问题的一个公式化结果：通过设立目标函数和约束函数，求取目标方程的最大值或最小值。因目标函数是线性的，所以称之为线性规划。

   通常，线性规划的一般形式是这样的：

   max
   x1≥0, x2≤0, x3
   v1x1 + v2x2 + v3x3 (1)
   such that 

   a1x1 + x2 + x3 ≤ b1 (2)
   x1 + a2x2 = b2 (3)
   a3x3 ≥ b3 (4)

   (1)是目标函数，(2)(3)(4)是约束函数，目标是求取(1)的最大值

   其中，x1,x2,x3是变量，而其他的参数则为常量（v1,v2,v3等）

2. 这里我们称以上所述的线性规划为原始线性规划，二元性则是通过数学过程，从原始线性规划得到二次线性规划。

3. 首先我们要重写原始线性规划的目标方程，使之成为一个标准形式。

   为了简便计算，我们尽量将目标方程改写为求##最小值##的形式

   从求最大值到求最小值也很简单，只需要将目标方程乘以-1即可

   所以我们可以获得改写后的目标方程：

   min
   x1≥0, x2≤0, x3
   −v1x1 − v2x2 − v3x3

4. 第二步，是将约束函数全部改写为小于或小于等于或等于零的形式。即将(2)(3)(4)改写为：

   a1x1 + x2 + x3 − b1 ≤ 0 (5)
   x1 + a2x2 − b2 = 0 (6)
   −a3x3 + b3 ≤ 0 (7)

5. 第三步，为每一个不等式约束函数设定一个非负的二元性参数，为每一个等式约束函数设定一个二元性参数（无限制负或非负）。如上所述，我们分别为(5)(7)设定二元性参数λ1 ≥ 0 和 λ3 ≥ 0，为(6)设定 λ2。

6. 第四步，将第三步设置的二元性参数与约束函数符号左边的部分相乘并将它们加入到目标函数之中，并在目标函数前面加上求最大值，即如下所示：

   max min  −v1x1 − v2x2 − v3x3

   +λ1 (a1x1 + x2 + x3 − b1) (8)

   +λ2 (x1 + a2x2 − b2) (9)

   +λ3 (−a3x3 + b3) (10)

7. 第五步，将目标函数的括号拆开，并进行整理。将包含原本变量（x1,x2,x3）的式子下放到约束函数之中，其余的项放入目标函数之中。即:

   max min  −b1λ1 − b2λ2 − b3λ3

   +x1(a1λ1 + λ2 − v1) (11)

   +x2(λ1 + a2λ2 − v2) (12)

   +x3(λ1 − a3λ3 − v3) (13)

8. 第六步，将约束函数表示为等式或不等式的形式。

   如果变量后面的参数是大于或等于零的，则去掉原始变量获得不等式该参数大于或等于零，

   如果变量后面的参数是小于或等于零的，则去掉原始变量获得不等式该参数小于或等于零，

   如果变量后面的参数是等于零的，则去掉原始变量获得不等式该参数等于零。

   同时，目标函数去除Min。

   max −b1λ1 − b2λ2 − b3λ3
   a1λ1 + λ2 − v1 ≥ 0 (14)
   λ1 + a2λ2 − v2 ≤ 0 (15)
   λ1 − a3λ3 − v3 = 0 (16)

9. 第七步，如果一开始将求最小值变成了最大值（目标函数乘以-1），则将目标函数再次反转，否则，不做任何步骤（可以整理约束函数），则获得最终结果。

   min b1λ1 + b2λ2 + b3λ3
   a1λ1 + λ2 ≥ v1 (17)
   λ1 + a2λ2 ≤ v2 (18)
   λ1 − a3λ3 = v3 (19)

   至此，已经获得了原始线性规划的二元线性规划。

   ​