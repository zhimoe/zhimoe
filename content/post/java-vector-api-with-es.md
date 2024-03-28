+++
title = 'Java Vector Api 和 Elasticsearch 性能提升'
date = '2023-11-18T09:45:54+08:00'
categories = ['编程']
tags = ['code','java']
toc = true
+++

OpenJDK project panama 中一个重点功能就是 vector api，可以显著提升矩阵计算密集型程序的性能，例如在图形计算、机器学习、大规模计算（Lucene）等。

<!--more-->

### 什么是 SIMD
CPU 的每个处理单元一次运算只能计算一个值，这个值成为标量值（scalar value）。处理单元需要 0 或多个周期（即 CPU 频率）完成一次操作计算。现代 CPU 包含多个核心，每个核心包含很多处理单元，这样就提供了并行在处理单元上面执行操作的能力。

当进行大规模计算时，例如从海量数据源中添加大量数字，程序可以将数据分割成更小的数据块并将它们分布在多个线程中，能够获得更快的处理速度。这是进行并行计算的方法之一。但是即便如此，这些操作还是属于 SISD，即单指令单数据。

SIMD(Single Instruction Multiple Data) 表示单指令多数据。SIMD 处理器是特殊的处理器，它没有多线程的概念，依赖多个处理单元，能够在一个 CPU 周期中执行相同的操作，但是每个处理单元的数据不相同，因此得名。SIMD 在现实中有很多实际的例子，比如两个数组相加。

与处理器从内存加载标量值的方式不同，SIMD 机器在操作之前将内存中的整数数组加载到寄存器中。SIMD 硬件的组织方式使得值数组的加载操作能够在单个周期内进行。SIMD 机器允许我们并行地对数组执行计算，而无需实际依赖并发编程。

由于 SIMD 机器将内存视为数组或一系列值，因此我们将其称为向量，并且 SIMD 机器执行的任何操作都成为向量操作。因此，这是一种利用 SIMD 架构原理来执行并行处理任务的非常强大且高效的方法。

### Java Vector API
Vector API 希望让开发人员能够以与平台无关的方式编写数据并行软件，充分利用 SIMD 处理器的优势但是不用接触 CPU 架构（AMD or Intel）。
还是以数组相加为例子：

```java
package moe.zhi.boot.app;

import jdk.incubator.vector.IntVector;
import jdk.incubator.vector.VectorSpecies;

import java.util.Arrays;

import static java.lang.StringTemplate.STR;

public class VectorDemo {
    public static void main(String[] args) {
        int[] a = new int[]{1, 2, 3, 4, 9, 8, 7, 6, 9, 8, 7, 6, 9, 8, 7, 6, 1, 1};
        int[] b = new int[]{9, 8, 7, 6, 9, 8, 7, 6, 9, 8, 7, 6, 9, 8, 7, 6, 9, 8, 7, 6};
        int[] c = simpleSum(a, b);
        System.out.println(Arrays.toString(c));


        int[] d = vectorSum(a, b);
        System.out.println(Arrays.toString(d));
    }

    public static int[] simpleSum(int[] a, int[] b) {
        var c = new int[a.length];
        for (var i = 0; i < a.length; i++) {
            c[i] = a[i] + b[i];
        }
        return c;
    }

    private static final VectorSpecies<Integer> SPECIES = IntVector.SPECIES_PREFERRED;

    public static int[] vectorSum(int[] a, int[] b) {
        var c = new int[a.length];
        var upperBound = SPECIES.loopBound(a.length);
        System.out.println(STR. "upperBound = \{ upperBound }" );
        var i = 0;
        for (; i < upperBound; i += SPECIES.length()) {
            var va = IntVector.fromArray(SPECIES, a, i);
            var vb = IntVector.fromArray(SPECIES, b, i);
            var vc = va.add(vb);
            vc.intoArray(c, i);
        }
        // Compute elements not fitting in the vector alignment.
        for (; i < a.length; i++) {
            System.out.println(STR. "current i is \{ i }" );
            c[i] = a[i] + b[i];
        }

        return c;

    }

}
// 运行
//❯ java --enable-preview --source 21 --add-modules jdk.incubator.vector VectorDemo.java
//WARNING: Using incubator modules: jdk.incubator.vector
//注：VectorDemo.java 使用 Java SE 21 的预览功能。
//注：有关详细信息，请使用 -Xlint:preview 重新编译。
//[10, 10, 10, 10, 18, 16, 14, 12, 18, 16, 14, 12, 18, 16, 14, 12, 10, 9]
//upperBound = 16
//current i is 16
//current i is 17
//[10, 10, 10, 10, 18, 16, 14, 12, 18, 16, 14, 12, 18, 16, 14, 12, 10, 9]
```
可以看到 vector api 相对更加复杂，包含了一个位宽移动和扫尾操作。简单的 for 循环使用 i++ 将索引加一；但是在 vector api 中，需要根据 SIMD 寄存器的数据宽度进行移位。最后，还需要处理未在数据宽度内对齐的所有剩余项目。

### Lanes, Shapes and Species 

CPU 的 SIMD 处理器位数一般为 64 到 512 位，一个 int 向量每个 int 值是 32 bits，对于 64 位的 SIMD 处理器，则有 2 个分量，在 vector api 中称为通道（lane）。这里 64 称为 vector 的 shape。vector 的 shape 和数据类型称为 species -- 通过`class VectorSpecies<E>`表示。

vector 的运算分为通道运算和跨通道运算（lane-wise operations and cross-lane operations）。是不是和线性代数的知识联系起来了？

`class Vector` 对于六种支持类型中的每一种都有六个抽象子类：`ByteVector、ShortVector、IntVector、LongVector、FloatVector 和 DoubleVector`。对于 SIMD 机器来说，特定的实现非常重要，这就是为什么形状特定的子类进一步扩展了每种类型的这些类。例如 `Int128Vector、Int512Vector` 等。


### Elasticsearch 中 vector api 的使用
ES 在 JDK20 以上已经可以使用 vector api，具体 PR:[Enable the Panama Vector module #96453](https://github.com/elastic/elasticsearch/pull/96453)。关于性能提升的一点[参考](https://github.com/elastic/elasticsearch/issues/96370)：

>   indexing-throughput improved by 30%
>   merge-time decreased by 40%
>  script-score-query-java-latency improved by 40%

[作者更加详细的博客：Accelerating vector search with SIMD instructions](https://www.elastic.co/cn/blog/accelerating-vector-search-simd-instructions)
需要注意的是 SIMD 严重依赖 CPU 架构，而且 API 当前还是 incubator 阶段，上面的结果只是参考。

具体使用 vector api 参考 [org.apache.lucene.internal.vectorization.PanamaVectorUtilSupport](https://github.com/apache/lucene/blob/b3ef869681a5cbabc7e6ae3b497c88bdec057b71/lucene/core/src/java20/org/apache/lucene/internal/vectorization/PanamaVectorUtilSupport.java#L29)