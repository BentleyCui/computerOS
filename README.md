## 一 操作系统结构

**内核**

​	**让内核作为应用连接硬件设备的桥梁**，应用程序只需关心与内核交互，不用关心硬件的细节。

<img src="https://user-images.githubusercontent.com/59153788/168106430-82fe3813-4947-4120-ba97-5a0d99954e64.png" alt="image" style="zoom: 80%;" />



内核的4个基本能力：

​	进程管理

​	内存管理

​	设备管理

​	系统调用：如果应用程序要运行更高权限运行的服务，那么就需要有系统调用，它是用户程序与操作系统之间	的接口。

大多数操作系统， 把内存分成了两个区域：

* 内核空间：只有内核程序可以访问
* 用户空间：专门给应用程序使用

用户空间的代码只能访问一个局部的内存空间，而内核空间的代码可以访问所有内存空间。因此，当程序使用用户空间时，我们常说该程序在**用户态**执行，而当程序使内核空间时，程序则在**内核态**执行。

### Linux 的设计

1. MutiTask，多任务：并发与并行

2. SMP，对称多处理：每个 CPU 都可以访问完整的内存和硬件资源，地位相等

3. **ELF**，可执行文件链接格式：Linux系统中**可执行文件的存储格式**，一个个分段

4. Monolithic Kernel，宏内核：Linux系统架构

![image](https://user-images.githubusercontent.com/59153788/168106665-b076f12c-6eb6-4717-a5de-75ab63a509a9.png)

我们编写的代码，首先通过「编译器」编译成汇编代码，接着通过「汇编器」变成目标代码，也就是目标文件，最后通过「链接器」把多个目标文件以及调用的各种函数库链接起来，形成一个可执行文件，也就是 ELF 文件。

执行 ELF 文件的时候，会通过「装载器」把 ELF 文件装载到内存里，CPU 读取内存中的指令和数据，于是程序就被执行起来了

**微内核**：有一个最小版本的内核，一些模块和服务则由用户态管理

**混合内核**：是宏内核和微内核的结合体，内核中抽象出了微内核的概念，也就是内核中会有一个小型的内核，其他模块就在这个基础上搭建，整个内核是个完整的程序

Linux的内核设计是采用宏内核，Windows的内核设计是采用的混合内核

## 二 内存管理

![image](https://user-images.githubusercontent.com/59153788/168106808-e0025619-c386-4c5d-bb2d-01cc42c6d9af.png)



### 虚拟内存

让操作系统为每个进程分配独立的一套**虚拟地址**，前提是每个进程都不能访问物理地址，虚拟地址转换为物理地址对进程来说是透明的

**操作系统会提供一种机制，将不同进程的虚拟地址和不同内存的物理地址映射起来。**

**虚拟内存地址**：程序所使用的的内存地址

**物理内存地址**：硬件里面的空间地址

**内存管理单元**MMU，在CPU中，负责地址转换

操作系统通过内存分段和内存分页，来管理虚拟地址与物理地址之间的关系

### 内存分段

![image](https://user-images.githubusercontent.com/59153788/168106902-7cbb8c1b-3974-4e33-93e7-163b83dcfbe9.png)

物理地址 = 段基址 + 段内偏移量

**段表**

分段机制会把程序的虚拟地址分成4个段：**0代码段**，**1数据段，2堆，3栈**

两个问题：1. 内存碎片 	2. 内存交换的效率低

**内存碎片**

​	**外部内存碎片**：也就是产生了多个不连续的小物理内存，导致新的程序无法被装载，**内存交换 Swap**

​	**内部内存碎片**：程序所有的内存都被装载到了物理内存，但是这个程序有部分的内存可能并不是很常使用，这	也会导致内存的浪费

内存交换空间：在Linux系统中，**Swap空间**是从硬盘划分出来的，专门用于内存与硬盘的空间交换。

但是硬盘的访问速度很慢，**如果内存交换的时候，交换的是一个占内存空间很大的程序，这样整个机器都会显得卡顿。**

分段的好处就是能产生连续的内存空间，但是会出现内存碎片和内存交换的空间太大的问题

### 内存分页

为了解决内存分段时 内存碎片和 内存交换效率低的问题

<img src="https://user-images.githubusercontent.com/59153788/168107015-bb0419c1-080e-45be-bf5e-a6328a4d9f4b.png" alt="image" style="zoom:80%;" />



**分页是把整个虚拟和物理内存空间切成一段段固定尺寸的大小**。这样一个连续并且尺寸固定的内存空间，我们叫**页**（*Page*）。在 Linux 下，每一页的大小为 `4KB`。

**页表**在内存中

**缺页异常**：虚拟地址在页表中查不到，进入系统内核空间分配物理内存、更新进程页表，最后再返回用户空间，恢复进程的运行

采用了分页，释放的内存都是以页为单位释放的， 也就不会产生无法给进程使用的小内存，解决了 内存碎片

如果内存空间不够 -> 换入换出，**页面置换算法**

分页的方式使得我们不需要一次性把程序全部加载到物理内存中

<img src="https://user-images.githubusercontent.com/59153788/168107229-ac010a11-6ead-4900-99c9-55396e538037.png" alt="image" style="zoom:80%;" />

每个进程都有自己的页表，占用很大内存

### 多级页表

**TLB**：程序的局部性原理，执行访问的存储空间也局限于某个内存区域。是个chahe，存放最常访问的页表项，快表

### 段页式内存管理

先将程序划分为多个有逻辑意义的段，段内分页

物理地址 = 段号 + 段内页号 + 页内偏移

### Linux内存管理

虽然每个进程都各自有独立的虚拟内存，但是**每个虚拟内存中的内核地址，其实关联的都是相同的物理内存**。



<img src="https://user-images.githubusercontent.com/59153788/168107463-45e6d515-5fb4-45ba-aaff-f20a5d50f363.png" alt="image" style="zoom:80%;" />



在这 7 个内存段中，堆和文件映射段的内存是**动态分配**的。比如说，使用 C 标准库的 `malloc()` 或者 `mmap()` ，就可以分别在堆和文件映射段动态分配内存

**总结**

每个进程都有自己的虚拟空间，而物理内存只有一个，所以当启用了大量的进程，物理内存必然会很紧张，于是操作系统会通过**内存交换**技术，把不常使用的内存暂时存放到硬盘（换出），在需要的时候再装载回物理内存（换入）。

为了解决简单分页产生的页表过大的问题，就有了**多级页表**，它解决了空间上的问题，但这就会导致 CPU 在寻址的过程中，需要有很多层表参与，加大了时间上的开销。于是根据程序的**局部性原理**，在 CPU 芯片中加入了 **TLB**，负责缓存最近常被访问的页表项，大大提高了地址的转换速度。

### malloc分配内存

虚拟地址空间分为**内核空间和用户空间**，用户空间由低到高分为：代码段、已初始化数据段、未初始化数据段、堆段、文件映射段、栈段

**堆**和**文件映射区**的内存是动态分配的

malloc（）库函数，用于动态分配内存。有两种方式向操作系统申请堆内存：

**（1）通过 brk（）系统调用从堆分配内存**

**（2）通过 mmap（）系统调用在文件映射区域分配内存**

方法一通过brk（）函数将**堆顶**指针向高地址移动，获得新的内存空间

![image](https://user-images.githubusercontent.com/59153788/168107560-650d0768-e145-477a-81a0-aea840777832.png)



方法二 mmap（）系统调用**「私有匿名映射」**的方式，在文件映射区分配一块内存，也就是从文件映射区“偷”了一块内存

![image](https://user-images.githubusercontent.com/59153788/168107640-eeaab099-45dd-4a95-89ef-532bd8cf5bee.png)

malloc() 源码里默认定义了一个阈值：

- 如果用户分配的内存小于 128 KB，则通过 brk() 申请内存；
- 如果用户分配的内存大于 128 KB，则通过 mmap() 申请内存；

malloc() 在分配内存的时候，并不是老老实实按用户预期申请的字节数来分配内存空间大小，而是**会预分配更大的空间作为内存池**。



对于 「malloc 申请的内存，free 释放内存会归还给操作系统吗？」这个问题，我们可以做个总结了：

- malloc 通过 **brk()** 方式申请的内存，free 释放内存的时候，**并不会把内存归还给操作系统，而是缓存在 malloc 的内存池中，待下次使用**；
- malloc 通过 **mmap()** 方式申请的内存，free 释放内存的时候，**会把内存归还给操作系统，内存得到真正的释放**。



## 三 进程管理

### 1. 进程、线程基础知识

<img src="https://user-images.githubusercontent.com/59153788/168107754-7fc78f19-857a-43c1-924e-d58cbd01dd7b.png" alt="image" style="zoom: 67%;" />

**并发与并行的区别**

进程的状态：运行状态，就绪状态，阻塞状态

**挂起状态**：进程没有占用实际的物理内存空间。在虚拟内存管理的操作系统中，通常会把阻塞状态的进程的物理内存空间换出到硬盘，等需要再次运行的时候，再从硬盘换入到物理内存。

阻塞挂起状态：进程在外存，并等待某个事件的出现

就绪挂起状态：进程在外存，但只要进入内存就能立即运行

#### 进程的控制

进程的控制结构：**进程控制块 PCB**。PCB是进程存在的唯一标识，包含：进程描述信息，进程控制和管理信息，资源分配清单，CPU相关信息

PCB是以**链表**的方式进行组织的，相同状态的进程链在一起，组成各种队列：**就绪队列，阻塞队列**

**创建进程，终止进程，阻塞进程，唤醒进程**的过程



#### 进程的上下文切换

各个进程是共享CPU资源的，一个进程切换到另一个进程运行，称为进程的上下文切换。

主要是保存**CPU寄存器** 和 **程序计数器的值**

**进程的上下文切换不仅包含了虚拟内存、栈、全局变量等用户空间的资源，还包括了内核堆栈、寄存器等内核空间的资源。**

通常会把交换的信息保存在进程的PCB中。

发生在**内核态**，（1）时间片耗完，（2）系统资源不足（3）进程通过sleep睡眠函数主动挂起（4）运行更高优先级的进程（5）发生硬件中断



#### 线程

线程是进程当中的一条执行流程，同一个进程内多个线程之间可以共享代码段、数据段、打开的文件等资源，但每个线程各自都有一套独立的寄存器和栈，这样可以确保线程的控制流是相对的。

**线程的优点：**

1. 一个进程中可以同时存在多个线程
2. 各个线程之间可以并发执行
3. 各个线程之间可以共享地址空间和文件等资源；



**进程与线程的比较：**

1. 进程是资源分配的基本单位，线程是CPU调度的基本单位

2. 进程拥有一个完整的资源平台，而线程只独享必不可少的资源，如寄存器和栈；

3. 线程同样具有就绪、阻塞、执行三种基本状态，同样具有状态之间的转换关系；

4. 线程能减少并发执行的时间和空间开销；

   开销小体现在：

   	1. 线程的创建时间比进程快
   	2. 线程的终止时间比进程快，因为线程释放的资源相比进程少很多；
   	3. 同一个进程内的线程切换比进程切换快，因为线程具有相同的地址空间
   	4. 由于同一进程的各线程间共享内存和文件资源，那么在线程之间数据传递的时候，就不需要经过内核了，这就使得线程之间的数据交互效率更高了；



**线程的上下文切换：**

1. 当两个线程不是属于同一个进程，则切换的过程就跟进程上下文切换一样；
2. **当两个线程是属于同一个进程，因为虚拟内存是共享的，所以在切换时，虚拟内存这些资源就保持不动，只需要切换线程的私有数据、寄存器等不共享的数据**；



**线程的实现：**

1. **用户线程：**在用户空间实现的线程，操作系统不直接参与

2. **内核线程：**在内核实现的线程
3. **轻量级线程：**在内核中来支持用户线程；



#### 调度算法

**先来先服务调度算法 FCFS**

**最短作业优先调度算法 SJF**: 优先选择运行时间短的进程

**高响应比优先调度算法 HRRN**：主要是为了权衡短作业和长作业  优先权 = （等待时间 + 要求服务时间） / 要求服务时间

**时间片轮转调度算法**：每个进程被分配一个时间片

**最高优先级调度算法**：静态优先级，创建进程的时候就已经确定了，整个运行期间优先级不变

​	动态优先级，随着等待时间的推移增加等待进程的优先级

**多级反馈队列调度算法**：

- 「多级」表示有多个队列，每个队列优先级从高到低，同时优先级越高时间片越短。
- 「反馈」表示如果有新的进程加入优先级高的队列时，立刻停止当前正在运行的进程，转而去运行优先级高的队列；

### 2.进程间的通信

每个进程的用户空间都是独立的，一般而言是不能相互访问的，但是内核空间是每个进程都共享的，所以进程间要通信必须通过内核

<img src="https://user-images.githubusercontent.com/59153788/168108139-a09f4fac-0e5a-4bf0-ab60-4ca8544d2aee.png" alt="image" style="zoom:80%;" />



**1.管道**

管道传输数据是单向的

```shell
// 1. 匿名管道
ps auxf | grep mysql	
// [ | ]表示的管道称为匿名管道，生存周期是随进程创建而建立，随进程的结束而销毁

// 2.命名管道 也叫FIFO
mkfifo myPipe
// 先用mkfifo命令来创建 并指定管道名
```

**管道这种通信方式效率低，不适合进程间频繁地交换数据**。所谓的管道，就是内核里面的一串缓存

使用 **fork** 创建子进程，创建的子进程会复制父进程的文件描述符，两个进程就可以通过各自的 fd 写入和读取同一个管道文件实现跨进程通信了

**对于匿名管道，它的通信范围是存在父子关系的进程**，没有管道实体

**对于命名管道，它可以在不相关的进程间也能相互通信**。因为命令管道，提前创建了一个类型为管道的设备文件，在进程里只要使用这个设备文件，就可以相互通信。

**2.消息队列**

消息队列是保存在内核中的消息链表，在发送数据时，会将数据分成一个一个独立的数据单元---消息体

消息队列生命周期随内核

消息队列不适合比较大数据的传输。 消息队列通信过程中，存在用户态与内核态之间的数据拷贝开销

**3.共享内存**

主要解决：消息队列读取和写入 过程中，用户态与内核态之间的消息拷贝过程

每个进程都有自己独立的虚拟内存空间，**共享内存的机制，就是拿出一块虚拟地址空间来，映射到相同的物理内存中**

新的问题：多个进程同时修改同一个共享内存，可能会冲突

**4.信号量**

共享资源在任意时刻只能被一个进程访问。信号量其实是一个**整型计数器**，主要用于实现进程间的互斥与同步，而不是用于缓存进程间通信的数据

**P操作**：信号量 -1

**V操作**：信号量 +1

**P操作是用在进入共享资源之前，V操作是用在离开共享资源之后，这两个操作必须成对出现**

**互斥信号量**：信号初始化为1，可以保证在任意时刻最多只有一个进程在访问

**同步信号量**：信号初始化为0，先执行V操作，再执行P操作



**5.信号**

上面说的进程间通信，都是常规状态下的工作模式。**对于异常情况下的工作模式，就需要用「信号」的方式来通知进程。**

```shell
$ kill -l	//查看linux中所有的信号
// 运行在shell 终端的进程，我们可以通过键盘输入某些组合键，给进程发送 信号
ctrl+c : SIGINT信号，表示终止该进程
ctrl+z : SIGTSTP信号，表示停止该进程，但还未结束
$ kill -9 1050 // 给PID 为1050的发送9号信号
```

信号是进程间通信机制中**唯一的异步通信机制**

**6. Socket**

前面提到的管道、消息队列、共享内存、信号量和信号都是在同一台主机上进行进程间通信，那要想**跨网络与不同主机上的进程之间通信，就需要 Socket 通信了。**

Socket 通信不仅可以跨网络与不同主机的进程间通信，还可以在同主机上进程间通信

```c++
int socket(int domain, int type, int protocal)
// domain参数用来指定协议族
//type参数用来指定通信特性,字节流、数据报、原始套接字
//protocal参数原本是用来指定通信协议的，但现在基本废弃
```

**实现TCP字节流通信**

**实现UDP数据报通信**

**实现本地进程间通信**

### 3.多线程冲突

线程是调度的基本单位，进程则是资源分配的基本单位

线程之间是可以共享进程的资源，比如代码段、堆空间、数据段、打开的文件等资源，但每个线程都有自己独立的栈空间

多个线程如果竞争共享资源，如果不采取有效的措施，则会造成共享数据的混乱。

**临界区**：它是访问共享资源的代码片段，一定不能给多线程同时执行

互斥也并不是只针对多线程。在多进程竞争共享资源的时候，也同样是可以使用**互斥**的方式来避免资源竞争造成的资源混乱。

互斥解决了并发进程/线程对临界区的使用问题

**同步的概念**

所谓同步，就是并发进程/线程在一些关键点上可能需要互相等待与互通消息，这种相互制约的等待与互通信息成为进程/线程同步

- 同步就好比：「操作 A 应在操作 B 之前执行」，「操作 C 必须在操作 A 和操作 B 都完成之后才能执行」等；
- 互斥就好比：「操作 A 和操作 B 不能在同一时刻执行」；

为了实现进程协作，操作系统提供了两种办法：

* **锁**：加锁、解锁操作
* **信号量**：P  V 操作

信号量比锁的功能更强一些，它还可以方便地实现进程/线程同步

**锁**

进入临界区加锁，完成临界区的资源访问后再解锁

​	**忙等待锁**（自旋锁）

​	原子操作指令：**Test - and - Set**，测试并设置指令

```c
int TestAndSet(int *old_ptr, int new){
    int old = *old_ptr;
    *old_ptr = new;
    return old;
}

typedef struct lock_t{
    int flag;
} lock_t;

void init(lock_t *lock){
    lock->flag = 0;
}

void lock(lock_t *lock){
    while(TestAndSet(&lock->flag, 1) == 1)	// 当flag一直为1时，一直忙等
        ;// do nothing
}

void unlock(lock_t *lock){
    lock->flag = 0;
}
```

**无等待锁**

**信号量**

**生产者-消费者问题**

![image](https://user-images.githubusercontent.com/59153788/168108246-bb6420c6-4600-4a14-a170-20189f55eabe.png)

任意时刻只能有一个线程操作缓冲区，**互斥**

缓冲区空时，消费者必须等待生产者生成数据；缓冲区满时，生产者必须等待消费者取出数据。说明生产者和消费者**需要同步**。

**互斥信号量 mutex**

**资源信号量 `fullBuffers`**：用于消费者询问缓冲区是否有数据，有数据则读取数据，初始化0

**资源信号量 `emptyBuffers`**：用于生产者询问缓冲区是否有空位，有空位则生成数据，初始化n

```c++
#define N 100
semaphore mutex = 1;	//互斥信号量
semaphore emptyBuffers = N;
semaphore fullBuffers = 0;

void producer(){
    while(true){
        P(emptyBuffers);	// 空槽个数 -1
        P(mutex);
        //将生成的数据放入到缓冲区中
        V(mutex);
        V(fullBuffers);	   //满槽个数 +1
    }
}

void consumer(){
    while(true){
        P(fullBuffers);
        P(mutex);
        //从缓冲区读取数据
        V(mutex);
        V(emptyBuffers);
    }
}
```

**哲学家进餐问题**

偶数编号的哲学家 先拿左边的叉子后拿右边的叉子， 奇数编号的哲学家 先拿右边的叉子后拿左边的叉子

**读者-写者问题**

读者只会读取数据，写者既可以读也可以修改数据

[ 读 - 读 ]：允许

[ 读 - 写]：互斥

[ 写 - 写 ]：互斥

### 4.怎样避免死锁

当两个线程为了保护两个不同的共享资源而使用了两个互斥锁，那么这两个互斥锁应用不当的时候，可能会造成**两个线程都在等待对方释放锁**，在没有外力的作用下，这些线程会一直相互等待，就没办法继续运行，这种情况就是发生了**死锁**。

死锁只有同时满足以下四个条件才会发生：

* 互斥条件
* 持有并等待条件
* 不可剥夺条件
* 环路等待条件

那么避免死锁问题就只需要破环其中一个条件就可以，最常见的并且可行的就是**使用资源有序分配法，来破环环路等待条件**。

### 5.悲观锁、乐观锁

![image](https://user-images.githubusercontent.com/59153788/168108392-d5bf513e-4296-40d4-8db0-bd5d2b0ed277.png)

为了选择合适的锁，我们不仅需要清楚知道加锁的成本开销有多大，还需要分析业务场景中访问的共享资源的方式，再来还要考虑并发访问共享资源时的冲突概率。

**互斥锁**

​	对于互斥锁加锁失败而阻塞的现象，是由操作系统内核实现的。内核会将线程置为 [睡眠] 状态，等到锁被释放后，内核会在合适的时机唤醒线程。

互斥锁加锁失败时，会从用户态陷入到内核态，让内核帮我们切换线程，虽然简化了使用锁的难度，但是存在一定的性能开销成本。**两次线程上下文切换**

互斥锁、自旋锁、读写锁都属于**悲观锁**，悲观锁做事比较悲观，它认为**多线程同时修改共享资源的概率比较高，于是很容易出现冲突，所以访问共享资源前，先要上锁**。

那相反的，如果多线程同时修改共享资源的概率比较低，就可以采用**乐观锁**。先修改完共享资源，再验证这段时间内有没有发生冲突，如果没有其他线程在修改资源，那么操作完成，如果发现有其他线程已经修改过这个资源，就放弃本次操作。

