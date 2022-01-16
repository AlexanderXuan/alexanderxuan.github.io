Title: python的进度条tqdm
Date: 2022-01-16 17:04
Category: python小技巧
Tags: python, tqdm
Author: Alexander Xuan
Summary: python中的多进程tqdm显示

#### tqdm进度条工具
在进行大量数据处理的时候，等待是一个比较难受的过程，能够显示数据的处理进度，预估一个差不多的时间对于我们来说是非常重要的。tqdm是一个非常容易使用的进度条工具。一般我们简单的使用进度条工具没什么难度，但对于大量数据多进程处理来说，进度条的显示就需要有一些小的调整，本文主要介绍tqdm的简单用法和多进程数据处理的整体显示和多进度条显示方法。

##### 需求
简单单进程数据处理任务显示进度条，多进程数据处理显示一个整体进度条，多进程数据处理显示多个进程各自的进度条

##### 实现方法
单进程的任务非常简单，直接加一个tqdm前缀即可：
```python
from tqdm import tqdm


if __name__ == "__main__":
	for i in tqdm(range(100)):
		pass

```
多进程数据处理显示同一个整体进度条需要如下设置：
```python
from tqdm import tqdm  
from multiprocessing import Pool


def foo(param):
	arg1, arg2 = param
	pass


if __name__ == "__main__":
	params = []
	for i in range(10):
		params.append((i, i+1))
    with Pool(8) as pool:
        res = list(tqdm(pool.imap(foo, params), total=len(params), desc='foo'))
    pool.close()
    pool.join()

```

多进程数据处理显示多个进度条：
```python
import time
from multiprocessing import Process, current_process
from tqdm import tqdm


def func(rank):
    for _ in tqdm(range(5), desc=f'sprocess {rank}', position=current_process()._identity[0]):
        time.sleep(1)
    

if __name__ == "__main__":
    num_jobs = 5
    process_list = []
    for i in range(num_jobs):
        p = Process(target=func, args=(i,))
        p.start()
        process_list.append(p)

    for p in process_list:
        p.join()
    print("Done.")
```

更多的多进程教程可以参考[[python多进程]]