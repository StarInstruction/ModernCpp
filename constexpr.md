# 1.动机

在 C++ 中，`constexpr` 关键字的引入主要是为了将计算从运行时提前到编译时，从而带来多方面的好处。其主要动机可以概括为以下几点：

1. **性能提升 (Performance Improvement):**
   - **编译时计算:** `constexpr` 允许变量和函数的结果在编译期间计算出来。这意味着程序运行时无需再执行这些计算，从而减少了运行时间开销，提升了程序执行效率。这对于性能敏感的应用，如游戏、嵌入式系统和高性能计算等，尤为重要。
   - **减少内存占用:** 在编译时计算出的常量可以直接嵌入到代码中，或者存储在只读内存区域，可能比运行时计算和存储变量占用更少的内存。
2. **增强代码表达能力和安全性 (Enhanced Code Expressiveness and Safety):**
   - **真正的常量:** `const` 关键字在 C++ 中表示“只读”，但其初始值可能在运行时才能确定。而 `constexpr` 确保了变量或表达式在编译时就是一个常量值。这使得程序员可以更明确地表达其意图，即某些值必须在编译期间就确定下来。
   - **编译时错误检查:** 如果一个 `constexpr` 表达式或函数不符合编译时计算的规则（例如，包含不允许在编译时执行的操作），编译器会报错。这有助于在早期发现潜在的逻辑错误，而不是等到运行时才暴露出来。
   - **替代宏定义:** 在 `constexpr` 出现之前，宏定义常被用来定义编译时常量。然而，宏定义缺乏类型安全，容易出错，并且难以调试。`constexpr` 提供了类型安全、作用域可控的编译时常量和函数，是宏的更优替代方案。
3. **更广泛的应用场景 (Broader Application Scenarios):**
   - **模板元编程 (Template Metaprogramming):** `constexpr` 函数可以用于模板元编程，使得在编译时进行更复杂的计算和类型操作成为可能，代码也更易读和维护。
   - **数组大小和枚举值:** `constexpr` 变量可以用来定义数组的大小、非类型模板参数、枚举初始值等，这些场景都要求在编译时确定常量值。
   - **静态初始化:** `constexpr` 可以用于确保静态对象在编译时初始化，避免了“静态初始化顺序问题 (static initialization order fiasco)”以及多线程环境下的初始化竞争条件。
   - **`if constexpr` (C++17):** `if constexpr` 语句允许根据编译时条件来包含或排除代码块，使得编写更灵活、更高效的模板代码成为可能，避免了实例化不必要的代码路径。
4. **支持更现代的 C++ 特性:**
   - C++11 引入 `constexpr` 是为了让 C++ 语言本身能够进行更多的编译时计算，为后续的语言特性（如 C++14、C++17、C++20 及以后对 `constexpr` 功能的扩展，例如 `constexpr` lambda、`constexpr new`、`consteval` 等）打下了基础。这些扩展进一步增强了在编译时编程的能力。

**总结来说，`constexpr` 的核心动机在于将计算尽可能地提前到编译阶段，以牺牲一定的编译时间为代价，换取程序运行时更高的效率、更好的安全性和更强的表达能力。** 它是 C++ 语言向着更安全、更高效、更现代方向发展的重要一步，使得开发者能够编写出在编译期就能完成更多工作的代码，从而生成更优化、更可靠的程序。





# 2.语言规范

`constexpr` 是 C++ 中一个强大的关键字，它允许在编译时进行计算和求值。这带来了诸多好处，例如提升运行时性能、增强代码的类型安全以及在编译期进行更强的检查。以下是 `constexpr` 的主要用法：

## 1. `constexpr` 变量

`constexpr` 变量是真正意义上的编译时常量。它们必须在声明时用==常量表达式==初始化。

- **特性：**

  - 必须在编译时确定其值。
  - 隐式地具有 `const` 属性（一旦初始化，其值不能被修改）。
  - 必须是字面值类型 (Literal Type)。常见的字面值类型包括所有内置类型（如 `int`, `float`, `char`）、指针类型以及满足特定条件的类类型。

- **用法：**

  C++

  ```c++
  constexpr int max_iterations = 100;
  constexpr double pi = 3.1415926535;
  constexpr const char* app_name = "My Application";
  
  int main() {
      int arr[max_iterations]; // 正确，max_iterations 是编译时常量
      // max_iterations = 200; // 错误，constexpr 变量是 const 的
      return 0;
  }
  ```

## 2. `constexpr` 函数

`constexpr` 函数是指那些**可以**在编译时求值的函数。如果传递给 `constexpr` 函数的所有参数都是常量表达式，那么对该函数的调用就可以在编译时完成。如果参数在运行时才能确定，那么 `constexpr` 函数的行为就像普通函数一样，在运行时执行。

- **特性 (随 C++ 标准版本有所放宽)：**

  - 返回值类型必须是字面值类型。
  - 所有参数类型也必须是字面值类型。
  - **C++11 中：** 函数体通常非常受限，基本上只能包含一个 `return` 语句，以及一些静态断言、`typedef` 等。不允许局部变量、循环、复杂的 `if` 语句（除非是 `?:` 运算符）。
  - **C++14 及以后：** 限制大幅放宽。函数体可以包含局部变量的声明和初始化、条件语句（如 `if`, `switch`）、循环语句（如 `for`, `while`）以及其他一些简单的语句，只要这些操作在编译时常量上下文中是合法的。
  - 不能是虚函数。
  - 当成员函数被声明为 `constexpr` 时，该成员函数也隐式地是 `const` 成员函数（除非是构造函数或析构函数）。

- **用法：**

  C++

  ```c++
  // C++11
  constexpr int factorial_cpp11(int n) {
      return (n <= 1) ? 1 : (n * factorial_cpp11(n - 1));
  }
  
  // C++14 及以后
  constexpr int factorial_cpp14(int n) {
      int res = 1;
      for (int i = 2; i <= n; ++i) {
          res *= i;
      }
      return res;
  }
  
  constexpr int square(int x) {
      return x * x;
  }
  
  int main() {
      constexpr int val1 = factorial_cpp11(5); // 编译时计算
      constexpr int val2 = factorial_cpp14(5); // 编译时计算
      int arr1[val1];                         // 正确
  
      int runtime_val = 6;
      int val3 = factorial_cpp14(runtime_val); // 运行时计算
  
      constexpr int num = square(10); // 编译时计算
      std::cout << num << std::endl;   // 输出 100
  
      int x = 5;
      std::cout << square(x) << std::endl; // 运行时计算，输出 25
      return 0;
  }
  ```

C++11：`constexpr` 函数体里出现对非‑`constexpr` 函数的调用是非法的，代码根本通不过编译。  
• C++14 及以后：这样的代码可以通过编译，但当且仅当编译器在「求常量表达式」的那条执行路径上没有真的去调用那个非‑`constexpr` 函数。  
  – 如果常量求值时走到了非‑`constexpr` 调用 → 不是常量表达式 → 在必须是常量表达式的上下文中会报错；  
  – 如果在运行期调用或者常量求值走到了别的分支 → 一切正常，只是在运行期执行。

## 3. `constexpr` 构造函数

`constexpr` 构造函数允许创建用户定义类型的编译时常量对象。

- **特性：**

  - 类必须是字面值类型。这意味着：
    - 所有非静态数据成员都必须是字面值类型。
    - 类不能有虚基类。
    - 类不能有虚成员函数 (C++20 中有所放宽，`constexpr` 虚函数是允许的，但有其特定规则)。
    - 所有基类也必须是字面值类型。
    - 构造函数体在 C++11 中非常受限（通常为空，或只进行成员初始化列表中的初始化）。C++14 及以后放宽了限制，允许更复杂的逻辑，前提是所有操作都可以在编译时完成。
    - 所有用于初始化非静态数据成员和基类子对象的构造函数都必须是 `constexpr` 构造函数。
    - 析构函数必须是平凡的 (trivial)。

- **用法：**

  C++

  ```c++
  class Point {
  public:
      constexpr Point(double x_val, double y_val) : x(x_val), y(y_val) {}
  
      constexpr double get_x() const { return x; }
      constexpr double get_y() const { return y; }
  
  private:
      double x;
      double y;
  };
  
  int main() {
      constexpr Point p1(1.0, 2.0);
      constexpr double x_coord = p1.get_x(); // 编译时计算
      // p1.x = 3.0; // 错误, p1 是 const 的
  
      double arr[static_cast<int>(x_coord)]; // 可以，如果 x_coord 能转换为整型常量
  
      return 0;
  }
  ```

## 4. `if constexpr` (C++17 及以后)

`if constexpr` 语句允许根据编译时条件来选择性地编译代码块。如果条件为 `false`，则 `else` 分支（如果存在）被编译，`if` 分支被丢弃；反之亦然。被丢弃的分支甚至不需要语法上完全正确（例如，如果它引用了在当前模板实例化中不存在的类型或成员），只要它在语法上是可解析的。

- **特性：**

  - 条件必须是常量表达式。
  - 主要用于模板元编程，以根据类型特性选择不同的实现。

- **用法：**

  C++

  ```c++
  #include <string>
  #include <type_traits> // For std::is_pointer_v, std::is_integral_v
  
  template<typename T>
  auto get_value(T t) {
      if constexpr (std::is_pointer_v<T>) {
          return *t; // 只有当 T 是指针类型时才编译这行
      } else {
          return t;  // 只有当 T 不是指针类型时才编译这行
      }
  }
  
  template<typename T>
  void process(T val) {
      if constexpr (std::is_integral_v<T>) {
          std::cout << "Processing an integral type: " << val << std::endl;
      } else if constexpr (std::is_same_v<T, std::string>) {
          std::cout << "Processing a string: " << val << std::endl;
      } else {
          std::cout << "Processing a different type." << std::endl;
      }
  }
  
  int main() {
      int a = 10;
      int* ptr_a = &a;
      std::cout << get_value(a) << std::endl;     // 输出 10
      std::cout << get_value(ptr_a) << std::endl; // 输出 10
  
      process(123);
      process(std::string("hello"));
      process(3.14);
      return 0;
  }
  ```

## 5. `constexpr` Lambda 表达式 (C++17 及以后)

从 C++17 开始，Lambda 表达式也可以是 `constexpr` 的，这意味着它们可以在编译时创建和调用（如果满足 `constexpr` 函数的要求）。

- **特性：**

  - 如果 Lambda 函数体满足 `constexpr` 函数的要求，并且所有捕获的变量（如果有的话）都是字面值类型并且在编译时是常量，那么该 Lambda 表达式可以被隐式地认为是 `constexpr` 的。
  - 也可以显式地用 `constexpr` 关键字声明 Lambda。
  - `constexpr` Lambda 可以用于初始化 `constexpr` 变量，或作为 `constexpr` 函数的参数/返回值。

- **用法：**

  C++

  ```c++
  int main() {
      constexpr auto add = [](int a, int b) constexpr { // 显式 constexpr (C++17)
          return a + b;
      };
  
      // C++17 及以后，如果满足条件，lambda 可以是隐式 constexpr
      constexpr auto multiply = [](int a, int b) {
          return a * b;
      };
  
      constexpr int sum_val = add(5, 3);       // 编译时计算
      constexpr int product_val = multiply(4, 5); // 编译时计算
  
      std::cout << "Sum: " << sum_val << std::endl;
      std::cout << "Product: " << product_val << std::endl;
  
      // 用于模板
      constexpr auto create_array_filler = [](auto val) {
          return [val](int index) { return val + index; };
      };
  
      constexpr auto filler = create_array_filler(10);
      constexpr int arr_val = filler(5); // 编译时计算 (10 + 5 = 15)
      std::cout << "Array value: " << arr_val << std::endl;
  
      return 0;
  }
  ```

**`constexpr` 的演进和限制**

- **C++11：** `constexpr` 函数的限制非常严格，函数体基本上只能包含一个 `return` 语句。
- **C++14：** 大大放宽了 `constexpr` 函数的限制，允许使用局部变量、循环和条件语句等。
- **C++17：** 引入了 `if constexpr` 和 `constexpr` Lambda，进一步增强了编译时编程的能力。
- **C++20 及以后：** 继续扩展 `constexpr` 的能力，例如允许在 `constexpr` 函数中使用 `try-catch`（尽管有限制）、允许 `constexpr` 分配内存 (`new` 和 `delete`，但对象生命周期必须在常量表达式内结束或使用特定机制）、`constexpr` 虚函数等。

**通用限制和注意事项：**

- **字面值类型 (Literal Types)：** `constexpr` 变量、函数参数和返回类型通常必须是字面值类型。
- **编译时上下文：** 只有当所有必要的输入（如函数参数、对象成员）在编译时都可知时，`constexpr` 表达式才会在编译时求值。否则，它可能会在运行时求值（对于 `constexpr` 函数而言）。
- **接口的一部分：** 将函数或构造函数声明为 `constexpr` 是其接口的一部分。移除 `constexpr` 可能会破坏依赖于其编译时求值能力的客户端代码。因此，只有当函数的核心逻辑确实适合并且有益于编译时计算时，才应将其标记为 `constexpr`。
- **调试：** 调试编译时错误可能比调试运行时错误更具挑战性，因为错误信息可能更复杂，并且直接的运行时调试器步骤不可用。

`constexpr` 是现代 C++ 中一个非常重要的特性，它使得开发者能够编写出更高效、更安全、更具表达力的代码。





## 6. const vs constexpr

| 特性     | const                         | constexpr                      |
| -------- | ----------------------------- | ------------------------------ |
| 修饰对象 | 只读属性 (可能在运行期初始化) | 必须初始化且倾向于编译期求值   |
| 修饰函数 | N/A                           | 允许/强制编译期调用            |
| 求值时机 | 取决于初始化表达式            | 能编译期则编译期，不行则运行期 |



##  7.`consteval` 与 `constexpr` 的区别（C++20 彩蛋）

```cpp
consteval int only_compile_time(int x) { return x * x; }
```

• `consteval` 表示“*无条件* 编译期求值”，调用时若不满足，编译器直接报错。  
• `constexpr` 则是“*尽量* 编译期求值，实在不行就运行期”。

---

## 8. 常见坑 & 提示

1. **动态内存**：`new` / `malloc` 统统不行（C++20 部分允许 `new`，但多数库函数还不 constexpr）。  
2. **I/O**：`std::cout`, `printf` 等纯 runtime 行为禁止在 constexpr 里用。  
3. **调试**：编译期调试不好下断点，必要时写双份实现或用单元测试 + `static_assert`。  
4. **编译时间**：把一堆复杂计算塞进 constexpr，可能让编译时间爆炸，需要权衡。  

---

## 9. 一句话总结

constexpr 函数 =  
“看起来像普通函数，但**当你给它常量实参时，它偷偷帮你在编译阶段就把结果算好**；如果不满足条件，又会优雅地退让成普通函数”。  

学会它后，你能写出既安全又高效，还兼顾可读性的现代 C++ 代码 ✨



# 3.相关概念

## 1. Constant expressions

在 C++ 中，**常量表达式 (Constant Expression)** 的核心定义是：

**一个其值可以在编译期间（而不是运行期间）被编译器计算出来的表达式。**

这意味着编译器能够在程序实际执行之前就确定该表达式的结果。

更详细地来说，一个表达式要成为常量表达式，通常需要满足以下条件：

1. **字面值 (Literals)**：
   - 整数常量 (如 5, 0xFF, 100L)。
   - 浮点数常量 (如 3.14, 1.0e-5f)。
   - 字符常量 (如 'a', L'あ')。
   - 字符串字面值 (如 "hello") 本身是常量，但用作指针时，指针本身可能不是常量表达式，除非是 constexpr char* 指向字符串字面值。
   - 布尔常量 (true, false)。
   - nullptr (空指针常量)。
2. **枚举成员 (Enumerators)**：枚举类型中的成员值。
3. **const 变量 (特定情况下)**：
   - 用常量表达式初始化的整型或枚举类型的 const 变量。
   - 在 C++11 之前，这是创建编译时常量的一种主要方式。
4. **constexpr 变量**：
   - 使用 constexpr 关键字声明的变量，其初始化器必须是常量表达式。这类变量本身就是常量表达式。
5. **sizeof 和 alignof 运算符的结果**：这些运算符在编译时求值。
6. **只包含常量表达式的操作数和运算符的组合**：
   - 算术运算 (+, -, *, /, % 等)，前提是操作数是常量表达式且运算不会导致未定义行为（如除以零）。
   - 位运算 (&, |, ^, ~, <<, >>)。
   - 逻辑运算 (&&, ||, !)。
   - 比较运算 (==, !=, <, >, <=, >=)。
   - 条件运算符 (? :)，如果条件和两个结果分支都是常量表达式。
7. **对 constexpr 函数的调用**：
   - 如果一个函数被声明为 constexpr，并且传递给它的所有参数都是常量表达式，那么对该函数的调用本身也可以是一个常量表达式。
8. **constexpr 构造函数创建的对象**：
   - 如果一个类的构造函数是 constexpr，并且所有构造函数参数都是常量表达式，那么创建的对象（如果其类型是字面值类型）的成员访问或对 constexpr 成员函数的调用（在特定条件下）可以是常量表达式的一部分。

**简单来说，常量表达式就是编译器在编译代码时就能“看透”并算出具体数值的表达式。**

引入 constexpr 关键字 (C++11) 极大地扩展和明确了常量表达式的概念，允许用户定义更复杂的编译时计算（如函数和用户定义类型的构造函数）。

**为什么需要常量表达式？**
它们是必需的，用于以下场景：

- 数组大小声明（非 VLA）。
- static_assert 的条件。
- 模板的非类型参数。
- case 语句的标签。
- 枚举成员的初始化值。
- alignas 说明符。

使用常量表达式可以带来性能提升（因为计算在编译时完成）和更强的编译时检查。

## 2. Literal type

在 C++ 中，**字面值类型 (Literal Types)** 是指那些其对象可以在编译时被完全构造、操作和确定的类型。这个特性使得它们可以在常量表达式 (constexpr) 中使用。

简单来说，一个类型是字面值类型，如果：

1. 它的对象可以**在编译时被创建** (通过 constexpr 构造函数或聚合初始化)。
2. 它的对象可以**在编译时被销毁** (意味着它必须有一个平凡的析构函数)。
3. 它的所有相关操作（如成员访问）如果用于 constexpr 上下文，也必须能在编译时完成。

**正式一点的定义，一个类型 T 是字面值类型，如果它满足下列条件之一：**

1. **void 类型**： void 本身被认为是字面值类型。
2. **标量类型 (Scalar Types)**：
   - 算术类型 (如 int, float, char, bool)。
   - 枚举类型 (enum)。
   - 指针类型 (如 int*, const char*)。
   - 指针到成员类型。
   - std::nullptr_t。
3. **字面值类型的数组 (Array of Literal Type)**：例如，int[10] 是字面值类型，如果 int 是字面值类型。
4. **引用类型 (Reference Types)**：引用到字面值类型的引用。
5. **类类型 (Class Types - class 或 struct) 或其 cv 限定版本 (const/volatile)**，该类类型需要满足以下所有条件：
   - **a. 平凡的析构函数 (Trivial Destructor)**：
     - 析构函数不能是用户提供的 (即，要么没有声明析构函数，要么使用 = default 声明)。
     - 如果析构函数是隐式定义的或 = default 的，那么所有非静态数据成员和所有直接基类的析构函数也必须是平凡的。
   - **b. 聚合类型 (Aggregate Type) 或者 至少有一个 constexpr 构造函数**：
     - **聚合类型**：没有用户声明的构造函数，没有私有或受保护的非静态数据成员，没有基类，没有虚函数。这种类型可以通过花括号列表初始化。
     - **constexpr 构造函数**：该类至少有一个构造函数被声明为 constexpr (这个构造函数不能是拷贝或移动构造函数，除非该类没有其他 constexpr 构造函数)。
   - **c. 所有非静态数据成员和基类都是字面值类型**：
     - 类的所有非静态数据成员的类型必须是字面值类型。
     - 如果类有基类，那么所有基类也必须是字面值类型。
     - (对于 C++11，非静态数据成员必须是 public 的，或者可以通过 constexpr 构造函数初始化。C++14 及以后放宽了这个限制)。

**为什么字面值类型很重要？**

- **constexpr 的基础**：只有字面值类型的对象才能被声明为 constexpr。 constexpr 函数的参数和返回类型通常也必须是字面值类型。
- **编译时计算**：它们允许在编译时创建和操作对象，这对于性能优化、编译时断言 (static_assert) 和元编程非常关键。
- **存储在 ROM 中**： constexpr 对象（其类型为字面值类型）的值在编译时确定，因此可以存储在只读存储器 (ROM) 中。

**简单示例：**

```c++
      // 标量类型是字面值类型
constexpr int count = 5;
constexpr double pi = 3.14;

// 简单的聚合结构体是字面值类型
struct Point2D_Aggregate {
    int x;
    int y;
};
constexpr Point2D_Aggregate p_agg = {10, 20}; // 编译时创建

// 带有 constexpr 构造函数的类是字面值类型
class Point3D {
public:
    constexpr Point3D(int x_val, int y_val, int z_val)
        : x(x_val), y(y_val), z(z_val) {}

    constexpr int getX() const { return x; }
    // ... 其他 constexpr 成员函数

    // 析构函数是隐式平凡的

private:
    int x; // int 是字面值类型
    int y; // int 是字面值类型
    int z; // int 是字面值类型
};
constexpr Point3D origin(0, 0, 0); // 编译时创建
constexpr int origin_x = origin.getX(); // 编译时调用

// 非字面值类型示例
class NonLiteral {
public:
    NonLiteral() { /* 复杂逻辑或IO */ } // 非 constexpr 构造函数
    ~NonLiteral() { /* 用户定义的非平凡析构函数 */ }
    // 或者包含一个非字面值类型的成员
    // std::string name; // std::string (通常) 不是字面值类型 (C++20 之前)
};

// NonLiteral nl; // 不能是 constexpr NonLiteral nl;
    
```

总结来说，字面值类型是 C++ 类型系统的一个子集，它们足够“简单”和“可预测”，以至于编译器可以在编译阶段就完全理解和处理它们的对象。



