---
title: python练习笔记
date: 2017-07-12
tags:
- python
categories:
- python
---

**获取当前路径**

```python
import os
print os.getcwd()

/Users/alan/Python Files/Untitled Folder
```

**计算机CPU数量**

```python
import multiprocessing
print multiprocessing.cpu_count()
```

**遍历文件夹的文档**

```python
from os import listdir
from os.path import isfile, join
files_list = [f for f in listdir('/home/students') if isfile(join('/home/students', f))]
print files_list
```

 **获取当前用户名**

```python
import getpass
print getpass.getuser()
```

**获取当前IP地址**

```python
import socket
print([l for l in ([ip for ip in socket.gethostbyname_ex(socket.gethostname())[2] 
if not ip.startswith("127.")][:1], [[(s.connect(('8.8.8.8', 53)), 
s.getsockname()[0], s.close()) for s in [socket.socket(socket.AF_INET, 
socket.SOCK_DGRAM)]][0][1]]) if l][0][0])
```

