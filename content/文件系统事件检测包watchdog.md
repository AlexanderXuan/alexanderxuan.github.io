Title: python文件系统监测watchdog
Date: 2022-01-16 17:06
Category: python小技巧
Tags: python, watchdog
Author: Alexander Xuan
Summary: python使用watchdog监测文件变动

watchdog是用于监测文件系统的包，可以监测文件的变动或者文件夹的变动，包含创建，删除，修改等，可以在相应动作发生后执行一些操作。

例程如下：
```python
import os
import sys
import time
import logging
import shutil
from tqdm import tqdm
from multiprocessing import Process, Queue
from watchdog.observers import Observer
from watchdog.events import FileSystemEventHandler


class FileEventHandler(FileSystemEventHandler):
    def __init__(self, queue):
        FileSystemEventHandler.__init__(self)
        self.queue = queue

    def on_moved(self, event):
        pass

    def on_created(self, event):
        if event.is_directory:
            pass
        else:
            self.queue.put(event.src_path)
            print(f'put {event.src_path} in queue')

    def on_deleted(self, event):
        pass

    def on_modified(self, event):
        pass


def get_new_file(rank_id, queue, ali_dir):
	exist_files = os.listdir(ali_dir) # 添加已有文件到队列
    for exist_file in exist_files:
        file_path = os.path.join(ali_dir, exist_file)
        queue.put(file_path)
    observer = Observer()
    event_handler = FileEventHandler(queue)
    observer.schedule(event_handler, ali_dir, False)
    observer.start()
    try:
        while True:
            pass
    except KeyboardInterrupt:
        observer.stop()
    observer.join()


def process_new_file(rank_id, queue, bkp_dir=None):
    while True:
        audio_path = queue.get()
        print(f'{rank_id} {audio_path} finished ...')
        shutil.move(audio_path, bkp_dir)
        # print(f'{rank_id} {audio_path} finished ...')


if __name__ == "__main__":
    ali_dir = sys.argv[1]
    bkp_dir = sys.argv[2]

    num_jobs = 4

    queue = Queue(100)
    
    process_list = []

    p = Process(target=get_new_file, args=(0, queue, ali_dir,))
    p.start()
    process_list.append(p)

    for i in range(num_jobs):
        p = Process(target=process_new_file, args=(i + 1, queue, bkp_dir,))
        p.start()
        process_list.append(p)

    for p in process_list:
        p.join()

    os.system('stty echo')
    print("Done.")

```
例程中包含了多进程的相关内容，具体可参考[[python多进程]]。
