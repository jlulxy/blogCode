---
title: leetcode-146LRUCache
date: 2020-01-05 12:34:03
tags: [LRU,Cache,leetcode]
---

### LRU Cahce 思路演进

#### LRU第一版本思想
**头尾位独立处理实现复杂容易出错。**
采用map实现o(1)获取
使用双向链表控制LRU，一直有个用例不过
golang 实现如下：测试用例 12 / 18 个通过测试用例.
经过debug后发现为题判断如果是头指针和移除节点一致时处理除了问题
```bash
type LRUCache struct {
    dataMap map[int]*ListDataNode
    head *ListDataNode
    tail *ListDataNode
    capacity int
    use int
}
type ListDataNode struct{
    data  int
    key   int
    pre *ListDataNode
    next *ListDataNode
}


func Constructor(capacity int) LRUCache {
    Cache :=LRUCache{
        dataMap :make(map[int]*ListDataNode,capacity),
        head :nil,
        tail :nil,
        capacity: capacity,
        use:0,
    }
    return Cache
}
//取出操作的节点
func (this *LRUCache) TakeOutNode( node *ListDataNode){
    if node.pre!=nil{
        node.pre.next = node.next
    }
    if node.next !=nil{
        node.next.pre = node.pre
    }
    //node 是尾节点，尾节点要前移动
    if node == this.tail{
        this.tail = node.pre
    }
    /*
    //node是头节点，头节点滞空
    if node == this.head{
        this.head = nil
    }
    */
    上面是错误的写法这里更正下,移除节点是头，并不意味着链表中没有头节点啦，所以这里不应应该直接指空
    if node == this.head{
        this.head = node.next
    }
    //养成好习惯不用的指针处理掉
    node.pre = nil
    node.next = nil
}
func (this *LRUCache) Get(key int) int {
    if node ,ok := this.dataMap[key];ok{
        this.TakeOutNode(node)
        this.AddHead(node)
        return node.data
    }
    return -1    
}

//把刚操作的放在最前面
func (this *LRUCache) Put(key int, value int)  {
    //检查是否存在，存在的直接更新值同时把节点移到head
    if  node ,ok := this.dataMap[key];ok{
		node.data = value
        node.key = key
        this.TakeOutNode(node)
		this.AddHead(node)
		return 
	}
    if this.use >= this.capacity{
        //队列满了先去掉队列尾巴把再插入
        this.DeleteTailNode()
    }
    node := &ListDataNode{
        data:value,
        key:key,
    }
    this.dataMap[key] = node
    this.AddHead(node)
    this.use++
}
//把节点插入头里面
func (this *LRUCache) AddHead(node *ListDataNode){
        if this.head == nil{
            this.head = node
            this.tail = node
        }else{
            //把该节点放入队列头
            tmp :=this.head
            this.head = node
            node.next = tmp
            tmp.pre = node          
        }
}

//删除尾部节点
func (this *LRUCache)DeleteTailNode(){
    //获取尾节点
    if this.tail == nil{
        return 
    }
    key := this.tail.key
    if _,ok :=this.dataMap[key];ok{
        this.dataMap[key] = nil
        delete(this.dataMap,key)
        //从尾部节点拿掉
        tmp := this.tail
        this.tail = tmp.pre
        if tmp.pre !=nil{
            tmp.pre.next = nil
        }
        this.use--
    }
}

```

#### LRU第二版本思想 
**独立申请头尾哨兵节点，数据节点不在可能是头或尾部节点，插入和移除不用考虑过多情况实现简洁，不过在空间很小的情况下会多占两个节点可能空间会多O(2),但是带来的简洁性更高**

代码如下：
```bash
type LRUCache struct {
	dataMap map[int]*ListDataNode
	head *ListDataNode
	tail *ListDataNode
	capacity int
}
type ListDataNode struct{
	data  int
	key   int
	pre *ListDataNode
	next *ListDataNode
}


func Constructor(capacity int) LRUCache {
	head ,tail := new(ListDataNode),new(ListDataNode)
    // 初始化cache 头尾相连作为哨兵，从头插入，从尾部移除
	head.next = tail
	tail.pre = head
	head.next = tail
	tail.pre = head
	Cache :=LRUCache{
		dataMap :make(map[int]*ListDataNode,capacity),
		head :head,
		tail :tail,
		capacity: capacity,
	}
	return Cache
}

// 移除节点
func (this *LRUCache) remove( node *ListDataNode){
	node.pre.next = node.next
	node.next.pre = node.pre
    node.next = nil
    node.pre = nil
}

// 插入头后面进入
func (this *LRUCache) insertHead (node *ListDataNode){
	head := this.head
	node.next = head.next
	node.pre = head
	head.next.pre = node
	head.next = node
}

func (this *LRUCache) Get(key int) int {
	if node ,ok := this.dataMap[key];ok{
		this.remove(node)
		this.insertHead(node)
		return node.data
	}
	return -1
}

//把刚操作的放在最前面
func (this *LRUCache) Put(key int, value int)  {
	if  node ,ok := this.dataMap[key];ok{
		node.data = value
		this.remove(node)
		this.insertHead(node)
		return
	}
	if len(this.dataMap) >= this.capacity{
		//队列满了先去掉队列尾巴把再插入
		node := this.tail.pre
		this.remove(node)
		delete(this.dataMap,node.key)
	}
	node := &ListDataNode{
		data:value,
		key:key,
	}
	this.dataMap[key] = node
	this.insertHead(node)
}
```