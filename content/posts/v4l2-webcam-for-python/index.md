+++
title = '在 Linux 下使用 V4L2 和 Python 操作 webcam'
date = '2024-05-01T18:28:53Z'
toc = true
categories = ['编程']
tags = ['linux', 'python']
+++

这篇博客介绍的是如何用 Python 创建对 V4L2 的封装，以实现摄像头的打开、关闭、读取等操作。

<!--more-->

## V4L2 for Python
[V4L2](https://zh.wikipedia.org/Video4Linux) 是 Linux 的一个驱动程序框架，用来驱动摄像头等设备。

所有的 V4L2 api 都可以通过 [ioctl](https://www.man7.org/linux/man-pages/man2/ioctl.2.html) 来访问，我们只需要把相关的头文件里面的枚举、结构体和常量用 Python 全部声明一遍就可以了。

### 实现 ioctl
尽管 Python 标准库有一个 [fcntl](https://docs.python.org/zh-cn/3/library/fcntl.html) 模块，该模块提供了 `ioctl` 函数。但这是远远不够的，我们还需要 `linux/ioctl.h` 头文件里的函数来生成调用 `ioctl` 所需的 `request` 参数。

下面的代码实现了这些函数，同时对 `_IOR`、`_IOW` 和 `_IOWR` 进行了封装。

```python
from ctypes import sizeof
from fcntl import ioctl

_IOC_NRBITS = 8
_IOC_TYPEBITS = 8
_IOC_SIZEBITS = 14
_IOC_DIRBITS = 2

_IOC_NRSHIFT = 0
_IOC_TYPESHIFT = _IOC_NRSHIFT + _IOC_NRBITS
_IOC_SIZESHIFT = _IOC_TYPESHIFT + _IOC_TYPEBITS
_IOC_DIRSHIFT = _IOC_SIZESHIFT + _IOC_SIZEBITS

_IOC_NONE = 0
_IOC_WRITE = 1
_IOC_READ = 2


def _IOC(dir, type, nr, size):
    return (
        (dir << _IOC_DIRSHIFT)
        | (type << _IOC_TYPESHIFT)
        | (nr << _IOC_NRSHIFT)
        | (size << _IOC_SIZESHIFT)
    )


def _IOR(type, nr, struct):
    request = _IOC(_IOC_READ, ord(type), nr, sizeof(struct))

    def f(fileno):
        buffer = struct()
        ioctl(fileno, request, buffer)
        return buffer

    return f


def _IOW(type, nr, struct):
    request = _IOC(_IOC_WRITE, ord(type), nr, sizeof(struct))

    def f(fileno, buffer):
        if not isinstance(buffer, struct):
            buffer = struct(buffer)
        ioctl(fileno, request, buffer)

    return f


def _IOWR(type, nr, struct):
    request = _IOC(_IOC_READ | _IOC_WRITE, ord(type), nr, sizeof(struct))

    def f(fileno, buffer):
        ioctl(fileno, request, buffer)
        return buffer

    return f

```

### 重写头文件
我们需要用 Python 重写以下 2 个头文件，包括部分用得到的枚举、结构体和常量。

1. `linux/videodev2.h`
2. `linux/v4l2-controls.h`

首先，导入所需模块。`_s32` 表示头文件中的 `__s32` 类型，以此类推。

```python
import ctypes
from ctypes import c_int32 as _s32
from ctypes import c_int64 as _s64
from ctypes import c_uint8 as _u8
from ctypes import c_uint16 as _u16
from ctypes import c_uint32 as _u32
from ctypes import c_void_p
from enum import IntEnum
```

C 中的枚举使用继承自 `IntEnum` 的类来表示。

```python
# /usr/include/linux/videodev2.h:139
class v4l2_buf_type(IntEnum):
    V4L2_BUF_TYPE_VIDEO_CAPTURE = 1
    V4L2_BUF_TYPE_VIDEO_OUTPUT = 2
    V4L2_BUF_TYPE_VIDEO_OVERLAY = 3
    V4L2_BUF_TYPE_VBI_CAPTURE = 4
    V4L2_BUF_TYPE_VBI_OUTPUT = 5
    V4L2_BUF_TYPE_SLICED_VBI_CAPTURE = 6
    V4L2_BUF_TYPE_SLICED_VBI_OUTPUT = 7
    V4L2_BUF_TYPE_VIDEO_OUTPUT_OVERLAY = 8
    V4L2_BUF_TYPE_VIDEO_CAPTURE_MPLANE = 9
    V4L2_BUF_TYPE_VIDEO_OUTPUT_MPLANE = 10
    V4L2_BUF_TYPE_SDR_CAPTURE = 11
    V4L2_BUF_TYPE_SDR_OUTPUT = 12
    V4L2_BUF_TYPE_META_CAPTURE = 13
    V4L2_BUF_TYPE_META_OUTPUT = 14
    V4L2_BUF_TYPE_PRIVATE = 0x80  # deprecated
```

C 中的结构体用 `ctypes.Structure` 构造。

```python
# /usr/include/linux/videodev2.h:435
class v4l2_capability(ctypes.Structure):
    _fields_ = [
        ("driver", _u8 * 16),
        ("card", _u8 * 32),
        ("bus_info", _u8 * 32),
        ("version", _u32),
        ("capabilities", _u32),
        ("device_caps", _u32),
        ("reserved", _u32 * 3),
    ]
```

`timeval` 结构体是在别处声明的，需要特别注意！

```python
class timeval(ctypes.Structure):
    _fields_ = [
        ("tv_sec", _s64),
        ("tv_usec", _s64),
    ]
```

引用自身的结构体需要做特殊处理，也请注意！

```python
# /usr/include/linux/videodev2.h:1183
class v4l2_clip(ctypes.Structure):
    pass


v4l2_clip._fields_ = [
    ("c", v4l2_rect),
    ("next", ctypes.POINTER(v4l2_clip)),
]
```

如果你想亲自动手完成以上所有枯燥且乏味的操作是非常好的。如果你很懒，作者在写这篇博客前就已经写好了博客中所有用得到的代码，所有的文件在 [GitHub](https://github.com/zhengxyz123/webcam) 上都可以找到。

## 操作摄像头
Linux 中“一切皆文件”，摄像头设备可以通过 `/dev/videoX`（`X` 是从 0 到 63 的整数）文件来访问。通常我们的电脑上只有一个摄像头，使用 `/dev/video0` 就可以了。

### 检查摄像头
第一步，打开设备。

```python
import ctypes, mmap, os

fd = os.open("/dev/video0", os.O_RDWR)
```

第二步，用 `VIDIOC_QUERYCAP` 检查设备是否支持视频捕获和流式读取。

```python
cap = VIDIOC_QUERYCAP(fd)
if not cap.capabilities & V4L2_CAP_VIDEO_CAPTURE:
    raise RuntimeError("can not capture video")
if not cap.capabilities & V4L2_CAP_STREAMING:
    raise RuntimeError("does not support streaming")

# 打印驱动名称
print("driver:", ctypes.string_at(cap.driver).decode())
# 打印设备名称
print("card:", ctypes.string_at(cap.card).decode())
```

第三步，获取设备支持的输出格式。由于不可能一股脑儿全部得到，所以使用循环来获取所有支持的格式。

```python
available_pixfmt = []
fmt = v4l2_fmtdesc(
    index=0,
    type=v4l2_buf_type.V4L2_BUF_TYPE_VIDEO_CAPTURE,
)
while True:
    try:
        VIDIOC_ENUM_FMT(fd, fmt)
    except OSError:
        break
    fmt.index += 1
    available_pixfmt.append(fmt.pixelformat)
```

第四步，设置设备的输出格式。这里我们假装设备支持 `MJPEG` 格式。

```python
fmt = v4l2_format(type=v4l2_buf_type.V4L2_BUF_TYPE_VIDEO_CAPTURE)
fmt.fmt.pix.width = 640
fmt.fmt.pix.height = 480
fmt.fmt.pix.pixelformat = V4L2_PIX_FMT_MJPEG
VIDIOC_S_FMT(fd, vfmt)

# 我们并不能确保设备支持 640x480 的分辨率
# 所以需要在设置格式后再确认一遍
size = (fmt.fmt.pix.width, fmt.fmt.pix.height)
```

### 申请并映射帧缓冲队列
首先，我们必须要向内核申请缓冲队列，并指定个数和类型。

```python
reqbuf = v4l2_requestbuffers(
    # 申请 4 个缓冲区
    count=4,
    type=v4l2_buf_type.V4L2_BUF_TYPE_VIDEO_CAPTURE,
    # 缓冲区的类型为内存映射
    memory=v4l2_memory.V4L2_MEMORY_MMAP,
)
VIDIOC_REQBUFS(fd, reqbuf)
```

之后逐个映射缓冲区。

```python
mmap_list = []
for i in range(reqbuf.count):
    buffer = v4l2_buffer(
        index=i, type=reqbuf.type, memory=v4l2_memory.V4L2_MEMORY_MMAP
    )
    # 查询缓冲信息
    VIDIOC_QUERYBUF(fd, buffer)
    # 放入缓冲区队列
    VIDIOC_QBUF(fd, buffer)
    mmap_list.append(
        mmap.mmap(fd, length=buffer.length, offset=buffer.m.offset)
    )
```

### 采集视频
首先要打开设备。

```python
VIDIOC_STREAMON(fd, v4l2_buf_type.V4L2_BUF_TYPE_VIDEO_CAPTURE)
```

之后才能获取图像。

```python
buffer = v4l2_buffer(
    type=v4l2_buf_type.V4L2_BUF_TYPE_VIDEO_CAPTURE,
    memory=v4l2_memory.V4L2_MEMORY_MMAP,
)
# 从缓冲区获取并保存一帧图像
VIDIOC_DQBUF(fd, buffer)
image = mmap_list[buffer.index].read(buffer.length)
with open("image.jpg", "wb") as f:
    f.write(image)
# 若要读取多帧图像，一定要重置文件的读取位置
mmap_list[buffer.index].seek(0)
# 将缓冲区放回缓冲队列
VIDIOC_QBUF(fd, buffer)
```

所有都完成后可以关闭设备，设备关闭后缓冲区会全部清空。

```python
VIDIOC_STREAMOFF(fd, v4l2_buf_type.V4L2_BUF_TYPE_VIDEO_CAPTURE)
for m in mmap_list:
    m.close()
os.close(fd)
```
