
# tar/zip

### tar打包
使用 `tar` 打包文件，并==忽略==特定文件或文件夹：

```bash
tar --exclude='./path/to/excluded_file_or_directory' -czf archive.tar.gz /path/to/source_directory
```

>--exclude-from='exclude-list.txt' 指定了包含忽略规则的文件。
-c 创建新的归档文件。
-z 使用 gzip 压缩。
-v 显示详细的操作过程。
-f 指定归档文件的名称。

例如，如果想忽略 `source_directory` 文件夹中的 `exclude_folder` 和 `exclude_file.txt`，可以这样写：
```bash
tar --exclude='./source_directory/exclude_folder' --exclude='./source_directory/exclude_file.txt' -czf archive.tar.gz -C /path/to/source_directory .
```

如果要忽略很多文件，使用 `tar` 的 `--exclude` 选项会变得不太方便。这种情况下，可以使用一个包含需要排除文件列表的文件。可以通过 `--exclude-from` 选项指定这个文件。
以下是具体步骤：

1. 创建一个包含要排除的文件或文件夹列表的文件，例如 `exclude.txt`：
```text
*.log
temp/
path/to/ignore/
another-file-to-ignore.txt
```

2. 使用 `tar` 命令打包文件，并通过 `--exclude-from` 选项排除文件列表中的文件或文件夹：
```bash
tar --exclude-from=exclude.txt -czf archive.tar.gz -C /path/to/source_directory .
```

### zip打包

##### 创建压缩包并包含所有子目录

```bash
zip -r archive.zip mydir
```

##### 创建压缩包并排除特定文件

```bash
zip -r archive.zip mydir -x '*.tmp' -x 'backup/*'
```
##### 更新压缩包中的文件

```bash
zip -u archive.zip updatedfile.txt
```
##### 从压缩包中删除文件

```bash
zip -d archive.zip oldfile.txt
```
##### 加密压缩包
```bash
zip -e archive.zip file.txt
```


#### zip使用排除列表文件忽略打包内容

如果需要忽略的文件和目录很多，或者忽略规则较复杂，可以使用一个排除列表文件的方法：

##### 创建一个排除列表文件：

创建一个文件（例如 `exclude-list.txt`），在文件中列出每一行需要忽略的文件或目录。例如：

```text
*.log
temp/*
path/to/ignore/*
another-file-to-ignore.txt
```
使用 `-x` 选项：

使用 `zip` 命令并逐一传递 `-x` 选项，或者通过 `shell` 的参数展开来指定这些排除规则：

```bash
zip -r archive.zip mydir $(cat exclude-list.txt | sed 's/^/-x /')
```
这里使用 `sed` 将每行前添加 `-x` ，以便 `zip` 命令能正确识别和应用这些排除规则。