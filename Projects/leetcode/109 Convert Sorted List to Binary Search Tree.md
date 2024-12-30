---
num: "109"
title: Convert Sorted List to Binary Search Tree
link: https://leetcode.cn/problems/convert-sorted-list-to-binary-search-tree/description/
tags:
  - leetcode/difficulty/medium
---
> [!error]- Deprecated
> 
> 这道题要求我们将一个排好序的链表转换为二叉搜索树，为了达成平均搜索次数最小的目的，二叉搜索树要求我们尽可能将每一层的结点填充得最为充实，即**它的任何两棵子树层数差都不超过1**，拥有$2^n-1$个结点的二叉搜索树搜索到目标结点最多只需要搜索n次
> 
> 参考Solution，运用**分治**的思想，我们需要五步来实现这个算法：
> 
> 1. 将数组的中间元素设置为根结点
> 2. 递归地对左半部分和右半部分做同样的事
> 3. 获取左半部分的中间元素，并使其成为步骤一中创建的根的左孩子
> 4. 获取右半部分的中间元素，并使其成为步骤一中创建的根的右孩子
> 5. 先序输出这棵树
> 
> 在进行递归之前，因为链表不支持随机访问，所以我们需要把它的值按顺序拿出来放入一个允许随机访问的有序数组中
> 
> 然后开始递归，以Example 1为例，这个数组的中间元素是0，那么0就是这棵树的根结点。如果数组总共有四个元素，那么我们选择第三个元素作为树的根结点（index=size/2，除法默认舍尾）
> 
> 0的左半边为\[-10,-3\]，中间元素为-3，将其作为0左子树的根结点；-3的左半边为\[-10\]，只有一个元素，-10的左右半边皆为空，将其看成中间元素，作为-3左子树的根结点，返回根结点-10；-3的右半边为空，返回根结点-3
> 
> 0的右半边为\[5,9\]，中间元素为9，将其作为0右子树的根结点；9的左半边为\[5\]，只有一个元素，5的左右半边皆为空，将其看成中间元素，成为9左孩子的根结点，返回根结点5；9的右半边为空，返回根结点9
> 
> 最后返回根结点0，至此二叉搜索树就建好了

上面这坨是我让程浩然写的，我还是自己总结一遍吧。首先，我自己没想出来比较好的做法，瞄了一眼之前的代码，总结出这个简单的思路：

1. 这个链表的中间元素就是最终的根节点；
2. 找到链表的左半部分和右半部分，对两半递归地做这个操作。

这样很简单，不断取中间元素，最后拼到一起就行了。核心代码：

```cpp
curr->left = cutRecursively(nums, low, mid - 1);  
curr->right = cutRecursively(nums, mid + 1, high);
```

这个方法需要注意的点：

1. 这是链表，所以得先转到一个数组里才能取low和high；
2. 注意边界情况，尤其是low > high的时候。

代码：

```cpp
static TreeNode *cutRecursively(const vector<int> &nums, int low, int high) {
    if (low == high) {
        return new TreeNode(nums[low]);
    }
    if (low > high) {
	    // 如果递归的时候，low = 0，mid = 0，
	    // 也就是第一个元素。那么下一次递归
	    // low = 0, high = -1。就直接返回空。
	    // 可以用[-10, -3, 0, 5, 9]这个例子验证。
        return nullptr;
    }
    int mid = (low + high) / 2;
    auto *curr = new TreeNode(nums[mid]);
    curr->left = cutRecursively(nums, low, mid - 1);
    curr->right = cutRecursively(nums, mid + 1, high);
    return curr;
}

TreeNode *Solution::sortedListToBST(ListNode *head) {
    if (head == nullptr) {
        return nullptr;
    }
    const auto *curr = head;
    vector<int> nums;
    while (curr) {
        nums.push_back(curr->val);
        curr = curr->next;
    }
    return cutRecursively(nums, 0, nums.size() - 1);
}
```

然后，我看了一眼题解。第一个方法和我这个思路是一模一样的，只是用了一种更快的方法找中间值。这个也很简单，就是我面蔚来实习的时候瑞瑞问过我的，找链表的中间节点的方法，用快慢指针。

但是，这样做的话，我们就不能拿到low和high这样的信息了。所以，我们需要换一个思路来想这个问题。

首先，第一次找的话，我们是能知道mid在哪里的，只需要这样的代码：

```cpp
static ListNode *getMedian(ListNode *head) {
    ListNode *fast = head;
    ListNode *slow = head;
    while (fast != nullptr && fast->next != nullptr) {
        fast = fast->next->next;
        slow = slow->next;
    }
    return slow;
}
```

那这次找之后，我们就有了head, mid这两个节点信息了。显然，我们可以找出mid，并分成左右两个部分。也就是$[0, mid)$和$[mid + 1, nullptr)$。注意这两个集合不包含mid。

那么，如果想设计成递归的形式，函数参数的设计就是很重要的。应该怎么设计呢？假设现在我们对左半部分递归，那么接下来依然需要调用getMedian函数。但是，我就不能只传一个head了，因为只有头信息的话，找到的还是整个链表的中间节点。**我们需要一个时机让快指针停止移动**。

显然，这个时机就在于while循环应该怎么写。如果写nullptr，那就是不为空不停止。因此，如果希望遍历到所有属于$[0, mid)$的元素，**就应该在fast指向mid - 1的时候就停止**。此时，`fast->next == mid`成立。所以，我们把getMedian改成这样：

```cpp
static ListNode *getMedian(ListNode *low, ListNode *high) {
    ListNode *fast = low;
    ListNode *slow = low;
    while (fast != high && fast->next != high) {
        fast = fast->next->next;
        slow = slow->next;
    }
    return slow;
}
```

这样，我们传`(head, nullptr)`，就是找整个链表的中间节点；在第一次左半递归中，传`(head, mid)`，就是找$[0, mid)$的中间节点。

这个最值得思考的问题说通，剩下的就迎刃而解了。直接给代码：

```cpp
static ListNode *getMedian(ListNode *low, ListNode *high) {
    ListNode *fast = low;
    ListNode *slow = low;
    while (fast != high && fast->next != high) {
        fast = fast->next->next;
        slow = slow->next;
    }
    return slow;
}

static TreeNode *cutRecursive2(ListNode *low, ListNode *high) {
    if (low == high) {
        return nullptr;
    }
    ListNode *mid = getMedian(low, high);
    TreeNode *root = new TreeNode(mid->val);
    root->left = cutRecursive2(low, mid);
    root->right = cutRecursive2(mid->next, high);
    return root;
}

TreeNode *Solution::sortedListToBST2(ListNode *head) {
    if (head == nullptr) {
        return nullptr;
    }
    return cutRecursive2(head, nullptr);
}
```

> [!question] 为啥我感觉我自己写的方法反而快一点？
> - [ ] #TODO tasktodo1735576813999 他这个方法每次递归的时候都要快慢指针扫一遍，我自己的方法就是一开始建一个数组，只遍历一次，剩下的都是直接计算就行。为啥最后他的方法反而更快？难道是vector构建的耗时吗？➕ 2024-12-31 🔽 🆔 f2f811 

- [ ] #TODO tasktodo1735576910421 后面还有一个结合中序遍历的优化思路。有时间看看。 ➕ 2024-12-31 🔽 🆔 iv0zzd 