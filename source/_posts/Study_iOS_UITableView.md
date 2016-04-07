title: Study iOS - UITableView
date: 2015-01-11 14:42:21
categories: 开发
tags: [iOS]

---

### Developing Applications for iOS 观后感系列：UITableView

*相关视频章节：11*

<!--more-->

---

#### UITableView 分类

按内容分：

* *`dynamic`*：在 storyboard 中设定 prototype，在代码中 load 内容。*（本篇主要涉及 dynamic NSTableView）*

* *`static`*：完全在 storyboard 中设定。

按样式分：

*`ungrouped`* / *`grouped`*

示例：

![UITableView](/img/Study_iOS_UITableView/11.0.UITableView.png)

#### UITableView 的构成

与 Mac 上的NSTableView 不同，iOS 中的 UITableView 都是一维的，只有一列。

![UITableView](/img/Study_iOS_UITableView/11.1.UITableView.png) ![UITableView](/img/Study_iOS_UITableView/11.2.UITableView.png)

#### UITableView 的使用

1. **设定 cell 的 `prototype`**

	![UITableView](/img/Study_iOS_UITableView/11.3.UITableView.png)

	![UITableView](/img/Study_iOS_UITableView/11.4.UITableView.png)
	
2. **实现 `UITableViewDataSource`**

	**UITableViewDataSource** 用于控制 what UITableView displays。主要有这些 API：

	* 基于 prototype，每个 cell 该如何显示在 table 中：
	
		![UITableView](/img/Study_iOS_UITableView/11.5.UITableView.png)
		
	* table 有多少 section，多少 row：

		![UITableView](/img/Study_iOS_UITableView/11.6.UITableView.png)

3.  **实现 `UITableViewDelegate`**

	**UITableDelegate** 用于控制 how UITableView displays，以及 observe what UITableView is doing（排序、选择、滚动、插入、删除、更新等等），参见文档。这里简单提几个：

	* 用户选中某行时，会调用 *`-(void)tableView:didSelectRowAtIndexPath:`*

	* 用户点击某行中的 *`accessory button`*，会调用 *`-(void)tableView:accessoryButtonTappedForRowWithIndexPath:`*

		![UITableView](/img/Study_iOS_UITableView/11.9.UITableView.png)
		
	* 当然，你可以不使用这些 delegate，而是借助 Segue：
	
		![UITableView](/img/Study_iOS_UITableView/11.10.UITableView.png)
	
		然后 prepareForSegue：
		
			-(void)prepareForSegue:(UIStoryboardSegue *)segue sender:(id)sender
			{
				NSIndexPath *indexPath = [self.tableView indexPathForCell:sender];
				// indexPath.row in indexPath.section
				// prepare segue.destinationController
			}
	
4. **reload**

	当 table 的 Model 发生变化后，你需要去 reload 来更新 table。
	
	`-(void)reloadData`：更新整个 table，重新调用一遍 UITableViewDataSource，比较 heavyweight。
	
	`-(void)reloadRowsAtIndexPaths:(NSArray *)indexPaths withRowAnimation:(UITableViewRowAnimation)animationStyle`：只更新指定的行，比较 lightweight。
	
5. **Spinner**

	UITableView 有个内置的 activity indicator Spinner：`@property (strong) UIRefreshControl *refreshControl;`
	
	![UITableView](/img/Study_iOS_UITableView/11.12.UITableView.png)
	
	`[self.refreshControl beginRefreshing]`：table 下伸，Spinner 出现并开始 spinning。
	
	`[self.refreshControl endRefreshing]`：table 上收，Spinner 停止 spinning 并消失。
	
	有了这个 Spinner，你就可以在 reload 的时候给用户提示适当的状态了。
	
	这个 Spinner 是可以 *`Control`* + *`Drag`* 出 target-action 的。如果用户 pull down table，那么这个 action 就会触发！这样，你就可以在 action 中让 Spinner start spinning，然后去更新 table。
	
	![UITableView](/img/Study_iOS_UITableView/11.13.UITableView.png)
	
	![UITableView](/img/Study_iOS_UITableView/11.14.UITableView.png)
	
	**注意**，用户 pull down 触发的是 Spinner 的 target-action，而不是触发 beginRefreshing。你需要在 action 中调用 beginRefreshing。
	