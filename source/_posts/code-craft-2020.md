---
title: 华为软件精英挑战赛2020初赛复盘
date: 2020-5-12 22:34:20
updated: 2020-5-12 22:34:20
categories: 
- 算法
tags: 
- IO优化
- Linux
- 递归
mathjax: true
---
> 刚刚结束的华为软挑初赛，运气不错地用非常native的方法进入了粤港澳赛区32强，趁现在还记得，在这里做个简单的总结

## 题目描述

通过金融风控的资金流水分析，可有效识别循环转账，辅助公安挖掘洗钱组织，帮助银行预防信用卡诈骗。基于给定的资金流水，检测并输出指定约束条件的所有循环转账，结果准确，用时最短者胜。

### 输入信息

输入为包含资金流水的文本文件，每一行代表一次资金交易记录，包含本端账号ID, 对端账号ID, 转账金额，用逗号隔开。
* 本端账号ID和对端账号ID为一个32位的无符号整数
* 转账金额为一个32位的无符号整数
* 转账记录最多为28万条
* 每个账号平均转账记录数< 10
* 账号A给账号B最多转账一次

### 输出信息

输出信息为一个文件，包含如下信息:

* 第一行输出：满足限制条件下的循环转账个数。
说明：数据集经过处理，会保证满足条件的循环转账个数小于300万。
* 第二行开始：输出所有满足限制条件的循环转账路径详情。

输出循环转账路径要按照指定排序策略进行排序：每条循环转账中，ID（ID转为无符号整数后）最小的第一个输出；总体按照循环转账路径长度升序排序；同一级别的路径长度下循环转账账号ID序列，按照字典序（ID转为无符号整数后）升序排序。

### 限制条件

循环转账的路径长度最小为3（包含3）最大为7（包含7），例如账户A给账户B转账，账户B给账户A转账，循环转账的路径长度为2，不满足循环转账条件。

## 数据处理

题目实际可以简化为在有向图上的循环问题。

首先是读入数据的部分，读取输入文件，将边的信息存入存入`rawGraph`邻接表，同时记录入边的入度。因为线上数据分布较为稀疏，我们先对rawGraph跑一遍拓扑然后进行拓扑排序，在稀疏图上，就可以剪掉很大一部分无效节点，后续搜索算法的解空间也就变小了。

拓扑排序用队列保存所有入度为0的点，在循环中将队首节点出队，然后将所有该节点可达的点的入度减一。重复这一过程，直到队列大小减为0。

```cpp
queue<int> qt;
sort(drec.begin(), drec.end());
char *buf = new char[30];
char *buf1 = new char[30];
for (auto &i : drec) {
    if (raw_indergree[i] == 0) {
        qt.push(i);
    }
    getreindexed[i] = cnt;
    getfactul[cnt] = i;
    sprintf(buf, "%d,", i);
    idsComma[cnt] = buf;
    sprintf(buf1, "%d\n", i);
    idsLine[cnt] = buf1;
    cnt++;
}
while (!qt.empty()) {
    int current = qt.front();
    qt.pop();
    int line_in_raw = get_raw_index[current];
    for (int i = 0; i < rawGraph[line_in_raw].size(); i++) {
        raw_indegree[rawGraph[line_in_raw][i].to]--;
        if (raw_indegree[rawGraph[line_in_raw][i].to] == 0) {
            qt.push(rawGraph[line_in_raw][i].to);
        }
    }
}
```

在拓扑排序结束后，遍历rawGraph,将入度不为0的点删去。建立我们在后续搜索中实际使用的图Graph和反向图rGraph.

## 搜索算法

一开始用的单向dfs