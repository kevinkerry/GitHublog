title: Study iOS - Life Cycle
date: 2015-01-05 20:38:32
categories: 开发
tags: [iOS]

---

### Developing Applications for iOS 观后感系列：Life Cycle

*相关视频章节：5，17*

---

这里提到的 Life Cycle 主要包括2方面：

* Application 的 Life Cycle

* UIViewController 的 Life Cycle

<!--more-->

### Application 的 Life Cycle

Application 的 Life Cycle 是靠一些 *`UIApplicationDelegate`* 函数来控制的。

新建一个 iOS 项目的时候，在 Xcode 生成的 AppDelegate.m 中，已经列出了常用的几个 delegate 函数：

* `(BOOL)application:(UIApplication *)application willFinishLaunchingWithOptions:(NSDictionary *)launchOptions`：app 即将完成启动。诸如响应 openURL 这样的事情，需要在这里做，在 app 彻底启动完毕之前，从 launchOptions 中拿到 URL。

* `(BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions`：app 彻底启动完毕。这里一般做 app 的初始化操作。

* `(void)applicationDidBecomeActive:(UIApplication *)application`：app 变得 active，也就是说，app 的 UI 可以接收 events 了。这通常是发生在 app 启动后出现 UI 的时候，或者从其它 app 切换而来的时候。伴随该 delegate 有一个 notification *`UIApplicationDidBecomeActiveNotification`*。

* `(void)applicationWillResignActive:(UIApplication *)application`：app 变得 not active，也就是说，app 的 UI 无法接收 events 了。这通常是发生在切换到其它 app 的时候，或者双击 Home 的时候，或者接听电话的时候。总之就是，该 app 的 UI 已经不再是 iOS 的 first responder UI 了。伴随该 delegate 有一个 notification *`UIApplicationWillResignActiveNotification`*。

	通常利用以上2个 delegate，来暂停并保存当前 UI 状态，然后在之后重新恢复 UI 状态。

* `(void)applicationDidEnterBackground:(UIApplication *)application`：app 进入 background。**注意**，当发生 applicationWillResignActive 后，并不一定会发生 applicationDidEnterBackground，例如，运行该 app 时突然双击 Home 或者接听电话，此时，只会发生applicationWillResignActive。只有等到该 app 进入 background 后，例如已经切换到其它 app 后，才会发生 applicationDidEnterBackground。**此外**，iOS 留给该 delegate 的运行时间很短，如果想在这里做些费时的操作，可以借助 *`beginBackgroundTaskWithExpirationHandler:`*。伴随该 delegate 有一个 notification *`UIApplicationDidEnterBackgroundNotification`*。

* `(void)applicationWillEnterForeground:(UIApplication *)application`：app 即将重新回到 foreground。通常做些 undo applicationDidEnterBackground 的操作。applicationWillEnterForeground 发生之后，很快就会发生 applicationDidBecomeActive。**但注意**，app 启动的时候，并没有 applicationWillEnterForeground，而是直接 applicationDidBecomeActive。所以要理解前面提到的“重新”二字，app 启动时并不是重新，只有发生 applicationDidEnterBackground 后，才对应地有“重新”的意思。伴随该 delegate 有一个 notification *`UIApplicationWillEnterForegroundNotification`*。

* `(void)application:(UIApplication *)application performFetchWithCompletionHandler:(void (^)(UIBackgroundFetchResult result))completionHandler`：如果 app 开启了 fetch background mode，那么当 iOS 给你机会去做 fetch 时，就会调用该 delegate。在这里你可以做任何操作，并不限于 download fetch。但这里只有30s时间，30s内你必须处理完，然后调用 completionHandler。通常的做法是，首先开线程异步去做该做的操作，然后就立即调用 completionHandler。

	**注意**，如果 iOS 给你机会在这里做 network fetch，那么你应该在这里发起一个普通的 *`ephemeral configuration 类型的 NSURLSession`*，而不是发起 background configuration 类型的 NSURLSession。因为background configuration 类型的 NSURLSession 是 discretionary 的，当 app 已经处于 background 时，这种 NSURLSession 可能会被 iOS 拒绝执行，而这显然不是我们想要的，我们既然已经在 background 了，那么 iOS 给机会执行该函数时，我们想要的是发起一个有效的 NSURLSession，保证它会被执行。当在这里发起 ephemeral configuration 类型的 NSURLSession 后，我们就需要在 NSURLSession 的 completion delegate 中调用 *`completionHandler:UIBackgroundFetchResultNewData`* 来更新 app 的 UI snapshot。

* `(void)application:(UIApplication *)application handleEventsForBackgroundURLSession:(NSString *)identifier completionHandler:(void (^)())completionHandler`：如果之前发起了一个 background configuration 类型的 NSURLSession，那么当 NSURLSession task 完成后，就会调用该 delegate。**注意**，如果发起 background configuration 类型的 NSURLSession，那么必须要实现该 delegate，在其中尽早地调用 completionHandler，只有这样，app 才能调用你之前为 NSURLSession 设定的那些 NSURLSessionDelegate，诸如 didReceiveResponse 等等；否则，那些 NSURLSessionDelegate 不会被调用。

* `LocalNotification / RemoteNotification 系列`：处理诸如闹钟（local）、Apple Push Service（remote）这样的 Notification。

* `State Restoration 系列`：用于保存并恢复 app 的 UI，即使退出 app 后 再重启，也可以恢复到之前的状态。

* `Data Protection 系列`：锁屏后，将文件保护起来。

最后图示一下：

![ApplicationLifeCycle](/img/Study_iOS_Life_Cycle/17.0.ApplicationLifeCycle.png)


### UIViewController 的 Life Cycle

UIViewController 的 Life Cycle 主要靠一些类似于 delegate 的函数来控制的。

按照执行时间顺序列举几个常用的函数：

* `init`：UIViewController 的初始化有2种，一种是从 storyboard 中释放出来，另一种是在代码中直接调用相关的 init 函数。两种只会选择一种来完成 init。

	1. **来自 storyboard：**`awakeFromNib`
	
		当 app 启动时，所有从 storyboard 释放出来的 object 都会调用该函数，包括其中的 UIViewController（通过在 storyboard 中设置其 class）！**注意**，该函数执行时，outlet 还尚未设置好，因此无法访问。
	
	2. **来自 code：**`initWithNibName:(NSString *)name bundle:(NSBundle *)bundle`
	
	因为有这2种 init 方式，所以，如果希望在 init 阶段做一些 setup，那么这2个地方都要执行 setup。因为从 storyboard 释放出来的 init 方式只会调用 awakeFromNib，不会调用 initWithNibName；同样，code 中调用 initWithNibName: 进行的 init 方式也不会再调用 awakeFromNib。
	
	通常在该 init 阶段能做的事很有限，因为 outlet 尚未设置好。所有直接访问 outlet 的操作都无法进行。
	
* `iOS 设置 outlet`

* `viewDidLoad`：app 运行到这里时，outlet 已经设置好，所以是进行 setup 操作的绝佳之地！但要**注意**：

	1. 这里总是需要调用一下 *`[super viewDidLoad]`*。
	2. 此时 view 的 geometry 尚未设置好，也就是说像 *`bounds 和 frame`* 这些属性是还没有确定的，不能在这里访问，所以不要做些与 geometry 相关的操作。
	3. viewDidLoad 只会调用一次，即在释放之前，view 只会 load 一次。
	
* `iOS 确定 geometry`

* `viewWillLayoutSubviews: / viewDidLayoutSubviews:`：geometry 确定好后，就可以进行 view 的布局了。这通常都是由 AutoLayout 自动完成的，准确的说，**AutoLayout 动作就是发生在 will 与 did 之间**。每当 view 的 frame 发生改变时，就会调用这组函数，这样该 view 的 subviews 就能够重新进行布局了。例如屏幕发生 autorotation 时，就会调用它们。你可以在这里重新设置 subviews 的 frame 或其它 geometry 相关的属性，然后让 AutoLayout 来完成布局。当然，你也可以不管这组函数，只是让 AutoLayout 来做事就行了。

		顺便提一下，如果想让 ViewController 响应手机的 rotation，必须同时满足以下3个条件：
			shouldAutorotate 要返回 YES
			supportedInterfaceOrientations 要返回相应的 orientation
			Info.plist 中要设置允许相应的 orientation
		
* `viewWillAppear: / viewDidAppear:`：Layout 完成后，就可以让 view 现身了。不像 viewDidLoad，这组函数是可以随着 view 的 visible 状态改变而执行多次的，正如其函数名称表示的意思。

* `在 view visible 期间，每当 geometry 改变就会调用viewWill/DidLayoutSubviews:这组函数。`

* `viewWillDisappear: / viewDidDisappear:`这组函数也会随着 view 的 visible 状态改变而执行多次。

	**注意**，在 viewWill/DidXXX 这4个函数里面总是需要调用 super 的对应函数。在 viewWillXXX 这2个函数里面不要做太费时的操作，因为时间有限，will 这个状态马上就会过去的。
