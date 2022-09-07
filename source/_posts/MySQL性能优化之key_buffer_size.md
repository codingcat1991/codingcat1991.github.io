---
title: MySQL性能优化之key_buffer_size
category: MySQL
tags:
- MySQL
date: 2020-9-23
description: MyISAM表中的索引块在所有线程之间缓存和共享，key_buffer_size用来设置索引块使用的缓冲大小，key buffer也称key cache，key buffer用来缓存MyISAM表经常使用的索引。
---

MyISAM表中的索引块在所有线程之间缓存和共享，key_buffer_size用来设置索引块使用的缓冲大小，key buffer也称key cache，key buffer用来缓存MyISAM表经常使用的索引。

在32位平台上key_buffer_size最大可设置为4294967295B，即4G，64位平台更大。根据操作系统和硬件平台实际可用的RAM，实际生效的值比设置的值偏小。该值的大小即为MySQL申请的内存大小，内部会为该变量尽可能分配内存以达到该值，实际上分配的可能会偏小。

该参数大小决定索引处理的速度，尤其是索引读的速度，在只使用MyISAM存储引擎的数据库中，可以将该值设置为机器总内存的25%，不宜设置过大（大于及其总内存的50% ），这样的话系统会频繁切换内存块（MySQL依赖操作系统缓存来缓存数据，因此需要留点空间给操作系统），导致系统变慢，如果有使用其他的存储引擎，也需要将其他存储引擎的内存使用考虑进去。

key_buffer_size是个全局参数，只对MyISAM表有作用，因为MySQL中磁盘临时表使用的是MyISAM引擎，因此也需要使用到该值，对于1G内存的机器，如果不使用MyISAM表，推荐值是16M（8-64M）。

可以通过show status，检查Key_read_requests, Key_reads, Key_write_requests, 以及Key_writes 状态变量的值来检查key buffer的性能，正常情况下，Key_reads/Key_read_requests的值应该小于0.01。Key_reads是从磁盘读取索引块到key cache的次数，Key_read_requests是请求读取索引块的次数，Key_reads较大，即Key_reads/Key_read_requests大于0.01时，可以增加key_buffer_size的值使其小于0.01来提升查询性能。更新多的时候Key_writes/Key_write_requests的值接近1，如果更新影响很多行或者使用DELAY_KEY_WRITE表选项时，Key_writes/Key_write_requests的值会更小。

通过公式1 - ((Key_blocks_unused * key_cache_block_size) / key_buffer_size)可以计算出索引块使用率，其中Key_blocks_unused 表示未使用的索引块数key_cache_block_size表示索引块大小。