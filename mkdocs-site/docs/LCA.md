## 概述

### 定义

对于 $x$ 和 $y$ 两个结点，我们需要找到一个节点 $z$，使得 $z$ 既是 $x$ 的祖先也是 $y$ 的祖先，并且 $z$ 离 $x$ 和 $y$ 的距离最近。此时，结点 $z$ 就是 $x$ 和 $y$ 的最近公共祖先（Least Common Ancestor, LCA）。值得注意的是，我们可以看作结点本身也是自己的祖先。

### 尝试求解

首先由浅至深找到两个结点的祖先列表，然后再由深至浅找到最近的公共祖先结点。值得注意的是，这种方法虽然是暴力，但是在二叉树当中是可以达到 $\mathcal O(\log_2 n)$ 的速度的，但是这里并不一定是二叉树，也就是说当这颗树退化到一条链的时候时，这种算法可能达到 $\mathcal O(n)$ 甚至 $\mathcal O(n^2)$。

### 尝试改进

这种方法可能会造成两个结点的祖先列表长度不一样，给我们匹配时带来麻烦。因此，我们可以跑一遍 dfs 求出任意节点离根结点的距离，然后在求解时先将深度较大的结点向上提，然后两个结点同时向上查找最近的公共祖先。这样子就省去了匹配的时间，但是时间复杂度最高还是有可能会达到 $\mathcal O(n)$，不符合期望。

## 算法

### 倍增思想

#### 优化

我们可以跑一边树上 dp，设 $dp_{(i,j)}$ 表示为结点 $i$ 往上 $2^j$ 个祖先的编号，那么我们就可以使用倍增算法在 $\mathcal O(\log_2 n)$ 的时间复杂度之内将两个结点的层级提升到同一层，接下来就是优化枚举公共节点了。

因为有单调性，也就是往上上升的高度越大，那么成为公共祖先的概率就越高，因此我们同样使用倍增思想，那么也可以在 $\mathcal O(\log_2 n)$ 的时间之内求出两个结点的最近公共祖先，注意这里仍然要使用之前的 $dp$ 数组。

#### 代码

```cpp
#include <iostream>
#include <vector>

using namespace std;

const int kMaxN = 5e5 + 5;

int dp[kMaxN][25], d[kMaxN], lg[kMaxN], n, m, s;
vector<int> e[kMaxN];

void dfs(int x, int f) {
  dp[x][0] = f, d[x] = d[f] + 1;         // 更新 dp 数组和距离数组
  for (int i = 1; i <= lg[d[x]]; i++) {  // 枚举往前 2^i 的祖先
    dp[x][i] = dp[dp[x][i - 1]][i - 1];  // 计算
  }
  for (int i : e[x]) {         // 枚举邻点
    f != i && (dfs(i, x), 0);  // 递归处理
  }
}

int LCA(int x, int y) {
  for (d[x] < d[y] && (x ^= y, y ^= x, x ^= y); d[x] > d[y]; x = dp[x][lg[d[x] - d[y]] - 1]) { }
  // 首先保持 x > y，然后进行倍增，直到 d[x] = d[y]
  if (x == y) {  // 如果已经等于了
    return x;    // 直接返回 x
  }
  for (int k = lg[d[x]] - 1; k >= 0; k--) {                // 往上做差分
    dp[x][k] != dp[y][k] && (x = dp[x][k], y = dp[y][k]);  // 取最优值
  }
  return dp[x][0];  // 记得返回上面一个数
}

int main() {
  cin >> n >> m >> s;
  for (int i = 1, u, v; i < n; i++) {
    cin >> u >> v;
    e[u].push_back(v), e[v].push_back(u);
  }
  for (int i = 1; i <= n; lg[i] = lg[i - 1] + (1 << lg[i - 1] == i), i++) { }
  dfs(s, 0);
  for (int i = 1, x, y; i <= m; i++) {
    cin >> x >> y;
    cout << LCA(x, y) << '\n';
  }
  return 0;
}
```

### Tarjan

#### 思路

这是一种离线算法。我们可以将一个结点的所有子树合并到这个结点内，然后当要查询这个子树内的任意两个结点的最近公共祖先的时候，那么这个结点就是答案了。而支持快速合并操作的就是并查集了。我们在函数内，首先我们需要递归处理子树，然后合并至当前结点，接着遍历所有需要求解的二元组，然后存进 $a$ 数组里面就行了。

这种算法是最快的，但是只能用作离线成了这个算法的最大弱点。并且你还需要将所有的询问读入进来在处理，在分批询问、有更改操作的情况下不能很好的支持。

!!! note "注意"
    朴素 Tarjan 的时间复杂度并不是 $\mathcal O(n+m)$ 的，而是 $\mathcal O(m\times\alpha(n+m,n)+n)$ 的。如果需要追求线性的话，可以看 [这篇英文论文](https://dl.acm.org/doi/pdf/10.1145/800061.808753)。

#### 代码

```cpp
#include <iostream>
#include <vector>

using namespace std;

const int kMaxN = 1e6 + 5;

struct Node {
  vector<int> e;             // 邻点
  vector<pair<int, int>> q;  // 查询
  int f;
} v[kMaxN];

int a[kMaxN], n, m, s;

int F(int x) {
  return !v[x].f ? x : v[x].f = F(v[x].f);
}

void tarjan(int x, int f) {
  for (int i : v[x].e) {  // 枚举邻点
    if (i != f) {         // 如果不为父亲
      tarjan(i, x);       // 递归处理
    }
  }
  for (auto i : v[x].q) {  // 枚举需要的答案
    if (i.first == x) {    // 如果正好等于 x
      a[i.second] = x;     // 直接记录
    } else {
      a[i.second] = F(i.first);  // 否则更改为 i.first 的根
    }
  }
  v[x].f = f;  // 合并
}

int main() {
  ios::sync_with_stdio(0), cin.tie(0), cout.tie(0);
  cin >> n >> m >> s;
  for (int i = 1, x, y; i < n; i++) {
    cin >> x >> y;
    v[x].e.push_back(y), v[y].e.push_back(x);
  }
  for (int i = 1, x, y; i <= m; i++) {
    cin >> x >> y;
    v[x].q.emplace_back(y, i), v[y].q.emplace_back(x, i);
  }
  tarjan(s, 0);
  for (int i = 1; i <= m; i++) {
    cout << a[i] << '\n';
  }
  return 0;
}
```

## 习题

!!! note "注意"
    最近公共祖先基本上不会考模板题，而是题目做着做着就会用一下的那种，所以需要背熟板子，不过基本上只用背倍增的那种就行了。比如接下来的题目中，就有很多是将一条路径拆分成两条路径的然后求解的那种。

### 判断路径

#### 题目大意

有一颗 $n$ 个节点的树，给出 $q$ 个询问，每一个询问有 $a,b,c,d$ 四个结点编号，问你 $a$ 到 $b$ 和 $c$ 到 $d$ 的路径是否碰到过。

#### 思路

这题我们使用倍增求解 LCA。然后就非常简单了，变成了判断是否碰到过的问题。首先，我们可以找出 $a$ 和 $b$ 的最近公共祖先，就假设为 $l_1$，然后当 $c$ 和 $l_1$ 的最近公共祖先证号为 $l_1$ 的时候，或者 $d$ 和 $l_1$ 的最近公共祖先正好为 $l_1$ 的时候，那么就说明碰到过。然后反过来再判断一次就行了。

#### 代码

```cpp
#include <iostream>
#include <vector>

using namespace std;

const int kMaxN = 5e5 + 5;

int dp[kMaxN][25], d[kMaxN], lg[kMaxN], n, q;
vector<int> e[kMaxN];

void dfs(int x, int f) {
  dp[x][0] = f, d[x] = d[f] + 1;
  for (int i = 1; i <= lg[d[x]]; i++) {
    dp[x][i] = dp[dp[x][i - 1]][i - 1];
  }
  for (int i : e[x]) {
    f != i && (dfs(i, x), 0);
  }
}

int LCA(int x, int y) {
  for (d[x] < d[y] && (x ^= y, y ^= x, x ^= y); d[x] > d[y]; x = dp[x][lg[d[x] - d[y]] - 1]) { }
  if (x == y) {
    return x;
  }
  for (int k = lg[d[x]] - 1; k >= 0; k--) {
    dp[x][k] != dp[y][k] && (x = dp[x][k], y = dp[y][k]);
  }
  return dp[x][0];
}

int main() {
  cin >> n >> q;
  for (int i = 1, u, v; i < n; i++) {
    cin >> u >> v;
    e[u].push_back(v), e[v].push_back(u);
  }
  for (int i = 1; i <= n; lg[i] = lg[i - 1] + (2 * lg[i - 1] == i), i++) { }
  dfs(1, 0);
  for (int x1, y1, x2, y2; q; q--) {
    cin >> x1 >> y1 >> x2 >> y2;
    int z1 = LCA(x1, y1), z2 = LCA(x2, y2);  // 获取祖先
    cout << ((LCA(x2, z1) == z1 || LCA(y2, z1) == z1) && (LCA(x1, z2) == z2 || LCA(y1, z2) == z2) ? 'Y' : 'N') << '\n';  // 判断
  }
  return 0;
}
```

### 求和

#### 题目大意

有一颗 $n$ 个结点的树，设 $d(i)$ 表示为第 $i$ 个结点离根节点的距离，那么每一次询问，都给你两个点和一个整数 $k$，然后假设两个点之间的路径的编号数组为 $a$，那么你需要求解 $\sum\limits_{i=1}^{|a|}d(a_i)^k$。

#### 思路

这题是一道树上差分的题目。我们设 $p_{(i,j)}$ 表示为根节点 $1$ 到 $i$ 号结点上，所有函数的值的 $j$ 次方的和。那么我们就可以跑树上 dp，求出 $dp$ 数组和 $p$ 数组，然后将每一次询问一刀剁成两半，分别作差分操作，最后累加起来就行了。注意这题需要对 $998244353$ 取模，然而差分的时候，我们可能会差分到负数，但是这题理论上不可能差分到复数的，所以我们每一次相减就先加上 $998277353$ 之后再取模。

#### 代码

```cpp
#include <iostream>
#include <vector>

using namespace std;
using ll = long long;

const ll kMaxN = 3e5 + 5, kMod = 998244353;

ll d[kMaxN], fa[kMaxN], p[kMaxN][51], dp[kMaxN][25], lg[kMaxN], n, m, u, v, k;
vector<ll> e[kMaxN];

void dfs(ll x, ll f) {
  dp[x][0] = f, d[x] = d[f] + 1, fa[x] = f;  // 计算父亲、距离
  ll pow = 1;                                // d[x] ^ 0
  for (ll i = 1; i <= 50; i++) {             // d[x] ^ i
    pow = (pow * d[x]) % kMod;               // 计算幂，记得取模
    p[x][i] = (p[f][i] + pow) % kMod;        // 存下来，记得取模
  }
  for (ll i = 1; i <= lg[d[x]]; i++) {       // 更新 dp 数组
    dp[x][i] = dp[dp[x][i - 1]][i - 1];
  }
  for (ll i : e[x]) {
    f != i && (dfs(i, x), 0);
  }
}

ll LCA(ll x, ll y) {  // 最近公共祖先模板
  for (d[x] < d[y] && (x ^= y, y ^= x, x ^= y); d[x] > d[y]; x = dp[x][lg[d[x] - d[y]] - 1]) { }
  if (x == y) {
    return x;
  }
  for (ll k = lg[d[x]] - 1; k >= 0; k--) {
    dp[x][k] != dp[y][k] && (x = dp[x][k], y = dp[y][k]);
  }
  return dp[x][0];
}

int main() {
  cin >> n, d[0] = -1;  // 根节点距离应为 0
  for (ll i = 1, u, v; i < n; i++) {
    cin >> u >> v;
    e[u].push_back(v);
    e[v].push_back(u);
  }
  for (ll i = 1; i <= n; lg[i] = lg[i - 1] + (1 << lg[i - 1] == i), i++) { }
  dfs(1, 0);
  for (cin >> m; m; m--) {
    cin >> u >> v >> k;
    u > v && (u ^= v, v ^= u, u ^= v);  // 保持 u <= v
    ll l = LCA(u, v);                   // 获取最近公共祖先
    if (l == u || l == v) {             // 如果是一条边
      cout << (p[v][k] - p[fa[u]][k] + kMod) % kMod << '\n';                     // 直接计算
    } else {
      cout << ((((p[v][k] - p[fa[l]][k] + kMod) % kMod +                         // 左边的和
                 (p[u][k] - p[fa[l]][k] + kMod) % kMod) % kMod -                 // 右边的和
                 (p[l][k] - p[fa[l]][k] + kMod) % kMod) + kMod) % kMod << '\n';  // 减去 d[l] ^ k
    }
  }
  return 0;
}
```

### 继续差分

#### 题目大意

$k$ 个询问，每一次 $u$ 到 $v$ 路径上面的每一个结点的值都要加 $1$，问你最后最大的值是多少。

#### 思路

还是可以作差分，将一条路径分成两半，作差分操作，最后跑一边 dfs 还原数组，取最大值就行了。

#### 代码

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

using namespace std;

const int kMaxN = 5e4 + 5;

int dp[kMaxN][25], d[kMaxN], lg[kMaxN], p[kMaxN], fa[kMaxN], a[kMaxN], n, k;
vector<int> e[kMaxN];

void dfs(int x, int f) {
  fa[x] = f;
  dp[x][0] = f, d[x] = d[f] + 1;
  for (int i = 1; i <= lg[d[x]]; i++) {
    dp[x][i] = dp[dp[x][i - 1]][i - 1];
  }
  for (int i : e[x]) {
    f != i && (dfs(i, x), 0);
  }
}

int LCA(int x, int y) {
  for (d[x] < d[y] && (x ^= y, y ^= x, x ^= y); d[x] > d[y]; x = dp[x][lg[d[x] - d[y]] - 1]) { }
  if (x == y) {
    return x;
  }
  for (int k = lg[d[x]] - 1; k >= 0; k--) {
    dp[x][k] != dp[y][k] && (x = dp[x][k], y = dp[y][k]);
  }
  return dp[x][0];
}

void solve(int x, int f) {  // 还原数组
  a[x] = p[x];              // 初始化为当前元素
  for (int i : e[x]) {      // 枚举邻点
    if (i != f) {
      solve(i, x);          // 递归处理
      a[x] += a[i];         // 累加和
    }
  }
}

int main() {
  cin >> n >> k;
  for (int i = 1, u, v; i < n; i++) {
    cin >> u >> v;
    e[u].push_back(v);
    e[v].push_back(u);
  }
  for (int i = 1; i <= n; i++) {
    lg[i] = lg[i - 1] + (1 << lg[i - 1] == i);
  }
  dfs(1, 0);
  for (int i = 1, u, v; i <= k; i++) {
    cin >> u >> v;
    int l = LCA(u, v);                   // 获取最近公共祖先
    p[u]++, p[v]++, p[l]--, p[fa[l]]--;  // 差分
  }
  solve(1, 0);                                     // 求解
  cout << *max_element(a + 1, a + n + 1) << '\n';  // 求最大值
  return 0;
}
```
