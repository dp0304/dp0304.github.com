---
layout: post
title: "leetcode: Min Stack"
description: ""
category: leetcode
tags: []
---
{% include JB/setup %}



题目：   

	Design a stack that supports push, pop, top, and retrieving the minimum element in constant time.

	push(x) -- Push element x onto stack.
	pop() -- Removes the element on top of the stack.
	top() -- Get the top element.
	getMin() -- Retrieve the minimum element in the stack.

[题目地址](https://oj.leetcode.com/problems/min-stack/)   
  
解题思路：  
  原本只是简单栈操作，但要取出栈中的最小值。比较一般的办法是另开一条数组，每次push和pop的时候进行排序。但是考虑到栈这一个特殊的格式，有优化的方法。  
  另外起一个栈。每次入栈前与top比较，便可以保证每次操作都是O(1)。



	class MinStack {
	public:	
	    void push(int x) {	
	         nodeList.push(x);	
	        if( minList.empty()||minList.top()>=x) { //等于也要加入栈中，因为pop的时候是通过相等就pop出
	            minList.push(x);
	        }
	    }
	
	    void pop() {
	
	        if( minList.top()==nodeList.top()) {
	            minList.pop();
	        }
	        nodeList.pop();
	    }
	
	    int top() {
	        if(nodeList.empty()){
	            return 0;
	        }
	        return nodeList.top();
	    }

	    int getMin() {
	        if(nodeList.empty()){
	            return 0;
	        }
	       return minList.top();
	    }
	
	private:
	   stack<int> nodeList;
	   stack<int> minList;
	};

	int main() {
	    MinStack ms;
	    ms.push(5);
	    ms.push(2);
	    ms.push(3);
	    ms.push(2);
	    ms.push(3);
	    ms.push(4);
		cout << ms.getMin() << endl;
		cout << ms.top() << endl;
	    ms.pop();
	    ms.pop();
	    ms.pop();
	    ms.pop();
	    ms.pop();
	    ms.pop();
		cout << ms.getMin() << endl;
		cout << ms.top() << endl;

    	return 0;

	}


原本是自己使用结构体写的。但是总会Memory Limit Exceeded出错。就改用stack了。

	struct Node {
	    int data;
	    Node *next;
	};
	
	struct MinNode {
	    int data;
	    MinNode *next;
	};
	
	class MinStack {
	public:
	    MinStack() {
	        firstNode = 0;
	        firstMinNode = 0;
	    }
	
	    ~MinStack() {
	    }
	
	    void push(int x) {
	        Node *newNode = new Node;
	        newNode->data = x;
	        newNode->next = firstNode;
	        firstNode = newNode;
	        if (firstMinNode == 0) {
	            MinNode *newMinNode =   new MinNode ;
	            newMinNode->data = x;
	            newMinNode->next = 0;
	            firstMinNode = newMinNode;
	        } else {
	            if (firstMinNode->data >= x) {
	                MinNode *newMinNode =new MinNode;
	                newMinNode->data = x;
	                newMinNode->next = firstMinNode;
	                firstMinNode = newMinNode;
	            }
	        }
	
	
	    }
	
	    void pop() {
	        if (firstNode != 0) {
	            if (firstNode->data == firstMinNode->data) {
	                MinNode  *minNode=firstMinNode->next;
	                delete firstMinNode;
	
	                firstMinNode = minNode;
	            }
	            Node *nextNode=firstNode->next;
	
	            delete firstNode;
	
	            cout<<firstNode->data<<endl;
	            firstNode = nextNode;
	        }
	    }
	
	    int top() {
	        if (firstNode != 0) {
	            return firstNode->data;
	        } else {
	            abort();
	        }
	    }
	
	    int getMin() {
	        if (firstNode != 0) {
	            return firstMinNode->data;
	        }else{
	            abort();
	        }
	    }
	
	private:
	    Node *firstNode;
	    MinNode *firstMinNode;
	};
	
	
	int main() {
	    MinStack ms;
	    ms.push(5);
	    ms.push(2);
	    ms.push(3);
	    ms.push(2);
	    ms.push(3);
	    ms.push(4);
	   cout << ms.getMin() << endl;
	    cout << ms.top() << endl;
	    ms.pop();
	    ms.pop();
	    ms.pop();
	    ms.pop();
	    ms.pop();
	    ms.pop();
	    cout << ms.getMin() << endl;
	    cout << ms.top() << endl;
	    return 0;
	
	}	