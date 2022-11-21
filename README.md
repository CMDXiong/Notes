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

用信号量解决进程同步问题(生产者-消费者问题)
进程合作：多进程共同完成一个任务
```C
// 共享数据
# define BUFFER_SIZE 10
typedef struct{...} item;
item buffer[BUFFER_SIZE];
int in = out = counter =0;
```

```C
// 生产者进程
while(true) {
  while(counter == BUFFER_SIZE) ; // 缓存区满，生产者要停
  buffer[in] = item;
  in = (in+1) % BUFFER_SIZE;
  counter++;    // 发信号让消费者再走
}
```
```C
while(true){
  while(counter == 0) ;  // 缓存区空，消费者要停
  item = buffer[out];
  out = (out+1) % BUFFER_SIZE;
  counter--;  // 发信号让生产者再走
}
```
用;来做无限while循环，但是会耗尽cpu，如下解决
```C
// 生产者进程
while(true) {
  while(counter == BUFFER_SIZE) 
  sleep();  // 缓存区满，生产者要停
  ...
  counter = counter + 1;    
  if (counter == 1) wakeup(消费者); // 发信号让消费者再走
}
```
```C
while(true){
  while(counter == 0)
  sleep(); // 缓存区空，消费者要停
  ...
  counter = counter + 1;  
  if(counter == BUFFER_SIZE-1) wakeup(生产者);  // 发信号让生产者再走
}
```
只发信号还不能解决全部问题：比如有多个生产者P1, P2，当缓冲区满后，生产者P1, P2都会sleep,消费者C执行1次循环，发信号给P1,P1被唤醒，当消费者C再执行1次循环，counter == BUFFER_SIZE-1不再成立，P2就不能被唤醒

信号量解决以上问题

<img width="808" alt="image" src="https://user-images.githubusercontent.com/29672091/202977564-fc424579-9b96-49c2-a7c2-e1cbbb278281.png">

#### 对信号量的临界区保护
1. 1
2. 2
3. 进入临界区Peterson算法
结合了标记和轮转两种思想
// P0 进程
```C
flag[0] = true;
turn = 1;
while(flag[1] && turn == 1) ;
  临界区
flag[0] = false;
  剩余区
```
// P1 进程
```C
flag[1] = true;
turn = 0;
while(flag[0] && turn == 0) ;
  临界区
flag[1] = false;
  剩余区
```
