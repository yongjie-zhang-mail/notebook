## 4 MixGC

G1对于老年代的GC比较特殊，本质上不是只针对老年代，也有部分年轻代，所以又叫MixGC。

- 初次标记，也是标记GCroot直接引的对象和所在Region，但是与CMS不同的是，这里不止标记O区。注意初次标记一般和YGC同时发生，利用YGC的STW时间，顺带把这事给干了。日志格式如下

```
[GC pause (G1 Evacuation Pause) (young) (initial-mark), 0.0062656 secs]
```

- RootRegion扫描，扫描GCroot所在的region到Old区的引用。日志格式

```
1.362: [GC concurrent-root-region-scan-start]
1.364: [GC concurrent-root-region-scan-end, 0.0028513 secs]
```

- 并发标记，类似CMS，但是标记的是整个堆，而不是只有O区。这期间如果发现某个region所有对象都是'垃圾'则标记为X。日志格式

```
1.364: [GC concurrent-mark-start]
1.645: [GC concurrent-mark-end, 0.2803470 secs]
```

![image](https://bolg.obs.cn-north-1.myhuaweicloud.com/1909/g1-3.png)

- 重新标记，类似CMS，但也是整个堆，并且上一步中的X区被删除。另外采用了初始标记阶段的SATB，重新标记的速度变快。日志格式

```
1.645: [GC remark 1.645: [Finalize Marking, 0.0009461 secs] 1.646: [GC ref-proc, 0.0000417 secs] 1.646: [Unloading, 0.0011301 secs], 0.0074056 secs]
[Times: user=0.01 sys=0.00, real=0.01 secs]
```

![image](https://bolg.obs.cn-north-1.myhuaweicloud.com/1909/g1-4.png)

- 复制/清理，选择所有Y区reigons和'对象存活率较低'的O区regions组成Csets，进行复制清理。日志格式：

```
1.652: [GC cleanup 1213M->1213M(1885M), 0.0030492 secs]
[Times: user=0.01 sys=0.00, real=0.00 secs]
```

![image](https://bolg.obs.cn-north-1.myhuaweicloud.com/1909/g1-5.png)
![image](https://bolg.obs.cn-north-1.myhuaweicloud.com/1909/g1-6.png)