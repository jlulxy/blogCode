---
title: leetcode-297BSTserilize
date: 2020-01-02 21:16:08
tags: [leetcode,BST]
---

### leetcode-297 BST序列化与反序列化

#### 题目要求如下
```
序列化是将一个数据结构或者对象转换为连续的比特位的操作，进而可以将转换后的数据存储在一个文件或者内存中，同时也可以通过网络传输到另一个计算机环境，采取相反方式重构得到原数据。

请设计一个算法来实现二叉树的序列化与反序列化。这里不限定你的序列 / 反序列化算法执行逻辑，你只需要保证一个二叉树可以被序列化为一个字符串并且将这个字符串反序列化为原始的树结构。

示例: 

你可以将以下二叉树：

    1
   / \
  2   3
     / \
    4   5

序列化为 "[1,2,3,null,null,4,5]"
提示: 这与 LeetCode 目前使用的方式一致，详情请参阅 LeetCode 序列化二叉树的格式。你并非必须采取这种方式，你也可以采用其他的方法解决这个问题。
说明: 不要使用类的成员 / 全局 / 静态变量来存储状态，你的序列化和反序列化算法应该是无状态的。
```

#### 思路分析
1.首先第一步我们要能遍历二叉树才能够将二叉树变成字符串存储（二叉树的遍历这里我们采用先序遍历）
2.树节点中有的值不知以为要能区分出那些数字字符是一个节点的值，这里我们定义了分隔符“|”
3.识别数的左右空节点，这里我们定义了占位符“#”
4.序列化开始，注意如果根为空直接插入占位符和分隔符
5.反序列化时候要注意采用和序列化一样的顺序，同时提取出数据后要做好指针的移动（拿到连续数据后，指针后移1位跨个分隔符，遇到占位符后移两位跨过占位符和分隔符）
6.递归的处理左子数和右子数
``` bash
package main

import (
	"strconv"
	"fmt"
)

// Your Codec object will be instantiated and called as such:
// Codec codec;
// codec.deserialize(codec.serialize(root));

const (
	PlaceHolder = "#"
	Delimiter   = "|"
)
type TreeNode struct{
	val int
	left *TreeNode
	right *TreeNode
}
func serialize(root *TreeNode) string{
	restr := ""
	//序列化
	if root == nil {
		restr += PlaceHolder
		return  restr
	}
	return  serilaizeDe(root)
}

func serilaizeDe(root *TreeNode) string{
	str := ""
	if root == nil {
		str = str + PlaceHolder + Delimiter
		return  str
	}
	str = strconv.FormatInt(int64(root.val),10) + Delimiter
	return str + serilaizeDe(root.left)+ serilaizeDe(root.right)

}
func deserialize(data string) *TreeNode{
	loc := 0
	if string(data[loc]) == PlaceHolder {
		return nil
	}
	return deserilaizeDe(data,&loc)
}
func deserilaizeDe ( data string,loc *int) *TreeNode{
	lenStr := *loc
	//数组结束或遇到占位返回 nil 同时指针后移两位
	if string(data[*loc]) == PlaceHolder || *loc >= len(data){
		*loc+=2
		return  nil
	}
	//正常数据指针后移
	for  string(data[*loc]) != Delimiter  {
		*loc ++
	}
	val,_ := strconv.ParseInt(string(data[lenStr:*loc]),10,64)
	//取数后跨个分隔符
	*loc ++
	node := &TreeNode{
		val:int(val),
		left:deserilaizeDe(data,loc),
		right:deserilaizeDe(data,loc),
	}
	return node
}
func main(){
	root := TreeNode{
		val:322,
		left:nil,
		right:nil,
	}
	leftNode_leftNode := TreeNode{
		val:9,
		left:nil,
		right:nil,

	}
	leftNode := TreeNode{
		val:2,
		left:&leftNode_leftNode,
		right:nil,
	}

	rightNode:= TreeNode{
		val:5,
		left:nil,
		right:nil,
	}
	root.left = &leftNode
	root.right = &rightNode
	str := serialize(&root)
	fmt.Println(str)
	root1 := deserialize(str)
	fmt.Println(serialize(root1))
}
```