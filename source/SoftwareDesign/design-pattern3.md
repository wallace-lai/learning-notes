# 【设计模式】单一职责类

作者：wallace-lai <br>
发布：2024-04-02 <br>
更新：2024-04-10 <br>

在软件组件的设计中，如果责任划分的不清晰，使用继承得到的结果往往是随着需求的变化，子类急剧膨胀，同时充斥着重复代码，这时的关键是划清责任。

典型的单一职责类模式有：

（1）Decorator

（2）Bridge

## 一、装饰器模式
### 1.1 动机
在某些情况下我们可能会**过度地使用继承来扩展对象的功能**，由于继承为类型引入的静态特质，使得这种扩展方式缺乏灵活性；并且随着子类增多（扩展功能的增多），各种子类的组合（扩展功能的组合）会导致更多子类的膨胀。

如何使**对象功能的扩展**能够根据需要来动态地实现？同时**避免扩展功能的增多带来的子类膨胀问题**？从而使得任何功能扩展变化所导致的影响降为最低？

### 1.2 模式定义
装饰器模式：**动态（组合）地给一个对象增加一些额外的职责。就增加功能而言，Decorator模式比生成子类（继承）更为灵活（消除重复代码并且减少子类个数）**。

假设我们需要编写一个流操作的库，代码如下所示。目前看起来，不存在任何问题。

```cpp
class Stream {
public:
    virtual char Read(int num) = 0;
    virtual void Seek(int pos) = 0;
    virtual void Write(char data) = 0;

    virtual ~Stream() {}
};

// 文件流
class FileStream : public Stream {
public:
    virtual char Read(int num) {
        // 读文件
    }

    virtual void Seek(int pos) {
        // 定位文件
    }

    virtual void Write(char data) {
        // 写文件
    }
};

// 网络流
class NetworkStream : public Stream {
public:
    virtual char Read(int num) {
        // 读取网络流
    }

    virtual void Seek(int pos) {
        // 定位网络流
    }

    virtual void Write(char data) {
        // 写网络流
    }
};

// 内存流
class MemoryStream : public Stream {
public:
    virtual char Read(int num) {
        // 读内存流
    }

    virtual void Seek(int pos) {
        // 定位内存流
    }

    virtual void Write(char data) {
        // 写内存流
    }
};
```

假如我们需要对以上各个流扩展一个加密功能，那么得继续添加代码，如下所示。

```cpp
// 加密文件流
class CryptoFileStream : public FileStream {
public:
    virtual char Read(int num) {
        // 额外的加密操作
        FileStream::Read(num);  // 读取文件流
    }

    virtual void Seek(int pos) {
        // 额外的加密操作
        FileStream::Seek(pos);  // 定位内存流
    }

    virtual void Write(char data) {
        // 额外的加密操作
        FileStream::Write(data);  // 写内存流
    }
};

// 加密网络流
class CryptoNetworkStream : public NetworkStream {
public:
    virtual char Read(int num) {
        // 额外的加密操作
        NetworkStream::Read(num);  // 读取文件流
    }

    virtual void Seek(int pos) {
        // 额外的加密操作
        NetworkStream::Seek(pos);  // 定位内存流
    }

    virtual void Write(char data) {
        // 额外的加密操作
        NetworkStream::Write(data);  // 写内存流
    }
};

// 加密内存流
class CryptoMemoryStream : public MemoryStream {
public:
    virtual char Read(int num) {
        // 额外的加密操作
        MemoryStream::Read(num);  // 读取文件流
    }

    virtual void Seek(int pos) {
        // 额外的加密操作
        MemoryStream::Seek(pos);  // 定位内存流
    }

    virtual void Write(char data) {
        // 额外的加密操作
        MemoryStream::Write(data);  // 写内存流
    }
};
```

这个时候你会发现，添加的代码几乎都是类似的，代码坏味道开始出现。假如还需要添加缓存流功能，那么还需要继续添加代码，如下所示。

```cpp
class BufferedFileStream : public FileStream {
    // ...
};

class BufferedNetworkStream : public NetworkStream {
    // ...
};

class BufferedMemoryStream : public MemoryStream {
    // ...
};
```

那假如还需要新增同时具有加密和缓存的流功能呢？还需要添加...

```cpp
class CryptoBufferedFileStream : public FileStream {
    // ...
};

class CryptoBufferedNetworkStream : public NetworkStream {
    // ...
};

class CryptoBufferedMemoryStream : public MemoryStream {
    // ...
};
```

用图表示上述这些代码中的继承情况如下所示，总共有13个类！如果操作数变多，那么子类个数将以指数的方式增长！

![继承情况](../media/images/SoftwareDesign/design-pattern5.png)



### 1.3 总结

（1）通过采用组合而非继承的手法，Decorator模式实现了在运行时动态扩展对象功能的能力，而且可以根据需要扩展多个功能。避免了使用继承带来的“灵活性差”和“多子类衍生的问题”。

（2）Decorator模式在接口上表现为is-a Component的继承关系，即Decorator类继承了Component类所具有的接口。但在实现上又表现为has-a Component的组合关系，即Decorator类又使用了另外一个Component类

（3）Decorator模式的目的并非解决“多子类衍生的多继承”问题，Decorator模式应用的要点在于解决“主体类在多个方向上的扩展功能”——是为“装饰”的含义。

未完待续...