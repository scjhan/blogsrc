---
title: 二叉树及其基本操作
toc: true
date: 2017-04-18 13:13:59
tags: bitree
categories: 数据结构与算法
---
## 0 概念
1.二叉树（Binary Tree）是含有n个结点的有限集合，具有以下几个特点：
- 有且只有一个称(root)的节点
- 其余结点划分为两个互不相交的子集L和R，称为左子树和右子树  
<!--more-->
2.二叉树的一些基本概念有：
- 度：结点的孩子个数称为度，对于二叉树，度可为0,1,2
- 层次：从根结点开始定义，根结点为1，根的孩子为2，以此类推
- 深度：二叉树中最大层次称为二叉树的深

3.二叉树性质
- 度0结点 = 度2结点 + 1
- ...

4.完全二叉树与满二叉树
- 满二叉树：深度为k且结点数为`2^k-1`的二叉树
- 完全二叉树：深度为k，前k-1层为满二叉树，最后一层结点排列在左边。

![](http://i4.buimg.com/567571/45a2489b24512042.png)

## 1 实现
```cpp
typedef struct btnode{
    int val;
    struct btnode *left;
    struct btnode *right;
} btnode, *btree;

btree create(int val, btree lt, btree rt)
{
    btree tree;
    tree = (btree)malloc(sizeof(btnode));

    if (tree == NULL) return NULL;

    tree->val = val;
    tree->left = lt;
    tree->right = rt;

    return tree;
}

void destroy(btree t)
{
    if (t == NULL) return;

    destroy(t->left);
    destroy(t->right);

    free(t);
    t = NULL;
}
```

## 2 遍历
### 1.1 中序遍历
中序遍历即**LDR**，先遍历左子树，再遍历根，最后遍历右子树。
- 递归形式的LDR
```cpp
/*中序遍历递归版*/
void ldr_rec(btree t, void (*visit)(int))
{
    if (t == NULL) return;
    ldr_rec(t->left, visit);
    visit(t->val);
    ldr_rec(t->right, visit);
}
```
- 非递归形式的LDR
```cpp
/*中序遍历非递归版*/
void ldr_nor(btree t, void (*visit)(int))  //先把所有左结点入栈，然后访问结点，最后转右子树
{
    if (t == NULL) return;

    btree p = t;
    std::stack<btnode*> s;
    while(!s.empty() || p)
    {
        if (p)
        {
            s.push(p);
            p = p->left;
        }
        else
        {
            p = s.top();
            s.pop();
            visit(p->val);
            p = p->right;
        }
    }
}
```
### 1.2 先序遍历
先序遍历即**DLR**，先访问根结点，然后访问左子树，最后访问右子树
- 递归形式的DLR
```cpp
/*先序遍历递归版*/
void dlr_rec(btree t, void (*visit)(int))
{
    if(t == NULL) return;
    visit(t->val);
    dlr_rec(t->left, visit);
    dlr_rec(t->right, visit);
}
```
- 非递归形式的DLR
```cpp
/*先序遍历非递归版*/
void dlr_nor(btree t, void (*visit)(int)) //先访问结点，然后往左走，直到没有左子树，再转右子树
{
    if (t == NULL) return;

    btree p = t;
    std::stack<btnode*> s;
    while (!s.empty() || p)
    {
        if (p)
        {
            visit(p->val);
            s.push(p);
            p = p->left;
        }
        else
        {
            p = s.top();
            s.pop();
            p = p->right;
        }
    }
}
```
### 1.3 后序遍历
后续遍历即**LRD**，先访问左子树，再访问右子树，最后访问根结点。  
非递归实现难度在于，要判断此次遍历是从左子树返回还是右子树返回，如果是左子树返回，则应往右走，如果是右子树返回，则可以访问根结点。
- 递归形式的LRD
```cpp
/*后序遍历递归版*/
void lrd_rec(btree t, void (*visit)(int))
{
    if (t == NULL) return;
    lrd_rec(t->left, visit);
    lrd_rec(t->right, visit);
    visit(t->val);
}
```
- 非递归形式的LRD
```cpp
/*后序遍历非递归版*/
void lrd_nor(btree t, void (*visit)(int)) //使用多一个指针记录上次访问位置，每次先访问最左，然后通过stack返回上层，与所记录的上次访问比较
{
    if (t == NULL) return;

    std::stack<btnode*> s;
    btree curr = t, last = NULL;
    //先把curr移动到最左
    while(curr)
    {
        s.push(curr);
        curr = curr->left;
    }
    while(!s.empty())
    {
        //curr == NULL
        curr = s.top();
        s.pop();
        if (curr->right == NULL || curr->right == last)
        {
            visit(curr->val);
            last = curr;
        }
        else
        {
            s.push(curr);
            //进入右子树,并将curr移动到最左
            curr = curr->right;
            while(curr)
            {
                s.push(curr);
                curr = curr->left;
            }
        }
    }
}

```
### 1.4 层次遍历
层次遍历即每次遍历按从左往右的顺序遍历，使用队列即可解决。
```cpp
/*层次遍历*/
void level_traverse(btree t, void (*visit)(int))
{
    if (t == NULL) return;

    btree p = NULL;
    std::queue<btnode*> que;
    que.push(t);
    while(!que.empty())
    {
        p = que.front();
        que.pop();

        visit(p->val);

        if (p->left) que.push(p->left);
        if (p->right) que.push(p->right);
    }
}

```

## 2 杂项问题
### 2.1 二叉树最大深度
- 递归法
```cpp
/*求树深度，递归实现*/
int depth(btree t)
{
    if (t == NULL) return 0;

    int ldepth = depth(t->left);
    int rdepth = depth(t->right);

    return 1 + std::max(ldepth, rdepth);
}
```
- 非递归法
受后序遍历启发，求树的深度实际上就是在LRD过程中，将访问结点改成修改深度值即可。对于左子树返回，则进入右子树，若为右子树返回，则记录下最大深度，然后深度-1，表示返回上一层。  
实际上，并不需要在后序遍历中记录深度值，LRD每次入栈都是入一层，进入左子树入栈，出来左子树出栈，进入右子树入栈，出来右子树出栈。也就是说，LDR过程中，栈的大小实际上就记录着二叉树的当前深度，只需要每次操作都记录最大深度即可。
```cpp
/*求树深度，非递归*/
int depth_nor(btree t)
{
    if (t == NULL) return 0;

    int d = 0, maxd = -1, maxstacksize = 0;
    btree curr = t, last = NULL;
    std::stack<btnode*> s;

    while(curr)
    {
        s.push(curr);
        curr = curr->left;
        //++d; 思路1
    }
    while(!s.empty())
    {
        curr = s.top();
        s.pop();

        if (curr->right == NULL || curr->right == last)
        {
            //返回上层
            last = curr;
            //--d;  思路1
        }
        else
        {
            //进入右子树
            s.push(curr);
            curr = curr->right;
            while(curr)
            {
                s.push(curr);
                curr = curr->left;
                //++d;  思路1
            }
        }
        //maxd = std::max(maxd, d);  思路1
        maxstacksize = std::max(maxstacksize, (int)s.size());
    }
    
    return maxstacksize;
}
```


### 2.2 二叉树最小深度
与二叉树的最大深度不同，不能单纯地使用递归，因为二叉树的深度必须是根结点到叶子结点的距离，不能单纯的比较左右子树的递归结果返回较小值，因为对于有单个孩子为空的节点，为空的孩子会返回0，但这个节点并非叶子节点，故返回的结果是错误的。  
因此，当发现当前处理的节点有单个孩子是空时，返回一个极大值INT_MAX，防止其干扰结果。
```cpp
/*二叉树深度*/
#define INT_MAX ((int)(~0U>>1))
int minDepth(btree t)
{
    if (t == NULL) return 0;

    if (t->left == NULL && t->right == NULL)
        return 1;

    int ld = minDepth(t->left) + 1;
    int rd = minDepth(t->right) + 1;

    //only one
    if (ld == 1) ld = INT_MAX;
    if (rd == 1) rd == INT_MAX;

    return std::min(ld, rd);
}
```
### 2.3 二叉树宽度
层次遍历中，每次都是逐行访问二叉树，这个过程其实就包含着二叉树的宽度。  
层次遍历中求二叉树宽度的思路是，每次循环都把上一行全部出队，把下一整行全部入队，入队前队列长度即为上一行宽度。
```cpp
/*二叉树宽度*/
int width(btree t)
{
    if (t == NULL) return 0;

    int maxwidth = 0;
    btree p = t;
    std::queue<btnode*> que;
    que.push(t);

    while (!que.empty())
    {
        int n = que.size();
        maxwidth = std::max(maxwidth, n);
        while (n--)
        {
            p = que.front();
            que.pop();
            if (p->left) que.push(p->left);
            if (p->right) que.push(p->right);
        }
    }

    return maxwidth;
}
```
### 2.4 重建二叉树
输入某二叉树的前序遍历和中序遍历的结果，输出原二叉树。  
思路：  
前序遍历的特点是最左边为父结点，右边分别是左子树和右子树。中序遍历的特点是中间为父结点，两边分别是左子树和右子树。如下图：  
```cpp
       前序遍历序列                     中序遍历序列  
{ 1, 2, 4, 7, 3, 5, 6, 8 }       { 4, 7, 2, 1, 5, 3, 8, 6 }
 fa----left----right----          ----left---fa---right----
```
由此，可由前序遍历结果得到的父结点找到其在中序遍历的位置index，并构建相应的父结点，然后递归构建index左边的左子树和右边的右子树。
```cpp
TreeNode* reConstructBinaryTree(vector<int> pre,vector<int> vin) 
{
    int i = 0;
    return getRoot(pre, vin, 0, vin.size(), i);
}
    
TreeNode* getRoot(vector<int> pre,vector<int> vin, int beg, int end, int &pi)
{
    //find mi
    struct TreeNode* root = new TreeNode(0);
    int mid;

    if (beg == end - 1)
    {
        root->val = vin[beg];
        return root;
    }
	else if (beg == end)
    {
        --pi;
        return NULL;
    }
    for (mid = beg; mid < end; ++mid)
        if (vin[mid] == pre[pi])
            break;
    
    root->val = vin[mid];
    root->left = getRoot(pre, vin, beg, mid, ++pi);
    root->right = getRoot(pre, vin, mid + 1, end, ++pi);
    
    return root; 
}
```
