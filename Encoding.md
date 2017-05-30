# Encoding
# 编码

This document describes the binary wire format for protocol buffer messages. You don't need to understand this to use protocol buffers in your applications, but it can be very useful to know how different protocol buffer formats affect the size of your encoded messages. 
本文档描述protocol buffer消息的二进制格式。您不需要在应用程序中使用protocol buffer来理解这一点，但是了解不同的protocol buffer格式对编码消息大小的影响会非常有用。

### A Simple Message
### 一个简单的消息
Let's say you have the following very simple message definition: 
假设你有以下非常简单的消息定义：

```
message Test1 {
  required int32 a = 1;
}
```

In an application, you create a Test1 message and set a to 150. You then serialize the message to an output stream. If you were able to examine the encoded message, you'd see three bytes: 
在应用程序中，您创建一个Test1消息并设置为150。然后你将消息序列化到一个输出流。如果你能够检查编码的消息，你会看到三字节：

```
08 96 01
```

So far, so small and numeric – but what does it mean? Read on... 
到目前为止，这么小和数字-但它是什么意思？读一读…

## Base 128 Varints
## Base 128 Varints
To understand your simple protocol buffer encoding, you first need to understand varints. Varints are a method of serializing integers using one or more bytes. Smaller numbers take a smaller number of bytes. 
了解你的简单协议缓冲区编码，首先你需要了解varints。varints是序列化整数使用一个或多个字节的方法。较小的数字占用较小的字节数。

Each byte in a varint, except the last byte, has the most significant bit (msb) set – this indicates that there are further bytes to come. The lower 7 bits of each byte are used to store the two's complement representation of the number in groups of 7 bits, least significant group first. 
在一个varint的每个字节，除了最后一个字节，具有最高有效位（msb）设置–这表明有进一步的字节来。较低的7位的每个字节用于存储两个补码表示的数目在组的7位，最低有效组第一。

So, for example, here is the number 1 – it's a single byte, so the msb is not set: 
所以，例如，这里是数字1–是单字节，所以 msb 没有设置：

```
0000 0001
```

And here is 300 – this is a bit more complicated: 
这里是300 -这是一个有点复杂：

```
1010 1100 0000 0010
```

How do you figure out that this is 300? First you drop the msb from each byte, as this is just there to tell us whether we've reached the end of the number (as you can see, it's set in the first byte as there is more than one byte in the varint): 
你怎么知道这是300？首先你把每个字节的最高位移除，它仅仅是在告诉我们，我们是否已经到了数字结尾（你可以看到，它的第一个字节中有多于一个字节的varint）：

```
1010 1100 0000 0010
→ 010 1100  000 0010
```

You reverse the two groups of 7 bits because, as you remember, varints store numbers with the least significant group first. Then you concatenate them to get your final value: 
你反两组7位，因为，你还记得，varints店数最小有意义的小组在前。然后你将他们获得的最终值：

```
000 0010  010 1100
→  000 0010 ++ 010 1100
→  100101100
→  256 + 32 + 8 + 4 = 300
```

## Message Structure
## 消息结构
As you know, a protocol buffer message is a series of key-value pairs. The binary version of a message just uses the field's number as the key – the name and declared type for each field can only be determined on the decoding end by referencing the message type's definition (i.e. the .proto file). 
如您所知，protocol buffer消息是一系列键值对。消息的二进制版本只使用字段的编号作为键--每个字段的名称和声明类型只能通过引用消息类型的定义（即 .proto 文件）来确定解码端

When a message is encoded, the keys and values are concatenated into a byte stream. When the message is being decoded, the parser needs to be able to skip fields that it doesn't recognize. This way, new fields can be added to a message without breaking old programs that do not know about them. To this end, the "key" for each pair in a wire-format message is actually two values – the field number from your .proto file, plus a wire type that provides just enough information to find the length of the following value. 
当消息被编码时，键和值被连接到字节流中。当消息被解码时，分析器需要能够跳过它不识别的字段。这样，新的字段可以被添加到消息中而不破坏不知道它们的旧程序。为此，“线”中的每一对“键”实际上是两个值--从你的 .proto 文件中提取的字段号，加上一个导线类型，只提供足够的信息来查找下列值的长度。

The available wire types are as follows: 
可用线类型如下：
|Type|Meaning|Used For|
|:--|:--|:--|
|0|Varint|int32, int64, uint32, uint64, sint32, sint64, bool, enum|
|1|64-bit|fixed64, sfixed64, double|
|2|Length-delimited|string, bytes, embedded messages, packed repeated fields|
|3|Start group|groups (deprecated)|
|4|End group|groups (deprecated)|
|5|32-bit|fixed32, sfixed32, float|

Each key in the streamed message is a varint with the value (field_number << 3) | wire_type – in other words, the last three bits of the number store the wire type. 
在传输消息每个关键是一个价值的变体（field_number << 3）| wire_type–换句话说，数字的存储线式最后三位。

Now let's look at our simple example again. You now know that the first number in the stream is always a varint key, and here it's 08, or (dropping the msb): 
现在让我们再看看我们简单的例子。你现在知道流中的第一个数始终是一个变体的关键，这里是08，或（删除 msb）:

```
000 1000
```

You take the last three bits to get the wire type (0) and then right-shift by three to get the field number (1). So you now know that the tag is 1 and the following value is a varint. Using your varint-decoding knowledge from the previous section, you can see that the next two bytes store the value 150. 
您采取最后三位得到电线类型（0），然后右移由三得到字段编号（1）。所以你现在知道，标签是1，下面的值是一个变体。用你的变体从以前的部分解码的知识，你可以看到两个字节存储的值为150。

```
96 01 = 1001 0110  0000 0001
       → 000 0001  ++  001 0110 (drop the msb and reverse the groups of 7 bits)
       → 10010110
       → 2 + 4 + 16 + 128 = 150
```

## More Value Types
## 更多的值类型

### Signed Integers
### Signed Integers
As you saw in the previous section, all the protocol buffer types associated with wire type 0 are encoded as varints. However, there is an important difference between the signed int types (sint32 and sint64) and the "standard" int types (int32 and int64) when it comes to encoding negative numbers. If you use int32 or int64 as the type for a negative number, the resulting varint is always ten bytes long – it is, effectively, treated like a very large unsigned integer. If you use one of the signed types, the resulting varint uses ZigZag encoding, which is much more efficient. 
正如你在上一节中看到的，所有的 protocol buuffer 类型与线0型相关的编码为varints。然而，有符号整数类型之间的重要区别（sint32和sint64）和“标准”的int类型（Int32和Int64）负数编码。如果你使用Int32或Int64为负数的类型，产生的变体总是十字节长的–是有效的，像对待一个非常大的无符号整数。如果你使用一个符号的类型，产生的变体使用字形编码，这是更有效的。

ZigZag encoding maps signed integers to unsigned integers so that numbers with a small absolute value (for instance, -1) have a small varint encoded value too. It does this in a way that "zig-zags" back and forth through the positive and negative integers, so that -1 is encoded as 1, 1 is encoded as 2, -2 is encoded as 3, and so on, as you can see in the following table: 
ZigZag编码的map符号整数无符号整数，数字与一个小的绝对值（例如，-11）有一个小的varint编码值。这是一种“zig-zags”通过正负整数来回，所以-1编码为1，1编码为2，-2编码为3，等等，你可以看到下面的表格：

|Signed Original|Encoded As|
|:--|:--|
|0|0|
|-1|1|
|1|2|
|-2|3|
|2147483647|4294967294|
|-2147483648|4294967295|

In other words, each value n is encoded using 
换句话说，每个值n使用

```
(n << 1) ^ (n >> 31)
```

for sint32 s, or
对于sint32，或

```
(n << 1) ^ (n >> 63)
```

for the 64-bit version. 
对于64位版本。

Note that the second shift – the (n >> 31) part – is an arithmetic shift. So, in other words, the result of the shift is either a number that is all zero bits (if n is positive) or all one bits (if n is negative).
请注意，第二个移位（（31））部分是算术移位。因此，换句话说，移位的结果是一个数字，即所有的零位（如果n是正）或所有的位（如果n是负）。

When the sint32 or sint64 is parsed, its value is decoded back to the original, signed version. 
当sint32或sint64解析，其价值是解码回到原来的，带符号。

### Non-varint Numbers
### Non-varint 数字
Non-varint numeric types are simple – double and fixed64 have wire type 1, which tells the parser to expect a fixed 64-bit lump of data; similarly float and fixed32 have wire type 5, which tells it to expect 32 bits. In both cases the values are stored in little-endian byte order. 
Non-varint数值类型是简单的–double和fixed64有编码为型1，它告诉解析器预期固定的64位块数据；同样，float　和　fixed32有编码为型５，并告诉它为32位。在这两种情况下的值存储在little-endian字节顺序。

### Strings
### Strings
A wire type of 2 (length-delimited) means that the value is a varint encoded length followed by the specified number of bytes of data. 
编码类型2（length-delimited）意味着值是一个varint编码长度的指定字节数的数据。

```
message Test2 {
  required string b = 2;
}
```

Setting the value of b to "testing" gives you: 
将B的值设置为“testing”给你：

```
12 07 74 65 73 74 69 6e 67

----- 74 65 73 74 69 6e 67
```

The red bytes are the UTF8 of "testing". The key here is 0x12 → tag = 2, type = 2. The length varint in the value is 7 and lo and behold, we find seven bytes following it – our string. 
红色是UTF8字节“testing”。这里的关键是0x12→tag= 2，type = 2。在值的长度为７的varint，我们找到七个字节之后–字符串。

## Embedded Messages
## 嵌入式信息
Here's a message definition with an embedded message of our example type, Test1: 
这是一个与我们的样列嵌入式消息定义, Test1：

```
message Test3 {
  required Test1 c = 3;
}
```
And here's the encoded version, again with the Test1's a field set to 150: 
这里的编码版本，再次与Test1的字段设置为150：
```
1a 03 08 96 01
```
As you can see, the last three bytes are exactly the same as our first example (08 96 01), and they're preceded by the number 3 – embedded messages are treated in exactly the same way as strings (wire type = 2). 
正如你所看到的，最后三个字节与我们的第一个例子（08 96 01）完全一样，它们前面的数字是-嵌入的消息被处理的方式完全相同的字符串（编码类型= 2）。

## Optional And Repeated Elements
## 可选和重复元素