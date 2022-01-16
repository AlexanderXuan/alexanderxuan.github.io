Title: python外部子进程subprocess
Date: 2022-01-16 17:05
Category: python小技巧
Tags: python, subprocess
Author: Alexander Xuan
Summary: python使用subprocess调用外部模块

#### subprocess模块
python的子进程可以调用python写的函数，但是有的时候有一些封装好的可执行文件（或者说一些命令），我们想用多进程的方式来调用这些外部的程序（多进程教程可以参考[[python多进程]]），希望可以传参给这些程序，获得其返回内容，返回码或者错误信息。subprocess就是一个可以实现这个功能的模块，我曾看过pydub的一小部分源码，就是用这个模块来调用ffmpeg进行音频操作的，有时候一些工具不好用或者出错可以自己用这个模块来改动或者重写。

##### 需求
希望利用python多进程的方式实现外部已有程序的调用，并且可以传参和接收返回的内容。

##### 实现方式
这里用一个使用ffmpeg读取和写出aac文件的程序来做演示，pydub之前这个部分是有问题的，目前还没有查看是否修正。
```python
import os
import sys
import subprocess
import struct
import numpy as np


def read_aac(aac_file):
    read_command = f"ffmpeg -i {aac_file} -acodec pcm_s16le -ac 1 -f s16le -"
    p = subprocess.Popen(read_command,
                         stdout=subprocess.PIPE, stderr=subprocess.PIPE)
    p_out, p_err = p.communicate()
    if p.returncode != 0 or len(p_out) == 0:
        raise Exception(
            "Decoding failed. ffmpeg returned error code: {0}\n\nOutput from ffmpeg/avlib:\n\n{1}".format(
                p.returncode, p_err.decode(errors='ignore')))
    data_len = len(p_out) // 2
    data = struct.unpack(f'<{data_len}h', p_out)
    return np.array(data, dtype=np.int16)


def write_aac(data, out_aac_file):  # data is a array include int16 numbers
    t_data = tuple(data)
    p_in = struct.pack(f"<{len(t_data)}h", *t_data)
    sr = 48000  # this can be a param
    ac = 1      # this can be a param
    write_command = f"ffmpeg -f s16le -ar {sr} -ac {ac} -i - -acodec aac -ab 128k -y {out_aac_file}"
    p = subprocess.Popen(write_command, stdin=subprocess.PIPE,
                         stdout=subprocess.PIPE, stderr=subprocess.PIPE)
    p_out, p_err = p.communicate(input=p_in)
    if p.returncode != 0:
        raise Exception(
            "write failed. ffmpeg returned error code: {0}\n\nOutput from ffmpeg/avlib:\n\n{1}".format(
                p.returncode, p_err.decode(errors='ignore')))
    return data


if __name__ == "__main__":
    input_file = r'test_aac.aac'
    output_file = r'out_aac.aac'
    data = read_aac(input_file)
    write_aac(data, output_file)
    print("Done.")

```
此外除了可执行文件，一些其他语言写的脚本例如shell，perl等也可以用这种方式来调用。