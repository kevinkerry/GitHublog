title: Study iOS - UIView Animation
date: 2015-01-06 22:29:21
categories: 开发
tags: [iOS]

---

### Developing Applications for iOS 观后感系列：UIView Animation

*相关视频章节：8*

---

本篇提到的 Animation 主要是基于 UIView 的，并没有涉及到基于 Layer 机制的 Animation。

基于 UIView 的 Animation 有以下3种：

### Animation of UIView Property

**首先一定要明确一点**：animation 虽然是经历一定时间 show 在屏幕上的，但对 property 的值的修改却是早就已经立即生效了。

UIView 的某些 property 的改变是可以施加 animation 的，例如：

	frame
	transform（translation，rotation，scale）
	alpha（opacity）
	
实现这一类 animation 的 API 是 UIView 的 `class method`：

	+(void)animateWithDuration:(NSTimeInterval)duration
						 delay:(NSTimeInterval)delay
					   options:(NSViewAnimationOptions)options
					animations:(void(^)(void))animationsBlock
					completion:(void(^)(BOOL finished))completionBlock
					
`animationsBlock` 里面执行你对 UIView property 所做的修改。

`completionBlock` 表示 animation 呈现结束后要做的事，如果 animation 成功地呈现完毕，finished 为 YES，否则，若中断了，则返回 NO。

**注意**：animation 是在该 API 返回后开始的，而该 API 返回的时刻是 `animationsBlock` 执行完毕的时刻。**也就是说**，调用该 API 后，会立刻执行其 `animationsBlock`，执行完后就立刻返回，然后 animation 开始呈现。所以对 UIView property 的修改在 `animationsBlock` 执行完后就立即生效了，只不过 property 的变化要经过 duration 才呈现完毕。

**另外**：finished 何时为 NO？如果在该 animation 开始后，又调用了该 API 对相同的 property 开始了另一个 animation，这样前一个 animation 没做完就得中断去做另一个 animation 了，这种情况下，前一个 animation 的 completionBlock 的 finished 就为 NO 了。


<!--more-->


### Animation of Entire UIView

例如整个 UIView 的 flipping、dissolving、curling 等效果，在呈现这些效果的同时，你可以对整个 UIView 进行任意修改，改变其 appearance。

实现这一类 animation 的 API 是 UIView 的 `class method`：

	+(void)transitionWithView:(UIView *)viewToModify
				     duration:(NSTimeInterval)duration
					  options:(NSViewAnimationOptions)options
				   animations:(void(^)(void))animationsBlock
				   completion:(void(^)(BOOL finished))completionBlock
					
`animationsBlock` 和 `completionBlock` 的机制同前面一种 animation 一样，只不过你不再是只单单地修改 UIView 的某个 property，你可以对整个 UIView 做任意的修改（例如重新绘制整个 UIView 等）。

`options` 可以指定 animation 采用的效果：flipping、dissolving、curling 等。

如果不是想对同一个 UIView 进行 animation 修改，而是要转为呈现另一个 UIView，则需要使用这个 API：

	+(void)transitionFromView:(UIView *)fromView
					   toView:(UIView *)toView
				     duration:(NSTimeInterval)duration
					  options:(NSViewAnimationOptions)options
				   completion:(void(^)(BOOL finished))completionBlock

此时已经不再需要 `animationsBlock` 了，因为要呈现的 animation 就是从 fromView 变为 toView。

`optioins` 可以指定：你是想 hide fromView 而呈现 toView，还是想 remove fromView 而重新 add toView。

### Dynamic Animator

这一类 animation 是指 *`animatable objects`*（通常就是 UIView）的 *`physics animation`*。将某些 physics 效果施加到 animatable objects 上，这些 objects 就会受到 physics 的影响自行开始动画效果，直到在 physics 的作用下 resolve to stasis（相对静止状态）。

Steps：

1. 创建 `UIDynamicAnimator`，并指定其 `delegate`。

		UIDynamicAnimator *animator = [[UIDynamicAnimator alloc] initWithReferenceView:aView];
		animator.delegate = aDelegate;
		
	设定 delegate 是为了响应 UIDynamicAnimatorDelegate 的2个函数：
	
	`-(void)dynamicAnimatorDidPause:(UIDynamicAnimator *)animator`：resolve to stasis 导致动画暂停
	
	`-(void)dynamicAnimatorWillResume:(UIDynamicAnimator *)animator`：动画重新开始

2. 向 UIDynamicAnimator 中添加各种 physics 效果，即 `UIDynamicBehavior`（gravity，collisions 等等）。

	UIDynamicBehavior 有很多子类，例如创建一个 gravity 作用效果：
	
		UIGravityBehavior *gravity = [[UIGravityBehavior alloc] init];
		[animator addBehavior:gravity];
		
	* 有一个比较特殊的 UIDynamicBehavior，即 `UIDynamicItemBehavior`，它的各个 property 用于指定 objects 的一些固有 physic 属性，例如：allowsRotation，friction，elasticity，density 等。这些属性会间接影响到其他 UIDynamicBehavior 的作用效果。
	
	* 你也可以 subclass UIDynamicBehavior，进行自定义。可以利用它的函数 `addChildBehavior:(UIDynamicBehavior *)behavior` 将若干个 UIDynamicBehavior 组合成一种新的 UIDynamicBehavior。通常你还需要 override 它的 `init`，`addItem`，`removeItem`。
	
	* UIDynamicBehavior 有一个 property 用于访问自己所在的 UIDynamicAnimator：`dynamicAnimator`。

	* UIDynamicBehavior 还有一个重要的 property：`@property (copy) void (^action)(void)`，每当 UIDynamicBehavior 在 objects 上生效时，该 block 都会被调用。所以，这里的操作最好不要太费时，因为它会被频繁地调用。

3. 将 `UIDynamicItem`（animatable objects，通常就是 UIView）添加到某些 UIDynamicBehavior 下。

	UIDynamicItem 是一个 protocol，指定3个 property：bounds，center，transform。而 UIView 实现了该 protocol。
	
		id <UIDynamicItem> item = XXX // 例如某个 UIView
		[gravity addItem:item];

4. 一旦完成 step 3，UIDynamicItem 就会自动在 UIDynamicBehavior 的作用下开始运动。

	**注意**，如果你在 UIDynamicAnimator 的作用下，手动修改了 UIDynamicItem 的属性，那么必须调用 UIDynamicAnimator 的函数 `updateItemUsingCurrentState:item` 来让 UIDynamicAnimator 重新获取其属性，这样作用在 item 上的 UIDynamicBehavior 才能根据当前的最新状态继续产生作用力。
