# 1.ISO标准

When certain criteria are met, an implementation is allowed to omit the copy/move construction of a class object, even if the constructor selected for the copy/move operation and/or the destructor for the object have side effects. In such cases, the implementation treats the source and target of the omitted copy/move operation as simply two different ways of referring to the same object. If the first parameter of the selected constructor is an rvalue reference to the object’s type, the destruction of that object occurs when the target would have been destroyed; otherwise, the destruction occurs at the later of the times when the two objects would have been destroyed without the optimization.105 This elision of copy/move operations, called copy elision, is permitted in the following circumstances (which may be combined to eliminate multiple copies):

- in a return statement in a function with a class return type, when the expression is the name of a non-volatile object with automatic storage duration (other than a function parameter or a variable introduced by the exception-declaration of a handler (14.4)) with the same type (ignoring cv-qualification) as the function return type, the copy/move operation can be omitted by constructing the object directly into the function call’s return object
- in a throw-expression (7.6.18), when the operand is the name of a non-volatile object with automatic storage duration (other than a function or catch-clause parameter) that belongs to a scope that does not contain the innermost enclosing compound-statement associated with a try-block (if there is one), the copy/move operation can be omitted by constructing the object directly into the exception object
- in a coroutine (9.5.4), a copy of a coroutine parameter can be omitted and references to that copy replaced with references to the corresponding parameter if the meaning of the program will be unchanged except for the execution of a constructor and destructor for the parameter copy object
- when the exception-declaration of an exception handler (14.1) declares an object of the same type (except for cv-qualification) as the exception object (14.2), the copy operation can be omitted by treating the exception-declaration as an alias for the exception object if the meaning of the program will be unchanged except for the execution of constructors and destructors for the object declared by the exception-declaration.

note 1 The exception object is always an lvalue, thus it is not implicitly moved from.

Copy elision is not permitted where an expression is evaluated in a context requiring a constant expression (7.7) and in constant initialization (6.9.3.2). 

note 2 It is possible that copy elision is performed if the same expression is evaluated in another context.

```c++
class Thing {
public:
    Thing();
    ~Thing();
    Thing(const Thing&);
};

Thing f() {
    Thing t;
    return t;
}

Thing t2 = f();

struct A {
    void *p;
    constexpr A(): p(this) {}
};

constexpr A g() {
    A loc;
    return loc;
}

constexpr A a; // well-formed, a.p points to a
constexpr A b = g(); // error: b.p would be dangling (7.7)

void h() {
    A c = g(); // well-formed, c.p can point to c or be dangling
}

```

Here the criteria for elision can eliminate the copying of the object t with automatic storage duration into the result object for the function call f(), which is the non-local object t2. Effectively, the construction of t can be viewed as directly initializing t2, and that object’s destruction will occur at program exit. Adding a move constructor to Thing has the same effect, but it is the move construction from the object with automatic storage duration to t2 that is elided.



```c++
class Thing {
public:
    Thing();
    ~Thing();
    Thing(Thing&&);
private:
    Thing(const Thing&);
};
Thing f(bool b) {
    Thing t;
    if (b)
    	throw t; // OK, Thing(Thing&&) used (or elided) to throw t
    return t; // OK, Thing(Thing&&) used (or elided) to return t
}
Thing t2 = f(false); // OK, no extra copy/move performed, t2 constructed by call to f
struct Weird {
    Weird();
    Weird(Weird&);
};
Weird g(bool b) {
    static Weird w1;
    Weird w2;
    if (b)
    	return w1; // OK, uses Weird(Weird&)
    else
    	return w2; // error: w2 in this context is an xvalue
}
int& h(bool b, int i) {
    static int s;
    if (b)
    	return s; // OK
    else
    	return i; // error: i is an xvalue
}
decltype(auto) h2(Thing t) {
	return t; // OK, t is an xvalue and h2’s return type is Thing
}
decltype(auto) h3(Thing t) {
	return (t); // OK, (t) is an xvalue and h3’s return type is Thing&&
}
```

```c++
template<class T> void g(const T&);
template<class T> void f() {
    T x;
    try {
        T y;
        try { g(x); }
        catch (...) {
            if (/∗...∗/)
            	throw x; // does not move
            throw y; // moves
        }
    	g(y);
    } catch(...) {
        g(x);
        g(y); // error: y is not in scope
    }
}
```





# 2.标准解读

这段英文描述的是C++中的一个重要优化技术——**拷贝省略 (Copy Elision)**。它的核心思想是，在特定情况下，编译器可以省略掉对象的拷贝或移动构造函数调用，即使这些构造函数或析构函数有副作用。这样做可以提高程序性能，避免不必要的对象创建和销毁。

让我们逐条解析：

1.  **核心概念**：
    *   “When certain criteria are met, an implementation is allowed to omit the copy/move construction of a class object, even if the constructor selected for the copy/move operation and/or the destructor for the object have side effects.”
        *   **中文理解**：当满足特定条件时，编译器（实现）被允许省略类对象的拷贝/移动构造。即使被选中的拷贝/移动构造函数和/或该对象的析构函数具有副作用（比如打印日志、修改全局变量等），这个省略也可能发生。
    *   “In such cases, the implementation treats the source and target of the omitted copy/move operation as simply two different ways of referring to the same object.”
        *   **中文理解**：在这种情况下，编译器将原本要进行拷贝/移动操作的源对象和目标对象视为引用同一个对象的两种不同方式。也就是说，它们实际上是同一个对象，只是在代码的不同位置用不同的名字表示。
    *   “If the first parameter of the selected constructor is an rvalue reference to the object’s type, the destruction of that object occurs when the target would have been destroyed; otherwise, the destruction occurs at the later of the times when the two objects would have been destroyed without the optimization.”
        *   **中文理解**：这部分描述了经过拷贝省略后，那个“单一”对象的析构时机：
            *   如果被省略的是一个**移动构造函数**（其第一个参数是对象类型的右值引用），那么这个（概念上融合后的）对象将在“目标对象”原本应该被析构的时刻析构。
            *   否则（如果被省略的是一个**拷贝构造函数**），这个对象将在“源对象”和“目标对象”两者中，较晚的那个析构时刻进行析构。（注：C++17 之后，对于prvalue的返回，规则更倾向于目标对象的生命周期，因为源对象根本没有被物化）。

2.  **拷贝省略允许的场景 (Copy elision is permitted in the following circumstances)**：

    *   **场景一：函数返回值 (NRVO - Named Return Value Optimization 和 RVO - Return Value Optimization)**
        *   “in a return statement in a function with a class return type, when the expression is the name of a non-volatile object with automatic storage duration (other than a function parameter or a variable introduced by the exception-declaration of a handler) with the same type (ignoring cv-qualification) as the function return type, the copy/move operation can be omitted by constructing the object directly into the function call’s return object”
        *   **中文理解**：当函数返回一个类类型对象时，如果 `return` 语句的表达式是一个**非易失的、具有自动存储期**的局部变量名（**不是函数参数，也不是catch子句声明的异常变量**），并且这个局部变量的类型与函数返回类型相同（忽略cv限定符，即const/volatile），那么编译器可以将这个局部变量直接构造在调用者提供的用于接收返回值的内存位置上，从而省略从局部变量到返回临时对象（如果需要的话），再到最终目标对象的拷贝/移动。
            *   **例子**：`Thing f() { Thing t; return t; } Thing t2 = f();` 这里 `t` 就是这样一个局部变量，它可以被直接构造在 `t2` 的位置（或者用于构造 `t2` 的临时对象的位置）。

    *   **场景二：抛出异常 (Throw-expression)**
        *   “in a throw-expression (7.6.18), when the operand is the name of a non-volatile object with automatic storage duration (other than a function or catch-clause parameter) that belongs to a scope that does not contain the innermost enclosing compound-statement associated with a try-block (if there is one), the copy/move operation can be omitted by constructing the object directly into the exception object”
        *   **中文理解**：当 `throw` 一个**非易失的、具有自动存储期**的局部变量名（**不是函数参数，也不是catch子句参数**）时，如果该变量的作用域**不包含**（即它位于内部或平级于）最内层 `try` 块的复合语句，那么可以省略拷贝/移动，将该对象直接构造到异常对象中。
            *   **例子** (来自提供的代码)：
                ```c++
                template<class T> void f() {
                    T x; // x 的作用域包含 try 块
                    try {
                        T y; // y 的作用域不包含（即在 try 块内部）try 块（的复合语句）
                        try { g(x); }
                        catch (...) {
                            if (/*...*/)
                                throw x; // x 从外部作用域来，其作用域包含 try 块，不能应用此特定规则省略，会拷贝
                            throw y; // y 在 try 块内部，可以应用此规则省略（或移动）
                        }
                        g(y);
                    } catch(...) { /*...*/ }
                }
                ```
                根据更通行的 cppreference 解释和实际行为：
                - `throw y;`: `y` 的作用域在 `try` 块内部，**不扩展到 `try` 块之外**。这种情况下，可以将 `y` 直接构造到异常对象中（或通过移动构造）。注释说 `// moves`，意味着如果未完全省略，则发生移动。
                - `throw x;`: `x` 的作用域在 `try` 块外部，**扩展到 `try` 块之外**。这种情况下，不能应用上述的特定省略规则，`x` 会被**拷贝**到异常对象中。注释说 `// does not move`，意味着会拷贝。

    *   **场景三：协程参数 (Coroutine parameter)**
        *   “in a coroutine (9.5.4), a copy of a coroutine parameter can be omitted and references to that copy replaced with references to the corresponding parameter if the meaning of the program will be unchanged except for the execution of a constructor and destructor for the parameter copy object”
        *   **中文理解**：在协程中，协程参数的副本可以被省略，对副本的引用会被替换为对相应参数的引用，前提是除了参数副本对象的构造和析构函数的执行外，程序的含义不会改变。

    *   **场景四：异常处理器声明 (Exception handler declaration)**
        *   “when the exception-declaration of an exception handler (14.1) declares an object of the same type (except for cv-qualification) as the exception object (14.2), the copy operation can be omitted by treating the exception-declaration as an alias for the exception object if the meaning of the program will be unchanged except for the execution of constructors and destructors for the object declared by the exception-declaration.”
        *   **中文理解**：在 `catch` 子句中声明一个与实际抛出的异常对象类型相同的对象时（忽略cv限定符），可以省略从实际异常对象到 `catch` 子句声明的对象的拷贝，将 `catch` 子句声明的对象视为实际异常对象的别名。前提条件同上，即除了构造和析构外，程序含义不变。
            *   **例子**：`catch (MyException e)`，这里的 `e` 可以直接是抛出的 `MyException` 对象的别名，而不是一个副本。
            *   **Note 1**: “The exception object is always an lvalue, thus it is not implicitly moved from.” 异常对象总是一个左值，因此不能从它隐式移动。

3.  **拷贝省略的限制**：

    *   “Copy elision is not permitted where an expression is evaluated in a context requiring a constant expression (7.7) and in constant initialization (6.9.3.2).”
        *   **中文理解**：在需要常量表达式的上下文中（如 `constexpr` 函数体、模板参数等）以及常量初始化时，不允许拷贝省略。这是因为在这些上下文中，对象的精确身份（例如地址）可能很重要，而拷贝省略会改变这一点。
        *   **例子**：
            ```c++
            struct A {
                void *p;
                constexpr A(): p(this) {} // p 指向当前对象
            };
            constexpr A g() {
                A loc;      // loc.p 指向 loc
                return loc; // 如果发生拷贝省略，loc 直接在返回值位置构造
            }
            constexpr A a;       // OK, a.p 指向 a
            constexpr A b = g(); // ERROR: 如果 loc 被拷贝到 g() 的返回值（一个临时对象），
                                 // 然后这个临时对象被拷贝到 b，那么 b.p 会是临时对象的地址，
                                 // 而临时对象随后销毁，导致 b.p 悬空。
                                 // 即使有拷贝省略，在 constexpr 上下文中，这种潜在的歧义或依赖于
                                 // 优化的行为是不允许的。必须保证在 constexpr 求值期间指针的有效性。
                                 // C++17 之后，对于 g() 返回 prvalue 的情况，是强制省略的，
                                 // loc 直接在 b 的位置构造，b.p 指向 b，这本应是 OK 的。
                                 // 标准原文的错误可能是基于旧规则或特定场景的解释，即如果省略
                                 // *没有* 发生，会导致问题，因此在 constexpr 中禁止这种 *可能* 依赖优化的行为。
                                 // 现代编译器对于这种 constexpr 返回局部变量的情况，会执行强制省略，
                                 // loc.p 指向 loc，然后 loc 被“视作” b，所以 b.p 指向 b，这是合法的。
                                 // 7.7 (Constant expressions) 中有关于指针值的规则，要求在常量表达式中，
                                 // 指针必须指向具有静态存储期或线程存储期的对象，或者指向其完整对象就是
                                 // 包含该指针成员的对象本身，或者指向非局部对象的子对象。
                                 // 错误的原因是，如果 g() 返回的 loc 被认为是一个临时对象，然后 b 从这个临时对象初始化，
                                 // b.p 会取临时对象的地址，这是不允许的。
            void h() {
                A c = g(); // 非 constexpr 上下文，允许拷贝省略。c.p 可以指向 c。
            }
            ```

    *   **Note 2**: “It is possible that copy elision is performed if the same expression is evaluated in another context.”
        *   **中文理解**：即使在常量表达式中不允许拷贝省略，同样的代码在其他上下文（如普通运行时代码）中，拷贝省略仍然可能发生。

4.  **其他例子解析**：

    *   **`Thing f(bool b)` 和 `Thing t2 = f(false);`**
        *   `throw t;`: `t` 是局部变量，抛出时可以移动构造（如果 `Thing` 有移动构造函数）或省略拷贝/移动到异常对象。
        *   `return t;`: `t` 是局部变量，返回时可以移动构造（如果 `Thing` 有移动构造函数）或省略拷贝/移动到返回值。
        *   `Thing t2 = f(false);`: 调用 `f(false)` 时，`f` 内部的 `t` 的构造可以直接视为 `t2` 的构造，没有额外的拷贝/移动。

    *   **`Weird g(bool b)`**
        *   `static Weird w1; Weird w2;`
        *   `return w1;`: `w1` 是静态对象（左值）。返回它时会调用拷贝构造函数 `Weird(Weird&)` (因为 `w1` 是左值，且 `Weird` 没有移动构造函数)。
        *   `return w2;`: `w2` 是局部自动变量。在 `return` 语句中，它被视为一个将亡值 (xvalue)，因为它符合 NRVO 的条件。但 `Weird` 只有 `Weird(Weird&)` 这个拷贝构造函数，它接受一个左值引用。将亡值不能绑定到非 const 左值引用。因此这里是**错误**的。如果 `Weird` 有 `Weird(const Weird&)` 或 `Weird(Weird&&)` 就会 OK。
            *   注意：文本注释 “error: w2 in this context is an xvalue” 指的是 `w2` 在这里作为返回值表达式时，被认为是 xvalue，而 `Weird(Weird&)` (非 const 左值引用参数) 无法绑定到 xvalue。

    *   **`int& h(bool b, int i)`**
        *   `return s;`: `s` 是静态变量，返回其引用是安全的。
        *   `return i;`: `i` 是函数参数，按值传递，它是一个局部于函数 `h` 的副本。返回对 `i` 的引用会导致**悬垂引用**，因为 `i` 在函数返回后就销毁了。注释 “error: i is an xvalue” 在这里可能有些误导，主要问题是返回局部变量的引用。实际上，根据规则，函数参数 `i` 在 `return i;` 语句中作为表达式时是左值。错误在于生命周期。

    *   **`decltype(auto) h2(Thing t)` 和 `decltype(auto) h3(Thing t)`**
        *   `decltype(auto) h2(Thing t) { return t; }`
            *   `t` 是函数参数（左值）。但在 `return t;` 语句中，如果 `t` 是一个具有自动存储期的对象（如参数或局部变量），且符合被移动的条件，它会被当作右值（具体是 xvalue）。`decltype(auto)` 推断 `return t;` 表达式的类型。由于 `t` 在这里被视为右值来初始化返回值，`h2` 的返回类型被推断为 `Thing` (按值返回)。
        *   `decltype(auto) h3(Thing t) { return (t); }`
            *   `(t)` 的值类别：如果 `t` 是左值，`(t)` 仍然是左值。然而，C++标准对 `return` 语句有一个特殊规则：如果 `return` 的操作数是（可能带括号的）一个标识符表达式，它命名了一个在最内层函数或lambda表达式主体中或作为其参数声明的自动存储期对象，那么该对象被视为右值。因此，`(t)` 在这里也被视为右值 (xvalue)。所以 `decltype(auto)` 会将 `h3` 的返回类型推断为 `Thing&&` (右值引用)。
            *   **重要**：这两个例子说明 `decltype(auto)` 结合 `return` 语句时，返回表达式的值类别（lvalue, xvalue, prvalue）直接影响推断出的返回类型 (`T&`, `T&&`, `T`)。

5.  **关于C++17的强制拷贝省略**：
    值得注意的是，C++17 引入了**强制拷贝省略 (Guaranteed Copy Elision)**。对于从返回纯右值 (prvalue) 的函数初始化对象（例如 `Thing t2 = f();` 如果 `f` 返回的是一个临时对象，而不是一个命名的局部变量），以及其他一些特定情况，拷贝/移动构造**必须**被省略。这意味着源和目标从一开始就是同一个对象，不再是“允许”省略，而是“必须”省略。文本中描述的主要是“允许的”拷贝省略，这仍然适用于 NRVO (Named Return Value Optimization，即返回命名局部变量) 等情况。

总结来说，拷贝省略是C++编译器的一项强大优化，它通过在逻辑上将多个对象实例合并为一个，避免了不必要的构造、析构和数据复制，从而提升性能。理解其规则和限制对于编写高效的C++代码非常有帮助。





# 3.Copy elision和RVO/NRVO

Copy Elision（拷贝省略）、RVO（Return Value Optimization，返回值优化）和 NRVO（Named Return Value Optimization，具名返回值优化）之间的关系可以这样理解：

1.  **Copy Elision (拷贝省略)**：
    *   这是一个**总称/概括性术语**。
    *   它指的是编译器在特定情况下，**省略掉类对象的拷贝构造函数或移动构造函数的调用**，即使这些构造函数或析构函数有副作用。
    *   目标是提高性能，避免不必要的对象创建和销毁。
    *   拷贝省略可以发生在多种情况下，不仅仅是函数返回值。例如，`throw` 表达式、异常处理器的声明等（如你提供的文本中所列）。
    *   从C++17开始，某些类型的拷贝省略是**强制性的 (mandatory)**，而另一些仍然是**允许的 (permitted)**（即编译器可以选择是否执行）。

2.  **RVO (Return Value Optimization - 返回值优化)**：
    *   这是拷贝省略的一种**特定形式**，专门针对**函数返回一个未命名的临时对象 (prvalue, 纯右值)** 的情况。
    *   **场景**：当一个函数返回一个通过构造函数直接创建的临时对象时，例如 `return MyClass();` 或 `return MyClass(arg1, arg2);`。
    *   **行为**：编译器可以直接在调用者为返回值预留的内存空间上构造这个临时对象，从而完全避免任何拷贝或移动。根本就不会有“源”临时对象和“目标”返回对象的区分，它们从一开始就是同一个对象。
    *   **C++17及以后**：对于返回纯右值的情况，这种优化是**强制性的**。标准规定，在这种情况下，不会有临时对象的拷贝或移动，源纯右值表达式直接初始化结果对象。

3.  **NRVO (Named Return Value Optimization - 具名返回值优化)**：
    *   这也是拷贝省略的一种**特定形式**，同样应用于函数返回值，但针对的是**函数返回一个具名的局部自动存储期对象**的情况。
    *   **场景**：当一个函数返回一个在函数内部声明的局部变量时，例如 `MyClass func() { MyClass obj; /* ... */; return obj; }`。
    *   **行为**：编译器可以将这个具名的局部对象 `obj` 直接构造在调用者为返回值预留的内存空间上。这样，当 `return obj;` 执行时，就不需要从 `obj` 拷贝或移动到返回值位置。
    *   **C++17及以后**：NRVO 仍然是**允许的 (permitted)**，而不是强制性的。这意味着编译器可以选择执行此优化，但不是必须的。然而，大多数现代编译器在满足条件时都会执行NRVO。
    *   **条件**：要应用NRVO，返回的命名对象通常需要满足一些条件：
        *   它是一个非 `volatile` 的自动存储期对象。
        *   它不是函数参数或 `catch` 子句声明的变量。
        *   它的类型（忽略cv限定符）与函数返回类型相同。
        *   （还有一些更细致的规则，比如函数内所有返回该对象的 `return` 语句都返回同一个对象，没有复杂的控制流导致返回不同的命名对象等）。

**总结关系：**

*   **Copy Elision 是最广泛的概念。**
*   **RVO 和 NRVO 都是 Copy Elision 在函数返回值优化方面的具体应用。**
    *   你可以把 RVO 和 NRVO 看作是实现函数返回场景下拷贝省略的两种不同但相关的技术或策略。
*   **RVO 处理的是返回未命名临时对象 (prvalues) 的情况，在C++17后通常是强制省略。**
*   **NRVO 处理的是返回命名局部对象的情况，通常是允许省略的优化。**

**一个简单的层级关系可以表示为：**

```
Copy Elision (总概念)
    |
    +-- Function Return Value Optimization (特定应用领域)
    |   |
    |   +-- RVO (针对返回 prvalue)  <-- C++17后通常强制
    |   |
    |   +-- NRVO (针对返回命名局部对象) <-- 允许
    |
    +-- Other Elision Scenarios (e.g., throw, catch) <-- 允许
```

理解这个关系有助于你更好地分析C++代码中对象的构造和析构行为，以及编译器可能应用的性能优化。