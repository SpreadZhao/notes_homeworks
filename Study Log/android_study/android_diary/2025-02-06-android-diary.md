---
title: XML is NOT Good
date: 2025-02-06
tags: 
mtrace: 
  - 2025-02-06
---

# XML is NOT Good

今天做需求的时候，突然想到一个XML的缺点。起因是，我希望做AB实验，需要加开关。然后需要实验的是UI。比如一个RelativeLayout的子View，线上情况是`layout_align_bottom`，然后，我希望在实验组改成`layout_below`。那这个时候就有问题了，因为XML是静态资源，所以只能在代码里加，而这个东西其实改起来还挺麻烦，要获取LayoutParams，然后再apply一些rule才行。

然后，我发现Compose就没这问题。因为它本身就是代码驱动的。这样改起来就会方便很多。

- [ ] #TODO tasktodo1738857283414 有时间验证一下这个观点是否正确，用Compose做AB实验是不是更方便。 ➕ 2025-02-06 🔽 🆔 2aetfu 