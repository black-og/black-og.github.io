---

title: mmdetection中的python装饰器
date:  2020-11-18 23:13:11
tags: 解惑
categories: python

---

python中的装饰器一般形式如下：

```python
def funcA(func):
    def funcC(a, b):
        print('start in funcC')
        funcB(a, b)
        print('end in funcC')
    return funcC
  
@funcA
def funcB(a, b):
    print('in funcB')
```

<!--more-->

当执行`funcB(a, b)`时，如果没有装饰器，运行结果为`in funcB`。而若像上面一样使用装饰器后运行结果为：

```
start in funcC
in funcB
end in funcC
```

这是因为使用装饰器时，

```python
@funcA
def funcB(a, b):
```

**相当于`funcB=funcA(funcB)`** 即`funcB=funcC`，因此执行`funcB(a, b)`时变成了执行`funcC(a, b)`，因此得到了上面的结果。

上面是装饰器的常见用法，今天在看mmdection相关代码时，发现它使用了另外一种形式，乍一看有点疑惑，实际上套入上面的套路后也不难理解。具体地，mmdetectino中使用了如下形式：

```python
def funcA():
    def funcC(func):
        print('start in funcC')
        func()
        print('end in funcC')
    return funcC
  
@funcA()
def funcB():
    print('in funcB')
```

以上代码是对mmdetection相关代码的抽象，仅保留了使用形式。相比于第一种形式，主要有三点不同：

- `@funcA`变为了`@funcA()`

- `def funcA(func)`变为了`def funcA()`，参数没了
- `def funcC(a, b)`变为了`def funcC(func)`，即原本和`funcB`对应的参数列表没了，转而变为接受一个函数变量。    

回想之前提到的装饰器的使用套路：

> 这是因为使用装饰器时，
>
> ```python
> @funcA
> def funcB(a, b):
> ```
>
> 相当于`funcB=funcA(funcB)`

同样的，mmdetection使用形式下，相当于发生了：`funcB=funcA()(funcB)`，可以看作两步：第一步执行了`funcA()`返回`funcC`，这也是为什么会导致第二点不同，此时不需要将`funcB`作为实参传递给`funcA`了，只是纯粹调用了`funcA()`，所以`funcA`可以有自己的参数列表，和`funcB`无关；第一步以后得到`funcB=funcC(funcB)`，这也导致了第三点不同，即`funcC`必须接受一个函数变量作为实参，而不是像之前拥有和`funcB`相同的参数列表。

最关键的是，在第一种形式下，只有显示调用`funcB()`才会实际执行；而第二种形式下，只要`import funcB`就会实际执行。因为第一种形式时`funcB=funcC`，只是进行了赋值；而第二种形式时`funcB=funcC(funcB)`，已经进行了函数调用。正是由于这种用法，在mmdetection中，不同的类在它被import进来的时候，就已经完成了注册。

总结一下，关于装饰器的用法，其实牢记`@f1 def f2()`相当于`f2=f1(f2)`即可，不同形式直接套用该流程即可具体分析。