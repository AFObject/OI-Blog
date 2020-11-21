---
layout:     post
title:      图论基础（5）——Bellman-Ford及SPFA算法
subtitle:   Bellman-Ford和SPFA最短路算法
date:       2020-11-21
author:     23786.NSObject
header-img: img/post-bg-miui6.jpg
catalog: true
tags:
    - C++
    - 图论
    - 上课笔记
---
# 图论基础（5）

## Bellman-Ford 算法

简称Ford算法， $O(N\cdot E)$。同样也是一种**单源最短路径算法**。能够处理存在负边权的情况。但无法处理存在负权回路的情况。

$N$ 为顶点数， $E$ 为边数。

- 设 $s$ 为起点， $dis_v$ 为 $s$ 到 $v$ 的最短距离， $prev_v$ 为 $v$ 的前驱， $w_j$ 是边 $j$ 的长度，且 $j$ 连接 $u, v$。

- $dis_s = 0, dis_v = \infty(v\neq s), prev_s = 0$。

	````cpp
	for (int i = 1; i <= n - 1; i += 1) {
	  for (int j = 1; j <= e; j += 1) {
	    const int u = edge[j][1]; // edge[a][1] 为第a条边的起点
	    const int v = edge[j][2]; // edge[a][1] 为第a条边的终点
	    if (dis[u] + w[j] < dis[v]) {
	      dis[v] = dis[u] + w[j];
	      prev[v] = u;
	    }
	  }
	}
	````

	

### Ford算法判断负权回路

----

```cpp
#include <cstdio>
#include <cstring>
#include <vector>
int n, m;
int from, to;
struct Edge {
	int from, to, v;
};
std::vector<Edge> v;
int dis[1001];
int main() {
	scanf("%d%d", &n, &m);
	int tFrom, tTo, tV;
	for (int i = 1; i <= m; i += 1) {
		scanf("%d%d%d", &tFrom, &tTo, &tV);
		v.push_back( (Edge){tFrom, tTo, tV} );
	}
	scanf("%d%d", &from, &to);
	memset(dis, 0x3f3f3f3f, sizeof(dis));
	dis[from] = 0;
	for (int i = 1; i < n; i += 1) {
		for (int j = 1; j <= m; j += 1) {
			if (dis[v[j].to] > v[j].v + dis[v[j].from]) {
				dis[v[j].to] = v[j].v + dis[v[j].from];
			}
		}
	}
	for (int j = 1; j <= m; j += 1) {
		if (dis[v[j].to] > v[j].v + dis[v[j].from]) {
			printf("-1\n");
			return 0;
		}
	}
	printf("%d\n", dis[to]);
	return 0;
}
```

----

## SPFA算法

Bellman-Ford算法更新边会出现浪费的情况，可以优化。

即Bellman-Ford算法的队列优化。

我们用数组 $dis$ 记录每个结点的当前的最短路径值。设立一个“先进先出”的队列用来保存待优化的结点，优化时每次取出队首结点 $u$，并且用 $u$ 点当前的最短路径值对 $u$ 点所指向的结点 $v$ 进行松弛操作，如果 $v$ 的最短路径估计值有所调整，且 $v$ 点不在当前的队列中，就将 $v$ 点放入队尾。这样不断从队列中取出结点来进行松弛操作，直至队列空为止。

- 设 $s$ 为起点， $dis_v$ 为 $s$ 到 $v$ 的最短距离， $prev_v$ 为 $v$ 的前驱， $w_{i, j}$ 是 $i$ 到 $j$ 的长度，且 $j$ 连接 $u, v$，$exist_u$ 代表 $u$ 是否在队列中。

- $dis_s = 0, dis_v = \infty(v\neq s), prev_s = 0$。

```cpp
while ( !q.empty() ) {
  u = q.front();
  q.pop();
  exist[u] = false;
  for (所有与u相连的点v) {
    if (dis[v] > dis[u] + w[u][v]) {
      dis[v] = dis[u] + w[u][v];
      prev[v] = u;
      // 队列中不存在点v则入队
      if ( !exist[v] ) {
        q.push( v );
        exist[v] = true;
      }
    }
  }
}
```

----

### 练习

#### P1828 [USACO3.2]香甜的黄油 Sweet Butter

本题评级为 $\color{green}{\texttt{普及+/提高}}$，但个人认为思路很好想。对于每一个牧场，跑一遍SPFA，之后求出距离总和最小的那个牧场。

代码实现也很简单，~~我竟然自己做出来了~~

```cpp
#include <cstdio>
#include <cstring>
#include <iostream>
#include <queue>
int n, m, c;
int result = 0x3f3f3f3f;
int mc[1001];
int dis[1001];
bool exist[1001];
struct Road {
	int to, w;
};
std::queue<int> q;
std::vector<Road> v[1001];
void spfa(int start) {
	memset( dis, 0x3f3f3f3f, sizeof(dis) );
	memset( exist, false, sizeof(exist) );
	dis[start] = 0;
	exist[start] = true;
	q = std::queue<int>();
	q.push(start);
	while ( !q.empty() ) {
		int u = q.front();
		q.pop();
		exist[u] = false;
		for (std::vector<Road>::iterator it = v[u].begin(); it != v[u].end(); it++) {
			if ( dis[it->to] > it->w + dis[u] ) {
				dis[it->to] = it->w + dis[u];
				if (!exist[it->to]) {
					q.push(it->to);
					exist[it->to] = true;
				}
			}
		}
	}
int main() {
	scanf("%d%d%d", &n, &m, &c);
	for (int i = 1; i <= n; i += 1) {
		int temp;
		scanf("%d", &temp);
		mc[temp] += 1;
	}
	for (int i = 1; i <= c; i += 1) {
		int f, t, w;
		scanf("%d%d%d", &f, &t, &w);
		v[f].push_back( (Road) { t, w } );
		v[t].push_back( (Road) { f, w } );
	}
	for (int i = 1; i <= m; i += 1) {
		spfa(i);
		int res = 0;
		for (int j = 1; j <= m; j += 1) {
			res += mc[j] * dis[j];
		}
		result = std::min(res, result);
	}
	printf("%d\n", result);
	return 0;
}
```

