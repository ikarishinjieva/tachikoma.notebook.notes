---
title: 20211001 - 为什么tcmalloc会慢
confluence_page_id: 1343775
created_at: 2021-10-01T03:20:13+00:00
updated_at: 2021-10-08T10:29:19+00:00
---

# 背景

根据 [20210929 - 测试 jemalloc] 的测试, jemalloc在统计malloc时会严重影响性能

而tcmalloc会严重影响性能, 需探究原因

# 测试1

修改tcmalloc源码: src/[heap-profiler.cc](<http://heap-profiler.cc>)

将RecordAlloc/RecordFree修改, 将其中代码全都去掉, 性能不再受到影响

# 测试2

修改tcmalloc源码: src/[heap-profiler.cc](<http://heap-profiler.cc/>)

将RecordAlloc/RecordFree修改, 仅保留 HeapProfileTable::GetCallerStackTrace: 

```
// Record an allocation in the profile.
static void RecordAlloc(const void* ptr, size_t bytes, int skip_count) {
  // Take the stack trace outside the critical section.
  void* stack[HeapProfileTable::kMaxStackDepth];
  int depth = HeapProfileTable::GetCallerStackTrace(skip_count + 1, stack);
  SpinLockHolder l(&heap_lock);
  if (is_on) {
  //  heap_profile->RecordAlloc(ptr, bytes, depth, stack);
  //  MaybeDumpProfileLocked();
  }
}

// Record a deallocation in the profile.
static void RecordFree(const void* ptr) {
  SpinLockHolder l(&heap_lock);
  if (is_on) {
  //  heap_profile->RecordFree(ptr);
  //  MaybeDumpProfileLocked();
  }
}
``` 

对程序性能影响极大, 认为HeapProfileTable::GetCallerStackTrace消耗过大

查看tcmalloc的编译日志, 发现有warning:

```
configure: WARNING: No frame pointers and no libunwind. Using experimental backtrace capturing via libgcc. Expect crashy cpu profiler.
``` 

安装libunwind-dev, 编译, 性能未见改善

# 结论

通过调试发现: 

将 jemalloc 和 tcmalloc 都调整成 backtracer = libgcc

两者对 _Unwind_Backtrace 调用次数有很大差异

发现 jemalloc 带有采样间隔 lg_prof_sample, 默认为 512 KiB, 意为: 每分配512 KiB, 进行一次采样

将 lg_prof_sample 调为0, 则jemalloc速度也很慢
