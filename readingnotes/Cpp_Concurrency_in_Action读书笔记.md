### 内存模型和原子类型操作
* #### 对象在内存中的位置4个原则
    1. 每一个变量都是都是一个对象，包括作为其成员变量的对象
    2. 每个对象至少占有一个内存位置
    3. 基本类型都有确定的内存位置
    4. 相邻位域是相同内存中的一部分
* 当两个线程访问同一个内存位置时，要小心
* 只读数据不需要保护和同步

* 为了避免条件竞争，两个线程需要一定的执行顺序
    1. 方式1：使用互斥量来确定访问顺序，当同一互斥量在两个线程同时访问前被锁住，那么在同一时间内只有一个线程能访问对应的内存位置，后一个访问必须在前一个访问之后、释放互斥量，才能获取互斥量并且访问对应的内存位置;
    2. 方式2： 使用原子操作同步机制

* #### 修改顺序
* #### 原子操作
    * 原子操作是一类补课分割的操作，当这样的操作在任意线程中执行到一半的时候，是不能查看的；它的状态要么是完成，要么是未完成。
    * ##### 六种内存序
        ```
        typedef enum memory_order {
            memory_order_relaxed,
            memory_order_consume,
            memory_order_acquire,
            memory_order_release,
            memory_order_acq_rel,
            memory_order_seq_cst
        } memory_order;
        ```
    * 将上述内存排序分成3类
        1. Store操作：memory_order_relaxed, memory_order_release, memory_order_seq_cst;
        2. Load操作： memory_order_relaxed, memory_order_consume, memory_order_acquire, memory_order_seq_cst;
        3. read-modify-write(读-改-写)操作： memory_order_relaxed, memory_order_consume,memory_order_acquire,memory_order_release,memory_order_acq_rel,memory_order_seq_cst；
    * 所有操作的默认顺序都是memory_order_seq_cst;

### 基于锁的并发数据结构的设计
原则：减小保护区域，减少序列化操作，提高并发访问能力。

建议：
* 确保访问安全，达到真正的并发访问
* 提供完整操作函数，而非操作步骤函数
* 降低死锁概率
* 减小保护区域，减小序列化操作范围，最大化并发访问效果