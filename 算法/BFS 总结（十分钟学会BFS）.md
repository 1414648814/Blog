# 前言

有时候，当你并不了解很多高级算法的时候，搜索不失为一种解决问题的好方法，而且很多高级算法有或多或少的会用到搜索或者搜索的思想。可见，搜索是一个基础并且必须要掌握的算法。

在这篇文章中，会对BFS进行一次系统的总结。好了，废话不多说，赶紧开始。

搜索里面包含了一下内容：

- 列表 
    - 线性搜索
    - 二分搜索
- 树/图
    - 广度优先搜索
        - 最良优先搜索
        - 均一开销搜索
        - A*算法
    - 深度优先搜索
        - 迭代深化深度优先搜索
        - 深度限制搜索
    - 双方向探索
    - 分支限定法
- 字符串
    - KMP算法
    - BM算法
    - AC自动机
    - Rabin-Karp算法
    - 位图算法

但是这里直讲关于BFS的内容，涉及到BFS的有树和图；

# 实现

## 树

注意，树里面分为二叉树和多叉树，处理方式有些不一样。

### 树的结构

```cpp
/**
 * 树
 */
struct TreeNode {
    int val;
    TreeNode *left;
    TreeNode *right;
    TreeNode *prev;
    TreeNode(int x) : val(x), left(NULL), right(NULL), prev(NULL) {}
};

struct MoreTreeNode {
    int val;
    std::vector<MoreTreeNode*> nexts;
    MoreTreeNode(int x) : val(x) {}
};

```

### 树的bfs实现

```cpp
/**
 * 树的最基本的BFS
 * @param root
 */
void Solution::DanXiangTreeBFS(TreeNode* root) {
    if (root == NULL) {
        return ;
    }

    std::queue<TreeNode*> queues;
    queues.push(root);

    while (!queues.empty()) {
        TreeNode* node = queues.front();
        queues.pop();

        std::cout << "value:" << node->val << std::endl;

        if (node->left) {
            queues.push(node->left);
        }

        if (node->right) {
            queues.push(node->right);
        }
    }
}

/**
 * 多插树的最基本的BFS
 * @param root
 */
void Solution::MoreTreeBFS(MoreTreeNode* root) {
    if (root) {
        return ;
    }

    std::queue<MoreTreeNode*> queues;
    queues.push(root);
    std::unordered_map<MoreTreeNode*, bool> visited;

    visited[root] = true;

    while (!queues.empty()) {
        MoreTreeNode* node = queues.front();
        queues.pop();

        // 访问所有子结点
        for (int i = 0; i < node->nexts.size(); i++) {
            MoreTreeNode* next = node->nexts[i];
            if (next && !visited[next]) {
                visited[next] = true;
                std::cout << "value:" << next->val << std::endl;
                queues.push(next);
            }
        }
    }
}

```

### 总结

有没有发现，树的BFS遍历必然会存在一个队列，一个纪录是否访问过该结点的数组，总而言之，就是如果没访问过就加入，一直循环直到全部结点访问完。

## 图

图的表示方式有好几种，这里只用了邻接表的表达方式。

### 图的结构

```cpp
/**
 * 图
 */
// 图的最大顶点数目
#define MaxGraphSize 10

// 定义边表结点
struct ArcNode {
    int val;
    ArcNode* next;
};

// 定义顶点表结点
template <class T>
struct VertexNode {
    T vertex;
    ArcNode* firstNode;
};

// 图
struct Graph {
    int edgeNum;
    int vertexNum;
    VertexNode<int> vertes[MaxGraphSize];
};

```

### 图的bfs实现

```cpp
/**
 * 图的BFS
 */
void Solution::GraphBFS(Graph* graph) {
    int value[MaxGraphSize];

    int head = 0, rear = 0;
    int queue[MaxGraphSize];
    int visited[MaxGraphSize];

    memset(visited, 0, sizeof(int)*MaxGraphSize);

    // 遍历所有的结点，并且以该结点作为起始点，迭代遍历起始结点的下一个结点
    for (int i = 0; i < graph->vertexNum; i++) {
        if (!visited[i]) {
            visited[i] = 1;
            queue[rear++] = i; //结点i入队列
        }

        while(head != rear) {
            int j = queue[head++]; //结点j出队列
            ArcNode* node = graph->vertes[j].firstNode; //找到对应的起始结点

            while (node != NULL) {
                int k = node->val; //结点k
                if (!visited[k]) {
                    visited[k] = 1;
                    queue[rear++] = k;
                }

                node = node->next;
            }
        }
    }
}

```

### 总结

是不是感觉有些不一样了呢，nono，实质还是一样的，只不过这里并没有用到stl，而是用下标head和rear去纪录出度和入度，实现一个队列，因为当head和rear相等的时候就是访问完所有结点的时候。

## 双向广搜

什么是双向广搜呢，就是说使用两个队列，一个从头用BFS搜，另一个从尾也用BFS搜，直到双方存在重叠的结点，则返回，至于返回什么根据题目的要求，可以是层数balabala；因为将搜索分为两个部分，搜索的时间自然会变得很快。

```cpp
/**
 * 双向广搜，存在两个方向上的BFS，当两个方向的搜索生成同一子结点时终止此搜索
 * 通常存在两种方法：1.两个方向交替扩展，2.选择结点个数较少的那个方向先扩展
 * @param root
 */
void Solution::ShuangXiangBFS(MoreTreeNode* begin, MoreTreeNode* target) {
    // 假设是图
    std::queue<MoreTreeNode*> min, max;
    min.push(begin);
    max.push(target);

    bool found = false;
    std::unordered_map<MoreTreeNode*, int> visited;
    visited[begin] = 1, visited[target] = 2; //初始状态下默认设置成1（正向）和2（反向）

    // flag作为区分两个队列的标志
    auto extend_node = [&](std::queue<MoreTreeNode*> queues, bool flag) {
        MoreTreeNode* node = queues.front();
        queues.pop();

        // 访问所有子结点
        for (int i = 0; i < node->nexts.size(); i++) {
            MoreTreeNode* next = node->nexts[i];

            if (flag) {
                if (visited[next] != 1) { // 没在正向队列中出现
                    if (visited[next] == 2) { // 在反向队列中出现过，说明已经找到了
                        found = true;
                        return;
                    }
                    visited[next] = 1; //做标记
                    queues.push(next);
                }
            }
            else {
                if (visited[next] != 2) { // 同上
                    if (visited[next] == 1) {
                        found = true;
                        return;
                    }
                    visited[next] = 2;
                    queues.push(next);
                }
            }
        }
    };

    while (!min.empty() || !max.empty()) {
        if (!min.empty()) {
            extend_node(min, true);
        }

        if (found) {
            return ;
        }

        if (!max.empty()) {
            extend_node(max, false);
        }

        if (found) {
            return ;
        }
    }
}

```

其实上面的代码并不完全正确，仍然需要进行优化，比如说会遇到这样的情况

![](http://i1.piimg.com/588926/46eebdf7dfab9bda.jpg)

求S-T的最短路，交替节点搜索（一次正向节点，一次反向节点）时
 
while(1):
S –> 1
T –> 3

while(2):
S -> 5
T -> 4
 
while(3):
1 -> 5
3 -> 5   返回最短路为4，错误的，事实是3，S-2-4-T

正确做法的是**交替逐层搜索**，保证了不会先遇到非最优解就跳出，而是检查完该层所有节点，得到最优值。也即如果该层搜索遇到了对方已经访问过的，那么已经搜索过的层数就是答案了，可以跳出了，以后不会更优的了。

下面的代码进行优化：优先去访问结点少的层。

```cpp
#include <stdio.h>
#include <stdlib.h>
#include <queue>
#include <iostream>
using namespace std;

int l; //4<=l<=300
int sx,sy,tx,ty;
int a[305][305];	//正向搜索层次
int b[305][305];	//反向搜索层次

struct point{
	int x,y;
};

struct point dir[]=
	{
	{1,2},{1,-2},{-1,2},{-1,-2},
	{2,1},{2,-1},{-2,1},{-2,-1}
	};

bool check(int x,int y){
	if(x>=0 && x<l && y>=0 && y<l)
		return true;
	else
		return false;
}

void dbfs(){
	memset(a,-1,sizeof(a));
	memset(b,-1,sizeof(b));
	a[sx][sy]=0;
	b[tx][ty]=0;

	queue<point> forQ,backQ;
	point p1,p2;
	p1.x=sx;
	p1.y=sy;
	p2.x=tx;
	p2.y=ty;
	forQ.push(p1);
	backQ.push(p2);
	
	//正反向队列至少还有一个可以扩展
	while(forQ.empty()==false || backQ.empty()==false){
		//优化：优先扩展元素少的队列（如果只有一个队列非空，则扩展非空队列）
		int forSize=forQ.size();
		int backSize=backQ.size();

		if(backSize==0 || forSize<backSize){
			//扩展正向队列一层
			int i;
			for(i=0;i<forSize;i++){
				point cur=forQ.front();
				forQ.pop();

				if(b[cur.x][cur.y]!=-1){
					printf("%d\n",a[cur.x][cur.y]+b[cur.x][cur.y]);
					return;
				}

				int j;
				for(j=0;j<8;j++){
					if(check(cur.x+dir[j].x, cur.y+dir[j].y)){
						point next={cur.x+dir[j].x, cur.y+dir[j].y};	//!注意struct的创建方式
						
						if(a[next.x][next.y]!=-1)//以前已经正向扩展过
							continue;

						a[next.x][next.y]=a[cur.x][cur.y]+1;

						forQ.push(next);
					}
				}
			}
		}else{
			//扩展反向队列一层
			int i;
			for(i=0;i<backSize;i++){
				point cur=backQ.front();
				backQ.pop();

				if(a[cur.x][cur.y]!=-1){
					printf("%d\n",a[cur.x][cur.y]+b[cur.x][cur.y]);
					return;
				}

				int j;
				for(j=0;j<8;j++){
					if(check(cur.x+dir[j].x, cur.y+dir[j].y)){
						point next={cur.x+dir[j].x, cur.y+dir[j].y};
						
						if(b[next.x][next.y]!=-1)
							continue;

						b[next.x][next.y]=b[cur.x][cur.y]+1;

						backQ.push(next);
					}
				}
			}
		}
	}//end while

	printf("0");
}

int main(void){
	int time;
	scanf("%d",&time);
	while(time-->0){
		scanf("%d%d%d%d%d\n",&l,&sx,&sy,&tx,&ty);
		dbfs();
	}

	return 0;
}

```


## 练习

网络上有这么一道题，[骑士移动](http://poj.org/problem?id=1915)

这道题就可以用到BFS，双向BFS，A*(有目的的BFS)来解决

### BFS

```cpp
/**
 * 骑士题目
 * 给定起点和终点，按照骑士的走法，求解最短路数
 */

// 使用bfs实现（核心代码）
void qishi_bfs() {
    typedef struct Node {
        int x, y;
        int step;
    };

    int dis[8][2]={{-2,1},{-2,-1},{-1,-2},{-1,2},{2,-1},{2,1},{1,-2},{1,2}};
    std::vector< std::vector<int>> visited(8, std::vector<int>(8, 0)); // 8*8
    Node start, end;

    // 边界判断
    auto isValid = [&](Node node) {
        if (node.x < 0 || node.y < 0 || node.x > 7 || node.y > 7) {
            return false;
        }
        return true;
    };

    // 是否为结果
    auto isTarget = [&](Node node1, Node node2) {
        if (node1.x == node2.x && node1.y == node2.y) {
            return true;
        }
        return false;
    };

    std::queue<Node> queue;
    queue.push(start);
    visited[start.x][start.y] = 1;

    auto state_bfs = [&]() {
        while(!queue.empty()) {
            Node next = queue.front();
            queue.pop();

            if (isTarget(start, next)) {
                std::cout << next.step << std::endl;
                break;
            }

            // 方可可以到达的哈什湖上
            for (int i = 0; i < 8; ++i) {
                Node t;
                t.x = next.x + dis[i][0];
                t.y = next.y + dis[i][1];
                if (isValid(t) && visited[t.x][t.y] == 0) {
                    visited[t.x][t.y] = 1;
                    t.step = next.step + 1;
                    queue.push(t);
                }
            }
        }
    };

    state_bfs();
}

```

### 双向BFS

```cpp
// 使用双队列的bfs实现（核心代码）
void qishi_double_bfs() {
    typedef struct Node {
        int x, y;
        int step;
    };

    int dis[8][2]={{-2,1},{-2,-1},{-1,-2},{-1,2},{2,-1},{2,1},{1,-2},{1,2}};
    std::vector< std::vector<int>> level(8, std::vector<int>(8, 0)); // 8*8
    std::vector< std::vector<int>> color(8, std::vector<int>(8, 0)); // 区分队列
    Node start, end;

    std::queue<Node> queue_start;
    queue_start.push(start);
    level[start.x][start.y] = 0;
    color[start.x][start.y] = 1;

    std::queue<Node> queue_end;
    queue_start.push(end);
    level[end.x][end.y] = 1; //因为起点和终点必然不是重合的，它们之间至少存在1的距离
    color[start.x][start.y] = 2;

    // 边界判断
    auto isValid = [&](Node node) {
        if (node.x < 0 || node.y < 0 || node.x > 7 || node.y > 7) {
            return false;
        }
        return true;
    };

    auto state_bfs = [&](std::queue<Node> queue, bool flag) {
        while(!queue.empty()) {
            Node next = queue.front();
            queue.pop();

            // 方可可以到达的哈什湖上
            for (int i = 0; i < 8; ++i) {
                Node t;
                t.x = next.x + dis[i][0];
                t.y = next.y + dis[i][1];

                if (isValid(t)) {
                    continue;
                }

                if (flag) {
                    if (color[t.x][t.y] == 0) {
                        color[t.x][t.y] = 1;
                        level[t.x][t.y] = level[next.x][next.y] + 1;
                        queue.push(t);
                    }
                    else if (color[t.x][t.y] == 2) {
                        return level[t.x][t.y] + level[next.x][next.y];
                    }
                }
                else {
                    if (color[t.x][t.y] == 0) {
                        color[t.x][t.y] = 2;
                        level[t.x][t.y] = level[next.x][next.y] + 1;
                        queue.push(t);
                    }
                    else if (color[t.x][t.y] == 1) {
                        return level[t.x][t.y] + level[next.x][next.y];
                    }
                }
            }
        }
    };

    // 避免出现起始或者结束中没有下一步的结点，也就是可能中间断掉了
    while (!queue_start.empty() || !queue_end.empty()) {
        if (!queue_start.empty()) {
            state_bfs(queue_start, true);
        }

        if (!queue_end.empty()) {
            state_bfs(queue_end, true);
        }
    }
}

```

### A*

因为这个稍微有些复杂，所以会单独写一篇文章来描述。

## 其他

当我们已经获取到了图的邻接表的时候，要求从起始结点到目标结点的路径，应该怎么求呢？

```cpp
/**
 * 反向生成路径，但是需要注意的地方在于，其只能生成一条路径
 * 要么你每次在主程序中得到目标结点后，就收敛然后使用这个函数
 * @tparam state_t
 * @param father
 * @param target
 * @return
 */
template <typename  state_t>
std::vector<state_t> gen_path(const std::unordered_map<state_t, state_t> father, const state_t& target) {
    std::vector<state_t> path;
    path.push_back(target);

    // 从叶结点一直到访问到根结点（不再存在父结点）
    for (state_t cur = target; father.find(cur) != father.end(); cur = father.at(cur)) {
        path.push_back(cur);
    }

    std::reverse(path.begin(), path.end());
    return path;
}

/**
 * 很显然，对于多条路径的计算方法，DFS再合适不过了，
 * 此时nexts为邻接表，纪录每个结点的相关的状态数组
 * @tparam state_t
 */
template <typename state_t>
std::vector<std::vector<state_t>> results;

template <typename state_t>
void get_more_path(const std::unordered_map<state_t, std::vector<state_t>> nexts, std::vector<state_t> path, state_t cur, state_t target) {
    // 达到目标后收敛
    if (cur == target) {
        results.push_back(path);
        return ;
    }
    else {
        std::vector<state_t> next = nexts[cur];
        for (int i = 0; i < next.size(); i++) {
            state_t now = next[i];
            path.push_back(now);
            get_more_path(nexts, path, now, target);
            path.pop_back();
        }
    }
}

```