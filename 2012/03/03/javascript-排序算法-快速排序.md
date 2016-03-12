# [javascript]排序算法-快速排序

排序算法：快速排序

[原文出处.](http://promiseforever.com/redirect?url=http%3A%2F%2Fwww.cnblogs.com%2Fluchen927%2Farchive%2F2012%2F02%2F29%2F2368070.html&key=d6ec3bed0030a30da2cd5dc52cacb8b5)


快速排序采用的思想是分治思想。

快速排序是找出一个元素（理论上可以随便找一个）作为基准(pivot),然后对数组进行分区操作,使基准左边元素的值都不大于基准值,基准右边的元素值 都不小于基准值，如此作为基准的元素调整到排序后的正确位置。递归快速排序，将其他n-1个元素也调整到排序后的正确位置。最后每个元素都是在排序后的正 确位置，排序完成。所以快速排序算法的核心算法是分区操作，即如何调整基准的位置以及调整返回基准的最终位置以便分治递归。 举例说明一下吧，这个可能不是太好理解。假设要排序的序列为 **2** 2 4 9 3 6 7 1 5 首先用2当作基准，使用i j两个指针分别从两边进行扫描，把比2小的元素和比2大的元素分开。首先比较2和5，5比2大，j左移 **2** 2 4 9 3 6 7 1 5 比较2和1，1小于2，所以把1放在2的位置 **2** 1 4 9 3 6 7 1 5 比较2和4，4大于2，因此将4移动到后面 **2** 1 4 9 3 6 7 4 5 比较2和7，2和6，2和3，2和9，全部大于2，满足条件，因此不变 经过第一轮的快速排序，元素变为下面的样子 [1] 2 [4 9 3 6 7 5] 之后，在把2左边的元素进行快排，由于只有一个元素，因此快排结束。右边进行快排，递归进行，最终生成最后的结果。

```c
int quicksort(vector <int>&v, int left, int right){
        if(left < right){
                int key = v[left];
                int low = left;
                int high = right;
                while(low < high){
                        while(low < high && v[high] > key){
                                high--;
                        }
                        v[low] = v[high];
                        while(low < high && v[low] < key){
                                low++;
                        }
                        v[high] = v[low];
                }
                v[low] = key;
                quicksort(v,left,low-1);
                quicksort(v,low+1,right);
        }
}</int>
```

