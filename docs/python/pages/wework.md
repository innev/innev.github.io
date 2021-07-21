# wework
> 企业微信库

## 功能
    1. 获取access_token
    2. 发送消息给指定的应用

## 消息类型
    1. 文本消息类型 - text
    2. 卡片消息类型 - textcard
    3. 图片消息类型 - image

## 使用方法
```python
import wework

robot = wework.WeRoBot()
robot.run()
```

## 未来计划
    1. 支持消息回调服务（需要启动WEB服务）
    2. 支持全部消息类型
    3. 支持其他企业微信其他API接口