# AVL树理解

## 简介

我们知道，AVL树也是平衡树中的一种，是自带平衡条件的二叉树，始终都在维护树的高度，保持着树的高度为logN，同时把**插入、查找、删除**一个结点的时间复杂度的最好和最坏情况都维持在**O(logN)**；增加和删除需要通过一次或者多次的树旋转来重新平衡这棵树。

其中规定每个结点的左子树和右子树的高度最多差1，也就是说任何结点的两个子树的最大高度为1。

## 性质

平衡树最重要的地方在于旋转，其中分为单旋转和双旋转，其中单旋转和双旋转又各分为两种子形式，分别为左单旋转，右单旋转，左右双旋转，右左双旋转；在左左和右右这样的单旋转的情况下，只需要进行一次旋转操作，但是在左右或者右左这样的双旋转的情况下，则需要进行二次旋转操作。

<!--more-->

如图：

![](http://odwv9d2u8.bkt.clouddn.com/16-10-7/87733545.jpg)

* 1，4为单旋转，1为左左，4为右右
* 2，3为双旋转，2为左右，3为右左

再看下面一张图：

![](http://i2.muimg.com/588926/48774a23b59dcc70.png)

* 左右（父亲结点在左边，子结点在右边）：子结点先要经过一次左旋，变成左左的情况，再对父亲结点进行一次右旋，即可达到平衡；
* 右左（父亲结点在右边，子结点在左边）：同理，先对子结点进行一次右旋，变成右右的情况，再对父亲结点进行一次左旋，即可达到平衡。

### 单旋转

![](http://odwv9d2u8.bkt.clouddn.com/16-10-7/99171693.jpg)

这是一个左左的情况，**因为是对称的，所以两者可以通过左右旋转相互到达；**

可以发现的是，k2为失去平衡的树的根结点，k1为旋转后重新平衡的树的根结点，k1结点的右结点经过旋转连接到了k2的左结点上，此时，k2的左结点连接到k1的右结点，而k2则连到k1的右结点上面；

### 双旋转

![](http://odwv9d2u8.bkt.clouddn.com/16-10-7/23041949.jpg)

**上面是左右的情况，但是右左的情况虽然不是对称的， 但是情况是类似的；**

我们先看子结点的部分，也就是以k1为父结点的部分，此时k1为失去平衡的树的根结点，k2为进行旋转以后平衡树的根结点，经过左旋转以后变成了以k2为父结点，k1为左子结点的部分，此时树变成了k3为失去平衡的树根结点，k2变成经过旋转以后平衡树的根结点，所以在旋转时，可以把k1看作是一个整体，把k2的右子结点连接到k3的左子结点上面，再将k3当作一个整体连接到k2的右子结点上面。

## 实现

TreeNode.hpp

```cpp
//
//  TreeNode.hpp
//  AVL
//
//  Created by George on 16/10/7.
//  Copyright © 2016年 George. All rights reserved.
//

#ifndef TreeNode_hpp
#define TreeNode_hpp

namespace AVL {
    
    template<class T>
    class TreeNode {
    public:
        TreeNode(T value, int height = 0) : value(value), height(height), left(nullptr), right(nullptr){};
        T value;
        int height;
        TreeNode* left;
        TreeNode* right;
    };
    
    template<class T>
    class AVLTree {
    public:
        AVLTree() = default;
        AVLTree(TreeNode<T>* ptr) : _root(ptr){};
        ~AVLTree();
        void Insert(T value);
        TreeNode<T>* find(T value);
        void Delete(T value);
        void Traversal();
        
    private:
        TreeNode<T> * _root;
        void insertNode(TreeNode<T>* &root, T value);
        TreeNode<T>* find(TreeNode<T>* root, T value);
        void deleteNode(TreeNode<T>* &root, T value);
        void traversal(TreeNode<T>* root);
        int height(TreeNode<T>* node) const;
        
        void singleRotateWithLeftChild(TreeNode<T>* &node);
        void singleRotateWithRightChild(TreeNode<T>* &node);
        void doubleRotateWithLeftRightChild(TreeNode<T>* &node);
        void doubleRotateWithRightLeftChild(TreeNode<T>* &node);
    };
    
    void insertTest();
    void deleteTest();
    void searchTest();
    void traversalTest();
}

#endif /* TreeNode_hpp */

```

TreeNode.cpp

```cpp
//
//  TreeNode.cpp
//  AVL
//
//  Created by George on 16/10/7.
//  Copyright © 2016年 George. All rights reserved.
//

#include "TreeNode.hpp"

#include <string>
#include <iostream>

namespace AVL {
    
    template<class T>
    void LogNode(TreeNode<T> *node) {
        std::cout << "value :" << node->value << ", height :" << node->height << std::endl;
    }
    
    std::string GetInputString(const std::string content) {
        std::cout << content << " :";
        std::string input;
        std::cin >> input;
        return input;
    }
    
    
    /*------------------------------AVLTree-------------------------------*/
    template<class T>
    int AVLTree<T>::height(TreeNode<T> *node) const {
        return node == nullptr ? -1 : node->height;
    }
    
    template<class T>
    void AVLTree<T>::singleRotateWithLeftChild(TreeNode<T>* &node) {
        TreeNode<T>* lNode = node->left;
        node->left = lNode->right;
        lNode->right = node;
        
        node->height = std::max(height(node->left), height(node->right)) + 1;
        lNode->height = std::max(height(lNode->left), height(lNode->right)) + 1;
        
        node = lNode;
    }
    
    template<class T>
    void AVLTree<T>::singleRotateWithRightChild(TreeNode<T>* &node) {
        TreeNode<T>* rNode = node->right;
        node->right = rNode->left;
        rNode->left = node;
        
        node->height = std::max(height(node->left), height(node->right)) + 1;
        rNode->height = std::max(height(rNode->left), height(rNode->right)) + 1;
        
        node = rNode;
    }
    
    template<class T>
    void AVLTree<T>::doubleRotateWithLeftRightChild(TreeNode<T>* &node) {
        singleRotateWithRightChild(node->left);
        singleRotateWithLeftChild(node);
    }
    
    template<class T>
    void AVLTree<T>::doubleRotateWithRightLeftChild(TreeNode<T>* &node) {
        singleRotateWithLeftChild(node->right);
        singleRotateWithRightChild(node);
    }
    
    template<class T>
    void AVLTree<T>::insertNode(TreeNode<T>* &root, T value) {
        if (!root)
            root = new TreeNode<T>(value);
        
        if (value < root->value) {
            insertNode(root->left, value);
            
            if (height(root->left) - height(root->right) == 2) {
                if (value < root->left->value)
                    singleRotateWithLeftChild(root);
                else
                    doubleRotateWithLeftRightChild(root);
            }
        }
        else if (value > root->value) {
            insertNode(root->right, value);
            
            if (height(root->right) - height(root->left) == 2) {
                if (value > root->right->value)
                    singleRotateWithRightChild(root);
                else
                    doubleRotateWithRightLeftChild(root);
            }
        }
        
        root->height = std::max(height(root->left), height(root->right)) + 1;
    }
    
    template<class T>
    TreeNode<T>* AVLTree<T>::find(TreeNode<T>* root, T value) {
        if (root == nullptr)
            return nullptr;
        
        if (value < root->value)
            return find(root->left, value);
        else if (value > root->value)
            return find(root->right, value);
        else
            return root;
        
    }
    
    template<class T>
    void AVLTree<T>::deleteNode(TreeNode<T> *&root, T value) {
        if (root == nullptr) return ;
        
        if (value < root->value) {
            deleteNode(root->left, value);
            
            if (height(root->right) - height(root->left) == 2) {
                // 判断是否存在右边子结点的左子结点
                if (root->right->left && height(root->right->left) > height(root->right->right))
                    doubleRotateWithRightLeftChild(root);
                else
                    singleRotateWithRightChild(root);
            }
        }
        else if (value > root->value) {
            deleteNode(root->right, value);
            
            if (height(root->left) - height(root->right) == 2) {
                // 判断是否存在左边子结点的右子结点
                if (root->left->right && height(root->left->right) > height(root->left->left))
                    doubleRotateWithLeftRightChild(root);
                else
                    singleRotateWithLeftChild(root);
            }
        }
        else {
            // 存在两个子结点
            if (root->left && root->right) {
                TreeNode<T>* ptr = root->right;
                
                // 找到右子树中最小的结点
                while (ptr->left != nullptr) {
                    ptr = ptr->left;
                }
                
                // 替换值
                root->value = ptr->value;
                // 删除右子树中最小值的结点
                deleteNode(root->right, ptr->value);
                
                if (height(root->left) - height(root->right) == 2) {
                    if (root->left->right && height(root->left->right) > height(root->left->left))
                        doubleRotateWithLeftRightChild(root);
                    else
                        singleRotateWithLeftChild(root);
                }
            }
            else { // 存在一个结点或没有结点
                TreeNode<T>* ptr = root;
                if (root->left == nullptr)
                    root = root->right;
                else if (root->right == nullptr)
                    root = root->left;
                
                delete ptr;
                ptr = nullptr;
            }
        }
        
        if (root) {
            root->height = std::max(height(root->left), height(root->right)) + 1;
        }
    }
    
    template<class T>
    void AVLTree<T>::traversal(TreeNode<T>* root) {
        if (!root) return;
        traversal(root->left);
        LogNode(root);
        traversal(root->right);
    }
    
    template<class T>
    void AVLTree<T>::Insert(T value) {
        insertNode(_root, value);
    }
    
    template<class T>
    TreeNode<T>* AVLTree<T>::find(T value) {
        return find(_root, value);
    }
    
    template<class T>
    void AVLTree<T>::Delete(T value) {
        deleteNode(_root, value);
    }
    
    template<class T>
    void AVLTree<T>::Traversal() {
        traversal(_root);
    }
    
    /*------------------------------Test-------------------------------*/
    static AVLTree<int>* tree = new AVLTree<int>();
    
    void insertTest() {
        for (int i = 1; i <= 10 ; i++) {
            tree->Insert(i);
        }
    }
    
    void deleteTest() {
        std::string input = GetInputString("Please input the delete value");
        int value = atoi(input.c_str());
        tree->Delete(value);
    }
    
    void searchTest() {
        std::string input = GetInputString("Please input the search value");
        int value = atoi(input.c_str());
        TreeNode<int>* node = tree->find(value);
        LogNode(node);
    }
    
    void traversalTest() {
        tree->Traversal();
    }
    
}

```

main.cpp

```cpp
//
//  main.cpp
//  AVL
//
//  Created by George on 16/10/7.
//  Copyright © 2016年 George. All rights reserved.
//

#include <iostream>
#include "TreeNode.hpp"

int main(int argc, const char * argv[]) {
    // insert code here...
    
    AVL::insertTest();

    AVL::traversalTest();
    
    AVL::deleteTest();
    
    AVL::traversalTest();
    
    AVL::searchTest();
    
    return 0;
}

```

运行结果

![](http://odwv9d2u8.bkt.clouddn.com/16-10-7/2564362.jpg)


## 应用

因为旋转消耗的时间较多，所以适用于插入删除次数较少的，比如windows对进程地址空间的管理就用到了AVL。

