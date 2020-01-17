---
title: 114-FlattenBinaryTreetoLinkedList
date: 2020-01-11 11:40:13
tags: [leetcode,BinaryTree]
---

### 114-FlattenBinaryTreetoLinkedList
#### 思路1-递归
通过观察我们可以发现原地咱开就是，把左子树变空，右子树要么接到左子树的右边要么接到当前节点的右边
递归实现如下：
**bugfree 开心😄**
```bash
func flattenRe(node *TreeNode) *TreeNode{
	if node == nil{
		return nil
	}
	nodeLeft := flattenRe(node.Left)
	nodeRight := flattenRe(node.Right)
	tmp := nodeLeft
	if nodeLeft != nil{
		node.Right = nodeLeft
		for tmp.Right != nil {
			tmp = tmp.Right
		}
		tmp.Right = nodeRight
	}else{
		node.Right = nodeRight
	}
	node.Left = nil
	return node
}
```

#### 思路二 非递归原地打开
处理完当前节点，把直接把指针知道当前节点的右节点上，由于当前节点所有子节点都被移到右节点了，所以不用盏就可以实现遍历

```bash
func flatten2 (root *TreeNode){
	if root == nil {
		return
	}
	var temp  *TreeNode
	for root != nil {
		if root.Left != nil{
			temp = root.Left
			for temp.Right != nil {
				temp = temp.Right
			}
			temp.Right = root.Right
			root.Right = root.Left
		}
		root = root.Right
	}
	return
}
```