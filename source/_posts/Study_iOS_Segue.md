title: Study iOS - Segue
date: 2015-01-10 23:01:35
categories: 开发
tags: [iOS, UI]

---

### Developing Applications for iOS 观后感系列：Segue

*相关视频章节：6，15，16*

---

*`Segue`*，中文为“继续”的意思，在 iOS 开发中，其 class 为 *`UIStoryboardSegue`*，用于 storyboard 中不同 UIViewController 之间的转场，即从一个 UIViewController scene 转场到另一个 UIViewController scene。你可以从一个 UIControl、UIBarItem、UIView、UIViewController 等等，向另一个 UIViewController 建立 Segue。

### 如何创建 Segue

起点（UIControl、UIBarItem、UIView、UIViewController 等等）

⇩

*`Control`* + *`Drag`*

⇩

终点（UIViewController）

⇩

设置 Segue 的 Identifier

### Segue 如何执行

1. **触发 Segue**：假如你从 UIButton 建立了一个 Segue，那么 button 点击时，就会触发 Segue。但有时你也需要从代码中手动触发 Segue，此时就需要调用 UIViewController 的 API：`-(void)performSegueWithIdentifier:(NSString *)segueID sender:(id)sender`。

2. **询问 Segue 是否允许执行**：实现这个 delegate：`-(BOOL)shouldPerformSegueWithIdentifier:(NSString *)identifier sender:(id)sender`，如果不实现，默认就是允许所有 Segue。

3. **准备 Segue**：真正呈现 *`destination UIViewController`* 之前，你需要为该 Segue 作 preparation。也就是说，Segue 会给 *`source UIViewController`* 机会去准备好将要呈现的 destination UIViewController。可以理解成 destination UIViewController 的 init。这一 preparation 过程是通过实现如下 delegate 来完成的：

		-(void)prepareForSegue:(UIStoryboardSegue *)segue sender:(id)sender
		{
			if ([segue.identifier isEqualToString:@"XXX"]) {
				if ([segue.destinationViewController isKindOfClass:[XXXViewController class]]) {
					XXXViewController *xxxVC = (XXXViewController *)segue.destinationViewController;
					xxxVC.xxxProperty = XXX; // setup xxxVC
				}
			}
		}

	**注意**，要记住之前 [MVCs] [1] 中提到的原则，这个 destination UIViewController 在 source UIViewController 的 MVC 中扮演着 View 的角色，不允许 destination UIViewController 向 source UIViewController 的直接通信，而是要使用 blind 的方式，如 delegate。
	
	**还要注意**，几乎所有类型的 Segue 都是 init 一个 **new** destination UIViewController 给你，所以当你第二次 prepareForSegue 时，第一次 prepareForSegue 拿到的那个 destination UIViewController 已经不复存在了，你需要重新 setup。**这里只有一个例外**，就是后面要提到的 *`Unwind Segue`*，它返给你的 destination UIViewController 就是之前已经存在的 UIViewController，这显然是合理的。举个例子，aVC 通过普通 Segue 转场到 bVC（bVC 会被重新 init），bVC 又通过 Unwind Segue 转回到 aVC，此时，Unwind Segue 的 destination UIViewController 显然必须是之前已经存在的那个 aVC，而不应该去重新 init 一个。
	
	**更要注意**，prepareForSegue 在执行的时候，destination UIViewController 中的 outlet 尚未 set，因为 prepareForSegue 做的事等价于帮助 destination UIViewController 去 init。而 outlet set 是在 init 之后，viewDidLoad 之前做的。参见 [Life Cycle] [2]。
	
4. **系统为你呈现 destination UIViewController**


<!--more-->


### Segue 分类

* **Push Segue**：一般用于 *`UINavigationController`* 中。感受一下 *`UINavigationController`*。

	![UINavigationController](/img/Study_iOS_Segue/6.0.UINavigationController.png)
	
	你可以从一个 Embedded MVC 向另一个 Embedded MVC 创建 Push Segue。
	
	每一个 Embedded MVC 都可以设定自己的 navigationItem.rightBarButtonItems 和 toolbarItems。
	
	每一个 Embedded MVC 中的 *`Back Button`* 都是 UINavigationController 自动添加的，显示的是上一个 Embedded MVC 的 Title，如果上一个没有设定 Title，就显示 “Back” 字样。
	
	![UINavigationController](/img/Study_iOS_Segue/6.1.UINavigationController.png)
	
	**另外**，一般都是通过 Segue 来呈现 destination UIViewController 的。但极少数情况下，你可能需要通过代码手动呈现，此时，你可以利用 UINavigationController 的 API `pushViewController:animated:`，那么要 push 的那个 UIViewController 如何获取呢？利用 UIStoryBoard 的 API `instantiateViewControllerWithIdentifier:`，传入参数是你在 Storyboard 中为该 UIViewController 设定的 Identifier。两者结合起来就是：
	
		-(IBAction)pushViewControllerByMyself
		{
			XXXViewController *xxxVC = [self.storyboard instantiateViewControllerWithIdentifier:@"XXX"];
			xxxVC.xxxProperty = XXX;
			[self.navigationController pushViewController:xxxVC animated:YES];
		}

* **Relationship Segue**：一般用于 *`UITabBarController`* 中。感受一下 *`UITabBarController`* 。

	![UITabBarController](/img/Study_iOS_Segue/6.6.UITabBarController.png)
	
	![UITabBarController](/img/Study_iOS_Segue/6.7.UITabBarController.png)
	
	![UITabBarController](/img/Study_iOS_Segue/6.8.UITabBarController.png)

* **Embedded Segue**：一般用于 *`Container View`* 中。Container View 的作用是，将一个 VC 的 self.view show 到另一个 VC 的 view 中。感受一下 *`Container View`* 。

	![Container View](/img/Study_iOS_Segue/15.0.EmbedSegue.png)
	
	创建 Embedded Segue 的方式同 Push Segue 一样，通过 *Control* + *Drag* 的方式，从 Container View 拖向 destination UIViewController，此时只有一种 Segue 可以选择，那就是 Embedded Segue。
	
	**注意**，这个 Embedded Segue 是在外层 UIViewController 呈现的时候自动触发的（*触发的具体时间点有待进一步研究*），你同样是需要去 prepareForSegue，不要忘了，此时 destination UIViewController 的 outlet 尚未 set。

* **Modal Segue**：一般用于呈现 *`Modal View Controller`*。

	创建 Modal Segue 的方式同 Push Segue 一样，通过 *Control* + *Drag* 的方式，从某个 UIButton（举例来说）拖向 destination UIViewController，选择 Modal 即可。当 Segue 触发时，就会以 Modal 的形式呈现 destination UIViewController。同样，你需要去 prepareForSegue。通过设定 destination UIViewController 的属性 modalTransitionStyle，可以让它以不同的动画形式呈现出来。
	
	当然，你也可以通过代码手动地以 Modal 的形式呈现 destination UIViewController，调用 `presentViewContainer:animated:completion:` 即可，这里的 completion block 会在 destination UIViewController viewDidAppear 后被调用。

* **Unwind Segue**：一般用于从 *`Modal View Controller`* 返回到它的 presenter。通过 Unwind Segue，你也只能是返回到自己的 presenter。Unwind Segue 也可以用在 *`UINavigationController`* 机制中。

	正如之前提到的，使用 Unwind Segue 在 prepareForSegue 时，它返给你的 destination UIViewController 就是之前已经存在的 presenter UIViewController，不会去重新 init 一个 new UIViewController。
	
	**那么创建 Unwind Segue 都需要做些什么呢？**
	
	举例来说，aVC 通过 Modal Segue 的形式呈现出 bVC，现在 bVC 要通过 Unwind Segue 的形式返回到 aVC。那么要创建这个 Unwind Segue 需要2步操作：
	
	1. 在 aVC 的代码中，实现一个如下这样原型的函数：

			-(IBAction)handleUnwindSegue:(UIStoryboardSegue *)segue
			{
				XXXViewController *bVC = (XXXViewController *)segue.sourceViewController; // source, not destination. bVC Unwind Segue to aVC, so bVC is source.
				// get what you want from bVC
			}
			
	2. 在 bVC 的 scene 中，通过 *Control* + *Drag* 拖向下面这个 icon：
	
		![Unwind Segue](/img/Study_iOS_Segue/16.2.UnwindSegue.png)

		然后 Xcode 就会列出 aVC 中所有你实现的那种原型的函数，选择其中匹配的那个 handleUnwindSegue:，并给该 Unwind Segue 设定 Identifier，这样就创建了 bVC 到 aVC 的一个 Unwind Segue。
	
	handleUnwindSegue: 会在 aVC 呈现后被调用，此时，bVC 会自动 dismiss。别忘了在 bVC 的代码中还是需要去 prepareForSegue 的，为这个 Unwind Segue 做 preparation，通常就是准备好供 aVC 在 handleUnwindSegue: 中访问的东西。
	
	**另外**，这里再多提一些关于 Modal View 的东西。前面讲到了，在 aVC 通过 Unwind Segue 被呈现出来的时候，bVC 会自动 dismiss。那么如果不借助 Unwind Segue，如何在代码中手动让其 dismiss 呢。可以利用 `dismissViewControllerAnimated:completion:` 来完成。**注意**，这个消息不是发给 bVC 自己的，而是发给 aVC 的！查看文档就会明白，aVC 调用这个函数后，会 dismiss 掉它 present 出来的 Modal View Controller。所以，在 bVC 的代码中，你通常是这么使用该函数：
	
			[self.presentingViewController dismissViewControllerAnimated:YES completion:XXX];
		
	在 bVC 中通过 self.presentingViewController 拿到 aVC。
	
	手动 dismiss 一般是用于 bVC 什么也不做就返回到 aVC，这样就不需要通过 Unwind Segue 向 aVC 返回数据。
	
[1]: ../../../../2015/01/05/Study_iOS_MVC/
[2]: ../../../../2015/01/05/Study_iOS_Life_Cycle/