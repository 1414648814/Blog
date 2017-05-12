# 前言

之前把一些LeetCode里面的题目的思路写在了本子上，现在把这些全都放到博客上，以后翻阅比较方便。

# 题目

## 99.Recover Binary Search Tree

### 题意

Two elements of a binary search tree (BST) are swapped by mistake.

Recover the tree without changing its structure.

恢复两个因错误而导致的左右子树。

### 思路

通常我们的做法是，根据BST的特质，可以用中序遍历来检查数值的变化，找到错误的左右子树。

中序遍历分为递归和非递归，其中非递归版本除了熟知的栈＋迭代的版本；

其实非递归还有另外一个算法－Morris Traversal。这个算法的好处在于，1⃣️O(1)的空间，常数空间；2⃣️二叉树的形状不会被更改（中间允许改变形状）。**其实就是增加纪录了叶子结点的前驱结点和后继结点（这样可以很便利的访问到某个叶子结点的上一个结点和下一个结点），用时间换取空间。**比如，这样做就可以从左子树的最右结点到达跟节点。

### 实现

```cpp
**
 *  使用了Morris traversal 算法， 花费常量内存空间
 *
 *  @param root <#root description#>
 */
void recoverTree3(TreeNode* root) {
    TreeNode *first = NULL;
    TreeNode *second = NULL;
    TreeNode *pred = NULL;  // 前驱结点
    TreeNode *prev = NULL;  // 用来记录之前结点，访问右子树
    TreeNode *curr = root;  // 当前结点
    
    while (curr != NULL) {
        // 和前面做法一样
        if (prev != NULL && curr->val <= prev->val) {
            if (first == NULL) first = prev;
            second = curr;
        }
        
        if (curr->left != NULL) {
            pred = curr->left;
            
            // 不断遍历，找到左子树的最右结点
            while (pred->right != NULL && pred->right != curr) {
                pred = pred->right;
            }
            
            // 如果前驱结点的右孩子为当前结点，将它的右孩子重新设置为空（恢复树的形状）
            // 当前结点更新为当前结点的右孩子
            if (pred->right == curr) {
                pred->right = NULL;
                prev = curr;
                curr = curr->right;
            }
            else {
                // 如果前驱结点的右孩子为空，将它的右孩子设置为当前结点。
                // 当前结点更新为当前结点的左孩子
                pred->right = curr;
                curr = curr->left;
            }
        }
        else {
            //访问右子树
            prev = curr;
            curr = curr->right;
        }
    }
    
    int temp = first->val;
    first->val = second->val;
    second->val = temp;
}

```

具体可参考 [博客](http://www.cnblogs.com/AnnieKim/archive/2013/06/15/MorrisTraversal.html)


## 105. Construct Binary Tree from Preorder and Inorder Traversal

### 题意

根据给定的前序和中序遍历构造BST

### 思路

- 前序：中左右
- 中序：左中右
- 后序：左右中

利用中序找到中间节点（需要操作的点），前序则用来确定左右子结点。同样的，因为是DFS，所以同样可以用递归和非递归实现。

## 106. Construct Binary Tree from Inorder and Postorder Traversal

同上,需要注意边界问题。

## 108. Convert Sorted Array to Binary Search Tree

### 思路

根据中序遍历的特点，递归或者取得数组的中间点作为父结点。


## 109. Convert Sorted List to Binary Search Tree   

### 思路

和108题目类似。注意这里是链表。

## 110. Flatten Binary Tree to Linked List

### 思路

注意将左子树的最右结点连接到跟节点上。

## 116. Populating Next Right Pointers in Each Node

### 题意

为每个结点加上兄弟节点。

### 思路

- 迭代：先找到树的每一层的最左边的结点，再往右进行迭代，直到右边不存在结点时则设置为null。
- 递归：专注于处理兄弟结点的处理，因为递归从上至下访问到每一个结点。

### 实现

```cpp
/**
 *  两个迭代
 *  一个是移动当前结点，一个是设置右子结点
 *
 *  @param root <#root description#>
 */
void connect(TreeLinkNode *root) {
    if (root == NULL)
        return ;
    TreeLinkNode * pre = root;
    TreeLinkNode * cur = NULL;
    
    // 访问每一层的最左子结点，也就是每一层的第一个子结点
    while (pre->left) {
        cur = pre;
        while (cur) {
            cur->left->next = cur->right;
            // 不同子树之间的连接
            if (cur->next)
                cur->right->next = cur->next->left;
            cur = cur->next;
        }
        pre = pre->left;
    }
}

```

## 117. Populating Next Right Pointers in Each Node II

### 题意

和前面的题目不同的点在于，这里的二叉树不再是平衡二叉树，而是任何的二叉树。

### 思路

无法保证右边结点是否存在，不能是遍历这一层的同时连接着一层，而是遍历这一层连接下一层。

### 实现

```cpp
/**
 *  start作为父亲结点，通过end结点移动，如果start有子结点，则让end连接起来
 *  levelEnd作为每一层的最右结点，也就是结束的结点，将其next设置为空
 *
 *  @param root <#root description#>
 */
void connect(TreeLinkNode *root) {
    if (root == NULL) return ;
    TreeLinkNode * start = root;
    TreeLinkNode * end = root;
    TreeLinkNode * levelEnd = root;
    
    while (start != NULL) {
        // 指向左子节点
        if (start->left != NULL) {
            end->next = start->left;
            end = end->next;
        }
        
        // 指向右子节点
        if (start->right != NULL) {
            end->next = start->right;
            end = end->next;
        }

        // 判断是否到右边最后的结点，否则一直向右移动 
        if (start == levelEnd) {
            start = start->next;
            levelEnd->next = NULL;
            levelEnd = end;
        }
        else {
            start = start->next;
        }
    }
}
```

下面3个结点：

- start 作为移动的点
- end 根据start是否有子结点，并将其每个子结点连接起来。
- levelEnd 纪录每一层的最后一个结点，用来和start进行判断是否已经到该层的最后一个结点，否则start移动，用end去更新levelEnd。


## 133. Clone Graph

### 题意

克隆无向图

### 思路

每个结点都维护着自己的邻居结点。因为是无向图，需要在迭代或者递归的时候避免重复访问无向图的结点，所以所有方法都需要维护着一个全局的unordered_map的哈希表，纪录访问过的结点。

- DFS迭代
- DFS递归
- BFS迭代
- BFS递归


## 207. Course Schedule

### 题意

题意给定了一系列的课程，而且要求上完一门课程才可以上下一门课程。判断是否能够上完题目给定的所有课程。

[题目](https://leetcode.com/problems/course-schedule/?tab=Description)

### 思路

这可以理解成一副有向图，对于这种要求有向图进行排序，每个顶点只能输出一次，不允许出现出现环，可以使用拓扑排序这种图论算法。

算法同样可以通过
- BFS实现
- DFS实现

注意图的三种表达方式
- 边表示法
- 邻接表法
- 邻接矩阵法

1. 初始化邻接表（根据要求），并且填满邻接表；
2. 计算每个结点的前驱结点的个数；
3. 每次找到一个没有前驱结点，也就是入度为0；
4. 把它指向其他结点的边去掉，减去出度的个数；
5. 直到所有结点都已经找到，或者没有符合条件的结点。


### 实现

```cpp
/**
 *  判断有向图是否有环
 *  通过DFS找环
 *
 *  @param matrix  <#matrix description#>
 *  @param visited <#visited description#>
 *  @param idx     <#idx description#>
 *  @param flag    <#flag description#>
 *
 *  @return <#return value description#>
 */
bool DFS(vector<unordered_set<int>> &matrix, unordered_set<int> &visited, int idx, vector<bool> &flag) {
    flag[idx] = true; // 标记该结点访问过
    visited.insert(idx);
    
    // 找出该结点的所有邻居结点，如果存在访问过的结点或者递归，则返回true
    for (auto it = matrix[idx].begin(); it != matrix[idx].end(); ++it) {
        if (visited.find(*it) != visited.end() || DFS(matrix, visited, *it, flag)) {
            return true;
        }
    }
    
    visited.erase(idx);
    return false;
}
bool canFinish(int numCourses, vector<pair<int, int>>& prerequisites) {
    vector<unordered_set<int>> matrix(numCourses);
    // 要想完成第一门课程，则先完成第二门课程(后面的是先要完成的课程)
    // 构建图
    for (int i = 0; i < prerequisites.size(); ++i) {
        matrix[prerequisites[i].second].insert(prerequisites[i].first);
    }
    
    unordered_set<int> visited; // 记录一个递归访问过的结点
    vector<bool> flag(numCourses, false); // 记录是否访问过结点
    
    /**
     *  遍历所有课程，也就是结点
     *  判断是否标记过结点，如果没有则进行DFS判断是否存在回路，存在回路则返回false
     */
    for (int i = 0; i < numCourses; ++i) {
        if (!flag[i])
            // 如果递归中存在访问过的结点，则该拓扑排序是不存在的，也就无法完成课程
            if (DFS(matrix, visited, i, flag))
                return false;
    }
    return true;
}

/**
 *  BFS的拓扑排序实现
 *
 *  @param numCourses      <#numCourses description#>
 *  @param vector<pair<int <#vector<pair<int description#>
 *  @param prerequisites   <#prerequisites description#>
 *
 *  @return <#return value description#>
 */
bool canFinish2(int numCourses, vector<pair<int, int>>& prerequisites) {
    vector<unordered_set<int>> matrix(numCourses);
    for (int i = 0; i < prerequisites.size(); i++) {
        matrix[prerequisites[i].second].insert(prerequisites[i].first);
    }
    
    vector<int> d(numCourses, 0);
    // 计算每个结点的入度
    // 增加每个结点的邻居结点的个数
    for (int i = 0; i < numCourses; ++i)
        for (auto it = matrix[i].begin(); it != matrix[i].end(); ++it)
            ++d[*it];;
    
    // 如果找不到，则返回不能完成
    for (int i = 0, j; i < numCourses; ++i) {
        for (j = 0; j < numCourses && d[j] != 0; ++j);
        if (j == numCourses)
            return false;
        
        d[j] = -1;
        for (auto it = matrix[j].begin(); it != matrix[j].end(); ++it)
            --d[*it];
    }
    return true;
}

```

## 210. Course Schedule II

### 题目

在前面的基础上，纪录下上完所有课程的顺序。

[题目](https://leetcode.com/problems/course-schedule-ii/?tab=Description)

### 思路

加个数组进行纪录路径。算法和之前一样。

### 实现

```cpp
/**
 *  拓扑排序的DFS实现，思想和1类似
 *
 *  @param graph    <#graph description#>
 *  @param node     <#node description#>
 *  @param onpath   <#onpath description#>
 *  @param visited  <#visited description#>
 *  @param toposort <#toposort description#>
 *
 *  @return <#return value description#>
 */
bool DFS(vector<unordered_set<int>>& graph, int node, vector<bool>& onpath, vector<bool>& visited, vector<int>& toposort) {
    // 标记该结点被访问过
    onpath[node] = visited[node] = true;
    
    for (int neigh : graph[node]) {
        if (onpath[neigh] || DFS(graph, node, onpath, visited, toposort))
            return true;
    }
    toposort.push_back(node);
    return onpath[node] = false;
}
vector<int> findOrder(int numCourses, vector<pair<int, int>>& prerequisites) {
    vector<unordered_set<int>> graph(numCourses);
    // 构建临接表
    for (auto pre : prerequisites)
        graph[pre.second].insert(pre.first);
    
    vector<bool> onpath(numCourses, false), visited(numCourses, false);
    vector<int> result;
    
    for (int i = 0; i < numCourses; i++) {
        if (!visited[i] && DFS(graph, i, onpath, visited, result))
            return {};
    }
    reverse(result.begin(), result.end());
    return result;
}

/**
 *  拓扑排序的BFS实现，思想和1类似
 *
 *  @param graph <#graph description#>
 *
 *  @return <#return value description#>
 */
vector<int> compute_indegree(vector<unordered_set<int>> &graph) {
    vector<int> degress(graph.size(), 0);
    for (auto neighbors : graph)
        for (int neigh : neighbors)
            degress[neigh] ++;
    return degress;
}

vector<int> findOrder2(int numCourses, vector<pair<int, int>>& prerequisites) {
    vector<unordered_set<int>> graph(numCourses);
    // 构建临接表
    for (auto pre : prerequisites)
        graph[pre.second].insert(pre.first);
    
    vector<int> degress = compute_indegree(graph);
    queue<int> zeros;
    vector<int> result;
    for (int i = 0; i < numCourses; i++) {
        if (!degress[i]) zeros.push(i);
    }
    
    for (int i = 0; i < numCourses; i++) {
        if (zeros.empty()) return {};
        int zero = zeros.front();
        zeros.pop();
        
        result.push_back(zero);
        for (int neigh : graph[zero]) {
            if (!--degress[neigh])
                zeros.push(neigh);
        }
    }
    
    return result;
}

```

## 257. Binary Tree Paths

### 题目

[题目](https://leetcode.com/problems/binary-tree-paths/?tab=Description)

返回从根节点到叶结点的所有路径。

### 思路

- 递归：更新路径字符串
- 迭代＋栈：使用了两个栈，一个用来存储当前结点，一个用来存储到达当前结点的路径字符串。

### 实现

```cpp
/**
 *  递归将更新字符串，到达叶结点时加入到数组中
 *
 *  @param result <#result description#>
 *  @param root   <#root description#>
 *  @param t      <#t description#>
 */
void binaryTreePaths(vector<string>& result, TreeNode* root, string t) {
    if (!root->left && !root->right) {
        result.push_back(t);
        return ;
    }
    
    if (root->left) binaryTreePaths(result, root->left, t + "->" + to_string(root->left->val));
    if (root->right) binaryTreePaths(result, root->right, t + "->" + to_string(root->right->val));
    
}
vector<string> binaryTreePaths(TreeNode* root) {
    vector<string> result;
    if (!root) return result;
    
    binaryTreePaths(result, root, to_string(root->val));
    return result;
}

/**
 *  迭代＋栈实现，用了两个栈，分别存储结点以及到达该结点的路径
 *
 *  @param root <#root description#>
 *
 *  @return <#return value description#>
 */
vector<string> binaryTreePaths2(TreeNode* root) {
    vector<string> result;
    if (root == NULL) return result;
    
    stack<TreeNode *> t_stack;
    stack<string> s_stack;
    t_stack.push(root);
    s_stack.push(to_string(root->val));
    
    while (!t_stack.empty()) {
        TreeNode * curNode = t_stack.top();
        t_stack.pop();
        
        string curPath = s_stack.top();
        s_stack.pop();
        
        if (!curNode->left && !curNode->right) {
            result.push_back(curPath);
            continue;
        }
        
        if (curNode->left) {
            t_stack.push(curNode->left);
            s_stack.push(curPath + "->" + to_string(curNode->left->val));
        }
        
        if (curNode->right) {
            t_stack.push(curNode->right);
            s_stack.push(curPath + "->" + to_string(curNode->right->val));
        }
    }
    
    return result;
}

```


## 其他

set和multiset都是集合类，差别在于set中不允许有重复元素，multiset中允许有重复元素，内部都以平衡二叉树实现。

1. 对于树的前序遍历（根左右），有效代码（处理函数）写在递归左子树之后，递归右子树之钱，也就是中间；
2. 对于树的中序遍历（左根右），则写在递归左右子树之前；
3. 对于树的后序遍历（左右根），则写在递归左右子树之后；


## 301. Remove Invalid Parentheses

### 题目

移除最小数量的无效括号，使得输入字符串有效

### 思路

- DFS＋剪枝
- BFS
- BFS＋剪枝

### 实现

```cpp
class Solution {
public:
    /**
     *  DFS＋剪枝
     *
     *  @param pair         多出来的遇见括号的个数
     *  @param index        记录字符串s的当前位置
     *  @param remove_left  左括号需要删除的个数
     *  @param remove_right 右括号需要删除的个数
     *  @param s            原始字符串
     *  @param solution     生成字符串
     *  @param result       存储所有的字符串结果
     */
    void helper(int pair, int index, int remove_left, int remove_right, const string& s, string solution, unordered_set<string> &result) {
        if (index == s.size()) {
            if (pair == 0 && remove_left == 0 && remove_right == 0)
                result.insert(solution);
            return ;
        }
        
        if (s[index] == '(') {
            // 删除左边括号
            if (remove_left > 0) helper(pair, index + 1, remove_left - 1, remove_right, s, solution, result);
            // 回溯
            helper(pair + 1, index + 1, remove_left, remove_right, s, solution + s[index], result);
        }
        else if (s[index] == ')') {
            // 删除右边括号
            if (remove_right > 0) helper(pair, index + 1, remove_left, remove_right - 1, s, solution, result);
            // 回溯
            if (pair > 0) helper(pair - 1, index + 1, remove_left, remove_right, s, solution + s[index], result);
        }
        else {
            helper(pair, index + 1, remove_left, remove_right, s, solution + s[index], result);
        }
    }
    vector<string> removeInvalidParentheses(string s) {
        int remove_left = 0, remove_right = 0;
        unordered_set<string> result; // 处理重复
        
        // 计算左右两边需要删除括号的个数
        for (int i = 0; i < s.size(); ++i) {
            if (s[i] == '(')
                remove_left++;
            else if (s[i] == ')') {
                if (remove_left > 0) remove_left--;
                else remove_right++;
            }
        }
        
        helper(0, 0, remove_left, remove_right, s, "", result);
        
        return vector<string>(result.begin(), result.end());
    }
    
    bool check(string s) {
        int count = 0;
        for (int i = 0; i < s.size(); i++) {
            char tmp = s[i];
            if (tmp == '(') count++;
            if (tmp == ')') {
                if (count == 0) return false;
                count--;
            }
        }
        return count == 0;
    }

    /**
     *  这个解法看似和上面解法类似，以为是DFS，但是其实是个BFS，思路和2是类似的，只不过用递归实现
     *  删除子串的每个字符，然后进行递归
     *
     *
     *  @param begin        记录原字符串的下标，为什么是BFS的原因
     *  @param remove_left  需要删除左边括号的个数
     *  @param remove_right 需要删除右边括号的个数
     *  @param s            子串
     *  @param result       结果
     */
    void helper2(int begin, int remove_left, int remove_right, string s, vector<string> &result) {
        // 如果左右已经没有要删除的括号，并且符合条件，则进行收敛
        if (remove_left == 0 && remove_right == 0) {
            if (check(s)) {
                result.push_back(s);
                return;
            }
        }
        
        // begin是个重点，意味着从begin开始往后删，前面的字符串不再动
        for (int i = begin; i < s.size(); ++i) {
            string temp = s;
            if (remove_left > 0 && remove_right == 0 && s[i] == '(') {
                // 删除子串的每个字符，同时避免重复
                if (begin == i || s[i] != s[i-1]) {
                    temp.erase(i, 1);
                    helper2(i, remove_left-1, remove_right, temp, result);
                }
            }
            if (remove_right > 0 && s[i] == ')') {
                if (begin == i || s[i] != s[i-1]) {
                    temp.erase(i, 1);
                    helper2(i, remove_left, remove_right-1, temp, result);
                }
            }
        }
        
    }
    vector<string> removeInvalidParentheses3(string s) {
        int remove_left = 0, remove_right = 0;
        vector<string> result; // 处理重复
        
        // 计算左右两边需要删除括号的个数
        for (int i = 0; i < s.size(); ++i) {
            if (s[i] == '(')
                remove_left++;
            else if (s[i] == ')') {
                if (remove_left > 0) remove_left--;
                else remove_right++;
            }
        }
        
        helper2(0, remove_left, remove_right, s, result);
        
        return result;
    }
    
    
    /**
     *  通过从输入字符串中移除每一个括号，生成新的字符串加入到队列中
     *  如果从队列中取出的字符串是有效的，则加入到结果列表中
     *  一旦发现有效的字符串，则不再向队列中补充新的字符串，其去掉的括号数一定是最小的
     *  而此时，队列中存在的元素与队列头元素去掉的括号数的差值 <= 1
     *  并且，只有与队列头元素去掉括号数目相同的元素才有可能是候选答案
     *
     *  @param s <#s description#>
     *
     *  @return <#return value description#>
     */
    vector<string> removeInvalidParentheses2(string s) {
        vector<string> result;
        if (s == "") {
            result.push_back(s);
            return result;
        }
        
        unordered_set<string> visited; // 控制是否访问过字符串，因为要求不可重复
        deque<string> queue;
        queue.push_back(s);
        visited.insert(s);
        
        bool found = false;
        while (!queue.empty()) {
            string temp = queue.front();
            queue.pop_front();
            
            // 每次遍历代表着需要加进来去掉一个括号的子串，层数代表删除括号的次数
            // 只要第一次符合情况了，说明该字符串已经是可去掉括号数目最小的字符串层
            // 意味着该层不再需要加进来任何的子串了，
            if (check(temp)) {
                result.push_back(temp);
                found = true;
            }
            
            if (found) continue;
            for (int i = 0; i < temp.size(); ++i) {
                if (temp[i] != '(' && temp[i] != ')') continue;
                // 删除括号，生成新的字符串
                string str = temp.substr(0, i) + temp.substr(i+1);
                
                if (visited.find(str) == visited.end()) {
                    queue.push_back(str);
                    visited.insert(str);
                }
            }
        }
        
        return result;
    }
    
    bool test() {
        string s = "(a)b)c)d";
        vector<string> result = removeInvalidParentheses3(s);
        cout << result.size() << endl;
        return true;
    }
};

```

这道题很好，抓在手里。

## 337. House Robber III

### 题意

[题目](https://leetcode.com/problems/house-robber-iii/?tab=Description)

大致是盗贼在不惊动警察的情况下（同时偷相邻层的情况下最多可以偷到的金钱数目，不允许偷相连，比如父结点和子结点）

### 思路

- DFS：每次遍历返回两个值：分别表示偷窃或者不偷窃当前结点可以获得的最大收益，需要额外的空间维护；
- DFS＋Memorization 记忆化搜索：计算子节点收益（当前结点不偷）和孙子结点收益加上当前结点收益（当前结点偷）

### 实现

```cpp
class Solution {
public:
    
    /**
     *  超时
     *  此结点的最大值有两种情况，一种是该结点值加上不包含左子结点
     *  和右子结点的返回值之和，另一种是左右子结点返回值之和不包含
     *  当前结点值，取两者的较大值返回即可；
     *
     *  @param root <#root description#>
     *
     *  @return <#return value description#>
     */
    int rob(TreeNode* root) {
        if (!root) {
            return 0;
        }
        int val = 0;
        if (root->left) {
            val += rob(root->left->left) + rob(root->left->right);
        }
        if (root->right) {
            val += rob(root->right->left) + rob(root->right->right);
        }
        return max(val + root->val, rob(root->left) + rob(root->right));
    }
    
    /**
     *  和上面做法相同，增加一个哈希表保存之前计算过的结点
     *
     *  @param root                   <#root description#>
     *  @param unordered_map<TreeNode <#unordered_map<TreeNode description#>
     *  @param map                    <#map description#>
     *
     *  @return <#return value description#>
     */
    int DFS(TreeNode * root, unordered_map<TreeNode*, int>& map) {
        if (!root)
            return 0;
        if (map.count(root))
            return map[root];
        int val = 0;
        if (root->left) {
            val += DFS(root->left->left, map) + DFS(root->left->right, map);
        }
        if (root->right) {
            val += DFS(root->right->left, map) + DFS(root->right->right, map);
        }
        val = max(val + root->val, DFS(root->left, map) + DFS(root->right, map));
        map[root] = val;
        return val;
    }
    int rob2(TreeNode * root) {
        unordered_map<TreeNode*, int> map;
        return DFS(root, map);
    }
    
    
    /**
     *  DFS + DP
     *  用的是大小为2的数组，分别保存不包含当前结点值的最大值和当前值的最大值
     *
     *  @param root <#root description#>
     *
     *  @return <#return value description#>
     */
    vector<int> robSub(TreeNode* root) {
        if (root == NULL) {
            return vector<int>(2, 0);
        }
        
        vector<int> left = robSub(root->left);
        vector<int> right = robSub(root->right);
        
        vector<int> result(2, 0);
        // 当前结点不偷的情况下，取左右子结点偷或不偷的最大收益
        result[0] = max(left[0], left[1]) + max(right[0], right[1]);
        // 当前结点偷的情况下，取左右子结点不偷的情况的收益
        result[1] = root->val + left[0] + right[0];
        
        return result;
    }
    int rob3(TreeNode* root) {
        vector<int> result  = robSub(root);
        // 取出的是跟结点里面偷和不偷的最大收益
        return max(result[0], result[1]);
    }
    
    /**
     *  DFS + 记忆化
     *
     *  @param root <#root description#>
     *  @param l    <#l description#>
     *  @param r    <#r description#>
     *
     *  @return <#return value description#>
     */
    int tryRob(TreeNode* root, int& l, int& r) {
        if (!root)
            return 0;
        int ll = 0, lr = 0, rl = 0, rr = 0;
        l = tryRob(root->left, l, r);
        r = tryRob(root->right, l, r);
        
        // 前者为偷当前结点，所以加上孙结点的收益
        // 后者为不偷当前结点，所以加上子结点的收益
        return max(root->val + ll + lr + rl + rr, l + r);
    }
    int rob4(TreeNode* root) {
        int l, r;
        return tryRob(root, l, r);
    }
};

```

需要另外学习记忆化搜索的知识。

## 329. Longest Increasing Path in a Matrix

### 题意

二维数组中选出一条递增的路径，返回路径的最大个数。

### 思路

- DFS＋Memorization：枚举起点，从每一个单元格出发，递归寻找最长递增路径；
- 排序＋DP

两种方法都需要借助DP数组（存储从单元格x,y出发的最长递增序列长度）来进行辅助。

### 实现

```cpp
class Solution {
public:
    /**
     *  DFS + DP
     *
     *  @param matrix <#matrix description#>
     *  @param states <#states description#>
     *  @param i      <#i description#>
     *  @param j      <#j description#>
     *  @param m      <#m description#>
     *  @param n      <#n description#>
     *
     *  @return <#return value description#>
     */
    int longestPath(vector<vector<int>>& matrix, vector<vector<int>>& states, int i, int j, int m, int n) {
        if (states[i][j] > 0)
            return states[i][j];
        
        int maxd = 0;
        
        // 对四个方向进行DFS，找到最长路径
        if (j > 0 && matrix[i][j-1] < matrix[i][j]) {
            int left = longestPath(matrix, states, i, j-1, m, n);
            maxd = max(maxd, left);
        }
        if (j < n-1 && matrix[i][j+1] < matrix[i][j]) {
            int right = longestPath(matrix, states, i, j+1, m, n);
            maxd = max(maxd, right);
        }
        if (i > 0 && matrix[i-1][j] < matrix[i][j]) {
            int up = longestPath(matrix, states, i-1, j, m, n);
            maxd = max(maxd, up);
        }
        if (i < m-1 && matrix[i+1][j] < matrix[i][j]) {
            int down = longestPath(matrix, states, i+1, j, m, n);
            maxd = max(maxd, down);
        }
        
        states[i][j] = maxd + 1;
        return states[i][j];
    }
    int longestIncreasingPath(vector<vector<int>>& matrix) {
        int m = matrix.size();
        if (m == 0) return 0;
        int n = matrix[0].size();
        
        int result = 0;
        vector<vector<int>> states(m, vector<int>(n, 0));
        for (int i = 0; i < m; ++i) {
            for (int j = 0; j < n; ++j) {
                result = max(result, longestPath(matrix, states, i, j, m, n));
            }
        }
        
        return result;
    }
    
    /**
     *  方法和上面类似，不过利用dirs+循环可以使函数简化
     */
    vector<vector<int>> dirs = {{1, 0}, {-1, 0}, {0, 1}, {0, -1}};
    int helper(vector<vector<int>>& matrix, vector<vector<int>>& visit, int i, int j, int m, int n) {
        if (visit[i][j] > 1) return visit[i][j];
        int result = 1;
        for (auto dir : dirs) {
            int x = i + dir[0], y = j + dir[1];
            if (x < 0 || x >= m || y < 0 || y >= n || matrix[i][j] > matrix[x][y])
                continue;
            result = max(result, helper(matrix, visit, i, j, m, n));
        }
        visit[i][j] = result;
        return result;
    }
    int longestIncreasingPath2(vector<vector<int>>& matrix) {
        int m = matrix.size();
        if (m == 0) return 0;
        int n = matrix[0].size();
        
        int result = 0;
        vector<vector<int>> visit(m, vector<int>(n, 0));
        for (int i = 0; i < m; ++i) {
            for (int j = 0; j < n; ++j) {
                result = max(result, helper(matrix, visit, i, j, m, n));
            }
        }
        
        return result;
    }
    
    /**
     *  BFS（拓扑排序）
     *  这里的最长体现于，从数组中找出所有相对最小值（四个方向）
     *  并且以每个相对最小值进行发散，看每个相对最小值能够到达的相对最大值，最远能到多远
     *  路径长度其实就是计算队列的层数，看下一层的队列，如果下一层队列还有值，说明还可以前进，层数自然也会相加
     *  这就像计算树的高度，只有还有下一层，高度就会增加
     *
     *  @param matrix <#matrix description#>
     *
     *  @return <#return value description#>
     */
    int longestIncreasingPath3(vector<vector<int>>& matrix) {
        const int m = matrix.size();
        if (m == 0) return 0;
        const int n = matrix[0].size();
        if (n == 0) return 0;
        
        // 队列存储四个方向上都小于的数，visit则记录当前点有多少个大于自己的数
        vector<vector<int>> visit(m, vector<int>(n, 0));
        queue<pair<int, int>> queue, nqueue;
        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                int result = 0;
                int cur = matrix[i][j];
                // left
                if (j-1 >= 0) result += cur > matrix[i][j-1];
                // right
                if (j+1 < n) result += cur > matrix[i][j+1];
                // up
                if (i-1 >= 0) result += cur > matrix[i-1][j];
                // down
                if (i+1 < m) result += cur > matrix[i+1][j];
                
                visit[i][j] = result;
                // 为0表示4个方向上都是递增，就是当前点比4个方向上的值都要小
                if (result == 0)
                    queue.emplace(i, j);
            }
        }
        
        // 判断四个方向上是否存在比队列（当前访问）中的值要大的数，有则加进队列nqueue(下一层访问)
        // 等待访问完当前队列，交换队列；
        // 对于在四个方向上的值，当其只存在一个比它小的值，也就是当前值，则加进来，这样保证最长长度
        int l = 0;
        while (!queue.empty()) {
            while (!queue.empty()) {
                pair<int, int> curPos = queue.front();
                queue.pop();
                
                int i = curPos.first;
                int j = curPos.second;
                
                int cur = matrix[i][j];
                
#define WALK(i, j)\
    int &result = visit[i][j];\
    if (matrix[i][j] > cur and result > 0) {\
        --result;\
        if (result == 0) {\
            nqueue.emplace(i, j);\
        }\
    }
                if (j-1 >= 0) {WALK(i,j-1)};
                if (j+1 <  n) {WALK(i,j+1)};
                if (i-1 >= 0) {WALK(i-1,j)};
                if (i+1 <  m) {WALK(i+1,j)};
#undef WALK
            }
            ++l;
            swap(queue, nqueue);
        }
        
        return l;
    }
    
    bool test() {
        vector<vector<int>> nums = {
            {9,9,4},
            {6,6,8},
            {2,1,1}
        };
        
        int result = longestIncreasingPath3(nums);
        cout << result << endl;
        
        return true;
    }
};

```

## 130. Surrounded Regions

### 题意

将围住的元素置为相同，没围住的不管

### 思路

- 并查集，判断图连通
- BFS
- DFS

### 实现

```cpp
// 并查集实现
class UF {
private:
    vector<int> id; // parent of i
    vector<int> rank; // rank of subtree rooted at i
    int count; // number of components
    
public:
    // construct
    UF(int N) {
        count = N;
        id.assign(N, 0);
        rank.assign(N, 0);
        for (int i = 0; i < N; i++) {
            id[i] = i;
        }
    }

    // free memory
    ~UF() {
        vector<int> temp;
        id.swap(temp);
        rank.swap(temp);
    }
    
    // path compression
    int find(int p) {
        while (p != id[p]) {
            id[p] = id[id[p]];
            p = id[p];
        }
        return p;
    }
    
    int getCount() {
        return count;
    }
    
    bool connected(int p, int q) {
        return find(p) == find(q);
    }
    
    void connect(int p, int q) {
        int i = find(p);
        int j = find(q);
        
        if (i == j) return ;
        if (rank[i] < rank[j]) id[i] = j;
        else if (rank[i] > rank[j]) id[j] = i;
        else {
            id[j] = i;
            rank[i]++;
        }
        count--;
    }
};

void solve3(vector<vector<char>>& board) {
    int m = board.size();
    if(m == 0) return ;
    int n = board[0].size();
    // 函数局部对象，函数返回时销毁对象
    UF uf = UF(n * m + 1);
    
    for (int i = 0; i < m; i++) {
        for (int j = 0; j < n; j++) {
            // connect to the dummy node
            if (i == 0 || i == m-1 || j == 0 || j == n-1)
                uf.connect(i * n + j, n * m);
            else if (board[i][j] == 'O') {
                if (board[i-1][j] == 'O')
                    uf.connect(i * n + j, (i-1) * n + j);
                if (board[i+1][j] == 'O')
                    uf.connect(i * n + j, (i+1) * n + j);
                if (board[i][j-1] == 'O')
                    uf.connect(i * n + j, i * n + j - 1);
                if (board[i][j+1] == 'O')
                    uf.connect(i * n + j, i * n + j + 1);
            }
        }
    }
    
    for (int i = 0; i < m; i++) {
        for (int j = 0; j < n; j++) {
            if (!uf.connected(i * n + j, n * m))
                board[i][j] = 'X';
        }
    }
    
}

```

需要另外学习并查集。


## 279. Perfect Squares

### 题意

由平方值组成的和的最小数组。

### 思路

- DP
- 静态DP
- 数学法（拉格朗日四方定理）
- BFS

### 实现

```cpp
class Solution {
public:
    /**
     *  将队列的值数值和平方数组进行一次类似二次循环
     *  找出其和等于结果的level，level每进行一次二次循环则增加1
     *  每次二次循环后队列的值都会全部更新到当前level的数值
     *  比如
     *  level = 1 的数值有 1，4，9
     *  level = 2 的数值有 2, 5, 8, 10, 13, 18
     *  ......
     *  (计算方法是自己相加或者和别人相加)
     *  level的数值其实是根据更新队列的次数
     *  队列的数值会更新，但是平方数组则不会更新
     *
     *  @param n <#n description#>
     *
     *  @return <#return value description#>
     */
    int numSquares(int n) {
        if (n <= 0)
            return 0;
        
        vector<int> squareNums;
        vector<int> dp;
        dp.assign(n, 0);
        
        for (int i = 1; i * i <= n; i++) {
            squareNums.push_back(i*i);
            dp[i*i-1] = 1;
        }
        
        if (squareNums.back() == n)
            return 1;
        
        // 队列加入平方值
        queue<int> searchQ;
        for (int i = 0; i < squareNums.size(); i++) {
            searchQ.push(squareNums[i]);
        }
        
        int level = 1;
        while (!searchQ.empty()) {
            level ++;
            // 现保存其的size，因为队列会发生变化
            int searchSize = searchQ.size();
            for (int i = 0; i < searchSize; i++) {
                int cur = searchQ.front();
                
                for (int j = 0; j < squareNums.size(); j++) {
                    
                    int temp = squareNums[j];
                    
                    if (temp + cur == n) {
                        return level;
                    }
                    else if (temp + cur < n && dp[temp + cur - 1] == 0){  //大于1表明不是第一次访问该结点
                        dp[cur + temp - 1]  = level;
                        searchQ.push(temp + cur);
                    }
                    else if (temp + cur > n) {
                        break;
                    }
                }

                searchQ.pop();
            }
        }
        return 0;
    }
    
    /**
     *  DP
     *
     *  @param n <#n description#>
     *
     *  @return <#return value description#>
     */
    int numSquares2(int n) {
        if (n <= 0)
            return 0;
        vector<int> dp(n + 1, INT_MAX);
        dp[0] = 0;
        
        for (int i = 1; i <= n; i++)
            for (int j = 1; j * j <= i; j++)
                dp[i] = min(dp[i], dp[i - j * j] + 1);
        
        return dp[n];
    }
    
    /**
     *  静态动态规划
     *
     *  @param n <#n description#>
     *
     *  @return <#return value description#>
     */
    int numSquares3(int n) {
        if (n <= 0)
            return 0;
        static vector<int> dp({0});
        
        while (dp.size() <= n) {
            int msize = dp.size();
            int curNum = INT_MAX;
            for (int i = 0; i * i <= msize; i++) {
                curNum = min(curNum, dp[msize - i*i]);
            }
            dp.push_back(curNum);
        }
        return dp[n];
    }

    /**
     *  数学法
     *  拉格朗日四方定理
     *
     *  @param n <#n description#>
     *
     *  @return <#return value description#>
     */
    int is_squre(int n) {
        int sqrt_n = (int)sqrt(n);
        return sqrt_n * sqrt_n == n;
    }
    int numSquares4(int n) {
        if (n <= 0)
            return 0;
        
        // n % 4 = 0
        while ((n & 3) == 0) {
            n >>= 2;
            
        }
        // n % 8 = 7
        if ((n & 7) == 7) {
            return 4;
        }
        
        // 检查2是否是结果
        int sqrt_n = (int)(sqrt(n));
        for (int i = 1; i <= sqrt_n; i++) {
            if (is_squre(n - i * i)) {
                return 2;
            }
        }
        
        return 3;
    }
    
    bool test() {
        int n = 11;
        int result = numSquares2(n);
        cout << result << endl;
        
        return true;
    }
};

```

需要另外学习静态DP，数学法。


## 310. Minimum Height Trees

### 题目

[题目](https://leetcode.com/problems/minimum-height-trees/?tab=Description)

从n个结点n-1条边（也就是树）中选出一个结点作为根结点，到图的其他结点路径最短（高度小）

### 思路

1. 取图的最长路径（从0开始）的中间结点。
2. 不停的删除入度为1的结点（也就是叶结点），直到不能再删，（剩下1到2个结点），取出一个即可。

- DFS
- BFS（借助拓扑排序的思想）

```cpp
struct TreeNode {
    set<int> list;  // 方便删除邻居
    TreeNode() {};
    // 如果
    bool isLeaf(){
        return list.size() == 1;
    };
};
class Solution {
public:
    vector<int> findMinHeightTrees(int n, vector<pair<int, int>>& edges) {
        if (n == 1) return {0}; // 当n为1的时候，初始化node1为空，结果为空（因为从0开始）
        
        // 用树结点存储这棵树，耗费空间为o(n+2e)
        vector<TreeNode> tree(n);
        for (auto edge : edges) {
            tree[edge.first].list.insert(edge.second);
            tree[edge.second].list.insert(edge.first);
        }
        
        vector<int> curNode(0);
        vector<int> newNode(0);
        
        // 记录叶子结点
        for (int i = 0; i < tree.size(); i++) {
            if (tree[i].isLeaf())
                curNode.push_back(i);
        }
        
        while (1) {
            //  遍历邻居删除
            for (auto leaf : curNode) {
                for (set<int>::iterator iter = tree[leaf].list.begin(); iter != tree[leaf].list.end(); iter++) {
                    int neighbor = *iter;
                    tree[neighbor].list.erase(leaf);
                    
                    // 删除结点后，如果还是叶子结点，则放到newNode中
                    if (tree[neighbor].isLeaf())
                        newNode.push_back(neighbor);
                    
                }
            }
            // 如果newNode为空，表明之前的curNode不是叶子结点，要么是剩下一个结点，要么是剩下两个相互连接的结点
            if (newNode.empty())
                break;
            curNode.clear();
            swap(curNode, newNode);
        }
        // curNode只剩下一个结点，因为邻居为空，所以不能压入newNode中
        if (curNode.size() != 0)
            return curNode;
        // curNode有两个相互连接的结点，邻居都为1，所以压入newNode中
        else if (curNode.size() == 0)
            return newNode;
        
        return {};
    }
    
    /**
     *  和上面思想相同
     *
     *  @param n               <#n description#>
     *  @param vector<pair<int <#vector<pair<int description#>
     *  @param edges           <#edges description#>
     *
     *  @return <#return value description#>
     */
    vector<int> findMinHeightTrees2(int n, vector<pair<int, int>>& edges) {
        if (n == 1)
            return {0};
        
        vector<int> result, degrees(n, 0);
        vector<vector<int>> graph(n, vector<int>());
        queue<int> queue;
        
        // 统计结点信息，保存到graph中
        for (auto edge : edges) {
            graph[edge.first].push_back(edge.second);
            graph[edge.second].push_back(edge.first);
            
            degrees[edge.first]++;
            degrees[edge.second]++;
        }
        
        // 统计入度数为1的结点，也就是叶结点
        for (int i = 0; i < degrees.size(); ++i) {
            if (degrees[i] == 1) {
                queue.push(i);
            }
        }
    
        // 删除结点，直到剩下1到2个结点，就是题目要求的最小高度的树的跟结点
        while (n > 2) {
            int qsize = queue.size();
            
            for (int i = 0; i < qsize; i++) {
                int temp = queue.front();
                queue.pop();
                --n;
                
                for (vector<int>::iterator iter = graph[temp].begin(); iter != graph[temp].end(); ++iter) {
                    --degrees[*iter];
                    
                    if (degrees[*iter] == 1)
                        queue.push(*iter);
                }
            }
        }
        
        while (!queue.empty()) {
            result.push_back(queue.front());
            queue.pop();
        }
        
        return result;
    }
    
    /**
     *  DFS
     *  思路在于，因为题目要求的是，从哪个结点开始，也就是作为跟结点，到所有其他结点的路径长度最短；
     *  所以跟结点在于最长路径的中间结点上；
     */
    int max_len;
    int v;
    void DFS(vector<vector<int>>& graph, vector<bool>& visited, vector<int>& path, int cur, int count) {
        // 将当前结点设为访问过，并且添加进路径中
        visited[cur] = true;
        count++;
        path.push_back(cur);
        
        // 访问当前结点的所有邻居结点
        for (auto it = graph[cur].begin(); it != graph[cur].end(); it++) {
            if (!visited[*it])
                DFS(graph, visited, path, *it, count);
        }
        
        if (count > max_len) {
            max_len = count;
            v = cur;
        }
        visited[cur] = false;
    }
    vector<int> findMinHeightTrees3(int n, vector<pair<int, int>>& edges) {
        vector<vector<int>> graph(n);
        // 添加邻居结点
        for (int i = 0; i < edges.size(); i++) {
            graph[edges[i].first].push_back(edges[i].second);
            graph[edges[i].second].push_back(edges[i].first);
        }
        
        vector<bool> visited(n, false);
        vector<int> path;
        
        // 计算最远路径，以及路径最后一个结点v
        DFS(graph, visited, path, 0, 0);
        
        max_len = 0;
        path.clear();
        
        // 从叶结点v开始，计算最长
        DFS(graph, visited, path, v, 0);
        
        vector<int> result;
        // 如果最长路径为偶数，取中间两个数，否则只取一个数
        if (max_len % 2 == 0) {
            result.push_back(path[max_len/2]);
            result.push_back(path[max_len/2 - 1]);
        }
        else {
            result.push_back(path[max_len/2]);
        }
        
        return result;
    }
    
    bool test() {
        int n = 3;
        vector<pair<int, int>> nodes = {
            {1, 0},
            {1, 2},
        };
        
        vector<int> result = findMinHeightTrees3(n, nodes);
        
        cout << result[0] << endl;
        
        return true;
    }
};

```

## 199. Binary Tree Right Side View

### 题意

从右边看二叉树的结点的数组

### 思想

- DFS：递归函数里纪录level变量，当level大于结果数组size时，加入当前结点值，优先遍历右子树；
- BFS：层次遍历，访问每层最右结点；
- 分治：用两个数组存储左右子树的结点，先取右边数组，大于右边数组则直取左边数组；

注意：右子树没有的时候看到的是左子树。

### 实现

```cpp
class Solution {
public:
    /**
     *  递归
     *
     *  @param root   <#root description#>
     *  @param level  <#level description#>
     *  @param result <#result description#>
     */
    void DFS(TreeNode* root, int level, vector<int> &result) {
        if (root == NULL) return ;
        // 如果当前层大于数组的值（存储每一层的最右值，代表已经访问到了第几层）
        if (level > result.size()) result.push_back(root->val);
        // 优先访问右结点
        DFS(root->right, level+1, result);
        DFS(root->left, level+1, result);
    }
    vector<int> rightSideView(TreeNode* root) {
        vector<int> result;
        DFS(root, 1, result);
        return result;
    }
    
    /**
     *  层次遍历
     *  访问每一层的结点，取最右元素
     *
     *  @param root <#root description#>
     *
     *  @return <#return value description#>
     */
    vector<int> rightSideView2(TreeNode* root) {
        vector<int> result;
        if (root == NULL) return result;
        queue<TreeNode *> queue;
        queue.push(root);
        
        while (!queue.empty()) {
            int count = (int)queue.size();
            for (int i = 0; i < count; i++) {
                TreeNode * curr = queue.front();
                queue.pop();
                // 最右结点
                if (i == count-1) {
                    result.push_back(curr->val);
                }
                if (curr->left != NULL) {
                    queue.push(curr->left);
                }
                if (curr->right != NULL) {
                    queue.push(curr->right);
                }
            }
        }
        return result;
    }
    
    /**
     *  分治法
     *
     *  @param root <#root description#>
     *
     *  @return <#return value description#>
     */
    vector<int> rightSideView3(TreeNode* root) {
        if (root == NULL) {
            return vector<int>();
        }

        //两个数组分别存储左右两颗子树的的结点
        vector<int> left = rightSideView3(root->left);
        vector<int> right = rightSideView3(root->right);;
        
        vector<int> result;
        result.push_back(root->val);
        
        //取左边和右边数组长度大的，大于右边数组的则取左边数组
        for (int i = 0; i < max(left.size(), right.size()); ++i) {
            if (i >= right.size())
                result.push_back(left.at(i));
            else
                result.push_back(right.at(i));
        }
        return result;
    }
    
    bool test() {
        TreeNode * root = new TreeNode(2);
        TreeNode * left = new TreeNode(1);
        TreeNode * right = new TreeNode(3);
        
        root->left = left;
        root->right = right;
        
        vector<int> result = rightSideView3(root);
        return true;
    }
};

```

## 其他

### 匹配

匹配分为正则表达式匹配和通配符区匹配;

前者pattern包含一些关键字，比如*的用法是紧跟在pattern的某个字符之后，表示这个字符可以出现任意多次（包括0次）；后者通配符匹配服，同样的符号可以代表匹配任意长度的任意字符组成的串。

### 全排列

1. 全排列就是第一个数字起每个数字分别与它后面的数字进行交换；
2. 去重的全排列就是从第一个数字起每个数分别与它后面非重复出现的数字进行交换；
3. 全排列的非递归实现就是由后往前找替代数和替代点，即由后往前找第一个比替换数大的数和替换数进行交换，最后颠倒替换点后的所有数据；
