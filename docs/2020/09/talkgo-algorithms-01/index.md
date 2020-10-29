# TalkGo 算法之美第一期

本文介绍 TalkGo 算法之美第一期题解

<!--more-->

## 一、前言

[talkgo地址](https://talkgo.org/t/topic/875/2)

*题目一*

输入一个递增排序的数组和一个数字 S，在数组中查找两个数，使得他们的和正好是 S，如果有多对数字的和等于 S，输出两个数的乘积最小的。

[LeetCode](https://leetcode-cn.com/problems/two-sum/)

*题目二*

一个有序数组，从随即一位截断，把前段放在后边，如 4 5 6 7 1 2 3 求中位数

## 二、解析

### 2.1 题目一

思考：
- 因为递增，负数和正数的情况一致，一组数据最小的值越小则乘积也越小
- 如果出现重复数字，也和正常情况一样的处理

#### 2.1.1 处理方式

1. 暴力法

```go
func TwoSum2For(sum int, array []int) []int {
	b, arr := checkArray(array)

	if !b {
		return arr
	}

	for i := 0; i < len(array); i++ {
		for j := len(array) - 1; j > i; j-- {
			if sum == array[i]+array[j] {
				return []int{array[i], array[j]}
			}
		}
	}

	return []int{}
}
```

直接两层 for 循环，如何匹配到数组两个位置的值等于传入的和，则直接返回。此方法简单易懂，可读性高。

- 时间复杂度 O(n^2)
- 空间复杂度 O(1)

2. 缓存法

```go
// TwoSumMap map 缓存法
func TwoSumMap(sum int, array []int) []int {
	b, arr := checkArray(array)

	if !b {
		return arr
	}

	// 通过 map 缓存数据
	cache := make(map[int]int, len(array))
	for i := range array {
		cache[array[i]] = i
	}

	// 返回值在数组中的位置
	for i := range array {
		// 通过减法找到另一个值，直接去 map 查找
		if _, ok := cache[sum-array[i]]; ok {
			return []int{array[i], sum - array[i]}
		}
	}

	return []int{}
}
```

借助 map 把数组的值进行了缓存，用到了加法变减法的思路根据 hash 算法可以直接找到对应的另一个值是否存在。这个写法很直观，比第一种暴力法更加简单，建议平日小数量的时候直接用就行。

- 时间复杂度 O(n)
- 空间复杂度 O(2n)

3. 双指针法

```go
func TwoSum2Point(sum int, array []int) []int {
	b, arr := checkArray(array)

	if !b {
		return arr
	}

	left, right := 0, len(array)-1
	for left < right {
		s := array[left] + array[right]
		if sum == s {
			return []int{array[left], array[right]}
		} else if sum > s {
			left++
		} else {
			right--
		}
	}

	return []int{}
}
```

双指针法是字符查找的时候经常使用的方法，在数据量大的场景下会比较有优势。它的写法代码看着比较高级。

### 2.2 题目二

思考：
- 先要回想下什么是中位数，奇偶的处理方式是不一样的
- 找到截断的位置，计算中位数涉及到的数字的 index

偶数中位数的一个例子：
{{< figure src="even.jpg" title="偶数" >}}

> [中位数在线学习](https://www.shuxuele.com/median.html)

代码：
```go
// MiddleNum 算中位数
func MiddleNum(nums []int) (float32, error) {
	l := len(nums)
	if l <= 0 {
		return 0, illegalArray
	}

	if l == 1 {
		return float32(nums[0]), nil
	} else if l == 2 {
		return float32((nums[0] + nums[1]) / 2), nil
	}

	p := Point(nums)

	if isEven(l) {
		target1 := l / 2
		target2 := l/2 + 1
		return (float32(nums[Position(l, p, target1)]) + float32(nums[Position(l, p, target2)])) / 2, nil
	} else {
		return float32(nums[Position(l, p, (l+1)/2)]), nil
	}
}

// Point 得到拐点
func Point(nums []int) int {
	if len(nums) < 3 {
		return 0
	}

	n, flag := nums[0], -1

	for i := 1; i < len(nums); i++ {
		cf := -1
		c := nums[i]
		if c < n {
			cf = 1 // 降
		} else if c > n {
			cf = 0 // 升
		}

		if flag >= 0 && cf >= 0 && cf != flag {
			return i
		}

		n = c
		if cf >= 0 {
			flag = cf
		}
	}

	return 0
}

// Position 获取位置
func Position(l, p, t int) int {
	if l-p > t {
		return p + t - 1
	} else {
		return t + p - l - 1
	}
}
```

## 代码

[题1](https://github.com/cityiron/algorithms/tree/main/talkgo/1)

[题2](https://github.com/cityiron/algorithms/tree/main/talkgo/2)

