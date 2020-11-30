# 例题一

[LC 200. 岛屿数量](https://leetcode-cn.com/problems/number-of-islands/)

给你一个由 `'1'`（陆地）和 `'0'`（水）组成的的二维网格，请你计算网格中岛屿的数量。

岛屿总是被水包围，并且每座岛屿只能由水平方向或竖直方向上相邻的陆地连接形成。

此外，你可以假设该网格的四条边均被水包围。

## 分析

初始时岛屿数量是图中 `'1'` 的个数。如果一个位置为 `'1'`，其与上下左右相邻的 `'1'` 就可以合并为一个大的岛屿，同时岛屿数量减 `1`。最终无法合并的岛屿数量就是我们需要求的结果。

该题还可以通过 `DFS` 和 `BFS` 求解，请读者自行完成。

## 代码

```cpp
#include <iostream>
#include <vector>

using namespace std;

class UnionFind {
   public:
    vector<int> parent;
    int cnt;  // 连通分量的个数
    UnionFind(int n, int cnt) {
        parent = vector<int>(n);
        for (int i = 0; i < n; ++i) {
            parent[i] = i;
        }
        this->cnt = cnt;
    }

    int Find(int x) {
        if (x != parent[x]) {
            parent[x] = Find(parent[x]);
        }
        return parent[x];
    }

    void Union(int x, int y) {
        int px = Find(x), py = Find(y);
        if (px != py) {
            parent[px] = py;
            --cnt;
        }
    }
};

class Solution {
   public:
    int numIslands(vector<vector<char>> &grid) {
        if (grid.empty() || grid[0].empty()) return 0;

        m = grid.size();
        n = grid[0].size();
        int cnt = 0;
        for (int i = 0; i < m; ++i) {
            for (int j = 0; j < n; ++j) {
                if (grid[i][j] == '1') {
                    ++cnt;
                }
            }
        }
        UnionFind *uf = new UnionFind(m * n, cnt);

        int direc[4][2] = {{0, 1}, {0, -1}, {1, 0}, {-1, 0}};
        for (int i = 0; i < m; ++i) {
            for (int j = 0; j < n; ++j) {
                if (grid[i][j] == '1') {
                    for (int d = 0; d < 4; ++d) {
                        int ni = i + direc[d][0], nj = j + direc[d][1];
                        if (ni < 0 || ni >= m || nj < 0 || nj >= n || grid[ni][nj] == '0') {
                            continue;
                        }
                        int id1 = get(i, j);
                        int id2 = get(ni, nj);
                        uf->Union(id1, id2);
                    }
                }
            }
        }
        return uf->cnt;
    }

   private:
    int m, n;
    int get(int i, int j) { 
        return i * n + j;
    }
};
```

# 例题二

[LC 947. 移除最多的同行或同列石头](https://leetcode-cn.com/problems/most-stones-removed-with-same-row-or-column/)

我们将石头放置在二维平面中的一些整数坐标点上。每个坐标点上最多只能有一块石头。

每次 move 操作都会移除一块所在行或者列上有其他石头存在的石头。

请你设计一个算法，计算最多能执行多少次 move 操作？

## 分析

将处于同一行或者同一列的石头两两相连，这样会得到一个图，互相连通的石子组成一个连通分量。在一个连通分量里，我们能找到一个最优的方法，进行 move 操作，直到最后只剩一个石头。首先，我们要知道每个石子都属于一个连通分量，同时在一个连通分量中移除石子不会影响到其他的连通分量。在有了这个前提之下，我们可以推断出，如果把连通分量作为一个生成树来看，每次都移除树中的叶子节点，重复这个操作，最后就只会剩下一个根节点。

如何使用并查集来解决这个问题？

如果考虑直接合并石头的话，当 `(a, b)` 点有石头时，我们需要遍历找到所有 `x=a` 或者 `y=b` 的石头进行合并，时间复杂度是 $O(n^2)$，开销比较大。

换个角度思考，当 `(a, b)` 这个点有石头时，相当于将 `x=a` 与 `y=b` 两条平行于坐标轴的线绑定在一起。集合的主体并不是石头，而是线。

当 `(a, b)` 这个点有石头时，进行 `Union(a, b)` 操作，为了保证每个点的唯一性，实际上使用的是 `Union(a, ~b)`。由于我们无法直接得到初始的节点数目，所以使用 `map` 来保存父节点信息。进行 `Find(x)` 操作时，如果 `x` 不在 `map` 里面，就将其加入 `map` 同时节点数加 `1`。如果 `x` 在 `map` 里面，说明之前已经添加过，不做任何操作。

## 代码

```cpp
#include <unordered_map>
#include <vector>

using namespace std;

class UnionFind {
   public:
    unordered_map<int, int> parent;
    int islands;

    UnionFind() {
        islands = 0;
    }

    int Find(int x) {
        if (!parent.count(x)) {
            parent[x] = x;
            ++islands;
        }
        if (x != parent[x]) {
            parent[x] = Find(parent[x]);
        }
        return parent[x];
    }

    void Union(int x, int y) {
        int px = Find(x), py = Find(y);
        if (px != py) {
            parent[px] = py;
            --islands;
        }
    }
};

class Solution {
   public:
    int removeStones(vector<vector<int>>& stones) {
        UnionFind* uf = new UnionFind();
        for (auto& v : stones) {
            uf->Union(v[0], ~v[1]);
        }
        return stones.size() - uf->islands;
    }
};
```

## 总结

介绍上面两个例题的目的在于：

1. 并查集里可以加入额外信息，比如统计图中连通分量的个数。
2. 并查集不一定局限于使用数组保存父节点信息，当节点数不确定时可以使用 `map`，相应的 `Find(x)` 做出修改。 