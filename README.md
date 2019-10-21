# 软件分析测试作业

程序分析工具：LLVM

构造的程序：homework.c

转换后的代码：homework.ll

## 构造的程序

在代码注释中解释当前状况。

```c
int main(){
    int a, b, *c, *d;  //a,b为int; c,d为int的一级指针
    int* W = &a;  //W为int的一级指针; 赋值：W指向a
    int* X = &b;  //X为int的一级指针; 赋值：X指向b
    int** y = &c;  //y为int的二级指针; 赋值：y指向c
    int** z = y;  //z为int的二级指针; 赋值：z和y指向的位置一样，即z指向c
    c = 0;  //赋值：将c置为0，即c现在是一个空指针
    *y = W;  //赋值：y指向的指针现在等于W，即c等于W，即c指向a 
    *z = X;  //赋值：z指向的指针现在等于X，即c等于X，即c指向b 
    y = &d;  //赋值：y指向d
    z = y;  //赋值：z和y指向的位置一样，即z指向d
    *y = W;  //赋值：y指向的指针现在等于W，即d等于W，即指向a 
    *z = X;  //赋值：z指向的指针现在等于X，即d等于X，即指向b
    return 0;
}
```

可以看到在赋值的过程中，y、z、c、d是有所改变的。

# 转换后的代码

以下是homework.ll的部分代码，IR中全局变量用@，局部变量用%。在代码行中的‘//’后解释当前状况。

```c
define i32 @main() #0 {
  %1 = alloca i32, align 4  // 内存分配一个32位int的%1
  %2 = alloca i32, align 4  // 内存分配一个32位int的%2（a1）  
  %3 = alloca i32, align 4  // 内存分配一个32位int的%3（给b1）
  %4 = alloca i32*, align 8  // 内存分配一个32位int一级指针的%4（c1）
  %5 = alloca i32*, align 8  // 内存分配一个32位int一级指针的%5（d1）
  %6 = alloca i32*, align 8  // 内存分配一个32位int一级指针的%6（W1）
  %7 = alloca i32*, align 8  // 内存分配一个32位int一级指针的%7（X1）
  %8 = alloca i32**, align 8  // 内存分配一个32位int二级指针的%8（y1）
  %9 = alloca i32**, align 8  // 内存分配一个32位int一级指针的%9（z1）
  store i32 0, i32* %1, align 4  //赋值：%1 = 0
  store i32* %2, i32** %6, align 8  //赋值：把%2的地址给%6    W1 = &a1
  store i32* %3, i32** %7, align 8  //赋值：%3的地址给%7     X1 = &b1
  store i32** %4, i32*** %8, align 8  //赋值：%4的地址给%8    y1 = &c1
  %10 = load i32**, i32*** %8, align 8  
  store i32** %10, i32*** %9, align 8  //赋值：使%9与%8相等    z1 = y1
  store i32* null, i32** %4, align 8  //赋值：将c1置为0       c1 = 0
  %11 = load i32*, i32** %6, align 8  
  %12 = load i32**, i32*** %8, align 8
  store i32* %11, i32** %12, align 8 //赋值：将%8(y1)拷贝给%12(y2) %12指向的与W1相同 *y1 = W1      
  %13 = load i32*, i32** %7, align 8
  %14 = load i32**, i32*** %9, align 8
  store i32* %13, i32** %14, align 8 //赋值：将%9(z1)拷贝给%14(z2) %14指向的与X1相同 *z1 = X1  
  store i32** %5, i32*** %8, align 8 //赋值：把%5的地址给%8  y2 = &d1
  %15 = load i32**, i32*** %8, align 8
  store i32** %15, i32*** %9, align 8 //赋值：把%15的地址给%9  z2 = &d1
  %16 = load i32*, i32** %6, align 8  
  %17 = load i32**, i32*** %8, align 8
  store i32* %16, i32** %17, align 8  //赋值：%16和%6指向的相同  *y2 = W1
  %18 = load i32*, i32** %7, align 8
  %19 = load i32**, i32*** %9, align 8
  store i32* %18, i32** %19, align 8  //赋值：%18和%7指向的相同  *z2 = X1
  ret i32 0
}
```

在这个过程中，我们仅发现了y、z的改变，从y1(z1)到y2(z2)，但其实c1和d1，他们表面上只被赋了一次值或者没有被赋值，但是其实它们的指向是一直在变的（c先存的为0，后存的是a的位置，再后来存的是b的位置；d先存的是a的位置，再后来存的是b的位置），它们并没有做到内存位置一旦赋值都不会发生改变。 hhh
