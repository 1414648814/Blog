
# 题意

[题目](https://leetcode.com/problems/trapping-rain-water-ii/#/description)

# 思路

我一开始想的时候只考虑到一个结点周围的边界的情况，并没有考虑到边界的高度其实影响到所有的结点盛水的高度。

我们可以发现，中间是否能够盛水取决于边界是否足够高于里面的高度，所以这必然是一个从外到内，从小到大的一个过程。因为优先队列必然首先访问的是边界中最小的高度，如果周围小于这个高度那么必然是存在可以盛水的地方，就算是没有也没有任何的损失，只是更新了高度，但是队列依然会从高度小的结点开始访问；

# 实现

```cpp
typedef struct node {
    int x, y;
    node(int x, int y) : x(x), y(y) {}
} Node;

class Solution {
public:
    /**
     [1,4,3,1,3,2],
     [3,2,1,3,2,4],
     [2,3,3,2,3,1]
     发现：四条边都不可以存储水，，也就是在这个二维数组中才能存储水
     从第二列开始，去找四周里面大于它自己的第一个数（前提是三面或者两面是大于它的），然后减去自己
     */
    int trapRainWater(vector<vector<int>>& heightMap) {
        if (heightMap.size() == 0) {
            return 0;
        }
        else if (heightMap.size() < 3 || heightMap[0].size() < 3) {
            return 0;
        }

        int row = heightMap.size();
        int col = heightMap[0].size();

        queue<Node*> queues;
        vector<vector<int>> visited(row, vector<int>(col, 0));

        Node *start = new Node(1, 1);

        visited[1][1] = 1;
        queues.push(start);

        auto state_center = [&](int x, int y) {
            if (x >= 1 && x <= heightMap.size()-2 && y >= 1 && y <= heightMap[0].size()-2) {
                return true;
            }
            return false;
        };

        //上下左右
        vector< vector<int>> layouts = {{1, 0}, {-1, 0}, {0, -1}, {0, 1}};
        int sum = 0;

        while (!queues.empty()) {
            Node* node = queues.front();
            queues.pop();

            // 取中间的值
            if (state_center(node->x, node->y)) {
                vector<int> priority;
                bool isfinished = true;
                // 找周围四个点
                for (int i = 0; i < layouts.size(); i++) {
                    vector<int> temp = layouts[i];
                    int newx = node->x + temp[0];
                    int newy = node->y + temp[1];

                    // 不符合条件
                    if (heightMap[newx][newy] > heightMap[node->x][node->y] && (newx == 0 || newx == heightMap.size()-1) &&
                            (newy == 0 || newy == heightMap[0].size()-1)) {
                        isfinished = false;
                        break;
                    }
                    else if (newx == 0 || newx == heightMap.size()-1 || newy == 0 || newy == heightMap[0].size()-1) {
                        priority.push_back(heightMap[newx][newy]);
                        continue;
                    }

                    Node *newnode = new Node(newx, newy);
                    if (!visited[newx][newy]) {
                        queues.push(newnode);
                        visited[newx][newy] = 1;
                    }
                }

                if (isfinished) {
                    sort(priority.begin(), priority.end());
                    int val = heightMap[node->x][node->y];
                    for (int i = 0; i < priority.size(); i++) {
                        if (priority[i] >= val) {
                            sum += priority[i] - val;
                            break;
                        }
                    }
                }
            }
        }

        return sum;
    }

    /**
     * 存放水的高度取决于周围的最小高度，BFS的思想
     * 首先存取四面的高度，用优先队列去存储，保证取出来的一定是队列中的最小值
     * 取较大的高度，如果存在没访问过并且小于当前最小高度的，则计算盛水高度，并且将该结点设置成已访问
     *
     * @param heightMap
     * @return
     */
    int trapRainWater2(vector<vector<int>>& heightMap) {
        if (heightMap.size() == 0) {
            return 0;
        }

        int row = heightMap.size();
        int col = heightMap[0].size();

        struct cmp {
            bool operator()(pair<int, int> a, pair<int, int> b) {
                if (a.first > b.first) {
                    return true;
                }
                else {
                    return false;
                }
            }
        };

        priority_queue<pair<int, int>, vector<pair<int, int>>, cmp> queue;
        vector<vector<int>> visited(row, vector<int>(col, 0));

        // 将外围的高度加入到队列中，找出最小值
        for (int i = 0; i < row; i++) {
            for (int j = 0; j < col; j++) {
                if (i == 0 || i == row-1 || j == 0 || j == col-1) {
                    queue.push(make_pair(heightMap[i][j], i * col + j));
                    visited[i][j] = 1;
                }
            }
        }

        vector< vector<int>> layouts = {{1, 0}, {-1, 0}, {0, -1}, {0, 1}};
        int maxHeight = INT_MIN, sum = 0;
        while (!queue.empty()) {
            pair<int, int> content = queue.top();
            queue.pop();

            int height = content.first;
            int x = content.second / col;
            int y = content.second % col;

            maxHeight = max(maxHeight, height);

            for (auto temp : layouts) {
                int newx = x + temp[0];
                int newy = y + temp[1];

                if (newx < 0 || newx >= row || newy < 0 || newy >= col || visited[newx][newy]) {
                    continue;
                }

                if (heightMap[newx][newy] < maxHeight) {
                    sum += maxHeight - heightMap[newx][newy];
                }

                visited[newx][newy] = 1;
                queue.push(make_pair(heightMap[newx][newy], newx * col + newy));
            }
        }

        return sum;
    }

    void test() {
        vector< vector<int>> water = {
                {12,13,1,12},
                {13,4,13,12},
                {13,8,10,12},
                {12,13,12,12},
                {13,13,13,13}
        };
        int result = this->trapRainWater2(water);
        cout << "sum:" << result << endl;
    }
};

```


[参考文章](http://www.cnblogs.com/grandyang/p/5928987.html)