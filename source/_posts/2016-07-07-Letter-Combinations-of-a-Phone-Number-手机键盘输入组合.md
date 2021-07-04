---
title: Letter Combinations of a Phone Number 手机键盘输入组合
date: 2016-07-07 12:50:29
categories: 
- Algorithms
- Medium
tags: 
- Backtracking
- String
- Python
---
## 题目

### [17. Letter Combinations of a Phone Number](https://leetcode.com/problems/letter-combinations-of-a-phone-number/)
Given a digit string, return all possible letter combinations that the number could represent.
A mapping of digit to letters (just like on the telephone buttons) is given below.

{% asset_img 1.webp %}

用户按数字键，程序输出所有的可能组合。

<!--more-->

#### 例子
```shell script
Input:Digit string "23"
Output: ["ad", "ae", "af", "bd", "be", "bf", "cd", "ce", "cf"].
```

## 思路
如果只有两个按键`1`、`2`，对应`[A, B, C]`、`[D, E, F]`，那么它有 3×3=9 种组合，为了一个不漏首先从最后一个来变换，也就是先写`A`，然后轮流写`D`、`E`、`F`，生成`AD`、`AE`、`AF`后将第一位换为B，这样有顺序的排列组合可以保证不遗漏所有情况。

接下来就是使用代码实现这种排列组合了，相当于是循环第一位可能对应的字母，每个字母后面再循环第二位可能对应的字母，可以使用递归来实现，但是一般语言递归都有深度限制，题目中未限制用户输入长度，所以需要将递归转换为循环。

如何将递归转换为循环，可以类比生活中箱子上的密码锁，4位的密码锁如果想破解可能有10000种组合，肯定是从`0000`开始试，一直到`9999`才不会漏掉，假设用户输入的数字每个可以对应3种可能，那就可以将这些组合用3进制表示，只需要列出`0000`、`0001`、`0002`、`0010`到`2222`就包含了所有组合，每一位数字对应用户输入所可能映射的列表下表。因为`7`、`9`对应4种字母，所以用4进制表示，遇到输入不是`7`、`9`的就跳过第四种可能。

## 代码

### Python
```python
class Solution(object):
    keyMapping = [[], [], ['a', 'b', 'c'], ['d', 'e', 'f'],
                  ['g', 'h', 'i'], ['j', 'k', 'l'], ['m', 'n', 'o'],
                  ['p', 'q', 'r', 's'], ['t', 'u', 'v'], ['w', 'x', 'y', 'z']]

    def letterCombinations(self, digits):
        """
        :type digits: str
        :rtype: List[str]
        """
        if digits == "":
            return []
        lenth = len(digits)
        result = [];
        num = 0
        while num < 4 ** lenth:
            str = ""
            numCopy = num
            for index in range(lenth):
                deep = numCopy // 4 ** (lenth - 1 - index)
                numCopy -= 4 ** (lenth - 1 - index) * deep
                if digits[index] != '7' and digits[index] != '9' and deep == 3:
                    num += 4 ** (lenth - 1 - index)
                    break
                str += self.keyMapping[int(digits[index])][deep]
            if len(str) != lenth:
                continue
            result.append(str)
            num += 1
        return result
```

## 优化
提交后耗时60ms，最快的用户是36ms，这里想到了两种优化方式
- 因为是模拟了4进制，数字比较特殊，在做数字与4进制变换时，可以用位运算移2位来代替除法，应该可以加快速度。
- 可以先跑表，既然可以列出0-9对应的字母，那么也可以列出00-99所对应的字母，`1`对应`[A, B, C]`，那么`11`就对应`[AA, AB, AC, BA, BB, BC, CA, CB, CC]`。将此表事先写好在程序里，处理用户输入的时候就可以两位两位处理，应该可以加快一倍速度。
