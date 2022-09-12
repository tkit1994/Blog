---
title: Windows下载关联python
date: 2018-05-01 18:30:37
taxonomies:
  tags: 
    - python
---

## 安装anaconda

### 下载anaconda

可以从[清华大学开源镜像站](https://mirrors.tuna.tsinghua.edu.cn/#)下载, 比从官方下载要快上很多。

```bash
wget https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/Anaconda3-5.1.0-Windows-x86_64.exe
```

运行下载的文件完成安装。

## 设置环境变量

- 右键`此电脑`，选择`属性`, `高级系统设置`, `环境变量`。
- 新建立如下环境变量`CONDA_HOME`为anaconda的安装目录。
- 编辑环境变量`path`, 加入`%CONDA_HOME%\`和`%CONDA_HOME%\Scripts`。

![环境变量](https://s2.loli.net/2022/09/12/JXDeZ6pd1gSTHzR.png)

![PATH](https://s2.loli.net/2022/09/12/Qw9hqBC1jdXtkuW.png)

## 下载winpython

- 从github上下载[Winpython](https://github.com/winpython/winpython)
- 修改winpython/associate.py
- 注释`unregister(sys.prefix)`这一行
- 运行`python -m winpython.associate`

```bash
git clone https://github.com/winpython/winpython.git
cd winpython
sed -i 's/unregister(sys.prefix)/#register(sys.prefix)/g' winpython/associate.py
python -m winpython.associate
```

![修改associate.py](https://s2.loli.net/2022/09/12/bx65CPwqydKtnEl.png)

## 修改注册表

打开注册表编辑器，查找`python.File`，修改

`计算机\HKEY_CURRENT_USER\Software\Classes\Python.File\shell\Edit with Spyder\command`

的值：

`"D:\ProgramData\Miniconda3\pythonw.exe" "D:\ProgramData\Miniconda3\Scripts\spyder" "%1"`

为：

`"D:\ProgramData\Miniconda3\pythonw.exe" "D:\ProgramData\Miniconda3\Scripts\spyder-script.py" "%1"`

![修改注册表](https://s2.loli.net/2022/09/12/A1Pe2E8kJfqsLmN.png)
