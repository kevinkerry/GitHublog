title: Study iOS - CoreData
date: 2015-01-11 16:51:58
categories: 开发
tags: [iOS]

---

### Developing Applications for iOS 观后感系列：CoreData

*相关视频章节：12，13*

<!--more-->

---

**`CoreData`** 是 iOS 提供的一套用于管理 app 的 Model 的 framework，它就像是个 object database，支持对 object 的各种操作：add、delete、query、undo、redo、autosave 等，灰常 NB。但它并不是 database 的某种实现，你可以基于持久化文件（如 sqlite 等）来使用 CoreData，也可以完全基于内存来使用。

使用 CoreData 会涉及 Xcode 和 code 两方面。

首先看一下在 Xcode 中需要做什么。

### 创建 Data Model

* 类似于 .storyboard 文件，这里你需要创建一个 *`.xcdatamodeld`* 文件，它存储了 app 的 Data Model。.xcdatamodeld 文件内容如下：

	![CoreData](/img/Study_iOS_CoreData/12.0.CoreData.png)

	就像设计数据库的 ER 图，在这里，你需要设计好 object 对应的 *`Entity`*。这里的 Entity 会对应到 code 中的 *`NSManagedObject（或其 subclass）`*。

	![CoreData](/img/Study_iOS_CoreData/12.1.CoreData.png)

	![CoreData](/img/Study_iOS_CoreData/12.2.CoreData.png)

* Model 构建好了之后，下面就需要创建出与这些 Entity 对应的 NSManagedObject，供 code 中使用。

	![CoreData](/img/Study_iOS_CoreData/12.12.CoreData.Add.png)

	![CoreData](/img/Study_iOS_CoreData/12.13.CoreData.Add.png)

	这样我们就可以直接使用这些 NSManagedObject subclass 了，而且也可以方便地访问它们的 properties，而不用去调用繁琐的 valueForKey 和 setValue:forKey。

	**注意**，以后当我们修改了 Model 的 Entity 后，需要重新执行一遍这一步，来更新这些 NSManagedObject subclass。这也意味着，这些 .h 和 .m 会被重新生成。所以这里常常使用的一个技巧是，为这些 NSManagedObject subclass 添加 *`category`*，在 category 中创建一些额外的函数供 NSManagedObject subclass 使用，这样以后更新 NSManagedObject subclass 时，只有最原本的 .h 和 .m 文件被重新生成，而不会影响到我们的 category。
	
Xcode 这边基本就完成了，下面开始进入 code。

### NSManagedObjectContext

CoreData 的基本要素就是 NSManagedObject。那么如何获取我们的 NSManagedObject 呢？答案就是从 *`NSManagedObjectContext`* 中获取。

有2种方式来获取这个 NSManagedObjectContext：

1. 创建一个 *`UIManagedDocument`*，它里面有个 property：managedObjectContext。Bingo！

2. 当创建 project 时，下面有个选项：*`"Use Core Data"`*，如果选中这个选项，那么 Xcode 会在 *`AppDelegate`* 中给你提供一个 property：managedObjectContext。以后 code 中访问它就行了。

本篇主要涉及第1种方式，下面就理解一下 UIManagedDocument。

### UIManagedDocument

UIManagedDocument 是 *`UIDocument`* 的 subclass，提供了管理存储内容的机制。它也支持 iCloud，可以把它理解成 CoreData Model 的一个容器。

* **创建 UIManagedDocument**：

		NSURL *documentsDir = [[[NSFileManager defaultManager] URLsForDirectory:NSDocumentDirectory
																	  inDomains:NSUserDomainMask] firstObject];
		NSString *managedDocumentName = @"XXX";
		NSURL *managedDocumentURL = [documentsDir URLByAppendingPathComponent:managedDocumentName];
		UIManagedDocument *managedDocument = [[UIManagedDocument alloc] initWithFileURL:managedDocumentURL];
		
	上面只是在内存中创建了一个 UIManagedDocument instance，想要从中获取 NSManagedObjectContext，你还需要下面操作。
	
* **通过 open / create UIManagedDocument 来获取 NSManagedObjectContext**

		if ([[NSFileManager defaultManager] fileExistsAtPath:[managedDocumentURL path]]) {
			[managedDocument openWithCompletionHandler:^(BOOL success) {
				if (success) {
					[self documentIsReady];
				}
			}];
		} else {
			[managedDocument saveToURL:managedDocumentURL
					  forSaveOperation:UIDocumentSaveForCreating
					 completionHandler:^(BOOL success) {
					 	if (success) {
							[self documentIsReady];
						}
					 }];
		}
		
		-(void)documentIsReady
		{
			if (managedDocument.documentState == UIDocumentStateNormal) {
				NSManagedObjectContext *context = managedDocument.managedObjectContext;
				// use the context to do Core Data stuff
			}
		}

	上面示例中注意2点：
	
	为什么需要 *`completionHandler`*？因为 open/save 操作是 *`asynchronous`*（想想看，iCloud 必须要 asynchronous）。
	
	关于 check *`documentState`*，还有其它一些取值，参见文档。
	
* **保存和关闭**

	![CoreData](/img/Study_iOS_CoreData/12.7.UIManagedDocument.png)

	虽然 NSManagedObjectContext 是 autosave 的，但你是可以观察到这一动作的：

	![CoreData](/img/Study_iOS_CoreData/12.8.UIManagedDocument.png)

好了，UIManagedDocument 有了，NSManagedObjectContext 也有了，现在轮到 NSManagedObject 了。

### NSManagedObject 的 Add

![CoreData](/img/Study_iOS_CoreData/12.14.CoreData.Add.png)

### NSManagedObject 的 Delete

![CoreData](/img/Study_iOS_CoreData/12.17.CoreData.Delete.png)

**注意**，不管是 Add 还是 Delete，这些 change 都是发生在内存中的，直到你手动 save 或由 UIManagedDocument 自己来 autosave。

### NSManagedObject 的 Query

query 操作是由 *`NSFetchRequest`* 来完成的。创建 NSFetchRequest 需要一下4点：

* 指定要 fetch 的 Entity。
* 指定 fetch 结果集的大小，默认是所有。
* 指定 NSSortDescriptor 对结果集进行排序。
* 指定 NSPredicate 对结果集进行过滤，默认是所有。

(关于 *`NSSortDescriptor`* 和 *`NSPredicate`*，参见文档。)

创建好 NSFetchRequest 后，就可以调用 `NSArray *results = [context executeFetchRequest:request error:&error]` 来获取结果了。

**注意**，出于内存优化，CoreData 在 execute NSFetchRequest 时采用了 *`lazy load`* 的机制，它不会一次性地把 results 中的所有 NSManagedObject 都加载到内存中，而是等到你真正地访问其中的某个 NSManagedObject 时，才会进行加载。

### CoreData 的 Thread Safety

**NSManagedObjectContext 不是线程安全的**，不过 CoreData 一般非常高效，所以没必要去多线程操作（例如查询）。

**这里需要注意**，NSManagedObjectContext 在哪个 queue 上创建，你就只能在那个 queue 上访问 NSManagedObjectContext 以及它的 NSManagedObject。很可能就是在 main Q 上。

鉴于上面这个问题，NSManagedObjectContext 提供了一个线程安全的 API：
	
	[context performBlock:^{
		// do stuff with NSManagedObjectContext & NSManagedObject
	}]
		
**context 会在自己的 safe queue（即被创建的 queue）上来执行 block。** :)

CoreData 本身就先提这些，下面还有个比较重要的话题。

### CoreData & UITableView

CoreData 中的数据通常会显示到 UITableView 上，因此 iOS 为我们提供了 *`NSFetchedResultsController`*，用于把 NSFetchRequest hook 到 UITableView 上，实现 CoreData 到 UITableView 上的自动映射。

* **NSFetchedResultsController 提供 UITableViewDataSource**

		-(NSUInteger)numberOfSectionsInTableView:(UITableView *)sender
		{
			return [[self.fetchedResultsController sections] count];
		}
	
		-(NSUInteger)tableView:(UITableView *)sender numberOfRowsInSection:(NSUInteger)section
		{
			return [[[self.fetchedResultsController sections] objectAtIndex:section] numberOfObjects];
		}
	
		-(UITableViewCell *)tableView:(UITableView *)sender
				cellForRowAtIndexPath:(NSIndexPath *)indexPath
		{
			UITableViewCell *cell = XXX;
			NSManagedObject *managedObject = [self.fetchedResultsController objectAtIndexPath:indexPath];
			// setup
			return cell;
		}
	
	上面需要注意的就是 NSFetchedResultsController 的一个 API：

	`-(NSManagedObject *)objectAtIndexPath:(NSIndexPath *)indexPath`

	也就是说，NSFetchedResultsController 以某种方式将结果集的结构封装的跟 UITableView 的 DataSource 一样，也有个 NSIndexPath。

	那么 NSFetchedResultsController 该如何创建呢？

* **创建 NSFetchedResultsController**

	![CoreData](/img/Study_iOS_CoreData/13.2.NSFetchedResultsController.png)

	**需要注意的是**：NSFetchRequest 中的 sortDescriptor 一定要与 NSFetchedResultsController init 函数中的 sectionNameKeyPath 匹配，这样 NSFetchedResultsController 才能正确地构造出前面提到的 NSIndexPath。**此外**，调用 init 函数后，你就不能再去修改 NSFetchedResultsController 的 fetchRequest 的属性了（例如修改 sortDescriptor 和 predicate）。
	
* **通过 NSFetchedResultsController 执行 query**

	调用 NSFetchedResultsController 的函数 `-(BOOL)performFetch:(NSError **)error`，即可实现手动 fetch，这之后，通常伴随着 UITableView 的 reload。
	
	除了手动 fetch，NSFetchedResultsController 更重要的功能是自动捕获 CoreData 的变化，并以此更新 UITableView。想要实现这一点，需要下面的东西。

* **NSFetchedResultsControllerDelegate**

	当给 NSFetchedResultsController 指定 delegate 后，它就会 keep tracking 自己的 NSFetchRequest，并将更新报告给它的 delegate。显然，在这里，它的 delegate 应该指定为 UITableViewController。如果不指定 delegate，它就不会实现这种机制。
	
	主要有这些 delegate 函数：
	
	* `controller:didChangeObject:atIndexPath:forChangeType:newIndexPath:`：track 单个 object 的更新（add、remove、move、update）

	* `controller:didChangeSection:atIndex:forChangeType:`：track 某个 section 的 add 或 remove
	
	* `-controllerDidChangeContent:`：track 整个 NSFetchRequest

参见文档 sample code。

