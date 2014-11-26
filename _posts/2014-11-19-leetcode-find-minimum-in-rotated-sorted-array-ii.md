---
layout: post
title: "leetcode: Find Minimum in Rotated Sorted Array II"
description: ""
category: leetcode
tags: []
---
{% include JB/setup %}
题目： 
  
	Follow up for "Find Minimum in Rotated Sorted Array":
	What if duplicates are allowed?

	Would this affect the run-time complexity? How and why?
	Suppose a sorted array is rotated at some pivot unknown to you beforehand.

	(i.e., 0 1 2 4 5 6 7 might become 4 5 6 7 0 1 2).

	Find the minimum element.

	You may assume no duplicate exists in the array.

[题目地址](https://oj.leetcode.com/problems/find-minimum-in-rotated-sorted-array-ii/)   
  
解题思路：  
  先做去重操作，再用上一次的函数进行运算。




	#include <iostream>
	#include <vector>
	
	
	using namespace std;
	
	
	class Solution {
	public:
	    int findMin(vector<int> &num) {
	        delDup(num);
	        return find(num, num.size() / 2);
	    }
	
	private:
	    int find(vector<int> &numVector, int pos) {
	        if (pos == 0 || numVector[pos] < numVector[pos == 0 ? 0 : pos - 1]) {
	            return numVector[pos];
	        } else {
	            if (numVector[0] < numVector[pos] && numVector[numVector.size() - 1] < numVector[pos]) {
	                numVector.erase(numVector.begin(), numVector.begin() + pos);
	                pos = numVector.size() / 2;
	                if (numVector.size() == 1) {
	                    return numVector[0];
	                } else {
	                    return find(numVector, pos);
	                }
	            } else {
	                numVector.erase(numVector.begin() + pos, numVector.end());
	                pos = numVector.size() / 2;
	                if (numVector.size() == 1) {
	                    return numVector[0];
	                } else {
	                    return find(numVector, pos);
	                }
	            }
	        }
	    }
	
	    void delDup(vector<int> &num) {
	        typedef vector<int>::iterator VIntIterator;
	        VIntIterator end = num.end()-1;
	        for (VIntIterator i = num.begin(); i != end; ++i) {
	
	            if (*i == *(i + 1)) {
	                num.erase(i);
	                --i;
	                --end;
	            }
	        }
	    }
	};
	
	int main() {
	    Solution s;
	
	    int n3[] = {2,2,1, 2, 2, 2, 2, 2, 2};
	    vector<int> a3(&n3[0], &n3[3]);
	    cout << s.findMin(a3) << endl;
	
	    return 0;
	
	}	