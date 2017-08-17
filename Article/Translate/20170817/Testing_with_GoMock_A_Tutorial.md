#[译] 使用GoMock进行测试

《Testing with GoMock: A Tutorial》
[https://blog.codecentric.de/en/2017/08/gomock-tutorial/]

GoMock是Golang的mock框架。它作为Golang官方Repo"github.com/golang"提供的库，提供了灵活而高效的接口，可以和"testing"包很好的协作工作。

本文用到的例子都可以在

GitHub: github.com/sgreben/testing-with-gomock.

找到。

## 安装
首先需要安装gomock的包 "github.com/golang/mock/gomock"以及对应配套的生成代码的工具
mockgen "github.com/golang/mock/mockgen"。从技术上来说，也可以不适用代码生成工具，
但是这样，就需要我们自己去些mock代码，而自己写则容易引入错误，所以还是适用工具比较好。

这两个包都可以通过`go get`来获取

    go get github.com/golang/mock/gomock
    go get github.com/golang/mock/mockgen

通过执行`mockgen`命令可以鉴别是否安装成功

    $GOPATH/bin/mockgen

执行命令后会打印命令的使用说明，这样我们就准备好了，就可以开始准备测试代码了。

## 基本使用

使用Gomock分成四个基本步骤

1. 使用`mockgen`命令为要mock的接口生成mock实现代码。
2. 在测试代码中创建一个`gomock.Controller`的示例，然后将该对象传递给你的mock对象的
构造函数并得到一个mock对象。
3. 在mock测试代码中调用`EXPECT()`，并设置其期望值和返回值。
4. 在mock测试代码中调用`Finish()`断言mock的期望值。

来看一个小的demo展示上面的过程。为了让demo尽可能的简单，我们仅使用两个文件：一个
`doer/doer.go`接口文件中的`Doer`接口，然后在`user/user.go`文件有个`struct User`
使用了`Doer`接口，通过mock来进行测试。

我们要mock的接口其实只有几行代码：一个拥有一个`int`和`string`的参数的并返回一个error的`DoSometing`的方法。

doer/doer.go

    package doer

    type Doer interface {
        DoSomething(int, string) error
    }

这个就是我们要mock的接口`Doer`的定义。

user/user.go


    package user

    import "github.com/sgreben/testing-with-gomock/doer"

    type User struct {
        Doer doer.Doer
    }

    func (u *User) Use() error {
        return u.Doer.DoSomething(123, "Hello GoMock")
    }

当前工程结构为：

    '-- doer
        '-- doer.go
    '-- user
        '-- user.go

我们从创建一个`mocks`的包含mock实现的目录开始,然后使用`mockgen`在`doer`包里面运行

    mkdir -p mocks
    mockgen -destination=mocks/mock_doer.go -package=mocks github.com/sgreben/testing-with-gomock/doer Doer

这里由于GoMock不会不会自动创建目录并且当目录不存在时还会报错，所以这里需要我们手动
来创建"mocks"目录。这里"mockgen"的参数意义是：

* -destination=mocks/mock_doer.go ：将生成的mocks存入文件mocks/mock_doer.go。
* -package=mocks： 将生成的mocks包名设置为 "mocks"
* github.com/sgreben/testing-with-gomock/doer: 为这个包生成mocks
* Doer: 为这个接口生成mocks。这个参数是必须的，因为我们需要指定为哪个接口生成mocks
。如果有多个接口，可以将他们用逗号进行分割，如`Doer1,Doer2`。


