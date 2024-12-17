---
num: "71"
title: "Simplify Path"
link: "https://leetcode.cn/problems/simplify-path/description/"
tags:
  - leetcode/difficulty/medium
---

# 题解

这题很简单，不需要额外的操作。按照思路一点点来就行。不过题解里有一个用栈的思路很好。其实，是能想到用栈的，因为`..`这种操作很显然就对应栈的pop。

首先说硬写。既然要我们简化，就只需要遍历一下原目录：

- 遇到名字就贴上去；
- 遇到`.`就不管它；
- 遇到`..`就回到上一级。

这里只需要注意两点：

1. 因为题里说如果有多个`/`，只要一个，并且我们本身也只是用`/`来分割，所以遇到`/`也不管它就行；
2. 怎么回到上一级？其实就是从后往前数，遇到第一个`/`之后，就把它和它后面的字符都删掉。

因此这里需要一个工具方法，就是从path中分割出一个名字：

```cpp
string getElem(string path, int start) {
    if (start >= path.size()) return "";
    int nextSlashIndex = path.find('/', start);
    string elem = path.substr(start, nextSlashIndex - start);
    return elem;
}
```

这个函数的作用画个图：

![[Projects/leetcode/resources/Drawing 2024-12-18 00.02.18.excalidraw.svg]]

然后退出的情况，就是`start = path.size()`的时候，已经找到头了，就直接返回空表示结束。

直接给代码：

```cpp
string Solution::simplifyPath(string path) {
    if (path == "/") return path;
    int i = 1;
    string res;
    while (true) {
        if (path[i] == '/') {
            i++;
            continue;
        }
        string elem = getElem(path, i);
        if (elem.empty()) {
            break;
        }
        if (elem == ".") {
            i++;
            continue;
        }
        if (elem == "..") {
            int lastSlashIndex = res.find_last_of('/');
            res = res.substr(0, lastSlashIndex);
        } else {
            res += "/" + elem;
        }
        i += elem.size();
    }
    // 最后这个是为了应对/../这种情况。
    if (res.empty()) {
        res = "/";
    }
    return res;
}
```

用栈的方法，看了一遍，发现反而还比硬写性能差。。。不过这个思路是没问题的，是更合适的。

- 先把path分成一个个元素，这里只取两个`/`中间的部分，题解里用了一个$\lambda$表达式来做；
- 其实和硬写是一样的，如果是`.`就不管，是`..`就pop，其它情况就push；
- 最后遍历这个栈，直接拼就万事了。

代码很简单，就直接上了。

```cpp
string Solution::simplifyPath2(string path) {
    auto split = [](const string &s, char delim) -> vector<string> {
        vector<string> res;
        string cur;
        for (char ch : s) {
            if (ch == delim) {
                res.push_back(cur);
                cur.clear();
            } else {
                cur += ch;
            }
        }
        res.push_back(cur);
        return res;
    };
    vector<string> names = split(path, '/');
    vector<string> stack;
    for (string &name : names) {
        if (name == ".." && !stack.empty()) {
            stack.pop_back();
        } else if (!name.empty() && name != "." && name != "..") {
	        // 这里name.empty()的判断是因为上面的lambda表达式会让names里有空字符，为了排除这些。
            stack.push_back(name);
        }
    }
    string res;
    if (stack.empty()) {
        res = "/";
    } else {
        for (string &name : stack) {
            res += "/" + name;
        }
    }
    return res;
}
```