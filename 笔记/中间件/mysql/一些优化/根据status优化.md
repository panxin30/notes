### **MySQL上线后根据status状态进行优化**

MySQL数据库上线后，可以等其稳定运行一段时间后再根据服务器的status状态进行适当优化，我们可以用如下命令列出MySQL服务器运行的各种状态值：

mysql > show global status;

我个人比较喜欢的用法是show status like '查询%';

**1.慢查询**

有时我们为了定位系统中效率比较低下的Query语法，需要打开慢查询日志，也就是Slow Que-ry log。打开慢查询日志的相关命令如下：

mysql>show variables like '%slow%';

+---------------------+-----------------------------------------+

| Variable\_name       |Value                                   |

+---------------------+-----------------------------------------+

| log\_slow\_queries    | ON                                     |

| slow\_launch\_time    | 2                                       |

+---------------------+-----------------------------------------+

mysql>show global status like '%slow%';

+---------------------+-------+

| Variable\_name       | Value|

+---------------------+-------+

| Slow\_launch\_threads | 0    |

| Slow\_queries        | 2128   |

+---------------------+-------+

打开慢查询日志可能会对系统性能有一点点影响，如果你的MySQL是主从结构，可以考虑打开其中一台从服务器的慢查询日志，这样既可以监控慢查询，对系统性能影响也会很小。另外，可以用MySQL自带的命令mysqldumpslow进行查询。比如：下面的命令可以查出访问次数最多的20个SQL语句：

`mysqldumpslow-s c -t 20 host-slow.log`

**2.连接数**

我们如果经常遇见MySQL：ERROR 1040：Too manyconnections的情况，一种情况是访问量确实很高，MySQL服务器扛不住了，这个时候就要考虑增加从服务器分散读压力。另外一种情况是MySQL配置文件中max\_connections的值过小。来看一个例子。

mysql>show variables like 'max\_connections';

+-----------------+-------+

| Variable\_name   | Value |

+-----------------+-------+

| max\_connections | 800   |

+-----------------+-------+

这台服务器最大连接数是256，然后查询一下该服务器响应的最大连接数；

mysql> show global status like 'Max\_used\_connections';

+----------------------+-------+

| Variable\_name        | Value|

+----------------------+-------+

| Max\_used\_connections | 245   |

+----------------------+-------+

MySQL服务器过去的最大连接数是245，没有达到服务器连接数的上线800，不会出现1040错误。

**Max\_used\_connections/max\_connections \* 100% = 85%**

**最大连接数占上限连接数的****85%****左右****,****如果发现比例在****10%****以下，则说明****MySQL****服务器连接数的上限设置得过高了。**
*****
**3.key\_buffer\_size**

key\_buffer\_size是设置MyISAM表索引缓存空间的大小，此参数对MyISAM表性能影响最大。下面是一台MyISAM为主要存储引擎服务器的配置：

mysql> show variables like 'key\_buffer\_size';

+-----------------+-----------+

| Variable\_name   | Value   |

+-----------------+-----------+

| key\_buffer\_size | 536870912 |

+-----------------+-----------+

从上面可以看出，分配了512MB内存给key\_buffer\_size。再来看key\_buffer\_size的使用情况：

mysql> show global status like 'key\_read%';

+-------------------+--------------+

| Variable\_name     | Value |

+-------------------+-------+

| Key\_read\_requests | 27813678766 |

| Key\_reads          |  6798830      |

+-------------------+--------------+

一共有27813678766个索引读取请求，有6798830个请求在内存中没有找到，直接从硬盘读取索引。

key\_cache\_miss\_rate = key\_reads /key\_read\_requests \* 100%

**比如上面的数据，****key\_cache\_miss\_rate****为****0.0244%****，****4000%****个索引读取请求才有一个直接读硬盘，效果已经很好了，****key\_cache\_miss\_rate****在****0.1%****以下都很好，如果****key\_cache\_miss\_rate****在****0.01%****以下的话，则说明****key\_buffer\_size****分配得过多，可以适当减少。**
*****
**4.临时表**

当执行语句时，关于已经被创建了隐含临时表的数量，我们可以用如下命令查询其具体情况：

mysql> show global status like 'created\_tmp%';

+-------------------------+----------+

| Variable\_name           |Value |

+-------------------------+----------+

| Created\_tmp\_disk\_tables | 21119   |

| Created\_tmp\_files       |6     |

| Created\_tmp\_tables      |17715532  |

+-------------------------+----------+

每次创建临时表时，Created\_tmp\_table都会增加，如果磁盘上创建临时表，Created\_tmp\_disk\_tables也会增加。Created\_tmp\_files表示MySQL服务创建的临时文件数，比较理想的配置是：

**Created\_tmp\_disk\_tables/ Created\_tmp\_files \* 100% <= 25%**

**比如上面的服务器****Created\_tmp\_disk\_tables/ Created\_tmp\_files \* 100% =1.20%****，就相当不错。我们在看一下****MySQL****服务器对临时表的配置：**

mysql>show variables where Variable\_name

 in('tmp\_table\_size','max\_heap\_table\_size');

+---------------------+---------+

| Variable\_name       |Value   |

+---------------------+---------+

| max\_heap\_table\_size | 2097152 |

| tmp\_table\_size      |2097152 |

+---------------------+---------+

**5.打开表的情况**

Open\_tables表示打开表的数量，Opened\_tables表示打开过的表数量，我们可以用如下命令查看其具体情况：

mysql> show global status like 'open%tables%';

+---------------+-------+

| Variable\_name | Value |

+---------------+-------+

| Open\_tables   | 351   |

| Opened\_tables | 1455 |

如果Opened\_tables数量过大，说明配置中table\_open\_cache的值可能太小。我们查询下服务器table\_open\_cache;

mysql> show variables like 'table\_open\_cache';

+------------------+-------+

| Variable\_name    | Value |

+------------------+-------+

| table\_open\_cache | 2048  |

+------------------+-------+

**比较合适的值为：**

**open\_tables/ opened\_tables\* 100% > = 85%**

**open\_tables/ table\_open\_cache\* 100% < = 95%**
*****
**6.进程使用情况**

如果我们在MySQL服务器的配置文件中设置了thread\_cache\_size，当客户端断开时，服务器处理此客户请求的线程将会缓存起来以响应一下客户而不是销毁(前提是缓存数未达上线)Thread\_created表示创建过的线程数，我们可以用如下命令查看：

mysql> show global status like 'thread%';

+-------------------+-------+

| Variable\_name     | Value |

+-------------------+-------+

| Threads\_cached    | 40    |

| Threads\_connected | 1     |

| Threads\_created   | 330   |

| Threads\_running   | 1     |

+-------------------+-------+

**如果发现****Threads\_created****的值过大的话，表明****MySQL****服务器一直在创建线程，这也是比较耗费资源的，可以适当增大配置文件中****thread\_cache\_size****的值**。查询服务器thread\_cache\_size配置如下：

mysql> show variables like 'thread\_cache\_size';

+-------------------+-------+

| Variable\_name     | Value |

+-------------------+-------+

| thread\_cache\_size | 100   |

+-------------------+-------+

示例中的MySQL服务器还是挺健康的。

**7.查询缓存(querycache)**

它主要涉及两个参数，query\_cache\_size是设置MySQL的Query Cache大小，query\_cache\_type是设置使用查询缓存的类型，我们可以用如下命令查看其具体情况：

mysql> show global status like 'qcache%';

+-------------------------+-----------+

| Variable\_name           |Value  |

+-------------------------+-----------+

| Qcache\_free\_blocks      |22756     |

| Qcache\_free\_memory      |76764704  |

| Qcache\_hits             | 213028692   |

| Qcache\_inserts          |208894227   |

| Qcache\_lowmem\_prunes    |4010916    |

| Qcache\_not\_cached       |13385031    |

| Qcache\_queries\_in\_cache | 43560    |

| Qcache\_total\_blocks     |111212    |

+-------------------------+-----------+

**MySQL****查询缓存变量的相关解释如下：**

Qcache\_free\_blocks：缓存中相邻内存快的个数。数目大说明可能有碎片。flush query cache会对缓存中的碎片进行整理，从而得到一个空间块。

Qcache\_free\_memory：缓存中的空闲空间。

Qcache\_hits：多少次命中。通过这个参数可以查看到Query Cache的基本效果。

Qcache\_inserts：插入次数，每插入一次查询时就增加1。命中次数除以插入次数就是命中比率。

Qcache\_lowmem\_prunes：多少条Query因为内存不足而被清楚出Query Cache。通过Qcache\_lowmem\_prunes和Query\_free\_memory相互结合，能够更清楚地了解到系统中Query Cache的内存大小是否真的足够，是否非常频繁地出现因为内存不足而有Query被换出的情况。

Qcache\_not\_cached：不适合进行缓存的查询数量，通常是由于这些查询不是select语句或用了now()之类的函数。

Qcache\_queries\_in\_cache：当前缓存的查询和响应数量。

Qcache\_total\_blocks：缓存中块的数量。

**我们在查询一下服务器上关于****query\_cache****的配置命令：**

mysql> show variables like 'query\_cache%';

+------------------------------+---------+

| Variable\_name               | Value   |

+------------------------------+---------+

| query\_cache\_limit           | 1048576 |

| query\_cache\_min\_res\_unit    | 2048    |

| query\_cache\_size            | 2097152 |

| query\_cache\_type            | ON      |

| query\_cache\_wlock\_invalidate | OFF     |

+------------------------------+---------+

**字段解释如下：**

query\_cache\_limit：超过此大小的查询将不缓存。

query\_cache\_min\_res\_unit：缓存块的最小值。

query\_cache\_size：查询缓存大小。

query\_cache\_type：缓存类型，决定缓存什么样的查询，示例中表示不缓存select sql\_no\_cache查询。

query\_cache\_wlock\_invalidat：表示当有其他客户端正在对MyISAM表进行写操作，读请求是要等WRITELOCK释放资源后再查询还是允许直接从Query Cache中读取结果，默认为OFF（可以直接从Query Cache中取得结果。）

query\_cache\_min\_res\_unit的配置是一柄双刃剑，默认是4KB，设置值大对大数据查询有好处，但如果你的查询都是小数据查询，就容易造成内存碎片和浪费。

**查询缓存碎片率****\=Qcache\_free\_blocks /Qcache\_total\_blocks \* 100%**

**如果查询碎片率超过****20%****，可以用****flushquery cache****整理缓存碎片，或者试试减少****query\_cache\_min\_res\_unit****，如果你查询都是小数据库的话。**

**查询缓存利用率****\=(Qcache\_free\_size – Qcache\_free\_memory)/query\_cache\_size \* 100%**

**查询缓存利用率在****25%****一下的话说明****query\_cache\_size****设置得过大，可适当减少****;****查询缓存利用率在****80%****以上而且****Qcache\_lowmem\_prunes> 50****的话则说明****query\_cache\_size****可能有点小，不然就是碎片太多。**

**查询命中率****\= (Qcache\_hits- Qcache\_insert)/Qcache)hits \* 100%**

**示例服务器中的查询缓存碎片率等于****20%****左右，查询缓存利用率在****50%****，查询命中率在****2%****，说明命中率很差，可能写操作比较频繁，而且可能有些碎片。**
*****
**8.排序使用情况**

它表示系统中对数据进行排序时所用的Buffer，我们可以用如下命令查看：

mysql> show global status like 'sort%';

+-------------------+----------+

| Variable\_name     | Value |

+-------------------+----------+

| Sort\_merge\_passes | 10        |

| Sort\_range        | 37431240   |

| Sort\_rows         | 6738691532|

| Sort\_scan         | 1823485     |

+-------------------+----------+

**Sort\_merge\_passes****包括如下步骤：****MySQL****首先会尝试在内存中做排序，使用的内存大小由系统变量****sort\_buffer\_size****来决定，如果它不够大则把所有的记录都读在内存中，而****MySQL****则会把每次在内存中排序的结果存到临时文件中，等****MySQL****找到所有记录之后，再把临时文件中的记录做一次排序。这次再排序就会增加****sort\_merge\_passes****。实际上，****MySQL****会用另外一个临时文件来存储再次排序的结果，所以我们通常会看到****sort\_merge\_passes****增加的数值是建临时文件数的两倍。因为用到了临时文件，所以速度可能会比较慢，增大****sort\_buffer\_size****会减少****sort\_merge\_passes****和创建临时文件的次数，但盲目地增大****sort\_buffer\_size****并不一定能提高速度。**

**9.文件打开数(open\_files)**

我们现在处理MySQL故障时，发现当Open\_files大于open\_files\_limit值时，MySQL数据库就会发生卡住的现象，导致Nginx服务器打不开相应页面。这个问题大家在工作中应注意，我们可以用如下命令查看其具体情况：

show global status like 'open\_files';

+---------------+-------+

| Variable\_name | Value |

+---------------+-------+

| Open\_files    | 1481   |

+---------------+-------+

mysql> show global status like 'open\_files\_limit';

+------------------+-------+

| Variable\_name    | Value |

+------------------+--------+

| Open\_files\_limit | 4509 |

+------------------+--------+

**比较合适的设置是：****Open\_files/ Open\_files\_limit \* 100% < = 75%**
*****
**10.InnoDB\_buffer\_pool\_cache合理设置**

InnoDB存储引擎的缓存机制和MyISAM的最大区别就在于，InnoDB不仅仅缓存索引，同时还会缓存实际的数据。此参数用来设置InnoDB最主要的Buffer的大小，也就是缓存用户表及索引数据的最主要缓存空间，对InnoDB整体性能影响也最大。

无论是MySQL官方手册还是网络上许多人分享的InnoDB优化建议，都是简单地建议将此值设置为整个系统物理内存的50%~80%。这种做法其实不妥，**我们应根据实际的运行场景来正确设置此项参数。**
