# 题目

[题目](https://leetcode.com/problems/zuma-game/#/description)


Think about Zuma Game. You have a row of balls on the table, colored red(R), yellow(Y), blue(B), green(G), and white(W). You also have several balls in your hand.

Each time, you may choose a ball in your hand, and insert it into the row (including the leftmost place and rightmost place). Then, if there is a group of 3 or more balls in the same color touching, remove these balls. Keep doing this until no more balls can be removed.

Find the minimal balls you have to insert to remove all the balls on the table. If you cannot remove all the balls, output -1.

Examples:

Input: "WRRBBW", "RB"
Output: -1
Explanation: WRRBBW -> WRR[R]BBW -> WBBW -> WBB[B]W -> WW

Input: "WWRRBBWW", "WRBRW"
Output: 2
Explanation: WWRRBBWW -> WWRR[R]BBWW -> WWBBWW -> WWBB[B]WW -> WWWW -> empty

Input:"G", "GGGGG"
Output: 2
Explanation: G -> G[G] -> GG[G] -> empty 

Input: "RBYYBBRRB", "YRBGB"
Output: 3
Explanation: RBYYBBRRB -> RBYY[Y]BBRRB -> RBBBRRB -> RRRB -> B -> B[B] -> BB[B] -> empty 

Note:
You may assume that the initial row of balls on the table won’t have any 3 or more consecutive balls with the same color.
The number of balls on the table won't exceed 20, and the string represents these balls is called "board" in the input.
The number of balls in your hand won't exceed 5, and the string represents these balls is called "hand" in the input.
Both input strings will be non-empty and only contain characters 'R','Y','B','G','W'.


# 思路

需要注意的是，题目已经提供了一些条件。

# 实现

```cpp
//
//

#include "../PreLoad.h"
#define MaxNum 6 //随意设定的值，只要比手中的球的数量多即可

class Solution {
public:
    /**
     * 第一种方法
     * @param board
     * @param hand
     * @return
     */
    int findMinStep(string board, string hand) {
        sort(hand.begin(), hand.end());
        int res = helper(board, hand);
        return res > hand.size() ? -1 : res;
    }

    /**
     * 遍历hand中的球，根据手中的球去找board里面的球
     * 如果board中存在连续两种一样的球，则进行递归消除
     * 或者是hand中存在连续两个一样的球，board中只有一个球的情况，同样进行递归删除
     */
    int helper(string board, string hand) {
        if (board.empty()) {
            return 0;
        }
        // board没消完，但是手中已经没有球，返回一个大值
        if (hand.empty()) {
            return MaxNum;
        }

        int res = MaxNum;
        for (int i = 0; i < hand.size(); i++) {
            int j = 0;
            int n = board.size();
            while (j < n) {
                // 找hand中的元素，不存在则返回
                int k = (int)board.find(hand[i], j);
                if (k == string::npos) {
                    break;
                }
                // board中存在两个hand[i]
                if (k < n-1 && board[k] == board[k+1]) {
                    string next_board = nextStr(board.substr(0, k) + board.substr(k+2));
                    // 已经没有什么可以消的了
                    if (next_board.empty()) {
                        return 1;
                    }
                    string next_hand = hand;
                    next_hand.erase(next_hand.begin() + i);
                    res = min(res, helper(next_board, next_hand) + 1);
                    k++; //因为下面会移动一位，所以这里要提前移动一位
                }
                // board中不能消hand[i]，但是hand中存在两个hand[i]
                else if (i < hand.size()-1 && hand[i] == hand[i+1]) {
                    string next_board = nextStr(board.substr(0, k) + board.substr(k+1));
                    if (next_board.empty()) {
                        return 2;
                    }
                    string next_hand = hand;
                    next_hand.erase(i, 2);
                    res = min(res, helper(next_board, next_hand) + 2);
                }
                j = k+1;
            }
        }

        return res;
    }

    // 删除字符串中连续的重复的3个以上的同样的字母
    string nextStr(string str) {
        while (!str.empty()) {
            int start = 0;
            bool isremove = false;
            for (int i = 0; i <= str.length(); i++) {
                if (i == str.length() || str[i] != str[start]) {
                    if (i >= start+3) {
                        isremove = true;
                        str = str.substr(0, start) + str.substr(i);
                        break;
                    }
                    start = i;
                }
            }

            if (!isremove) {
                break;
            }
        }

        return str;
    }

    int findMinStep2(string board, string hand) {
        sort(hand.begin(), hand.end());
        int res = INT_MAX;
        map<char, int> count;
        for (char c : hand) count[c]++;
        helper2(board, count, hand.size(), res, 0);
        return res == INT_MAX  ? -1 : res;
    }

    /**
     * 大体上和上面的思想是类似的
     *
     * @param board
     * @param count
     * @param res
     * @param idx
     */
    void helper2(string board, map<char, int>& count, int numOfBalls, int& res, int cur) {
        // 不可能存在小于0的情况，因为就是从1开始的，也不能消除次数大于手中的球的数量，说明没消除干净
        if (res <= 0 || cur > numOfBalls) {
            return ;
        }

        for (int i = 0; i < board.size(); ) {
            // 存在不相等的情况，或者是board中只有一个元素的话，避免出现i+1越界，所以要写在前面
            // 同时这里也说明如果存在连续相等的元素的话，那么其连续的最后一位的位置则为i，因为i+1就不再想等了
            if (i < board.size()-1 && board[i] == board[i+1]) {
                if (count[board[i]]) {
                    count[board[i]]--;
                    string new_board = board;
                    new_board.insert(new_board.begin() + i, board[i]);
                    new_board = nextStr(new_board);
                    if (new_board.size()) {
                        // 加一是因为已经使用了hand中的元素消除了一个，所以需要记录下来
                        helper2(new_board, count, numOfBalls, res, cur + 1);
                    } else {
                        res = min(res, cur + 1);
                    }
                    // 恢复
                    count[board[i]]++;
                    i++;
                }
            }
            else {
                if (count[board[i]] >= 2) {
                    count[board[i]] -= 2;
                    string new_board = board;
                    new_board.insert(new_board.begin() + i, board[i]);
                    new_board.insert(new_board.begin() + i, board[i]);
                    new_board = nextStr(new_board);
                    if (new_board.size()) {
                        // 加一是因为已经使用了hand中的元素消除了一个，所以需要记录下来
                        helper2(new_board, count, numOfBalls, res, cur + 2);
                    } else {
                        res = min(res, cur + 2);
                    }
                    // 恢复
                    count[board[i]] += 2;
                }
            }
            i++;
        }
    }

    /**
     * 思想都是类似的，都是消除
     */
    int helper3(string board, string hand) {
        // 没有消除完
        if (hand.size() == 0 && board.size() != 0) {
            return MaxNum;
        }
        else if (board.size() == 0) {
            return 0;
        }

        int res = MaxNum;
        sort(hand.begin(), hand.end());

        for (int i = 0; i < hand.size(); i++) {
            // 不允许出现相等的
            if (i > 0 && hand[i] == hand[i - 1]) {
                continue;
            }

            // 生成新的字符串
            string newhand = hand.substr(0, i) + hand.substr(i+1);

            // 遍历
            for (int j = 0; j < board.size(); j++) {
                // 如果不能消除或者是前面的和后面的值相同，则继续
                if (board[j] != hand[i] || (j > 0 && board[j] == board[j-1])) {
                    continue ;
                }

                int k = j;
                // 统计相同的球的个数
                while (k < board.size()-1 && board[k] == board[k+1]) {
                    k++;
                }

                //存在两个以上的元素
                if (k - j >= 2) {
                    // 左右位置的下标
                    int l = j - 1, r = k;

                    while (l > 0 && r < board.size() && board[l] == board[r]) {
                        // 判断前后面是否还存在相等的元素，如果不存在则退出
                        if ((l > 0 && board[l - 1] == board[l]) || (r < board.size()-1 && board[r + 1] == board[r])) {
                            // 判断前面是否还存在相等的
                            while (l > 0 && board[l - 1] == board[l]) {
                                l--;
                            }
                            // 判断后面是否还存在相等的
                            while (r < board.size()-1 && board[r + 1] == board[r]) {
                                r++;
                            }
                            l--;
                            r++;
                        }
                        else {
                            break;
                        }
                    }

                    // 消除的结果
                    string newboard = board.substr(0, l + 1) + board.substr(r);

                    int nres = helper3(newboard, newhand);
                    res = min(res, nres + 1);
                }
                else {
                    // 没有什么可以消除的，往字符串中添加新元素
                    string newboard = board.substr(0, j) + hand[i] + board.substr(j);

                    int nres = helper3(newboard, newhand);
                    res = min(res, nres + 1);
                }
            }
        }

        return res;
    }

    /**
     *
     * @param board
     * @param hand
     * @return
     */
    int findMinStep3(string board, string hand) {
        int res = helper3(board, hand);
        return res == MaxNum ? -1 : res;
    }

    void test() {
        string board = "WRRBBW", hand = "RB";
        cout << findMinStep3(board, hand) << endl;

//        string result = board.substr(0, 3) + board.substr(6);
//        board.erase(3, 3);
    }
};

```

# 总结

我写了好几种方法，但是其实其的本质是类似的，你可以把它想象成将hand中的球作为起始结点，以board加入hand中的球后消除得到新的board作为下一步的扩展结点，board被消除完或者是没被消除完。