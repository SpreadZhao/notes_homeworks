---
num: "206"
title: "Reverse Linked List"
link: "https://leetcode.cn/problems/reverse-linked-list/description/"
tags:
  - leetcode/difficulty/easy
---

# 题解

想到这题其实可以原地反转。但是没想出来，所以还是先用的头插法。就直接给代码了：

```cpp
ListNode *Solution::reverseList(ListNode *head) {
    if (head == nullptr) {
        return head;
    }
    auto *newHead = new ListNode(head->val);
    ListNode *p = head->next;
    while (p != nullptr) {
        const int value = p->val;
        auto *node = new ListNode(value);
        node->next = newHead;
        newHead = node;
        p = p->next;
    }
    return newHead;
}
```

那么原地怎么做呢？其实很简单，我们画个图：

![[Projects/leetcode/resources/Drawing 2024-11-24 21.25.39.excalidraw.svg]]

其实，我们只需要有两个指针：prev和curr，这样每次就能把curr的next指向prev了：

![[Projects/leetcode/resources/Drawing 2024-11-24 21.27.34.excalidraw.svg]]

但是这里的问题是，我们让`curr->next = prev`之后，curr没法往后走了。这个问题很好解决，我们再用一个指针next，提前保存一下`curr->next`。这样问题就解决了：

```cpp
ListNode *Solution::reverseList2(ListNode *head) {
    ListNode *prev = nullptr;
    ListNode *curr = head;
    while (curr != nullptr) {
        ListNode *next = curr->next;  // 提前保存next
        curr->next = prev;
        prev = curr;
        curr = next;
    }
    return prev;
}
```

