---
title: Redis常见数据类型与命令
tags: [Redis]      #添加的标签
categories: Redis                           #添加的分类
description: 
#cover: 
---



## 全局命令

1. 查看键

   ```redis
   keys *
   ```

   支持模糊匹配，如：

   ```redis
   keys novel_to_novel_*_pahina
   查询结果：
   1) "novel_to_novel_doc_pahina"
   2) "novel_to_novel_gatne_pahina"
   ```

2. 键总数

   ```redis
   dbsize
   ```

   dbsize命令在计算键总数时不会遍历所有键，而是直接获取Redis内置的键总数变量，所以dbsize命令的时间复杂度是O（1）。而keys 命令会遍历所有键，所以它的时间复杂度是O（n）。

3. 检查键是否存在

   ```redis
   exists key
   ```

   如果键存在则返回1，不存在则返回0。

4. 删除键

   ```redis
   del key [key ...]
   ```

   返回结果为成功删除键的个数，假设删除一个不存在的键，就会返回0。

5. 键过期

   ```redis
   expire key seconds
   ```

   单位秒。ttl命令会返回键的剩余过期时间，它有3种返回值：

   - 大于等于0的整数：键剩余的过期时间。
   - -1：键没设置过期时间。
   - -2：键不存在。

   

   可以通过ttl命令观察键hello的剩余过期时间:

   ```redis
   ttl key
   ```

6. 键的数据结构类型

   ```redis
   type key
   ```





## 字符串类型

字符串类型的值实际可以是字符串（简单的字符串、复杂的字符串（例如 JSON、XML））、数字（整数、浮点数），甚至是二进制（图片、音频、视频），但是值最大不能超过512MB。

### 命令

1. 设置值

   ```redis
   set key value [ex seconds] [px milliseconds] [nx|xx]
   ```

   set命令有几个选项：

   - ex seconds：为键设置秒级过期时间。如设置过期时间50秒：set key value ex 50
   - px milliseconds：为键设置毫秒级过期时间。如设置过期时间50毫秒：set key value px 50
   - nx：键必须不存在，才可以设置成功，用于添加。如：set key value nx 50
   - xx：与nx相反，键必须存在，才可以设置成功，用于更新。

   除了set选项，Redis还提供了setex和setnx两个命令：

   ```redis
   setex key seconds value
   setnx key value
   ```

   setex在逻辑上等价于set和expire合并的操作或者等价于set+ex参数，设置一个值并且添加过期时间；

   setnx在逻辑上等价于set+nx参数。

2. 获取值

   ```redis
   get key		//键不存在，则返回空（nil）
   ```

3. 批量设置值

   ```redis
   mset key value [key value ...]
   ```

4. 批量获取值

   ```redis
   mget key [key ...]
   ```

   批量操作命令可以有效提高开发效率，如果使用get命令批量获取n个值，则需要耗时：<u>n次get时间 = n次网络时间 + n次命令时间</u>，模型如下：

   ![redis n次get的模型](https://gitee.com/hu-zhihong/picbed/raw/master/redis%20n%E6%AC%A1get%E7%9A%84%E6%A8%A1%E5%9E%8B.png)

   使用mget命令后，需要耗时：<u>n次get时间 = 1次网络时间 + n次命令时间</u>，模型如下：

   ![redis 一次mget的模型](https://gitee.com/hu-zhihong/picbed/raw/master/redis%20%E4%B8%80%E6%AC%A1mget%E7%9A%84%E6%A8%A1%E5%9E%8B.png)

5. 计数

   ```redis
   incr key
   ```

   incr命令用于对值做自增操作，返回结果分为三种情况：

   - 值不是整数，返回错误。
   - 值是整数，返回自增后的结果。
   - 键不存在，按照值为0自增，返回结果为1。



字符串类型命令时间复杂度如下：

![redis字符串类型命令时间复杂度](https://gitee.com/hu-zhihong/picbed/raw/master/redis%E5%AD%97%E7%AC%A6%E4%B8%B2%E7%B1%BB%E5%9E%8B%E5%91%BD%E4%BB%A4%E6%97%B6%E9%97%B4%E5%A4%8D%E6%9D%82%E5%BA%A6.png)



### 内部编码

字符串类型的内部编码有3种：

- int：8个字节的长整型。

- embstr：小于等于39个字节的字符串。

- raw：大于39个字节的字符串。 

Redis会根据当前值的类型和长度决定使用哪种内部编码实现。



## 哈希

在Redis中，哈希类型是指键值本身又是一个键值对结构，形如`value={{field1，value1}，...{fieldN， valueN}}`，Redis键值对和哈希类型二者的关系如下图：

![redis哈希类型关系](https://gitee.com/hu-zhihong/picbed/raw/master/redis%E5%93%88%E5%B8%8C%E7%B1%BB%E5%9E%8B%E5%85%B3%E7%B3%BB.png)

### 命令

1. 设置值

   ```redis
   hset key field value
   ```

   如果设置成功会返回1，反之会返回0。此外Redis提供了hsetnx命令，它们的关系就像set和setnx命令一样，只不过作用域由键变为field。

2. 获取值

   ```redis
   hget key field //如果键或field不存在，会返回nil
   ```

3. 删除field

   ```redis
   hdel key field [field ...]
   ```

   hdel会删除一个或多个field，返回结果为成功删除field的个数。

4. 计算field个数

   ```redis
   hlen key
   ```

5. 批量设置或获取field-value

   ```redis
   hmget key field [field ...]
   hmset key field value [field value ...]
   ```

6. 获取所有field

   ```redis
   hkeys key
   ```

7. 获取所有value

   ```redis
   hvals key
   ```

8. 获取所有的field-value

   ```redis
   hgetall key
   ```

   在使用hgetall时，如果哈希元素个数比较多，会存在阻塞Redis 的可能。如果开发人员只需要获取部分field，可以使用hmget，如果一定要获取全部field-value，可以使用hscan命令，该命令会渐进式遍历哈希类型。



哈希类型命令的时间复杂度如下：

![redis哈希类型命令的时间复杂度](https://gitee.com/hu-zhihong/picbed/raw/master/redis%E5%93%88%E5%B8%8C%E7%B1%BB%E5%9E%8B%E5%91%BD%E4%BB%A4%E7%9A%84%E6%97%B6%E9%97%B4%E5%A4%8D%E6%9D%82%E5%BA%A6.png)



### 内部编码

- ziplist（压缩列表）：当哈希类型元素个数小于hash-max-ziplist-entries配置（默认512个）、同时所有值都小于hash-maxziplist-value配置（默认64字节）时，Redis会使用ziplist作为哈希的内部实现，ziplist使用更加紧凑的结构实现多个元素的连续存储，所以在节省内存方面比hashtable更加优秀。
- hashtable（哈希表）：当哈希类型无法满足ziplist的条件时， Redis会使用hashtable作为哈希的内部实现，因为此时ziplist的读写效率会下降，而hashtable的读写时间复杂度为O（1）。



## 列表
