## 基础知识 - 多线程
> 对于 `python` 爬虫这种 IO 密集型任务来说，多线程可以很大程度的提升爬虫的爬取速度

### 直接使用 `threading.Thread` 类
```python
import time
import threading
def target(second):
    print(f'Threading {threading.current_thread().name} is running')
    print(f'Threading {threading.current_thread().name} sleep {second} s')
    time.sleep(second)
    print(f'Threading {threading.current_thread().name} is ended')

# 使用 threading.Thread 创建多线程
def test_thread():
    print(f'Threading {threading.current_thread().name} is running')
    for i in [1,5]:
        t = threading.Thread(target=target, args=[i])
        t.start()
        # t.join() # 调用后 主线程 需要等待子线程结束才会结束
        # t.join(1) # 设置超时时间，超过该时间后不等待 
        # t.setDaemon() # 设置为守护进程，主线程退出了就退出了
    print(f'Threading {threading.current_thread().name} is ended')
```

### 继承 `Thread` 类 

```python
import time
import threading
class Th(threading.Thread):
    def __init__(self):
        threading.Thread.__init__(self)

    # 这里是为了理解多线程的共享问题
    # 以为取的时候可能取到了同一个count
    # 赋值的时候 count 可能已经改变
    # 为了防止这种情况，共享数据需要加锁
    def run(self):
        global count
        lock.acquire()      # 请求锁
        temp = count + 1
        time.sleep(0.0001)
        count = temp
        lock.release()      # 释放锁

def test_thread_2():
    threads = []
    for _ in range(10):
        th = Th()
        th.start()
        threads.append(th)
    for th in threads:
        th.join()
    print(f'Final count: {count}')
```

### 基础知识 - 多进程
> 由于 GIL 全局解释器锁的存在，`python` 中一个进程同时只能运行一个进程，即是并发的，并非并行的
> 而在多进程可以同时调用多个核并行运行进程，充分发挥多核的优势
> 由于爬虫是 IO 密集型任务，而非计算密集型任务，多进程提升较小
> 同时因为是不同进程，不可以直接共享变量

### 直接使用 `multiprocessing.Process` 类

```python
import time
import multiprocessing as mp

def process(index):
    time.sleep(index)
    print(f'Process: {index}')

def test_1():
    for i in range(5):
        # target : 进程执行的方法
        # args: 方法的参数，
        # 如果是个元组，只有一个参数需要加一个逗号
        # p = mp.Process(target=process, args=(i, ))
        # 可以用列表得方式传参
        p = mp.Process(target=process, args=[i])
        p.start()
    print(f'CPU number: {mp.cpu_count()}')  # 打印 CPU 核心数(可以并行的进程数)
    for p in mp.active_children():          # mp.active_childern() 是当前在运行得进程
        print(f'Child process name: {p.name} id: {p.pid}')
    print('Process Ended')
```

### 继承 `Process` 类

```python
class Pr(mp.Process):
    def __init__(self, loop):
        mp.Process.__init__(self)
        self.loop = loop

    def run(self):
        for count in range(self.loop):
            time.sleep(1)
            print(f'Pid: {self.pid} LoopCount: {count}')

def test_2():
    for i in range(2, 5):
        p = Pr(i)
        # p.daemon = True # 设置为守护进程，父进程退出子进程也会退出
        p.start()
        p.join()          # 设置父进程等待子进程结束才结束
        # p.join(1)          # 设置超时时间，如果超过该时间，则父进程不等待该子进程, 单位秒
```

### 进程的终止

```python
def target(second):
    print(f'Threading {threading.current_thread().name} is running')
    print(f'Threading {threading.current_thread().name} sleep {second} s')
    time.sleep(second)
    print(f'Threading {threading.current_thread().name} is ended')
# 进程的终止
print('--------------------------------------')
p = mp.Process(target=target, args=[1])
print('Before:', p, p.is_alive())       # p.is_alit() == False
p.start()
print('During:', p, p.is_alive())       # True
p.terminate()
print('Terminate:', p, p.is_alive())    # True
# terminate 后 使用 join 才会刷新状态
p.join()
print('Joined:', p, p.is_alive())       # False
```

### 并行互斥锁
> 使用多进程运行 `pring('xxx')` 会出现下面的情况

```cmd
# 假设是2个进程
xxx\n
xxx\n
# 或者
xxxxxx
\n
\n
```

> 因为是共用了同一个窗口进行打印，当一个进程还没来得及输出换行时，另一个进程打印出了他的结果而出现上面这种情况
> 资源出现竞争，可以通过加锁的方式解决

```python
from multiprocessing import Process, Lock
class Pr(Process):
    def __init__(self, loop, lock):
        Process.__init__(self)
        self.loop = loop
        self.lock = lock

    def run(self):
        for count in range(self.loop):
            time.sleep(1)
            self.lock.acquire()  # 申请锁
            # 进程互斥锁保证其他进程不会在这个时候打印
            print(f'Pid: {self.pid} LoopCount: {count}')
            self.lock.release()  # 释放锁

def test_2():
    lock = Lock()
    for i in range(2, 5):
        p = Pr(i, lock)
        p.start()
```

### 信号量 - 生产者问题

> 信号量 控制临界资源的数量
> 实现多个进程同时共享资源，限制进程的并发量
> 使用 `multiprocessing.Semaphore` 来实现

```python
from multiprocessing import Process

```