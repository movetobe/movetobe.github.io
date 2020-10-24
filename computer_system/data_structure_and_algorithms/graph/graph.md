## 一、图搜索
#### 广度优先搜索
层级搜索，故适合求解类似于最短路径/最少等问题。基本方法：  
1. 起点入队列
1. 遍历队列，如果需要在最短路径情况的解，需要按层遍历，层的宽度通过获取当前队列的长度的决定
1. 对该层的所有点，判断其四个方向点是否满足条件，可以入队列
1. 循环2-3步

通常会定义一个cost[i][j]来存储到达(i, j)时的最少消耗/最大收益，需要仔细判断什么时候能够更新这个cost[i][j]值，同时将该点入队列

广度优先搜索因为每搜索到一层，就是从起点到当前层的最短路径，故天然地适合去解决最短路径上问题。很适合去求解路径上需要消除障碍物或者路径上获取最大价值之类的问题。

#### *[\[leetcode-1293\]](https://leetcode-cn.com/problems/shortest-path-in-a-grid-with-obstacles-elimination/) 网格中的最短路径*
*给你一个 m x n 的网格，其中每个单元格不是 0（空）就是 1（障碍物）。每一步，您都可以在空白单元格中上、下、左、右移动。如果您 最多 可以消除 k 个障碍物，请找出从左上角 (0, 0) 到右下角 (m-1, n-1) 的最短路径，并返回通过该路径所需的步数。如果找不到这样的路径，则返回 -1。如：  
输入：  
grid =   
[[0,0,0],  
 [1,1,0],  
 [0,0,0],  
 [0,1,1],  
 [0,0,0]],   
k = 1  
输出：6*

* 要找最多消除k个障碍物到达目的地
* 要求最短路径
* 故采用广度优先搜索，按层遍历，每次找到可以往下走的点。
* cost[][]保存每个单元格消除障碍物的数量，什么时候能够更新呢？
> 当从其他路径可以用消除更少的障碍物，而到达该单元格时需要更新单元格的cost  
* 两种更新场景，更新代表找到了更优的解，需要将当前解入队列：
    - 该单元格有障碍物，cost需要加1
    - 该单元格无障碍物，cost无需增加，就是到达上个节点的cost

```
int cross_obstacles(int **matrix, struct position *start, struct position *end, int k)
{
	int row = end->x + 1, col = end->y + 1;
	int **cost = calloc(1, sizeof(int *) * row);
	int **visited = calloc(1, sizeof(int *) * row);
	int i, j;
	struct position current, next;
	struct queue *q = init_queue();
	int level_size;
	struct position directions[] =  {% raw %}{{-1, 0}, {0, 1}, {1, 0}, {0, -1}} {% endraw %};
	int path_len = 0;;
	int find = 0;

	for (i = 0; i < row; i++) {
		cost[i] = calloc(1, sizeof(int) * col);
		visited[i] = calloc(1, sizeof(int) * col);
		for (j = 0; j < col; j++) {
			cost[i][j] = INT_MAX;
		}
	}

	/* enqueue first point, start/end must be 0 */
	enqueue(q, start);
	visited[start->x][start->y] = 1;
	cost[start->x][start->y] = 0;
	while (queue_size(q) > 0) {
		level_size = queue_size(q);
		path_len++;
		for (i = 0; i < level_size; i++) {
			dequeue(q, &current);
			visited[current.x][current.y] = 0;
			if (current.x == end->x && current.y == end->y) {
				find = 1;
				break;
			}
			for (j = 0; j < 4; j++) {
				next.x = current.x + directions[j].x;
				next.y = current.y + directions[j].y;
				if ((next.x < 0) || (next.x >= row) || (next.y < 0) || (next.y >= col)) {
					continue;
				}
				/* next is not an obstacles */
				if ((matrix[next.x][next.y] == 0) && (cost[next.x][next.y] > cost[current.x][current.y])) {
					cost[next.x][next.y] = cost[current.x][current.y];
					if (!visited[next.x][next.y]) {
						enqueue(q, &next);
						visited[next.x][next.y] = 1;
					}
				}
                /* next is an obstacles */
				if (matrix[next.x][next.y] && (cost[current.x][current.y] + 1 <= k)
					&& (cost[next.x][next.y] > cost[current.x][current.y] + 1)) {
					/* next cost is greater than from current->next */
					cost[next.x][next.y] = cost[current.x][current.y] + 1;
					if (!visited[next.x][next.y]) {
						enqueue(q, &next);
						visited[next.x][next.y] = 1;
					}
				}
			}
		}
		if (find) {
			break;
		}
	}
	for (i = 0; i < row; i++) {
		free(cost[i]);
		free(visited[i]);
	}
	free(cost);
	free(visited);
    destroy_queue(q);
	return (find ? path_len - 1: -1);
}
```

类似的题目如*[\[leetcode-5347\]](https://leetcode-cn.com/problems/minimum-cost-to-make-at-least-one-valid-path-in-a-grid/) 使网格图至少有一条有效路径的最小代价*, 此时，由于只需要有效路径，故只需要能让cost变小的路径就应该更新cost。

```
/* d[i][j] -> d[i + dx][j + dy] update everytime if the value less. */          
int min_cost(int **grid, int grid_size, int *grid_col_size)
{                        
    int i, j;                                                                   
    int **cost = calloc(1, sizeof(int *) * gridSize);        
    int **visited = calloc(1, sizeof(int *) * gridSize);                     
    struct queue *q = init_queue();                                             
    struct position pos = {0, 0}, next;                                         
    struct position direct[] =  {% raw %}{{0, 0}, {0, 1}, {0, -1}, {1, 0}, {-1, 0}} {% endraw %};        
    int m;                                                                      
                                                                                
    for (i = 0; i < gridSize; i++) {                                            
        cost[i] = calloc(1, sizeof(int) * gridColSize[i]);                      
        visited[i] = calloc(1, sizeof(int) * gridColSize[i]);
        for (j = 0; j < gridColSize[i]; j++) {                                  
            /* set to be max */                                                 
            cost[i][j] = gridSize + gridColSize[i];                             
        }                                                                       
    }                                                                           
                                                    
    enqueue(q, &pos);
    cost[0][0] = 0;
    visited[pos.x][pos.y] = 1;                                                           
    while (!queue_empty(q)) {                                                
        dequeue(q, &pos);
        visited[pos.x][pos.y] = 0;
        for (i = 1; i <= 4; i++) {                                              
            if ((pos.x + direct[i].x < 0) || (pos.x + direct[i].x >= gridSize)  
                || (pos.y + direct[i].y < 0) || (pos.y + direct[i].y >= gridColSize[0])) {
                continue;                                                       
            }            
            next.x = pos.x + direct[i].x;                                       
            next.y = pos.y + direct[i].y;                                                          
            if ((grid[pos.x][pos.y] == i)                                       
                && (cost[pos.x][pos.y] < cost[pos.x + direct[i].x][pos.y + direct[i].y])) {
                cost[pos.x + direct[i].x][pos.y + direct[i].y] = cost[pos.x][pos.y];
                if (!visited[next.x][next.y]) {
                    enqueue(q, &next);    
                    visited[next.x][next.y] = 1;
                }
            } else if ((cost[pos.x][pos.y] + 1) < cost[pos.x + direct[i].x][pos.y + direct[i].y]){
                cost[pos.x + direct[i].x][pos.y + direct[i].y] = cost[pos.x][pos.y] + 1;
                if (!visited[next.x][next.y]) {
                    enqueue(q, &next);    
                    visited[next.x][next.y] = 1;
                }
            }                                                                  
        }                                                                       
    }                                                                           
                                                                                
    m = cost[gridSize - 1][gridColSize[0] - 1];                                 
    destroy_queue(q);                                                           
    for (i = 0; i < gridSize; i++) {                                            
        free(cost[i]);                                     
        free(visited[i]);                     
    }     
    free(cost);    
    free(visited);                                                             
    return m;                                                                   
}    
```

#### 深度优先搜索

## 二、拓扑排序
实际上，拓扑排序可以看作是
拓扑排序的基本方法：  
1. 从图中选取所有入度为0的点，入队列
1. 出队列取值，得到入度为0的点A
1. 遍历所有节点，如果节点的入度为A，删除该节点的入边A，如果删除之后入度为0，则入队列
1. 循环处理2-3，直到所有都处理完，即队列为空

基本框架：

```
void topological_sort(int **graph, int *ingree, int num, int *path)
{                                                                                                        
    int i = 0, j = 0;                                                                                         
    int current;                                                                
    struct queue *q = init_queue();

    /* all nodes that ingree == 0 enqueue. */                                                      
    for (j = 0; j < num; j++) {                                          
        if (ingree[j] == 0) {                                                   
            enqueue(q, j);                                                      
        }                                                                       
    }                                                                           
    while (!queue_empty(q)) {                                                   
        current = dequeue(q);                                                   
        path[i] = current;                                     
        i++;            
        /* remove current is it is the others ingree */                              
        for (j = 0; j < num; j++) {                                      
            if (graph[current][j]) {                                            
                ingree[j]--;                                                    
                graph[current][j] = 0;                                          
                if (ingree[j] == 0) {                                           
                    enqueue(q, j);                                              
                }                                                               
            }                                                                   
        }                                                                       
    }
    destroy_queue(q);
    return;            
}
```