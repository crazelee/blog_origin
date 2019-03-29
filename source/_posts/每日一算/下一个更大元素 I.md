---
title: 下一个最大元素 I
date: 2019-02-26 17:54:08
categories: 每日一算
tags:
- 面试
- 算法
copyright: false
---

## 题源
LeetCode题目：https://leetcode-cn.com/problems/next-greater-element-i/

> 给定两个没有重复元素的数组 nums1 和 nums2 ，其中nums1 是 nums2 的子集。找到 nums1 中每个元素在 nums2 中的下一个比其大的值。

> nums1 中数字 x 的下一个更大元素是指 x 在 nums2 中对应位置的右边的第一个比 x 大的元素。如果不存在，对应位置输出-1。

<!--more-->

>示例 1:

>输入: nums1 = [4,1,2], nums2 = [1,3,4,2].
输出: [-1,3,-1]
解释:
    对于num1中的数字4，你无法在第二个数组中找到下一个更大的数字，因此输出 -1。
    对于num1中的数字1，第二个数组中数字1右边的下一个较大数字是 3。
    对于num1中的数字2，第二个数组中没有下一个更大的数字，因此输出 -1。

>nums1和nums2中所有元素是唯一的。
nums1和nums2 的数组大小都不超过1000。


## 解法

关键词：单调栈
![](/images/15512482510276.jpg)

1. 循环遍历较长的数组组成单调递减栈
2. 当栈顶小于当前元素时，出栈，存储到数组或hashmap中
3. 循环较短的数组，查找是否在hashmap中

## 源码

```php
<?php

class Solution {

    /**
     * @param Integer[] $nums1
     * @param Integer[] $nums2
     * @return Integer[]
     */
    function nextGreaterElement($nums1, $nums2) {
        $hashMap = [];
        $stack = [];
        foreach ($nums2 as $value) {
            while (!empty($stack) && end($stack) < $value) {
                $hashMap[array_pop($stack)]=$value;
            }
            array_push($stack, $value);
        }

        $res = [];
        foreach ($nums1 as $value) {
            if (isset($hashMap[$value])) {
                $res[] = $hashMap[$value];
            } else {
                $res[] = -1;
            }
        }

        return $res;
    }
}

$s = new Solution();
$res = $s->nextGreaterElement([1,3,5,2,4],[6,5,4,3,2,1,7]);
var_dump($res);
```



























