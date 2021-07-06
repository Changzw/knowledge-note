
# 使用Command处理命令解析业务

## 背景

server 传来 command json，要求客户端根据 json 解析 command，然后客户端按照一定规则执行 command

## 方案

第一反应：使用Command模式，这个模式应用场景算是个特例，可以直接拿一个 demo 抄下
内部组成：

1. command（单独解析）
2. engine（执行command组合）
3. receiver（被污染的源）

## 具体细节

1. 定义 command 通用接口

```swift
// 如果json 可以部分数据，可以思考使用泛型
protocol AnyCommand {
  var context: [Any]{ get }
  func execute(receiver: [String: Any]) throws -> [String: Any]
}
```

2. engine

* 得到 receiver
* 得到 context
* run

```swift
try modelDict = contexts
  .map { $0.command() }
  .reduce(receiver, {
    try $1.execute(receiver: $0)
})
```
