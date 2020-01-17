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
分别找到最左端和最右端
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
	for left < right {
		mid = (left +right) /2
		if nums[mid] == target{
			right = mid
		}else if nums[mid] >target{
			right = mid
		}else if nums[mid] < target{
			left = mid + 1
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
	for left < right {
		mid = (left +right) /2
		if nums[mid] == target{
			left = mid + 1
		}else if nums[mid] > target{
			right = mid
		}else if nums[mid] < target{
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