## 工厂和抽象工厂

其实抽象工厂中的每一个创建产品的接口都是一个工厂方法，只不过**工厂方法创建一个产品，而抽象工厂创建一系列产品**。

### **抽象工厂**

一般来说，抽象工厂最简单形态也至少有4个元素：

* 客户端（client）
* 工厂（factory）
* 产品A（product A）
* 产品B（product B）

理解抽象工厂模式的关键在于如何理解“**产品之间特定关系**”。

1. 用户要调用产品之间的这个特定关系
2. 这个特定关系只有产品A和产品B之间才有，所以我们需要产品A和产品B
3. 要获得产品A和产品B，我们要去生产这个产品A和产品B的那个工厂，叫工厂生产这个产品A和产品B

所以理顺抽象工厂的特点是什么?就是如下几个特点：

- 工厂是独立的（独立的类）
- 工厂是生产一整套有产品的（至少要生产两个产品)，这些产品**必须相互是有关系或有依赖的**
- 工厂是可以抽象的，工厂生产是可以抽象的
- 产品是可以抽象的，产品关系是可以抽象的
- 客户端是用来调用并理顺这些产品之间的关系（或指定工作流程）
- 不同工厂生产出的产品实例之间是不接触的，这个是靠客户端来封装实现的。
  - 就是客户端加载工厂实例后，保证只使用这个工厂的生产的产品和产品之间的关系，确保不和其他工厂的产品实例进行接触。

最终当我们调用客户端的行为时候，只要让客户端“加载”实例化的特定工厂，返回结果就是这个**“特定工厂**”所加工出来的“**特定产品**”的**“特定关系”**方法的结果了。

#### 一个常见的抽象工厂如下

```c++
#include <iostream>

class Ammo
{
public:
    virtual ~Ammo() = default;
    virtual  void use() = 0;
};

class Ammo762 : public Ammo
{
public:
	void use() override
	{
        std::cout << "using 7.62mm ammo" << std::endl;
	}
};

class Ammo556 : public Ammo
{
public:
    void use() override
    {
        std::cout << "using 5.56mm ammo" << std::endl;
    }
};

class Gun
{
public:
    virtual ~Gun() = default;
    virtual void shoot() = 0;
    virtual void reload() = 0;
};

class AK47 : public Gun
{
public:
    void shoot() override
    {
        std::cout << "AK47 shooting" << std::endl;
    }
	void reload() override
    {
        std::cout << "reload 7.62mm ammo" << std::endl;
    }
};

class M4A1 : public Gun
{
public:
    void shoot() override
    {
        std::cout << "M4A1 shooting" << std::endl;
    }
    void reload() override
    {
        std::cout << "reload 5.56mm ammo" << std::endl;
    }
};

class Factory
{
public:
    virtual ~Factory() = default;
    virtual std::unique_ptr<Gun> createGun() = 0;
    virtual std::unique_ptr<Ammo> createAmmo() = 0;
};

/**
 * 对于一个阵营来说，生产的枪械和子弹一一对应，但是不同阵营之间的生产方式和结果不相同，而且之间不应该有任何关系
 */
class CTGunFactory : public Factory
{
public:
    std::unique_ptr<Gun> createGun() override
	{
        return std::make_unique<M4A1>();
	}
	std::unique_ptr<Ammo> createAmmo() override
    {
        return std::make_unique<Ammo556>();
    }
};

class TGunFactory : public Factory
{
public:
    std::unique_ptr<Gun> createGun() override
    {
        return std::make_unique<AK47>();
    }
    std::unique_ptr<Ammo> createAmmo() override
    {
        return std::make_unique<Ammo762>();
    }
};

class Human
{
public:
    void shoot() const
    {
        gun->shoot();
    }
    void reload() const 
    {
        ammo->use();
        gun->reload();
    }

    Human(std::unique_ptr<Factory> factory) : gun(factory->createGun()), ammo(factory->createAmmo()) {}
	
protected:
    virtual ~Human() = default;

	/**
	 * 本来应该是 gun 含有成员变量 ammo 的，
	 * 但是这样就会导致 createAmmo() 失去意义，
	 * 所以这么设计
	 */
    std::unique_ptr<Gun> gun;
    std::unique_ptr<Ammo> ammo;
};

class T : public Human
{
public:
    using Human::Human;

    using gun_type = AK47;
    using ammo_type = Ammo762;
};
class CT : public Human
{
public:
    using Human::Human;
	
    using gun_type = M4A1;
    using ammo_type = Ammo556;
};

// 创建对象
auto t = std::make_unique<T>(std::make_unique<TGunFactory>());
auto ct = std::make_unique<CT>(std::make_unique<CTGunFactory>());

t->shoot();
t->reload();

ct->shoot();
ct->reload();
```



### 工厂

工厂方法就两个元素：

- creator（创建者）
- product（产品）

#### 一个常见的工厂如下

```c++
#include <iostream>

class Product
{
public:
	virtual ~Product() = default;
    virtual void use() = 0;
};

class ProductA : public Product
{
public:
	void use() override
	{
        std::cout << "use product A" << std::endl;
	}
};

class ProductB : public Product
{
public:
    using type = ProductB;
	void use() override
	{
        std::cout << "use product B" << std::endl;
	}
};

class Factory
{
public:
	template<typename  product>
	std::shared_ptr<product> createProduct() const
	{
		return std::make_shared<product>();
	}
};

// 创建工厂
const auto factory = std::make_unique<Factory>();
	
auto product_a = factory->createProduct<ProductA>();
product_a->use();

auto product_b = factory->createProduct<ProductB>();
product_b->use();
```



### 抽象工厂和工厂方法不同点

- 抽象工厂关键在于产品之间的抽象关系，所以至少要两个产品；工厂方法在于生成产品，不关注产品间的关系，所以可以只生成一个产品。

- 抽象工厂中客户端把产品的抽象关系理清楚，在最终使用的时候，一般使用客户端（和其接口），产品之间的关系是被封装固定的；而工厂方法是在最终使用的时候，使用产品本身（和其接口）。

- 抽象工厂更像一个复杂版本的策略模式，策略模式通过更换策略来改变处理方式或者结果；而抽象工厂的客户端，通过更改工厂还改变结果。所以在使用的时候，就使用客户端和更换工厂，而看不到产品本身。

- 工厂方法目的是生产产品，所以能看到产品，而且还要使用产品。当然，如果产品在创建者内部使用，那么工厂方法就是为了完善创建者，从而可以使用创建者。另外创建者本身是不能更换所生产产品的。

- 抽象工厂的工厂是类；工厂方法的工厂是方法。

  - 抽象工厂的工厂类就做一件事情生产产品。生产的产品给客户端使用，绝不给自己用。
  - 工厂方法生产产品，可以给系统用，可以给客户端用，也可以自己这个类使用。自己这个类除了这个工厂方法外，还能有其他功能性的方法

  

