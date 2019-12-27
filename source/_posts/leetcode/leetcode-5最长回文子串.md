---
title: leetcode-5最长回文子串
date: 2019-12-22 20:09:58
tags: [leetcode,golang]
---

### 最长回文子串

```bash
给定一个字符串 s，找到 s 中最长的回文子串。你可以假设 s 的最大长度为 1000。

示例 1：

输入: "babad"
输出: "bab"
注意: "aba" 也是一个有效答案。
示例 2：

输入: "cbbd"
输出: "bb"
```

### leetcode官方分析
首先排除暴力列举法，用中心扩展法：
事实上，只需使用恒定的空间，我们就可以在 O(n^2) 的时间内解决这个问题。

我们观察到回文中心的两侧互为镜像。因此，回文可以从它的中心展开，并且只有 2n - 12n−1 个这样的中心。

你可能会问，为什么会是 2n - 12n−1 个，而不是 nn 个中心？原因在于所含字母数为偶数的回文的中心可以处于两字母之间（例如 \textrm{“abba”}“abba” 的中心在两个 \textrm{‘b’}‘b’ 之间）。


### 自己的分析和实现误区
一开始直观的思想就是遍历 i->n,以i为中心，分别判断其作为奇回文串和偶回文串的情况，取最大值，记录最大值和起始i，实际写代码的时候由于有两种情况，写的代码十分繁杂，最后返回的时候还要根据长度取是否是奇偶，来判断返回的s，结果就是返回字符串起始计算很懵逼，最后不通过。

### 走出思维误区
做一个函数，传入起始位置，起始位置相等，往两边扩张（注意边界条件0和len），最后返回right - left -1 (因为符合条件多减了2)
如果返回的长度大于之前记录的更新，起始长度。
golang代码如下

```bash
func max(a,b int) int{
    if a>b {
        return a
    }
    return b
}

func longestPalindrome(s string) string {
    if s == ""{
        return ""
    }
    start ,end := 0,0
    sLen := len(s) 
    for i:=0;i<sLen;i++{
        len1 := expanCenter(s,i,i)
        len2 := expanCenter(s,i,i+1)
        len := max(len1,len2)
        if len > end - start{
            start = i - (len - 1)/2
            end = i + len/2;
        }
    }
    return s[start:end + 1]
}

func  expanCenter(s string,left,right int) int {
    L,R:=left,right
    for ;L >= 0 && R<len(s) && s[L] == s[R];{
        L--;
        R++;
    }
    return R - L -1
}

```