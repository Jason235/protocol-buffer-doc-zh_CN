# Language Guide (proto3 语言指南)

标签（空格分隔）： protobuf

[TOC]

---

This guide describes how to use the protocol buffer language to structure your protocol buffer data, including .proto file syntax and how to generate data access classes from your .proto files. It covers the proto3 version of the protocol buffers language: for information on the older proto2 syntax, see the Proto2 Language Guide.
本指南介绍如何使用 Protocol buffer 语言构造 Protocol buffer 数据，包括 .proto 文件语法以及如何从 .proto 文件生成数据访问类。它涵盖了 protocol buffers 语言 proto3 版本：对老版 proto2 语法信息，请参看 [proto2 语言指南]。

This is a reference guide – for a step by step example that uses many of the features described in this document, see the tutorial for your chosen language (currently proto2 only; more proto3 documentation is coming soon).
该参考指南是使用循序渐进的例子来描述 protocal buffer 的特性，请参看[教程]你选择的语言（目前只有 proto2；更多 proto3 文文档稍后推出）。

## Defining A Message Type 定义消息类型

First let's look at a very simple example. Let's say you want to define a search request message format, where each search request has a query string, the particular page of results you are interested in, and a number of results per page. Here's the .proto file you use to define the message type.

首先让我们看一个非常简单的例子。假设您想要定义一个搜索请求消息格式，其中每个搜索请求包含查询字符串、您期待的特定页数据和期望每页数据调条数。以下是用于定义该消息类型的 .proto 文件。

``` protobuf
syntax = "proto3";

message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3;
}
```

- The first line of the file specifies that you're using proto3 syntax: if you don't do this the protocol buffer compiler will assume you are using proto2. This must be the first non-empty, non-comment line of the file.
- 该文件的第一行指定你用 proto3 语法：如果你不这样做的 protocol buffer 编译器会假设您使用的是 [proto2]。文件第一行必须非空的，非注释的。

- The SearchRequest message definition specifies three fields (name/value pairs), one for each piece of data that you want to include in this type of message. Each field has a name and a type.
- SearchRequest 消息定义指定的三个字段, 每一个你想要包含在消息中的数据数据都以(名称/值 对)形式存在。每个字段都有名称和类型。

### Specifying Field Types 指定字段类型

In the above example, all the fields are scalar types: two integers (page_number and result_per_page) and a string (query). However, you can also specify composite types for your fields, including enumerations and other message types.
在上面的例子中，所有的字段都是数值类型：两个整数（page_number 和 result_per_page）和一个字符串（query）类型。然而，你也可以指定复合类型，包括枚举和其他消息类型。

### Assigning Tags 分配标签

As you can see, each field in the message definition has a unique numbered tag. These tags are used to identify your fields in the message binary format, and should not be changed once your message type is in use. Note that tags with values in the range 1 through 15 take one byte to encode, including the identifying number and the field's type (you can find out more about this in Protocol Buffer Encoding). Tags in the range 16 through 2047 take two bytes. So you should reserve the tags 1 through 15 for very frequently occurring message elements. Remember to leave some room for frequently occurring elements that might be added in the future.
正如您所看到的，消息定义中的每个字段都有唯一的编号标记。这些标记用于标识消息二进制格式中的字段，并且在您的消息类型使用后不应更改该字段。请注意，标签取值范围在1到15之间，需要一个字节来编码，包括标识号和字段类型（您可以在 [Protocol Buffer Encoding] 中找到更多信息）。标签取值范围在16到2047内，需要两个字节。因此，您应该保留标签1到15为非常频繁使用的消息元素。记住留一些空间，频繁使用的元素，可能会在未来增加。

The smallest tag number you can specify is 1, and the largest is 2^29 - 1, or 536,870,911. You also cannot use the numbers 19000 through 19999 (FieldDescriptor::kFirstReservedNumber through FieldDescriptor::kLastReservedNumber), as they are reserved for the Protocol Buffers implementation - the protocol buffer compiler will complain if you use one of these reserved numbers in your .proto. Similarly, you cannot use any previously reserved tags.
您可以指定的最小标签数是1，最大的是2^29 - 1，或 536870911。你也不能用数字19000到19999（FieldDescriptor::KFirstReservedNumber 到FieldDescriptor:: KLastReservedNumber），这是 protocol buffer 预留的 - 当你在 .proto 文件中使用保留编号 protocol buffer 编译器将报警。同样，您不能使用任何保留的标签。

### Specifying Field Rules 指定字段的规则

Message fields can be one of the following:
消息字段可以是下列之一：

- singular: a well-formed message can have zero or one of this field (but not more than one).
- 单一: 一个格式化好的消息字段数据可以有零或一个(但不超过一个)。

- repeated: this field can be repeated any number of times (including zero) in a well-formed message. The order of the repeated values will be preserved.
- 重复：该字段可以重复任何次（包括零次）在一个格式化好的消息中重复值的顺序不变。

In proto3, repeated fields of scalar numeric types use packed encoding by default.
在 proto3 中，重复字段数量的数值类型使用默认压缩编码。

You can find out more about packed encoding in Protocol Buffer Encoding.
您可以在 [Protocol Buffer Encodeing] 中找到更多关于编码的信息。

### Adding More Message Types 添加更多消息类型

Multiple message types can be defined in a single .proto file. This is useful if you are defining multiple related messages – so, for example, if you wanted to define the reply message format that corresponds to your SearchResponse message type, you could add it to the same .proto:
可以在单个 .proto 文件中定义多个消息类型。如果需要定义多个相关消息，这很有用 -- 例如，如果您想定义对应于 SearchResponse 消息类型的回应消息格式，可以将其添加到相一个 .proto 中：

``` protobuf
message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3;
}

message SearchResponse {
 ...
}
```

### Adding Comments 添加注释

To add comments to your .proto files, use C/C++-style // and /* ... */ syntax.
要对 .proto 文件添加注释，请使用C/C++样式 // 和 /* ... */ 语法。

``` protobuf
/* SearchRequest represents a search query, with pagination options to
 * indicate which results to include in the response. */

message SearchRequest {
  string query = 1;
  int32 page_number = 2;  // 请求的页号?
  int32 result_per_page = 3;  // 每页返回结果数量.
}
```

### Reserved Fields 保留字段

If you update a message type by entirely removing a field, or commenting it out, future users can reuse the tag number when making their own updates to the type. This can cause severe issues if they later load old versions of the same .proto, including data corruption, privacy bugs, and so on. One way to make sure this doesn't happen is to specify that the field tags (and/or names, which can also cause issues for JSON serialization) of your deleted fields are reserved. The protocol buffer compiler will complain if any future users try to use these field identifiers.
如果通过完全删除字段或注释它来更新消息类型，未来的用户在对类型进行更新时可以重复使用标记号。这可能会导致严重的问题，如果他们以后加载旧版本的 .proto，将会导致包括数据损坏，隐私错误，等等问题。确保这种情况不会发生的一种方法是指定字段标记（或名称，这也可能会导致JSON序列化的问题）已删除的保留字段。如果将来的用户尝试使用这些字段标识符，protocol buffer 编译器会报错。

``` protobuf
message Foo {
  reserved 2, 15, 9 to 11;
  reserved "foo", "bar";
}
```

Note that you can't mix field names and tag numbers in the same reserved statement.
注意，不能在相同的保留语句中混合字段名称和标记编号。

### What's Generated From Your .proto? 你的.proto会产生了什么？

When you run the protocol buffer compiler on a .proto, the compiler generates the code in your chosen language you'll need to work with the message types you've described in the file, including getting and setting field values, serializing your messages to an output stream, and parsing your messages from an input stream.
当您运行[protocol buffer编译器]编译 .proto ，编译器生成你选择的并将使用的语言的代码文件，该代码文件包括获取和设置字段的值，信息到输出流，从输入流解析信息。

- For C++, the compiler generates a .h and .cc file from each .proto, with a class for each message type described in your file.
- 对于C++，编译器从每个 .proto 文件中生成一个 .h 和 .cc 文件，为每个消息类型定义一个 class 。
- For Java, the compiler generates a .java file with a class for each message type, as well as a special Builder classes for creating message class instances.
- 对于java，编译器为每一个消息类型生成一个 .java 文件，以及一个特殊的 Builder classes 去创建消息类的实例。
- Python is a little different – the Python compiler generates a module with a static descriptor of each message type in your .proto, which is then used with a metaclass to create the necessary Python data access class at runtime.
- Python有一点不一样 Python编译器为 .proto 文件生成一个模块，并为每一个消息生成静态的描述符，然后与 metaclass 一起使用，以便在运行时创建必要的Python数据访问类。
- For Go, the compiler generates a .pb.go file with a type for each message type in your file.
- 对于 Go，编译器为每个消息类型生成 .pb.go 文件。
- For Ruby, the compiler generates a .rb file with a Ruby module containing your message types.
- 对于 Ruby，编译器生成一个 .rb 文件和一个 Ruby module 去包含消息类型。
- For JavaNano, the compiler output is similar to Java but there are no Builder classes.
- 对于 JavaNano，编译器的输出类似于 java 但没有 Builder classes。
- For Objective-C, the compiler generates a pbobjc.h and pbobjc.m file from each .proto, with a class for each message type described in your file.
- 对于 Objective-C，编译器会为每一个 .proto 生成一个 pbobjc.h 和 pbobjc.m 文件，并与为文件中描述的每种消息类型定义一个类
- For C#, the compiler generates a .cs file from each .proto, with a class for each message type described in your file.
- 对于 C#，编译器为每一个 .proto 生成一个 .cs 文件，在文件中为每个消息类型定义一个类。

You can find out more about using the APIs for each language by following the tutorial for your chosen language (proto3 versions coming soon). For even more API details, see the relevant API reference (proto3 versions also coming soon).
你可以找到更多关于使用API的每一种语言都按照你所选择的语言的教程（proto3版本即将推出）。更多API的细节，看到相关的API（proto3版本也即将推出）。

## Scalar Value Types 标量值类型

A scalar message field can have one of the following types – the table shows the type specified in the .proto file, and the corresponding type in the automatically generated class: 
一个标量消息字段可以有以下类型之一-表显示.proto文件中指定的类型，以及自动生成类中的相应类型：

|.proto Type|Notes|C++ Type|Java Type|Python Type[2]|Go Type|Ruby Type|C# Type|PHP Type|
|:---|:---|:---|:---|:---|:---|:---|:---|:---|
|double||double|double|float|float64|Float|double|float|
|float||float|float|float|float32|Float|float|float|
|int32|Uses variable-length encoding. Inefficient for encoding negative numbers – if your field is likely to have negative values, use sint32 instead.[使用可变长度编码。负数编码效率低–如果你的字段可能有负值，使用sint32替代]|int32|int|int|int32|Fixnum or Bignum(as required)|int|integer|
|int64|Uses variable-length encoding. Inefficient for encoding negative numbers – if your field is likely to have negative values, use sint64 instead.[使用可变长度编码。负数编码效率低–如果你的字段可能有负值，使用sint64替代]|int64|long|int/long[3]|int64|Bignum|long|integer/string[5]|
|uint32|Uses variable-length encoding.[使用可变长度编码。]|uint32|int[1]|int/long[3]|uint32|Fixnum or Bignum(as required)|uint|integer|
|uint64|Uses variable-length encoding.[使用可变长度编码。]|uint64|long[1]|int/long[3]|uint64|Bignum|ulong|integer/string|
|sint32|Uses variable-length encoding. Signed int value. These more efficiently encode negative numbers than regular int32s.[使用可变长度编码。带符号int值。比普通int32编码更有效。]|int32|int|int|int32|Fixnum or Bignum(as required)|int|integer|
|sint64|Uses variable-length encoding. Signed int value. These more efficiently encode negative numbers than regular int64s.[使用可变长度编码。带符号int值。比普通int64编码更有效。]|int64|long|int/long[3]|int64|Bignum|long|integer/string|
|fixed32|Always four bytes. More efficient than uint32 if values are often greater than 2^28.[始终四字节。如果值大于2^28往往比UInt32更有效]|uint32|int[1]|int|uint32|Fixnum or Bignum(as required)|uint|integer|
|fixed64|Always eight bytes. More efficient than uint64 if values are often greater than 2^56.[始终八字节。如果值大于2^56往往比UInt64更有效]|uint64|long[1]|int/long[3]|uint64|Bignum|ulong|integer/string[5]|
|sfixed32|Always four bytes.[始终四字节。]|int32|int|int|int32|Fixnum or Bignum(as required)|int|integer|
|sfixed64|Always eight bytes.[始终八字节。]|int64|long|int/long[3]|int64|Bignum|long|integer/string[5]|
|bool||bool|boolean|bool|bool|TrueClass/FalseClass|bool|boolean|
|string|A string must always contain UTF-8 encoded or 7-bit ASCII text.[字符串必须包含UTF-8编码或7位ASCII文本。]|string|String|str/unicode[4]|string|String(UTF-8)|string|string|
|bytes|May contain any arbitrary sequence of bytes.[可能包含任意字节序列。]|string|ByteString|str|[]byte]|String(ASCII-8BIT)|ByteString|string|

You can find out more about how these types are encoded when you serialize your message in Protocol Buffer Encoding. 
你可以找到更多关于这些类型进行编码，在Protocol Buffer编码文档中。

[1] In Java, unsigned 32-bit and 64-bit integers are represented using their signed counterparts, with the top bit simply being stored in the sign bit.
[1] java，无符号32位和64位整数使用签名，表示，与前位被存储在符号位。
[2] In all cases, setting values to a field will perform type checking to make sure it is valid.
[2] 在所有情况下，设置值到一个字段将执行类型检查，以确保它是有效的。
[3] 64-bit or unsigned 32-bit integers are always represented as long when decoded, but can be an int if an int is given when setting the field. In all cases, the value must fit in the type represented when set. See [2].
[3] 在解码时，64位或无符号的32位整数总是表示为长，但如果设置字段时给出int。在所有的情况下，值必须符合设置时所表示的类型。参见[2]。
[4] Python strings are represented as unicode on decode but can be str if an ASCII string is given (this is subject to change).
[4] Python字符串在解码时表示为Unicode，但如果给出ASCII字符串，则可以是STR（这可能会发生变化）。
[5] Integer is used on 64-bit machines and string is used on 32-bit machines.
[5] 整数用于64位机器，字符串用于32位机器。

## Default Values
## 默认值
When a message is parsed, if the encoded message does not contain a particular singular element, the corresponding field in the parsed object is set to the default value for that field. These defaults are type-specific: 
当解析消息时，如果已编码的消息不包含特定的奇异元素，则解析对象中的相应字段将设置为该字段的默认值。这些默认值是特定类型的：
- For strings, the default value is the empty string.
- 对于字符串，默认值是空字符串。
- For bytes, the default value is empty bytes.
- 对于字节，默认值为空字节。
- For bools, the default value is false.
- 对于 bools，默认值是false。
- For numeric types, the default value is zero.
- 对于数值类型，默认值为零。
- For enums, the default value is the first defined enum value, which must be 0.
- 对枚举的默认值是第一个定义的枚举值，它必须是0。
- For message fields, the field is not set. Its exact value is language-dependent. See the generated code guide for details. 
- 对于消息字段，字段没有设置。其确切值是语言相关的。有关详细信息，请参见生成的代码指南。

The default value for repeated fields is empty (generally an empty list in the appropriate language).
重复字段的默认值为空（通常是相应语言中的空列表）。

Note that for scalar message fields, once a message is parsed there's no way of telling whether a field was explicitly set to the default value (for example whether a boolean was set to false) or just not set at all: you should bear this in mind when defining your message types. For example, don't have a boolean that switches on some behaviour when set to false if you don't want that behaviour to also happen by default. Also note that if a scalar message field is set to its default, the value will not be serialized on the wire. 
注意，对于标量消息字段，一旦消息被解析，就无法判断字段是否显式设置为默认值（例如，是否将一个布尔设置为false）或根本没有设置：在定义消息类型时，您应该记住这一点。例如，如果你不希望这种行为在默认情况下发生，那么当你设置为false时，不要有一个布尔值来切换某些行为。还要注意的是，如果标量消息字段设置为默认值，则该值不会在线程上序列化。

See the generated code guide for your chosen language for more details about how defaults work in generated code. 
有关所选语言的默认代码在生成代码中的作用，请参见所选语言的生成代码指南。

## Enumerations
## 枚举
When you're defining a message type, you might want one of its fields to only have one of a pre-defined list of values. For example, let's say you want to add a corpus field for each SearchRequest, where the corpus can be UNIVERSAL, WEB, IMAGES, LOCAL, NEWS, PRODUCTS or VIDEO. You can do this very simply by adding an enum to your message definition with a constant for each possible value. 
定义消息类型时，可能希望其字段中只有一个预定义的值列表。例如，假设你想为每个SearchRequest添加一个语料库，语料库，可以通用，网页，图像，本地新闻，产品或视频。你可以很简单的通过添加一个枚举你的消息定义一个常数为每个可能的值。

In the following example we've added an enum called Corpus with all the possible values, and a field of type Corpus:
在下面的例子中我们已经添加了一个枚举所有可能的值称为Corpus，一个Corpus字段：

```
message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3;
  enum Corpus {
    UNIVERSAL = 0;
    WEB = 1;
    IMAGES = 2;
    LOCAL = 3;
    NEWS = 4;
    PRODUCTS = 5;
    VIDEO = 6;
  }
  Corpus corpus = 4;
}
```

As you can see, the Corpus enum's first constant maps to zero: every enum definition must contain a constant that maps to zero as its first element. This is because: 
你可以看到，Corpus的枚举的第一常量为零：每个枚举定义必须包含一个常数，零作为它的第一个元素。这是因为：
- There must be a zero value, so that we can use 0 as a numeric default value. 
- 必须有一零个值，以便我们可以使用0作为数值默认值。
- The zero value needs to be the first element, for compatibility with the proto2 semantics where the first enum value is always the default. 
- 零值需要的第一个值，与proto2语义，第一个枚举值始终是默认的兼容性。

You can define aliases by assigning the same value to different enum constants. To do this you need to set the allow_alias option to true, otherwise the protocol compiler will generate an error message when aliases are found. 
你可以通过不同的枚举常量分配相同的值定义的别名。为此你需要设置allow_alias选项为true，否则协议编译器将生成错误消息当别名被发现。

```
enum EnumAllowingAlias {
  option allow_alias = true;
  UNKNOWN = 0;
  STARTED = 1;
  RUNNING = 1;
}
enum EnumNotAllowingAlias {
  UNKNOWN = 0;
  STARTED = 1;
  // RUNNING = 1;  // Uncommenting this line will cause a compile error inside Google and a warning message outside.
}

```

Enumerator constants must be in the range of a 32-bit integer. Since enum values use varint encoding on the wire, negative values are inefficient and thus not recommended. You can define enums within a message definition, as in the above example, or outside – these enums can be reused in any message definition in your .proto file. You can also use an enum type declared in one message as the type of a field in a different message, using the syntax MessageType.EnumType. 
枚举常量必须在一个32位的整数范围。由于枚举值使用变体的编码，负值是低效的，因此不推荐。你可以在一个消息定义枚举，在上面的例子中，或在–这些枚举可以在你的任何消息定义重复使用原文件。你也可以使用一个枚举类型在一个消息在不同的信息字段的类型声明，使用语法MessageType.EnumType。

When you run the protocol buffer compiler on a .proto that uses an enum, the generated code will have a corresponding enum for Java or C++, a special EnumDescriptor class for Python that's used to create a set of symbolic constants with integer values in the runtime-generated class. 
当您运行protocol buffer编译器上 .proto 使用枚举，生成的代码会有一个相应的java或C++的枚举，Python是用于在运行时的整数的值创建一组符号常量的特殊enumdescriptor类生成类。

During deserialization, unrecognized enum values will be preserved in the message, though how this is represented when the message is deserialized is language-dependent. In languages that support open enum types with values outside the range of specified symbols, such as C++ and Go, the unknown enum value is simply stored as its underlying integer representation. In languages with closed enum types such as Java, a case in the enum is used to represent an unrecognized value, and the underlying integer can be accessed with special accessors. In either case, if the message is serialized the unrecognized value will still be serialized with the message. 
在反序列化期间，无法识别的枚举值将保存在消息，尽管这是代表当消息反序列化是依赖语言。语言支持的开放枚举类型以外的指定符号的范围值，如C++和Go，未知的枚举值是存储作为其基础的整数表示。在封闭的枚举类型如java语言，在枚举的情况下被用来代表一个未知的价值，和基本的整数可以用特殊的访问器访问。在这两种情况下，如果消息序列化，则无法识别的值仍将以消息序列化。

For more information about how to work with message enums in your applications, see the generated code guide for your chosen language. 
为更多的信息关于如何在你的应用程序消息枚举的工作，看你选择的语言代码生成向导。

## Using Other Message Types
## 使用其他消息类型
You can use other message types as field types. For example, let's say you wanted to include Result messages in each SearchResponse message – to do this, you can define a Result message type in the same .proto and then specify a field of type Result in SearchResponse: 
可以使用其他消息类型作为字段类型。例如，让我们说你想有结果的消息在每个SearchResponse消息–要做到这一点，你可以定义在同一个结果消息类型.proto然后指定SearchResponse式的结果：

```
message SearchResponse {
  repeated Result results = 1;
}

message Result {
  string url = 1;
  string title = 2;
  repeated string snippets = 3;
}
```

### Importing Definitions
### 导入定义
In the above example, the Result message type is defined in the same file as SearchResponse – what if the message type you want to use as a field type is already defined in another .proto file? 
在上面的例子中，结果消息类型是在同一个文件中定义为SearchResponse–如果你想使用一个字段类型已经定义在另一个消息类型 .proto 文件吗？

You can use definitions from other .proto files by importing them. To import another .proto's definitions, you add an import statement to the top of your file: 
您可以使用其他 .proto 文件的定义导入它们。若要导入另一个 .proto 的定义，请在文件顶部添加导入语句：

```
import "myproject/other_protos.proto";
```

By default you can only use definitions from directly imported .proto files. However, sometimes you may need to move a .proto file to a new location. Instead of moving the .proto file directly and updating all the call sites in a single change, now you can put a dummy .proto file in the old location to forward all the imports to the new location using the import public notion. import public dependencies can be transitively relied upon by anyone importing the proto containing the import public statement. For example: 
默认情况下，只能使用直接导入的.proto文件的定义。但是，有时您可能需要将一个.proto文件移动到一个新位置。而不是直接移动.proto文件，并在一次更改中更新所有的调用站点，现在您可以在旧位置放置一个虚拟 .proto 文件，使用导入公共概念将所有导入转发到新位置。引入大众的依赖可以传递依赖任何人进口含进口公共声明原。例如:

```
// new.proto
// All definitions are moved here
```

```
// old.proto
// This is the proto that all clients are importing.
import public "new.proto";
import "other.proto";
```

```
// client.proto
import "old.proto";
// You use definitions from old.proto and new.proto, but not other.proto
```

The protocol compiler searches for imported files in a set of directories specified on the protocol compiler command line using the -I/--proto_path flag. If no flag was given, it looks in the directory in which the compiler was invoked. In general you should set the --proto_path flag to the root of your project and use fully qualified names for all imports.
该协议编译器搜索导入的文件集合中指定的目录的协议编译器命令行使用-I/--proto_path 标志。如果没有给定标志，它将在编译器调用的目录中查找。一般来说你应该设置 --proto_path标志你的项目的根和使用产品的完全限定名称。

### Using proto2 Message Types
### 使用proto2消息类型
It's possible to import proto2 message types and use them in your proto3 messages, and vice versa. However, proto2 enums cannot be used directly in proto3 syntax (it's okay if an imported proto2 message uses them). 
引入proto2消息类型和使用它们在您的proto3消息这是可能的，反之亦然。然而，proto2枚举是不能直接用于proto3语法（好吧如果引入proto2消息使用它们）。

## Nested Types
## 嵌套类型
You can define and use message types inside other message types, as in the following example – here the Result message is defined inside the SearchResponse message: 
你可以定义和使用在其他类型的消息的消息类型，如下面的示例–这里的结果消息在SearchResponse消息定义：

```
message SearchResponse {
  message Result {
    string url = 1;
    string title = 2;
    repeated string snippets = 3;
  }
  repeated Result results = 1;
}
```

If you want to reuse this message type outside its parent message type, you refer to it as Parent.Type: 
如果要在父消息类型之外重用此消息类型，则将其引用为Parent.Type：

```
message SomeOtherMessage {
  SearchResponse.Result result = 1;
}
```

You can nest messages as deeply as you like: 
你可以像你一样深深地嵌套消息：

```
message Outer {                  // Level 0
  message MiddleAA {  // Level 1
    message Inner {   // Level 2
      int64 ival = 1;
      bool  booly = 2;
    }
  }
  message MiddleBB {  // Level 1
    message Inner {   // Level 2
      int32 ival = 1;
      bool  booly = 2;
    }
  }
}
```

## Updating A Message Type
## 更新消息类型
If an existing message type no longer meets all your needs – for example, you'd like the message format to have an extra field – but you'd still like to use code created with the old format, don't worry! It's very simple to update message types without breaking any of your existing code. Just remember the following rules: 
如果现有的消息类型不再满足您的所有需求-例如，您希望消息格式有一个额外的字段-但您仍然喜欢使用旧格式创建的代码，不要担心！在不破坏任何现有代码的情况下更新消息类型非常简单。只要记住以下规则：
- Don't change the numeric tags for any existing fields. 
- 不要更改任何现有字段的数字标记。
- If you add new fields, any messages serialized by code using your "old" message format can still be parsed by your new generated code. You should keep in mind the default values for these elements so that new code can properly interact with messages generated by old code. Similarly, messages created by your new code can be parsed by your old code: old binaries simply ignore the new field when parsing. See the Unknown Fields section for details. 
- 如果添加新字段，则使用“旧”消息格式序列化的任何消息仍可以由新生成的代码解析。您应该记住这些元素的默认值，以便新代码可以正确地与旧代码生成的消息进行交互。同样，由新代码创建的消息可以通过旧代码解析：旧的二进制文件在解析时忽略新字段。详情见未知字段部分。
- Fields can be removed, as long as the tag number is not used again in your updated message type. You may want to rename the field instead, perhaps adding the prefix "OBSOLETE_", or make the tag reserved, so that future users of your .proto can't accidentally reuse the number. 
- 字段可以被删除，只要在更新的消息类型中不再使用标记号。你可能想改名的磁场相反，也许添加前缀“OBSOLETE_“，或使标签保留，以便将来你的用户.proto 不小心重用数字。
- int32, uint32, int64, uint64, and bool are all compatible – this means you can change a field from one of these types to another without breaking forwards- or backwards-compatibility. If a number is parsed from the wire which doesn't fit in the corresponding type, you will get the same effect as if you had cast the number to that type in C++ (e.g. if a 64-bit number is read as an int32, it will be truncated to 32 bits). 
- int32 uint32、int64、uint64 和 bool都是兼容的–这意味着你可以改变一场从这些类型到另一个没有打破向前或向后兼容性。如果一个数从线不适合在相应的类型解析，你会得到同样的效果，如果你把数的C型（例如，如果一个64位的数字读为Int32，它将被截断为32位）。
- sint32 and sint64 are compatible with each other but are not compatible with the other integer types. 
- sint32和sint64相互兼容但不与其他整数类型兼容。
- string and bytes are compatible as long as the bytes are valid UTF-8. 
- 字符串和字节兼容，只要是有效的UTF-8字节
- Embedded messages are compatible with bytes if the bytes contain an encoded version of the message. 
- 如果字节包含消息的编码版本，则嵌入消息与字节兼容。
- fixed32 is compatible with sfixed32, and fixed64 with sfixed64. 
- fixed32兼容sfixed32，和fixed64与sfixed64。
- enum is compatible with int32, uint32, int64, and uint64 in terms of wire format (note that values will be truncated if they don't fit). However be aware that client code may treat them differently when the message is deserialized: for example, unrecognized proto3 enum types will be preserved in the message, but how this is represented when the message is deserialized is language-dependent. Int fields always just preserve their value. 
- 枚举和Int32 UInt32、Int64兼容，并在线格式条款uint64（注意值将被截断，如果他们不配合）。但要注意，客户端代码可以区别对待当消息反序列化：例如，未知的proto3枚举类型将被保存在消息中，但这是代表当消息反序列化是依赖语言。int字段总是保存它们的值。

## Unknown Fields
## 未知的字段
Unknowns fields are well-formed protocol buffers serialized data representing fields that the parser does not recognize. For example, when an old binary parses data sent by a new binary with new fields, those new fields become unknown fields in the old binary. 
未知字段是格式良好的protocol buffer序列化的数据代表解析器不识别的字段。例如，当旧二进制解析新字段的新二进制发送的数据时，这些新字段将成为旧二进制中未知字段。

Proto3 implementations can parse messages with unknown fields successfully, however, implementations may or may not support preserving those unknown fields. You should not rely on unknown fields being preserved or dropped. For most Google protocol buffers implementations, unknown fields are not accessible in proto3 via the corresponding proto runtimes, and are dropped and forgotten at deserialization time. This is different behaviour to proto2, where unknown fields are always preserved and serialized along with the message. 
Proto3可以成功分析信息与未知字段，然而，实现可能或可能不支持保存那些未知的字段。您不应该依赖未知字段保存或删除。对于大多数谷歌protocol buffer的实现，未知字段的不可proto3通过相应原运行时在反序列化的时候被丢弃被遗忘。这是与proto2不同的行为，在未知的领域总是保留和序列化的信息一起。

## Any
## Any
The Any message type lets you use messages as embedded types without having their .proto definition. An Any contains an arbitrary serialized message as bytes, along with a URL that acts as a globally unique identifier for and resolves to that message's type. To use the Any type, you need to import google/protobuf/any.proto. 
任何消息类型允许您使用消息作为嵌入类型而不具有它们的.proto定义。任何包含一个任意序列化的消息作为字节，以及作为一个全局唯一标识符，并解析到该消息的类型的URL。使用任何类型，你需要引入google/protobuf/any.proto。

```
import "google/protobuf/any.proto";

message ErrorStatus {
  string message = 1;
  repeated google.protobuf.Any details = 2;
}
```

The default type URL for a given message type is type.googleapis.com/packagename.messagename. 
对于一个给定的消息类型的默认类型的URL是type.googleapis.com/packagename.messagename。

Different language implementations will support runtime library helpers to pack and unpack Any values in a typesafe manner – for example, in Java, the Any type will have special pack() and unpack() accessors, while in C++ there are PackFrom() and UnpackTo() methods: 
不同的语言实现，支持运行库来打包和解包的任何值在例如，类型安全的方式–java的任何类型都会有特别的pack()和unpack()访问器，而C++有packfrom()和unpackto()方法：

```
// Storing an arbitrary message type in Any.
NetworkErrorDetails details = ...;
ErrorStatus status;
status.add_details()->PackFrom(details);

// Reading an arbitrary message from Any.
ErrorStatus status = ...;
for (const Any& detail : status.details()) {
  if (detail.Is<NetworkErrorDetails>()) {
    NetworkErrorDetails network_error;
    detail.UnpackTo(&network_error);
    ... processing network_error ...
  }
}
```

<strong>Currently the runtime libraries for working with Any types are under development. </strong>

<strong>目前正在运行的任何类型的运行库正在开发中。 </strong>

If you are already familiar with proto2 syntax, the Any type replaces extensions. 
如果你已经熟悉proto2语法，任何替代的扩展。

## Oneof
## Oneof

If you have a message with many fields and where at most one field will be set at the same time, you can enforce this behavior and save memory by using the oneof feature. 
如果您有多个字段的消息，并且最多会同时设置一个字段，则可以通过使用 Oneof 功能强制执行此行为并保存内存。

Oneof fields are like regular fields except all the fields in a oneof share memory, and at most one field can be set at the same time. Setting any member of the oneof automatically clears all the other members. You can check which value in a oneof is set (if any) using a special case() or WhichOneof() method, depending on your chosen language. 
Oneoof字段就像一个共享内存中的所有字段一样的正则字段，最多可以同时设置一个字段。设置一个成员自动清除所有其他成员。你可以检查Oneof的值设置（如果有）使用一种特殊的case()或whichoneof()方法，取决于你选择的语言。

### Using Oneof
### 使用 Oneof
To define a oneof in your .proto you use the oneof keyword followed by your oneof name, in this case test_oneof: 
在你的定义Oneof。.proto你用Oneof关键词之后，你oneof名字，在这种情况下test_oneof：

```
message SampleMessage {
  oneof test_oneof {
    string name = 4;
    SubMessage sub_message = 9;
  }
}
```

You then add your oneof fields to the oneof definition. You can add fields of any type, but cannot use repeated fields. 
然后，将 oneof 字段添加到 oneof 定义中。可以添加任何类型的字段，但不能使用重复字段。

In your generated code, oneof fields have the same getters and setters as regular fields. You also get a special method for checking which value (if any) in the oneof is set. You can find out more about the oneof API for your chosen language in the relevant API reference. 
在你生成的代码，oneof 字段有相同的getter和setter规则字段。您还可以得到一个特殊的方法，检查其中的一个值（如果有的话）。您可以在相关API引用中找到关于所选语言的API的更多信息。

### Oneof Features
### Oneof 特性
- Setting a oneof field will automatically clear all other members of the oneof. So if you set several oneof fields, only the last field you set will still have a value. 
- 设置 oneof 字段将自动清除所有其他成员oneof。因此，如果设置多个 oneof 字段，则仅设置的最后一个字段仍具有值。
```
SampleMessage message;
message.set_name("name");
CHECK(message.has_name());
message.mutable_sub_message();   // Will clear name field.
CHECK(!message.has_name());
```
- If the parser encounters multiple members of the same oneof on the wire, only the last member seen is used in the parsed message. 
- 如果解析器遇到多个相同的oneof成员在线上，只有最后一个成员看到在分析的消息。
- A oneof cannot be repeated. 
- oneof 不能被重复。
- Reflection APIs work for oneof fields. 
- 反射API工作的oneof字段。
- If you're using C++, make sure your code doesn't cause memory crashes. The following sample code will crash because sub_message was already deleted by calling the set_name() method. 
- 如果使用C++，请确保代码不会导致内存崩溃。下面的示例代码将崩溃，因为sub_message已经通过调用set_name()方法删除。
```
SampleMessage message;
SubMessage* sub_message = message.mutable_sub_message();
message.set_name("name");      // Will delete sub_message
sub_message->set_...            // Crashes here
```
- Again in C++, if you Swap() two messages with oneofs, each message will end up with the other’s oneof case: in the example below, msg1 will have a sub_message and msg2 will have a name. 
- 再在C++语言中，如果你swap()两消息oneofs，每个消息将结束与对方的一个案例：在下面的例子中，会有一个sub_message msg1和msg2将有一个名字。
```
SampleMessage msg1;
msg1.set_name("name");
SampleMessage msg2;
msg2.mutable_sub_message();
msg1.swap(&msg2);
CHECK(msg1.has_sub_message());
CHECK(msg2.has_name());
```

### Backwards-compatibility issues
### 向后兼容的问题
Be careful when adding or removing oneof fields. If checking the value of a oneof returns None/NOT_SET, it could mean that the oneof has not been set or it has been set to a field in a different version of the oneof. There is no way to tell the difference, since there's no way to know if an unknown field on the wire is a member of the oneof. 
添加或移除oneof字段时要小心。如果检查一个oneof值回报不not_set，它可能意味着一个尚未建立或已被设置为在不同版本的一个领域。没有办法告诉区别，因为没有办法知道导线上的未知字段是否是其中oneof成员。

#### Tag Reuse Issues
#### 标签化的问题
- <strong>Move fields into or out of a oneof:</strong> You may lose some of your information (some fields will be cleared) after the message is serialized and parsed. 
- <strong>移动字段进入或退出 oneof ：</strong>您可能会丢失您的信息（某些字段将被清除）后，消息序列化和解析。
- <strong>Delete a oneof field and add it back: </strong>This may clear your currently set oneof field after the message is serialized and parsed. 
- <strong>删除Oneof字段并将其添加回：</strong>这可能会清除您当前设置的oneof字段后的消息序列化和解析。
- <strong>Split or merge oneof: </strong>>This has similar issues to moving regular fields. 
- <strong>拆分或合并 oneof：<strong>这与移动正则字段有类似的问题。

## Maps
## Maps
If you want to create an associative map as part of your data definition, protocol buffers provides a handy shortcut syntax: 
如果要创建关联映射作为数据定义的一部分，协议缓冲区提供了一个方便快捷的语法：

```
map<key_type, value_type> map_field = N;
```

...where the key_type can be any integral or string type (so, any scalar type except for floating point types and bytes). The value_type can be any type except another map. 
…在key_type可以是任何整数或字符串类型（因此，任何标量类型除了浮点类型和字节）。的value_type可以除任何类型的另一个地图。

So, for example, if you wanted to create a map of projects where each Project message is associated with a string key, you could define it like this: 
例如，如果您想创建一个项目图，其中每个项目消息都与一个字符串键关联，您可以定义如下：

```
map<string, Project> projects = 3;
```

- Map fields cannot be repeated.
- 不能重复映射字段。
- Wire format ordering and map iteration ordering of map values is undefined, so you cannot rely on your map items being in a particular order.
- 地图格式的线格式排序和地图迭代排序是未定义的，因此不能依赖于特定顺序的映射项。
- When generating text format for a .proto, maps are sorted by key. Numeric keys are sorted numerically.
- 当为原始文本生成文本格式时，地图按关键字排序。数值键排序数值。
- When parsing from the wire or when merging, if there are duplicate map keys the last key seen is used. When parsing a map from text format, parsing may fail if there are duplicate keys. 
- 当从线或合并时进行解析时，如果有重复的地图键，则使用最后一个键。当从文本格式解析map时，如果有重复键，解析可能会失败。

The generated map API is currently available for all proto3 supported languages. You can find out more about the map API for your chosen language in the relevant API reference. 
生成的地图API目前所有proto3支持的语言。您可以在有关API引用中找到有关所选语言的地图API的更多信息。

### Backwards compatibility
### 向后兼容
The map syntax is equivalent to the following on the wire, so protocol buffers implementations that do not support maps can still handle your data: 
地图语法相当于线上的下列语法，所以不支持映射的协议缓冲区实现仍然可以处理数据：
```
message MapFieldEntry {
  key_type key = 1;
  value_type value = 2;
}

repeated MapFieldEntry map_field = N;
```

## Packages
## 包
You can add an optional package specifier to a .proto file to prevent name clashes between protocol message types. 
你可以添加一个可选包符到 .proto 文件以防止协议消息类型名称之间的冲突。

```
package foo.bar;
message Open { ... }
```

You can then use the package specifier when defining fields of your message type: 
然后你可以使用包说明符定义的消息类型字段时：

```
message Foo {
  ...
  foo.bar.Open open = 1;
  ...
}
```

The way a package specifier affects the generated code depends on your chosen language:
一个包装说明符影响生成的代码的方式取决于你选择的语言：

- In <strong>C++</strong> the generated classes are wrapped inside a C++ namespace. For example, Open would be in the namespace foo::bar.
- 在 <strong>C++</strong> 中生成的类包在一个C++命名空间中。例如，打开将放在命名空间 foo::bar。
- In <strong>Java</strong>, the package is used as the Java package, unless you explicitly provide an option java_package in your .proto file.
- 在 <strong>Java</strong> 中，包装作为java包，除非你明确地提供一个选项java_package你的 .proto 文件。
- In <strong>Python</strong>, the package directive is ignored, since Python modules are organized according to their location in the file system.
- 在 <strong>Python</strong> 中，包指令被忽略，因为Python模块根据文件系统中的位置进行组织。
- In <strong>Go</strong>, the package is used as the Go package name, unless you explicitly provide an option go_package in your .proto file.
- 在 <strong>Go</strong> 中，包作为 G o包名称，除非你明确地提供一个选项go_package你 .proto 文件。
- In <strong>Ruby</strong>, the generated classes are wrapped inside nested Ruby namespaces, converted to the required Ruby capitalization style (first letter capitalized; if the first character is not a letter, PB_ is prepended). For example, Open would be in the namespace Foo::Bar.
- 在 <strong>Ruby</strong> 中，生成的类包裹里面嵌套命名空间转换为所需的Ruby，Ruby值风格（第一个字母大写；如果第一个字符是不字符，pb_前面）。例如，打开将在命名空间 Foo::Bar。
- In <strong>JavaNano</strong> the package is used as the Java package, unless you explicitly provide an option java_package in your .proto file.
- 在 <strong>JavaNano</strong> 中包使用的java包，除非你明确地提供一个选项java_package你.proto 文件。
- In <strong>C#</strong> the package is used as the namespace after converting to PascalCase, unless you explicitly provide an option csharp_namespace in your .proto file. For example, Open would be in the namespace Foo.Bar. 
- 在 <strong>C#</strong> 包转换到PascalCase后为命名空间的使用，除非你明确地提供一个选项csharp_namespace你 .proto 文件。例如，开放将命名空间foo.bar。

### Packages and Name Resolution
### 包和名称解析
Type name resolution in the protocol buffer language works like C++: first the innermost scope is searched, then the next-innermost, and so on, with each package considered to be "inner" to its parent package. A leading '.' (for example, .foo.bar.Baz) means to start from the outermost scope instead.
Protocol buffer语言中的类型名称解析工作如下：首先搜索最内层的范围，然后是下一个最内层，然后将每个包视为“内部”到其父包。一个先导'.'（例如，.foo.bar.Baz）指的是从最外层的范围开始代替。

The protocol buffer compiler resolves all type names by parsing the imported .proto files. The code generator for each language knows how to refer to each type in that language, even if it has different scoping rules. 
protocol buffer编译器通过解析导入的 .proto 文件来解析所有类型名称。每种语言的代码生成器知道如何是指语言每种类型，即使它有不同的作用域规则。


## Defining Services
## 服务的定义
If you want to use your message types with an RPC (Remote Procedure Call) system, you can define an RPC service interface in a .proto file and the protocol buffer compiler will generate service interface code and stubs in your chosen language. So, for example, if you want to define an RPC service with a method that takes your SearchRequest and returns a SearchResponse, you can define it in your .proto file as follows: 
如果要使用RPC（远程过程调用）系统的消息类型，可以在 .proto 文件中定义RPC服务接口，protocol buffer 编译器将在所选语言中生成服务接口代码和存根。所以，例如，如果你想用一个方法把你SearchRequest并返回一个SearchResponse定义一个RPC服务，你可以定义它在你原文件如下：

```
service SearchService {
  rpc Search (SearchRequest) returns (SearchResponse);
}
```

The most straightforward RPC system to use with protocol buffers is gRPC: a language- and platform-neutral open source RPC system developed at Google. gRPC works particularly well with protocol buffers and lets you generate the relevant RPC code directly from your .proto files using a special protocol buffer compiler plugin.
最简单的RPC系统使用protocol buffers是GRPC：语言和平台中立的开源RPC系统由谷歌开发。gRPC作品尤其protocol buffer，可以让你从你的直接生成相应的RPC代码.proto文件使用一种特殊的protocol buffer编译器插件。

If you don't want to use gRPC, it's also possible to use protocol buffers with your own RPC implementation. You can find out more about this in the Proto2 Language Guide.
如果你不想使用gRPC，使用protocol buffers与自己的RPC实现也有可能。你可以找到更多关于这在proto2语言指南。

There are also a number of ongoing third-party projects to develop RPC implementations for Protocol Buffers. For a list of links to projects we know about, see the third-party add-ons wiki page. 
也有一些正在进行的第三方项目，为协议缓冲区开发RPC实现。有关我们所知道的项目的链接列表，请参见第三方附加组件Wiki页面。

## JSON Mapping
## JSON 映射
Proto3 supports a canonical encoding in JSON, making it easier to share data between systems. The encoding is described on a type-by-type basis in the table below. 
Proto3支持JSON规范编码，使它更容易共享数据系统之间的。在下面的表格中，按类型按类型描述编码。

If a value is missing in the JSON-encoded data or if its value is null, it will be interpreted as the appropriate default value when parsed into a protocol buffer. If a field has the default value in the protocol buffer, it will be omitted in the JSON-encoded data by default to save space. An implementation may provide options to emit fields with default values in the JSON-encoded output. 
如果JSON编码的数据中缺少值，或者它的值为NULL，则当解析为协议缓冲区时，它将被解释为适当的默认值。如果字段在协议缓冲区中有默认值，则默认情况下将在JSON编码的数据中省略，以节省空间。一个实现可以提供在JSON编码输出中具有默认值的字段的选项。

|proto3|JSON|JSON example|Notes|
|:---|:---|:---|:---|
|message|object|{"fBar": v, "g": null, …}|Generates JSON objects. Message field names are mapped to lowerCamelCase and become JSON object keys. null is accepted and treated as the default value of the corresponding field type.[生成JSON对象。消息字段名称映射到lowercamelcase成JSON对象的key。NULL接受并处理为相应字段类型的默认值。]|
|enum|string|"FOO_BAR"|The name of the enum value as specified in proto is used.[名称的枚举值指定的proto用。]|
|map<K,V>|object|{"k": v, …}|All keys are converted to strings.[所有键都转换为字符串。]|
|repeated V|array|[v, …]|null is accepted as the empty list [].[NULL被接受为空列表[ ]。]|
|bool|true, false|true, false||
|string|string|"Hello World!"||
|bytes|base64 string|"YWJjMTIzIT8kKiYoKSctPUB+"|JSON value will be the data encoded as a string using standard base64 encoding with paddings. Either standard or URL-safe base64 encoding with/without paddings are accepted.[JSON值将被编码为base64编码的使用标准与芯子字符串数据。标准或URL安全BASE64编码/无芯子被接受。]|
|int32, fixed32, uint32|number|1, -10, 0|JSON value will be a decimal number. Either numbers or strings are accepted.[JSON值将是十进制数。接受数字或字符串。]|
|int64, fixed64, uint64|string|"1", "-10"|JSON value will be a decimal number. Either numbers or strings are accepted.[JSON值将是十进制数。接受数字或字符串。]|
|float, double|number|1.1, -10.0, 0, "NaN", "Infinity"|JSON value will be a number or one of the special string values "NaN", "Infinity", and "-Infinity". Either numbers or strings are accepted. Exponent notation is also accepted. [JSON值将是一个数字或一个特殊字符串值“NaN”，“Infinity”，和“-Infinity”。接受数字或字符串。指数符号也被接受。]|
|Any|object|{"@type": "url", "f": v, … }|If the Any contains a value that has a special JSON mapping, it will be converted as follows: {"@type": xxx, "value": yyy}. Otherwise, the value will be converted into a JSON object, and the "@type" field will be inserted to indicate the actual data type.[如果任何包含一个值，有一个特殊的JSON映射，它将被转换为：{“@type”：xxx，“value”：yyy }。否则，该值将被转换为JSON对象，并且将插入“@类型”字段以指示实际数据类型。]|
|Timestamp|string|"1972-01-01T10:00:20.021Z"|Uses RFC 3339, where generated output will always be Z-normalized and uses 0, 3, 6 or 9 fractional digits.[使用RFC 3339，在产生的输出会z-normalized和使用0，3，6或9的小数位数。]|
|Duration|string|"1.000340012s", "1s"|Generated output always contains 0, 3, 6, or 9 fractional digits, depending on required precision. Accepted are any fractional digits (also none) as long as they fit into nano-seconds precision.[生成的输出总是包含0，3，6，或9个小数位数，取决于所需的精度。接受的任何分数数字（也没有），只要他们适合纳米秒精度]|
|Struct|object|{ … }|Any JSON object. See struct.proto.[生成的输出总是包含0，3，6，或9分位数，丹妮的JSON对象。看到struct.proto。]|
|Wrapper types|various types|2, "2", "foo", true, "true", null, 0, …|Wrappers use the same representation in JSON as the wrapped primitive type, except that null is allowed and preserved during data conversion and transfer.[包装器使用JSON作为包原始类型的相同表示形式，但在数据转换和传输过程中允许null保留。]|
|FieldMask|string|"f.fooBar,h"|See fieldmask.proto.|
|ListValue|array|[foo, bar, …]||
|Value|value||Any JSON value|
|NullValue|null||JSON null|

## Options
## 选项
Individual declarations in a .proto file can be annotated with a number of options. Options do not change the overall meaning of a declaration, but may affect the way it is handled in a particular context. The complete list of available options is defined in google/protobuf/descriptor.proto.
在原始文件中的单个声明可以用许多选项来注释。选项不会改变声明的整体含义，但可能会影响其在特定上下文中处理的方式。可用选项的完整列表是google/protobuf/descriptor.proto定义。

Some options are file-level options, meaning they should be written at the top-level scope, not inside any message, enum, or service definition. Some options are message-level options, meaning they should be written inside message definitions. Some options are field-level options, meaning they should be written inside field definitions. Options can also be written on enum types, enum values, service types, and service methods; however, no useful options currently exist for any of these.
一些选项文件的选项，这意味着他们应该写在顶层范围内，没有任何消息，枚举，或服务定义。有些选项是消息级别选项，这意味着它们应该写在消息定义内。一些选项是字段级别选项，这意味着它们应该写在字段定义中。也可以选择写在枚举类型，枚举值，服务类型和服务方式；然而，对于这些目前不存在有用的选项。

Here are a few of the most commonly used options:
以下是一些最常用的选项：

- java_package (file option): The package you want to use for your generated Java classes. If no explicit java_package option is given in the .proto file, then by default the proto package (specified using the "package" keyword in the .proto file) will be used. However, proto packages generally do not make good Java packages since proto packages are not expected to start with reverse domain names. If not generating Java code, this option has no effect. 
- java_package（文件选项）：要用于生成java类的封装。如果没有明确的java_package选项给出的原文件，然后默认的原包装（指定使用“package”关键词在.proto文件）将被使用。然而.proto packages 一般不作很好的java包自原包都不会开始反向域名。如果没有生成java代码，此选项不起作用。
```
option java_package = "com.example.foo";
```
- java_multiple_files (file option): Causes top-level messages, enums, and services to be defined at the package level, rather than inside an outer class named after the .proto file.
- java_multiple_files（文件选项）：使顶层消息，枚举和服务可以在包级别上定义的，而不是在一个外部类命名的.proto文件。
```
option java_multiple_files = true;
```
- java_outer_classname (file option): The class name for the outermost Java class (and hence the file name) you want to generate. If no explicit java_outer_classname is specified in the .proto file, the class name will be constructed by converting the .proto file name to camel-case (so foo_bar.proto becomes FooBar.java). If not generating Java code, this option has no effect. 
- java_outer_classname（文件选项）：最外层的java类的类名（即文件名）你想生成。如果没有显式指定的java_outer_classname。原文件，类的名称将由转换。原文件名骆驼的情况（所以foo_bar.proto变成foobar。java）。如果没有生成java代码，此选项不起作用。
```
ption java_outer_classname = "Ponycopter";
```
- optimize_for (file option): Can be set to SPEED, CODE_SIZE, or LITE_RUNTIME. This affects the C++ and Java code generators (and possibly third-party generators) in the following ways: 
- optimize_for（文件选项）：可设置 SPEED，CODE_SIZE，或LITE_RUNTIME。这会影响C++和java代码生成器（可能还有第三方生成器）在以下几个方面：
  - SPEED (default): The protocol buffer compiler will generate code for serializing, parsing, and performing other common operations on your message types. This code is highly optimized.
  - SPEED（默认）：protocol buffer编译器会生成代码序列化，解析，和你的信息类型进行一些常用操作。此代码高度优化。
  - CODE_SIZE: The protocol buffer compiler will generate minimal classes and will rely on shared, reflection-based code to implement serialialization, parsing, and various other operations. The generated code will thus be much smaller than with SPEED, but operations will be slower. Classes will still implement exactly the same public API as they do in SPEED mode. This mode is most useful in apps that contain a very large number .proto files and do not need all of them to be blindingly fast.
  - CODE_SIZE：protocol buffer编译器会生成最小的类并将依靠共享，基于反射的代码来实现serialialization，解析，和其他各种操作。由此产生的代码将因此大大小于SPEED，但操作会慢。类仍将在SPEED模式中实现完全相同的公共API。这种模式，包含大量的应用程序是最有用的。原文件和不需要的都是如此之快。
  - LITE_RUNTIME: The protocol buffer compiler will generate classes that depend only on the "lite" runtime library (libprotobuf-lite instead of libprotobuf). The lite runtime is much smaller than the full library (around an order of magnitude smaller) but omits certain features like descriptors and reflection. This is particularly useful for apps running on constrained platforms like mobile phones. The compiler will still generate fast implementations of all methods as it does in SPEED mode. Generated classes will only implement the MessageLite interface in each language, which provides only a subset of the methods of the full Message interface.
  - LITE_RUNTIME：protocol buffer编译器会生成，仅仅依靠“lite”运行时库类（libprotobuf Lite代替libprotobuf）。lite的运行时远小于完整的库（约一个数量级较小），但省略了某些功能，如描述符和反射。这对于在诸如移动电话这样的受限平台上运行的应用程序特别有用。编译器仍然会在SPEED模式下生成所有方法的快速实现。生成的类只能实现各语言MessageLite接口，它只提供完整的信息接口的方法的子集。
  ```
  option optimize_for = CODE_SIZE;
  ```
- cc_enable_arenas (file option): Enables arena allocation for C++ generated code. 
- cc_enable_arenas（文件选项）：C++生成的代码　使用arena allocation 
- objc_class_prefix (file option): Sets the Objective-C class prefix which is prepended to all Objective-C generated classes and enums from this .proto. There is no default. You should use prefixes that are between 3-5 uppercase characters as recommended by Apple. Note that all 2 letter prefixes are reserved by Apple. 
- cc_enable_arenobjc_class_prefix（文件选项）：集Objective-C类前缀，前面所有的Objective-C生成的类和枚举于此 .proto 。没有默认。你应该使用由苹果推荐的3-5个大写字符的前缀。注意所有2个字母前缀都由苹果保留。
- deprecated (field option): If set to true, indicates that the field is deprecated and should not be used by new code. In most languages this has no actual effect. In Java, this becomes a @Deprecated annotation. In the future, other language-specific code generators may generate deprecation annotations on the field's accessors, which will in turn cause a warning to be emitted when compiling code which attempts to use the field. If the field is not used by anyone and you want to prevent new users from using it, consider replacing the field declaration with a reserved statement. 
- deprecated（字段选项）：如果设置为true，表示该字段是过时的和不应该用新的代码使用。在大多数语言中，这没有实际效果。在java，这已成为一个@Deprecated的注释。在未来，发电机等特定语言的代码可能产生贬低注释字段的访问器，这将反过来导致发射时编译代码试图使用领域的警告。如果字段不被任何人使用，并且希望阻止新用户使用该字段，请考虑使用保留语句替换字段声明。
```
int32 old_field = 6 [deprecated=true];
```

### Custom Options
### 自定义选项
Protocol Buffers also allows you to define and use your own options. This is an advanced feature which most people don't need. If you do think you need to create your own options, see the Proto2 Language Guide for details. Note that creating custom options uses extensions, which are permitted only for custom options in proto3. 
Protocol Buffers 还允许您定义和使用您自己的选项。这是大多数人不需要的高级特性。如果你认为你需要创建你自己的选择，详见proto2语言指南。请注意，创建自定义选项使用的扩展，这是只有在proto3自定义选项允许。

## Generating Your Classes
## 生成你的类
To generate the Java, Python, C++, Go, Ruby, JavaNano, Objective-C, or C# code you need to work with the message types defined in a .proto file, you need to run the protocol buffer compiler protoc on the .proto. If you haven't installed the compiler, download the package and follow the instructions in the README. For Go, you also need to install a special code generator plugin for the compiler: you can find this and installation instructions in the golang/protobuf repository on GitHub. 
生成的Java、Python、C++、Go、Ruby、JavaNano，Objective-C，C#代码你需要在一个定义的消息类型工作 .proto 文件，你需要在运行的protocol buffer编译 .proto 协议。如果没有安装编译器，请下载包并按照说明书中的说明执行。Go，你还需要安装一个特殊的编译器代码生成器插件：你可以找到和安装说明在golang/protobuf库GitHub上。

The Protocol Compiler is invoked as follows: 
协议编译器被调用如下：

```
protoc --proto_path=IMPORT_PATH --cpp_out=DST_DIR --java_out=DST_DIR --python_out=DST_DIR --go_out=DST_DIR --ruby_out=DST_DIR --javanano_out=DST_DIR --objc_out=DST_DIR --csharp_out=DST_DIR path/to/file.proto
```

- IMPORT_PATH specifies a directory in which to look for .proto files when resolving import directives. If omitted, the current directory is used. Multiple import directories can be specified by passing the --proto_path option multiple times; they will be searched in order. -I=IMPORT_PATH can be used as a short form of --proto_path. 
- IMPORT_PATH指定的目录中寻找 .proto 文件导入指令时解决。如果省略，则使用当前目录。多导入目录可以通过-- proto_path选项多次指定；他们将搜索顺序。-I=IMPORT_PATH可以作为短期的形式proto_path。
- You can provide one or more output directives: 
- 您可以提供一个或多个输出指令：
  - --cpp_out generates C++ code in DST_DIR. See the C++ generated code reference for more. 
  - --cpp_out生成C++代码DST_DIR。查看C++生成的代码参考更多。
  - --java_out generates Java code in DST_DIR. See the Java generated code reference for more. 
  - --java_out生成java代码在DST_DIR。看到Java生成更多的代码参考。
  - --python_out generates Python code in DST_DIR. See the Python generated code reference for more.
  - --python_out生成Python代码在DST_DIR。查看Python生成的代码参考更多。
  - --go_out generates Go code in DST_DIR. See the Go generated code reference for more.
  - --go_out生成代码DST_DIR去。查看GO生成的代码引用更多。
  - --ruby_out generates Ruby code in DST_DIR. Ruby generated code reference is coming soon!
  - --ruby_out产生Ruby代码在DST_DIR。Ruby生成代码参考即将推出！
  - --javanano_out generates JavaNano code in DST_DIR. The JavaNano code generator has a number of options you can use to customize the generator output: you can find out more about these in the generator README. JavaNano generated code reference is coming soon!
  - --javanano_out产生javanano代码DST_DIR。的javanano代码生成器有许多选项可以自定义生成器输出：你可以找到更多关于这些生成器的自述。javanano即将生成的代码参考！
  - --objc_out generates Objective-C code in DST_DIR. See the Objective-C generated code reference for more.
  - --objc_out Objective-C代码生成DST_DIR。看到Objective-C生成更多的代码参考。
  - --csharp_out generates C# code in DST_DIR. See the C# generated code reference for more.
  - --csharp_out生成C#代码DST_DIR。看到C#产生更多的代码参考。
  - --php_out generates PHP code in DST_DIR. See the PHP generated code reference for more. 
  - --php_out生成PHP代码在DST_DIR。查看PHP生成的代码参考更多。

As an extra convenience, if the DST_DIR ends in .zip or .jar, the compiler will write the output to a single ZIP-format archive file with the given name. .jar outputs will also be given a manifest file as required by the Java JAR specification. Note that if the output archive already exists, it will be overwritten; the compiler is not smart enough to add files to an existing archive.
作为一个额外的方便，如果DST_DIR结束。zip或jar，编译器将输出写入到一个ZIP格式的档案文件的名字。。罐的输出也将给出一个清单文件的java jar规范要求。注意，如果输出存档已经存在，它将被覆盖；编译器不够聪明，无法将文件添加到现有存档中。
- You must provide one or more .proto files as input. Multiple .proto files can be specified at once. Although the files are named relative to the current directory, each file must reside in one of the IMPORT_PATHs so that the compiler can determine its canonical name. 
- 必须提供一个或多个原始文件作为输入。可以同时指定多个原始文件。虽然文件被命名为相对于当前目录，每个文件必须驻留在使编译器可以确定其名称的import_paths。