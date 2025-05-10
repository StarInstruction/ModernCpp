# 1. 继承

## 1. 构造

### 派生类的构造函数 (Constructor)

1.  **执行顺序**：
    *   **首先，调用基类的构造函数**。目的是完成基类部分的初始化。
        *   如果派生类的构造函数没有在成员初始化列表中显式地调用基类的某个构造函数，那么编译器会自动尝试调用基类的**默认构造函数** (无参构造函数)。
        *   如果基类没有默认构造函数，或者派生类需要调用基类的带参数构造函数，则必须在派生类构造函数的**成员初始化列表**中显式指定。
    *   **其次，按照成员在派生类中声明的顺序，初始化派生类自己新增的成员变量**。
        *   如果成员变量在成员初始化列表中被初始化，则使用该初始化。
        *   如果成员变量是类类型且没有在初始化列表中，则调用其默认构造函数。
        *   内置类型成员若未在初始化列表指定，则其值未定义 (除非是静态或全局变量，它们会被零初始化)。
    *   **最后，执行派生类构造函数的函数体**。

2.  **调用基类构造函数的方式**：
    使用成员初始化列表：
    ```cpp
    class Base {
    public:
        int base_val;
        Base() : base_val(0) {
            std::cout << "Base default constructor called." << std::endl;
        }
        Base(int val) : base_val(val) {
            std::cout << "Base parameterized constructor called with value: " << val << std::endl;
        }
    };
    
    class Derived : public Base {
    public:
        int derived_val;
    
        // 显式调用基类的带参构造函数
        Derived(int b_val, int d_val) : Base(b_val), derived_val(d_val) {
            std::cout << "Derived constructor called. derived_val = " << derived_val << std::endl;
        }
    
        // 如果不显式调用，会尝试调用 Base()
        Derived(int d_val) : derived_val(d_val) { // 隐式调用 Base()
            std::cout << "Derived constructor (implicit Base()) called. derived_val = " << derived_val << std::endl;
        }
    };
    ```

3.  **构造函数的继承 (C++11及以后)**：
    派生类可以使用 `using` 声明来继承基类的构造函数，这样就不必为每个基类构造函数都写一个对应的派生类构造函数。
    
    ```cpp
    class DerivedWithUsing : public Base {
    public:
        int derived_extra;
        using Base::Base; // 继承 Base 的所有构造函数
    
        // 如果需要，可以添加新的构造函数或覆盖特定签名
        DerivedWithUsing(int b_val, int d_extra_val) : Base(b_val), derived_extra(d_extra_val) {
            std::cout << "DerivedWithUsing specific constructor." << std::endl;
        }
    };
    
    // DerivedWithUsing d(10); // 会调用继承来的 Base(int val)
    // d.derived_extra 未初始化 (除非 Base 构造函数初始化它，但这通常不是其职责)
    // 通常，如果继承的构造函数不能完全初始化派生类，你还是需要提供自己的构造函数。
    // 或者，如果继承的构造函数后，新增成员有默认值，那也可以：
    // int derived_extra = 100;
    ```
    当使用 `using Base::Base;` 时，编译器会为派生类生成与基类构造函数签名相匹配的构造函数。这些生成的构造函数会调用对应的基类构造函数，并对派生类中新增的成员进行默认初始化 (如果它们有类内初始值或默认构造函数)。

## 2. 析构

### 派生类的析构函数 (Destructor)

1.  **执行顺序 (与构造函数相反)**：
    *   **首先，执行派生类析构函数的函数体**。
    *   **其次，按照成员在派生类中声明的相反顺序，销毁派生类自己新增的成员变量** (如果它们是类类型，则调用它们的析构函数)。
    *   **最后，自动调用基类的析构函数**。完成基类部分的清理。

2.  **自动调用**：
    派生类的析构函数执行完毕后，基类的析构函数会被**自动调用**，不需要也不应该在派生类的析构函数中显式调用基类的析构函数 (例如 `Base::~Base();` 是错误的)。

3.  **虚析构函数 (Virtual Destructor)**：
    这是一个非常重要的概念，尤其是在使用基类指针指向派生类对象并打算通过该指针 `delete` 对象时。

    *   **问题**：如果基类的析构函数不是虚函数，当你通过基类指针删除一个派生类对象时：
        ```cpp
        Base* ptr = new Derived();
        delete ptr; // 如果 Base::~Base() 不是 virtual，则只会调用 Base::~Base()
                    // Derived::~Derived() 不会被调用，导致派生类部分的资源可能泄露
        ```
        这会导致未定义行为，通常表现为派生类部分的资源没有被正确释放。

    *   **解决方案**：将基类的析构函数声明为 `virtual`。
        ```cpp
        class Base {
        public:
            virtual ~Base() { // 虚析构函数
                std::cout << "Base destructor called." << std::endl;
            }
            // ...
        };
        
        class Derived : public Base {
        public:
            ~Derived() override { // 最好也用 override 显式标记
                std::cout << "Derived destructor called." << std::endl;
            }
            // ...
        };
        ```
        当基类的析构函数是虚函数时，通过基类指针 `delete` 对象会确保调用链是正确的：首先调用实际对象类型 (派生类) 的析构函数，然后自动调用基类的析构函数。

    *   **规则**：如果一个类打算作为基类，并且可能会有通过基类指针删除派生类对象的情况，那么它的析构函数**应该**声明为 `virtual`。如果一个类不打算作为基类，或者确定不会通过基类指针删除派生类对象，则虚析构函数不是必需的 (它会带来一点点开销，如vtable指针和虚函数调用)。

## 3. 访问级别

1.  **从基类继承来的数据成员**：
    *   派生类会继承基类中所有**非`private`**的数据成员（即 `public` 和 `protected` 成员）。
    *   基类的`private`成员虽然也被派生类对象在内存中“包含”了（派生类对象的大小会包含基类私有成员的空间），但是它们**不能被派生类的成员函数直接访问**。它们仍然是基类私有的，只能通过基类的`public`或`protected`成员函数来间接访问（如果基类提供了这样的接口）。

2.  **派生类自己新增的数据成员**：
    *   派生类可以在其定义中声明新的数据成员，这些成员是派生类特有的。

------

**1. 对于从基类继承来的数据成员：**

其在派生类中的访问控制级别会受到继承方式的影响。规则如下（取成员在基类中的访问级别和继承方式中“更严格”的那个）：

| 基类成员访问级别 | `public` 继承      | `protected` 继承   | `private` 继承     |
| :--------------- | :----------------- | :----------------- | :----------------- |
| `public`         | `public`           | `protected`        | `private`          |
| `protected`      | `protected`        | `protected`        | `private`          |
| `private`        | 在派生类中不可访问 | 在派生类中不可访问 | 在派生类中不可访问 |

**2. 对于派生类自己新增的数据成员：**

这些成员的访问控制由它们在派生类中声明时使用的访问修饰符 (`public`, `protected`, `private`) 决定，与继承方式无关。

- 如果在派生类中声明为 `public`，则它们是公有的。
- 如果在派生类中声明为 `protected`，则它们是保护的。
- 如果在派生类中声明为 `private`，则它们是私有的。



```c++
#include <iostream>
#include <string>

class Base {
private:
    int base_private_data = 1; // 基类私有

protected:
    int base_protected_data = 2; // 基类保护

public:
    int base_public_data = 3; // 基类公有

    void printBaseData() const {
        std::cout << "Base private: " << base_private_data << std::endl;
        std::cout << "Base protected: " << base_protected_data << std::endl;
        std::cout << "Base public: " << base_public_data << std::endl;
    }
};

// Public inheritance
class DerivedPublic : public Base {
public:
    int derived_public_data = 10;

    void accessBaseMembers() {
        // std::cout << base_private_data; // 错误: 基类的private成员不可访问
        std::cout << "DerivedPublic accessing base_protected_data: " << base_protected_data << std::endl; // OK: protected成员在派生类中可访问 (这里是protected)
        std::cout << "DerivedPublic accessing base_public_data: " << base_public_data << std::endl;     // OK: public成员在派生类中可访问 (这里是public)
        std::cout << "DerivedPublic accessing derived_public_data: " << derived_public_data << std::endl;
    }
};

// Protected inheritance
class DerivedProtected : protected Base {
public:
    int derived_public_data = 20;

    void accessBaseMembers() {
        // std::cout << base_private_data; // 错误: 基类的private成员不可访问
        std::cout << "DerivedProtected accessing base_protected_data: " << base_protected_data << std::endl; // OK: protected成员在派生类中可访问 (这里是protected)
        std::cout << "DerivedProtected accessing base_public_data: " << base_public_data << std::endl;     // OK: public成员在派生类中可访问 (这里是protected)
        std::cout << "DerivedProtected accessing derived_public_data: " << derived_public_data << std::endl;
    }
};

// Private inheritance
class DerivedPrivate : private Base {
public:
    int derived_public_data = 30;

    void accessBaseMembers() {
        // std::cout << base_private_data; // 错误: 基类的private成员不可访问
        std::cout << "DerivedPrivate accessing base_protected_data: " << base_protected_data << std::endl; // OK: protected成员在派生类中可访问 (这里是private)
        std::cout << "DerivedPrivate accessing base_public_data: " << base_public_data << std::endl;     // OK: public成员在派生类中可访问 (这里是private)
        std::cout << "DerivedPrivate accessing derived_public_data: " << derived_public_data << std::endl;
    }
};


int main() {
    std::cout << "--- DerivedPublic ---" << std::endl;
    DerivedPublic d_pub;
    d_pub.accessBaseMembers();
    std::cout << "Outside access to d_pub.base_public_data: " << d_pub.base_public_data << std::endl; // OK: 继承为public
    // std::cout << d_pub.base_protected_data; // 错误: base_protected_data 在DerivedPublic中是protected
    // std::cout << d_pub.base_private_data; // 错误: 不可访问
    std::cout << "Outside access to d_pub.derived_public_data: " << d_pub.derived_public_data << std::endl; // OK

    std::cout << "\n--- DerivedProtected ---" << std::endl;
    DerivedProtected d_prot;
    d_prot.accessBaseMembers();
    // std::cout << d_prot.base_public_data; // 错误: base_public_data 在DerivedProtected中是protected
    // std::cout << d_prot.base_protected_data; // 错误: base_protected_data 在DerivedProtected中是protected
    // std::cout << d_prot.base_private_data; // 错误: 不可访问
    std::cout << "Outside access to d_prot.derived_public_data: " << d_prot.derived_public_data << std::endl; // OK

    std::cout << "\n--- DerivedPrivate ---" << std::endl;
    DerivedPrivate d_priv;
    d_priv.accessBaseMembers();
    // std::cout << d_priv.base_public_data; // 错误: base_public_data 在DerivedPrivate中是private
    // std::cout << d_priv.base_protected_data; // 错误: base_protected_data 在DerivedPrivate中是private
    // std::cout << d_priv.base_private_data; // 错误: 不可访问
    std::cout << "Outside access to d_priv.derived_public_data: " << d_priv.derived_public_data << std::endl; // OK

    // 派生类的派生类 (以DerivedPublic为例)
    class GrandChild : public DerivedPublic {
    public:
        void accessGrandParentMembers() {
            // std::cout << base_private_data; // 错误
            std::cout << "GrandChild accessing base_protected_data: " << base_protected_data << std::endl; // OK: DerivedPublic中是protected
            std::cout << "GrandChild accessing base_public_data: " << base_public_data << std::endl;     // OK: DerivedPublic中是public
            std::cout << "GrandChild accessing derived_public_data: " << derived_public_data << std::endl; // OK
        }
    };
    std::cout << "\n--- GrandChild (from DerivedPublic) ---" << std::endl;
    GrandChild gc;
    gc.accessGrandParentMembers();
    std::cout << "Outside access to gc.base_public_data: " << gc.base_public_data << std::endl; // OK

    return 0;
}
```

# 2. 多态



## 1. virtual