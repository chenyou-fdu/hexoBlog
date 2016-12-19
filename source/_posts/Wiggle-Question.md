title: "LeetCode上的Wiggle类题目"
date: 2016-12-17 09:51:19
tags: [算法, LeetCode]
category: 算法
mathjax: true
---
LeetCode里有几道关于摇摆（Wiggle，数组元素满足一高一低这类条件）数组的题，这里来总结一下。

## Wiggle Sort 
这题在LeetCode上要钱，我在LintCode上找到了替代——[摆动排序](http://www.lintcode.com/zh-cn/problem/wiggle-sort/)。这里数组内元素可以相等，所以比较简单，一般有两种解法。
第一种是通过排序后，除了第一个元素之外，每两个元素交换一下位置就行（例如`nums[1]`和`nums[2]`，`nums[3]`和`nunms[4]`）。参考代码：
```cpp
void wiggleSort(vector<int>& nums) {
    sort(nums.begin(), nums.end());
    for(int i = 2; i < nums.size(); i += 2)
        swap(nums[i], nums[i-1]);
}
```

另一种就是用贪心法来解，设当前位置下标为`i`，如果`i`是奇数，那么需要满足`nums[i-1] >= nums[i]`；如果`i`是偶数，那么需要满足`nums[i-1] <= nums[i]`，如果出现不满足的情况，就交换下`nums[i-1]`和`nums[i]`就行了。
这种解法的证明可以参考[My explanations of the best voted Algo](https://discuss.leetcode.com/topic/42605/my-explanations-of-the-best-voted-algo)。参考代码：
```cpp
void wiggleSort(vector<int>& nums) {
    for(int i = 1; i < nums.size(); i++)
        if(i % 2 == 1 && nums[i-1] > nums[i] || 
        	i % 2 == 0 && nums[i-1] < nums[i]) 
            swap(nums[i-1], nums[i]);
}
```
<!-- more  -->

## Wiggle Sort II
LeetCode上有Wiggle Sort这题的加难版，[Wiggle Sort II](https://leetcode.com/problems/wiggle-sort-ii/)。这题比较直观的就是排序来解。解法的时间复杂度为$O(nlogn)$，需要用到$O(n)$大小的额外空间。排序后把数组分成两部分，然后两部分都逆序依次填入新数组中。参考代码：
```cpp
void wiggleSort(vector<int>& nums) {
    // 注意这里mid的求法，不能简单的size/2
    int size = nums.size(), mid = 0 + (size-1)/2, fPtr = mid, sPtr = size-1, ptr = 0;
    vector<int> tmp = nums;
    sort(tmp.begin(), tmp.end());
    while(fPtr >= 0 || sPtr > size/2) {
        nums[ptr++] = fPtr >= 0 ? tmp[fPtr--] : tmp[sPtr--];
        nums[ptr++] = sPtr > mid ? tmp[sPtr--] : tmp[fPtr--];
    }
}
```

进一步优化一下，可以发现这里不一定需要排序的。可以首先寻找到中值（用Kth Largest方法或者STL自带的`nth_element`函数都可以），然后把数组用解[荷兰国旗问题（Dutch National Flag）](https://en.wikipedia.org/wiki/Dutch_national_flag_problem)的三路划分算法分成小于中值的、等于中值的和大于中值的。然后就和上一种解法一样了，逆序依次填入新数组中就行。
这种解法的时间复杂度是$O(n)$（`nth_element`函数一般的实现都是用[Quickselect算法](https://en.wikipedia.org/wiki/Quickselect)，这种算法的平均时间复杂度是$O(n)$，最坏时间复杂度是$O(n^2)$），需要用到$O(n)$的额外空间。
```cpp
void ThreeWayPartition(vector<int>& nums, int midVal) {
    int size = nums.size(), fPtr = 0, tPtr = size-1, ptr = 0;
    while(ptr < tPtr) {
        if(nums[ptr] > midVal) swap(nums[ptr], nums[tPtr--]);
        else if(nums[ptr] < midVal) swap(nums[ptr++], nums[fPtr++]);
        else ptr++;
    }
}
void wiggleSort(vector<int>& nums) {
    int size = nums.size();
    // 寻找中值
    auto midPtr = nums.begin()+size/2;
    nth_element(nums.begin(), midPtr, nums.end());
    int midVal = *midPtr;
    // 三路划分
    ThreeWayPartition(nums, midVal);
    // 逆序插入
    int mid = 0 + (size-1)/2, fPtr = mid, sPtr = size-1, ptr = 0;
    vector<int> tmp = nums;
    while(fPtr >= 0 || sPtr > size/2) {
        nums[ptr++] = fPtr >= 0 ? tmp[fPtr--] : tmp[sPtr--];
        nums[ptr++] = sPtr > mid ? tmp[sPtr--] : tmp[fPtr--];
    }
}
```

还可以更进一步优化，就是比较难了。可以观察到，大于中值的，顺着放到奇数格子里；小于中值的逆着放在偶数格子里，剩下的放等于中值的。这种解法可以总结出一个下标映射的数学规律，这样就可以在做三路划分时直接放，不用额外空间（这里前提条件是找中值没用用额外空间）。
下标映射用的是`(1 + 2 * index) % (n|1)`，这个正好可以把大于中值的映射到奇数上，小于中值的映射到偶数上。具体列表一下就可以看出来，取模是为了保证不超过数组长，而`n|1`是保证奇偶性。
```cpp
void wiggleSort(vector<int>& nums) {
    int size = nums.size();
    if(size <= 1) return;
    // 寻找中值
    auto midPtr = nums.begin()+size/2;
    nth_element(nums.begin(), midPtr, nums.end());
    int midVal = *midPtr;
    auto indexMap = [size](int index) { return (1+2*index) % (size | 1);};
    // 注意这里和一般三路划分算法的不同，小于中值的放后面，大于中值的放前面
    int i = 0, j = size-1, k = 0;
    while(k <= j) {
        if(nums[indexMap(k)] < midVal) swap(nums[indexMap(k)], nums[indexMap(j--)]);
        else if(nums[indexMap(k)] > midVal) swap(nums[indexMap(k++)], nums[indexMap(i++)]);
        else k++;
    }
}
```
## Wiggle Subsequence
[Wiggle Subsequence](https://leetcode.com/problems/wiggle-subsequence/)和之前两题不一样，是个动态规划的题。
 最直观的DP，时间复杂度是$O(n^2)$，类似最长上升子序列那种解法，但是这里需要两个数组，一个记录当前最长序列，另一个来记录当前序列是需要一个上升的元素，还是一个下降的元素。参考代码

```cpp
int wiggleMaxLength(vector<int>& nums) {
    if(nums.empty()) return 0;
    int size = nums.size();
    // dpFlag[i]表示第i个元素是需要大于i-1还是小于i-1
    // 0表示需要小于i-1，1表示需要大于i-1，而-1是指第一个元素，大小无所谓
    // dpValue[i]表示从0...i这些元素中，符合要求的最长子序列长度
    vector<int> dpValue(size, 1), dpFlag(size, -1);
    for(int i = 1; i < size; i++)
        for(int j = 0; j < i; j++) {
            if(nums[i] > nums[j] && dpFlag[j] != 0 && dpValue[i] < dpValue[j] + 1) {
                dpValue[i] = dpValue[j] + 1;
                dpFlag[i] = 0;
            }
            if(nums[i] < nums[j] && dpFlag[j] != 1 && dpValue[i] < dpValue[j] + 1) {
                dpValue[i] = dpValue[j] + 1;
                dpFlag[i] = 1;
            }
        }
    return dpValue[size-1];
}
```

参考[官方题解](https://leetcode.com/articles/wiggle-subsequence/)和[C++ 0ms O(N) dynamic programming solution](https://discuss.leetcode.com/topic/52052/c-0ms-o-n-dynamic-programming-solution)，可以用一种时间复杂度$O(n)$的解法来解。
* `up[i]`记录到i这个元素为止（不一定以`nums[i]`结尾），末尾两个元素为上升状态的子序列的长度最大值，而`down[i]`则相反，表示下降。
* 例如`nums[i] > nums[i-1]`时，可以构造一个新的上升末尾，那么就需要用前一个下降末尾状态的值加一得到新的上升末尾状态。这里假设当前下降末尾状态的序列以`N`结尾。
	* 如果`nums[i] > N`，那么很明显可以构造一个新的，更长的上升末尾状态。
	* 如果`nums[i] <= N`：
		* 首先`N != nums[i-1]`，`N = nums[i-1] >= nums[i]`和 `nums[i] > nums[i-1]`矛盾了。
		* 所以，利用条件`N >= nums[i] > nums[i-1]`这个条件，可以用`nums[i-1]`来替代`N`。这样仍然满足下降条件，同时又使`nums[i] > newN = nums[i-1]`，可以构成新的更长的上升末尾状态。
* 而对于之前的下降末尾状态呢，用`nums[i]`是无法构成一个更长的下降末尾的，首先下降某末尾要是`nums[i-1]`，那肯定不行因为`nums[i]`更大；要末尾不是`nums[i-1]`时，用`nums[i]`来增加长度和用`nums[i-1]`来增加长度没有区别了。
* 当`nums[i] < nums[i-1]`情况类似。当然这个DP也可以优化空间到`O(1)`，比较简单了。

```cpp
int wiggleMaxLength(vector<int>& nums) {
    if(nums.empty()) return 0;
    int size = nums.size();
    vector<int> up(size, 0), down(size, 0);
    up[0] = 1, down[0] = 1;
    for(int i = 1; i < size; i++) {
        if(nums[i] > nums[i-1]) {
            up[i] = down[i-1]+1;
            down[i] = down[i-1];
        } else if(nums[i] < nums[i-1]) {
            up[i] = up[i-1];
            down[i] = up[i-1]+1;
        } else {
            up[i] = up[i-1];
            down[i] = down[i-1];
        }
    }
    return max(up[size-1], down[size-1]);
}
```

还有一种更简单的解法。如果每次构造上升序列时，都选择值最大的那个元素（递增元素里最后的那个）；而构造下降序列时，选择值最小的那个元素（递减元素里最后那个），这个序列肯定是最长的。这样就可以贪心法来解，参考代码：

```cpp
int wiggleMaxLength(vector<int>& nums) {
    if(nums.size() <= 1) return (int)nums.size();
    int prevDiff = nums[1] - nums[0], cnt = prevDiff == 0 ? 1 : 2;
    for(int i = 2; i < nums.size(); i++) {
        int nowDiff = nums[i] - nums[i-1];
        // 如果一直递增或者一直递减，就不选当前的元素，直到发生变化
        // 注意这里是大于等于
        if((prevDiff <= 0 && nowDiff > 0) || (prevDiff >= 0 && nowDiff < 0)) {
            prevDiff = nowDiff;
            cnt++;
        } 
    }
    return cnt;
}
```

## 总结
这几个题感觉都挺好的，难道适中，考察的也是很重要的点。比如荷兰国旗问题的解法，还有TopK问题的解法，以及动态规划的一些东西。

### 其他参考
* [Clean average O(n) time in c++](https://discuss.leetcode.com/topic/32904/clean-average-o-n-time-in-c)
* [nth_element implementations complexities](http://stackoverflow.com/questions/11068429/nth-element-implementations-complexities)
* [O(n)-time O(1)-space solution with detail explanations](https://discuss.leetcode.com/topic/32920/o-n-time-o-1-space-solution-with-detail-explanations)
