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