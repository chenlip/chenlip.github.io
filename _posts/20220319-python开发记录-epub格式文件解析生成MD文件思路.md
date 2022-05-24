---
layout: post
title:  "python 开发学习记录"
date:   2022-03-18 22:27:00
categories: python
tags:  python pipenv halo
author: Hoeking
---

* content
{:toc}

# 20220318 python开发学习记录

还是环境配置的问题。

## 一、背景说明

1. 我在公司电脑上有一个python开发环境 Windows10 + python V3.8 + pipenv + vscode
2. 在公司电脑有一个项目源代码，其中的pipenv创建的虚拟环境就放在项目的根目录下，如下：

![image-20220318205831776](https://gitee.com/hoeking/sitepic/raw/master/blogpic/image-20220318205831776.png)

​		其中pipfile 文件内容如下：()

```
[[source]]
url = "https://pypi.tuna.tsinghua.edu.cn/simple"
verify_ssl = true
name = "custom"

[packages]
lxml = "*"
requests = "*"

[dev-packages]

[requires]
python_version = "3.8"
```



3. 该项目需在家里开发，但是家里的环境是 Windows10 + **python V3.9.4** + pipenv + vscode
3. 问题出现了，但这个项目复制到家里时，进行VSCode开发，发现很多包都找不到，需要从新配置虚拟环境哦。




## 二、实际操作

1. 我先将家里电脑环境中的pip换成国内固定源，（因为我一般不再家干活，我现在水平也不高，高级的库估计也用不上，用国内镜像应该足够了）

![image-20220318211437268](https://gitee.com/hoeking/sitepic/raw/master/blogpic/image-20220318211437268.png)

```
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
[install]
trusted-host=pypi.tuna.tsinghua.edu.cn
```

2. 
   使用pipenv创建虚拟环境，设置好虚拟环境的相关参数


​	我想以后估计还的用很多不同的项目虚拟环境，所以我准备单独设置一个固定的虚拟环境目录，而不用pipenv自动默认的虚拟环境目录：C:\Users\XXXXXX

​	我固定的虚拟环境目录：C:\ venvtemp \ .venv

​	**设置系统环境变量 `WORKON_HOME`**

​	**为了pipenv也使用国内的pip源，还需要设置环境变量：`PIPENV_TEST_INDEX`**

| 项目             | 键                               | 值                                         |
| ---------------- | -------------------------------- | ------------------------------------------ |
| **自动换源**     | `PIPENV_TEST_INDEX`              | `https://pypi.tuna.tsinghua.edu.cn/simple` |
|                  | PIPENV_PYPI_MIRROR(好像不起作用) | `https://pypi.tuna.tsinghua.edu.cn/simple` |
| **环境目录修改** | `WORKON_HOME`                    | `PIPENV_VENV_IN_PROJECT`                   |

![image-20220318213307681](https://gitee.com/hoeking/sitepic/raw/master/blogpic/image-20220318213307681.png)



通过以上设置后，成功完成了pipenv 虚拟环境的重置。

![image-20220318220041593](https://gitee.com/hoeking/sitepic/raw/master/blogpic/image-20220318220041593.png)



但是，并没有使用我设置的固定的目录（C:\venvtemp \ .venv）作为虚拟环境运行目标；

靠，原因是我刚刚设置后，没有重开一下CMD的终端窗口，重开后，就正常了。

3. 设置vscode开发环境

   先setting json；打开setting.json配置文件

   - Ctrl+Shift+P，输入settings，选择Open Settings(JSon)

   - 将之前得到的Pipenv环境路径添加进去

     “python.venvPath”: “C:\Users\Algorithm\.virtualenvs”

   

   ![image-20220318221807507](https://gitee.com/hoeking/sitepic/raw/master/blogpic/image-20220318221807507.png)

   重启Vscode ；可以选择 python:select interpreter.....

   ![image-20220318222021669](https://gitee.com/hoeking/sitepic/raw/master/blogpic/image-20220318222021669.png)

4. 完成

   

## 三、经验记录

   - pipenv 的命令行有点难记，主要还是我python的开发比较少啊
   - vscode的python配置每次都记不住
   - 

   





