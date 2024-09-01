参考链接：
  https://zhuanlan.zhihu.com/p/54820733

快速省电模式是在L PS的基础上来的，被称为PS-NULL-Poll机制。这个PS-null-Poll的机制是当STA wake up后，解析到beacon中有自己的缓存帧，则会发送一个null data帧给AP，并将PS位置为0，表示不节能了，需要获取所以缓存的数据，AP收到这个null-data后，从缓存队列中依次取出一个缓存data帧发送给STA，直到所有的缓存数据帧发送完，当然最好一个data帧的more data位会被置0。STA收到所有缓存帧后，直接发一个null data帧 给AP，并将PS位置1，表示自己再次进入PS状态。

快速PS和LPS的区别显而易见，LPS是一个PS-poll获取一个缓存帧，一次一帧，而且ps-poll ack都不能少，而Fast PS是一次null-data表示sleep or wake状态。并获取所以缓存帧，就好比搬运货物，LPS是徒手，一次一个箱子，来回跑，而Fast PS是用传送带，建立连接后，箱子一个一个的往传送带上搬运。

![](https://img2020.cnblogs.com/blog/745188/202109/745188-20210921221721556-1592311310.jpg)
