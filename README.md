# Question
1. (Q1) GEM5 + NVMAIN BUILD-UP (40%)     
2. (Q2) Enable L3 last level cache in GEM5 + NVMAIN (15%) ( 看到 log 裡面有 L3 cache 的資訊 )
3. (Q3) Config last level cache to  2-way and full-way associative cache and test performance (15%)
必須跑benchmark quicksort在 2-way跟 full way (直接在 L3 cache implement，可以用 miss rate 判斷是否成功 )
4. (Q4) Modify last level cache policy based on frequency based replacement policy (15%)
5. (Q5) Test the performance of write back and write through policy based on 4-way associative cache with 
isscc_pcm(15%) 
必須跑benchmark multiply 在 write through 跟 write back ( gem5 default 使用 write back，可以用 write request 
的數量判斷write through 是否成功 )
6. Bonus (10%)
 Design last level cache policy to reduce the energy consumption of pcm_based main memory 
Baseline:LRU

## Q1 GEM5 + NVMAIN BUILD-UP
參考助教提供的簡報(略)

## Q2 啟用L3 cache
需修改以下檔案:

### 1. BaseCPU.py
引入的XBar新增L3XBar
```
from XBar import L2XBar, L3XBar
```

在class BaseCPU裡新增:
```
    def addThreeLevelCacheHierarchy(self, ic, dc, l3c, iwc = None, dwc = None):
        self.addPrivateSplitL1Caches(ic, dc, iwc, dwc)
        self.toL3Bus = L3XBar()
        self.connectCachedPorts(self.toL3Bus)
        self.l3cache = l3c
        self.toL2Bus.master = self.l3cache.cpu_side
        self._cached_ports = ['l3cache.mem_side']
```


### 2. XBar.py
仿照class L2XBar新增以下程式碼:
```
class L3XBar(CoherentXBar):
    # 256-bit crossbar by default
    width = 32

    # Assume that most of this is covered by the cache latencies, with
    # no more than a single pipeline stage for any packet.
    frontend_latency = 1
    forward_latency = 0
    response_latency = 1
    snoop_response_latency = 1

    # Use a snoop-filter by default, and set the latency to zero as
    # the lookup is assumed to overlap with the frontend latency of
    # the crossbar
    snoop_filter = SnoopFilter(lookup_latency = 0)

    # This specialisation of the coherent crossbar is to be considered
    # the point of unification, it connects the dcache and the icache
    # to the first level of unified cache.
    point_of_unification = True
```

### 3. CacheConfig.py
將下列程式
```
        dcache_class, icache_class, l2_cache_class, walk_cache_class = \
            O3_ARM_v7a_DCache, O3_ARM_v7a_ICache, O3_ARM_v7aL2, \
            O3_ARM_v7aWalkCache
    else:
        dcache_class, icache_class, l2_cache_class, walk_cache_class = \
            L1_DCache, L1_ICache, L2Cache, None
```
新增l3_cache_class
```
        dcache_class, icache_class, l2_cache_class, l3_cache_class, walk_cache_class = \
            O3_ARM_v7a_DCache, O3_ARM_v7a_ICache, O3_ARM_v7aL2, O3_ARM_v7aL3, \
            O3_ARM_v7aWalkCache
    else:
        dcache_class, icache_class, l2_cache_class, l3_cache_class, walk_cache_class = \
            L1_DCache, L1_ICache, L2Cache, L3Cache, None

```


將下列程式
```
    if options.l2cache:
        # Provide a clock for the L2 and the L1-to-L2 bus here as they
        # are not connected using addTwoLevelCacheHierarchy. Use the
        # same clock as the CPUs.
        system.l2 = l2_cache_class(clk_domain=system.cpu_clk_domain,
                                   size=options.l2_size,
                                   assoc=options.l2_assoc)

        system.tol2bus = L2XBar(clk_domain = system.cpu_clk_domain)
        system.l2.cpu_side = system.tol2bus.master
        system.l2.mem_side = system.membus.slave
```
修改為
```
    if options.l2cache and options.l3cache:
        system.l2 = l2_cache_class(clk_domain=system.cpu_clk_domain,
                                   size=options.l2_size,
                                   assoc=options.l2_assoc)


        system.l3 = l3_cache_class(clk_domain=system.cpu_clk_domain,
                                   size=options.l3_size,
                                   assoc=options.l3_assoc)

        system.tol2bus = L2XBar(clk_domain = system.cpu_clk_domain)
        system.tol3bus = L3XBar(clk_domain = system.cpu_clk_domain)

        system.l2.cpu_side = system.tol2bus.master
        system.l2.mem_side = system.tol3bus.slave

        system.l3.cpu_side = system.tol3bus.master
        system.l3.mem_side = system.membus.slave

    elif options.l2cache:
        # Provide a clock for the L2 and the L1-to-L2 bus here as they
        # are not connected using addTwoLevelCacheHierarchy. Use the
        # same clock as the CPUs.
        system.l2 = l2_cache_class(clk_domain=system.cpu_clk_domain,
                                   size=options.l2_size,
                                   assoc=options.l2_assoc)

        system.tol2bus = L2XBar(clk_domain = system.cpu_clk_domain)
        system.l2.cpu_side = system.tol2bus.master
        system.l2.mem_side = system.membus.slave
```

### 4. Caches.py
仿照class L2Cache，新增:
```
class L3Cache(Cache):
    assoc = 2
    tag_latency = 35
    data_latency =35
    response_latency = 35
    mshrs = 35
    tgts_per_mshr = 30
    write_buffers = 16
    #Q4 frequency based policy
    #replacement_policy = Param.BaseReplacementPolicy(LFURP(),"Replacement policy")
```
replacement_policy依照Q4所需，要將註解去掉

### 5. Options.py
```
    parser.add_option("--caches", action="store_true")
    parser.add_option("--l2cache", action="store_true")
```
在以上程式碼下新增
```
    parser.add_option("--l3cache", action="store_true")
```

> ### Q3之後的在folder裡
