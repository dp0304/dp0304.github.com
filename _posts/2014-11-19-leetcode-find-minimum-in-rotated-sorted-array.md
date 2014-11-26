---
layout: post
title: "leetcode: Find Minimum in Rotated Sorted Array"
description: ""
category: leetcode
---
{% include JB/setup %}


题目：   

	Suppose a sorted array is rotated at some pivot unknown to you beforehand.

	(i.e., 0 1 2 4 5 6 7 might become 4 5 6 7 0 1 2).

	Find the minimum element.

	You may assume no duplicate exists in the array.

[题目地址](https://oj.leetcode.com/problems/find-minimum-in-rotated-sorted-array/)   
  
解题思路：  
  二分法查找，当pos小于pos-1的时候，则为要求的值。而不命中时候，通过pos和begin，end的大小比较来选择保留左端还是右端。当pos>begin且pos>end的时候，保留右端。


	
	#include <iostream>
	#include <vector>
	
	
	using namespace std;
	
	
	class Solution {
	public:
	    int findMin(vector<int> &num) {
	
	        return find(num, num.size()/2)   ;
	    }
	
	private:
	    int find(vector<int> &numVector,int pos){
	       if(pos==0||numVector[pos]<numVector[pos==0?0:pos-1]){
	           return numVector[pos];
	       } else{
	           if(numVector[0]<numVector[pos]&&numVector[numVector.size()-1]<numVector[pos]){
	               numVector.erase(numVector.begin(), numVector.begin()+pos) ;
	               pos = numVector.size()/2;
	               if(numVector.size()==1){
	                   return   numVector[0];
	               } else {
	                   return find(numVector, pos);
	               }
	           } else{
	               numVector.erase(numVector.begin()+pos, numVector.end()) ;
	               pos = numVector.size()/2;
	               if(numVector.size()==1){
	                   return   numVector[0];
	               } else {
	                   return find(numVector, pos);
	               }
	           }
	       }
	    }
	};
	
	int main() {
	    Solution s;
	    int n[] = { 1,2};
	    vector<int> a(&n[0], &n[2]);
	    cout<<s.findMin(a)<<endl;
	
	
	
	    int n2[] = {3,4,5,2};
	    vector<int> a2(&n2[0], &n2[5]);
	    cout<<s.findMin(a2)<<endl;
	
	
	    int n3[] = {4,5,6,7,0,1,2};
	    vector<int> a3(&n3[0], &n3[7]);
	    cout<<s.findMin(a3)<<endl;
	
	    int n4[] = {2,3,1};
	    vector<int> a4(&n4[0], &n4[3]);
	    cout<<s.findMin(a4)<<endl;
	
	    int n5[] = { 5,1,2,3,4};
	    vector<int> a5(&n5[0], &n5[5]);
	    cout<<s.findMin(a5)<<endl;
	
	    return 0;
	
	}