## 一、二叉树
一直以为树比较简单，因为都是按照左子树/右子树递归来解决问题。但是这两天做了leetcode竞赛，其中关于二叉树的题目，不算难，也是按照左子树/右子树递归来解决，但是，在应用实现中，还是有值得思考的地方。

#### *[\[leetcode-5346\]](https://leetcode-cn.com/problems/linked-list-in-binary-tree/)二叉树中的列表*
*给你一棵以 root 为根的二叉树和一个 head 为第一个节点的链表。
如果在二叉树中，存在一条一直向下的路径，且每个点的数值恰好一一对应以 head 为首的链表中每个节点的值，那么请你返回 True ，否则返回 False 。
一直向下的路径的意思是：从树中某个节点开始，一直连续向下的路径。*
```
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     struct ListNode *next;
 * };
 */
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     struct TreeNode *left;
 *     struct TreeNode *right;
 * };
 */
```
根据题目简要分析：

* 如果head->val == root->val，则只需要判断head->next为首的链表是否存在于root->left为根的左子树或者以root->right为根的右子树中。
* 否则只需判断head链表是否存在root->left或者root->right的子树中

很容易就可以得到如下简洁清晰的代码：
```
int is_sub_path(struct ListNode* head, struct TreeNode* root){
    /* end */
    if (!head) {
        return 1;
    }
    if (!root) {
        return 0;
    }
    
    /* options */
    if (head->val == root->val) {
        if (isSubPath(head->next, root->left) || isSubPath(head->next, root->right)) {
            return 1;
        }
    }
    if (isSubPath(head, root->left)) {
        return 1;
    }
    if (isSubPath(head, root->right)) {
        return 1;
    }
    return 0;
}
```

很不幸的是，上面代码在leetcode上会超时。如果换一种写法，就能通过。

```
int sub_path(struct ListNode* head, struct TreeNode* root){
    /* end */
    if (!head) {
        return 1;
    }
    if (!root) {
        return 0;
    }
    if (head->val != root->val) {
        return 0;
    }
    /* options */
    return (sub_path(head->next, root->left) || sub_path(head->next, root->right));
}

int is_sub_path(struct ListNode* head, struct TreeNode* root){
    /* end */
    if (!head) {
        return 1;
    }
    if (!root) {
        return 0;
    }

    /* options */
    return (sub_path(head, root) || isSubPath(head, root->left) || isSubPath(head, root->right));
}
```
惭愧，至写此文时，还没能很好的分析出第二种写法性能好的原因。


#### *[\[leetcode-1339\]](https://leetcode-cn.com/problems/maximum-product-of-splitted-binary-tree/)分裂二叉树的最大乘积*
*给你一棵二叉树，它的根为 root 。请你删除 1 条边，使二叉树分裂成两棵子树，且它们子树和的乘积尽可能大。
由于答案可能会很大，请你将结果对 10^9 + 7 取模后再返回。*

* 分为以root'为根的子树，与root树其他部分
* root'为根的子树和，递归求解左子树和右子树+root'->val
* 求解以root'为根的子树与其余部分的乘积
* 左子树和右子树在递归求解子树和时，对任何一个root''，均会做这三步

这样求解，就从叶子到根，求解以任何一个节点为根的树，与其余部分的乘积

```
#define MAX(a, b) ((a) > (b) ? (a) : (b))
#define M 1000000007
long sum(struct TreeNode *root)
{
    if (!root) {
        return 0;
    }
    return (root->val + sum(root->left) + sum(root->right));
}
long cut(struct TreeNode *root, long *product, long total)
{
    long current_sum;

    if (!root) {
        return 0;
    }
    current_sum = root->val + cut(root->left, product, total) + cut(root->right, product, total);
    *product = MAX(*product, (total - current_sum)* current_sum);
    return current_sum;
}
int max_product(struct TreeNode* root)
{
    long product = 0;

    cut(root, &product, sum(root));
    return product % M;
}
```