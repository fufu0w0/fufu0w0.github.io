---
title: 数据结构复习笔记-AVL树
date: 2020-2-17 22:34:20
updated: 2020-2-17 22:34:20
categories: 
- 算法
tags: 
- avl树
- 笔记
- 数据结构
mathjax: true
---

## 预备知识

`高度`：从叶子到当前节点的最长路径，根的高度就是树的高度，叶子高度为0

`深度`：从根到当前节点的路径长，根的深度为0

## 递推式

高度为h的`AVL树`，至少包含$S(h)=fib(h+3)-1$个节点。因此可以得到递推式

$$
\begin{aligned}
S(h) =fib(h+3)-1\\
S(h-1)=fib(h+2)-1\\
S(h-2)=fib(h+1)-1\\
S(h)=S(h-1)+S(h-2)+1\\
\end{aligned}
$$

有$N$个节点的AVL树，查询的时间复杂度为$O(logN)$

## 平衡

`平衡因子`：节点左右子树的高度之差（左子树高度-右子树高度）

`平衡`：各节点的平衡因子都是 0, -1 或 1

`删除`：在节点删除时，**至多**只有一个节点失衡

`插入`：在节点插入时，**至多**引起$logN$个节点失衡

## 插入

插入节点`x`到节点`v`之后，若同时有多个节点失衡，最低者不会低于`x`的祖父。

`左旋`(zag)：

* 失衡节点右侧子树太高
* 把失衡节点接手右孩子的左子树，右孩子的左子树变为此时的以失衡节点为根的子树。

```c++
node* leftRotate(node *a){
    node *temp = a->right;
    a->right = temp->left;
    temp->left = a;
    return temp;
}
```

`右旋`(zig)：

* 失衡节点左侧子树太高。
* 把失衡节点接手左孩子的右子树，左孩子的右子树变为此时的以失衡节点为根的子树。

```c++
node* rightRotate(node *a){
    node *temp = a->l;
    a->l = temp->r;
    temp->r = a;
    return temp;
}
```

`单旋`：

* 三代节点`g`,`p`,`v`方向一致
* 在左侧插入，方向一致向左的情况称为`zig-zig` (失衡节点的左孩子的值大于插入节点的值，插入的节点在失衡节点左孩子的左侧)
![20200218001636.png](https://cdn.jsdelivr.net/gh/fufu0w0/myblogpic/20200218001636.png)
* 在右侧插入，方向一致向右的情况称为`zag-zag` (失衡节点的右孩子的值小于插入节点的值，插入的节点在失衡节点右孩子的右侧)
![20200218001742.png](https://cdn.jsdelivr.net/gh/fufu0w0/myblogpic/20200218001742.png)
* 因为方向一致，在旋转后会保持BST当前节点大于左侧节点小于右侧节点要求
* 一次右旋或者左旋，最低的失衡节点恢复平衡，其上所有失衡节点也恢复平衡，全树恢复平衡
* 只涉及常数个节点，时间复杂度$O(1)$
`双旋`：

* 三代节点`g`,`p`,`v`方向不一致
* 在左侧插入，失衡节点的左孩子小于插入节点，插入节点在失衡节点左孩子的右侧，先对失衡节点的左孩子左旋，然后再对失衡节点右旋，这称为`zig-zag`
![20200218002614.png](https://cdn.jsdelivr.net/gh/fufu0w0/myblogpic/20200218002614.png)
* 在右侧插入，失衡节点的右孩子大于插入节点，插入节点在失衡节点右孩子的左侧，先对失衡节点的右孩子右旋，然后再对失衡节点左旋，这称为`zag-zig`![20200218002412.png](https://cdn.jsdelivr.net/gh/fufu0w0/myblogpic/20200218002412.png)
* 在这种情况下，若只进行一次左旋或右旋将会破坏二叉搜索树左侧节点小于右侧节点的特性，因此要先对左孩子进行左旋操作或右孩子进行右旋操作。
* 只涉及常数个节点，时间复杂度$O(1)$

如果不理解可以在草稿本上画个图。

插入操作的代码实现如下：

```c++
node* insert(node *root, int value) {
    if (root == NULL) {
        root = new node;
        root->v = value;
        return root;
    }
    if(value<root->v){
        root->l = insert(root->l, value);
        if(getHeight(root->l)-getHeight(root->r)>1||getHeight(root->l)-getHeight(root->r)<-1){
            if(root->l->v>value){//zig-zig
                root=rightRotate(root);
            }
            else{
                root->l = leftRotate(root->l);//zag-zig
                root = rightRotate(root);
            }
        }
    }
    else{
        root->r=insert(root->r, value);
        if(getHeight(root->l)-getHeight(root->r)>1||getHeight(root->l)-getHeight(root->r)<-1){
            if(root->r->v<value){//zag-zag
                root=leftRotate(root);
            }
            else{
                root->r = rightRotate(root->r);//zig-zag
                root = leftRotate(root);
            }
        }
    }
    return root;//返回子树的根
}
```

没写完，删除等有时间再写。

## 参考

[《数据结构(C++语言版)》 邓俊辉](https://book.douban.com/subject/25859528/)

[PAT 1166 Root of AVL Tree  (30分)](https://pintia.cn/problem-sets/994805342720868352/problems/994805404939173888)

[PAT 1123 Is It a Complete AVL Tree (30分)](https://pintia.cn/problem-sets/994805342720868352/problems/994805351302414336)
