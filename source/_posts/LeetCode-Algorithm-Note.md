---
title: LeetCode_Algorithm_Note
date: 2018-01-23 22:50:32
updated:
tags: [算法, leetcode]
categories: [算法]
---


刷题注意事项

每道题需要总结的

思路
算法
核心代码
这个题得到的启示!!!重点是bug free的能力

<!--more-->

linked list理解

enter image description here

结果两个都是 1 2 3

node是存在main函数里的局部变量, 还是全局变量?
局部

node1 是一个指针, 在32位即中占有4个字节. 也可以理解为引用, 存的是一个内存的地址. 所以一个ListNode占8个字节, val占4个, next占四个.
可以理解为, 内存是个大数组, ref和pointer都是数组的下标, 是index

enter image description here

Reverse Nodes in k-Groups

Given a linked list, reverse the nodes of a linked list k at a time and return its modified list.

If the number of nodes is not a multiple of k then left-out nodes in the end should remain as it is.

You may not alter the values in the nodes, only nodes itself may be changed.

Only constant memory is allowed.

复杂问题的解决方案

复杂的, 不能一眼看到结果的, 要拆分.

拆分就是要步骤化, 先框架, 再细节.

复杂的问题通过一个for循环, 一个while循环, 或者一个123步怎么做, 变成一个更小的问题.

每个function需要明确

dummy node

java要new， c++不要new，c++如果new了要删除

HEAD = DUMMY 这句话总是需要么？

几乎

什么时候使用 DUMMY NODE?

链表结构发生变化的时候

DUMMY NODE 是否需要删除？

不需要

使用 DUMMY NODE 算面试官会说我耗费了额外空间么？

他没那么神精病， O(1)的额外空间不算额外空间，O(n)才算

DUMMY NODE 非用不可么？

不是, 但是用了代码比较简洁

DUMMY NODE 初始化的值重要么？

不重要, 只需要它的next的值



# LeetCode 小结
## 数组:
### leetcode 1

add two numbers to be the target，返回这2个数的位置

用hash map能实现O(n)甚至O(1)，但是不太清楚原理

### leetcode 561

直接用std::stable_sort排序，从小到大每2个取第一个，然后累加就是最大的最小数对和了

这个的时间复杂度应该是O(nlogn)，有没有

### leetcode 566

看了下Solution才知道有3种方法
1. 要求把能够重组的矩阵重组，按照从行到列的顺序重排，所以我一开始考虑把原本的矩阵数组输入到一个队列里，然后一个个pop，不过想了下，没有必要浪费空间，直接用迭代器，每到这行矩阵的末尾，就直接推进到下一行的begin()就好了。
2. 把原始的矩阵加到队列里去
3. 使用整除和取余符号,直接进行换行赋值

时空复杂度都是O(m*n).

python可以有numpy等等模块直接使用

不过在用python进行这个逻辑时， 始终提示列表越界……
``` python
class Solution:
    def matrixReshape(self, nums, r, c):
        """
        :type nums: List[List[int]]
        :type r: int
        :type c: int
        :rtype: List[List[int]]
        """
        if (len(nums[0])*len(nums)) != r*c:
            return nums
         #tmp = [[None] * c for i in range(r)]
        tmp = [[None for row in range(r)] for col in range(c)]

        i = 0
        j = 0
        leng_x = len(nums[0])
        x = 0
        y = 0
        for i in range(r):
            for j in range(c):
                tmp[i][j] = nums[x][y]
                y += 1
                if y == leng_x:
                    x += 1
                    y = 0


        return tmp

```

``` python
import numpy as np
class Solution:
    def matrixReshape(self, nums, r, c):
        """
        :type nums: List[List[int]]
        :type r: int
        :type c: int
        :rtype: List[List[int]]
        """
        if len(nums)*len(nums[0]) != r*c:
            return nums
        ans = [[None for row in range(r)] for col in range(c)]
        for i in range(r*c):
            # if i//c < r and i%c < c and ((i//len(nums[0]) < len(nums)) and ((i%len(nums[0])) < len(nums[0])):
            ans[i//c][i%c] = nums[i//len(nums[0])][i%len(nums[0])]
        return ans
```

这2份代码`tmp[i][j] = nums[x][y]` 究竟是哪里越界了呢？

### leetCode 485 最大连续1长度
O(n)时间复杂度，应该已经是最好了的吧，遍历一遍

### leetCode 717 自动机
比较简单，
状态0:刚得到的是0的string
状态1:收到1个1，接下来读取0\1
状态2:刚刚结束10/11的string

最后判断是状态0还是状态2就可以了

时间复杂度是O(n)

### leetCode 695 找最大的岛屿，第一反应就是DFS或者BFS
边界条件 越界or已被标记or为0

迭代和递归两种方法
时间复杂度O(r*c)

C++做迭代的时候，对allocator不怎么理解，stack<vector<int>>的时候一直报错……
最后用了pair

### leetCode 448
题目要求不使用额外空间。

想了想，第一个思路是不符合要求的，想到的是建个flag标记数组，然后输出没标记的。

没想到第二个方法，看了下答案，才意识到，可以把每个位置的值作为坐标来对该坐标位置进行*-1标记，既不影响值，又可以进行标记

### leetCode 283

遍历一遍，j作为非0位的位置，i作为遍历位置，向前移位，最后再从j的位置置0到end。

不过看到std::remove,可以找到value，直接向前补足，起到了上面第一个部分的功能，然后调用std::fill，结束

### leetCode 697

不理解题目……看了别人的代码以后才理解的题目是要的最长下标……不是说要最长子序列吗？



### leetCode 122 股票问题

完全没思路，看了下解题报告

1. 一个叫Peak Valley Approach，寻找极值点来靠近最优解
2. 直接后一个减去前一个，只要大于0，就加到ans就好了

O(n)时间复杂度，O(1)空间复杂度

### leetCode 167

只有输出上在leetCode 1基础上+1,并排序

不过说起来，理论上，输出的时候应该本就是从低到高的吧，为什么偶尔会输出逆序的呢？

似乎是输出找到的最后一个，那么应该需要返回换下序就好了

### leetCode 169

打算写个标记的，结果写完跑出来RE，才意识到输入是有负数的，

### leetCode 661

比较简单，就是遍历二维数组，O(n^2)的时间复杂度，O(1)的空间复杂度

### leetCode 628
打算直接用max_element找，结果漏考虑了负数也可以得到最大值了。
还是需要用到nth_element，来获取前K大和K小,
应该是O(nlog)复杂度，还不如直接排序了

### leetCode 268
要求是O(n)，用set.find()吧

忘记了可以用异或运算了

### leetCode 674

除了第一个，只要比前一个大，就可以+1

### leetCode 121

参考答案是不停的找最低点，然后之后的每个数都减一下他，获取最大利润。

完全没想到，我的思路是一开始先找到最低点/最高点，然后再找另一个，但是没想到方法。

### leetCode 442

第一反应就是插入set，使用find函数

时间复杂度应该是O(nlogn),空间复杂度O(n)

看了下discuss，有O(n)的方法，把nums[i]放到nums[nums[i]]的位置，最后不在自己位置的就是，还有一种更好的，每一个反转一下，有2个就会为正

### leetCode 238

要求常数空间复杂度和不用除法的O(n)的时间复杂度。

参考答案给出的方案是，因为每个数组中的结果都需要乘N-1次，而循环的次数也差不多是N-1次，每个循环都同时对2个结果进行累乘，而使用的参数from_begin和from_end则在不停累乘，得到除了self以外的乘积。

天哪，genius!

### leetCode 53

要去给出一个连续的能求出最大和的数组序列

依旧是动态规划，不过这个时间复杂度只有O(n)了

### leetCode 338

想到了应该是在临近的几位之间的位运算有状态变化，但没想到是位与


### leetCode 141

判断是否有环

我一开始想简单了，忘了只要中间有一个相交点就算环，而不是一定回到head

### leetCode 160

目前来看，看到了2种方法，一种就是先进行链表长度的计算，然后保证同步前进

第二种，就是两个进行交互遍历，A跑完了就指向B的表头，再开始跑，B跑完了就指向A的表头，总会有最大公约数的时候。

### leetCode 445

用了栈把输入的链表逆序输出了，然后再进行计算，头插

### leetCode 328

把奇数序号的节点移到最前面，要求O(1)空间复杂度和O(n)时间复杂度。

用了tail_odd,curr_odd,head_even,curr_even，搞定

### leetCode 24

要求每两个都交换节点。

记得有pre来连接每2个之间就可以了

### leetCode 19

这道题n直接说明定义域是从1开始就好了，还得自己测试


### leetCode 75

折半查找 255ms，
直接插入，冒泡，希尔都是3ms
可能因为只有0，1，2排序？

### leetCode 101

一开始想到的是层次遍历二叉树，然后用迭代器正反迭代匹配一下，有一个不匹配就返回false。

忘了写中间为空时填充一个nullptr，导致有些案例也返回了true。

又发现如果找不到就填充nullptr，会导致队列永远不会为空，内存溢出MLE的情况，试图在匹配过程中发现空指针，就erase掉，但是迭代过程中erase会导致后续的所有迭代器前移

并且，erase反向迭代器需要进行一下显示类型转换，并在erase时转换成正向迭代器(++it_r).base()。

因为我是new了一个ptr来替代nullptr来实现普遍性，考虑到智能指针可能可以省力，但是实际情况下，似乎容器中不能添加智能指针。虽然我用shared_ptr.get()传递进了容器，但是这样猜测应该是违背容器管理的。

一个容器全部被erase后，迭代器得到的并不是end()，所以需要添加一个empty的判断条件

但是，我又发现，在erase最后一个，之后得到的是地址的顺推，但是end()因为在队列中被释放了2次，所以得到的是原本的倒数第二个，与实际不相符合了。所以需要添加一个计数器来管理结束

看了下参考答案，原来是在遍历的时候就直接正反遍历，最后直接匹配树就好了，我想复杂了。

不过也是成功实现了。

### leetCode 88

题目讲的也太不清楚了，m和n的边界功能完全没交代，难怪差评那么多。

### leetCode 105

DFS版本，似乎没法做出 BFS版本来

引用传递指针，忘了这条，导致定义的指针始终没能指向函数中分配的堆地址

### leetCode 287

这道题，一开始没注意可以多次重复这件事，所以应该不是用异或运算了，而是利用值在1-n之间的这个条件。

看了一下，大致的解法有2种，1种就是用slow和fast两个迭代器，进行前进
另一种是用二分查找，但是这个挺不能理解的，因为没有规定重复的一定是连续的呀？

### leetCode 11

从n的中间点开始向两边遍历，找分别最长的2条线，

后来发现，还是直接从两端还是朝里走更简单，可以少去一些边界条件

### leetCode 134

### leetCode 142

我们注意到第一次相遇时

慢指针走过的路程S1 = 非环部分长度 + 弧A长

快指针走过的路程S2 = 非环部分长度 + n * 环长 + 弧A长

S1 * 2 = S2，可得 非环部分长度 = n * 环长 - 弧A长

让指针A回到起始点后，走过一个非环部分长度，指针B走过了相等的长度，也就是n * 环长 - 弧A长，正好回到环的开头。

要注意，让fast和slow同时从起点出发，否则遇到长度为2的环，就会TLE了

### leetCode 632

### leetcode 324

这道题，应该从后向前赋值，因为最后一位需要与他身后那一位比较，就可以免去与正向最大那一位的比较

### leetCode 524

sort中的比较函数compare要声明为静态成员函数或全局函数，不能作为普通成员函数，否则会报错。Line 26: invalid use of non-static member function
因为：非静态成员函数是依赖于具体对象的，而std::sort这类函数是全局的，因此无法再sort中调用非静态成员函数。静态成员函数或者全局函数是不依赖于具体对象的, 可以独立访问，无须创建任何对象实例就可以访问。同时静态成员函数不可以调用类的非静态成员。

### leetCode 713

窗口每次向右增加一个数字，然后左边去掉需要去掉的数字后，窗口的大小就是新的子数组的个数，每次加到结果res中即可

### leetCode 662

这里max({a,b,c})和max(a,max(b,c))得到的是两个结果，后一个完全错误了

没能想到是为什么，难道是对栈处理的问题？

换成引用值，在过程中使用max,久一点没有问题了

啊啊啊，

### leetCode 372

遵循公式
f(a,1234567) = f(a, 1234560) * f(a, 7) % k = f(f(a, 123456),10) * f(a,7)%k;

### leetCode 387
直觉，遍历过程中在之后的子串中进行find,找到就不是第一个，但是这个因为没有进行标记，完全可能把第二个出现的给标记成结果，导致错误。

### leetCode 696

始终没看懂题意，总算看懂以后也没看懂题解的方法，最后用了一个O(n^2)的解法，先找出所有的临界点，然后从临界点向外延伸

### leetcode 441

这道题感觉可能是可以理解成一个数学题的，但是完全想不到是什么keyword

等差数列求和公式、阶梯、一元二次方程

### leetcode 241

stoi和atoi还是有点不一样，处理string上，我把stoi换成atoi c_str就可以跑出结果，虽然还是错的

----------------------------------------------------

其实没有问题，只是我把substr传入范围出错了……天哪，我卡了一个晚上

### leetcode 90 subsets

为什么这里必须要先排序呢？我都用set来接收数据了，

### leetcode 60 第k个全排列数
这道题不能用回溯法了，需要寻找排列组合的规律


### leetcode 204 求素数数量

简单的话还是用素数筛选算法，

如果是应用需要的话，还是得用基于费马小定理的Miller Rabin算法，可以用快速幂取模优化一下

似乎这算是一个蒙特卡洛算法

因为是随机检测的原因吧
