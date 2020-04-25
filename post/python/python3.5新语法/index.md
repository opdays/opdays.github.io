


>python3.5用了差不多一年了 和2.7有什么差别呢 就知道一个print,马丹的直接去官网看看去


# 一、新语法
- [PEP 492](https://docs.python.org/3.5/whatsnew/3.5.html#whatsnew-pep-492) 协程的 async
- [PEP 465](https://docs.python.org/3.5/whatsnew/3.5.html#whatsnew-pep-465) @操作矩阵的操作符 不太懂
- [PEP 448](https://docs.python.org/3.5/whatsnew/3.5.html#whatsnew-pep-492) *,**的解构赋值

<!--more-->
# 二、新模块
- [typing](https://docs.python.org/3.5/whatsnew/3.5.html#whatsnew-pep-484) 函数参数和返回值预期返回值类型


```python3
def greeting(name: str) -> str:
    return 'Hello ' + name
```

- [zipapp](https://docs.python.org/3.5/whatsnew/3.5.html#whatsnew-zipapp) 打包python包为一个文件


```Terminfo
ls myapp/

__main__.py 1.py 2.py

python -mzipapp myapp

python myapp.pyz 

python -mzipapp myapp -p "/usr/bin/env python"

./myapp.pyz
```
---
# 三、新内建特征

- [PEP 461 ](https://docs.python.org/3.5/whatsnew/3.5.html#whatsnew-pep-461)%可以格式化 bytes and bytearray.
- 新增了3个函数 bytes.hex(), bytearray.hex() and memoryview.hex()
- [memoryview](https://docs.python.org/3.5/library/stdtypes.html#memoryview)memoryview类 支持索引切片
- collections.OrderedDict 快了4-100倍
- RecursionError 一个新异常
- Generators have a new gi_yieldfrom attribute
- 新函数 os.scandir() 更快更好的遍历目录
- functools.lru_cache() 用C重新实现 性能更好
- 运行子进程的新方式subprocess.run()
- traceback模块 增强 方便开发人员
---