# RecoverFromRecoveryMode
Factory reset from RecoveryMode without enter into Normal System

## 需求分析
  刷新手机系统，目前主要是线刷，即通过PC以及PC端的flashtool软件完成烧录，或者通过进入Android系统，设置里面进行出厂恢复操作。
  但是，如果手机无法进入Normal模式且没有合适的flashtool软件，如果恢复手机系统到出厂状态。

## 基于某平台手持设备的启动流程
   BIOS->OS Loader(kernelflinger)->boot.img(Linux+Ramdisk)->system.img(Android)

## 开发问题
###类似PC上的一键还原，将恢复系统预置到隐藏分区
    不需联网
    占用空间大（>2G）
###利用recovery中机制，使用PC配合恢复
    工作量小
    依赖PC，界面不友好
###扩展recovery，增加网络连接及恢复包下载，恢复等功能
    占用空间小
    开发量大，界面不友好
###压缩system，打包到一个bootimage中*
    界面与标准Android一致，空间占用中等
    
## 涉及的修改
###OS Loader
    添加启动项

###Image的生成
    抽取出必要的文件
      从user版本还是eng版本抽？
      预抽取还是每次编译抽取？
      如何抽取？
    制作image
      参考了recovery image的生成方式
###一键还原系统的启动顺序
    BIOS->OS Loader(kernelflinger)->onekeyrecovery.img(Linux+Ramdisk[including basic system])
    
## Linux内核代码修改
    build		--制作恢复的image
    hardware/intel/kernelflinger	--添加启动项及提示信息
    device/intel/build		--整合编译
    device/intel/cherrytrail		--添加分区及刷机脚本
    device/lenovo/common		--添加了预抽取的文件
    system/extras			--boot_signer签名问题，Java –Xms
    frameworks/base		--修改关机菜单

## 问题和解决
### 如何找出system分区中哪些文件被使用过？
    kernel中有个fsnotify
    借用了此机制
    当只读ext4文件系统中的文件被打开时，将此文件的inod号记录下来
    然后根据inod号去system中提取出对应的文件
    局限性
    kernel代码中使用filp_open()的文件不被notify，需单独添加。如wifi的FW：



 
