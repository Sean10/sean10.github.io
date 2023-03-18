---
title: Python脚本常用模块记录
subtitle: share
date: 2018-11-19 22:41:31
updated:
tags: [Python, script]
categories: [专业]
---

最近老是写脚本，感觉主要用Python的动力就在于Python存在大量造好的轮子，开箱即用，性能和全面性又能比自己现造的轮子要好。

## 计数

首先就拿`Counter`类个例子

``` python
a = [10, 8, 6, 7, 2, 8, 4, 10, 3, 7, 8, 4, 5, 7, 2, 2, 3, 8, 8, 9, 6, 2, 2, 7, 8, 7, 4, 8, 5, 2]

    e = dict()
    for item in a:
        if item in e:
            e[item] += 1
        else:
            e[item] = 1
    

    b = []
    for item in set(a):
        b.append((item, a.count(item)))

    c = defaultdict(int)
    for i in a:
        c[i] += 1

    d = Counter(a)

```

有4种方式，逐渐演变出来，内建函数就支持了计数功能

## 性能优化

``` python
import cProfile
import timeit
from collections import Counter, defaultdict

a = [10, 8, 6, 7, 2, 8, 4, 10, 3, 7, 8, 4, 5, 7, 2, 2, 3, 8, 8, 9, 6, 2, 2, 7, 8, 7, 4, 8, 5, 2]


def count1():
    e = dict()
    for item in a:
        if item in e:
            e[item] += 1
        else:
            e[item] = 1

def count2():
    b = {item: a.count(item) for item in set(a)}

def count3():

    c = defaultdict(int)
    for i in a:
        c[i] += 1

def count4():
    d = Counter(a)


def main():
    count1()
    count2()
    count3()
    count4()

if __name__ == '__main__':
    
    cProfile.run("main()")
    # timeit.timeit("main()", number=1)

```

像这样就不需要再像下面这样手动写了
``` python
start = time.time()

....

stop = time.time()
total = stop - start
```
## 内存监控

memory_profiler

``` python
from memory_profiler import profile

@profile
def main():
    xxx


if __name__ == '__main__':
    main()

```


## 解析配置文件

``` python
import configparser

cf = configparser.ConfigParser()
cf.read('conf.ini')

secs = cf.sections()

print('sections:', secs, type(secs))

opts = cf.options('baseconf')

print("opts", opts, type(opts))
host = cf.get('baseconf', 'host')
print(host)

for sec in cf.sections():
    for opt in cf.options(sec):
        print(cf.get(sec, opt))

cf.set('baseconf', 'host', '0.0.0.0')
cf.write(open('temp.conf', 'w'))

```

读取无section名的常用linux conf文件

``` python
import configparser

with open('temp.conf', 'r') as f:
    config_string = '[dummy_section]\n' + f.read()
cf = configparser.ConfigParser()
cf.read_string(config_string)

for sec in cf.sections():
    for opt in cf.options(sec):
        print(cf.get(sec, opt))

```

## 读写xml

``` python
with open('temp.xml', 'r') as f:
    doc = xmltodict.parse(f.read())
import json
print(json.dumps(doc, indent=4))
print(doc['domain'])
print(doc['domain']['cpu']['@mode'])
doc['domain']['cpu']['@mode'] = 123
print(doc['domain']['memory']['#text'])
doc['domain']['memory']['#text'] = "1111111"
with open('out.xml', 'w') as f:
    f.write(xmltodict.unparse(doc, pretty=True))

```

## 读写json

``` python
import json

with open('temp.json', 'r') as f:
    content = f.read()
    jsonData = json.loads(content)
print(jsonData)

print(json.dumps(jsonData, indent=2))
```

## 起一个简单的http接口服务

``` python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def index():
    return "hello world"

if __name__ == '__main__':
    app.run(host="0.0.0.0", port=80)

```

## RPC接口 相比http性能更好

## Pipe管道

Python默认是前缀语法，是下面这么多个括号嵌套
``` python
sum(select(where(take_while(fib(), lambda x: x < 1000000) lambda x: x % 2), lambda x: x * x))
```
这种的，而不是下面这样的中缀语法，类似这样链式调用
``` python
from pipe import *

fib() | take_while(lambda x: x < 1000000) \
      | where(lambda x: x % 2) \
      | select(lambda x: x * x) \
      | sum()

```




