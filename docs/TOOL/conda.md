# Conda 命令


### 基础命令
#### 1.查看Conda版本

```bash
conda --version
```

#### 2.更新Conda

```bash
conda update conda
```

#### 3.获取Conda帮助

```bash
conda --help
```
### 环境管理

#### 1.创建新环境


创建新环境并指定Python版本

```bash
conda create --name myenv python=3.8
```
#### 2.克隆环境

```bash
conda create --name myenv_clone --clone myenv
```

#### 3.激活环境

```bash
conda activate myenv
```

#### 4.停用环境

```bash
conda deactivate
```

#### 5.列出所有环境
```bash
conda env list
# 或
conda info --envs
```

#### 6.删除环境
```bash
conda remove --name myenv --all
```

### 包管理
#### 1.安装包

```bash
conda install numpy
```

#### 2.安装特定版本的包

```bash
conda install numpy=1.18.1
```
#### 3.从特定的频道安装包

```bash
conda install -c conda-forge numpy
```

#### 4.更新包

```bash
conda update numpy
```

#### 5.更新所有包
```bash
conda update --all
```

#### 6.删除包

```bash
conda remove numpy
```

#### 7.强制删除包

```bash
conda remove numpy --force
```

#### 8.查看已安装的包

```bash
conda list
```

#### 9.查看某个包的详细信息及依赖

```bash
conda info numpy
```

### 环境导出与导入

#### 1.导出环境到YAML文件

```bash
conda env export > environment.yml
```

#### 2.从YAML文件创建环境

```bash
conda env create -f environment.yml
```

#### 3.导出环境到简单列表

```bash
conda list --export > requirements.txt
```
#### 4.从简单列表创建环境

```bash
conda create --name myenv --file requirements.txt
```

### 清理缓存
#### 1.清理已下载的包文件

```bash
conda clean --packages
```

#### 2.清理所有缓存

```bash
conda clean --all
```


### Conda 配置管理

Conda频道是包集合的源，Conda通过这些渠道来查找、下载和安装软件包。不同的频道可能包含不同版本的软件包，或者有不同的维护和更新策略。

>常见的Conda频道
**默认频道（defaults）**:  
这是Conda的官方频道，默认情况下所有Conda安装都使用这个频道。它包含了大量常用的包，维护和更新由Anaconda团队负责。
URL: [https://repo.anaconda.com/pkgs/main/](https://repo.anaconda.com/pkgs/main/)  
**conda-forge**:  
这是一个社区驱动的频道，包含了更多包和更新版本，通常比默认频道更新更快。  
URL: [https://conda-forge.org/](https://conda-forge.org/)  
**r**:  
专门提供 R 语言相关的包。  
URL: https://repo.anaconda.com/pkgs/r/  
**free**:  
包含了所有许可证下的自由和开放源码软件包，这是默认频道的一部分，但在Anaconda 5.0之后被合并到了 main 频道中。  
URL: https://repo.anaconda.com/pkgs/free/  

#### 1.添加频道

```bash
conda config --add channels conda-forge
```

#### 2.添加频道并置于最高优先级
```bash
conda config --add channels conda-forge --prepend
```
#### 3.添加其他频道
```bash
conda config --add channels defaults
```
#### 4.移除频道

```bash
conda config --remove channels conda-forge
```

#### 5.删除所有频道

```bash
conda config --remove-key channels
```
#### 6.验证频道
```bash
conda config --get channels
```

#### 7.设置HTTP代理

```bash
conda config --set proxy_servers.http http://proxy_address:port
```
#### 8.设置HTTPS代理

```bash
conda config --set proxy_servers.https https://proxy_address:port
```

#### 9.设置环境的默认目录

```bash
conda config --set envs_dirs /path/to/envs
```

#### 10.启用详细输出

```bash
conda config --set verbosity 2
```

#### 11.禁用包版本检查

```bash
conda config --set auto_update_conda false
```

#### 12.列出所有配置

```bash
conda config --show
```
#### 13.查看包信息

```bash
conda search numpy --info
```

#### 14.查看可用包

```bash
conda search numpy
```

