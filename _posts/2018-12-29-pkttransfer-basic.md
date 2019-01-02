---
layout:     post
title:      "dpdk报文转发算法"
category :  basictheory
date:       2018-12-28
author:     "Max"
header-img: "img/post-2018.jpg"
catalog:    true
tags:
    - dpdk
---

除了良好的转发框架外，转发中很重要的一部分内容就是对报文字段的匹配和识别。DPDK 中主要用到了精确匹配（Exact Match）算法和最长前缀匹配（Longest Prefix Matching， LPM）算法来进行报文的匹配从而获得相应的信息。

## 1. 精确匹配算法

核心思想就是利用哈希算法对所要匹配的值进行哈希，从而加快查找速度。

决定哈希性能的主要参数是负载参数 $L=\frac n k$，其中 n 为总的条目数目，k 为总的哈希桶的数目。当负载参数 L 值在某个合理的数值区间时哈希算法的效率比较高。L 值越大，发生冲突的概率越大。

精确匹配主要需要解决两个问题：进行数据的签名（哈希），解决哈希的冲突。

### 1.1 哈希算法

DPDK 主要支持 CRC32 和 Jhash 两个数字签名的不同算法。

CRC 校验原理实际上就是在一个 p 位二进制数据序列之后附加一个 r 位二进制检验码，从而构成一个总长为 $n = p + r$ 位的二进制序列。附加的检验码与数据序列的内容之间存在某种特定关系，通过检查这一关系，就可以实现对数据正确性的校验。

CRC中的多项式模 2 运行，实际上就是按位异或，不考虑进位、借位。当进行 CRC 校验时，发送方和接收方需要事先约定一个除数，即生成多项式，一般记作 $G(x)$。生成多项式的最高位与最低位必须是 1。

在 CRC32 算法上，DPDK 将数据流按照 8 字节 或 4 字节为单位，直接使用 IA 的硬件指令来一次处理，或使用查表的方法进行一次处理，利用空间换时间。

### 1.2 解决冲突

解决哈希冲突的方法主要有以下两种：

1. 分离链表（Seperate chaining）
   
   所有发生冲突的项通过链式相连，在查找元素时需要遍历某个哈希桶下对应的整条链。不需要额外占用哈希桶，但是速度较慢。

2. 开发地址（Open addressing）
   
   所有发生冲突的项自动往当前所对应可使用的哈希桶的下一个哈希桶进行填充。不需要链表操作，但有时会加剧冲突的发生。

DPDK 哈希桶的结构定义如下所示，每个桶可以盛 8 项，算是上述两种方法的一个折中。
```
struct rte_hash_bucket {
    hash_sig_t sig_current[RTE_HASH_BUCKET_ENTRIES];
    uint32_t key_idx[RTE_HASH_BUCKET_ENTRIES];
    hash_sig_t sig_alt[RTE_HASH_BUCKET_ENTRIES];
    uint8_t flag[RTE_HASH_BUCKET_ENTRIES];
} __rte_cache_aligned;
```

## 2. 最长前缀匹配算法

DPDK 中 LPM 的具体实现综合考虑了空间和时间，由一张 $2^{24}$ 条目的表和多张（配置指定） $2^8$ 条目的表组成。前者称为表 tlb24， 后者称为表 tlb8。

表条目的字段如下：
```
struct rte_lpm_tbl_entry {
        uint32_t depth       :6; /**< Rule depth. */
        /**
         * For tbl24:
         *  - valid_group == 0: entry stores a next hop
         *  - valid_group == 1: entry stores a group_index pointing to a tbl8
         * For tbl8:
         *  - valid_group indicates whether the current tbl8 is in use or not
         */
        uint32_t valid_group :1;
        /* Using single uint8_t to store 3 values. */
        uint32_t valid       :1;   /**< Validation flag. */
        /**
         * Stores Next hop (tbl8 or tbl24 when valid_group is not set) or
         * a group index pointing to a tbl8 structure (tbl24 only, when
         * valid_group is set)
         */
        uint32_t next_hop    :24;
};
```

用 IP 地址的前 24 位进行查找时，先看 tbl24 中的 entry， 当 valid 字段有效而 valid_group 为 0 时，直接命中，查看 next_hop 知道下一跳。当 valid 为 1 且 valid_group 为 1 时， 查看 next_hop 字段知道 tlb8 的 index， 此时根据 IP 中的后 8 位确定 tlb8 中具体 entry 的下标，然后依据 tlb8 表项中的 next_hop 找下一跳地址。
```
/** @internal bitmask with valid and valid_group fields set */
#define RTE_LPM_VALID_EXT_ENTRY_BITMASK 0x03000000

int rte_lpm_lookup(struct rte_lpm *lpm, uint32_t ip, uint32_t *next_hop)
{
    unsigned tbl24_index = (ip >> 8);
    uint32_t tbl_entry;
    const uint32_t *ptbl;

    /* Copy tbl24 entry */
    ptbl = (const uint32_t *)(&lpm->tbl24[tbl24_index]);
    tbl_entry = *ptbl;

    /* Copy tbl8 entry (only if needed) */
    if (unlikely((tbl_entry & RTE_LPM_VALID_EXT_ENTRY_BITMASK) == RTE_LPM_VALID_EXT_ENTRY_BITMASK)) {
        unsigned tbl8_index = (uint8_t)ip + (((uint32_t)tbl_entry & 0x00FFFFFF) * RTE_LPM_TBL8_GROUP_NUM_ENTRIES);
        ptbl = (const uint32_t *)&lpm->tbl8[tbl8_index];
        tbl_entry = *ptbl;
    }

    *next_hop = ((uint32_t)tbl_entry & 0x00FFFFFF);
    return (tbl_entry & RTE_LPM_LOOKUP_SUCCESS) ? 0 : -ENOENT;
}
```

## 3. 参考

1. 《深入浅出DPDK》
