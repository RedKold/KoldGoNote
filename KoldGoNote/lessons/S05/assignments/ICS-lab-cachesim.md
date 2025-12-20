## 构造 Cache

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251204150110.png)
- 做一个 mask
```c
	tag_mask = (uint64_t)(block_amt -1) << (63 - tag_width);
	set_mask = (uint64_t)(_set_amt -1 )<< (63 - tag_width - set_width);
	offset_mask = (uint64_t)(BLOCK_SIZE-1) << (63 - tag_width - set_width - BLOCK_WIDTH);
```

我们不需要将掩码左移；相反，我们使用**右移**将目标字段移动到地址的低位，然后用一个只有字段位宽的掩码(`mask_with_len(width)`)来提取它。

你可以食用 `mask_with_len` 宏


**注意偏移量**都是从 LSB 开始。

```
// 在 init_cache 中正确设置偏移量：

// 1. Block Offset: 始终从 0 位开始
offset_offset = 0; 

// 2. Set Index: 紧随 Block Offset 之后
set_offset = BLOCK_WIDTH;

// 3. Tag: 紧随 Set Index 之后
tag_offset = BLOCK_WIDTH + set_width;

// tag_width 的计算是正确的：
// tag_width = 64 - set_width - BLOCK_WIDTH;
```


```c
#include "common.h"
#include <inttypes.h>
#include <stdlib.h>
#include <stdio.h>
#include <math.h> 
#include <time.h> // for random replacement

// --- 内存访问桩函数（用于编译和测试）---
// 在实际项目中，这些函数可能在其他文件实现
// 我们假设这里有一个模拟的主存空间
#define PHYSICAL_MEM_SIZE MEM_SIZE
uint8_t main_memory[PHYSICAL_MEM_SIZE]; 

void mem_read(uintptr_t block_num, uint8_t* buf) {
    uintptr_t start_addr = block_num * BLOCK_SIZE;
    if (start_addr + BLOCK_SIZE > PHYSICAL_MEM_SIZE) {
        // 确保地址在模拟的内存范围内
        fprintf(stderr, "MEM_READ ERROR: Address 0x%lx out of bounds.\n", start_addr);
        return;
    }
    cycle_increase(200); // 假设内存访问需要 200 周期
    // printf("MEM_READ: Reading block %lu (Addr: 0x%lx) from memory.\n", block_num, start_addr);
    // 从主存复制到缓存行
    memcpy(buf, &main_memory[start_addr], BLOCK_SIZE);
}

void mem_write(uintptr_t block_num, const uint8_t* buf) {
    uintptr_t start_addr = block_num * BLOCK_SIZE;
    if (start_addr + BLOCK_SIZE > PHYSICAL_MEM_SIZE) {
        fprintf(stderr, "MEM_WRITE ERROR: Address 0x%lx out of bounds.\n", start_addr);
        return;
    }
    cycle_increase(200); // 假设内存访问需要 200 周期
    // printf("MEM_WRITE: Writing block %lu (Addr: 0x%lx) to memory.\n", block_num, start_addr);
    // 从缓存行复制到主存
    memcpy(&main_memory[start_addr], buf, BLOCK_SIZE);
}
// --- 桩函数结束 ---


static uint64_t cycle_cnt = 0;

// 地址划分的位宽
static int tag_width;
static int set_width;

// 地址划分的掩码
static uint64_t set_mask;

// 地址划分的偏移量 (右移位数)
static uint32_t tag_offset;
static uint32_t set_offset;
static uint32_t offset_offset = 0; // Block Offset 偏移量始终为 0

// 用于从地址中提取 tag 部分的完整掩码
static uint64_t tag_mask_for_addr = 0; 

// 设计缓存结构
typedef struct CacheLine
{
    int valid;
    int dirty; // 1 表示数据被修改过，需要写回
    uint32_t tag; // 存储 Tag
    uint8_t data[BLOCK_SIZE]; // 存储整个块的数据 (64 字节)
} CacheLine;

typedef struct CacheSet
{
    CacheLine* lines;
} CacheSet;


typedef struct Cache
{
    int set_amt;
    int associativity;
    CacheSet* cache_sets;
} Cache;

Cache* cache = NULL; 

void cycle_increase(int n)
{
    cycle_cnt += n;
}

// 帮助函数: 计算地址的 Tag, Set Index, Block Offset
// 注意：uintptr_t 是 64 位的，我们用 uint64_t 确保位操作正确
static inline uint32_t get_tag(uintptr_t addr) {
    return (uint32_t)((addr & tag_mask_for_addr) >> tag_offset);
}
static inline uint32_t get_set_index(uintptr_t addr) {
    return (uint32_t)((addr >> set_offset) & set_mask);
}
static inline uint32_t get_offset(uintptr_t addr) {
    return (uint32_t)(addr & mask_with_len(BLOCK_WIDTH));
}

// 帮助函数: 将整个 Cache Line 写回内存 (用于 Write-Back 策略)
static void write_back_block(CacheLine* line, uint32_t set_index) {
    if (line->valid && line->dirty) {
        // 重构块地址 = Tag + Set Index + 0
        uintptr_t block_addr = 
            ((uintptr_t)line->tag << tag_offset) |
            ((uintptr_t)set_index << set_offset);
           read 
        uintptr_t block_num = block_addr / BLOCK_SIZE;

        // 执行写回
        mem_write(block_num, line->data);
        line->dirty = 0; // 写回后清除 dirty 位
        // printf("Write-Back: Block 0x%lx (Tag: 0x%x, Set: %u) written to memory.\n", block_num, line->tag, set_index);
    }
}

// --- init_cache 实现 ---
void init_cache(int total_size_width, int associativity_width)
{
    srand(time(NULL)); // 初始化随机数生成器

    uint32_t total_size = exp2(total_size_width);
    uint32_t associativity = exp2(associativity_width);
    
    // 地址划分位宽计算
    set_width = total_size_width - BLOCK_WIDTH - associativity_width;
    tag_width = 64 - set_width - BLOCK_WIDTH; // 假设地址是 64 位

    if (set_width < 0) {
        fprintf(stderr, "Error: Cache parameters are invalid. Set width is negative.\n");
        return;
    }

    int _set_amt = exp2(set_width);

    // 缓存结构分配
    cache = (Cache*)malloc(sizeof(Cache));
    assert(cache);
    
    cache->set_amt = _set_amt;
    cache->associativity = associativity;
    
    cache->cache_sets = (CacheSet*)malloc(_set_amt * sizeof(CacheSet));
    assert(cache->cache_sets);

    // 为每个 Set 内部的 CacheLine 分配内存并初始化
    for (int i = 0; i < _set_amt; i++)
    {
        cache->cache_sets[i].lines = (CacheLine*) malloc(associativity * sizeof(CacheLine));
        assert(cache->cache_sets[i].lines);

        // 初始化 CacheLines：valid=0, dirty=0
        for (int j = 0; j < associativity; j++) {
            cache->cache_sets[i].lines[j].valid = 0;
            cache->cache_sets[i].lines[j].dirty = 0; 
            cache->cache_sets[i].lines[j].tag = 0;
        }
    }

    // 地址划分的偏移量 (右移位数)
    set_offset = BLOCK_WIDTH;
    tag_offset = BLOCK_WIDTH + set_width;

    // 地址划分的掩码
    set_mask = mask_with_len(set_width); 
    // Tag 掩码用于从完整地址中获取 tag 部分的位
    tag_mask_for_addr = mask_with_len(tag_width);
    tag_mask_for_addr <<= tag_offset;

    printf(
"--- Cache Init Summary ---\n\
Total Size: %u B, Block Size: %d B\n\
Sets: %d, Associativity: %u\n\
Bit Widths: Tag: %d, Set: %d, Offset: %d\n\
Offsets (Right Shift): Tag: %u, Set: %u, Offset: %u\n\
Set all valid bit to 0. Cache init complete.\n"
,total_size, BLOCK_SIZE, _set_amt, associativity, tag_width, set_width, BLOCK_WIDTH,
tag_offset, set_offset, offset_offset);
}


// --- cache_read 实现 ---
uint32_t cache_read(uintptr_t addr)
{
    cycle_increase(1); 
    
    uint32_t tag = get_tag(addr);
    uint32_t set_index = get_set_index(addr);
    uint32_t offset = get_offset(addr);
    
    CacheSet* set_it = &cache->cache_sets[set_index];
    int associativity = cache->associativity;
    int hit_line_index = -1;

    // 1. 查找缓存
    for (int i = 0; i < associativity; i++) {
        CacheLine* line = &set_it->lines[i];
        if (line->valid && line->tag == tag) {
            hit_line_index = i;
            break;
        }
    }

    if (hit_line_index != -1) {
        // Cache Hit
        
        uint8_t* data_ptr = &set_it->lines[hit_line_index].data[offset];
        
        // 返回 4 字节数据 (假设 word 对齐)
        return *(uint32_t*)data_ptr;
    } else {
        // Cache Miss

        // 2. 缓存缺失：选择替换行 (随机替换策略)
        int replace_line_index = rand() % associativity;
        CacheLine* line_to_replace = &set_it->lines[replace_line_index];

        // 3. (Write-Back) 如果替换行有效且为脏，则写回主存
        write_back_block(line_to_replace, set_index);

        // 4. 从主存读取整个块 (Allocate)
        uintptr_t block_addr = addr & tag_mask_for_addr; // 使用 Tag Mask 配合 Set/Offset 掩码可以得到块地址
        block_addr |= ((uintptr_t)set_index << set_offset);
        uintptr_t block_num = block_addr / BLOCK_SIZE;

        mem_read(block_num, line_to_replace->data);

        // 5. 更新缓存行状态
        line_to_replace->valid = 1;
        line_to_replace->dirty = 0; // 新加载的行是干净的
        line_to_replace->tag = tag;

        // 6. 返回所需数据
        uint8_t* data_ptr = &line_to_replace->data[offset];
        return *(uint32_t*)data_ptr;
    }
}


// --- cache_write 实现 ---
void cache_write(uintptr_t addr, uint32_t data, uint32_t wmask)
{
    cycle_increase(1); 
    
    uint32_t tag = get_tag(addr);
    uint32_t set_index = get_set_index(addr);
    uint32_t offset = get_offset(addr);
    
    CacheSet* set_it = &cache->cache_sets[set_index];
    int associativity = cache->associativity;
    int hit_line_index = -1;

    // 1. 查找缓存
    for (int i = 0; i < associativity; i++) {
        CacheLine* line = &set_it->lines[i];
        if (line->valid && line->tag == tag) {
            hit_line_index = i;
            break;
        }
    }

    CacheLine* line_to_update;
    if (hit_line_index != -1) {
        // Cache Hit: 直接写入缓存
        line_to_update = &set_it->lines[hit_line_index];
        
    } else {
        // Cache Miss (Write-Allocate): 
        // 2. 选择替换行 (随机替换策略)
        int replace_line_index = rand() % associativity;
        line_to_update = &set_it->lines[replace_line_index];

        // 3. (Write-Back) 如果替换行有效且为脏，则写回主存
        write_back_block(line_to_update, set_index);

        // 4. 从主存读取整个块 (Allocate)
        uintptr_t block_addr = addr & tag_mask_for_addr;
        block_addr |= ((uintptr_t)set_index << set_offset);
        uintptr_t block_num = block_addr / BLOCK_SIZE;

        mem_read(block_num, line_to_update->data);

        // 5. 更新缓存行状态
        line_to_update->valid = 1;
        line_to_update->tag = tag;
        // dirty 位将在下一步设置
    }
    
    // 6. 写入缓存数据并设置 dirty 位
    uint8_t* data_ptr = &line_to_update->data[offset];
    uint32_t* word_in_cache = (uint32_t*)data_ptr;
    
    // 根据 wmask 写入字 (4 字节)
    // 逻辑: 保持不需要写入的位 (*word_in_cache & ~wmask) 
    //      并写入需要更新的位 (data & wmask)
    *word_in_cache = (*word_in_cache & ~wmask) | (data & wmask);
    
    // 设置 dirty 位
    line_to_update->dirty = 1;
}

void display_statistic(void)
{
    printf("\n--- Cache Statistics ---\n");
    printf("Total cycles: %" PRIu64 "\n", cycle_cnt);
}
```


修正计算
```c
int _set_amt = block_amt / associativity;
int set_width = total_size_width - associativity_width- BLOCK_WIDTH;
int tag_width = 64 - set_width -BLOCK_WIDTH;
```

我们的内存是用一个 `uint8_t` 的数组模拟的
```c
static uint8_t mem[MEM_SIZE];
```


注意标准的不带 cache 的访存是
```c
uint32_t mem_uncache_read(uintptr_t addr){
	uint32_t *p = (void*)mem_diff + (addr & ~0x3); // 对齐到word
	return *p;
}
```

我们要注意对齐的问题。
**来修正他**。修正之后，pass 了大多数 READ
但是发现执行 80000 行左右报错，推测可能是其他未做对齐处理地方的问题.

修正后，通过了测试

