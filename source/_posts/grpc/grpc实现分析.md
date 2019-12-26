---
title: grpc通信分析
date: 2019-12-25 16:59:14
tags: grpc
---

今天要修改下框架方面的东西，把接口请求返回的默认上报，过滤掉外部拼凑的url，想得是把router的map传进去，未命中的就不做上报，看到grpc的时候发现其并没有像http一样维护一个grpc的方法map，对grpc的原理之前有一些了解，今天希望能更进一步详细，了解下grpc注册和发现相关的逻辑。

### grpc
#### grpc的服务端注册
1. 首先生成srv grpcserver（google的官方sever）
2. 通过pb.go生成的Register**Server，绑定实现了pb.go定义的接口的服务类
```shell
//绑定实现类到 生成的server里，同时注册serviceDesc到
func RegisterNewsPoolServer(s *grpc.Server, srv NewsPoolServer) {
	s.RegisterService(&_NewsPool_serviceDesc, srv)
}
//serviceDesc 如下也是服务端收到请求能够打到具体方法的核心配置，相当于httpserver里面的路由信息
var _NewsPool_serviceDesc = grpc.ServiceDesc{
	ServiceName: "content.NewsPool",
	HandlerType: (*NewsPoolServer)(nil),
	Methods: []grpc.MethodDesc{
		{
			MethodName: "GetNewsContent",
			Handler:    _NewsPool_GetNewsContent_Handler,
		},
		{
			MethodName: "UpdateNewsContent",
			Handler:    _NewsPool_UpdateNewsContent_Handler,
		},
		{
			MethodName: "DelNewsContent",
			Handler:    _NewsPool_DelNewsContent_Handler,
		},
		{
			MethodName: "GetVideoContent",
			Handler:    _NewsPool_GetVideoContent_Handler,
		},
	},
	Streams:  []grpc.StreamDesc{},
	Metadata: "newspool.proto",
}
//其中一个handler定义如下
func _NewsPool_GetNewsContent_Handler(srv interface{}, ctx context.Context, dec func(interface{}) error, interceptor grpc.UnaryServerInterceptor) (interface{}, error) {
	in := new(NewsPoolRequest)
	if err := dec(in); err != nil {
		return nil, err
	}
    //interceptor 直接断言绑定的srv结构体调用其函数
	if interceptor == nil {
		return srv.(NewsPoolServer).GetNewsContent(ctx, in)
	}
	info := &grpc.UnaryServerInfo{
		Server:     srv,
		FullMethod: "/content.NewsPool/GetNewsContent",
	}
    //interceptor 把具体srv处理函数封装成handler传入interceptor中，可以在interceptor中做请求的预处理和其它操作，我们框架里面的熔断限流和一些统一操作也是在这里进行处理的
	handler := func(ctx context.Context, req interface{}) (interface{}, error) {
		return srv.(NewsPoolServer).GetNewsContent(ctx, req.(*NewsPoolRequest))
	}
	return interceptor(ctx, in, info, handler)
}
```
3. srv的实现绑定和配置绑定ok后，启动srv.server()会把服务启动接受请求并，并把请求路由到响应的hanlder
