---
title: OpenCV边缘检测
tags:
  - OpenCV
  - 边缘检测
categories:
  - 文章
toc: false
date: 2019-09-14 14:04:05
---

# OpenCV边缘检测

![image.png](/images/2019/09/14/54fef350-d6b5-11e9-ae59-4f17e83b2f0c.png)

```python
import cv2
import numpy as np
from matplotlib import pyplot as plt

img = cv2.imread('1024.jpg',0)
edges = cv2.Canny(img,100,200)

plt.subplot(121),plt.imshow(img,cmap='gray')
plt.title('original'),plt.xticks([]),plt.yticks([])
plt.subplot(122),plt.imshow(edges,cmap='gray')
plt.title('edge'),plt.xticks([]),plt.yticks([])

plt.show()
```

