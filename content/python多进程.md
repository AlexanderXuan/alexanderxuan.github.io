Title: python多进程
Date: 2022-01-16 17:00
Category: python小技巧
Tags: python, mutiprocessing
Author: Alexander Xuan
Summary: python的多进程示例

### 多进程

python使用多进程更多一些，因为由于GIL的原因，多线程只会在同一个cpu核上进行，所以无法充分发挥多核cpu的性能，两者写起来差异其实不大。这里使用multiporcessing包来实现。

#### 多进程模版
首先multiprocessing中的子进程对象叫Process，创建一个子进程就是实例化一个Process对象。

```python
import os
from multiprocessing import Process


# 子进程要执行的代码
def run_proc(name):
    print('Run child process %s (%s)...' % (name, os.getpid()))


if __name__=='__main__':
    print('Parent process %s.' % os.getpid())
    p = Process(target=run_proc, args=('test',))
    print('Child process will start.')
    p.start()	# 进程开始
    p.join()	# 等待进程结束
    print('Child process end.')

```

这是单个子进程的情况，多个子进程只需要for循环即可：
```python
import os
from multiprocessing import Process


# 子进程要执行的代码
def run_proc(name):
    print('Run child process %s (%s)...' % (name, os.getpid()))


if __name__=='__main__':
    print('Parent process %s.' % os.getpid())
	
	process_list = []
	for i in range(10):
    	p = Process(target=run_proc, args=('test',))
    	print(f'{i}th Child process will start.')
    	p.start()	# 进程开始
		process_list.append(p)
		
	for p in process_list:
    	p.join()	# 等待进程结束
		
    print('Done.')

```

multiprocessing中还封装了一次性创建大量进程的进程池对象：
```python
from multiprocessing import Pool
import os, time, random

def long_time_task(name):
    print('Run task %s (%s)...' % (name, os.getpid()))
    start = time.time()
    time.sleep(random.random() * 3)
    end = time.time()
    print('Task %s runs %0.2f seconds.' % (name, (end - start)))

if __name__=='__main__':
    print('Parent process %s.' % os.getpid())
    p = Pool(4)	# 实例化进程池并指定同时执行的子进程数量
    for i in range(5):	# 这里指定5，所以第五个任务不是与前四个同时执行
        p.apply_async(long_time_task, args=(i,))
    print('Waiting for all subprocesses done...')
    p.close()
    p.join()
    print('All subprocesses done.')
```
另外还有进程池的map方法和imap方法，用法是一样的，我们这里演示imap方法，因为imap不会将传入的可迭代对象转换为列表，且处理完一个进程之后立即返回结果，map会将所有进程处理完之后一同返回结果。
```python
from multiprocessing import Pool


def foo(param):
	arg1, arg2 = param
	pass


if __name__ == "__main__":
	params = []
	for i in range(10):
		params.append((i, i+1))
    with Pool(8) as pool:
        pool.imap(foo, params)
    pool.close()
    pool.join()
```
多进程处理数据也可以显示进度条，具体教程可以参考[[进度条tqdm工具多进程显示]]

##### 进程间通信
进程之间的通信会有很大的用处，multiprocessing也提供了可用于进程间通信的Queue，Pipe和Manager，其中Queue和Pipe差不多，只是一个是队列形式写入，一个是管道形式写入，但都是不同进程放入数据和拿取数据的操作，Manager可以作为进程间的共享数据来进行共同修改，可以当作是一个全局变量。
Queue模版：
```python

import os
import sys
import time
from multiprocessing import Process, Queue


def func_input(input_queue, name_list, i):
    for name in name_list:
        print(f"process {i} put data {name}")
        input_queue.put(name)
        time.sleep(1)
    input_queue.put('quit')
    print('exit input.')


def func_get(get_queue, i):
    while True:
        if not get_queue.empty():
            name = get_queue.get()
            if name == 'quit':
                get_queue.put('quit')
                print('exit get.')
                break
            print(f"process {i} get data {name}")


if __name__ == "__main__":
    queue = Queue()
    name_lists = [['a', 'b', 'c'], ['d', 'e', 'f'], ['g', 'h']]
    num_put_jobs = 2
    num_get_jobs = 3
    process_list = []
    for i in range(num_put_jobs):
        p = Process(target=func_input,args=(queue, name_lists[i], i,))
        p.start()
        process_list.append(p)

    for i in range(num_get_jobs):
        p = Process(target=func_get,args=(queue, i + num_put_jobs,))  #注意args里面要把q对象传给我们要执行的方法，这样子进程才能和主进程用Queue来通信
        p.start()
        process_list.append(p)

    for p in process_list:
        p.join()
    
    print("Done.")

```
这里只是一个例程，包含了子进程的条件退出，这个条件需要根据put jobs和get jobs的数量来做一点调整，如果不加这个条件的ge子进程会一直处于等待状态，不会退出。处理这种条件退出，还可以设置等待时间等条件退出。

Manager模版：
```python

from multiprocessing import Process, Manager

def fun1(dic,lis,index):

    dic[index] = 'a'
    dic['2'] = 'b'    
    lis.append(index)    #[0,1,2,3,4,0,1,2,3,4,5,6,7,8,9]
    #print(l)

if __name__ == '__main__':
    with Manager() as manager:
        dic = manager.dict()#注意字典的声明方式，不能直接通过{}来定义
        l = manager.list(range(5))#[0,1,2,3,4]

        process_list = []
        for i in range(10):
            p = Process(target=fun1, args=(dic,l,i))
            p.start()
            process_list.append(p)

        for res in process_list:
            res.join()
        print(dic)
        print(l)

```
Manager还包含了一些其他数据类型，可以读源码来尝试其他的数据类型。

python多进程除了可以调用一些自己的函数或者模块，还可以利用subprocess模块调用已有的可执行文件，具体教程可以参考[[python外部子进程subprocess]]