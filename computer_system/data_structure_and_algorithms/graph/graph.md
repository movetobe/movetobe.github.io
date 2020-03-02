## 最短路径
#### Dijkstra
#### Shortest Path Fast Algorithm


## 拓扑排序
拓扑排序的基本方法：  
1. 从图中选取所有入度为0的点，入队列
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
    destroy_queue(q);                                                                                                               return;                                                         
}
```