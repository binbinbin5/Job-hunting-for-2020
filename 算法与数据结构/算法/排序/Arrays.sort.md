Timsort排序是结合了合并排序（merge sort）和插入排序（insertion sort）而得出的排序算法。（参考：Timsort原理介绍https://blog.csdn.net/sinat_35678407/article/details/82974174）

Timsort的核心过程

>TimSort 算法为了减少对升序部分的回溯和对降序部分的性能倒退，将输入按其升序和降序特点进行了分区。排序的输入的单位不是一个个单独的数字，而是一个个的块-分区。其中每一个分区叫一个run。针对这些 run 序列，每次拿一个 run 出来按规则进行合并。每次合并会将两个 run合并成一个 run。合并的结果保存到栈中。合并直到消耗掉所有的 run，这时将栈上剩余的 run合并到只剩一个 run 为止。这时这个仅剩的 run 便是排好序的结果。

https://www.jianshu.com/p/d7ba7d919b80