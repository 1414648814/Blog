# 题意

[题目](http://www.cnblogs.com/George1994/p/6597661.html)
 
# 思路
 
我一开始的时候想的是最简单的方法，就是遍历所有的值为1的元素，再根据其为起点进行BFS，计算层数，但是这个方法超时了；

其实，可以不用从1开始遍历，从0开始遍历，找到和值为1相邻的0，将其的层数设置为1就行了，为什么可以不用从1开始，因为并没有要求从规定的起点到指定的位置，计算最小距离，而是计算一整个周围，只要周围存在1，则将其加入到队列，计算相应的距离（又可能存在别多个1包围的1的情况），注意的是，在访问过1的结点后下次不可以再进行计算。


# 实现

```cpp
//
//

#include "../PreLoad.h"

class Solution {
public:

    /**
     * 三重循环，最外层为所有1的结点，里面两层是实现BFS
     * 导致时间复杂度过高，待优化
     * @param matrix
     * @return
     */
    vector< vector<int>> layouts = {{-1, 0}, {1, 0}, {0, -1}, {0, 1}};

    vector<vector<int>> updateMatrix(vector<vector<int>>& matrix) {
        vector<vector<int>> result(matrix);
        if (matrix.size() == 0) {
            return result;
        }

        size_t row = matrix.size();
        size_t col = matrix[0].size();

        deque<pair<int, int>> queues;
        vector<vector<int>> visited(row, vector<int>(col , 0));

        bool isHaveOne = false;
        for (size_t i = 0; i < row; i++) {
            for (size_t j = 0; j < col; j++) {
                if (matrix[i][j]) {
                    queues.push_back({i, j});
                    isHaveOne = true;
                }
            }
        }

        if (!isHaveOne) {
            return result;
        }

        while (!queues.empty()) {
            auto content = queues.front();
            queues.pop_front();

            bool found = false;
            int level = 0;

            deque<pair<int, int>> tqueue;
            tqueue.push_back(content);

            vector<vector<int>> tvisited(visited);
            tvisited[content.first][content.second] = 1;

            while (!found && !tqueue.empty()) {
                level++;

                int queue_len = tqueue.size();
                // 保证队列中的每个数都能加上基本的平方数
                for (int i = 0; i < queue_len; i++) {
                    auto tcontent = tqueue.front();
                    tqueue.pop_front();

                    for (auto temp : layouts) {
                        int newx = tcontent.first + temp[0];
                        int newy = tcontent.second + temp[1];

                        if (newx < 0 || newy < 0 || newx >= row || newy >= col || tvisited[newx][newy]) {
                            continue;
                        }

                        if (!matrix[newx][newy]) {
                            found = true;
                            break;
                        }

                        tvisited[newx][newy] = 1;
                        tqueue.push_back({newx, newy});
                    }
                }
            }

            tvisited = visited;

            result[content.first][content.second] = level;
        }

        return result;
    }

    // 做法错误
    vector<vector<int>> updateMatrix2(vector<vector<int>>& matrix) {
        vector<vector<int>> result(matrix);
        if (matrix.size() == 0) {
            return result;
        }

        size_t row = matrix.size();
        size_t col = matrix[0].size();

        deque<pair<int, int>> queues;

        bool isHaveOne = false;
        for (size_t i = 0; i < row; i++) {
            for (size_t j = 0; j < col; j++) {
                if (matrix[i][j]) {
                    queues.push_back({i, j});
                    isHaveOne = true;
                }
            }
        }

        if (!isHaveOne) {
            return result;
        }

        vector<vector<int>> visited(row, vector<int>(col , 0));
        vector<vector<int>> tvisited(visited);
        while (!queues.empty()) {
            auto content = queues.front();
            queues.pop_front();

            int level = 0;
            bool found = false;

            DFSHelper(matrix, result, visited, level, content.first, content.second, row, col, found);

            tvisited = visited;
            //result[content.first][content.second] = level;
        }

        return result;
    }

    // 无法保证取得的路径是最短的，因为是深度递归，所以有可能找的那条路径全都是1的，所以这样使用DFS是错误的
    void DFSHelper(vector<vector<int>>& matrix, vector<vector<int>>& result, vector<vector<int>>& visited,
                   int& dis, int x, int y, int row, int col, bool& found) {
        if (found) {
            return ;
        }

        dis++;
        visited[x][y] = 1;

        if (!matrix[x][y]) {
            result[x][y] = dis;
            found = true;
            return ;
        }
        else {
            for (auto temp : layouts) {
                int newx = x + temp[0];
                int newy = y + temp[1];

                if (found) {
                    return ;
                }

                if (newx < 0 || newy < 0 || newx >= row || newy >= col || visited[newx][newy]) {
                    continue;
                }

                DFSHelper(matrix, result, visited, dis, newx, newy, row, col, found);

                if (found) {
                    return ;
                }
            }
        }
        dis--;
        visited[x][y] = 0;
    }

    /**
     * 将二维数组中为0的加入到队列中，1的则置为－1
     * 因为必然存在和1的元素相邻的元素0，所以当找到这样的周围是1的0时
     * 则把这个1的元素同样加入到队列中，因为可能会存在被1包围的1
     * 同时设置其距离，这个则需要将其值设为初始元素的值（同样是1的元素）＋1，
     * 这个时候其的值不再是-1，同时起到了纪录其已经访问过了的作用
     *
     * 可以理解为如果上一个（初始位置）如果是0，则说明其在周围，自然为1
     * 但是也会碰到被1包围的1，同样根据上面计算出来的1去计算后面的1的元素
     * 有些dp的思想
     *
     * @param matrix
     * @return
     */
    vector<vector<int>> updateMatrix3(vector<vector<int>>& matrix) {
        vector<vector<int>> result(matrix);
        if (matrix.size() == 0) {
            return result;
        }

        size_t row = matrix.size();
        size_t col = matrix[0].size();

        typedef pair<int, int> tp;

        deque<pair<int, int>> queues;
        for (size_t i = 0; i < row; i++) {
            for (size_t j = 0; j < col; j++) {
                if (matrix[i][j] == 0) {
                    queues.push_back(tp(i, j));
                }
                else {
                    result[i][j] = -1;
                }
            }
        }

        while (!queues.empty()) {
            auto content = queues.front();
            queues.pop_front();

            for (auto temp : layouts) {
                int newx = content.first + temp[0];
                int newy = content.second + temp[1];

                if (newx >= 0 && newx < row && newy >= 0 && newy < col && result[newx][newy] == -1) {
                    result[newx][newy] = result[content.first][content.second] + 1; //注意不是自增
                    queues.push_back({newx, newy});
                }
            }
        }

        return result;
    }


    // 计算层数，并将其设置为负数，作为访问过的标记
    vector<vector<int>> updateMatrix4(vector<vector<int>>& matrix) {
        if (matrix.size() == 0) {
            return matrix;
        }

        size_t row = matrix.size();
        size_t col = matrix[0].size();

        typedef pair<int, int> tp;

        deque<pair<int, int>> queues;
        for (size_t i = 0; i < row; i++) {
            for (size_t j = 0; j < col; j++) {
                if (matrix[i][j] == 0) {
                    queues.push_back(tp(i, j));
                }
            }
        }

        int dis = 0;
        while (!queues.empty()) {
            dis++;

            int queue_len = queues.size();
            // 保证队列中的每个数都能加上基本的平方数
            for (int i = 0; i < queue_len; i++) {
                auto content = queues.front();
                queues.pop_front();

                for (auto temp : layouts) {
                    int newx = content.first + temp[0];
                    int newy = content.second + temp[1];

                    if (newx >= 0 && newx < row && newy >= 0 && newy < col && matrix[newx][newy] == 1) {
                        matrix[newx][newy] = -dis; //做标记
                        queues.push_back({newx, newy});
                    }
                }
            }
        }

        for(int i = 0; i < row; ++i){
            for(int j = 0; j < col; ++j)
                if(matrix[i][j] < 0) matrix[i][j] = -matrix[i][j];
        }

        return matrix;
    }

    void test() {
        vector< vector<int>> water = {
                {0, 0, 0},
                {0, 1, 0},
                {0, 0, 0},
        };

        vector<vector<int>> result = this->updateMatrix4(water);
        for (auto i = 0; i < result.size(); i++) {
            for (auto j = 0; j < result[0].size(); j++) {
                cout << result[i][j] << ", ";
            }
            cout << endl;
        }
    }
};

```