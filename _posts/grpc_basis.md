---
title: gRPC框架基础
toc: true
comments: true
date: 2017-07-11 09:47:07
tags: gRPC
categories: gRPC
---



### 1 概述

[GRPC](https://github.com/grpc/grpc)是google开源的一个高性能、跨语言的RPC框架，基于HTTP2协议和[protobuf 3](https://github.com/google/protobuf)。开发步骤分为四步：

- 定义proto接口文件
- 使用protoc工具生成指定语言代码
- 启动Server端
- 启动多个Client端


gRPC资源：

- [官方文档](http://grpc.mydoc.io)
- [GO-gRPC](https://segmentfault.com/a/1190000007880647)


<!--more-->



### 2 gRPC proto

gRPC使用protobuf来声明数据模型和RPC接口服务，按照的不同，可以分为四种类型的RPC：

- 简单RPC：这是最简单的RPC类型，服务端从客户端拿到一个数据对象，然后返回另一个数据对象。
- 服务端流式RPC：服务端从一个stream来**写多个**响应，客户端通过这个stream来读取数据。
- 客户端流式：服务端从客户端获取一个stream，然后通过它来**读多个**数据，并可以通过它来**写单个**响应。
- 双向流式RPC：服务端从客户端获取一个stream，然后通过它来**读多个**数据，并可以通过它来**写多个**响应。

具体形式如下：

```protobuf
//声明服务
service Service {
  //简单RPC
  rpc NormalRPC(RequestArgs) returns (ReturnArgs) {}
  
  //服务端流式RPC
  rpc ServerStreamRPC(RequestAgrs) returns (stream ReturnStream) {}
  
  //客户端流式RPC
  rpc ClientStreamRPC(stream RequestStream) returns (ReturnArgs) {}
  
  //双向流式RPC
  rpc TwoWayStreamRPC(stream TWStream) returns (TWStream) {}
}
```



### 3 四种gRPC类型

#### 3.1 简单RPC

```go
func (s *ServiceServer) NormalRPC(ctx context.Context, reqAgs *RequestArgs) (*ReturnAgrs, error) {
  retAgrs := &ReturnArgs{}
  //...
  return retAgrs, nil
}
```

#### 3.2 服务端流式RPC

```go
//proto 生成
type ServiceServer_ServerStreamRPCServer interface {
	Send(*ReturnStream) error
	grpc.ServerStream
}
type serviceServerServerStreamRPCServer struct {
	grpc.ServerStream
}

//实现interface
func (x *serviceServerServerStreamRPCServer) Send(m *ReturnStream) error {
	return x.ServerStream.SendMsg(m)
}

//实现业务逻辑
func (s *ServiceServer) ServerStreamRPC(reqArgs *RequestArgs, stream pb.serviceServerServerStreamRPCServer) error {
	for {
      	retStream := &ReturnStream{}
      	//...
    	if err := stream.Send(retStream); err != nil {
          return err
    	}
	}
  	return nil
}
```

#### 3.3 客户端流式RPC

```go
//proto生成
type ServiceServer_ClientStreamRPCServer interface {
	SendAndClose(*ReturnArgs) error
	Recv() (*RequestStream, error)
	grpc.ServerStream
}
type serviceServerClientStreamRPCServer struct {
	grpc.ServerStream
}

//实现interface
func (x *serviceServerClientStreamRPCServer) SendAndClose(m *ReturnArgs) error {
	return x.ServerStream.SendMsg(m)
}
func (x *serviceServerClientStreamRPCServer) Recv() (*RequestStream, error) {
	m := new(RequestStream)
	if err := x.ServerStream.RecvMsg(m); err != nil {
		return nil, err
	}
	return m, nil
}

//实现业务逻辑
func (s *ServiceServer) ClientStreamRPC(stream pb.serviceServerClientStreamRPCServer) error {
	for {
      	reqStream, err := stream.Recv()
      	if err == io.EOF {
          	retArgs := &ReturnArgs{}
          	//...
          	return stream.SendAndClose(retArgs)
      	}
      	//...
	}
  	return nil
}
```

#### 3.4 双向流式RPC

```go
//proto生成
type ServiceServer_TwoWayStreamRPCServer interface {
	Send(*TWStream) error
	Recv() (*TWStream, error)
	grpc.ServerStream
}
type serviceServerTwoWayStreamRPCServer struct {
	grpc.ServerStream
}

//实现interface
func (x *serviceServerTwoWayStreamRPCServer) Send(m *TWStream) error {
	return x.ServerStream.SendMsg(m)
}
func (x *serviceServerTwoWayStreamRPCServer) Recv() (*TWStream, error) {
  	m := new(TWStream)
  	if err := x.ServerStream.RecvMsg(m); err != nil {
    	return nil, err
  	}
  	return m, nil
}

//实现业务逻辑
func (s *ServiceServer) TwoWayStreamRPC(stream pb.serviceServerTwoWayStreamRPCServer) error {
	for {
      	in, err := stream.Recv()
      	if err == io.EOF {
          	return nil
      	}
      	if err != nil {
          	return err
      	}
      
     	for {
        	twStream := &TWStream{}
          	//...
          	if err := stream.Send(twStream); err != nil {
              	return err
          	}
     	}
	}
  	return nil
}
```



### 4 启动服务

#### 4.1 启动服务端

- 指定监听端口
- `grpc.NewServer()`创建一个gPRC Server实例
- 注册自定义服务
- `Server()`阻塞等待连接

```go
func main() {
	flag.Parse()
	lis, err := net.Listen("tcp", fmt.Sprintf(":%d", *port))
	if err != nil {
		grpclog.Fatalf("failed to listen: %v", err)
	}
	var opts []grpc.ServerOption
	
    // set options
  	
	grpcServer := grpc.NewServer(opts...)
	pb.RegisterServiceServer(grpcServer, newServer())
	grpcServer.Serve(lis)
}
```

#### 4. 2 启动客户端

```go
func main() {
	flag.Parse()
  	//set options
  	conn, err := grpc.Dial("serverAddr", opts...)
  	if err != nil {
      	//handle error
  	}
  	defer conn.Close()
  
  	client := pb.NewServiceClient(conn)
}

//调用四种服务
```

