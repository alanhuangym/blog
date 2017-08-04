---
title: ES输入设置
date: 2017-08-03 10:31:08
tags:
- python
categories:
- python
---

# PDF

### 1.安装PDFMiner

从[官网](http://www.unixuser.org/~euske/python/pdfminer/)上下载源安装包。

通过命令行，运行安装安装包。(注意需要到解压后安装包的根目录)

```
$ python setup.py install
```

测试是否安装成功，可以紧接着运行以下的代码。

```
pdf2txt.py samples/simple1.pdf
```

如果能读取PDF内容则安装成功。

如果是中文、韩文、日文等特殊文字PDF，则需要额外安装特殊文字补充包。

紧接着上述的安装程序，也是在根目录下运行以下代码：

```
$ make cmap
...
$ python setup.py install
```

运行完这两行之后，则可以正常运行PDFMiner了。

```python
#!/usr/bin/python
#-*- coding: utf-8 -*-

from pdfminer.converter import PDFPageAggregator
from pdfminer.pdfparser import PDFParser
from pdfminer.pdfdocument import PDFDocument
from pdfminer.pdfpage import PDFPage
from pdfminer.pdfpage import PDFTextExtractionNotAllowed
from pdfminer.pdfinterp import PDFResourceManager
from pdfminer.pdfinterp import PDFPageInterpreter
from pdfminer.layout import *
import re

#打开一个pdf文件
fp = open(u'F:\\pdf\\2013\\000001_平安银行_2013年年度报告_2562.pdf', 'rb')
#创建一个PDF文档解析器对象
parser = PDFParser(fp)
#创建一个PDF文档对象存储文档结构
#提供密码初始化，没有就不用传该参数
#document = PDFDocument(parser, password)
document = PDFDocument(parser)
#检查文件是否允许文本提取
if not document.is_extractable:
    raise PDFTextExtractionNotAllowed
#创建一个PDF资源管理器对象来存储共享资源
#caching = False不缓存
rsrcmgr = PDFResourceManager(caching = False)
# 创建一个PDF设备对象
laparams = LAParams()
# 创建一个PDF页面聚合对象
device = PDFPageAggregator(rsrcmgr, laparams=laparams)
#创建一个PDF解析器对象
interpreter = PDFPageInterpreter(rsrcmgr, device)
#处理文档当中的每个页面

# doc.get_pages() 获取page列表
#for i, page in enumerate(document.get_pages()):
#PDFPage.create_pages(document) 获取page列表的另一种方式
replace=re.compile(r'\s+');
# 循环遍历列表，每次处理一个page的内容
for page in PDFPage.create_pages(document):
    interpreter.process_page(page)
    # 接受该页面的LTPage对象
    layout=device.get_result()
    # 这里layout是一个LTPage对象 里面存放着 这个page解析出的各种对象
    # 一般包括LTTextBox, LTFigure, LTImage, LTTextBoxHorizontal 等等
    for x in layout:
        #如果x是水平文本对象的话
        if(isinstance(x,LTTextBoxHorizontal)):
            text=re.sub(replace,'',x.get_text())
            if len(text)!=0:
                print text
```

# DOC&DOCX

pip install python-docx

安装第三方库[python-docx](https://python-docx.readthedocs.io/)

直接使用可以读取.docx文件

```python
from docx import Document
doc = Document(r'/Users/alan/Downloads/123.docx')
para = doc.paragraphs
for i in para:
    print i.text
```

.doc好像暂时还没方法读取



# PPTX

pip install python-pptx

安装第三方库[python-pptx](https://python-pptx.readthedocs.io/en/latest/index.html)

```python
from pptx import Presentation
ppt = Presentation('1.pptx')
text_runs = []

for slide in prs.slides:
    for shape in slide.shapes:
        if not shape.has_text_frame:
            continue
        for paragraph in shape.text_frame.paragraphs:
            for run in paragraph.runs:
                text_runs.append(run.text)
print text_runs
```





