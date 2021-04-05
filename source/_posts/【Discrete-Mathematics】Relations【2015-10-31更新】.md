---
title: 【Discrete Mathematics】Relations【2015.10.31更新】
tags: [离散数学,关系]
date: 2015-10-29 21:22:00
categories: 专业
---

【Discrete Mathematics】Relations

这是离散数学课的个人阶段性总结，如有纰漏，敬请指出，一起学习~太过简单，也请见谅Orz

使用教材为《离散数学及其应用》（第7版.英文版）

现在主要拉定义，加上个人理解= =

__Chapter 9.1 Relations and their properties__

>__Definition1.__
Let A and B be sets. A binary relation from A to B is a subset of A × B._

二元关系就是序偶的集合R，记号为aRb，表示(a,b)\inR.

函数是关系的一种。对应于集合A的每一个元素，恰好有且仅有一个集合B中的元素与之对应。

>__Definition2.__
A relation on a set A is relation from A to A.

>__Definition3.__
A relation R on a set A is called _reflexive_ if (a, a)\in R for every element a \in A. 

Remark.我们看到\forall a((a,a)\in R),那么集合上的关系是自反的。这里的论域是A中所有元素的集合。

>__Definition4.__
A relation R on a set A is called _symmetric_ if(b,a) \in R whenever (a,b)\in R ,for all a,b \in A.
A relation R on a set A such that for all a,b \in A, if (a,b) \in R and (b,a) \i R, then a = b is called antisymmetric.

Remark.使用量词，可以看到如果\forall a \forall b((a,b) \in R -> (b,a) \in R),则A上的关系R是对称的。类似的，如果\forall a \forall b(((a,b) \in R ^ (b,a) \in R)->(a=b),则A上的关系R是反对称的。

就是说，关系R是对称的，当且仅当如果a与b相关则b与a就相关，关系R是反对称的，当且仅当不存在由不同元素a和b构成的序偶使得a与b相关并且b与a也相关。对称与反对称的概念不是对立的，因为一个关系可以同时有这两种性质或者两种性质都没有。一个关系如果包含了某些形如(a,b)的对，其中a != b，这个关系就不可能同时是对称和反对称的。

>__Definition5.__
A relation R on a set A is called _transitive_ if whenever (a,b) \in R and (b,c) \in R,then (a,c) \in R,for all a,b,c \in A.

Remark.使用量词，我们可以看到在集合A上的关系R是传递的，如果\forall a \forall b \forall c(((a,b)\in R ^(b,c)\in R)-> (a,c)\in R).

>__Definition6.__
Let R be a relation from a set A to a set B and S a relation from B to a set C.The _composite_ of R and S is the relation consisting of ordered pairs(a,c),where a\in A,c\in C,and for which there exists an element b \in B such that (a,b) \in R and (b,c) \in S.We denote the composite of R and S by S\cdot R.

>__Definition7.__
Let R be a relation on the set A.The powers R^n,n= 1,2,3...,are defined recursively by R^1=R and R^(n+1) = R^n\cdot R.

>__Theorem1.__
The relation R on a set A is transitive if and only if R^n \subseteq R for n = 1,2,3...

>__Supplement__
A relation R on the set A is _irreflexive_ if for every a \in A,(a,a) \nin R.That is ,R is irreflexive if no element in A is related to itself.
A relation R is called _asymmetric_ if (a,b)\in R implies that (b,a) \nin R.
Let R be a relation from a set A to a set B.The __inverse relation__ from B to A,denoted by R^-1 ,is the set of ordered pairs {(b,a)|(a,b)\inR }.The __complementary relation__ \neg R is the set of ordered pairs{(a,b)|(a,b)\nin R}.

__Chapter 9.2 n-ary relations and their applications__

>__Definition1.__
Let A1,A2,...An be sets.An _n-ary relation_ on these sets is a subset of A1×A2×...×An.The sets A1,A2,...An are called the _domains_ of the relation ,and n is called its _degree_.

>__*Supplement__
Relations used to represent databases are also called __tables__, because these relations are often displayed as tables. Each column of the table corresponds to an attribute of the database. For instance, the same database of students is displayed in Table 1\. The attributes of this database are Student Name, ID Number, Major, and GPA.
A domain of an n-ary relation is called a __primary key__ when the value of the n-tuple from
this domain determines the n-tuple. That is, a domain is a primary key when no two n-tuples in the relation have the same value from this domain.
Records are often added to or deleted from databases. Because of this, the property that a domain is a primary key is time-dependent.Consequently, a primary key should be chosen that remains one whenever the database is changed. The current collection of n-tuples in a relation
is called the __extension__ of the relation. The more permanent part of a database, including the name and attributes of the database, is called its __intension__. When selecting a primary key, the goal should be to select a key that can serve as a primary key for all possible extensions of the database. To do this, it is necessary to examine the intension of the database to understand the set of possible n-tuples that can occur in an extension.

>Combinations of domains can also uniquely identify n-tuples in an n-ary relation. When the values of a set of domains determine an n-tuple in a relation, the Cartesian product of these domains is called a __composite key__.

>__Definition2.__
Let R be an n-ary relation and C a condition that elements in R may satisfy. Then the selection operator sC maps the n-ary relation R to the n-ary relation of all n-tuples from R that satisfy the condition C.

>__Definition3.__
The projection P(i1,i2,...,im) where i1 < i2 < · · · < im, maps the n-tuple (a1, a2, . . . , an) to the m-tuple (a(i1), a(i2), . . . , a(im)), where m ≤ n.

>__Definition4.__
Let R be a relation of degree m and S a relation of degree n. The join Jp(R, S), where p ≤ m and p ≤ n, is a relation of degree m + n − p that consists of all (m + n − p)-tuples (a1, a2, . . . , a(m−p), c1, c2, . . . , cp, b1, b2, . . . , b(n−p)), where the m-tuple (a1, a2, . . . , a(m−p), c1, c2, . . . , c(p)) belongs to R and the n-tuple (c1, c2, . . . , cp, b1, b2, . . . ,b(n−p)) belongs to S.

__Chapter 9.3 representing relations__
Representing relations using Matrices

A relation between finite sets can be represented using a zero–one matrix. Suppose that R is a relation from A = {a1, a2, . . . , am} to B = {b1, b2, . . . , bn}. (Here the elements of the sets A and B have been listed in a particular, but arbitrary, order. Furthermore, when A = B we use the same ordering for A and B.) The relation R can be represented by the matrix MR = [mij], where m(ij)={1 if(ai,bj)\in R,0 if(ai,bj)\nin R}.

![matrix](http://images.cnblogs.com/cnblogs_com/sean10/750413/o_figure2.png)

Representing relations using Digraphs

We have shown that a relation can be represented by listing all of its ordered pairs or by using a zero–one matrix. There is another important way of representing a relation using a pictorial representation. Each element of the set is represented by a point, and each ordered pair is represented using an arc with its direction indicated by an arrow. We use such pictorial representations when we think of relations on a finite set as __directed graphs__, or __digraphs.__

>__Definition1.
A directed graph, or digraph, consists of a set V of vertices (or nodes) together with a set E of ordered pairs of elements of V called edges (or arcs). The vertex a is called the initial vertex of the edge (a, b), and the vertex b is called the terminal vertex of this edge.

An edge of the form (a, a) is represented using an arc from the vertex a back to itself. Such an edge is called a loop.

__Chapter9.4 closures of relations__
The relation R = {(1, 1), (1, 2), (2, 1), (3, 2)} on the set A = {1, 2, 3} is not reflexive. How can we produce a reflexive relation containing R that is as small as possible? This can be done by adding (2, 2) and (3, 3) to R, because these are the only pairs of the form (a, a) that are not in R. Clearly, this new relation contains R. Furthermore, any reflexive relation that contains R must also contain (2, 2) and (3, 3). Because this relation contains R, is reflexive, and is contained within every reflexive relation that contains R, it is called the __reflexive closure__ of R.

Paths in Directed Graphs
>__Definition1.__
A path from a to b in the directed graph G is a sequence of edges (x0, x1), (x1, x2), (x2, x3), . . . , (xn−1, xn) in G, where n is a nonnegative integer, and x0 = a and xn = b, that is, a sequence of edges where the terminal vertex of an edge is the same as the initial vertex in the next edge in the path. This path is denoted by x0, x1, x2, . . . , xn−1, xn and has length n. We view the empty set of edges as a path of length zero from a to a. A path of length n ≥ 1 that begins and ends at the same vertex is called a circuit or cycle.

>__Theorem1.__
Let R be a relation on a set A. There is a path of length n, where n is a positive integer, from a to b if and only if (a, b) ∈ Rn.

>__Definition2.__
Let R be a relation on a set A. The connectivity relation R* consists of the pairs (a, b) such that there is a path of length at least one from a to b in R.

Because Rn consists of the pairs (a, b) such that there is a path of length n from a to b, it follows that R* is the union of all the sets Rn. In other words,
![formula1](http://images.cnblogs.com/cnblogs_com/sean10/750413/o_Capture.PNG)

>__Theorem2.__
The transitive closure of a relation R equals the connectivity relation R*.

>__Lemma1.__
Let A be a set with n elements, and let R be a relation on A. If there is a path of length at least one in R from a to b, then there is such a path with length not exceeding n. Moreover, when a != b, if there is a path of length at least one in R from a to b, then there is such a path with length not exceeding n − 1.
![path](http://images.cnblogs.com/cnblogs_com/sean10/750413/o_figure.path.PNG)

>__Theorem3.__
Let MR be the zero–one matrix of the relation R on a set with n elements. Then the zero–one matrix of the transitive closure R* is
![formula2](http://images.cnblogs.com/cnblogs_com/sean10/750413/o_formula2.png)

>__Lemma2.__
![](http://images.cnblogs.com/cnblogs_com/sean10/750413/o_Capture2.PNG)

![algorithm](http://images.cnblogs.com/cnblogs_com/sean10/750413/o_algorithm.png)

__Chapter9.5 Equivalence Relations__

>__Definition1.__
A relation on a set A is called an _equivalence relation_ if it is reflexive, symmetric, and transitive.

>__Definition2.__
Two elements a and b that are related by an equivalence relation are called _equivalent_. The notation a~b is often used to denote that a and b are equivalent elements with respect to a particular equivalence relation.

>__Definition3.__
Let R be an equivalence relation on a set A. The set of all elements that are related to an element a of A is called the equivalence class of a. The _equivalence class_ of a with respect to R is denoted by [a]R. When only one relation is under consideration, we can delete the subscript R and write [a] for this equivalence class.

In other words, if R is an equivalence relation on a set A, the equivalence class of the element a is [a]R = {s | (a, s) ∈ R}.

If b ∈ [a]R, then b is called a __representative__ of this equivalence class. Any element of a class can be used as a representative of this class. That is, there is nothing special about the particular element chosen as the representative of the class.

>__Theorem1.__
Let R be an equivalence relation on a set A. These statements for elements a and b of A are equivalent:(i) aRb (ii) [a] = [b]  (iii) [a]∩[b] != 0

>__Theorem2.__
Let R be an equivalence relation on a set S. Then the equivalence classes of R form a partition of S. Conversely, given a partition {Ai | i ∈ I} of the set S, there is an equivalence relation R that has the sets Ai, i ∈ I, as its equivalence classes.

>a partition of a set S is a collection of disjoint
nonempty subsets of S that have S as their union. In other words, the collection of subsets Ai, i ∈ I (where I is an index set) forms a partition of S if and only if 
Ai != 0 for i ∈ I, 
Ai ∩ Aj = 0 when i != j,
and ![c](http://images.cnblogs.com/cnblogs_com/sean10/750413/o_1.png)

![](http://images.cnblogs.com/cnblogs_com/sean10/750413/o_2.png)

>__Supplement__
A partition P1 is called a refinement of the partition P2 if every set in P1 is a subset of one of the sets in P2.

__Chapter9.6 Partial Orderings__

>__Definition1.__
A relation R on a set S is called a _partial ordering_ or _partial order_ if it is reflexive, antisymmetric, and transitive. A set S together with a partial ordering R is called a _partially ordered_ set, or poset, and is denoted by (S, R). Members of S are called elements of the poset.

>__Definition2.__
![4](http://images.cnblogs.com/cnblogs_com/sean10/750413/o_Capture3.PNG)

>__Definition3.__
![3](http://images.cnblogs.com/cnblogs_com/sean10/750413/o_Capture4.PNG)

>__Definition4.__
![2](http://images.cnblogs.com/cnblogs_com/sean10/750413/o_Capture1.1.PNG)

>__Theorem1.__
![1](http://images.cnblogs.com/cnblogs_com/sean10/750413/o_Capture1.2.PNG)

这一章就这样了，都是基础，摘录一下。找离散的符号太困难了，只能贴图了。