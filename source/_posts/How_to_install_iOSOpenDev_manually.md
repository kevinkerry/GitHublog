title: How to install iOSOpenDev manually
date: 2016-03-29 23:30:35
categories: 开发
tags: [iOS]

---

OS X 10.10+ & Xcode 7+

[https://github.com/wzqcongcong/iOSOpenDev] [1]

-----

1. **Dependency**

 `brew install ldid dpkg`

2. **Theos**

 [Wiki](https://github.com/theos/theos/wiki)
    
 [Setup](http://iphonedevwiki.net/index.php/Theos/Setup)

3. **iOSOpenDev**

 [Wiki](https://github.com/kokoabim/iOSOpenDev/wiki)

 3.1. update **.zshrc**
 
 `export iOSOpenDevPath=$HOME/Tools/iOS/iOSOpenDev`
 
 `export iOSOpenDevDevice=`
 
 `export PATH=$iOSOpenDevPath:$PATH`

 3.2. get **iOSOpenDev**
 
 * `git clone --recursive https://github.com/kokoabim/iOSOpenDev.git $iOSOpenDevPath`

 * copy the [**templates**] [1] dir into *$iOSOpenDevPath*

 3.3. setup **Xcode**

 * copy the [**.xcspec**] [1] files into */Applications/Xcode/Content/Developer/Platforms/[PLATFORM_PATH]/Developer/Library/Xcode/Specifications*
    
 * mkdir of */Applications/Xcode/Content/Developer/Platforms/[PLATFORM_PATH]/Developer/usr/bin*
	
 * `ln -fhs $iOSOpenDevPath/bin/iosod /Applications/Xcode/Content/Developer/Platforms/[PLATFORM_PATH]/Developer/usr/bin`
	
 * `ln -fhs $iOSOpenDevPath/bin/ldid /Applications/Xcode/Content/Developer/Platforms/[PLATFORM_PATH]/Developer/usr/bin`
	
 3.4. setup **SDK**
 
 * add the following key/value into **[SDK_PATH]/SDKSettings.plist**:

            DefaultProperties.CODE_SIGNING_REQUIRED => NO
            DefaultProperties.ENTITLEMENTS_REQUIRED => NO
            DefaultProperties.AD_HOC_CODE_SIGNING_ALLOWED => YES

 3.5. setup **templates**
 
 * `ln -fhs $iOSOpenDevPath/templates ~/Library/Developer/Xcode/Templates/iOSOpenDev`
 * open **TemplateInfo.plist** of **Base.xctemplate** and **Empty Project.xctemplate** with Xcode, then set the value of user defined key *[iOSOpenDevPath]* to *$HOME/Tools/iOS/iOSOpenDev*.


[1]: https://github.com/wzqcongcong/iOSOpenDev