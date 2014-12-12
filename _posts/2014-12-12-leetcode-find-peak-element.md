---
layout: post
title: "leetcode: Find Peak Element "
description: ""
category: 
tags: []
---
{% include JB/setup %} 

题目： 
	A peak element is an element that is greater than its neighbors.

	Given an input array where num[i] ≠ num[i+1], find a peak element and return its index.

	The array may contain multiple peaks, in that case return the index to any one of the peaks is fine.

	You may imagine that num[-1] = num[n] = -∞.

	For example, in array [1, 2, 3, 1], 3 is a peak element and your function should return the index number 2.   

[题目地址](https://oj.leetcode.com/problems/find-peak-element/)    



	#include <iostream>
	#import <vector>
	
	
	using namespace std;
	
	class Solution {
	public:
	    int findPeakElement(const vector<int> &num) {
	        int i=0;
	        for(;i<num.size()-1;++i){
	            if(num[i]>num[i+1]){
	                return i;
	            }
	        }
	        return  i;
	    }
	};
	int main() {
	    Solution s;
	    int n[] = { 1,2,3,2,3,4,5,3,2};
	    vector<int> a(&n[0], &n[9]);
	    cout<<s.findPeakElement(a)<<endl;
	    return 0;
	
	}

