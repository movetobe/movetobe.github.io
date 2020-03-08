### *[\[leetcode-5354. 通知所有员工所需的时间\]](https://leetcode-cn.com/problems/time-needed-to-inform-all-employees/)*
*公司里有 n 名员工，每个员工的 ID 都是独一无二的，编号从 0 到 n - 1。公司的总负责人通过 headID 进行标识。  
在 manager 数组中，每个员工都有一个直属负责人，其中 manager[i] 是第 i 名员工的直属负责人。对于总负责人，manager[headID] = -1。题目保证从属关系可以用树结构显示。  
公司总负责人想要向公司所有员工通告一条紧急消息。他将会首先通知他的直属下属们，然后由这些下属通知他们的下属，直到所有的员工都得知这条紧急消息。  
第 i 名员工需要 informTime[i] 分钟来通知它的所有直属下属（也就是说在 informTime[i] 分钟后，他的所有直属下属都可以开始传播这一消息）。  
返回通知所有员工这一紧急消息所需要的分钟数 。  
如果员工 i 没有下属，informTime[i] == 0 。*

思路上比较常规，采用深度优先搜索，搜索出具有最大通知时间的路径，即所有员工被通知到的最大时间。

深度优先搜索采用递归实现，每次到员工i时，递归通知其所有下属，这里是一个优化点，如何最快的时间找到其所有下属？
> * 可以遍历所有的节点，判断它的上司是不是i员工；
> * 事先做好保存员工i的所有下属的关系；

对于第一个选择，复杂度较高，会超时;  
对于第二个选择，如C++/Java/Python等语言都比较容易实现，但是对于C语言来说，不容易实现。同样的问题也会出现在 *[leetcode-582. 杀死进程]* 中。
这种问题有两种优化方法：
#### 1. 定义数据结构，事先计算每个员工有多少个下属，分配空间后，再遍历保存下属信息；
```
struct node {
    int child_size;
    int index;
    int *childs;
};
int inform(int n, int headID, struct node *parents, int *inform_time)
{
    int i = 0;
    int cost, max_cost = 0;

    if (inform_time[headID] == 0) {
        return 0;
    }
    for (i = 0; i < parents[headID].child_size; i++) {
        cost = inform(n, parents[headID].childs[i], parents, inform_time) + inform_time[headID];
        max_cost = MAX(max_cost, cost);
    }
    return max_cost;
}
int numOfMinutes(int n, int headID, int* manager, int managerSize, int* informTime, int informTimeSize)
{
    int i;
    int cost;
    int index;
    struct node *parents = calloc(1, sizeof(struct node) * n);

    for (i = 0; i < n; i++) {
        if (i != headID) {
            parents[manager[i]].child_size++;
        }
    }
    for (i = 0; i < n; i++) {
        parents[i].childs = calloc(1, sizeof(int) * parents[i].child_size);
    }
    for (i = 0; i < n; i++) {
        if (i != headID) {
            index = parents[manager[i]].index;
            parents[manager[i]].childs[index] = i;
            parents[manager[i]].index++;
        }
    }
    cost = inform(n, headID, parents, informTime);
    for (i = 0; i < n; i++) {
        free(parents[i].childs);
    }
    free(parents);
    return cost;
}
```

#### 2. 用两个数组，一个保存员工i的最后一个孩子，一个保存员工i的上一个兄长。
```
int inform(int n, int headID, int *last_child, int *sibling, int *inform_time)
{
    int i = 0;
    int cost, max_cost = 0;

    if (inform_time[headID] == 0) {
        return 0;
    }
    for (i = last_child[headID]; i >= 0; i = sibling[i]) {
        cost = inform(n, i, last_child, sibling, inform_time) + inform_time[headID];
        max_cost = MAX(max_cost, cost);
    }
    return max_cost;
}
int numOfMinutes(int n, int headID, int* manager, int managerSize, int* informTime, int informTimeSize)
{
    int i;
    int cost;
    int *last_child = calloc(1, sizeof(int) * n);
    int *sibling = calloc(1, sizeof(int) * n);

    memset(last_child, -1, sizeof(int) * n);
    for (i = 0; i < n; i++) {
        if (i != headID) {
            sibling[i] = last_child[manager[i]];
            last_child[manager[i]] = i;
        }
    }
    cost = inform(n, headID, last_child, sibling, informTime);
    free(last_child);
    free(sibling);
    return cost;
}
```


### *[\[leetcode-5355. T秒后青蛙的位置\]](https://leetcode-cn.com/problems/frog-position-after-t-seconds/)*
*给你一棵由 n 个顶点组成的无向树，顶点编号从 1 到 n。青蛙从 顶点 1 开始起跳。规则如下：  
在一秒内，青蛙从它所在的当前顶点跳到另一个 未访问 过的顶点（如果它们直接相连）。  
青蛙无法跳回已经访问过的顶点。  
如果青蛙可以跳到多个不同顶点，那么它跳到其中任意一个顶点上的机率都相同。  
如果青蛙不能跳到任何未访问过的顶点上，那么它每次跳跃都会停留在原地。
无向树的边用数组 edges 描述，其中 edges[i] = [fromi, toi] 意味着存在一条直接连通 fromi 和 toi 两个顶点的边。  
返回青蛙在 t 秒后位于目标顶点 target 上的概率。*

跟上一题很类似，就是深度优先搜索，但要注意一些细节：
> * 无向树，所有相连的节点都可以认为是孩子
> * 开始节点是1，可以认为它跟-1相连
> * 叶子节点只有跟唯一一个节点相连
> * 如果目标点是叶子节点，那么t >= 0都可以
> * 如果目标点不是叶子节点，那么t == 0才行

我们采用第一种方法，构建结构体来存储孩子节点：

```
struct node {
    int child_size;
    int index;
    int *childs;
};
double jump_success(struct node *parents, int from, int start, int t, int target)
{
    int i;
    double possible = 0.0;

    /* end */
    if (t < 0) {
        return 0.0;
    }
    if ((t == 0) && (start == target)) {
        return 1.0;
    }
    /* leaf */
    if (parents[start].child_size <= 1) {
        return (start == target ? 1.0 : 0.0);
    }

    /* options */
    for (i = 0; i < parents[start].child_size; i++) {
        if (parents[start].childs[i] != from) {
            possible += jump_success(parents, start, parents[start].childs[i], t - 1, target);
        }
    }
    return possible / (parents[start].child_size - 1);
}
double frogPosition(int n, int** edges, int edgesSize, int* edgesColSize, int t, int target){
    int i, j;
    double possible = 0.0;
    struct node *parents;
    int index1, index2;
    int begin = 1;

    n  = n + 1;
    parents = calloc(1, sizeof(struct node) * n);
    for (i = 0; i < edgesSize; i++) {
        parents[edges[i][0]].child_size++;
        parents[edges[i][1]].child_size++;
    }
    parents[begin].child_size++;
    for (i = 0; i < n; i++) {
        parents[i].childs = calloc(1, sizeof(int) * parents[i].child_size);
    }
    for (i = 0; i < edgesSize; i++) {
        index1 = parents[edges[i][0]].index;
        index2 = parents[edges[i][1]].index;
        parents[edges[i][0]].childs[index1] = edges[i][1];
        parents[edges[i][1]].childs[index2] = edges[i][0];
        parents[edges[i][0]].index++;
        parents[edges[i][1]].index++;
    }
    /* begin is from -1 */
    parents[begin].childs[parents[begin].index] = -1;
    parents[begin].index++;
    possible = jump_success(parents, -1, begin, t, target);

    for (i = 0; i < n; i++) {
        free(parents[i].childs);
    }
    free(parents);
    return possible;
}
```