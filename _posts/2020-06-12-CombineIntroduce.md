---
layout: post
title: "Combine 初见"
date: 2020-06-06
---



# Combine基础
> 通过预先对事件处理（解析JSON、校验输入等）的操作进行组合，来对异步事件（网络、UI 事件、动画结束等）进行汇总后处理。 Combine框架提供了一组响应式编程的API来实现这套逻辑，框架主要由三部分组集成，分别是Publisher、Operator、Subscriber。

# Publisher
> 负责在源头发布事件

    public protocol Publisher {
        associatedtype Output
        associatedtype Failure : Error
        func receive<S>(subscriber: S) where  S : Subscriber,Self.Failure ?? S.Failure,Self.Output ?? S.Input
    }

##Publisher 可以发布的事件类型
* Output： 正常事件流的值（一次、多次、0次）
* Failure：异常结束回调。（一次或 0次）

##Operator
> 加工处理数据，转换类型
> 处理后将会返回一个新的Publisher

## 例如 San 与 Map
    let buttonClicked: AnyPublisher<Void, Never>
    buttonClicked
        .scan(0) { value, _ in value + 1 }
        .map { String($0) }


#Subscriber
> 通过多个Operator进行处理和转换，最终由Subscriber消费


    public protocol Subscriber{
        associatedtype Input
        associatedtype Failure : Error
        func receive(subscription: Subscription)
        func receive(_ input: Self.Input) ?? Subscribers.Demand
        func receive(completion: Subscribers.Completion<Self.Failure>)
    }
##Subscriber组成
* Input:与Output类型一致，接收其发布的新数据
* Failure：与Output类型一致，接收其发布的错误
* 回调方法：接收数据、错误结束、正常结束

# 常见Subscriber 
###Sink 与Assign
> 在Operator组合处理完各种异步事件处理逻辑后，将处理后的数据输出。不同的是，Assign支持将数据通过KVC写入对象。

    class Foo {
        var bar: String = ""
    }
    let foo = Foo()
    let buttonClicked: AnyPublisher<Void, Never>
    buttonClicked
        .scan(0) { value, _ in value + 1 }
        .map { String($0) }
        .assign(to: \.bar, on: foo)
        //.sink { print("Button pressed count: \($0)") }


#Scheduler
> 主要解决的问题是在什么时候，什么地方执行代码。

    URLSession.shared
        .dataTaskPublisher(for: URL(string: "https:??example.com")!)
        .compactMap { String(data: $0.data, encoding: .utf8) }
        .receive(on: RunLoop.main)
		//.delay(for: .seconds(2), scheduler: Runloop.main)
		//.bebounce(for: .seconds(1), scheduler:Runloop.main)
        //.sink(receiveCompletion: { _ in
    }, receiveValue: {
        textView.text = $0
    })