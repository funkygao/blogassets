---
title: perceptual hash
date: 2017-08-24 07:43:24
tags: algorithm
---

这种算法的优点是简单快速，不受图片大小缩放的影响，缺点是图片的内容不能变更。如果在图片上加几个文字，它就认不出来了。所以，它的最佳用途是根据缩略图，找出原图。
 
实际应用中，往往采用更强大的pHash算法和SIFT算法，它们能够识别图片的变形。只要变形程度不超过25%，它们就能匹配原图。这些算法虽然更复杂，但是原理与上面的简便算法是一样的，就是先将图片转化成Hash字符串，然后再进行比较。


比较图片的相似度
图片尺寸可以不同，不同的长宽比例，小范围的颜色不同(亮度、对比度等)
对旋转都不具有鲁棒性：但可以sample argument

```
img = resize(img, (8, 8))
img.grayscalize(level=64)
avg = img.average() // e,g. 78
for i, b = range img { // (8, 8) => img[0...63]
    if b >= avg {
        img[i] = 1
    } else {
        img[i] = 0
    }
}
// img现在就是一个64位的二进制整数, 这就是这张图片的指纹

HanmingDistance(img1, img2) // 汉明距离，如果小于5就说明两副图相似
```

