

# Optimizing by Greedy

- Coin Change Problem
	- candidate: A finite set of coins, of 1,5,10 and 25 units, with enough number for each value
	- constraints: Pay an exact amount by a selected set of coins
	- optimization: a smallest possible number of coins in the selected smallest
	- 设计货币：如人民币的货币系统的面值有1, 5, 10, 20, 50, 100. 这就是一个最优化问题
- For any amount n:
$$
	min\{i+j+k+l\}
$$
$$
	s.t.\; i+5j+10k+25l=n, 	\text{where $i,j,k,l\in \mathbb{N}$}
$$

For a Greedy Problem, prove the correctness of algorithm is often harder than designing the algorithm.

How to prove?

to prove 
$$
(i^*,j^*,l^*,k^*)==(i,j,k,l);
$$
## Greedy Stratedy
Constructing the final solution by expanding the partial solution step by step, in each of which a selection is made from a set of candidates, with the choice made must be:
- **feasible** 适合性：每时每刻不能与条件矛盾
- **locally optimal** 局部最优导出到全局最优
- **irrevocable** （不回头）已经选择的方案不能在之后被取消选择
**KEY**: trading off **feasible** and locally **optimal**
```python
set greedy(set candidate)

    set S=Ø;

    while not solution(S) and   candidate!= Ø

        select locally optimizing x   from candidate;

        candidate=candidate-{x};

        if feasible(x) then S=S union {x};

    if solution(S) then return S

            else return (“no solution”)
```

## Use of Greedy Strategy
### Weighted Graph and MST
MST(Minimum Spanning Tree)
![[Pasted image 20241127143226.png]]

it can be proved that, 
for traditional Graph Traversal strategy such as DFS and BFS, there are cases that graph traversal tree **cannot** be minimum spanning tree, with the vertices explored in **any order**.

### Greedy Algorithms for MST
- Prim' s algorithm
	- Difficult selecting
	- easy circle checking
- Kruskal' s algorithm
	- easy selecting
	- hard circle checking
### Merging Two Vertices
Merging $v_1$ and $v_{2}$ , is delete $v_2$ and give all the neighbors of $v_2$ to $v_{1}$
### Prim' s Algorithm
- Greedy strategy
	For each set of fringe vertex, select the edge with the minimal weight , which is **local optimal**
	*Prim*算法以图团去探索边。同时避免了成环。

![[Pasted image 20241127150554.png|500]]


```cpp
void primMST(G,n)

    Initialize the priority queue pq as empty;

    //Select vertex s to start the tree;

    //Set its candidate edge to (-1,s,0);

    insert(pq,s,0);

    while (pq is not empty)

        v=getMin(pq); deleteMin(pq);

        add the candidate edge of v to the tree;

        updateFringe(pq,G,v);

    return

void updateFringe(pq,G,v)

    for all vertices w adjcent to v //2m loops

        newWgt=w(v,w);

        if w.status is unseen then

            //Set its candidate edge to (v,w,newWgt);

            insert(pq,w,newWgt)

        else

            if newWgt<getPriorty(pq,w)

                //Revise its candidate edge to (v,w,newWgt);

                decreaseKey(pq,w,newWgt)

    return

```
优先队列Priority Queue可用Heap or List实现

---
**最小生成树性质**
**Minimum Spanning Tree Property：**
- A spanning tree T of a connected, weighted graph has MST property **if and only if** for any non-tree edge uv, TÈ{uv} contain a cycle in which uv is one of the maximum-weight edge.

- All the spanning trees having MST property have the same weight.![[Pasted image 20241127154119.png |600]]
上图直观展示了对==MST性质的证明==。

### Kruskal Algorithm
From the set of edges not yet included in the partially built MST, select the edge with the minimal weight, that is, local optimal, in another sense.
*Kruskal* *算法* 直观的扫描边集，从中选择权重**最小**的、不会导致成环的边作为新的边。也是局部最优。

怎么**高效**的 CHECK 是否有环？**并查集**（Union-Find set）

### 前缀码

**哈夫曼编码**是广泛地用于数据文件压缩的十分有效的编码方法。其压缩率通常在20%～90%之间。哈夫曼编码算法用字符在文件中出现的频率表来建立一个用0，1串表示各字符的最优表示方式。

给出现频率高的字符较短的编码，出现频率较低的字符以较长的编码，可以大大缩短总码长。

- **前缀码**

  对每一个字符规定一个0,1串作为其代码，并要求任一字符的代码都不是其它字符代码的前缀。这种编码称为前缀码。
  这意味着任意字符的代码的前缀都不是合法的。（电话号码例子：你不能再拨打11位电话号码过程中触发8位的电话拨打）
- **哈夫曼编码**
	- 编码的前缀性质使得建构编码非常简单。
	- 利用 **完全二叉树**
	表示最优前缀码的二叉树总是一棵完全二叉树，即树中任一结点都有2个儿子结点。
	则此时二叉树每个节点都对应不同的号码，且到他的路径唯一，即不包含其他代码的前缀

# 再论贪心算法
**基本要素**：
- **贪心选择性质**：
	是指所求问题的 **整体最优解** 可以通过一系列 **局部最优**的选择。自顶向下，每一次贪心的决策都可以让问题变为更小的子问题。而 **分而治之**思想的解决问题合并过程， 是划分小问题再合并的思想。