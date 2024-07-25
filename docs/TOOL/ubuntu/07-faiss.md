
# faiss-c 安装

FAISS (Facebook AI Similarity Search) 是一个高效的相似性搜索库，广泛用于向量搜索和聚类任务。FAISS 提供了 C++ 和 Python 的接口。以下是如何在 C++ 项目中使用 FAISS 作为第三方库，并编译它的方法。

首先，你需要在你的系统中安装 FAISS。FAISS 可以通过源码编译安装，也可以通过包管理器（如 conda）安装。下面是从源码编译的步骤：

### 1. 克隆FAISS仓库

```bash
git clone https://github.com/facebookresearch/faiss.git
cd faiss

```

也可以从[https://github.com/facebookresearch/faiss/releases](https://github.com/facebookresearch/faiss/releases)下载release版本

### 2. 编译和安装

```bash
cmake -B build -DFAISS_ENABLE_GPU=OFF -DFAISS_ENABLE_PYTHON=OFF .
cmake --build build -j4
sudo cmake --install build
```
> 升级cmake
> [https://github.com/Kitware/CMake/releases](https://github.com/Kitware/CMake/releases)
> 下载sh版本，直接安装
> 临时用新的cmake
> `export PATH=/home/xujg/cmake-3.26.6-linux-aarch64/bin:$PATH`



