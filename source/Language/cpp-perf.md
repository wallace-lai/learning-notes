# 【C++】C/C++代码级性能优化

作者：wallace-lai </br>
发布：2023-11-19 </br>
更新：2023-11-19 </br>

程序优化最重要的就是找出待优化的地方，也就是找出程序的哪些部分或者哪些模块运行缓慢亦或消耗大量的内存。只有程序的各部分经过了优化，程序才能执行的更快。程序中运行最多的部分，特别是那些被程序内部循环重复调用的方法最该被优化。

根据经验，内部或嵌套循环，调用第三方库的方法通常是导致程序运行缓慢的最主要的原因。

## 一、整型数
### 1. 非负的情况下使用unsigned int
如果我们确定整数非负，那么就应该使用unsigned int而不是int。因为有些处理器处理无符号整型数的效率远远高于有符号整型数。因此，在一个紧密的循环语句中，声明一个整型的最好办法是：
```c
register unsigned int variable_name;
```

使用register不能保证编译器一定使用寄存器变量，但这是通用的做法。

### 2. 能用整型运算就不用浮点运算
整型运算速度要高于浮点型，可以被处理器直接运算完成，不需要借助浮点运算单元或者第三方库。比如，我们需要结果精确到小数点后两位，可以将其乘以100，再取后两位的值。

## 二、除法
### 1. 除法和余数
在标准处理器中，对于分子和分母，一个32位的除法需要使用20至140次循环操作。这是一个消耗很大的操作，应该尽可能的避免执行。能用乘法代替除法就使用乘法

### 2. 合并除法和取余数
在既需要除法又需要取余数的情况下，编译器可以通过调用一次除法操作返回除法的结果和余数。因此，我们可以将除法和余数写在一次，一次运算即可得到结果。

```c
unsigned int func1(unsigned int a, unsigned int b)
{
    return a / b;
}

unsigned int func2(unsigned int a, unsigned int b)
{
    return a % b;
}

void func3(unsigned int a, unsigned int b,
    unsigned int *ret1, unsigned int *ret2)
{
    *ret1 = a / b;
    *ret2 = a % b;
}
```

上述代码的汇编结果（结果来源于x86-64 gcc 13.2，经过了O2优化，下同）如下所示。可以看到除法和取余分开会导致执行两次div，但放一起就只需执行一次即可。
```
func1(unsigned int, unsigned int):
        mov     eax, edi
        xor     edx, edx
        div     esi
        ret
func2(unsigned int, unsigned int):
        mov     eax, edi
        xor     edx, edx
        div     esi
        mov     eax, edx
        ret
func3(unsigned int, unsigned int, unsigned int*, unsigned int*):
        mov     r8, rdx
        mov     eax, edi
        xor     edx, edx
        div     esi
        mov     DWORD PTR [r8], eax
        mov     DWORD PTR [rcx], edx
        ret
```

### 3. 通过2的幂次进行除法和取余数
如果除法中的除数是2的幂次，编译器使用移位操作来执行除法。因此，我们需要尽可能的设置除数为2的幂次（例如64而不是66）。
```c
typedef unsigned int uint;

uint div1(uint a)
{
    return a / 32;
}

uint div2(uint a)
{
    return (a >> 5);
}
```

汇编结果如下，可以知道——如果编译器发现除数是2的幂次方，那么它将主动将除法优化成移位操作，与代码主动移位的结果相同。
```
div1(unsigned int):
        mov     eax, edi
        shr     eax, 5
        ret
div2(unsigned int):
        mov     eax, edi
        shr     eax, 5
        ret
```

并且依然记住，无符号unsigned整数除法执行效率高于有符号signed整形。
```c
typedef unsigned int uint;
typedef int sint;

uint div1(uint a)
{
    return a / 32;
}

sint div2(sint a)
{
    return a / 32;
}
```

汇编结果如下所示，可以看到unsigned int的汇编结果比int简洁了不少。
```
div1(unsigned int):
        mov     eax, edi
        shr     eax, 5
        ret
div2(int):
        test    edi, edi
        lea     eax, [rdi+31]
        cmovns  eax, edi
        sar     eax, 5
        ret
```

## 三、数组
### 1. 使用数组下标
当你想根据输入不同的输入来设置不同的变量值，如果可以，请使用数组小标的形式。注意，这个案例非常符合我们平时所遇到的需求，因此该案例价值很高。

我们大概可以写出以下三种风格，从表明代码看不出谁性能更好。

```c
char func1(unsigned num)
{
    char letter;

    switch (num) {
        case 0: letter = 'W'; break;
        case 1: letter = 'V'; break;
        case 2: letter = 'X'; break;
        default: letter = 'U'; break;
    }

    return letter;
}

char func2(unsigned num)
{
    char letter;

    if (num == 0) {
        letter = 'W';
    } else if (num == 1) {
        letter = 'V';
    } else if (num == 2) {
        letter = 'X';
    } else {
        letter = 'U';
    }

    return letter;
}

char func3(unsigned num)
{
    static const char *data = "WVXU";
    return data[num >= 3 ? 3 : num];
}
```

上述代码的汇编代码如下。可以看到switch和if的两种不同形式被优化成了同一段汇编代码（如果分支数多于某个值，二者汇编代码会不同），都是数组下标的方式。第三段代码也被编译成了数组下标方式，但更简洁。如果分支数很多，那么第三段代码和前面两段代码的性能优势就会更加明显。
```
func1(unsigned int):
        mov     eax, 85
        cmp     edi, 2
        ja      .L1
        mov     edi, edi
        movzx   eax, BYTE PTR CSWTCH.1[rdi]
.L1:
        ret
CSWTCH.1:
        .byte   87
        .byte   86
        .byte   88
```

```
func2(unsigned int):
        mov     eax, 85
        cmp     edi, 2
        ja      .L1
        mov     edi, edi
        movzx   eax, BYTE PTR CSWTCH.1[rdi]
.L1:
        ret
CSWTCH.1:
        .byte   87
        .byte   86
        .byte   88
```

```
.LC0:
        .string "WVXU"
func3(unsigned int):
        mov     eax, 3
        cmp     edi, eax
        cmova   edi, eax
        mov     edi, edi
        movzx   eax, BYTE PTR .LC0[rdi]
        ret
```

## 四、变量
### 1.  不要将全局变量放在重要的循环中
全局变量绝不会位于寄存器中。因此，编译器不能将全局变量的值缓存在寄存器中，以至于在使用全局变量时便需要额外的（常常是不必要的）读取和存储。所以，在重要的循环中我们不建议使用全局变量。

如果函数过多的使用全局变量，比较好的做法是拷贝全局变量的值到局部变量，这样它才可以存放在寄存器。这种方法仅仅适用于全局变量不会被我们调用的任意函数使用。例子如下：
```c
int f(void);
int g(void);
int errs;
void test1(void)
{  
    errs += f();  
    errs += g();
} 
void test2(void)
{  
    int localerrs = errs;  
    localerrs += f();  
    localerrs += g();  
    errs = localerrs;
}
```

上述代码中，test1必须在每次操作时加载并存储全局变量errs，而test2中的局部变量localerrs存储与寄存器中的话只需要一个计算机指令即可完成加法。如果对errs的操作很多的话，test2的性能优势将会更明显。

### 2. 显式加载在运算过程中不变的量
考虑如下的例子：
```c
void anyfunc(int, int);

void func1( int *data )
{    
    int i;     
    for(i=0; i<10; i++)    
    {          
        anyfunc( *data, i);    
    }
}
```

尽管`*data`的值可能从未被改变，但编译器并不知道anyfunc函数不会修改它，所以程序必须在每次使用它的时候从内存中读取它。如果我们知道变量的值不会被改变，那么就应该使用如下的编码：
```c
void func2(int *data)
{
    int i;
    int local = *data;
    for (i = 0; i < 10; i++) {
        anyfunc(local, i);
    }
}
```

上述两函数的汇编代码如下，可以看到test1方向需要每次循环时都读取一遍`*data`，而test2方法只需要在循环开始前读取一遍即可。

```
func1(int*):
        push    rbp
        mov     rbp, rdi
        push    rbx
        xor     ebx, ebx
        sub     rsp, 8
.L2:
        mov     edi, DWORD PTR [rbp+0]
        mov     esi, ebx
        add     ebx, 1
        call    anyfunc(int, int)
        cmp     ebx, 10
        jne     .L2
        add     rsp, 8
        pop     rbx
        pop     rbp
        ret
func2(int*):
        push    rbp
        push    rbx
        xor     ebx, ebx
        sub     rsp, 8
        mov     ebp, DWORD PTR [rdi]
.L7:
        mov     esi, ebx
        mov     edi, ebp
        add     ebx, 1
        call    anyfunc(int, int)
        cmp     ebx, 10
        jne     .L7
        add     rsp, 8
        pop     rbx
        pop     rbp
        ret
```

同样的道理，还有我们最常见到的strlen()、vec.size()等。
```c
void func1(char *src, char *buffer)
{
    int i;
    for (i = 0; i < strlen(src); i++) {
        buffer[i] = src[i] + 'A';
    }
}

void func2(char *src, char *buffer)
{
    int i;
    int len = strlen(src);
    for (i = 0; i < len; i++) {
        buffer[i] = src[i] + 'A';
    }
}
```

```
func1(char*, char*):
        push    r12
        mov     r12, rsi
        push    rbp
        mov     rbp, rdi
        push    rbx
        xor     ebx, ebx
        jmp     .L2
.L3:
        movzx   eax, BYTE PTR [rbp+0+rbx]
        add     eax, 65
        mov     BYTE PTR [r12+rbx], al
        add     rbx, 1
.L2:
        mov     rdi, rbp
        call    strlen
        cmp     rbx, rax
        jb      .L3
        pop     rbx
        pop     rbp
        pop     r12
        ret
```

```
func2(char*, char*):
        push    rbp
        mov     rbp, rsi
        push    rbx
        mov     rbx, rdi
        sub     rsp, 8
        call    strlen
        test    eax, eax
        jle     .L1
        lea     ecx, [rax-1]
        xor     eax, eax
.L3:
        movzx   esi, BYTE PTR [rbx+rax]
        lea     edx, [rsi+65]
        mov     BYTE PTR [rbp+0+rax], dl
        mov     rdx, rax
        add     rax, 1
        cmp     rcx, rdx
        jne     .L3
.L1:
        add     rsp, 8
        pop     rbx
        pop     rbp
        ret
```

编译器比较聪明，两者都只调用了一次strlen。

### 4. 变量类型
合理正确地使用变量类型至关重要，这有助于减少代码和数据的大小并增加程序的性能。

### 5. 局部变量尽量不使用char和short
我们应该尽可能的不使用char和short类型的局部变量。对于char和short类型，编译器需要在每次赋值的时候将局部变量减少到8或者16位。这对于有符号变量称之为有符号扩展，对于无符号变量称之为零扩展。

```c
int wordinc (int a)
{   
    return a + 1;
}
short shortinc (short a)
{    
    return a + 1;
}
char charinc (char a)
{    
    return a + 1;
}
```

```
wordinc(int):
        lea     eax, [rdi+1]
        ret
shortinc(short):
        lea     eax, [rdi+1]
        ret
charinc(char):
        lea     eax, [rdi+1]
        ret
```

## 五、指针
### 1. 优先使用传指针的方式传递大对象
我们应该尽可能的使用引用值的方式传递结构数据，也就是说使用指针，否则传递的数据会被拷贝到栈中，从而降低程序的性能

### 2. 指针链
指针链经常被用于访问结构数据。例如，常用的代码如下：
```c
typedef struct { int x, y, z; } Point3;
typedef struct { Point3 *pos, *direction; } Object;
 
void InitPos1(Object *p)
{
   p->pos->x = 0;
   p->pos->y = 0;
   p->pos->z = 0;
}
```

然而，这种的代码在每次操作时必须重复调用p->pos，因为编译器不知道p->pos->x与p->pos是相同的（**真的吗？**）。一种更好的方法是缓存p->pos到一个局部变量：
```c
void InitPos2(Object *p)
{
   Point3 *pos = p->pos;
   pos->x = 0;
   pos->y = 0;
   pos->z = 0;
}
```

另一种方法是在Object结构中直接包含Point3类型的数据，这能完全消除对Point3使用指针操作（破坏了对象的封装性）。

可以看到两个方法的汇编是一样的，也即编译器没有愚蠢到看不懂两个方法其实是一样的。
```
InitPos1(Object*):
        mov     rax, QWORD PTR [rdi]
        mov     QWORD PTR [rax], 0
        mov     DWORD PTR [rax+8], 0
        ret
InitPos2(Object*):
        mov     rax, QWORD PTR [rdi]
        mov     QWORD PTR [rax], 0
        mov     DWORD PTR [rax+8], 0
        ret
```

## 六、表达式
### 1. 尽量将条件集中起来
如下所示，由于条件被集中在了一起，编译器能够将它们集中处理。
```c
int g(int a, int b, int c, int d)
{
   if (a > 0 && b > 0 && c < 0 && d < 0)
   //  grouped conditions tied up together//
      return a + b + c + d;
   return -1;
}
```

### 2. 范围检查
以下的范围检查语句可以转换为等价的下面那个检查语句。
```c
(xmin <= x && x <= xmax ) && (ymin <= y && y <= ymax)
```

当xmin或ymin等于0时，该语句的性能优势更明显。
```c
(x - xmin) <= (xmax - xmin) && (y - ymin) <= (ymax - ymin)
```

### 3. 布尔表达式和零值比较
处理器的标志位在比较指令操作后被设置。标志位同样可以被诸如MOV、ADD、AND、MUL等基本算术和裸机指令改写。如果数据指令设置了标志位，N和Z标志位也将与结果与0比较一样进行设置。N标志表示结果是否是负值，Z标志表示结果是否是0。

C语言中，处理器中的N和Z标志位与下面的指令联系在一起：有符号关系运算`x<0`，`x>=0`，`x==0`，`x!=0`；无符号关系运算`x==0`，`x!=0`（或者`x>0`）。

C代码中每次关系运算符的调用，编译器都会发出一个比较指令。如果操作符是上面提到的，编译器便会优化掉比较指令。例如：
```c
int aFunction(int x, int y)
{
   if (x + y < 0)
      return 1;
  else
     return 0;
}
```
尽可能的使用上面的判断方式，这可以在关键循环中减少比较指令的调用，进而减少代码体积并提高代码性能。C语言没有借位和溢出位的概念，因此，如果不借助汇编，不可能直接使用借位标志C和溢出位标志V。但编译器支持借位（无符号溢出），例如：
```c
int sum(int x, int y)
{
   int res;
   res = x + y;
   if ((unsigned) res < (unsigned) x) // carry set?  //
     res++;
   return res;
}
```

### 4. 利用短路特性
在AND表达式中，务必让最可能较快出结果的条件放在第一个位置。
```c
if (a > 10 && b == 4) {
	// ...
}
```

### 5. 利用switch替代if-else
在if()语句中，如果最后一条语句命中，之前的条件都需要被测试执行一次。Switch允许我们不做额外的测试。如果必须使用if…else…语句，将最可能执行的放在最前面。

编译器可能会自动将if-else优化成switch语句

### 6. 二分查找
在比较时尽量使用二分查找的形式，比如：
```c
if(a==1) {
} else if(a==2) {
} else if(a==3) {
} else if(a==4) {
} else if(a==5) {
} else if(a==6) {
} else if(a==7) {
} else if(a==8) {
}
```

可以优化成：
```c
if (a <= 4) {
	if (a == 1) {
		
	} else if (a == 2) {
		
	} else if (a == 3) {
		
	} else {
		
	}
} else {
	if (a == 5) {
		
	} else if (a == 6) {
		
	} else if (a == 7) {
		
	} else {
		
	}
}
```

### 7. switch vs 查找表
在很多场景下，switch语句都能被查找表代替。如果可以，尽量使用查找表去替代switch语句。
```c
char * Condition_String1(int condition) {
  switch(condition) {
     case 0: return "EQ";
     case 1: return "NE";
     case 2: return "CS";
     case 3: return "CC";
     case 4: return "MI";
     case 5: return "PL";
     case 6: return "VS";
     case 7: return "VC";
     case 8: return "HI";
     case 9: return "LS";
     case 10: return "GE";
     case 11: return "LT";
     case 12: return "GT";
     case 13: return "LE";
     case 14: return "";
     default: return 0;
  }
}
 
char * Condition_String2(int condition) {
   if ((unsigned) condition >= 15) return 0;
      return
      "EQ\0NE\0CS\0CC\0MI\0PL\0VS\0VC\0HI\0LS\0GE\0LT\0GT\0LE\0\0" +
       3 * condition;
}
```

## 七、循环
循环是大多数程序中的常用的结构；程序执行的大部分时间发生在循环中，因此十分值得在循环执行时间上下一番功夫。

### 1. 循环终止
如果不加注意，循环终止条件的编写会导致额外的负担。我们应该使用计数到零的循环和简单的循环终止条件。简单的终止条件消耗更少的时间。看下面计算n！的两个程序。第一个实现使用递增的循环，第二个实现使用递减循环。
```c
int fact1_func (int n)
{
    int i, fact = 1;
    for (i = 1; i <= n; i++)
      fact *= i;
    return (fact);
}
 
int fact2_func(int n)
{
    int i, fact = 1;
    for (i = n; i != 0; i--)
       fact *= i;
    return (fact);
}
```

循环终止条件是每次循环都需要执行的，第二段代码的循环终止条件更简单，因此第二段代码效率更高。

### 2. 更快的for循环
这是一个简单而高效的概念。通常，我们编写for循环代码如下：
```c
for( i=0;  i<10;  i++){ ... }
```

i从0循环到9。如果我们不介意循环计数的顺序，我们可以这样写：
```c
for( i=10; i--; ) { ... }
```

这样快的原因是因为它能更快的处理i的值–测试条件是：i是非零的吗？如果这样，递减i的值。对于上面的代码，处理器需要计算“计算i减去10，其值非负吗？如果非负，i递增并继续”。

简单的循环却有很大的不同。这样，i从9递减到0，这样的循环执行速度更快。

这里的语法有点奇怪，但确实合法的。循环中的第三条语句是可选的（无限循环可以写为for(;;)）。如下代码拥有同样的效果：
```c
for(i=10; i; i--){}
```
或者更进一步的：
```c
for(i=10; i!=0; i--){}
```
这里我们需要记住的是循环必须终止于0（因此，如果在50到80之间循环，这不会起作用），并且循环计数器是递减的。使用递增循环计数器的代码不享有这种优化。

### 3. 合并循环
如果一个循环能解决问题坚决不用二个（**循环任务简单的情况下**）。但如果你需要在循环中做很多工作，这种情况下，两个分开的循环可能会比单个循环执行的更快
（**复杂任务下分开可能会更快**）。下面是一个例子
```c
// original
for (i = 0; i < 100; i++) {
	stuff();
}
for (i = 0; i < 100; i++) {
	morestuff();
}
```

```c
// it would be faster
for (i = 0; i < 100; i++) {
	stuff();
	morestuff();
}
```

### 4. 循环展开
简单的循环可以展开以获取更好的性能，但需要付出代码体积增大的代价。如果循环迭代次数只有几次，那么完全可以展开循环，以便消除循环带来的负担。

```c
for (int i = 0; i < 3; i++) {
	something(i)
}
```

```c
something(0);
something(1);
something(2);
```

有一种循环展开形式更加常见。以下是我们经常写出来的代码，但是它性能不高。
```c
int limit = 33;
for (int i = 0; i < limit; i++) {
	printf("process(%d)\n", i);
}
```

我们可以让每次循环执行8次，也即每次展开8次作为一次循环，这样性能会更好。
```c
#define BLOCKSIZE (8)
int i = 0;
int limit = 33;
int blockLimit = (limit / BLOCKSIZE) * BLOCKSIZE;
while (i < blockLimit) {
	printf("process(%d)\n", i);
	printf("process(%d)\n", i + 1);
	printf("process(%d)\n", i + 2);
	printf("process(%d)\n", i + 3);
	printf("process(%d)\n", i + 4);
	printf("process(%d)\n", i + 5);
	printf("process(%d)\n", i + 6);
	printf("process(%d)\n", i + 7);
	
	i += 8;
}

if (i < limit) {
	switch (limit - i) {
		case 7: printf("process(%d)\n", i); i++;
		case 6: printf("process(%d)\n", i); i++;
		case 5: printf("process(%d)\n", i); i++;
		case 4: printf("process(%d)\n", i); i++;
		case 3: printf("process(%d)\n", i); i++;
		case 2: printf("process(%d)\n", i); i++;
		case 1: printf("process(%d)\n", i); i++;
	}
}
```

同样的道理也可以运用在统计非零位的数量场景当中。
```c
int count1(uint n)
{
	int bits = 0;
	while (n != 0) {
		if (n & 1) {
			bits++;
		}
		n >>= 1;
	}
	
	return bits;
}
```

```c
int count2(uint n)
{
	int bits = 0;
	while (n != 0) {
		if (n & 1) bits++;
		if (n & 2) bits++;
		if (n & 4) bits++;
		if (n & 8) bits++;
		n >>= 4;
	}
	
	return bits;
}
```

### 5. 尽早终止循环
只要能够终止循环，就立刻终止循环，不要做额外的无用功。举个例子，在10000个整数中查找是否存在特殊值99。把代码写成一下的形式是绝对不合格的，无用功太多。

```c
int found = 0;
for (int i = 0; i < 10000; i++) {
	if (list[i] == 99) {
		found = 1;
	}
}

if (found) {
	printf("Yes, we found it\n");
}
```

正确的写法应该是找到99时就立刻终止循环。
```c
int found = 0;
for (int i = 0; i < 10000; i++) {
	if (list[i] == 99) {
		found = 1;
		break;
	}
}

if (found) {
	// ...
}
```

## 八、函数
设计小而简单的函数是个好习惯，这允许寄存器执行一些诸如寄存器变量申请的优化，这是非常高效的。

### 1. 函数调用的性能消耗
函数调用对于处理器的性能消耗是很小的，只占函数执行中性能消耗的一小部分。但是要让函数参数直接传入寄存器中则有一定的限制。要求参数必须是整型兼容的（char、shorts、ints和floats都可在一个字以内表示）或者小于四个字大小（double和long long占用2个字）。

如果系统对参数数量限制是4的话，那么从第5个参数开始后面参数会存储在栈上，这使得函数在调用时需要从栈上加载参数，增加了性能消耗。因此，函数参数尽量不要超过4个。

### 2. 降低函数参数传递的性能消耗
- 尽量保证函数使用少于4个参数
- 如果函数需要多于4个参数，尽量确保后面参数的价值高于让其存储于栈所付出的代价
- 通过指针传递参数的引用而不是传递参数结构本身
- 将参数放入一个结构体并通过指针传入函数，这样可以减少参数的数量并提高可读性
- 尽量少用占用两个字大小的long类型，double同理，也需少用
- 避免变参，变参函数的所有参数都放在栈中

### 3. 叶子函数
不调用任何函数的函数被称为叶子函数，叶子函数非常高效。因此，请尽可能地将经常调用的函数写成叶子函数。

### 4. 内联函数
内联函数禁用所有的编译选项，所有内联函数在调用处被直接替换为函数体。这样的代码调用更快，但增加了代码的大小。因此，仅对重要的简短函数使用inline。

### 5. 使用查找表
函数（部分）通常可以设计成查找表，这样可以显著提升性能。对于某些非常消耗计算性能的程序，如果不在意其精确性，可以用查找表的方式来加速计算过程。

### 6. 浮点运算
浮点运算无论是何种处理器都是非常耗时的，但有时又离不开浮点运算。对于浮点运算，有几点需要注意。
- 浮点除法很慢。浮点除法比加法或者乘法都慢，如果可以，请使用乘法代替除法
- 使用float代替double。如果精度够用，尽可能使用float
- 避免使用先验函数。诸如sin、exp和log等先验函数，非常非常慢，尽量不使用它们
- 简化浮点运算表达式。对于整型表达式3 * (x / 3)，编译器通常可以将其优化成x。但是对于浮点数表达式，编译器无法优化。因此，有必要对浮点表达式进行手工优化。

## 其他
性能优化的一大思想是利用空间换时间。如果你能缓存经常用的数据而不是重新计算，比如使用sin和cos查找表，这些都可以显著提升程序性能。

最重要的性能优化手段是**将编译器优化选项打开**！其他的性能优化技巧：

- 尽量不在循环中使用`++`和`--`，这会导致编译器难以优化
- 减少全局变量的使用（why？）
- 除非想声明为全局变量，否则使用static修饰变量为文件内访问（不要出现在别的地方extern某个变量，这很糟糕）
- 尽量使用一个字大小的变量（int、long等，而不是char、short、double等），使用一个字大小的变量，机器可以运行地更快
- 能使用递推就不使用递归
- 不在循环中使用sqrt开平方函数，计算平方根非常消耗性能
- 一维数组比多维数组快地多
- 避免将相关函数拆分到不通文件中，放一起，编译器可以更好地处理它们
- float比double快
- 浮点乘法比浮点除法更快，尽量使用浮点乘法代替浮点除法
- 加法操作比乘法快，使用`val + val + val`而不是`val * 3`
- put函数比printf函数快，但不灵活
- 使用宏函数的方式替换常用的小函数
- 二进制、未格式化的文件访问要比格式化的文件访问更快，因为程序不需要在ASCII和二进制之间进行转换。因此，如果你不需要阅读文件内容，那么将文件保存为二进制格式
- 如果你的库支持mallopt()函数（用于控制malloc），尽量使用它。MAXFAST的设置，对于调用很多次malloc工作的函数由很大的性能提升。如果一个结构一秒钟内需要多次创建并销毁，试着设置mallopt选项


