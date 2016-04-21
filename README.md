##Python 笔记六：入门爬虫坑--网页数据压缩

> 源码github地址在此，记得点星：
https://github.com/brandonxiang/get_html_deflate_gzip

****

运用urllib2做一个小型爬虫是相当简单的入门实验，但是会出现编码，压缩等问题。这一个坑很多人踩过，甚至有人处理编码问题，会出现一种情况，就是5分钟开发完成，25分钟处理编码问题。更不用说数据压缩，数据会面目全非。网页压缩主要两种，区别可参考[gzip和deflate的几点区别](http://www.webkaka.com/tutorial/server/2015/021013/)。

在这里用python举个栗子，小项目，用urllib2爬网页十分简单。

```
data = urllib2.urlopen(url).read()
```

在解决编码的问题上，网上有各种各样的解决方法。但是都没有很完美的解决方案。有些讲的是deflate，有的讲的是gzip。由于国内落后的网站还有很多，特别是一些政府企事业的网站。我希望能提供一种两全齐美的解决方案。

####deflate

```
import zlib
def deflate(data): 
	try:               
		return zlib.decompress(data, -zlib.MAX_WBITS)
	except zlib.error:
		return zlib.decompress(data)
```

####gzip

```
from gzip import GzipFile
from StringIO import StringIO
def gzip(data):
	buf = StringIO(data)
	f = gzip.GzipFile(fileobj=buf)
	return f.read()
```

####将两者结合写成一个整合方法

通过对`Content-Encoding`属性的判断，将两个方法结合在一起。

```
import urllib2
from gzip import GzipFile
from StringIO import StringIO
import zlib

def loadData(url):
	request = urllib2.Request(url)
	request.add_header('Accept-encoding', 'gzip,deflate')
	response = urllib2.urlopen(request)
	content = response.read()
	encoding = response.info().get('Content-Encoding')
	if encoding == 'gzip':
		content = gzip(content)
	elif encoding == 'deflate':
		content = deflate(content)
	return content

def gzip(data):
	buf = StringIO(data)
	f = gzip.GzipFile(fileobj=buf)
	return f.read()

def deflate(data):
	try:
		return zlib.decompress(data, -zlib.MAX_WBITS)
	except zlib.error:
		return zlib.decompress(data)

def main():
	url = "http://www.szxuexiao.com/"
	content = loadData(url)
	print content

if __name__ == '__main__':
	main()

```

转载，请表明出处。[总目录Awesome GIS](http://www.jianshu.com/p/3b3efa92dd6d)
