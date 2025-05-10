# 源代码

```c++

#include<cstdint>

struct Large {
    int64_t a = 0x1;
    int64_t b = 0x2;
    int64_t c = 0x3;
    int64_t d = 0x4;
    int64_t e = 0x5;
    int64_t f = 0x6;
    int64_t g = 0x7;
    int64_t h = 0x8;
    int64_t i = 0x9;
    int64_t j = 10;
};


Large func1() {
    Large l;
    l.j = 11;
    return l;
}

Large func2() {
    return Large{};
}

Large func3(int64_t x, int64_t y, int64_t z, Large lg) {
    lg.j = x + y + z;
    return lg;
}

Large func4(int64_t x, int64_t y, int64_t z) {
    Large lg;
    lg.j = x + y + z;
    return lg;
}


void g1() {
    int64_t x = 61;
    int64_t y = 62;
    int64_t z = 63;

    Large lg;

    func1();
    func2();
    func3(x, y, z, lg);
    func4(x, y, z);
}


void g2() {
    int64_t x = 61;
    int64_t y = 62;
    int64_t z = 63;

    Large lg0;

    Large lg1 = func1();
    Large lg2 = func2();
    Large lg3 = func3(x, y, z, lg0);
    Large lg4 = func4(x, y, z);
}

```



# 汇编代码

```c++
func1():
        pushq   %rbp
        movq    %rsp, %rbp
        movq    %rdi, -8(%rbp)
        movq    -8(%rbp), %rax
        movq    $1, (%rax)
        movq    -8(%rbp), %rax
        movq    $2, 8(%rax)
        movq    -8(%rbp), %rax
        movq    $3, 16(%rax)
        movq    -8(%rbp), %rax
        movq    $4, 24(%rax)
        movq    -8(%rbp), %rax
        movq    $5, 32(%rax)
        movq    -8(%rbp), %rax
        movq    $6, 40(%rax)
        movq    -8(%rbp), %rax
        movq    $7, 48(%rax)
        movq    -8(%rbp), %rax
        movq    $8, 56(%rax)
        movq    -8(%rbp), %rax
        movq    $9, 64(%rax)
        movq    -8(%rbp), %rax
        movq    $10, 72(%rax)
        movq    -8(%rbp), %rax
        movq    $11, 72(%rax)
        nop
        movq    -8(%rbp), %rax
        popq    %rbp
        ret
func2():
        pushq   %rbp
        movq    %rsp, %rbp
        movq    %rdi, -8(%rbp)
        movq    -8(%rbp), %rax
        movq    $1, (%rax)
        movq    -8(%rbp), %rax
        movq    $2, 8(%rax)
        movq    -8(%rbp), %rax
        movq    $3, 16(%rax)
        movq    -8(%rbp), %rax
        movq    $4, 24(%rax)
        movq    -8(%rbp), %rax
        movq    $5, 32(%rax)
        movq    -8(%rbp), %rax
        movq    $6, 40(%rax)
        movq    -8(%rbp), %rax
        movq    $7, 48(%rax)
        movq    -8(%rbp), %rax
        movq    $8, 56(%rax)
        movq    -8(%rbp), %rax
        movq    $9, 64(%rax)
        movq    -8(%rbp), %rax
        movq    $10, 72(%rax)
        movq    -8(%rbp), %rax
        popq    %rbp
        ret
func3(long, long, long, Large):
        pushq   %rbp
        movq    %rsp, %rbp
        pushq   %rbx
        movq    %rdi, -16(%rbp)
        movq    %rsi, -24(%rbp)
        movq    %rdx, -32(%rbp)
        movq    %rcx, -40(%rbp)
        movq    -24(%rbp), %rdx
        movq    -32(%rbp), %rax
        addq    %rax, %rdx
        movq    -40(%rbp), %rax
        addq    %rdx, %rax
        movq    %rax, 88(%rbp)
        movq    -16(%rbp), %rax
        movq    16(%rbp), %rcx
        movq    24(%rbp), %rbx
        movq    %rcx, (%rax)
        movq    %rbx, 8(%rax)
        movq    32(%rbp), %rcx
        movq    40(%rbp), %rbx
        movq    %rcx, 16(%rax)
        movq    %rbx, 24(%rax)
        movq    48(%rbp), %rcx
        movq    56(%rbp), %rbx
        movq    %rcx, 32(%rax)
        movq    %rbx, 40(%rax)
        movq    64(%rbp), %rcx
        movq    72(%rbp), %rbx
        movq    %rcx, 48(%rax)
        movq    %rbx, 56(%rax)
        movq    80(%rbp), %rcx
        movq    88(%rbp), %rbx
        movq    %rcx, 64(%rax)
        movq    %rbx, 72(%rax)
        movq    -16(%rbp), %rax
        movq    -8(%rbp), %rbx
        leave
        ret
func4(long, long, long):
        pushq   %rbp
        movq    %rsp, %rbp
        movq    %rdi, -8(%rbp)
        movq    %rsi, -16(%rbp)
        movq    %rdx, -24(%rbp)
        movq    %rcx, -32(%rbp)
        movq    -8(%rbp), %rax
        movq    $1, (%rax)
        movq    -8(%rbp), %rax
        movq    $2, 8(%rax)
        movq    -8(%rbp), %rax
        movq    $3, 16(%rax)
        movq    -8(%rbp), %rax
        movq    $4, 24(%rax)
        movq    -8(%rbp), %rax
        movq    $5, 32(%rax)
        movq    -8(%rbp), %rax
        movq    $6, 40(%rax)
        movq    -8(%rbp), %rax
        movq    $7, 48(%rax)
        movq    -8(%rbp), %rax
        movq    $8, 56(%rax)
        movq    -8(%rbp), %rax
        movq    $9, 64(%rax)
        movq    -8(%rbp), %rax
        movq    $10, 72(%rax)
        movq    -16(%rbp), %rdx
        movq    -24(%rbp), %rax
        addq    %rax, %rdx
        movq    -32(%rbp), %rax
        addq    %rax, %rdx
        movq    -8(%rbp), %rax
        movq    %rdx, 72(%rax)
        nop
        movq    -8(%rbp), %rax
        popq    %rbp
        ret
g1():
        pushq   %rbp
        movq    %rsp, %rbp
        pushq   %rbx
        subq    $440, %rsp
        movq    $61, -24(%rbp)
        movq    $62, -32(%rbp)
        movq    $63, -40(%rbp)
        movq    $1, -448(%rbp)
        movq    $2, -440(%rbp)
        movq    $3, -432(%rbp)
        movq    $4, -424(%rbp)
        movq    $5, -416(%rbp)
        movq    $6, -408(%rbp)
        movq    $7, -400(%rbp)
        movq    $8, -392(%rbp)
        movq    $9, -384(%rbp)
        movq    $10, -376(%rbp)
        leaq    -368(%rbp), %rax
        movq    %rax, %rdi
        call    func1()
        leaq    -288(%rbp), %rax
        movq    %rax, %rdi
        call    func2()
        leaq    -208(%rbp), %rdi
        movq    -40(%rbp), %r8
        movq    -32(%rbp), %rdx
        movq    -24(%rbp), %rsi
        subq    $80, %rsp
        movq    %rsp, %rax
        movq    -448(%rbp), %rcx
        movq    -440(%rbp), %rbx
        movq    %rcx, (%rax)
        movq    %rbx, 8(%rax)
        movq    -432(%rbp), %rcx
        movq    -424(%rbp), %rbx
        movq    %rcx, 16(%rax)
        movq    %rbx, 24(%rax)
        movq    -416(%rbp), %rcx
        movq    -408(%rbp), %rbx
        movq    %rcx, 32(%rax)
        movq    %rbx, 40(%rax)
        movq    -400(%rbp), %rcx
        movq    -392(%rbp), %rbx
        movq    %rcx, 48(%rax)
        movq    %rbx, 56(%rax)
        movq    -384(%rbp), %rcx
        movq    -376(%rbp), %rbx
        movq    %rcx, 64(%rax)
        movq    %rbx, 72(%rax)
        movq    %r8, %rcx
        call    func3(long, long, long, Large)
        addq    $80, %rsp
        leaq    -128(%rbp), %rax
        movq    -40(%rbp), %rcx
        movq    -32(%rbp), %rdx
        movq    -24(%rbp), %rsi
        movq    %rax, %rdi
        call    func4(long, long, long)
        nop
        movq    -8(%rbp), %rbx
        leave
        ret
g2():
        pushq   %rbp
        movq    %rsp, %rbp
        pushq   %rbx
        subq    $440, %rsp
        movq    $61, -24(%rbp)
        movq    $62, -32(%rbp)
        movq    $63, -40(%rbp)
        movq    $1, -128(%rbp)
        movq    $2, -120(%rbp)
        movq    $3, -112(%rbp)
        movq    $4, -104(%rbp)
        movq    $5, -96(%rbp)
        movq    $6, -88(%rbp)
        movq    $7, -80(%rbp)
        movq    $8, -72(%rbp)
        movq    $9, -64(%rbp)
        movq    $10, -56(%rbp)
        leaq    -208(%rbp), %rax
        movq    %rax, %rdi
        call    func1()
        leaq    -288(%rbp), %rax
        movq    %rax, %rdi
        call    func2()
        leaq    -368(%rbp), %rdi
        movq    -40(%rbp), %r8
        movq    -32(%rbp), %rdx
        movq    -24(%rbp), %rsi
        subq    $80, %rsp
        movq    %rsp, %rax
        movq    -128(%rbp), %rcx
        movq    -120(%rbp), %rbx
        movq    %rcx, (%rax)
        movq    %rbx, 8(%rax)
        movq    -112(%rbp), %rcx
        movq    -104(%rbp), %rbx
        movq    %rcx, 16(%rax)
        movq    %rbx, 24(%rax)
        movq    -96(%rbp), %rcx
        movq    -88(%rbp), %rbx
        movq    %rcx, 32(%rax)
        movq    %rbx, 40(%rax)
        movq    -80(%rbp), %rcx
        movq    -72(%rbp), %rbx
        movq    %rcx, 48(%rax)
        movq    %rbx, 56(%rax)
        movq    -64(%rbp), %rcx
        movq    -56(%rbp), %rbx
        movq    %rcx, 64(%rax)
        movq    %rbx, 72(%rax)
        movq    %r8, %rcx
        call    func3(long, long, long, Large)
        addq    $80, %rsp
        leaq    -448(%rbp), %rax
        movq    -40(%rbp), %rcx
        movq    -32(%rbp), %rdx
        movq    -24(%rbp), %rsi
        movq    %rax, %rdi
        call    func4(long, long, long)
        nop
        movq    -8(%rbp), %rbx
        leave
        ret
```

