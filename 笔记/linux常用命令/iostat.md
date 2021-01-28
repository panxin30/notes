简单的说,sar -u看出来的cpu利用率iowait 不实用，iostat -x 中的 svctm   和util 参数
### 命令形式: **iostat -x 1**

每隔一秒输出下

其中的**svctm**参数代表平均每次设备I/O操作的服务时间 (毫秒)，反应了磁盘的负载情况，如果该项大于15ms，并且util%接近100%，那就说明，磁盘现在是整个系统性能的瓶颈了。\
**await** 参数代表平均每次设备I/O操作的等待时间 (毫秒)， 也要多和 svctm 来参考。差的过高就一定有 IO 的问题。如果 svctm 比较接近 await，说明 I/O 几乎没有等待时间；如果 await 远大于 svctm，说明 I/O 队列太长，应用得到的响应时间变慢。\
正常情况下svctm应该是小于await值的，而svctm的大小和磁盘性能有关，CPU、内存的负荷也会对svctm值造成影响，过多的请求也会间接的导致svctm值的增加。\
await值的大小一般取决与svctm的值和I/O队列长度以及I/O请求模式，如果svctm的值与await很接近，表示几乎没有I/O等待，磁盘性能很好，如果await的值远高于svctm的值，则表示I/O队列等待太长，系统上运行的应用程序将变慢，此时可以通过更换更快的硬盘来解决问题。\
%util项的值也是衡量磁盘I/O的一个重要指标，如果%util接近100%，表示磁盘产生的I/O请求太多，I/O系统已经满负荷的在工作，该磁盘可能存在瓶颈。长期下去，势必影响系统的性能，可以通过优化程序或者通过更换更高、更快的磁盘来解决此问题 。 
svctm一项正常时间在20ms左右，原因：
高速cpu会造成很高的iowait值，但这并不代表磁盘是系统的瓶颈。唯一能说明磁盘是系统瓶颈的方法，就是很高的read/write时间，一般来说超过20ms，就代表了不太正常的磁盘性能。为什么是20ms呢？一般来说，一次读写就是一次寻到+一次旋转延迟+数据传输的时间。由于，现代硬盘数据传输就是几微秒或者几十微秒的事情，远远小于寻道时间2~20ms和旋转延迟4~8ms，所以只计算这两个时间就差不多了，也就是15~20ms。只要大于20ms，就必须考虑是否交给磁盘读写的次数太多，导致磁盘性能降低了。\
%iowait并不能反应磁盘瓶颈
iowait实际测量的是cpu时间：
%iowait = (cpu idle time)/(all cpu time)
\
$iostat -x 1
linux 2.6.33-fukai (fukai-laptop)          _i686_    (2 CPU)
avg-cpu:  %user   %nice %system %iowait  %steal   %idle
5.47    0.50    8.96   48.26    0.00   36.82
\
Device:         rrqm/s   wrqm/s     r/s     w/s   rsec/s   wsec/s avgrq-sz avgqu-sz   await  svctm  %util
sda               6.00   273.00   99.00    7.00  2240.00  2240.00    42.26     1.12   10.57   7.96  84.40
sdb               0.00     4.00    0.00  350.00     0.00  2068.00     5.91     0.55    1.58   0.54  18.80
\
rrqm/s:          每秒进行 merge 的读操作数目。即 delta(rmerge)/s
wrqm/s:         每秒进行 merge 的写操作数目。即 delta(wmerge)/s
r/s:            每秒完成的读 I/O 设备次数。即 delta(rio)/s
w/s:            每秒完成的写 I/O 设备次数。即 delta(wio)/s
rsec/s:         每秒读扇区数。即 delta(rsect)/s
wsec/s:         每秒写扇区数。即 delta(wsect)/s
rkB/s:          每秒读K字节数。是 rsect/s 的一半，因为每扇区大小为512字节。(需要计算)
wkB/s:          每秒写K字节数。是 wsect/s 的一半。(需要计算)
avgrq-sz:       平均每次设备I/O操作的数据大小 (扇区)。delta(rsect+wsect)/delta(rio+wio)
avgqu-sz:       平均I/O队列长度。即 delta(aveq)/s/1000 (因为aveq的单位为毫秒)。
await:          平均每次设备I/O操作的等待时间 (毫秒)。即 delta(ruse+wuse)/delta(rio+wio)
svctm:          平均每次设备I/O操作的服务时间 (毫秒)。即 delta(use)/delta(rio+wio)
%util:          一秒中有百分之多少的时间用于 I/O 操作，或者说一秒中有多少时间 I/O 队列是非空的。即 delta(use)/s/1000 (因为use的单位为毫秒)
\
如果 %util 接近 100%，说明产生的I/O请求太多，I/O系统已经满负荷，该磁盘可能存在瓶颈。
