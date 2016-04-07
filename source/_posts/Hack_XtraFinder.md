title: Hack XtraFinder
date: 2016-01-12 23:38:55
categories: 开发
tags: [Mac]

---

事情是这样的，我的系统是 OS X 10.11，就是那个有 SIP 的 10.11 😂。然后我装了一个 Finder 的插件：XtraFinder，这个插件有些功能在 10.11 下是不 work 的，所以插件作者出于好心，在检测到你的系统是 10.11 且 SIP 没有关闭的情况下，会弹出一个 alert 窗口作为提醒。

<!--more-->

![Alert](/img/Hack_XtraFinder/alert.png)

那么问题来了，作者检测 SIP 是否关闭的方法有个 bug，导致我明明是完全关闭了 SIP，但插件却认为 SIP 还在启用。第二个问题，这个 alert 每次重启插件（包括每次重启系统）时都会弹出来啊，出来啊，来啊，啊，搞得你每次都要去点一下确认才能将其关闭，烦的很 😒。我曾经给作者发邮件，希望他能 update 一下，只弹一次 alert 就好。当然了，作者没有鸟我。

作为偶尔会用 Hopper Disassembler 偷窥一下别人代码的我，就开始意淫能不能 hack 一下这个插件，改掉这个恶心的行为。

### 工具

* Hopper Disassembler
* Hex Fiend
* Developer ID Application 证书

### 步骤

1. 用 Hopper Disassembler 定位 hack point

    这个 alert 是插件一启动的时候就弹出的，所以很容易想到先去 check 一下 `[AppDelegate applicationDidFinishLaunching:]` 这个方法。Load Mach-O 文件后，直奔那个函数的伪代码：
    
    ![Hopper Disassembler](/img/Hack_XtraFinder/hopper1.png)

    从图中可以看到，插件作者通过 `csrutil status` 的返回结果来判断 SIP 的状态，只有返回结果中匹配到了字符串 `"Debugging Restrictions: disabled"` 才认为 SIP 关闭。但是，我是完全关闭 SIP 的，运行 `csrutil status` 的结果是 `System Integrity Protection status: disabled.`，显然无法匹配那个字符串，所以就被误认为是 SIP 尚未关闭。
    
    注意，上图中蓝色框中的内容是我修改之后的，原内容是 `if (rax != rcx) goto loc_100001df7;`，也就是当 `[rax rangeOfString:rdx] != NSNotFound` 的时候，字符串匹配成功，就跳过不弹 alert。看到这里你已经知道了，只要把条件反转就大功告成了。
    
    当然，除了反转判断条件之外，还可以修改那个用于匹配的字符串，比如修改成 `"disabled"` 就好了，都能够匹配成功。
    
    好了，伪代码是看明白了，下面就得看如何让这个判断条件反转了。
    
    ![Hopper Disassembler](/img/Hack_XtraFinder/hopper2.png)
    
    上图中红色框中的汇编代码对应前面分析的那块伪代码，而蓝色框中的十六进制 `0F 84 E1 01 00 00` 代表伪代码块中的最后那条命令 `0x0000000100001c10         je         0x100001df7`。`je` 这个指令表示 *`如果等于则跳转到`*，它的反指令是 `jne`，所以我们要做的就是把最后这条命令改为 `jne         0x100001df7`。参考一下[这篇文章] [1]，其实就是把 `0F 84 E1 01 00 00` 改为 `0F 85 E1 01 00 00`，把 4 改成 5。

2. 用 Hex Fiend 修改 Mach-O 文件

    明白了改什么，下面就是开始操作了。用 Hex Fiend 打开 Mach-O 文件，直接查找替换即可。
    
    ![Hex Fiend](/img/Hack_XtraFinder/hex1.png)
    
    只有一点需要注意：如果直接查找 `0F 84 E1 01 00 00` 的话，可能有多处，所以为了准确定位，我们可以连同下一条命令一起来查找，如上图，我们查找 `0F 84 E1 01 00 00 48 8B 3D 6B 14 01 00`，这样就会只找到唯一的一处。
    
    替换完成后保存即可。

3. 重新签名

    修改后的 Mach-O 文件是无法运行的，因为签名不对了。所以需要我们用自己的开发者证书重签一下：`codesign -f -s "XXX 证书的 Common Name" /Applications/XtraFinder.app/Contents/MacOS/XtraFinder`。
    
    大功告成，覆盖重启插件后，再也没有弹出 alert，XtraFinder 照常 work 😆。

[1]: http://neuzxy.blog.51cto.com/10270223/1716326