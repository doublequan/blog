---

layout: post
title: 'Subsets问题 Bitwise Operation 位操作 '
date: '2015-11-17'
header-img: "img/post-bg-web.jpg"
tags:
     - bit
     - 位操作
author: 'Bill Quan'

---

## Subsets

来源： [https://leetcode.com/problems/subsets/](https://leetcode.com/problems/subsets/ "subsets")

Given a set of distinct integers, nums, return all possible subsets.

Note:

Elements in a subset must be in non-descending order.
The solution set must not contain duplicate subsets.

For example,
If nums = [1,2,3], a solution is:

	[
	  [3],
	  [1],
	  [2],
	  [1,2,3],
	  [1,3],
	  [2,3],
	  [1,2],
	  []
	]

### Solutions:

迭代解法**Iteratively**:

	class Solution(object):
	    def subsets(self, nums):
	        """
	        :type nums: List[int]
	        :rtype: List[List[int]]
	        """
	        list = [[]]
	        for n in sorted(nums):
	            for i in range(0, len(list)):
	                list += [list[i] + [n]]
	        return list

位操作解法**Bit Manipulation**    

	# Bit Manipulation    
	def subsets2(self, nums):
	    res = []
	    nums.sort()
	    for i in xrange(1<<len(nums)):
	        tmp = []
	        for j in xrange(len(nums)):
	            if i & 1 << j:  # if i >> j & 1:
	                tmp.append(nums[j])
	        res.append(tmp)
	    return res

解释explanation：
	Number of subsets for {1 , 2 , 3 } = 2^3 .
	 why ? 
	case    possible outcomes for the set of subsets
	  1   ->          Take or dont take = 2 
	  2   ->          Take or dont take = 2  
	  3   ->          Take or dont take = 2 
	
	therefore , total = 2*2*2 = 2^3 = { { } , {1} , {2} , {3} , {1,2} , {1,3} , {2,3} , {1,2,3} }
	
	Lets assign bits to each outcome  -> First bit to 1 , Second bit to 2 and third bit to 3
	Take = 1
	Dont take = 0
	
	0) 0 0 0  -> Dont take 3 , Dont take 2 , Dont take 1 = { } 
	1) 0 0 1  -> Dont take 3 , Dont take 2 ,   take 1       =  {1 } 
	2) 0 1 0  -> Dont take 3 ,    take 2       , Dont take 1 = { 2 } 
	3) 0 1 1  -> Dont take 3 ,    take 2       ,      take 1    = { 1 , 2 } 
	4) 1 0 0  -> take 3      , Dont take 2  , Dont take 1 = { 3 } 
	5) 1 0 1  -> take 3      , Dont take 2  ,     take 1     = { 1 , 3 } 
	6) 1 1 0  -> take 3      ,    take 2       , Dont take 1 = { 2 , 3 } 
	7) 1 1 1  -> take 3      ,      take 2     ,      take 1     = { 1 , 2 , 3 } 
	
	In the above logic ,Insert S[i] only if (j>>i)&1 ==true   { j E { 0,1,2,3,4,5,6,7 }   i = ith element in the input array }
	
	element 1 is inserted only into those places where 1st bit of j is 1 
	   if( j >> 0 &1 )  ==> for above above eg. this is true for sl.no.( j )= 1 , 3 , 5 , 7 
	
	element 2 is inserted only into those places where 2nd bit of j is 1 
	   if( j >> 1 &1 )  == for above above eg. this is true for sl.no.( j ) = 2 , 3 , 6 , 7
	
	element 3 is inserted only into those places where 3rd bit of j is 1 
	   if( j >> 2 & 1 )  == for above above eg. this is true for sl.no.( j ) = 4 , 5 , 6 , 7 
	
	Time complexity : O(n*2^n) , for every input element loop traverses the whole solution set length i.e. 2^n



<br>

> 如有任何知识产权、版权问题或理论错误，还请指正。
>
> 转载请注明原作者及以上信息。
