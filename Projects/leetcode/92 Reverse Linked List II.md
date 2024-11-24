---
num: "92"
title: "Reverse Linked List II"
link: "https://leetcode.cn/problems/reverse-linked-list-ii/description/"
tags:
  - leetcode/difficulty/medium
---

# 题解

[[Projects/leetcode/206 Reverse Linked List|206 Reverse Linked List]]的升级版，只反转一部分。我记得面试我被问过，显然当时太菜没做出来。

这个我一开始想到的也是，拆成3个链表，然后把中间那部分通过[[Projects/leetcode/206 Reverse Linked List|206 Reverse Linked List]]的方法反转，然后再合成一个新的。这里要处理很多边界case，也就是这三个链表中可能会有空。这个方法没啥含金量并且很复杂，我就不说了。还是题解里的原地反转。

我们还是画个图，看看反转前和反转后的区别：

![[Projects/leetcode/resources/Drawing 2024-11-24 21.34.46.excalidraw.svg]]

我们发现，3 4 5 6的箭头指向改变是和[[Projects/leetcode/206 Reverse Linked List|206 Reverse Linked List]]一致的。但是额外多了两个：2的下一个需要从指向3变成指向6，也就是**反转部分的最后一个节点**；而3的下一个要从指向4变成指向7，也就是**反转部分后面的第一个节点**。

因此，我们需要很多额外的变量来记录这些位置。在此说明一下：

- `leftNode`: 反转部分的第一个节点，对应上图的3；
- `leftPrev`：反转部分第一个节点的prev，对应上图的2；

置于为什么不用保存反转部分的最后一个节点和反转部分最后一个节点的next（上图中的6和7），因为遍历的时候已经有了。

下面开始。首先，需要确定`leftNode`和`leftPrev`的值：

```cpp
#define REPEAT(X) for (int i = 0; i < (X); i++)

ListNode *leftNode = head;
ListNode *leftPrev = nullptr;

REPEAT(left - 1) {
	if (i == left - 2) {
		leftPrev = leftNode;
	}
	leftNode = leftNode->next;
}
```

很好理解，如果从head开始，那么移动left - 1次就能定位到反转部分的第一个节点；因此当最后一次移动之前（`i == left - 2`）时，此时的节点就是反转部分的第一个节点的prev。而默认值是nullptr，代表如果$left \leqslant 1$时，此时就是从第一个节点开始反转，因此没有前置节点，正好就是空。

接下来，就可以确定[[Projects/leetcode/206 Reverse Linked List|206 Reverse Linked List]]题中的prev和curr了：

![[Projects/leetcode/resources/Drawing 2024-11-24 21.49.23.excalidraw.svg]]

```cpp
ListNode *prev = leftNode;
ListNode *curr = prev->next;
```

那么下一个问题也很显然：反转循环进行多少次？因为不是全部反转，所以要受到right控制。这里有一个简单的想法：每一次循环会反转一条边。因此**我们看反转部分中有几条边就行了**。上图中有4个节点，3条边，正好是right - left。因此，我们只需要执行right - left次[[Projects/leetcode/206 Reverse Linked List|206 Reverse Linked List]]题中的循环即可：

```cpp
REPEAT(right - left) {
	ListNode *next = curr->next;
	curr->next = prev;
	prev = curr;
	curr = next;
}
```

这样做完之后，是啥情况呢？如下图：

![[Projects/leetcode/resources/Drawing 2024-11-24 21.52.41.excalidraw.svg]]

我们发现，此时的prev正好就是反转部分的最后一个节点，curr正好是反转部分最后一个节点的next，因此直接可以用了。最后我们还差两部：修改2的指向和修改3的指向。分别对应`leftPrev`和`leftNode`。

- 将`leftPrev`的下一个指向反转部分最后一个节点，也就是prev；
- 将`leftNode`的下一个指向反转部分最后一个节点的next，也就是curr。

这里要注意，`leftPrev`有可能为空，前面提到过。因此需要注意一下判空：

```cpp
if (leftPrev != nullptr) {
	leftPrev->next = prev;
}
leftNode->next = curr;
```

至此本题就做完了。然而，依然遗漏了一个点。在下面说明。

# 遗漏的case

我最后返回的，一开始就是`head`。但是出现了问题。也就是，**当反转部分包含第一个节点时，会有问题**。

这也很好想，如果是1 2 3 4 5，然后left = 1, right = 3。那么反转后的结果是3 2 1 4 5。此时返回head，head指向的可是1，所以最后的链表就变成1 4 5了。

也就是说，在这种情况下，我们应该返回的是反转部分的最后一个节点，也就是上面的prev。下面给出全部代码：

```cpp
ListNode *Solution::reverseBetween(ListNode *head, int left, int right) {
    if (head == nullptr || left == right) {
        return head;
    }
    ListNode *leftNode = head;
    ListNode *leftPrev = nullptr;
    REPEAT(left - 1) {
        if (i == left - 2) {
            leftPrev = leftNode;
        }
        leftNode = leftNode->next;
    }
    ListNode *prev = leftNode;
    ListNode *curr = prev->next;
    REPEAT(right - left) {
        ListNode *next = curr->next;
        curr->next = prev;
        prev = curr;
        curr = next;
    }
    if (leftPrev != nullptr) {
        leftPrev->next = prev;
    }
    leftNode->next = curr;
    if (leftNode == head) {
        return prev;
    }
    return head;
}
```