# 系统备份

> 本文所述的系统备份思路适用于类`unix`系统，相关示例以`gentoo`为平台运行。

## 备份动机

### 安装系统过程

-   磁盘或者其他块设备设置分区

    ```
    fdisk /dev/sda && ...
    ```

-   配置通用块层(可选)

    ```
    pvcreate /dev/sda[b]1 && ...
    ```

-   创建文件系统

    ```
    mkfs.ext2 /dev/sda2 && ...
    ```

-   复制系统文件快照

    ```
    tar xvjpf stage3-*.tar.bz2 --xattrs --numeric-owner && ...
    ```

-   获取最新更新

    ```
    emerge --sync && ...
    ```

-   自定义配置文件

    ```
    vim /etc/locale.gen && locale-gen && env-update && ...
    ```

-  安装必要软件

    ```
    emerge --ask sys-kernel/gentoo-sources && ...
    ```

-   自定义内核和`ramdisk`

    ```
    cd /usr/src/linux && make menuconfig && genkernel && ...
    ```

-   安装引导

    ```
    grub-install /dev/sda && grub-mkconfig -o /boot/grub/grub.cfg && ...
    ```

**过程分析:**

-   根据硬件环境和用户需求配置适合自己硬件分区方案(例:`BIOS`或者`GPT`)，配置通用
    块层(例:`lvm`)和文件系统(例: `ext4`或者`ntfs`)。很多针对块设备的优化都有特殊
    应有场景，过度使用高级特性，会增加系统操作难度和增加不必要性能消耗。

-   不同硬件设备需要不同的设备驱动，内核通过编译链接或者模块的方式加载驱动。自定
    义匹配最小化的驱动，可以避免无效驱动的带来性能消耗，提高硬件驱动效率。

-   操作系统为了更加通用，很多选择以配置文件提供给用户选择(例：国际化和多时区)。

-   最小化的软件安装，可以避免过多内存占用和系统稳定性。(例：服务端软件都是通过`
    ssh`进行维护，通常不需要一个图形化，`Xorg`不仅会增加性能的消耗，也会降低系统
    稳定性)。

> 安装系统的过程虽然繁琐耗时，但是为了更好的贴合硬件和用户需求，上述核心步骤是难
> 以省略的。

### 备份可行性

#### 重新安装分析

-   **重新安装优点:**

    当运行中系统遭遇误操作等情况时，旧系统难以启动或者是错误难以排查的时候。重新
    安装系统可以避免陷在旧系统的复杂问题中。

-   **重新安装的缺点：**

    重装通常需要格式化块设备，以达到重置环境的作用。这个时候旧系统的数据将会丢失
    ,而恢复配置完备的环境和各类用户数据还需要较多时间。

#### 备份原理

-   一旦系统安装配置完备之后，此时的系统快照较之默认系统快照(`stage-3`)相比既贴
    合设备硬件，也符合用户的应用场景。同时也保留了当下各种系统文件和用户数据。

## 全量备份

### 备份思路

全量备份时间最好为需求功能确定，初次安装并配置完成系统之后。此时通过`tar`命令打
包压缩当前系统全量快照。

```
tar --exclude-from=$exclude_file --xattrs -czpvf $backupfile /
```

> 安装操作系统实质是通过计算把数据写入到块设备完成持久化。而`unix`系统过`VFS`规
> 范把不同的块设备抽象为文件，通过格式化和挂在成为目录使用。通过相同的系统调用
> (例:`read`)进行操作。`tar`的备份就是基于文件级别的备份。`unix`系统中的七种类型
> :`d`(目录文件)，`l`(符号链接)，`s`(套接字)，`b`(设备文件)，`p`(符号链接)，普通
> 文件遵循`POSIX`规范。而文件中字符和二进制数据也是遵循相应的编码规范(例:`utf-8
> `)。基于文件的备份可以有更好移植性。

[全量备份脚本](https://raw.githubusercontent.com/crux-wild/system-backup/master/full-system-backup)

### 还原思路

```
tar --xattrs -xpf $backupfile
```

## 差量备份

## 相关链接

