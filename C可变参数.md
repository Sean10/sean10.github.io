---
title: C可变参数
subtitle: stdarg
date: 2020-04-29 01:29:14
updated:
tags: [C, linux, gcc]
categories: [专业]
---

旧笔记整理后发布

<!--more-->

C99编译器允许可变参数宏(variadic macros), GCC默认使用的标准是GNU89/90标准，这个标准在C89的基础上增加了一些C99的功能（就比如这个Variadic macro）[^3]。

``` bash
man gcc

...
 gnu90
           gnu89
               GNU dialect of ISO C90 (including some C99 features). This is the default for C code.
```

## 查看gcc版本
``` bash
gcc -E -dM - </dev/null | grep "STDC_VERSION"
```

``` c
#include <stdio.h>

int main(void) {
#ifdef __STDC__
	printf("%s\n", "stardard c");
#endif
#ifdef __STDC_VERSION__
	printf("%d\n", __STDC_VERSION__);
#endif
	return 0;
}
```


在这个版本中增加的宏`__VA_ARGS__`前增加`##`可以完成对逗号的处理，在没有传入额外参数时，gcc对代码进行预处理时，识别到`##`会将逗号给去除。如果不加入这个，那在不传入额外参数时，就会编译错误。[^4]

## example
``` c
#include <syslog.h>
#include <stdarg.h>
#include <stdio.h>
#define BUFF_LEN_O 1024
#define LOGGER(priority, format, ...) logger(priority, format, __FUNCTION__, __LINE__, ##__VA_ARGS__)

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
	printf("%s %d\n", __FUNCTION__, __LINE__);
	LOGGER(LOG_INFO, "hello world %s", "123");
	LOGGER(LOG_ERR, "new");
	logger(LOG_INFO, "hello world %s", __FUNCTION__, __LINE__, "123");
	return 0;
}
```

## Reference
1. [C99 open\-std\.org/JTC1/SC22/WG14/www/docs/n897\.pdf](http://open-std.org/JTC1/SC22/WG14/www/docs/n897.pdf)
2. [C11 ISO/IEC 9899:201x](http://open-std.org/JTC1/SC22/WG14/www/docs/n1570.pdf)
3. [C Extensions \- Using the GNU Compiler Collection \(GCC\)](https://gcc.gnu.org/onlinedocs/gcc-4.8.5/gcc/C-Extensions.html#C-Extensions)
4. [Variadic Macros \- Using the GNU Compiler Collection \(GCC\)](https://gcc.gnu.org/onlinedocs/gcc-4.8.5/gcc/Variadic-Macros.html#Variadic-Macros)