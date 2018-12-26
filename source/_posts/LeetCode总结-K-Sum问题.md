---
title: LeetCode总结--K Sum问题
date: 2017-08-03 10:46:53
tags: [算法,Leetcode]
---

最近在刷 LeetCode 上的题为找工作提前做准备，在刷 Array 类问题的 Easy 难度的题的时候觉得还好，大部分的题还是能够想得出来，但是在刷到 Medium 难度的时候，明显感觉难度提升了，其中有一类题型连续出现引起了我的注意，就是 K Sum 问题。

>K Sum问题就是，给出一个数作为 target，和一个数组作为待查数组，在这个待查数组里找出 K 个数之和等于 target.

## 最简单的 2 Sum

2 Sum应该是最简单的了，但是它是解决我们后面 3 Sum和 4 Sum 问题的最小子问题了。如果一开始就用暴力循环的方法当然是做的出来，但是，一是 LeetCode 可能会判超时(在 3 Sum 和 4 Sum 问题中确实会超时) ，二是暴力穷举也没什么意思。最简单的穷举法是 O(n^2) 的时间复杂度，那么我们可以将数组排好序后，用头尾两个指针一次迭代即可找出，时间复杂度降为了 O(nlogn).

## 3 Sum 问题

接下来就是3 Sum问题了，用暴力法 LeetCode 已经给出了超时的提醒了，所以O(n^3)的时间复杂度肯定是不能接受的，那么我们可以想像如何分解问题，毕竟3 Sum=2 Sum + 1 Sum，那么我们完全可以用将上面 2 Sum 问题的解法嵌入到一层循环中，也就是先排序然后一个指针从头开始循环，在数组的其他数中找出 2个数的和加上这个指针指向的数的和等于 target就行了，也就是说我们已经有了循环指针指向的这个数 V，那么我们只需要在其他书中找出 num1+num2=target-V就行了，这就又把问题带回到了2 Sum上，中不过多了一层循环，时间复杂度就降为了 O(n^2)。

代码实例：

```java
/*注题目
Given an array S of n integers, are there elements a, b, c in S such that a + b + c = 0? Find all unique triplets in the array which gives the sum of zero.

Note: The solution set must not contain duplicate triplets.
*/
public class Solution {
   public static List<List<Integer>> threeSum(int[] nums) {
        List<List<Integer>> result=new ArrayList<>();
        Arrays.sort(nums);
        for(int i=0;i<nums.length-2;i++)
        {
            if(i>0&&nums[i]==nums[i-1]) //跳过相同的数以避免相同的List的产生
            {
                continue;
            }
            find(result,nums,i+1,nums.length-1,nums[i]);
        }
       return result;
   }
    public static void find(List<List<Integer>> res,int[] nums,int start,int end,int target)
    {
        int l=start,r=end;
        while(l<r)
        {
            if(target+nums[l]+nums[r]==0)
            {
                List<Integer> list=new ArrayList<>();
                list.add(target);
                list.add(nums[l]);
                list.add(nums[r]);
                res.add(list);
                while(l<r&&nums[l]==nums[l+1])//跳过相同的数
                {
                    l++;
                }
                while(l<r&&nums[r]==nums[r-1])//跳过相同的数
                {
                    r--;
                }
                l++;
                r--;
            }else if(target+nums[l]+nums[r]<0)//如果和小于0(target),l右移扩大和
            {
                l++;
            }else//否则左移缩小和
            {
                r--;
            }
        }
    }
        
}
```



## 3Sum Closest 问题

3Sum Closest 问题是3 Sum 问题的一个变种，意思就是给出一个 target，找出数组中最接近 target的 3 个数之和。这道题和 3 Sum 很像，不同的是需要用一个临时变量来保存当前最接近 target 的值，值得注意的是我们需要求 3个数的和减去 target 的差的绝对值来求最接近 target 的数。

```java
/*
Given an array S of n integers, find three integers in S such that the sum is closest to a given number, target. Return the sum of the three integers. You may assume that each input would have exactly one solution.
    For example, given array S = {-1 2 1 -4}, and target = 1.

    The sum that is closest to the target is 2. (-1 + 2 + 1 = 2).
*/
public class Solution {
     public static int threeSumClosest(int[] nums, int target) {
        int result=Integer.MAX_VALUE;//用来保存当前最接近target的值
        Arrays.sort(nums);
        for(int i=0;i<nums.length-2;i++)
        {
            int temp=find(nums,i+1,nums.length-1,target,nums[i]);
            if(i==0)
            {
                result=temp;
                continue;
            }
            if(Math.abs(temp-target)<Math.abs(result-target))
            {
                result=temp;
            }
        }
        return result;
    }

    public static int find(int[] nums,int start,int end,int target,int curV)
    {
        int result=Integer.MAX_VALUE;
        int l=start,r=end;
        int c=Integer.MAX_VALUE;
        while(l<r&&c!=0)
        {
            if(Math.abs(curV+nums[l]+nums[r]-target)<c)
            {
                result =curV+nums[l]+nums[r];
                c=Math.abs(curV+nums[l]+nums[r]-target);
            }
            if(curV+nums[l]+nums[r]>target)
            {
                r--;
            }else
            {
                l++;
            }
        }
        return result;
    }
}
```



## 4 Sum 问题

4 Sum问题也显而易见了，就是求 4 个数的和等于target，那么我们可以如法炮制，前面的3 Sum我们是固定一个数，然后求两个数的和，将其分解为 2 Sum + 1 Sum，呢么 4 Sum 我们可以同样的先固定两个数，与另外两个活动的数相加等于target，也就输 4 Sum=3 Sum+1Sum。再在3 Sum的基础上嵌套一层循环就可以达到目的了，时间复杂度也降低为了O(n^3)。

```java
/*
Given an array S of n integers, are there elements a, b, c, and d in S such that a + b + c + d = target? Find all unique quadruplets in the array which gives the sum of target.

Note: The solution set must not contain duplicate quadruplets.

For example, given array S = [1, 0, -1, 0, -2, 2], and target = 0.

A solution set is:
[
  [-1,  0, 0, 1],
  [-2, -1, 1, 2],
  [-2,  0, 0, 2]
]
*/
public class Solution {
    public static List<List<Integer>> fourSum(int[] nums, int target) {
        List<List<Integer>> res=new ArrayList<>();
        Arrays.sort(nums);
        for(int i=0;i<nums.length-3;i++)//固定第一个数
        {
            if(i>0&&nums[i]==nums[i-1])//跳过相同的数
            {
                continue;
            }
            for(int j=i+1;j<nums.length-2;j++)//固定第二个数
            {
                if(j>i+1&&nums[j]==nums[j-1])//跳过相同的数
                {
                    continue;
                }
                find(nums,res,j+1,nums.length-1,target,nums[i],nums[j]);
            }
        }
        return res;
    }
    public static void find(int[] nums,List<List<Integer>> res,int start,int end,int target,int v1,int v2)
    {
        int l=start,r=end;
        while(l<r)
        {
            if(nums[l]+nums[r]+v1+v2==target)
            {
                List<Integer> list=new ArrayList<>();
                list.add(nums[l]);
                list.add(nums[r]);
                list.add(v1);
                list.add(v2);
                res.add(list);
                while(l<r&&nums[l]==nums[l+1])
                {
                   l++;
                }
                while(l<r&&nums[r]==nums[r-1])
                {
                   r--;
                }
                l++;
                r--;
            }else if(nums[l]+nums[r]+v1+v2<target)
            {
                l++;
            }else
            {
                r--;
            }
        }
    }
    
}
```

## 总结

做了 30 多到题，发现一个规律，如果排序后不使用二分查找或者双指针利用排序特性的话，那么排序书没有意义的，同时利用 Hash 特性也可以在增大空间复杂的同时减小时间复杂度。