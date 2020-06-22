---
title: 关于Warshall、Roy对寻找传递闭包方法的不同表达的探讨
tags: [离散数学]
date: 2015-11-17 19:23:00
categories: 专业
---

一、引言
	在计算机科学中，Floyd-Warshell-Roy算法是用于在有向图或负权图中寻找最短路径的一种算法。运行一次能够找到所有两个顶点间的最短路径，不过并不输出所有路径。该算法同时也可以被用于寻找关系R的传递闭包。
	Floyd-Warshell算法是一个动态规划的例子，在1962年为Robert Floyd发表。然而，在1959年，相同的算法已经被Bernard Roy发表。同年，Stephen Warshell发表了找到传递闭包的这个算法。三人被认定为独立发现这个算法。
	该算法的主要作用是将常规算法的时间复杂度由Θ(n^4)降低到了Θ(n^3).
	本文中，我们出于寻找Roy、Warshall的算法被认定为独立发现的的缘由对两人的算法进行分析。

二、分析与讨论
1.	Warshall 算法
```
procedure Warshall (MR : n × n zero–one matrix)
W : = MR
for k : = 1 to n
   for i : = 1 to n
      for j : = 1 to n
         w_ij : = w_ij ∨ (w_ik)∧ w_kj )
return W{W = [w_ij] is MR∗}
```
![essay_](http://images.cnblogs.com/cnblogs_com/sean10/750413/o_essay_0.png)

![essay_](http://images.cnblogs.com/cnblogs_com/sean10/750413/o_essay_1.png)

![essay](http://images.cnblogs.com/cnblogs_com/sean10/750413/o_essay_2.png)

三、结论
	虽然Bernard Roy 提出该算法在Robert Floyd和Stephen Warshall之前，但他论文的主体依旧是以对图的定义为主，而Warshall 和Floyd两人同年发表了成文的简单算法。所以，可能这是一部分影响单独发表的原因。

四、展望
	根据[1]，我们知道了Roy的传递闭包计算方法是采用了Kleene已经提出了的深层技术，而Warshall和Floyd则是采用了第三个参数。不过基于时间以及水平原因，并没有能够找到这两者之间所说的深层技术，也并没能确定是否Warshall和Floyd所采用的关键技术是在于中间点k。因而，可以沿这个方向继续接下去进行查询发掘。

五、参考文献
[1]. Jeff Erickson, Kleene-Roy-Floyd-Warshall. [https://courses.engr.illinois.edu/cs498dl1/sp2015/notes/22-apsp.pdf]
[2]. Warshall, Stephen (January 1962). "A theorem on Boolean matrices". Journal of the ACM 9 (1): 11–12\. doi:10.1145/321105.321107 .
[3]. Roy, B. "Transitivité et connexité." C. R. Acad. Sci. Paris 249, 216-218, 1959\. [http://gallica.bnf.fr/ark:/12148/bpt6k3201c]
[4]. Bouyssou, D., Jacquet-Lagrèze, E., Perny, P., Slowiński, R., Vanderpooten, D., Vincke, P,《Aiding Decisions with Multiple Criteria: Essays in Honor of Bernard Roy》,24,2001\. 
[5]. Wikipedia. [https://en.wikipedia.org/wiki/Floyd%E2%80%93Warshall_algorithm]
[6]. Purdom, Paul, Jr. “A Transitive Closure Algorithm.” Bit 10, no. 1 (March 1970): 76–94\. doi:10.1007/BF01940892.