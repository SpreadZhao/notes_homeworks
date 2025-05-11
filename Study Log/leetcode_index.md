```dataviewjs
let res = [];
const pages = dv.pages('"Projects/leetcode"');
for (const page of pages) {
	if (page.num == undefined) {
		continue;
	}
	const num = page.num;
	const title = "[[" + page.file.path + "|" + page.title + "]]";
	const link = page.link;
	const difficulty = "#" + page.tags[0];
	const tags = page.tags.slice(1).map(tag => `#${tag}`);
	let realtag = "";
	if (tags != undefined) {
		for (let i = 0; i < tags.length; i++) {
			const tag = tags[i]
			if (tag.indexOf("#") !== 0) {
				tags[i] = "#" + tag
			}
			realtag = realtag + tags[i]
			if (i != tags.length - 1) {
				realtag += " "
			}
		}
	} else {
		realtag += "No tag"
	}
	res.push({num, title, link, difficulty, realtag});
}
res.sort((a, b) => a.num - b.num);
dv.table(
	["Num", "Title", "Link", "Difficulty", "Tags"],
	res.map(x => [x.num, x.title, x.link, x.difficulty, x.realtag])
);
```

## TODO

```tasks
path regex matches /Projects/leetcode/
tags include TODO
sort by priority
```