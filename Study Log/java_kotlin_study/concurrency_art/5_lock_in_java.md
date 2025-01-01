---
title: 5 Java中的锁
order: "5"
chapter_root: true
---

```dataviewjs
let res = []
for (let page of dv.pages('"Study Log/java_kotlin_study/concurrency_art"')) {
	if (page.chapter == 5) {
		const link = page.file.link
		const title = page.title
		const order = page.order
		const info = link + ": " + title
		res.push({info, order})
	}
}
res.sort((a, b) => a.order - b.order)
dv.list(res.map(x => x.info))
```

- [ ] #TODO tasktodo1735668626456 有时间看看这个StampedLock：[深入理解 Java StampedLock：原理、实践与面试指南](https://mp.weixin.qq.com/s/vuTQ8Khu1r5PjBmD0OaJEw)。 ➕ 2025-01-01 ⏫ 🆔 l97f4t 