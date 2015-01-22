title: How to get File Attributes in Mac
date: 2015-01-22 15:16:18
categories: 开发
tags: [Mac, File System]

---

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

