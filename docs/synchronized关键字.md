##synchronized关键字

> JAVA关键字，为了解决的是多个线程之间访问资源的同步性而诞生，synchronized关键字可以保证被它修饰的方法或者代码块在任意时刻只能有一个线程执行

###作用范围
● 修饰普通方法

普通方法加锁是对象级别的，不同的对象锁也不一样与静态方法是全局的不太一样

● 修饰静态方法

静态方法来说 synchronized 加锁是全局的，同一个时间只能一个线程访问，与修饰普通方法不同对象掉拿到的锁也不同，两者的差异如此

● 修饰代码块

日常中最常用的方式，不会直接修饰在方法上，而是修饰在代码上

**synchronized修饰的方法与代码块，无论正常执行完毕还是抛出异常，都会释放锁，与lock锁需要手动释放不太一样**

###锁的升级
####偏向锁
bject head 对象头 mark wrok会使用54bit记录具体锁的指针，锁标志为100，如果没有发生竞争，则不需要发生上下文切换，避免线程cpu用户态切换到内核态开销
####自旋锁
两个线程通过自旋cas的方式 将Mark word 62 bit 改为 自己栈空间中的 lock Record，哪一个线程修改成功，则哪一个线程竞争到锁，则另外一个线程尝试获取锁会继续往复cas做空循环自旋操作，锁标志为01， 默认是10次，可以通过`-XX:PreBlockSpin` 配置
####自适应自选锁
处理器自旋时间不固定，而是根据前一次在同一个锁上的自旋时间和锁的拥有者的状态来决定的，避免出现刚好自旋完，锁就被释放的情况
####重量级锁
自旋超过10次或等待的线程超过cpu核心数的1/3，则线程竞争比较激烈，自选旋已经无法保证高效的情况，膨胀为重量级锁，mark wrok 锁标志为10，获取不到锁直接进入阻塞状态

####可重入锁
一个线程在最外层方法中获取到锁，进入里层方法时候，如果锁对象是同一个的。则不需要再次获取
![](https://foruda.gitee.com/images/1685612189459292357/f471616f_5094274.png)
**轻量级锁是用户态，不需要和内核打交道**

###CAS的具体实现

jdk 11 jdk.internal.misc.unsafe 

```
    @HotSpotIntrinsicCandidate
    public final long compareAndExchangeLongAcquire(Object o, long offset,
                                                           long expected,
                                                           long x) {
        return compareAndExchangeLong(o, offset, expected, x);
    }
```
**unsafe 的具体实现是由c++汇编指令实现**

```
inline jint     Atomic::cmpxchg    (jint     exchange_value, volatile jint*     dest, jint     compare_value) {
  int mp = os::is_MP();
  __asm__ volatile (LOCK_IF_MP(%4) "cmpxchgl %1,(%3)"
                    : "=a" (exchange_value)
                    : "r" (exchange_value), "a" (compare_value), "r" (dest), "r" (mp)
                    : "cc", "memory");
  return exchange_value;
}
```
`cmpxchgl`  对应底层汇编指令 `lock  cmpxchgl`，通过lock指令会锁定CPU的缓存行或者锁总线来执行原子操作

###JAVA对象头
![](https://foruda.gitee.com/images/1685612213913887394/cbd298fa_5094274.png)
**64位虚拟机中对象头**
![](https://foruda.gitee.com/images/1685612241666875965/aaa2f199_5094274.png)
举个例子 ， 如果new 一个空的对象出来，那么内存分布如下

```
java.lang.Object object internals:
 OFFSET  SIZE   TYPE DESCRIPTION                               VALUE
      0     4        (object header)                           05 00 00 00 (00000101 00000000 00000000 00000000) (5)
      4     4        (object header)                           00 00 00 00 (00000000 00000000 00000000 00000000) (0)
      8     4        (object header)                           00 10 00 00 (00000000 00010000 00000000 00000000) (4096)
     12     4        (loss due to the next object alignment)
Instance size: 16 bytes
Space losses: 0 bytes internal + 4 bytes external = 4 bytes total
```
**Mark Word**

对象头Mark Word 8字节
![](https://foruda.gitee.com/images/1685612275113145669/461ba4d0_5094274.png)
**Calss Pointer**

指针一般为8个字节，java对象指针经过压缩后为4字节
![](https://foruda.gitee.com/images/1685612315729473960/3dbaf687_5094274.png)
**Instance Data**

对象数据，因为我们new出来的属性都是空，所以 Instance Data 为 0 字节
**padding**
> loss due to the next object alignment

以上总共加起来为12字节，由于内存对齐符合以下设计规范

● 通常内存是由一个个字节组成的，cpu在存取数据时，并不是以字节为单位存储，而是以块为单位存取

● 块的大小为内存存取力度。频繁存取字节未对齐的数据，会极大降低cpu的性能，所以可以通过减少存
最后又`padding`进行内存对其，补齐4个字节，最后object 字节大小为16字节

###Monitor.Enter() 和Monitor.Exit()
synchronized 会在开始时候在同步代码块开始时添加 Monitor.Enter 指令再同步代码块结束时添加Monitor.Exit指令