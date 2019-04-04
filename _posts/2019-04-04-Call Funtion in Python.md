---
layout:     post
title:      Call Funtion in Python
subtitle:   
date:       2019-04-04
author:     CescMessi
header-img: img/python.png
catalog: true
tags:
    - Python
---

# 问题
昨天在学习Pytorch时，遇到了这样的代码：
```python
self.conv1 = nn.Conv2d(1, 6, 5)
self.conv1(x)
```
第一眼看去以为`nn.Conv2d()`返回了一个函数，这样的确可以说通。但是看到源代码发现事情没有那么简单，`nn.Conv2d`其实是一个类。从名字上看也的确是一个类应该有的样子。源代码如下：
```python
@weak_module
class Conv2d(_ConvNd):
    def __init__(self, in_channels, out_channels, kernel_size, stride=1,
                 padding=0, dilation=1, groups=1,
                 bias=True, padding_mode='zeros'):
        kernel_size = _pair(kernel_size)
        stride = _pair(stride)
        padding = _pair(padding)
        dilation = _pair(dilation)
        super(Conv2d, self).__init__(
            in_channels, out_channels, kernel_size, stride, padding, dilation,
            False, _pair(0), groups, bias, padding_mode)

    @weak_script_method
    def forward(self, input):
        if self.padding_mode == 'circular':
            expanded_padding = ((self.padding[1] + 1) // 2, self.padding[1] // 2,
                                (self.padding[0] + 1) // 2, self.padding[0] // 2)
            return F.conv2d(F.pad(input, expanded_padding, mode='circular'),
                            self.weight, self.bias, self.stride,
                            _pair(0), self.dilation, self.groups)
        return F.conv2d(input, self.weight, self.bias, self.stride,
                        self.padding, self.dilation, self.groups)
```
仔细看看并没有什么特别的，但是的确将实例当作函数使用了，查了书上也没有看到类似的用法，网上一些博客称这里是调用了`forward`函数，但是并没有给出原因。是因为除了`__init__`只有一个方法吗？于是写了下面的代码测试了一下，不出意外果然报错……
```python
class A:
    def __init__(self):
        pass
    
    def aaa(self, str):
        print(str)

a = A()
a('aaa')
```
![报错屏幕截图](https://i.loli.net/2019/04/04/5ca59035756d9.png)
原因是A对象不是callable，感觉接近事实真相了。

# 解决
网上对callable的解释：
>callable() 函数用于检查一个对象是否是可调用的。如果返回 True，object 仍然可能调用失败；但如果返回 False，调用对象 object 绝对不会成功。
>对于函数、方法、lambda 函式、 类以及实现了 `__call__` 方法的类实例, 它都返回 True。

看到这里就明白了，应该是这个类实例实现了`__call__`方法。尽管在`Conv2d`类里没有这个方法，但可以看到它是`_ConvNd`的子类，二`_ConvNd`又是`Moudle`的子类，在`Moudle`类里确实存在一个`__call__`方法：
```python
def __call__(self, *input, **kwargs):
        for hook in self._forward_pre_hooks.values():
            hook(self, input)
        if torch._C._get_tracing_state():
            result = self._slow_forward(*input, **kwargs)
        else:
            result = self.forward(*input, **kwargs)
        for hook in self._forward_hooks.values():
            hook_result = hook(self, input, result)
            if hook_result is not None:
                raise RuntimeError(
                    "forward hooks should never return any values, but '{}'"
                    "didn't return None".format(hook))
        if len(self._backward_hooks) > 0:
            var = result
            while not isinstance(var, torch.Tensor):
                if isinstance(var, dict):
                    var = next((v for v in var.values() if isinstance(v, torch.Tensor)))
                else:
                    var = var[0]
            grad_fn = var.grad_fn
            if grad_fn is not None:
                for hook in self._backward_hooks.values():
                    wrapper = functools.partial(hook, self)
                    functools.update_wrapper(wrapper, hook)
                    grad_fn.register_hook(wrapper)
        return result
```
这个代码就很明显了，的确有调用`forward`方法，不过调用的方式不是直接调用`Conv2d`里面的`forward`方法，而是通过`Moudle.__call__()`来调用的。

之后在[StackOverFlow](https://stackoverflow.com/questions/54518808/pytorch-weak-script-method-decorator)上发现也有人看到这个代码有着同样的疑问，他猜想是修饰器的作用，回答里面也很好地解答了这个问题。
>`embedding(input)` follows the Python function call syntax, which can be used with both "traditional" functions and with objects which define the `__call__(self, *args, **kwargs)` magic function. 

当然，我们之前的代码这样修改就不会报错了：
```python
class A:
    def __init__(self):
        pass
    
    def __call__(self, str):
        print(str)

a = A()
a('aaa')
```
![不报错截图](https://i.loli.net/2019/04/04/5ca597b66289c.png)