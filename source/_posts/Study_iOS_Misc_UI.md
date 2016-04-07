title: Study iOS - Misc UI
date: 2015-01-17 15:30:03
categories: 开发
tags: [iOS]

---

### Developing Applications for iOS 观后感系列：Misc UI

*相关视频章节：all*

<!--more-->

---

之前的文章已经学习了几种 iOS 中比较大而重要的 UIView，例如 UIScrollView、UITableView 等，本篇主要了解一下其它一些比较常用的小型 UI 类型。

### UITextView

**`UITextView`** 提供了一个 multi-line、selectable、editable、scrollable 的编辑区，通过 *`NSMutableAttributedString`* 来设定它的文本内容和各种属性。

UITextView 有3个比较重要的 property，它们共同完成整个 UITextView 内容的显示。

* *`textStorage`*：NSTextStorage 是 NSMutableAttributedString 的 subclass，也就是说，UITextView 的文本内容和文本属性是通过该 property 来决定的，对其赋值什么样的 NSMutableAttributedString，就会显示什么样的内容。

* *`textContainer`*：NSTextContainer 决定了 NSTextStorage 将会布局在整个 UITextView 的哪个区域中，即 where text can be。你甚至可以通过它来指定一个 *`exclusion zones`*（例如一个 image），这样 NSTextStorage 就会围绕这个 exclusion  zones 来布局，如同 Word 中的图片环绕效果，灰常 NB。

* *`layoutManager`*：前面2个 property，一个决定内容，一个决定样式，那么剩下的 NSLayoutManager 就是负责从 NSTextStorage 中获取到 text，然后将它们 lay down 到 NSTextContainer 设定的有效区域中。

### UITextField

**`UITextField`** 就像是 UILabel 的升级版，它是 editable 的。这里主要了解下 *`UITextFieldDelegate`* 以及 *`Keyboard`* 的使用。当然 UITextView 也有这些东西，这里就不赘述了。

* 当 UITextField 成为 *`first responder`* 时，iOS 的 Keyboard 就会自动弹出。有2种方式可以实现：

	1. 用户点击 UITextField，它自动就会变成 first responder。

	2. 通过代码主动调用 *`becomeFirstResponder`*。

	反过来，想要让 Keyboard 收回，调用 *`resignFirstResponder`* 即可。
	
* *`UITextFieldDelegate`*，举几个例子：

	`-(BOOL)textFieldShouldReturn:(UITextField *)textField`：当用户按下 Return 键后，该 delegate 就会触发。在这里，除了返回 BOOL，通常你还会调用 [textField resignFirstResponder] 来收回 Keyboard。
	
	`-(void)textFieldDidEndEditing:(UITextField *)textField`：当 UITextField resign first responder 后，该 delegate 就会触发。在这里，你可以利用新的 UITextField 来 update UI。
	
* UITextField 还有几个相关的 *`Notifications`*，用于实时反馈 UITextField 当前的状态。例如 
UITextFieldTextDidChangeNotification。

* *`target-action`*：UITextField 也是个 UIControl，所以你可以对其设定 target-action。UITextField 有许多 UIControlEvents 供设定，具体参见在 Xcode 中右击 UITextField 显示的菜单。

* *`Keyboard`*

	一般来说，我们提到 Keyboard 时，总是指通过某个 UITextField 或者 UITextView 而弹出来的那个 Keyboard，两者是相关的，对该 Keyboard 的设置通常就是通过与其相关的那个 UITextField 或者 UITextView 来设定的。

	如何控制 Keyboard 的外观：通过 *`UITextInputTraits`* 这个 protocol。UITextField 和 UITextView 已经实现了。该 protocol 中有若干 property 用于自定义你的 Keyboard。
	
	你可以给 Keyboard 添加一个 accessory view 来实现自定义的 toolbar，通过设置 UITextField 的 property *`inputAccessoryView`* 来完成。此外，UITextField 还有些其它的 accessory view，例如 left view、right view、clear button 等等。
	
	**注意**，Keyboard 出现时，总是会 cover 其它 view，所以，你需要自己来处理 Keyboard 弹出后的 UI 布局。例如移动位置等等。你可以通过 *`UIKeyboardXXXNotification`* 来获取 Keyboard 的状态，从而做出响应。例如，如果 UITableView 中有个 UITextField，那么 UITableViewController 就需要监听 UIKeyboardDidShowNotification，这样当用户编辑某行的 UITextField 导致 Keyboard 弹出时，table 可以进行 scroll，从而保证该行不会被 Keyboard 覆盖。

### UIActionSheet

**`UIActionSheet `** 用于呈现一个选择列表。通常是从屏幕下方弹出，让用户做出选择。

* *`init`*：`initWithTitle:delegate:cancelButtonTitle:destructiveButtonTitle:otherButtonTitles:`，其中，destructiveButton 会显示为红色，用于警示用户该操作的严重性。其它 button 为普通颜色。

* *`button`*：init 之后，你还可以通过 `addButtonWithTitle:` 继续给 action sheet 添加其它普通 button。之后想要获取 action sheet 中的这些 button 时，可以通过 action sheet 的这些 property 来完成：*`cancelButtonIndex`*、*`destructiveButtonIndex`*、*`numberOfButtons`*、*`buttonTitleAtIndex:`* 等等。
	
* *`display`*：

	* `[actionSheet showInView:]`
	
	* `[actionSheet showFromBarButtonItem:animated:]`，**这里需要注意一点**，对于这种 display 方式（通常是在 iPad 中），action sheet 是显示在一个 popover 中的，是不会显示 cancel button 的，显然这是合理的，因为点击屏幕其它地方后 popover 就可以自己消失。但是别忘了，popover 有个 passthroughViews，点击这里面的 view 不会让 popover 消失，而 bar button item 是在 passthroughViews 中的，所以，如果你不停地点击它，就会不停地叠加 action sheet。所以这里需要你手动处理下，保证只弹出一个 action sheet。
	
	* `[actionSheet showFromRect:inView:animated:]`
	
* *`delegate`*：通过 delegate，你可以知道用户点击的是哪个 button，即 `actionSheet:didDismissWithButtonIndex:`。

	此外，你可以通过 code 手动触发点击某个 button 这种行为，即 `dismissWithClickedButtonIndex:animated:`。

### UIAlertView

**`UIAlertView`** 与 UIActionSheet 很相似，具有相似的 init 方式、添加新 button 的函数等。这里只了解几点不同的地方。

* *`display`*：只有一种方式，即 `[alert show]`，alert 就会显示在屏幕中间。

* *`add UITextField`*：你可以在 UIAlertView 中添加 UITextField，让用户在其中输入，例如登录框。方法是设置 UIAlertView 的 property *`alertViewStyle`*。

### UIImagePickerController

**`UIImagePickerController `** 用于呈现一个 Modal view，让用户选择一张照片（photo library 中已有的照片，或者用 camera 新拍一张照片），然后将该照片返回供 app 使用。

**注意**，既然是个 Modal view，那就意味着需要这么呈现：通过 `Modal Segue`，或者调用 `presentViewContainer:animated:completion:`。参见 [Segue] [1]。

UIImagePickerController 的使用流程如下：

1. *`init & delegate`*：delegate 一般设置为 UIImagePickerController 的 presenter。

2. *`configuration`*

	主要是设置 UIImagePickerController 的 source type、media type 等等。设置之前，通常需要先进行检查，看看该 device 是否支持要设置的值。

	* *`source type`*：有的 device 有 camera，有的则没有 camera，只有 photo library。因此你需要检查该 device 是否支持 UIImagePickerController 的某种 source type。通过调用 `isSourceTypeAvailable:` 即可。

		有3种 source type：UIImagePickerControllerSourceTypePhotoLibrary、UIImagePickerControllerSourceTypeCamera、UIImagePickerControllerSourceTypeSavedPhotosAlbum。

	* *`media type`*：每种 source type 支持的 media type 也不同，有的支持 video，有的只支持 picture。因此 media type 也需要进行检查。通过调用 `+(NSArray *)availableMediaTypesForSourceType:`即可。

		有2种 media type：kUTTypeImage 和 看UTTypeMovie。
		
		还有其它一些检查函数，例如检查 front/rear camera 等等，参见文档。
		
	* *`editability`*：是否允许用户对选取的图片进行编辑。如果允许，UIImagePickerController 会把原始图片和编辑后的图片都返给 app。

	* 其他 configuration，参见文档。

3. *`present`*：前面已经提到了，要采用 Modal view 的呈现方式。

4. *`respond to delegate`*

	当用户完成选取照片后：`imagePickerController:(UIImagePickerController *)picker didFinishPickingMediaWithInfo:(NSDictionary *)info`，其中，info 中提供了要返给 app 的图片数据，参见文档。通过 *`ALAssetsLibrary`*，你可以将该图片保存到用户的 photo library。
	
	当用户取消选取照片后：`imagePickerControllerDidCancel:(UIImagePickerController *)picker`。
	
	在用户完成/取消选取照片后，你都需要关闭当前的这个 Modal view，即调用 `[self dismissViewControllerAnimated:completion:]`。

### Settings

**`Settings `** 指的是 iOS 中的 Settings app，你可以在里面添加自己 app 的设置选项。这个机制利用的是 *`NSUserDefaults`*。

直接上图

![Settings](/img/Study_iOS_Misc_UI/18.10.Settings.png)

![Settings](/img/Study_iOS_Misc_UI/18.11.Settings.png)

![Settings](/img/Study_iOS_Misc_UI/18.12.Settings.png)

![Settings](/img/Study_iOS_Misc_UI/18.13.Settings.png)

**注意**：code 通常需要监听 NSUserDefaults 的变化，来及时获取 Settings 中新的值。


[1]: ../../../../2015/01/10/Study_iOS_Segue/