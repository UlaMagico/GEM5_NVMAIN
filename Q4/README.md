## Q4 Modify last level cache policy based on frequency based replacement policy (15%)
### 更改policy
原本的policy預設為LRU

在Caches.py將replacement policy設為LFU:

```
class L3Cache(Cache):
    '
    '
    '
    #Q4 frequency based policy
    replacement_policy = Param.BaseReplacementPolicy(LFURP(),"Replacement policy")
```

### 比較l3 cache performance
original policy:
* miss rate=0.328266
* avg miss latency=107424.258412

 LFURP:
* miss rate=0.337272
* avg miss latency=106137.509985

LFU的miss rate雖然高於原本policy的，但是avg miss latency比較低
