---
title: PAT 1030 Travel Plan (30分) dfs+回溯
date: 2020-01-08 22:34:20
categories: 
- 算法
tags:
- PAT 
- 图论
---

用dfs求图上两点的最短路。

## 题目

A traveler’s map gives the distances between cities along the highways, together with the cost of each highway. Now you are supposed to write a program to help a traveler to decide the shortest path between his/her starting city and the destination. If such a shortest path is not unique, you are supposed to output the one with the minimum cost, which is guaranteed to be unique.

### Input Specification

Each input file contains one test case. Each case starts with a line containing 4 positive integers N, M, S, and D, where N (≤500) is the number of cities (and hence the cities are numbered from 0 to N−1); M is the number of highways; S and D are the starting and the destination cities, respectively. Then M lines follow, each provides the information of a highway, in the format:

```bash
City1 City2 Distance Cost
```

where the numbers are all integers no more than 500, and are separated by a space.

### Output Specification

For each test case, print in one line the cities along the shortest path from the starting point to the destination, followed by the total distance and the total cost of the path. The numbers must be separated by a space and there must be no extra space at the end of output.

### Sample Input

```bash
4 5 0 3
0 1 1 20
1 3 2 30
0 3 4 10
0 2 2 20
2 3 1 20
```

### Sample Output

```bash
0 2 3 3 40
```

## 题目大意

给定各个城市之间的距离和路费，求起点到终点的最短路，如果最短路不止一条，输出路费最短的。

## 解体思路

题目是求 weighted graph 的最短路，因为边带有除了距离外其他的权重，不符合三角不等式，用迪杰斯特拉也要找到所有最短路径后再回溯，过程繁琐。因此使用了深度优先遍历图，找到一条起点到终点的路径后，判断当前路径是否比已经找到的最短路更优，如果是则更新路径、路程和花费。dfs 用递归实现，return 发生在到达终点或当前节点的邻接节点都已访问（走不通了）。遍历起点的邻接表，依次对每一个邻接节点进行dfs。如果是连通图，这样可以遍历图的所有边。用 visited 数组保存在一条路径的搜索中的点是否被访问过，防止出现环路。回溯时把当前点标记为未访问，这意味着从邻接表中的下一个点扩展时，可以从这个点到终点。这样才能实现遍历时所有边都被访问，所以一定可以找到过路费最少的最短路。

可以剪枝，减去当前距离大于最短距离的部分路径，但不剪枝也不会超时。

## 代码

```c++
#include<iostream>
#include<vector>
#include<list>
using namespace std;
struct adnode{
    int to;
    int distace;
    int cost;
};
class graph{
    public:
    vector<list<adnode>> adjlist;
    vector<bool> visited;
    int start;
    int distinct;
    graph(int n,int s,int d){
        adjlist.resize(n);
        visited.resize(n);
        for (int i = 0; i < n;i++){
            visited[i] = false;
        }
        start = s;
        distinct = d;
        visited[s] = true;
    }
    void addEdge(int s,int d,int dis,int w){
        adnode a,b;
        a.to = d;
        a.distace = dis;
        a.cost = w;
        b.to = s;
        b.distace = dis;
        b.cost = w;
        adjlist[s].push_back(a);
        adjlist[d].push_back(b);
    }
    int min_distance=9999, min_cost=9999;
    int distace = 0,cost = 0;
    vector<int> route;
    vector<int> t_route;
    void dfs(int s){
        //剪枝
        if(distace>min_distance){
            return;
        }
        //到达终点
        if(s==distinct){
            if(distace<min_distance){
                min_distance = distace;
                min_cost = cost;
                route = t_route;
            }
            if(distace==min_distance){
                if(cost<min_cost){
                    min_cost = cost;
                    route = t_route;
                }
            }
            return;
        }
        for(auto i:adjlist[s]){
            if(visited[i.to]==false){
                visited[i.to] = true;
                distace = distace + i.distace;
                cost = cost + i.cost;
                t_route.push_back(i.to);
                dfs(i.to);
                //回溯
                t_route.pop_back();
                visited[i.to] = false;
                distace = distace - i.distace;
                cost = cost - i.cost;
            }
        }
        return;
    }
};
int main(){
    int n, m, start, distinction;
    cin >> n >> m>>start>>distinction;
    graph mygraph(n, start, distinction);
    for (int i = 0; i < m;i++){
        int s, d, dis, c;
        cin >> s >> d >> dis >> c;
        mygraph.addEdge(s, d, dis, c);
    }
    mygraph.dfs(start);
    cout << start;
    for (int i = 0; i < mygraph.route.size();i++){
        cout << " " << mygraph.route[i];
    }
    cout << " ";
    cout << mygraph.min_distance << " " << mygraph.min_cost;
}
```
