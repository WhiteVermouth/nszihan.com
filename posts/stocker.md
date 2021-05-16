---
title: "Stocker"
date: "2021-04-11 23:00:00"
---

![](/images/stocker/logo.png)

## 简介
Stocker是一款JetBrains IDE证券行情插件，主要功能是在IntelliJ IDEA和其他基于IntelliJ Platform的IDE（如PyCharm、RubyMine、WebStorm等）中实时追踪股票、加密货币的价格行情，其中股票支持A股、港股、美股。

## 运行时效果

![](/images/stocker/screenshot.png)

## 安装
Stocker已上传至JetBrains Marketplace，打开IDE内的 Plugins，然后搜索**Stocker**，找到对应项并点击安装，安装完成后可能需要重启IDE，依据提示操作即可。

<iframe frameborder="none" width="384px" height="319px" src="https://plugins.jetbrains.com/embeddable/card/15443"></iframe>

## 如何使用
### 添加股票、加密货币代码
点击Stocker窗口上的 **➕** 弹出搜索窗口，利用关键字（代码、名字）找到目标标的，点击 **Add** 按钮：

![](/images/stocker/search-add.png)

### 管理已添加的股票、加密货币代码
点击Stocker窗口上的 **➖** 弹出管理窗口，不同市场的标的归类在不同的Tab中，点击 **Delete** 可删除不再需要的标的：

![](/images/stocker/manage-delete.png)

### 停止刷新
Stocker会每5秒通过HTTP请求拉取最新数据，如果有时不希望占用网络（比如休市时段），可点击在窗口Action栏的停止刷新按钮。若要重新开始刷新，点击刷新按钮。

### 配置修改
Stocker的配置在**Preferences/Settings -> Tools -> Stocker**，当前支持修改配色：红涨绿跌、绿涨红跌、无配色：

![](/images/stocker/preferences.png)

## 其他操作

### 使用工具栏
Stocker的Action（搜索窗口、管理窗口、刷新、停止刷新）可在工具栏**Tools -> Stocker**中找到：

![](/images/stocker/tool-actions.png)

### 使用Action搜索

通过Action快速搜索也可触发Stocker的Action：

![](/images/stocker/dialog-actions.png)

## 错误处理
如果遇到任何使用问题可以先尝试重装、重启。无法解决的问题可在GitHub提[Issue](https://github.com/WhiteVermouth/intellij-investor-dashboard/issues)。