---
title: Min Stack
slug: min-stack
date: 2021-05-24T09:03:23.000Z
date_updated: 2021-05-24T09:03:22.000Z
tags: 
  - stack
  - algorithm
  - java
---

Design a stack that supports push, pop, top, and retrieving the minimum element in constant time.

Implement the `MinStack` class:

- `MinStack()` initializes the stack object.
- `void push(val)` pushes the element `val` onto the stack.
- `void pop()` removes the element on the top of the stack.
- `int top()` gets the top element of the stack.
- `int getMin()` retrieves the minimum element in the stack.

**Example 1:**

    Input
    ["MinStack","push","push","push","getMin","pop","top","getMin"]
    [[],[-2],[0],[-3],[],[],[],[]]
    
    Output
    [null,null,null,null,-3,null,0,-2]
    
    Explanation
    MinStack minStack = new MinStack();
    minStack.push(-2);
    minStack.push(0);
    minStack.push(-3);
    minStack.getMin(); // return -3
    minStack.pop();
    minStack.top();    // return 0
    minStack.getMin(); // return -2

**Solution:**
{% codeblock lang:java %}
class MinStack {
    private List<Integer> values = null;
    private List<Integer> mins = null;
    
    public MinStack() {
        this.values = new ArrayList<Integer>();
        this.mins = new ArrayList<Integer>();
    }
    
    private void setMin(int val) {
        if(this.mins.isEmpty() || val <= this.getMin()) {
            this.mins.add(val);
        } 
    }
    
    public void push(int val) {
        this.values.add(val);
        setMin(val);
    }
    
    public void pop() {
        if(this.values.remove(this.values.size() - 1) == this.getMin()) {
            this.mins.remove(this.mins.size() - 1);
        }
    }
    
    public int top() {
        return this.values.get(this.values.size() - 1);
    }
    
    public int getMin() {
        return this.mins.get(this.mins.size() - 1);
    }
}
{% endcodeblock %}

