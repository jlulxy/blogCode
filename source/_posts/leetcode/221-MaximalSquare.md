---
title: 221-MaximalSquare
date: 2020-01-11 12:12:18
tags: [leetcode,DP]
---

### 221-MaximalSquare
```bash
Given a 2D binary matrix filled with 0's and 1's, find the largest square containing only 1's and return its area.

Example:

Input: 

1 0 1 0 0
1 0 1 1 1
1 1 1 1 1
1 0 0 1 0

Output: 4

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/maximal-square
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。
```

###  解题思路
可以想到当前节点如果能作为最大的正方形+1，那他上面一定是一个正方形，且形成的正方形，包含其上左右三个点，
所以以该点为底点的变成应该是上左右三点变长+1，dp如下

```bash
func maximalSquare(matrix [][]byte) int {
	//使用dp转移
	//dp[i][j] = min(dp[i-1,j-1],dp[i-1],[j],dp[j]) +1
	rows := len(matrix)
	if rows <= 0 {
		return 0
	}
	cols:= len(matrix[0])
	dp:= make([][]int,len(matrix))
	len := 0
	//初始化dp数组
	for i:=0;i<rows;i++{
		dp[i] = make([]int,cols)
		for j :=0 ;j<cols;j++{
			if matrix[i][j] == '1' {
				if i==0||j==0 {
					dp[i][j] = 1
				}else{
					dp[i][j] = min(dp[i-1][j-1],dp[i-1][j],dp[i][j-1]) +1
				}
				}else{
				dp[i][j]=0
			}
			if dp[i][j] > len{
				len = dp[i][j]
			}
		}
	}
	return len *len
}
```