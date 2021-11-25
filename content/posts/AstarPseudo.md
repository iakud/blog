---
categories:
- program
date: "2015-09-09T12:00:00Z"
excerpt: ""
title: Astar伪代码
---

简单说明Astar算法_(伪代码)_：
```bash
while (OPEN != NULL) {
	从OPEN表中取估价值f最小的节点n;
	if (n节点==目标节点) {
		break;
	}

	for (当前节点n的每个子节点X) {
		算X的估价值;
		if (X in OPEN) {
			if (X的估价值小于OPEN表的估价值){
				把n设置为X的父亲;
				更新OPEN表中的估价值; // 取最小路径的估价值
			}
		}

		if (X in CLOSE) {
			if (X的估价值小于CLOSE表的估价值) {
				把n设置为X的父亲;
				更新CLOSE表中的估价值;
				把X节点放入OPEN; //取最小路径的估价值
			}
		}

		if (X not in both) {
			把n设置为X的父亲;
			求X的估价值;
			并将X插入OPEN表中; //还没有排序
		}
	} // end for

	将n节点插入CLOSE表中;
	按照估价值将OPEN表中的节点排序;
	// 实际上是比较OPEN表内节点f的大小，从最小路径的节点向下进行。
} // end while (OPEN != NULL)
```