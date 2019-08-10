---
title: nsq源码阅读(nsqlookupd)
date: 2018-06-12 11:36:59
tags:
- 源码阅读
- nsqlookupd
categories:
- nsq学习
---

![美图](http://p3euxxfa8.bkt.clouddn.com/2018-06-12-15-52-47.png)

从官网文档来看, nsqlookupd是一个用来管理集群的守护进程. 客户端通过查询nsqlookupd来发现对应特定主题的nsqd, 同时nsqd也会广播自己的topic和 channel 信息.

当nsq集群中有多个nsqlookupd服务时，因为每个nsqd都会向所有的nsqlookupd上报本地信息，因此nsqlookupd具有最终一致性

## 基本结构

在nsq的目录中, apps目录下对应的文件夹中既有各个组件的main函数. 这里nsqlookupd的main及其相关的struct如下:

```go
//Mutex为互斥锁
type Mutex struct {
	state int32
	sema  uint32
}

//RWMutex（读写锁）
type RWMutex struct {
	w           Mutex  // held if there are pending writers
	writerSem   uint32 // semaphore(信号) for writers to wait for completing readers
	readerSem   uint32 // semaphore for readers to wait for completing writers
	readerCount int32  // number of pending readers
	readerWait  int32  // number of departing readers
}


type NSQLookupd struct {
	sync.RWMutex
	opts         *Options
	tcpListener  net.Listener
	httpListener net.Listener
	waitGroup    util.WaitGroupWrapper
	DB           *RegistrationDB
}

type program struct {
	nsqlookupd *nsqlookupd.NSQLookupd
}

func main() {
	prg := &program{} //实例化对象, 返回的是实例化对象的指针
	if err := svc.Run(prg, syscall.SIGINT, syscall.SIGTERM); err != nil {
		log.Fatal(err)
	}
}
```

这里正常实例化一个program对象后, 通过go-svc提供服务的运行. 这里`svc.Run`的函数原型为

`func Run(service Service, sig ...os.Signal) error`

即接收一个service 以及一系列的信号, 返回错误值. 若无错误, 将会调用 `ws.run()` 方法, 从而实现服务的运行.

这里的service服务实例应该实现了init, start以及stop方法. 在源码中依次实现.

## service基本的方法
### init方法

```go
func (p *program) Init(env svc.Environment) error {
	if env.IsWindowsService() {
		dir := filepath.Dir(os.Args[0])
		return os.Chdir(dir)
	}
	return nil
}
```

默认的windwos服务的启动目录为系统的Windows32目录, 个人服务不适合在这个位置启动, 所以切换目录到编译生成的可执行文件所在的目录.

### start方法
```go
func (p *program) Start() error {
	opts := nsqlookupd.NewOptions()

	flagSet := nsqlookupdFlagSet(opts)
	flagSet.Parse(os.Args[1:])

	if flagSet.Lookup("version").Value.(flag.Getter).Get().(bool) {
		fmt.Println(version.String("nsqlookupd"))
		os.Exit(0)
	} // 如果是查看版本信息, 显示版本对应信息后退出

	var cfg map[string]interface{}
	configFile := flagSet.Lookup("config").Value.String()
	if configFile != "" {
		_, err := toml.DecodeFile(configFile, &cfg)
		if err != nil {
			log.Fatalf("ERROR: failed to load config file %s - %s", configFile, err.Error())
		}
	} // 从配置文件加载配置信息

	options.Resolve(opts, flagSet, cfg)
	daemon := nsqlookupd.New(opts) 

	daemon.Main() // 调用nsqlookupd的Main函数
	p.nsqlookupd = daemon
	return nil
}
```
当start函数返回后, 整个程序阻塞在svc.Run方法内部的信号channel上.

#### nsqlookupd.Main()
```go
func (l *NSQLookupd) Main() {
    ctx := &Context{l} 
    /*
    type Context struct {
        nsqlookupd *NSQLookupd
    } 
    */

	tcpListener, err := net.Listen("tcp", l.opts.TCPAddress)
	if err != nil {
		l.logf(LOG_FATAL, "listen (%s) failed - %s", l.opts.TCPAddress, err)
		os.Exit(1)
	}
	httpListener, err := net.Listen("tcp", l.opts.HTTPAddress)
	if err != nil {
		l.logf(LOG_FATAL, "listen (%s) failed - %s", l.opts.HTTPAddress, err)
		os.Exit(1)
	}

	l.tcpListener = tcpListener
	l.httpListener = httpListener

	tcpServer := &tcpServer{ctx: ctx}
	l.waitGroup.Wrap(func() {
		protocol.TCPServer(tcpListener, tcpServer, l.logf)
	})
	httpServer := newHTTPServer(ctx)
	l.waitGroup.Wrap(func() {
		http_api.Serve(httpListener, httpServer, "HTTP", l.logf)
	})
}
```

l.waitGroup是sync.WaitGroup的子类，该类的Wrap方法用于在新的goroutine调用参数func函数；因此在执行Main方法之后，此时nsqlookupd进程就另外开启了两个goroutine，一个用于执行tcp服务，一个用于执行http服务

#### nsqlookupd的tcp服务
nsqlookupd的tcp服务用于处理nsqd的上报信息. 其tcp实现的方法为
```go
[internal/protocol/tcp_server.go]

type TCPHandler interface {
	Handle(net.Conn)
}

func TCPServer(listener net.Listener, handler TCPHandler, logf lg.AppLogFunc) {
	logf(lg.INFO, "TCP: listening on %s", listener.Addr())

	for {
		clientConn, err := listener.Accept()
		if err != nil {
			if nerr, ok := err.(net.Error); ok && nerr.Temporary() {
				logf(lg.WARN, "temporary Accept() failure - %s", err)
				runtime.Gosched()
				continue
			}
			// theres no direct way to detect this error because it is not exposed
			if !strings.Contains(err.Error(), "use of closed network connection") {
				logf(lg.ERROR, "listener.Accept() - %s", err)
			}
			break
		}
		go handler.Handle(clientConn)
	}

	logf(lg.INFO, "TCP: closing %s", listener.Addr())
}

```

这个函数和一般的go的网络函数一样, 在一个for循环中, 开启一个新的goroutine用来处理客户端的连接.

##### TCPHandler.Handle()
```go
[tcp.go]

type tcpServer struct {
	ctx *Context
}

func (p *tcpServer) Handle(clientConn net.Conn) {
	p.ctx.nsqlookupd.logf(LOG_INFO, "TCP: new client(%s)", clientConn.RemoteAddr())

	// The client should initialize itself by sending a 4 byte sequence indicating
	// the version of the protocol that it intends to communicate, this will allow us
	// to gracefully upgrade the protocol away from text/line oriented to whatever...
	buf := make([]byte, 4)
	_, err := io.ReadFull(clientConn, buf)
	if err != nil {
		p.ctx.nsqlookupd.logf(LOG_ERROR, "failed to read protocol version - %s", err)
		clientConn.Close()
		return
	}
	protocolMagic := string(buf)

	p.ctx.nsqlookupd.logf(LOG_INFO, "CLIENT(%s): desired protocol magic '%s'",
		clientConn.RemoteAddr(), protocolMagic)

	var prot protocol.Protocol
	switch protocolMagic {
	case "  V1":
		prot = &LookupProtocolV1{ctx: p.ctx}
	default:
		protocol.SendResponse(clientConn, []byte("E_BAD_PROTOCOL"))
		clientConn.Close()
		p.ctx.nsqlookupd.logf(LOG_ERROR, "client(%s) bad protocol magic '%s'",
			clientConn.RemoteAddr(), protocolMagic)
		return
	}

	err = prot.IOLoop(clientConn)
	if err != nil {
		p.ctx.nsqlookupd.logf(LOG_ERROR, "client(%s) - %s", clientConn.RemoteAddr(), err)
		return
	}
}
```

tcpserver的只有一个成员ctx, 即这个nsqlookupd实例的地址.  当协议为V1时, 调用LookupProtocolV1.IOLoop(). 
##### LookupProtocolV1.IOLoop()

```go
func (p *LookupProtocolV1) IOLoop(conn net.Conn) error {
	var err error
	var line string

	client := NewClientV1(conn)
	reader := bufio.NewReader(client)
	for {
		line, err = reader.ReadString('\n')
		if err != nil {
			break
		}

		line = strings.TrimSpace(line)
		params := strings.Split(line, " ")

		var response []byte
		response, err = p.Exec(client, reader, params)
		if err != nil {
			ctx := ""
			if parentErr := err.(protocol.ChildErr).Parent(); parentErr != nil {
				ctx = " - " + parentErr.Error()
			}
			p.ctx.nsqlookupd.logf(LOG_ERROR, "[%s] - %s%s", client, err, ctx)

			_, sendErr := protocol.SendResponse(client, []byte(err.Error()))
			if sendErr != nil {
				p.ctx.nsqlookupd.logf(LOG_ERROR, "[%s] - %s%s", client, sendErr, ctx)
				break
			}

			// errors of type FatalClientErr should forceably close the connection
			if _, ok := err.(*protocol.FatalClientErr); ok {
				break
			}
			continue
		}

		if response != nil {
			_, err = protocol.SendResponse(client, response)
			if err != nil {
				break
			}
		}
	}

	conn.Close()
	p.ctx.nsqlookupd.logf(LOG_INFO, "CLIENT(%s): closing", client)
	if client.peerInfo != nil {
		registrations := p.ctx.nsqlookupd.DB.LookupRegistrations(client.peerInfo.id)
		for _, r := range registrations {
			if removed, _ := p.ctx.nsqlookupd.DB.RemoveProducer(r, client.peerInfo.id); removed {
				p.ctx.nsqlookupd.logf(LOG_INFO, "DB: client(%s) UNREGISTER category:%s key:%s subkey:%s",
					client, r.Category, r.Key, r.SubKey)
			}
		}
	}
	return err
}
```

在这个IOLoop中, 执行的流程就是不断读取用户请求, 将用户请求字符串按空格切割成字符串数组, 然后在调用Exec方法去执行具体的请求.

##### LookupProtocolV1.Exec
```go
func (p *LookupProtocolV1) Exec(client *ClientV1, reader *bufio.Reader, params []string) ([]byte, error) {
	switch params[0] {
	case "PING":
		return p.PING(client, params) //nsqd每隔一段时间都会向nsqlookupd发送心跳，表明自己还活着
	case "IDENTIFY":
		return p.IDENTIFY(client, reader, params[1:]) // 当nsqd第一次连接nsqlookupd时，发送IDENTITY，验证身份
	case "REGISTER":
		return p.REGISTER(client, reader, params[1:]) // 当nsqd创建一个topic或者channel时，向nsqlookupd发送REGISTER请求，在nsqlookupd上更新当前nsqd的topic或者channel信息
	case "UNREGISTER":
		return p.UNREGISTER(client, reader, params[1:]) // 当nsqd删除一个topic或者channel, 向nsqlookupd发送UNREGISTER请求，在nsqlookupd上更新当前nsqd的topic或者channel信息
	}
	return nil, protocol.NewFatalClientErr(nil, "E_INVALID", fmt.Sprintf("invalid command %s", params[0]))
}
```

#### http服务

nsqlookupd的http服务是用于向nsqadmin提供查询接口的，本质上，就是一个web服务器，提供http查询接口

```go
func Serve(listener net.Listener, handler http.Handler, proto string, logf lg.AppLogFunc) {
	logf(lg.INFO, "%s: listening on %s", proto, listener.Addr())

	server := &http.Server{
		Handler:  handler,
		ErrorLog: log.New(logWriter{logf}, "", 0),
	}
	err := server.Serve(listener) //实例化http.Server模块，然后调用server.Server(listner)函数开启http服务
	// theres no direct way to detect this error because it is not exposed
	if err != nil && !strings.Contains(err.Error(), "use of closed network connection") {
		logf(lg.ERROR, "http.Serve() - %s", err)
	}

	logf(lg.INFO, "%s: closing %s", proto, listener.Addr())
}
```

















