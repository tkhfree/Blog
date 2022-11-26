---
title: P4Runtime了解
toc: true
thumbnail: 'https://pic.downk.cc/item/5eda03efc2a9a83be5241038.jpg'
comments: true
tags:
  - P4
  - SDN
abbrlink: b757d80d
date: 2020-08-27 08:44:05
urlname:
categories:
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