---
layout: post
title: "leetcode: Evaluate Reverse Polish Notation "
description: ""
category: leetcode
tags: []
---
{% include JB/setup %}
题目： 
  
	Evaluate the value of an arithmetic expression in Reverse Polish Notation.

	Valid operators are +, -, *, /. Each operand may be an integer or another expression.

	Some examples:
  	["2", "1", "+", "3", "*"] -> ((2 + 1) * 3) -> 9
  	["4", "13", "5", "/", "+"] -> (4 + (13 / 5)) -> 6

[题目地址](https://oj.leetcode.com/problems/evaluate-reverse-polish-notation/)   
  
解题思路：  
  用栈做存储，数字都push进去，当遇到运算符时候pop出两个数字运算，计算完的结果在push回去。  


	#include <iostream>
	#import <vector>
	#include <stack>
	
	using namespace std;
	class Solution {
	public:
	    int evalRPN(vector<string> &tokens) {
	        stack<int> *num =new stack<int>;
	        for(int index=0;index<tokens.size();index++){
	            switch (tokens[index][0]){
	                case '*': {
	                    int a=num->top();
	                    num->pop();
	                    int b=num->top();
	                    num->pop();
	                    num->push(b*a);
	                    continue;
	                }
	                case '/': {
	                    int a=num->top();
	                    num->pop();
	                    int b=num->top();
	                    num->pop();
	                    num->push(b/a);
	                    continue;
	                }
	                case '+': {
	                    int a=num->top();
	                    num->pop();
	                    int b=num->top();
	                    num->pop();
	                    num->push(b+a);
	                    continue;
	                }
	                case '-': {
	                    if(tokens[index].size()==1){
	                    int a=num->top();
	                    num->pop();
	                    int b=num->top();
	                    num->pop();
	                    num->push(b-a);  }
	                    else{
	                        num->push(stoi(tokens[index]));
	                    }
	                    continue;
	                }
	                default: {
	                    num->push(stoi(tokens[index]));
	                }
	
	            }
	        }
	        return num->top();
	    }
	};
	
	
	int main() {
	    Solution s;
	
	    string n4[] = {"-2","3","+","4","32","-","*"};
	    vector<string> a4(&n4[0], &n4[7]);
	    cout<<s.evalRPN(a4)<<endl;
	
	    return 0;
	
	}	