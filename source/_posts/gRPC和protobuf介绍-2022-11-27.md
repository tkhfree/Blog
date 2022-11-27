---

title: gRPC和protobuf介绍
date: 2022-11-27 12:36
tags:
  -  grpc
  -  protobuf

categories:
  - 应用：应用笔记

comments: true
toc: true

---
# Protocol Buffer

是一种序列化成二进制的数据缓冲编码协议

### 定义消息类型

```protobuf
syntax = "proto3"

message SearchRequest{
	String query = 1;
	int32 page_num = 2;
	int32 result_per_page = 3;
}
//分配字段类型、字段编号
//消息字段主要由两个规则：singular0次或者1次、repeated重复任意次，默认是repeated

message SearchResponse{
	string result = 1;
}

//可以保留字段，如果直接删除字段，以后用户在对消息类型进行更新时可能会重复使用删除掉的字段号，如果再次加载旧版本就会造成冲突
message foo{
	reserved 2,3,4 to 9;
	reserved "query";
}
```

如何使用.proto文件

```powershell
protoc --proto_path=IMPORT_PATH --cpp_out=DST_DIR --java_out=DST_DIR --python_out=DST_DIR --go_out=DST_DIR --ruby_out=DST_DIR --objc_out=DST_DIR --csharp_out=DST_DIR path/to/file.proto
```

使用protoc编译proto文件，将会以选择的语言输出代码，代码内容主要是根据你编写的message类型获取和设置字段，序列化输出以及解析序列化的输入。

- For **C++**, the compiler generates a `.h` and `.cc` file from each `.proto`, with a class for each message type described in your file.
- For **Java**, the compiler generates a `.java` file with a class for each message type, as well as a special `Builder` classes for creating message class instances.
- **Python** is a little different – the Python compiler generates a module with a static descriptor of each message type in your `.proto`, which is then used with a *metaclass* to create the necessary Python data access class at runtime.
- For **Go**, the compiler generates a `.pb.go` file with a type for each message type in your file.
- For **Ruby**, the compiler generates a `.rb` file with a Ruby module containing your message types.
- For **Objective-C**, the compiler generates a `pbobjc.h` and `pbobjc.m` file from each `.proto`, with a class for each message type described in your file.
- For **C#**, the compiler generates a `.cs` file from each `.proto`, with a class for each message type described in your file.
- For **Dart**, the compiler generates a `.pb.dart` file with a class for each message type in your file.

### 枚举

```protobuf
message SearchRequest{
	string query = 1;
	enum Corpus{
		UNIBERSAL = 0; //第一个值必须为0
		WEB = 1;
		IMAGEs = 2;
	}
	Corpus corpus = 2;
}
```

也可以使用别名，将`allow_alias`选项设置为`true`

```protobuf
message MyMessage1{
	enum Enum1{
		option allow_alias = true;
		UNKnow = 0;
		STATUS = 1;
		RUNNING = 1;
	}
}
```

### 使用其他消息类型

```protobuf
message SearchRequest{
	repeated Result result = 1;
}

message Result{
	string url = 1;
	string title = 2;
	repeated string snippet = 3;
}
```

如果没有定义在同一个.proto文件里，则可以导入

```protobuf
import "myproject/anotherproto.proto";
```

### 嵌套消息类型

```protobuf
message SearchResponse{
	message Result{
		string url = 1;
		string title = 2;
		repeated string snippets = 3;
	}
	repeated Result results = 1;
}
```

在另一个message里使用Result可以

```protobuf
message OtherMessage{
	SearchResponse.Result results = 1;
}
```

### 更新消息类型

- 不要更改现有的字段的编号，不适用的使用reserve保留字段。
- 添加了新的字段也可以使用新生成的代码来解析使用“旧”消息格式通过代码序列话的任何消息。同样，旧的代码也可以解析新代码创建的消息，旧的在解析时只会忽略新字段。

### Any消息类型

```protobuf
import "google/protobuf/any.proto

message Errorstatus{
	string message = 1;
	repeat google.protobuf.Any details = 2;
}
```

### Oneof消息类型

类似于case，当消息包含多个字段但是最多设置一个字段

```protobuf
message SampleMessage{
	oneof test_oneof{
		string name = 1;
		SubMessage sub_message = 9;
	}
}
```

### Maps

protocol buffers定义了一种关联映射

```protobuf
map<key_type, value_type> map_field = N;
```

key_type可以是integral和string类型，enum不能作为key，vaue_type可以是任何除了map的类型。

```protobuf
map<string, Project> projects = 3;
```

- map不能设置repeated字段
- 映射顺序是未定义的
- 从.proto文件生成的代码，映射是按照key值排列
- 不允许有重复的key值
- 当value值为空时，序列化后的值依据编译的语言而定

#### 向后兼容

```protobuf
message MapFieldEntry {
  key_type key = 1;
  value_type value = 2;
}

repeated MapFieldEntry map_field = N;
```

### package

加上包声明，可以在不同的包定义相同的消息类型

```protobuf
package foo.bar;
message Open{
	string name = 1;
}
//就可以使用完整名称引用
message Foo{
	foo.bar.Open open = 1; 
}
```

- In **C++** the generated classes are wrapped inside a C++ namespace. For example, `Open` would be in the namespace `foo::bar`.
- In **Java**, the package is used as the Java package, unless you explicitly provide an `option java_package` in your `.proto` file.
- In **Python**, the package directive is ignored, since Python modules are organized according to their location in the file system.
- In **Go**, the package is used as the Go package name, unless you explicitly provide an `option go_package` in your `.proto` file.
- In **Ruby**, the generated classes are wrapped inside nested Ruby namespaces, converted to the required Ruby capitalization style (first letter capitalized; if the first character is not a letter, `PB_` is prepended). For example, `Open` would be in the namespace `Foo::Bar`.
- In **C#** the package is used as the namespace after converting to PascalCase, unless you explicitly provide an `option csharp_namespace` in your `.proto` file. For example, `Open` would be in the namespace `Foo.Bar`.

### Defining Services

如果想在RPC里调用消息类型，需要定义服务接口，编译器编译的时候就会产生service interface 和stubs

```protobuf
service ServiceRPC{
	rpc Search(SearchRequest) returns (SearchResponse);
}
```

gRPC与protocol buffers都是谷歌开发的系统，因此可以使用一个gRPC编译器直接产生一个RPC调用代码。

### JSON Mapping

Proto3支持JSON编码

# protocol buffer tutorials for python

对于serialize和retrieve数据来说

- python有内置的picking方法，但是不够方便，与其他语言进行共享也不够完善
- 自己设计一种编码方式
- 使用XML，使用方便，应用广泛，但是XML需要巨大的性能消耗

使用Protocol buffers定义数据结构，然后使用编译器自动产生类，这个类依据有效的二进制格式实现.proto文件进行编码和解析（encoding and parser）。这个类提供getters和setters，并负责将.proto文件作为一个单元进行读写的细节操作，因此我们只需要关心暴露出来的接口。

### 写.proto文件

```protobuf
syntax = "proto2";

package tutorial;

message Person {
  required string name = 1;
  required int32 id = 2;
  optional string email = 3;

  enum PhoneType {
    MOBILE = 0;
    HOME = 1;
    WORK = 2;
  }

  message PhoneNumber {
    required string number = 1;
    optional PhoneType type = 2 [default = HOME];
  }

  repeated PhoneNumber phones = 4;
}

message AddressBook {
  repeated Person people = 1;
}
```

### 编译

```
protoc -I=$SRC_DIR --python_out=$DST_DIR $SRC_DIR/addressbook.proto
```

### The Protocol Buffer API

```python
import addressbook_pb2
person = addressbook_pb2.Person()
person.id = 1234
person.name = "John Doe"
person.email = "jdoe@example.com"
phone = person.phones.add()
phone.number = "555-4321"
phone.type = addressbook_pb2.Person.HOME
```

#### 枚举

枚举的被扩展成一组常量

#### 标准消息方法

每个消息类还包含许多其他方法，可用于检查或操作整个消息，包括：

- `IsInitialized()`：检查是否已设置所有必填字段。
- `__str__()`：返回消息的可读形式，对于调试特别有用。（通常以`str(message)`或调用`print message`。）
- `CopyFrom(other_msg)`：使用给定消息的值覆盖消息。
- `Clear()`：将所有元素清除为空状态。

#### 解析和序列化

- `SerializeToString()`：序列化消息并以字符串形式返回。
- `ParseFromString(data)`：解析给定字符串中的消息

### 写一个消息

现在写一个地址进入地址薄文件。需要创建并填充protocol buffer类的实例，然后将他们写入输出流。

这个python程序是读AddressBook，加入一个Person，把AddressBook放回文件。

```python
#! /usr/bin/python

import addressbook_pb2
import sys

# This function fills in a Person message based on user input.
def PromptForAddress(person):
  person.id = int(raw_input("Enter person ID number: "))
  person.name = raw_input("Enter name: ")

  email = raw_input("Enter email address (blank for none): ")
#  if email != "":
#    person.email = email

  while True:
    number = raw_input("Enter a phone number (or leave blank to finish): ")
    if number == "":
      break

    phone_number = person.phones.add()
    phone_number.number = number

    type = raw_input("Is this a mobile, home, or work phone? ")
    if type == "mobile":
      phone_number.phonetype = addressbook_pb2.Person.PhoneType.Mobile
    elif type == "home":
      phone_number.phonetype = addressbook_pb2.Person.PhoneType.Home
    elif type == "work":
      phone_number.phonetype = addressbook_pb2.Person.PhoneType.Work
    else:
      print "Unknown phone type; leaving as default value."

# Main procedure:  Reads the entire address book from a file,
#   adds one person based on user input, then writes it back out to the same
#   file.
if len(sys.argv) != 2:
  print "Usage:", sys.argv[0], "ADDRESS_BOOK_FILE"
  sys.exit(-1)

address_book = addressbook_pb2.AddressBook()

# Read the existing address book.
try:
  f = open(sys.argv[1], "rb")
  address_book.ParseFromString(f.read())
  f.close()
except IOError:
  print sys.argv[1] + ": Could not open file.  Creating a new one."

# Add an address.
PromptForAddress(address_book.person.add())

# Write the new address book back to disk.
f = open(sys.argv[1], "wb")
f.write(address_book.SerializeToString())
f.close()
```

### 读一个消息

```python
#! /usr/bin/python

import addressbook_pb2
import sys

# Iterates though all people in the AddressBook and prints info about them.
def ListPeople(address_book):
  for person in address_book.person:
    print "Person ID:", person.id
    print "  Name:", person.name
#    if person.HasField('email'):
#      print "  E-mail address:", person.email

    for phone_number in person.phones:
      if phone_number.phonetype == addressbook_pb2.Person.PhoneType.Mobile:
        print "  Mobile phone #: ",
      elif phone_number.phonetype == addressbook_pb2.Person.PhoneType.Home:
        print "  Home phone #: ",
      elif phone_number.phonetype == addressbook_pb2.Person.PhoneType.Work:
        print "  Work phone #: ",
      print phone_number.number

# Main procedure:  Reads the entire address book from a file and prints all
#   the information inside.
if len(sys.argv) != 2:
  print "Usage:", sys.argv[0], "ADDRESS_BOOK_FILE"
  sys.exit(-1)

address_book = addressbook_pb2.AddressBook()

# Read the existing address book.
f = open(sys.argv[1], "rb")
address_book.ParseFromString(f.read())
f.close()

ListPeople(address_book)
```

# gRPC

![](https://pic.downk.cc/item/5f518fa9160a154a67600318.jpg)

gRPC指定通过参数和返回类型远程调用的方法。默认情况gRPC使用protocol buffer作为接口定义语言（IDL），描述服务接口和有效负载消息的结构。

- Unary RPCs 单个请求，单个响应

  ```protobuf
  rpc SayHello(HelloRequest) returns (HelloResponse);
  ```

- Server streaming RPCs 服务器流式RPC，客户端向服务器发送请求，并获取流以读取一系列消息。客户端从返回的流读取，直到没有消息，gRPC保证单个RPC调用中的消息顺序。

  ```protobuf
  rpc LotsOfReplies(HelloRequest) returns (stream HelloResponse);
  ```

- Client streaming RPCs 客户端流式RPC，客户端编写消息，并且以流的方式发送到服务器。客户端写完消息后，它将等待服务器读取消息并返回响应。gRPC再次保证了在单个RPC调用中的消息顺序。

  ```protobuf
  rpc LotsOfGreetings(stream HelloRequest) returns (HelloResponse);
  ```

- Bidirectional streaming RPCs 双向流式RPC，双方都使用读写流发送一系列消息。这两个流是独立运行的，因此客户端和服务器可以按照自己喜欢的顺序进行读写：例如，服务器可以在写响应之前等待接收所有客户端消息，或者可以先读取消息再写入消息，或读写的其他组合。每个流中的消息顺序都会保留。

  ```protobuf
  rpc BidiHello(stream HelloRequest) returns (stream HelloResponse);
  ```

### 调用API

从编译.proto文件开始，使用gRPC编译器插件，可以生成客户端和服务端的API代码，在客户端调用这些API，在服务端实现这些API。

- 在服务器端，服务器实现服务声明的方法，并运行gRPC服务器来处理客户端调用。gRPC服务端解码传入的protocol buffers请求，并执行对应的服务方法，对服务器的响应进行编码protocol buffers发送给客户端。
- 在客户端，客户端具有称为*stub*的本地对象（对于某些语言，首选术语是*client*），该对象实现与服务端相同的方法。然后，客户端可以只在本地调用这些方法，将调用的参数包装在适当的protocol buffers类型中。然后gRPC实现将请求发送到服务器并接受服务器返回的protocol buffers响应。

### Synchronous vs. asynchronous 

同步RPC调用是阻塞客户端直到收到服务端发送回来的响应，这种比较贴近抽象意义上的远程过程调用。但是本质上，网络是异步的，而且很多情况下需要不阻塞线程来调用RPC。

### RPC的生命周期

- Unary RPCs 单个请求，单个响应
  1. 客户端调用stub方法后，PRC会通知服务器客户端已调用的方法的metadata，方法名称和指定的存活期 。
  2. 然后，服务器可以立即直接发送自己的init metadata（必须在任何响应之前发送），或者等待客户端的请求消息。发生哪一种是依据应用程序。
  3. 如果是第二种，服务器收到客户端的请求消息后，它将创建和填充响应所需的所有工作。然后将响应（如果成功）连同状态详细信息（状态代码和可选状态消息）以及可选一起附加在metadata返回。
  4. 如果响应状态为OK，则客户端将获得响应，从而在客户端完成呼叫。

- Server streaming RPCs 服务器流式RPC

  与unary RPCs相似，不同之处在于服务端响应客户端的请求返回的是消息流，发送完所有消息后，服务器的状态信息（状态码和可选状态消息）附加在metadata返回给客户端。

- Client streaming RPCs 客户端流式RPC

  与unary RPCs不同之处客户端发送的是消息流而不是单个消息。服务器返回的消息不一定在它收到所有的客户端消息之后。

- Bidirectional streaming RPCs 双向流式RPC

  双向流式RPC中，调用是客户端使用函数方法产生的，服务端收到客户端的metadata和方法名以及截止日期，服务端返回它的metadata或者等待客户端发送消息流。

  客户端的流和服务器端的流是跟应用程序相关的，是相互独立的。因此客户端和服务器端可以按照任何顺序读取和写入消息。

  例如服务器端可以等收到所有消息后再发送消息，或者可以接受一条发送一条。

**需要注意的是，客户端和服务器端都可以独立的判定调用是否成功，有可能服务器端认为成功返回RPC调用，但是客户端却认为失败了**

### Metadata

元数据是以键值对列表的形式提供的有关特定RPC调用的信息（例如 [身份验证详细信息](https://grpc.io/docs/guides/auth)），其中键是字符串，值通常是字符串，但可以是二进制数据。

### Channels

一个channel提供一个特殊的host和port对客户端和服务端。它是客户端或者stub创建时使用的，因此客户端可以指定参数来修改channel，例如是否开启消息压缩。

一个channel有connected和idle状态。

gRPC怎么处理通道是跟使用的语言有关。

### Generate gRPC code 

```sh
 python -m grpc_tools.protoc -I../../protos --python_out=. --grpc_python_out=. ../../protos/helloworld.proto
```

.proto文件

```protobuf
service RouteGuide {
   // (Method definitions not shown)
   // Obtains the feature at a given position.
   rpc GetFeature(Point) returns (Feature) {}
   // Obtains the Features available within the given Rectangle.  Results are
// streamed rather than returned at once (e.g. in a response message with a
// repeated field), as the rectangle may cover a large area and contain a
// huge number of features.
	rpc ListFeatures(Rectangle) returns (stream Feature) {}
	// Accepts a stream of Points on a route being traversed, returning a
// RouteSummary when traversal is completed.
	rpc RecordRoute(stream Point) returns (RouteSummary) {}
	// Accepts a stream of RouteNotes sent while a route is being traversed,
// while receiving other RouteNotes (e.g. from other users).
	rpc RouteChat(stream RouteNote) returns (stream RouteNote) {}
}
	// Points are represented as latitude-longitude pairs in the E7 representation
// (degrees multiplied by 10**7 and rounded to the nearest integer).
// Latitudes should be in the range +/- 90 degrees and longitude should be in
// the range +/- 180 degrees (inclusive).
message Point {
  int32 latitude = 1;
  int32 longitude = 2;
}

```

生成的代码文件被称为 `route_guide_pb2.py`与`route_guide_pb2_grpc.py`和包含以下内容：

- route_guide.proto中定义的消息的类
- route_guide.proto中定义的服务的类
  - `RouteGuideStub`，客户端可以使用它来调用RouteGuide RPC
  - `RouteGuideServicer`，它定义了RouteGuide服务的实现接口
- route_guide.proto中定义的服务功能
  - `add_RouteGuideServicer_to_server`，这会将RouteGuideServicer添加到 `grpc.Server`

### 创建服务器

创建和运行`RouteGuide`服务器分为两个工作项：

- 使用我们执行服务的实际“工作”的功能来实现从我们的服务定义中生成的服务程序接口。
- 运行gRPC服务器以侦听来自客户端的请求并传输响应。

`route_guide_server.py`有一个`RouteGuideServicer`子类，该子类继承了生成的类`route_guide_pb2_grpc.RouteGuideServicer`：

```python
# RouteGuideServicer provides an implementation of the methods of the RouteGuide service.
class RouteGuideServicer(route_guide_pb2_grpc.RouteGuideServicer):
```

`RouteGuideServicer`实现所有`RouteGuide`服务方法。

##### 简单的RPC

```python
def GetFeature(self, request, context):
  feature = get_feature(self.db, request)
  if feature is None:
    return route_guide_pb2.Feature(name="", location=request)
  else:
    return feature
```

#### 启动服务器

```python
def serve():
  server = grpc.server(futures.ThreadPoolExecutor(max_workers=10))
  route_guide_pb2_grpc.add_RouteGuideServicer_to_server(
      RouteGuideServicer(), server)
  server.add_insecure_port('[::]:50051')
  server.start()
  server.wait_for_termination()
```

服务器`start()`方法是非阻塞的。将实例化一个新线程来处理请求。`server.start()`在此期间，线程调用通常不会有任何其他工作。在这种情况下，您可以调用 `server.wait_for_termination()`以干净地阻止调用线程，直到服务器终止。

### 创建客户端

#### 创建存根

我们实例化 由.proto生成`RouteGuideStub`的`route_guide_pb2_grpc`模块的类。

```python
channel = grpc.insecure_channel('localhost:50051')
stub = route_guide_pb2_grpc.RouteGuideStub(channel)
```

#### 致电服务方式

对于返回单个响应的RPC方法（“响应一元”方法），gRPC Python支持同步（阻塞）和异步（非阻塞）控制流语义。对于响应流式RPC方法，调用立即返回响应值的迭代器。调用该迭代器的`next()`方法块，直到从迭代器产生的响应变为可用为止。

##### 简单的RPC

对简单RPC的同步调用`GetFeature`几乎与调用本地方法一样简单。RPC调用等待服务器响应，并且将返回响应或引发异常：

```python
feature = stub.GetFeature(point)
```

异步调用与之`GetFeature`类似，但是就像在线程池中异步调用本地方法一样：

```python
feature_future = stub.GetFeature.future(point)
feature = feature_future.result()
```
# 对比thrift与AMQP
#### Thrift简介

##### 什么是thrift

简单来说,是Facebook公布的一款开源跨语言的RPC框架.

与此类似的是rabbitMQ，属于AMQP，进行RPC通信

##### 什么是RPC框架?

RPC (Remote Procedure Call Protocal)，远程过程调用协议

RPC, 远程过程调用直观说法就是A通过网络调用B的过程方法。

- 简单的说，RPC就是从一台机器（客户端）上通过参数传递的方式调用另一台机器（服务器）上的一个函数或方法（可以统称为服务）并得到返回的结果。
- RPC 会隐藏底层的通讯细节（不需要直接处理Socket通讯或Http通讯） RPC 是一个请求响应模型。
- 客户端发起请求，服务器返回响应（类似于Http的工作方式） RPC 在使用形式上像调用本地函数（或方法）一样去调用远程的函数（或方法）。

**RPC 框架的目标就是让远程服务调用更加简单、透明**，RPC 框架负责屏蔽底层的传输方式（TCP 或者 UDP）、序列化方式（XML/Json/ 二进制）和通信细节。**服务调用者可以像调用本地接口一样调用远程的服务提供者，而不需要关心底层通信细节和调用过程。**

RPC 框架的调用原理图如下所示：

![](https://pic.downk.cc/item/5fb4deffb18d62711359873d.jpg)

早期单机时代，一台电脑上运行多个进程，大家各干各的，老死不相往来。假如A进程需要一个画图的功能，B进程也需要一个画图的功能，程序员就必须为两个进程都写一个画图的功能。这不是整人么？于是就出现了IPC（Inter-process communication，单机中运行的进程之间的相互通信）。OK，现在A既然有了画图的功能，B就调用A进程上的画图功能好了，程序员终于可以偷下懒了。

到了网络时代，大家的电脑都连起来了。以前程序只能调用自己电脑上的进程，能不能调用其他机器上的进程呢？于是就程序员就把IPC扩展到网络上，这就是RPC（远程过程调用）了。现在不仅单机上的进程可以相互通信，多机器中的进程也可以相互通信了。要知道实现RPC很麻烦呀，什么多线程、什么Socket、什么I/O，都是让咱们普通程序员很头疼的事情。于是就有牛人开发出RPC框架（比如，CORBA、RMI、Web Services、RESTful Web Services等等）。OK，现在可以定义RPC框架的概念了。简单点讲，RPC框架就是可以让程序员来调用远程进程上的代码一套工具。有了RPC框架，咱程序员就轻松很多了，终于可以逃离多线程、Socket、I/O的苦海了。

##### thrift的跨语言特型

thrift通过一个中间语言IDL(接口定义语言)来定义RPC的数据类型和接口,这些内容写在以.thrift结尾的文件中,然后通过特殊的编译器来生成不同语言的代码,以满足不同需要的开发者,比如java开发者,就可以生成java代码,c++开发者可以生成c++代码,生成的代码中不但包含目标语言的接口定义,方法,数据类型,还包含有RPC协议层和传输层的实现代码.

##### thrift的协议栈结构

thrift是一种c/s的架构体系。在最上层是用户自行实现的业务逻辑代码。
 　第二层是由thrift编译器自动生成的代码，主要用于结构化数据的解析，发送和接收。TServer主要任务是高效的接受客户端请求，并将请求转发给Processor处理。Processor负责对客户端的请求做出响应，包括RPC请求转发，调用参数解析和用户逻辑调用，返回值写回等处理。
 　从TProtocol以下部分是thirft的传输协议和底层I/O通信。TProtocol是用于数据类型解析的，将结构化数据转化为字节流给TTransport进行传输。TTransport是与底层数据传输密切相关的传输层，负责以字节流方式接收和发送消息体，不关注是什么数据类型。底层IO负责实际的数据传输，包括socket、文件和压缩数据流等。

#### Thrift安装

安装环境：window 7

- 在[官网](https://link.jianshu.com?t=http://archive.apache.org/dist/thrift/0.9.3/)上下载thrift-0.9.3.exe包到一个新建文件夹(博主的文件夹名称为Thrift)中
- 然后将此文件夹放到环境变量Path中。例如博主就是将D:Thrift添加到Path中
- cmd，打开终端，输入`thrift -version`，即可看到相应的版本号，就算是成功安装啦

# AMQP(Advanced message queuing protocol)

RabbitMQ：服务端采用Erlang语言，客户端支持多种语言。进行RPC通讯。

AMQP基本概念：

![](https://pic.downk.cc/item/5f0433f714195aa5947e67cb.png)

## AMQP的消息转发

AMQP server里有两个部件：

- EXChange 接受Product发送的消息，按照一定的规则转发到响应的message Queues中
- message queues再将消息转发到相应的consumers

Product1产生的消息经过AMQP server转发到Consumer_n，起关键作用的路由标识是字符串Routing Key。

每个Product生产消息时都会带上Routing key，而Consumer会告诉AMQP server它希望接受的消息Routing Key时什么。

Message Queue匹配时有三种匹配方式

- direct Exchanges 全值匹配
- Topic Exchanges 通配符匹配，`*`代表一个字符串，`#`代表任意多个子字符串，product生产的字符串是用`.`隔开的子字符串
- Fanout Exchanges 广播匹配，没有Routing key

基于AMQP的以上三种消息转发模型，有三种通信模式：

- RPC(远程过程调用)
- 发布-订阅
- 广播

后两种基于通配符匹配和广播匹配就行了，主要讲一下RPC：

RPC是一种C/S通信模型，当Client发送request时，C是Product，S是Consumer；

当server回复response时，S是Product，C是Consumer。

因此基于AMQP的RPC实现原理如下图：

![](https://pic.downk.cc/item/5f043a0c14195aa59480f992.png)