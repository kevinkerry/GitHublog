title: Study iOS - Misc
date: 2015-01-05 22:34:11
categories: 开发
tags: [iOS]

---

### Developing Applications for iOS 观后感系列：Misc

*相关视频章节：all*

---

#### NSObject

* `(NSString *)description`：当你自己实现一个 myClass 时，如果实现了 `(NSString *)description` 这个函数，那么你就可以利用 `NSLog(@"%@", myClassInstance)` 来输出 description 的内容了，这里的 *`%@`* 会被替换成 `[myClassInstance description]`。

* `(id)copy`：得到 object 的 unmutable 版，不管 object 本身是不是 mutable 的。

* `(id)mutableCopy`：得到 object 的 mutable 版，不管 object 本身是不是 mutable 的。

	**注意**，并不是所有的 class 都实现了 copy 和 mutableCopy 机制，如果没有实现，而你调用了，就会抛出异常。另外，这2种 copy 操作效率很高，所以不要有所担心。
	
#### UIFont

* 给文本内容（content）设定字体的最佳方式：`UIFont *font = [UIFont preferredFontForTextStyle:UIFontTextStyleXXX]`

* 给控件（例如 button 的 title）设定字体的最佳方式：`UIFont *font = [UIFont systemFontOfSize:(CGFloat)pointSize]`

	**注意**，preferredFontForTextStyle: 也可以用于设定控件字体，但反过来 systemFontOfSize: 却不能用于设定 content 字体。
	
	
<!--more-->

	
#### NSNotification

* 对于向 NSNotificationCenter 注册过的 observer，如果不再需要 listen 某个 notification，必须要调用 `removeObserver:`，因为 NSNotificationCenter 对 observer 的引用方式是 *`unsafe retained`*，所以，如果 observer 变为 nil 了，NSNotificationCenter 会出现引用异常，造成 crash。

	常见的调用 removeObserver: 的地方是 *`viewWillDisappear`* 或者 *`dealloc`*。
	
#### Category

* 添加 Category 无需事先得到 class 的 .m 文件，即无需知道 class 的原始实现。

* `Category 不能添加 instance 变量，只能添加 property 和 function。`而 property 本质上也是 function，既然不能添加 instance 变量，*`那也意味着，@synthesize 也是不允许的。`*总结起来就是，Category 中添加的 property 和 function 只能对原有的 instance 变量进行操作。Extension 可以添加 instance 变量。

#### Network Activity Indicator

* UIApplication 有个属性：*`BOOL networkActivityIndicatorVisible`*，将其设为 YES，status bar 上就会出现表示网络活动的 spinner，设为 NO，spinner 就会消失。使用时需要注意，当多个线程操作该 networkActivityIndicatorVisible 时，需要给它加个**计数的机制**，防止错误地开关该属性。

