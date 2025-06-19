### Q3 Config last level cache to 2-way and full-way associative cache and test performance (15%) 
> 必須跑benchmark quicksort在 2-way跟 full way (直接在 L3 cache implement，可以用 miss rate 判斷是否成功 )

因為原先測試時發現miss rate沒改變，所以分old與new，old與new的區別為l3_size不同

old的l3_size=1MB，new的l3_size=512kB

eeclass繳交的log檔為new的那一份


---
### 參數
#### old:
2_way
```
./build/X86/gem5.opt configs/example/se.py -c ./quicksort \
--cpu-type=TimingSimpleCPU \
--caches --l2cache --l3cache \
--l1d_size=32kB --l2_size=128kB --l3_size=1MB \
--l3_assoc=2 \
--mem-type=NVMainMemory \
--nvmain-config=../NVmain/Config/PCM_ISSCC_2012_4GB.config
```

full_way
```
./build/X86/gem5.opt configs/example/se.py -c ./quicksort \
--cpu-type=TimingSimpleCPU \
--caches --l2cache --l3cache \
--l1d_size=32kB --l2_size=128kB --l3_size=1MB \
--l3_assoc=2 \
--mem-type=NVMainMemory \
--nvmain-config=../NVmain/Config/PCM_ISSCC_2012_4GB.config
```


#### new:
2_way
```
./build/X86/gem5.opt configs/example/se.py -c ./quicksort \
--cpu-type=TimingSimpleCPU \
--caches --l2cache --l3cache \
--l1d_size=32kB --l2_size=128kB --l3_size=512kB \
--l3_assoc=2 \
--mem-type=NVMainMemory \
--nvmain-config=../NVmain/Config/PCM_ISSCC_2012_4GB.config
```
full_way
```
./build/X86/gem5.opt configs/example/se.py -c ./quicksort \
--cpu-type=TimingSimpleCPU \
--caches --l2cache --l3cache \
--l1d_size=32kB --l2_size=128kB --l3_size=512kB \
--l3_assoc=8192 \
--mem-type=NVMainMemory \
--nvmain-config=../NVmain/Config/PCM_ISSCC_2012_4GB.config
```


在測試時將l3_size設為1MB時，之後將l3_assoc設為2(2-way)與16384(full-way)觀察stats.txt發現miss_rate皆不變，推測l3 cache容量夠大幾乎不會發生conflict miss，所以改變associativity後miss rate沒改變

將l3_size設為512kB後，將l3_assoc設為2(2-way)與8192(full-way)觀察stats.txt發現full-way的miss rate會比2-way的miss rate小一些，推測為改善了conflict miss讓總體miss rate下降
