
# 题目

[题目](https://leetcode.com/problems/decode-string/#/description)

```
s = "3[a]2[bc]", return "aaabcbc".
s = "3[a2[c]]", return "accaccacc".
s = "2[abc]3[cd]ef", return "abcabccdcdcdef".
```

大概意思是根据指定的格式将字符串进行展开

# 思路

* 递归的思路

递归计算出括号最里面的字符串，依次再处理外面一层的字符串，每个单元内的字符串类似于一个结点，个数则为结点的个数。父结点则是将这些个数的字符串组合在一起，以此类推到跟结点，就是我们要求的结果。

* 迭代的思路

用两个栈来分别保存下单元中的个数，另一个则保存单元中的字符串，注意的是，要将最新的处理完后的字符串加入到栈中。一直加入，直到最后返回栈顶字符串则为所求结果。

# 实现

```cpp
//
//

#include "../PreLoad.h"

/*
s = "3[a]2[bc]", return "aaabcbc".
s = "3[a2[c]]", return "accaccacc".
s = "2[abc]3[cd]ef", return "abcabccdcdcdef".
 */

class Solution {
public:
    // 错误做法
    string decodeString(string s) {
        if (s == "") {
            return "";
        }

        string result = "";
        stack<pair<char, int>> codes;
        codes.push({s[0], 0});

        while (!codes.empty()) {
            auto content = codes.top();
            codes.pop();

            // 表明开始是数字
            if (content.first >= '0' && content.first <= '9') {
                int start = content.second;
                string nums = "";
                nums.push_back(content.first);
                while (s[start] != '\n' && s[start] != '[') {
                    content.first += s[start];
                    start++;
                }

                // 计算单位的字符串
                int num = atoi(nums.c_str());
                string str = "";
                for (int i = 0; i < num; i++) {
                    str += content.first;
                }
                result += str;

                // 找到下一个单位起始的位置
                while (s[start] != '\n' && s[start] != ']') {
                    start++;
                }
                if (start != s.size()-1) {
                    start++;
                    codes.push({s[start], start});
                }
            }
        }

        return result;
    }

    /**
     * 迭代DFS的做法
     *
     * @param s
     * @return
     */
    string decodeString2(string s) {
        int cnt  = 0; //每个单元里的数字个数
        string content = "";

        int len = s.size();
        int i = 0;
        stack<string> strs;
        stack<int> cnts;

        while (i < len) {
            // 为数字的情况
            if (i < len && s[i] >= '0' && s[i] <= '9') {
                cnt = cnt * 10 + (s[i] - '0');
            }
            // 为左括号的情况
            else if (i < len && s[i] == '[') {
                cnts.push(cnt);
                strs.push(content); //因为存在内嵌的情况，所以保存下之前的单元内的字符串

                cnt = 0;
                content.clear(); //清空用来保存后面单元的字符串
            }
            // 为右括号的情况
            else if (i < len && s[i] == ']') {
                int cnt_temp = cnts.top();
                cnts.pop();

                while (cnt_temp--) {
                    strs.top() += content;
                }

                // 将之前的字符串进行累加以后得到新的单元的内容
                content = strs.top();
                strs.pop();
            }
            // 中间的字符串
            else {
                content += s[i];
            }

            i++;
        }

        // 可能存在没有]的单元
        return strs.empty() ? content : strs.top();
    }

    /**
     * 递归DFS
     * 计算单元格内字符串的内容，再根据个数进行累加
     * 注意字符串下标和边界条件
     *
     * @param s
     * @return
     */
    string decodeString3(string s) {
        int len = s.size();
        if (len == 0) {
            return "";
        }
        int i = 0;
        return decodeHelper(s, i);
    }

    string decodeHelper(string s, int& i) {
        string result = "";
        int len = s.size();
        while (i < len && s[i] != ']') {
            // 非数字而是字符的情况下
            if (s[i] < '0' || s[i] > '9') {
                result += s[i++];
            }
            else {
                // 计算数字
                int cnt = 0;
                while (s[i] >= '0' && s[i] <= '9' && i < len) {
                    cnt = cnt * 10 + (s[i++] - '0');
                }
                // 移动到字符串开始的位置
                i++;
                // 获得单元内的字符串，此时递归返回i的位置是]的位置
                string str = decodeHelper(s, i);
                //移动到下一个单元的位置
                i++;

                while (cnt--) {
                    result += str;
                }
            }
        }

        return result;
    }

    void test() {
        string str = "3[a2[c]]";
        cout << "result : " << decodeString3(str) << endl;
    }
};

```

总结：这些和在树中使用DFS的思想是类似的。