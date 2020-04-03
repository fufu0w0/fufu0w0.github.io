---
title: 1064 Complete Binary Search Tree
date: 2020-02-19 18:25:20
categories: 
- 算法
tags: 
- PAT
- 二叉树
- 递归
---

写这篇是为了记录一下从中序遍历和后续遍历序列中建立二叉树的方法。

## 题目

Suppose that all the keys in a binary tree are distinct positive integers. A unique binary tree can be determined by a given pair of postorder and inorder traversal sequences. And it is a simple standard routine to print the numbers in level-order. However, if you think the problem is too simple, then you are too naive. This time you are supposed to print the numbers in "zigzagging order" -- that is, starting from the root, print the numbers level-by-level, alternating between left to right and right to left. For example, for the following tree you must output: 1 11 5 8 17 12 20 15.
![20200219182948.png](https://cdn.jsdelivr.net/gh/fufu0w0/myblogpic/20200219182948.png)

### Input Specification

Each input file contains one test case. For each case, the first line gives a positive integer N (≤30), the total number of nodes in the binary tree. The second line gives the inorder sequence and the third line gives the postorder sequence. All the numbers in a line are separated by a space.

### Output Specification

For each test case, print the zigzagging sequence of the tree in a line. All the numbers in a line must be separated by exactly one space, and there must be no extra space at the end of the line.

### Sample Input

```bash
8
12 11 20 17 1 15 8 5
12 20 17 11 15 8 5 1
```

### Sample Output

```bash
1 11 5 8 17 12 20 15
```

## 题目大意

给出一棵二叉树的中序遍历和后续遍历序列，要求输出该二叉树zigzaging序列。

## 解体思路

解题分两步：

* 从中序遍历和后续遍历序列建立二叉树，这一步用递归实现。
* 从建立的二叉树中得到翻转的层次遍历序列，这一步用BFS实现。

从后往前遍历后续遍历序列，后续遍历（左右中）的最后一个数就是当前树的根，然后在中序遍历序列中找到这个数，它在中序遍历序列中的位置为i，用它把中序遍历序列分为左右两个部分。

递归遍历中序遍历序列的右边和左边两个部分，它们也就是二叉树的左子树和右子树（顺序不能交换,因为每次取后续遍历的最后，右边的各个子树都访问了，才访问左子树），递归建树的顺序就是当前的根，建立当前的根的右子树，建立当前的根的左子树，按照这个顺序，每一次取到的根就是后续遍历序列从后向前依次访问的顺序。

将根节点入队，然后依次将它的左孩子和右孩子入队，然后根节点出队。在入队的过程中记录每次节点的入队顺序，深度和当前level的输出翻转标记位`f`，f为1表示这一行是从前到后，f为-1表示这一行从后到前输出。将节点推入答案数组。

对数组ans排序后输出，在排序时，层数小的在前；若层数相同，当前节点的f和入队顺序的乘积小的在前。最后输出排序后的数组。

## 代码

```c++
#include <algorithm>
#include <iostream>
#include <queue>
#include <vector>
using namespace std;
vector<int> inorder;
vector<int> postorder;
struct node {
    int value;
    node* left = NULL;
    node* right = NULL;
    int f;
    int order;
    int level = 0;
};
void post(node* r);
void build(node*& root, int l, int r);
int tail;
bool cmp(node* a, node* b) {
    if (a->level == b->level) {
        return a->f * a->order < b->f * b->order;
    }
    return a->level < b->level;
}
int main() {
    int n;
    cin >> n;
    inorder.resize(n);
    postorder.resize(n);
    tail = n - 1;
    node* root = NULL;
    for (int i = 0; i < n; i++) {
        cin >> inorder[i];
    }
    for (int i = 0; i < n; i++) {
        cin >> postorder[i];
    }
    build(root, 0, n - 1);
    queue<node*> qt;
    qt.push(root);
    vector<node*> ans;
    int order = 0;
    while (!qt.empty()) {
        node* c = qt.front();
        c->order = order++;
        qt.pop();
        if (c->left != NULL) {
            c->left->level = c->level + 1;
            qt.push(c->left);
        }
        if (c->right != NULL) {
            c->right->level = c->level + 1;
            qt.push(c->right);
        }
        if (c->level % 2 == 0) {
            c->f = -1;
        } else {
            c->f = 1;
        }
        ans.push_back(c);
    }
    sort(ans.begin(), ans.end(), cmp);
    for (int i = 0; i < ans.size(); i++) {
        if (i != 0) {
            cout << " ";
        }
        cout << ans[i]->value;
    }
}
void build(node*& root, int l, int r) {
    if (l > r) {
        return;
    }
    root = new node();
    root->value = postorder[tail--];
    int i;
    for (i = l; i <= r; i++) {
        if (inorder[i] == root->value) {
            break;
        }
    }
    build(root->right, i + 1, r);
    build(root->left, l, i - 1);
}
```
