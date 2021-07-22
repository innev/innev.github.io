# python学习笔记
> 记录python的学习笔记

## 一、基础语法学习

### 1.1. 语法特点（区别于其他主流语言）
- 使用`def`来定义方法
- 没有使用`{}`作为代码快的隔离，而是使用`:`作为代码块开始，然后使用`空格`作为代码换行缩紧
- 函数的参数支持多种类型：必须参数、默认参数、可变参数（不定长参数）、关键字参数、组合参数。用法非常灵活

### 1.2. 特有变量类型
> list和tuple都有非常强大的切片，分割等功能，会在具体实例中展示
- 元组(tuple)，以`(<item>, <item>, ...)`的形式出现
- 列表(list)，以`[<item>, <item>, ...]`的形式出现。类似js中的数组和PHP中的索引数组
- 字典（dict），以`{<item>, <item>, ...}`的形式出现。类似js中的Object和PHP中的关联数组

## 二、python环境版本

### 2.1. 可用版本
> 目前稳定可用的版本是3.7.8，更高的版本或多或少会不稳定

### 2.2. 更新pip到当前python可支持的最新版本
```shell
python -m pip install --upgrade pip
```
### 2.3. 虚拟环境
```python
# 创建虚拟环境
virtualenv -p python3 --system-site-packages .venv

# 进入虚拟环境
source .venv/bin/activate

# 退出虚拟环境
deactivate
```

## 三、模块和包的应用

### 3.1. 自定义模块

### 3.2. 自定义包

### 3.3. 发布自定义包到远程仓库

## 四、常用第三方包

### 4.1. pandas
> pandas是一个强大的数据处理包，可以横向、纵向操作数据。方便的切片、分割、过滤、排序、导入、导出等数据操作

### 4.2. [WeRoBot](werobot/index.html)
> WeRoBot 是一个微信公众号开发框架。

[打开`WeRoBot`文档](werobot/index.html)

### 4.3. [Sphinx](sphinx/index.md)
> Sphinx是一个可以用于Python的自动文档生成工具，可以自动的把docstring转换为文档，并支持多种输出格式包括html，latex，pdf等。

[打开`sphinx`文档](sphinx/index.md)

### 4.4. [APScheduler](scheduler/APScheduler.md)
> APScheduler是一个 Python 定时任务框架，使用起来十分方便。提供了基于日期、固定时间间隔以及 crontab 类型的任务，并且可以持久化任务、并以 daemon 方式运行应用。

[打开`APScheduler`文档](scheduler/APScheduler.md)

## 五、最佳项目实践

### 5.1. [自定义帮助包](pages/helper.md)

### 5.2. [企业微信包](pages/wework.md)

### 5.3. 自定义数据包

### 5.4. [文件或图片上传](pages/upload.md)

### 5.5. list切片操作