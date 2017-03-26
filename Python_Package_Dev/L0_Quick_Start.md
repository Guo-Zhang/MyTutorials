# Python库的开发：快速入门


本系列中，我们会介绍一个完整的Python库的发布流程。本篇文章首先简单介绍一下流程，给大家一个初步的印象。大体上，我们可以把一个Python库的发布过程分为三步：主体程序编写、编写setup.py文件和发布到PyPI上。

本文环境：
- 操作系统：Mac OS 10.12.3
- Python版本：2.7.13（尚未测试3.x是否可用）


## 主体程序

这个程序想实现的功能很简单，是删除某个目录下所有的tex临时文件，然后输出所有处理的文件和对应的文件夹。为了用着更方便，我试图打包成了命令行命令，方便使用。

### 开发功能

首先我编写了一个函数，实现删除tex临时文件的功能。这里的主要逻辑是，用扩展名是不是可能是临时文件的后缀+文件有没有同名tex文件判断是不是tex临时文件，如果是，则删除并输出信息。

```
# texcleaning.py
from __future__ import print_function

import os


def texcleaning(path):
    """
    Clean temporary files created by tex engines

    Parameters
    ----------
     path: the path of input dictionary

    """
    _delete = False

    for fname in os.listdir(path):
        if os.path.isdir(os.path.join(path, fname)):
            texcleaning(os.path.join(path, fname))

        name, ext = os.path.splitext(fname)
        latex_exts = [".aux",
                      ".fdb_latexmk",
                      ".fls",
                      ".gz",
                      ".log",
                      ".out",
                      ".nav",
                      ".snm",
                      ".toc",
                      ]
        if (ext in latex_exts) and \
                (name.replace(".synctex", "") + ".tex" in os.listdir(path)):
            os.remove(os.path.join(path, fname))
            _delete = True
            print("Deleted: %s" % (os.path.join(path, fname)))

    if _delete:
        print("All tex temporary files in %s cleaned." % (path))
```


### 命令行命令设计

这里我使用了标准库的optparse来生成命令行命令，main函数定义了解析命令行命令的方法。

```
# texcleaning.p

from optparse import OptionParser


def main():

    usage = "usage: %prog [options] arg1 arg2"
    parser = OptionParser(usage=usage, version="%prog 0.0.1.dev4")
    parser.add_option('-p', '--path', default=os.getcwd())
    (options, args) = parser.parse_args()

    texcleaning(options.path)
```


## 编写setup.py

setup文件是用来实现打包和安装功能的文件，非常重要。这里我使用了比distutils更为先进的setuptools编写setup文件。
```
# setup.py

from setuptools import setup, find_packages


setup(
    name='texcleaning',
    version='0.0.1.dev4',

    packages=find_packages(
        exclude=['tests', 'tests.*', '*.tests', '*.tests.*']),
    scripts=['texcleaning.py'],

    description='Clean temporary files by tex engines',
    author='Guo Zhang',
    author_email='zhangguo@stu.xmu.edu.cn',

    entry_points={
        'console_scripts': [
            'texcleaning=texcleaning:main',
        ],
    },
)

```
解释一下几个参数。“packages”和“scripts”是主体程序的文件夹和文件；entry_points是用来生成命令行工具或者GUI工具的（理论上是跨平台的），比如这里我生成了一个texcleaning的命令来代替texcleaning.py的main函数，安装成功以后就可以直接使用“texcleaning”命令，但是正是因为如此，这里可能需要请求管理员权限。


## 发布和升级Python库
### 发布

在工作目录下，首先生成PKG-INFO：

```
$ python setup.py egg_info
```

然后[上传PKG-INFO](https://pypi.python.org/pypi?%3Aaction=submit_form)到PyPI上，这样PyPI便登记了这个库的信息。


### 升级
PyPI上登记信息之后，或者后续需要升级，在工作目录下打包并发布即可。

打包之前，删除dist文件夹中的旧版本打包文件，然后生成新文件：
```
$ sudo python setup.py sdist
```
否则上传时会报错说旧版本已经上传过了。

然后把新版本的打包文件上传到PyPI上面：

```
$ twine upload dist/*
```
这里会要求输入PyPI的账号密码，把注册过的账号密码输入即可。现在我们可以根据[文档](https://github.com/Guo-Zhang/texcleaning)中的方法使用库了。


## 小结

到此为止，一个Python库的开发、打包、发布和升级的步骤就完成了。在后面的部分，我们会详细介绍更多关于Python库开发过程中的各个方面。

本文生成的库的地址是：[texcleaning 0.0.1.dev3](https://pypi.python.org/pypi/texcleaning)

源码地址是：[Guo-Zhang/texcleaning](https://github.com/Guo-Zhang/texcleaning)
