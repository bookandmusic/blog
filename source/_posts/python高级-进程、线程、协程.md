---
title: python多任务机制
date: 2019-05-17 20:53:32
categories:
  - 技术
  - python
  - 系统
tags:
  - 线程
  - 进程
  - 协程
  - 多任务
---

### python 线程与进程简介

#### 1. 进程与线程的历史

我们都知道计算机是由**硬件**和**软件**组成的。

- 硬件中的 CPU 是计算机的核心，它承担计算机的所有任务。
- 操作系统是运行在硬件之上的软件，是计算机的管理者，它负责资源的管理和分配、任务的调度。
- 程序是运行在系统上的具有某种功能的软件，比如说浏览器，音乐播放器等。

每次执行程序的时候，都会完成一定的功能，比如说浏览器帮我们打开网页，为了保证其独立性，就需要一个专门的管理和控制执行程序的数据结构——**进程控制块**。

进程就是一个程序在一个数据集上的一次动态执行过程。

- 进程一般由程序、数据集、进程控制块三部分组成。
  - 我们编写的程序用来描述进程要完成哪些功能以及如何完成；
  - 数据集则是程序在执行过程中所需要使用的资源；
  - 进程控制块用来记录进程的外部特征，描述进程的执行变化过程，系统可以利用它来控制和管理进程，它是系统感知进程存在的唯一标志。

- 在早期的操作系统里，计算机只有一个核心，进程是执行程序的最小单位，任务调度采用时间片轮转的抢占式方式进行进程调度。
- 每个进程都有各自的一块独立的内存，保证进程彼此间的内存地址空间的隔离。

随着计算机技术的发展，进程出现了很多弊端：

- 一是进程的创建、撤销和切换的开销比较大
- 二是由于对称多处理机（对称多处理机（SymmetricalMulti-Processing）又叫 SMP，是指在一个计算机上汇集了一组处理器 (多 CPU)，各 CPU 之间共享内存子系统以及总线结构）的出现，可以满足多个运行单位，而多进程并行开销过大。

这个时候就引入了线程的概念。

- 线程也叫轻量级进程，它是一个基本的 CPU 执行单元，也是程序执行过程中的最小单元，由线程 ID、程序计数器、寄存器集合 和堆栈共同组成。
- 线程的引入减小了程序并发执行时的开销，提高了操作系统的并发性能。
- 线程没有自己的系统资源，只拥有在运行时必不可少的资源。
- 但线程可以与同属与同一进程的其他线程共享进程所拥有的其他资源。

#### 2.  进程与线程之间的关系

- 线程是属于进程的，线程运行在进程空间内，同一进程所产生的线程共享同一内存空间，当进程退出时该进程所产生的线程都会被强制退出并清除。
- 线程可与属于同一进程的其它线程共享进程所拥有的全部资源，但是其本身基本上不拥有系统资源，只拥有一点在运行中必不可少的信息 (如程序计数器、一组寄存器和栈)。

#### 3. 守护线程与守护进程的区别

> 强调：运行完毕，并非是终止

- 线程与进程运行完毕的区别：

  - 主进程运行完毕指的是主进程代码运行完毕

  - 主线程运行完毕指的是所在的进程内的所有非守护线程运行完毕后，主线程才算运行完毕

- 守护进程：主进程代码运行完毕，守护进程也就结束 **(守护的是主进程)**

  - 主进程要等非守护进程都运行完毕后再回收子进程的资源（否则会产生僵尸进程）才结束

  - 主进程等子进程是因为主进程要给子进程收尸（代用wait方法向操作系统发起回收资源信号（pid号，状态信息））

- 守护线程：非守护线程代码运行完毕，守护线程也就结束 **(守护的是非守护线程)**

  - 主线程在其他非守护线程运行完毕后才算结束（守护线程在此时就会被回收）

  - 强调：主线程也是非守护线程（进程包含了线程）

> 总结:主线程的结束意味着进程结束，进程整体的资源都会被回收，而进程必须保证非守护线程都运行完毕后才能结束

- 守护进程：主进程代码运行完毕，守护进程也就结束
- 守护线程：非守护线程运行完毕，守护线程结束

### python 线程

Threading 用于提供线程相关的操作，线程是应用程序中工作的最小单元。

#### 1、threading 模块

threading 模块建立在 _thread 模块之上。thread 模块以低级、原始的方式来处理和控制线程，而 threading 模块通过对 thread 进行二次封装，提供了更方便的 api 来处理线程。

```python
import threading
import time


def worker(num):
    """
    thread worker function
    :return:
    """
    time.sleep(3)
    print("The num is  %d" % num)
    return


if __name__ == '__main__':
    for i in range(5):
        t = threading.Thread(target=worker, args=(i,), name="t.%d" % i)
        t.start()

```

上述代码创建了5个"前台"线程，然后控制器就交给了CPU，CPU根据指定算法进行调度，分片执行指令。

> Thread参数介绍：

- group：参数未使用，默认值为None。

- target：表示调用对象，即子线程要执行的任务。

- args：表示调用的位置参数元组。

- kwargs：表示调用对象的字典。如kwargs = {'name':Jack, 'age':18}。

- name：子进程名称。

- daemon：设置为守护线程(True)或非守护线程(默认：False)

>Thread属性方法说明

|  方法/属性  | 说明                                                                                                                                                                                                             |
| :---------: | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|   start()   | 激活线程。使用该方法启动一个子线程，线程名就是我们定义的name，想启动多线程，就必须使用此方法                                                                                                                     |
|    run()    | 线程被cpu调度后自动执行线程对象的run方法,直接使用该方法并不启动一个新线程，就是在主线程中调用了一个普通函数而已                                                                                                  |
|    ident    | 获取线程的标识符。线程标识符是一个非零整数，只有在调用了start()方法之后该属性才有效，否则它只返回None。                                                                                                          |
|    name     | 获取或设置线程的名称                                                                                                                                                                                             |
|  getName()  | 获取线程的名称                                                                                                                                                                                                   |
|  setName()  | 设置线程的名称                                                                                                                                                                                                   |
| is_alive()  | 判断线程是否为激活状态                                                                                                                                                                                           |
|  isAlive()  | 判断线程是否为激活状态                                                                                                                                                                                           |
| setDaemon() | 设置为守护线程或非守护线程；默认值为False，如果设置为True，代表该进程为后台守护线程；当非守护线程(不只是主线程)终止时，该线程也随之终止；并且设置为True后，该线程不能创建子线程，设置该属性设置必须在start()之前 |
| isDaemon()  | 判断是否为守护线程                                                                                                                                                                                               |
|   join()    | 阻塞等待该线程执行完毕，该方法主要让主线程阻塞等待所有子线程执行完毕                                                                                                                                             |

#### 2、线程锁threading.RLock和threading.Lock

由于线程之间是进行随机调度，并且每个线程可能只执行n条执行之后，CPU接着执行其他线程。为了保证数据的准确性，引入了锁的概念。所以，可能出现如下问题：

假设两个线程t1和t2都要对全局变量g_num(默认是0)进行加1运算，t1和t2都各对g_num加1000000次，g_num的最终的结果应该为2000000。

但是由于是多线程同时操作，有可能出现下面情况：

在g_num=0时，t1取得g_num=0。此时系统把t1调度为”sleeping”状态，把t2转换为”running”状态，t2也获得g_num=0
然后t2对得到的值进行加1并赋给g_num，使得g_num=1
然后系统又把t2调度为”sleeping”，把t1转为”running”。线程t1又把它之前得到的0加1后赋值给g_num。
这样导致虽然t1和t2都对g_num加1，但结果仍然是g_num=1
**如果多个线程同时对同一个全局变量操作，会出现资源竞争问题，从而数据结果会不正确，锁的出现解决了这个问题。**

```python
import threading

g_num = 0
lock = threading.Lock() #创建锁


def work1(num):
    global g_num
    for i in range(num):
        lock.acquire() # 加锁
        g_num += 1
        lock.release() # 解锁
    print("----in work1, g_num is %d---" % g_num)


def work2(num):
    global g_num
    for i in range(num):
        lock.acquire()
        g_num += 1
        lock.release()
    print("----in work2, g_num is %d---" % g_num)


if __name__ == '__main__':
    print("---线程创建之前g_num is %d---" % g_num)
    t1 = threading.Thread(target=work1, args=(1000000,))
    t2 = threading.Thread(target=work2, args=(1000000,))

    t1.start()
    t2.start()

    t1.join()
    t2.join()

    print("2个线程对同一个全局变量操作之后的最终结果是:%s" % g_num)
```

#### 3、threading.RLock和threading.Lock 的区别

  RLock允许在同一线程中被多次acquire。而Lock却不允许这种情况。 

  如果使用RLock，那么acquire和release必须成对出现，即调用了n次acquire，必须调用n次的release才能真正释放所占用的锁。

- Lock多次加锁演示

```python
import threading
lock = threading.Lock()    #Lock对象
lock.acquire()
lock.acquire()  #产生了死锁。
lock.release()
lock.release()
```

- RLock多次加锁演示

```python
import threading
rLock = threading.RLock()  #RLock对象
rLock.acquire()
rLock.acquire()    #在同一线程内，程序不会堵塞。
rLock.release()
rLock.release()
```

#### 4、threading.Event

通过threading.Event()可以创建一个事件管理标志，该标志（event）默认为False，event对象主要有四种方法可以调用：

- event.wait(timeout=None)：调用该方法的线程会被阻塞，如果设置了timeout参数，超时后，线程会停止阻塞继续执行；
- event.set()：将event的标志设置为True，调用wait方法的所有线程将被唤醒；
- event.clear()：将event的标志设置为False，调用wait方法的所有线程将被阻塞；
- event.isSet()：判断event的标志是否为True。

```python
import threading

def do(event):
    print('start')
    event.wait()
    print('execute')

event_obj = threading.Event()
for i in range(10):
    t = threading.Thread(target=do, args=(event_obj,))
    t.start()

event_obj.clear()
inp = input('input:')
if inp == 'true':
    event_obj.set()
```

#### 5、threading.Condition

Python提供的Condition对象提供了对复杂线程同步问题的支持。

- Condition被称为条件变量，除了提供与Lock类似的acquire和release方法外，还提供了wait和notify方法。

##### Condition的处理流程如下：

- 首先acquire一个条件变量，然后判断一些条件。
- 如果条件不满足则wait；
- 如果条件满足，进行一些处理改变条件后，通过notify方法通知其他线程，其他处于wait状态的线程接到通知后会重新判断条件。
- 不断的重复这一过程，从而解决复杂的同步问题。

##### Condition的基本原理如下：

可以认为**Condition对象**维护了一个**锁（Lock/RLock)**和一个**waiting池**。线程通过acquire获得Condition对象，当调用wait方法时，线程会释放Condition内部的锁并进入blocked状态，同时在waiting池中记录这个线程。当调用notify方法时，Condition对象会从waiting池中挑选一个线程，通知其调用acquire方法尝试取到锁。

Condition对象的构造函数可以接受一个Lock/RLock对象作为参数，如果没有指定，则Condition对象会在内部自行创建一个RLock。

除了notify方法外，Condition对象还提供了notifyAll方法，可以通知waiting池中的所有线程尝试acquire内部锁。由于上述机制，**处于waiting状态的线程只能通过notify方法唤醒，所以notifyAll的作用在于防止有的线程永远处于沉默状态。**

##### Condition的案例演示如下：

演示条件变量同步的经典问题是生产者与消费者问题：假设有一群生产者(Producer)和一群消费者（Consumer）通过一个市场来交互产品。生产者的”策略“是如果市场上剩余的产品少于1000个，那么就生产100个产品放到市场上；而消费者的”策略“是如果市场上剩余产品的数量多余100个，那么就消费3个产品。

用Condition解决生产者与消费者问题的代码如下：

```python
# -*- coding: utf-8 -*-
"""
Created on Wed Nov 28 17:15:29 2018

@author: 18665
"""

import threading
import time

class Producer(threading.Thread):
    # 生产者函数
    def run(self):
        global count
        while True:
            if con.acquire():
                # 当count 小于等于1000 的时候进行生产
                if count > 1000:
                    con.wait()
                else:
                    count = count+100
                    msg = self.name+' produce 100, count=' + str(count)
                    print(msg)
                    # 完成生成后唤醒waiting状态的线程，
                    # 从waiting池中挑选一个线程，通知其调用acquire方法尝试取到锁
                    con.notify()
                con.release()
                time.sleep(1)

class Consumer(threading.Thread):
    # 消费者函数
    def run(self):
        global count
        while True:
            # 当count 大于等于100的时候进行消费
            if con.acquire():
                if count < 100:
                    con.wait()

                else:
                    count = count-5
                    msg = self.name+' consume 5, count='+str(count)
                    print(msg)
                    con.notify()
                    # 完成生成后唤醒waiting状态的线程，
                    # 从waiting池中挑选一个线程，通知其调用acquire方法尝试取到锁
                con.release()
                time.sleep(1)

count = 500
con = threading.Condition()

def test():
    for i in range(2):
        p = Producer()
        p.start()
    for i in range(5):
        c = Consumer()
        c.start()
if __name__ == '__main__':
    test()
```

#### 6、queue模块

queue.Queue 就是消息队列，可以利用它实现线程间的安全通信。

```python
import queue

q = queue.Queue(maxsize=0)  # 构造一个先进显出队列，maxsize指定队列长度，为0 时，表示队列长度无限制。

q.join()    # 等到队列为kong的时候，在执行别的操作
q.qsize()   # 返回队列的大小 （不可靠）
q.empty()   # 当队列为空的时候，返回True 否则返回False （不可靠）
q.full()    # 当队列满的时候，返回True，否则返回False （不可靠）
q.put(item, block=True, timeout=None) #  将item放入Queue尾部，item必须存在，可以参数block默认为True,表示当队列满时，会等待队列给出可用位置，为False时为非阻塞，此时如果队列已满，会引发queue.Full 异常。 可选参数timeout，表示 会阻塞设置的时间，过后，如果队列无法给出放入item的位置，则引发 queue.Full 异常
q.get(block=True, timeout=None) #   移除并返回队列头部的一个值，可选参数block默认为True，表示获取值的时候，如果队列为空，则阻塞，为False时，不阻塞，若此时队列为空，则引发 queue.Empty异常。 可选参数timeout，表示会阻塞设置的时候，过后，如果队列为空，则引发Empty异常。
q.put_nowait(item) #   等效于 put(item,block=False)
q.get_nowait() #    等效于 get(item,block=False)
```

代码如下：

```python
import queue
import threading
import time

message = queue.Queue(3)


def producer():
    for i in range(10):
        message.put(i)
        print("PUT>>>%d" % i)


def consumer():
    for i in range(10):
        msg = message.get()
        print("GET>>>%d" % msg)
        time.sleep(1)


if __name__ == '__main__':
    t1 = threading.Thread(target=producer)
    t2 = threading.Thread(target=consumer)

    t1.start()
    t2.start()
```

### Python 进程

#### 1、multiprocess模块

process模块是一个创建进程的模块，借助这个模块，就可以完成进程的创建。

```shell
语法：Process([group [, target [, name [, args [, kwargs]]]]])
```

```python
import multiprocessing
import time


def worker(num):
    """
    process worker function
    :return:
    """
    time.sleep(3)
    print("The num is  %d" % num)
    return


if __name__ == '__main__':
    for i in range(5):
        t = multiprocessing.Process(target=worker, args=(i,), name="t.%d" % i)
        t.start()
```

> Process参数介绍：

- group：参数未使用，默认值为None。

- target：表示调用对象，即子进程要执行的任务。

- args：表示调用的位置参数元组。

- kwargs：表示调用对象的字典。如kwargs = {'name':Jack, 'age':18}。

- name：子进程名称。

- daemon：设置为守护进程(True)或非守护进程(默认：False)

> Process属性方法介绍

|    方法/属性    | 说明                                                                                                                                                                                   |
| :-------------: | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|     start()     | 启动进程，调用进程中的run()方法。                                                                                                                                                      |
|      run()      | 进程启动时运行的方法，正是它去调用target指定的函数，我们自定义类的类中一定要实现该方法 。                                                                                              |
|   terminate()   | 强制终止进程，不会进行任何清理操作。如果该进程终止前，创建了子进程，那么该子进程在其强制结束后变为僵尸进程；如果该进程还保存了一个锁那么也将不会被释放，进而导致死锁。使用时，要注意。 |
|   is_alive()    | 判断某进程是否存活，存活返回True，否则False。                                                                                                                                          |
| join([timeout]) | 主线程等待子线程终止。timeout为可选择超时时间；需要强调的是，p.join只能join住start开启的进程，而不能join住run开启的进程 。                                                             |
|     daemon      | 默认值为False，如果设置为True，代表该进程为后台守护进程；当该进程的父进程终止时，该进程也随之终止；并且设置为True后，该进程不能创建子进程，设置该属性必须在start()之前                 |
|      name       | 进程名称。                                                                                                                                                                             |
|       pid       | 进程pid                                                                                                                                                                                |
|    exitcode     | 进程运行时为None，如果为-N，表示被信号N结束了。                                                                                                                                        |
|     authkey     | 进程身份验证，默认是由os.urandom()随机生成32字符的字符串。这个键的用途是设计涉及网络连接的底层进程间的通信提供安全性，这类连接只有在具有相同身份验证才能成功。                         |

> 注意:

- **在进程python的设计里面只有主进程可以接收input()的输入，子进程没有输入模式，运行直接报错**
- **更坑的是进程池，表面上不会出错，但是input()函数一旦进入无法退出，一直在接收输入的数据**

#### 2、消息队列multiprocess.Queue

可以使用multiprocessing模块的Queue实现多进程之间的数据传递，Queue本身是一个消息列队程序

> Queue属性方法说明

|             方法/属性             | 说明                                                                                                                                                                                                                                                                                                                    |
| :-------------------------------: | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|   get( [ block [ ,timeout ] ] )   | 返回q中的一个元素。如果q为空，此方法将阻塞，直到队列中有项目可用为止。block用于控制阻塞行为，默认为True. 如果设置为False，将引发Queue.Empty异常（定义在Queue模块中）。timeout是可选超时时间，用在阻塞模式中。如果在制定的时间间隔内没有项目变为可用，将引发Queue.Empty异常。                                            |
|           get_nowait( )           | 同q.get(False)方法。                                                                                                                                                                                                                                                                                                    |
| put(item [, block [,timeout ] ] ) | 将item放入队列。如果队列已满，此方法将阻塞至有空间可用为止。block控制阻塞行为，默认为True。如果设置为False，将引发Queue.Empty异常（定义在Queue库模块中）。timeout指定在阻塞模式中等待可用空间的时间长短。超时后将引发Queue.Full异常。                                                                                   |
|             q.qsize()             | 返回队列中目前项目的正确数量。此函数的结果并不可靠，因为在返回结果和在稍后程序中使用结果之间，队列中可能添加或删除了项目。在某些系统上，此方法可能引发NotImplementedError异常。                                                                                                                                         |
|             q.empty()             | 如果调用此方法时 q为空，返回True。如果其他进程或线程正在往队列中添加项目，结果是不可靠的。也就是说，在返回和使用结果之间，队列中可能已经加入新的项目。                                                                                                                                                                  |
|             q.full()              | 如果q已满，返回为True. 由于线程的存在，结果也可能是不可靠的（参考q.empty（）方法）。。                                                                                                                                                                                                                                  |
|          其他方法(了解)           |
|             q.close()             | 关闭队列，防止队列中加入更多数据。调用此方法时，后台线程将继续写入那些已入队列但尚未写入的数据，但将在此方法完成时马上关闭。如果q被垃圾收集，将自动调用此方法。关闭队列不会在队列使用者中生成任何类型的数据结束信号或异常。例如，如果某个使用者正被阻塞在get（）操作上，关闭生产者中的队列不会导致get（）方法返回错误。 |
|      q.cancel_join_thread()       | 不会再进程退出时自动连接后台线程。这可以防止join_thread()方法阻塞。                                                                                                                                                                                                                                                     |
|          q.join_thread()          | 连接队列的后台线程。此方法用于在调用q.close()方法后，等待所有队列项被消耗。默认情况下，此方法由不是q的原始创建者的所有进程调用。调用q.cancel_join_thread()方法可以禁止这种行为。                                                                                                                                        |

#### 3、进程池

为什么要有进程池?进程池的概念。

在程序实际处理问题过程中，忙时会有成千上万的任务需要被执行，闲时可能只有零星任务。那么在成千上万个任务需要被执行的时候，我们就需要去创建成千上万个进程么？首先，创建进程需要消耗时间，销毁进程也需要消耗时间。第二即便开启了成千上万的进程，操作系统也不能让他们同时执行，这样反而会影响程序的效率。因此我们不能无限制的根据任务开启或者结束进程。那么我们要怎么做呢？

在这里，要给大家介绍一个进程池的概念，定义一个池子，在里面放上固定数量的进程，有需求来了，就拿一个池中的进程来处理任务，等到处理完毕，进程并不关闭，而是将进程再放回进程池中继续等待任务。如果有很多任务需要执行，池中的进程数量不够，任务就要等待之前的进程执行任务完毕归来，拿到空闲进程才能继续执行。也就是说，池中进程的数量是固定的，那么同一时间最多有固定数量的进程在运行。这样不会增加操作系统的调度难度，还节省了开闭进程的时间，也一定程度上能够实现并发效果。

##### multiprocess.Pool模块

```shell
语法：Pool([numprocess  [,initializer [, initargs]]])
```

> Pool参数介绍

- numprocess:要创建的进程数，如果省略，将默认使用cpu_count()的值

- initializer：是每个工作进程启动时要执行的可调用对象，默认为None

- initargs：是要传给initializer（可迭代）的参数组

> Pool主要方法介绍

|               方法/属性               | 说明                                                                                                                                                                                                                                         |
| :-----------------------------------: | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|    apply(func [, args [, kwargs]])    | 在一个池工作进程中执行func(*args,**kwargs),然后返回结果。'需要强调的是：此操作并不会在所有池工作进程中并执行func函数。如果要通过不同参数并发地执行func函数，必须从不同线程调用apply()函数或者使用p.apply_async()                             |
| apply_async(func [, args [, kwargs]]) | 在一个池工作进程中执行func(*args,**kwargs),然后返回结果。此方法的结果是AsyncResult类的实例，callback是可调用对象，接收输入参数。当func的结果变为可用时，将理解传递给callback。callback禁止执行任何阻塞操作，否则将接收其他异步操作中的结果。 |
|                close()                | 关闭进程池，防止进一步操作。如果所有操作持续挂起，它们将在工作进程终止前完成                                                                                                                                                                 |
|              terminate()              | 不管任务是否完成，立即停止工作进程。在对pool对象进程垃圾回收的时候，会立即调用terminate()                                                                                                                                                    |
|                jion()                 | 等待所有工作进程退出。此方法只能在close（）或teminate()之后调用                                                                                                                                                                              |
|            其他方法(了解）            | 方法apply_async()和map_async（）的返回值是AsyncResul的实例obj。实例具有以下方法                                                                                                                                                              |
|               obj.get()               | 返回结果，如果有必要则等待结果到达。timeout是可选的。如果在指定时间内还没有到达，将引发一场。如果远程操作中引发了异常，它将在调用此方法时再次被引发。                                                                                        |
|              obj.ready()              | 如果调用完成，返回True                                                                                                                                                                                                                       |
|           obj.successful()            | 如果调用完成且没有引发异常，返回True，如果在结果就绪之前调用此方法，引发异常                                                                                                                                                                 |
|          obj.wait([timeout])          | 等待结果变为可用。                                                                                                                                                                                                                           |
|            obj.terminate()            | 立即终止所有工作进程，同时不执行任何清理或结束任何挂起工作。如果p被垃圾回收，将自动调用此函数                                                                                                                                                |

##### 进程池中的Queue

如果要使用Pool创建进程，就需要使用`multiprocessing.Manager()`中的`Queue()`，而不是`multiprocessing.Queue()`

### Python 协程

协程，又称微线程，纤程。英文名Coroutine。协程是python个中另外一种实现多任务的方式，只不过比线程更小占用更小执行单元（理解为需要的资源）。 为啥说它是一个执行单元，因为它自带CPU上下文。这样只要在合适的时机， 我们可以把一个协程 切换到另一个协程。 只要这个过程中保存或恢复 CPU上下文那么程序还是可以运行的。

通俗的理解：**在一个线程中的某个函数，可以在任何地方保存当前函数的一些临时变量等信息，然后切换到另外一个函数中执行，注意不是通过调用函数的方式做到的，并且切换的次数以及什么时候再切换到原来的函数都由开发者自己确定**

#### 1. 协程和线程差异

在实现多任务时, 线程切换从系统层面远不止保存和恢复 CPU上下文这么简单。 操作系统为了程序运行的高效性每个线程都有自己缓存Cache等等数据，操作系统还会帮你做这些数据的恢复操作。 所以线程的切换非常耗性能。但是协程的切换只是单纯的操作CPU的上下文，所以一秒钟切换个上百万次系统都抗的住。

#### 2. 实现方式

- yield

  ```python
  import time

  def work1():
      while True:
          print("----work1---")
          yield
          time.sleep(0.5)

  def work2():
      while True:
          print("----work2---")
          yield
          time.sleep(0.5)

  def main():
      w1 = work1()
      w2 = work2()
      while True:
          next(w1)
          next(w2)

  if __name__ == "__main__":
      main()

  ```

- greenlet

为了更好使用协程来完成多任务，python中的greenlet模块对其封装，从而使得切换任务变的更加简单

**安装方式**
使用如下命令安装greenlet模块:

```shell
sudo pip3 install greenlet
```

```python
#coding=utf-8

from greenlet import greenlet
import time

def test1():
    while True:
        print "---A--"
        gr2.switch()
        time.sleep(0.5)

def test2():
    while True:
        print "---B--"
        gr1.switch()
        time.sleep(0.5)

gr1 = greenlet(test1)
gr2 = greenlet(test2)

#切换到gr1中运行
gr1.switch()
```

- gevent

greenlet已经实现了协程，但是这个还的人工切换，是不是觉得太麻烦了，不要捉急，python还有一个比greenlet更强大的并且能够自动切换任务的模块gevent

其原理是当一个greenlet遇到IO(指的是input output 输入输出，比如网络、文件操作等)操作时，比如访问网络，就自动切换到其他的greenlet，等到IO操作完成，再在适当的时候切换回来继续执行。

由于IO操作非常耗时，经常使程序处于等待状态，有了gevent为我们自动切换协程，就保证总有greenlet在运行，而不是等待IO。

**安装方式**
使用如下命令安装gevent模块:

```shell
pip3 install gevent
```

在gevent中不能直接使用time.sleep类似的方法，要么使用gevent.sleep，要么利用gevent.monkey打补丁

```python
from gevent import monkey
import gevent
import random
import time

def coroutine_work(coroutine_name):
    for i in range(10):
        print(coroutine_name, i)
        time.sleep(random.random())

gevent.joinall([
        gevent.spawn(coroutine_work, "work1"),
        gevent.spawn(coroutine_work, "work2")
])

```

### 获得进程和线程信息

想要获取线程和进程相关信息，代码如下：

```python
# -*- coding: utf-8 -*-
"""
获得线程, 进程 ID,NAME
总结:

"""
import threading
import psutil
import os
import datetime

# 1 获取线程ID,NAME
t = threading.currentThread()
# 线程ID
print('Thread id : %d' % t.ident)
# 线程NAME
print('Thread name : %s' % t.getName())

# 2 获取线程ID,NAME
pid = os.getpid()
p = psutil.Process(pid)
print('----------------')
# 进程ID
print('Process id : %d' % pid)
# 进程NAME
print('Process name : %s' % p.name())
# 获取进程bin路径
print('Process bin  path : %s' % p.exe())
# 获取pid对应的路径
print('Process path : %s' % p.cwd())
# 进程状态
print('Process status : %s' % p.status())
# 进程运行时间
print('Process creation time : %s' % datetime.datetime.fromtimestamp(p.create_time()).strftime("%Y-%m-%d %H:%M:%S"))
# CPU使用情况
print(p.cpu_times())
# 内存使用情况
print('Memory usage : %s%%' % p.memory_percent())
# 硬盘读取信息
print(p.io_counters())
# 打开进程socket的namedutples列表
print(p.connections())
# 此进程的线程数
print('Process number of threads : %s' % p.num_threads())
```