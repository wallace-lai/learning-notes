# 【设计模式】状态变化类

作者：wallace-lai <br>
发布：2024-04-02 <br>
更新：2024-04-27 <br>

在组件构建过程中，某些对象的状态经常面临变化，如何对这些变化进行有效管理？同时又维持高层模块的稳定？状态变化类模式为这一问题提供了一种解决方案。

典型的状态变化类模式：

（1）State - 状态模式

（2）Memento - 备忘录模式

## 一、状态模式
### 1.1 动机
在软件构建过程中，某些对象的状态如果发生改变，其行为也会随之而发生变化。比如文档处于只读状态，其支持的行为和读写状态支持的行为就可能完全不同。

如何在运行时根据对象的状态来透明地更改对象行为？而不会为对象操作和状态转化之间引入紧耦合？

### 1.2 定义

### 1.3 总结

## 二、备忘录模式
### 2.1 动机
在软件构建过程中，某些对象的状态在转换过程中，可能由于某种需要，要求程序能够回溯到对象之前处于某个点时的状态。如果使用一些共有接口来让其他对象得到对象的状态，变会暴露对象的细节实现。

如何实现对象状态的良好保存与恢复？但同时又不会因此而破坏对象本身的封装性。

### 2.2 定义
备忘录模式：**在不破坏封装性的前提下，捕获一个对象的内部状态，并在该对象之外保存这个状态。这样以后就可以将该对象恢复到原先保存的状态**。

下面给出一个示意性的案例。

```cpp
class Memento {
    string state;

public:
    Memento(const string &s) : state(s) {}
    string GetState() const { return state; }
    void SetState(const string &s) { state = s; }
};

class Originator {
    string state;
public:
    Originator() {}

    // 不破坏Originator的封装性
    Memento CreateMomento() {
        Memento m(state);
        return m;
    }

    void SetMomento(const Memento &m) { state = m.GetState(); }
};

int main()
{
    Originator originator;

    // 存储到备忘录中，捕获对象内部状态
    Memento mem = originator.CreateMomento();

    // 改变originator的状态
    // ...

    // 从备忘录中恢复
    originator.SetMomento(mem);
}
```

备忘录模式的结构如下图所示，其中：

![备忘录模式结构图](../media/images/SoftwareDesign/design-pattern13.png)

（1）Originator是需要被捕获内部状态的对象

- CreateMemento ：创建一个内部状态的快照抛给外部

- SetMemento ：根据快照恢复内部状态

（2）Memento ：备忘录对象

（3）Caretaker ：使用备忘录模式的对象

### 2.3 总结
（1）备忘录（Memento）存储原发器（Originator）对象的内部状态，在需要时恢复原发器的状态。

（2）备忘录模式的核心是信息隐藏，即Originator需要向外界隐藏信息，保持其封装性。但同时又需要将状态保持到外界。

（3）由于现代语言运行时（如C#、Java等）都具有相当的对象序列化支持，因此往往采用效率较高、又较容易正确实现的序列化方案来实现备忘录模式。



未完待续...