---
title: gRPC安全认证
toc: true
comments: true
date: 2017-07-11 10:17:56
tags:
	- gRPC
	- 安全认证
categories: gRPC
---

### 1 认证

gRPC支持以下认证方式

- SSL/TLS认证
- OAuth 2.0认证
- 自定义拓展认证


<!--more-->

### 2 SSL/TLS认证

#### 2.1 准备证书

- 制作私钥

  ```shell
  # Key considerations for algorithm "RSA" ≥ 2048-bit
  openssl genrsa -out server.key 2048
      
  # Key considerations for algorithm "ECDSA" ≥ secp384r1
  # List ECDSA the supported curves (openssl ecparam -list_curves)
  openssl ecparam -genkey -name secp384r1 -out server.key
  ```

- 制作公钥

  ```shell
  openssl req -new -x509 -sha256 -key server.key -out server.pem -days 3650

  Country Name (2 letter code) [AU]:CN
  State or Province Name (full name) [Some-State]:Beijing
  Locality Name (eg, city) []:Beijing
  Organization Name (eg, company) [Internet Widgits Pty Ltd]:nil
  Organizational Unit Name (eg, section) []:nil
  Common Name (e.g. server FQDN or YOUR name) []:Han
  Email Address []:nil
  ```

#### 2.2 服务端TLS认证

```go
lis, err := net.Listen("tcp", "127.0.0.1:12345")

//TLS认证
creds, err := credentials.NewServerTLSFromFile("../keys/server.pem", "../keys/server.key")

grpcServer := grpc.NewServer(grpc.Creds(creds))
```

#### 2.3 客户端TLS认证

```go
creds, err := credentials.NewClientTLSFromFile("../keys/server.pem", "Han") //Common Name

conn, err := grpc.Dial("127.0.0.1:12345", grpc.WithTransportCredentials(creds))
```



### 3 自定义拓展认证

除了TLS认证之外，gRPC还提供了自定义的认证方式，即Token认证。

#### 3.1 Token认证步骤

①客户端

- 定义一个customCredential结构，实现下面两个接口

  ```go
  func (c customCredential) GetRequestMetadata(ctx context.Context, uri ...string) (map[string]string, error)
  func (c customCredential) RequireTransportSecurity() bool
  ```

- client设置rpc连接选项

  ```go
  var opts []grpc.DialOption := grpc.WithPerRPCCredentials(new(customCredential))
  conn, err := grpc.Dial("127.0.0.1:12345", opts...)
  ```

通过上述两个步骤，客户端在调用服务端接口的时候，就会将Token信息通过请求的`metadata`发送到服务端（`GetRequestMetadata`中设置），服务端在请求的`metadata`中取出Token数据就可以进行校验了。

②服务端

服务端调用在处理逻辑之前先从请求的`metadata`中取出Token数据进行校验。其中`md`是`map[string][]string`类型，带有Token信息。

```go
md, ok := metadata.FromContext(ctx)
```

#### 3.2 客户端Token认证

```go
//实现gRPC自定义认证接口
type customCredential struct{}

func (c customCredential) GetRequestMetadata(ctx context.Context, uri ...string) (map[string]string, error) {
	return map[string]string{
		"appid":  "12345",
		"appkey": "owqp3Zrxtbdt5kVe",
	}, nil
}
func (c customCredential) RequireTransportSecurity() bool {
	return IsOpenTLS
}

//设置gRPC选项
if IsOpenTLS {
	creds, err := credentials.NewClientTLSFromFile("../keys/server.pem", "Han") //Common Name
	if err != nil {
		log.Fatalf("Failed to generate credentials: %v", err)
	}
	opts = append(opts, grpc.WithTransportCredentials(creds))
} else {
	opts = append(opts, grpc.WithInsecure())
}

opts = append(opts, grpc.WithPerRPCCredentials(new(customCredential)))

conn, err := grpc.Dial("127.0.0.1:12345", opts...)
```

#### 3.3 服务端校验Token

```go
func (s helloServer) SayHello(ctx context.Context, request *pb.HelloRequest) (*pb.HelloReply, error) {
	//校验token
	md, ok := metadata.FromContext(ctx)
	if !ok {
		return nil, grpc.Errorf(codes.Unauthenticated, "no token")
	}
	if md["appid"][0] != "12345" || md["appkey"][0] != "Xr2nveGW7RL0DTCl" {
		return nil, grpc.Errorf(codes.Unauthenticated, "Failed to check token!, appid = %q, appkey = %q",
			md["appid"][0], md["appkey"][0])
	}

	reply := &pb.HelloReply{
		Message: "Hello " + request.GetName(),
	}
	return reply, nil
}
```



### 4 gRPC拦截器

gRPC中，服务端接收到请求，可以通过使用拦截器interceptor优先对请求数据进行一些处理，再转交给指定的服务，比较适合处理验证，日志等流程。

使用拦截器时，服务端先声明一个interceptor结构来实现具体的处理逻辑：

```go
var interceptor grpc.UnaryServerInterceptor
interceptor = func(ctx context.Context, req interface{}, info *grpc.UnaryServerInfo,
	handler grpc.UnaryHandler) (resp interface{}, err error) {
	err = auth(ctx)	//具体处理逻辑
	if err != nil {
		return nil, err
	}
	return handler(ctx, req)
}
```

然后调用`grpc.UnaryInterceptor`来注册该拦截器

```go
opts = append(opts, grpc.UnaryInterceptor(interceptor))

grpcServer := grpc.NewServer(opts...)
```

