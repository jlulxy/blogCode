---
title: 34SearchRange
date: 2020-01-17 17:33:27
tags:
---
### 34-SearchRange

```bash
Given an array of integers nums sorted in ascending order, find the starting and ending position of a given target value.

Your algorithm's runtime complexity must be in the order of O(log n).

If the target is not found in the array, return [-1, -1].

Example 1:

Input: nums = [5,7,7,8,8,10], target = 8
Output: [3,4]
Example 2:

Input: nums = [5,7,7,8,8,10], target = 6
Output: [-1,-1]

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/find-first-and-last-position-of-element-in-sorted-array
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。
```

### 方法1
分别找到最左端和最右端,正常二分查找找到值直接返回，这里我们只需要对=情况区分处理，
找最左的时候如果发现目标相等，移动右指针，是左右想相等的情况逼近最左值
找右的时候正好相反
```bash
func searchRange(nums []int, target int) []int {
	left :=binaryLeftSerach(nums,target)
	right :=binaryRightSerach(nums,target)
	return []int{left,right}
}

func binaryLeftSerach(nums[]int ,target int ) int {
	lenN := len(nums)
	if lenN ==0 {
		return -1
	}
	right,left,mid := lenN,0,0
	//开区间[0,n) 左闭右开所以左边要+1 右边不处理，相等的时候右边动往地位走，left和right一点点逼近最左值，
	// 极限条件是left=lenN才能退出循环，所以判断下极限条件 当然如果right=lenN-1可不用判断
	for left < right {
		mid = (left +right) /2
		if nums[mid] < target{
			left = mid+1
		}else{
			right = mid
		}
	}
	if left == lenN{
		return -1
	}
	if nums[left] ==target {
		return left
	}
	return  -1
}

func binaryRightSerach(nums[]int ,target int ) int {
	lenN := len(nums)
	if lenN ==0 {
		return -1
	}
	right,left,mid := lenN,0,0
	//开区间[0,n) 左闭右开所以左边要+1 右边不处理，相等的时候左边动往高位走，left和right一点点逼近最有右值，
	// 极限条件是left=lenN才能退出循环，所以判断下极限条件 当然如果right=lenN-1可不用判断
	for left < right {
		mid = (left +right) /2
		/*
		if nums[mid] == target{
			left = mid + 1
		}else if nums[mid] > target{
			right = mid
		}else if nums[mid] < target{
			left = mid + 1
		}
		*/
		if nums[mid] > target{
			right = mid
		} else{
			left = mid + 1
		}
	}
	if right - 1 < 0 {
		return -1
	}
	if nums[right-1] == target{
		return right-1
	}
	return  -1
}
```

### 放在一个函数方法

```bash
func searchRangeV2(nums []int, target int) []int {
	l, r := 0, len(nums)-1
	if r < 0 {
		return []int{-1, -1}
	}
	//找左边界，所以右边界=或>target都收缩右边界，当l=r的时候就是左边界
	for l < r {
		mid := (l + r) / 2
		if nums[mid] < target {
			l = mid + 1
		} else {
			r = mid
		}
	}
	res1 := l
	l, r = 0, len(nums)-1
	for l < r {
		mid := (l+r)/2 + 1
		if nums[mid] > target {
			r = mid - 1
		} else {
			l = mid
		}
		fmt.Println(l, r)
	}
	res2 := l
	if nums[res1] != target {
		return []int{-1, -1}
	} else {
		return []int{res1, res2}
	}
}
```