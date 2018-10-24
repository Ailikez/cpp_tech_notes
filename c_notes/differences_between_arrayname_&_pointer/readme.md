## [从编译器角度分析C语言中数组名和指针的区别](http://www.spongeliu.com/28.html)

*On August 21, 2010, in C语言, 语言学习, by sponge*

数组名和指针是两个往往很容易让人们混淆的概念，很多人以为数组名就是一个指针，也有很多人知道数组名不同于指针但是仅知道数组名的值不能像指针一样改变，例如你可以写出下面这样的代码：

```c
int *p;
p++;
```

却不能写这样的代码：

```c
int a[];
a++;
```

那么数组名跟指针之间到底有什么区别呢？

**第一**，在声明上，除了作为函数参数的数组名总是编译器转化成指针，其他情况下，数组名就是数组名，指针就是指针，二者不能混淆，你不能在一个文件中定义一个数组，而在另一个文件中把它声明成一个指针。

```c
char a[];  //定义一个数组a
extern char* a; //在另一个文件中将a声明成一个指针
```

在编译器中，符号表用来存放C语言中有关标识符的属性信息，这些信息集中反应了标识符的特征属性。等到词法分析到代码声称的各个阶段的时候，编译器需要根据源代码提出的要求，从表中获取不同标识符的不同属性。值得注意的是，数组标识符的属性和指针是完全不同的。因此，在一个文件中定义一个数组的时候，编译器会把它记录在符号表中，而在编译器分析另一个文件中的声明时，通过符号的语义检查发现不一致。也许这样的声明可以编译通过(在gcc下通过了)，但是把一个数组名当成一个指针来使用，别指望它能运行起来。

**第二**，指针是一个变量，而数组名不是。数组名是数组的首地址，即它本身就是一个地址，对应到汇编语言级别就是一个常量，一个固定的数（地址）。因此数组名不能进行++,--等运算。

在大多数编译器中，对数组的引用a[i]总是被编译器改写成(a+i)的格式。也就是说，编译器每遇到a[i]，都会把它当作(a+i)来处理。我们都知道，addr表示内存中（addr）的位置存储的值，比如*0x8048000就表示地址为0x8048000的内存中所存储的值。所以a[i]就表示a的值加上i所得到的数作为一个内存地址里面所存储的值。

那么a的值是什么呢？编译器在做词法分析和语法分析的时候，遇到一个数组的定义，就会把数组的有关信息汇集在一个叫做“内情向量”或“信息向量”的表各种，其中的信息包括数组的类型，维数，各维的上、下边界，以及数组的首地址，然后将这个“内情向量”相关信息存储在符号表中。数组定义后位置就是固定的，因此其首地址可以在编译阶段得到。当编译器到达代码生成的各阶段时，每次遇到数组名这个标识符，编译器都会从符号表中取出这个数组的首地址，然后用这个地址来替代数组名，例如，假设数组a起始地址是0x8048000，则a[1]就会被编译器转化成*(0x8048000+1)，因此在生成的汇编代码中，数组名已经完全被转化成一个常量，一个固定的数(地址)。

但是，对于指针p，它是一个变量，其值存储在地址&p中，这个值在编译时是不能得到的。因为是变量，所以指针可以作为表达式中的左值操作，如++或－－，而被认为是常量的数组名却不可以，正如你可以骑走一辆自行车，但是不能骑走一棵树。另外，**C语言把数组下标改写成指针偏移量的根本原因是指针和偏移量是底层硬件所使用的基本模型。**

**第三**，对数组的引用，如a[i]，或(a+1)，需要访存一次；而对指针的引用，如(p+1)，需要访存两次。

如果理解了第二条的解释，这个应该就不难理解。因为a被认为是常数，所以取*(a+1)的值只需将a所表示的常数加1，然后从得到的地址里访存取一次即可。而对于指针，需要先从&p这个地址里把p的值取出来，然后加1，再从得到的地址里访存取一次，一共需要两次访存。

**第四**，假设a是一个数组名，而p是一个指针，当你使用 a 和 &a 时，得到值是一样的，都是数组的起始地址。而使用 p 和 ＆p 时，得到的值是不一样的， p 表示指针 p 所指向的地址，而 &p 表示 p 这个变量的地址。再假设

```c
p = a;
```

则 p 就表示数组 a 的起始地址，而 &p 是存储数组 a 的起始地址的那个地址。

这是因为编译器把 a 当成数组首地址，而 &a 当作数组第一个元素的地址，因此得到的值是一样的。

另外，sizeof(a) 得到的是 a 所表示的数组的大小，而 sizeof(p) 得到的是指针的大小。