---
title: 236-LowestCommonAncestorofaBinaryTree
date: 2020-01-12 12:16:35
tags: [leetcode,BinaryTree]
---

### 236. Lowest Common Ancestor of a Binary Tree
```bash
Given a binary tree, find the lowest common ancestor (LCA) of two given nodes in the tree.

According to the definition of LCA on Wikipedia: “The lowest common ancestor is defined between two nodes p and q as the lowest node in T that has both p and q as descendants (where we allow a node to be a descendant of itself).”

Given the following binary tree:  root = [3,5,1,6,2,0,8,null,null,7,4]


 

Example 1:

Input: root = [3,5,1,6,2,0,8,null,null,7,4], p = 5, q = 1
Output: 3
Explanation: The LCA of nodes 5 and 1 is 3.
Example 2:

Input: root = [3,5,1,6,2,0,8,null,null,7,4], p = 5, q = 4
Output: 5
Explanation: The LCA of nodes 5 and 4 is 5, since a node can be a descendant of itself according to the LCA definition.
 

Note:

All of the nodes' values will be unique.
p and q are different and both values will exist in the binary tree.

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/lowest-common-ancestor-of-a-binary-tree
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。
```
#### 思路如下：
要记住两个点，从跟节点到目标节点p，q 因该是root->*->*...(一直都是单节点)，到最近父节点分拆分别指向p，q(p,q相等除外)
所以要找的节点有以下特点：
1. 前面都是单路径只有，只有目标节点在其自身或子树下有能左右两个正确返回
2. root到p，q的路径上，到最进夫节点分叉节点
3. 如果使用中序遍历，p，q之间的结果节点应该是中序遍历中节点最浅的节点
```bash
/**
 * Definition for TreeNode.
 * type TreeNode struct {
 *     Val int
 *     Left *ListNode
 *     Right *ListNode
 * }
 */
 package main

 //初始化_ans以防未找到空指针
 var _ans *TreeNode =&TreeNode{}

 func lowestCommonAncestor(root, p, q *TreeNode) *TreeNode {
	RecursionAns(root,p,q)
    return _ans
}


//递归
//由于是二叉树，只有一个节点的子节点能包含两个节点，该节点的父节点只能有左节点有或右节点有搜索节点
func RecursionAns(node,p,q *TreeNode) bool{
	ret := false
	if node ==nil {
		return false
	}
	mid ,left,right := 0,0,0
	if RecursionAns(node.Left,p,q) {
		left = 1
	}
	if RecursionAns(node.Right,p,q) {
		right = 1
	}
	if node == p||node == q{
		mid = 1
	}
	if (mid + left + right) > 1{
		 ret = true
	}
	//如果自己或者左右大于2证明找到该节点，由于是二叉树全局只有一个节点符合条件不知道能否体检结束
	if (mid + left +right) >= 2{
		_ans = node
	}
	return ret
}
```
