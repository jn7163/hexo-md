---
title: c语言 - 判断与循环
date: 2017-07-02 16:23:00
categories:
- c
tags:
- c
keywords: C语言判断与循环
---

> 
c语言 - 判断与循环

<!-- more -->

## if else
如果只有一条语句，则可以省略`{}`
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <stdlib.h>

int main(){
    char c = '\n';
    printf("char: ");
    c = getchar();

    if(c == '\n'){
        printf("你没有输入任何东西!\n");
        exit(1);
    }

    printf("你输入的字符为: %c\n", c);

    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int main(){
    int age;
    printf("请输入你的年龄：");
    scanf("%d", &age);
    if(age>=18){
        printf("恭喜，你已经成年，可以使用该软件！\n");
    }else{
        printf("抱歉，你还未成年，不宜使用该软件！\n");
    }
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int main(){
    int num = -1;
    printf("输入一个正整数，判断大小范围: ");
    scanf("%d", &num);

    if(num >= 100){
        printf("大于等于100\n");
    }else if(num >= 80){
        printf("在80 ~ 100之间\n");
    }else if(num >= 60){
        printf("在60 ~ 80之间\n");
    }else if(num >= 40){
        printf("在40 ~ 60之间\n");
    }else if(num >= 20){
        printf("在20 ~ 40之间\n");
    }else if(num >= 0){
        printf("在0 ~ 20之间\n");
    }else{
        printf("输入有误\n");
    }

    return 0;
}
</script></code></pre>

## switch
当存在过多`if else`分支时，容易造成代码逻辑混乱，像下面这种情况就可以用`switch`语句替代
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int main(){
    int a;
    printf("Input integer number:");
    scanf("%d", &a);

    switch(a){
        case 1:
            printf("Monday\n");
            break;
        case 2:
            printf("Tuesday\n");
            break;
        case 3:
            printf("Wednesday\n");
            break;
        case 4:
            printf("Thursday\n");
            break;
        case 5:
            printf("Friday\n");
            break;
        case 6:
            printf("Saturday\n");
            break;
        case 7:
            printf("Sunday\n");
            break;
        default:
            printf("Error\n");
    }

    return 0;
}
</script></code></pre>

`break`用于跳出当前语句块，`default`语句不是必需的，要注意`case`后接的必须是一个`整型数值`或者是`整型表达式`(整型表达式在编译期间也会被优化成整型数值)，不能为变量名，即使变量的值为整型数值

## while
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int main(){
    int i = 0, array[5] = {1, 2, 3, 4, 5};
    while(i < 5){
        printf("array[%d] = %d\n", i, array[i]);
        i++;
    }
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~ [19:25:06]
$ gcc a.c

# root @ localhost in ~ [19:25:08]
$ ./a.out
array[0] = 1
array[1] = 2
array[2] = 3
array[3] = 4
array[4] = 5
</script></code></pre>

`do...while`是先执行`do`语句，然后再判断`while`语句
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int main(){
    int i = 0, array[5] = {1, 2, 3, 4, 5};
    do{
        printf("array[%d] = %d\n", i, array[i]);
        i++;
    }while(i < 5);
    return 0;
}
</script></code></pre>

## for
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int main(){
    int i, array[5] = {1, 2, 3, 4, 5};
    for(i = 0; i < 5; i++){
        printf("array[%d] = %d\n", i, array[i]);
    }
    return 0;
}
</script></code></pre>

## break continue
> 
`break`直接跳出循环
`continue`直接开始下一轮循环
