# O(n) 排序算法

## 前言

前面有总结过各类常用的排序算法，但是那些排序算法平均的时间复杂度是O(nlogn)，所以我要介绍三种时间复杂度为O(n)的线性时间复杂度的排序算法。

## 计数排序

计数排序利用了哈希的性质，将一个中间数组来记录数值对应的下标，最后查询对应的下标进行放置；

步骤如下：

1. 找出待排序的数组中最小和最大值，计算最大和最小值之间的差值；
2. 计算每个数值出现的次数，接着进行累加计算出数值的位置；
3. 反向填充数组，根据查询下标找到位置后填充数值；


### 实现

```cpp
#include <iostream>
#include <vector>
using namespace std;

vector<int> counting_sort(vector<int> nums) {
    int max = nums[0], min = nums[0];
    size_t len = nums.size();
    for (size_t i = 1; i < len; i++) {
        if (max < nums[i]) {
            max = nums[i];
        }
        if (min > nums[i]) {
            min = nums[i];
        }
    }
    int k = max - min + 1;
    vector<int> temp(k, 0);
    // 第一步：计算每个数字出现的次数
    for (size_t i = 0; i < len; i++) {
        temp[nums[i] - min] += 1;
    }
    // 第二步：累加
    for (size_t i = 1; i < len; i++) {
        temp[i] += temp[i-1];
    }
    vector<int> result(len, 0);
    // 第三步：将数字放在相应的位置
    for (size_t i = 0; i < len; i++) {
        result[--temp[nums[i] - min]] = nums[i];
    }
    return result;
}

int main() {
    vector<int> res = counting_sort({10, 9, 8, 7, 6, 5, 4, 3, 2, 1});
    for (auto re : res) {
        cout << re << "  ";
    }
    return 0;
}
```

### 缺点和优点

利用了哈希的原理，其时间复杂度为n，但是这是用空间复杂度来换的，即便上面有进行过优化，但是面对一个较大值和较小值的数组，其仍然会对空间造成很大的浪费。

## 基数排序

将所有数值在每一位上面进行排序，排序方法利用计数排序的原理；

步骤：

1. 计算数值中最大值的位数，用作后面比较的次数；
2. 计算所有数值在每一位上面的排序，参考计数排序；

### 实现

```cpp
void redis_sort(vector<int>& nums) {
    int bits = max_bit(nums);
    int len = nums.size();
    vector<int> temp(len, 0), count(10, 0);
    for (int i = 1, redix = 1; i <= bits; i++, redix *= 10) {
        // 注意，每次分配前需要清空计数器
        count.assign(10, 0);
        // 第一步：计算每个数值下标出现的次数
        for (int j = 0; j < len; j++) {
            count[(nums[j]/redix)%10]++;
        }
        // 第二步：累加计算下标
        for (int j = 1; j < 10; j++) {
            count[j] += count[j-1];
        }
        // 第三步：根据bit的下标找到位置来填充
        for (int j = len-1; j >= 0; j--) {
            int k = (nums[j]/redix)%10;
            temp[count[k]-1] = nums[j];
            count[k]--;
        }
        // 第四部：排好序的数组赋值
        for (int j = 0; j < len; j++) {
            nums[j] = temp[j];
        }
    }
}
```

### 缺点和优点

因为其下标在0-10之间，所以有效的控制了空间复杂度，但是其复杂度较计数排序增加了，明显其时间复杂度为O(k * n)，k代表数字位数，这取决于数字位的选择，比如比特位数，其决定了要进行多少轮的处理；虽然增加了时间复杂度，但依旧比那些需要进行比较的排序算法较快一些。

## 桶排序

桶排序的原理在于将数组分配到一定数量的桶中，每个桶在个别排序，最后合并排序。

### 实现

```cpp
const int BUCKET_NUM = 10;

// 链表的插入排序
LinkNode* insert(LinkNode* head, int val) {
    LinkNode *newhead = new LinkNode(0);
    newhead->_next = head;

    LinkNode *node = new LinkNode(val);
    LinkNode *temp = newhead;
    while (temp->_next != NULL && temp->_next->_data <= val) {
        temp = temp->_next;
    }
    node->_next = temp->_next;
    temp->_next = node;
    return newhead->_next;
}

// 两个排序链表的合并
LinkNode* merge(LinkNode* head, LinkNode* bucket_node) {
    LinkNode* newhead = new LinkNode(0);
    LinkNode* temp = newhead;
    while (head && bucket_node) {
        if (head->_data > bucket_node->_data) {
            temp->_next = bucket_node;
            bucket_node = bucket_node->_next;
        }
        else {
            temp->_next = head;
            head = head->_next;
        }
        temp = temp->_next;
    }
    if (head != NULL) {
        temp->_next = head;
    }
    else if (bucket_node != NULL) {
        temp->_next = bucket_node;
    }
    return newhead->_next;
}

vector<int> BucketSort(vector<int> nums) {
    int len = nums.size();
    vector<LinkNode*> buckets(BUCKET_NUM, (LinkNode*)(0));
    // 第一步：对数值进行插入排序
    for (int i = 0; i < len; i++) {
        int idx = nums[i] % BUCKET_NUM;
        LinkNode* head = buckets[idx];
        buckets[idx] = insert(head, nums[i]);
    }
    // 第二步：将桶中的值进行合并
    LinkNode *head = NULL;
    for (int i = 0; i < BUCKET_NUM; i++) {
        head = merge(head, buckets[i]);
    }
    // 第三步：将排序好的链表赋值
    vector<int> result(len, 0);
    for (int i = 0; i < len, head != NULL; i++, head = head->_next) {
        result[i] = head->_data;
    }
    return result;
}
```

### 缺点和优点

如果数组中的每个数值都会均匀的落入每个桶中，则其最优的时间复杂度在n，但是如果数值都集中的加入到固定的几个桶中，甚至是都落入一个桶中，那么这样在对数值进行插入排序的时候就变成了双层循环，则其最差时间复杂度为n^2。


## 比较

![](http://odwv9d2u8.bkt.clouddn.com/17-5-17/91925318-file_1494996708175_6be9.png)


