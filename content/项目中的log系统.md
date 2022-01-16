Title: python的log系统
Date: 2022-01-16 17:03
Category: python小技巧
Tags: python, logging
Author: Alexander Xuan
Summary: python中的logging在项目中的使用

#### python的自带日志记录系统logging
在一个较为完备的项目中，有时需要日志来进行程序调试或者需要打印一些运行内容来进行后续的程序运行性能分析。所以就需要一个比较方便的日志记录系统来打印出风格统一，内容详实的日志。python项目中可以使用python自带的logging包，只要设置好后续的使用非常简单。

##### 需求
我这里的项目是多个文件构成的，希望只设置一次logging的格式等，后续就可以在每个文件中使用相同的设置，并且不想每个接口都传递一个logger类，增加接口的复杂度。

##### 实现方式
在项目中添加一个单独的log配置文件，这里命名为logger.py，然后在文件中设置全局想设置的内容，例如：
```python

import logging  
import os   
import time


def init_log(  
    level=logging.DEBUG,  
    logdir=None,  
    filename=None,
    enable_fileout=False,  
    enable_stdout=False,  
):  
    if enable_fileout:
        if filename is None:  
            filename = 'asr_{}_{}.log'.format(  
                os.getpid(),  
                time.strftime('%Y-%m-%d-%H-%M-%S', time.localtime(time.time())))  
    
        if logdir:  
            os.makedirs(logdir, exist_ok=True) 
            filename = os.path.join(logdir, filename)  
    
        logging.basicConfig(  
            filename=filename,  
            filemode='w',  
            format='%(asctime)s [%(levelname)s] %(name)s: %(message)s',  
            datefmt='%Y-%m-%d %H:%M:%S',  
            level=level)  
  
    if enable_stdout:  
        console = logging.StreamHandler()  
        formatter = logging.Formatter(  
            '%(asctime)s [%(levelname)s] %(name)s: %(message)s')  
        console.setFormatter(formatter)  
        log = logging.getLogger()  
        log.addHandler(console)  
        log.setLevel(level)
		
		
```

这里配置了两种输出，一种是输出到log文件中，另一种是输出到控制台，当然也可以两种都输出。详细的配置的话就是配置了信息的输出格式，时间的输出格式等。

在运行程序时先导入这个接口，然后先调用这个初始化函数，其他文件中直接添加下面的代码就可以统一输出设置的格式：

```python

import logging

logger = logging.getLogger(__name__)

logger.info("test info.")

```

需要注意的是输出log的级别设置，只有比设置的log级别高的log才会打印出来，log的级别如下：
-   FATAL/CRITICAL = 重大的，危险的
-   ERROR = 错误
-   WARNING = 警告
-   INFO = 信息
-   DEBUG = 调试

程序中的写法如下：
```python

import logging


logging.debug("This is a debug log.")

logging.info("This is a info log.")

logging.warning("This is a warning log.")

logging.error("This is a error log.")

logging.critical("This is a critical log.")

```
或者也可以如下写：
```python

logging.log(logging.DEBUG, "This is a debug log.")

logging.log(logging.INFO, "This is a info log.")

logging.log(logging.WARNING, "This is a warning log.")

logging.log(logging.ERROR, "This is a error log.")

logging.log(logging.CRITICAL, "This is a critical log.")

```