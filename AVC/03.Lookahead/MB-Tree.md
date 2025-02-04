# MB Tree

帧之间存在引用关系，参考帧的质量显然更重要。

## 原理

根据宏块在帧间预测中贡献给未来帧（在编码顺序里位于当前帧之后的帧）的信息，即被参考的情况，来调整该宏块的QP值。简言之，如果该MB贡献给后续帧的信息越多，则其重要性越高，应当提高该区域的编码质量，减少QP，反之，则增大该区域的QP。

要知道当前宏块对未来帧的贡献程度，则需要从未来帧反推有多少信息是来自于当前宏块的，由于未来帧还没有被编码，需要通过前向预测（lookahead）来进行预测。前向预测通过对一定数量的未编码帧做快速运动估计，对这些帧的编码代价进行预估。经过前向预测可以得到未编码帧的如下参数的估计值：

+ 帧类型
+ 每个宏块的类型和运动矢量
+ 帧内编码和帧间编码的SATD值

MB-tree算法对于每个宏块计算了两个参数：遗传代价propagate cost和继承率propagate fraction，遗传代价表示了当前宏块贡献给未来帧的信息，继承率代表当前宏块的信息有多少来自于参考帧。假设lookahead的数量为n帧，则宏块-tree按照编码顺序相反的方向，从lookahead的最后一帧开始到当前帧的所有帧，依次对每一帧的宏块重复如下步骤：

1. 读取当前宏块的帧内代价intra cost，帧间代价inter cost，propagate cost。前向预测的最后一帧propagate cost为0。

2. 根据当前宏块的inter cost 和intracost来计算propagate fraction，假设一个宏块的inter cost只有intracost的80%，说明帧间预测节省了20% 的编码消耗，则该宏块的信息有20%来自于参考帧，propagate fraction为0.2。

3. 和当前宏块有关的所有信息总和大约为intra cost+ propagate cost（自身信息和提供给后续帧的信息）, 使用这个总和乘以继承率propagate fraction，可以得到来继承自参考帧的信息量propagate amount。

4. 将propagate amount划分给给参考帧中相关的宏块，由于当前宏块在参考帧中运动搜索得到的补偿区域可能涉及多个宏块，即参考帧中的多个宏块都参与了当前宏块的运动补偿，所以我们根据参考帧宏块参与补偿的部分尺寸大小来分配propagate amount 。特别的，对于B帧，我们把propagate amount 先平分给两个参考帧，再进一步分配给参考帧中的宏块。参考帧中的宏块最终被分到的propagate amount加起来就是它的propagate cost。

5. 从前向预测的最后一帧向前一直计算到当前帧，可以得到当前帧中每个宏块对后续n帧的propagatecost，最后根据当前帧每个宏块的propagate cost，计算相应的偏移系数qp_offset

