---
categories:
- program
date: "2015-04-17T12:00:00Z"
title: 旋转矩形碰撞检测 OBB方向包围盒算法
typora-root-url: ..
---

　　在Cocos2dx中进行矩形的碰撞检测时需要对旋转过的矩形做碰撞检查，由于游戏没有使用Box2D等物理引擎，所以采用了OBB(Oriented bounding box)方向包围盒算法，这个算法是基于SAT(Separating Axis Theorem)分离轴定律的。
<!--more-->

　　**分离轴定律：两个凸多边形物体，如果我们能找到一个轴，使得两个在物体在该轴上的投影互不重叠，则这两个物体之间没有碰撞发生，该轴为Separating Axis。**也就是说两个多边形在所有轴上的投影都发生重叠，则判定为碰撞；否则，没有发生碰撞。

![obb](/images/OBB.jpg)

　　现在，我们来考虑一下矩形，矩形有4条边，那么就有4条轴，由于矩形的对边是平行的，所以有两条轴是重复的，我们仅需要检查相邻的两个轴，那么两个矩形就需要检查4个轴。

检查投影有两种方法：

- **第一种，把每个矩形的4个顶点投影到一个轴上，这样算出4个顶点最长的连线距离，以后同样对待第二个矩形，最后判断2个矩形投影距离是否重叠。**
- **第二种，把2个矩形的半径距离投影到轴上，以后把2个矩形的中心点连线投影到轴上，以后判断2个矩形的中心连线投影，和2个矩形的半径投影之和的大小。**

　　由于已经有很多文章来介绍OBB的原理，所以这里并不过多解释，我只将我实现的源码列出来仅供大家参考，代码已经经过测试，如下：
```cpp
#ifndef _OBBRECT_H_
#define _OBBRECT_H_

#include <math.h>

class OBBRect {
public:
	OBBRect(float x, float y, float width, float height,
		float rotation = 0.0f)
		: x(x), y(y), width(width), height(height),
			rotation(rotation) {
		resetVector();
	}

	bool intersects(OBBRect& other) {
		float distanceVector[2] = {
			other.x - x,
			other.y - y
		};
		for (int i = 0; i < 2; ++i) {
			if (getProjectionRadius(vectors[i]) +
				other.getProjectionRadius(vectors[i])
				<= dot(distanceVector, vectors[i])) {
				return false;
			}
			if (getProjectionRadius(other.vectors[i]) +
				other.getProjectionRadius(other.vectors[i])
				<= dot(distanceVector, other.vectors[i])) {
				return false;
			}
		}
		return true;
	}

private:
	void resetVector() {
		vectors[0][0] = cos(rotation);
		vectors[0][1] = sin(rotation);
		vectors[1][0] = -vectors[0][1];
		vectors[1][1] = vectors[0][0];
	}

	float dot(float a[2], float b[2]) {
		return abs(a[0] * b[0] + a[1] * b[1]);
	}

	float getProjectionRadius(float vector[2]) {
		return (width * dot(vectors[0], vector) / 2 
			+ height * dot(vectors[1], vector) / 2);
	}

	float x;
	float y;
	float width;
	float height;
	float rotation;
	float vectors[2][2];
};

#endif // _OBBRECT_H_
```