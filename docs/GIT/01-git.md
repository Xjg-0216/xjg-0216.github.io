

### 一、Git 基础概念

1. **Git 是什么？**
    
    - Git 是一个开源的分布式版本控制系统，它能够帮助开发者跟踪文件的变化，协同工作以及管理多个版本的代码。
2. **常见的术语**
    
    - **Repository (仓库)**：保存项目文件及版本历史记录的地方。
    - **Commit**：记录对文件的修改。
    - **Branch (分支)**：不同的工作线，可以在不同的分支上并行开发。
    - **Merge (合并)**：将两个分支的代码合并到一起。
    - **Clone (克隆)**：将远程仓库的内容复制到本地。
    - **Push**：将本地的更改推送到远程仓库。
    - **Pull**：从远程仓库拉取最新的更改。

### 二、安装 Git

1. **在 Linux 上安装 Git**
    
    ```bash
    sudo apt update
    sudo apt install git
    ```
    
2. **在 Windows 上安装 Git**
    
    - 访问 Git 官网：[https://git-scm.com/download/win](https://git-scm.com/download/win)，下载并安装。
3. **在 macOS 上安装 Git**
    
    ```bash
    brew install git
    ```
    

### 三、配置 Git

1. **设置用户名和邮箱** 配置 Git 的用户名和邮箱是必需的，这些信息会显示在你的每个提交记录中。
    
    ```bash
    git config --global user.name "Your Name"
    git config --global user.email "youremail@example.com"
    ```
    
2. **查看配置**
    
    ```bash
    git config --list
    ```
    
3. **更改配置** 可以通过 `--global` 参数设置全局配置，也可以在项目中使用本地配置：
    
    ```bash
    git config user.name "New Name"
    git config user.email "newemail@example.com"
    ```
    

### 四、创建和管理仓库

1. **初始化本地仓库** 如果你想在一个已有文件夹中使用 Git，首先需要初始化 Git 仓库：
    
    ```bash
    git init
    ```
    
2. **克隆远程仓库** 使用 `git clone` 命令从远程仓库创建一个本地副本：
    
    ```bash
    git clone https://github.com/username/repository.git
    ```
    
3. **查看仓库状态** 查看当前仓库的状态：
    
    ```bash
    git status
    ```
    

### 五、基本操作

1. **添加文件到暂存区** 将文件的修改添加到 Git 的暂存区：
    
    ```bash
    git add <file_name>    # 添加单个文件
    git add .              # 添加当前目录下所有更改过的文件
    ```
    
2. **提交更改** 提交已添加到暂存区的更改：
    
    ```bash
    git commit -m "Your commit message"
    ```
    
3. **查看提交历史** 查看提交的日志记录：
    
    ```bash
    git log
    ```
    
4. **查看文件的修改** 查看文件修改的详细内容：
    
    ```bash
    git diff
    ```
    
5. **撤销更改** 撤销暂存区和工作区的更改：
    
    ```bash
    git store # 修改了工作区，但还没有添加到暂存区（使用 `git add`），你可以使用 `git restore` 来恢复文件到最近一次提交的状态
    git reset # 撤销最近一次提交，丢弃暂存区的修改，保留在工作区
    git reset --soft HEAD~1 # 撤销最近一次提交，但保留修改在暂存区和工作区
    git reset --mixed HEAD~1 # 撤销最近一次提交，丢弃暂存区的修改，保留在工作区
	git reset --hard HEAD~1 # 完全撤销最近的提交并丢弃暂存区和工作区的修改
    ```
    
6. **撤销更改** 撤销推送的提交：
	```bash
	git reset --hard <commit_hash>   # 重置到指定提交
	git push origin <branch_name> --force   # 强制推送到远程仓库,强制推送（`--force`）会覆盖远程仓库的历史
    ```
    
1. **删除文件** 删除文件并提交删除操作：
    
    ```bash
    git rm <file_name>
    git commit -m "Remove file"
    ```
    

### 六、分支管理

1. **查看现有分支**
    
    ```bash
    git branch
    ```
    
2. **创建新分支** 创建一个新的分支并切换到该分支：
    
    ```bash
    git branch <branch_name>      # 创建分支
    git checkout <branch_name>    # 切换分支
    ```
    
3. **创建并切换到新分支** 创建并同时切换到新分支：
    
    ```bash
    git checkout -b <branch_name>
    ```
    如果远程分支而本地没有该分支，通过以下命令在本地创建`branch_name`分支，并追踪远程`origin/branch_name`
    ```bash
    git checkout -b <branch_name> origin/<branch_name>
    ```
    
4. **合并分支** 将当前分支与目标分支合并：
    
    ```bash
    git merge <branch_name>
    ```
    
5. **删除分支** 删除本地和远程的分支：
    
    ```bash
    git branch -d <branch_name>  # 删除本地分支
    git push origin --delete <branch_name>  # 删除远程分支
    ```
    

### 七、远程仓库操作

1. **查看远程仓库** 查看配置的远程仓库地址：
    
    ```bash
    git remote -v
    ```
    
2. **添加远程仓库** 将远程仓库添加到本地仓库：
    
    ```bash
    git remote add origin <remote_repository_url>
    ```
    
3. **推送更改** 将本地分支的更改推送到远程仓库：
    
    ```bash
    git push origin <branch_name>
    ```
    
4. **拉取远程更改** 从远程仓库拉取最新的更改：
    
    ```bash
    git pull origin <branch_name>
    ```
    
5. **从远程仓库获取最新的更改** 获取最新更改但不合并到本地分支：
    
    ```bash
    git fetch origin
    ```
6. **删除远程仓库** ：
    
    ```bash
    git remote remove origin
    ```
    


### 八、工作流

#### 1. **Git 常见工作流**

- **集中式工作流**：所有开发者将代码推送到一个公共仓库，每个人在自己的本地机器上工作。
- **功能分支工作流**：每个新功能都在一个独立的分支上开发，开发完成后合并回主分支。
- **Gitflow 工作流**：将开发分成多个阶段，如 `feature`、`develop`、`release` 和 `hotfix` 等。

#### 2. **Pull Request 流程**

- 1. 创建一个新的分支进行开发。
- 2. 完成开发后，提交更改并推送到远程分支。
- 3. 在 GitHub 等平台上创建 Pull Request（PR）。
- 4. 进行代码审查、测试等，最终合并到主分支。

### 九、Git 高级命令

1. **Git stash**
    
    - 将当前的更改暂时保存起来，便于切换分支等操作：
        
        ```bash
        git stash
        git stash pop   # 恢复更改
        ```
        
2. **Git rebase**
    
    - 将一个分支的更改合并到另一个分支，通常用于保持提交历史的整洁：
        
        ```bash
        git rebase <branch_name>
        ```
        
3. **Git cherry-pick**
    
    - 选择性地将某个提交应用到当前分支：
        
        ```bash
        git cherry-pick <commit_hash>
        ```
        
4. **Git tag**
    
    - 创建标签来标记重要的提交（如发布版本）：
        
        ```bash
        git tag -a v1.0 -m "Release version 1.0"
        ```
        

### 十、Git 配置和优化

1. **设置别名** 你可以为常用的 Git 命令创建别名，简化操作：
    
    ```bash
    git config --global alias.co checkout
    git config --global alias.br branch
    git config --global alias.ci commit
    ```
    
2. **忽略文件** 使用 `.gitignore` 文件来忽略某些不需要版本控制的文件：
    
    ```bash
    echo "*.log" >> .gitignore
    ```
    
