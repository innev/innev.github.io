# helper
> 工具类

## 功能
    1. 配置解析
    2. 变量类型判断

## 使用方法

### 变量类型判断实例
```python
from helper import typeof, getType

#判断变量是否为整数
money=120
print("{0}是{1}".format(money,getType(money)))
#判断变量是否为字符串
money="120"
print("{0}是{1}".format(money,getType(money)))
money=12.3
print("{0}是{1}".format(money,getType(money)))
#判断变量是否为列表
students=['studentA']
print("{0}是{1}".format(students,getType(students)))
#判断变量是否为元组
students=('studentA','studentB')
print("{0}是{1}".format(students,getType(students)))
#判断变量是否为字典
dictory={"key1":"value1","key2":"value2"}
print("{0}是{1}".format(dictory,getType(dictory)))
#判断变量是否为集合
apple={"apple1","apple2"}46 print("{0}是{1}".format(apple,getType(apple)))
```

### 配置解析实例
```python
from helper import Configuration

config = Configuration().parse()
test = config.option("config", "test")
print(test)
```

## 未来计划