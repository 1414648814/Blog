# 题意

[题目](https://leetcode.com/problems/pacific-atlantic-water-flow/#/description)

# 思路

一开始想用双向广搜来做，找他们相碰的点，但是发现对其的理解还是不够完全，导致没写成功。不过，后来想清楚了，之前的错误可能在于从边界点进行BFS，其访问顺序应该是找到下一个比当前那个要大的点，但是我写反了。。可以先对左边的队列进行BFS，保存其visited，再接着对右边的队列进行BFS，当访问到之前已经访问过的结点时，则加入到结果中。

# 实现

```cpp
//
//

#include "../PreLoad.h"

/*
Given the following 5x5 matrix:

  Pacific ~   ~   ~   ~   ~
       ~  1   2   2   3  (5) *
       ~  3   2   3  (4) (4) *
       ~  2   4  (5)  3   1  *
       ~ (6) (7)  1   4   5  *
       ~ (5)  1   1   2   4  *
          *   *   *   *   * Atlantic

Return:

[[0, 4], [1, 3], [1, 4], [2, 2], [3, 0], [3, 1], [4, 0]] (positions with parentheses in above matrix).

 */

class Solution {
public:
    /**
     * 从边界的点开始进行BFS，找出交叉的点，即为可以到达两边的点
     * @param matrix
     * @return
     */
    vector<pair<int, int>> pacificAtlantic(vector<vector<int>>& matrix) {
        vector<pair<int, int>> result;
        if (matrix.size() == 0) {
            return result;
        }

        // 双向BFS，找重合点
        queue<pair<int, int>> pa_queue; //太平洋
        queue<pair<int, int>> at_queue; //大西洋

        size_t row = matrix.size();
        size_t col = matrix[0].size();

        vector<vector<int>> layouts = {{-1, 0}, {1, 0}, {0, -1}, {0, 1}};
        auto state_bfs = [&](queue<pair<int, int>>& queue, vector<vector<int>>& visit) {
            while (!queue.empty()) {
                pair<int, int> content = queue.front();
                queue.pop();

                int height = content.first;
                int x = content.second / col;
                int y = content.second % col;

                for (auto temp : layouts) {
                    int newx = temp[0] + x;
                    int newy = temp[1] + y;

                    // 因为是从边界结点开始，题目又要求某个点从高往低到两个海洋，所以边界点访问顺序则为递增
                    if (newx < 0 || newx >= row || newy < 0 || newy >= col || visit[newx][newy] || matrix[newx][newy] < height) {
                        continue;
                    }

                    queue.push({matrix[newx][newy], newx * col + newy});
                    visit[newx][newy] = 1;
                }
            }
        };

        vector<vector<int>> visited1(row, vector<int>(col, 0));
        vector<vector<int>> visited2(row, vector<int>(col, 0));

        // 第一行和第一列
        for (int i = 0; i < col; i++) {
            pa_queue.push({matrix[0][i], i});
            visited1[0][i] = 1;

            at_queue.push({matrix[row-1][i], (row-1) * col + i});
            visited2[row-1][i] = 1;
        }
        for (int i = 0; i < row; i++) {
            pa_queue.push({matrix[i][0], i * col});
            visited1[i][0] = 1;

            at_queue.push({matrix[i][col-1], i * col + col-1});
            visited2[i][col-1] = 1;
        }

        state_bfs(pa_queue, visited1);
        state_bfs(at_queue, visited2);

        // 交叉点则为可以到达两边的点
        for (int i = 0; i < row; i++) {
            for (int j = 0; j < col; j++) {
                if (visited1[i][j] && visited2[i][j]) {
                    result.push_back({i, j});
                }
            }
        }

        return result;
    }


    vector<pair<int, int>> pacificAtlantic2(vector<vector<int>>& matrix) {
        vector<pair<int, int>> result;
        if (matrix.size() == 0) {
            return result;
        }

        // 双向BFS，找重合点
        queue<pair<int, int>> pa_queue; //太平洋
        queue<pair<int, int>> at_queue; //大西洋

        size_t row = matrix.size();
        size_t col = matrix[0].size();

        vector<vector<int>> layouts = {{-1, 0}, {1, 0}, {0, -1}, {0, 1}};
        auto state_bfs = [&](queue<pair<int, int>>& queue, vector<vector<int>>& visit, bool flag) {
            while (!queue.empty()) {
                pair<int, int> content = queue.front();
                queue.pop();

                int height = content.first;
                int x = content.second / col;
                int y = content.second % col;

                for (auto temp : layouts) {
                    int newx = temp[0] + x;
                    int newy = temp[1] + y;

                    // 因为是从边界结点开始，题目又要求某个点从高往低到两个海洋，所以边界点访问顺序则为递增
                    if (newx < 0 || newx >= row || newy < 0 || newy >= col || matrix[newx][newy] < height) {
                        continue;
                    }

                    if (flag) {
                        if (visit[newx][newy] != 1) {
                            // 已经被另外一个访问过
                            if (visit[newx][newy] == 2) {
                                result.push_back({newx, newy});
                                continue;
                            }

                            queue.push({matrix[newx][newy], newx * col + newy});
                            visit[newx][newy] = 1;
                        }
                    }
                    else {
                        if (visit[newx][newy] != 2) {
                            // 已经被另外一个访问过
                            if (visit[newx][newy] == 1) {
                                result.push_back({newx, newy});
                                continue;
                            }

                            queue.push({matrix[newx][newy], newx * col + newy});
                            visit[newx][newy] = 2;
                        }
                    }
                }
            }
        };

        vector<vector<int>> visited1(row, vector<int>(col, 0));
        vector<vector<int>> visited2(row, vector<int>(col, 0));

        // 第一行和第一列
        for (int i = 0; i < col; i++) {
            pa_queue.push({matrix[0][i], i});
            visited1[0][i] = 1;

            at_queue.push({matrix[row-1][i], (row-1) * col + i});
            visited2[row-1][i] = 2;
        }
        for (int i = 0; i < row; i++) {
            pa_queue.push({matrix[i][0], i * col});
            visited1[i][0] = 1;

            at_queue.push({matrix[i][col-1], i * col + col-1});
            visited2[i][col-1] = 2;
        }

        while (!pa_queue.empty() || !at_queue.empty()) {
            if (!pa_queue.empty()) {
                state_bfs(pa_queue, visited1, true);
            }

            if (!at_queue.empty()) {
                state_bfs(at_queue, visited2, false);
            }
        }

        // 交叉点则为可以到达两边的点
        for (int i = 0; i < row; i++) {
            for (int j = 0; j < col; j++) {
                if (visited1[i][j] && visited2[i][j]) {
                    result.push_back({i, j});
                }
            }
        }

        return result;
    }

    void test() {
        vector<vector<int>> matrix = {
                {1, 2, 2, 3, 5},
                {3, 2, 3, 4, 4},
                {2, 4, 5, 3, 1},
                {6, 7, 1, 4, 5},
                {5, 1, 1, 2, 4}
        };

        vector<pair<int, int>> result = this->pacificAtlantic2(matrix);
        for (int i = 0; i < result.size(); i++) {
            pair<int, int> content = result[i];
            cout << "(" << content.first << ", " << content.second << ")" << endl;
        }
    }
};

```