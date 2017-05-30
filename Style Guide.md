# Style Guide 
# 风格指南
This document provides a style guide for .proto files. By following these conventions, you'll make your protocol buffer message definitions and their corresponding classes consistent and easy to read. 
本文档提供了 .proto 文件的样式指南。通过遵循这些约定，您将使您的协议缓冲区消息定义及其相应的类一致且易于读。

## Message And Field Names
## 消息和字段名
Use CamelCase (with an initial capital) for message names – for example, SongServerRequest. Use underscore_separated_names for field names – for example, song_name. 
用CamelCase（首字母大写）消息的名字–例如，SongServerRequest。使用underscore_separated_names字段名称–例如，song_name。

```
message SongServerRequest {
  required string song_name = 1;
}
```

Using this naming convention for field names gives you accessors like the following: 
使用这种命名字段名称给你访问如下：

```
C++:
  const string& song_name() { ... }
  void set_song_name(const string& x) { ... }

Java:
  public String getSongName() { ... }
  public Builder setSongName(String v) { ... }
```

## Enums
## 枚举
Use CamelCase (with an initial capital) for enum type names and CAPITALS_WITH_UNDERSCORES for value names:
用CamelCase（首字母大写）为枚举类型名字，用CAPITALS_WITH_UNDERSCORES为枚举值名称：

```
enum Foo {
  FIRST_VALUE = 0;
  SECOND_VALUE = 1;
}
```

Each enum value should end with a semicolon, not a comma. 
每个枚举值应以分号结束，不是一个逗号。

## Services
## Services
If your .proto defines an RPC service, you should use CamelCase (with an initial capital) for both the service name and any RPC method names: 
如果你的 .proto 定义一个RPC服务，你应该使用CamelCase（首字母大写）为服务名称和任何RPC方法名称：
```
service FooService {
  rpc GetSomething(FooRequest) returns (FooResponse);
}
```