---
分类:
aliases: []
title: Anaconda新手安装+配置+环境创建教程
date created: 2026-03-02
date modified: 2026-03-02
tags:
  - python
  - anaconda
  - 开发环境
  - 教程
source: https://blog.csdn.net/qq_44000789/article/details/142214660
publish: false
---

# Anaconda新手安装+配置+环境创建教程

## 一、Anaconda简介

[[Anaconda]] 是一个开源的 Python 和 R 语言的发行版本,主要用于数据科学、机器学习和科学计算。

### 主要特点

1. **包管理**
   - 使用 `conda` 作为包管理工具
   - 支持多种操作系统(Windows、macOS、Linux)

2. **环境管理**
   - 创建独立的 Python 环境
   - 每个环境可有不同的 Python 版本和包依赖
   - 使用 `conda env` 命令管理

3. **集成开发环境**
   - Jupyter Notebook 和 JupyterLab
   - Spyder IDE

4. **预装包**
   - NumPy、Pandas、Matplotlib、SciPy、Scikit-learn
   - TensorFlow、PyTorch 等

5. **社区支持**
   - 活跃的社区和丰富的学习资源

---

## 二、Anaconda下载

> [!tip] 下载推荐
> 推荐使用清华镜像源下载,速度更快

### 方式一:官网下载(不推荐)

- 官网地址: [Anaconda官网](https://www.anaconda.com/)
- 缺点:国内网络下载速度慢

### 方式二:清华镜像源下载(推荐)

- 镜像地址: [Anaconda清华镜像源](https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/)
- 选择对应系统和版本下载
- 示例: `Anaconda3-2024.06-1-windows-x86_64.exe`

---

## 三、安装步骤

> [!warning] 重要提示
> - 选择 **All Users** 安装(避免权限问题)
> - 建议安装到非系统盘(如D盘、F盘)

### 安装流程

1. **启动安装**:双击下载的 `.exe` 文件
2. **同意协议**:点击 "I Agree"
3. **选择用户**:选择 "All Users"(重要!)
4. **选择路径**:建议修改为非C盘
5. **高级选项**:
   - ✅ 创建开始菜单
   - ✅ base环境以Python 3.12创建
   - ✅ 清除包缓存
6. **完成安装**:取消最后两个勾选框

---

## 四、环境变量配置

### 添加环境变量

> [!note] 路径说明
> 以下路径需要根据实际安装位置调整,示例为D盘安装

在系统环境变量 `Path` 中添加:

```shell
D:\anaconda3
D:\anaconda3\Scripts
D:\anaconda3\Library\bin
D:\anaconda3\Library\mingw-w64\bin
```

### 验证安装

以**管理员身份**打开 Anaconda Prompt,执行:

```bash
conda --version
```

显示版本号即表示安装成功。

---

## 五、默认路径和镜像源配置

### 修改环境保存路径

> [!tip] 为什么要修改?
> 默认环境和包保存在C盘,会占用系统盘空间

1. **查看当前配置**:

   ```bash
   conda info
   ```

2. **编辑配置文件**:
   - 位置:`C:\Users\用户名\.condarc`
   - 如果没有,执行:

	 ```bash
     conda config --set show_channel_urls yes
     ```

3. **修改 `.condarc` 文件内容**:

   ```yaml
   envs_dirs:
     - F:\Anaconda_envs\envs
   pkgs_dirs:
     - F:\Anaconda_envs\pkgs
   ```

### 配置国内镜像源

> [!info] 镜像源优势
> 加快包下载速度,避免国外网络延迟

在 Anaconda Prompt(管理员)中执行:

```bash
# 清华源
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/pytorch/

# 阿里云源
conda config --add channels https://mirrors.aliyun.com/anaconda/pkgs/free/
conda config --add channels https://mirrors.aliyun.com/anaconda/pkgs/main/

# 中科大源
conda config --add channels https://mirrors.ustc.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.ustc.edu.cn/anaconda/pkgs/main/
conda config --add channels https://mirrors.ustc.edu.cn/anaconda/cloud/conda-forge/

# 显示通道地址
conda config --set show_channel_urls yes
```

### 验证配置

```bash
conda info
```

检查:

- `envs directories` 和 `package cache` 路径是否已修改
- `channels` 中是否包含配置的镜像源

---

## 六、常用命令速查

### 环境管理

```bash
# 创建新环境
conda create --name myenv python=3.9

# 激活环境
conda activate myenv

# 退出环境
conda deactivate

# 查看所有环境
conda env list

# 删除环境
conda env remove --name myenv

# 导出环境
conda env export > environment.yml

# 导入环境
conda env create -f environment.yml
```

### 包管理

```bash
# 安装包
conda install numpy

# 安装指定版本
conda install numpy=1.21.0

# 更新包
conda update numpy

# 更新所有包
conda update --all

# 删除包
conda remove numpy

# 查看已安装包
conda list

# 搜索包
conda search numpy
```

---

## 相关链接

- [Anaconda官网](https://www.anaconda.com/)
- [Anaconda清华镜像源](https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/)
- [Conda官方文档](https://docs.conda.io/)

---

> [!quote] 原文来源
> 本笔记整理自: [最新版最详细Anaconda新手安装+配置+环境创建教程](https://blog.csdn.net/qq_44000789/article/details/142214660)
