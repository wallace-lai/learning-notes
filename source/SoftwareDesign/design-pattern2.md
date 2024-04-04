# 【设计模式】组件协作类

作者：wallace-lai <br>
发布：2024-04-02 <br>
更新：2042-02-02 <br>

现代软件专业分工之后的第一个结果是**框架与应用程序的划分**，组件协作模式通过**晚期绑定**，来实现框架与应用程序之间的松耦合，是二者之间协作时常用的模式。

组件协作的典型模式：

（1）Template Method

（2）Strategy

（3）Observer/Event

## 一、模板方法

### 1.1 动机
在软件构建过程中，对于某一项任务，它常常**有稳定的整体结构，但各个子步骤却有很多改变的需求**。或者由于固有的原因（比如框架与应用之间的关系）而无法和任务的整体结构同时实现。

如何在确定稳定操作结构的前提下，来灵活应对各个子步骤的变化或者晚期实现的需求？

### 1.2 定义

**模板方法定义一个操作中的算法的骨架（稳定），而将一些步骤延迟到子类中。模板方法使得子类可以不改变（复用）一个算法的结构即可重定义（override重写）该算法的某些特定步骤**。

下面是一个采用结构化设计思想的案例：

```cpp
// 程序库开发人员
// 程序库只提供了稳定的Step1(), Step3()和Step5()方法的实现
// 程序主流程和Step2()以及Step4()方法均由应用程序实现。由于
// 程序主流程依赖了程序库的Step1(), Step3()和Step5()方法
// 所以这是个应用程序依赖程序库的早绑定方式。
class Library {
public:
	void Step1() {
		// ...
	}

	void Step3() {
		// ...
	}

	void Step5() {
		// ...
	}
};
```
```cpp
// 应用程序开发人员
class Application {
public:
	bool Step2() {
		// ...
	}

	void Step4() {
		// ...
	}
};

// 主流程
int main() {
	Library lib();
	Application app();

	lib.Step1();

	if (app.Step2()) {
		lib.Step3();
	}

	for (int i = 0; i < 4; ++i) {
		app.Step4();
	}

	lib.Step5();
	return 0;
}
```

对于上述结构化软件设计案例，可以用图来表示：

![结构化软件设计](../media/images/SoftwareDesign/design-pattern1.png)

可以看到上述结构化软件设计案例中，是应用程序依赖了库函数，这是早绑定的。

应用模板方法，可以将上面的例子设计成以下的形式：

```cpp
// 程序库开发人员
// 程序主流程Run和Step1(), Step2(), Step3()方法是稳定的
// 稳定的东西由程序库来实现，需要容纳变化的Step2和Step4()方
// 法采用虚函数的方式应用程序人员提供实现。这样就实现了晚绑定
class Library {
public:
	// 流程也是稳定的，所以应该由程序库提供
	void Run() {
		Step1();

		// 支持变化，虚函数的多态调用
		if (Step2()) {
			Step3();
		}

		// 支持变化，虚函数的多态调用
		for (int i = 0; i < 4; ++i) {
			Step4();
		}

		Step5();
	}

	virtual ~Library() {}

protected:
	void Step1() {	// 稳定
		// ...
	}

	void Step3() {	// 稳定
		// ...
	}

	void Step5() {	// 稳定
		// ...
	}

	virtual bool Step2() = 0;	// 变化
	virtual void Step4() = 0;	// 变化
};
```

```cpp
// 应用程序开发人员需要重写父类的虚函数
class Application : public Library {
protected:
	virtual bool Step2() {
		// ...子类重写实现
	}

	virtual void Step4() {
		// ...子类重写实现
	}
};

// 使用程序库
int main() {
	Library *pLib = new Application();
	pLib->Run();

	delete pLib;
}
```

### 1.3 注意

（1）模板方法的结构如下所示，其中红色的部分是稳定的，蓝色是变化的部分；

![Template Method](../media/images/SoftwareDesign/design-pattern2.png)

（2）模板方法模式成立的前提是算法骨架（Run方法）是稳定的。如果算法骨架都不是稳定的，那么它就不适合用模板方法方法；

（3）如果所有步骤都是稳定不变的（极端情况下），那么就没有使用设计模式的必要了。设计模式的意义在于在变化（部分）和稳定（部分）之间寻找隔离点，从而管理变化；

（4）稳定和变化是相对的，没有绝对不变的东西，只有某部分代码相对其他代码更稳定一点；

### 1.4 总结
（1）模板方法模式是一种非常基础性的设计模式，在面向对象系统中有着大量的应用。它用最简洁的机制（**虚函数的多态性**）为很多应用程序框架提供了灵活的扩展点，是代码复用方面的基本实现结构。

（2）除了可以灵活地应对子步骤的变化外，“不要调用我，让我调用你”的反向控制结构是模板方法的典型应用。“不要调用我，让我调用你”，即你应用程序不要调用我程序库，让我来调用你。

（3）在具体实现方面，被模板方法调用的虚方法可以具有实现，也可以没有任何实现（抽象方法、纯虚方法），但一般推荐将它们设计为protected方法，不供外界调用。

## 二、策略模式

### 2.1 动机
在软件构建过程中，某些对象使用的算法可能多种多样，经常改变，如果将这些算法都编码到对象中，将会使对象变得异常复杂。而且有时候支持不使用的算法也是一个性能负担。

如何在运行时根据需要透明地更改对象的算法？将算法与对象本身解耦，从而避免上述问题？

### 2.2 定义
**定义一系列算法，把它们一个个封装起来，并且使它们可互相替换（变化）。该模式使得算法可独立于使用它的客户程序（稳定）而变化（扩展、子类化）**。

如下所示，有同样功能的两段代码。

不使用策略模式：

```cpp
enum TaxBase {
	CN_TAX,
	US_TAX,
	DE_TAX,
	FR_TAX	// 更改
};

class SalesOrder {
	TaxBase tax;

public:
	double CalculateTax() {
		// ...
		if (tax == CN_TAX) {
			// ...
		}
		else if (tax == US_TAX) {
			// ...
		}
		else if (tax == DE_TAX) {
			// ...
		}
		else if (tax == FR_TAX) {	// 更改
			// ...
		}
	}
};
```

不使用策略模式存在的问题：

（1）假如要新增`FR_TAX`类型的税种计算，那么就会产生更改。这违背了开闭原则

使用策略模式：

```cpp
class TaxStrategy {
public:
	virtual double Calculate(const Context& context) = 0;
	virtual ~TaxStrategy() {}
};

class CNTax : public TaxStrategy {
public:
	virtual double Calculate(const Context& context) {
		// ...
	}
};

class USTax : public TaxStrategy {
public:
	virtual double Calculate(const Context& context) {
		// ...
	}
};

class DETax : public TaxStrategy {
public:
	virtual double Calculate(const Context& context) {
		// ...
	}
};

// 添加而非更改
class FRTax : public TaxStrategy {
public:
	virtual double Calculate(const Context& context) {
		// ...
	}
};

class SalesOrder {
private:
	TaxStrategy* strategy;

public:
	SalesOrder(StrategyFactory* strategyFactory) {
		this->strategy = strategyFactory->NewStrategy();
	}

	~SalesOrder() {
		delete this->strategy;
	}

	public double CalculateTax() {
		// ...
		Context context();
		// 多态调用
		double val = strategy->Calculate(context);
		// ...
	}
};
```

使用策略模式的好处：

（1）假如要新增`FR_TAX`类型的税种计算，那么只需要进行扩展而无需进行更改；

（2）上述代码的可复用性得到了提升；

### 2.3 总结

（1）策略模式的结构图如下，其中红色是稳定部分，蓝色为变化部分；

![Strategy](../media/images/SoftwareDesign/design-pattern3.png)

（2）Strategy及其子类为组件提供了一系列可重用的算法，从而可以使得类型在**运行时**方便地根据需要在各个算法之间进行切换；

（3）Strategy模式提供了用条件判断语句以外的另一种选择，消除条件判断语句，就是在解耦合。含有许多条件判断语句的代码通常需要Strategy模式；

（4）如果Strategy对象没有实例变量，那么各个上下文可以共享同一个Strategy对象，从而节省对象开销；

## 三、观察者模式




未完待续...