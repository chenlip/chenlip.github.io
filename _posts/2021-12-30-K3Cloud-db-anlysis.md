---
layout: post
title:  "K3Cloud直接调拨单信息数据获取以及单据下推过程中的数据查询"
categories: Other
tags:  K3Cloud、SQL、数据库、单据下推 
author: Hoeking CLP
---

* content
{:toc}

11月的时候，应业务要求编制了VBA用于计算提成，前期已经写过一篇文章《使用VBA中的XMLHTTP群发企业微信消息》，12月时，业务提出了新的需求，开始理解其实挺简单的，就是去读取金蝶K3Cloud的数据库中的“直接调拨单或订单中的数据”，实际开始编制数据查询语句时发现不简单啊，过程虽然曲折，但是也弄清楚了K3Cloud的单据的数据库设计的思路，特此记录。

本文阅读者要求：（好像不好找这种人才哦，哈哈，自我陶醉一下）
- 熟悉MS SQL；熟悉SQL语言
- 熟悉金蝶ERP业务（产销存）
- 了解K3Cloud的单据功能
- 了解WK3Cloud的数据库结构
- 了解K3Cloud的WebAPI接口的查询及调试方法

本文过程中需要查阅的相关资料：
- 《金蝶云cloud7.1数据字典》
    用于了解数据库字段信息和定义，（可以在我的微博中下载），下载地址：

- 《K3Cloud的WebAPI接口》
    主要用于查找自定义单据中新增的数据，官方数据字典中不包含二开、自定义的单据的数据字段说明。

本文过程中需要的工具：
- 【MSSM V18】 微软官方的数据库管理工具
- 【VSCODE】 微软开源的源代码编辑器，万能工具，我用这个来编写SQL

## 解决对策

因为业务核算业务员提成时采用的是线下手工核算，因为业务员提成的异常情况和变化太多，目前难以用系统来实现，所以财务部门核算会计只能人工核算，核算结果如上图，现在需要将核算结果批量发送给业务员 。我们提出的解决对策：临时在Excel中编制消息发送脚本。

简单的说，一个简单的企业微信消息发送原理如下：

- 首先我们在企业微信上部署一个消息应用，需要通过这个消息应用进行信息发送
- 然后我们后端开发人员封装一个企业微信消息发送Webapi，部署在公司内网的服务器上
- 对提成核算表进行必要的改造，以方便更好的编制VBA代码
- 在Excel中循环每个业务员的提成数据，构造成一个JSON消息体
- 在Excel中使用XMLHTTP调用内网的消息发送的webapi，根据业务员手机号发送消息
- 

## 约束条件

- 首先，业务员必须使用公司的手机号注册加入到企业微信（现状：大部分业务员已经加入企业微信）
- 其次，提成核算表中的业务员手机号与企业微信中注册的手机号必须保持一致
- 核算会计的电脑上必须安装Excel  2013 或以上版本（WPS 对VBA支持不好）
- 为是VBA能正常执行，Excel采用的文件格式为.xlsm 
- 该Excel只能在公司内网才能使用，带出公司就不能发送消息了