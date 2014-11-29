title: Communications between sandbox apps in Mac
date: 2014-11-28 00:44:04
categories: 开发
tags: [Mac, Sandbox]

---

由于 Mac 对 Sandbox 的种种限制，使得 Sandb app 间的通信方式与普通 non Sandbox app 间的通信方式有所不同，普通的通信方式，如 *`NSDistributedNotification`*、*`Distributed Objects`*、 *`XPC`* 等不能直接套用到 Sandbox 机制上。

Mac 引入了 *`Sandbox App Group`* 的机制，使得同属于一个 app group 的不同 app 间有了合适的通信方式。

<!--more-->

### Sandbox App Group 是什么

Sandbox App Group 表示由同一个 Developer ID 签名的 app group。每个 Developer ID 可以签名多个 Sandbox App Group，每个 Sandbox App Group 可以包含若干个 Sandbox app。

如果想要声明一个 Sandbox app 属于某个 Sandbox App Group，需要在该 app 的 *`.entitlements`* 中添加一个 *`com.apple.security.application-groups`* array，它的每个元素表示该 app 属于的某个 Sandbox app group，用 Group Bundle ID 表示。

**`Group Bundle ID`** := <Team-ID>.com.company.XXX，<Team-ID> 其实也可以不用，但 Apple 的标准用法是以 <Team-ID> 作为 Group Bundle ID 的 prefix。

每个 Sandbox App Group 会在 *`~/Library/Group Containers`* 下生成一个对应的目录，以 Group Bundle ID 命名。该目录的结构与 Sandbox app 的 Container 目录结构相同。该目录可以被属于该 group 的所有 Sandbox app 访问，没有限制。访问该目录的方法是调用 *`[[NSFileManager defaultManager] containerURLForSecurityApplicationGroupIdentifier:]`*。

### Sandbox App Group 能够做什么

借助 Sandbox App Group，有以下这么几种方式可以用来实现同 group 下的 app 间的通信。

#### 1. NSUserDefaults

对于单个 Sandbox app 来说，它的 *`NSUserDefaults`* 只能由自己访问，别的 Sandbox app 无法访问。而有了 Sandbox App Group，每个 group 可以拥有自己的 *`NSUserDefaults`*，而它是可以被下属的所有 Sandbox app 共享访问的。

* 创建 Group User Defaults

		NSUserDefaults *defaults = [[NSUserDefaults alloc] init];
	
		[defaults addSuiteNamed:<Group Bundle ID>];
		
* Get

		[defaults persistentDomainForName:];
		
* Set

		[defaults setPersistentDomain: forName:]
		
关于 *`NSUserDefaults`*，有一篇 [很 NB 的文章] [1] 可以学习。

**e.g.** [MyDiskCleaner] [2]

#### 2. Group Containers 目录

尽情利用吧。尽情读写吧。

比如用 *`FSEventStreamRef`* 来监控其下的某个目录，从而将该目录的变化通知到不同的 Sandbox app 。虽然这种通信方式有点原始~~

其实 *`NSUserDefaults`* 本质上就是存在于 Group Containers 里的 Preferences 目录中。

**e.g.** [MyDiskCleaner] [2]

#### 3. XPC

是的，有了 Sandbox App Group，Sandbox app 间可以使用 XPC 了。
但使用方式需要有所注意。

**使用条件**：

*	XPC Listener 必须是位于 Helper app 中，由主 app 通过 *`SMLoginItemSetEnabled`* 来启动该 Helper app，从而启动 XPC Listener。

*	Helper app 的 Bundle ID 必须以 Group Bundle ID 作为 prefix，即 **`Helper Bundle ID`** := <Group-Bundle-ID>.XXX

*	Helper app 在创建 Listener 时需要使用 *`[[NSXPCListener alloc] initWithMachServiceName:]`*，主 app 在创建 connection 时需要使用 *`[[NSXPCConnection alloc] initWithMachServiceName:options:0]`*。

*	XPC service name 必须等于 Helper Bundle ID。因为实际上是，在调用 *`SMLoginItemSetEnabled`* 启动Helper app 后，LaunchServices 自动为其注册了一个 XPC service，而 name 就是 Helper Bundle ID。

**e.g.** Apple Demo *"AppSandboxLoginItemXPCDemo"*

### 除了 Sandbox App Group，还有什么？

我们知道，在 iOS app 中，有 *`URL Scheme`* 这么个机制。如果某个 app 注册了某个特殊的 *`URL Scheme`*，那么其他 app 可以通过访问 URL 的形式来 launch 该 app。

iOS app 也是采用 Sandbox 机制的，那么在 Mac 上，该方法也是同样适用的！不管其他 app 是不是属于同一 Sandbox App Group，都可以用这种方式来 launch 该 app，并将访问的 URL 传递给该 app。而如果该 app 已经启动，那么它同样也会捕捉到被访问的 URL，从而做出响应。

* 让 app 注册 URL Scheme

	在本 app 的 Info.plsit 中，添加一个 *`URL types`* array，它的每个元素代表一个 *`URL Schemes`* array，而该 array 的每个元素就是一个 URL Scheme，通常取一个特殊的名字，用于区别其它 app 的 URL Schemes。
	
* 监听自己注册的 URL Scheme

	在本 app 的 *`applicationWillFinishLaunching:`* delegate 函数中，利用如下代码注册一个系统 event listener：
	
		[[NSAppleEventManager sharedAppleEventManager] setEventHandler:andSelector:forEventClass:andEventID:];
		
	callback 函数：
	
		-(void)getUrl:(NSAppleEventDescriptor *)event withReplyEvent:(NSAppleEventDescriptor*)reply
		{
    		NSURL *url = [NSURL URLWithString:[[event paramDescriptorForKeyword:keyDirectObject] stringValue]];
    		...
		}
		
	**注意**：必须要在 *`applicationWillFinishLaunching:`* 中注册，否则通过这种方式启动 app 后，它无法获取被访问的 URL 信息。
		
* 其它 app 访问该 URL Scheme

	在其它 app 中，构造 URL 并访问：
	
		[[NSWorkspace sharedWorkspace] openURL:[[NSURL alloc] initWithScheme:host:path:]];

**e.g.** [MyDiskCleaner] [2]


[1]: http://realmacsoftware.com/blog/shared-preferences-between-sandboxed-applications
[2]: https://github.com/wzqcongcong/MyDiskCleaner