## gRPC基础



传统的服务可能开发的语言多种多样，如果我们要调用对方的程序，那么传统的方式是通过提供一个api接口来进行获取结果，也就是说基于http

![1581141748338](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1581141748338.png)

http协议是重量级的，它需要很多协议，而rpc是轻量级的，而且他们都是基于tcp协议的。

学习网址：<http://doc.oschina.net/grpc?t=58008>

#### gRPC是什么？

​	在 gRPC 里**客户端**应用可以像调用本地对象一样直接调用另一台不同的机器上**服务端**应用的方法，使得您能够更容易地创建分布式应用和服务。与许多 RPC 系统类似，gRPC 也是基于以下理念：定义一个*服务*，指定其能够被远程调用的方法（包含参数和返回类型）。在服务端实现这个接口，并运行一个 gRPC 服务器来处理客户端调用。在客户端拥有一个*存根*能够像服务端一样的方法。

​	gRPC 客户端和服务端可以在多种环境中运行和交互，例如用 java 写一个服务端， 可以用 go 语言写客户端调用 。

![1581142378047](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1581142378047.png)

#### 为什么使用gRPC？

有了gRPC，我们可以一次性的在一个.protp文件中定义服务并使用任何支持它的语言去实现客户端和服务器，反过来，它们可以在各种环境中。



#### gRPC环境搭建

官网的安装命令:

```
go get -u google.golang.org/grpc
```

如果上面的代码下载不了的话，可以使用下面的语句，在cmd中下载：

```
# 如果已经安装了proto和protoc-gen-go的话就不用安装了，如果是win就分开执行
go get -u github.com/golang/protobuf/{proto,protoc-gen-go}

# 下载grpc-go
git clone https://github.com/grpc/grpc-go.git %GOPATH%/src/google.golang.org/grpc

# 下载golang/net
git clone https://github.com/golang/net.git %GOPATH%/src/golang.org/x/net

# 下载golang/text
git clone https://github.com/golang/text.git %GOPATH%/src/golang.org/x/text

# 下载go-genproto
git clone https://github.com/google/go-genproto.git %GOPATH%/src/google.golang.org/genproto

# 安装
cd $GOPATH/src/
go install google.golang.org/grpc
```



在win平台下载protoc插件网站,不同版本不同下载：

https://github.com/protocolbuffers/protobuf/releases



#### **gRPC** **与** **Protobuf** **介绍**

- 微服务架构中，由于每个服务对应的代码库是独立运行的，无法直接调用，彼此间 的通信就是个大问题 

- gRPC 可以实现微服务，将大的项目拆分为多个小且独立的业务模块，也就是服务， 各服务间使用高效的 protobuf 协议进行 RPC 调用，gRPC 默认使用 protocol buffers， 这是 google 开源的一套成熟的结构数据序列化机制（当然也可以使用其他数据格式如 JSON） 

- 可以用 proto files 创建 gRPC 服务，用 message 类型来定义方法参数和返回类型



#### protocol buffers 介绍

**protocol buffers简介**：protocol buffers 是由Google公司启动的开源项目，并且项目的设计开发编码权由Google完成，项目中被用于网络上传输交通信息

**protocol buffers是如何工作**： 首先需要在一个.proto文件中定义你需要做序列化的数据结构信息，每个protocol buffer 信息是一小段逻辑记录，包含一系列的键值对。这里有个非常简单的.proto文件定义了用户的一些服务。

![1581173564243](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1581173564243.png)



#### Protobuf语法和规范

##### 规范：

- 文件以.proto 做为文件后缀，除结构定义外的语句以分号结尾 ，官方说明最好以"包名.服务.proto"

-  结构定义可以包含：message、service、enum 

- rpc 方法定义结尾的分号可有可无 

- Message 命名采用驼峰命名方式，字段命名采用小写字母加下划线分隔方式 



#### Protobuf语法详解

学习网址：<https://www.cnblogs.com/tohxyblog/p/8974763.html>

​	先来看一个非常简单的例子。假设你想定义一个“搜索请求”的消息格式，每一个请求含有一个查询字符串、你感兴趣的查询结果所在的页数，以及每一页多少条查询结果。可以采用如下的方式来定义消息类型的.proto文件了：

```python
syntax = "proto3";
 
message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3;
}

 # 文件的第一行指定了你正在使用proto3语法：如果你没有指定这个，编译器会使用proto2。这个指定语法行必须是文件的非空非注释的第一个行。
 # SearchRequest消息格式有3个字段，在消息中承载的数据分别对应于每一个字段。其中每个字段都有一个名字和一种类型。
```

##### 指定字段类型

​	在上面的例子中，所有字段都是标量类型：两个整型（page_number和result_per_page），一个string类型（query）。当然，你也可以为字段指定其他的合成类型，包括枚举（enumerations）或其他消息类型。

##### 分配标号

​	正如你所见，在消息定义中，每个字段都有唯一的一个数字标识符。这些标识符是用来在消息的二进制格式中识别各个字段的，一旦开始使用就不能够再改变。注：[1,15]之内的标识号在编码的时候会占用一个字节。[16,2047]之内的标识号则占用2个字节。所以应该为那些频繁出现的消息元素保留 [1,15]之内的标识号。切记：要为将来有可能添加的、频繁出现的标识号预留一些标识号。

最小的标识号可以从1开始，最大到2^29 - 1, or 536,870,911。不可以使用其中的[19000－19999]（ (从FieldDescriptor::kFirstReservedNumber 到 FieldDescriptor::kLastReservedNumber)）的标识号， Protobuf协议实现中对这些进行了预留。如果非要在.proto文件中使用这些预留标识号，编译时就会报警。同样你也不能使用早期[保留](https://developers.google.com/protocol-buffers/docs/proto3?hl=zh-cn#reserved)的标识号。

##### 指定字段规则

所指定的消息字段修饰符必须如下之一：

- singular：一个格式良好的消息应该有0个或者1个这种字段（但是不能超过1个）。

- repeated：在一个格式良好的消息中，这种字段可以重复任意多次（包括0次）。重复的值的顺序会被保留。

  在proto3中，repeated的标量域默认情况虾使用packed。

  你可以了解更多的pakced属性在[Protocol Buffer 编码](https://developers.google.com/protocol-buffers/docs/encoding?hl=zh-cn#packed)

##### 添加更多的消息类型

在一个.proto文件中可以定义多个消息类型。在定义多个相关的消息的时候，这一点特别有用——例如，如果想定义与SearchResponse消息类型对应的回复消息格式的话，你可以将它添加到相同的.proto文件中，如：

```
message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3;
}
 
message SearchResponse {
 ...
}

注意：向.proto文件添加注释，可以使用C/C++/java风格的双斜杠（//） 语法格式
```

##### 从.proto文件生成了什么？

当用protocol buffer编译器来运行.proto文件时，编译器将生成所选择语言的代码，这些代码可以操作在.proto文件中定义的消息类型，包括获取、设置字段值，将消息序列化到一个输出流中，以及从一个输入流中解析消息。

- 对C++来说，编译器会为每个.proto文件生成一个.h文件和一个.cc文件，.proto文件中的每一个消息有一个对应的类。
- 对Java来说，编译器为每一个消息类型生成了一个.java文件，以及一个特殊的Builder类（该类是用来创建消息类接口的）。
- 对Python来说，有点不太一样——Python编译器为.proto文件中的每个消息类型生成一个含有静态描述符的模块，，该模块与一个元类（metaclass）在运行时（runtime）被用来创建所需的Python数据访问类。
- 对go来说，编译器会位每个消息类型生成了一个.pd.go文件。
- 对于Ruby来说，编译器会为每个消息类型生成了一个.rb文件。
- javaNano来说，编译器输出类似域java但是没有Builder类
- 对于Objective-C来说，编译器会为每个消息类型生成了一个pbobjc.h文件和pbobjcm文件，.proto文件中的每一个消息有一个对应的类。
- 对于C#来说，编译器会为每个消息类型生成了一个.cs文件，.proto文件中的每一个消息有一个对应的类。

你可以从如下的文档链接中获取每种语言更多API(proto3版本的内容很快就公布)。[API Reference](https://developers.google.com/protocol-buffers/docs/reference/overview?hl=zh-cn)

##### 枚举

当需要定义一个消息类型的时候，可能想为一个字段指定某“预定义值序列”中的一个值。例如，假设要为每一个SearchRequest消息添加一个 corpus字段，而corpus的值可能是UNIVERSAL，WEB，IMAGES，LOCAL，NEWS，PRODUCTS或VIDEO中的一个。 其实可以很容易地实现这一点：通过向消息定义中添加一个枚举（enum）并且为每个可能的值定义一个常量就可以了。

在下面的例子中，在消息格式中添加了一个叫做Corpus的枚举类型——它含有所有可能的值 ——以及一个类型为Corpus的字段：

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

##### 更新一个消息类型

如果一个已有的消息格式已无法满足新的需求——如，要在消息中添加一个额外的字段——但是同时旧版本写的代码仍然可用。不用担心！更新消息而不破坏已有代码是非常简单的。在更新时只要记住以下的规则即可。

- 不要更改任何已有的字段的数值标识。
- 如果你增加新的字段，使用旧格式的字段仍然可以被你新产生的代码所解析。你应该记住这些元素的默认值这样你的新代码就可以以适当的方式和旧代码产生的数据交互。相似的，通过新代码产生的消息也可以被旧代码解析：只不过新的字段会被忽视掉。注意，未被识别的字段会在反序列化的过程中丢弃掉，所以如果消息再被传递给新的代码，新的字段依然是不可用的（这和proto2中的行为是不同的，在proto2中未定义的域依然会随着消息被序列化）
- 非required的字段可以移除——只要它们的标识号在新的消息类型中不再使用（更好的做法可能是重命名那个字段，例如在字段前添加“OBSOLETE_”前缀，那样的话，使用的.proto文件的用户将来就不会无意中重新使用了那些不该使用的标识号）。
- int32, uint32, int64, uint64,和bool是全部兼容的，这意味着可以将这些类型中的一个转换为另外一个，而不会破坏向前、 向后的兼容性。如果解析出来的数字与对应的类型不相符，那么结果就像在C++中对它进行了强制类型转换一样（例如，如果把一个64位数字当作int32来 读取，那么它就会被截断为32位的数字）。
- sint32和sint64是互相兼容的，但是它们与其他整数类型不兼容。
- string和bytes是兼容的——只要bytes是有效的UTF-8编码。
- 嵌套消息与bytes是兼容的——只要bytes包含该消息的一个编码过的版本。
- fixed32与sfixed32是兼容的，fixed64与sfixed64是兼容的。
- 枚举类型与int32，uint32，int64和uint64相兼容（注意如果值不相兼容则会被截断），然而在客户端反序列化之后他们可能会有不同的处理方式，例如，未识别的proto3枚举类型会被保留在消息中，但是他的表示方式会依照语言而定。int类型的字段总会保留他们的

##### Any

​	Any类型消息允许你在没有指定他们的.proto定义的情况下使用消息作为一个嵌套类型。一个Any类型包括一个可以被序列化bytes类型的任意消息，以及一个URL作为一个全局标识符和解析消息类型。为了使用Any类型，你需要导入`import google/protobuf/any.proto`

```
import "google/protobuf/any.proto";
 
message ErrorStatus {
  string message = 1;
  repeated google.protobuf.Any details = 2;
}
```

