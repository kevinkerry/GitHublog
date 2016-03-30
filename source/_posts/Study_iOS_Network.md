title: Study iOS - Network
date: 2015-01-07 10:15:27
categories: 开发
tags: [iOS]

---

### Developing Applications for iOS 观后感系列：Network

*相关视频章节：10*

---

iOS 中处理 Network 的 API 主要是 *`NSURLSession`*，当然，基于此，*Mattt Thompson* 开发了一套更加强大易用的 *AFNetworking*。不管怎样，还是很有必要了解一下 iOS 自己的 *NSURLSession* 的。

#### *NSURLSession* 的一般使用流程：

	NSURLRequest *request = [NSURLRequest requestWithURL:XXX];
	NSURLSessionConfiguration *configuration = XXX;
	NSURLSession *session = XXX; // use configuration
	NSURLSessionXXXTask *task = [session XXXTaskWithRequest:request]; // downloadTask, uploadTask, dataTask, ...
	[task resume];
	
基本上就是上面这几步。当然，如果使用 *`NSURLSessionDelegate `* 的话，还需要实现这些 delegate；如果不使用 delegate，也可以选择使用 *`completionHandler`* 这种 block 的方式。具体选择哪种，取决于 *NSURLSessionXXXTask* 是如何创建的，下面会详细提到。

下面分别理解一下这些具体步骤：

1. **`NSURLRequest`**：比较简单，不多说了。

2. **`NSURLSessionConfiguration`**：这个 configuration 控制着后面创建的 session 的若干属性。通常比较关注这3个 API：

	* `+(NSURLSessionConfiguration *)defaultSessionConfiguration`：创建一个 default session，会使用 cache，cookie 机制。
	
	* `+(NSURLSessionConfiguration *) ephemeralSessionConfiguration`：创建一个 ephemeral session，不使用 cache，cookie 机制，所有东西都在内存中，除了下载文件时才会真正地有写入 disk 的动作。
	
	* `+(NSURLSessionConfiguration *) backgroundSessionConfigurationWithIdentifier:(NSString *)identifier`：创建一个 background session，可以让 session 在 app 处于 background 时能够继续 run。

		这种 session 会把控制权交给系统，正如之前一篇笔记（[*`Life Cycle`*] [1]）中讲到的那样，有个 `UIApplicationDelegate` 是专门配合这种 session 使用的，即 `(void)application:(UIApplication *)application handleEventsForBackgroundURLSession:(NSString *)identifier completionHandler:(void (^)())completionHandler`，系统首先通过这个 delegate 响应 network events 后，才会再把控制权交给你的 *`NSURLSessionDelegate`*。
		
		**注意**，这种 session 比前面2种多了一个 `identifier`，有了它，当 app 被系统 terminate 和 relaunch 后，系统会使用同一个 identifier 重建 background session，继续之前未完成的 network transfer。但是，这里仅限于 app 是被系统 terminate 和 relaunch，如果是用户手动退出 app，则不会去重建 session。
		

<!--more-->

	
3. **`NSURLSession`**：有了 configuration，就可以创建 session 了。session 的 init 方式决定了 network events 的 handle 方式，以及 handle 操作的执行环境是哪里。主要有2种 init 方式：

	* `+(NSURLSession *)sessionWithConfiguration:(NSURLSessionConfiguration *)configuration delegate:(id<NSURLSessionDelegate>)delegate delegateQueue:(NSOperationQueue *)queue`：
	
		**delegate** 若不为 nil，就意味着你实现了自己的 *`NSURLSessionDelegate`* 来 handle network events。若为 nil，就意味着该 session 只接受 *`completionHandler`* 这种 handle 操作。
		
		**queue** 用于指定 handle 操作（不管是 NSURLSessionDelegate 还是 completionHandler）是在什么 NSOperationQueue 上执行的。可以传递 [NSOperationQueue mainQueue]；也可以传递 **[[NSOperationQueue alloc] init]（根据文档，这样 init 出来的 NSOperationQueue 总是并行执行其中的 operation 的**）；还可以传递 **nil，此时，session 会自动创建一个串行 queue（非 mainQueue）**来执行所有的 NSURLSessionDelegate 或者 completionHandler。
	
	* `+(NSURLSession *)sessionWithConfiguration:(NSURLSessionConfiguration *)configuration`：前一种的简化版，只接受 *`completionHandler`* 这种 handle 操作，同时，**session 会自动创建一个串行的 NSOperationQueue（非 mainQueue）来执行 completionHandler**。

	**注意**，session 的 delegate 和 delegateQueue 属性是只读的，无法事后修改。

4. **`NSURLSessionXXXTask `**：这里不提具体的各种 task 是什么样的，只关注下 session 创建 task 的方式。主要有2类：

	* `XXXTaskWithRequest:completionHandler:`：指定 completionHandler 来 handle network events。你可以指定它为 NULL，但前提是在之前创建 session 时已经指定好了 NSURLSessionDelegate，这样就会通过 delegate 来 handle，否则，既没有 completionHandler 又没有 delegate，你就无法 handle 了。

	* `XXXTaskWithRequest:`	：只通过 NSURLSessionDelegate 来 handle。

	**注意**，XXXTask 本身执行时并不在 mainQueue 上，所以不必担心，你所关注的只是 NSURLSessionDelegate 或者 completionHandler 是在哪个 queue 上执行。
	
5. **不要忘了 `[task resume];`**


[1]: ../../../../2015/01/05/Study_iOS_Life_Cycle/