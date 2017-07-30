---
title: (c语言)数据结构与算法 - 查找和排序
date: 2017-07-30 15:48:00
categories:
- c
- 数据结构与算法
tags:
- c
- 数据结构与算法
keywords: c语言 数据结构与算法 查找 排序 算法
---

> 
一些最基本的查找、排序算法(Ｔ▽Ｔ)：`二分查找`、`冒泡排序`、`选择排序`、`插入排序`

<!-- more -->

## 查找
**`二分查找`**
> 
二分査找也称折半査找，其优点是查找速度快，缺点是要求所要査找的数据必须是`有序序列`

<pre><code class="language-c line-numbers"><script type="text/plain">int binsearch(int *arr, int len, int key){
    int min = 0, max = len - 1;
    while(min <= max){
        int mid = (min + max) / 2;
        int midval = arr[mid];
        if(key > midval){
            min = mid + 1;
        }else if(key < midval){
            max = mid - 1;
        }else{
            return mid;
        }
    }
    return -1;
}
</script></code></pre>

## 排序
<pre><code class="language-c line-numbers"><script type="text/plain">// 冒泡排序
void bubble_sort(int *arr, int len){
    for(int i=0; i<len-1; i++){
        int sorted = 1;
        for(int j=0; j<len-1-i; j++){
            if(arr[j] > arr[j+1]){
                sorted = 0;
                int tmp = arr[j];
                arr[j] = arr[j+1];
                arr[j+1] = tmp;
            }
        }
        if(sorted){
            break;
        }
    }
}

// 选择排序
void select_sort(int *arr, int len){
    for(int i=0; i<len-1; i++){
        int min_index = i;
        for(int j=i+1; j<len; j++){
            if(arr[j] < arr[min_index]){
                min_index = j;
            }
        }
        if(min_index != i){
            int tmp = arr[i];
            arr[i] = arr[min_index];
            arr[min_index] = tmp;
        }
    }
}

// 插入排序
void insert_sort(int *arr, int len){
    for(int i=1; i<len; i++){
        int j = 0;
        while(arr[j] < arr[i] && j < i){
            j++;
        }
        if(j != i){
            int tmp = arr[i];
            for(int k=i; k>j; k--){
                arr[k] = arr[k-1];
            }
            arr[j] = tmp;
        }
    }
}
</script></code></pre>
