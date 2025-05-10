# Type Conversion[Todo]

## 1.Fundamental types

- bool
- char
- integer types: short/int/long long
- floating point types: float/double
- void
- std::nullptr_t(值为nullptr)

## 2.Compound types

- arrays
- functions
- pointers
- reference
  - lvalue reference
  - rvalue reference
- classes
- unions
- enumerations
- pointers to non-static class members



## 3.隐式类型转换

### 3.1 integrals <-> floating point

> `cvtsi2sd`：integrals -> floating point
>
> `cvttsd2si`：floating point -> integrals

```c++
int test() {
    int ival = 0x9;
    double dval = 3.14;

    int result = ival + dval;

}

test():
        pushq   %rbp
        movq    %rsp, %rbp
        movl    $9, -4(%rbp)
        movsd   .LC0(%rip), %xmm0
        movsd   %xmm0, -16(%rbp)
        pxor    %xmm0, %xmm0
        cvtsi2sd        -4(%rbp), %xmm0
        addsd   -16(%rbp), %xmm0
        cvttsd2si       %xmm0, %eax
        movl    %eax, -20(%rbp)
        nop
        popq    %rbp
        ret
.LC0:
        .long   1374389535
        .long   1074339512
```



### 3.2 bool/char -> int

> 运算最低需要用到32位寄存器

```c++
int test1() {
    bool bval = true;
    char ch = 'A';

    char result = bval + ch;
    int ival = bval + ch;
}

test1():
        pushq   %rbp
        movq    %rsp, %rbp
        movb    $1, -1(%rbp)
        movb    $65, -2(%rbp)
        movzbl  -1(%rbp), %edx
        movzbl  -2(%rbp), %eax
        addl    %edx, %eax
        movb    %al, -3(%rbp)
        movzbl  -1(%rbp), %edx
        movsbl  -2(%rbp), %eax
        addl    %edx, %eax
        movl    %eax, -8(%rbp)
        nop
        popq    %rbp
        ret
```



### 3.3 integrals/floating-point/pointer -> bool

> `cmp $0` :用于integrals和pointer
>
> `ucomisd` : 用于floating-point

```c++
int test2() {
    int ival = 0x9;
    double dval = 3.14;
    int* p = nullptr;

    int result = 0x6;

    if(ival) {
        if(dval) {
            if(p) {
                result = 0x7;
            }
        }
    }
}

test2():
        pushq   %rbp
        movq    %rsp, %rbp
        movl    $9, -4(%rbp)			;变量ival
        movsd   .LC0(%rip), %xmm0
        movsd   %xmm0, -16(%rbp)		;变量dval
        movq    $0, -24(%rbp)			;变量p		
        movl    $6, -28(%rbp)			;变量result
        cmpl    $0, -4(%rbp)			;if(ival)
        je      .L2
        pxor    %xmm0, %xmm0
        ucomisd -16(%rbp), %xmm0		;if(dval)
        jp      .L4
        pxor    %xmm0, %xmm0
        ucomisd -16(%rbp), %xmm0
        je      .L2
.L4:
        cmpq    $0, -24(%rbp)			;if(p)
        je      .L2
        movl    $7, -28(%rbp)
.L2:
        nop
        popq    %rbp
        ret
.LC0:
        .long   1374389535
        .long   1074339512
```





### 3.4 unsigned <-> signed

> 底层汇编指令未做任何转换

```c++
int test3() {
    int ival = -100;
    unsigned int uival = 100u;

    int result = ival + uival;
}

test3():
        pushq   %rbp
        movq    %rsp, %rbp
        movl    $-100, -4(%rbp)
        movl    $100, -8(%rbp)
        movl    -4(%rbp), %edx
        movl    -8(%rbp), %eax
        addl    %edx, %eax
        movl    %eax, -12(%rbp)
        nop
        popq    %rbp
        ret
```



### 3.5 array -> pointer

- arr -> &arr[0]
- 以下情形例外
  - `decltype`(declaraton type)
  - `&`(取地址)
  - `sizeof`(返回值是constant expression)
  - `typeid`
  - `Example: int (&ref) [10] = arr`;



### 3.6 其他

1. 0/nullptr -> 其他类型指针

2. 其他non const指针 -> void*

3. 其他类型指针 -> const void*

4. non const -> const

   1. ```c++
      int a = 10;
      
      const int *p = &a;
      const int &ref = a;
      ```

5. const char* -> std::string

6. std::cin/std::cout -> bool





### 3.7 初始化时的类型转换

```c++
class Test {
private:
    int val;

public:
    Test(int x): val(x) {

    }
};


int main() {

    Test t = 0x6;
    //等价于
    //Test t = Test{0x6};
}
```



> 编译选项：-O0 -fno-elide-constructors -std=c++11

```c++
Test::Test(int) [base object constructor]:
        pushq   %rbp
        movq    %rsp, %rbp
        movq    %rdi, -8(%rbp)
        movl    %esi, -12(%rbp)
        movq    -8(%rbp), %rax
        movl    -12(%rbp), %edx
        movl    %edx, (%rax)
        nop
        popq    %rbp
        ret
Test::Test(Test&&) [base object constructor]:
        pushq   %rbp
        movq    %rsp, %rbp
        movq    %rdi, -8(%rbp)
        movq    %rsi, -16(%rbp)
        movq    -8(%rbp), %rax
        movq    -16(%rbp), %rdx
        movl    (%rdx), %edx
        movl    %edx, (%rax)
        nop
        popq    %rbp
        ret
main:
        pushq   %rbp
        movq    %rsp, %rbp
        subq    $16, %rsp
        leaq    -4(%rbp), %rax
        movl    $6, %esi
        movq    %rax, %rdi
        call    Test::Test(int) [complete object constructor]
        leaq    -4(%rbp), %rdx
        leaq    -8(%rbp), %rax
        movq    %rdx, %rsi
        movq    %rax, %rdi
        call    Test::Test(Test&&) [complete object constructor]
        movl    $0, %eax
        leave
        ret
```





## 4.C++11 Explicit Conversions

### 4.1 const_cast

> A *const_cast operator* adds or removes a `const` or `volatile` modifier to or from a type.

The result of the expression const_cast(v) is of type T. 

- If T is an lvalue reference to object type, the result is an lvalue; 
- if T is an rvalue reference to object type, the result is an xvalue; 
- otherwise, the result is a prvalue and the lvalue-to-rvalue (4.1), array-to-pointer (4.2), and function-to-pointer (4.3) standard conversions are performed on the expression v. 

```c++
int ival = 0x3;
const int *p = ival;
const int& ref = ival;

const_cast<int*>(p);

const_cast<int&>(ref) = 0x9; //lvalue
```









### 4.2 static_cast

The result of the expression static_cast(v) is the result of converting the expression v to type T. 

- If T is an lvalue reference type or an rvalue reference to function type, the result is an lvalue; 
- if T is an rvalue reference to object type, the result is an xvalue; 
- otherwise, the result is a prvalue. The static_cast operator shall not cast away constness (5.2.11).



### 4.3 dynamic_cast

The result of the expression dynamic_cast(v) is the result of converting the expression v to type T. 

- T shall be a pointer or reference to a complete class type, or “pointer to cv void.” 
- The dynamic_cast operator shall not cast away constness (5.2.11).



### 4.4 reinterpret_cast

The result of the expression reinterpret_cast(v) is the result of converting the expression v to type T. 

- If T is an lvalue reference type or an rvalue reference to function type, the result is an lvalue; 
- if T is an rvalue reference to object type, the result is an xvalue; 
- otherwise, the result is a prvalue and the lvalue-torvalue (4.1), array-to-pointer (4.2), and function-to-pointer (4.3) standard conversions are performed on the expression v.