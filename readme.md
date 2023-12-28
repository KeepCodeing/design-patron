# 设计模式之禅
> 设计模式之禅（第2版）笔记

## 六大原则
六大原则（SOLID原则）给出了设计类和接口时需要思考那些东西，这样做的好处有哪些之类的规范。

1. 单一职责原则（Single Responsibility Principle，SRP）：一个类应该只有一个引起变化的原因。换句话说，一个类应该只有一个责任。这样可以提高类的内聚性，使其更易于理解、维护和扩展。
2. 开放封闭原则（Open-Closed Principle，OCP）：软件实体（类、模块、函数等）应该对扩展开放，对修改关闭。这意味着通过扩展现有的实体来实现新的功能，而不是修改已有的代码。这样可以保持原有代码的稳定性和可复用性。
3. 里氏替换原则（Liskov Substitution Principle，LSP）：子类型必须能够替换其基类型。换句话说，如果一个类A是类B的子类，那么可以在任何需要类B的地方使用类A，而不会引发错误或导致不正确的行为。
4. 接口隔离原则（Interface Segregation Principle，ISP）：客户端不应该依赖于它不需要的接口。一个类不应该强迫其客户端依赖于它们不需要的方法。相反，应该将接口分解为更小、更具体的部分，以确保只有需要的方法被实现。
5. 依赖倒置原则（Dependency Inversion Principle，DIP）：高层模块不应该依赖于低层模块，它们应该依赖于抽象。抽象不应该依赖于具体实现细节，而是具体实现细节应该依赖于抽象。这样可以降低模块之间的耦合度，并提高代码的可测试性和可维护性。
6. 迪米特法则（Law of Demeter，LoD）：模块之间应该通过最少的接口进行通信。一个对象应该对其他对象有尽可能少的了解，只与其直接的朋友通信。这样可以减少对象之间的依赖关系，降低耦合度。

这其中最重要的是开闭原则，它可以说是其它原则的综合应用。

### 单一职责
单一职责中的“单一”应该更多的从业务角度考虑，其次才是从功能角度。它所强调的是

用最常见的用户管理功能举例，假设我们设计了一个用户类，它同时包括了**用户信息和修改信息**两个功能：

```cpp
class PersonManager {
	public:
		GetUserName();
		GetUserAge();
		GetUserGender();
		
		UpdateUserName();
		UpdateUserAge();
		UpdateUserGender();
		UpdateUserPassword();
		
	private:
		string m_sUsername;
		int m_iUserAge;
		bool m_bGender;
		string m_sPassowrd;
};
```

上面的类很明显没有把用户的属性和行为区分开来。如果将来某个接口想要单独调用`UpdateUserPassword`等行为方法，就必须额外引入`GetUserName`这些无关的方法和属性。

接下来将它改造成一个用户对象，一个管理对象：
```cpp
class PersonInfo {
	public:
		GetUserName();
		...
	
	private:
		string m_sUsername;
		string m_sPassword;
		...
};

class PersonManager {
	public:
		UpdateUserName(PersonInfo info);
		...
};

// 后续扩展
class TeacherInfo: PersonInfo {
	public:
		// 新增Teacher所需的字段
};

class TeacherManager: PersonManager {
	public:
		// 新增或者覆写TeacherManager所需的方法
};
```

某些场景下，也不一定需要完全遵守单一职责，避免增加代码复杂度：
```java
// 这个类打电话和挂电话可以归为协议功能，通话可以归为通信功能
// 如果要遵循单一职责，还可以将这两个互不干扰的功能抽离成接口
public interface IPhone {
	// 拨通电话
	public void dial(String phoneNumber);
	// 通话
	public void chat(Object o);
	// 通话完毕，挂电话
	public void hangup();
}

// 抽离
public interface IConnectionManager {
	...
}

public interface IDataTransfer {
	public IDataTransfer(IConnectionManager cm);
}

public interface INewPhone implements IConnectionManager, IDataTransfer {
	// ...
}
```

上面的代码完全遵守了单一职责，但也提升了代码复杂度。设计类或者接口时，根据某个功能变更影响范围是否可以接受再考虑使用单一职责设计。

除了类和接口的职责单一，方法同样需要功能单一。功能越原子化，能够复用的场景就越广泛，需要修改时改动点影响面也会越小。


### 里氏替换
里氏替换原则要求所有用到父类的地方，能够使用其子类代替，且程序运行没有错误。

注意：在类中调用其他类时务必要使用父类或接口，如果不能使用父类或接口，则说明类的设计已经违背了LSP原则。

满足里氏替换原则，需要满足下面几个条件：

#### 子类实现所有父类方法
这个原则在面向接口编程时是必须实现的。父类作为抽象类，子类实现父类所有方法是必要的。

```cpp
class Father {
	void say() = 0;
};

class Son: Father {
	// 必须实现父类抽象方法
	void say() {
		cout << "son said" << endl;
	}
};

class Son2: Father {
	void say() {
			cout << "son 2 said" << endl;
	}
};

// 调用者无需知道具体调用的是父类还是子类，或者是那个子类
void Proc(Father *method) {
	method->say();
}

// 将父类替换成子类
Father *met = new Son();

Proc(met);

Father *met2 = new Son2();

Proc(met2);

delete met;
delete met2;
```

#### 子类不能被父类替换
虽然里氏替换原则规定父类存在的地方都能使用子类替换，但反过来是不行的，也就是父类没法替换子类。

这是因为向下转型是不安全的。

#### 重载父类方法子类输入类型范围要比父类大
这里要区分覆写和重载：覆写表示方法同名，输入参数相同；重载则表示方法同名，输入参数不同。

```cpp
struct Base {
	int id;
};

struct Info: Base {
	string name;
	int age;
	bool gender;
};

class Father {
	void doSomething(Base val) {
		cout << "val = " << val << endl;
	}
	
	void saySomething(Info data) {
		// ...
	}
};

class Son: Father {
	// 覆写父类方法，子类替换父类时运行子类方法
	@override
	void doSomething(Base val) {
		cout << "son val = " << val << endl;
	}
	
	// 重载父类方法，子类替换时，由于参数范围比父类大，不会运行
	void saySomething(Base data) {
		// ...
	}
};

Base *base = new Base();
Base *info = new Info();


Father *met = new Son();

// 此时调用了父类方法
met->saySomething(static_cast<Info>(info));

// 此时调用了子类方法
met->doSomething(static_cast<Info>(info));

delete base;
delete info;
delete met;
```

#### 子类返回值类型范围要比父类小
父类的一个方法的返回值是一个类型T，子类的相同方法（重载或覆写）的返回值为S，那么里氏替换原则就要求S必须小于等于T，也就是说，要么S和T是同一个类型，要么S是T的子类。

这样就方便对父类子类返回值进行统一处理。

```cpp

// 父类方法，返回Base
Base Father::getResult();

// 子类方法，返回Info
Info Son::getResult();

Base ret = father->getResult();

Base ret2 = son->getResult();

// 统一处理
proc(ret);
proc(ret2);

```

### 依赖倒置
依赖倒置的核心依据就是抽象。当细节依赖于抽象时，就可以将细节提取出来，转而使用抽象适配不同细节。

依赖倒置可以认为是里氏替换的一种具体应用。

对比实现依赖细节和实现依赖于抽象：
```cpp
// 将不同人抽象成Person，他们都有work行为
class Person {
	public:
		void work() = 0;
};

class Man: Person {
	public:
		void work() {
			cout << "man working" << endl;
		}
};

class Woman: Person {
	public:
		void work() {
			cout << "woman working" << endl;
		}
};

// 调用者依赖具体细节，没法泛化应用场景
void ProcWork1(Man man) {
	man->work();
}

// 调用者依赖抽象，无需关心实现细节，可复用多种场景
void ProcWork2(Person person) {
	person->work();
}
```

### 接口隔离
接口隔离要求一个类不应该强迫一个它不使用的接口。换句话说，一个类应该只与它需要的接口交互，而不用关心其他不需要的接口。这一点与单一职责类似。

**遵循接口隔离：**

```cpp
#include <iostream>

// 定义接口1
class Worker {
public:
    virtual void work() = 0;
};

// 定义接口2
class Eater {
public:
    virtual void eat() = 0;
};

// 实现类，仅实现需要的接口
class SuperWorker : public Worker, public Eater {
public:
    void work() override {
        std::cout << "SuperWorker is working" << std::endl;
    }

    void eat() override {
        std::cout << "SuperWorker is eating" << std::endl;
    }
};
```

**不遵循接口隔离:**

```cpp
#include <iostream>

// 定义一个不符合接口隔离原则的接口
class WorkerAndEater {
public:
    virtual void work() = 0;
    virtual void eat() = 0;
};

// 实现类，实现了所有接口的方法，包括不需要的方法
class SuperWorkerBad : public WorkerAndEater {
public:
    void work() override {
        std::cout << "SuperWorkerBad is working" << std::endl;
    }

    void eat() override {
        std::cout << "SuperWorkerBad is eating" << std::endl;
    }

    // 不需要的方法，但仍被强迫实现
    void sleep() {
        std::cout << "SuperWorkerBad is sleeping" << std::endl;
    }
};
```