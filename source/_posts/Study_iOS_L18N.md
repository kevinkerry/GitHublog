title: Study iOS - L18N
date: 2015-01-18 13:35:11
categories: 开发
tags: [iOS]

---

### Developing Applications for iOS 观后感系列：L18N

*相关视频章节：18*

---

L18N 主要涉及2方面：Xcode + code。

* **Xcode**

	首先需要在 Xcode 中设置 app 支持的语言：

	![L18N](/img/Study_iOS_L18N/18.1.L18N.png)

	然后你就可以对 storyboard 进行 L18N了，Xcode 会生成相应语言的 .strings 文件。

* **code**

	对于不在 storyboard 中的 strings 如何 L18N 呢？那就得通过 code 来处理了。

	![L18N](/img/Study_iOS_L18N/18.2.L18N.png)

	在代码中调用上述 macros 之后，可以通过下面命令自动生成 .strings 文件：

	![L18N](/img/Study_iOS_L18N/18.3.L18N.png)

	**注意**，当你从 *`Bundle`* 中获取某个文件的 path 时，其搜索顺序是：先 bundle 的 top-level 层，后 .lproj 层。
	

<!--more-->


### Locale

提到 L18N，就不得不提 Locale，因为它是整个 L18N 得以实现的基础。此外，Locale 也决定了 date 和 number 的格式。

**注意**，Locale 和 language 不是等价的，language 一样的时候，Locale 不一定一样。

* **`NSLocale`**：iOS 中主要靠该 class 来获取 Locale 相关的信息，该 class 知道如何设定不同格式的 date 和 number。也就是说，对 date 和 number 的设定主要就是给他们传递对应的 NSLocale 参数。

* **`NSNumberFormatter`**

	![L18N](/img/Study_iOS_L18N/18.6.L18N.NSNumberFormatter.png)
	
* **`NSDateFormatter`**

	![L18N](/img/Study_iOS_L18N/18.7.L18N.NSDateFormatter.png)
		
* **`NSString`**

	![L18N](/img/Study_iOS_L18N/18.8.L18N.NSString.png)
	
* **`UIImage`**

	当你调用 UIImage 的 `imageNamed:*` 时，它也是会去搜索相应的 .lproj 目录的。