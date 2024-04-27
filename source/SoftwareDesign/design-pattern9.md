# 【设计模式】行为变化

作者：wallace-lai <br>
发布：2024-04-02 <br>
更新：2024-04-28 <br>

在组件的构建过程中，组件行为的变化经常导致组件本身剧烈的变化。行为变化模式将组件的行为和组件本身进行解耦，从而支持组件行为的变化，实现两者之间的松耦合。

典型的行为变化模式有：

（1）Command - 命令模式

（2）Visitor - 访问器模式

## 一、命令模式
### 1.1 动机
在软件构建过程中，行为请求者和行为实现者通常呈现一种紧耦合。但在某些场合，比如需要对行为进行记录、撤销、重做、事务等处理，这种无法抵御变化的紧耦合是不合适的。

在这种情况下，如何将行为请求者与行为实现者解耦？将一组行为抽象为对象，可以实现二者之间的松耦合。

### 1.2 定义
命令模式：**将一个请求（行为）封装成一个对象，从而使你可用不同的请求对客户进行参数化；对请求排队或记录请求日志，以及支持可撤销的操作**。

```cpp
// 行为对象化为Command对象
class Command {
public:
    virtual void execute() = 0;
};

class CCommand1 : public Command {
    string args;
public:
    CCommand1(const string &args) : args(args) {}

    void execute() override
    {
        cout << "#1 process ..." << args << endl;
    }
};

class CCommand2 : public Command {
    string args;
public:
    CCommand2(const string &args) : args(args) {}

    void execute() override
    {
        cout << "#2 process ..." << args << endl;
    }
};

// 利用组合模式创建复合Command对象，支持命令组合
class MacroCommand : public Command {
    vector<Command *> cmds;
public:
    void addCmd(Command *c) { cmds.push_back(c); }
    void execute() override
    {
        for (auto &cmd : cmds) {
            cmd->execute();
        }
    }
};

int main()
{
    CCommand1 cmd1("Arg 1");
    CCommand2 cmd2("Arg 2");

    MacroCommand macro;
    macro.addCmd(&cmd1);
    macro.addCmd(&cmd2);

    macro.execute();
}
```

命令模式的结构如下图所示：

![命令模式结构](../media/images/SoftwareDesign/design-pattern14.png)

（1）Command：行为对象化基类

（2）ConcreteCommand：具体的行为对应的对象


### 1.3 总结
（1）命令模式的根本目的在于将行为请求者与行为实现者解耦，在面向对象语言中，常见的实现手段是将行为抽象为对象。

（2）实现命令模式接口的具体命令对象有时候根据需要可能会保存一些额外的状态信息。通过组合模式，可以将多个命令封装为一个复合命令。

（3）命令模式与C++中的函数对象有些类似。但两者定义行为接口的规范有所区别：命令模式以面向对象中的“接口-实现”来定义行为接口规范，更严格，但有性能损失；C++函数对象以函数签名来定义行为接口规范，更灵活，性能更高。


## 二、访问器模式
### 2.1 动机
在软件构建过程中，由于需求的改变，某些类层次结构中常常需要增加新的行为（方法），如果直接在基类中这样的更改，将会给子类带来很繁重的变更负担，甚至破坏原有的设计。

如何在不更改类层次结构的前提下，在运行时根据需要透明地为类层次结构上的各个类动态地添加新的操作，从而避免上述问题？

### 2.2 定义

### 2.3 总结


未完待续...