[[Netty]]

**Arena**  

![image-20250525213401167](Netty-%E5%86%85%E5%AD%98%E7%AE%A1%E7%90%86.assets/image-20250525213401167.png)

`PoolThreadCache` 通过为每个线程维护一个缓存池，将小块内存的分配和回收操作本地化，从而显著提高了内存操作的效率。这还实现了内存申请与释放的无锁化。不过为了平衡开销，默认只会为 Reactor 线程以及  FastThreadLocalThread 类型的线程创建 PoolThreadCache。

![image-20250525213648175](Netty-%E5%86%85%E5%AD%98%E7%AE%A1%E7%90%86.assets/image-20250525213648175.png)

Netty有不同规格的内存块。

SizeClasses

Netty 内存池最小的管理单位是 page(默认为 8k)，而内存池单次向 OS 申请内存的单位是 Chunk(默认为4M)。

Netty 在向 OS 申请到一个 PoolChunk 的内存空间（4M）之后，会通过 SizeClasses 近一步将这 4M 的内存空间切分成 68 种规格的内存块来进行池化管理。其中最小规格的内存块为 16 字节，最大规格的内存块为 4M 。

这 68 种内存规格分为了三类：

1. [16B , 28K] 这段范围内的规格被划分为 **Small 规格**。

2. [32K , 4M] 这段范围内的规格被划分为 **Normal 规格**。
3. 超过 4M 的内存规格被划分为 **Huge 规格**。

![图片](Netty-%E5%86%85%E5%AD%98%E7%AE%A1%E7%90%86.assets/640-1748180551525-3.webp)

- 其中 Small 和 Normal 规格的内存块会被内存池（PoolArena）进行池化管理。

- 当我们向内存池申请 Huge 规格的内存块时，内存池是直接向 OS 申请内存，释放的时候也是直接释放回 OS ，内存池并不会缓存这些 Huge 规格的内存块。

![图片](Netty-%E5%86%85%E5%AD%98%E7%AE%A1%E7%90%86.assets/640-1748180594099-6.webp)

Small 和 Normal 这两种规格的内存块在 PoolArena 中是如何被管理起来的呢 ？

内存池的第四个模型 ——  PoolChunk 

PoolChunk 的设计参考了 Linux 内核中的伙伴系统，在内核中，内存管理的基本单位也是 Page（4K），这些 Page 会按照伙伴的形式被内核组织在伙伴系统中。

> 内核中的伙伴指的是大小相同并且在物理内存上连续的两个或者多个 page（个数必须是 2 的次幂）。

![图片](Netty-%E5%86%85%E5%AD%98%E7%AE%A1%E7%90%86.assets/640-1748180631892-9.webp)

**细节**

数组 free_area[MAX_ORDER] 中的索引表示的是分配阶 order，这个 order 用于指定对应 free_area 结构中组织管理的内存块包含多少个 page。比如 free_area[0] 中管理的内存块都是一个一个的 Page , free_area[1] 中管理的内存块尺寸是 2 个 Page ， free_area[10] 中管理的内存块尺寸为 1024 个 Page。

核首先会到伙伴系统中的 free_area[order] 对应的双向链表 free_list 中查看是否有空闲的内存块，如果有则从 free_list 将内存块摘下并分配出去，如果没有，则继续向上到 free_area[order + 1] 中去查找，反复这个过程，直到在 free_area[order + n] 中的 free_list 链表中找到空闲的内存块。

但是此时我们在 free_area[order + n] 链表中找到的空闲内存块的尺寸是 2 ^ (order + n) 大小，而我们需要的是 2 ^ order 尺寸的内存块，于是内核会将这 2 ^ (order + n)  大小的内存块逐级减半分裂，将每一次分裂后的内存块插入到相应的 free_area 数组里对应的 free_list 链表中，并将最后分裂出的  2 ^ order 尺寸的内存块分配给进程使用。

![图片](Netty-%E5%86%85%E5%AD%98%E7%AE%A1%E7%90%86.assets/640-1748180782507-12.webp)

内存回收过程，

当我们向内核释放 2 ^ order 个 Page 的时候，内核首先会检查 free_area[order]  对应的 free_list 中是否有与我们要释放的内存块在内存地址上连续的空闲内存块，如果有地址连续的内存块，则将两个内存块进行合并，然后在到上一级 free_area[order + 1] 中继续查找是否有空闲内存块与合并之后的内存块在地址上连续，如果有则继续重复上述过程向上合并，如果没有，则将合并之后的内存块插入到  free_area[order + 1]  中。

示例

假设我们现在需要将一个编号为 10 的 Page 释放回下图所示的伙伴系统中，连续的编号表示内存地址连续。首先内核会在 free_area[0] 中发现有一个空闲的内存块 page11 与要释放的 page10 连续，于是将两个连续的内存块合并，合并之后的内存块的分配阶 order = 1。

![图片](Netty-%E5%86%85%E5%AD%98%E7%AE%A1%E7%90%86.assets/640-1748180921709-15.webp)

随后内核在 free_area[1] 中发现 page8 和 page9 组成的内存块与 page10 和 page11 合并后的内存块是伙伴，于是继续将这两个内存块（分配阶 order = 1）继续合并成一个新的内存块（分配阶 order = 2）。随后内核会在 free_area[2] 中查找新合并后的内存块伙伴。

接着内核在 free_area[2] 中发现 page12，page13，page14，page15 组成的内存块与 page8，page9，page10，page11 组成的新内存块是伙伴，于是将它们从 free_area[2] 上摘下继续合并成一个新的内存块（分配阶 order = 3），随后内核会在 free_area[3] 中查找新内存块的伙伴。

但在 free_area[3] 中的内存块（page20 到 page 27）与新合并的内存块（page8 到 page15）虽然大小相同但是物理上并不连续，所以它们不是伙伴，不能在继续向上合并了。于是内核将 page8 到 pag15 组成的内存块（分配阶 order = 3）插入到 free_area[3] 中，整个伙伴系统回收内存的过程如下如所示：

![图片](Netty-%E5%86%85%E5%AD%98%E7%AE%A1%E7%90%86.assets/640-1748180933610-18.webp)

Netty的PoolChunk 

Netty 中的伙伴系统一共有 32 个 Page 级别的内存块尺寸。PoolChunk 中管理的这些 Page 级别的内存块尺寸只要是 Page 的整数倍就可以，而不是内核中要求的 2 的次幂个 Page。

因此 runsAvail 数组中一共有 32 个元素，数组下标就是上图中的 pageIndex ， 数组类型为 IntPriorityQueue（优先级队列），数组中的每一项存储着所有相同 size 的内存块，这里的 size 就是上图中 pageIndex 对应的 size 。

比如 runsAvail[0] 中存储的全部是单个 Page 的内存块，runsAvail[1] 中存储的全部是尺寸为 2 个 Page 的内存块，runsAvail[2] 中存储的全部是尺寸为 3 个 Page 的内存块，runsAvail[31] 中存储的是尺寸为 512 个 Page 的内存块。

![图片](Netty-%E5%86%85%E5%AD%98%E7%AE%A1%E7%90%86.assets/640-1748181080201-21.webp)

当我们向 PoolChunk 申请 Page 级别的内存块时，Netty 首先会从上面的 Page 规格表中获取到内存块尺寸对应的 pageIndex，然后到 runsAvail[pageIndex] 中去获取对应尺寸的内存块。

如果没有空闲内存块，Netty 的处理方式也是和内核一样，逐级向上去找，直到在 runsAvail[pageIndex + n] 中找到内存块。然后从这个大的内存块中将我们需要的内存块尺寸切分出来分配，剩下的内存块直接插入到对应的 `runsAvail[剩下的内存块尺寸 index]`中，并不会像内核那样逐级减半分裂。

PoolChunk 的内存块回收过程则和内核一样，回收的时候会将连续的内存块合并成更大的，直到无法合并为止。最后将合并后的内存块插入到对应的  `runsAvail[合并后内存块尺寸 index]` 中。

但 Netty 的伙伴系统采用的是 IntPriorityQueue ，一个优先级队列来组织相同尺寸的内存块，它会按照内存地址从低到高组织这些内存块，我们每次从 IntPriorityQueue 中获取的都是内存地址最低的内存块。Netty 这样设计的目的主要还是为了降低内存碎片，牺牲一定的局部性。

而在 Netty 的应用场景中，往往频繁申请的都是那些小规格的内存块，针对这种频繁使用的 Small 规格的内存块，Netty 在设计上就必须要保证局部性，因为这块是热点，所以性能的考量是首位。

而 Normal 规格的大内存块，往往不会那么频繁的申请，所以在 PoolChunk 的设计上，内存碎片的考量是首位。

那  Small 规格的内存块在哪里管理呢 ？这就需要引入内存池的第五个模型 —— PoolSubpage 。

PoolSubpage 可以看做是 Netty 中的 slab 。

Netty 借鉴了 Linux 内核中 Slab 的设计思想，当我们第一次申请一个 Small 规格的内存块时，Netty 会首先到 PoolChunk 中申请一个或者若干个 Page 组成的大内存块（Page 粒度），这个大内存块在 Netty 中的模型就是 PoolSubpage 。然后按照对应的 Small 规格将这个大内存块切分成多个尺寸相同的小内存块缓存在 PoolSubpage 中。

![图片](Netty-%E5%86%85%E5%AD%98%E7%AE%A1%E7%90%86.assets/640-1748181265587-24.webp)

每次申请这个规格的内存块时，Netty 都会到对应尺寸的 PoolSubpage 中去获取，每次释放这个规格的内存块时，Netty 会直接将其释放回对应的 PoolSubpage 中。而且每次申请 Small 规格的内存块时，Netty 都会优先获取刚刚释放回 PoolSubpage 的内存块，保证了局部性。当 PoolSubpage 中缓存的所有内存块全部被释放回来后，Netty 就会将整个 PoolSubpage 重新释放回 PoolChunk 中。

<img src="Netty-%E5%86%85%E5%AD%98%E7%AE%A1%E7%90%86.assets/640-1748181334775-27.webp" alt="图片" style="zoom: 50%;" />

这就引入了内存池的第六个模型 —— PoolChunkList 。

![图片](Netty-%E5%86%85%E5%AD%98%E7%AE%A1%E7%90%86.assets/640-1748181378525-33.webp)

但事实上，对于 PoolArena 中的这些众多 PoolChunk 来说，可能不同 PoolChunk 它们的内存使用率都是不一样的，于是 Netty 又近一步根据 PoolChunk 的内存使用率设计出了 6 个 PoolChunkList 。每个 PoolChunkList 管理着内存使用率在一定范围内的 PoolChunk。

<img src="Netty-%E5%86%85%E5%AD%98%E7%AE%A1%E7%90%86.assets/640-1748181455601-36.webp" alt="图片" style="zoom:50%;" />