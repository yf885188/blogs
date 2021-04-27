# 思路
在工作线程和函数执行之间引入中间层，去掉锁的使用，降低资源竞争。

![][StrandsFrame]

[StrandsFrame]: ./images/StrandsFrame.jpg


- run: 每次从strand中取队列前的第一个handler进行处理,是一个while loop

> 实际上就是保证strand的资源大体是一致的？因为strand内的函数是无法并行执行的，所以降低了资源的竞争，提高了效率。
