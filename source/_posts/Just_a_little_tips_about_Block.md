title: Just a little tips about Objective-C Block
date: 2014-12-03 17:35:36
categories: 开发
tags: [Mac]

---

大爱 *`Block`*，方便易用，但是，有些坑还是需要认真对待的~~

其实问题主要是关于内存管理的。

<!--more-->

#### 1. 捕获闭包的状态

一切问题的起源：*`Block`* 除了包含一段可执行代码，更重要的是，它还会捕获（*`capture`*）其闭包内的状态。

例（1）：

	- (void)testMethod {
	    int a = 1;
	    void (^testBlock)(void) = ^{
	        NSLog(@"Integer is: %i", a);
	    };
	    a = 2;
	    testBlock(); // output is still 1.
	}

在定义 testBlock 时，它会捕获 a 的**类型、值以及修饰符**。但是，捕获只意味着对 a 做了一次**快照**，而并不是真正地拥有 a。也就是说，在定义 testBlock 的时候，外界 a 的值是多少，它捕获进来的值就是多少。外界对 a 值的修改不会影响到 testBlock，同样 testBlock 内部对 a 值的修改也不会影响到外界。

**注意**：

* 捕获闭包的状态是发生在 *`Block`* 定义的时候！而不是 *`Block`* 真正 run 的时候！

* 捕获的是**类型 + 值 + 修饰符**，而不单单是值！

* 对于标量来说，因为 *`Block`* 只是捕获了该变量的值，当然是无法对其进行有效地修改。而对于对象指针来说，捕获进来的是指针的值，无法修改指针的值表示不能让该指针指向另一个对象，但这不影响 *`Block`* 对该指针所指的对象进行操作。例如，如果例（1）中的 a 换成一个数组指针，在 *`Block`* 内部我们是可以做类似于 `[a addObject:]` 这种操作的，外界可以看到对指针所指对象的修改。前提是我们要在 *`Block`* 定义之前就要给 a 所指的对象分配好内存，即在外界 `NSMutableArray *a = [[NSMutableArray alloc] init]`，之后在 *`Block`* 内部就可以任意修改该对象了，而不是修改该对象的指针。

#### 2. 如何让 Block 跟外界共享捕获的内容

前面提到，通常 *`Block`* 只是对变量做了一个快照，那么如何让 block 与外界共享该变量，从而能够修改其捕获的值呢？

全靠 *`__block`* 这个修饰符了！

如果一个变量在声明的时候指定了 *`__block`* 这个修饰符，那就意味着，所有在该变量的作用域内定义的 block，都能够共享该变量。

例（2）：

	- (void)testMethod {
	    __block int a = 1;
	    void (^testBlock)(void) = ^{
	        NSLog(@"Integer is: %i", a);
	    };
	    a = 2;
	    testBlock(); // output is 2 now.
	}

虽然在 testBlock 定义的时候捕获到的值是1，但 a 是 `__block` 修饰的，在 testBlock run 之前，值被改成了2，所以 testBlock run 时能够共享到这一改变。同样，如果 testBlock 内部修改了 a 的值，外界也是会共享到的。

#### 3. 当 Block 被 copy 时会发生什么

首先要明白2点，*`Block`* 什么时候会被 copy，被 copy 到哪里？

一般情况下，*`Block`* 是存储在 stack 中的，当其从 stack 弹出后就消失了。例如例（1），testBlock 是在 testMethod 的 stack 中创建的，自己 run 结束后就被销毁了。这种情况下，*`Block`* 不会对其捕获的变量的内存管理或者生命周期有影响。

但如果 *`Block`* 需要被保存下来，使其在定义结束后的其它作用域下 run，那么这种情况下，它可以被 copy 到 *`heap`* 中进行存储。这个时候，block 被当成了 object 来看待。

比较常见的一种情景，例（3）：

	- (void)configureBlock {
	    self.block = ^{
	        [self doSomething];
	    };
	}

block 被存储到 self.block 中供以后调用，显然是被 copy 到了 heap 中。

**一旦 block 被 copy，那么问题就来了！**

*`Block`* 会对捕获的 self 进行 *`strong reference`*，这极易造成 *`strong reference cycle`*。因为如果该 block 不释放，那么 self 就无法释放，而 block 的释放又需要 self 先被释放，deadlock！除非在 block 用完后，`self.block = nil`，先把 block 释放掉。

只要 block 被这样定义，不管它是否在 run，问题都会一直存在，因为捕获是发生在 block 定义的时候，那个时候它就被 copy 到 heap 中了！

#### 4. weakSelf 和 strongSelf

为了解决例（3）中可能的 strong reference cycle 问题，就引入了 *`weakSelf`*。

例（4）：

	__weak __typeof__(self) weakSelf = self;
	dispatch_group_async(_operationsGroup, _operationsQueue, ^ {
		[weakSelf doSomething];
	});

weakSelf 发挥作用的关键在于，*`Block`* 捕获的是**类型 + 值 + 修饰符**，也就是说，它会捕获到 `__weak` 这个修饰符！这样 block 就不会对 self 进行 strong reference。

采用了 `__weak` 后，block 在定义的时候不会强引用 self，当 block 在真正 run 的时候，weakSelf 要么是 self，要么是 nil。之所以会为 nil，是因为此时 self 可能已经被释放了。

进一步，考虑例（5）：

	__weak __typeof__(self) weakSelf = self;
	dispatch_group_async(_operationsGroup, _operationsQueue, ^ {
		__typeof__(weakSelf) strongSelf = weakSelf;
		[strongSelf doSomething];
		[strongSelf doSomethingElse];
	});

又来了个 *`strongSelf`*！有什么用处？

是这样的，如果执行 doSomething 时，self 还在，而执行 doSomethingElse 时，self被释放了，这是不是有点不妥~~

所以 block 在 run 时，通过内部的 strongSelf 对 self 进行 strong reference，这样可以保证，如果 weakSelf 在 block 刚 run 时是 self，那么在整个 run 的期间也一直是 self，不会让 self 被释放而变成 nil。

**注意**：这个 strongSelf 不会导致 strong reference cycle，因为它不是在 block 定义的时候被捕获进来的，它只是局部变量，只存在于 block run 的期间，当 block run 结束后，对 self 的 strong reference 就消失了。只有 weakSelf 才是在定义的时候被捕获进来的，而它又是 `__weak` 的。


关于 weakSelf 和 strongSelf，有2篇文章可以帮助理解：[【1】] [1] [【2】] [2]

#### 5. 补充

* dispatch_async 对 block 的处理：会将 block copy 到 heap 中，因为它是需要异步处理的，所以需要将 block 存储下来。

* dispatch_sync 对 block 的处理：由于是同步处理的，所以 block 只存在于当前的 stack 中，run 完就释放了，不会被存储在 heap 中。


[1]: http://www.fantageek.com/1090/understanding-weak-self-and-strong-self/
[2]: http://dhoerl.wordpress.com/2013/04/23/i-finally-figured-out-weakself-and-strongself/
