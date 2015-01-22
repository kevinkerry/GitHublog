title: How to get File Attributes in Mac
date: 2015-01-22 15:16:18
categories: 开发
tags: [Mac, File System]

---

##### Mac File Attributes

在 Mac 开发中，想要获取 file 或者 dir 的属性，有这么4种方式：

<!--more-->

* **Linux C api**

    `int lstat(const char *restrict path, struct stat *restrict buf)`

    获取到的 stat 有如下属性：

        struct stat { /* when _DARWIN_FEATURE_64_BIT_INODE is defined */
            dev_t           st_dev;           /* ID of device containing file */
            mode_t          st_mode;          /* Mode of file (see below) */
            nlink_t         st_nlink;         /* Number of hard links */
            ino_t           st_ino;           /* File serial number */
            uid_t           st_uid;           /* User ID of the file */
            gid_t           st_gid;           /* Group ID of the file */
            dev_t           st_rdev;          /* Device ID */
            struct timespec st_atimespec;     /* time of last access */
            struct timespec st_mtimespec;     /* time of last data modification */
            struct timespec st_ctimespec;     /* time of last status change */
            struct timespec st_birthtimespec; /* time of file creation(birth) */
            off_t           st_size;          /* file size, in bytes */
            blkcnt_t        st_blocks;        /* blocks allocated for file */
            blksize_t       st_blksize;       /* optimal blocksize for I/O */
            uint32_t        st_flags;         /* user defined flags for file */
            uint32_t        st_gen;           /* file generation number */
        };


* **Metadata**

    这种方式是基于 Spotlight 的，优点是在访问大量文件时速度快，缺点是要依赖 Spotlight 功能的开启。当 Spotlight 获取到 item 的 metadata 后，就可以通过下面2种方法来获取这些 metadata：

    * `CFTypeRef MDItemCopyAttribute(MDItemRef item, CFStringRef name)`，metadata attribute key 格式为 kMDItemXXX，详见文档。

    * `NSMetadataQuery`，通过执行 Spotlight 查询来获取 result，metadata attribute key 格式同上。

    **注意**，这种方式是基于 Spotlight 获取到了这些 metadata，否则拿到的值可能就是 NULL 了。

    **另外**，在 terminal 中，这种方式也有相应的命令 `mdls` 可以使用。


* **NSFileManager**

    通过调用 NSFileManager 的 `attributesOfItemAtPath:error:` 也可以拿到 file 的 attribute，attribute key 格式为 `NSFileXXX`，详见文档。


* **NSURL**

    通过调用 NSURL 的 `getResourceValue:forKey:error:` 也可以拿到 file 的 attribute，resource key 格式为 `NSURLXXX`，详见文档。对比采用 NSFileManager，这种方式以 URL 的形式来访问该 file，拿到的 attribute 比 NSFileManager 也多。**推荐使用。**



##### 这里顺便提一下关于 delete file/dir 的一点注意事项

首先，NSFileManager 有一个判断 file/dir 能否删除的 api：`isDeletableFileAtPath:`，其次，还有一个负责删除的 api：`removeItemAtPath:error:`。

那么问题来了，`isDeletableFileAtPath:` 和 `removeItemAtPath:error:` 的行为不太一致！

`removeItemAtPath:error:` 总是能够做出正确的行为，即能删除的话就删除，不能删除的话就不会删除。但是，`isDeletableFileAtPath:` 有时候不能做出正确的判断。

* 对于 file 来说，`isDeletableFileAtPath:` 总是可以做出正确的判断，这个没问题。

    也就是说，根据 Apple 文档，当 file 的 parent dir 具有可写权限时，file 就是 deletable 的，不管 file 本身的权限如何。否则，就是 undeletable 的。

* 对于 dir 来说，就会出现刚才说的问题。

    理论上，根据 Apple 文档，除了判断 parent dir 的可写权限之外，`isDeletableFileAtPath:` 还会递归地去判断 child item，只有所有的 child item 都是 deletable 时，才会返回 YES。

    但是，事实上，`isDeletableFileAtPath:` 貌似没有去递归地检查 child item。

    举例说明，有这么一个 path：<user_dir>/<root_dir>/<file_no_matter_root_or_user>，path 中的名称标明了对应的权限。现在用 `isDeletableFileAtPath:` 去检查 <root_dir>。

    理论上，<root_dir> 中有个 <file_no_matter_root_or_user>，而 <file_no_matter_root_or_user> 是 undeletable 的（因为它的 parent dir <root_dir> 是 root 的，普通用户没有可写权限），所以 <root_dir> 就是 undeletable 的，所以应该返回 NO。但是 `isDeletableFileAtPath:` 返回的却是 YES。此时如果调用 `removeItemAtPath:error:` 去删除 <root_dir> 时，就会出现 error，提示没有权限。也就是说，`removeItemAtPath:error:` 跟实际情况是一致的，但 `isDeletableFileAtPath:` 却做出了错误的判断。所以对于这种情况，你需要手动地去递归调用 `isDeletableFileAtPath:` 来检查 <root_dir>。

    感觉好坑啊，难道打开的姿势不对~~

