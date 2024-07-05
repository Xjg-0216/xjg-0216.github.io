<!--
 * @Descripttion: 
 * @Author: xujg
 * @version: 
 * @Date: 2024-07-03 23:44:51
 * @LastEditTime: 2024-07-04 00:29:50
-->
# apt

在 `Linux `中，`apt` 是一个非常常用的命令行工具，用于管理软件包。`apt` 代表 `Advanced Package Tool`，是一个包管理器。以下是 `apt` 命令的一些基本用法和常用选项的详细解释：
#### 1. 更新包列表
在安装或更新软件包之前，首先需要更新包列表以确保获取最新的软件包信息。

```bash
sudo apt update
```
该命令从软件源下载最新的包列表，但并不会安装或更新任何软件包。

#### 2. 升级系统中的所有软件包
```bash
sudo apt upgrade
```
该命令会安装所有已经安装的软件包的最新版本，但不会删除或安装任何新的包。

#### 3. 升级系统中的所有软件包（包括删除和安装新的包）
```bash
sudo apt full-upgrade
```
这个命令类似于 `upgrade`，但它会处理包之间的依赖关系，并允许删除现有的包以便进行系统的全面升级。

#### 4. 安装软件包
```bash
sudo apt install <package_name>
```
这个命令用于安装指定的软件包
#### 5. 删除软件包
```bash
sudo apt remove <package_name>
```
这个命令用于删除指定的软件包，但会保留配置文件。
#### 6. 完全删除软件包（包括配置文件）
```bash
sudo apt purge <package_name>
```
这个命令不仅会删除软件包，还会删除相关的配置文件。
#### 7. 搜索软件包
```bash
apt search <package_name>
```
这个命令用于搜索软件包。
#### 8. 获取软件包的信息
```bash
apt show <package_name>
```
这个命令用于显示有关指定软件包的详细信息。

#### 9. 清理不需要的包
```bash
sudo apt autoremove
```
这个命令会自动删除那些作为依赖安装但现在不再需要的包。

#### 10. 清理已下载的包文件
```bash
sudo apt clean
```
这个命令会清理本地缓存的包文件以释放磁盘空间。

#### 11. 列出已安装的软件包
```bash
apt list --installed
```
这个命令会列出系统中所有已安装的软件包。

# apt-get
`apt` 和 `apt-get` 都是 `Ubuntu` 系统中用于管理软件包的命令行工具。`apt` 是相对较新的工具，目的是简化和统一 `apt-get` 和 `apt-cache` 的功能，以提供一个更用户友好的命令集合。尽管 `apt `和 `apt-get` 功能上有很多重叠，但它们的使用场景和命令略有不同。

#### apt 和 apt-get 的关系
`apt`：是 `apt-get` 和 `apt-cache` 的更高级的替代品，提供了一个更简单和更统一的接口，适用于大多数常见的包管理任务。
`apt-get`：是一个功能强大的工具，适合高级用户和脚本编写，提供了更多的选项和更精细的控制。
### `apt-get` 的详细用法
#### 1. 更新包列表
```bash
sudo apt-get update
```
这个命令会从软件源下载最新的包列表。

#### 2. 升级系统中的所有软件包
```bash
sudo apt-get upgrade
```
这个命令会升级所有已经安装的软件包，但不会删除或安装任何新的包。

#### 3. 升级系统中的所有软件包（包括删除和安装新的包）
```bash
sudo apt-get dist-upgrade
```
这个命令类似于 upgrade，但它会处理包之间的依赖关系，并允许删除现有的包以便进行系统的全面升级。

#### 4. 安装软件包
```bash
sudo apt-get install <package_name>
```
这个命令用于安装指定的软件包
#### 5. 删除软件包
```bash
sudo apt-get remove <package_name>
```
这个命令用于删除指定的软件包，但会保留配置文件。
#### 6. 完全删除软件包（包括配置文件）
```bash
sudo apt-get purge <package_name>
```
这个命令不仅会删除软件包，还会删除相关的配置文件。
#### 7. 搜索软件包
```bash
apt-cache search <package_name>
```
这个命令用于搜索软件包。
#### 8. 获取软件包的信息
```bash
apt-cache show <package_name>
```
这个命令用于显示有关指定软件包的详细信息。
#### 9. 清理不需要的包
```bash
sudo apt-get autoremove
```
这个命令会自动删除那些作为依赖安装但现在不再需要的包。

#### 10.  清理已下载的包文件
```bash
sudo apt-get clean
```
这个命令会清理本地缓存的包文件以释放磁盘空间。

#### 11.  列出已安装的软件包
```bash
dpkg --list
```
这个命令会列出系统中所有已安装的软件包。


以下是一些 apt 和 apt-get 常见命令的对比：
| 功能               | apt 命令                   | apt-get 命令                 |
|--------------------|----------------------------|------------------------------|
| 更新包列表         | sudo apt update            | sudo apt-get update          |
| 升级所有包         | sudo apt upgrade           | sudo apt-get upgrade         |
| 完全升级           | sudo apt full-upgrade      | sudo apt-get dist-upgrade    |
| 安装包             | sudo apt install <package_name> | sudo apt-get install <package_name> |
| 删除包             | sudo apt remove <package_name>  | sudo apt-get remove <package_name>  |
| 完全删除包         | sudo apt purge <package_name>   | sudo apt-get purge <package_name>   |
| 清理不需要的包     | sudo apt autoremove        | sudo apt-get autoremove      |
| 清理已下载的包文件 | sudo apt clean             | sudo apt-get clean           |

