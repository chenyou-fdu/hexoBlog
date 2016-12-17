title: "图类问题解法"
date: 2015-05-06 22:50:24
tags: [算法]
category: 算法
mathjax: true
---

根据TopCoder上[Introduction to Graphs and Their Data Structures](https://www.topcoder.com/community/data-science/data-science-tutorials/introduction-to-graphs-and-their-data-structures-section-1/)这篇Tutorial以及平时自己的一些实现心得进行笔记。

## 图的数据结构
邻接矩阵表示使用`vector< vector<int> >`。这里注意，在C++11之前，`<<`不能连写。
邻接表的实现需要自定义一个结点类：
```cpp
class Edge { //how to store adjancy list with weight
public:
    int target;
    int weight;
    Edge(int t, int w) :target(t), weight(w){};
};
```
然后使用一个`vector<vector<Edge>> adjList`来存储这个邻接表。这样的话，很容易可以查找到每个结点的出度，也就是每个节点的`size()`。
当边数远小于结点数时，使用邻接表要更节省空间，而且操作更加快速，不需要遍历每一行。
<!-- more  -->

## 图的算法
### 深度优先遍历
从某一个节点开始，递归的遍历其所有子节点，当某一个节点的子节点都遍历了，就变成黑，被遍历的变成灰；时间$O(V+E)$。
DFS实现方式：（递归访问所用节点）

	1. 访问初始顶点`v`并标记顶点`v`已访问；
	2. 查找顶点`v`的第一个邻接顶点w；
	3. 若顶点`v`的邻接顶点w存在，则继续执行；否则回溯到`v`，再找`v`的另外一个未被访问过的邻接点；
	4. 若顶点`w`尚未被访问，则继续访问顶点`w`并标记顶点`w`为已访问；
	5. 继续查找顶点`w`的下一个邻接顶点wi，重复直到都被访问；
 DFS的一些技巧：

	1.  递归实现更加简单，使用循环实现也可，要使用堆栈，每次把所有相连节点都入栈，注意不管访问过没有都入，在出栈时进行判断；
	2.  边权重、节点权重的比较可以用一个参数来记录，每次递归调用函数时，加上变化量就好，然后用一个全局的变量记录最大（最小）值；
	3.  若有回朔，可以在递归回朔后把是否访问（`vis数组`）还原；

### 广度优先遍历
从某一个节点开始，遍历与其相连的所有节点，然后将该节点设为黑，被遍历的节点设为灰，重复这个过程，直到所有的节点都是黑色，注意遍历的先后顺序，用一个`FIFO`表来记录；时间$O(V+E)$。
BFS实现方式：

	1.  顶点`v`入队列；
	2.  当队列非空时则继续执行步骤3；
	3.  出队列取得队头顶点`v`；访问顶点v并标记顶点v被访问；
	4.  查找顶点`v`的第一个邻接节点`col`。
	5.  若v的邻接顶点`col`是未被访问过的，则`col`入队列（入队列后需要标记为访问过）。
	6.  继续查找顶点`v`的另一个新的邻接顶点`col`，转到步骤5。直到顶点`v`的所有未被访问过的邻接处理完。转到步骤2。

### Dijkstra算法
Dijkstra算法是一种单源最短路径算法。
算法过程：

	1.  从源点开始，每次添加距离当前结点最近的一个结点进入路径。
	2.  然后更新其他结点到该路径的最短距离，与Prim不同的是，是新选入结点与其他结点距离加上源点到新选入结点距离，再于之前值比较，得到最后的距离。
	3.  直到所有结点都被遍历，算法停止。
如果使用最小堆实现的优先级队列，时间复杂度为$O(ElgV)$。使用斐波那契堆实现的话，时间复杂度为$O(VlgV+E)$。
普通实现（使用邻接矩阵，优先级队列和邻接表）：
```cpp
void DijsktraAdjMat(int adjMat[][20], int s, int len)
{
    int *isVisited = new int[len];
    int *dist = new int[len];
    for(int i = 0; i < len; ++i)
    {
        dist[i] = adjMat[s][i];
        isVisited[i] = 0;
    }
    isVisited[s] = 1;
    dist[0] = 0;
    cout << s << " ";
    for(int i = 1; i < len; ++i)
    {
        int tempMin = maxValue;
        int tempIndex = -1;
        for(int j = 0; j < len; ++j)
        {
            if(!isVisited[j] && tempMin > dist[j])
            {
                tempMin = dist[j];
                tempIndex = j;
            }
        }
        isVisited[tempIndex] = 1;
        cout << tempIndex << " ";
        for(int j = 0; j < len; ++j)
        {
            if(adjMat[tempIndex][j] != maxValue)
            {
                int tempDist = adjMat[tempIndex][j] + dist[tempIndex]; //get to node j through tempIndex
                if(tempDist < dist[j])
                    dist[j] = tempDist;
            }
        }
    }
    cout << endl;
    delete[] isVisited;
    delete[] dist;
}
class Edge { //how to store adjancy list with weight
public:
    int target;
    int weight;
    Edge(int t, int w) :target(t), weight(w){};
};
void DijsktraAdjList(vector<vector<Edge>> adjList, int root)
{
    int vNum = (int)adjList.size();
    vector<int> isVisited = vector<int>(vNum, 0);
    vector<int> dist = vector<int>(vNum, maxValue);
    dist[root] = 0;
    for (int i = 0; i < vNum; i++)
    {
        int tempIndex = -1;
        int tempMin = maxValue;
        for (int j = 0; j < vNum; ++j)
        {
            if (!isVisited[j] && dist[j] < tempMin)
            {
                tempIndex = j;
                tempMin = dist[j];
            }
        }
        isVisited[tempIndex] = 1;
        cout << tempIndex << " ";
        for (int j = 0; j < adjList[tempIndex].size(); ++j) 
        {
            int target = adjList[tempIndex][j].target;
            int weight = adjList[tempIndex][j].weight;
            if (dist[target] > weight + dist[tempIndex])
                dist[target] = weight + dist[tempIndex];
        }
    }
    cout << endl;
}
```

优先级队列实现（使用邻接表）：
```cpp
void DijsktraPriority(vector<vector<Edge>> adjList, int root)
{
    int vNum = (int)adjList.size();
    vector<int> isVisited = vector<int>(vNum, 0);
    vector<int> dist = vector<int>(vNum, maxValue);
    auto EdgeGreater = [](const Edge& e1, const Edge& e2){return e1.weight > e2.weight; };
    priority_queue < Edge, vector<Edge>, decltype(EdgeGreater)> distQueue;
    dist[root] = 0;
    distQueue.push(Edge(0, 0));
    while (!distQueue.empty())
    {
        Edge tempMin = distQueue.top();
        distQueue.pop();
        int tempMinTarget = tempMin.target;
        if (isVisited[tempMinTarget]) continue;
        cout << tempMinTarget << " ";
        isVisited[tempMinTarget] = 1;
        for (int i = 0; i < adjList[tempMinTarget].size(); ++i)
        {
            int target = adjList[tempMinTarget][i].target;
            int weight = adjList[tempMinTarget][i].weight;
            if (dist[target] > weight + dist[tempMinTarget])
            {
                dist[target] = weight + dist[tempMinTarget];
                distQueue.push(Edge(target, dist[target])); //push the new establish min link into queue
            }
        }
    }
    cout << endl;
}
```

### Prim算法
Prim算法是一种针对无向图的最小生成树算法。
算法过程：

	1.  从源点开始，每次添加距离Cut最近的一个节点进入这个Cut；
	2.  要一个表，记录图中每个节点到这个Cut的最短距离，然后每次添加了新的节点进入Cut时，需要对这个表进行更新；
    3.  刚添加进的那个节点，表中距离设为0，再把该节点的相邻的节点距离，与表中之前记录距离比较，选择小的那个；
	4.  这个表如果使用优先队列实现的话，效率好些，可以用`STL库`中的`priority_queue`或者自己实现一个最小堆；
普通实现（使用邻接矩阵）：
```cpp
void Prim(int adjMat[][20], int source, int len)
{
    int *dist = new int[len];
    int *isVisited = new int[len];
    vector<int> chosenNode;
    for (int i = 0; i < len; ++i)
    {
        //if (adjMat[source][i] != maxValue) //maxvalue in adjMat means no path between these two node
        dist[i] = adjMat[source][i];
        //else
        //    dist[i] = maxValue;
        isVisited[i] = 0;
    }
    dist[source] = 0;
    isVisited[source] = 1;
    chosenNode.push_back(source);
    for (int i = 1; i < len; ++i)
    {
        int tempMin = maxValue;
        int tempIndex = -1;
        for (int j = 0; j < len; ++j)
        {
            if (!isVisited[j] && tempMin > dist[j])
            {
                tempMin = dist[j];
                tempIndex = j;
            }
        }
        chosenNode.push_back(tempIndex);
        isVisited[tempIndex] = 1;
        for (int j = 0; j < len; ++j)
            if (adjMat[tempIndex][j] < dist[j]) //chosen node will be 0, which is small enough to be kept
                dist[j] = adjMat[tempIndex][j];
    }
    delete[] dist;
    delete[] isVisited;
    for (int i = 0; i < len; ++i)
        cout << chosenNode[i] << " ";
    cout << endl;
}
```

### Kruskal算法
Kruskal算法算法是一种针对无向图的最小生成树算法。
算法过程：

	1.  实现自定义`Edge`类，包含首位两个节点和边权重作为类成员，再把所有的边按权重大小进行排序，存入数组；
	2.  在边的数组中由小到大进行选择，选择的边要确保不会构成环路；
    3.  这样就要使用一个`VertexRoot`数组，记录每个节点所属的Cut，保证不把连接属于同一个Cut的两个节点的边选入；
	4.  最开始每个节点都属于自己这个Cut，所以`VertexRoot`的初值是其自己；
    5.  然后每次选择一条边时，就把这两个Cut连接在一起，这需要循环遍历来修改`VertexRoot`的值，保证两个Cut里所有节点在VertexRoot存的值一样；
	6.  处理完边的数组里所有的边，就产生了一个最小生成树；
算法实现：
```cpp
class Edge{
public:
    Edge(int s, int e, int v) : start(s), end(e), value(v){}
    int start, end, value;
};
bool cmpEdge(const Edge &n1, const Edge &n2)
{
    return n1.value < n2.value;
}
void Kruskal(vector<Edge> edgeVec, int len)
{
    sort(edgeVec.begin(), edgeVec.end(), cmpEdge);
    int *nodeRoot = new int[len];
    for (int i = 0; i < len; i++)
    {
        nodeRoot[i] = i;//save the root of every node, to ensure that no cycle is contained
    }
    for (int i = 0; i < edgeVec.size(); i++)
    {
        Edge tempEdge = edgeVec[i];
        if (nodeRoot[tempEdge.start] != nodeRoot[tempEdge.end])
        {
            int temp = nodeRoot[tempEdge.end]; 
            //keep in mind that nodeRoot[tempEdge.end] could be changed during the loop below,
            //so it's necessary to save the value into some temp var for compare
            for (int j = 0; j < len; ++j)
            {
                if (nodeRoot[j] == temp)
                    nodeRoot[j] = nodeRoot[tempEdge.start];
                else
                    continue;
            }
            cout << tempEdge.start << "---" << tempEdge.end << " ";
        }
    }
    cout << endl;
}
```

### Floyd-Washall 算法
Floyd-Washall算法可以计算出一个图内部所有结点与所有结点间的最短路径，而且可以计算负边的情况。
这种算法的时间复杂度是$O(n^3)$，大概的代码如下：
```cpp
for (k = 1 to n)
    for (i = 1 to n)
        for (j = 1 to n)
            adj[i][j] = min(adj[i][j], adj[i][k] + adj[k][j]);
```

### 性能分析与比较
Kruskal算法中的不相交集合数据结构要是使用按秩结合和路径压缩的启发式方法，来实现不相交集合森林的话，效果最好，总时间为$O(ElgV)$；而Prim算法要是使用二叉最小堆来实现优先队列的话，时间也为$O(ElgV)$，要是使用斐波那契堆的话，可以把时间改进为$O(E+VlgV)$（这里内容详见算法导论23章）。

如果是非常稠密的图（边数超过节点数很多），那么使用Prim算法更好，而当图很正常的话，使用Kruskal算法更好，因为数据结构更简单。

Prim算法和Dijkstra算法的区别就在于，Prim算法求得是如何把这些节点连起来，而Dijkstra算法求得是如何从源节点到目标节点。两者的实现都需要维持一个数组记录距离，Prim算法的数组表示的是其他未访问结点到这个Cut的最短距离，而Dijkstra算法的数组表示的则是从源点出发，经过当前结点到其他结点的最短距离。

### 其他
参考[TopCoder Tutorial Introduction to Graphs and Their Data Structures](https://www.topcoder.com/community/data-science/data-science-tutorials/introduction-to-graphs-and-their-data-structures-section-1/)

参考[演算法笔记-Path](http://www.csie.ntnu.edu.tw/~u91029/Path.html#1)。
