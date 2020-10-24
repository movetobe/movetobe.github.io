## 一、动态规划
对于可以分解成最优子问题的题目，通常可以采用递归求解。而递归一个明显的缺点就是需要求解重复的子问题。对于重复子问题，我们在递归章节采用了缓存的方法来规避。其实，将递归方法采用循环方式实现，加上子问题的缓存，就是动态规划。

## 二、动态规划应用
动态规划通常用在求解最值问题，如最大值/最小值问题。比较常见的问题在字符串和图中的求解。

### 字符串中的动态规划
解决两个字符串的动态规划问题，通常可以用两个指针分别指向字符串的最后，然后一步步往前走，缩小问题规模。  最为经典的字符串题目就是求解两个字符串的最长公共子序列。

#### *[\[leetcode-1143\]](https://leetcode-cn.com/problems/longest-common-subsequence/) 最长公共子序列*
*给定两个字符串 text1 和 text2，返回这两个字符串的最长公共子序列。  
一个字符串的 子序列 是指这样一个新的字符串：它是由原字符串在不改变字符的相对顺序的情况下删除某些字符（也可以不删除任何字符）后组成的新字符串。  
例如，"ace" 是 "abcde" 的子序列，但 "aec" 不是 "abcde" 的子序列。两个字符串的「公共子序列」是这两个字符串所共同拥有的子序列。*

类似于数学归纳法的分析方法。我们先讨论字符串$$text1$$从$$text1[0] \rightarrow text1[i - 1]$$和字符串$$text2$$从$$text2[0] \rightarrow text2[j - 1]$$的最长公共子序列。假设它们的最长公共子序列长度是$$lcs[i - 1][j - 1]$$，则我们用归纳法，$$text1$$和$$text2$$都增加一个字符$$text1[i]$$和$$text2[j]$$：

* 如果$$text1[i] == text2[j]$$，那么$$lcs[i][j] = lcs[i - 1][j- 1] + 1$$
* 如果$$text1[i] != text2[j]$$, 则$$text1[i]$$和$$text2[j]$$不可能同时为最长公共子串的字符，只能选择一个作为最长公共子串的字符：
	* 如果$$text1[i]$$作为最长公共子串的一个字符，则$$lcs[i][j] = lcs[i][j - 1]$$
	* 如果$$text2[j]$$作为最长公共子串的一个字符，则$$lcs[i][j] = lcs[i - 1][j]$$

那么$$text1[i]$$和$$text2[j]$$选取哪个作为最长公共子串的字符呢？既然我们要找的是最长的，那么对于$$lcs[i][j - 1]$$和$$lcs[i - 1][j]$$，我们当然是哪一个大就选哪一个。  
因此，就得到了最长公共子序列的数学表达式：

$$ lcs[i][j] = \left\{
\begin{aligned}
& lcs[i - 1][j - 1] + 1                     &{text1[i] == text2[j]}\\
& max\{lcs[i][j - 1], lcs[i - 1][j]\}    &{text1[i] != text2[j]}
\end{aligned}
\right. $$

>**为了实现方便，动态规划二维数组下标可以从 1 开始**

动态规划还有很重要的一点就是对数据的初始化，因为上述的递推关系，最后都要落到$$i == 0$$或者$$j == 0$$。通常对于有限的条件下，我们是可以比较容易的得到相应的数据。如上述$$lcs[i][j]$$，当$$i == 0$$或者$$j == 0$$时，意味着$$text1$$或者$$text2$$为空，那么它们的最长公共子串长度当然是0，即:  

```
lcs[0][0] = 0;                                                                 
for (i = 1; i <= len1; i++) {                                                  
    lcs[i][0] = 0;                                                             
}                                                                              
for (j = 0; j <= len2; j++) {                                                  
    lcs[0][j] = 0;                                                             
}
```
然后就是对前述递推表达式的实现：

```
for (i = 1; i <= len1; i++) {                                                  
    for (j = 1; j <= len2; j++) {                                              
        if (text1[i - 1] == text2[j - 1]) {                                    
            lcs[i][j] = lcs[i - 1][j - 1] + 1;                                 
        } else {                                                               
            lcs[i][j] = (lcs[i - 1][j] > lcs[i][j - 1] ? lcs[i - 1][j] : lcs[i][j - 1]);
        }              
    }                                                                          
} 
```

最后只需要取$$i == strlen(text1) \&\& j == strlen(text2)$$时的$$lcs[i][j]$$即是$$text1$$和$$text2$$的最长公共子序列的长度值。  

```
int longest_common_subsequence(char *text1, char *text2)                           
{                                                                                  
    int len1 = strlen(text1);                                                      
    int len2 = strlen(text2);                                                      
    int i, j, longest;                                                                      
    int **lcs = calloc(1, sizeof(int *) * (len1 + 1));                                                          
                                                                                   
    for (i = 0; i <= len1; i++) {                                                  
        lcs[i] = calloc(1, sizeof(int) * (len2 + 1));                              
    }
    /* Init */
    lcs[0][0] = 0;                                                                 
    for (i = 1; i <= len1; i++) {                                                  
        lcs[i][0] = 0;                                                             
    }                                                                              
    for (j = 0; j <= len2; j++) {                                                  
        lcs[0][j] = 0;                                                             
    }
    /* According express to get next value. */
    for (i = 1; i <= len1; i++) {                                                  
        for (j = 1; j <= len2; j++) {                                              
            if (text1[i - 1] == text2[j - 1]) {                                    
                lcs[i][j] = lcs[i - 1][j - 1] + 1;                                 
            } else {                                                               
                lcs[i][j] = (lcs[i - 1][j] > lcs[i][j - 1] ? lcs[i - 1][j] : lcs[i][j - 1]);
            }              
        }                                                                          
    } 
    
    longest = lcs[len1][len2];
    for (i = 0; i <= len1; i++) {                                                  
        free(lcs[i]);                                                              
    }                                                                              
    free(lcs);
    return longest;                                                             
}
```

### 图中的动态规划
图一般通过二维数组表示，如果其求解的最值问题是单方向增长的，就可以比较容易地想到使用动态规划求解。  
什么叫做单方向增长呢？比如说，网格上找路径，如果寻找方向只能从左上角->右下角搜索，那么这里就称为单方向增长。如果寻找方向可以上下左右搜索，那么更加适合递归地去搜索。如下面的最短路径和，就是典型的图中动态规划的题目，它的增长方向就是从左上角到右下角。

#### *[\[leetcode-64\]](https://leetcode-cn.com/problems/minimum-path-sum/)最短路径和*
*给定一个包含非负整数的 m x n 网格，请找出一条从左上角到右下角的路径，使得路径上的数字总和为最小。  
说明：每次只能向下或者向右移动一步。  
如，输入:   
1,3,1  
1,5,1  
4,2,1  
输出: 7   
解释: 因为路径 1->3->1->1->1 的总和最小。*

分析方法跟字符串中动态规划的分析方法类似，采用归纳法：  
假设当前走到了网格中的$$grid[i][j]$$节点，此时路径上的数字总和最小为$$ps[i][j]$$，那么：  

* 走到$$grid[i - 1][j]$$时，数字总和最小为$$ps[i - 1][j]$$
* 走到$$grid[i][j - 1]$$时，数字总和最小为$$ps[i][j - 1]$$

显然，因为每个节点只能从两个方向过来，故：  
$$ps[i][j] = min\{ps[i - 1][j], ps[i][j - 1]\}$$

初始化过程：
同样对于$$i == 0$$或者$$j == 0$$需要做数据初始化。当$$i == 0$$或者$$j == 0$$时，只有一条路，就是要么向右直走，要么向下直走。故$$ps[i][0]$$或者$$ps[0][j]$$初始值就是它们一路的路径之和，即:

```
ps[0][0] = grid[0][0];
for (i = 1; i < gridSize; i++) {                                               
    ps[i][0] = ps[i - 1][0] + grid[i][0];                                      
}
for (j = 1; j < gridColSize[0]; j++) {                                         
    ps[0][j] = ps[0][j - 1] + grid[0][j];                                      
}     
```
然后按上述递推式，完成所有数据：

```
for (i = 1; i < gridSize; i++) {                                               
    for (j = 1; j < gridColSize[0]; j++) {                                     
        ps[i][j] = (ps[i - 1][j] < ps[i][j - 1] ? ps[i - 1][j] : ps[i][j - 1]) + grid[i][j];
    }                                                                          
}   
```
最后所要求取的，就是走到右下角，即$$ps[i][j]$$的右下角的值。

```
/* ps[i][j] = min{ps[i - 1][j], p[i][j - 1]} + grid[i][j] */                       
int min_path_sum(int **grid, int gridSize, int *gridColSize)                         
{                                                                                  
    int i, j, min_sum;                                                                    
    int **ps = calloc(1, sizeof(int *) * gridSize);
    
    for (i = 0; i < gridSize; i++) {                                               
        ps[i] = calloc(1, sizeof(int) * gridColSize[0]);                           
    }                                                                              
                                                                                   
    ps[0][0] = grid[0][0];
    for (i = 1; i < gridSize; i++) {                                               
        ps[i][0] = ps[i - 1][0] + grid[i][0];                                      
    }
    for (j = 1; j < gridColSize[0]; j++) {                                         
        ps[0][j] = ps[0][j - 1] + grid[0][j];                                      
    }                                                                              
                                                                                   
    for (i = 1; i < gridSize; i++) {                                               
        for (j = 1; j < gridColSize[0]; j++) {                                     
            ps[i][j] = (ps[i - 1][j] < ps[i][j - 1] ? ps[i - 1][j] : ps[i][j - 1]) + grid[i][j];
        }                                                                          
    }                                                                              
                                                                                   
    min_sum = ps[gridSize - 1][gridColSize[0] - 1];
    for (i = 0; i < gridSize; i++) {                                               
        free(ps[i]);                                                               
    }                                                                              
    free(ps);
    return min_sum;                                                                
}       
```

## 三、总结
动态规划解答通常分为3步：

* 递归表达式
* 初始化数据
* 循环求解数据

最难的就是递归表达式的描述了。