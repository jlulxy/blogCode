---
title: 673-NumberofLongestIncreasingSubsequence
date: 2020-01-18 13:59:39
tags: [array,leetcode]
---

```bash
func findNumberOfLIS(nums []int) int {
	//DP[j] = max(DP[j],DP[i]+1) 0<i<j if nums[j]>nums[i]
	//numsLen[j]  以j为结尾的最长递增序列的长度
	if len(nums) == 0{
		return  0
	}
	dp := make([]int,len(nums))
	numsLen := make([]int,len(nums))
	maxLen := 0
	for i := 0 ;i <len(dp);i++{
		dp[i] = 1
		numsLen[i] = 1
	}
	for i:=0;i<len(nums);i++{
		for j:=0;j<i;j++{
			if nums[i] >nums[j] {
				// 更新最大长度，把之前的排列加起来
				if  dp[j] + 1 > dp[i] {
					numsLen[i] = numsLen[j]
					dp[i] = dp[j]+1
				}else if dp[j] +1 == dp[i]{
					numsLen[i] += numsLen[j]
				}
			}
		}
		if maxLen < dp[i] {
			maxLen = dp[i]
		}

	}
	ret := 0
	for i := 0;i<len(nums);i++{
		if maxLen == dp[i] {
			ret += numsLen[i]
		}
	}
	return ret

}

func Max(a,b int) int {
	if a>b{
		return  a
	}
	return b
}
```
