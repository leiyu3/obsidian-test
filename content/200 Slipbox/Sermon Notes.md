---
cdate: 2023-12-22
aliases: 
title: Sermon Notes
publish: true
tags:
  - category
---
Some sermon note that I have curated over the years. Still figuring out if I want to have these saved here… Doesn’t seem to be very helpful (for me at least) to archive them like this.

[[../attachments/Sunday Sermon.pdf|Sunday Sermon.pdf]]


```dataview
TABLE without id
	link(file.link, title) as "Sermon Title",
	speaker as Speaker,
	dateformat(cdate, "yyyy-MM-dd") as Date,
	scripture-passage as Passage
FROM #sermon 
SORT cdate DESC
```
