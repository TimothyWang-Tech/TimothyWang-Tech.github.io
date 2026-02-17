---
title: 高层次芯片设计之Software Basic
publishDate: 2026-02-18 08:00:00
description: '整理了高层次芯片设计课程Software Basic部分的知识要点'
tags:
  - hlcd
  - 课程笔记
heroImage: { src: './hlcdfig.png', color: '#B4C6DA' }
language: '中文'
---
写在前面：25-26学年高层次芯片课程主要分为：软硬件基础、高层次综合算法、SystemC&Bluespec语言三个部分，该学年也是第一次有期末考试的学年，课程知识密度极大，总任务量中等偏上；由于梁老师上课讲课速度还是偏快的，而博客上的这三篇笔记完全是笔者上课所记的notion直接上传过来的，有时为了跟上梁老师的思路笔记记的非常简略，所以该笔记仅供参考；每年讲课内容和考察内容应该不一样，考试前梁老师会说哪些不考。
# 课程lab
总共有4次lab，最后一次有大作业性质，这里笔者找到github上lab2的文档供参考[https://github.com/pku-liang/hlcd-hls-lab-handout](https://github.com/pku-liang/hlcd-hls-lab-handout)。

# Hardware Basic
该部分请参考小数电课程(是的，梁老师一节课讲完了小数电课程的所有知识点x)，期末是有这部分的考察的，考了逻辑运算和状态机。

# Software Basic
## Block

如何划分block的算法（期末考了）

## SSA形式

 0.  SSA: 每个变量只被赋值一次但有不同版本；phi function: 同一变量不同版本融合函数

1. 如何插入phi节点
    
    1.1 dominator 
    
    X dominate Y: 到达Y必须经过X; 
    
    X strictly dominate Y: X支配Y并且Y≠X;
    
    X immediately dominate Y: X严格支配Y并且X不严格支配任何其他 严格支配X的节点;
    
    dominance frontier (DF): DF(X)=Y：X支配Y的前驱，但是X不支配Y;
    
    1.2 在DF(block_variable)的位置插入phi节点并且不断迭代
    

![image.png](%E9%AB%98%E5%B1%82%E6%AC%A1%E8%8A%AF%E7%89%87%E8%AE%BE%E8%AE%A1%E4%B9%8BSoftware%20Basic/image.png)

1. 优化
    
    2.1 Common Subexpression Elimination
    
    2.2 Copy Propagation
    
    2.3 Code Motoin
    

![image.png](%E9%AB%98%E5%B1%82%E6%AC%A1%E8%8A%AF%E7%89%87%E8%AE%BE%E8%AE%A1%E4%B9%8BSoftware%20Basic/image%201.png)

![image.png](%E9%AB%98%E5%B1%82%E6%AC%A1%E8%8A%AF%E7%89%87%E8%AE%BE%E8%AE%A1%E4%B9%8BSoftware%20Basic/image%202.png)

![image.png](%E9%AB%98%E5%B1%82%E6%AC%A1%E8%8A%AF%E7%89%87%E8%AE%BE%E8%AE%A1%E4%B9%8BSoftware%20Basic/image%203.png)

## Software pipeline

![image.png](%E9%AB%98%E5%B1%82%E6%AC%A1%E8%8A%AF%E7%89%87%E8%AE%BE%E8%AE%A1%E4%B9%8BSoftware%20Basic/image%204.png)

![image.png](%E9%AB%98%E5%B1%82%E6%AC%A1%E8%8A%AF%E7%89%87%E8%AE%BE%E8%AE%A1%E4%B9%8BSoftware%20Basic/image%205.png)

1. 对II的求解
    1. II的最小值：一次循环n个操作，硬件资源有R个，m个循环，则(II*(m-1)+s)*R≥m*n；得II≥(n/R)
2. 操作之间依赖性
    
    3.1 $<\delta, d>$ 表示
    
    <2,1>表示这个操作需要在-2 iteration结束后1个clk后才可以开始
    
    ![image.png](%E9%AB%98%E5%B1%82%E6%AC%A1%E8%8A%AF%E7%89%87%E8%AE%BE%E8%AE%A1%E4%B9%8BSoftware%20Basic/image%206.png)
    
    3.2 $S(n2)≥S(n1)+d-\delta*II$
    
    无环图的不等式(左图)  有环图的不等式(右图)
    

![image.png](%E9%AB%98%E5%B1%82%E6%AC%A1%E8%8A%AF%E7%89%87%E8%AE%BE%E8%AE%A1%E4%B9%8BSoftware%20Basic/image%207.png)

![image.png](%E9%AB%98%E5%B1%82%E6%AC%A1%E8%8A%AF%E7%89%87%E8%AE%BE%E8%AE%A1%E4%B9%8BSoftware%20Basic/image%208.png)

无环图的每个节点会有最小约束关系，环图内每个节点都有最小最大约束关系

3.3 复杂图算法

1.先写出II和每个节点的约束关系

2.找到所有强连通分量，变成无环图

3.从一个强连通分量开始，从其中的一个单元开始，写出最小最大距离，从最小开始枚举，是否出现资源冲突，若冲突则增大，最大都不可以则增大上一级枚举的单元

4.如果所有单元都增大到最大，则增大II

![image.png](%E9%AB%98%E5%B1%82%E6%AC%A1%E8%8A%AF%E7%89%87%E8%AE%BE%E8%AE%A1%E4%B9%8BSoftware%20Basic/image%209.png)