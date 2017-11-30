# Processes And Threads

| 对比维度 | 多进程 | 多线程 |
| -------| ----- | ----- |
| 数据共享、同步 | 数据共享复杂，需要用IPC；数据是分开的，同步简单 | 因为共享进程数据，数据共享简单，但也是因为这个原因导致同步复杂 |
| 内存、CPU | 占用内存多，切换复杂，CPU利用率低 | 占用内存少，切换简单，CPU利用率高 |
| 创建销毁、切换 | 创建销毁、切换复杂，速度慢 | 创建销毁、切换简单，速度很快 |
| 编程、调试 | 编程简单，调试简单 | 编程复杂，调试复杂 |
| 可靠性 | 进程间不会互相影响 | 一个线程挂掉将导致整个进程挂掉 |
| 分布式 | 适应于多核、多机分布式；如果一台机器不够，扩展到多台机器比较简单 | 适应于多核分布式 |

## 提高多线程程序效率的一般方法
* 不要频繁创建，销毁线程，使用线程池
* 减少线程间同步和通信（最为关键）
* 避免需要频繁共享写的数据合理安排共享数据结构，避免伪共享（false sharing）
* 使用非阻塞数据结构/算法
* 避免可能产生可伸缩性问题的系统调用（比如mmap）
* 避免产生大量缺页异常，尽量使用Huge Page
* 可以的话使用用户态轻量级线程代替内核线程