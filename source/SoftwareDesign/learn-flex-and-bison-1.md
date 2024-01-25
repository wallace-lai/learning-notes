# 【flex与bison学习笔记】第二章

作者：wallace-lai <br/>
发布：2024-01-25 <br/>
更新：2024-01-25 <br/>

第二章主要是讲述flex工具的高级话题。其中的大部分话题可能不太好理解，目前先暂时略过这些内容，等后续项目中需要用到它们时再来学习。第二章的内容可以总结成以下几个问题：

（1）flex中如何设置其从何处读取数据？

（2）yywrap()是什么，如何关闭它？

（3）flex中如何读取多个文件作为输入？

（4）flex的输入和输出管理层次（简述）

（5）flex中的起始状态和嵌套输入文件特性（略过）

（6）flex中的符号表是什么？如何管理并使用它？（略过）

（7）flex如何与C语言进行交叉引用（略过）

## 一、flex中如何设置其从何处读取数据？
flex默认从标准输入读取数据，具体而言是flex总是通过名为yyin的句柄读取输入，而在不指定的默认情况下yyin指向标准输入。因此，要设置flex从何处读取数据，只需要修改yyin为想要的句柄即可。

具体修改yyin的案例在下一节中展示。

## 二、yywrap是什么，如何关闭它？
`yywrap()`是早期flex版本遗留下来的一个“鸡肋”。当flex词法分析器到达yyin的结束位置时，它将调用`yywrap()`，这样做的目的是当有另外一个输入文件时，`yywrap()`可以调整yyin的值并且通过返回0来重新启动词法分析。

但是，以上的内容都无需记忆，因为flex的最佳实践是使用`%option noyywrap`选项来关闭yywrap并且使用flex其它的IO管理特性。

下面是一个关闭yywrap并修改yyin以使flex从文件中读取输入的案例。在以后的案例中都推荐关闭yywrap，使用本节讲解的IO管理方式。

```
%option noyywrap

%{
    int chars = 0;
    int words = 0;
    int lines = 0;
%}

%%
[^ \t\n\r\f\v]+     { words++; chars += strlen(yytext); }
\n                  { chars++; lines++; }
.                   { chars++; }
%%

/**
 * 功能：关闭默认的yywrap()并修改yyin以实现修改flex程序的输入来源
 */
int main(int argc, char **argv)
{
    if (argc > 1) {
        if (!(yyin = fopen(argv[1], "r"))) {
            perror(argv[1]);
            return (1);
        }
    }

    yylex();
    printf("%8d%8d%8d\n", lines, words, chars);

    return 0;
}
```

具体而言，只要用户指定了额外的参数作为文件路径，那么上述程序将通过调用fopen来打开该文件并将返回的句柄赋值给yyin。这样在调用`yylex()`开始进行词法分析时，flex将读取文件内容作为输入数据。

## 三、flex中如何读取多个文件作为输入？
可以使用`yyrestart()`来读取多个文件作为flex的输入，具体见下面的案例。

```
%option noyywrap

%{
    int chars = 0;
    int words = 0;
    int lines = 0;

    int totchars = 0;
    int totwords = 0;
    int totlines = 0;
%}

%%
[^ \t\n\r\f\v]+     { words++; chars += strlen(yytext); }
\n                  { chars++; lines++; }
.                   { chars++; }
%%

/**
 * 功能：通过yyrestart()来读取多个文件作为输入
 */
int main(int argc, char **argv)
{
    int i;

    if (argc < 2) { /* 读取标准输入 */
        yylex();
        printf("%8d%8d%8d\n", lines, words, chars);
        return 0;
    }

    for (i = 1; i < argc; i++) {
        FILE *f = fopen(argv[i], "r");
        if (!f) {
            perror(argv[i]);
            return (1);
        }

        yyrestart(f);
        yylex();
        fclose(f);
        printf("%8d%8d%8d %s\n", lines, words, chars, argv[i]);

        totchars += chars; chars = 0;
        totwords += words; words = 0;
        totlines += lines; lines = 0;
    }

    if (argc > 1) {
        printf("%8d%8d%8d total\n", totlines, totwords, totchars);
    }

    return 0;
}
```

具体而言是在调用`yylex()`开始进行词法分析之前，使用`yyrestart(f)`将打开的文件句柄传递给flex作为输入数据来源。可以看到，显示地调用`yyrestart()`的逻辑要比默认的yywrap机制更加简洁易懂。

## 四、flex的输入和输出管理层次（简述）
flex中的三个输入管理层次为：

- （1）设置yyin来读取所需文件，如上面的案例所示

- （2）创建并使用YY_BUFFER_STATE输入缓冲区

- （3）重定义YY_INPUT

对于YY_BUFFER_STATE，flex使用称为YY_BUFFER_STATE的数据结构来处理输入。该结构定义了一个单一的输入源，它包含了一个字符串缓冲区以及一些变量和标记，通常会有一个指向所读取文件的FILE指针。

但是我们也可以创建一个与文件无关的YY_BUFFER_STATE来分析已经在内存中的字符串，相当于是输入来源成了内存，不再是文件了。至于YY_BUFFER_STATE如何创建，这里不再深究。

对于YY_INPUT而言，每当词法分析其的输入缓冲区为空时，flex就会调用YY_INPUT。因此，我们可以重新定义该宏用于设置flex的输入数据来源。如下所示，buf是缓冲区，max_size是缓冲区的大小，而result则用来存放实际读取的长度。具体如何重定义YY_INPUT宏，这里不再深究。

```
#define YY_INPUT(buf, result, max_size) ...
```

flex的输出管理层次：flex中默认所有没有被匹配的输入都会被拷贝到yyout。比如下面的案例中，所有不是colour和smart的单词（没有命中模式）都会原封不动地输出。

```
%%
"colour"        { printf("color"); }
"smart"         { printf("elegant"); }
%%
```




