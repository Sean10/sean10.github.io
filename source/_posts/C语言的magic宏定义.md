---
title: C语言的magic宏定义
subtitle: macro
date: 2020-05-29 01:07:01
updated:
tags: [C, macro]
categories: [专业]
---

## 可变参...和 ##__VA_ARGS

基本使用方式如下，我主要用来记日志了

<!--more-->

``` c
#include <syslog.h>
#include <stdarg.h>
#include <stdio.h>
#define BUFF_LEN_O 1024
#define LOGGER(priority, format, ...) do{ \
            logger(priority, format, __FUNCTION__, __LINE__, ##__VA_ARGS__); \
            }while(0)
#define PRINT(x, a)  printf("\n")

static void logger(int priority, const char *format, ...)
{
    openlog("test_log", LOG_PID | LOG_NDELAY, LOG_LOCAL0);
    va_list ap;
    char buff[BUFF_LEN_O] = {0};
    char placeholder[] = "function{%s},line{%d}";
    va_start(ap, format);
    snprintf(buff, sizeof(buff) - 1, "%s %s", placeholder, format);
    vsyslog(priority, buff, ap);
    va_end(ap);
}

int main()
{
    LOGGER(LOG_INFO, "hello world %s", "123");
    LOGGER(LOG_ERR, "new");
    logger(LOG_INFO, "hello world %s", __FUNCTION__, __LINE__, "123");
    return 0;
}

```

## #字符串化
会直接把表达式按照字符串输出，并过滤掉注释部分以及前后的空白。
``` c
#include <stdio.h>
#define PRINT(x)  do {printf("\n"#x"\n");}while(0)

int main()
{
    PRINT(print("hello world")/*new*/);
    return 0;
}
```

## ##Token拼接符号[^4]

这个就比较有意思了，可以通过这个结合gcc的typeof模拟出不少泛型的方式，在编译前预处理时扩展出不少函数或变量定义。参考[^4]文章的大佬用宏扩展出了弱图灵完备的函数。

按照大佬说的，这个库[Cloak/cloak\.h at master · pfultz2/Cloak](https://github.com/pfultz2/Cloak/blob/master/cloak.h)非常多的例子

### 延迟展开

``` c
#define EXPAND(...) __VA_ARGS__
#define EMPTY()
#define DEFER(id) id EMPTY()
#define FOO() printf("macro\n");


DEFER(FOO)() /*这个解到最后就是FOO()*/
//    DEFER(FOO)();
EXPAND(DEFER(FOO)()); /*这个最后输出macro*/
```


## _Generic[^6]

使用方式，`_Generic((var), type1 : ..., type2 : ..., ……, default : ...)`，感觉果然和大佬们说的一样，比较像是对基本类型进行switch。

比起这个，可能gcc内建的像typeof这些关键字用起来更具有扩展性。
``` c

#include <stdio.h>
#include <complex.h>
 
#define CUSTOM_GENERIC(_var_) _Generic((_var_), \
/*signed char*/                 signed char : printf("type signed char, var:%d\n", _var_), \
/*signed short*/                signed short : printf("type signed short, var:%hd\n", _var_), \
/*signed int*/                  signed int : printf("type signed int, var:%d\n", _var_), \
/*signed long int */            signed long int : printf("type signed long int, var:%ld\n", _var_), \
/*signed long long int*/        signed long long int : printf("type signed long long int, var:%lld\n", _var_), \
/*unsigned char*/               unsigned char : printf("type unsigned char, var:%c\n", _var_), \
/*unsigned short*/              unsigned short : printf("type unsigned short, var:%hu\n", _var_), \
/*unsigned int*/                unsigned int : printf("type unsigned int, var:%u\n", _var_), \
/*unsigned long int*/           unsigned long int : printf("type unsigned long int, var:%lu\n", _var_), \
/*unsigned long long int*/      unsigned long long int : printf("type unsigned long long int, var:%llu\n", _var_), \
/*float*/                       float : printf("type float, var:%f\n", _var_), \
/*double*/                      double : printf("type double, var:%lf\n", _var_), \
/*long double*/                 long double : printf("type long double, var:%llf\n", _var_),  \
/*_Bool*/                       _Bool : printf("type _Bool, var:%d\n", _var_),  \
/*float _Complex*/              float _Complex : printf("type float _Complex, var:%f+%fi\n", crealf(_var_), cimagf(_var_)),  \
/*double _Complex*/             double _Complex : printf("type double _Complex, var:%f+%fi\n", creal(_var_), cimag(_var_)),  \
/*long double _Complex*/        long double _Complex : printf("type long double _Complex, var:%lf+%lfi\n", creall(_var_), cimagl(_var_)),  \
/*default*/                     default : printf("type default!") \
)
 
int main(void)
{
    int a = 10;
    float f = 100.0f;
    float _Complex fCex = 100.0f + 1.0if;
 
    CUSTOM_GENERIC(a);              //type signed int, var:10
    CUSTOM_GENERIC(f);              //type float, var:100.000000
    CUSTOM_GENERIC(fCex);           //type float _Complex, var:100.000000+1.000000i
    CUSTOM_GENERIC(12);             //type signed int, var:12
    return 0;
}
```






## Reference
1. [宏定义的黑魔法 \- 宏菜鸟起飞手册](https://onevcat.com/2014/01/black-magic-in-macro/)
2. [C preprocessor \- Wikipedia](https://en.wikipedia.org/wiki/C_preprocessor#Token_concatenation)
3. [C11新增关键字：\_Generic\(泛型\)\_c/c\+\+\_南雨兮\-CSDN博客](https://blog.csdn.net/qq_31243065/article/details/80904613)
4. [宏定义黑魔法\-从入门到奇技淫巧 \(3\) \| Feng\.Zone](http://feng.zone/2017/05/20/%E5%AE%8F%E5%AE%9A%E4%B9%89%E9%BB%91%E9%AD%94%E6%B3%95-%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E5%A5%87%E6%8A%80%E6%B7%AB%E5%B7%A7-3/)