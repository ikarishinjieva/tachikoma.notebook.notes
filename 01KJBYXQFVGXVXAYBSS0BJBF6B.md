---
title: 20220726 - TensorDB gpu 诊断
confluence_page_id: 1933427
created_at: 2022-07-26T02:10:56+00:00
updated_at: 2022-08-04T11:31:42+00:00
---

# 目标

用工具 NVIDIA® Nsight™ Compute 诊断tensorDB的gpu使用, 解释相关观测项

# TensorDB使用

<http://10.186.18.21/tensordb/tensordb/-/tree/0.22.07.99>

预训练: 

  - python main.py --dataset=ann_1m_sq8 --algorithm=ntq_bf --train_database 

  - 用不到gpu, 需要十几分钟

计算 (要诊断的目标函数): 

  - python main.py --dataset=ann_1m_sq8 --algorithm=ntq_bf --query_size=500 --gpu --recall

    - 日志

```
2022-07-26 02:10:17,397 ｜ INFO ｜ logic.py ｜ search_from_database ｜ 136 ｜ time_all = 3.40189790725708
2022-07-26 02:10:17,397 ｜ INFO ｜ logic.py ｜ search_from_database ｜ 137 ｜ time_query_cost = 2.936005115509033
2022-07-26 02:10:18,907 ｜ INFO ｜ logic.py ｜ compute_recall ｜ 193 ｜ topK=1000,  total precision=99.905%
2022-07-26 02:10:18,912 ｜ INFO ｜ logic.py ｜ search_from_database ｜ 143 ｜ compute_recall_time_cost :1.5152885913848877
``` 
  - python main.py --dataset=ann_1m_sq8 --algorithm=ntq_bf --query_size=500 --recall 不使用gpu的计算
    - 日志

```
...
2022-07-26 01:53:59,532 ｜ INFO ｜ logic.py ｜ search_from_database ｜ 136 ｜ time_all = 77.0284035205841
2022-07-26 01:53:59,532 ｜ INFO ｜ logic.py ｜ search_from_database ｜ 137 ｜ time_query_cost = 77.02717018127441
2022-07-26 01:54:01,021 ｜ INFO ｜ logic.py ｜ compute_recall ｜ 193 ｜ topK=1000,  total precision=99.978%
2022-07-26 01:54:01,026 ｜ INFO ｜ logic.py ｜ search_from_database ｜ 143 ｜ compute_recall_time_cost :1.4945480823516846
``` 

# Nsight使用

注意: 需要指定环境变量:

```
PATH=DISPLAY=:0;PATH=/root/.cargo/bin:/root/anaconda3/bin:/root/anaconda3/bin:/root/anaconda3/condabin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:/usr/local/cuda/bin
``` 

![image2022-7-26 17:55:48.png](/assets/01KJBYXQFVGXVXAYBSS0BJBF6B/image2022-7-26%2017%3A55%3A48.png)

使用手工profile命令 (需要执行很久): 

```
CUPY_CACHE_SAVE_CUDA_SOURCE=1 CUPY_CUDA_COMPILE_WITH_DEBUG=1 /tmp/var/target/linux-desktop-glibc_2_11_3-x64/ncu --export /tmp/var/cuda.out --force-overwrite --target-processes application-only --replay-mode kernel --kernel-name-base function --launch-skip-before-match 0 --section-folder /tmp/var/sections --section LaunchStats --section Occupancy --section SpeedOfLight --section MemoryWorkloadAnalysis --sampling-interval 30 --sampling-max-passes 5 --sampling-buffer-size 33554432 --profile-from-start 1 --cache-control all --clock-control base --rule LaunchConfiguration --rule Occupancy --rule SOLBottleneck --import-source no --check-exit-code yes -k gpu_hamming_core  python main.py  --dataset=ann_1m_sq8  --algorithm=ntq_bf  --query_size=10  --gpu  --recall
``` 

获取结果文件: /tmp/var/cuda.out.ncu-rep

可以从NVIDIA Nsight compute打开: 

![image2022-7-26 17:57:6.png](/assets/01KJBYXQFVGXVXAYBSS0BJBF6B/image2022-7-26%2017%3A57%3A6.png)

查看Occupancy Calculator输出: 

![image2022-7-26 18:6:27.png](/assets/01KJBYXQFVGXVXAYBSS0BJBF6B/image2022-7-26%2018%3A6%3A27.png)

点开时间占比最高的函数: 

![image2022-7-26 18:7:15.png](/assets/01KJBYXQFVGXVXAYBSS0BJBF6B/image2022-7-26%2018%3A7%3A15.png)

![image2022-7-26 18:9:7.png](/assets/01KJBYXQFVGXVXAYBSS0BJBF6B/image2022-7-26%2018%3A9%3A7.png)

# 对Occupancy Calculator输出的解释

![image](http://8.134.54.170:8330/download/attachments/1933427/image2022-7-26%2018%3A6%3A27.png?version=1&modificationDate=1658829987576&api=v2)

  - 资料: 
    - <https://www.nvidia.com/content/PDF/fermi_white_papers/NVIDIA_Fermi_Compute_Architecture_Whitepaper.pdf>
    - <https://geniusturtle6174.github.io/tutor_CUDA/occupancy.htm>
    - <https://www.cnblogs.com/cuancuancuanhao/p/7789224.html>
  - 输入项:
    - Compute Capability: SM的版本
    - Shared Memory Size Config: 65535 bytes: 每个SM最多包括的shared memory
    - Threads per block: 256: 程序中 每个block包括的线程数量
    - Registers Per thread: 44: 每个线程使用的寄存器数量
    - User Shared Memory per Block (bytes): 每个block使用的shared memory数量
  - 输出1: Physical Limit of GPU(7.5): GPU的硬件能力限制
    - Multiprocessor = StreamingMultiprocessor (SM), 包括: 
      - ![image2022-7-27 11:9:13.png](/assets/01KJBYXQFVGXVXAYBSS0BJBF6B/image2022-7-27%2011%3A9%3A13.png)
      - CUDA Cores
      - Thread Block (线程块, 不依赖于其他线程块的线程集合)
        - (Max Thread Blocks per Multiprocessor = 16)
        - (Maximum Thread Block Size =1024)
        - Warp (线程束)
          - (Max warps per Multiprocessor = 32)  

          - (Warp Allocation Granularity = 4, warp申请基数, 每次申请都需要是4的倍数)
          - Thread (线程)
            - (Threads per warp = 32)
            - (Max Threads per Multiprocessor = 32*32 = 1024)  

      - Register File
        - (Registers per Multiprocessor = 65536)
        - (Max Registers per Thread Block = 65536)
        - (Max Registers per Thread = 255)
        - (Register Allocation Unit Size = 256, 寄存器申请数量基数)
        - (Register Allocation Granularity = warp, 寄存器申请粒度, 一直为warp)
      - Share memory
        - (Shared Memory per Multiprocessor = 32768 bytes) 
        - (Max Shared Memory per Block = 32768 bytes) 
        - (Shared Memory Allocation Unit Size = 256, Shard Memory申请数量基数)
        - (Shared Memory Per Block (CUDA runtime use) = 0 bytes, 不明)
  - 输出2: Occupancy Data: 占用率
    - (Active Threads per Multiprocessor): SM的活跃线程数
    - (Active Warps per Multiprocessor): SM的活跃Warp数
    - (Active Thread Blocks per Multiprocessor): SM的活跃Block数
    - (Occupancy of each Multiprocessor): SM的占用率
  - 输出3: Allocated Resources
    - Warps: 每个SM能申请出的Warp数 (受到Threads数量限制)
      - Per Block: 每个Block中, 有多少Warps
      - Limit Per SM: 每个SM中, 最大有多少Warps
      - Allocatable Blocks Per SM: 每个SM可申请出多少Block
    - Registers: 每个SM能申请出的Warp数 (受到寄存器数量限制)
      - Per Block: 每个Block中, 有多少Warps
      - Limit Per SM: 每个SM中, 最大有多少Warps (Registers per Multiprocessor / Registers Per Thread / Threads per Wrap)
      - Allocatable Blocks Per SM: 每个SM可申请出多少Block
    - Shared Memory: 每个SM能申请出的Warp数 (受到共享内存数量限制)
      - Per Block: 每个Block中, 有多少Warps
      - Limit Per SM: 每个SM中, 最大有多少Warps: 与registers的计算公式类似
      - Allocatable Blocks Per SM: 每个SM可申请出多少Block
  - 输出4: Occupancy Limiter: 
    - Max Warps or Max Blocks per Multiprocessor: 由于Max Warps或者Max Blocks的限制
      - Blocks per SM: 每个SM能分配出的Block
    - Registers per Multiprocessor: 由于寄存器数量的限制
      - Blocks per SM: 每个SM能分配出的Block
    - Shared Memory per Multiprocessor: 由于共享内存数量的限制
      - Blocks per SM: 每个SM能分配出的Block

# 对Memory Throughput Breakdown的解读

  - 参考: 
    - <https://docs.nvidia.com/nsight-compute/ProfilingGuide/>
    - <https://ec.europa.eu/programmes/erasmus-plus/project-result-content/52dfac24-28e9-4379-8f28-f8ed05e225e0/lec03_gpu_architectures.pdf>
      - Work-items are automatically grouped into hardware threads called “wavefronts” (AMD) or “warps” (NVIDIA)

    - <https://docs.nvidia.com/nsight-graphics/AdvancedLearning/index.html#gt_l1_throughput>
      - ![image](https://docs.nvidia.com/nsight-graphics/AdvancedLearning/graphics/gt_l1_throughput.PNG)
  - L1: Data Pipe Lsu Wavefronts [%]: 完成数据阶段的 warps 的数量 / 峰值
    - LSU = load/store Unit
    - wavefront = warps
    - ![image2022-7-28 10:57:53.png](/assets/01KJBYXQFVGXVXAYBSS0BJBF6B/image2022-7-28%2010%3A57%3A53.png)
    - 其监控项为: l1tex__data_pipe_lsu_wavefronts.avg.pct_of_peak_sustained_elapsed
    - (关于监控项命名规则, 参考: <https://forums.developer.nvidia.com/t/metric-references-and-description/111750/2>)  
  

  - L1: Lsu Writeback Active [%]: writeback开启的时长/峰值
    - ![image2022-7-28 11:3:25.png](/assets/01KJBYXQFVGXVXAYBSS0BJBF6B/image2022-7-28%2011%3A3%3A25.png)
  - L1: Lsuin Requests [%]: LSU接受/处理指令的数量/峰值
    - ![image2022-7-28 11:6:29.png](/assets/01KJBYXQFVGXVXAYBSS0BJBF6B/image2022-7-28%2011%3A6%3A29.png)
  - ...

# 对Compute Throughput Breakdown的解读

  - SM: Inst Executed Pipe Lsu [%], 通过LSU流水线执行的warp指令数/峰值
    - ![image2022-7-28 11:32:37.png](/assets/01KJBYXQFVGXVXAYBSS0BJBF6B/image2022-7-28%2011%3A32%3A37.png)
  - SM: Pipe Alu Cycles Active [%], ALU流水线的工作时长/峰值  

    - ![image2022-7-28 11:33:40.png](/assets/01KJBYXQFVGXVXAYBSS0BJBF6B/image2022-7-28%2011%3A33%3A40.png)

# Launch Statistics

  - ![image2022-7-28 20:14:52.png](/assets/01KJBYXQFVGXVXAYBSS0BJBF6B/image2022-7-28%2020%3A14%3A52.png)
    - Grid Size
      - ![image2022-7-28 20:17:20.png](/assets/01KJBYXQFVGXVXAYBSS0BJBF6B/image2022-7-28%2020%3A17%3A20.png)
      - 发往kernel的block数量 ??
      - grid是block构成的矩阵
    - Block Size
      - ![image2022-7-28 20:45:27.png](/assets/01KJBYXQFVGXVXAYBSS0BJBF6B/image2022-7-28%2020%3A45%3A27.png)
      - block是thread构成的矩阵
    - Threads  

      - ![image2022-7-28 20:51:3.png](/assets/01KJBYXQFVGXVXAYBSS0BJBF6B/image2022-7-28%2020%3A51%3A3.png)
      - 总线程数 = Grid Size * Block Size
    - Waves Per SM  

      - ![image2022-7-28 20:54:23.png](/assets/01KJBYXQFVGXVXAYBSS0BJBF6B/image2022-7-28%2020%3A54%3A23.png)
      - 对于每个SM, 下发block的轮数
      - 参考: <https://developer.nvidia.com/blog/cuda-pro-tip-minimize-the-tail-effect/>
        - 每个wave, 会并行下发若干个block

# Source页 - 诊断 Live Registers

![image2022-7-29 11:31:32.png](/assets/01KJBYXQFVGXVXAYBSS0BJBF6B/image2022-7-29%2011%3A31%3A32.png)

  - 需要找到将SASS对应到源码中
    - CUPY_CACHE_SAVE_CUDA_SOURCE=1 python ..., 可以生成CUDA的源码
      - 可以在~/.cupy/kernel_cache中, 先删除所有cache, 然后执行以上命令, 可以看到源码: 
        - ![image2022-7-29 17:30:21.png](/assets/01KJBYXQFVGXVXAYBSS0BJBF6B/image2022-7-29%2017%3A30%3A21.png)
    - CUPY_CUDA_COMPILE_WITH_DEBUG=1 python ..., 可以生成带debug信息的cubin包
      - 需要先删除缓存
      - 执行时间从 1s 变成 60s, 对性能影响严重
    - 带有 CUPY_CUDA_COMPILE_WITH_DEBUG, 再次运行ncu, 可以看到源文件: 
      - ![image2022-7-29 18:46:42.png](/assets/01KJBYXQFVGXVXAYBSS0BJBF6B/image2022-7-29%2018%3A46%3A42.png)

# 调整不同的THREADS_PER_BLOCK, 观察结果

## THREADS_PER_BLOCK = 16

![image2022-8-1 15:33:42.png](/assets/01KJBYXQFVGXVXAYBSS0BJBF6B/image2022-8-1%2015%3A33%3A42.png)

可以识别Block Size问题

## THREADS_PER_BLOCK = 32

![image2022-8-1 15:36:23.png](/assets/01KJBYXQFVGXVXAYBSS0BJBF6B/image2022-8-1%2015%3A36%3A23.png)

可以识别是由于SM上支持的Block数不足: 

![image2022-8-1 15:39:20.png](/assets/01KJBYXQFVGXVXAYBSS0BJBF6B/image2022-8-1%2015%3A39%3A20.png)

## THREADS_PER_BLOCK = 64

![image2022-8-1 15:40:5.png](/assets/01KJBYXQFVGXVXAYBSS0BJBF6B/image2022-8-1%2015%3A40%3A5.png)

可以识别出由于Register限制, 限制了block的并发数量: 

![image2022-8-1 18:35:40.png](/assets/01KJBYXQFVGXVXAYBSS0BJBF6B/image2022-8-1%2018%3A35%3A40.png)

register限制的算法: 

  - Registers per Multiprocessor = 65536
  - Register Per Thread = 74
  - Threads Per Block = 64
  - Register Allocation Unit Size = 256, Register Allocation Granularity = warp
    - 每个warp按256为粒度, 分配register
    - register per warp = 74 * 32 = 2368 (向上取整 256) = 2560
    - register per block = 2560 * 64/32 = 5120
    - block per multiprocessor (limited by register) = 65536/5120 = 12.8 (向下取整) = 12

发现range出来的结果是long long, 使用了过多的寄存器: 

![image2022-8-1 19:10:56.png](/assets/01KJBYXQFVGXVXAYBSS0BJBF6B/image2022-8-1%2019%3A10%3A56.png)

改变range的写法: 

![image2022-8-1 19:9:57.png](/assets/01KJBYXQFVGXVXAYBSS0BJBF6B/image2022-8-1%2019%3A9%3A57.png)

重新采样后, register数量下降: 

![image2022-8-1 19:11:34.png](/assets/01KJBYXQFVGXVXAYBSS0BJBF6B/image2022-8-1%2019%3A11%3A34.png)

## 诊断Memory Throughput

识别出Memory throughput和compute throughput偏差过大, 认为是内存使用不当:

![image2022-8-2 0:9:49.png](/assets/01KJBYXQFVGXVXAYBSS0BJBF6B/image2022-8-2%200%3A9%3A49.png)

在ncu命令中添加 --section MemoryWorkloadAnalysis: 

![image2022-8-2 0:9:17.png](/assets/01KJBYXQFVGXVXAYBSS0BJBF6B/image2022-8-2%200%3A9%3A17.png)

分析报告中会多出以下部分: 

![image2022-8-2 0:18:25.png](/assets/01KJBYXQFVGXVXAYBSS0BJBF6B/image2022-8-2%200%3A18%3A25.png)

内存模型: 

  - <https://docs.nvidia.com/nsight-compute/ProfilingGuide/>
  - SM -> L1
    - ![image](https://docs.nvidia.com/nsight-compute/ProfilingGuide/graphics/hw-model-l1tex.png)
  - L1 -> L2(LTS) -> Memory
    - ![image](https://docs.nvidia.com/nsight-compute/ProfilingGuide/graphics/hw-model-lts.png)

字段解释:

  - Mem Busy [%]: 内存系统的繁忙程度
    - ![image2022-8-2 11:17:31.png](/assets/01KJBYXQFVGXVXAYBSS0BJBF6B/image2022-8-2%2011%3A17%3A31.png)
      - 通过命令, 可以拆分该监控项:

```
/tmp/var/target/linux-desktop-glibc_2_11_3-x64/ncu --metrics breakdown:gpu__compute_memory_access_throughput.avg.pct_of_peak_sustained_elapsed  --print-rule-details --details-all -c 1 python main.py  --dataset=ann_1m_sq8  --algorithm=ntq_bf  --query_size=10  --gpu  --recall
...
[28869] python3.9@127.0.0.1
  cupy_copy__int64_uint16, 2022-Aug-02 03:37:38, Context 1, Stream 7
    Section: Command line profiler metrics
    ---------------------------------------------------------------------- --------------- ------------------------------
    l1tex__data_bank_reads.avg.pct_of_peak_sustained_elapsed                             %                           0.01
    l1tex__data_bank_writes.avg.pct_of_peak_sustained_elapsed                            %                           0.01
    l1tex__data_pipe_lsu_wavefronts.avg.pct_of_peak_sustained_elapsed                    %                           0.04
    l1tex__data_pipe_tex_wavefronts.avg.pct_of_peak_sustained_elapsed                    %                              0
    l1tex__f_wavefronts.avg.pct_of_peak_sustained_elapsed                                %                           0.13
    lts__d_atomic_input_cycles_active.avg.pct_of_peak_sustained_elapsed                  %                              0
    lts__d_sectors.avg.pct_of_peak_sustained_elapsed                                     %                           0.14
    lts__t_sectors.avg.pct_of_peak_sustained_elapsed                                     %                           0.40
    lts__t_tag_requests.avg.pct_of_peak_sustained_elapsed                                %                           0.13
    ---------------------------------------------------------------------- --------------- ------------------------------
```

    -   - Mem Pipes Busy [%]: SM发出的内存指令的繁忙程度
    - ![image2022-8-2 11:18:15.png](/assets/01KJBYXQFVGXVXAYBSS0BJBF6B/image2022-8-2%2011%3A18%3A15.png)
    - 通过命令, 可以拆分该监控项:

```
/tmp/var/target/linux-desktop-glibc_2_11_3-x64/ncu --metrics breakdown:sm__memory_throughput.avg.pct_of_peak_sustained_elapsed --details-all -c 1 python main.py  --dataset=ann_1m_sq8  --algorithm=ntq_bf  --query_size=10  --gpu  --recall
 
...
 
[28527] python3.9@127.0.0.1
  cupy_copy__int64_uint16, 2022-Aug-02 03:34:56, Context 1, Stream 7
    Section: Command line profiler metrics
    ---------------------------------------------------------------------- --------------- ------------------------------
    idc__request_cycles_active.avg.pct_of_peak_sustained_elapsed                         %                              0
    sm__inst_executed_pipe_adu.avg.pct_of_peak_sustained_elapsed                         %                           0.02
    sm__inst_executed_pipe_ipa.avg.pct_of_peak_sustained_elapsed                         %                              0
    sm__inst_executed_pipe_lsu.avg.pct_of_peak_sustained_elapsed                         %                           0.05
    sm__inst_executed_pipe_tex.avg.pct_of_peak_sustained_elapsed                         %                              0
    sm__mio2rf_writeback_active.avg.pct_of_peak_sustained_elapsed                        %                           0.01
    sm__mio_pq_read_cycles_active.avg.pct_of_peak_sustained_elapsed                      %                           0.01
    sm__mio_pq_write_cycles_active.avg.pct_of_peak_sustained_elapsed                     %                           0.01
    ---------------------------------------------------------------------- --------------- ------------------------------
```

## 尝试优化Memory Throughput

发现cupy能力有限, 比如无法获得二维数组的某行 (无法使用bitwise_xor(array) 这类的函数进行批处理)

尝试使用 cupy User-Defined Kernels, 将kernel用c++代码嵌入

代码: 

```
def calc_hamming_use_gpu(query_bin, database_bin_gpu, count_1_table):
    hamming_kernel = cp.RawKernel(r'''
extern "C" __global__ 
void gpu_hamming_core(const unsigned char *query, int query_length, 
        const unsigned char *database, int database_length,
        const unsigned short *ct_device, 
        const unsigned short *dh, 
        int dim) {
}
''', 'gpu_hamming_core')
    with cp.cuda.Device(GPU_DEVICE_NUMBER):
        query_bin_gpu = cp.asarray(query_bin, dtype=cp.uint8)
        [n1, c] = cp.shape(query_bin_gpu)
        [n2, _] = cp.shape(database_bin_gpu)
        dh = cp.zeros((n1, n2), cp.uint16)
        threads_per_block = THREADS_PER_BLOCK
        blocks_per_grid = math.ceil(n2 / threads_per_block)
        # gpu_hamming_core[blocks_per_grid, threads_per_block](query_bin_gpu, database_bin_gpu, count_1_table, dh, c)
        hamming_kernel((blocks_per_grid,), (threads_per_block,), 
            (query_bin_gpu, n1, database_bin_gpu, n2, count_1_table, dh, c))
        cp.cuda.runtime.deviceSynchronize()
        return dh
``` 
    
    
    hamming_kernel没有实际计算的任务, 是个空函数, 在此情况, 使用ncu测定空函数的消耗: (由于源码使用错误, 此处block size变成了256, 后续调回64)

![image2022-8-3 13:30:22.png](/assets/01KJBYXQFVGXVXAYBSS0BJBF6B/image2022-8-3%2013%3A30%3A22.png)

空kernel的内存消耗很低

将python kernel转换到c++ kernel上: 

```
def calc_hamming_use_gpu(query_bin, database_bin_gpu, count_1_table):
    hamming_kernel = cp.RawKernel(r'''
extern "C" __global__ 
void gpu_hamming_core(const unsigned char *query, int query_length, 
        const unsigned char *database, int database_length,
        const unsigned short *ct_device, 
        unsigned short *dh, 
        int dim) {
    unsigned int tid = ((blockIdx.x * blockDim.x) + threadIdx.x);
    int i,j;
    unsigned short distance, x;
    for (i = 0; i < query_length; i++) {
        distance = 0;
        for (j = 0; j < dim; j++) {
            distance = distance + ct_device[query[i*dim+j] ^ database[tid*dim+j]];
        }
        dh[i*database_length + tid] = (unsigned short)(distance);
    }
}
''', 'gpu_hamming_core')
    with cp.cuda.Device(GPU_DEVICE_NUMBER):
        query_bin_gpu = cp.asarray(query_bin, dtype=cp.uint8)
        [n1, c] = cp.shape(query_bin_gpu)
        [n2, _] = cp.shape(database_bin_gpu)
        dh = cp.zeros((n1, n2), cp.uint16)
        threads_per_block = THREADS_PER_BLOCK
        blocks_per_grid = math.ceil(n2 / threads_per_block)
        # gpu_hamming_core[blocks_per_grid, threads_per_block](query_bin_gpu, database_bin_gpu, count_1_table, dh, c)
        hamming_kernel((blocks_per_grid,), (threads_per_block,), 
            (query_bin_gpu, n1, database_bin_gpu, n2, count_1_table, dh, c))
        cp.cuda.runtime.deviceSynchronize()
        return dh
``` 

使用ncu测定消耗: (由于源码使用错误, 此处block size变成了256, 后续调回64)

![image2022-8-3 14:16:29.png](/assets/01KJBYXQFVGXVXAYBSS0BJBF6B/image2022-8-3%2014%3A16%3A29.png)

经测定, kernel中对database的访问, 占用了大量的memory throughput. 对其进行缓存: 

```
def calc_hamming_use_gpu(query_bin, database_bin_gpu, count_1_table):
    hamming_kernel = cp.RawKernel(r'''
extern "C" __global__ 
void gpu_hamming_core(const unsigned char *query, int query_length, 
        const unsigned char *database, int database_length,
        const unsigned short *ct_device, 
        unsigned short *dh, 
        int dim) {
    unsigned int tid = ((blockIdx.x * blockDim.x) + threadIdx.x);

    unsigned char database_buf[1024];
    memcpy(database_buf, &database[tid*dim], sizeof(unsigned char) * dim);

    int i,j;
    unsigned short distance;
    for (i = 0; i < query_length; i++) {
        distance = 0;
        for (j = 0; j < dim; j++) {
            distance = distance + ct_device[query[i*dim+j] ^ database_buf[j]];
        }
        dh[i*database_length + tid] = (unsigned short)(distance);
    }
}
''', 'gpu_hamming_core')
    with cp.cuda.Device(GPU_DEVICE_NUMBER):
        query_bin_gpu = cp.asarray(query_bin, dtype=cp.uint8)
        [n1, c] = cp.shape(query_bin_gpu)
        [n2, _] = cp.shape(database_bin_gpu)
        dh = cp.zeros((n1, n2), cp.uint16)
        threads_per_block = THREADS_PER_BLOCK
        blocks_per_grid = math.ceil(n2 / threads_per_block)
        # gpu_hamming_core[blocks_per_grid, threads_per_block](query_bin_gpu, database_bin_gpu, count_1_table, dh, c)
        hamming_kernel((blocks_per_grid,), (threads_per_block,), 
            (query_bin_gpu, n1, database_bin_gpu, n2, count_1_table, dh, c))
        cp.cuda.runtime.deviceSynchronize()
        return dh
``` 

ncu测定结果: 

![image2022-8-3 15:10:29.png](/assets/01KJBYXQFVGXVXAYBSS0BJBF6B/image2022-8-3%2015%3A10%3A29.png)

压力重新回到compute throughput中

## 对compute throughput进行调优

ncu命令, 增加 --section ComputeWorkloadAnalysis 参数: 

```
CUPY_CACHE_SAVE_CUDA_SOURCE=1 CUPY_CUDA_COMPILE_WITH_DEBUG=1 /tmp/var/target/linux-desktop-glibc_2_11_3-x64/ncu --export /tmp/var/cuda.out --force-overwrite --target-processes application-only --replay-mode kernel --kernel-name-base function --launch-skip-before-match 0 --section-folder /tmp/var/sections --section LaunchStats --section Occupancy --section SpeedOfLight --section MemoryWorkloadAnalysis --section ComputeWorkloadAnalysis --sampling-interval 30 --sampling-max-passes 5 --sampling-buffer-size 33554432 --profile-from-start 1 --cache-control all --clock-control base --rule LaunchConfiguration --rule Occupancy --rule SOLBottleneck --import-source no --check-exit-code yes -k gpu_hamming_core  python main.py  --dataset=ann_1m_sq8  --algorithm=ntq_bf  --query_size=10  --gpu  --recall
``` 

![image2022-8-3 17:9:26.png](/assets/01KJBYXQFVGXVXAYBSS0BJBF6B/image2022-8-3%2017%3A9%3A26.png)

逐步拆解代码, 查看日志中 calc_hamming_dist_time_cost 的信息:

```
2022-08-03 09:12:51,603 ｜ DEBUG ｜ ntq_bf_algorithm.py ｜ search_from_trained_database_gpu ｜ 112 ｜ calc_hamming_dist_time_cost :0.00037026405334472656
``` 

  - 空内核: 0.0003702640533447265
  - 只做 database_buf 初始化: 0.0003833770751953125
  - 只做 i 的循环: 0.00041937828063964844
  - 做j的循环, 但只进行 distance += ct_device[0] : 0.0006403923034667969
  - 只进行 distance += ct_device[query[i*dim+j]]: 0.007751941680908203
  - 只进行 distance += ct_device[database_buf[j]]: 0.020306110382080078
  - 进行 distance += ct_device[query[i*dim+j] ^ database_buf[j]]: 0.023843765258789062
  - 对 distance += ct_device[database_buf[j]] 进行调优: 
    - 只进行 distance += ct_device[j]: 0.003811359405517578
    - 通过以下代码对比, 确定是 database_buf[j] 的操作导致延迟: 

```
 
        for (j = 0; j < dim; j++) {
            x = database_buf[j];
            distance += ct_device[j];
            distance += x;
        }
0.01667165756225586

------
 
 
 
		for (j = 0; j < dim; j++) {
            x = database_buf[j];
            distance += ct_device[j];
            distance += 255;
        }
 
0.0038025379180908203
```

查看 database_buf[j] 的PTX汇编码: 

![image2022-8-3 18:15:25.png](/assets/01KJBYXQFVGXVXAYBSS0BJBF6B/image2022-8-3%2018%3A15%3A25.png)

简化数据类型后, 可以缩减PTX 汇编码: 

![image2022-8-4 10:35:1.png](/assets/01KJBYXQFVGXVXAYBSS0BJBF6B/image2022-8-4%2010%3A35%3A1.png)

但无法改善性能.

继续简化: 

![image2022-8-4 12:53:16.png](/assets/01KJBYXQFVGXVXAYBSS0BJBF6B/image2022-8-4%2012%3A53%3A16.png)

确认是ld造成的延迟

尝试使用shared memory: 

![image2022-8-4 13:51:13.png](/assets/01KJBYXQFVGXVXAYBSS0BJBF6B/image2022-8-4%2013%3A51%3A13.png)

比本地私有变量的性能要慢: 0.005378007888793945 v.s. 0.00407719612121582

没有想到更好的优化策略

TODO: 

  - 改变算法的拆片方式
  - 使用Cuda 8.0以上的高级特性

# 结论

原耗时

![image2022-8-4 14:20:42.png](/assets/01KJBYXQFVGXVXAYBSS0BJBF6B/image2022-8-4%2014%3A20%3A42.png)

调优后: 

![image2022-8-4 14:20:50.png](/assets/01KJBYXQFVGXVXAYBSS0BJBF6B/image2022-8-4%2014%3A20%3A50.png)
