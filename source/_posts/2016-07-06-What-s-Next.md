---
title: What's Next?
date: 2016-07-06 23:34:03
categories:
- Computers & Technology
- Algorithm
tags:
- String
- Python
---
## 题目

### [What's Next?](https://www.hackerrank.com/challenges/whats-next)
Johnny is playing with a large binary number, *B*. The number is so large that it needs to be compressed into an array 
of integers, *A*, where the values in even indices `(0, 2, 4,...)` represent some number of consecutive `1` bits and 
the values in odd indices `(1, 3, 5,...)` represent some number of consecutive `0` bits in alternating substrings of *B*.

For example, suppose we have array *A* = `{4, 1, 3, 2, 4}`. *A<sub>0</sub>* represents `"1111"`, *A<sub>1</sub>* 
represents `"0"`, *A<sub>2</sub>* represents `"111"`, *A<sub>3</sub>* represents `"00"`, and *A<sub>4</sub>* represents 
`"1111"`. The number of consecutive binary characters in *i<sup>th</sup>* the substring of *B* corresponds to integer 
*A<sub>i</sub>*, as shown in this diagram:

{% asset_img 1.webp %}

When we assemble the sequential alternating sequences of `1`'s and `0`'s, we get *B* = `11110111001111`.

We define setCount(*B*) to be the number of `1`'s in a binary number, *B*. Johnny wants to find a binary number, *D*, 
that is the smallest binary number > *B* where setCount(*B*) = setCount(*D*). He then wants to compress *D* into an 
array of integers, *C* (in the same way that integer array *A* contains the compressed form of binary string *B*).

Johnny isn't sure how to solve the problem. Given array *A*, find integer array *C* and print its length on a new line. 
Then print the elements of array *C* as a single line of space-separated integers.

<!--more-->

#### Input Format
The first line contains a single positive integer, *T*, denoting the number of test cases. Each of the *2T* subsequent 
lines describes a test case over *2* lines:

1. The first line contains a single positive integer, , denoting the length of array *A*.
2. The second line contains  positive space-separated integers describing the respective elements in integer array *A* 
   (i.e., *A<sub>0</sub>*, *A<sub>1</sub>*,..., *A<sub>n-1</sub>*).

#### Constraints
- 1 *T* 100
- 1 *n* 10

#### Subtasks
- For a  score, .
- For a  score, .

#### Output Format
For each test case, print the following  lines:

1. Print the length of integer array  (the array representing the compressed form of binary integer ) on a new line.
2. Print each element of  as a single line of space-separated integers.
It is guaranteed that a solution exists.

#### Sample Input
```Plaintext
1
5
4 1 3 2 4
```

#### Sample Output
```Plaintext
7
4 1 3 1 1 1 3
```

#### Explanation
*A* = `{4, 1, 3, 2, 4}`, which expands to *B* = `11110111001111`. We then find setCount(B) = `11`. The smallest binary 
number > *B* which also has eleven `1`'s is *D* = `11110111010111`. This can be reduced to the integer array *C* 
= `{4, 1, 3, 1, 1, 1, 3}`. This is demonstrated by the following figure:

{% asset_img 2.webp %}

Having found *C*, we print its length (`7`) as our first line of output, followed by the space-separated elements in 
*C* as our second line of output.

大致意思就是一串二进制数，现在用十进制数来表示几个`1`和几个`0`，比如`31`就是3个`1`和1个`0`,`11110`。现在需要在保持`1`的个数不变的情况下，
找出大于他的最小的一个数的十进制表示。

## 思路
用十进制标识有明确的转换说明，不需要考虑，只考虑二进制数的话就是两种情况。

- 最后跟的全是`1`：最简单的就是最后全是`1`，只需要将第一个`1`和前面的`0`互换位置就可以，`100111` => `101011`。

- 最后跟的全是`0`：最后全`0`的时候，需要将前面的全`1`的第一个`1`和左边的`0`呼唤，然后将剩下所有的`1`和右侧所有的`0`互换位置。

代码里面分了四种情况，只有一组`1`的情况单独做了处理，分别是类似于`1111`、`11110000`的这两种。因为要用十进制标识，那互换位置之类的就是对应位的
数值进行加减。

## 代码

### Python
```Python
class Solution(object):
    def one(self, nums):
        if nums[0] - 1 == 0:
            return 2, [1, 1]
        else:
            return 3, [1, 1, nums[0] - 1]

    def two(self, nums):
        if nums[0] - 1 == 0:
            return 2, [1, nums[1] + 1]
        else:
            return 3, [1, nums[1] + 1, nums[0] - 1]

    def odd(self, num, nums):
        if nums[-1] - 1 == 0:
            if nums[-2] - 1 == 0:
                del nums[-2]
                nums[-2] += 1
            else:
                nums.insert(-1, 1)
                nums[-3] -= 1
        else:
            nums[-1] -= 1
            nums.insert(-1, 1)
            if nums[-3] - 1 == 0:
                del nums[-3]
                nums[-3] += 1;
            else:
                nums[-3] -= 1
                nums.insert(-2, 1)
        return len(nums), nums

    def even(self, num, nums):
        if nums[-2] - 1 == 0:
            del nums[-2]
            nums[-1] += 1
            if nums[-2] - 1 == 0:
                del nums[-2]
                nums[-2] += 1
            else:
                nums.insert(-1, 1)
                nums[-3] -= 1
        else:
            nums[-1] += 1
            nums.append(nums[-2] - 1)
            del nums[-3]
            if nums[-3] - 1 == 0:
                nums[-4] += 1
                del nums[-3]
            else:
                nums[-3] -= 1
                nums.insert(-2, 1)
        return len(nums), nums

    def solve(self, num, nums):
        if num == 1:
            return self.one(nums)
        if num == 2:
            return self.two(nums)
        if num % 2 == 1:
            return self.odd(num, nums)
        else:
            return self.even(num, nums)



if __name__ == '__main__':
    solution = Solution()
    total = int(input(""))
    result1 = 0
    result2 = []
    for index in range(total):
        num = int(input(""))
        numsString = input("")
        nums = list(map(int, str.split(numsString)))
        result1,result2 = solution.solve(num, nums)
        print(result1)
        print(" ".join(str(x) for x in result2))
```