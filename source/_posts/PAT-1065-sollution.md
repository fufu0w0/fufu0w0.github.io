---
title: 1064 Complete Binary Search Tree
date: 2020-01-23 22:34:20
categories: 
- 算法
- PAT
tags: 
- 二叉树
- 递归
mathjax: true
---

二叉搜索树性质和递归构造满二叉树的方法。

## 题目

A Binary Search Tree (BST) is recursively defined as a binary tree which has the following properties:

The left subtree of a node contains only nodes with keys less than the node's key.

* The right subtree of a node contains only nodes with keys greater than or equal to the node's key.
* Both the left and right subtrees must also be binary search trees.
* A Complete Binary Tree (CBT) is a tree that is completely filled, with the possible exception of the bottom level, which is filled from left to right.

Now given a sequence of distinct non-negative integer keys, a unique BST can be constructed if it is required that the tree must also be a CBT. You are supposed to output the level order traversal sequence of this BST.

### Input Specification

Each input file contains one test case. For each case, the first line contains a positive integer $N (≤1000)$. Then N distinct non-negative integer keys are given in the next line. All the numbers in a line are separated by a space and are no greater than 2000.

where the numbers are all integers no more than 500, and are separated by a space.

### Output Specification

For each test case, print in one line the level order traversal sequence of the corresponding complete binary search tree. All the numbers in a line must be separated by a space, and there must be no extra space at the end of the line.

### Sample Input

```bash
10
1 2 3 4 5 6 7 8 9 0
```

### Sample Output

```bash
6 3 8 1 5 7 9 0 2 4
```

## 题目大意

用给出的数字序列构造**满**二叉搜索树。

## 解体思路

排序后得到BST的中序遍历序列（左中右），然后构造满二叉树输出。在用数组表示的满二叉树中，当前根节点在数组中的下标为 $r$ ,其左孩子的下标是$(2*r+1)$,右孩子的下标是$(2*r+2)$,数组下标从0记。据此，我们可以递归的构造二叉树。递归基为当前的“根”的下标大于等于树的节点数时，从根节点开始，构造左子树，把写入根节点，构造右子树。

这样的方法同样适用于通过先序、中序、后续遍历序列构造出满二叉树，只是递归左子树，写入根节点，递归右子树的顺序不同，思路什么的都是一样的，在这里简单记录一下。

## 代码

```c++
#include <algorithm>
#include <iostream>
#include <vector>
using namespace std;
vector<int> ans;
vector<int> rec;
int index = 0;
void build(int r);
int main() {
    int n;
    cin >> n;
    ans.resize(n);
    rec.resize(n);
    for (int i = 0; i < n; i++) {
        cin >> rec[i];
    }
    sort(rec.begin(), rec.end());
    build(0);
    for (int i = 0; i < n; i++) {
        cout << ans[i];
        if (i != n - 1) {
            cout << " ";
        }
    }
}
void build(int r) {
    if (r >= rec.size()) {
        return;
    }
    build(2 * r + 1);
    ans[r] = rec[index++];
    build(2 * r + 2);
}
```
