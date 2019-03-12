# 上下文

上下文管理器协议包含 `__enter__` 和 `__exit__` 两个方法。with 语句开始运行时，会在上下文管理器对象上调用 `__enter__` 方法。with 语句运行结束后，会在上下文管理器对象上调用 `__exit__` 方法，以此扮演 finally 子句的角色。

```python
class LookingGlass:
    def __enter__(self):
        import sys
        self.original_write = sys.stdout.write
        sys.stdout.write = self.reverse_write
        return 'JABBERWOCKY'
   	
    def reverse_write(self, text):
        self.original_write(text[::-1])
        
    def __exit__(self, exc_type, exc_value, traceback):
        import sys
        sys.stdout.write = self.original_write
        if exc_type is ZeroDivisionError:
            print('Please DO NOT divide by zero!')
            return True
```

## contextlib 模块的实用工具

### 使用 `@contextmanager`

这个装饰器把简单的生成器函数变成上下文管理器，这样就不用创建类去实现管理器协议。

在使用 `@contextmanager` 装饰的生成器中，yield 语句的作用是把函数的定义体分为两部分：yield 语句前面的所有代码在 with 块开始时（即解释器调用 `__enter__` 方法时）执行，yield 语句后面的代码在 with 块结束时（即调用 `__exit__` 方法时）执行。

```python
import contextlib

@contextlib.contextmanager
def looking_glass():
    import sys
    original_write = sys.stdout.write
    
    def reverse_write(text):
        original_write(text[::-1])
    sys.stdout.write = reverse_write
    
    msg = ''
    try:
		yield 'JABBERWOCKY'
        # 在使用 @contextmanager 装饰器时，要把 yield 语句放在 try/finally 语句中（或 with），这是无法避免的，因为我们永远不知道上下文管理器的用户会在 with 块中做什么。
    except ZeroDivisionError:
        msg = '...'
    finally:
	    sys.stdout.write = original_write
        if msg:
            print(msg)
```

