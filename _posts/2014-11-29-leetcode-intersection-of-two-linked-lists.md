---
layout: post
title: "leetcode: Intersection of Two Linked Lists  "
description: ""
category: leetcode
tags: []
---
{% include JB/setup %}

题目： 
	Write a program to find the node at which the intersection of two singly linked lists begins.
	
	
	For example, the following two linked lists:
	
	A:          a1 → a2
	                   ↘
	                     c1 → c2 → c3
	                   ↗            
	B:     b1 → b2 → b3
	begin to intersect at node c1.
	
	
	Notes:
	
	If the two linked lists have no intersection at all, return null.
	The linked lists must retain their original structure after the function returns.
	You may assume there are no cycles anywhere in the entire linked structure.
	Your code should preferably run in O(n) time and use only O(1) memory.

[题目地址](https://oj.leetcode.com/problems/intersection-of-two-linked-lists/)   
  
解题思路：  
  两条链如果有重合部分，一定是尾部重合。所以，先判断两条链的长度差。把较长的链剪去头部。然后两条相同的链便可以对应位置进行判断。  


		#include <iostream>
		
		
		using namespace std;
		
		
		 struct ListNode {
		     int val;
		     ListNode *next;
		     ListNode(int x) : val(x), next(NULL) {}
		 };
		
		class Solution {
		public:
		    ListNode *getIntersectionNode(ListNode *headA, ListNode *headB) {
		        if(headA== NULL || headB==NULL){
		            return NULL;
		        }		
		        int gapLength=compareLength(headA,headB);
		        //剪去长度
		        if(gapLength>0){
		            for(int i=0;i<gapLength;++i){
		               headA=headA->next;
		            }
		        }else {
		        	//负数取绝对值
		            int i = gapLength >> 31;
		            gapLength= ((gapLength ^ i) - i);
		            for(i=0;i<gapLength;++i){
		                headB=headB->next;
		            }
		        }
				//相同位置的比较
		        for(;;){
		            if(headA!=headB){
		                 if(headA->next!= NULL){
		                     headA=headA->next;
		                     headB=headB->next;
		                 }else{
		                     return NULL;
		                 }
		            } else{
		                return headA;
		            }
		        }
		
		
		    }
			//比较长度
		    int compareLength(ListNode *headA, ListNode *headB) {
		        ListNode *l=new ListNode(1);
		        l->next=headA->next;
		        for(;;){
		          if(l->next!= NULL){
		              l->next=l->next->next ;
		              ++l->val;
		          }else{
		              break;
		          }
		        }
		        l->next=headB->next;
		        --l->val;
		        for(;;){
		            if(l->next!= NULL){
		                l->next=l->next->next ;
		                --l->val;
		            }else{
		                break;
		            }
		        }
		        return l->val;
		    }
		};
		
		
		int main() {
		    Solution s;
		    ListNode *l1=new ListNode(1);
		    ListNode *l2=new ListNode(1);
		    ListNode *l3=new ListNode(1);
		    ListNode *l4=new ListNode(1);
		    ListNode *l5=new ListNode(1);
		    ListNode *l6=new ListNode(1);
		    ListNode *l7=new ListNode(1);
		    ListNode *l8=new ListNode(1);
		    ListNode *l9=new ListNode(1);
		    ListNode *l10=new ListNode(1);
		
		    l1->next=l2;
		    l2->next=l3;
		    l3->next=l4;
		    l4->next=l5;
		    l5->next=l6;
		    l6->next=l7;
		    l7->next=l8;
		    l8->next=l9;
		    l9->next=l10;
		    l10->next=NULL;
		
		
		
		    cout<<s.getIntersectionNode(l1,l6)<<endl;
		
		    return 0;
		
		}		