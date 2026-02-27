---
title: 20210519 - 内存去哪儿了
confluence_page_id: 754139
created_at: 2021-05-19T13:01:06+00:00
updated_at: 2021-05-19T13:01:06+00:00
---

# /proc/meminfo

| 举例 | 说明 |
| --- | --- |
| MemTotal: 263556340 kB |  |
| MemFree: 64098076 kB |  |
| MemAvailable: 195827428 kB |  |
| Buffers: 1550160 kB | 块设备(block device)所占用的缓存页包括：直接读写块设备、以及文件系统元数据(metadata)比如SuperBlock所使用的缓存页“Buffers”所占的内存同时也在LRU list中，被统计在Active(file)或Inactive(file) |
| Cached: 107107528 kB | 普通文件所占用的缓存页 |
| SwapCached: 0 kB | 参看SwapCache章节 |
| Active: 67328824 kB | 参考LRU章节 | Inactive: 92182272 kB Active(anon): 44415664 kB Inactive(anon): 3338092 kB Active(file): 22913160 kB Inactive(file): 88844180 kB Unevictable: 0 kB
| Mlocked: 0 kB| 参考Mlocked章节
| SwapTotal: 267845628 kB|
| SwapFree: 267845628 kB|
| Dirty: 12120 kB| 脏页
| Writeback: 0 kB| 正准备回写硬盘的缓存页
| AnonPages: 50829032 kB| 匿名页包括THP页面
| Mapped: 5161572 kB| Cached中 所有的与进程相关的页面 (与进程无关的页面, 可能是进程已经结束)
| Shmem: 5930952 kB| Shmem
| Slab: 31129820 kB| 参考"内存分配分类", 通过slabtop观察
| SReclaimable: 21754596 kB| Slab 可回收的部分
| SUnreclaim: 9375224 kB| Slab 不可回收的部分
| KernelStack: 620432 kB| 每个用户线程都会对应一个内核栈 内核栈的总大小
| PageTables: 760712 kB| 页表大小
| NFS_Unstable: 0 kB| NFS_Unstable是发给NFS server但尚未写入硬盘的缓存页
  
NFS_Unstable的内存被包含在Slab中  
Bounce: 0 kB| 参考Bounce章节  
WritebackTmp: 0 kB|   
CommitLimit: 399623796 kB|   
Committed_AS: 359418704 kB|   
VmallocTotal: 34359738367 kB| -  
VmallocUsed: 0 kB| 参考Vmalloc章节  
VmallocChunk: 0 kB| largest contiguous block of vmalloc area which is free  
HardwareCorrupted: 0 kB| 由于硬件故障, 被放弃使用的内存大小  
AnonHugePages: 163840 kB| 参考 匿名大页 章节  
ShmemHugePages: 0 kB| -  
ShmemPmdMapped: 0 kB| -  
CmaTotal: 0 kB| -  
CmaFree: 0 kB| -  
HugePages_Total: 0| 参考 传统大页 章节  
HugePages_Free: 0  
HugePages_Rsvd: 0  
HugePages_Surp: 0  
Hugepagesize: 2048 kB| 大页大小  
DirectMap4k: 14431040 kB| TLB(Translation Lookaside Buffer)是位于CPU上的缓存，用于将内存的虚拟地址翻译成物理地址TLB作为页表的缓存DirectMap 是 TLB中不同内存大小的块, 用于评估TLB的效率  
DirectMap2M: 206231552 kB  
DirectMap1G: 47185920 kB  
  
# 资料

  - <https://people.engr.tamu.edu/bettati/Courses/613/2013A/Slides/dynamic_memory.pdf>
    - ![image](http://10.186.18.11/confluence/download/attachments/27854113/image2021-3-18%2012%3A58%3A38.png?version=1&modificationDate=1616050246000&api=v2)
    - ![image](http://10.186.18.11/confluence/download/attachments/27854113/image2021-3-18%2012%3A59%3A25.png?version=1&modificationDate=1616050246000&api=v2)
      - 此处对vmalloc分类错误, 参考: <https://lwn.net/Articles/711653/> (The alternative to the slab allocator is vmalloc())
  - <https://lwn.net/Articles/711653/>
  - <http://linuxperf.com/?p=142>

# 内存分配分类

  - 页分配
    - 分配方法: alloc_pages() and __get_free_pages()
    - 分配算法为buddy system
    - (参考<https://www.jianshu.com/p/391f42f8fb0d>) 不会自动参与内存统计
      - 通过alloc_pages分配的内存不会自动统计，除非调用alloc_pages的内核模块或驱动程序主动进行统计，否则我们只能看到free memory减少了，但从/proc/meminfo中看不出它们具体用到哪里去了。比如在VMware guest上有一个常见问题，就是VMWare ESX宿主机会通过guest上的Balloon driver(vmware_balloon module)占用guest的内存，有时占用得太多会导致guest无内存可用，这时去检查guest的/proc/meminfo只看见MemFree很少、但看不出内存的去向，原因就是Balloon driver通过alloc_pages分配内存，没有在/proc/meminfo中留下统计值，所以很难追踪

  - slab分配: 在 页分配 机制基础上建立
    - 分配方法: kmalloc
    - 以字节为单位分配
    - 在kernel内存地址空间中分配
    - 分配物理地址连续的内存块
    - 特点: 分配速度快, 不合适大块内存分配
  - vmalloc
    - 以字节为单位分配
    - 在独立地址空间中分配
    - 分配虚拟地址连续的内存块
    - 特点: 与kmalloc相反
  - 用户态分配
    - 分配方法: malloc
    - 以字节为单位分配
    - 在用户地址空间中分配
    - 分配虚拟地址连续的内存块

# VMallocUsed

  - 由 内核 / 其他内核模块 分配的, 通过/proc/vmallocinfo可以查看分配细节
  - 内核模块占用的内存信息, 通过lsmod的size列查看

# Bounce

<http://linuxperf.com/?p=142>

有些老设备只能访问低端内存，比如16M以下的内存，当应用程序发出一个I/O 请求，DMA的目的地址却是高端内存时（比如在16M以上），内核将在低端内存中分配一个临时buffer作为跳转，把位于高端内存的缓存数据复制到此处。这种额外的数据拷贝被称为“bounce buffering”，会降低I/O 性能。大量分配的bounce buffers 也会占用额外的内存。

# 传统大页 Hugepage

  - <http://linuxperf.com/?p=142>
    - ![image](http://10.186.18.11/confluence/download/attachments/27854113/image2021-3-18%2014%3A18%3A50.png?version=1&modificationDate=1616050246000&api=v2)
    - 通过/proc//smaps, VmFlags中"ht"代表Hugepages, kernelPageSize是大小
  - 传统大页不计入RSS
  - 大页数量 = HugePages_Total * Hugepagesize

# 匿名大页 AnonHugePages

  - <http://linuxperf.com/?p=142>
    - ![image](http://10.186.18.11/confluence/download/attachments/27854113/image2021-3-18%2014%3A21%3A23.png?version=1&modificationDate=1616050246000&api=v2)
    - 在/proc//smaps 中, AnonHugePages 是大小

# LRU

  - <http://linuxperf.com/?p=142>
    - Page cache和所有用户进程的内存（kernel stack和huge pages除外）都在LRU lists上。
    - 分类:
      - LRU_INACTIVE_ANON – 对应 Inactive(anon)

      - LRU_ACTIVE_ANON – 对应 Active(anon)

      - LRU_INACTIVE_FILE – 对应 Inactive(file)

      - LRU_ACTIVE_FILE – 对应 Active(file)

      - LRU_UNEVICTABLE – 对应 Unevictable

    - Active与Inactive的区别: Inactive list里的是长时间未被访问过的内存页，Active list里的是最近被访问过的内存页
    - 用户进程的内存页分为两种：file-backed pages（与文件对应的内存页）和anonymous pages（匿名页）
    - Unevictable LRU list上是不能pageout/swapout的内存页

# SwapCache

/proc/meminfo中的SwapCached背后的含义是：系统中有多少匿名页曾经被swap-out、现在又被swap-in并且swap-in之后页面中的内容一直没发生变化。也就是说，如果这些匿名页需要被swap-out的话，是无需进行I/O write操作的。

  - 仅包括匿名页
  - 这些页已在内存中, 也在交换设备上, 并且一致
  - 相当于: 内存中的这些页 是 交换设备上的页 的Cache

# Mlocked

  - “Mlocked”统计的是被mlock()系统调用锁定的内存大小。
  - 被锁定的内存因为不能pageout/swapout，会从Active/Inactive LRU list移到Unevictable LRU list上。
  - “Mlocked”并不是独立的内存空间，它与以下统计项重叠：LRU Unevictable，AnonPages，Shmem，Mapped等。

# 脏页

系统中全部dirty pages = ( Dirty + NFS_Unstable + Writeback )  

# Shmem

  - 包括
    - shared memory: 通过tmpfs实现
    - tmpfs和devtmpfs
  - 不计入AnonPages, 而计入Cached(page cache) 和 Mapped(当shmem被attached时候)
  - 也计入 Inactive(anon) 或 Active(anon) 或 unevictable
