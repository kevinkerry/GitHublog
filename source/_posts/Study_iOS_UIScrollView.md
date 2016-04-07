title: Study iOS - UIScrollView
date: 2015-01-11 13:46:10
categories: 开发
tags: [iOS]

---

### Developing Applications for iOS 观后感系列：UIScrollView

*相关视频章节：10*

<!--more-->

---

**`UIScrollView`** 是 iOS 中很基础很重要的一种 UIView，用于呈现远大于 app window 大小的 content area，用户通过 *`swiping`* 手势可以在 content area 中进行 scroll，通过 *`pinching`* 手势可以进行 content 的 zoom。就像是拿着一个放大镜在四处移动着观察一副很大的地图。

UIScrollView 也是很多其它重要 UIView 的 super class，例如 [*`UITableView`*] [1]、*`UICollectionView`*、[*`UITextView`*] [2] 等等。

#### 创建 UIScrollView

通过 Storyboard 或者代码中通过 alloc initWithFrame。

#### 使用 UIScrollView

* 向 UIScrollView 的 content area 中添加一个 subview1：

![UIScrollView](/img/Study_iOS_UIScrollView/10.3.UIScrollView.png)

* 再向 UIScrollView 的 content area 中添加一个很大的 subview2：

	![UIScrollView](/img/Study_iOS_UIScrollView/10.4.UIScrollView.png)

	**注意**，千万不要忘记设置 UIScrollView 的 *`contentSize`* property！它指定了 UIScrollView 的 content area 的大小，也就是 UIScrollView 可以游走的范围。所有添加到 UIScrollView 中的 subviews 的 frame 都是位于 UIScrollView 的 content area 坐标系中，即：原点为 (0,0)，size 为 contentSize。

	通常，contentSize 的设定需要考虑 content area 里面容纳的 subviews 的大小。
	
* **当前 visible area**

	![UIScrollView](/img/Study_iOS_UIScrollView/10.6.UIScrollView.png)
	
* **当前 visible area 在 content area 坐标系中的位置**

	![UIScrollView](/img/Study_iOS_UIScrollView/10.5.UIScrollView.png)
	
* **当前 visible area 在原 subview 坐标系中的大小**

	![UIScrollView](/img/Study_iOS_UIScrollView/10.7.UIScrollView.png)
	
* **Zoom**

	所有 UIView 都有一个 property：*`CGAffineTransform transform`*，包含 *`translate`*，*`scale`*，*`rotate`*。而 UIScrollView 实现 Zoom 的原理就是改变 subview 的 transform 的值。
	
	**注意**，Zoom 时也会影响 *`contentSize`* 和 *`contentOffset`* 的值。
	
	**想要支持 Zoom 效果，必须做到下面2点**：
	
	1. 设定 UIScrollView 的 *`minimumZoomScale`* 和 *`maximumZoomScale`*。不设定的话，默认都是 1.0，即不缩放。

	2. 实现 delegate 函数：`-(UIView *)viewForZoomingInScrollView:(UIScrollView *)sender;`，该函数用于指定你要缩放的是 UIScrollView 中的哪个 subview。

	通常缩放都是伴随着 pinching 手势的发生而触发的，但你也可以在代码中手动触发缩放。主要是借助这2个 API：`-(void)setZoomScale: animated:` 和 `-(void)zoomToRect: animated:`。
	
[1]: ../../../../2015/01/11/Study_iOS_UITableView/
[2]: ../../../../2015/01/11/Study_iOS_Common_UI/