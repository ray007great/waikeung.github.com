---
layout: post
title: "浅酌算法:希尔排序"
description: "关于希尔排序的一些事"
category: 数据结构与算法
tags: [算法, 排序]
---

希尔排序是插入排序的扩展，通过允许非相邻的元素进行交换来提高执行效率。

希尔排序的基本思路是，将待排序的序列分成几个区域。插入排序的间隔是1,而希尔排序以更大的间隔来比较和交换元素，这样最大的元素就能更快的到达序列的结尾。

1. 选取一个步长序列，
2. 按步长序列的个数k，对序列进行k糖排序
3. 每趟排序跟据对应的步长gap，将待排序的序列分割若干段子序列，分别对各子序列进行插入排序。
4. 最后一趟的步长为1,就是直接插入排序。

待排序的数组 49,38,65,97,76,13,27,55,04,01

在这里我们设定步长gap为`n/2`，并对步长取半值直到1。

第一趟排序: gap = n/2 = 5

将数组分组

```
49 38 65 97 76
13 27 55 04 01
```

将每列进行排序

```
13 27 55 04 01
49 38 65 97 76
```

这样一趟排序完成，数组变为 13,27,55,04,01,49,38,65,97,76

第二趟排序：gap = 5/2 = 2

将数组分组

```
13 27
55 04
01 49
38 65
97 76
```
同样将每列进行排序

```
01 04
13 27
38 49
55 65
97 76
```
第二趟排序完成，数组变为 01,04,13,27,38,49,55,65,97,76

第三趟排序:gap = 2/2 = 1

当步长为1时，就是最后一趟排序，直接插入排序就可以了。到这趟之前的一趟，数组已经基本有序，从这个例子看，上趟结束后，就剩最后两个元素还没到正确位置，这时用直接插入排序，移动的元素就少了。

第三趟排序完成，数组变为 01,04,13,27,38,49,55,65,76,97 。数组排序完成。

```c
void shellsort(int array[], int n)
{
    int i, j, k, gap;
    int key;

    for (gap = n/2; gap > 0; gap /= 2) {    /* step */
        for (i = 0; i < gap; i++) {         /* gap insert sort*/
            for (j = i+gap; j < n; j += gap) {  /* one insert sort*/
                if (array[j] < array[j-gap]) {
                    key = array[j];
                    k = j - gap;
                    while (k >= 0 && array[k] > key) {
                        array[k+gap] = array[k];
                        k -= gap;
                    }
                    array[k+gap] = key;
                }
            }
        }
    }
}
```

希尔排序的步长选择是重要的部分。可以参照[维基百科](http://zh.wikipedia.org/wiki/%E5%B8%8C%E5%B0%94%E6%8E%92%E5%BA%8F)。
