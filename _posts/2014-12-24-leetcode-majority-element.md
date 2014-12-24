---
layout: post
title: "leetcode: Majority Element "
description: ""
category: leetcode
tags: []
---
{% include JB/setup %}

题目：  

	Given an array of size n, find the majority element. The majority element is the element that appears more than ⌊ n/2 ⌋ times.

	You may assume that the array is non-empty and the majority element always exist in the array.

	Credits:
	Special thanks to @ts for adding this problem and creating all test cases.  

[题目地址](https://oj.leetcode.com/problems/majority-element/)    

	#include <iostream>
	#include <vector>
	#include <unordered_map>
	
	using namespace std;
	
	class Solution {
	public:
	    int majorityElement(vector<int> &num) {
		unordered_map<int,int> *map=new unordered_map<int,int>();
		int max=1;
		vector<int>::iterator iter=num.begin();
		map->insert(make_pair(*iter,1));
		int result=*iter;
		++iter;
		vector<int>::iterator end = num.end();
	        for(;iter!=end;++iter){
			unordered_map<int,int>::iterator tmp=map->find(*iter);
			if(tmp!=map->end()){
				if(tmp->second+1>max){
					result=*iter;
					max=tmp->second+1;
				}
				map->erase(*iter);
				map->insert(make_pair(*iter,tmp->second+1));
			}else{
				map->insert(make_pair(*iter,1));
			}
		}
		return result;
	    }
	};
	int main() {
	    Solution s;
	    int n[] = { 1,1,3,3,3,24,85,23,2};
	    vector<int> a(&n[0], &n[5]);
	    cout<<s.majorityElement(a)<<endl;
	    return 0;
	}


