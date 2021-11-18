---
layout: post
title:  "使用VBA中的XMLHTTP群发企业微信消息"
categories: Other
tags:  Excel VBA XMLHTTP 企业微信 
author: Hoeking CLP
---

* content
{:toc}

关于Excel VBA，开发方法这里不多讲了，VBA开发可以做很多的工作，可以很快速的满足业务需要，公司很多的提高业务工作效率，减低加班的可能的应用功能，都采用了VBA的写法。
本文阅读者要求：
- 熟悉Excel、熟悉VBA开发
- 了解企业微信消息发送原理
- 了解WebAPI技术原理

## 业务需求
财务部门在核算业务员提成后，需要将核算出来的提成数据（Excel）发送给业务员，业务员如果有疑问可再次联系财务部门，核算会计如果一条条的复制发送，很麻烦，工作量大，而且容易遗漏，所以业务部门提出问题给IT，希望IT给与一个解决方法。

![image-20211118090228115](https://chenlip.github.io/css/Img/image-20211118090228115.png)



## 解决对策

因为业务核算业务员提成时采用的是线下手工核算，因为业务员提成的异常情况和变化太多，目前难以用系统来实现，所以财务部门核算会计只能人工核算，核算结果如上图，现在需要将核算结果批量发送给业务员 。我们提出的解决对策：临时在Excel中编制消息发送脚本。

简单的说，一个简单的企业微信消息发送原理如下：

- 首先我们在企业微信上部署一个消息应用，需要通过这个消息应用进行信息发送
- 然后我们后端开发人员封装一个企业微信消息发送Webapi，部署在内网的服务器上
- 对提成核算表进行必要的改造，以方便更好的编制VBA代码
- 在Excel中循环每个业务员的提成数据，构造成一个JSON消息体
- 在Excel中使用XMLHTTP调用内网的消息发送的webapi，根据业务员手机号发送消息
- 

## 约束条件

- 首先，业务员必须使用公司的手机号注册加入到企业微信（现状：大部分业务员已经加入企业微信）
- 其次，提成核算表中的业务员手机号与企业微信中注册的手机号必须保持一致
- 核算会计的电脑上必须安装Excel  2013 或以上版本（WPS 对VBA支持不好）
- 为是VBA能正常执行，Excel采用的文件格式为.xlsm 

## 实现

### 一、在企业微信中创建消息发送应用

![image-20211118092740144](https://chenlip.github.io/css/Img/image-20211118092740144.png)

	需要企业微信管理员创建一个消息发送的自建应用，详细的方法可参考企业微信服务端API帮助文档。

### 二、后端开发人员编写消息发送的Webapi

关于如何编写企业微信发送消息的webapi，这里也不详细描述了，参考企业微信官方的服务端api帮助即可，详细的后端代码也不贴了，部署内网的过程是以前已经规划好的，我这里也不多讲，只说明如何调用webapi：

调用URL： http://192.168.x.xx:8020/api/wcUser/setSend
方法：POST
消息体：JSON

```js
{
    "姓名":"666",
    "身份证号码":"37565231.....",
    "phone":"18354.....",
    "销售提成":"销售提成",
    "租赁提成":"租赁提成",
    "提成合计":"提成合计",
    "冲借款":"冲借款",
    "冲垫付款":"冲垫付款",
    "应发提成":"应发提成",
    "代缴个税":"代缴个税",
    "实发提成":"实发提成",
    "title":"這裏是標題",
    "sender":"發送人"
}.
```

* 安全提示，这里的后端代码并未做发送者的身份认证，理论上通过这个webapi，谁都可以发送消息。鉴于我们目前在内网使用，暂时不考虑安全性了。

### 三、对原提成核算Excel进行改进

为方便进行消息构造及信息的发送对原来的提成核算Excel表格进行改进，改进主要以下几点：

	- 增加发送内容的标题、发送人输入单元格
	- 增加发送按钮
	- 在每行第一列增加“是否发送”的判断项，1-可以发送；0-不发送
	- 在要发送消息的标题列上，加上字符“Y”

![image-20211118101750231](https://chenlip.github.io/css/Img/image-20211118101750231.png)

### 四、编写VBA代码

代码注释已经比较详细，各位看官能读懂：

**VBA主程序：**（按钮点击执行的代码）Sub btn_sendmsg_Click()

```js
'--------------------------------------------------------------------
'Create by Chenlip at 2021-11-11
'EXCEL VBA 提成单消息发送-企业微信
'53258372@qq.com
'--------------------------------------------------------------------
Public msgurl As String
Public str_ActShtName As String

Private Sub btn_sendmsg_Click()
    '0- 上下文环境初始
    msgurl = "http://192.168.X.XX:8020/api/wcUser/setSend"
    str_ActShtName = ActiveSheet.Name
    
    Dim strResponse As String
    Dim strHeaders As String
    Dim objxml As Object
    Set objxml = CreateObject("Msxml2.XMLHTTP.6.0") '创建MSXML2.XMLHTTP 对象 执行http访问

    ' 用Variant和String对象，解决Xmlhttp.send 参数（最好使用Variant数据类型）错误的问题，这是因为xmlhttp的不同windows环境下的版本问题
    Dim msgstr As String, v_msgbody As Variant
    
    Dim salesname As String

    Dim arr_index()
    Dim arr_keys()
    Dim arr_vals()
    
    Call SetArrayVal(arr_index, arr_keys)

    '1-循环所有的记录
    Dim i As Integer
    i = 4 '从第四行开始循环，从第四行开始必须是数据
    Dim issend As String
    
    Do While i > 0
        salesname = Trim(Worksheets(str_ActShtName).Cells(i, 2).Value) ' 姓名
        issend = Trim(Worksheets(str_ActShtName).Cells(i, 1).Value) ' 是否发送消息

        ReDim arr_vals(UBound(arr_index()))

        If Len(salesname) <= 0 Then
            Exit Do
        Else
            ' 构造消息体
            For j = 0 To UBound(arr_index())
                If Not IsEmpty(Worksheets(str_ActShtName).Cells(i, arr_index(j))) Then
                    arr_vals(j) = Trim(Worksheets(str_ActShtName).Cells(i, arr_index(j)).Value)
                End If
            Next
            
        End If
        
        ' 发送消息请求
        If CInt(issend) = 1 Then
            msgstr = GenJsonFromKeyVal(arr_keys, arr_vals)
            ' MsgBox "JSON" & i & "====" & msgstr
            v_msgbody = msgstr
            
            objxml.Open "POST", msgurl, False ' POST方法
            objxml.SetRequestHeader "Content-Type", "application/text; charset=UTF-8"
            objxml.Send v_msgbody

            strResponse = objxml.responsetext  '获取WEBAPI返回结果
            strHeaders = objxml.GetAllResponseHeaders '获取WEBAPI返回Headers

            If strResponse = "发送不成功" Then
                '本函数直接返回空串并跳出,设置行首个单元格的背景颜色值为红色
                Worksheets(str_ActShtName).Cells(i, 1).Interior.ColorIndex = 6
                Exit Sub
            Else
                Worksheets(str_ActShtName).Cells(i, 1).Interior.ColorIndex = 10
            End If
        End If
        
        ' 每行发送消息完成后，清空数组内的数据。
        Erase arr_vals
        msgstr = ""
        
        i = i + 1
    Loop
    
    
    'end 循环
End Sub
```

**设置消息内容序列的函数方法：**Sub SetArrayVal(arr_inx, arr_kys)

```js
' 获取需要发送数据列，并设置好数组值
Public Sub SetArrayVal(arr_inx, arr_kys)
    Dim coltitle As String
    Dim s As Integer, m As Integer
    m = 1
    s = 0
    ' 获取数组的大小
    Do While m > 0
        coltitle = Trim(Worksheets(str_ActShtName).Cells(3, m).Value) '从第三行开始，第三行必须是表头
        If Len(coltitle) <= 0 Then
            Exit Do
        Else
            If InStr(coltitle, "Y") <> 0 Then
                s = s + 1
            End If
        End If

        m = m + 1
    Loop

    ReDim arr_inx(0 To s - 1)
    ReDim arr_kys(0 To s - 1)

    ' 设置数组值
    m = 1
    s = 0
    Do While m > 0
        coltitle = Trim(Worksheets(str_ActShtName).Cells(3, m).Value) '从第三行开始，第三行必须是表头
        If Len(coltitle) <= 0 Then
            Exit Do
        Else
            If InStr(coltitle, "Y") <> 0 Then
                coltitle = Replace(coltitle, "Y", "")
                coltitle = Replace(coltitle, "手机号", "phone")
                arr_inx(s) = m
                arr_kys(s) = Trim(coltitle)

                s = s + 1
            End If
        End If

        m = m + 1
    Loop

End Sub

```

**构造消息体JSON函数方法：**Function GenJsonFromKeyVal(arr_ks, arr_vls) As String

```js
' 将key数组和value数组组合成JSon字符串；
Public Function GenJsonFromKeyVal(arr_ks, arr_vls) As String
    Dim postjson As String
    Dim arr_len As Integer
    Dim str_title As String, str_sender As String ' 消息标题；消息发送人
    str_title = Trim(Worksheets(str_ActShtName).Cells(2, 3).Value)
    str_sender = Trim(Worksheets(str_ActShtName).Cells(2, 5).Value)

    arr_len = UBound(arr_ks)

    postjson = "{"
    
    If UBound(arr_vls) > 1 Then
        For i = 0 To arr_len
            postjson = postjson & Chr(34) & arr_ks(i) & Chr(34) & ":" & Chr(34) & arr_vls(i) & Chr(34) & ","
        Next
    End If
    postjson = postjson & Chr(34) & "title" & Chr(34) & ":" & Chr(34) & str_title & Chr(34) & ","
    postjson = postjson & Chr(34) & "sender" & Chr(34) & ":" & Chr(34) & str_sender & Chr(34)
    postjson = postjson & "}"

    GenJsonFromKeyVal = postjson

End Function
```


## 补充

在使用调试过程中遇到一个大坑啊，特此记录。

**现象描述：**

1、我的本地电脑：win10 + Office 2016； 执行发送消息成功，在企业微信中正常收到发送的消息。

2、业务的电脑配置原先是Windows7 + WPS 2000；先让运维卸载WPS；安装Office 2016（带Excel 2016标准版本），点击发送按钮，藕荷报错了：

![image-20211118103411954](https://chenlip.github.io/css/Img/image-20211118103411954.png)

**问题解决过程：**

开始我以为是XMLHTTP版本的问题，从各种版本都试了，我的本机都没有出现问题，总是可以发送，但是业务电脑就是不行。
当然在搜索问题的过程中还是小有收获的，了解了XMLHTTP的一些基本信息，

![image-20211118104430105](https://chenlip.github.io/css/Img/image-20211118104430105.png)

**百度国内根本找不到问题描述和解决方法**，后来使用了微软的Bing(google要梯子，好梯子要钱啊)，这里搜索有个小技巧（使用全英文的关键词搜索，bing会返回一些国外的链接，如果含中文的关键词搜索，得到的结果都是中文的）；**看来百度还是只关注中国地区啊**。

	关键词：excel vba para error 

在一个国外的论坛上终于找到了一段描述，确实跟OS版本有关系，XMLHTTP的版本要求send方法的参数定义的问题

	Pure speculation - You appear to be working in an environment that allows strong typing (VBA in Office?) and that the type of the parameter for the send is defined as a Variant when examining the object and that you have dimensioned BodyOfRequest as String.  Maybe the 3.0 code performs a typecast and the 6.0 does not.  Of course, I would have expected that to cause a "type mismatch" error, but ...

于是对代码进行修改：

设置一个临时的Variant类型变量，将构造好的String先赋值给临时变量；再发送.send 临时变量

```js
..............
' 用Variant和String对象，解决Xmlhttp.send 参数（最好使用Variant数据类型）错误的问题，这是因为xmlhttp的不同windows环境下的版本问题
    Dim msgstr As String, v_msgbody As Variant
...
...
			msgstr = GenJsonFromKeyVal(arr_keys, arr_vals)
            ' MsgBox "JSON" & i & "====" & msgstr
            v_msgbody = msgstr
            
            objxml.Open "POST", msgurl, False ' POST方法
            objxml.SetRequestHeader "Content-Type", "application/text; charset=UTF-8"
            objxml.Send v_msgbody
    
```

在Windows7 上测试，OK，解决问题 。

## 后记

这次开发共用时初步统计：8h（跨了3天）

- 1-webapi开发修改估计：2h
- 2-VBA开发估计：4h
- 3-调试和解决问题：2h

今天11月18日，又发现一个问题，**发送的消息在苹果手机上显示异常**：

![image-20211118143201947](https://chenlip.github.io/css/Img/image-20211118143201947.png)

目前还在诊断原因中，估计是企业微信本身的webapi的问题。

从这个程序只需简单修改，可以扩展适用应用场景，比如：

- 工资条按人员发送通知消息；
- 企业内部特殊的信息批量发送；
- 员工生日祝福；
- 节日祝福批量发送（给客户、员工）；

从原理上来看，只需要有**合适的后端webapi接口，还可以扩展发送微信、qq、短信、钉钉等多种消息方式**。
