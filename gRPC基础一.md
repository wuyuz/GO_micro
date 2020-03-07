## gRPC基础二



下面我们就gRPC的proto文件进行修改

```protobuf
syntax = "proto3";

package hello;

// 定义服务
service HelloService{
    rpc SayHello(HelloRequest) returns (HelloResponse) {
    };
}

message HelloRequest{
    // 在HelloRequest局部中使用,局部自定义
    message Time{
        int64 start = 1;
        int64 end = 2;
    }
    // 可传入列表，这里的repeated表示该类型可以重复
    repeated int64 id = 1;
    Time time = 2;
    User user = 3;
}

message HelloResponse{
    string name = 1;
    User user = 2;
}

// 自定义类型
message User{
    string name = 1;
}
```



#### 简单实例：

![1581409182577](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1581409182577.png)

服务的

netstat -a

imp文件夹下的helloServiceimpl.go

```go
package imp

import (
	"context"
	protofile_hello "grpc_example/protofile"
)

// 服务的实现
type (
	HelloService struct {
	}
)

func (h *HelloService) SayHello(ctx context.Context, request *protofile_hello.HelloRequest) (*protofile_hello.HelloResponse, error) {
	return &protofile_hello.HelloResponse{
		Name: "XiaoWang",
	}, nil
}
```

protofile文件夹的hello.proto文件用于定义proto文件，使用脚本命令：

protoc --go_out=plugins=grpc:. hello.proto; 会自动生成hello.pb.go文件，

```protobuf
syntax = "proto3";

package protofile.hello;

// 定义服务
service HelloService{
    rpc SayHello(HelloRequest) returns (HelloResponse) {
    };
}

message HelloRequest{

}

message HelloResponse{
    string name = 1;
}
```

下面书写服务helloServer.go

```go
package main

import (
	"fmt"
	"google.golang.org/grpc"
	"grpc_example/imp"
	protofile_hello "grpc_example/protofile"
	"log"
	"net"
)

// 启动服务
func main() {
	lis, err := net.Listen("tcp", fmt.Sprintf(":%d", 9000))
	if err != nil {
		log.Fatal("failed to listen: %v", err)
	}
	fmt.Println("Service to listen:9000")
	grpcServer := grpc.NewServer()
	// 注册
	protofile_hello.RegisterHelloServiceServer(grpcServer, &imp.HelloService{})
	grpcServer.Serve(lis)
}
```



#### 编写客户端

```go
package main

import (
	"context"
	"fmt"
	"google.golang.org/grpc"
	protofile_hello "grpc_example/protofile"
	"log"
)

func main() {
	conn,err := grpc.Dial("127.0.0.1:9000",grpc.WithInsecure())
	if err !=nil {
		log.Fatalf("has error %s",err)
	}
	defer conn.Close()
	helloServiceClient := protofile_hello.NewHelloServiceClient(conn)
	res,err := helloServiceClient.SayHello(context.Background(), &protofile_hello.HelloRequest{})
	if nil != err {
		log.Fatal(err)
	}
	fmt.Println(res.Name)
}
```

![1583551611355](C:\Users\Administrator\Desktop\Code_Pro\grpc\assets\1583551611355.png)

​	分析上图： 首先protofile文件下创建客户端和服务端的桥梁；.proto文件定义了服务的参数和响应的服务接口的实现，helloServer.go是服务端的服务启动，里面会掉imp文件下的函数实现接口；客户端client文件下的goClient.go文件去实现protofile文件下的hello.pb.go文件



#### gRPC复杂类型应用

下面我们将我们实现gRPC调用的全过程，以后书写微服务就按这种步骤

- 首先需要写好proto文件，建立定好客户端和服务端的桥梁，在protofile文件目录下user.proto

  ```protobuf
  syntax = "proto3";
  
  package protofiles.user;
  
  // 定义rpc服务
  service UserService {
      // 这里我们需要实现UserRequst的类型和UserRequest类型
      rpc UserList (UserRequest) returns (UserResponse) {
      };
  }
  
  // 定义各自的消息传输格式
  message UserRequest {
      string name = 1;
      string mobile =2;
  }
  
  message UserResponse {
      // 返回值可以是多个User类型
      repeated User user=1;
  }
  
  message User {
      string name = 1;
      string mobile = 2;
      int64 age = 3;
  }
  ```

  在对应的protofile文件目录下：protoc --go_out=plugins=grpc:. user.proto 生成对应的user.pb.go文件

- 然后我们需要写服务端接口实现，根据user.pd.go文件中的服务端程序接口，在imp目录下写服务端接口实现userserviceimpl.go

  ```go
  package imp
  
  import (
  	"context"
  	protofiles_user "grpc_example/protofile"
  	"strconv"
  	"strings"
  )
  
  type (
  	UserService struct {
  	}
  )
  
  // 首先去user.pd.go中找到服务端要实现的函数UserList函数
  func (u *UserService) UserList(ctx context.Context, request *protofiles_user.UserRequest) (*protofiles_user.UserResponse, error) {
  	return &protofiles_user.UserResponse{
  		User: u.GetUser(request.Name),
  	}, nil
  }
  
  func (u *UserService) GetUser(name string) []*protofiles_user.User {
  	users := make([]*protofiles_user.User, 0)
  	newusers := make([]*protofiles_user.User, 0)
  	for i := 0; i < 10; i++ {
  		n := "v_" + strconv.Itoa(i)
  		users = append(users, &protofiles_user.User{
  			Name: n,
  			Age:  int64(i),
  		})
  	}
  	if  "" != strings.TrimSpace(name) {
  		for _,u := range users {
  			if name == u.Name {
  				newusers = append(newusers,u)
  			}
  		}
  	}else{
  		newusers = users
  	}
  
  	return newusers
  }
  ```

- 接着就是写服务端启动代码，用来注册函数，调用接口

  ```go
  package main
  
  import (
  	"fmt"
  	"google.golang.org/grpc"
  	"grpc_example/imp"
  	protofile_user "grpc_example/protofile"
  	"log"
  	"net"
  )
  
  // 启动服务
  func main() {
  	lis, err := net.Listen("tcp", fmt.Sprintf(":%d", 9000))
  	if err != nil {
  		log.Fatal("failed to listen: %v", err)
  	}
  	fmt.Println("Service to listen:9000")
  	grpcServer := grpc.NewServer()
  	// 注册
  	protofile_user.RegisterUserServiceServer(grpcServer, &imp.UserService{})
  	grpcServer.Serve(lis)
  }
  ```

- 最后写客户端代码，根据user.pd.go写客户端代码

  ```go
  package main
  
  import (
  	"context"
  	"fmt"
  	"google.golang.org/grpc"
  	protofile_user "grpc_example/protofile"
  	"log"
  )
  
  func main() {
  	conn,err := grpc.Dial("127.0.0.1:9000",grpc.WithInsecure())
  	if err !=nil {
  		log.Fatalf("has error %s",err)
  	}
  	defer conn.Close()
  	userServiceClient := protofile_user.NewUserServiceClient(conn)
  	res,err := userServiceClient.UserList(context.Background(), &protofile_user.UserRequest{
  		Name:"v_2",
  	})
  	if nil != err {
  		log.Fatal(err)
  	}
  	fmt.Println(res.User)
  }
  ```



#### grpc与mysql交互

首先我们需要安装插件

```
cmd中执行：
	go get github.com/go-sql-driver/mysql
	go get -u github.com/xormplus/xorm
```

- 首先写服务端和客户端的桥梁实现user.proto

  ```protobuf
  syntax = "proto3";
  
  package protofiles.user;
  
  // 定义rpc服务
  service UserService {
      // 这里我们需要实现UserRequst的类型和UserRequest类型
      rpc UserList (UserRequest) returns (UserResponse) {
      };
  }
  
  // 定义各自的消息传输格式
  message UserRequest {
      string name = 1;
      string mobile =2;
  }
  
  message UserResponse {
      // 返回值可以是多个User类型
      repeated User user=1;
  }
  
  message User {
      string name = 1;
      string mobile = 2;
      int64 age = 3;
  }
  ```

- 写服务端代码userserver.go

  ```go
  package main
  
  import (
  	"fmt"
  	_ "github.com/go-sql-driver/mysql"
  	"github.com/xormplus/xorm"
  	"google.golang.org/grpc"
  	"grpc_example/imp"
  	protofile_user "grpc_example/protofile"
  	"log"
  	"net"
  )
  
  // 启动服务
  func main() {
  	// 创建myslq引擎
  	engine, err := xorm.NewMySQL("mysql", "root:123@/test?charset=utf8")
  
  	lis, err := net.Listen("tcp", fmt.Sprintf(":%d", 9000))
  	if err != nil {
  		log.Fatal("failed to listen: %v", err)
  	}
  	fmt.Println("Service to listen:9000")
  	// 创建服务
  	grpcServer := grpc.NewServer()
  	// 注册服务，调用服务接口实现，传入数据库连接
  	protofile_user.RegisterUserServiceServer(grpcServer, &imp.UserService{Engine:engine})
  	// 监听连接
  	grpcServer.Serve(lis)
  }
  ```

- 写服务端接口实现userserviceimp.go

  ```go
  package imp
  
  import (
  	"context"
  	"github.com/xormplus/xorm"
  	models "grpc_example/model"
  	protofiles_user "grpc_example/protofile"
  )
  
  type (
  	UserService struct {
  		// 接收服务端的传入的数据库连接
  		Engine  *xorm.Engine
  	}
  )
  
  // 首先去user.pd.go中找到服务端要实现的函数UserList函数，该接口实现了桥梁中的服务应该实现的函数UserList
  func (u *UserService) UserList(ctx context.Context, request *protofiles_user.UserRequest) (*protofiles_user.UserResponse, error) {
  	return &protofiles_user.UserResponse{
  		// 接收桥梁传入的参数
  		User: u.GetUser(request.Name),
  	}, nil
  }
  
  func (u *UserService) GetUser(name string) []*protofiles_user.User {
  	// 从数据库中取数据，从数据库连接中调用数据库模型
  	userModel := models.NewUserModel(u.Engine)
  	res,_ := userModel.FindAllUserByCondition(name)
  	userList := make([]*protofiles_user.User,0)
  
  	for _,u := range res {
  		userList = append(userList,&protofiles_user.User{
  			Name :u.Name,
  		})
  	}
  	return userList
  }
  ```

- 写ORM，用于操作数据库，更多知识：<https://www.kancloud.cn/xormplus/xorm/167101>

  ```go
  package model
  
  import (
  	"github.com/xormplus/xorm"
  	"strings"
  )
  
  type (
  	User struct {
  		Id int `json:"id"`
  		Name string `json:"name"`
  		Age int `json:"age"`
  	}
  
  	UserModel struct {
  		engine *xorm.Engine
  	}
  )
  
  // 数据库操作该表的结构体
  func NewUserModel(engine *xorm.Engine) *UserModel {
  	return &UserModel{engine: engine}
  }
  
  // 数据库的表名
  func (u *User) TableName() string {
  	return "user"
  }
  
  // 该结构体实现的方法
  func (um *UserModel) FindAllUserByCondition(name string) ([]User,error) {
  	session := um.engine.Where("1=1")
  	userList := make([]User,0)
  	if "" != strings.TrimSpace(name) {
  		session = session.Where("name = ?",name)
  	}
  	// 是先根据session的查询结构返切片
  	err := session.Find(&userList)
  	if nil != err {
  		return nil, err
  	}
  	return userList,nil
  }
  ```

- 最后写客户端，goClient.go

  ```go
  package main
  
  import (
  	"context"
  	"fmt"
  	"google.golang.org/grpc"
  	protofile_user "grpc_example/protofile"
  	"log"
  )
  
  func main() {
  	conn,err := grpc.Dial("127.0.0.1:9000",grpc.WithInsecure())
  	if err !=nil {
  		log.Fatalf("has error %s",err)
  	}
  	defer conn.Close()
  	userServiceClient := protofile_user.NewUserServiceClient(conn)
  	res,err := userServiceClient.UserList(context.Background(), &protofile_user.UserRequest{
  		Name:"jaa",
  	})
  	if nil != err {
  		log.Fatal(err)
  	}
  	fmt.Println(res.User)
  }
  
  ```

  ![1583567346673](C:\Users\Administrator\Desktop\Code_Pro\grpc\assets\1583567346673.png)



#### 如何让python写微服务服务端

- 首先我们需要安装python的模块

  ```shell
  pip3 install -i https://pypi.douban.com/simple grpcio
  pip3 install -i https://pypi.douban.com/simple protobuf
  pip3 install -i https://pypi.douban.com/simple grpcio-tools
  ```

- 学习网址：<https://studygolang.com/articles/24862?fr=sidebar>

