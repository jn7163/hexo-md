---
title: c语言 - 获取随机数
date: 2017-07-06 14:20:00
categories:
- c
tags:
- c
keywords: C语言获取随机数, 伪随机数, srand(), rand()
---

> 
c语言 - 获取随机数，实际上，计算机只能为我们提供`伪随机数`，所谓`伪随机数`就是按照一定算法模拟产生的，其结果是确定的，是可见的；
计算机产生随机数的过程，是根据一个**种子**为基准，以某个**递推公式**推算出来的一系列数，当递推的**范围足够大、往复性足够强、又符合正态分布或平均分布时**，我们就可以认为这是一个近似的**真随机数**

<!-- more -->

## 随机数
C语言中，头文件`stdlib.h`为我们提供了`rand()`函数来`生成随机数`和`srand()`来提供`随机数种子`
所谓种子可以看作随机数序列的名字，`一个种子对应一串随机数序列`，当种子不变时，就会按照随机数序列依次输出随机数
如果我们没有设置种子，那么默认种子就是1
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <stdlib.h>

int main(){
    int i;
    for(i=0; i<10; i++){
        printf("%d\n", rand());
    }
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~ [14:29:25]
$ gcc a.c

# root @ localhost in ~ [14:29:45]
$ ./a.out
1804289383
846930886
1681692777
1714636915
1957747793
424238335
719885386
1649760492
596516649
1189641421

# root @ localhost in ~ [14:29:46]
$ ./a.out
1804289383
846930886
1681692777
1714636915
1957747793
424238335
719885386
1649760492
596516649
1189641421
</script></code></pre>

## 设置种子
`srand()`函数用于设置种子，种子必须是一个`整数`
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <stdlib.h>

int main(){
    int i;
    srand(10);
    for(i=0; i<10; i++){
        printf("%d\n", rand());
    }
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~ [14:31:35]
$ gcc a.c

# root @ localhost in ~ [14:31:37]
$ ./a.out
1215069295
1311962008
1086128678
385788725
1753820418
394002377
1255532675
906573271
54404747
679162307
</script></code></pre>

可以发现，要想有随机效果，种子必须不同
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <stdlib.h>
#include <time.h>

int main(){
    int i;
    srand(time(NULL));  // 以当前的时间为种子，time(NULL)返回自1970-01-01 00:00:00到现在经过的秒数
    for(i=0; i<10; i++){
        printf("%d\n", rand());
    }
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~ [14:34:15]
$ gcc a.c

# root @ localhost in ~ [14:35:03]
$ ./a.out
755659272
918696383
1713099657
910474541
458968796
1562218559
1956314424
1331007040
316575616
708425496

# root @ localhost in ~ [14:35:04]
$ ./a.out
459856438
1815399079
224249789
1607706283
328815573
2046924047
1669414821
904580298
985276193
1977635311

# root @ localhost in ~ [14:35:05]
$ ./a.out
150856145
546472499
870233700
1215991304
1250685114
1442271952
291226543
452263359
560933092
1079063603
</script></code></pre>

套用两次运算，让随机序列之间的差异更明显
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <stdlib.h>
#include <time.h>

int main(){
    int i;
    srand(time(NULL));
    srand(rand() * rand());
    for(i=0; i<10; i++){
        printf("%d\n", rand());
    }
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~ [14:40:23]
$ gcc a.c

# root @ localhost in ~ [14:40:42]
$ ./a.out
1170207345
363934337
411175658
140900928
1727039420
1508971834
2103549883
1663524748
1111330508
1720806146

# root @ localhost in ~ [14:40:43]
$ ./a.out
2118351144
1741386816
1854616703
2110552475
593903520
2026527922
1809913760
1523501166
824123056
209918970
</script></code></pre>
