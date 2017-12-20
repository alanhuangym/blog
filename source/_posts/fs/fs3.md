---
title: 文件系统3
date: 2017-09-13 14:23:06
tags:
- filesystem
categories:
- filesystem
---

### [Mongo Connector 设置](https://my.oschina.net/tianlele/blog/848860)

为ES添加pinyin搜索

google open search

[https://aaronparecki.com/20...](https://disq.us/url?url=https%3A%2F%2Faaronparecki.com%2F2011%2F07%2F11%2F3%2Fhow-to-let-google-power-opensearch-on-your-website%3AHsv8oJR1pwEePhD0jpqWGdgJoCU&cuid=3833100)



### TIKA读取附件

tika — help

**1.tika-python 调用**

[tika-python](https://github.com/chrismattmann/tika-python)是python的一个第三方库，可以快捷地调用tika很实用

安装库

```
pip install tika
```

第一次调用

```python
import tika
tika.initVM()
from tika import parser

parsed = parser.from_file(r'/Users/alan/Downloads/1/1.pptx')
print(parsed["metadata"])
print(parsed["content"])
```

会下载tika的整个java包，所以耗时较长，请耐心等待

下载完之后就可以直接使用，方便快捷

metadata包括以下数据：

```
height
tiff:ImageLength
Data PlanarConfiguration
resourceName
tiff:BitsPerSample
X-Parsed-By
Compression Lossless
width
Compression NumProgressiveScans
Transparency Alpha
Chroma ColorSpaceType
X-TIKA:parse_time_millis
Compression CompressionTypeName
Data BitsPerSample
IHDR
Data SampleFormat
Chroma BlackIsZero
Chroma NumChannels
Dimension ImageOrientation
embeddedRelationshipId
X-TIKA:embedded_resource_path
tiff:ImageWidth
Content-Type
Dimension PixelAspectRatio
```

!使用tika-python会打开一个tika服务器（post文件进去返回解析的html），内存占用较大



**2.命令行调用**

```
java -jar tika-app-1.16.jar --text doc
```

doc是需要解析的文件路径



为了在node.js中内部调用，我们使用了child_process库

```javascript
var exec = require('child_process').exec;

exec(('java -jar tika-app-1.16.jar --text '+'/Users/alan/Downloads/1/bb.pdf'),function(error,stdout,stderr){
	if (error !== null){
		console.error('error');
	};
	console.log(stdout);
});
```



**3.服务器模式调用**

命令行启动服务器模式

```
java -jar tika-app-1.16.jar --server --port 12345
```

这是简单的服务器模式启动，仅供测试环境使用

更完善的服务器模式tika看https://wiki.apache.org/tika/TikaJAXRS