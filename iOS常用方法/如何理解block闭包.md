> 一句话总结block : 带有局部变量的匿名函数

#### 闭包在其它编程语言的名称
| 编程语言 | Block名称 |
|:-------:|:-------:|
| C+Block| Block |
| Smalltalk| Block|
| Ruby| Block|
| LISP| Lambda|
| Python| Lambda|
| C++ 11| Lambda|
| Javascript| Anonymous function|
 

#### iOS闭包的声明与定义
博主iOS开发，下面从iOS角度入手讲解如何理解block闭包：
在OC中声明一个block的形式如下：
````
返回值类型  (^block名)  (形参列表) 
````
在OC中定义一个block的形式如下：
````
^ 返回值类型  (形参列表)  {表达式}
````
举一个赋值的例子：
````
int (^blk) (int a , int b) = ^ int (int a , int b) { return a + b;};
````
这是规范的写法，有时我们也可以使用简略后的写法：
````
^ 返回值类型  (形参列表)  {表达式}
  (省略)     (如空，省略）
````
上面的例子变成：
````
int (^blk) (int a , int b) = ^(int a , int b) {return a + b;};
````
但是，每次声明block时都需要书写这么长的表达式显得不那么方便，我们像使用函数指针类型时那样，使用typedef来解决该问题：
````
typedef int (^blk_t) (int a , int b);
````
这样在声明一个block变量时，就变为：
````
blk_t blk;
blk = ^(int a , int b ) {return a + b;}
````
#### 从C的角度看block闭包
##### block的实质
clang(LLVM编译器)的"-rewrite-objc"选项可将Objective-C的源代码重写成C++的源代码，我们可以将带有block的OC代码转成C++代码，看一看苹果的实现。
````
clang -rewrite-objc 源代码文件名
````
写一段带有block的代码：
````
typedef int (^blk_t) (int a , int b);

int main(int argc, const char * argv[]) {
    @autoreleasepool {
  
        blk_t blk = ^ int (int a , int b) {
            return a + b;
        };
        
        int c = blk(1,2);
        NSLog(@"%d",c);
    }
    return 0;
}
````
转换成C++代码 (由于生成大量的辅助代码，这里只展示关键代码）
````
typedef int (*blk_t) (int a , int b);

struct __block_impl {
  void *isa;
  int Flags;
  int Reserved;
  void *FuncPtr;
};

struct __main_block_impl_0 {
  struct __block_impl impl;
  struct __main_block_desc_0* Desc;
  __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, int flags=0) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};

static int __main_block_func_0(struct __main_block_impl_0 *__cself, int a, int b) {
  return a + b;
}

static struct __main_block_desc_0 {
  size_t reserved;
  size_t Block_size;
} __main_block_desc_0_DATA = { 0, sizeof(struct __main_block_impl_0)};

int main(int argc, const char * argv[]) {
    /* @autoreleasepool */ { __AtAutoreleasePool __autoreleasepool; 
        blk_t blk = ((int (*)(int, int))&__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA));

        int c = ((int (*)(__block_impl *, int, int))((__block_impl *)blk)->FuncPtr)((__block_impl *)blk, 1, 2);
        NSLog((NSString *)&__NSConstantStringImpl__var_folders_4c_fgfj11vj2t985qr4_0px2sbw0000gn_T_main_a3fdb5_mi_4,c);
    }
    return 0;
}

````
在C++源码中，block最终被转化成函数来处理，blk_t闭包被转换成了结构体，blk_t类型的blk变量初始化时调用了\__main_block_impl_0结构体的构造函数，blk中存储的是构造后的\__main_block_impl_0结构体的首地址。
下面来分解下上面的代码：
````
//block转化后的结构体类型 
struct __main_block_impl_0 {
  //成员结构体变量 保存block需要调用的函数地址和一些辅助信息 
  struct __block_impl impl;
  //成员结构体变量 保存block大小和一个保留字段
  struct __main_block_desc_0* Desc;
  //构造函数 函数名是由编译器根据block所处的函数名和block在该函数中出现的顺序值 （此处0）所生成的
  __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, int flags=0) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};
````

````
//记录block需要调用的函数地址和一些辅助信息
struct __block_impl {
  //指向class_t结构体实例 
  void *isa;
  //一般为0
  int Flags;
  //保留字段 为后续升级做准备
  int Reserved;
  //需要调用的函数地址指针
  void *FuncPtr;
};
````

````
//block大小描述结构体
static struct __main_block_desc_0 {
  //保留字段 为后续升级做准备
  size_t reserved;
  //描述block大小的结构体
  size_t Block_size;
} __main_block_desc_0_DATA = { 0, sizeof(struct __main_block_impl_0)};//直接进行赋值
````
````
//block所需调用的函数的定义（static代表函数的作用域仅限于本文件，不必担心与其他文件中的函数同名）
static int __main_block_func_0(struct __main_block_impl_0 *__cself, int a, int b) {
  return a + b;
}
````
上面的函数参数表中，多了一个struct \__main_block_impl_0 *__cself参数，这个参数类似于C++中的this，类似于OC中的self，是用来指向调用函数本身对象的指针，因为我们可能会在对象的函数体内进行一些对象的成员变量的获取或修改或调用对象的其他成员函数，因为block所调用的函数被解释成了一个C语言形式全局函数，这个函数并不知道是哪个对象在调用它，所以我们需要将调用它的对象指针传递进去。即使这个函数没有int a,int b这两的参数，参数表为空，struct \__main_block_impl_0 \*__cself参数也是必不可少的，必须作为函数参数表中的第0个参数传入。
````
int main(int argc, const char * argv[]) {
    /* @autoreleasepool */ { __AtAutoreleasePool __autoreleasepool; 
        //blk变量的初始化，调用__main_block_impl_0的构造函数，并将其转化成了block
        blk_t blk = ((int (*)(int, int))&__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA));
        //将__main_block_impl_0类型的结构体转换成__block_impl类型结构体，并获取需要调用的函数指针，调用函数
        int c = ((int (*)(__block_impl *, int, int))((__block_impl *)blk)->FuncPtr)((__block_impl *)blk, 1, 2);
        //输出返回值
        NSLog((NSString *)&__NSConstantStringImpl__var_folders_4c_fgfj11vj2t985qr4_0px2sbw0000gn_T_main_a3fdb5_mi_4,c);
    }
    return 0;
}
````
因为\__block_impl结构体是\__main_block_impl_0结构体的成员变量，且是第一个成员变量，那么\__main_block_impl_0的指针和\__block_impl指针的值是相同的（如果不太明白可以去查下结构体在内存上的分配），所以在main函数中可以将\__main_block_impl_0的指针强转成\__block_impl的指针。
在main中，去掉转化部分，便简化为：
````
(*blk->impl.FuncPtr)(blk,1,2);
````

**总结**：block类型的变量，会被转换成 : 结构体 + 函数的形式，在内存上的形式时是以结构体变量进行存储的。结构体当中保存有调用函数的指针，当block发生调用时，会从结构体中将函数指针的值取出，根据函数指针来调用函数，以此便实现了block的调用。

##### block捕获变量
在程序中，共有如下几种类型的变量：
- 局部变量
- 静态变量
- 静态全局变量
- 全局变量
block的变量完全符合上述类型变量。

###### 1) block捕获局部变量
````
typedef void (^blk_t1) (void);
int main(int argc, const char * argv[]) {
        //捕获局部变量
        int d = 10;
        blk_t1 blk1 = ^ {
            NSLog(@"%d",d);
        };
        d = 11;
        blk1();
}
````
打印的结果为：
````
10
````
虽然局部变量d的值实在调用block之前修改的，但是block打印出的值仍未修改之前的，这说明block捕获的是局部变量瞬时的值。 下面将源码转换成C++源码：
````
//其他结构同上面block实质中代码相同 这里展示不同部分
struct __main_block_impl_0 {
  struct __block_impl impl;
  struct __main_block_desc_0* Desc;
  int d;
  __main_block_impl_0(void *fp, struct __main_block_desc_1 *desc, int _d, int flags=0) : d(_d) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};
````
````
int main(int argc, const char * argv[]) {
        int d = 10;
        blk_t1 blk1 = ((void (*)())&__main_block_impl_1((void *)__main_block_func_1, &__main_block_desc_1_DATA, d));
        d = 11;
        ((void (*)(__block_impl *))((__block_impl *)blk1)->FuncPtr)((__block_impl *)blk1);
}
//简化后
int main(int argc, const char * argv[]) {
        int d = 10;
        blk_t1 blk1 = ((void (*)())&__main_block_impl_1((void *)__main_block_func_1, &__main_block_desc_1_DATA, d));
        d = 11;
        (*blk1->impl.FuncPtr)(blk1);
}
````
可以看到`__main_block_impl_0`结构体中多出了一个int d成员变量，在main中d修改之前d就已经作为参数传递给了`__main_block_impl_0`结构体构造函数了，这时`__main_block_impl_0`结构体中的`int d`成员的值已经赋值完毕，在调用block时就会打印成员变量`d`的值。所以局部变量`d`的值的改变不会再影响到block中的`d`的值。

###### 2)修改局部变量
````
typedef void (^blk_t1) (void);
int main(int argc, const char * argv[]) {
        //修改局部变量
        int d = 10;
        blk_t1 blk1 = ^ {
            d = 12;
            NSLog(@"%d",d);
        };
        blk1();
        NSLog(@"%d",d);
}
````
这样是行不通的，直接报错：
````
Variable is not assignable (missing __block type specifier)
````
提示我们缺少`__block`修饰符。更改后代码如下：
````
typedef void (^blk_t1) (void);
int main(int argc, const char * argv[]) {
        //修改局部变量
        __block int d1 = 10;
        blk_t1 blk1 = ^ {
            d1 = 12;
            NSLog(@"%d",d1);
        };
        blk1();
        NSLog(@"%d",d1);
}
````
打印的结果为：
````
12
12
````
达到预期，下面将源码转换成C++源码：
````
struct __Block_byref_d1_0 {
  void *__isa;
__Block_byref_d1_0 *__forwarding;
 int __flags;
 int __size;
 int d1;
};

struct __main_block_impl_2 {
  struct __block_impl impl;
  struct __main_block_desc_2* Desc;
  __Block_byref_d1_0 *d1; // by ref
  __main_block_impl_2(void *fp, struct __main_block_desc_2 *desc, __Block_byref_d1_0 *_d1, int flags=0) : d1(_d1->__forwarding) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};

static void __main_block_func_2(struct __main_block_impl_2 *__cself) {
  __Block_byref_d1_0 *d1 = __cself->d1; // bound by ref

            (d1->__forwarding->d1) = 12;
            NSLog((NSString *)&__NSConstantStringImpl__var_folders_4c_fgfj11vj2t985qr4_0px2sbw0000gn_T_main_c7e4e3_mi_2,(d1->__forwarding->d1));
}

int main(int argc, const char * argv[]) {
        __attribute__((__blocks__(byref))) __Block_byref_d1_0 d1 = {(void*)0,(__Block_byref_d1_0 *)&d1, 0, sizeof(__Block_byref_d1_0), 10};
        blk_t1 blk2 = ((void (*)())&__main_block_impl_2((void *)__main_block_func_2, &__main_block_desc_2_DATA, (__Block_byref_d1_0 *)&d1, 570425344));
        ((void (*)(__block_impl *))((__block_impl *)blk2)->FuncPtr)((__block_impl *)blk2);
        NSLog((NSString *)&__NSConstantStringImpl__var_folders_4c_fgfj11vj2t985qr4_0px2sbw0000gn_T_main_c7e4e3_mi_3,(d1.__forwarding->d1));
};
}
````
可以看到，通过\__block修饰符修饰的变量，都会以结构体的形式在block结构体重存在，而不是向之前直接以基本类型int d的方式存在。`struct __Block_byref_d1_0`结构体中保存了d1的值，和一些其它的辅助信息：
![QQ20170723-130023@2x.png](http://upload-images.jianshu.io/upload_images/2312315-2456ec918390749d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
`struct __Block_byref_d1_0`结构体中的`__Block_byref_d1_0 *__forwarding`成员变量，保存的是自身的地址值，在`static void __main_block_func_2`函数中，`(d1->__forwarding->d1) = 12;`使用forwarding来获取到d1的值。原因在下文呈现。

###### 3)修改静态变量
````
int main(int argc, const char * argv[]) {
        //修改静态变量
        static int d2 = 10;
        blk_t1 blk3 = ^ {
            d2 = 13;
            NSLog(@"%d",d2);
        };
        blk3();
        NSLog(@"%d",d2);
}
````
打印的结果为：
````
13
13
````
与局部变量不同的是，在block内部修改静态变量是不需要添加`__block`修饰符的,可以直接修改静态变量的值。原因如下：
````
struct __main_block_impl_3 {
  struct __block_impl impl;
  struct __main_block_desc_3* Desc;
  int *d2;
  __main_block_impl_3(void *fp, struct __main_block_desc_3 *desc, int *_d2, int flags=0) : d2(_d2) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};

static void __main_block_func_3(struct __main_block_impl_3 *__cself) {
  int *d2 = __cself->d2; // bound by copy

  (*d2) = 13;
  NSLog((NSString *)&__NSConstantStringImpl__var_folders_4c_fgfj11vj2t985qr4_0px2sbw0000gn_T_main_60608c_mi_4,(*d2));
}
````
静态变量会保存在内存的数据段，它的地址是固定分配的，所以block可以直接根据静态变量的地址去操作静态变量，不需要做额外的信息保存操作。
###### 4)修改静态全局变量
````
static int d3;
int main(int argc, const char * argv[]) {
        //修改静态全局变量
        blk_t1 blk4 = ^ {
            d3 = 14;
            NSLog(@"%d",d3);
        };
        blk4();
        NSLog(@"%d",d3);
}
````
打印结果为：
````
14
14
````
由于d3为静态全局变量，d3拥有全局作用范围，所以直接在所有函数中使用d3就可以对d3进行操作，不必向静态变量那样使用指针来去找到变量。
````
struct __main_block_impl_4 {
  struct __block_impl impl;
  struct __main_block_desc_4* Desc;
  __main_block_impl_4(void *fp, struct __main_block_desc_4 *desc, int flags=0) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};

static void __main_block_func_4(struct __main_block_impl_4 *__cself) {

  d3 = 14;
  NSLog((NSString *)&__NSConstantStringImpl__var_folders_4c_fgfj11vj2t985qr4_0px2sbw0000gn_T_main_2fa84d_mi_6,d3);
}
````
###### 4)修改全局变量
````
int d4;
int main(int argc, const char * argv[]) {
        //修改全局变量
        blk_t1 blk5 = ^ {
            d4 = 15;
            NSLog(@"%d",d4);
        };
        blk5();
        NSLog(@"%d",d4);
}
````
打印结果为：
````
15
15
````
由于修改全局变量和修改静态全局变量在底层的实现方式是一模一样的，这里就不再展示转换后的源码了。
##### block的生命周期
block可以向C语言变量一样使用，block在OC中的形式也是以对象的形式存在的。那么block也是有自己的声明周期的，如果block的作用域是在一个函数内的，那么block的生命周期就是函数调用期间在栈上存在的期间。一旦函数执行完毕，函数中的变量从栈帧中弹出，那么block的生命周期随即结束。

有时，我们需要让block的生命周期变长一些来满足业务需要，我们会将block从栈上拷贝到堆上，这时就涉及到一个问题，block中引用的变量怎么办？静态变量和局部变量还好，引用局部变量就变得比较棘手。下面就来回答上面使用`__block`修饰符修改局部变量所留下来的问题：

block 与 __block变量的实质:

| 名称 | 实质 |
| :----: | :----: |
| block | 栈上block的结构体实例 |
| __block变量 | 栈上__block变量的结构体实例 |

在OC中，block被当做类来处理，block有一下三种类：
- _NSConcreteStackBlock
- _NSConcreteGlobalBlock
- _NSConcreteMallocBlock

block不同类在内存上的存储区：

| 类 | 设置对象的存储区域 |
| :----: | :----: |
| _NSConcreteStackBlock | 栈 |
| _NSConcreteGlobalBlock | 数据段 |
| _NSConcreteMallocBlock | 堆 |

内存区域划分如下：
![image.png](http://upload-images.jianshu.io/upload_images/2312315-b0a88d2eb4de0561.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

分配在数据段的全局block，可以在整个程序的作用域中通过指针来使用。栈上的block当函数调用完毕即销毁。堆上的block当没有对象在对其进行引用时，也会被释放，但是一般的我们会将栈上的block拷贝到堆上，延长block的生命周期。

在OC中，我们对一个对象调用copy方法，便会将对象拷贝到堆上去，并得到一个指向堆上对象的指针。如果block中有`__block`类型的变量，那么`__block`类型的变量也会一同被复制到堆上去。

通过`__block`修饰的类型都会以结构体的方式来存储表示：
````
struct __Block_byref_d1_0 {
  void *__isa;
__Block_byref_d1_0 *__forwarding;
 int __flags;
 int __size;
 int d1;
};
````
当block拷贝时，会将栈中block中的`struct __Block_byref_d1_0`结构体中\__forwarding指针指向堆上的那个`struct __Block_byref_d1_0`结构体，而不是栈上的。此后，栈上的block和堆上的block都使用堆上的`__block`类型变量，以此来达到数据共享与一致性。这是因为，当栈上的block释放后，堆上的block依然需要使用`__block`类型的变量，所以需要在堆上开辟空间来存放`__block`类型变量，而堆上的`__block`变量又必须与栈上的`block`对象保持一致，所以就干脆让栈上的block和堆上的block都对堆上的`__block`变量进行操作。

#### 由block导致的内存泄露问题
在一些使用引用计数来进行内存管理的垃圾回收机制，block很容易行程循环引用而导致内存泄露。

举个例子：
````
int main(int argc, const char * argv[]) {
         //产生循环引用 导致内存泄露 
        CopyBlock *copyBlock = [[CopyBlock alloc] init];
        blk_t3 blk6 = ^ (id obj){
            NSLog(@"%@",copyBlock);
        };
        copyBlock.blk = [blk6 copy];
}
````
blk6 调用copy后复制到堆区，并对copyBlock 进行了引用持有。而copyBlock 对象调用alloc在堆上申请空间，并对blk6在堆上的对象进行持有，现在blk6堆上的对象与copyBlock相互持有，循环引用，导致这两个对象使用占有空间，不会释放，垃圾回收机制无法回收，导致内存泄露。

可能有人会问，为什么要让blk6调用copy方法，因为如果blk6不调用copy方法，那么blk6是就在栈上的一个变量，而调用后，会在堆上对栈上的blk6进行一个拷贝，堆栈上现在同时有blk6，以为栈上的blk6当函数调用完毕会自动回收，我们控制不了，所以在堆上对blk6进行一个拷贝，来使程序导致内存泄露。而我们大多数时候都是面对对象编程，会反复的在堆上申请空间，block变量也会反复在堆上申请空间进行保存，所以才这么做来演示内存泄露。

如果我的文章对你有用，不妨在github上给我一个star吧！
[代码地址](https://github.com/wanghanfeng/iOS_Demo/tree/master/Block)