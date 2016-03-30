title: XPC mechanism of Mac
date: 2014-11-28 00:44:50
categories: 开发
tags: [Mac]

---

在 Mac 中，进程间的通信机制，除了 *`NSDistributedNotification`*、*`Distributed Objects`*，还有 XPC。XPC 可以在同一个 app 的不同 bundle 间使用，也可以在不同的 app 间使用。

XPC 机制是通过 *`NSXPCConnection`* 作为 channel 来通信的，每个 *`NSXPCConnection`* 有 *Client* 和 *Listener* 两个 endpoint。

### 1. XPC 角色

#### Listener

通过 *`NSXPCListener`* API 来创建 Listener，并指定该 Listener 的 *`NSXPCListenerDelegate`*，该 delegate 需要实现的函数是 *`(BOOL)listener:shouldAcceptNewConnection:`*。该 delegate 函数用于决定是否响应新的 connection，并设置该 connection 的 *`exportedInterface`* 和 *`exportedObject`*。

* *`exportedObject`*：Listener 的导出对象，提供接口供 Client 访问。

* *`exportedInterface`*：列举出 *`exportedObject`* 能够导出哪些可用的接口供 Client 调用，函数原型通常是：*`(void)doTaskWithInfo:(NSDictionary *)info callback:(void(^)(NSDictionary *dic))callback`*。

#### Client

Client 通过 *`NSXPCConnection`* API 来创建一个新的 connection，并指定该 connection 的 *`remoteObjectInterface`*，它代表 Listener 端的 *`exportedInterface`*。然后就可以通过 *`remoteObjectProxyWithErrorHandler:`* 来获取 Listener 端的 *`exportedObject`*，调用其导出的接口。Listener 在执行自己的导出函数时，可以通过调用传递进来的 callback 向 Client 端 send back 数据。

#### NSXPCConnection

*`NSXPCConnection`* 的发起是单向的，只能由 Client 端到 Listener 端，因为每一个发起的 connection 必须要由 Listener 端通过 *`NSXPCListenerDelegate`* 函数来决定是否要进行响应。但 Client 和 Listener 都可以有自己的 *`exportedObject`* 和 *`remoteObjectProxy`*，在 connect 之后可以相互访问对方，一方的 *`exportedObject`* 对应另一方的 *`remoteObjectProxy`*。参见 Apple 文档中的图示。

**注意**：不管是 Client 还是 Listener，设置自己的 *`exportedObject`* 和 *`remoteObjectProxy`* 必须要先于调用 *`resume`*。


<!--more-->


### 2. XPC 搭建

* 同一个 app 中使用的 XPC 称为 XPC Service，在 Xcode 中可以创建 *XPC Service* 这种 target，生成的 bundle 是 .xpc。在主 app 中需要将 .xpc bundle deploy 到自己的 *`Contents/XPCServices`* 目录下，Mac 会在该目录中寻找相应的 .xpc bundle 去 load。

	**Listener**：在 *XPC Service* bundle 的代码中，需要使用 *`[NSXPCListener serviceListener]`* 来创建 Listener。执行 *`resume`* 之后永远不会 return。
	
	**Client**：在 Client 端，也就是主 app 中，需要使用 *`[[NSXPCConnection alloc]
     initWithServiceName:]`* 来创建 connection，为其传递 *XPC Service* 的 Bundle ID。
     
     **注意**：*XPC Service* 的启动或者退出完全由 OS 自己决定，你要做的就是启动主 app，并在代码中需要的地方请求 XPC connection。举个例子，.xpc bundle 可能是在主 app 发起 XPC 请求时才被 load 和启动 XPC Service，而不是早在之前就去启动。此外，*`Contents/XPCServices`* 目录下的 .xpc bundle 只能由所属的主 app 调用，其它 app 无法调用。
     
     **e.g.**
     
		Apple Demo *"SandboxingAndNSXPCConnection"*
	
		Apple Demo *"AppSandboxLoginItemXPCDemo"*
     
* 不同 app 间的 XPC 不是通过创建 .xpc bundle 来实现的，因为前面这种方式创建的 .xpc bundle 只能被自己所属的 app 调用。而是通过创建普通的 target 来充当 XPC 的Listener。

	**Listener**：在 Listener app 中，需要使用 *`[[NSXPCListener alloc] initWithMachServiceName:]`* 来创建 Listener，为其传递自行设定的 XPC service name。执行 *`resume`* 之后会立即 return，所以需要自行启动 Runloop。
	
	**Client**：在 Client app 中，需要使用 *`[[NSXPCConnection alloc] initWithMachServiceName:options:]`* 来创建 connection，为其传递 Listener 中指定的 XPC service name。option 用于说明你的 XPC service 是否是在 admin 权限下。
	
	**e.g.** [SMJobBlessXPC] [1] | [MyDiskCleaner] [2] | [AppleTunerUpdater] [3]


[1]: https://github.com/wzqcongcong/SMJobBlessXPC
[2]: https://github.com/wzqcongcong/MyDiskCleaner
[3]: https://github.com/wzqcongcong/AppleTunerUpdater