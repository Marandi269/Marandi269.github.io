# Python笔记 包

> https://python3-cookbook.readthedocs.io/zh_CN/latest/chapters/p10_modules_and_packages.html

## 10.3 使用相对路径名导入包中子模块

### 问题

将代码组织成包,想用import语句从另一个包名没有硬编码过的包中导入子模块。

### 解决方案

使用包的相对导入，使一个模块导入同一个包的另一个模块 举个例子，假设在你的文件系统上有mypackage包，组织如下：

```
mypackage/
    __init__.py
    A/
        __init__.py
        spam.py
        grok.py
    B/
        __init__.py
        bar.py
```

如果模块mypackage.A.spam要导入同目录下的模块grok，它应该包括的import语句如下：

```python
# mypackage/A/spam.py
from . import grok
```

如果模块mypackage.A.spam要导入不同目录下的模块B.bar，它应该使用的import语句如下：

```python
# mypackage/A/spam.py
from ..B import bar
```

两个import语句都没包含顶层包名，而是使用了spam.py的相对路径。



如果包的部分被作为脚本直接执行，那它们将不起作用 例如：

```bash
% python3 mypackage/A/spam.py # Relative imports fail
```

另一方面，如果你使用Python的-m选项来执行先前的脚本，相对导入将会正确运行。 例如：

```
% python3 -m mypackage.A.spam # Relative imports work
```

更多的包的相对导入的背景知识,请看 [PEP 328](http://www.python.org/dev/peps/pep-0328) .



## 10.7 运行目录或压缩文件

### 问题

您有一个已成长为包含多个文件的应用，它已远不再是一个简单的脚本，你想向用户提供一些简单的方法运行这个程序。

### 解决方案

如果你的应用程序已经有多个文件，你可以把你的应用程序放进它自己的目录并添加一个__main__.py文件。 举个例子，你可以像这样创建目录：

```
myapplication/
    spam.py
    bar.py
    grok.py
    __main__.py
```

如果__main__.py存在，你可以简单地在顶级目录运行Python解释器：

```
bash % python3 myapplication
```

解释器将执行__main__.py文件作为主程序。

如果你将你的代码打包成zip文件，这种技术同样也适用，举个例子：

```
bash % ls
spam.py bar.py grok.py __main__.py
bash % zip -r myapp.zip *.py
bash % python3 myapp.zip
... output from __main__.py ...
```

## 讨论

创建一个目录或zip文件并添加__main__.py文件来将一个更大的Python应用打包是可行的。这和作为标准库被安装到Python库的代码包是有一点区别的。相反，这只是让别人执行的代码包。

由于目录和zip文件与正常文件有一点不同，你可能还需要增加一个shell脚本，使执行更加容易。例如，如果代码文件名为myapp.zip，你可以创建这样一个顶级脚本：

```
#!/usr/bin/env python3 /usr/local/bin/myapp.zip
```



## 10.8 读取位于包中的数据文件

### 问题

你的包中包含代码需要去读取的数据文件。你需要尽可能地用最便捷的方式来做这件事。

### 解决方案

假设你的包中的文件组织成如下：

```
mypackage/
    __init__.py
    somedata.dat
    spam.py
```

现在假设spam.py文件需要读取somedata.dat文件中的内容。你可以用以下代码来完成：

```
# spam.py
import pkgutil
data = pkgutil.get_data(__package__, 'somedata.dat')
```

由此产生的变量是包含该文件的原始内容的字节字符串。



## 10.9 将文件夹加入到sys.path

### 问题

你无法导入你的Python代码因为它所在的目录不在sys.path里。你想将添加新目录到Python路径，但是不想硬链接到你的代码。

### 解决方案

有两种常用的方式将新目录添加到sys.path。第一种，你可以使用PYTHONPATH环境变量来添加。例如：

```bash
bash % env PYTHONPATH=/some/dir:/other/dir python3
Python 3.3.0 (default, Oct 4 2012, 10:17:33)
[GCC 4.2.1 (Apple Inc. build 5666) (dot 3)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> import sys
>>> sys.path
['', '/some/dir', '/other/dir', ...]
>>>
```

在自定义应用程序中，这样的环境变量可在程序启动时设置或通过shell脚本。

第二种方法是创建一个.pth文件，将目录列举出来，像这样：

```bash
# myapplication.pth
/some/dir
/other/dir
```

这个.pth文件需要放在某个Python的site-packages目录，通常位于/usr/local/lib/python3.3/site-packages 或者 ~/.local/lib/python3.3/sitepackages。当解释器启动时，.pth文件里列举出来的存在于文件系统的目录将被添加到sys.path。安装一个.pth文件可能需要管理员权限，如果它被添加到系统级的Python解释器。



## 不常用的特性

+ 通过字符串引包

  ```python
  import importlib
  math = importlib.import_module('math')
  math.sin(2)
  
  import importlib
  # Same as 'from . import b'
  b = importlib.import_module('.b', __package__)
  ```

+ 从远程服务器引包

  ```python
  from urllib.request import urlopen
  u = urlopen('http://localhost:15000/fib.py')
  data = u.read().decode('utf-8')
  print(data)
  """
  # fib.py
  print("I'm fib")
  
  def fib(n):
      if n < 2:
          return 1
      else:
          return fib(n-1) + fib(n-2)
          
  
  """
  
  
  import imp
  import urllib.request
  import sys
  
  def load_module(url):
      u = urllib.request.urlopen(url)
      source = u.read().decode('utf-8')
      mod = sys.modules.setdefault(url, imp.new_module(url))
      code = compile(source, url, 'exec')
      mod.__file__ = url
      mod.__package__ = ''
      exec(code, mod.__dict__)
      return mod
  
  fib = load_module('http://localhost:15000/fib.py')
  I'm fib
  >>> fib.fib(10)
  89
  >>> spam = load_module('http://localhost:15000/spam.py')
  I'm spam
  >>> spam.hello('Guido')
  Hello Guido
  >>> fib
  <module 'http://localhost:15000/fib.py' from 'http://localhost:15000/fib.py'>
  >>> spam
  <module 'http://localhost:15000/spam.py' from 'http://localhost:15000/spam.py'>
  >>>
  ```

  

+ 安装私有的包

  ```bash
  # ~/.local/lib/python3.3/site-packages
  python3 setup.py install --user
  pip install --user packagename
  ```

+ python 自带的虚拟环境工具

  ```bash
  python3 -m venv /path/to/new/virtual/environment
  ```

  





## 参考阅读

https://python3-cookbook.readthedocs.io/zh_CN/latest/c10/p11_load_modules_from_remote_machine_by_hooks.html



