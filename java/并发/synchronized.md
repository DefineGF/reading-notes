### synchronized

原理：依赖于底层的操作系统的Mutex Lock来实现的，但是由于使用Mutex Lock需要将当前线程挂起并从用户态切换到内核态来执行，这种切换的代价是非常昂贵的



#### 状态

##### 无锁

##### 偏向锁

- 当一个线程访问同步块并获取锁时，会在对象头和栈帧中的锁记录里存储锁偏向的线程ID，
- 该线程在进入和退出同步块时不需要进行CAS操作来加锁和解锁。只需要简单的测试一下对象头的`Mark Word`里是否存储着指向当前线程的偏向锁。
  - 如果成功，表示线程已经获取到了锁

##### 轻量级锁

- 轻量级锁并不是替代重量级锁的，而是对在大多数情况下同步块并不会有竞争出现提出的一种优化；

- 它可以减少重量级锁对线程的阻塞带来的线程开销。从而提高并发性能。

##### 重量级锁



#### 特性

- 一把锁只能被一个线程获取；
- 一个实例只有一把锁（锁对象是*.class以及synchronized修饰的是static方法的时候，所有对象公用同一把锁）
- 无论正常执行或者抛出异常，都会释放锁；



#### 可重入

引用计数：

- 0：表示尚未有线程持有；
- 1：表示某线程获取锁，当该线程再次执行方法时，引用计数 + 1；
- 方法执行完毕，引用计数 - 1；



#### 优化

##### 锁粗化

将多个连续的锁扩展成一个范围更大的锁



##### 锁消除

消除一些没有在当前同步块以外被其他线程共享的数据的锁保护



##### 轻量锁

当存在锁竞争的情况下，执行CAS指令失败的线程将调用操作系统互斥锁进入到阻塞状态，当锁被释放的时候被唤醒



##### 偏向锁

为了在无锁竞争的情况下避免在锁获取过程中执行不必要的CAS原子指令



##### 适应性自旋

当线程在获取轻量级锁的过程中执行CAS操作失败时，在进入与monitor相关联的操作系统重量级锁(mutex semaphore)前会进入忙等待(Spinning)然后再次尝试，当尝试一定的次数后如果仍然没有成功则调用与该monitor关联的semaphore(即互斥锁)进入到阻塞状态



#### from chatgpt

##### 什么是偏向锁

> Java虚拟机（JVM）在对象级别实现的锁优化策略之一.
>
> 1. 初始状态下，一个对象的锁是无锁状态。
> 2. 当某个线程尝试获取这个对象的锁时，JVM会检查该对象是否处于偏向状态。
> 3. 如果对象处于无锁状态，JVM会尝试将线程ID记录在对象的锁记录中，并将对象的状态设置为偏向锁。
> 4. 当其他线程尝试获取该对象的锁时，JVM会检查对象的状态。
> 5. 如果对象处于偏向状态且线程ID与当前线程ID相同，那么该线程可以直接获取锁，无需进行额外的同步操作。
> 6. 如果对象处于偏向状态但线程ID与当前线程ID不同，表示有其他线程竞争该锁，JVM会撤销偏向锁，升级为重量级锁.
>
> 偏向锁适用于无竞争的场景，当存在多个线程竞争同一个锁时，偏向锁就会失效，转而使用其他的锁机制（如轻量级锁或重量级锁）


##### 轻量级锁和偏向锁

> 场景：
>
> - 偏向锁：适用于多线程环境下存在大量无竞争的情况。即一个线程多次获取同一个对象的锁。
> - 轻量级锁：适用于多线程环境下存在短时间内的竞争情况。即多个线程交替竞争同一个对象的锁。
>
> 状态：
>
> - 偏向锁：对象的初始状态是无锁状态，当一个线程获取到该对象的锁时，会将对象的状态设置为偏向锁。如果其他线程尝试获取该对象的锁，会撤销偏向锁，升级为重量级锁。
> - 轻量级锁：对象的初始状态是无锁状态。当一个线程获取到该对象的锁时，会将对象的锁记录（Lock Record）替换为线程自己的锁记录。如果其他线程尝试获取该对象的锁，会进行自旋操作，尝试获取锁。如果自旋失败，会升级为重量级锁。
>
> 开销：
>
> - 偏向锁：在无竞争的情况下，获取偏向锁的操作几乎没有额外的开销，提高了程序的性能。只有在存在竞争时，才会撤销偏向锁，转为重量级锁，涉及到额外的开销。
> - 轻量级锁：在无竞争的情况下，获取轻量级锁的操作也较为轻量，使用自旋进行短暂的等待。当自旋失败时，会升级为重量级锁，涉及到额外的开销。