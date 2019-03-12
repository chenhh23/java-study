### Reference
- [为什么新生代内存需要有两个Survivor区
](https://blog.csdn.net/antony9118/article/details/51425581)
- [两个Survivor区提升性能](https://segmentfault.com/q/1010000006886669)

### 结论
**为了解决s0区垃圾清除产生的内存碎片问题(垃圾清除对复制算法自身产生的内存碎片问题)。** <br/>
PS:另外一种解决方案是将s0区live object重新移动，使得live object占用连续内存，但这样会损失性能。

- 对象的内存分配在eden区
- 垃圾回收发生在eden区和s0区(其实是作为垃圾回收，新生代的内存空间都需要被回收，但是s1区是空，所以相当于发生在eden区和s0区)
- s0和s1区可以角色互换，相互复制
- 永远能够保持一个survivor是空的

基于 eden:from:to = 8:1:1, 上述机制最大的好处就是，整个过程中，永远有一个survivor space是空的，另一个非空的survivor space无碎片。刚刚新建的对象在Eden中，经历一次Minor GC，Eden中的存活对象就会被移动到第一块survivor space S0，Eden被清空；等Eden区再满了，就再触发一次Minor GC，Eden和S0中的存活对象又会被复制送入第二块survivor space S1（这个过程非常重要，因为这种复制算法保证了S1中来自S0和Eden两部分的存活对象占用连续的内存空间，避免了碎片化的发生）。S0和Eden被清空，然后下一轮S0与S1交换角色，如此循环往复。如果对象的复制次数达到16次，该对象就会被送到老年代中。<br/>


#### JVM内存
![image](https://img-blog.csdn.net/20160516144358110)

#### 两块Survivor垃圾回收过程
![image](https://img-blog.csdn.net/20160516174938778)

#### 一块Survivor垃圾回收过程
![image](https://img-blog.csdn.net/20160516173704870){
