---
title: Median of Two Sorted Arrays 双数组中位数
date: 2016-07-08 16:37:43
categories: 
- Computers & Technology
- Algorithm
tags: 
- Binary Search
- Array
- Divide and Conquer
- Python
---
## 题目

### [4. Median of Two Sorted Arrays](https://leetcode.com/problems/median-of-two-sorted-arrays/)
There are two sorted arrays nums1 and nums2 of size m and n respectively.
Find the median of the two sorted arrays. The overall run time complexity should be O(log (m+n)).

求两个有序数组合并后的中位数。

<!--more-->

#### 例子
```Plaintext
nums1 = [1, 3]
nums2 = [2]

The median is 2.0
```

```Plaintext
nums1 = [1, 2]
nums2 = [3, 4]

The median is (2 + 3)/2 = 2.5
```

## 思路
首先需要一个前提条件，那就是两个有序数组，他们的中位数位置都很容易知道，当合并时，如果第二个数组中位数比第一个数组大，那么合并后的中位数一定不在第一个数组中位数的左边。把数字对应到天平上，每个数字重量一样的话，中位数就是保持平衡的支撑点，另一个数组合并进去因为第一个数组中位数在第二个数组中位数左边，所以到第一个数组中位数右边的数要多于左边的数，如果支撑点不移动右侧肯定比左侧沉，自然支撑点要右移。

有了这个前提，就不需要合并之后对所有的数进行排序了，只要计算出最后这个重心向右偏移了多少步，就可以算出合并后的中位数了。如果偏移了两位，那就从这两个数组里面往大的方向数两位。这里需要求出第一个数组中位数在第二个数组中的位置，使用二分法就可以做到题目中要求的`O(log (m+n))`的复杂度了。

## 代码

### Python
```Python
class Solution(object):
    def findMedianSortedArrays(self, nums1, nums2):
        """
        :type nums1: List[int]
        :type nums2: List[int]
        :rtype: float
        """
        # 获取中位数所在索引，长度+1后折半算出位数，-1转换为数组下标
        medianIndex1 = ((len(nums1) + 1) / 2.0) - 1
        medianIndex2 = ((len(nums2) + 1) / 2.0) - 1

        # 找出中位数
        if medianIndex1 >= 0:
            if medianIndex1 % 1 == 0:
                median1 = nums1[int(medianIndex1)]
            else:
                median1 = (nums1[int(medianIndex1 // 1)] + nums1[int(medianIndex1 // 1) + 1]) / 2.0
        if medianIndex2 >= 0:
            if medianIndex2 % 1 == 0:
                median2 = nums2[int(medianIndex2)]
            else:
                median2 = (nums2[int(medianIndex2 // 1)] + nums2[int(medianIndex2 // 1) + 1]) / 2.0

        if medianIndex1 < 0:
            return median2
        if medianIndex2 < 0:
            return median1

        # 让第一列中位数在前，第二列中位数在后，中位数相等直接返回
        if median1 == median2:
            return median1
        if median1 < median2:
            numsList1, numsList2 = nums1, nums2
        else:
            numsList1, numsList2 = nums2, nums1
            median1, median2 = median2, median1
            medianIndex1, medianIndex2 = medianIndex2, medianIndex1

        # 偏移量记录结果中位数相对于第一列的偏移量
        offset = 0

        # 计算偏移量
        median1InNumsList2 = -1
        if median1 < numsList2[0]:
            offset = len(numsList2) / 2.0
        else:
            start = 0
            end = int(medianIndex2 + 0.5)
            while start < end:
                mid = start + (end - start) // 2
                if numsList2[int(mid)] < median1:
                    start = mid + 1
                elif numsList2[int(mid)] > median1:
                    end = mid - 1
                else:
                    median1InNumsList2 = mid
                    break
            if numsList2[start] > median1:
                median1InNumsList2 = start - 0.5
            if numsList2[start] < median1:
                median1InNumsList2 = start + 0.5
            if numsList2[start] == median1:
                median1InNumsList2 = start
            offset = medianIndex2 - median1InNumsList2

        if offset == 0:
            return median1

        # 按偏移查找中位数，保证指针都在中位数后面
        if median1InNumsList2 % 1 == 0.5:
            ptr2 = int(median1InNumsList2 + 0.5)
        else:
            ptr2 = int(median1InNumsList2 + 1)

        result = median1
        # 处理第一个中位数为0.5的情况
        if medianIndex1 % 1 == 0.5:
            # 如果第一个中位数和第二个数组中有数字重合，相当于把这个数字放入了第一组数种，那么偏移不需要补偿
            if median1InNumsList2 % 1 == 0.5 or median1InNumsList2 == -1:
                offset += 0.5
            ptr1 = int(medianIndex1 + 0.5)
        else:
            ptr1 = int(medianIndex1 + 1)

        # 进行偏移
        while offset > 0.5:
            if ptr1 >= len(numsList1):
                result = numsList2[ptr2]
                ptr2 += 1

            elif ptr2 >= len(numsList2):
                result = numsList1[ptr1]
                ptr1 += 1

            elif numsList1[ptr1] > numsList2[ptr2]:
                result = numsList2[ptr2]
                ptr2 += 1

            else:
                result = numsList1[ptr1]
                ptr1 += 1

            offset -= 1

        # 还剩半个偏移时取中间值
        if offset == 0.5:
            if ptr1 >= len(numsList1):
                return (result + numsList2[ptr2]) / 2.0

            elif ptr2 >= len(numsList2):
                return (result + numsList1[ptr1]) / 2.0

            elif numsList1[ptr1] > numsList2[ptr2]:
                return (result + numsList2[ptr2]) / 2.0

            else:
                return (result + numsList1[ptr1]) / 2.0

        return result
```