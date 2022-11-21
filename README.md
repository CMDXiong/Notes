# 操作系统
## 操作系统之进程与线程
### 操作系统之CPU调度策略
调度的含义：从就绪队列中找一个进程来执行
调度策略需要考虑的点：
1. 周转时间
2. 响应时间
3. 任务的特点
4. 优先级等
#### 1. FIFO
就绪队列中：先进先调度，简单有效
#### 2. Priority
按任务的优先级进行调度，优先级会动态进行调整
#### 3. First Come, First Served(FCFS)
#### 4. SJF 短作业优先
#### 5. 按时间片轮转调度Round Robin
时间片大：响应时间太长；时间片小：吞吐量小
折衷：时间片10-100ms,切换时间0.1-1ms(1% )
#### 6. 优先调度
如果是固定优先级任务调度：导致后台任务处于饥饿，可能会有进程一直得不到调度机会，比如一直有前台任务在执行

解决方案：后台任务优先级动态升高，同时导致新的问题出现：后台任务（比较大的后台任务）一旦执行，前台任务的响应时间又会变得很长

#### 一个实际的schedule函数

### 进程同步与信号量
多个进程在合作完成一件事情，推进顺序就要合理，进程间便需要进行通信。即：进程什么时候睡眠，什么时候唤醒？
进程间的同步就是多个进程走走停停，合理有序推进，信号量便可以决定进程什么时候睡眠，什么时候唤醒。信号量是某种资源的数量，当信号量大于0时，表示进程还有资源可以使用，小于0时，表示进程无资源使用，应当睡眠

信号量：
```C
struct semaphore
{
  int value;  // 记录资源个数
  PCB *queue; // 记录等待在该信号量上的进程
}
P(semaphore s);  // 消费资源
V(semaphore s);  // 产生资源

P(semaphore s) 
{
  s.value--;
  if(s.value < 0){
    sleep(s.queue);
  }
}

V(semaphore s)
{
  s.value++;
  if(s.value <= 0){  // 有进程在睡眠
    wakeup(s.queue);
  }
}
```

用信号量解决进程同步问题
