# -ReFS-C-
解决ReFS作为C盘文件系统找不到恢复分区的问题

这篇文章仅适用于已经长时间接触ReFS的Windows用户，如果你并不知道ReFS是什么，那么不建议继续阅读下面的内容。

在ReFS作为C盘文件系统的情况下，如果你没有提前创建恢复分区，将会遇到包括但不限于以下问题：

使用Windows恢复进行重置此电脑流程时，提示找不到恢复环境

Windows Defender 脱机扫描失效

频繁的系统更新失败（由ReFS导致的更新失败问题已被微软在最新的正式版和预览版本中修复）

由于ReFS文件系统无法压缩的特性，只能在硬盘格式化阶段为恢复分区预留空间，以便在后续手动建立恢复分区。截止这篇文章发布时，微软仍然没有在最新的预览版本中添加相关支持，Windows安装程序无法自动为ReFS格式下的磁盘自动建立恢复分区，即使已经预留了空间。

警告：对于想要安装Experimental Future Platforms（版本号29xxx +)的用户，你需要提前在UEFI中关闭安全启动，否则安装程序重启后将无法通过安全启动校验，并且对于UEFI的更改将无法被保存，如果你已经遇到了这种情况，请联系主板售后关闭安全启动即可正常进入系统。这个问题曾在2026年春季的Canary版本中出现，不确定后续版本是否仍然存在。


<img width="1888" height="1654" alt="FABE99D1244E7789EF06AB3FCF82C577" src="https://github.com/user-attachments/assets/4451c5d6-a63c-48b1-9624-81e890e97624" />

安装主界面
使用U盘等工具进入Windows原生安装界面，选择你需要作为C盘的硬盘（建议整盘方便操作）

<img width="1988" height="1750" alt="9F43050BFC20512860ACAEEFA2892195" src="https://github.com/user-attachments/assets/d2e88cdf-2e87-4aa4-9628-6165e754c716" />

删除分区
使用删除操作合并掉所有分区，然后点击创建分区

<img width="4076" height="1838" alt="C1A5F89D3C305A2D45C605955A3290A9" src="https://github.com/user-attachments/assets/c2c390d4-c921-4577-8bb5-70fd5e140282" />

创建分区
这个时候你会看到左侧的一串数字，你应当将其更改为【当前显示的数字】-200【引导分区】-16【保留分区】-1024或2048【恢复分区】

我建议有条件的用户使用2048MB（2GB）的恢复分区，因为现在的恢复分区使用量已接近1024MB(1GB)

举例：

显示为488384如果你需要1024MB恢复分区大小则应更改为488384-200-16-1024=487144

显示为488384如果你需要2048MB恢复分区大小则应更改为488384-200-16-2048=486120

点击确定后等待分区划分完成，Alt+F10组合键调出cmd

<img width="3320" height="1878" alt="82014FAB5BE3EA1D087AE39D56F3EB7D" src="https://github.com/user-attachments/assets/b84556f9-9235-4762-bba3-222752b495f5" />

cmd界面，在安装环境下使用Alt+F10调出
键入

    format C: /Q/FS:ReFS

然而，如果你已经有另外一块格式化完成的硬盘存在，这里应当键入

    format D: /Q/FS:ReFS

反回的确认信息中会提示需要格式化的分区文件系统类型为NTFS，即可键入

    Y

继续按提示可以直接回车，然后可以关闭cmd

<img width="2464" height="1766" alt="03A6DCB9FE03313047F326D356686B57" src="https://github.com/user-attachments/assets/05d2f613-5edf-4fa9-983a-6e91cf99d57b" />

刷新页面
可以通过刷新后分区的剩余空间减小来确认其已被格式化为ReFS

选中第3分区进行下一步的安装进程

OOBE期间请不要登录微软账户，企业版系统可以直接域加入，其他版本自行解决，如果登录了微软账户会导致BitLocker被自动激活。BitLocker的关闭是必须的，它会阻止恢复分区的创建。



关闭设备加密
进入系统后找到设置-隐私与安全-设备加密并将其关闭，如果遇到蓝屏重启请重复操作直到解密完成开关变为关闭。不建议在控制面板里操作，可能会导致系统假死


创建并格式化硬盘分区
搜索并找到【创建并格式化硬盘分区】，右键刚才预留的黑色区块新建简单卷-不分配盘符-默认格式化

右键win图标打开管理员cmd,键入

    diskpart

    list disk

如果你的C盘是disk 0则键入

    select disk 0

    list partition

    select partition 4

    set id=de94bba4-06d1-4d40-a16a-bfd50179d6ac

    gpt attributes=0x8000000000000001

    exit

    Reagentc /enable

这里提示恢复分区激活成功就大功告成了
