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
# 个人尝试不知道为啥，full.acquire() 这里一直请求不到
from logging import PercentStyle
from multiprocessing import Process, Semaphore, Lock, Queue, current_process
import time
import sys

buffer = Queue(10)
empty = Semaphore(2)
full = Semaphore(0)
lock = Lock()

class Consumer(Process):
    def run(self):
        global buffer, empty, full, lock
        while True:
            print(f'{self.name} full acquire')
            full.acquire()
            print(f'{self.name} lock acquire')
            lock.acquire()
            print(f'Consumer get {buffer.get()}')
            time.sleep(1)
            print(f'{self.name} lock release')
            lock.release()
            print(f'{self.name} empty release')
            empty.release()

class Producer(Process):
    def run(self):
        global buffer, empty, full, lock
        while True:
            print(f'{self.name} empty acquire')
            empty.acquire()
            print(f'{self.name} lock acquire')
            lock.acquire()
            print('Producer put 1')
            buffer.put(1)
            time.sleep(1)
            print(f'{self.name} lock release')
            lock.release()
            print(f'{self.name} full release')
            full.release()

if __name__ == '__main__':
    p = Producer()
    c = Consumer()
    p.daemon = c.daemon = True
    p.start()
    c.start()
    p.join()
    c.join()
    print('Main Process Ended')
```

### 进程资源共享
上述示例中, 不能用queue.Queue, 一定要用 multiprocessing.Queue, 因为进程之间的资源不共享
 - multiprocessing.Queue: 队列, 进程资源共享
 - multiprocessing.Pipe:  管道, 进行手法消息(deplex = False 为单向管道)
 ```python
from multiprocessing import Process, Pipe

class Consumer(Process):
    def __init__(self, pipe):
        Process.__init__(self)
        self.pipe = pipe
    
    def run(self):
        self.pipe.send('Consumer Words')
        print(f'Consumer Received: {self.pipe.recv()}')

class Producer(Process):
    def __init__(self, pipe):
        Process.__init__(self)
        self.pipe = pipe
    
    def run(self):
        print(f'Consumer Received: {self.pipe.recv()}')
        self.pipe.send('Prodecer Words')

if __name__ == '__main__':
    pipe = Pipe()
    p = Producer(pipe[0])
    c = Consumer(pipe[1])
    p.daemon = c.daemon = True
    p.start()
    c.start()
    p.join()
    c.join()
    print('Main Ended')
 ```

### 进程池
```python
from multiprocessing import Pool
import time

def function(index):
    print(f'Start process: {index}')
    time.sleep(3)
    print(f'End process {index}')

if __name__ == '__main__':
    # 不指定参数时会按照CPU核心来决定数量
    pool = Pool(processes=3)
    for i in range(20):
        pool.apply_async(func=function, args=[i])

    print('Main Process Started')
    pool.close()
    pool.join()
    print('Main Process Ended')
```

#### `Map` 方法

`Map` 方法会将一个可迭代对象依次传入指定方法执行

```python
from multiprocessing import Pool
import urllib.request
import urllib.error

def scrape(url):
    try:
        urllib.request.urlopen(url)
        print(f'URL {url} Scraped')
    except (urllib.error.HTTPError, urllib.error.URLError):
        print(f'URL {url} not Scraped')

if __name__ == '__main__':
    pool = Pool(processes=2)
    urls = [
        'https://www.baidu.com',
        'https://wallhaven.cc',
        'http://www.posase.im',
        'https://github.com'
    ]
    pool.map(scrape, urls)
    pool.close()

```


## 爬虫入门

### `Requests` 库的使用

**安装**: `pip install requests`

#### 发起请求

```python
import requests

# Get 请求
r = requests.get('http://httpbin.org/get')
print(r.text)

# 带参数的 Get
r = requests.get('http://httpbin.org/get?name=posase&sex=male')

# 使用 params 参数
data = {'name': 'posase', 'sex' : 'male'}
r = requests.get('http://httpbin.org/get', params=data)

# Post 请求
# 注意 Post 的参数使用 data 字段传递
data = {'name': 'posase', 'sex' : 'male'}
r = requests.post('http://httpbin.org/post', data=data)
print(r.text)
```

#### `Response` 响应数据

**常见字段**
| 字段名 | 说明 |
| --- | --- |
| `status_code` | 状态码 |
| `headers` | 响应头 |
| `cookies` | Cookies |
| `url` | 请求的 Url |
| `history` | 请求历史 |
| `encoding`| 编码格式 |

```python
# 内置状态码 其实就是 200
if r.status_code == requests.codes.ok
    print('Request Successfully')
```

#### 解析数据

```python
# 该网站返回的都是 json 格式的数据
# 直接调用 json 方法可以进行解析数据
print(type(r.text))     # <class 'str'>
print(r.json())         # json to dict {'x' : 'y'}
print(type(r.json()))   # <class 'dict'>

# 使用正则表达式获取我们想要的内容
import re
import requests as req
url = 'https://www.xbiquge.la'
r = req.get(url)
pa = re.compile(r'<dt><span>(.*)</span>.*>(.*)</a>')
res = re.findall(pa, r.text)
print(res)

# 二进制数据 图片，视频，音乐等
r = requests.get('https://github.com/favicon.ico')
print(r.content) # b'\x00\x00....'
with open('favicon.ico', 'wb') as f:
    f.write(r.content) # 保存为一个文件
```

> `Python re` 模块[官方文档](https://docs.python.org/zh-cn/3/library/re.html)

#### 设置 `Headers`

> 某些网站会过滤 `Headers` 不符合要求的请求

```python
# 比如设置 User-Agent
headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.121 Safari/537.36'
}
r = requests.get('https://www.baidu.com', headers=headers)
print(r.text)
```

#### 文件上传

```python
import requests
files = {'file': open('favicon.ico', 'rb')}
r = requests.post('http://httpbin.org/post', files=files)
print(r.text)
```

#### 设置 `Cookies`
```python
# 打印 Cookies
import requests
r = requests.get('http://www.baidu.com')
print(r.cookies)
for key, val in r.cookies.items():
    print(f'{key} = {val}')

# 设置 Cookies 模拟登录
headers = {
    'Cookie': 'xxxx'
}
r = requests.get('xxx', headers=headers)

# 构造 Cookies 对象
cookies = 'xxxx'
jar = requests.cookies.RequestsCookieJar()
for cookie in cookies.split(';'):
    key, val = cookie.split('=', 1)
    jar.set(key, val)
r = requests.get('xxx', cookies=jar)
```

#### `Session` 维持

```python
import requests
s = requests.Session()
s.get('xxx')
r = s.get('xxx')
```

#### `SSL` 验证

```python
# 不验证 SSl 证书，防止不受信任的证书无法访问
import logging
import requests
from requests.packages import urllib3
res = requests.get('xxx', verify=False)

# 但是还是会有提示警告 要在请求前使用
urllib3.disable_warnings() # 设置忽略警告(可选)
# 或者日志记录警告
logging.captureWarnings(True)
# 也可以指定本地证书 可以是单个文件，或者两个文件路径的元组
res = requests.get('xx', cert=('x.crt', 'y.key'))

```

#### 超时检测
```python
# 默认永久等待(None)
# 连接+读取 1 秒内没响应则异常
requests.get('xx', timeout=1)
# 连接超时时间 1s，读取超时时间 2s
requests.get('xx', timeout=(1, 2))
```

#### 身份认证
```python
# 密码错误 返回 401
from requests.auth import HTTPBasicAuth
r = requests.get('xx', auth=HTTPBasicAuth('user', 'password'))
# 可以直接简写
r = requests.get('xx', auth=('user', 'password'))

# 其他身份认证 OAuth 认证
# 需要安装  pip install requests_oauthlib
from requests_oauthlib import OAuth1
auth = OAuth1('key', 'key_secret','token', 'toke_secret')
requests.get('xx', auth=auth)
```
> 更多参考[官方文档](https://requests-oauthlib.readthedocs.org/)

#### 代理设置
```python
proxies = {
    'http': 'http://xxxxx',
    'https': 'http://xxx'
    # 需要账号密码 user:pd@host
    'http': 'http://user:password@0.0.0.0:88',
    # 还支持 Socks 代理，需要安装 pip install "requests[socks]"
    'http': 'socks5://xxxx'
}
req.get('xx', proxies=proxies)
```