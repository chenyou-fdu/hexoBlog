title: "位图排序"
date: 2015-08-21 11:02:53
tags: [算法]
category: 算法
---

这篇是看过编程珠玑里位图排序的部分后，进行了一些实现，然后简单记录下。

## 实现位向量
注意这里实现一个位向量的方法：
```cpp
#define BITPERWORD 32 //这里用的是int类型，在这里是32位
#define N 10000000 //指定一共多少个数字需要在位向量中表示
#define SHIFT 5 //定义了一个偏移量5，意义是 2^5 = 32
#define MASK 0x1F //0x1F的二进制表示是31个1，用于位操作     
int a[1+N/BITSPERWORD]; // 初始化了一个数组，用(1+N/BITSPERWORD)个32位数表示N个需要记录的数

//set函数将某一位置为1
//首先做i>>SHIFT，相当于i/32，这样就确定了是用第几个int中的某一位来存这个数
//然后是i&MASK，这里相当于i%32，确定这个int中的第几位用来存这个数。
//然后根据上面的值对1左移，做OR操作就可以了。
void set(int i){
    a[i>>SHIFT] |=  (1<<(i&MASK)); 
}

//clr函数将某一位清0，这里的过程和之前的set函数基本一样
//但是要注意的是这里用来清0的数要取反，而且做的是AND操作
void clr(int i){
    a[i>>SHIFT] &= ~(1<<(i&MASK));
}

//这里验证某一位是否为0
//但是注意这里当某一位不是0时，返回的不是1，而是该位为1其他位是0的一个数
void test(int i){ 
    return a[i>>SHIFT] &   (1<<(i&MASK));
}
```
<!-- more  -->

## 位图排序
这里我实现位图排序用的是STL库的`vector`，不需要提前定义`N`。
```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <ctime>
#define BITSPERWORD 32
#define MASK 0x1F
#define SHIFT 5
using namespace std;
void set(int i, vector<int>& bitVector) {
    bitVector[i >> SHIFT] |= (1 << (i&MASK));
}
void clr(int i, vector<int>& bitVector) {
    bitVector[i >> SHIFT] &= (1 << ~(i&MASK));
}
int get(int i, const vector<int>& bitVector) {
    return bitVector[i >> SHIFT] & (1 << (i&MASK));
}
void bitVectorSort(vector<int>& nums) {
    int size = nums.size();
    vector<int> bitVector(size/BITSPERWORD+1, 0);
    //需要计算需要排序的数组中最大值和最小值
    int minNum = INT_MAX, maxNum = INT_MIN; 
    for (auto i : nums) { //用位向量记录数
        minNum = min(minNum, i);
        maxNum = max(maxNum, i);
        set(i, bitVector);
    }
    int cnt = 0;
    //顺序输出到原数组中，完成排序
    for (int i = minNum; i <= maxNum; i++) {
        if (get(i, bitVector) != 0) nums[cnt++] = i;
    }
}
```

这里我简单的进行了一下速度上的比较，在这种输入数据情况下，位图排序要比STL库中的排序快5倍左右。
```cpp
int main() {
    vector<int> numsA;
    vector<int> numsB;
    for (int i = 10000000; i >= 0; i--) {
        numsA.push_back(i);
        numsB.push_back(i);
    }
    clock_t startTime = clock();
    bitVectorSort(numsA);
    float secsElapsed1 = (float)(clock() - startTime) / CLOCKS_PER_SEC;
    startTime = clock();
    sort(numsB.begin(), numsB.end());
    float secsElapsed2 = (float)(clock() - startTime) / CLOCKS_PER_SEC;
    cout << secsElapsed1 << endl;
    cout << secsElapsed2 << endl;
    getchar();
    return 0;
}
```