title: Study iOS - UIView
date: 2015-01-06 21:23:11
categories: 开发
tags: [iOS, UIView]

---

### Developing Applications for iOS 观后感系列：UIView

*相关视频章节：7*

---

<!--more-->

### UIView 的 init

与 UIViewController 的 init 类似，UIView 的 init 也有2种形式：一种是从 storyboard 中释放出来，另一种是在代码中直接调用相关的 init 函数。两种只会选择一种来完成 init。

1. **来自 storyboard：** *`awakeFromNib`*
	
	awakeFromNib 是 NSObject 的函数，颤抖吧。
	
2. **来自 code：** *`initWithFrame:(CGRect)rect`*
	
因为有这2种 init 方式，所以，如果希望在 init 阶段做一些 setup，那么这2个地方都要执行 setup。因为从 storyboard 释放出来的 init 方式只会调用 awakeFromNib，不会调用 initWithFrame:；同样，code 中调用 initWithFrame: 进行的 init 方式也不会再调用 awakeFromNib。

### Coordinate

![Coordinate](/img/Study_iOS_UIView/7.1.Coordinate.png)

![Coordinate](/img/Study_iOS_UIView/7.2.Coordinate.png)

### Draw

基于 CPU 进行绘制时，靠的是 *`drawRect:`* 这个函数。如何实现自己的 *`drawRect:`* 呢？两种方式：

* **Core Graphics （C API）**

	1. 获取 draw 操作所在的 *`context`*。**注意**，每次调用 *`drawRect:`* 时，iOS 都会重新给你提供一个新的 context，只有这个最新的对本次 *`drawRect:`* 才是有效的，因此不要去 cache 之前的某个 context 来复用，因为它对本次的 *`drawRect:`* 是无效的。
	
		context 决定了 *`drawRect:`* 操作发生在哪里（Screen，Offscreen，PDF，Printer 等）。通过在 *`drawRect:`* 中调用 *`UIGraphicsGetCurrentContext()`* 来获取当前 CGContextRef。
	
	2. 创建 path
	
	3. 设置 color、font、width 等等属性
	
	4. 调用 stroke 或者 fill 进行绘制

* **UIBezierPath（Cocoa API）**

	利用 UIBezierPath 会得到一个 object，然后直接通过该 object 来 stroke 或者 fill。UIBezierPath 是不需要设置 context 的，它会自动地在当前 context 下进行绘制。
	
	可以利用 UIBezierPath 来 clip 你的后续绘制操作：
	
		UIBezierPath *roundedRect = [UIBezierPath bezierPathWithRoundedRect:rect cornerRadius:radius];
		[roundedRect addClip]; // this would clip all drawing to be inside the roundedRect
		
基于 GPU 的 draw 用的是另一套机制：Layer。本篇未涉及。
		
### Redraw

![Redraw](/img/Study_iOS_UIView/7.6.Redraw.png)

### UIGestureRecognizer

两种添加方式：

1. `code 中添加`

	![UIGestureRecognizer](/img/Study_iOS_UIView/7.7.UIGestureRecognizer.png)

	![UIGestureRecognizer](/img/Study_iOS_UIView/7.8.UIGestureRecognizer.png)

	![UIGestureRecognizer](/img/Study_iOS_UIView/7.9.UIGestureRecognizer.png)

2. `storyboard 中拖放 UIGestureRecognizer 控件，并设置 target-action。`