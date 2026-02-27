---
title: 20210819 - MySQL 内存, 除了p_s统计的, 还有哪些 2
confluence_page_id: 1343639
created_at: 2021-08-19T15:12:56+00:00
updated_at: 2021-08-19T15:48:50+00:00
---

接 [20210724 - MySQL 内存, 除了p_s统计的, 还有哪些]

# 脚本

使用bcc, 改造其memleak脚本, 加入对pfs_memory_alloc_vc的观察, 使得malloc与pfs_memory_alloc_vc的观察相互抵消, 剩下的就是未被p_s统计的部分

脚本: 

```
#! /usr/bin/python2
#
# memleak   Trace and display outstanding allocations to detect
#           memory leaks in user-mode processes and the kernel.
#
# USAGE: memleak [-h] [-p PID] [-t] [-a] [-o OLDER] [-c COMMAND]
#                [--combined-only] [-s SAMPLE_RATE] [-T TOP] [-z MIN_SIZE]
#                [-Z MAX_SIZE] [-O OBJ]
#                [interval] [count]
#
# Licensed under the Apache License, Version 2.0 (the "License")
# Copyright (C) 2016 Sasha Goldshtein.

from bcc import BPF
from time import sleep
from datetime import datetime
import resource
import argparse
import subprocess
import os

class Allocation(object):
    def __init__(self, stack, size, ps_alloc_size):
        self.stack = stack
        self.count = 1
        self.size = size
        self.ps_alloc_size = ps_alloc_size

    def update(self, size, ps_alloc_size):
        self.count += 1
        self.size += size
        self.ps_alloc_size += ps_alloc_size

examples = """
EXAMPLES:

./memleak -p $(pidof allocs)
        Trace allocations and display a summary of "leaked" (outstanding)
        allocations every 5 seconds
./memleak -p $(pidof allocs) -t
        Trace allocations and display each individual allocator function call
./memleak -ap $(pidof allocs) 10
        Trace allocations and display allocated addresses, sizes, and stacks
        every 10 seconds for outstanding allocations
./memleak
        Trace allocations in kernel mode and display a summary of outstanding
        allocations every 5 seconds
./memleak -o 60000
        Trace allocations in kernel mode and display a summary of outstanding
        allocations that are at least one minute (60 seconds) old
./memleak -s 5
        Trace roughly every 5th allocation, to reduce overhead
"""

description = """
Trace outstanding memory allocations that weren't freed.
Supports both user-mode allocations made with libc functions and kernel-mode
allocations made with kmalloc/kmem_cache_alloc/get_free_pages and corresponding
memory release functions.
"""

parser = argparse.ArgumentParser(description=description,
        formatter_class=argparse.RawDescriptionHelpFormatter,
        epilog=examples)
parser.add_argument("-p", "--pid", type=int, default=-1,
        help="the PID to trace; if not specified, trace kernel allocs")
parser.add_argument("-t", "--trace", action="store_true",
        help="print trace messages for each alloc/free call")
parser.add_argument("interval", nargs="?", default=5, type=int,
        help="interval in seconds to print outstanding allocations")
parser.add_argument("count", nargs="?", type=int,
        help="number of times to print the report before exiting")
parser.add_argument("-a", "--show-allocs", default=False, action="store_true",
        help="show allocation addresses and sizes as well as call stacks")
parser.add_argument("-o", "--older", default=500, type=int,
        help="prune allocations younger than this age in milliseconds")
parser.add_argument("--combined-only", default=False, action="store_true",
        help="show combined allocation statistics only")
parser.add_argument("-s", "--sample-rate", default=1, type=int,
        help="sample every N-th allocation to decrease the overhead")
parser.add_argument("-T", "--top", type=int, default=10,
        help="display only this many top allocating stacks (by size)")
parser.add_argument("-z", "--min-size", type=int,
        help="capture only allocations larger than this size")
parser.add_argument("-Z", "--max-size", type=int,
        help="capture only allocations smaller than this size")
parser.add_argument("-O", "--obj", type=str, default="c",
        help="attach to allocator functions in the specified object")
parser.add_argument("-x", "--exe", type=str,
    dest="path", metavar="PATH", help="path to binary")

args = parser.parse_args()

pid = args.pid
trace_all = args.trace
interval = args.interval
min_age_ns = 1e6 * args.older
sample_every_n = args.sample_rate
num_prints = args.count
top_stacks = args.top
min_size = args.min_size
max_size = args.max_size
obj = args.obj
mysqld_path = args.path

if pid == -1:
        print("pid is required")
        exit(1)
if mysqld_path is None:
        print("path is required")
        exit(1)
if min_size is not None and max_size is not None and min_size > max_size:
        print("min_size (-z) can't be greater than max_size (-Z)")
        exit(1)

bpf_source = """
#include <uapi/linux/ptrace.h>
#define USDT

struct alloc_info_t {
        u64 size;
        u64 timestamp_ns;
        int stack_id;
        u64 ps_alloc_size;
};

struct combined_alloc_info_t {
        u64 total_size;
        u64 number_of_allocs;
        u64 ps_alloc_total_size;
};

BPF_HASH(sizes, u64);
BPF_HASH(current_alloc_addrs, u64);
BPF_TABLE("hash", u64, struct alloc_info_t, allocs, 1000000);
BPF_HASH(memptrs, u64, u64);
BPF_STACK_TRACE(stack_traces, 10240)
BPF_TABLE("hash", u64, struct combined_alloc_info_t, combined_allocs, 10240);

static inline void update_statistics_add(u64 stack_id, u64 sz) {
        struct combined_alloc_info_t *existing_cinfo;
        struct combined_alloc_info_t cinfo = {0};

        existing_cinfo = combined_allocs.lookup(&stack_id);
        if (existing_cinfo != 0)
                cinfo = *existing_cinfo;

        cinfo.total_size += sz;
        cinfo.number_of_allocs += 1;

        combined_allocs.update(&stack_id, &cinfo);
}

static inline void update_statistics_add_ps_alloc(u64 stack_id, u64 sz) {
        struct combined_alloc_info_t *existing_cinfo;
        struct combined_alloc_info_t cinfo = {0};

        existing_cinfo = combined_allocs.lookup(&stack_id);
        if (existing_cinfo != 0)
                cinfo = *existing_cinfo;

        cinfo.ps_alloc_total_size += sz;

        combined_allocs.update(&stack_id, &cinfo);
}

static inline void update_statistics_del(u64 stack_id, u64 sz) {
        struct combined_alloc_info_t *existing_cinfo;
        struct combined_alloc_info_t cinfo = {0};

        existing_cinfo = combined_allocs.lookup(&stack_id);
        if (existing_cinfo != 0)
                cinfo = *existing_cinfo;

        if (sz >= cinfo.total_size) {
                cinfo.total_size = 0;
                cinfo.ps_alloc_total_size = 0;
        } else {
                cinfo.total_size -= sz;
                cinfo.ps_alloc_total_size -= sz;
        }

        if (cinfo.number_of_allocs > 0)
                cinfo.number_of_allocs -= 1;

        combined_allocs.update(&stack_id, &cinfo);
}

static inline int gen_alloc_enter(struct pt_regs *ctx, size_t size) {
        SIZE_FILTER
        if (SAMPLE_EVERY_N > 1) {
                u64 ts = bpf_ktime_get_ns();
                if (ts % SAMPLE_EVERY_N != 0)
                        return 0;
        }

        u64 pid = bpf_get_current_pid_tgid();
        u64 size64 = size;
        sizes.update(&pid, &size64);

        if (SHOULD_PRINT)
                bpf_trace_printk("alloc entered, size = %u\\n", size);
        return 0;
}

static inline int gen_alloc_exit2(struct pt_regs *ctx, u64 address) {
        u64 pid = bpf_get_current_pid_tgid();
        u64* size64 = sizes.lookup(&pid);
        struct alloc_info_t info = {0};

        if (size64 == 0)
                return 0; // missed alloc entry

        info.size = *size64;
        sizes.delete(&pid);

        info.timestamp_ns = bpf_ktime_get_ns();
        info.stack_id = stack_traces.get_stackid(ctx, STACK_FLAGS);
        allocs.update(&address, &info);
        update_statistics_add(info.stack_id, info.size);

        current_alloc_addrs.update(&pid, &address);

        if (SHOULD_PRINT) {
                bpf_trace_printk("alloc exited, size = %lu, result = %lx\\n",
                                 info.size, address);
        }
        return 0;
}

static inline int gen_alloc_exit(struct pt_regs *ctx) {
        return gen_alloc_exit2(ctx, PT_REGS_RC(ctx));
}

static inline int gen_free_enter(struct pt_regs *ctx, void *address) {
        u64 pid = bpf_get_current_pid_tgid();
        current_alloc_addrs.delete(&pid);

        u64 addr = (u64)address;
        struct alloc_info_t *info = allocs.lookup(&addr);
        if (info == 0)
                return 0;

        allocs.delete(&addr);
        update_statistics_del(info->stack_id, info->size);

        if (SHOULD_PRINT) {
                bpf_trace_printk("free entered, address = %lx, size = %lu\\n",
                                 address, info->size);
        }
        return 0;
}

int pfs_memory_alloc_vc_start(struct pt_regs *ctx) {
        u64 pid = bpf_get_current_pid_tgid();
        u64 size = (u64) PT_REGS_PARM2(ctx);

        u64 *addr_ptr = current_alloc_addrs.lookup(&pid);
        if (addr_ptr == 0)
                return 0;
        u64 addr = *addr_ptr;

        struct alloc_info_t *info_ptr = allocs.lookup(&addr);
        if (info_ptr == 0)
                return 0;

        struct alloc_info_t info = *info_ptr;
        info.ps_alloc_size = size;
        allocs.update(&addr, &info);

        update_statistics_add_ps_alloc(addr, size);

        if (SHOULD_PRINT)
                bpf_trace_printk("pfs_memory_alloc_vc entered, size = %u\\n", size);
        return 0;
}

int malloc_enter(struct pt_regs *ctx, size_t size) {
        return gen_alloc_enter(ctx, size);
}

int malloc_exit(struct pt_regs *ctx) {
        return gen_alloc_exit(ctx);
}

int free_enter(struct pt_regs *ctx, void *address) {
        return gen_free_enter(ctx, address);
}

int calloc_enter(struct pt_regs *ctx, size_t nmemb, size_t size) {
        return gen_alloc_enter(ctx, nmemb * size);
}

int calloc_exit(struct pt_regs *ctx) {
        return gen_alloc_exit(ctx);
}

int realloc_enter(struct pt_regs *ctx, void *ptr, size_t size) {
        gen_free_enter(ctx, ptr);
        return gen_alloc_enter(ctx, size);
}

int realloc_exit(struct pt_regs *ctx) {
        return gen_alloc_exit(ctx);
}

int posix_memalign_enter(struct pt_regs *ctx, void **memptr, size_t alignment,
                         size_t size) {
        u64 memptr64 = (u64)(size_t)memptr;
        u64 pid = bpf_get_current_pid_tgid();

        memptrs.update(&pid, &memptr64);
        return gen_alloc_enter(ctx, size);
}

int posix_memalign_exit(struct pt_regs *ctx) {
        u64 pid = bpf_get_current_pid_tgid();
        u64 *memptr64 = memptrs.lookup(&pid);
        void *addr;

        if (memptr64 == 0)
                return 0;

        memptrs.delete(&pid);

        if (bpf_probe_read(&addr, sizeof(void*), (void*)(size_t)*memptr64))
                return 0;

        u64 addr64 = (u64)(size_t)addr;
        return gen_alloc_exit2(ctx, addr64);
}

int aligned_alloc_enter(struct pt_regs *ctx, size_t alignment, size_t size) {
        return gen_alloc_enter(ctx, size);
}

int aligned_alloc_exit(struct pt_regs *ctx) {
        return gen_alloc_exit(ctx);
}

int valloc_enter(struct pt_regs *ctx, size_t size) {
        return gen_alloc_enter(ctx, size);
}

int valloc_exit(struct pt_regs *ctx) {
        return gen_alloc_exit(ctx);
}

int memalign_enter(struct pt_regs *ctx, size_t alignment, size_t size) {
        return gen_alloc_enter(ctx, size);
}

int memalign_exit(struct pt_regs *ctx) {
        return gen_alloc_exit(ctx);
}

int pvalloc_enter(struct pt_regs *ctx, size_t size) {
        return gen_alloc_enter(ctx, size);
}

int pvalloc_exit(struct pt_regs *ctx) {
        return gen_alloc_exit(ctx);
}
"""

bpf_source = bpf_source.replace("SHOULD_PRINT", "1" if trace_all else "0")
bpf_source = bpf_source.replace("SAMPLE_EVERY_N", str(sample_every_n))

size_filter = ""
if min_size is not None and max_size is not None:
        size_filter = "if (size < %d || size > %d) return 0;" % \
                      (min_size, max_size)
elif min_size is not None:
        size_filter = "if (size < %d) return 0;" % min_size
elif max_size is not None:
        size_filter = "if (size > %d) return 0;" % max_size
bpf_source = bpf_source.replace("SIZE_FILTER", size_filter)

stack_flags = "BPF_F_REUSE_STACKID"
stack_flags += "|BPF_F_USER_STACK"
bpf_source = bpf_source.replace("STACK_FLAGS", stack_flags)

bpf = BPF(text=bpf_source)

print("Attaching to pid %d, Ctrl+C to quit." % pid)

def attach_probes(sym, fn_prefix=None, can_fail=False):
        if fn_prefix is None:
                fn_prefix = sym

        try:
                bpf.attach_uprobe(name=obj, sym=sym,
                                  fn_name=fn_prefix + "_enter",
                                  pid=pid)
                bpf.attach_uretprobe(name=obj, sym=sym,
                                     fn_name=fn_prefix + "_exit",
                                     pid=pid)
        except Exception:
                if can_fail:
                        return
                else:
                        raise

attach_probes("malloc")
attach_probes("calloc")
attach_probes("realloc")
attach_probes("posix_memalign")
attach_probes("valloc")
attach_probes("memalign")
attach_probes("pvalloc")
attach_probes("aligned_alloc", can_fail=True)  # added in C11
bpf.attach_uprobe(name=obj, sym="free", fn_name="free_enter",
                          pid=pid)

regex = "\\w+pfs_memory_alloc_vc\\w+"
symbols = BPF.get_user_functions_and_addresses(mysqld_path, regex)
if len(symbols) == 0:
    print("Can't find function 'pfs_memory_alloc_vc' in %s" % (mysqld_path))
    exit(1)

(mysql_func_name, mysql_func_addr) = symbols[0]
bpf.attach_uprobe(name=mysqld_path, sym=mysql_func_name,
                  fn_name="pfs_memory_alloc_vc_start")

def print_outstanding():
        print("[%s] Top %d stacks with outstanding allocations:" %
              (datetime.now().strftime("%H:%M:%S"), top_stacks))
        alloc_info = {}
        allocs = bpf["allocs"]
        stack_traces = bpf["stack_traces"]
        for address, info in sorted(allocs.items(), key=lambda a: a[1].size):
                if BPF.monotonic_time() - min_age_ns < info.timestamp_ns:
                        continue
                if info.stack_id < 0:
                        continue
                if info.stack_id in alloc_info:
                        alloc_info[info.stack_id].update(info.size, info.ps_alloc_size)
                else:
                        stack = list(stack_traces.walk(info.stack_id))
                        combined = []
                        for addr in stack:
                                combined.append(bpf.sym(addr, pid,
                                        show_module=True, show_offset=True))
                        alloc_info[info.stack_id] = Allocation(combined,
                                                               info.size,
                                                               info.ps_alloc_size)
                if args.show_allocs:
                        print("\taddr = %x size = %s ps_alloc_size=%s" %
                              (address.value, info.size, info.ps_alloc_size))
        to_show = sorted(alloc_info.values(),
                         key=lambda a: a.size)[-top_stacks:]
        for alloc in to_show:
                ps_alloc_missing = ""
                if not (alloc.size == alloc.ps_alloc_size) and not (alloc.size == alloc.ps_alloc_size+32*alloc.count):
                    ps_alloc_missing = " (ps_alloc missing)"
                print("\t%d bytes in %d allocations from stack%s\n\t\t%s" %
                      (alloc.size, alloc.count, ps_alloc_missing, "\n\t\t".join(alloc.stack)))

def print_outstanding_combined():
        stack_traces = bpf["stack_traces"]
        stacks = sorted(bpf["combined_allocs"].items(),
                        key=lambda a: -a[1].total_size)
        cnt = 1
        entries = []
        for stack_id, info in stacks:
                try:
                        trace = []
                        for addr in stack_traces.walk(stack_id.value):
                                sym = bpf.sym(addr, pid,
                                                      show_module=True,
                                                      show_offset=True)
                                trace.append(sym)
                        trace = "\n\t\t".join(trace)
                except KeyError:
                        trace = "stack information lost"

                ps_alloc_missing = ""
                if not (info.total_size == info.ps_alloc_total_size) and not (info.total_size == info.ps_alloc_total_size+32*info.number_of_allocs):
                    ps_alloc_missing = " (ps_alloc missing)"

                entry = ("\t%d bytes in %d allocations from stack%s\n\t\t%s" %
                         (info.total_size, info.number_of_allocs, ps_alloc_missing, trace))
                entries.append(entry)

                cnt += 1
                if cnt > top_stacks:
                        break

        print("[%s] Top %d stacks with outstanding allocations:" %
              (datetime.now().strftime("%H:%M:%S"), top_stacks))

        print('\n'.join(reversed(entries)))

count_so_far = 0
while True:
        if trace_all:
                print(bpf.trace_fields())
        else:
                try:
                        sleep(interval)
                except KeyboardInterrupt:
                        exit()
                if args.combined_only:
                        print_outstanding_combined()
                else:
                        print_outstanding()
                count_so_far += 1
                if num_prints is not None and count_so_far >= num_prints:
                        exit()

``` 

# 堆栈1

存储writeset的容器, 由glibc自动扩容

```
	196608 bytes in 1 allocations from stack (ps_alloc missing+)
		operator new(unsigned long)+0x18 [libstdc++.so.6.0.25]
		Rpl_transaction_write_set_ctx::add_write_set(unsigned long)+0x105 [mysqld]
		add_pke(TABLE*, THD*, unsigned char*)+0x8db [mysqld]
		binlog_log_row(TABLE*, unsigned char const*, unsigned char const*, bool (*)(THD*, TABLE*, bool, unsigned char const*, unsigned char const*))+0x215 [mysqld]
		handler::ha_write_row(unsigned char*)+0x148 [mysqld]
		write_record(THD*, TABLE*, COPY_INFO*, COPY_INFO*)+0x5f5 [mysqld]
		Query_result_insert::send_data(THD*, mem_root_deque<Item*> const&)+0xea [mysqld]
		Query_expression::ExecuteIteratorQuery(THD*)+0x3a2 [mysqld]
		Query_expression::execute(THD*)+0x2c [mysqld]
		Sql_cmd_dml::execute_inner(THD*)+0x3ab [mysqld]
		Sql_cmd_dml::execute(THD*)+0x698 [mysqld]
		mysql_execute_command(THD*, bool)+0xab8 [mysqld]
		dispatch_sql_command(THD*, Parser_state*)+0x418 [mysqld]
		dispatch_command(THD*, COM_DATA const*, enum_server_command)+0x16c1 [mysqld]
		do_command(THD*)+0x174 [mysqld]
		handle_connection+0x248 [mysqld]
		pfs_spawn_thread+0x15c [mysqld]
		start_thread+0xdb [libpthread-2.27.so]
```
