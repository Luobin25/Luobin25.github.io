---
layout:     post
title:      Union-find分析
subtitle:   algorithm
date:       2019-01-11
author:     LB
header-img: img/post-bg-debug.png
catalog: true
tags:
    - 算法
---
# Union-find 算法

## quick-find
原理：当且仅当id[p]等于id[q]时p和q是连通的。换句话说，在同一个连通分量中的所有触点在id[]中的值必须全部相同。

>存在的问题:

```
	public void union(int p, int q) {
        int pID = id[p];   // needed for correctness
        int qID = id[q];   // to reduce the number of array accesses

        // p and q are already in the same component
        if (pID == qID) return;

        for (int i = 0; i < id.length; i++)
            if (id[i] == pID) id[i] = qID;
        count--;
    }
```
对于每一对union的输入，我们都需要扫描整个id【】数组，假设我们使用quick-find来解决连通性问题，并且最后只得到了一个联通分量。至少需要～N^2次数组访问
	
    quick-find 是一个平方级别算法
    
## quick-union
原理：运用了树的模型，对union进行了优化，对于两个触点，只需要找出其根触点，然后将根触点的值改成另外一个。但find的时间增多了
> 存在的问题：

```
    public int find(int p) {
        while (p != parent[p])
            p = parent[p];
        return p;
    }
    public void union(int p, int q) {
        int rootP = find(p);
        int rootQ = find(q);
        if (rootP == rootQ) return;
        parent[rootP] = rootQ; 
        count--;
    }
```

![quick-union](https://raw.githubusercontent.com/Luobin25/algorithm/master/union-find/pic/quick-union%20bad%20situation.png?token=Af_Nlg-iqg9jVfxmXfUlJ7xPm3Y5VSx2ks5caro_wA%3D%3D)

对于极端情况，如上图，union操作访问数组的次数为2i+2（触点0的深度为i，触点i的深度为0)。因此，处理N地整数所需的所有find（）操作总次数为2(1+2+...N) ~N^2
## 加权quick-union算法
原理：与其在union（）中随意将一棵树连接到另一棵树，我们现在会记录每一棵树的大小并总是将较小的树连接到较大的树上。
```
    public void union(int p, int q) {
        int rootP = find(p);
        int rootQ = find(q);
        if (rootP == rootQ) return;

        // make smaller root point to larger one
        if (size[rootP] < size[rootQ]) {
            parent[rootP] = rootQ;
            size[rootQ] += size[rootP];
        }
        else {
            parent[rootQ] = rootP;
            size[rootP] += size[rootQ];
        }
        count--;
    }
```

![comprare weight or not](https://raw.githubusercontent.com/Luobin25/algorithm/master/union-find/pic/compre%20with%20quick%20union.png?token=Af_Nlg907RhZYUoP5u_ERukD4BbfX5Qrks5car1twA%3D%3D)

在最坏的情况下，假定树总共有2^N个节点。当我们归并两个含有2^n个节点的树时，树的高度增加到了n+1.所以对于N个触点，由加权quick-union算法构造的森林中任意节点的深度最多为lgN
## 路径压缩的加权quick-union
>路径压缩
	
	parent[p] = parent[parent[p]];使得find的过程中调整树的高度
	
>完全路径压缩
	
	parent[p] = Find(parent[p])通过递归faltten out树，使得
    每个leaf code指向root code
    
## Redundant Connection
题目：
给予一个二维数组，找出可以移除的edge（其中一个value）使得connection不再冗余（leetcode 684题）
```
Input: [[1,2], [1,3], [2,3]]
Output: [2,3]
Explanation: The given undirected graph will be 
like this:
  1
 / \
2 - 3
```
**分析**
：这道题的本质就是在考虑对union-find算法的使用。一开始，每个触点都是互不联通，我们逐一调用union互通，而每次调用union都会返回个boolean值（修改了模板union的返回类型），如果为false，表明之前已经连通了，再次union会让连通冗余
```
WeightedQuickUnionUF uf = new WeightedQuickUnionUF(edges.length);
        for(int[] edge : edges){
            if(!uf.union(edge[0], edge[1])){
                return edge;
            }
        }
        return null;
```
## Friend Circles
题目：
There are N students in a class. Some of them are friends, while some are not. Their friendship is transitive in nature. For example, if A is a direct friend of B, and B is a direct friend of C, then A is an indirect friend of C. And we defined a friend circle is a group of students who are direct or indirect friends.（leetcode 547)
```
Input: 
[[1,1,0],
 [1,1,0],
 [0,0,1]]
Output: 2
Explanation:The 0th and 1st students are direct friends, 
so they are in a friend circle. 
The 2nd student himself is in a friend circle. So return 2.
```
**分析**:
这道题本质是寻找有多少根树，即存在多少的根节点。首先，初始化朋友圈时设定一个变量group（每个人都是自己的朋友圈），然后通过循环遍历，调用union连通，通过union返回的boolean值。如果为true，代表新的两个人建立朋友圈，此时group数减一。
```
public int findCircleNum(int[][] M) {
        int length = M.length;
        int group = length;
        WeightedQuickUnionUF uf = new WeightedQuickUnionUF(length);
        for(int i = 0; i < length; i++){
            for(int j = i + 1; j < length; j++)
                if(M[i][j] == 1){
                    if(uf.union(i, j)){
                        group--;
                    }
                }
            }
        return group;    
    }
```
## 总结
![quick-union](https://raw.githubusercontent.com/Luobin25/algorithm/master/union-find/pic/all%20of%20union-find.png?token=Af_NluMTNkuflG9Ir72h62a7pdS9gEoFks5car2fwA%3D%3D)

通过对union-find的探讨和改良，我们实现了时间复杂度从平方到接近常数时间。

    
