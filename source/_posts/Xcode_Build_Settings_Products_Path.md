title: Xcode Build Settings - Products Path
date: 2015-11-16 20:21:54
categories: 开发
tags: [Xcode]

---

假设一个 Project 的路径是 `XXX/ProjectX`。

当 build 完成后，我们会关注 2 个与 **Product Files** 有关的 settings ：

<!--more-->

**`$BUILT_PRODUCTS_DIR`** 和 **`$TARGET_BUILD_DIR`**

这 2 个 settings 决定了我们 build 出来的 Products 被放置到了哪里。

#### 官方解释 ####

* **`$BUILT_PRODUCTS_DIR`**：Directory path. Identifies the directory under which all the product’s files can be found. This directory contains either product files or symbolic links to them.

* **`$TARGET_BUILD_DIR`**：Directory path. Identifies the root of the directory hierarchy that contains the product’s files (no intermediate build files).

乍一看好像都是用来存放 **Product Files** 的目录，但是它们的作用是不一样的：

刚编译完的时候，Xcode 会先把 **Product Files** 生成到 **`$BUILT_PRODUCTS_DIR`** 中，然后再根据具体的 configuration 来决定要不要对生成的 **Product Files** 进行 deploy，如果不需要 deploy，那么 **Product Files** 最后还是留在 **`$BUILT_PRODUCTS_DIR`** 中，如果需要 deploy，那么 **Product Files** 会被 move 到 **`$TARGET_BUILD_DIR`** 中，而原来的 **`$BUILT_PRODUCTS_DIR`** 中就只有 symbolic links 了。

#### 举个例子 ####

普通的 app 在 build 完之后，一般放置在 `$BUILT_PRODUCTS_DIR` 中即可，而对于 Xcode Plugin 来说，在 build 完之后一般会直接将其 deploy 到 Xcode 的 **Plug-ins** 目录中，此时 `$TARGET_BUILD_DIR` 就是 **Plug-ins** 目录，而 `$BUILT_PRODUCTS_DIR` 中只有 Plugin 的 symbolic links。

*可以 clone 一下我之前写的一个[小插件] [1]，build 一下看结果。*

-----

#### 下面就看一下这 2 个路径是如何设置的 ####

**首先解释一些相关的 settings：**

**settingName (settingDisplayNameInXcodeBuildSettings)**

* **`$SRCROOT`**：`XXX/ProjectX`

* **`$SYMROOT (Build Products Path)`**：Directory path. Identifies the root of the directory hierarchy that contains product files and intermediate build files. Product and build files are placed in subdirectories of this directory. 看完官方解释，我再说一下它具体如何设置。
 1. 默认值是 `$SRCROOT/build`
 
 2. 可以在 Xcode 的 Preferences 里面设置 `Build Location`，如下图
 ![Build Location](/img/Xcode_Build_Settings_Products_Path/location.png)

 3. 直接在 Target 的 `Build Settings` 里面设置 `Build Products Path`，如下图
 ![Build Products Path](/img/Xcode_Build_Settings_Products_Path/build_products_path.png)

 这 3 种设置的优先级依次递增，也就是说，如果你同时设置了 2 和 3，那么最后 build 是按照 3 的值来处理的。

* **`$CONFIGURATION`**：Debug、Release、自定义的等等。

* **`$CONFIGURATION_BUILD_DIR (Per-Configuration Build Products Path)`**：= `$SYMROOT/$CONFIGURATION`，例如 `XXX/ProjectX/build/Debug`。

* **`$DEPLOYMENT_LOCATION (Deployment Location)`**：Boolean value. Specifies whether product files are placed in the installation or the build directory. 它决定了要不要对 **Product Files** 进行 deploy。

* **`$SKIP_INSTALL (Skip Install)`**：Boolean value. Specifies whether to place the product at the location indicated by *`$DSTROOT`* or the uninstalled products directory inside the directory indicated by *`$TARGET_TEMP_DIR`*. 它决定了具体要把 **Product Files** deploy 到哪里。

**下面就轮到 2 个主角了。**

* **`$BUILT_PRODUCTS_DIR`**：= `$CONFIGURATION_BUILD_DIR` = `$SYMROOT/$CONFIGURATION` *（其实除了这个值，还有一种取值，不过基本不用，所以这里为了便于理解把它忽略了。详情可查阅[官方文档] [2]）*

* **`$TARGET_BUILD_DIR`**：
 1. 如果 `$DEPLOYMENT_LOCATION` = `NO`，即不需要 deploy，那么 `$TARGET_BUILD_DIR` 就直接等同于 `$BUILT_PRODUCTS_DIR`。
 
 2. 如果 `$DEPLOYMENT_LOCATION` = `YES`，即需要 deploy，那么：
 
 2.1. 如果 `$SKIP_INSTALL` = `NO`，那么 `$TARGET_BUILD_DIR` = `$INSTALL_DIR` = `$DSTROOT/$INSTALL_PATH`。其中，`$DSTROOT` 的 display name 是 `Installation Build Products Location`，`$INSTALL_PATH` 的 display name 是 `Installation Directory`。另外需要注意，为了拼接出一个有效的 `$INSTALL_DIR`，`$INSTALL_PATH` 必须以 `/` 开头。

 2.2. 如果 `$SKIP_INSTALL` = `YES`，那么 `$TARGET_BUILD_DIR` = `$TARGET_TEMP_DIR/UninstalledProducts`。

 一般需要 deploy 的时候，都是设置 `$SKIP_INSTALL` 为 `NO`，然后设置具体的 `$DSTROOT` 和 `$INSTALL_PATH`，所以这里忽略对 `$TARGET_TEMP_DIR` 理解。



[1]: https://github.com/wzqcongcong/AtAutoCompletion
[2]: https://developer.apple.com/library/mac/documentation/DeveloperTools/Reference/XcodeBuildSettingRef/0-Introduction/introduction.html

