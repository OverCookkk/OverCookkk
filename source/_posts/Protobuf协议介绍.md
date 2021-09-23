---
title: Protobuf协议介绍
---

## 一、Protobuf简介
protobuf(Google Protocol Buffers)是Google提供一个具有高效的数据传输的协议，可以用于结构化数据序列化，支持 Java、Python、C++、Go 等语言。

## 二、Protobuf消息定义
关键字message: 代表了实体结构，由多个消息字段(field)组成。
消息字段(field): 包括数据类型、字段名、字段规则、字段唯一标识、默认值
数据类型：常见的原子类型都支持(在FieldDescriptor::kTypeToName中有定义)
字段规则：

- required：必须初始化字段，如果没有赋值，在数据序列化时会抛出异常

- optional：可选字段，可以不必初始化。

- repeated：数据可以重复(相当于java 中的Array或List)

默认值(default)：在定义消息字段时可以给出默认值。

```protobuf
message BookBehaviorPb
{
    optional uint32         bid              = 1;
    repeated BaseBehaviorPb show_pb          = 2; // 按产品去重 -> 不去重
    repeated BaseBehaviorPb click_pb         = 3; // 按产品去重 -> 不去重
    repeated BaseBehaviorPb read_pb          = 4; // 按章节汇总
    optional uint32         data_type        = 5 [default = 0]; // 0novel, 1comic, 2hear, 3avg
    repeated BaseBehaviorPb buy_pb           = 6; 
}
```

## 三、Protobuf
* Xml、Json是目前常用的数据交换格式，它们直接使用字段名称维护序列化后类实例中字段与数据之间的映射关系，一般用字符串的形式保存在序列化后的字节流中。消息和消息的定义相对独立，可读性较好。但序列化后的数据字节很大，序列化和反序列化的时间较长，数据传输效率不高。
	
	相对于 JDK、JSON，由于 Hessian 更加高效，生成的字节数更小，有非常好的兼容性和稳定性，所以 Hessian 更加适合作为 RPC 框架远程通信的序列化协议。
	
	Protobuf和Xml、Json序列化的方式不同，采用了二进制字节的序列化方式，用字段索引和字段类型通过算法计算得到字段之前的关系映射，从而达到更高的时间效率和空间效率，特别适合对数据大小和传输速率比较敏感的场合使用
	
	 
	
	protobuf把消息结果message也是通过 key-value对来表示，protobuf是由字段索引(fieldIndex)与数据类型(type)计算(fieldIndex<<3|type)得出的key维护字段之间的映射且只占一个字节，所以相比json与xml文件，protobuf的序列化字节没有过多的key与描述符信息，所以占用空间要小很多。