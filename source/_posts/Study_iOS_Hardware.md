title: Study iOS - Hardware
date: 2015-01-18 15:42:25
categories: 开发
tags: [iOS, Hardware]

---

### Developing Applications for iOS 观后感系列：Hardware

*相关视频章节：14、17*

---

关于 iOS device 的 hardware，本篇主要了解2个方面：CoreLocation + CoreMotion

### CoreLocation

* **`CLLocation`**：CoreLocation 中最基本的 class。

	CLLocation 的重要 property：
	
	*`coordinate`*：位置坐标，包含 latitude 和 longitude 两个元素。
	
	*`altitude`*：海拔。
	
	*`horizontal/verticalAccuracy`*：精度。Cellular < WiFi < GPS。
	
* **`CLLocationManager`**：用于获取 CLLocation。

	使用流程：
	
	1. *`check device`*

		主要是检查 device 是否支持以某种方式来获取某种 location，以及是否有权限来获取。
		
		关于权限，有3中状态：Authorized、Denied、Restricted。
		
	2. *`init & delegate`*：通过 CLLocationManager 来获取 CLLocation 可以有2种方式，一种是主动 ask（poll），一种是 delegate。通常采用 delegate 的方式，delegate 触发后，你就可以访问 CLLocationManager 的 property *`location`* 来获取当前的 CLLocation 了。

	3. *`configuration`*

		通过 property *`desiredAccuracy`* 来设定想要的精度。
		
		通过 property *`distanceFilter`* 告诉 CLLocationManager，只有当 location 的变化超过该范围时，才去触发 update delegate。

	4. *`start monitor`*

		你可以进行如下几种形式的 monitor，每种 monitor 都有相应的 delegate 函数，参见文档。
		
		* *`normal monitor`*：standard monitor，通过设定精度，每当 update 触发时，就会通知 app。

		* *`significant change monitor`*：仅当 location 发生明显变化时才通知 app，这里的 significant 并未定义具体是多少。

		* *`region monitor`*：当 device 进入某个指定的 region 时，通知 app。

		* *`beacon monitor`*：当 device 进入某个指定的 beacon 时，通知 app，这种 monitor 通常与 *`Core Bluetooth`* 相关。

	**注意**，当你不再需要 monitor 时，不要忘了 stop monitor。

	**注意**，即使 app not running，这些 monitor 也还是能够 work 的。想要在 background 模式下继续 monitor，需要在 project 中设置一下，类似于 background fetch 的设置。当 delegate 触发后，app 就会被启动。这种情况下，`application:didFinishLaunchingWithOption:` 的 option dictionary 中就会包含一个 key `UIApplicationLaunchOptionsLocationKey`。
	

<!--more-->


### CoreMotion

主要用于获取各种 motion sensor 的信息，如 accelerometer、gyro、magnetometer 等等。

iOS 中提供的 class 是 *`CMMotionManager`*，你可以通过 alloc/init 来获取。**但是注意**，处于性能考虑，每个 app 中只能有一个全局的 CMMotionManager instance，因此你需要创建一个单例模式来实现全局单例。

使用流程：

1. *`check device`*：检查 device 是否支持获取某种 motion data。

	这里需要提一下，除了 accelerometer、gyro、magnetometer 之外，iOS 还提供了一个 *`CMDeviceMotion`*，对前面3种 data 进行了综合封装，可以通过它一次性获取这3种 data。
	
	![CoreMotion](/img/Study_iOS_Hardware/17.9.CoreMotion.png)
	
	![CoreMotion](/img/Study_iOS_Hardware/17.10.CoreMotion.png)

2. *`start & get motion data`*：有两种获取 motion data 的方式：
	
	* 主动 ask（poll）：*`start/stopXXXUpdates`*
	
	* 通过设置 rate，注册 callback，被动接收。

		![CoreMotion](/img/Study_iOS_Hardware/17.11.CoreMotion.png)
		
		![CoreMotion](/img/Study_iOS_Hardware/17.12.CoreMotion.png)
		
		![CoreMotion](/img/Study_iOS_Hardware/17.13.CoreMotion.png)

