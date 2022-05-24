---
layout: post
title: "用python解析epub格式文件"
categories: 其他
date: 2022-03-20 21:30:00
tags: python,epub,xml
author: Hoeking
---

* content
{:toc}


![](http://minio.sc-model.cn:9000/sitepic/blogpic/image-20220320161636714.png)

# 用Python解析Epub格式文件

## 一、开发环境

### 1.1、系统环境

- windows 10 + python 3.9
- Vscode V1.65.2 
- vs插件：xmltool、Python

### 1.2、注意事项

-  配置好自己的vscode的代码片段

  - 安装好vscode的关键插件(xmltool、Python)
    - 设置好xmltool的Xpath的 快捷方式
  - 配置好vscode的python开发环境
  - 配置好python 的虚拟环境（我使用的pipenv）
      - 我的环境**vscode V1.65.2**
      - 请注意需要断点调试的话，一定要加配置："env": {"PYTHONPATH":"${workspaceRoot}"},
      - ![image-20220328211435362](http://minio.sc-model.cn:9000/mdimages/blog/2022/3/28/image-20220328211435362.png)

### 1.3、Python的包环境

 - 系统包

 - 第三方包，需要在虚拟环境中安装好

    | 包名    | 作用                                                       | 安装   |
    | ------- | ---------------------------------------------------------- | ------ |
    | yaml    | 应用或工具的配置文件信息读取                               | pipenv |
    | json    | json的处理包                                               |        |
    | zipfile | zip压缩文件的处理包                                        |        |
    | pathlib | python 3.x 的目录路径处理包，替代os.path                   |        |
    | lxml    | 解析xml文件，不过只能读取xml文件                           |        |
    | logging | 用于记录运行日志，同时也可以用于调试过程中的错误或数据检查 |        |
    | PIL     | 需要处理电子书的封面图像，可能需要缩略图                   |        |
    | Jinja2  | 用于根据模板生成文件的方法 （生成MD格式文件）              |        |
    
    其他包如果有变化，后期在更新说明。

### 1.4、应用程序目录结构

​    比较简单，这里就不贴了.......



## 二、主要知识点

### 2.1、学习使用yaml进行配置信息读取

​		yaml的配置......

​		读取yaml文件前需要安装pyyaml和导入yaml模块

>  pipenv install pyyaml

​		最新版本是6.0 ；PyYAML==6.0

​		**示例：**

​		（关于yaml的目录结构，在开发时需要注意，这里我就不细节展开说明目录的处理了，请自行思考）

​		《**yaml文件内容**》===============：
 ```
import json
from jinja2 import Template

temps = Template('haha namepad= {{ namepad }} !!!!--- strfn= {{ strfn }}')
dic_var = { "namepad" : "who123" , "strfn" : "ssshahah123" }
s = temps.render(dic_var)

print(s)
 ```

​		《**python读取yaml**》=============：

```
import yaml
import os

def getYamlData(ymal_fpath):
    '''
    打开yaml文件,读取并返回一个数据字典。
    '''
    fl = open(ymal_fpath,'r', encoding="utf-8")
    fl_data = fl.read()
    fl.close()

    yl_data = yaml.load(fl_data,Loader=yaml.FullLoader) #返回字典类型
    return yl_data
#getYamlData

def getYamlByKey(yaml_fpath,strkey):
    '''
    1-打开yaml文件,读取并返回一个数据字典。
    2-读取yaml文件中某个配置的值，并返回字符串。
    异常：
        如果Key输入的错误，或没有找到这个Key，则返回字符串 Key_Error
    '''
    yldict = getYamlData(yaml_fpath)
    try:
        return yldict[strkey]
    except KeyError:
        return "Key_Error"
        raise KeyError
    #try
#getYamlByKey


curpath = os.path.abspath("../conf") # 获取配置文件所在的目录
yl_path = os.path.join(curpath,"config.yaml")

if(os.path.isfile(yl_path)):
    yldict = getYamlData(yl_path)
    print(yldict)
    print(yldict['psw'])
    print(yldict['use1'])
    print(yldict['use1']['a'])

    print(getYamlByKey(yl_path,'psw'))
else:
    print("配置文件未找到，请确认配置文件是否真的存在。")
```

​		[python：yaml模块 - 简书 (jianshu.com)](https://www.jianshu.com/p/eaa1bf01b3a6)

​		[python配置yaml - 测试-安静 - 博客园 (cnblogs.com)](https://www.cnblogs.com/qican/p/11773967.html)

### 2.2、学习zip文件处理

​		zip文件处理.....

​		[Python中的zipfile模块使用实例 - 简书 (jianshu.com)](https://www.jianshu.com/p/b9da5fd2e5cf)

### 2.3、学习json的数据处理

​		[Python JSON | 菜鸟教程 (runoob.com)](https://www.runoob.com/python/python-json.html)

​		[Python3 JSON 数据解析 | 菜鸟教程 (runoob.com)](https://www.runoob.com/python3/python3-json.html)

### 2.4、学习path目录或文件的处理方式

​		[python路径操作新标准：pathlib 模块 - 和牛 - 博客园 (cnblogs.com)](https://www.cnblogs.com/heniu/p/12872604.html)

​		[Python文件操作，看这篇就足够 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/56909212?utm_source=wechat_session)

### 2.5、学习xml文件处理（主要是lxml读取）

​		学习URL地址：

​		<u>**问题1💀：**解析xml格式的文件时要注意名称空间的问题：</u>

​		示例代码：

```
	parser = etree.XMLParser(encoding = "utf-8")
    ht_str = etree.parse(opffile, parser=parser)
    print(ht_str)

    xpathquery_str = "//package/metadata/dc:publisher"
    q_result = ht_str.xpath(xpathquery_str)
```

​		我的XML文件内容：

<img src="http://minio.sc-model.cn:9000/sitepic/blogpic/image-20220321204737938.png" class="mdimg"/>


​		**报错：lxml.etree.XPathEvalError: Undefined namespace prefix**

​			修改上面的python代码最后一行：在运行就不报这个错误了。

```
q_result = ht_str.xpath(xpathquery_str, namespace={'opf': 'http://www.idpf.org/2007/opf'})
```

​		<u>**问题2💀：**namesapce字符拼写问题：</u>

​		🔥🔥🔥

​				特别注意：**namespace  和 namespaces 绝对不能错哦**  ；上面的代码是不会报错；但是会报一个另外的错误啊，这个另外错误花费了两个小时啊......

​				lxml.etree.XPathResultError: Unknown return type: dict

​		🔥🔥🔥

```
	xpathquery_str = "//dc:title/text()"
    q_result = ht_str.xpath(xpathquery_str, namespaces=NS)
```



​		<u>**问题3💀：**opf文件的xpath查询问题：</u>

​		这个问题难点是，中文的很多文章中并没有对解析xpath查询讲得更深入吧，（咱英语也不是很好啊，慢慢看能看懂的水平.....😕）, 我已经花了两个多小时了啊，代码是这样的：

```
	# XMLNS
    # Namespaces (Epub文件中Opf文件中定义的名称空间，用一个字典变量定义) 
    NS = { 'opf': 'http://www.idpf.org/2007/opf',
            'xsi': 'http://www.w3.org/2001/XMLSchema-instance',
            'dcterms': 'http://purl.org/dc/terms/',
            'ns0': 'http://www.idpf.org/2007/opf',
            'dc': 'http://purl.org/dc/elements/1.1/',
          }

    # etree.XMLParser(encoding = "utf-8")
    parser = etree.XMLParser(encoding = "utf-8")
    ht_str = etree.parse(opffile, parser=parser)
    print(ht_str)

    xpathquery_str = "//dc:title/text()"
    q_result = ht_str.xpath(xpathquery_str, namespaces=NS)
    print("图书标题=" + q_result[0])

    # result=html.xpath('//li[@class="item-1"]/a/text()') #获取a节点下的内容
    xpathquery_str = "//dc:identifier[@opf:scheme='ISBN']/text()"
    q_result = ht_str.xpath(xpathquery_str, namespaces=NS)
    print("ISBN=" + q_result[0])
    #log.logger.info(q_result)

    xpathquery_str = "//dc:identifier[@opf:scheme='ASIN']/text()"
    q_result = ht_str.xpath(xpathquery_str, namespaces=NS)
    print("ASIN=" + q_result[0])

    xpathquery_str = "//dc:date/text()"
    q_result = ht_str.xpath(xpathquery_str, namespaces=NS)
    print("创建时间=" + q_result[0])

    xpathquery_str = "//dc:creator/text()"
    q_result = ht_str.xpath(xpathquery_str, namespaces=NS)
    print("创建人=" + q_result[0])

    xpathquery_str = "//reference/@href"
    q_result = ht_str.xpath(xpathquery_str)
    print(q_result)

    xpathquery_str = "//item[@properties='cover-image']/@href"
    q_result = ht_str.xpath(xpathquery_str)
    print(q_result)

    xpathquery_str = "//item[@properties='media-type']"
    q_result = ht_str.xpath(xpathquery_str, namespaces=NS)
    print(q_result)
```

​		关键是上面的代码中，前部分（dc:xxx）的哪些元素能获取到信息，但是后面哪些查询就不灵了啊。

​		content.opf就不帖了；

​		试错过程：

​			1、我不停的试xpath的查询语句，怎么都不行，我怀疑还是跟名称空间有关系。

​			2、所以用一个string的方式（去掉名称空间的str）来测试，代码参考网上的（w3cschool）。

​		🔥🔥🔥 **真是太难了啊：**

​			对于content.opf 格式的XML文件解析，带很多的namespace的；这样的文件使用两种方式进行xpath进行查询

​			🔥**对于不带ns的标记**，使用HTML的格式化后在xpath查询；

```
  html=etree.HTML(text.encode("utf-8")) #初始化生成一个XPath解析对象
  result=etree.tostring(html,encoding='utf-8')   #解析对象输出代码

  xpq_str = "//item[@id='id_6']/@href"
  print(html.xpath(xpq_str))
```

​			🔥**对于带ns的标记**，使用XML的格式化后的xpath查询：

```
  dom = etree.XML(text.encode("utf-8"))
  #xpq_str = "//dc:identifier[@opf:scheme='ISBN']/text()"
  xpq_str = "//dc:title/text()"
  aid = dom.xpath(xpq_str, namespaces=NS)
  print(aid)
```

​		其实看这一篇就差不多了：

​		[【S01E01】lxml.html、lxml.etree和Xpath - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/41969389)

​		其他参考

​		[lxml 库安装教程_w3cschool](https://www.w3cschool.cn/lxml/_lxml-ziay3fjr.html)

​		[Python3 XML 解析 | 菜鸟教程 (runoob.com)](https://www.runoob.com/python3/python3-xml-processing.html)

​		[python3解析库lxml - Py.qi - 博客园 (cnblogs.com)](https://www.cnblogs.com/zhangxinqi/p/9210211.html)

​		[使用lxml解析xml文件 - 阿布_alone - 博客园 (cnblogs.com)](https://www.cnblogs.com/tjp40922/p/14058901.html)

​		[lxml解析包（二）:对于xpath中含有命名空间的xml文件解析_不愿透露姓名の网友的博客-CSDN博客](https://blog.csdn.net/qq_40558166/article/details/103817547)

​		[Python3：使用lxml库来解析xml文件和html文件(使用xpath方式解析)_你是小KS的博客-CSDN博客_python xpath 解析xml](https://blog.csdn.net/weixin_45492007/article/details/103481551?spm=1001.2101.3001.6661.1&utm_medium=distribute.pc_relevant_t0.none-task-blog-2~default~CTRLIST~TopBlog-1.topblog&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2~default~CTRLIST~TopBlog-1.topblog&utm_relevant_index=1)

​		[python使用xpath（超详细） - 梦想家haima - 博客园 (cnblogs.com)](https://www.cnblogs.com/mxjhaima/p/13775844.html)

### 2.6、学习logging日志包的使用方法

​		logging可以在哪些场景下使用，比如：替代print；记录日志到文件、输出结果到控制台......

​		logging的包使用其实不难，但是想做一个AOP的开发试验，就需要了解Python的装饰器知识了，日志这个方模需要学习内容还是很多的。

  - 编写一个可以输出到控制台、输出到日志文件中的类

  - 在其他需要调用该类的方法引用该类，并将调用输出日志信息，调用方法大概如下：( 这个程度目前应该够用了吧...... )

    ```
    from logtest import Loggers
    
    ......... 
    if(os.path.isfile(yl_path)):
        yldict = getYamlData(yl_path)
        
        log = Loggers(level='info')
        log.logger.info("我的logging测试==" + yldict)
        log.logger.info(getYamlByKey(yl_path,'psw'))
    
    else:
        print("配置文件未找到，请确认配置文件是否真的存在。")
    ```

    

参考：

​		[python 中面向切面编程AOP和装饰器 - 简书 (jianshu.com)](https://www.jianshu.com/p/4c588eec1be1)

​		[python中logging日志模块详解 - 咸鱼也是有梦想的 - 博客园 (cnblogs.com)](https://www.cnblogs.com/xianyulouie/p/11041777.html)

​		[python logging详解及自动添加上下文信息 - xybaby - 博客园 (cnblogs.com)](https://www.cnblogs.com/xybaby/p/9197032.html)

​		[一看就懂，Python 日志 logging 模块详解及应用 | 静觅 (cuiqingcai.com)](https://cuiqingcai.com/6921.html)

​		[Python之日志处理（logging模块） - 云游道士 - 博客园 (cnblogs.com)](https://www.cnblogs.com/yyds/p/6901864.html)



### 2.7、最重要的是学习了Epub格式规范和解析

​		解析OPF文件没啥问题

#### 		2.7.1 解析NCX文件遇到名称空间定义的问题

​		NCX文件样例：

​		![image-20220329115342022](http://minio.sc-model.cn:9000/mdimages/blog/2022/3/29/image-20220329115342022.png)

​		以上红框中的NCX代码必须修改为：（加上名称空间的别名“ncx”）

```
	<ncx xmlns:ncx="http://www.daisy.org/z3986/2005/ncx/" version="2005-1">
```

​		[Epub2基础知识介绍 - 大西瓜3721 - 博客园 (cnblogs.com)](https://www.cnblogs.com/Alex80/p/4450046.html)

​		[EPUB3.0内容文件规范中文版_免费.pdf (book118.com)](https://max.book118.com/html/2017/0425/102365892.shtm)

​		[epub 格式 · Python 学习笔记 (gitbooks.io)](https://goosebaobao.gitbooks.io/python-note/content/epub-txt/epub.html)

### 2.8、学习Jinja2 模板生成MD格式文件

​		使用Jinja2 可试用模板的方式，构造MD文件

​		请注意先安装Jinja2；

>  pipenv install Jinja2 

​		**示例1:** 

```
import json
from jinja2 import Template

temps = Template('haha namepad= {{ namepad }} !!!!--- strfn= {{ strfn }}')
dic_var = { "namepad" : "who123" , "strfn" : "ssshahah123" }
s = temps.render(dic_var)

print(s)
```

​		**示例2：**



### 2.9、学习图片文件处理

​		图片图像文件处理.....

​		[Python基础模块：图像处理模块@PIL(批量分类处理图片及添加水印)_可以叫我才哥的博客-CSDN博客](https://blog.csdn.net/dxawdc/article/details/113839984)

​		[python:PIL库学习笔记 - Jessie- - 博客园 (cnblogs.com)](https://www.cnblogs.com/linjiaxin59/p/12695287.html)

​		[python PIL 图像处理 - 简书 (jianshu.com)](https://www.jianshu.com/p/e8d058767dfa)

​		[python+glob+PIL+plt 图片批量导入+显示_ClearLon的博客-CSDN博客_pil批量读取图片](https://blog.csdn.net/qq_41977618/article/details/107669707)



## 三、应用需求

- ​	**输入：**
    1. 一个epub电子书文档
    2. 一个MD文章的模板文件

- ​	**处理：**
    1. 获取epub电子书的书名
    2. 获取电子书的封面
    3. 获取电子书的内容简介
    4. 生成一个该电子书的介绍的MarkDown的文章

- ​	**输出：**
    1. 输出电子书的封面文件（.jpg / .png），保存在特定目录
    2. 输出md文件，保存在特定目录

	
	
## 四、设计思考

​	首先epub文件处理（读取或写入）应该是单独的功能，可以按包的思路去实现，未来这个可以复用。

​	使用epub的这个文件处理工具，可以进行epub文件的信息获取，用获取的信息可以做其他的应用了，比方：这里的需求是生成MD格式文件

---

![image-20220320161901320](http://minio.sc-model.cn:9000/sitepic/blogpic/image-20220320161901320.png)



## 五、学习验证测试

### 5.1  测试操作epub

​	构建一个类 epub

  - 导入主要的包

  - 创建一个类的构建函数

  - 定义类的公有属性

    - zip文件（电子书epub文件）实例；
    - opf文件实例，一个lxml对象；
    - toc文件实例，一个lxml对象；
    - opf中的元数据信息实例，用字典类型保存；
    - 电子书epub文件的封面图片对象实例；

  - 创建公有方法，**获取电子书名**

  - 创建公有方法，获取电子书一些基本信息，并复制到Metadata中

  - 创建公有方法，获取opf文件实例

  - 创建公有方法，获取toc文件实例

  - 创建公有方法，获取电子书封面文件 

  - 创建公有方法，改名保存的方法

    

### 5.2  测试生成MD文件

今天先就这样吧，休息.............

---

<img src="http://minio.sc-model.cn:9000/sitepic/blogpic/image-20220320162206279.png" class="mdimg"/>

兴奋起来吧.....; 好看吧！



## 六、开始实现

### 6.1、先编写epub文件处理工具

### 6.2、再编写生成MD文件函数



源码：[pyepub](https://gitee.com/hoeking/pyepub)

---

![image-20220320161751222](http://minio.sc-model.cn:9000/sitepic/blogpic/image-20220320161751222.png)

<style>
	img.mdimg { border: 1px solid #ddd; border-radius: 4px; padding: 5px;  box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.2), 0 6px 20px 0 rgba(0, 0, 0, 0.19);}
</style>
