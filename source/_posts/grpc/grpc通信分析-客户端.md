---
title: grpc通信分析-客户端
date: 2019-12-27 17:42:29
tags: grpc
---

其实在了解了server端的实现后，client基本上就可以想到大致流程，下面我们还是看下源码，熟悉下具体的实现

#### grpc客户端的实现
1. 再没有其它框架封装的情况下，调用createGRPCClientList（建立一个grpcClient）与server不一样他不需要绑定太多信息，最主要的就是address，client要知道去哪里通信。
2. 拿到clientConn只有调用pb.go里面的newsClient把client放入pb.go定义号的client里面,这样就可以通过改函数返回的client调用实现的具体方法。
```bash
type NewsPoolClient interface {
	GetNewsContent(ctx context.Context, in *NewsPoolRequest, opts ...grpc.CallOption) (*NewsPoolReply, error)
	UpdateNewsContent(ctx context.Context, in *NewsPoolRequest, opts ...grpc.CallOption) (*UpdateReply, error)
	DelNewsContent(ctx context.Context, in *NewsPoolRequest, opts ...grpc.CallOption) (*DelReply, error)
	// 获取视频类文章列表
	GetVideoContent(ctx context.Context, in *NewsPoolRequest, opts ...grpc.CallOption) (*NewsPoolReply, error)
}

type newsPoolClient struct {
	cc *grpc.ClientConn
}

func NewNewsPoolClient(cc *grpc.ClientConn) NewsPoolClient {
	return &newsPoolClient{cc}
}

//这里client的具体方法实现,可以看到主要的逻辑是grpc.Ivoke,同时method是按照server注册的同样接口传入method
func (c *newsPoolClient) GetNewsContent(ctx context.Context, in *NewsPoolRequest, opts ...grpc.CallOption) (*NewsPoolReply, error) {
	out := new(NewsPoolReply)
	err := grpc.Invoke(ctx, "/content.NewsPool/GetNewsContent", in, out, c.cc, opts...)
	if err != nil {
		return nil, err
	}
	return out, nil
}
```
3. 通过invoker函数发请求，给服务端同时把获得的数据访日out中返回，我们看下Invoke的具体实现
```bash
//grpc.Invoke 里面封装了clent.cc的invoke就是我们最开始createGRPCClientList生成的clentconnect
func (cc *ClientConn) Invoke(ctx context.Context, method string, args, reply interface{}, opts ...CallOption) error {
	// allow interceptor to see all applicable call options, which means those
	// configured as defaults from dial option as well as per-call options
	opts = combine(cc.dopts.callOptions, opts)

	if cc.dopts.unaryInt != nil {
		return cc.dopts.unaryInt(ctx, method, args, reply, cc, invoke, opts...)
	}
	return invoke(ctx, method, args, reply, cc, opts...)
}
```

#### 小节
通过前后两个文章我们大致了解grpc c/s大致是如何通信的，后续有时间或有精力，会继续分析一些底层网络的实现。包括invoke的具体细节，消息发送和接受使用的具体函数和方法