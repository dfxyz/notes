# Java GC

## GC Roots
* 虚拟机栈中引用的对象
* Native栈中引用的对象
* 方法区: 类静态属性 \ 常量
* JVM内部引用: 基本数据类型的Class \ ClassLoader \ 常驻的异常对象

## 引用类型
* Strong Reference
* Soft Reference: 快OOM时才列入GC范围
* Weak Reference: 只能生存到下一次GC前
* Phantom Reference: 没法用来访问对象, "used for scheduling pre-mortem cleanup actions in a more flexible way than is possible with the Java finalization mechanism"

## 回收方法区 (永久代或元空间)
* 常量, 主要是字符串
* 类型, 需满足以下条件
    * 该类实例均已回收
    * 该类的Class对象不被任何人引用
    * 该类的ClassLoader已被回收

## 分代收集理论
* 大部分对象存活期极短
* 挺过多次GC的对象难以死亡
* 跨代引用的情况极少

## Mark-Sweep
* mark: 标记出待回收的对象 (或标记存活的对象)
* sweep: 就地回收

## Mark-Copy
* 将管理的内容分成两块, 每次只用其中一块
* mark: 标记出待回收的对象 (或标记存活的对象)
* copy: 将存活的对象复制到内存的另一块中, 并使用另一块内存
* 改进: Eden+Survivor的内存布局
    * 将内存划分为一块Eden区, 两块Survivor区
    * 每次使用Eden区和其中一块Survivor区
    * GC时将存活对象复制到另一块Survivor区
    * 如果Survivor区空间不足, 需要分配到其他内存区域中 (如: 老年代)

## Mark-Compact
* mark: 标记出待回收的对象 (或标记存活的对象)
* compact: 回收后移动存活对象消除内存碎片 (可能会 "stop the world")



## 根节点枚举
* 需要STW
* 通过OopMap记录引用在栈 \ 寄存器中的位置

## 安全点
* 记录OopMap的特定位置
* 用户逻辑执行到安全点时才能停下来做GC
* 一般放在方法调用 \ 循环跳转 \ 异常跳转处
* 让所有进程停在安全点:
    * 主动式: 设置一个标志 (可以用内存不可访问的trap来做), 每个线程执行时轮询, 标志生效时走到安全点时即主动停止
    * 抢占式: 强制停止所有用户线程, 如果线程不在安全点上, 恢复执行直至走到安全点

## 安全区域
* 在某个代码片段中, 引用关系不会变化 (片段中任意点都可开始GC)
* 解决线程挂起状态时, 没法进入安全点的问题
* 进安全区域时做个标记
* 出安全区域前检查是否已完成根节点枚举

## 记忆集
* 解决跨代引用的问题, 避免根节点枚举扫描整个老年代
* 卡表: 每个卡页对应一个内存块, 每个内存块有跨代引用时将卡页标记为脏
* 写屏障: 修改对象引用字段时, 检查并更新卡表

## 并发的可达性分析
* 遍历对象图, 使用三色标记:
    * 白色: 未访问的对象; 遍历结束时为需要GC的对象
    * 灰色: 已访问的对象, 但引用链上至少有一个对象还未访问过
    * 黑色: 已访问的对象, 且引用链上的其他对象都访问了
* 并发的用户逻辑可能影响遍历结果的正确性, 当且仅当以下两个条件同时发生:
    * 某黑色对象新增一条到白色对象的引用
    * 所有从灰色对象指向某白色对象的直接或间接引用都被删除
    * (即某个灰色对象直接或间接引用的白色对象转移给某个黑色对象来引用)
* 解决遗漏白色对象的方法:
    * 增量更新: 记录新增白色对象引用的黑色对象, 并发扫描完成后重新扫描这些黑色对象
    * 原始快照: 删除灰色对象到白色对象的引用时, 记录这个引用关系, 并发扫描完成后重新扫描这些 "灰色对象"
    * (需要用写屏障实现)

## Serial \ Serial Old
* 新生代: 单线程Mark-Copy (STW)
* 老年代: 单线程Mark-Compact (STW)
* 低内存开销

## ParNew
* 新生代: 多线程Mark-Copy (STW)
* 老年代: 使用Serial Old或CMS

## Parallel Scavenge \ Parallel Old 
* 吞吐量优先
* 支持自适应调节GC相关参数 (新生代大小 \ Eden与Survivor的比例等)
* 新生代: 多线程Mark-Copy (STW)
* 老年代: 多线程Mark-Sweep (STW)

## Concurrent Mark Sweep
* 六个主要阶段:
    * initial mark: STW, 标记GC Roots直接关联的对象
    * concurrent mark: 遍历对象图并进行标记
    * concurrent preclean: 重新扫描concurrent mark阶段中引用关系发生变化的对象
    * concurrent abortable preclean: 与concurrent preclean阶段一样, 但阶段结束时间可控
    * final remark: STW, 使用增量更新的方法, 重新扫描引用关系发生变化的对象
    * concurrent sweep: 就地清理死亡对象
* Concurrent Mode Failure发生时, 需要Full GC; Full GC时可用参数控制是否进行空间整理

## G1GC
* 将内存看成多个region, 每个region是单次回收的最小单位
* 每个region可以是Eden \ Survivor \ 老年代空间; 有特殊的区域专门放大对象
* G1GC负责跟踪各region中垃圾回收的价值大小, 并维护一个优先级队列, 每次优先回收价值大的region
* 解决跨区引用: 每个region的记忆集需要记录指向自己的其他region
* 解决并发标记时引用关系变化: 使用原始快照的方法; 新分配的对象放在特定区域, 隐式标记为存活
* 回收步骤:
    * initial marking: STW, 标记GC Roots直接关联的对象
    * concurrent marking: 并发扫描整堆的对象, 标记出可回收的
    * final marking: STW, 处理并发扫描时引用关系变动的对象
    * live data counting & evacuating: STW, 更新各region的统计数据, 指定回收计划, 将存活对象拷贝到新的region中

## Shenandoah
* 与G1GC有着类似region内存布局
* 回收步骤:
    * initial marking: STW, 同G1GC
    * concurrent marking: 同G1GC
    * final marking: STW, 同G1GC
    * concurrent cleanup: 并发清理完全没有存活对象的region
    * concurrent evacuation: 并发将存活对象复制到其他region中 (其他对象访问该对象时, 使用读屏障和转发指针实现访问的是已转移的对象)
    * initial update reference: STW, 准备将指向存活对象的指针更新, 该阶段先确保复制已经完成
    * concurrent update reference: 并发更新堆中对象到被转移的存活对象的引用 (直接按内存地址顺序线性扫描)
    * final update reference: STW, 更新GC Roots中到被转移的存活对象的引用
    * concurrent cleanup: 并发清理已转移的存活对象原先所在的region

## ZGC
* 基于region的内存布局, 但region可动态创建销毁, 有多种容量大小, 暂无分代
* 染色指针: 地址只使用低42位 (4TB), 43-46位用来辅助GC; 需要做虚拟地址映射
* 回收步骤:
    * pause mark start: STW, 标记GC Roots直接关联的对象
    * concurrent mark: 遍历对象图, 做可达性分析; GC标记记录在对象指针上
    * pause mark end: STW, 再次标记引用关系发生变化的对象
    * concurrent prepare for relocate: 根据特定条件统计本次GC要处理哪些region (有别于G1GC的优先回收行为)
    * pause relocate start: STW, 该阶段先确保上一阶段已结束
    * concurrent relocate: 将存活对象复制到新的region中, 原region负责维护一个转发表, 记录旧对象到新对象的转移; 通过引用访问已转移的旧对象时, 会转发到新对象, 同时也更新该引用 (这个过程被称为 "自愈", 只会有一次转发的开销); 某region中的对象全部转移后, 可以立即回收该region (转发表还得保留)
    * concurrent remap: 并发修正堆中其他对象到已转移存活对象的引用, 全修正后即可释放转发表
