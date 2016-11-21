title: "#算法笔记# 堆排序和优先队列"
date: 2016-05-08 16:31:12
tags: [笔记, 算法]
category: 算法笔记
---
最近重新复习了下堆排序和优先队列，这里做下笔记。

## 堆排序实现
堆排序实现使用最大堆，时间复杂度最好最差平均都是$O(NlogN)$。
```cpp
int leftChild(int idx) {
    return (idx + 1) * 2 - 1;
}
int rightChild(int idx) {
    return (idx + 1) * 2;
}
void maxHeap(int idx, vector<int>& nums, int size) {
    if (idx >= size) return;
    int maxVal = nums[idx], maxIdx = idx;
    int leftIdx = leftChild(idx), rightIdx = rightChild(idx);
    if (leftIdx < size && maxVal < nums[leftIdx]) {
        maxIdx = leftIdx;
        maxVal = nums[leftIdx];
    }
    if (rightIdx < size && maxVal < nums[rightIdx]) {
        maxIdx = rightIdx;
        maxVal = nums[rightIdx];
    }
    if (maxIdx != idx) {
        swap(nums[idx], nums[maxIdx]);
        maxHeap(maxIdx, nums, size);
    }
}
void heapSort(vector<int>& nums) {
    int size = nums.size();
    //建堆从后往前，size/2-1是最后一个有子节点的结点
    for (int i = size / 2 - 1; i >= 0; i--) maxHeap(i, nums, size);
    while (size > 0) {
        swap(nums[0], nums[size - 1]);
        size--;
        maxHeap(0, nums, size);
    }
}
```
<!-- more  -->
## 优先队列实现
优先队列实现这里我实现了个最大堆的。其中插入一个元素，去掉最大的元素，这些操作时间复杂度都是$O(lgN)$，$N$是指当前队列中元素数量。
可以用优先队列求解TopK问题，时间复杂度是$Nlog(K)$，$N$指总的元素的数量。
```cpp
class pQueue {
public:
    pQueue(int s) : maxSize(s), size(0) {
        space = vector<int>(maxSize, 0);
    }
    int getMax() {
        if (size == 0) return INT_MIN;
        return space[0];
    }
    bool extractMax(int& maxRes) {
        if (size == 0) return false;
        maxRes = space[0];
        swap(space[0], space[size - 1]);
        size--;
        maxHeap(0);
        return true;
    }
    bool insert(int val) {
        if (size == maxSize) return false;
        size++;
        space[size - 1] = INT_MIN;
        increaseHeap(size - 1, val);
    }
private:
    int leftChild(int idx) {
        return (idx + 1) * 2 - 1;
    }
    int rightChild(int idx) {
        return (idx + 1) * 2;
    }
    int parent(int idx) {
        return idx % 2 == 0 ? idx / 2 - 1 : idx / 2;
    }
    void maxHeap(int idx) {
        if (idx >= size) return;
        int maxVal = space[idx], maxIdx = idx;
        int leftIdx = leftChild(idx), rightIdx = rightChild(idx);
        if (leftIdx < size && maxVal < space[leftIdx]) {
            maxIdx = leftIdx;
            maxVal = space[leftIdx];
        }
        if(rightIdx < size && maxVal < space[rightIdx]) {
            maxIdx = rightIdx;
            maxVal = space[rightIdx];
        }
        if (maxIdx != idx) {
            swap(space[idx], space[maxIdx]);
            maxHeap(maxIdx);
        }
    }
    //注意这里，是自底向上的进行修改、交换
    bool increaseHeap(int idx, int val) {
        if (idx >= size || space[idx] >= val) return false;
        space[idx] = val;
        while (parent(idx) >= 0 && space[parent(idx)] < space[idx]) {
            swap(space[parent(idx)], space[idx]);
            idx = parent(idx);
        }
        return true;
    }
    vector<int> space;
    int size;
    int maxSize;
};
```

## 堆排序总结
堆排序时间复杂度最好最差平均都是$O(NlogN)$，不需要额外的空间，但是不稳定。

虽然堆排序在时间复杂度上与快排相比要更优。但是堆排序的时间常数比快排要大一些，因此通常会慢一些。

而且快排在递归进行部分的排序的时候，只会访问局部的数据，因此缓存能够更大概率的命中；而堆排序的建堆过程是整个数组各个位置都访问到的，后面则是所有未排序数据各个位置都可能访问到的，所以不利于缓存发挥作用。简答的说就是快排的存取模型的局部性（locality）更强，堆排序差一些。