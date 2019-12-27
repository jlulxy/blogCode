---
title: grpc通信分析-服务端
date: 2019-12-25 16:59:14
tags: grpc
---

今天要修改下框架方面的东西，把接口请求返回的默认上报，过滤掉外部拼凑的url，想得是把router的map传进去，未命中的就不做上报，看到grpc的时候发现其并没有像http一样维护一个grpc的方法map，对grpc的原理之前有一些了解，今天希望能更进一步详细，了解下grpc注册和发现相关的逻辑。

### grpc的服务端注册
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
3. srv的实现绑定和配置绑定ok后，启动srv.server()会把服务启动接受请求启动协程进行处理。我们看下具实现：我们来简单分析一下 下面是部分代码
 ```bash
 func (s *Server) Serve(lis net.Listener) error {
	s.mu.Lock()
	s.printf("serving")
	s.serve = true
    //server的lisMap如果是空就直接关闭lis
	if s.lis == nil {
		// Serve called after Stop or GracefulStop.
		s.mu.Unlock()
		lis.Close()
		return ErrServerStopped
	}

	s.serveWG.Add(1)
	defer func() {
		s.serveWG.Done()
		select {
		// Stop or GracefulStop called; block until done and return nil.
		case <-s.quit:
			<-s.done
		default:
		}
	}()

	ls := &listenSocket{Listener: lis}
	s.lis[ls] = true

	if channelz.IsOn() {
		ls.channelzID = channelz.RegisterListenSocket(ls, s.channelzID, lis.Addr().String())
	}
	s.mu.Unlock()

	defer func() {
		s.mu.Lock()
		if s.lis != nil && s.lis[ls] {
			ls.Close()
			delete(s.lis, ls)
		}
		s.mu.Unlock()
	}()

	var tempDelay time.Duration // how long to sleep on accept failure

	for {
        //接受连接
		rawConn, err := lis.Accept()
		tempDelay = 0
		// Start a new goroutine to deal with rawConn so we don't stall this Accept
		// loop goroutine.
		//
		// Make sure we account for the goroutine so GracefulStop doesn't nil out
		// s.conns before this conn can be added.
		s.serveWG.Add(1)
		go func() {
            //开启协程处理连接流
			s.handleRawConn(rawConn)
			s.serveWG.Done()
		}()
	}
}
 ```
4. 最后一步请求路由到相应的hanlder,例如	上面的：Handler:    _NewsPool_GetVideoContent_Handler (依然只截取部分代码)
```bash
    func (s *Server) handleStream(t transport.ServerTransport, stream *transport.Stream, trInfo *traceInfo) {
	//获取要调用的方法
    sm := stream.Method()
	if sm != "" && sm[0] == '/' {
		sm = sm[1:]
	}
	pos := strings.LastIndex(sm, "/")
    //省略了些错误处理
    ...
	service := sm[:pos]
	method := sm[pos+1:]

    //获取到回应的md，在processUnaryRPC中调用md.method 就是注册的_NewsPool_GetVideoContent_Handler
	if srv, ok := s.m[service]; ok {
		if md, ok := srv.md[method]; ok {
			s.processUnaryRPC(t, stream, srv, md, trInfo)
			return
		}
		if sd, ok := srv.sd[method]; ok {
			s.processStreamingRPC(t, stream, srv, sd, trInfo)
			return
		}
	}
	// Unknown service, or known server unknown method.
	if unknownDesc := s.opts.unknownStreamDesc; unknownDesc != nil {
		s.processStreamingRPC(t, stream, nil, unknownDesc, trInfo)
		return
	}
	if trInfo != nil {
		trInfo.tr.LazyLog(&fmtStringer{"Unknown service %v", []interface{}{service}}, true)
		trInfo.tr.SetError()
	}
	errDesc := fmt.Sprintf("unknown service %v", service)
	if err := t.WriteStatus(stream, status.New(codes.Unimplemented, errDesc)); err != nil {
		if trInfo != nil {
			trInfo.tr.LazyLog(&fmtStringer{"%v", []interface{}{err}}, true)
			trInfo.tr.SetError()
		}
		grpclog.Warningf("grpc: Server.handleStream failed to write status: %v", err)
	}
	if trInfo != nil {
		trInfo.tr.Finish()
	}
}

//进入到这个函数就可以看到，handler就是我们实现的getNewsContent函数，其实如果定义了interceptor会在interceptor中执行hander，做一些通一的操作。
func _NewsPool_GetNewsContent_Handler(srv interface{}, ctx context.Context, dec func(interface{}) error, interceptor grpc.UnaryServerInterceptor) (interface{}, error) {
	in := new(NewsPoolRequest)
	if err := dec(in); err != nil {
		return nil, err
	}
	if interceptor == nil {
		return srv.(NewsPoolServer).GetNewsContent(ctx, in)
	}
	info := &grpc.UnaryServerInfo{
		Server:     srv,
		FullMethod: "/content.NewsPool/GetNewsContent",
	}
	handler := func(ctx context.Context, req interface{}) (interface{}, error) {
		return srv.(NewsPoolServer).GetNewsContent(ctx, req.(*NewsPoolRequest))
	}
	return interceptor(ctx, in, info, handler)
}
```

grpc的服务注册，启动，监听和路由到对应的实现就完成了，下一篇看下grpclient的建立，怎样把一个grpc请求转化为具体的调用，这样就把grpc的实现完整的了解啦