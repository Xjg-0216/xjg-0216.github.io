<!--
 * @Descripttion: 
 * @Author: xujg
 * @version: 
 * @Date: 2024-07-03 10:13:49
 * @LastEditTime: 2024-07-04 00:32:33
-->
# 系统备份

Timeshift 是类似于 macOS「时间机器」的备份工具，它能备份整个系统，并提供文件备份选项和备份计划功能。

#### 备份
###### apt安装timeshift

```bash
sudo apt update
sudp apt install timeshift
```

打开 Timeshift，选择 RSYNC 或 BTRFS 快照类型。通常选择 RSYNC

选择备份存储位置，点击「下一步」

###### 创建

###### 恢复系统快照
启动 Timeshift：
```bash
sudo timeshift-launcher
```
#### 恢复
###### 选择快照：

在 Timeshift 界面中，选择您想要恢复的快照。

###### 恢复快照：
点击 “Restore” 按钮并按照提示进行恢复操作。
重新启动系统：
恢复完成后，重新启动系统。
##### 命令行操作
如果无法进入桌面环境，可以在终端中使用命令行恢复快照。

###### 列出快照：
```bash
sudo timeshift --list
```

###### 恢复快照：
使用命令行恢复特定的快照（假设要恢复到 2024-07-01 的快照）：
```bash
sudo timeshift --restore --snapshot "2024-07-01_12-00-00"
```