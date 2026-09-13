---
title: "高效C++系列-设计模式"
slug: "design-pattern"
date: 2026-08-22T12:00:00+08:00
draft: false   # true=草稿，构建默认忽略
tags: ["设计模式", "c++"]
categories: ["技术笔记"]
summary: "常用23类设计模式，抽象工厂被归纳到工厂模式，包含面试高频设计模式。"
toc: true
comments: true
description: "设计模式"
---
# 面试高频设计模式

- [单例模式](设计模式.md#单例模式)
- [工厂模式](设计模式.md#工厂模式)
- [模板方法模式](设计模式.md#模板方法模式)
- [策略模式](设计模式.md#策略模式)
- [装饰器模式](设计模式.md#装饰器模式)：拦截器
- [观察者模式](#观察者模式)
- [适配器模式](#适配器模式)
- [代理模式](#代理模式)
- [门面模式](#门面模式)：系统入口
  \## UML速览
- 单例模式

```
classDiagram
    class Singleton{
        -static Singleton* instance
        -Singleton()
        +static GetInstance() Singleton*
    }
```

- 工厂模式

```
classDiagram
    class Product{
        <<abstract>>
        +Operation()
    }
    class ConcreteProductA
    class ConcreteProductB
    class Creator{
        <<abstract>>
        +FactoryMethod() Product*
        +SomeLogic()
    }
    class ConcreteCreatorA
    class ConcreteCreatorB

    Product <|-- ConcreteProductA
    Product <|-- ConcreteProductB
    Creator <|-- ConcreteCreatorA
    Creator <|-- ConcreteCreatorB
    Creator --> Product
    ConcreteCreatorA --> ConcreteProductA
    ConcreteCreatorB --> ConcreteProductB
```

抽象工厂：

```
classDiagram
    class AbstractFactory{
        <<abstract>>
        +CreateProductA() ProductA*
        +CreateProductB() ProductB*
    }
    class ConcreteFactory1
    class ConcreteFactory2
    class ProductA{<<abstract>>}
    class ProductA1
    class ProductA2
    class ProductB{<<abstract>>}
    class ProductB1
    class ProductB2

    AbstractFactory <|-- ConcreteFactory1
    AbstractFactory <|-- ConcreteFactory2
    ProductA <|-- ProductA1
    ProductA <|-- ProductA2
    ProductB <|-- ProductB1
    ProductB <|-- ProductB2

    ConcreteFactory1 --> ProductA1
    ConcreteFactory1 --> ProductB1
    ConcreteFactory2 --> ProductA2
    ConcreteFactory2 --> ProductB2
```

- 模板方法模式

```
classDiagram
    class AbstractClass{
        <<abstract>>
        +TemplateMethod()
        #PrimitiveStep1()
        #PrimitiveStep2()
    }
    class ConcreteClassA
    class ConcreteClassB

    AbstractClass <|-- ConcreteClassA
    AbstractClass <|-- ConcreteClassB
```

- 策略模式

```
classDiagram
    class Context{
        -Strategy* strategy
        +SetStrategy(Strategy*)
        +DoBusiness()
    }
    class Strategy{
        <<abstract>>
        +Algorithm()
    }
    class ConcreteStrategyA
    class ConcreteStrategyB

    Strategy <|-- ConcreteStrategyA
    Strategy <|-- ConcreteStrategyB
    Context --> Strategy
```

- 装饰器模式

```
classDiagram
    class Component{
        <<abstract>>
        +Operation()
    }
    class ConcreteComponent
    class Decorator{
        #Component* component
        +Operation()
    }
    class ConcreteDecoratorA
    class ConcreteDecoratorB

    Component <|-- ConcreteComponent
    Component <|-- Decorator
    Decorator <|-- ConcreteDecoratorA
    Decorator <|-- ConcreteDecoratorB
    Decorator --> Component
```

- 观察者模式

```
classDiagram
    class Subject{
        -vector<Observer*> observers
        +Attach(Observer*)
        +Detach(Observer*)
        +Notify()
    }
    class Observer{
        <<abstract>>
        +Update()
    }
    class ObserverA
    class ObserverB

    Observer <|-- ObserverA
    Observer <|-- ObserverB
    Subject --> Observer
```

- 适配器模式

```
classDiagram
    class Target{
        +Request()
    }
    class Adapter{
        +Request()
    }
    class Adaptee{
        +SpecificRequest()
    }

    Target <|-- Adapter
    Adapter --> Adaptee
```

- 代理模式

```
classDiagram
    class Subject{
        <<abstract>>
        +Request()
    }
    class RealSubject
    class Proxy{
        -RealSubject* realSubject
        +Request()
    }

    Subject <|-- RealSubject
    Subject <|-- Proxy
    Proxy --> RealSubject
```

- 门面模式

```
classDiagram
    class Facade{
        +DoFacadeWork()
    }
    class SubSystemA
    class SubSystemB
    class SubSystemC

    Facade --> SubSystemA
    Facade --> SubSystemB
    Facade --> SubSystemC
```

# 6大设计原则：

## 单一职责原则：

- 概念：应当有且只有一个原因引起类、接口、方法的变更。
- 实际：类很难做到这点，==接口和方法要严格遵循单一职责==。
  ![设计模式](../design-pattern/1f0a142f0e72921a1162873e9acff0b76a495530.png)
  \## 里式替换原则：
- 概念：父类出现的地方能够无条件替换为子类，但是反过来不行。
- 实际：==如果子类不能完成地实现父类的方法建议采用依赖、组合等关系代替继承==。重载父类方法时输入参数可以放大不能缩小。重写父类方法时输出结果可以缩小不能放大。
  ![设计模式](../design-pattern/72099f322b009291feb3108ef7069560c8613d72.png)
  \## 依赖倒置原则：（面向接口编程）
- 概念：模块间的依赖通过抽象进行，==实现类依赖抽象==（抽象包括接口和抽象类）。
- 依赖注入：实现类构造函数注入依赖。抽象设置setter方法注入依赖。接口声明方法注入依赖。
- 实际：每个实现类尽量都要有接口或抽象类。变量的表面类型尽量是抽象类。尽量不从实现类派生子类。尽量不要重写基类已经实现的方法。
  ![设计模式](../design-pattern/990a11d81275b563254e185e5ae77ad4076e33c3.png)

## 接口隔离原则：

- 概念：接口功能尽量细化，同时==接口中的方法尽量少==。
- 实际：虽然接口要尽量小但是不能违反单一职责原则。接口要高内聚，也就是接口中尽量少用public方法。为不同的访问者定制服务可拆分接口。
  ![设计模式](../design-pattern/b8267d812abbf5312281fd5c8ceda844502f9cf6.png)
  \## 最少知识原则：（迪米特法则）
- 概念：一个==类应当只需要了解自己的直接朋友类==，不需要了解间接朋友类。
- 实际：直接朋友类应当高内聚，不用公开太细粒度的方法给朋友。谨慎使用serializable。如果一个方法放在本类中，既不增加类间关系，对本类也不产生负面影响，那就放置在本类中。
  ![设计模式](../design-pattern/72d963278bcc9e8a1aa5785740192073f1c4b07f.png)
  \## 开闭原则：（根本原则）
- 概念：一个==软件实体（模块、类、抽象、方法）应当对扩展开放，对修改关闭==。
- 实际：实际中有时会采用替换实现类来完成变化。最佳使用方法还是通过扩展抽象或实现类来完成变化。使用扩展完成变化时，高层次模块的修改是必然的。元数据（配置参数）控制模块行为减少重复开发。项目章程很重要。封装未来的变化-\>设计模式。
- 示例：
  ![设计模式](../design-pattern/1a42d3a639df13d88ce420329c74e412a87a9542.png)

![设计模式](../design-pattern/ca83387b82a6e4f95d152c18995de4eb05c761d4.png)
\# 构造类模式
\## 单例模式
![设计模式](../design-pattern/92b77a13c5b614366a57765cee5877be0f96b6b4.png)
\### 饿汉式
- 类建立时初始化

``` cpp
    // 饿汉式单例模式
    class SingletonEager
    {
    public:
        // 禁用拷贝、赋值
        SingletonEager(const SingletonEager &) = delete;
        SingletonEager &operator=(const SingletonEager &) = delete;

        static SingletonEager &getInstance()
        {
            return instance;
        }

    private:
        // 私有构造
        SingletonEager() = default;
        static SingletonEager instance;
    };
    SingletonEager SingletonEager::instance;
```

### 懒汉式

- 方法调用时初始化，可能存在竞争问题

``` cpp
    // 懒汉式单例模式
    class SingletonLazyUnsafe
    {
    public:
        SingletonLazyUnsafe(const SingletonLazyUnsafe &) = delete;
        SingletonLazyUnsafe &operator=(const SingletonLazyUnsafe &) = delete;

        static SingletonLazyUnsafe *getInstance()
        {
            if (instance == nullptr)
            {
                instance = new SingletonLazyUnsafe();
            }
            return instance;
        }

    private:
        SingletonLazyUnsafe() = default;
        static SingletonLazyUnsafe *instance;
    };
    SingletonLazyUnsafe *SingletonLazyUnsafe::instance = nullptr;
```

- 解决竞争问题（线程安全）

``` cpp
    // 懒汉式单例模式
    // 线程安全
    class SingletonLazyFinal
    {
    public:
        SingletonLazyFinal(const SingletonLazyFinal &) = delete;
        SingletonLazyFinal &operator=(const SingletonLazyFinal &) = delete;

        static SingletonLazyFinal &getInstance()
        {
            // 首次调用才构造，C++11 自动线程安全
            static SingletonLazyFinal inst;
            return inst;
        }

    private:
        SingletonLazyFinal() = default;
    };
```

### ==多例模式==

- 按key隔离多例

``` cpp
    // 多例模式
    // 按 Key 隔离多例
    class MultitonSafe
    {
    public:
        MultitonSafe(const MultitonSafe &) = delete;
        MultitonSafe &operator=(const MultitonSafe &) = delete;

        static MultitonSafe *getInstance(const std::string &key)
        {
            std::lock_guard<std::mutex> guard(mtx);

            auto it = pool.find(key);
            if (it != pool.end())
            {
                return it->second;
            }
            MultitonSafe *obj = new MultitonSafe(key);
            pool[key] = obj;
            return obj;
        }

        void info() const
        {
            std::cout << "Key = " << key_ << "\n";
        }

    private:
        explicit MultitonSafe(std::string k) : key_(std::move(k)) {}

        std::string key_;
        static std::mutex mtx;
        static std::map<std::string, MultitonSafe *> pool;
    };
    std::mutex MultitonSafe::mtx;
    std::map<std::string, MultitonSafe *> MultitonSafe::pool;
```

- 有限多例

``` cpp
    // 有限多例模式
    class LimitedMultiton
    {
    public:
        LimitedMultiton(const LimitedMultiton &) = delete;
        LimitedMultiton &operator=(const LimitedMultiton &) = delete;

        static LimitedMultiton *getInstance()
        {
            std::lock_guard<std::mutex> guard(mtx);
            if (instances.size() < MAX_COUNT)
            {
                auto p = new LimitedMultiton(instances.size());
                instances.push_back(p);
                return p;
            }
            // 策略1：达到上限返回nullptr
            return nullptr;
        }

        void showId()
        {
            std::cout << "Instance id:" << id_ << "\n";
        }

    private:
        explicit LimitedMultiton(int id) : id_(id) {}
        int id_;

        static constexpr size_t MAX_COUNT = 3;
        static std::mutex mtx;
        static std::vector<LimitedMultiton *> instances;
    };
    std::mutex LimitedMultiton::mtx;
    std::vector<LimitedMultiton *> LimitedMultiton::instances;
```

### 总结

- 优点：
  - 减少系统内存开支和性能开销。
  - 避免对资源的多重占用，便于资源共享。
- 缺点：
  - 扩展困难（没接口）。
  - 与单一职责原则冲突。
  - 不利于单元测试。
- 使用环境：
  - 生成唯一序列号。
  - 需要共享访问数据。
  - 创建一个实例需要消耗资源过多。
  - 需要定义大量静态常量和方法。
- 应用：
  - Spring的Bean就是单例的。
    \## 工厂模式
    ![设计模式](../design-pattern/e20c71357e9f4cde7dd046ac0ee8624a4eb747bc.png)
    \### ==简单工厂==
- 全局一个具体工厂类，一个抽象产品类，通过参数判断创建不同产品，违反**开闭原则**

``` cpp
    class Product
    {
    public:
        virtual ~Product() = default;
        virtual void show() = 0;
    };

    class ProductA : public Product
    {
    public:
        void show() override { std::cout << "Product A\n"; }
    };

    class ProductB : public Product
    {
    public:
        void show() override { std::cout << "Product B\n"; }
    };

    class SimpleFactory
    {
    public:
        static std::unique_ptr<Product> createProduct(const std::string &type)
        {
            if (type == "A")
            {
                return std::make_unique<ProductA>();
            }
            else if (type == "B")
            {
                return std::make_unique<ProductB>();
            }
            return nullptr;
        }
    };
```

### 工厂方法

- 全局一个抽象工厂类，一个抽象产品类，一个抽象工厂对应一个抽象产品

``` cpp
    // 抽象产品
    class Product
    {
    public:
        virtual ~Product() = default;
        virtual void show() = 0;
    };

    class ProductA : public Product
    {
    public:
        void show() override { std::cout << "Product A\n"; }
    };
    class ProductB : public Product
    {
    public:
        void show() override { std::cout << "Product B\n"; }
    };

    // 抽象工厂
    class Factory
    {
    public:
        virtual ~Factory() = default;
        virtual std::unique_ptr<Product> create() = 0;
    };

    class FactoryA : public Factory
    {
    public:
        std::unique_ptr<Product> create() override
        {
            return std::make_unique<ProductA>();
        }
    };

    class FactoryB : public Factory
    {
    public:
        std::unique_ptr<Product> create() override
        {
            return std::make_unique<ProductB>();
        }
    };
```

### 抽象工厂

- 全局一个抽象工厂类，多个抽象产品类，一个抽象工厂对应多个抽象产品

``` cpp
    // 产品1：按钮
    class Button
    {
    public:
        virtual ~Button() = default;
        virtual void render() = 0;
    };
    // 产品2：文本框
    class TextBox
    {
    public:
        virtual ~TextBox() = default;
        virtual void render() = 0;
    };

    // Windows族
    class WinButton : public Button
    {
    public:
        void render() override { std::cout << "Windows Button\n"; }
    };
    class WinTextBox : public TextBox
    {
    public:
        void render() override { std::cout << "Windows TextBox\n"; }
    };

    // Mac族
    class MacButton : public Button
    {
    public:
        void render() override { std::cout << "Mac Button\n"; }
    };
    class MacTextBox : public TextBox
    {
    public:
        void render() override { std::cout << "Mac TextBox\n"; }
    };

    // 抽象工厂
    class GUIFactory
    {
    public:
        virtual ~GUIFactory() = default;
        virtual std::unique_ptr<Button> createButton() = 0;
        virtual std::unique_ptr<TextBox> createTextBox() = 0;
    };

    class WinFactory : public GUIFactory
    {
    public:
        std::unique_ptr<Button> createButton() override
        {
            return std::make_unique<WinButton>();
        }
        std::unique_ptr<TextBox> createTextBox() override
        {
            return std::make_unique<WinTextBox>();
        }
    };

    class MacFactory : public GUIFactory
    {
    public:
        std::unique_ptr<Button> createButton() override
        {
            return std::make_unique<MacButton>();
        }
        std::unique_ptr<TextBox> createTextBox() override
        {
            return std::make_unique<MacTextBox>();
        }
    };
```

### 总结

- 优点：
  - 良好的封装性。
  - 优秀的扩展性。
  - 屏蔽产品类的实现。
- 缺点：
- 使用环境：
  - 在所有需要生成实例的地方都可以使用。
  - 需要灵活可扩展的框架时建议使用。
- 应用：
  - JDBC链接数据库，数据库从Mysql切换到Oracle。
  - JDBC链接数据库，MaxConnections。
    \## 建造者模式
    ![设计模式](../design-pattern/62c11af56202e6e6a1c738709d88e82143b69ea7.png)
- **分多步骤组装产品**，可以灵活调整构建步骤，得到不同表现的实例
- 一个具体建造者对应一个具体产品，抽象建造者统一构建步骤（让指挥者面向接口编程）

``` cpp
    // 【1】产品：复杂对象（电脑，包含CPU、显卡、内存）
    class Computer
    {
    public:
        void setCpu(std::string cpu) { cpu_ = std::move(cpu); }
        void setGpu(std::string gpu) { gpu_ = std::move(gpu); }
        void setRam(std::string ram) { ram_ = std::move(ram); }

        void showInfo() const
        {
            std::cout << "电脑配置:\n"
                      << "CPU: " << cpu_ << "\n"
                      << "GPU: " << gpu_ << "\n"
                      << "内存: " << ram_ << "\n\n";
        }

    private:
        std::string cpu_;
        std::string gpu_;
        std::string ram_;
    };

    // 【2】抽象建造者
    class ComputerBuilder
    {
    public:
        virtual ~ComputerBuilder() = default;
        virtual void buildCpu() = 0;
        virtual void buildGpu() = 0;
        virtual void buildRam() = 0;
        virtual std::unique_ptr<Computer> getResult() = 0;
    };

    // 【3】具体建造者：游戏电脑
    class GamingPcBuilder : public ComputerBuilder
    {
    public:
        void buildCpu() override
        {
            pc_->setCpu("Intel i9");
        }
        void buildGpu() override
        {
            pc_->setGpu("RTX 4090");
        }
        void buildRam() override
        {
            pc_->setRam("64GB DDR5");
        }

        std::unique_ptr<Computer> getResult() override
        {
            return std::move(pc_);
        }

    private:
        std::unique_ptr<Computer> pc_ = std::make_unique<Computer>();
    };

    // 【4】指挥者：固定组装顺序
    class Director
    {
    public:
        void construct(ComputerBuilder &builder)
        {
            builder.buildCpu();
            builder.buildGpu();
            builder.buildRam();
        }
    };
```

- 指挥者是可选的，直接手动调用构建步骤也是可以的

``` cpp
int main() {
    GamingPcBuilder builder;
    builder.buildCpu();
    builder.buildRam(); // 手动调换顺序，不构建显卡
    auto pc = builder.getResult();
    pc->showInfo();
    return 0;
}
```

### 总结

- 优点：
  - 封装性高。
  - 建造者独立，容易扩展。
  - 建造者独立，便于控制细节风险。
- 缺点：
- 使用场景：
  - 相同的基本方法，不同的执行顺序。
  - 不同个数的零部件组装一个对象。
  - 产品类非常复杂。
- 应用：
  - Spring里面有很多建造者模式。
    \## 原型模式
    ![设计模式](../design-pattern/f66300c9207676b7524bf5c601ca16600ea646b2.png)
- 用一个已经存在的抽象原型对象，克隆出新对象，不通过 new + 构造函数创建实例

``` cpp
    // 抽象原型
    class Prototype
    {
    public:
        virtual ~Prototype() = default;
        // 克隆接口：返回自身副本
        virtual std::unique_ptr<Prototype> clone() const = 0;
        virtual void show() const = 0;
    };

    // 具体原型
    class Monster : public Prototype
    {
    public:
        Monster(std::string name, int hp) : name_(std::move(name)), hp_(hp) {}

        // 实现克隆：深拷贝
        std::unique_ptr<Prototype> clone() const override
        {
            return std::make_unique<Monster>(*this);
        }

        void setHp(int hp) { hp_ = hp; }

        void show() const override
        {
            std::cout << "怪物[" << name_ << "] HP = " << hp_ << "\n";
        }

    private:
        std::string name_;
        int hp_;
    };
```

- 原型池（简单工厂）

``` cpp
    class PrototypeManager
    {
    public:
        void registerProto(const std::string &key, std::unique_ptr<Prototype> proto)
        {
            protos_[key] = std::move(proto);
        }

        std::unique_ptr<Prototype> getClone(const std::string &key) const
        {
            auto it = protos_.find(key);
            if (it == protos_.end())
                return nullptr;
            return it->second->clone();
        }

    private:
        std::unordered_map<std::string, std::unique_ptr<Prototype>> protos_;
    };
```

### 总结

- 优点：
  - 内存复制，性能比new更强。
- 缺点：
  - 逃避了构造函数的约束。
- 使用场景：
  - 类初始化需要太多资源。
  - 性能要求。
  - 并发编程。
- 应用：
  - 常常与工厂模式一起使用。
    \# 结构类模式（对象组合）
    \## 装饰器模式
    ![设计模式](../design-pattern/68a9056cc288b8282d5f439a431fe47eaf3067a5.png)
- **动态给对象附加额外职责**，不修改原有类代码，不用继承，通过包装（组合）方式扩展功能。

``` cpp
// 抽象组件
class Coffee {
public:
    virtual ~Coffee() = default;
    virtual double cost() = 0;
    virtual std::string desc() = 0;
};

// 具体组件：黑咖啡
class BlackCoffee : public Coffee {
public:
    double cost() override {
        return 10.0;
    }
    std::string desc() override {
        return "黑咖啡";
    }
};

// 抽象装饰器
class CoffeeDecorator : public Coffee {
protected:
    std::shared_ptr<Coffee> coffee;
public:
    explicit CoffeeDecorator(std::shared_ptr<Coffee> c) : coffee(std::move(c)) {}
};

// 具体装饰器：牛奶
class MilkDecorator : public CoffeeDecorator {
public:
    using CoffeeDecorator::CoffeeDecorator;
    double cost() override {
        return coffee->cost() + 2.5;
    }
    std::string desc() override {
        return coffee->desc() + " +牛奶";
    }
};

// 具体装饰器：糖
class SugarDecorator : public CoffeeDecorator {
public:
    using CoffeeDecorator::CoffeeDecorator;
    double cost() override {
        return coffee->cost() + 1.0;
    }
    std::string desc() override {
        return coffee->desc() + " +糖";
    }
};
```

### 总结：

- 优点：
  - 扩展性很好
  - 分离装饰类和被装饰类。
  - 是继承关系的替代方案。
  - 可以动态扩展一个实现类的功能。
- 缺点：
  - 多层的装饰很复杂，而且不易找到出错的地方。
- 使用场景：
  - 给一个类添加附加功能或扩展这个类的功能。
  - 动态给一个类增加或删除功能。
  - 为一批兄弟类进行改装或加装功能。
- 应用：
  \## 适配器模式
  ![设计模式](../design-pattern/05e9e7f3b035927bb7b138649d609579da819c1f.png)
- **接口转换**，把已有类的接口，转换成客户端期望的目标接口。让原本接口不兼容的类可以一起工作
  \##### 对象适配器
- 组合持有被适配对象

``` cpp
// 被适配者：旧圆孔充电器（已有老组件，接口不匹配）
class RoundCharger {
public:
    void roundPlug() {
        std::cout << "圆孔充电器输出电流\n";
    }
};

// 目标接口：客户端期望的TypeC接口
class TypeC {
public:
    virtual ~TypeC() = default;
    virtual void typeCPlug() = 0;
};

// 对象适配器：组合持有旧充电器，完成接口转换
class ChargerAdapter : public TypeC {
private:
    std::shared_ptr<RoundCharger> adaptee;
public:
    explicit ChargerAdapter(std::shared_ptr<RoundCharger> r) : adaptee(std::move(r)) {}
    void typeCPlug() override {
        std::cout << "适配器转换接口 → ";
        adaptee->roundPlug(); // 调用被适配者原有逻辑
    }
};
```

##### 类适配器

- 继承被适配者实现

``` cpp
// 类适配器：同时继承目标接口 + 被适配者
class ClassAdapter : public TypeC, private RoundCharger {
public:
    void typeCPlug() override {
        std::cout << "类适配器转换接口 → ";
        roundPlug();
    }
};
```

### 总结：

- 优点：
  - 能够关联两个没有任何关系的类。
  - 增加了类透明性。
  - 提高类复用度。
  - 灵活性很高。
- 缺点：
- 使用场景：
  - 系统扩展以后，需要将新建立的类接入系统的通用接口。
  - 开发阶段不咋需要，维护阶段很重要。
  - 对象适配器使用的场景较多。
- 应用：
  \## 代理模式
  ![设计模式](../design-pattern/ae7cec45ece402b4977785f0d7548470854d333d.png)
  \### 静态代理
- 编译期就知道自己代理哪个类，c++原生只支持静态代理
- 为一个接口提供代理类，由代理对象控制所有继承同一个接口具体对象的访问

``` cpp
    // 【抽象主题】统一接口
    class Subject
    {
    public:
        virtual ~Subject() = default;
        virtual void request(const std::string &data) = 0;
    };

    // 【真实主题：真正执行业务】
    class RealSubject : public Subject
    {
    public:
        void request(const std::string &data) override
        {
            std::cout << "真实对象执行任务: " << data << "\n";
        }
    };

    // 【代理】
    class Proxy : public Subject
    {
    public:
        // 构造注入真实对象
        Proxy(std::unique_ptr<Subject> real) : real_(std::move(real)) {}

        void request(const std::string &data) override
        {
            // 前置增强：日志、权限校验、限流
            std::cout << "[代理前置：权限校验]\n";

            // 转发调用真实对象
            real_->request(data);

            // 后置增强：统计耗时、日志收尾
            std::cout << "[代理后置：调用结束统计]\n";
        }

    private:
        std::unique_ptr<Subject> real_;
    };
```

- 虚拟代理（懒加载，延迟创建重量级对象，用到时才创建）

``` cpp
// 虚拟代理示例思路
void Proxy::request(){
    if (!real_) {
        real_ = std::make_unique<RealSubject>();
    }
    real_->request();
}
```

### 动态代理

- 运行时才知道自己代理哪个类，不会预先实现接口
  \### 总结
- 优点：
  - 职责清晰。
  - 高扩展性。
  - 智能化。
- 缺点：
- 使用场景：
  - 减轻被代理类的负担，让被代理类专注于业务实现。
- 应用：
  - Spring AOP。
  - AspectJ
  - \$Proxy0标志表示代理。
    \## 门面模式
    ![设计模式](../design-pattern/0802ea050c11b9ea9a45faa041e5631d7dba7f92.png)
- **为复杂的多个子系统提供一个高层统一入口**，封装子系统内部复杂调用逻辑，对外暴露简单接口。

``` cpp
// 子系统：灯光
class Light {
public:
    void dim() { std::cout << "灯光调暗\n"; }
    void bright() { std::cout << "灯光调亮\n"; }
};

// 子系统：投影仪
class Projector {
public:
    void on() { std::cout << "投影仪开机\n"; }
    void off() { std::cout << "相机关机\n"; }
};

// 子系统：音响
class Sound {
public:
    void on() { std::cout << "音响开启\n"; }
    void off() { std::cout << "音响关闭\n"; }
};

// 子系统：播放器
class Player {
public:
    void play() { std::cout << "开始播放电影\n"; }
    void stop() { std::cout << "停止播放\n"; }
};

// 门面：家庭影院，封装整套流程
class HomeTheaterFacade {
private:
    Light light;
    Projector projector;
    Sound sound;
    Player player;
public:
    void watchMovie() {
        std::cout << "=====开始观影=====\n";
        light.dim();
        projector.on();
        sound.on();
        player.play();
    }

    void endMovie() {
        std::cout << "=====结束观影=====\n";
        player.stop();
        sound.off();
        projector.off();
        light.bright();
    }
};
```

### 总结：

- 优点：
  - 子系统解耦。
  - 灵活性。
  - 安全性。
- 缺点：
  - 门面对象不符合开闭原则。
- 使用场景：
  - 为复杂的子系统提供外界接口。
  - 系统是黑箱。
  - 预防低水平人员带来的风险扩散。
- 应用：
  - 尽量使用门面模式防止低水平开发人员的代码风险。
    \## 组合模式
    ![设计模式](../design-pattern/384e241aec557b7114d1dac498a8d87310fb8bad.png)
- **将对象组织成树形结构，用来表示 "整体‑部分" 的层次关系**。客户端可以用一致的方式对待单个对象（叶子节点）和对象组合（容器节点），无需区分是叶子还是容器。

``` cpp
// 抽象构件
class FileComponent {
public:
    virtual ~FileComponent() = default;
    virtual void show(int depth = 0) = 0;
    // 容器才支持增删，叶子空实现
    virtual void add(std::shared_ptr<FileComponent>) {}
    virtual void remove(std::shared_ptr<FileComponent>) {}
};

// 叶子：文件
class FileLeaf : public FileComponent {
private:
    std::string name;
public:
    explicit FileLeaf(std::string n) : name(std::move(n)) {}
    void show(int depth) override {
        for(int i = 0; i < depth; ++i) std::cout << "  ";
        std::cout << "- 文件:" << name << "\n";
    }
};

// 组合：文件夹，可以包含子节点
class FolderComposite : public FileComponent {
private:
    std::string name;
    std::vector<std::shared_ptr<FileComponent>> children;
public:
    explicit FolderComposite(std::string n) : name(std::move(n)) {}

    void add(std::shared_ptr<FileComponent> c) override {
        children.push_back(std::move(c));
    }
    void remove(std::shared_ptr<FileComponent> c) override {
        for(auto it = children.begin(); it != children.end(); ++it) {
            if(*it == c) {
                children.erase(it);
                break;
            }
        }
    }

    void show(int depth) override {
        for(int i = 0; i < depth; ++i) std::cout << "  ";
        std::cout << "+ 文件夹:" << name << "\n";
        // 递归打印子节点
        for(auto& child : children) {
            child->show(depth + 1);
        }
    }
};
```

### 总结：

- 优点：
  - 高层模块调用简单。
  - 节点自由增加。
- 缺点：
  - 与依赖倒置原则冲突，没有面向接口编程。
- 使用场景：
  - 树形结构。
  - 整体和局部关系明显。
- 应用：
  - XML
    \## 桥接模式
    ![设计模式](../design-pattern/eaaa031fd69cdc7bc4df2ff1c5fa8d620cce52df.png)
- **分离抽象与实现，二者可以独立变化**。使用组合代替继承，避免继承带来的类爆炸。

``` cpp
// 实现接口：颜色
class ColorImpl {
public:
    virtual ~ColorImpl() = default;
    virtual void fill() = 0;
};

class RedColor : public ColorImpl {
public:
    void fill() override { std::cout << "红色"; }
};

class BlueColor : public ColorImpl {
public:
    void fill() override { std::cout << "蓝色"; }
};

// 抽象：图形
class Shape {
protected:
    std::unique_ptr<ColorImpl> color;
public:
    explicit Shape(std::unique_ptr<ColorImpl> c) : color(std::move(c)) {}
    virtual ~Shape() = default;
    virtual void draw() = 0;
};

// 扩充抽象：圆形
class Circle : public Shape {
public:
    using Shape::Shape;
    void draw() override {
        std::cout << "绘制圆形，颜色：";
        color->fill();
        std::cout << "\n";
    }
};

// 扩充抽象：矩形
class Rectangle : public Shape {
public:
    using Shape::Shape;
    void draw() override {
        std::cout << "绘制矩形，颜色：";
        color->fill();
        std::cout << "\n";
    }
};
```

### 总结：

- 优点：
  - 抽象和实现分离。
  - 扩展性。
  - 抽象层完成封装，对用户透明。
- 使用场景：
  - 不希望或不适用继承的场景。继承具有强侵入性，会强制子类实现某个方法。
  - 接口或抽象类不稳定。
  - 重用性要求高的场景。
- 应用：
  - 继承负责不变的方法，桥梁负责变化的方法。
    \## 享元模式
    ![设计模式](../design-pattern/6eac6a1ae342828c3c12d46545d2cf37a6f67315.png)
- **复用大量细粒度重复对象，减少内存占用**。把对象拆分为**内部状态（不变，可共享）**与**外部状态（变化，不可共享，外部传入）**，通过工厂缓存共享对象，避免重复创建大量相同实例

``` cpp
// 抽象享元
class Flyweight {
public:
    virtual ~Flyweight() = default;
    // 外部状态通过参数传入，不保存到对象
    virtual void render(int x, int y) = 0;
};

// 具体享元：粒子，内部状态：粒子类型key（颜色纹理）
class ParticleFlyweight : public Flyweight {
private:
    std::string particleType; // 内部状态，不变，共享
public:
    explicit ParticleFlyweight(std::string type) : particleType(std::move(type)) {}
    void render(int x, int y) override {
        std::cout << "渲染粒子[" << particleType << "] 坐标 x=" << x << " y=" << y << "\n";
    }
};

// 享元工厂，维护对象池
class FlyweightFactory {
private:
    std::unordered_map<std::string, std::unique_ptr<Flyweight>> pool;
public:
    Flyweight* getFlyweight(const std::string& key) {
        if(pool.find(key) == pool.end()) {
            std::cout << "工厂创建新享元对象：" << key << "\n";
            pool[key] = std::make_unique<ParticleFlyweight>(key);
        }else{
            std::cout << "复用缓存享元对象：" << key << "\n";
        }
        return pool[key].get();
    }
};

int main() {
    FlyweightFactory factory;

    // 大量粒子，只创建2份共享对象，外部坐标每次传入
    Flyweight* p1 = factory.getFlyweight("fire");
    p1->render(100,200);

    Flyweight* p2 = factory.getFlyweight("fire");
    p2->render(150,220);

    Flyweight* p3 = factory.getFlyweight("ice");
    p3->render(200,300);

    Flyweight* p4 = factory.getFlyweight("ice");
    p4->render(250,330);
    return 0;
}
```

### 总结：

- 优点：
  - 高效率。
  - 降低对象的内存占用。
- 缺点：
  - 系统复杂。
- 使用场景：
  - 系统中存在大量相似对象。
  - 需要缓冲池。
- 应用：
  - 对象池：Apache的commons-pool
  - String类的intern方法。
  - 对象池着重复用，享元着重共享细粒度对象。
    \# 行为类模式（对象交互）
    \## 模板方法模式
    ![设计模式](../design-pattern/bbe2034ab00c570f2e7eccd3b3c516df8e92fc77.png)
- 将抽象类的方法分成可变方法（virtual）和固定方法（非virtual），固定方法调用可变方法确定框架，固定方法禁止子类重写保证流程固定

``` cpp
    // 抽象父类：定义算法骨架
    class AbstractCook
    {
    public:
        virtual ~AbstractCook() = default;

        // 【模板方法】禁止子类重写，保证流程固定！
        void cook()
        {
            prepare(); // 准备材料（可变）
            fry();     // 翻炒（可变）
            plate();   // 装盘（可变）
            std::cout << "===== 菜品制作完成 =====\n\n";
        }

    protected:
        // 可变步骤：交给子类实现
        virtual void prepare() = 0;
        virtual void fry() = 0;
        virtual void plate() = 0;
    };

    // 具体子类1：番茄炒蛋
    class TomatoEgg : public AbstractCook
    {
    protected:
        void prepare() override
        {
            std::cout << "准备：番茄、鸡蛋";
        }
        void fry() override
        {
            std::cout << " → 热油，先炒鸡蛋再放番茄翻炒\n";
        }
        void plate() override
        {
            std::cout << "装盘：番茄炒蛋成品\n";
        }
    };

    // 具体子类2：青椒肉丝
    class PepperPork : public AbstractCook
    {
    protected:
        void prepare() override
        {
            std::cout << "准备：青椒、肉丝";
        }
        void fry() override
        {
            std::cout << " → 爆香肉丝，加入青椒大火翻炒\n";
        }
        void plate() override
        {
            std::cout << "装盘：青椒肉丝成品\n";
        }
    };
```

- 钩子函数：一些特殊可变方法，在固定方法中影响判断或循环等执行逻辑，父类提供默认实现

``` cpp
    class AbstractCook
    {
    public:
        virtual ~AbstractCook() = default;
        void cook()
        {
            prepare();
            fry();
            if (needSeasoning())
            { // 钩子：判断是否额外调味
                addSeasoning();
            }
            plate();
        }

    protected:
        virtual void prepare() = 0;
        virtual void fry() = 0;
        virtual void plate() = 0;

        // 钩子函数：提供默认实现，子类可选择重写
        virtual bool needSeasoning() { return true; }
        virtual void addSeasoning()
        {
            std::cout << "添加基础调料\n";
        }
    };
```

### 总结

- 优点：
  - 封装不变部分，扩展可变部分。
  - 提取公共代码，便于维护。
  - 行为父类控制，子类实现。
- 缺点：
  - 子类对父类产生了影响。
- 使用场景：
  - 多子类有公有逻辑。
  - 重要、复杂的算法可以把核心算法设计为模板方法。
  - 重构时经常使用。
- 应用：
  - 项目一般不允许父类调用子类的方法，但模板模式可以视为父类调用子类的方法。
    \## 策略模式
    ![设计模式](../design-pattern/3f4e259c36808aea2c0ad8c1a04877c0d22f0ed4.png)
- **把一系列可变算法 / 业务逻辑封装成独立策略类**，可以互相替换；上下文持有策略对象，实现算法与业务解耦

``` cpp
// 抽象策略
class PaymentStrategy {
public:
    virtual ~PaymentStrategy() = default;
    virtual void pay(double money) = 0;
};

// 具体策略：微信支付
class WechatPay : public PaymentStrategy {
public:
    void pay(double money) override {
        std::cout << "微信支付：" << money << "元\n";
    }
};

// 具体策略：支付宝支付
class AliPay : public PaymentStrategy {
public:
    void pay(double money) override {
        std::cout << "支付宝支付：" << money << "元\n";
    }
};

// 上下文：订单
class OrderContext {
private:
    std::shared_ptr<PaymentStrategy> strategy;
public:
    void setStrategy(std::shared_ptr<PaymentStrategy> s) {
        strategy = std::move(s);
    }
    void doPay(double money) {
        if(strategy) {
            strategy->pay(money);
        } else {
            std::cout << "未设置支付策略\n";
        }
    }
};
```

### 总结：

- 优点：
  - 算法自由切换。
  - 避免使用多重条件判断。
  - 扩展性良好。
- 缺点：
  - 策略类数量过多。
  - 策略需要对外暴露。
- 使用场景：
  - 多个类只有在算法上稍有不同。
  - 算法需要自由切换。
  - 需要屏蔽算法规则。
- 应用：
  \## 观察者模式
  ![设计模式](../design-pattern/4e805d91ed6e7aa6bcac9208edd6d7552891f911.png)
- **一对多**的订阅发布依赖关系。当目标（被观察者）状态发生变化时，所有订阅它的观察者会收到通知，自动更新。
- 使用`std::weak_ptr`保存观察者，避免循环引用内存泄漏。
  \##### 推送模式
- 被订阅的主题把需要的数据发送给观察者

``` cpp
// 抽象观察者
class Observer {
public:
    virtual ~Observer() = default;
    virtual void update(float temp, float humidity) = 0;
};

// 抽象主题（被观察者）
class Subject {
protected:
    std::vector<std::weak_ptr<Observer>> observers;
public:
    virtual ~Subject() = default;
    void attach(std::shared_ptr<Observer> obs) {
        observers.push_back(obs);
    }
    void detach(std::shared_ptr<Observer> obs) {
        for(auto it = observers.begin(); it != observers.end(); ){
            auto sp = it->lock();
            if(!sp || sp == obs){
                it = observers.erase(it);
            }else{
                ++it;
            }
        }
    }
    virtual void notify() = 0;
};

// 具体主题：天气数据
class WeatherData : public Subject {
private:
    float temp{0};
    float humidity{0};
public:
    void setMeasure(float t, float h) {
        temp = t;
        humidity = h;
        notify(); // 状态变化，推送通知
    }
    void notify() override {
        for(auto& wk : observers) {
            if(auto sp = wk.lock()) {
                sp->update(temp, humidity);
            }
        }
    }
};

// 具体观察者：手机APP
class PhoneApp : public Observer {
public:
    void update(float temp, float humidity) override {
        std::cout << "[手机APP] 温度:" << temp << "℃ 湿度:" << humidity << "%\n";
    }
};

// 具体观察者：户外大屏
class ScreenDisplay : public Observer {
public:
    void update(float temp, float humidity) override {
        std::cout << "[户外大屏] 更新天气 " << temp << "℃\n";
    }
};
```

##### 拉取模式

- 被订阅的主题把抽象主题指针发送给观察者，观察者主动拉取需要的数据

``` cpp
virtual void update(Subject* sub) = 0;
```

### 总结：

- 优点：
  - 被观察者和观察者的高扩展性。
  - 完整触发链。
- 缺点：
  - 调试复杂，运行效率不佳，观察者容易单点故障，这时建议异步。
- 使用场景：
  - 可拆分的关联行为。
  - 事件多级触发。
  - 跨系统的消息交换。
  - 经验建议，消息最多转发一次传递两次。
  - 传播过程中消息是随时更改的。
- 应用：
  - 消息队列。
  - 文件系统。
  - EJB3中的MDB，订阅发布模型。
    \## 状态模式
    ![设计模式](../design-pattern/a20284d6e17d9545eb5abc6de702ca07bbeea8c9.png)
- **对象的行为依赖于它的状态，将每一个状态封装为独立状态类；运行时切换状态对象，对象行为随状态自动改变，消除大量 if‑else 状态分支**。

``` cpp
// 前置声明
class PlayerContext;

// 抽象状态
class State {
public:
    virtual ~State() = default;
    virtual void play(PlayerContext* ctx) = 0;
    virtual void pause(PlayerContext* ctx) = 0;
    virtual void stop(PlayerContext* ctx) = 0;
};

// 上下文：播放器
class PlayerContext {
private:
    std::unique_ptr<State> currentState;
public:
    explicit PlayerContext(std::unique_ptr<State> s) : currentState(std::move(s)) {}
    void setState(std::unique_ptr<State> s) {
        currentState = std::move(s);
    }
    // 对外接口，委托给状态对象
    void play()  { currentState->play(this); }
    void pause() { currentState->pause(this); }
    void stop()  { currentState->stop(this); }
};

// 前向声明具体状态类
class StopState;
class PlayState;
class PauseState;

// 具体状态：停止
class StopState : public State {
public:
    void play(PlayerContext* ctx) override;
    void pause(PlayerContext* ctx) override;
    void stop(PlayerContext* ctx) override;
};

// 具体状态：播放
class PlayState : public State {
public:
    void play(PlayerContext* ctx) override;
    void pause(PlayerContext* ctx) override;
    void stop(PlayerContext* ctx) override;
};

// 具体状态：暂停
class PauseState : public State {
public:
    void play(PlayerContext* ctx) override;
    void pause(PlayerContext* ctx) override;
    void stop(PlayerContext* ctx) override;
};

// StopState实现
void StopState::play(PlayerContext* ctx) {
    std::cout << "停止→开始播放\n";
    ctx->setState(std::make_unique<PlayState>());
}
void StopState::pause(PlayerContext* ctx) {
    std::cout << "已停止，无法暂停\n";
}
void StopState::stop(PlayerContext* ctx) {
    std::cout << "已经是停止状态\n";
}

// PlayState实现
void PlayState::play(PlayerContext* ctx) {
    std::cout << "已经正在播放\n";
}
void PlayState::pause(PlayerContext* ctx) {
    std::cout << "播放→暂停\n";
    ctx->setState(std::make_unique<PauseState>());
}
void PlayState::stop(PlayerContext* ctx) {
    std::cout << "播放→停止\n";
    ctx->setState(std::make_unique<StopState>());
}

// PauseState实现
void PauseState::play(PlayerContext* ctx) {
    std::cout << "暂停→继续播放\n";
    ctx->setState(std::make_unique<PlayState>());
}
void PauseState::pause(PlayerContext* ctx) {
    std::cout << "已经是暂停状态\n";
}
void PauseState::stop(PlayerContext* ctx) {
    std::cout << "暂停→停止\n";
    ctx->setState(std::make_unique<StopState>());
}
```

### 总结：

- 优点：
  - 结构清晰，避免大量分支。
  - 遵循开闭原则。
  - 封装性好。
- 缺点：
  - 子类太多。
- 使用场景：
  - 行为随着状态改变而改变的情况。
  - 过多的条件分支语句。
  - 对象状态最好不要超过5个。
- 应用：
  - 建造者模式+状态模式很合适，具有很强的封装作用。
  - 状态机。
    \## 命令模式
    ![设计模式](../design-pattern/fefe9c2cb08f351028c0db88fd789d3bcaa91f3b.png)
- **把请求封装成对象**，让调用者和接收者解耦。

``` cpp
// 接收者：真正做事情
class Light {
public:
    void on()  { std::cout << "开灯\n"; }
    void off() { std::cout << "关灯\n"; }
};

// 抽象命令
class Command {
public:
    virtual ~Command() = default;
    virtual void execute() = 0;
    virtual void undo() = 0;
};

// 具体命令：开灯命令
class LightOnCommand : public Command {
private:
    Light& light;
public:
    LightOnCommand(Light& l) : light(l) {}
    void execute() override { light.on(); }
    void undo() override { light.off(); }
};

// 具体命令：关灯命令
class LightOffCommand : public Command {
private:
    Light& light;
public:
    LightOffCommand(Light& l) : light(l) {}
    void execute() override { light.off(); }
    void undo() override { light.on(); }
};

// 调用者：遥控器
class RemoteControl {
private:
    Command* cmd{nullptr};
public:
    void setCommand(Command* c) { cmd = c; }
    void press() {
        if(cmd) cmd->execute();
    }
    void pressUndo() {
        if(cmd) cmd->undo();
    }
};
```

- 批处理命令

``` cpp
class MacroCommand : public Command {
private:
    std::vector<Command*> cmds;
public:
    void add(Command* c) { cmds.push_back(c); }
    void execute() override {
        for(auto c : cmds) c->execute();
    }
    void undo() override {
        // 撤销逆序执行
        for(auto it = cmds.rbegin(); it != cmds.rend(); ++it) (*it)->undo();
    }
};
```

### 总结

- 优点：
  - 类间解耦，调用者与接收者解耦。
  - 命令可扩展。
  - 结合责任链可以实现命令族解析。
  - 结合模板可以减少命令子类的膨胀。
- 缺点：
  - 命令个数过多会造成命令子类过多。
- 使用场景：
  - 只要是存在命令的地方就可以采用命令模式。
  - 回滚操作也是一个命令，是一般命令的反命令。
- 应用：
  \## 责任链模式
  ![设计模式](../design-pattern/0ebfd692f2c4eb4fa23beafab71bc4632ee8fc34.png)
- **将请求沿着处理者链依次传递**，每个处理者自己判断是否处理该请求；可以处理就执行，不能处理就传给下一个处理者。
  \##### 独占处理
- 请求被其中一个 handler 处理，处理完毕链路终止

``` cpp
// 请求：请假
struct Request {
    std::string name;
    int days;
    Request(std::string n, int d) : name(std::move(n)), days(d) {}
};

// 抽象处理者 Handler
class Handler {
protected:
    std::shared_ptr<Handler> next; // 下一个处理者
public:
    virtual ~Handler() = default;
    void setNext(std::shared_ptr<Handler> h) {
        next = std::move(h);
    }
    virtual void handle(const Request& req) = 0;
};

// 组长
class GroupLeader : public Handler {
public:
    void handle(const Request& req) override {
        if(req.days <= 1) {
            std::cout << "组长审批：" << req.name << "，请假" << req.days << "天，通过\n";
        } else if(next) {
            std::cout << "组长无权审批，上交上级\n";
            next->handle(req);
        } else {
            std::cout << "无人处理该请求\n";
        }
    }
};

// 经理
class Manager : public Handler {
public:
    void handle(const Request& req) override {
        if(req.days <= 3) {
            std::cout << "经理审批：" << req.name << "，请假" << req.days << "天，通过\n";
        } else if(next) {
            std::cout << "经理无权审批，上交上级\n";
            next->handle(req);
        } else {
            std::cout << "无人处理该请求\n";
        }
    }
};

// 总监
class Director : public Handler {
public:
    void handle(const Request& req) override {
        std::cout << "总监审批：" << req.name << "，请假" << req.days << "天，通过\n";
    }
};
```

##### 全链路处理

- 每个 handler 都处理一遍，不终止。例如 web 过滤器、日志过滤器

``` cpp
void handle(Request& req) override {
    doFilter(req); // 当前处理
    if(next) next->handle(req); // 继续往下走，不中断
}
```

### 总结

- 优点：
  - 请求和处理分开。
- 缺点：
  - 链过长会有性能问题。
  - 调试不便。
- 使用场景：
  - 不需要知道请求是谁处理的情况。
  - 作为业务扩展的补救方案。
- 应用：
  - 普通用户与VIP用户的不同业务处理。
    \## 迭代器模式
    ![设计模式](../design-pattern/854bfcb97812046ddd2bf645622483726ebbb243.png)
- **把集合遍历逻辑抽离成迭代器对象**，对外提供统一遍历接口；**不暴露集合内部存储结构**，客户端通过迭代器访问集合元素

``` cpp
// 抽象迭代器
template<typename T>
class Iterator {
public:
    virtual ~Iterator() = default;
    virtual bool hasNext() = 0;
    virtual T next() = 0;
};

// 抽象聚合容器
template<typename T>
class Aggregate {
public:
    virtual ~Aggregate() = default;
    virtual std::unique_ptr<Iterator<T>> createIterator() = 0;
};

// 具体聚合：自定义数组容器
template<typename T>
class ArrayAggregate : public Aggregate<T> {
private:
    T* data;
    int size;
public:
    ArrayAggregate(T arr[], int n) : size(n) {
        data = new T[n];
        for(int i = 0; i < n; ++i) data[i] = arr[i];
    }
    ~ArrayAggregate() { delete[] data; }

    // 内部具体迭代器
    class ArrayIterator : public Iterator<T> {
    private:
        ArrayAggregate* agg;
        int index{0};
    public:
        explicit ArrayIterator(ArrayAggregate* a) : agg(a) {}
        bool hasNext() override {
            return index < agg->size;
        }
        T next() override {
            return agg->data[index++];
        }
    };

    std::unique_ptr<Iterator<T>> createIterator() override {
        return std::make_unique<ArrayIterator>(this);
    }
};
```

### 总结：

- 优点：
- 缺点：
  - 淘汰的设计模式，JAVA提供的Iterator一般够用了。
- 使用场景：
- 应用：
  \## 访问者模式
  ![设计模式](../design-pattern/434ee3d9ad7ecfcb3d198b759f350487ad394b78.png)
- **把作用于对象结构内各个元素的操作抽离到访问者类中**。元素类稳定不变，操作可以不断新增，不用修改元素本身代码。
- ✨双分派：

1.  第一次分派（动态绑定/多态）：调用`element->accept(visitor)`，根据 element 实际类型决定调用哪个 accept。
2.  第二次分派（静态绑定/函数重载）：accept 内部调用`visitor->visit(this)`，`this`是具体元素类型，匹配 visitor 重载方法。

``` cpp
// 前置声明
class FileElement;
class FolderElement;

// 抽象访问者
class Visitor {
public:
    virtual ~Visitor() = default;
    virtual void visit(FileElement*) = 0;
    virtual void visit(FolderElement*) = 0;
};

// 抽象元素
class Element {
public:
    virtual ~Element() = default;
    virtual void accept(Visitor*) = 0;
};

// 具体元素：文件
class FileElement : public Element {
private:
    std::string name;
    int size;
public:
    FileElement(std::string n, int s) : name(std::move(n)), size(s) {}
    std::string getName() const { return name; }
    int getSize() const { return size; }

    void accept(Visitor* v) override {
        v->visit(this); // 双分派：传入自身实际类型FileElement*
    }
};

// 具体元素：文件夹
class FolderElement : public Element {
private:
    std::string name;
    std::vector<std::shared_ptr<Element>> children;
public:
    explicit FolderElement(std::string n) : name(std::move(n)) {}
    void add(std::shared_ptr<Element> e) { children.push_back(std::move(e)); }
    std::string getName() const { return name; }
    const std::vector<std::shared_ptr<Element>>& getChildren() const { return children; }

    void accept(Visitor* v) override {
        v->visit(this);
    }
};

// 具体访问者1：统计总大小
class SizeVisitor : public Visitor {
private:
    int total{0};
public:
    void visit(FileElement* f) override {
        total += f->getSize();
    }
    void visit(FolderElement* dir) override {
        for(auto& child : dir->getChildren()) {
            child->accept(this);
        }
    }
    int getTotal() const { return total; }
};

// 具体访问者2：打印目录树
class PrintVisitor : public Visitor {
private:
    int depth{0};
    void printIndent() { for(int i=0;i<depth;++i) std::cout<<"  "; }
public:
    void visit(FileElement* f) override {
        printIndent();
        std::cout << "- " << f->getName() << "\n";
    }
    void visit(FolderElement* dir) override {
        printIndent();
        std::cout << "+ " << dir->getName() << "\n";
        depth++;
        for(auto& child : dir->getChildren()) {
            child->accept(this);
        }
        depth--;
    }
};
```

### 总结

- 优点：
  - 符合单一职责原则。
  - 访问者的扩展性。
  - 访问者的灵活性很高。
- 缺点：
  - 访问者了解太多细节。
  - 具体元素变更困难。
  - 访问者依赖具体元素，破坏了依赖倒置原则。
- 使用场景：
  - 一个对象有很多类，需要根据具体类别执行不同方法。
  - 需要对类进行很多不同且无关的操作，不想破坏类的单一职责原则。
  - 业务要求遍历多个不同对象。
  - 适用于大型重构模式。
- 应用：
  - 过滤器和拦截器。
    \## 备忘录模式
    ![设计模式](../design-pattern/3f3f57157ee866e47b8115e48049803185bdf4fa.png)
- **在不破坏对象封装性前提下，利用friend特性捕获对象内部状态，并在对象外部保存这个状态；后续可以把对象恢复到该保存点**。

``` cpp
// 备忘录，内部状态私有，只允许原发器访问
class Memento {
private:
    std::string text;
    // 构造私有，只有Originator可以创建
    explicit Memento(std::string s) : text(std::move(s)) {}
    friend class Editor; // 友元：仅Editor可以访问

public:
    // 禁止外部读取内部状态
    std::string getSavedText() const { return text; }
};

// 原发器：编辑器
class Editor {
private:
    std::string content;
public:
    void write(const std::string& s) {
        content += s;
    }
    void show() const {
        std::cout << "当前内容：" << content << "\n";
    }
    // 创建快照（备忘录）
    std::unique_ptr<Memento> save() {
        return std::make_unique<Memento>(content);
    }
    // 从备忘录恢复状态
    void restore(const Memento* m) {
        if(m) content = m->getSavedText();
    }
};

// 负责人：管理历史快照栈，只保管，不解读内容
class Caretaker {
private:
    std::vector<std::unique_ptr<Memento>> history;
public:
    void push(std::unique_ptr<Memento> m) {
        history.push_back(std::move(m));
    }
    std::unique_ptr<Memento> pop() {
        if(history.empty()) return nullptr;
        auto m = std::move(history.back());
        history.pop_back();
        return m;
    }
};
```

### 总结：

- 优点：
- 缺点：
  - 要主动管理备忘录的生命周期，不使用要立刻删除引用。
  - 大量使用备忘录备份大对象会影响系统性能。
- 使用场景：
  - 需要保存和恢复。
  - 需要回滚。
  - 需要监控备份。
- 应用：
  - JDBC驱动。
  - 不要使用数据库的临时表备份数据，而是使用备忘录模式。
    \## 中介者模式
    ![设计模式](../design-pattern/6ff79a797e10301edef029f34e48c36883d419b0.png)
- 解耦多个对象之间的网状交互，所有对象不直接互相通信，统一通过中介者中转沟通，变成星形拓扑
- 每个抽象对象持有一个抽象中介者，所有交互逻辑堆在中介，中介者容易膨胀

``` cpp
    // 前置声明
    class Mediator;

    // 【抽象同事】
    class Colleague
    {
    public:
        virtual ~Colleague() = default;
        explicit Colleague(Mediator &m) : mediator_(m) {}

        virtual void receiveMsg(const std::string &from, const std::string &msg) = 0;
        virtual std::string getName() const = 0;

    protected:
        Mediator &mediator_;
    };

    // 【抽象中介者】
    class Mediator
    {
    public:
        virtual ~Mediator() = default;
        // 同事发送消息，交给中介者分发
        virtual void send(const std::string &from, const std::string &msg) = 0;
        virtual void registerColleague(std::shared_ptr<Colleague> c) = 0;
    };

    // 【具体中介者：聊天室中介】
    class ChatRoomMediator : public Mediator
    {
    public:
        void registerColleague(std::shared_ptr<Colleague> c) override
        {
            colleagues_.push_back(c);
        }

        void send(const std::string &from, const std::string &msg) override
        {
            // 广播给所有其他同事
            for (auto &c : colleagues_)
            {
                if (c->getName() != from)
                {
                    c->receiveMsg(from, msg);
                }
            }
        }

    private:
        std::vector<std::shared_ptr<Colleague>> colleagues_;
    };

    // 【具体同事：聊天室用户】
    class ChatUser : public Colleague
    {
    public:
        ChatUser(Mediator &m, std::string name) : Colleague(m), name_(std::move(name)) {}

        void send(const std::string &text)
        {
            std::cout << "[" << name_ << "] 发送消息：" << text << "\n";
            mediator_.send(name_, text);
        }

        void receiveMsg(const std::string &from, const std::string &msg) override
        {
            std::cout << "【" << name_ << "】收到【" << from << "】消息：" << msg << "\n";
        }

        std::string getName() const override
        {
            return name_;
        }

    private:
        std::string name_;
    };
```

### 总结

- 优点：
  - 减少类间依赖和耦合。
- 缺点：
  - 中介者会很臃肿且逻辑复杂。
- 使用场景：
  - 类图中出现了全连接网络（3个以上的类）。
  - 多个类的依赖行为不确定，有发生改变的可能。
- 应用：
  - MVC中的C就是中介者。
  - 媒体网关。
    \## 解释器模式
    ![设计模式](../design-pattern/f1074786b46f8080559abbdb8872c056ad0ce751.png)
- 定义语言文法，构建抽象语法树，用对象表示语法中每一条规则，实现对自定义语言表达式的解释执行。

``` cpp
// 上下文：保存变量键值对
class Context {
public:
    std::unordered_map<std::string,int> vars;
    void set(const std::string& name, int val) { vars[name] = val; }
    int get(const std::string& name) { return vars[name]; }
};

// 抽象表达式
class AbstractExpression {
public:
    virtual ~AbstractExpression() = default;
    virtual int interpret(Context* ctx) = 0;
};

// 终结符表达式：变量
class VarExpression : public AbstractExpression {
private:
    std::string name;
public:
    explicit VarExpression(std::string n) : name(std::move(n)) {}
    int interpret(Context* ctx) override {
        return ctx->get(name);
    }
};

// 非终结符：加法
class AddExpression : public AbstractExpression {
private:
    std::unique_ptr<AbstractExpression> left;
    std::unique_ptr<AbstractExpression> right;
public:
    AddExpression(std::unique_ptr<AbstractExpression> l, std::unique_ptr<AbstractExpression> r)
        : left(std::move(l)), right(std::move(r)) {}
    int interpret(Context* ctx) override {
        return left->interpret(ctx) + right->interpret(ctx);
    }
};

// 非终结符：减法
class SubExpression : public AbstractExpression {
private:
    std::unique_ptr<AbstractExpression> left;
    std::unique_ptr<AbstractExpression> right;
public:
    SubExpression(std::unique_ptr<AbstractExpression> l, std::unique_ptr<AbstractExpression> r)
        : left(std::move(l)), right(std::move(r)) {}
    int interpret(Context* ctx) override {
        return left->interpret(ctx) - right->interpret(ctx);
    }
};
```

### 总结：

- 优点：
  - 非终结符表达式的扩展性。
- 缺点：
  - 语法规则复杂时类膨胀。
  - 递归调用调试复杂。
  - 递归调用效率低。
- 使用场景：
  - 一个简单语法需要解释。
  - 重复发生的问题。
  - 不要在重要模块中使用。可以使用shell等脚本语言来代替。
- 应用：
  - 实际使用非常少，有MESP、Expression4J等开源的解析工具包。
    \# 模式组合
    \## 门面+单例
- 全局manager，单例作为全局入口，门面封装内部复杂逻辑，对外接口简单。

``` cpp
#include <iostream>

//子模块A：账号模块
class AccountService{
public:
    bool checkAccount(){ std::cout<<"校验账号\n"; return true; }
};
//子模块B：角色模块
class RoleService{
public:
    void loadRole(){ std::cout<<"加载角色数据\n"; }
};
//子模块C：背包模块
class BagService{
public:
    void initBag(){ std::cout<<"初始化背包\n"; }
};

//门面+单例：玩家管理器
class PlayerManager{
private:
    PlayerManager()=default;
    AccountService accSvc;
    RoleService roleSvc;
    BagService bagSvc;
public:
    static PlayerManager& GetInstance(){
        static PlayerManager ins;
        return ins;
    }
    //门面对外简单接口，内部编排多个子系统调用
    bool login(){
        std::cout<<"====玩家登录流程====\n";
        if(!accSvc.checkAccount()) return false;
        roleSvc.loadRole();
        bagSvc.initBag();
        std::cout<<"登录成功\n";
        return true;
    }
};

int main(){
    //外部只调用门面接口，完全不感知内部多个service
    PlayerManager::GetInstance().login();
    return 0;
}
```

## 模板方法+工厂方法

- 模板方法固定整体流程，子类工厂生产需要组件

``` cpp
#include <iostream>
#include <memory>

//产品
class Product{
public:
    virtual ~Product()=default;
    virtual void use()=0;
};
class ProductA:public Product{
public:
    void use() override { std::cout<<"使用产品A\n"; }
};
class ProductB:public Product{
public:
    void use() override { std::cout<<"使用产品B\n"; }
};

//抽象框架类：模板方法定义完整流程
class AbstractFramework{
public:
    virtual ~AbstractFramework()=default;
    //【工厂方法】留给子类重写，负责创建对象
    virtual std::unique_ptr<Product> createProduct() = 0;

    //模板方法：固定整体流程骨架，不可修改
    void runProcess(){
        std::cout<<"1.前置准备\n";
        auto p = createProduct(); //调用子类工厂方法创建组件
        p->use();
        std::cout<<"3.后置收尾\n";
    }
};

//子类A，实现工厂方法产出ProductA
class ConcreteA:public AbstractFramework{
public:
    std::unique_ptr<Product> createProduct() override{
        return std::make_unique<ProductA>();
    }
};

//子类B，实现工厂方法产出ProductB
class ConcreteB:public AbstractFramework{
public:
    std::unique_ptr<Product> createProduct() override{
        return std::make_unique<ProductB>();
    }
};

int main(){
    ConcreteA a;
    a.runProcess();
    std::cout<<"---\n";
    ConcreteB b;
    b.runProcess();
    return 0;
}
```

## 组合+访问者

- 树结构。组合模式建树；访问者对树做多种操作。

``` cpp
#include <iostream>
#include <vector>
#include <memory>

//前置声明
class FileNode;
class DirNode;

//访问者抽象
class Visitor{
public:
    virtual ~Visitor()=default;
    virtual void visit(FileNode*)=0;
    virtual void visit(DirNode*)=0;
};

//抽象树元素
class Node{
public:
    virtual ~Node()=default;
    virtual void accept(Visitor*)=0;
};

//叶子 文件
class FileNode:public Node{
public:
    std::string name;
    int size;
    FileNode(std::string n,int s):name(n),size(s){}
    void accept(Visitor* v) override {v->visit(this);}
};

//目录（组合节点，可以包含子节点）
class DirNode:public Node{
public:
    std::string name;
    std::vector<std::shared_ptr<Node>> children;
    DirNode(std::string n):name(n){}
    void add(std::shared_ptr<Node> n){children.push_back(n);}
    void accept(Visitor* v) override {v->visit(this);}
};

//具体访问者：统计总大小
class SizeVisitor:public Visitor{
public:
    int total=0;
    void visit(FileNode* f) override { total += f->size; }
    void visit(DirNode* d) override {
        for(auto& c:d->children) c->accept(this);
    }
};

int main(){
    auto root = std::make_shared<DirNode>("/root");
    auto f1 = std::make_shared<FileNode>("a.txt",100);
    auto f2 = std::make_shared<FileNode>("b.txt",200);
    root->add(f1);
    root->add(f2);

    SizeVisitor vis;
    root->accept(&vis);
    std::cout<<"总大小："<<vis.total<<"\n";
    return 0;
}
```

## 工厂+策略

- 工厂用于产出不同策略对象

``` cpp
#include <iostream>
#include <memory>

//策略抽象：攻击算法
class AttackStrategy{
public:
    virtual ~AttackStrategy()=default;
    virtual void doAttack()=0;
};

//具体策略A：近战
class MeleeAttack:public AttackStrategy{
public:
    void doAttack() override { std::cout<<"近战砍杀\n"; }
};

//具体策略B：远程
class RangedAttack:public AttackStrategy{
public:
    void doAttack() override { std::cout<<"远程射箭\n"; }
};

//工厂方法：生产策略
class StrategyFactory{
public:
    enum MonsterType{MELEE_MONSTER,RANGED_MONSTER};
    std::unique_ptr<AttackStrategy> createStrategy(MonsterType t){
        if(t == MELEE_MONSTER){
            return std::make_unique<MeleeAttack>();
        }else{
            return std::make_unique<RangedAttack>();
        }
    }
};

//业务怪物
class Monster{
public:
    std::unique_ptr<AttackStrategy> strategy;
    void setStrategy(std::unique_ptr<AttackStrategy> s){
        strategy = std::move(s);
    }
    void attack(){
        strategy->doAttack();
    }
};

int main(){
    StrategyFactory fac;
    Monster m1,m2;

    m1.setStrategy(fac.createStrategy(StrategyFactory::MELEE_MONSTER));
    m1.attack();

    m2.setStrategy(fac.createStrategy(StrategyFactory::RANGED_MONSTER));
    m2.attack();
    return 0;
}
```
