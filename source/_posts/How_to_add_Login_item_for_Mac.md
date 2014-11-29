title: How to add Login item for Mac
date: 2014-11-27 22:35:05
categories: 开发
tags: [Mac, Objective-C]

---

在 Mac OS 中，有许多机制可以用于添加 Login item，不同的 Login item 类型需要用不同的方式来添加，每种方式都有自己的工作流程，应用范围，以及最终的表现效果。

从开发者的角度来看，可以分成两大类，**一类**是在代码中通过调用 Mac 的 API 来添加，由 Mac 来自动管理 Login item；**另一类**是开发者直接拿 File System 开刀，手动控制。

<!--more-->

## 类型 1

### 1. Service Management framework

* **API**: **`SMLoginItemSetEnabled`**

* **How**: main app 的 *`Contents/Library/LoginItems`* 目录下包含一个 helper app，该 helper app 将作为 Login item 由 Mac 启动。

	也就是说，main app 的 Build Phases 中需要添加一个 Copy Files 阶段，将 helper app deploy 到自己的上述目录下，然后，在 main app 的代码中，通过调用 *`SMLoginItemSetEnabled:YES`* 将 helper app 启动，同时设置成 Login item。同样，禁用该 helper app 也是由 main app 在代码中控制，调用 *`SMLoginItemSetEnabled:NO`* 来实现，将会杀掉 helper app，并取消 Login item。
	
	main app 会传递给 *`SMLoginItemSetEnabled`* 需要启动的 helper app 的 Bundle ID，然后该 API 就会在 *`Contents/Library/LoginItems`* 中寻找该 helper app，执行相应的动作。

	**注意**：helper app 的 `Info.plist` 中需要设置 *`LSUIElement`* 或者 *`LSBackgroundOnly`* 。

* 该方法添加的 Login item 对用户是不可见的，在 System Preference 中也看不到。

* **e.g.** [MyDiskCleaner] [1]

### 2. SMJobBless

* **API**: **`AuthorizationCreate` `SMJobBless`**

* **How**: 同样也是一个 main app 和一个 helper app，helper app 一般是 Command Line Tool 类型的 target。helper app deploy 到 main app 的 *`Contents/Library/LaunchServices`* 目录下，由 main app 将其注册到 Mac 的 *`LaunchDaemon`* 中，实现自启动。

	main app 代码中首先需要调用 *`AuthorizationCreate`* 来获取 admin 权限，然后将获取到的权限连同 helper app 的 Bundle ID 一起传给 *`SMJobBless`*，*`SMJobBless`* 会将该 helper app 复制到 Mac 的 *`/Library/PrivilegedHelperTools`* 目录下，同时在 *`/Library/LaunchDaemons`* 目录下生成一个相应的 plist 文件，Mac 启动时，由 *`launchctl`* load 该 plist，从而 load helper app。
	
	**注意**：helper app 这个 target 需要一些特殊处理。首先，需要为其新建一个 *`<Bundle ID>-Launchd.plist`*，该 plist 中会写明如何被 *`launchctl`* load，然后，在 Build Settings 中需要设置 *`Other Linker Flags`* 的值，在其中写入 helper app 的 Info.plist 和上面的 Launchd.plist。这样，在编译出的 target 可执行文件中，会有一段特殊的代码区记录着它的 Info.plist 和 Launchd.plist 信息。*`SMJobBless`* 调用后在 *`/Library/LaunchDaemons`* 目录生成的 plist 文件就是直接从 target 可执行文件中读取到的。
	
	**注意**：*`SMJobBless`* 执行时，如果 *`/Library/PrivilegedHelperTools`* 目录下已经有了相同 Bundle ID 的 helper app，那么除非其 *`CFBundleShortVersion`* 不同，否则是不会重新覆盖的。所以当 helper app 代码发生改变时，为了希望能 deploy 新的 helper app，需要同时更新其 *`CFBundleShortVersion`*，这样 *`SMJobBless`* 才会将新的有效的 helper app deploy 到 *`/Library/PrivilegedHelperTools`* 目录下，并将其 load 起来。

* 该方法添加的 Login item 对用户是可见的，可执行文件和相应的 plist 文件的位置如上所述。用户可以手动执行 *`launchctl`* 来 load/unload。该方法不仅添加了 Login item，而且还实现了提权操作，也就是说，helper app 将会以 admin 权限来运行。

* **e.g.** [AppleTunerUpdater] [2]

### 3. OS Launch Services

* **API**: **`LSSharedFileList` 系列**

* **How**: 貌似过时了？文档中没有？StackOverflow 中有相关帖子介绍[【1】] [3] [【2】] [4]

* 该方法添加的 Login item 对用户是可见的，在 System Preference 的 Users & Groups 中可以查看到相应的 Login item，用户可以选择启用或者禁用。


## 类型 2

### 4. LaunchAgents / LaunchDaemons 目录

编写需要的 plist 文件，写明如何被 *`launchctl`* load，然后将 plist 放置到 *`LaunchAgents / LaunchDaemons`* 目录。

* ~/Library/LaunchAgents	（user 权限，只对当前 user 有效）
* /Library/LaunchAgents		（root 权限，对所有 user 有效）
* /Library/LaunchDaemons	（root 权限，系统级别）

---

	对于 non Sandbox app 来说，上述4种方法都是可行的。
	对于 Sandbox app 来说，方法1适用，方法4写入 *`~/Library/LaunchAgents`* 也适用，其它方法不适用。


[1]: https://github.com/wzqcongcong/MyDiskCleaner
[2]: https://github.com/wzqcongcong/AppleTunerUpdater
[3]: http://stackoverflow.com/questions/5449135/how-can-a-cocoa-application-add-itself-as-a-global-login-item
[4]: http://stackoverflow.com/questions/14889956/launch-cocoa-application-for-all-users-during-login
