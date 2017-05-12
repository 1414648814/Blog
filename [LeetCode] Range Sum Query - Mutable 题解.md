# 题目

[题目](https://leetcode.com/problems/range-sum-query-mutable/#/description)

# 思路

一看就是单点更新和区间求和，故用线段树做。

一开始没搞清楚，题目给定的i是从0开始还是从1开始，还以为是从1开始，导致后面把下标都改掉了，还有用区间更新的代码去实现单点更新，虽然两者思路是一样的，但是导致TLE，因为区间会把所有都递归一遍，加了个判断，就ok了。

```cpp
if (idx <= middle) {
    this->updateHelper(curIdx << 1, leftIdx, middle, idx, val);
}
else {
    this->updateHelper((curIdx << 1) | 1, middle+1, rightIdx, idx, val);
}

```

# 实现

```cpp
//

#include "../PreLoad.h"

class Solution {
public:
    class NumArray {
    public:
        struct Node {
            int val;
            int sum;
        };

        vector<Node> nodes;
        vector<int> nums;

        NumArray(vector<int> nums) {
            this->nums = nums;
            this->nodes.reserve(4 * nums.size());
            for (int i = 1; i <= 4 * nums.size(); i++) {
                Node node;
                node.val = 0;
                node.sum = 0;
                this->nodes.push_back(node);
            }

            this->buildTree(1, 1, (int)nums.size());
        }

        // 单点更新
        void update(int i, int val) {
            if (i < 0 || i > this->nums.size()) {
                return ;
            }

            this->updateHelper(1, 1, (int)this->nums.size(), i+1, val);
            this->nums[i] = val;
        }

        int sumRange(int i, int j) {
            if (i > j) {
                return 0;
            }

            return this->sumHelper(1, 1, (int)this->nums.size(), i+1, j+1);
        }

    protected:
        void buildTree(int curIdx, int leftIdx, int rightIdx) {
            if (leftIdx == rightIdx) {
                this->nodes[curIdx].val = this->nums[leftIdx-1];
                this->nodes[curIdx].sum = this->nums[leftIdx-1];
                return ;
            }
            else if (leftIdx > rightIdx) {
                return ;
            }

            int middle = (leftIdx + rightIdx) / 2;
            this->buildTree(curIdx << 1, leftIdx, middle);
            this->buildTree((curIdx << 1) | 1, middle+1, rightIdx);

            this->updateFromSon(curIdx);
        }

        void updateFromSon(int curIdx) {
            int leftIdx = curIdx << 1;
            int rightIdx = leftIdx | 1;

            this->nodes[curIdx].sum = this->nodes[leftIdx].sum + this->nodes[rightIdx].sum;
        }

        int sumHelper(int curIdx, int leftIdx, int rightIdx, int leftRange, int rightRange) {
            // 不在范围内
            if (leftIdx > rightRange || rightIdx < leftRange) {
                return 0;
            }

            // 在范围内
            if (leftIdx >= leftRange && rightIdx <= rightRange) {
                return this->nodes[curIdx].sum;
            }

            int middle = (leftIdx + rightIdx) / 2;
            int left = sumHelper(curIdx << 1, leftIdx, middle, leftRange, rightRange);
            int right = sumHelper((curIdx << 1) | 1, middle+1, rightIdx, leftRange, rightRange);
            return left + right;
        }

        void updateHelper(int curIdx, int leftIdx, int rightIdx, int idx, int val) {
            if (leftIdx > rightIdx) {
                return;
            }

            if (leftIdx == rightIdx) {
                if (idx == leftIdx) {
                    this->nodes[curIdx].val = val;
                    this->nodes[curIdx].sum = val;
                }
                return ;
            }

            int middle = (leftIdx + rightIdx) / 2;
            if (idx <= middle) {
                this->updateHelper(curIdx << 1, leftIdx, middle, idx, val);
            }
            else {
                this->updateHelper((curIdx << 1) | 1, middle+1, rightIdx, idx, val);
            }

            this->updateFromSon(curIdx);
        }
    };

    void test() {
        vector<int> nums = {7, 2, 7, 2, 0};

        NumArray *obj = new NumArray(nums);
        int idx, val;
        while (cin >> idx >> val) {
            obj->update(idx, val);

            int sum = obj->sumRange(0, 4);
            cout << "sum： " << sum << endl;
        }
    }
};
/**
 * Your NumArray object will be instantiated and called as such:
 * NumArray obj = new NumArray(nums);
 * obj.update(i,val);
 * int param_2 = obj.sumRange(i,j);
 */
```