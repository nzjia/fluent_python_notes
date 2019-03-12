# `else`

else 子句不仅能在 if 语句中使用，还能在 for、while 和 try 语句中使用。

- for
  仅当 for 循环运行完毕时（即 for 循环没有被 break 语句中止）才运行 else 块
- while
  仅当 while 循环因为条件为假值而退出时（即 while 循环没有被 break 语句中止）才运行 else 块。
- try
  仅当 try 块中没有异常抛出时才运行 else 块。

在所有情况下，如果异常或者 return、break 或 continue 语句导致控制权跳到了复合语句的主块之外，else 子句也会被跳过。

```python
for item in my_list:
    if item.flavor == 'banana':
        break
else:
    raise ValueError('...')
```

