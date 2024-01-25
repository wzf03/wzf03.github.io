---
layout: post
title: Pytorch TorchDynamo
date: 2024-01-25 14:48 +0800
image: assets/img/covers/pytorch.png
categories:
    - pytorch
---
## 概要

TorchDynamo的作用是从应用中抓取计算图，相比与TorchScript和TorchFX，更加灵活和可靠性更高，使用更方便。

## 简单用法

### 对函数使用

```python
def foo(x, y):
    a = torch.sin(x)
    b = torch.cos(y)
    return a + b
opt_foo1 = torch.compile(foo)
print(opt_foo1(torch.randn(10, 10), torch.randn(10, 10)))
```

```python
@torch.compile
def opt_foo2(x, y):
    a = torch.sin(x)
    b = torch.cos(y)
    return a + b
print(opt_foo2(torch.randn(10, 10), torch.randn(10, 10)))
```

### 对Module使用

```python
class MyModule(torch.nn.Module):
    def __init__(self):
        super().__init__()
        self.lin = torch.nn.Linear(100, 10)

    def forward(self, x):
        return torch.nn.functional.relu(self.lin(x))

mod = MyModule()
opt_mod = torch.compile(mod)
print(opt_mod(torch.randn(10, 100)))
```

## 参数

```python
def torch.compile(model: Callable,
  *,
  mode: Optional[str] = "default",
  dynamic: bool = False,
  fullgraph:bool = False,
  backend: Union[str, Callable] = "inductor",
  # advanced backend options go here as kwargs
  **kwargs
) -> torch._dynamo.NNOptimizedModule
```

### mode

编译器优化的角度。

- `default` 默认行为，平衡内存、编译时间、性能
- `reduce-overhead` 略微增加内存，大幅度降低框架的额外开销
- `max-autotune` 编译需要较长时间，但带来最大的提升

### dynamic

启用对动态Shape的支持。会关闭一些编译器优化。

### fullgraph

将整个程序编译为一个图，有局限，但可获得极限性能。

### backend

编译器后端。

## 原理

TorchDynamo的编译发生在将要执行前，翻译字节码并捕获计算图。
通过PEP-523提供的CPython Frame Evaluation API，通过回调函数获取字节码，并把修改后的字节码返回给编译器执行，或执行预先编译好的字节码。

TorchDynamo实现了一个Python虚拟机的模拟器，模拟字节码执行过程中构建出的计算图。
TorchDynamo会为被编译的函数创建`Guard`，检测被编译函数的输入属性是否发生变化，并决定是否重新编译。
被编译好的函数被保存在cache中，cache的大小为64，因此一个Python函数的输入最多可以有64种变化，超过限制后不再编译该函数。

### Graph Break

TorchDynamo碰到无法支持的算子会创建graph break，把计算图切分成可以支持的几张子图，并用python解释器处理它无法处理的算子。
比如用张量的值作为`if`语句的条件。

TorchDynamo未编译的子图会在首次执行的时候被捕获并编译。

### DistributedDataParallel

为了能够在一个bucket中的梯度就绪时能够及时被allreduce通信，TorchDynamo会在每个bucket的边界引入graph break。

### 优化措施

- 循环展开
- 内联函数
- 特化

## 参考

- [一文搞懂TorchDynamo原理](https://zhuanlan.zhihu.com/p/630933479)
- [Introdution to `torch.compile`](https://pytorch.org/tutorials/intermediate/torch_compile_tutorial.html)
