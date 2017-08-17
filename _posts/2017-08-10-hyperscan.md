---
layout:     post
title:      "认识 Hyperscan"
subtitle:   "basic knownledge of Hyperscan"
category :  basictheory
date:       2017-08-12
author:     "Max"
header-img: "img/post-dk-2016.jpg"
catalog:    true
tags:
    - opensource
---

## 1. 引言

项目上考虑使用 Hyperscan 优化匹配性能，因此做一些了解及记录。

[Hyperscan](https://01.org/zh/hyperscan) 是 Intel 推出的一款专注于高性能的多模、流式匹配的正则表达式引擎。其 C API 主要包含 **编译（Compilation）** 和 **扫描（Scanning）** 两部分，源码可从 [GitHub Repo](https://github.com/01org/hyperscan) 下载。

## 2. 编译

Hyperscan 编译器将传入的正则表达式转换成模式数据库，在扫描阶段使用。编译过程会分析传入的多个正则表达式，决定扫描阶段的算法，尽量减少运行时计算量。

#### 2.1 API

API 提供了三个函数用于将正则表达式转换为模式数据库：
1. `hs_compile()`
    将单个正则表达式转换为模式数据库。
2. `hs_compile_multi()`
    将多个正则表达式转换为一个模式数据库，在扫描阶段进行并发式的匹配。如果某一正则表达式成功匹配，使用用户提供的方式返回。
3. `hs_compile_ext_multi()`
    与 `hs_compile_multi()` 类似，但是允许为每一个正则表达式添加扩展的参数。扩展的参数为一个 `hs_expr_ext_t` 结构体，包含以下值域：
    * flags : 管理结构体中哪些值域处于激活状态
    * min_offset : 限定结束位置的最小偏移量，不满足的不视为成功匹配
    * max_offset : 限定结束位置的最大偏移量，不满足的不视为成功匹配
    * min_length : 限定匹配的最小长度，不满足的不视为成功匹配
    * edit_distance : 匹配与表达式的[编辑距离](https://en.wikipedia.org/wiki/Levenshtein_distance)在指定范围内的结果

另外，编译过程需要依据扫描的数据类型选用对应的模式：
* 流模式（Streaming mode）

    目标数据是持续的流，多个数据块顺序扫描并匹配。多次函数调用期间，每一条流都需要一块内存来记录其扫描状态。
    
* 块模式（Block mode）

    目标数据块各自独立，每个数据块可在一次函数调用中完成扫描，无需额外的记录空间。

* 矢量模式（Vectored mode）

    目标数据包含一系列非连续的数据块，但是所有数据块可在一次函数调用中完成扫描，因此无需额外的记录空间。

*注：编译为某个模式的模式数据库只能在该模式下使用。在某个 Hyperscan 版本下编译的模式数据库，也只能在同版本的 Hyperscan 下使用（扫描）。*

#### 2.2 语法

Hyperscan 正则表达式遵循 [PCRE 库](http://www.pcre.org/)的语法，但并不是 PCRE 的构造全都支持。支持的详情参见 [hyperscan开发文档](http://01org.github.io/hyperscan/dev-reference/compilation.html#pattern-support)。

虽然 Hyperscan 遵循 libpcre 语法，但是在流式匹配和多模匹配部分，其语义还是有所出入：
1. 多模匹配

    Hyperscan 支持同时匹配多个模式串，与 `|` 分隔的模式串组合并不相同。后者从左至右轮流进行匹配。

2. 顺序缺失

    Hyperscan 若返回多个匹配，并不保证返回结果与其在目标数据内的位置顺序一致。
3. 返回结束位置偏移量

    Hyperscan 默认只返回匹配成功的结束位置的偏移量。若要[返回起始位置的偏移量](http://01org.github.io/hyperscan/dev-reference/compilation.html#start-of-match)，需要在编译阶段为每个表达式添加特定的 flag。
4. 返回所有匹配

    若以 `/foo.*bar/` 扫描 `fooxyzbarbar`，libpcre 将返回一个匹配结果，贪婪模式（默认）下为 `fooxyzbarbar`，非贪婪模式下为 `fooxyzbar`。但是 Hyperscan 会将两个结果全部返回。

#### 2.3 硬件优化

利用 x86 架构的处理器的某些特性，Hyperscan 可以提高扫描性能。部分特性需要在编译时选定（默认情况下，Hyperscan 使用 `-march=native` 编译，可以使用宿主机上 C 编译器支持的所有指令）。所有的 Hyperscan API 都可以接收一个 `hs_platform_info_t` （可选）参数用来设置目标平台架构信息，用来生成相应的模式数据库，以提高扫描性能。该参数为 NULL 时默认使用当前宿主机的平台架构。`hs_platform_info_t` 包含两个值域：
  * tune

    用于指定目标平台架构，引导相应优化操作。指定该值并不会使得编译出的模式数据库只能用于该处理架构，但是会影响其扫描性能。
  * cpu\_features

    用于设置目标平台 CPU 特性掩码。如 `HS_CPU_FEATURES_AVX2` 可以打开对 Intel® Advanced Vector Extensions +2 (Intel® AVX2) 指令集的支持。指定该值会使得的编译出的模式数据库只能用于支持该特性的 CPU 上。

## 3. 扫描

Hyperscan 提供了三种不同的扫描模式，其 API 函数皆以 `hs_scan` 作为前缀。

#### 3.1 回调函数

当一个匹配出现时，所有这些函数都会调用一个用户提供的回调函数，其类型声明如下：
```
typedef (* match_event_handler)(unsigned int id, unsigned long long from, unsigned long long to, unsigned int flags, void *context)
```
  * id ： 编译时指定的正则表达式标识号
  * from ：匹配的起始位置偏移量，仅在（编译阶段）设置了返回起始位置偏移量的情况下有效
  * to ： 匹配的结束位置偏移量
  * flags ：预留标记
  * context ： 用户提供给 `hs_scan()`, `hs_scan_vector()` 或 `hs_scan_stream()` 函数的指针。

回调函数保留结束扫描状态的能力，此时返回值非零。

#### 3.1 流模式（Streaming Mode）

流模式下 API 提供了 `hs_open_stream()`、 `hs_scan_stream()` 和 `hs_close_stream()` 分别用于打开、扫描、关闭数据流。Hyperscan 库中会维护一个流表，用来保存匹配状态，这是跨数据块匹配得以实现的关键。代价即是额外的存储空间及性能影响。

为了支持多个匹配的严格排序，匹配结果（偏移量）会在当前流写入\结束后返回。

流模式下，还有其它的多个 API 方便流管理：
* `hs_reset_stream()` : 重置流至其初始化状态。相当于调用 `calling hs_close_stream()` 却不释放其存储状态的空间。
* `hs_copy_stream()` : 构建一个流的副本。会申请新的存储状态的空间。
* `hs_reset_and_copy_stream()` : 先重置目标流，然后构建一个副本。该调用不会像 `hs_copy_stream()` 一样申请新的存储空间。

#### 3.2 块模式（Block Mode）

块模式下仅有一个 API —— `hs_scan()`。这一个函数却相当于完整调用了`hs_open_stream()`、`hs_scan_stream()`、`hs_close_stream()`并且没有流相关的开销。

#### 3.3 矢量模式（Vectored Mode）

该模式下同样也只有一个 API —— `hs_scan_vector()`。这个函数接收一系列的数据指针、长度，对内存不连续的一系列数据块像连续内存一样进行扫描。从用户角度而言，该 API 效果和直接扫描单一数据块没有任何不同。

#### 3.4 草稿空间（Scratch Space）

扫描数据时，Hyperscan 需要少量临时内存来存储即时内部数据。 这个空间很大，不适合从堆栈申请，特别是对于嵌入式应用程序而言，而动态分配开销很大，因此必须为扫描功能预先分配一块“草稿”空间。

`hs_alloc_scratch()` 函数可以为给定模式数据库分配一块足够的空间。其声明如下：
```
hs_error_t HS_CDECL hs_alloc_scratch(const hs_database_t * db, hs_scratch_t ** scratch)
```
  * db ： 指向 `hs_compile()` 等产生的模式数据库。
  * scratch ： 传入一个值为 NULL 的指针，则申请一块新的空间并获取其地址。若传入指针非空，则检查其是否满足传入的数据库所需，否则申请一个更大的空间并返回其地址。返回的地址满足新传入的数据库及之前地址对应所有数据库所需。
  * 空间申请成功，函数返回 HS\_SUCCESS；否则返回失败原因，如 HS\_NOMEM。

即使使用多个模式数据库，也可以只为它们分配一个足够大的草稿空间。但是，Hyperscan 库是可重入的，草稿空间不是。当进行一个递归或嵌套扫描（如在回调函数内）时，就需要申请多个草稿空间了。没有递归扫描的情况下，每个线程只需要一个这样的空间，并且应该在数据扫描开始之前完成分配。如果数据库由一个主线程编译，数据将由多个工作线程进行扫描，可以使用 `hs_clone_scratch()` 方便的为每个线程复制一个相同的草稿空间，无需调用者将数据库传给每个线程并重新申请空间。

#### 3.5 自定义分配器

Hyperscan 默认使用系统提供的 `malloc()` 和 `free()` 方法分配内存空间，但是也提供自定义的方法：
* `hs_set_database_allocator()` ：为编译的模式数据库设置分配、释放函数
* `hs_set_scratch_allocator()` ：为草稿空间设置分配、释放函数
* `hs_set_stream_allocator()` ：流模式下，为流状态空间设置分配、释放函数
* `hs_set_misc_allocator()` ：为各种数据（如编译错误信息等）结构体设置分配、释放函数
* `hs_set_allocator()` ：将所有自定义分配器设置为同一组分配、释放函数

## 4. 序列化处理

在某些场景下，将模式串事先编译为模式数据库不能满足需求。用户希望：
1. 在不同宿主机编译模式数据库
2. 存储编译好的模式数据库，但当模式串变更时，重新编译模式数据库
3. 控制/操作编译好的模式数据库所在的内存区域

鉴于此，Hyperscan 提供了序列化和反序列化编译模式数据库的功能。有以下 API：
* `hs_serialize_database()`: 将模式数据库序列化存储到一块可重复定位的存储空间上
* `hs_deserialize_database()`: 以 `hs_serialize_database()` 的输出重新构建一个新的模式数据库
* `hs_deserialize_database_at()`: 以 `hs_serialize_database()` 的输出，在给定内存位置重新构建一个新的模式数据库
* `hs_serialized_database_size()`: 给定一个序列化了的模式数据库，返回将其反序列化所需内存空间大小
* `hs_serialized_database_info()`: 给定一个序列化了的模式数据库，返回一个包含该数据库信息的字符串，与`hs_database_info()`功能类似

包含编译及扫描部分的完整库 —— libhs 需要依赖 C++ 标准库。对于一些嵌入式程序而言，模式串编译通常发生其它主机上，它们直接使用序列化了的模式数据库。此时，去除编译部分对 C++ 标准库的依赖，这些程序可以只链接一个运行时版本的 Hyperscan 库 —— libhs_runtime。

## 5. 性能优化建议

Hyperscan 的性能受许多因素影响，主要有
* 模式串的数量和构成

    某些模式串可以明显降低扫描性能。

* 目标数据

    包括目标数据中可以匹配或接近匹配的占比等。

* 编译标记
* 扫描模式

    块模式下事先可以确定扫描数据的大小，拥有最高的性能。而流模式下不仅数据量未知，还需要额外存储分布在多个数据块之间的状态信息，相较而言性能最低。

* （硬件）平台架构

    Hyperscan 可以利用现代处理器的部分特性提高性能，如 Intel® Advanced Vector Extensions 2 (Intel® AVX2) 和 Intel® Bit Manipulation Instructions 2 (Intel® BMI2) 等。

接下来给出一些构建模式串或模式串集的性能优化建议。

#### 5.1 不要手动优化正则表达式

大部分正则表达式都有多种不同的写法，Hyperscan 都可以支持。除非明确需要，不要手动将一种写法改为另一种。

#### 5.2 不要手动优化库的使用

Hyperscan 支持处理小的输入，非常大或小的模式串集，因此除非是一些已知的库的特定性能问题，最好用简单直接的方式使用 Hyperscan。例如，除非流式数据块非常小，每次 1-2 字节，否则无需将数据缓存为更大的块再行处理。

Hyperscan 在少量模式串时运行更快。然后随着模式串数量的增大，Hyperscan 处理速度以一种平滑的趋势变慢，而非达到某一数量阈值时的骤然降低。

Hyperscan 还支持线程绑定 CPU 提高吞吐量，因此一般不需要缓冲数据来提供可用的并行处理（it is not usually necessary to buffer data to supply Hyperscan with available parallelism）。

#### 5.3 尽量使用块模式代替流模式

若数据出现在离散记录中，或者需要某种转化（如 URI 归一化），需要处理之前积累的所有数据，此时应以块模式替代流式模式进行扫描。

如果存在块和流模式下模式串的混合，则除非流模式对应的模式串数量远远超过块模式，否则应该将其编译到不同的模式数据库中进行扫描。

#### 5.4 避免非必须的模式数据库的联合

如果有 5 种不同类型的网络流量 T1 到 T5 需要扫描 5 个不同的模式集，那么构建 5 个独立的数据库并扫描相应的数据流将比合并所有 5 个模式集更有效率。即使在模式集之间存在重大重叠的情况下也是如此。只有模式集的共同子集绝对大（例如，90％的模式串出现在所有 5 种流量类型中），并且共同子集外的模式串没有性能问题时，才需要考虑合并所有 5 个模式集的数据库。

#### 5.5 在模式数据库编译或反序列化后，立即申请草稿空间

草稿空间的申请开销一般都不小。作为模式数据库编译或反序列化后的第一次使用，Hyperscan 会对其做一些检查，然后申请内存空间。因此尽量避免直到扫描前才执行该操作。最好在模式数据库编译或反序列化后，立即申请草稿空间。

#### 5.6 每一个扫描上下文仅申请一个草稿空间

多个模式数据库可以共用一个草稿空间，但是每一个并发的扫描操作（如一个扫描线程）需要一个独立的草稿空间。

#### 5.7 如果一个模式串应该出现在起始位置，确保将其锚定

锚定位置的模式串要比一般模式串匹配起来简单很多，特别是在起始位置的。但是结束位置的模式串，尤其在流模式下，仅能带来非常小的性能增益。

有许多方式可以锚定一个模式串的位置：
* `^`/`\A` 将模式串锚定在缓冲区的起始位置
    
    如 `/^foo/` 只可能返回偏移量 3 （返回结束位置）。

* `$`/`\z`/`\Z` 将模式串锚定在缓冲区的结束位置

    如 `/foo\z/` 只可以 匹配到扫描缓冲区结尾的 "foo"。 但需注意，缓冲区末尾的换行符不影响锚定，因此 `/foo\z/` 可以匹配到 "abc foo" 或 "abc foo\n"。

* `min_offset`/`max_offset` 编译参数也可以用来限制模式串的出现位置

    如在max\_offset=10的前提下，模式串 `/foo/` 仅可以匹配到偏移量在 10 以内的字符串。该模式串也可以在 `HS_FLAG_DOTALL` 编译标记下写作 `/^.{0,7}foo/`。

#### 5.8 尽量避免可以到处匹配的模式串

与 libpcre 等使用贪婪模式的匹配引擎不同，Hyperscan 会返回所有满足模式串的匹配，而非某一个。可以到处匹配的模式串（如 `/.*/`）一般会有大量的返回值，扫描速度无疑会受到影响。另一个角度而言，因为有多个返回值，可能并不能获取预期的结果。

#### 5.9 限定次数的重复在流模式下开销非常大，尽量避免

一个限定重复次数的构造（如 `/X.{1000,1001}abcd/`）需要记录大量可能的返回值，特别是在流模式下还可能分在不同的数据块上，开销非常大。

#### 5.10 尽可能使用文字而非通配符，即使文字串很长

带文字的模式串的匹配速度要快于仅有通配符的模式串。在流模式下，模式串中出现较长的文字串或靠前的位置出现文字串都有很大优势。

但是，建议 5.1 依然适用。如 `/(03|04|05|13|14|15|23|24|25).*\w\w/` 并不会比 `/[0-2][3-5].*\w\w/` 匹配速度更快。

#### 5.11 尽可能使用“dot all”模式

PCRE 语法中， `.` 不能匹配换行符。不使用 `HS_FLAG_DOTALL` 模式串编译标记时， `/A.*B/` 与 `/A[^\n]*B/` 效果等同，会有高昂开销。

然而[文档](http://01org.github.io/hyperscan/dev-reference/performance.html#dot-all-mode)中接下来又说多数使用场景中，不带 DOTALL 标记可以更好地“一次一行”地执行扫描任务，让人疑惑。

> Not using the HS_FLAG_DOTALL pattern flag can be expensive, as implicitly, it means that patterns of the form /A.*B/ become /A[^\n]*B/.

> It is likely that scanning tasks without the DOTALL flag are better done ‘line at a time’, with the newline sequences marking the beginning and end of each block.

> This will be true in most use-cases (an exception being where the DOTALL flag is off but the pattern contains either explicit newlines or constructs such as \s that implicitly match a newline character).

#### 5.12 如果可能，考虑限制每个模式串的匹配结果仅为一个

`HS_FLAG_SINGLEMATCH` 模式串编译标记可以使 Hyperscan 只返回每个模式串的第一个成功匹配。该标记将在内部激活许多优化，提高匹配速度，减少流模式下的状态存储量。

但是记录每个模式串是否已经匹配过会花费额外的空间，对匹配次数不多的模式串反而会降低性能。

#### 5.13 在非必要的情况下，不要使用返回匹配起始位置的功能

收集匹配的起始位置信息需要存储大量的状态信息，特别是在流模式下。因此如非必要，不要使用 `HS_FLAG_SOM_LEFTMOST` 标记编译模式串。

#### 5.14 近似匹配是实验性质的特性，不要在非必要情况下使用

由于匹配的特异性降低，通常会出现与近似匹配相关的性能影响，且这种影响会因模式串和编辑距离而显著变化。

## 6. 实用工具

Hyperscan 库中还包含一些实用的小工具。

#### 6.1 性能测试工具 hsbench

hsbench 工具提供了一种简单的方法来衡量 Hyperscan 对一组特定的模式和数据集的扫描性能。

数据集需要构建为一种特定的 SQLite 数据库的形式，以便将数据资料轻松的分解成块和流。tools/hsbench/scripts 目录下提供了一些可以将 PCAP、TEXT 等格式的文件转化为指定数据库形式的 Python 脚本。

hsbench 接受的模式串也需要满足一定的格式。事实上，所有的 Hyperscan 工具所需的模式串都需要满足这样的格式。模式集存储为一个文本文件，每行代表一个模式串，格式如下：
```
<integer id>:/<regex>/<flags>
```
  * \<integer id\> 是用于 Hyperscan 匹配成功时标识模式串的，需要保证唯一性。
  * \<regex\> 遵循 PCRE 语法。
  * \<flags\> 是可映射为 Hyperscan 编译标记的单个字符，如下：

      Character	| API Flag	| Description
      --- | --- | ---
      i	| HS_FLAG_CASELESS	| 忽略大小写匹配
      s | HS_FLAG_DOTALL    | Dot (.) 允许匹配换行符
      m | HS_FLAG_MULTILINE	| 启用行锚定功能。否则 `^/$` 只能匹配整个输入缓冲区的起始与结束位置
      H | HS_FLAG_SINGLEMATCH |	只返回第一个成功匹配
      V | HS_FLAG_ALLOWEMPTY | 允许模式串匹配到空结果，如 `.?`、`.*`、`(a|)`
      8 | HS_FLAG_UTF8	 | 令 Hyperscan 将模式串视为 UTF-8 字符串
      W | HS_FLAG_UCP	| 允许 Hyperscan 使用 Unicode 内容替代默认的 ASCII 含义
      P | HS_FLAG_PREFILTER	 | 令 Hyperscan 编译一个近似版本的模式串，一般用于前置过滤器
      L | HS_FLAG_SOM_LEFTMOST | 返回成功匹配的起始位置（默认返回结束位置）
  
示例：
```
1:/hatstand.*teakettle/s
2:/(hatstand|teakettle)/iH
3:/^.{10,20}hatstand/m
```

另外，模式串也同样支持 2.1 节提到的扩展参数，在标记字符后以 “key=value” 的格式添加。示例：
```
1:/hatstand.*teakettle/s{min_offset=50,max_offset=100}
```

hsbench 使用示例如下：
```
$ hsbench -e /tmp/patterns -c /tmp/corpus.db

Signatures:        /tmp/patterns
Hyperscan info:    Version: 4.3.1 Features:  AVX2 Mode: STREAM  // 编译模式数据库的 Hyperscan 版本；目标平台架构；扫描模式
Expression count:  200            // 模式串的数量
Bytecode size:     342,540 bytes  // 编译好的模式数据库的大小
Database CRC:      0x6cd6b67c     // 编译好的模式数据库的 CRC32
Stream state size: 252 bytes
Scratch size:      18,406 bytes
Compile time:      0.153 seconds
Peak heap usage:   78,073,856 bytes  // 编译模式数据库时所用的峰值内存

Time spent scanning:     0.600 seconds
Corpus size:             72,138,183 bytes (63,946 blocks in 8,891 streams)
Scan matches:            81 (0.001 matches/kilobyte)
Overall block rate:      2,132,004.45 blocks/sec
Overall throughput:      19,241.10 Mbit/sec
```

hsbench 还可以接收 “-T” 参数使 Hyperscan 在指定的一个或多个核心上工作。



## 7. 资料来源

1. [官方开发手册](http://01org.github.io/hyperscan/dev-reference/intro.html)
2. [PERFORMANCE ANALYSIS OF HYPERSCAN WITH HSBENCH](https://01.org/zh/node/7367)
