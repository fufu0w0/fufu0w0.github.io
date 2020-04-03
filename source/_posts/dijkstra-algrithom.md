---
title: Dijkstra单源点最短路径算法详解
date: 2019-10-11 22:34:20
categories: 
- 算法
tags: 
- 图论
- 最短路
mathjax: true
---

## 概述

dijkstra算法是典型的单源点最短路的算法，可以求取图中各点到初始点的最短路，适用于边的权值非负的图（有向或无向）。

将图的点集分为两个集合，从源点到该点的最短路径已确定的点的集合$S$，和未确定到源点最短路径的点的集合R。从集合R中选取只经过在集合S中的点到源点距离最短的点加入集合S，并更新此时在集合R中的点到源点的最小距离，更新规则为

```bash
dis[v] =  (dis[v] > dis[u] + g.map[u][v]) ? (dis[u] + g.map[u][v]) : dis[v];
```

在集合S加入新的点以后，比较在集合R中的点经过这个点到源点的距离和不经过这个点到源点的距离，选取较小者作为此时该点距离源点的距离，这个过程叫松弛（relaxtion)操作。

tips : 将最短距离减小的过程被叫做’relaxation’是因为各点到源点最短路相当于是被“拉伸”的，因此更新最短路的操作就是在使最短路径“松弛”。

The notion of “relaxation” comes from an analogy between the estimate of theshortest path and the length of a helical tension spring, which is notdesigned for compression. Initially, the cost of the shortest path is anoverestimate, likened to a stretched out spring. As shorter paths are found,the estimated cost is lowered, and the spring is relaxed. Eventually, theshortest path, if one exists, is found and the spring has been relaxed to itsresting length

重复从集合$R$取点，加入集合$S$的过程，直到所有的点都加入到了集合$S$中，此时就求得了源点到其余各点的最小距离，算法结束。

综上，dijkstra算法实际就是包含了两个对集合的操作操作：

1.包括从$R$中取出到源点最小的点加入到集合$S$中;

2.更新$R$中剩下的点到源点可能的最短距离。

## 实现

### 遍历实现

遍历是Dijkstra算法最朴素的实现方式，过程描述如下：

遍历所有点，从中找出未加入集合S且距离源点最短的点加入集合S中。

遍历所有点，从中找出未加入集合S的点更新其到远点的最短距离。

重复1和2，循环直到所有可达的点都加入到集合S.

时间复杂度是$O(n^2)$，$n$为图上点的个数。

c++实现如下：

```c++
struct mygragh{
    int map[100][100];//邻接矩阵存储
    int n;
};
void dijkstra(mygragh g,int dis[],int pre[],int v0){
    int n = g.n;
    vector<bool> visit(n, false);
    for (int i = 0; i < n;i++){
        if(g.map[v0][i]!=0){
            dis[i] = g.map[v0][i];
            pre[i] = v0;
        }
        else{
            dis[i] = INT_MAX;
            pre[i] = -1;
        }
    }
    dis[v0] = 0;
    pre[v0] = v0;
    visit[v0]=true;
    for(int i=0;i<n;i++){
        int min = INT_MAX, u = -1;
        for (int j = 0; j < n;j++){
            if(visit[j]==false&&dis[j]<min){
                min = dis[j];
                u = j;
            }
        }
        if(u==-1){
            break;
        }
        visit[u] = true;//如果要找某个点到源点的最短距离，我们可以在选取的最短距离的点是目标点时跳出循环。
        for (int j = 0; j < n;j++){
            if(visit[j]==false&&dis[j]>dis[u]+g.map[u][j]&&g.map[u][j]!=0){
                dis[j] = dis[u] + g.map[u][j];
                pre[j] = u;
            }
        }
    }
}
```

### 小顶堆实现

通过邻接矩阵和堆来实现dijkstra最短路径算法。过程描述如下：

初始化，建堆，节点的权值（是当前从源点到该节点最短的距离）初始化为INT_MAX。

取顶，取堆顶元素（当前不在最短路径树中距离源点距离最小的节点）加入最短路径树，交换堆顶和堆底，堆化（从当前节点开始下滤）恢复堆序性。

松弛操作，将点v加入最短路径树以后，更新所有和v相邻的点到源点的距离,通过decresekey，根据pos数组找到点在堆中对应的位置，减小权值，并进行上滤操作。（权值只会减小)改变堆中某点的w之后调整他在堆中的位置（因为w只会减小，v点以下堆序性不会被破坏，因此只需要执行上滤操作即可恢复整体堆序性).

重复2,3，直到堆中元素全部取出。

时间复杂度为$O(e*logv)$，$e$是图上边的条数，$v$是图上点的个数。在这里用的是二叉堆实现，如果用Fibonacci堆，时间耗费还可以更低。

```bash
预备知识：堆
1.建堆，从第一个有孩子的节点开始进行下滤，直到根节点
2.取顶，交换，下滤(在有的教材中也叫堆化)
3.插入，在堆底插入，从它开始上滤。
4.上滤：该节点以上的堆除该节点以外皆有序，以该节点为根子树均有序，上滤一次即可恢复堆序性。
5.下滤：该节点的左右子树均有序，下滤一次，以该节点为根的子树恢复堆序性。
```

c++实现如下

```c++
struct adListNode{
    int weight;
    int id;
    adListNode *next;
};
struct adList{
    adListNode *head;
};
struct mygragh{
    int n;//点的个数
    adList *array;//存储边的邻接表
};
struct adListNode* createAdListNode(int v,int weight){
    struct adListNode *node = (struct adListNode *)malloc(sizeof(struct adListNode));
    node->id = v;
    node->weight = weight;
    node->next = NULL;
    return node;
}
struct mygragh* creatMyGraph(int n){
    struct mygragh *graph = (struct mygragh *)malloc(sizeof(struct mygragh));
    graph->array = (struct adList *)malloc(n * sizeof(struct adList));
    //array是元素类型为adList的数组，有几个点就有几个adList所以它分配空间大小是n*sizeof(struct adList);
    graph->n = n;
    for (int i = 0; i < n;i++){
        graph->array[i].head = NULL;
    }
        return graph;
}
void addEdge(mygragh *g,int s,int d,int w){
    adListNode *node = createAdListNode(d, w);
    node->next = g->array[s].head;
    g->array[s].head = node;
    //无向图
    node = createAdListNode(s, w);
    node->next = g->array[d].head;
    g->array[d].head = node;
}
struct heapNode{
    int id;
    int w;
};
struct miniHeap{
    int capasity;
    int size;
    int *pos;//存储图的点在堆中的位置
    heapNode **array;
};
struct heapNode* createheapNode(int id,int weight){
    heapNode *node = (struct heapNode *)malloc(sizeof(struct heapNode));
    node->id = id;
    node->w = weight;
    return node;
}
struct miniHeap* createMiniHeap(int n){
    struct miniHeap *heap = (struct miniHeap *)malloc(sizeof(struct miniHeap));
    heap->pos = (int *)malloc(n * sizeof(int));
    heap->capasity = n;
    heap->size = 0;
    heap->array = (struct heapNode **)malloc(n * sizeof(struct heapNode *));
    return heap;
}
void swap(heapNode **a,heapNode **b){
    heapNode *t;
    t = *b; //因为he.array数组中的元素是指向heapNode的指针，要在函数中实现元素的交换，要要传入指针的地址。和swap(int *a,int *b)是一样的，不然传入函数以后是在内存堆栈上重新开辟了一个空间，然后把这两个指针复制到那个空间中，交换也只在这个函数的空间中进行，而实际想要交换的两个指向堆中节点的指针没有交换。
    *b = *a;
    *a = t;
}
void heapify(struct miniHeap* h,int inx){
    //下滤
    int left, right, mini;
    mini = inx;
    left = inx * 2 + 1;
    right = inx * 2 + 2;
    if(left<h->size&&((h->array[left]->w)<(h->array[mini]->w))){
        mini = left;
    }
    if(right<h->size&&((h->array[right]->w)<(h->array[mini]->w))){
        mini = right;
    }
    if(inx!=mini){
        struct heapNode *p = h->array[inx];
        struct heapNode *minist = h->array[mini];
        h->pos[p->id] = mini;
        h->pos[minist->id] = inx;
        swap(&h->array[inx], &h->array[mini]);//交换堆中的节点。节点向下一层
        heapify(h, mini);//递归的调用heapify,每次向下一层直到该节点到达它在堆中应该的位置
    }
}struct heapNode* getTop(miniHeap *h){
    if(h->size==0){
        return NULL;
    }
    struct heapNode *node = h->array[0];
    h->array[0] = h->array[h->size - 1];
    h->pos[h->array[0]->id] = 0;
    h->pos[node->id] = h->size - 1;//把该点对应在堆中的坐标放到堆末位，再把size--，就相当于这个节点已经不在堆中
    h->size--;
    heapify(h, 0);
    return node;
}
void decreasekey(miniHeap *h,int v,int w){
    //上滤
    if((h->pos[v])<(h->size)){//判断该节点是否还在堆里。
        h->array[h->pos[v]]->w = w;
        int a = h->pos[v];
        int p = (a - 1) / 2;
        while(a>0){
            if(h->array[a]->w>h->array[p]->w){
                break;
            }
            h->pos[h->array[a]->id] = p;
            h->pos[h->array[p]->id] = a;
            swap(&h->array[a], &h->array[p]);
            a = p;
            p = (a - 1) / 2;
        }
    }
}

bool inHeap(miniHeap *h,int v){
    if(h->pos[v]<h->size){
        return true;
    }
    return false;
} 
void dijkstra(mygragh *g, int *dis, int *pre, int v0){
    int n = g->n;
    //cout << n << "\n";
    struct miniHeap *h = createMiniHeap(n);
    for (int i = 0; i < n;i++){
        dis[i] = INT_MAX;
        h->pos[i] = i;
        struct heapNode *node = createheapNode(i, INT_MAX);//初始化所有元素值相等，为INT_MAX,堆不用建，一开始就是好的。
        h->array[i] = node;
        pre[i] = v0;
        h->size++;
    }
    dis[v0] = 0;
    pre[v0] = v0;
    decreasekey(h, v0, 0);
    while(h->size>0){
        struct heapNode *node = getTop(h);
        struct adListNode *a = g->array[node->id].head;
        while(a!=NULL){
            //cout<<"route : " << node->id << " " << a->id << "\n";
            if((dis[node->id]+a->weight)<(dis[a->id])&&inHeap(h,a->id)){
                dis[a->id] = dis[node->id] + a->weight;
                decreasekey(h, a->id, dis[a->id]);
                pre[a->id] = node->id;
            }
            a = a->next;
        }
    }
}
```

实现细节

堆中维护一个pos数组保存节点到节点在堆中所在位置，在松弛操作中需要知道这个位置以更新他们的权值。每次交换堆中的节点后，pos数组中保存点的位置也要交换。

在函数中交换指向节点的指针（被交换的对象是地址），函数调用时要交换两个指针，函数的参数应该是指针的地址，上面*p=h->array[inx],是把这个指针复制了一分给p，而p和h->array[inx]的值相同，都是指向i指向堆中的节点的值针，即在他们的值是节点的地址。但是他们不是同一个，他们有不同的地址，如果交换swap(&p,&mini),是p和mini这两个变量中存放的地址发生交换，而不会交换h->array[inx]和h->array[mini]中存放的地址，而数组中存放的地址，才是我们实际需要交换的，所以不可以用swap(&p,&mini).换句话说，这两个指针有相同的值，但是他们的地址是不一样的。

## 优先级队列实现

第三种实现方式是通过使用STL库中的优先级队列实现。priority_queue不支持降低节点权值的操作，要解决此问题，我们通过插入与该节点同名但值更小的另一个节点来实现decreasekey()操作。

在优先队列中有多个相同节点，它们有不同的权值。每当一个节点的到源点的最短路径减小时，就在priority_queue中添加一个同名但值更小的节点。即使存在多个同名节点，我们也只考虑到源点的距离最小最小的节点，而忽略其他节点。

时间复杂度是$O(ELogV)$

c++实现如下：

```c++

struct queueNode{
    int v;
    int w;
    bool operator < (queueNode a) const{
        if(w==a.w){
            return v > a.v;
        }
        return w > a.w;
    }
};
struct queueNode createQueueNode(int v,int w){
    struct queueNode qnode;
    qnode.v = v;
    qnode.w = w;
    return qnode;
}
void dijkstra(mygragh *g, int *dis, int *pre, int v0){
    int n = g->n;
    int size = n;
    cout << n << size << '\n';
    priority_queue<queueNode> pq;
    for (int i = 0; i < n;i++){
        dis[i]=INT_MAX;
        pre[i] = v0;
        queueNode pno;
        pno=createQueueNode(i, INT_MAX);
        pq.push(pno);
    }
    dis[v0] = 0;
    queueNode pno = createQueueNode(v0, 0);
    pq.push(pno);
    while(!pq.empty()){
        pno = pq.top();
        cout << "getto :  " << "   " << pno.v << "  " << pno.w << "\n";
        pq.pop();
        adListNode *adnode = g->array[pno.v].head;
        while(adnode!=NULL){
            if(adnode->weight+dis[pno.v]<dis[adnode->id]){
                dis[adnode->id] = adnode->weight+dis[pno.v];
                queueNode minode = createQueueNode(adnode->id, adnode->weight+dis[pno.v]);
                pq.push(minode);//relax,因为不能改变优先级队列中已有的节点的权值，因此插入一个同名权值更小的节点到优先级队列里啦
                pre[adnode->id] = pno.v;
            }
            adnode = adnode->next;
        }
    }
}
//图用邻接矩阵存储，和堆实现中的代码一样。
```

## 总结

Dijkstra算法是Dijkstra在咖啡馆休息时想出的，他没有想到这个 20 分钟的发明会成为他最著名的成就之一，以他的名字命名，还因此获得了计算机科学领域的最高奖项图灵奖。作为计算单源点最短路的算法广泛应用在很多领域（当然我知道的比赛题目之外的应用只有路由算法）。之前数模校赛地时候就用上了Dijkstra算法，上学期学算法的时候觉得最简单的就是贪心算法，但自己实现了一遍才发现之前有很多没有学懂的地方。

说点题外话作为这篇博客的结尾吧，到现在我也记不住Dijkstra的拼写，每次打他的名字都是打完’Dijie’发现想不起来后面的字母是什么才老老实实翻到前面ctrl c，因为Dijkstra是荷兰人，名字不符合英语发音规则。

## 参考

[Dijkstra’s Algorithm for Adjacency List Representation](https://www.geeksforgeeks.org/dijkstras-algorithm-for-adjacency-list-representation-greedy-algo-8/)

[Dijkstra’s Shortest Path Algorithm using priority_queue of STL](https://www.geeksforgeeks.org/dijkstras-shortest-path-algorithm-using-priority_queue-stl/)