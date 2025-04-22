---
link: https://youtube.com/playlist?list=PLblh5JKOoLUICTaGLRoHQDuF_7q2GfuJF&si=EXNk6p-uIFzwF_vL
---

# StatQuest Learning

```dataviewjs
let res = []
for (let page of dv.pages('"Study Log/machine_learning/StatQuest"')) {
	const date = new Date(page.date)
	const link = "[[" + page.file.path + "|" + getDateString(date) + "]]"
	let title = page.title
	res.push({link, date, title})
}
function getDateString(date) {
	const year = date.getFullYear()
	const month = date.getMonth() + 1
	const day = date.getDate()
	return year + "年" + month + "月" + day + "日"
}
res.sort((a, b) => b.date - a.date)
dv.table(
	["Date&Link", "Title"], 
	res.map(x => [x.link, x.title])
)
```