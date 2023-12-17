# 全源最短路算法 - Johnson | [阅读pdf](https://files.cnblogs.com/files/blogs/805728/document.zip?t=1701164005&download=true)

## 题面描述

给定一个 $n$ 个点，$m$ 条边的有向图。假设 $i$ 到 $j$ 的最短距离是 $d_{(i,j)}$，那么你需要输出 $n$ 行，第 $i$ 行输出 $\sum\limits_{j=1}^{n}d_{(i,j)}$。对于不连通的两个点 $i,j$，$d_{(i,j)}=10^9$。特别地，如果这个图当中有负环，你需要输出 $-1$。

## 算法简介

### 初步思考

首先我们想一想，全源负权最短路应该使用什么算法解决呢？🤔 当然是 Floyd 啦，可以在 $\mathcal O(n^3)$ 的时间复杂度之内求出任意两点之间的最短路。但是此题 $1\le n\le 3\times 10^3$，很明显会 TLE。😅

我们之前还学习过了很多的单源最短路算法，其实可以跑 $n$ 边这样的算法就行了。😉 比如 Bellman-Ford，跑 $n$ 遍的时间复杂度是 $\mathcal O(n^2m)$；还有我们熟知的更快最短路算法但其实大多数时候并没有那么快的 SPFA，最好 $\mathcal O(n^2+nm)$，但是最坏情况跟 Bellman-Ford 一样，是 $\mathcal O(n^2m)$ 的。可是，这些最短路算法都不能满足要求，那我们应该怎么去写呢？😞

### 算法思想

我们知道 Dijkstra 是可以在非常快的时间内求出最短路的，跑 $n$ 遍的时间复杂度为 $\mathcal O(nm\log_2 n)$，但是处理不了负边权的图。Bellman-Ford 好写，还可以处理负边权的图，但是时间复杂度非常高。那么我们如何选择呢？😌

💡 其实早就有人思考了这个问题，提出了对应的解决方案：

???+ quote "算法背景"
    Johnson 是一种求负边权图任意两点之间的距离的算法，该算法在 1977 年被 Donald B. Johnson 提出。

我们可以想出一种简单的标注边权的方法。对于任意一条边，我们将边权加上 $x$，然后求出的最短路假设长度是 $k$，我们就将答案减去 $x\times k$。但是这种标注边权的方法是错误的。因为由于加上了 $x$ 的关系，那么 Dijkstra 的优先队列就会依赖经过边的数量，但是很显然，经过的边的数量会影响到算法的处理。但是 Johnson 算法使用了一种全新的标注边权的方式。🔌

我们使用一个虚拟节点，编号可以为任意值。将虚拟节点向任意节点都连上一条边，权值为 $0$，然后从虚拟节点开始跑一边 Bellman-Ford。跑完之后，我们就能求出从任意一个点到 $i$ 的距离了。这时，你可能会说：哎呀，到 $i$ 的距离最短不就是直接从 $i$ 开始嘛。正权图确实是这样子的，但是这里是负权图。

跑完之后，我们就需要重新标注边权。假设第 $i$ 条边起始点为 $u_i$，终止点为 $v_i$，边权为 $w_i$，那么我们就将 $w_i$ 的值改为 $w_i+d_{(u_i)}-d_{(v_i)}$。🧮 最后，我们通过跑 $n$ 遍 Dijkstra 就能求出答案了。注意：求出的答案同样需要进行处理。

## 代码实现

### Bellman-Ford + 修改边权

首先我们可以轻松的写出一个 Bellman-Ford。这种算法特别简单，只需要枚举 $n$ 次，也就是松弛的次数，然后枚举 $m$ 条边，直接进行松弛操作就行了。算法时间复杂度是 $\mathcal O(nm)$ 的。注意我的代码，虽然是三重循环，但其实二三层循环加起来是 $\mathcal O(m)$ 的。

```cpp
for (ll i = 0; i <= n; i++) {              // 枚举 n 次松弛
  for (ll j = 0; j <= n; j++) {            // 枚举 n 个点
    for (auto k : e[j]) {                  // 枚举每条边
      d[k.to] = min(d[k.to], d[j] + k.v);  // 进行松弛操作
    }
  }
}
```

然后我们就需要判断负环。很简单，我们重新遍历这 $m$ 条边，若还可以进行松弛操作就代表出现了负环。因为我们知道最短路的最大长度是 $n-1$ 的，循环 $n$ 次是为了保险。有一个很好的思路，就是使用一个变量表示是否进行了松弛操作，如果没有了就退出循环。但是题目数据比较水，而且此处 Bellman-Ford 的复杂度还没有 $n$ 遍 Dijkstra 高，因此就没有进行优化。

```cpp
bool hasNegativeRing(vector<ll> &d) {  // d 表示距离
  for (ll i = 1; i <= m; i++) {        // 枚举 m 条边
    if (d[u[i]] + w[i] < d[v[i]]) {    // 如果还可以进行松弛操作
      return 1;                        // 返回有负环
    }
  }
  return 0;
}
```

接着就是修改边权的部分，照着上面的思路写就行了。

```cpp
void clearEdge() { fill(e, e + n + 1, vector<Node>()); }  // 清空所有的边

void build(vector<ll> &d) {                               // 重新建图
  clearEdge();                                            // 去掉之前的边
  for (ll i = 1; i <= m; i++) {                           // 枚举边
    e[u[i]].push_back({v[i], w[i] + d[u[i]] - d[v[i]]});  // 使用邻接表进行存边
  }
}
```

### Dijkstra + 统计答案

这个我们已经烂熟于心了，标准代码我都能倒背如流，让我背一遍 `)}v.i + ]to.to[...`……	

```cpp
vector<ll> dijkstra(ll s) {                            // 从 s 开始
  vector<ll> d(n + 1, 1e9), f(n + 1, 0);               // 距离数组和访问数组
  for (q.push({s, 0}), d[s] = 0; q.size(); q.pop()) {  // 加入队列、更改距离为 0；如果队列不为空；退出队列
    if (f[(t = q.top()).to]) {                         // 如果是访问过的元素
      continue;                                        // 重新循环
    }
    f[t.to] = 1;                                       // 打上标记
    for (auto i : e[t.to]) {                           // 枚举邻点
      if (d[t.to] + i.v < d[i.to]) {                   // 如果更优
        q.push({i.to, d[i.to] = d[t.to] + i.v});       // 更改并入队
      }
    }
  }
  return d;
}
```

然后我们就可以愉快地写出板子了。

## 最终代码

```cpp
#include <iostream>  // 输入 / 输出 流
#include <vector>    // 动态数组
#include <queue>     // 优先队列

using namespace std;   // 使用 std 名字空间
using ll = long long;  // 记得开 long long

const ll kMaxN = 6e3 + 5;  // 点 / 边 的最大数量

struct Node {                                            // 状态的结构体
  ll to, v;                                              // 终止节点和边权

  friend bool operator<(const Node &a, const Node &b) {  // 重载小于运算符
    return a.v > b.v;                                    // 小根堆，因此填大于号
  }
} t;                                                     // 临时变量，Dijkstra 里面用的

ll u[kMaxN], v[kMaxN], w[kMaxN], n, m;  // 边、n 和 m
vector<Node> e[kMaxN];                  // 邻接表
priority_queue<Node> q;                 // 优先队列

void read() {                         // 读入函数
  cin >> n >> m;                      // 输入 n,m
  for (ll i = 1; i <= m; i++) {       // 输入 m 条边
    cin >> u[i] >> v[i] >> w[i];      // 需要使用数组记录
    e[u[i]].push_back({v[i], w[i]});  // 存入邻接表
  }
}

void createVisualStartPoint() {  // 创建虚拟节点，编号为 0
  for (ll i = 1; i <= n; i++) {  // 枚举 n 个点
    e[0].push_back({i, 0});      // 每个点都建一条长度为 0 的边
  }
}

bool hasNegativeRing(vector<ll> &d) {  // 判断是否有负环
  for (ll i = 1; i <= m; i++) {        // 枚举 m 条边
    if (d[u[i]] + w[i] < d[v[i]]) {    // 如果还可以进行松弛操作
      return 1;                        // 有负环
    }
  }
  return 0;                            // 没有负环
}

void clearEdge() { fill(e, e + n + 1, vector<Node>()); }  // 清空邻接表，给每一个 vector 都附一个空的值

void build(vector<ll> &d) {                               // 重新建图
  clearEdge();                                            // 清空
  for (ll i = 1; i <= m; i++) {                           // 枚举之前的 m 条边
    e[u[i]].push_back({v[i], w[i] + d[u[i]] - d[v[i]]});  // 更改边权
  }
}

vector<ll> bellmanFord(ll s) {               // 从虚拟节点开始跑最短路
  vector<ll> d(n + 1, 1e9);                  // 距离数组，初始为 10^9
  d[s] = 0;                                  // 标记起始点
  for (ll i = 0; i <= n; i++) {              // 枚举 n 次松弛
    for (ll j = 0; j <= n; j++) {            // 枚举 n 个点
      for (auto k : e[j]) {                  // 枚举边
        d[k.to] = min(d[k.to], d[j] + k.v);  // 进行松弛操作
      }
    }
  }
  if (hasNegativeRing(d)) {                  // 如果有负环
    exit((cout << "-1\n", 0));               // 输出 -1 并退出
  }
  build(d);                                  // 重新建图
  return d;                                  // 返回距离数组
}

vector<ll> dijkstra(ll s) {                            // 从 s 开始
  vector<ll> d(n + 1, 1e9), f(n + 1, 0);               // 距离数组和访问数组
  for (q.push({s, 0}), d[s] = 0; q.size(); q.pop()) {  // 加入队列、更改距离为 0；如果队列不为空；退出队列
    if (f[(t = q.top()).to]) {                         // 如果是访问过的元素
      continue;                                        // 重新循环
    }
    f[t.to] = 1;                                       // 打上标记
    for (auto i : e[t.to]) {                           // 枚举邻点
      if (d[t.to] + i.v < d[i.to]) {                   // 如果更优
        q.push({i.to, d[i.to] = d[t.to] + i.v});       // 更改并入队
      }
    }
  }
  return d;                                            // 返回距离数组
}

void solve() {                               // 解
  createVisualStartPoint();                  // 建立虚拟
  auto d1 = bellmanFord(0);                  // 跑 Bellman-Ford，存进 d1
  for (int i = 1; i <= n; i++) {             // 枚举 n 个点
    auto d2 = dijkstra(i);                   // 跑 Dijkstra，存进 d2
    ll ans = 0;                              // 答案
    for (int j = 1; j <= n; j++) {           // 枚举其他的点
      if (d2[j] == 1e9) {                    // 如果不能到达
        ans += j * 1e9;                      // 直接累加
      } else {                               // 否则
        ans += j * (d2[j] + d1[j] - d1[i]);  // 距离就要加上 d1[j] 减去 d1[i]
      }
    }
    cout << ans << '\n';                     // 输出答案
  }
}

int main() {  // 主函数
  read();     // 输入
  solve();    // 输出
  return 0;   // 好习惯
}
```