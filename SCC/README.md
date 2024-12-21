# 更新日志

upd 2024.01.31 写好文章基本内容

upd 2024.01.31 发表于 [洛谷](https://www.luogu.com.cn/blog/941575/qiang-lian-tong-fen-liang-scc-strongly-connected-components-xue-xi-bi-j)

upd 2024.02.01 同步发表于 [CSDN](https://blog.csdn.net/taoyiwei17_QFOI/article/details/135965697)

upd 2024.02.01 同步发表于 [博客园cnblogs](https://www.cnblogs.com/zaixiaqingshang/p/18001336)

upd 2024.02.01 增加内容 `difficult PRO 例题详解——P2746`

upd 2024.06.04 收录至《[提高级算法集合&总结](https://www.luogu.com.cn/article/uxrsb5j3)》

# 强连通分量（SCC，Strongly Connected Components）
## 定义
### 强连通
有向图（DAG）中若其中两点 $x$，$y$ 能彼此到达（不一定是直接连边），称 $x$ 和 $y$ 是强连通的。

![](https://s11.ax1x.com/2024/01/31/pFMegKJ.png)

下图中 $1$ 和 $7$ 是其中一组强连通的点对。

### 强连通分量
有向图 $G$ 中存在一个极大子图 $G1$，$G1$ 内任意 $2$ 点都是强连通的，且 $G1$ 不能包含更多的点，那么 $G1$ 为 $G$ 的一个 SCC。

注意：强连通分量要么是 $1$ 个单点，要么是 $1$ 个环。



## 如何寻找强连通分量
### 法1——Tarjan 算法
#### 引入
Robert E. Tarjan（罗伯特·塔扬，1948~），生于美国加州波莫纳，计算机科学家。

Tarjan 发明了很多算法和数据结构。不少他发明的算法都以他的名字命名，以至于有时会让人混淆几种不同的算法。比如求各种连通分量的 Tarjan 算法，求 LCA（Lowest Common Ancestor，最近公共祖先）的 Tarjan 算法。并查集、Splay、Toptree 也是 Tarjan 发明的。

#### 前置知识——时间戳
也称 $dfs$ 序。表示从源点开始搜索 $x$ 是第几个被搜到的。

#### 前置知识——关键点
指搜索到每个环时第一个被搜到的环内点。

特征（设某一关键点为 $x$）：`dfn[x]==low[x]`。

#### 过程
Tarjan 是所有求强连通分量的算法中最常见也最实用的算法，基于 dfs（深度优先搜索，Deep First Search）算法。

定义数组 $dfn_x$ 表示点 $x$ 在 $dfs$ 搜索中的最小时间戳；$low_x$ 表示点 $x$ 能走到的点的最小时间戳。

$dfn_x$ 的维护：定义计数器 $cnt$，每 $dfs$ 一次，`cnt++,dfn[x]=cnt`。


$low_x$ 的维护：初始时 `low[x]=dfn[x]`，在回溯后对 $x$ 指向的点的 $low$ 或 $dfn$ 取最小值。

##### $low_x$ 维护代码
```cpp
	for(int nxt:nbr[cur])
	{
		if(!dfn[nxt])
		{
			tarjan(nxt);
			low[cur]=min(low[cur],low[nxt]);
		}
		else if(viss[nxt])
		{
			low[cur]=min(low[cur],dfn[nxt]);
		}
	}
```

最后若 $dfn[x]==low[x]$，将站内位置在 $x$ 以上的所有结点以及 $x$ 出栈（已确认这些结点属于这个 SCC），而且可以确定多了一个强连通分量。

时间复杂度：$O(n + m)$

#### Tarjan 代码
```cpp
void tarjan(int cur)
{
	stk.push(cur);
	viss[cur]=1;
	low[cur]=dfn[cur]=++cnt;
	for(int nxt:nbr[cur])
	{
		if(!dfn[nxt])
		{
			tarjan(nxt);
			low[cur]=min(low[cur],low[nxt]);
		}
		else if(viss[nxt])
		{
			low[cur]=min(low[cur],dfn[nxt]);
		}
	}
	if(dfn[cur]==low[cur])
	{
		sum++;
		while(stk.top()!=cur)
		{
			int tmp=stk.top();
			stk.pop();
			scc[tmp]=sum;
			ans[sum].push_back(tmp);
			viss[tmp]=0;
		}
		stk.pop();
		ans[sum].push_back(cur);
		scc[cur]=sum;
		viss[cur]=0;
	}
}
```

### 法2——Kosaraju 算法

#### 引入
Kosaraju 算法由 S. Rao Kosaraju 在 1978 年一篇未发表的论文中提出，但 Micha Sharir 最早发表了它。

#### 过程
创建反图。

两遍 dfs。

第一次 dfs，选取任意顶点作为起点，遍历所有未访问过的顶点，并在回溯之前给顶点编号，也就是后序遍历。

第二次 dfs，对于反向后的图，以标号最大的顶点作为起点开始 DFS。这样遍历到的顶点集合就是一个强连通分量。对于所有未访问过的结点，选取标号最大的，重复上述过程。

两次 dfs 结束后，强连通分量就找出来了，Kosaraju 算法的时间复杂度为 $O(n+m)$。

由于 Kosaraju 算法应用面没有 Tarjan 算法广，故在此几笔带过。

#### Kosaraju 代码
```cpp
void dfs1(int u) {
  vis[u] = true;
  for (int v : g[u])
    if (!vis[v]) dfs1(v);
  s.push_back(u);
}

void dfs2(int u) {
  color[u] = sccCnt;
  for (int v : g2[u])
    if (!color[v]) dfs2(v);
}

void kosaraju() {
  sccCnt = 0;
  for (int i = 1; i <= n; ++i)
    if (!vis[i]) dfs1(i);
  for (int i = n; i >= 1; --i)
    if (!color[s[i]]) {
      ++sccCnt;
      dfs2(s[i]);
    }
}
```

其中 $g$ 是原图，$g2$ 是反图。

### 法3——Garbow 算法
Garbow 算法是 Tarjan 算法的另一种实现（注意，并不是优化），Tarjan 算法是用 $dfn$ 和 $low$ 来计算强连通分量的根，Garbow 维护一个节点栈，并用第二个栈来确定何时从第一个栈中弹出属于同一个强连通分量的节点。

从节点 $w$ 开始的 dfs 过程中，当一条路径显示这组节点都属于同一个 SCC 时，只要栈顶节点的时间戳大于根节点 $w$ 的时间戳，就从第二个栈中弹出这个节点，那么最后只留下根节点 $w$。

在这个过程中每一个被弹出的节点都属于同一个 SCC。

当回溯到某一个节点 $w$ 时，如果这个节点在第二个栈的顶部，就说明这个节点是 SCC 的起始节点，在这个节点之后搜索到的那些节点都属于同一个 SCC，于是从第一个栈中弹出那些节点，构成 SCC。

Garbow 算法的时间复杂度为 $O(n+m)$。

由于 Tarjan 算法的广泛出现，Garbow 算法没有人会系统的学习，故在此几笔带过。

### Garbow 代码
``` cpp
int garbow(int u) {
  stack1[++p1] = u;
  stack2[++p2] = u;
  low[u] = ++dfs_clock;
  for (int i = head[u]; i; i = e[i].next) {
    int v = e[i].to;
    if (!low[v])
      garbow(v);
    else if (!sccno[v])
      while (low[stack2[p2]] > low[v]) p2--;
  }
  if (stack2[p2] == u) {
    p2--;
    scc_cnt++;
    do {
      sccno[stack1[p1]] = scc_cnt;
      // all_scc[scc_cnt] ++;
    } while (stack1[p1--] != u);
  }
  return 0;
}

void find_scc(int n) {
  dfs_clock = scc_cnt = 0;
  p1 = p2 = 0;
  memset(sccno, 0, sizeof(sccno));
  memset(low, 0, sizeof(low));
  for (int i = 1; i <= n; i++)
    if (!low[i]) garbow(i);
}
```

### 例题：[B3609 [图论与代数结构 701] 强连通分量](https://www.luogu.com.cn/problem/B3609)

#### 题目描述

给定一张 $n$ 个点 $m$ 条边的有向图，求出其所有的强连通分量。

**注意，本题可能存在重边和自环。**

#### 输入格式

第一行两个正整数 $n$ ， $m$ ，表示图的点数和边数。

接下来 $m$ 行，每行两个正整数 $u$ 和 $v$ 表示一条边。

#### 输出格式

第一行一个整数表示这张图的强连通分量数目。

接下来每行输出一个强连通分量。第一行输出 1 号点所在强连通分量，第二行输出 2 号点所在强连通分量，若已被输出，则改为输出 3 号点所在强连通分量，以此类推。每个强连通分量按节点编号大小输出。

#### 样例 #1

##### 样例输入 #1

```
6 8
1 2
1 5
2 6
5 6
6 1
5 3
6 4
3 4
```

##### 样例输出 #1

```
3
1 2 5 6
3
4
```

#### 提示

对于所有数据，$1 \le n \le 10000$，$1 \le m \le 100000$。

### 代码+解析注释——Tarjan
本题主要难点在第二问“接下来每行输出一个强连通分量。第一行输出 1 号点所在强连通分量，第二行输出 2 号点所在强连通分量，若已被输出，则改为输出 3 号点所在强连通分量，以此类推。每个强连通分量按节点编号大小输出。”上。

```cpp
#include<bits/stdc++.h>
#define int long long
using namespace std;
const int N=1e5+5;
int low[N], dfn[N], cnt, sum, scc[N], n, m;
bool viss[N];
stack<int> stk;
vector<int> nbr[N], ans[N];
void tarjan(int cur)//找 SCC
{
	stk.push(cur);
	viss[cur]=1;
	low[cur]=dfn[cur]=++cnt;
	for(int nxt:nbr[cur])
	{
		if(!dfn[nxt])
		{
			tarjan(nxt);
			low[cur]=min(low[cur],low[nxt]);
		}
		else if(viss[nxt])
		{
			low[cur]=min(low[cur],dfn[nxt]);
		}
	}
	if(dfn[cur]==low[cur])
	{
		sum++;
		while(stk.top()!=cur)
		{
			int tmp=stk.top();
			stk.pop();
			scc[tmp]=sum;
			ans[sum].push_back(tmp);
			viss[tmp]=0;
		}
		stk.pop();
		ans[sum].push_back(cur);
		scc[cur]=sum;
		viss[cur]=0;
	}
}
signed main()
{
	cin>>n>>m;
	for(int i=1;i<=m;i++)
	{
		int u, v;
		cin>>u>>v;
		nbr[u].push_back(v);
	}
	for(int i=1;i<=n;i++)
	{
		if(!dfn[i])
		{
			tarjan(i);
		}
	}
	cout<<sum<<"\n";
	for(int i=1;i<=n;i++)
	{
		if(dfn[i]==0)
		{
			continue;
		}
		sort(ans[scc[i]].begin(), ans[scc[i]].end());//vector 从小到大排序
		for(int cur:ans[scc[i]])
		{
			cout<<cur<<" ";
			dfn[cur]=0;
		}
		cout<<"\n";
	}
}
```

动态数组 $ans$ 表示每个 SCC 包含的点。第二问将每个 SCC 的点排个序然后输出即可。

## 强连通分量试炼场
### difficult 1
[P1455 搭配购买](https://www.luogu.com.cn/problem/P1455)

[P3916 图的遍历](https://www.luogu.com.cn/problem/P3916)

### difficult 1 例题详解——P3916
先 Tarjan 缩点。

在同一个 SCC 中的点可以相互到达，那么我们可以在缩点时记录每个 SCC 中编号最大的点。

一个 SCC 中所有点能到达的最大编号的点即为这个 SCC 能到达的最大的编号。

缩点代码就不放了，在这里提供普通 dfs 的代码。

#### 代码
```cpp
#include<bits/stdc++.h>
#define int long long
using namespace std;
int n, m, ans[1000005];
vector<int> nbr[100005];
void dfs(int cur, int maxi)
{
	if(ans[cur]!=0)
	{
		return ;
	}
	ans[cur]=maxi;
	for(int nxt:nbr[cur])
	{
		dfs(nxt,maxi);
	}
}
signed main()
{
	cin>>n>>m;
	for(int i=1;i<=m;i++)
	{
		int x, y;
		cin>>x>>y;
		nbr[y].push_back(x);
	}
	for(int i=n;i>=1;i--)
	{
		if(ans[i]==0)
		{
			dfs(i,i);
		}
	}
	for(int i=1;i<=n;i++)
	{
		cout<<ans[i]<<" ";
	}
}
```

### difficult 2
[ P3387 【模板】缩点](https://www.luogu.com.cn/problem/P3387)

[P2863 USACO06JAN The Cow Prom S](https://www.luogu.com.cn/problem/P2863)

[P2341 USACO03FALL / HAOI2006 受欢迎的牛 G](https://www.luogu.com.cn/problem/P2341)

[P2002 消息扩散](https://www.luogu.com.cn/problem/P2002)

[P1073 NOIP2009 提高组 最优贸易](https://www.luogu.com.cn/problem/P1073)

### difficult 2 例题详解——P3387

先来看一张图

![](https://pic.imgdb.cn/item/65ba4ec7871b83018a3ee3cb.png)

在这个图中，有 $5$ 个 SCC。

将这 $5$ 个 SCC 缩成一个点，得到 $5$ 个新点 $a,b,c,d,e$。

其中

点 $1$ --------> 点 $a$

点 $2,3,4,5$ --------> 点 $b$

点 $6$ --------> 点 $c$

点 $7,8,9$ --------> 点 $d$

点 $10$ --------> 点 $e$

这就是所谓的缩点，缩成的点的点权为原 SCC 内点权之和。这也是本题的重要思想。

为什么要缩点呢？在本题中，点权最大的路径可以使用拓扑排序+dp，但拓扑排序的前提是有向无环图（DAG），而本题明显有环。缩点就让环变成一个点了。

#### 代码
```cpp
#include<bits/stdc++.h>
#define int long long
using namespace std;
const int maxn=1e5+5;
vector<int>nbr[maxn];
stack<int>stk;
int in[maxn];
int a[maxn], sum[maxn];
int dp[maxn];
int n, m, dfn[maxn], used[maxn], vis[maxn], low[maxn], scc[maxn], num[maxn], cntc, cnt, ans;
struct node
{
    int x, y;
}e[maxn];
void tarjan(int cur)
{
    dfn[cur]=low[cur]=++cnt;
    stk.push(cur);
    for(int q:nbr[cur])
    {
        if(!dfn[q])
        {
            tarjan(q);
            low[cur]=min(low[cur],low[q]);
        }
        else if(!scc[q]) 
        {
            low[cur]=min(low[cur],dfn[q]);
        }
    }
    if(low[cur]==dfn[cur])
    {
        cntc++;
        scc[cur]=cntc;
        num[cntc]++;
        while(stk.top()!=cur)
        {
            int t=stk.top();
            stk.pop();
            scc[t]=cntc;
            num[cntc]++;
        }
        stk.pop();
    }
}
void topo()
{
	queue<int> q;
	for(int i=1;i<=cntc;i++)
	{
		if(in[i]==0)
		{
			q.push(i);
			dp[i]=sum[i];
		}
	}
	while(q.empty()==0)
	{
		int cur=q.front();
		q.pop();
		for(int nxt:nbr[cur])
		{
			in[nxt]--;
		    dp[nxt]=max(dp[nxt],dp[cur]+sum[nxt]);
			if(in[nxt]==0)
			{
				q.push(nxt);
			}
		} 
	}
	int maxi=-1e9;
	for(int i=1;i<=cntc;i++)
	{
	    maxi=max(maxi,dp[i]);
	}
	cout<<maxi;
	return ;
}
signed main()
{
	cin>>n>>m;
    for(int i=1;i<=n;i++)
    {
        cin>>a[i];
    }
    for(int i=1;i<=m;i++)
    {
        cin>>e[i].x>>e[i].y;
        nbr[e[i].x].push_back(e[i].y);
    }
    for(int i=1;i<=n;i++)
    {
        if(!dfn[i])
        {
            tarjan(i);
        } 
    }
    for(int i=1;i<=n;i++)
    {
        sum[scc[i]]+=a[i];
    }
    for(int i=1;i<=10004;i++)
    {
        nbr[i].clear();
    }
    for(int i=1;i<=m;i++)
    {
        if(scc[e[i].x]!=scc[e[i].y])
        {
            nbr[scc[e[i].x]].push_back(scc[e[i].y]);
            in[scc[e[i].y]]++;
        }
    }
    topo(); 
    return 0;
}
```

### difficult 2 例题详解——P2863
记录每个 SCC 的大小，最后判断是不是大于 $1$ 并输出即可。

#### 代码
```cpp
#include<bits/stdc++.h>
using namespace std;
const int maxn=1e4+5;
vector<int>nbr[maxn];
stack<int>s;
int n, m, dfn[maxn], used[maxn], vis[maxn], low[maxn], color[maxn], num[maxn], cntc, cnt, ans;
void add(int x)
{
    s.pop();
    color[x]=cntc;
    num[cntc]++;
    vis[x]=false;
}
void tarjan(int x)
{
    dfn[x]=low[x]=++cnt;
    s.push(x);
    vis[x]=used[x]=true;
    for(int q:nbr[x])
    {
        if(!dfn[q])
        {
            tarjan(q);
            low[x]=min(low[x],low[q]);
        }
        else if(vis[q]) 
        {
            low[x]=min(low[x],dfn[q]);
        }
    }
    if(low[x]==dfn[x])
    {
        cntc++;
        while(s.top()!=x)
        {
            int t=s.top();
            add(t);
        }
        add(x);
    }
}
int main()
{
    cin>>n>>m;
    for(int i=1;i<=m;i++)
    {
        int u,v;
        cin>>u>>v;
        nbr[u].push_back(v);
    }
    for(int i=1;i<=n;i++)
    {
        if(!used[i])
        {
            tarjan(i);
        } 
    }
    for(int i=1;i<=cntc;i++)
    {
        if (num[i]>1)
        {
            ans++;
        } 
    }
    cout<<ans;
    return 0;
}

```

### difficult 2 例题详解——P2341
Tarjan

由题可得，受欢迎的奶牛只有可能是图中唯一的入度为零的强连通分量中的所有奶牛，所以若出现两个以上入度为 $0$ 的强连通分量则不存在明星奶牛。


#### 代码
```cpp
#include<bits/stdc++.h>
using namespace std;
const int maxn=1e5+5;
vector<int>nbr[maxn];
stack<int>s;
int ans1=0;
int in[maxn];
int a[maxn];
int n, m, dfn[maxn], used[maxn], vis[maxn], low[maxn], color[maxn], num[maxn], cntc, cnt, ans;
void add(int x)
{
    color[x]=cntc;
    num[cntc]++;
}
void tarjan(int x)
{
    dfn[x]=low[x]=++cnt;
    s.push(x);
    for(int q:nbr[x])
    {
        if(!dfn[q])
        {
            tarjan(q);
            low[x]=min(low[x],low[q]);
        }
        else if(!color[q]) 
        {
            low[x]=min(low[x],dfn[q]);
        }
    }
    if(low[x]==dfn[x])
    {
        cntc++;
        add(x);
        while(s.top()!=x)
        {
            int t=s.top();
            add(t);
            s.pop();
        }
        s.pop();
    }
}
int main()
{
    cin>>n>>m;
    for(int i=1;i<=m;i++)
    {
        int u,v;
        cin>>u>>v;
        nbr[v].push_back(u);
    }
    for(int i=1;i<=n;i++)
    {
        if(!dfn[i])
        {
            tarjan(i);
        } 
    }
    for(int i=1;i<=n;i++)
    {
        for(int nxt:nbr[i])
        {
            if(color[i]!=color[nxt])
            {
                in[color[nxt]]++;
            }
        }
    }
    int ans=0, cntu=0;
    for(int i=1;i<=cntc;i++)
    {
        if(!in[i])
        {
            ans=num[i];
            cntu++;
        } 
    }
    if(cntu==1)cout<<ans;
    else cout<<0;
    return 0;
}
```

### difficult PRO
放几道题在这，难度很大，诸位加油~。

[P3627 APIO2009 抢掠计划](https://www.luogu.com.cn/problem/P3627)

[UVA11324 The Largest Clique](https://www.luogu.com.cn/problem/UVA11324)

[P2746 USACO5.3 校园网Network of Schools](https://www.luogu.com.cn/problem/P2746)

[P2812 校园网络【USACO Network of Schools加强版】](https://www.luogu.com.cn/problem/P2812)

[P1407 国家集训队 稳定婚姻](https://www.luogu.com.cn/problem/P1407)

[P2169 正则表达式](https://www.luogu.com.cn/problem/P2169)

[P4819 中山市选 sr游戏](https://www.luogu.com.cn/problem/P4819)

[P2403 SDOI2010 所驼门王的宝藏](https://www.luogu.com.cn/problem/P2403)

[P4339 ZJOI2018 迷宫](https://www.luogu.com.cn/problem/P4339)

[P4233 射命丸文的笔记](https://www.luogu.com.cn/problem/P4233)

### difficult PRO 例题详解——P2746
#### 提炼大意
1. 给定 $n$ 个点若干条有向边。

目标 1：求最少发几个软件，使得 $n$ 个点都能得到。

目标 2：最少连几条边，使得无论发 $1$ 个软件给哪个点其余点都能得到。

#### 分析
1. 跑 Tarjan 缩点，建新图，统计入度。

2. 入度为 $0$ 的强连通分量（SCC）的个数就是答案 $1$。

3. 入度为 $0$ 的强连通分量（SCC）和出度为 $0$ 的强连通分量（SCC）的个数的较大值为答案 $2$。

#### 代码
```cpp
#include<bits/stdc++.h>
#define int long long
using namespace std;
const int N=1e5+5;
int n, m, a[N], num[N], low[N], dfn[N], in[N], sum[N], cnt, cntc, scc[N];
bool viss[N];
struct node
{
	int x, y;
}e[N<<2];
queue<int> q;
int dp[N], out[N];
vector<int> nbr[N];
stack<int> stk;
void tarjan(int cur)
{
	low[cur]=dfn[cur]=++cnt;
	viss[cur]=1;
	stk.push(cur);
	for(int nxt:nbr[cur])
	{
		if(!dfn[nxt])
		{
			tarjan(nxt);
			low[cur]=min(low[cur],low[nxt]);
		}
		else if(viss[nxt])
		{
			low[cur]=min(low[cur],dfn[nxt]);
		}
	}
	if(dfn[cur]==low[cur])
	{
		cntc++;
		scc[cur]=cntc;
		num[cntc]++;
		viss[cur]=0;
		while(stk.top()!=cur)
		{
			int nxt=stk.top();
			stk.pop();
			num[cntc]++;
			viss[nxt]=0;
			scc[nxt]=cntc;
		}
		stk.pop();
	}
}
int cntd=0;
signed main()
{
	cin>>n;
	for(int i=1;i<=n;i++)
	{
		int x;
		while(cin>>x)
		{
			if(x==0)
			{
				break;
			}
			nbr[i].push_back(x);
			cntd++;
			e[cntd].x=i;
			
			e[cntd].y=x;
		}
	}
	for(int i=1;i<=n;i++)
	{
		if(!dfn[i])
		{
			tarjan(i);
		}
	}
	int ind=0, outd=0;
	for(int i=1;i<=cntd;i++)
	{
		if(scc[e[i].x]!=scc[e[i].y])
		{
			in[scc[e[i].y]]++;
			out[scc[e[i].x]]++;
		}
	}
	for(int i=1;i<=cntc;i++)
	{
		ind+=!in[i];
		outd+=!out[i];
	}
	if(cntc==1)
	{
		cout<<ind<<"\n"<<0;
	}
	else
	{
		cout<<ind<<"\n"<<max(ind,outd);
	}
} 
```

# 写在最后
这篇文章算是篇幅最长的了，倾注了许多时间和精力在上面，如有不足或可增改的地方欢迎大家评论区或私信联系我，非常感谢。

文章中部分参考于 [OI Wiki](https://oi-wiki.org/)

在此特别鸣谢 [来追梦信息学](https://gonoi.com.cn/)，感谢杨老师的悉心教导。

bye~
