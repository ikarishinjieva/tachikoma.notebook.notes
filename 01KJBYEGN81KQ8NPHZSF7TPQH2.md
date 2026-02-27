---
title: 20210825 - Linux smaps的相关文档
confluence_page_id: 1343678
created_at: 2021-08-25T06:26:42+00:00
updated_at: 2021-08-25T08:24:08+00:00
---

# 文档1

<https://gist.github.com/CMCDragonkai/10ab53654b2aa6ce55c11cfc5b2432a4>

  - vdso解释: 
    - [vdso] stands for virtual dynamic shared object, its used by system calls to switch to kernel mode
    - So what is vdso and vsyscall? Both are mechanisms to allow faster syscalls, that is syscalls without context switching between user space and kernel space. The vsyscall has now been replaced by vdso, but the vsyscall is left there for compatibility reasons. 

  - Entry point: 

```
$ readelf --file-header ./memory_layout | grep 'Entry point address'
  Entry point address:               0x400720
```

  - 通过 gcore 获取镜像 (TODO)
  - brk和mmap的关系: 
    - The malloc in glibc, internally invokes either brk or mmap syscalls to acquire memory from the OS. The brk syscall is generally used to increase the size of the heap, while mmap will be used to load shared libraries, create new regions for threads, and many other things. It actually switches to using mmap instead of brk when the amount of memory requested is larger than the MMAP_THRESHOLD
    - when malloc switches to using mmap, the regions acquired are not part of the so called [heap] region, which is only provided by the brk/sbrk calls. It's unlabelled! 

# 文档2

<https://sploitfun.wordpress.com/2015/02/10/understanding-glibc-malloc/>

  - 有详细的实验, 说明 smaps中 main arena 和 thread arena 的申请过程
  - 对于64位系统, arena的个数上限是 8*核数, 每个线程一个arena, 超过上限后, 开始共用arena
  - 一个 thread arena, 可以包括多个heap. 每个heap, 只属于一个arena
  - 一个heap, 包括多个chunk
