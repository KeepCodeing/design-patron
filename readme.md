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

### 迪米特法则
迪米特法则的核心观念就是类间解耦，弱耦合，只有弱耦合了以后，类的复用率才可以提高。其要求的结果就是产生了大量的中转或跳转类，导致系统的复杂性提高，同时也为维护带来了难度。

该法则要求类间暴露的方法尽量少（高内聚），类内部对输入或输出的参数依赖尽量少（低耦合）。

考虑一个聊天室功能，假设它需要经过以下步骤：
1. 指定聊天室ID
2. 用对应ID建立连接
3. 判断连接是否成功
4. 收发消息
5. 终止连接

很容易可以发现，上述功能是没必要全部暴露出去的。如果调用类需要一步步将这些步骤组装起来，那么当聊天室功能发生变化时，例如需要在1，2之间加入指定白名单的功能，就会波及所有调用类。

在设计类时，考虑如何暴露最少的`public`方法和属性，减少由于变更引起的风险扩散范围。

在实际应用中经常会出现这样一个方法：放在本类中也可以，放在其他类中也没有错，那怎么去衡量呢？你可以坚持这样一个原则：如果一个方法放在本类中，既不增加类间关系，也对本类不产生负面影响，那就放置在本类中

```cpp
class Chat {
public:
	// 暴露给外部的方法，开箱即用，调用者不需要知道细节
	bool CreateChatRoom(int id);
	
	bool SendMsg(string msg);
	
	string RecMsg();
	
	bool CloseConnection();
	
private:
	int m_iRoomId;
	
	// 内部方法，根据功能细分组合，修改时不影响调用者
	bool CreateConnection();
	bool CheckConnetionState();
	bool StopConnection();

};
```

### 开闭原则
开闭原则的定义：软件实体应该对扩展开放，对修改关闭，其含义是说一个软件实体应该通过扩展来实现变化，而不是通过修改已有的代码来实现变化。

开闭原则可以说是上面所有原则的综合应用。类的设计越满足开闭原则，在功能变更时对调用者的影响越小。

当然，也不是所有场景都需要用开闭原则来避免修改，下面分情况讨论：
1. 逻辑变化：假设要把`a+b`改成`a-b`，且调用者不受该改动影响（整体逻辑变更），那么直接修改即可；
2. 子模块变化：如果有的地方要改成`a*b`，有的地方又要改成`a+b`，那么直接修改就不满足了。这时就需要通过继承对该修改进行扩展。

```cpp
class Calculator {
public:
	int calc(int a, int b) {
		// 改为a*b后，整体逻辑变更，不需要扩展后再处理
		return a + b;
	}
};


void Proc() {
	Calculator calcor;
	
	calcor.calc(1, 2);
}


class Reader {
public:
	void ReadData() = 0;
};

// 从磁盘读取数据
class DeskReader: public Reader {
public:
	void ReadData() {
		cout << "read data from desk" << endl;
	}
};

class AdvDeskReader: public DeskReader {
public:
	void ReadData() {
		cout << "read data from desk and process it" << endl;
	}
}

void GetData(Reader *reader) {
	reader->ReadData();
}

int main() {
	Reader *dReader = new DeskReader;
	
	// 从磁盘读取数据
	GetData(dReader);

	// 假设某天需求变了，要在读取数据后，对数据进行处理
	// 直接修改DeskReader，可以完成需求，但也会影响到别
	// 处的GetData逻辑。
	// 所以通过继承覆写来对该需求实现扩展。
	
	Reader *advReader = new AdvDeskReader();
	
	// 调用者不受该需求修改影响，一样能够完成功能
	GetData(advReader);
	

	return 0;
}
```

## 设计模式

### 单例模式
单例模式是一种较为简单的设计模式，它通过限制类的实例化个数（通常为一个），为系统提供一个可供全局访问的对象。

它有以下优缺点：

**优点：**：

1. **全局唯一实例：** 单例模式确保一个类只有一个实例存在，这有助于确保全局唯一的访问点，避免了重复创建相同类型的对象。
2. **懒加载（延迟实例化）：** 单例模式可以采用懒加载的方式，在需要时才创建实例，从而提高性能，减少资源浪费。
3. **全局访问点：** 通过单例模式，可以提供一个全局的访问点，使得其他对象可以轻松地获取到单例实例，简化了对象之间的交互。
4. **避免资源竞争：** 在某些情况下，如果多个对象都需要访问某一资源，使用单例模式可以避免资源竞争问题。

**缺点：**

1. **全局状态：** 单例模式引入了全局状态，可能会导致对象之间的耦合增加，使得代码更难理解和维护。
2. **测试困难：** 单例模式的全局状态可能会增加代码的测试难度，因为单例实例在不同测试用例之间可能保持状态。
3. **违反单一职责原则：** 单例模式通常负责两个职责，即创建实例和提供全局访问点，这可能违反了单一职责原则。
4. **可能引起性能问题：** 在多线程环境下，懒加载的单例模式需要考虑线程安全性，可能需要使用额外的同步机制，这可能会引起性能问题。
5. **隐藏依赖关系：** 单例模式可能隐藏了对象之间的依赖关系，使得系统设计变得不够清晰。

综合来说，单例模式在某些情况下是有用的，但在设计时需要权衡其优缺点，并确保其使用是合理的。在某些情况下，可能有更好的替代方案，如依赖注入等。

单例模式又分为**懒汉模式**和**饿汉模式**。懒汉模式即懒加载：在获取实例时才将其实例化，可能存在多线程资源分配问题，但可以节省一些内存；饿汉模式则是程序启动后就实例化。

不是所有需要全局访问的功能都要使用单例模式实现。例如一个工具类`util`，将它内部所有方法声明为`static`同样能达成目的。

#### 饿汉模式
```cpp
class Singleton {
private:
    static Singleton* instance;
    
    // 私有构造函数，防止外部通过构造函数创建实例
    Singleton() {}
    
public:
    static Singleton* getInstance() {
        return instance;
    }
};

// 在类外初始化静态成员变量，线程安全，无法延迟加载，可能有资源浪费
Singleton* Singleton::instance = new Singleton();
```

#### 懒汉模式
```cpp
class Singleton {
private:
    static Singleton* instance;
	static std::mutex mutex;  // 用于多线程安全
    
    // 私有构造函数，防止外部通过构造函数创建实例
    Singleton() {}
    
public:
	// 延迟加载，非线程安全，需要额外加锁
    static Singleton* getInstance() {
		std::lock_guard<std::mutex> lock(mutex);  // 确保多线程安全
        if (instance == nullptr) {
            instance = new Singleton();
        }
        return instance;
    }
};

// 初始化静态成员变量为nullptr
Singleton* Singleton::instance = nullptr;
std::mutex Singleton::mutex;
```

### 工厂模式
工厂模式旨在提供一种统一的方式来创建不同类型的对象，而无需在客户端代码中直接实例化具体的对象类。它通过将对象的创建逻辑封装在一个工厂类中，客户端只需调用工厂类的方法即可获取所需的对象实例。

工厂模式主要包含以下几个角色：
1. 抽象产品（Abstract Product）：定义创建的产品的接口，是具体产品类的共同父类或接口。
2. 具体产品（Concrete Product）：实现抽象产品接口，是具体的产品类。
3. 抽象工厂（Abstract Factory）：定义创建产品的接口，包含一个或多个创建产品的抽象方法。
4. 具体工厂（Concrete Factory）：实现抽象工厂接口，负责具体产品的创建，并返回具体产品的实例。

工厂模式有很多变种，下面依次介绍。

#### 简单工厂
简单工厂（静态工厂）通过一个工厂类来封装对象的创建过程，使得客户端无需直接实例化具体的对象类，而是通过调用工厂类的方法来获取所需的对象实例。

简单工厂模式的优点包括：
1. 将对象的创建逻辑封装在工厂类中，客户端无需关心具体的对象创建过程，降低了客户端代码的复杂度。
2. 通过使用工厂类，可以实现对象的创建和使用的解耦，提高了代码的灵活性和可维护性。
3. 可以集中管理对象的创建过程，便于统一修改和扩展，符合开闭原则。

简单工厂模式的限制和缺点：
1. 添加新的产品类型需要修改工厂类的代码，违反了开闭原则。
2. 工厂类职责较重，包含了所有产品的创建逻辑，当产品类型较多时，工厂类可能变得庞大而复杂。
3. 工厂类通常使用静态方法创建对象，不易于扩展和替换，不适用于复杂的对象创建场景。

使用`C++`实现简单工厂：
```cpp
#include <iostream>

// 抽象产品
class Product {
public:
    virtual void use() = 0;
};

// 具体产品A
class ConcreteProductA : public Product {
public:
    void use() {
        std::cout << "Using Product A" << std::endl;
    }
};

// 具体产品B
class ConcreteProductB : public Product {
public:
    void use() {
        std::cout << "Using Product B" << std::endl;
    }
};

// 简单工厂
class SimpleFactory {
public:
	// 静态创建方法
    static Product* createProduct(char productType) {
        if (productType == 'A')
            return new ConcreteProductA();
        else if (productType == 'B')
            return new ConcreteProductB();
        else
            return nullptr;
    }
};

int main() {
    Product* productA = SimpleFactory::createProduct('A');
    if (productA != nullptr) {
        productA->use();
        delete productA;
    }

    Product* productB = SimpleFactory::createProduct('B');
    if (productB != nullptr) {
        productB->use();
        delete productB;
    }

    return 0;
}
```

使用泛型简化操作，但无法限制泛型范围：
```cpp
#include <iostream>

// 抽象产品
template <typename T>
class Product {
public:
    virtual void use() = 0;
};

// 具体产品A
template <typename T>
class ConcreteProductA : public Product<T> {
public:
    void use() {
        std::cout << "Using Product A" << std::endl;
    }
};

// 具体产品B
template <typename T>
class ConcreteProductB : public Product<T> {
public:
    void use() {
        std::cout << "Using Product B" << std::endl;
    }
};

// 简单工厂
template <typename T>
class SimpleFactory {
public:
    static Product<T>* createProduct() {
        return new T();
    }
};

int main() {
    Product<ConcreteProductA<int>>* productA = SimpleFactory<ConcreteProductA<int>>::createProduct();
    productA->use();
    delete productA;

    Product<ConcreteProductB<std::string>>* productB = SimpleFactory<ConcreteProductB<std::string>>::createProduct();
    productB->use();
    delete productB;

    return 0;
}
```

#### 抽象工厂
抽象工厂模式的主要思想是提供一个接口，用于创建一系列相关或依赖的产品对象。客户端通过与抽象工厂和抽象产品进行交互，而无需关心具体的产品创建过程。这样可以实现对象的创建和使用的解耦，提供了一种灵活、可扩展的方式来创建不同类型的相关对象。

抽象工厂相对简单工厂而言进一步降低了依赖的耦合性。简单工厂中有多少个产品工厂内就需要创建多少个对象，无法满足开闭原则；抽象工厂通过工厂接口将不同产品的创建划分，能够更加原子化的创建产品实例。

```cpp
#include <iostream>

// Abstract Product
class Product {
public:
    virtual void use() = 0;
};

// Concrete Product A
class ConcreteProductA : public Product {
public:
    void use() {
        std::cout << "Using Product A" << std::endl;
    }
};

// Concrete Product B
class ConcreteProductB : public Product {
public:
    void use() {
        std::cout << "Using Product B" << std::endl;
    }
};

// Abstract Factory
class AbstractFactory {
public:
    virtual Product* createProduct() = 0;
};

// Concrete Factory A
class ConcreteFactoryA : public AbstractFactory {
public:
    Product* createProduct() {
        return new ConcreteProductA();
    }
};

// Concrete Factory B
class ConcreteFactoryB : public AbstractFactory {
public:
    Product* createProduct() {
        return new ConcreteProductB();
    }
};

// Client code
int main() {
    // Create Concrete Factory A
    AbstractFactory* factoryA = new ConcreteFactoryA();
    // Use Concrete Factory A to create Product A
    Product* productA = factoryA->createProduct();
    productA->use();

    // Create Concrete Factory B
    AbstractFactory* factoryB = new ConcreteFactoryB();
    // Use Concrete Factory B to create Product B
    Product* productB = factoryB->createProduct();
    productB->use();

    // Cleanup
    delete productA;
    delete factoryA;
    delete productB;
    delete factoryB;

    return 0;
}
```

需要注意的是，抽象工厂在扩展新产品时，能较为简单的实现；但如果要扩展产品功能（对产品族而言），就会有侵入式破坏了。这是因为前者只需要新增一个抽象产品和对应的抽象工厂即可，但后者要求整个产品族都实现该功能。

```cpp
class Product {
public:
	void use() = 0;
};

class ProductA: Product {
public:
	void use() {
		cout << "use A" << endl;
	}
};

class ProductB: Product {
public:
	void use() {
		cout << "use B" << endl;
	}
};

class Factory {
public:
	Product creator() = 0;
};

class FactoryA: Factory {
public:
	Product createor() {
		cout << "create A" << endl;
	}
};

class FactoryB: Factory {
public:
	Product createor() {
		cout << "create B" << endl;
	}
};

// 假设新增一个产品C，那么只需要增加对应产品和工厂即可
// 无需改动源代码，对扩展开放
class ProductC: Product {
public:
	void use() {
		cout << "use C" << endl;
	}
};

class FactoryC: Factory {
public:
	Product creator() {
		cout << "create C" << endl;
	}

};

// 但是如果要给所有产品增加个save方法，那上面的所有产品都会受到影响
class Product {
public:
	void use() = 0;
	// 导致A、B、C都需要更改
	void save() = 0;
};
```

### 模板方法模式
模板方法模式是一种行为设计模式，它定义了一个算法的骨架，将算法中某些步骤的具体实现延迟到子类中。通过模板方法模式，可以在不改变算法结构的情况下，允许子类重新定义算法中的某些步骤，以满足特定的需求。

在模板方法模式中，有两种关键角色：
1. 抽象类：抽象类定义了一个模板方法，它作为算法的骨架，包含了多个步骤，其中有些步骤的实现可以在抽象类中定义，而有些步骤的实现则由子类提供。抽象类可以包含其他具体方法，这些方法可以由模板方法调用或被子类使用。
2. 具体类：具体类是抽象类的子类，它实现了抽象类中定义的那些需要子类提供具体实现的步骤。具体类可以根据需要重写模板方法中的某些步骤，以定制算法的行为。

模板方法模式的关键思想是将算法的不变部分封装在抽象类中，而将可变部分留给具体子类来实现。这样可以确保算法的一致性，同时也允许子类根据特定需求进行定制。

模板方法中使用一个调用方法将实现步骤组合起来，不同子类不同行为的方法则使用虚函数交由子类实现。在这个过程中，可以只将调用方法暴露给外界（符合迪米特法则），并且加上`final`防止被重写。

下面是一份不使用模板方法模式的代码，可以发现两个子类的调用逻辑是完全一样的：
```cpp
#include <iostream>
using namespace std;

class Chat {
public:
	virtual void say() = 0;
	virtual void connect() = 0;
	virtual void close() = 0;
};

class ChatA: public Chat {
public:
	void say() {
		connect();
		cout << "chat A says" << endl;
		close();
	}
	
	void connect() {
		cout << "connect to room" << endl;
	}
	
	void close() {
		cout << "close connection" << endl;		
	}
};

class ChatB: public Chat {
public:
	void say() {
		connect();
		cout << "chat B says" << endl;
		close();
	}
	
	void connect() {
		cout << "connect to room" << endl;
	}
	
	void close() {
		cout << "close connection" << endl;		
	}
	
};

int main() {
	Chat *chatA = new ChatA;
	Chat *chatB = new ChatB;
	
	chatA->say();
	chatB->say();
	
	delete chatA;
	delete chatB;
	
	return 0;
}
```

接下来是使用模板方法的简化：
```cpp
#include <iostream>

// 抽象类
class Cook {
public:
    // 模板方法，定义炒菜的算法骨架
	// 加上final防止被子类重写
    void cookDish() final {
        prepareIngredients();
        heatOil();
        cookFood();
        addSeasoning();
        serveDish();
    }

// 只暴露调用方法
private:
    // 具体方法，准备食材
    void prepareIngredients() {
        std::cout << "准备食材" << std::endl;
    }

    // 具体方法，加热油
    void heatOil() {
        std::cout << "加热油" << std::endl;
    }

    // 抽象方法，烹饪食物
    virtual void cookFood() = 0;

    // 抽象方法，添加调料
    virtual void addSeasoning() = 0;

    // 具体方法，上菜
    void serveDish() {
        std::cout << "上菜" << std::endl;
    }
};

// 具体类，炒青椒
class StirFryGreenPepper : public Cook {
private:
    void cookFood() final {
        std::cout << "炒青椒" << std::endl;
    }

    void addSeasoning() final {
        std::cout << "加盐和鸡精" << std::endl;
    }
};

// 具体类，炒土豆丝
class StirFryShreddedPotatoes : public Cook {
private:
    void cookFood() final {
        std::cout << "炒土豆丝" << std::endl;
    }

    void addSeasoning() final {
        std::cout << "加盐和酱油" << std::endl;
    }
};

// 客户端代码
int main() {
    std::cout << "炒青椒：" << std::endl;
    Cook* greenPepper = new StirFryGreenPepper();
    greenPepper->cookDish();

    std::cout << "------------------" << std::endl;

    std::cout << "炒土豆丝：" << std::endl;
    Cook* shreddedPotatoes = new StirFryShreddedPotatoes();
    shreddedPotatoes->cookDish();

    delete greenPepper;
    delete shreddedPotatoes;

    return 0;
}
```

上面实现的模板方法，对重复逻辑进行了封装，但无法对被封装方法进行控制。下面扩展一个下饺子的方法，它不需要热油：
```cpp
// 具体类，下饺子
class BoilingJiaozi : public Cook {
private:
    void cookFood() final {
        std::cout << "下饺子" << std::endl;
    }

    void addSeasoning() final {
        std::cout << "加盐和鸡精" << std::endl;
    }
};
```

执行后会发现不希望执行的热油操作也被调用了。这时就需要使用钩子函数来对这种差异化的调用进行扩展了。
```cpp
// 抽象类
class Cook {
public:
    // 模板方法，定义炒菜的算法骨架
	// 加上final防止被子类重写
    void cookDish() final {
        prepareIngredients();
		// 判断是否需要热油
		if (shouldHotOil()) {
			heatOil();
		}
        cookFood();
        addSeasoning();
        serveDish();
    }
	
	virtual bool shouldHotOil() = 0;

// 只暴露调用方法
private:
   	// ...
};


class BoilingJiaozi : public Cook {
private:
    void cookFood() final {
        std::cout << "下饺子" << std::endl;
    }

    void addSeasoning() final {
        std::cout << "加盐和鸡精" << std::endl;
    }
	
	// 不需要热油
	bool shouldHotOil() {
		return false;
	}
};
```

### 建造者模式
建造者模式是一种创建型设计模式，它可以将复杂对象的构建过程和表示分离，使得同样的构建过程可以创建不同的表示。

建造者模式的核心思想是将一个复杂对象的构建过程分解为多个简单的步骤，并抽象出一个Builder接口来定义构建过程中的各个步骤。然后，具体的构建过程由不同的具体建造者类来实现，每个具体建造者类都实现了Builder接口，负责构建特定的部分。最后，一个指挥者（Director）类使用具体的建造者对象来构建最终的复杂对象。

建造者模式的主要参与者包括：
1. Product（产品）：要构建的复杂对象。它通常由多个部分组成，这些部分可以是相互关联的对象。
2. Builder（抽象建造者）：定义了构建复杂对象的接口。它包含多个抽象方法，用于构建不同部分的对象。
3. ConcreteBuilder（具体建造者）：实现了Builder接口，负责具体的构建过程。它实现了各个部分的构建方法，并负责追踪和获取最终构建的对象。
4. Director（指挥者）：负责使用具体的建造者来构建复杂对象。它定义了构建的顺序和流程，通过调用建造者的方法来构建最终的对象。

使用建造者模式可以将对象的构建过程灵活地组合和配置，使得同样的构建过程可以创建出不同的对象表示。同时，它也隐藏了复杂对象的构建细节，使得客户端代码与具体的构建过程解耦，提高了代码的可维护性和扩展性。

建造模式某种程度上是工厂模式和模板方法模式的结合，主要体现在两个方面：
1. 构建实例，与工厂的区别在于实例是在产品Builder里构建的；
2. 组合方法，将Builder实例的功能在指示类中组合起来，变成模板方法供外界调用。

```cpp
#include <iostream>
using namespace std;

// 产品接口
class Chat {
public:
	virtual bool Connect(int id) = 0;
	virtual bool Send(string str) = 0;
	virtual string Recv() = 0;
	virtual bool Close() = 0;
	
	virtual bool CheckConnectionState() {
		
		return true;
	}
};

// 实现类
class ChatA: public Chat {
public:
	bool Connect(int id) override {
		cout << "A connect to " << id << endl;
		return true;
	}
	
	bool Send(string str) override {
		cout << "A send message: " << str << endl;
		return true;
	}
	
	string Recv() override {
		cout << "A recved message" << endl;
		return "A";
	}
	
	bool Close() override {
		cout << "A close connection" << endl;
		return true;
	}
};

// 实现类
class ChatB: public Chat {
public:
	bool Connect(int id) override {
		cout << "B connect to " << id << endl;
		return true;
	}
	
	bool Send(string str) override {
		cout << "B send message: " << str << endl;
		return true;
	}
	
	string Recv() override {
		cout << "B recved message" << endl;
		return "B";
	}
	
	bool Close() override {
		cout << "B close connection" << endl;
		return true;
	}
};

// 构建器接口
class Builder {
public:
	virtual void BuildConnection(int id) = 0;
	virtual void BuildSend() = 0;
	virtual void BuildRecv() = 0;
	virtual void BuildClose() = 0;
	
	virtual Chat *getInstance() = 0;
};

// ChatA构建器实现
class BuildChatA: public Builder {
public:
	void BuildConnection(int id) {
		chatA->Connect(id);
	}
	
	// 对单个功能进行包装扩展
	void BuildSend() {
		if (chatA->CheckConnectionState()) {
			chatA->Send("hello world");
		}
	}
	
	void BuildRecv() {
		if (chatA->CheckConnectionState()) {
			chatA->Recv();
		}
	}
	
	void BuildClose() {
		chatA->Close();
	}
	
	// 获取产品实例不一定要暴露给外界
	Chat *getInstance() {
		return chatA;		
	}

private:
	// 实例化ChatA，对它的功能进行组合
	Chat *chatA = new ChatA();
};

// 指示类，用于将Builder功能组合
// 可以根据产品不同再细化，这里就用一个
class Director {
public:
	Director(Builder *bder): builder(bder) {
		
	}
	
	Builder *getBuilder() {
		return builder;
	}
	
	// 组成一个模板方法
	void CreateChat(int id) {
		builder->BuildConnection(id);
		builder->BuildSend();
		builder->BuildRecv();
		builder->BuildClose();
	}
	
private:
	Builder *builder = nullptr;
};

int main() {
	Builder *chatABuilder = new BuildChatA();
	Director *director = new Director(chatABuilder);
	
	director->CreateChat(10);
	
	// 创建ChatB同理
	// ...
	
	return 0;
}
```

### 代理模式
代理模式是一种结构型设计模式，它提供了一个代理对象来控制对另一个对象的访问。代理对象充当了客户端和目标对象之间的中介，通过代理对象可以间接访问目标对象，从而在访问过程中增加了额外的控制和功能。

代理模式的主要目的是为其他对象提供一种代理，以控制对这个对象的访问。代理对象持有对目标对象的引用，并在客户端请求时，使用目标对象的接口来完成实际的操作。代理对象可以在调用目标对象之前或之后执行一些附加的操作，例如权限验证、缓存、延迟加载等。

代理模式的参与者包括：
1. Subject（主题）：定义了目标对象和代理对象共同实现的接口，客户端通过该接口与目标对象进行交互。
2. RealSubject（真实主题）：实际的目标对象，它是代理对象所代表的对象，提供了真正的业务逻辑。
3. Proxy（代理）：代理对象，它持有对真实主题的引用，并实现了与主题相同的接口。在客户端请求时，代理对象可以在调用真实主题之前或之后执行额外的操作。

代理模式和建造者模式以及模板方法模式有些类似，它同样是对类方法进行组合，然后提供更强的功能。区别在于代理模式可以更加原子化的对单个方法进行增强，不一定要将原有方法组合在一起。

通过代理模式，能在避免对源对象进行修改的前提下，使用代理对象增加额外功能：
```cpp
#include <iostream>
#include <string>

// 主题接口
class Subject {
public:
    virtual void request() = 0;
};

// 真实主题
class RealSubject : public Subject {
public:
    void request() override {
        std::cout << "RealSubject handles the request." << std::endl;
    }
};

// 代理
class Proxy : public Subject {
public:
    Proxy(Subject* realSubject) : realSubject_(realSubject) {}

    void request() override {
        // 在调用真实主题之前可以执行额外的操作
        std::cout << "Proxy handles the request." << std::endl;

        // 调用真实主题
        realSubject_->request();

        // 在调用真实主题之后可以执行额外的操作
        std::cout << "Proxy finishes handling the request." << std::endl;
    }

private:
    Subject* realSubject_;
};

// 客户端代码
int main() {
    // 创建真实主题对象
    RealSubject* realSubject = new RealSubject();

    // 创建代理对象，并将真实主题对象传递给代理对象
    Proxy* proxy = new Proxy(realSubject);

    // 通过代理对象调用请求
    proxy->request();

    delete proxy;
    delete realSubject;

    return 0;
}
```

#### 透明代理
透明代理隐藏了客户端和真实主题之间的交互细节，客户端无需知道代理的存在。

在透明代理的示例中，客户端与真实主题交互时，实际上是通过透明代理来进行的。客户端无需知道代理的存在，直接调用透明代理的request()方法。透明代理在方法的调用前后可以执行额外的操作，同时调用真实主题的request()方法。

```cpp
#include <iostream>

// 主题接口
class Subject {
public:
    virtual void request() = 0;
};

// 真实主题
class RealSubject : public Subject {
public:
    void request() override {
        std::cout << "RealSubject handles the request." << std::endl;
    }
};

// 透明代理
class TransparentProxy : public Subject {
public:
    TransparentProxy() : realSubject_(new RealSubject()) {}

    void request() override {
        // 在调用真实主题之前可以执行额外的操作
        std::cout << "TransparentProxy handles the request." << std::endl;

        // 调用真实主题
        realSubject_->request();

        // 在调用真实主题之后可以执行额外的操作
        std::cout << "TransparentProxy finishes handling the request." << std::endl;
    }

private:
    RealSubject* realSubject_;
};

// 客户端代码
int main() {
    // 创建透明代理对象
    TransparentProxy proxy;

    // 通过透明代理对象调用请求
    proxy.request();

    return 0;
}
```

使用透明代理，对类方法进行增强：
```cpp
#include <iostream>
using namespace std;

// 响应式（C++中没有什么应用场景）：
// 1. 使用透明代理
// 2. 重载=之类的操作符
// 3. 拦截到操作后调用真实对象方法
// 4. 调用其它影响函数方法

// 普通代理：
// 在不修改原有方法的情况下，对原有方法进行扩展（AOP）或者延迟加载（效果类似闭包）

class Chat {
public:
	virtual void Connect() = 0;
	virtual void Disconnect() = 0;
	virtual void Send() = 0;
};

// 网络聊天实现类，提供了最基本的方法
class NetChat: public Chat {
public:
	void Connect() {
		cout << "Connect" << endl;
	}
	
	void Disconnect() {
		cout << "Disconnect" << endl;
		
	}
	
	void Send() {
		cout << "Send msg to client" << endl;
	}
};

class Proxy {
public:
	virtual void SendMsg() = 0;
	
};

// 透明代理，对发送消息进行增强
class ChatProxy: public Proxy { 
public:
	ChatProxy(): chat(new NetChat()) {
		
	}
	
	~ChatProxy() {
		delete chat;
	}
	
	// 假设每次发送消息都需要发送前连接，发送后断开
	// 这里同时实现了功能组合和增强
	void SendMsg() {
		chat->Connect();
		chat->Send();
		chat->Disconnect();
	}
	
private:
	Chat *chat;
};

int main() {
	Proxy *chatProxy = new ChatProxy();
	
	chatProxy->SendMsg();
	
	delete chatProxy;
	return 0;
}
```
上面的例子`ChatProxy`没有继承`Chat`，而是用`SendMsg`直接进行了组合。考虑到功能比较简单这样做是没问题的，但明显丢失了代理源对象的特征（也就是丢失了`subject`）。

#### 强制代理
强制代理要求调用者只能通过代理对象访问源对象。

在强制代理的示例中，客户端需要先进行密码认证，只有在认证成功后才能访问真实主题。强制代理的authenticate()方法用于进行密码校验，如果密码匹配成功，则创建真实主题对象；否则，拒绝访问。客户端通过强制代理的request()方法访问真实主题，只有在认证成功并且真实主题存在时，才会调用真实主题的方法。

```cpp
#include <iostream>
#include <string>

// 主题接口
class Subject {
public:
    virtual void request() = 0;
};

// 真实主题
class RealSubject : public Subject {
public:
    void request() override {
        std::cout << "RealSubject handles the request." << std::endl;
    }
};

// 强制代理
class ProtectionProxy : public Subject {
public:
    ProtectionProxy(const std::string& password) : password_(password), realSubject_(nullptr) {}

    void authenticate(const std::string& password) {
        if (password == password_) {
            realSubject_ = new RealSubject();
        } else {
            std::cout << "Authentication failed." << std::endl;
        }
    }

    void request() override {
        if (realSubject_) {
            realSubject_->request();
        } else {
            std::cout << "Access denied. Please authenticate first." << std::endl;
        }
    }

private:
    std::string password_;
    RealSubject* realSubject_;
};

// 客户端代码
int main() {
    // 创建强制代理对象，并设置密码
    ProtectionProxy proxy("password123");

    // 认证密码
    proxy.authenticate("password123");

    // 通过强制代理对象调用请求
    proxy.request();

    return 0;
}
```

#### 动态代理
动态代理是一种在运行时动态生成代理对象的技术，它允许在不修改源代码的情况下创建代理对象。与静态代理相比，动态代理的主要特点是代理类是在程序运行期间动态生成的，而不是在编译时手动编写的。

它可以避免手动编写大量代理类，其作用类似给某个类的所有方法都加上了代理方法。

下面给出一个`Java`的实现例子，`C++`中实现运行时动态生成代理类的功能需要借助库实现。

```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

// 主题接口
interface Subject {
    void request();
}

// 真实主题
class RealSubject implements Subject {
    public void request() {
        System.out.println("RealSubject handles the request.");
    }
}

// 调用处理器
class ProxyHandler implements InvocationHandler {
    private Subject realSubject;

    public ProxyHandler(Subject realSubject) {
        this.realSubject = realSubject;
    }

    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        // 在调用真实主题之前可以执行额外的操作
        System.out.println("Proxy handles the request.");

        // 调用真实主题的方法
        Object result = method.invoke(realSubject, args);

        // 在调用真实主题之后可以执行额外的操作
        System.out.println("Proxy finishes handling the request.");

        return result;
    }
}

// 客户端代码
public class Main {
    public static void main(String[] args) {
        // 创建真实主题对象
        Subject realSubject = new RealSubject();

        // 创建调用处理器对象
        ProxyHandler handler = new ProxyHandler(realSubject);

        // 创建动态代理对象
        Subject proxy = (Subject) Proxy.newProxyInstance(
                realSubject.getClass().getClassLoader(),
                realSubject.getClass().getInterfaces(),
                handler
        );

        // 通过动态代理对象调用请求
        proxy.request();
    }
}
```

可以看到上面`RealSubject`并没有对应的代理类，而是用`InvocationHandler`接口对每个方法进行了拦截，从而实现减少重复代码和动态生成的目的。

### 原型模式
原型模式是一种创建型设计模式，其目的是通过复制现有对象来创建新对象，而不是通过使用构造函数创建新对象。原型模式允许我们创建具有相同属性的对象，同时又能够避免构造函数的复杂性。

实现原型模式的时候需要考虑引用对象深拷贝的问题。

```cpp
#include <iostream>

// 原型类
class Prototype {
public:
    int data;

    // 默认构造函数
    Prototype() : data(0) {}

    // 拷贝构造函数
    Prototype(const Prototype& other) : data(other.data) {}

    // 克隆方法
    virtual Prototype* clone() {
        return new Prototype(*this);
    }
};

// 客户端代码
int main() {
    // 创建原型对象
    Prototype prototype;

    // 修改原型对象的数据
    prototype.data = 10;

    // 克隆原型对象
    Prototype* clone = prototype.clone();

    // 修改克隆对象的数据
    clone->data = 20;

    // 打印原型对象和克隆对象的数据
    std::cout << prototype.data << std::endl;  // 输出: 10
    std::cout << clone->data << std::endl;     // 输出: 20

    // 释放克隆对象的内存
    delete clone;

    return 0;
}
```