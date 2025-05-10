---
num: "208"
title: Implement Trie Prefix Tree
link: https://leetcode.cn/problems/implement-trie-prefix-tree/description/
tags:
  - leetcode/difficulty/medium
  - algo/hash-table
  - algo/string
  - algo/trie
---

# 题解

这破题居然卡了我半天。。。

其实前缀树很简单，就是要尽可能复用节点。比如下面这个例子：

![[Projects/leetcode/resources/Drawing 2025-05-11 00.18.37.excalidraw.svg|400]]

这里就有三个单词，apple，app和act。

显然，可以用树来实现。但不是一个二叉树，而是一个最多26叉树。

所以，对于一个节点，它没有`left`和`right`指针，而是一个children集合，表示所有的孩子。

现在的问题就是，应该用什么形式来表示这些孩子。

如果一个节点是一个`TrieNode`，那么显然，它的孩子就是一群`TrieNode`。而一群究竟有多少？取决于后面的单词有多少种不同的字母。

另一个问题是，如何确定是哪个孩子？就像二叉树里的左孩子和右孩子一样。显然，访问哪个孩子取决于是哪个字母。所以，我这里用一个map来实现。所以，现在`TrieNode`的结构为：

```cpp

class TrieNode {
public:
	map<char, TrieNode *> children;
};
```

你可能会有疑问：为啥没有表示当前`TrieNode`的字母？我们以上面的图来举例子。对于根节点a，它有两个孩子p和c。那么，在a这个`TrieNode`中，`children`就应该是一个有两个元素的map，分别是`<'p', TrieNode>`和`<'c', TrieNode>`。我们可以发现，在对节点进行搜索的时候，**如果我通过map的get来获取到这个`TrieNode`，那么我是一定知道这个`TrieNode`所表示的字母的**！比如，如果我对a这个`TrieNode`调用：

```cpp
TrieNode *child = trieNode['p'];
```

那么显然，`child`代表的字母是啥？当然是p了！所以，在搜索的过程中，一个字母一个字母找，都是能通过这种方式确认，根本不需要额外信息。

然而有一个例外。那就是根节点。跟节点显然是要直接存在`Trie`中的一系列`TrieNode`，最多有26个。而对于每一个节点，需要有一个字母来表示。

所以，我这里的方法就是用一个26元素数组了。

最后，还有一个可能被遗漏的问题。app和application是两个单词。所以search这两个单词的结果都应该是true。而碰巧的是，app就是application的前缀（对应节点复用）。所以，我们不能以是否是根节点来判断是否是一个完整的单词。这里我的做法是，在`TrieNode`中记一个标记位。下面是`TrieNode`以及`Trie`的完整结构：

```cpp
class Trie {
public:
	Trie();

	void insert(string word);

	bool search(string word);

	bool startsWith(string prefix);

private:
	class TrieNode {
	public:
		bool is_last = false;
		map<char, TrieNode *> children;
	};

	TrieNode *roots[26] { nullptr };
};
```

知道了结构，也知道了遍历的方式，剩下的就没什么困难了。这里有两个小点要注意。

第一个是，因为我们的根节点用数组存，所以一开始需要一个首字母定位根节点的过程。大概是这样：

```cpp
if (word.empty()) {
	return;
}
// find root
TrieNode *curr = this->roots[word[0] - 'a'];
if (curr == nullptr) {
	curr = new TrieNode();
	this->roots[word[0] - 'a'] = curr;
}
// now curr == root
...
```

上面是插入的例子，搜索是同理的。

第二点，是插入过程中我遇到的一个c++的问题。因为我的`children`用的是正常的变量而非指针。所以，如果我这么写：

```cpp
auto children = curr->children;
```

我对`children`的更改，是不会应用到`curr->children`上的！这就是写java写多了导致的，因为java自动给你用引用包了一下，所以没有这些问题。所以应该加上引用。

完整的实现代码：

```cpp
Solution::Trie::Trie() = default;

void Solution::Trie::insert(string word) {
    if (word.empty()) {
        return;
    }
    // find root
    TrieNode *curr = this->roots[word[0] - 'a'];
    if (curr == nullptr) {
        curr = new TrieNode();
        this->roots[word[0] - 'a'] = curr;
    }
    // now curr == root
    for (size_t i = 1; i < word.size(); i++) {
        char ch = word[i];
        auto &children = curr->children;
        if (children.find(ch) == children.end()) {
            children[ch] = new TrieNode();
            curr = children[ch];
        } else {
            curr = children[ch];
        }
    }
    curr->is_last = true;
}
bool Solution::Trie::search(string word) {
    if (word.empty()) {
        return false;
    }
    TrieNode *curr = this->roots[word[0] - 'a'];
    if (curr == nullptr) {
        return false;
    }
    for (size_t i = 1; i < word.size(); i++) {
        char ch = word[i];
        auto children = curr->children;
        if (children.find(ch) == children.end()) {
            return false;
        }
        curr = children[ch];
    }
    return curr->is_last;
}

bool Solution::Trie::startsWith(string prefix) {
    if (prefix.empty()) {
        return false;
    }
    TrieNode *curr = this->roots[prefix[0] - 'a'];
    if (curr == nullptr) {
        return false;
    }
    for (size_t i = 1; i < prefix.size(); i++) {
        char ch = prefix[i];
        auto children = curr->children;
        if (children.find(ch) == children.end()) {
            return false;
        }
        curr = children[ch];
    }
    return true;
}
```