# 4. Cache Memories
> lecture source : [12-cache-memories.pdf](https://www.cs.cmu.edu/afs/cs/academic/class/15213-f15/www/lectures/12-cache-memories.pdf)

## Memory hierarchy

## Cache Memories

### [1] General cache concept
- adress bits : [__(t bits)__|__(s bits)__|__(b bits)__]
  - t - tag
  - s - set index
  - b - block offset
- cache (S, E, B)
  - S - $2^s$ sets
  - E - associativity (block per each set)
  - B - Each block has $B = 2^b$ bytes of size
  
<p align="center">
**Example**
(S, E, B) = ($2^2$, 2, $2^2$)
  - Represents a 2-way set-associative cache with 4-sets and a block size of 4 bytes.

| set index |     Block(Line) 1      |      Block(Line) 2     |
|:---------:|:----------------------:|:----------------------:|
|     00    | (tag) [00\|01\|10\|11] | (tag) [00\|01\|10\|11] |
|     01    | (tag) [00\|01\|10\|11] | (tag) [00\|01\|10\|11] |
|     10    | (tag) [00\|01\|10\|11] | (tag) [00\|01\|10\|11] |
|     11    | (tag) [00\|01\|10\|11] | (tag) [00\|01\|10\|11] |
</p>

- cache read

(ex) load from mem. adress 0x9e, 2 byte (short int), (s, E, b) = (2, 2, 2)
  - 0x9e = [1001][11][10]
1. search for set 0b11(decimal 3), tag 0x9.
2. cache HIT or MISS:
    - HIT: If a block with tag 0x9 exists in set 0b11, it's a cache hit. Load the 2 bytes of data from block offsets [10-11]. - End with case (1)
    - MISS: If no block with tag 0x9 is found, it's a cache miss. Proceed to step 3.
3. Handling MISS:
    - Empty Block Available: if there is an empty block in set 0b11, write tag 0x9 and M[0x9e-0x9f] to block offsets [10-11] load B[10-11] to reg. - (2)
    - Eviction Required: If there are no empty blocks, select a block for eviction based on the replacement policy (e.g., LRU, random). Replace the evicted block's contents with the new data and load it to the register. - (3)
    
  - case (1) - HIT
  - case (2) - MISS
  - case (3) - MISS EVICT

  - Eviction and Replacement Policies
Eviction occurs when a cache miss happens, and there are no empty blocks available in the relevant set. A block must be selected for eviction to make room for the new data.
Replacement Policies include Random, Least Recently Used (LRU), and others. These policies determine which block to evict in case of a cache miss.
