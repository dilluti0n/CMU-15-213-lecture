# 4. Cache Memories
> lecture source : [12-cache-memories.pdf](https://www.cs.cmu.edu/afs/cs/academic/class/15213-f15/www/lectures/12-cache-memories.pdf)

## $4.1. Memory hierarchy

### 4.1.1. Storage technologies and trends
**Memories**
- Random-Acess Memory - `RAM`
  - Baisic storage unit - `cell` (1 bit/cell)
  - Multiple `RAM` chips form a memory
  - two varieties: Static RAM, Dynamic RAM
  - `SRAM`: fast, expensive - Cache memories
  - `DRAM`: slow, cheap - Main memories, Frame buffers
- Nonvolatile Memories - `ROM`, `SSD`, Disk caches...
  - Read-only memory - `ROM` - cannot be programmed
  - Programmable ROM - `PROM` - can be programmed once
  - Eraseable PROM - `EPROM` - can be `bulk` erased (UV, X-Ray)
  - Electrically eraseable PROM - `EEPROM` - electronic erase capability
  - Flash memory: EPROMs.. with partial (block-level) erase capability
	- wears out after about 100,000 erasing

**Bus Structure**
- `Bus` - collection of parallel wires that carry adress, data, control signs.
- Memory read(Load) `movq A, %rax` (`A`: adress)
  1. CPU places `A` on the memory bus.
  2. Main memory reads `A` from the memory bus, retrieves data, and places it on the bus.
  3. CPU read data from the bus and copies it into `%rax`.
- Memory write(Store) `movq %rax, A` (`A`: adress)
  1. CPU places `A` on bus. Main memory reads it and waits for the corresponding data to arrive.
  2. CPU places data on the bus.
  3. Main memory reads data from the bus and stores it at adress `A`.
  
**Disk Drive**
- structure
  - `Disks` - consist of `platters` - each with two `surfaces`
  - each `surface` - consists of concentric rings, `tracks`
  - each `track` - consists of `sectors` sepatated by `gaps`
  - `recoding zone` - 
- capacity - maximum number of bits that can be stored.
  - `Recording density` [bits/in] - number of bits / 1 inch segment of a `track`
  - `Track density` [tracks/in] - number of `tracks` / 1 inch of radial segment
  - `Areal density` [bits/in^2] - `Recoding density` * `Track density`
  
## $4.2. Cache Memories

### 4.2.1. General cache concept
- adress bits : [t bits | s bits | b bits]
  - t - tag
  - s - set index
  - b - block offset
- cache : (S, E, B)
  - S - $2^s$ sets
  - E - associativity (line per each set)
  - B - Each block has $B = 2^b$ bytes of size

**Example**
<div align="center">

$(S, E, B) = (2^2, 2, 2^2)$

*2-way set-associative cache with 4-sets and a block size of 4 bytes*

| set index |     Line 1      |      Line 2     |
|:---------:|:----------------------:|:----------------------:|
|     00    | (tag) [00\|01\|10\|11] | (tag) [00\|01\|10\|11] |
|     01    | (tag) [00\|01\|10\|11] | (tag) [00\|01\|10\|11] |
|     10    | (tag) [00\|01\|10\|11] | (tag) [00\|01\|10\|11] |
|     11    | (tag) [00\|01\|10\|11] | (tag) [00\|01\|10\|11] |
</div>

### 4.2.2 cache read and write
**Read-Example**

> Load `2 bytes` of data from adress `0x9e` of little endian machine (i.e. data from `M[0x9e-0x9f]`)
>
> given $(s, E, b) = (2, 2, 2)$
  - `0x9e` = [1001|11|10]
  - tag - `0x9`
  - set index - `3`
  - block offset = `[10-11]`(or `[2-3]`)

1. search for set `3`, tag `0x9`.
2. cache `HIT` or `MISS`:
    - `HIT`: If a line with tag `0x9` exists in set `3`, Load the data from block offsets `[10-11]` of that line. - `case (1)`
    - `MISS`: If there is no line with tag `0x9`, go to `step 3`.
3. Handling `MISS`:
    - Empty Block Available: if there is an empty line in set `0b11`, write tag `0x9` and Load `M[0x9e-0x9f]` to block offsets `[10-11]`. load `B[10-11]` to reg. - `case (2)`
    - `EVICTION` Required: If there are no empty lines, select a block for `eviction` based on the `replacement policy` (e.g., LRU, random). Replace the evicted line's tag to `0x9` and Load `M[0x9e-0x9f]` to block `[10-11]`. - load `B[10-11]` to reg. - `case (3)`

<p align="center">

| case (1) | case (2) |  case (3)  |
|:--------:|:--------:|:----------:|
|    HIT   |   MISS   | MISS EVICT |
</p>

- `replacement policy`
  - least recently used(LRU) : replace the least-recently used line when evicted.

see also [CMU-15213-lab-sol/.../csim.c](https://github.com/dilluti0n/CMU-15213-lab-sol/blob/master/4_CacheLab/cachelab-handout/csim.c) : the cache simulater with least-recently used policy, implemented by quene.

**Write**
1. `hit`
    - `write-through`: memory always mirror the contents of cache.
    - `write-back`: defer write to memory until replacement of line
      - need a `dirty-bit` (information about diffrent lines from memory)
2. `miss`
    - `write-allocate`: load into cache and update it in cache
    - `no-write-allocate`: writes straight to memory, do not load into cache
- Typical comb.
  - `write-through` + `no-write-allocate`
  - `Write-back` + `Write-allocate` (normal)

### 4.2.3. Writing cache-friendly code

see [CMU-15213-lab-sol/4_cachelab](https://github.com/dilluti0n/CMU-15213-lab-sol/tree/master/4_CacheLab)
