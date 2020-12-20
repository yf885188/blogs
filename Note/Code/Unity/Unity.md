# 协程
协程在主线程中，实现的实质是一个状态机。

# 线程池
降低频繁创建/销毁线程产生的消耗。

有如下对应关系：
- 非工作者线程 —————— 全局队列 ： FIFO
- 工作者线程 ——————— 线程工作队列 ：LIFO

> 工作者进程以LIFO的方式从自己的工作队列中获取任务进行处理。如果自己的队列为空的话，取相邻的工作者进程(这时以FIFO的方式)的任务，如果工作者线程队列都为空，则从全局队列(以FIFO的方式)来获取任务进行处理。