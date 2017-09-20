算法描述：
一个栈stack，具有push和pop操作，其时间复杂度皆为O(1)。
设计算法max操作，求栈中的最大值，该操作的时间复杂度也要求为O(1)。
可以修改栈的存储方式，push，pop的操作，但是要保证O(1)的时间复杂度，空间时间复杂度无要求。
``` javascript
import java.util.ArrayList;
import java.util.EmptyStackException;
import java.lang.NullPointerException;

public class StackForPriorImpl<E extends Comparable<E>> implements 
IStackForPrior<E> {

private int numberOfObject;
private ArrayList<E> dataForOrginal;
private ArrayList<E> dataForMaximun;
private ArrayList<E> dataForMinimun;
private E maxElement;
private E minElement;

public StackForPriorImpl(){
this.numberOfObject = 0;
this.dataForOrginal = new ArrayList<E>();
this.dataForMaximun = new ArrayList<E>();
this.dataForMinimun = new ArrayList<E>();
}

@Override
public void push(E e) {

if(e==null){
throw new NullPointerException();
}

dataForOrginal.add(e);

if(numberOfObject==0){
maxElement = e;
minElement = e;
this.dataForMaximun.add(maxElement);
this.dataForMinimun.add(minElement);
}else{

if(e.compareTo(maxElement)>0){
dataForMaximun.add(e);
maxElement = e;
}else{
dataForMaximun.add(maxElement);
}

if(e.compareTo(minElement)<0){
dataForMinimun.add(e);
minElement = e;
}else{
dataForMinimun.add(minElement);
}
}

++numberOfObject;

}

@Override
public E pop() {

if(numberOfObject==0) {
throw new EmptyStackException();
}

int index = --numberOfObject;

dataForMaximun.remove(index);
maxElement = dataForMaximun.get(index-1);

dataForMinimun.remove(index);
minElement = dataForMinimun.get(index-1);

E e = dataForOrginal.get(index);
dataForOrginal.remove(index);
return e;
}

@Override
public E peekMedian() {

if(numberOfObject==0){
throw new EmptyStackException();
}

Object[] elementArray=this.dataForOrginal.toArray();
int lengthOfElementArray = elementArray.length;
quickSort(elementArray,0,lengthOfElementArray-1);

return (E)elementArray[lengthOfElementArray/2];
}

@Override
public E peekMaximum() {

if(numberOfObject==0){
throw new EmptyStackException();
}

int index = numberOfObject - 1;
return this.dataForMaximun.get(index);
}

@Override
public E peekMinimum() {

if(numberOfObject==0) {
throw new EmptyStackException();
}

int index = numberOfObject - 1;
return this.dataForMinimun.get(index);
}

@Override
public int size() {

return this.numberOfObject;
}

private static <E> void quickSort(E[] elementArray,int low,int high){
if(low<high){
int q = partition(elementArray,low,high);
quickSort(elementArray,low,q-1);
quickSort(elementArray,q+1,high);
}
} 

private static <E> int partition(E[] elementArray,int low,int high){
int i = low, j=high+1;
E e= elementArray[low];
while(true){
while(elementArray[++i].toString().compareTo(e.toString())<0 && i<high);
while(elementArray[--j].toString().compareTo(e.toString())>0);
if(i>=j) break;

E temp = elementArray[i];
elementArray[i]=elementArray[j];
elementArray[j]=temp;
}

elementArray[low] = elementArray[j];
elementArray[j]=e;

return j;
}

public static void main(String[] args){
StackForPriorImpl<Integer> stack=new StackForPriorImpl<Integer>();

stack.push(new Integer(2));
stack.push(new Integer(1));
stack.push(new Integer(2));
stack.push(new Integer(2));
stack.push(new Integer(6));
stack.push(new Integer(4));
stack.push(new Integer(2));
stack.push(new Integer(5));

System.out.println("size = "+stack.size());
System.out.println("peekMaximun = "+stack.peekMaximum().toString());
System.out.println("peekMinimun = "+stack.peekMinimum().toString());
System.out.println("peekMedian = "+stack.peekMedian().toString());

}
}
```

**主要的思路：还是使用数组作为栈的存储，不过要另外添加一个链表的数据结构用来存储最大值序列。这个最大值序列的真正含义是基于栈的以下特点，一个栈在任何一个状态下，里面的数据都是顺序出入的。最大值序列就是从栈尾到栈顶的最大递增序列，序列中的任何两个元素之间的元素永远都不会被当作最大值求出，因此这些元素不需要关注了，这是能够找到O(1)max算法的关键所在。**
```java
package sunfa;

import java.util.Arrays;
import java.util.Comparator;
import java.util.LinkedList;
import java.util.Random;

public class LinkedListMaxVal<E> {
	public static void main(String[] args) {
		LinkedListMaxVal<Integer> stack = new LinkedListMaxVal<Integer>(
				new Comparator<Integer>() {
					public int compare(Integer o1, Integer o2) {
						return o1 - o2;
					}
				});
		Random ran = new Random();
		for (int i = 1; i <= 20; i++) {
			stack.push(ran.nextInt(100));
			System.out.println(stack + "," + stack.max()+","+stack.maxStack);
			if (i % 4 == 0) {
				Integer ee = stack.peek();
				System.out.println("=============>pop:"+ee+"," + stack + "," + stack.max()+","+stack.maxStack);
				stack.pop();
			}
		}
		System.out.println("pop all>>>>>>>>>>>>>------------------");
		System.out.println("删除的元素  | 数据栈   |  最大值  |  最大值栈");
		while(!stack.isEmpty()){
			Integer ee = stack.peek();
			System.out.println("pop:" +ee+","+ stack + "," + stack.max()+","+stack.maxStack);
			stack.pop();
		}
		System.out.println("---end-----");
		System.out.println(stack);
	}

	private Object[] item;
	private int count;//栈内元素数目    ，栈顶元素索引为count-1
	private LinkedList<E> maxStack = new LinkedList<E>();//最大值备用栈,为省事借用链表
	private final String EMPTY_STACK_STR = "空栈";
	private Comparator<E> comp;

	public LinkedListMaxVal(Comparator<E> c) {
		item = new Object[10];
		comp = c;
	}

	public void push(E e) {
		if (e == null)
			throw new NullPointerException();
		if (count + 1 == item.length)
			item = Arrays.copyOf(item, item.length << 1);
		item[count++] = e;

		if (count == 1)
			maxStack.push(e);
		else {
			if (compare(maxStack.peekFirst(), e) <= 0)//=号不要丢了，这很关键
				maxStack.push(e);
		}
	}

	public E pop() {
		if (isEmpty())
			throw new NullPointerException(EMPTY_STACK_STR);
		E e = peek();
		item[count - 1] = null;

		if (count == 1) 
			maxStack.pop();
		 else {
			if (compare(maxStack.peekFirst(), e) == 0)
				maxStack.pop();
		}

		count--;
		return e;
	}

	public E max() {
		if (maxStack.isEmpty())
			throw new NullPointerException(EMPTY_STACK_STR);
		return maxStack.peekFirst();
	}

	public E peek() {
		if (isEmpty())
			throw new NullPointerException(EMPTY_STACK_STR);
		return (E) item[count - 1];
	}

	private int compare(E e1, E e2) {
		return comp != null ? (((comp).compare(e1, e2)))
				: (((Comparable<E>) e1).compareTo(e2));
	}

	public boolean isEmpty() {
		return count == 0;
	}

	public String toString() {
		String s = "[";
		for (int i = 0; i < item.length; i++) {
			if (item[i] != null)
				s += item[i] + ",";
		}
		if (s.length() == 1) {
			s += "]";
		} else {
			s = s.substring(0, s.length() - 1) + "]";
		}
		return s;
	}
}
```


> The idea would be to keep track of the max through using pairs in the stack. If you insert something into the stack, you update the max accordingly.
```java
class Stack {
private: 
    stack<pair<int,int>> s;

public:
    bool empty() const {
        return s.empty();
    }

    int max() const {
        assert (empty() == false);
        return s.top().second;
    }

    int pop() {
        int ans = s.top().first;
        s.pop();
        return ans;
    }

    void push(int x) {
        if (s.empty() || x > s.top().second)
        {
            s.emplace(x, x);
        }
        else
        {
            s.emplace(x, s.top().second);
        }
    }
};
```

```
I am interested in creating a Java data structure similar to a stack that supports the following operations as efficiently as possible:

Push, which adds a new element atop the stack,
Pop, which removes the top element of the stack,
Find-Max, which returns (but does not remove) the largest element of the stack, and
Find-Min, which returns (but does not remove) the smallest element of the stack, and
What would be the fastest implementation of this data structure? How might I go about writing it in Java?

This is a classic data structures question. The intuition behind the problem is as follows - the only way that the maximum and minimum can change is if you push a new value onto the stack or pop a new value off of the stack. Given this, suppose that at each level in the stack you keep track of the maximum and minimum values at or below that point in the stack. Then, when you push a new element onto the stack, you can easily (in O(1) time) compute the maximum and minimum value anywhere in the stack by comparing the new element you just pushed to the current maximum and minimum. Similarly, when you pop off an element, you will expose the element in the stack one step below the top, which already has the maximum and minimum values in the rest of the stack stored alongside it.

Visually, suppose that we have a stack and add the values 2, 7, 1, 8, 3, and 9, in that order. We start by pushing 2, and so we push 2 onto our stack. Since 2 is now the largest and smallest value in the stack as well, we record this:

 2  (max 2, min 2)
Now, let's push 7. Since 7 is greater than 2 (the current max), we end up with this:

 7  (max 7, min 2)
 2  (max 2, min 2)
Notice that right now we can read off the max and min of the stack by looking at the top of the stack and seeing that 7 is the max and 2 is the min. If we now push 1, we get

 1  (max 7, min 1)
 7  (max 7, min 2)
 2  (max 2, min 2)
Here, we know that 1 is the minimum, since we can compare 1 to the cached min value stored atop the stack (2). As an exercise, make sure you understand why after adding 8, 3, and 9, we get this:

 9  (max 9, min 1)
 3  (max 8, min 1)
 8  (max 8, min 1)
 1  (max 7, min 1)
 7  (max 7, min 2)
 2  (max 2, min 2)
Now, if we want to query the max and min, we can do so in O(1) by just reading off the stored max and min atop the stack (9 and 1, respectively).

Now, suppose that we pop off the top element. This yields 9 and modifies the stack to be

 3  (max 8, min 1)
 8  (max 8, min 1)
 1  (max 7, min 1)
 7  (max 7, min 2)
 2  (max 2, min 2)
And now notice that the max of these elements is 8, exactly the correct answer! If we then pushed 0, we'd get this:

 0  (max 8, min 0)
 3  (max 8, min 1)
 8  (max 8, min 1)
 1  (max 7, min 1)
 7  (max 7, min 2)
 2  (max 2, min 2)
And, as you can see, the max and min are computed correctly.

Overall, this leads to an implementation of the stack that has O(1) push, pop, find-max, and find-min, which is as asymptotically as good as it gets. I'll leave the implementation as an exercise. :-) However, you may want to consider implementing the stack using one of the standard stack implementation techniques, such as using a dynamic array or linked list of objects, each of which holds the stack element, min, and max. You could do this easily by leveraging off of ArrayList or LinkedList. Alternatively, you could use the provided Java Stack class, though IIRC it has some overhead due to synchronization that might be unnecessary for this application.

Interestingly, once you've built a stack with these properties, you can use it as a building block to construct a queue with the same properties and time guarantees. You can also use it in a more complex construction to build a double-ended queue with these properties as well.

Hope this helps!

EDIT: If you're curious, I have C++ implementations of a min-stack and a the aforementioned min-queue on my personal site. Hopefully this shows off what this might look like in practice!
```
```c++
/******************************************************************************
 * File: MinStack.hh
 * Author: Keith Schwarz (htiek@cs.stanford.edu)
 *
 * A LIFO stack class that supports O(1) push, pop, and find-min.  Here, the
 * find-min operation returns (but does not remove) the minimum value in the
 * stack.  This sort of stack is often used as a subroutine in other problems.
 * It can be used to construct a queue with equivalent properties by
 * using one of the many stack-to-queue constructions, for example.
 *
 * The implementation of the min-stack is actually quite simple.  The idea is
 * that since the stack can only grow and shrink at the top, we only need to
 * consider two ways that the minimum element of the stack can change:
 *
 *  1. The minimum element is on the top, and it is popped off, and
 *  2. A new element is added to the stack that is smaller than the minimum.
 *
 * Because of this, we can augment a standard stack to track the minimum value
 * as follows.  Whenever we push an element atop the stack, we compare it to
 * the current minimum value.  If it is smaller, we augment the value we just
 * added into the stack by recording that it is the minimum.  If not, we store
 * a pointer down to the element of the stack that actually is the minimum.  In
 * this way, we can find the minimum element in constant time by simply
 * inspecting the top element of the stack and following its pointer to the
 * minimum element.
 */
#ifndef MinStack_Included
#define MinStack_Included

#include <deque>
#include <functional> // For std::less
#include <utility>    // For std::pair

/**
 * Class: MinStack<T, Comparator = std::less<T>>
 * Usage: MinStack<int> myMinStack;
 * ----------------------------------------------------------------------------
 * A class representing a LIFO stack supporting constant-time push, pop, and
 * find-min.  The comparator may be customized.
 */
template <typename T, 
          typename Comparator = std::less<T> > 
class MinStack {
public:
  /**
   * Constructor: MinStack(Comparator = Comparator());
   * Usage: MinStack<T> myStack;
   *        MinStack<T, C> myStack(myComparator);
   * --------------------------------------------------------------------------
   * Constructs a new MinStack that uses the specified comparator to make
   * comparisons.
   */
  explicit MinStack(Comparator = Comparator());

  /**
   * void push(const T& val);
   * Usage: myStack.push(137);
   * --------------------------------------------------------------------------
   * Pushes a new element atop the stack.
   */
  void push(const T& val);

  /**
   * void pop();
   * Usage: myStack.pop();
   * --------------------------------------------------------------------------
   * Pops the top element off the stack.  If the stack is empty, the behavior
   * is undefined.
   */
  void pop();

  /**
   * const T& top() const;
   * Usage: cout << myStack.top() << endl;
   * --------------------------------------------------------------------------
   * Returns an immutable view of the top element of the stack.  If the stack
   * is empty, the behavior is undefined.
   */
  const T& top() const;

  /**
   * const T& min() const;
   * Usage: cout << myStack.min() << endl;
   * --------------------------------------------------------------------------
   * Returns an immutable view of the minimum element of the stack.  If the
   * stack is empty, the behavior is undefined.  If multiple elements in the
   * stack are tied for the minimum element, returns a reference to the lowest
   * (eldest) of them.
   */
  const T& min() const;

  /**
   * bool   empty() const;
   * size_t size()  const;
   * Usage: while (!myStack.empty()) { ... }
   *        if (myStack.size() == 3) { ... }
   * --------------------------------------------------------------------------
   * Returns whether the stack is empty and its size, respectively.
   */
  bool   empty() const;
  size_t size() const;

private:
  /* The actual stack.  Each entry is a pair of an element and the index of the
   * minimum element at or below this point.
   */
  std::deque< std::pair<T, size_t> > mStack;

  /* The comparator used to determine what the smallest element is. */
  Comparator mComp;
};

/* * * * * Implementation Below This Point * * * * */

/* Constructor stores the comparator for future use. */
template <typename T, typename Comparator>
MinStack<T, Comparator>::MinStack(Comparator c) 
  : mStack(), mComp(c) {
  // Handled in initialization list
}

/* Size and empty queries forward directly to the underlying deque. */
template <typename T, typename Comparator>
size_t MinStack<T, Comparator>::size() const {
  return mStack.size();
}
template <typename T, typename Comparator>
bool MinStack<T, Comparator>::empty() const {
  return mStack.empty();
}

/* Returning the top element looks at the back of the deque. */
template <typename T, typename Comparator>
const T& MinStack<T, Comparator>::top() const {
  return mStack.back().first;
}

/* Returning the min element looks at the element in the deque that is the
 * smallest so far.  It's held at the index at the top of the stack. */
template <typename T, typename Comparator>
const T& MinStack<T, Comparator>::min() const {
  return mStack[mStack.back().second].first;
}

/* Inserting a new element adds it to the stack and updates the minimum element
 * if the new element is smaller.
 */
template <typename T, typename Comparator>
void MinStack<T, Comparator>::push(const T& elem) {
  /* If the stack is empty, add the new element and mark that it's the smallest
   * so far.
   */
  if (empty()) {
    mStack.push_back(std::make_pair(elem, 0));
  }
  else {
    /* Otherwise, find the index of the smallest element and insert the new
     * element annotated with that index.
     */
    size_t smallestIndex = mStack.back().second;

    /* If this new element is smaller, the smallest element will now be at the
     * back of the list.
     */
    if (mComp(elem, min()))
      smallestIndex = mStack.size();

    /* Add the element in. */
    mStack.push_back(std::make_pair(elem, smallestIndex));
  }
}

/* Popping an element off the stack just removes the top pair from the
 * deque.
 */
template <typename T, typename Comparator>
void MinStack<T, Comparator>::pop() {
  mStack.pop_back();
}

#endif
```
```java
import java.util.Stack;

public class DS {
    static Stack<Integer> stack;
    static Stack<Integer> minStack;
    static Stack<Integer> maxStack;

    public DS(){
        stack = new Stack<Integer>();
        minStack = new Stack<Integer>();
        maxStack = new Stack<Integer>();
    }

    // Push Method
    public void push(int k){        
        stack.push(k);
        if(!minStack.isEmpty()){
            minStack.push(Math.min(k, minStack.peek()));
        }else{
            minStack.push(k);
        }
        if(!maxStack.isEmpty()){
            maxStack.push(Math.max(k, maxStack.peek()));
        }else{
            maxStack.push(k);
        }
    }

    // Pop Method
    public void pop(){
        if(!stack.isEmpty() && !minStack.isEmpty() && !maxStack.isEmpty()){
            stack.pop();
            minStack.pop();
            maxStack.pop();
        }
    }

    // Find Min 
    public int findMin(){
        if(!minStack.isEmpty()){
            return minStack.peek();
        }
        return Integer.MIN_VALUE;
    }

    // Find Max
    public int findMax(){
        if(!maxStack.isEmpty()){
            return maxStack.peek();
        }
        return Integer.MAX_VALUE;
    }

    public static void main(String[] args) {

        DS ds = new DS();

        System.out.println("Push 7, 6, 5: ");
        ds.push(7);
        ds.push(6);
        ds.push(5);

        System.out.println("S1: " + stack);
        System.out.println("S2: " + minStack);
        System.out.println("S3: " + maxStack);

        System.out.println("Min till now: " + ds.findMin());
        System.out.println("Max till now: " + ds.findMax());

        System.out.println("Push 4, 3: ");
        ds.push(4);
        ds.push(3);

        System.out.println(stack);
        System.out.println(minStack);
        System.out.println(maxStack);

        System.out.println("Min till now: " + ds.findMin());
        System.out.println("Max till now: " + ds.findMax());

        System.out.println("1 pop(): ");
        ds.pop();
        System.out.println("Min till now: " + ds.findMin());
        System.out.println("Max till now: " + ds.findMax());


        System.out.println("1 pop(): ");
        ds.pop();
        System.out.println("Min till now: " + ds.findMin());
        System.out.println("Max till now: " + ds.findMax());
    }
}
```

```java
import java.util.Stack;

public class MyDS {

    Stack<Integer> s;
    Stack<Integer> minStack;
    Stack<Integer> maxStack;

    public MyDS(){
        s = new Stack<Integer>();
        minStack = new Stack<Integer>();
        maxStack = new Stack<Integer>();
    }

    // Push Method
    public void push(int k){

        if(minStack.isEmpty()){
            minStack.push(k);
        }else if(k <= minStack.peek()){
            minStack.push(k);
        }

        if(maxStack.isEmpty()){
            maxStack.push(k);
        }else if(k >= maxStack.peek()){
            maxStack.push(k);
        }   
        s.push(k);  
    }

    // Pop Method
    public void pop(){

        int popped;
        if(!s.isEmpty()){
            popped = s.pop();   
        }else{
            popped = -1;
        }

        if(popped == min()){
            minStack.pop();
        }

        if(popped == max()){
            maxStack.pop();
        }
    }

    // Min Method
    public int min(){
        if(!minStack.isEmpty()){
            return minStack.peek();
        }else{
            return Integer.MIN_VALUE;
        }
    }

    // Max Method
    public int max(){
        if(!maxStack.isEmpty()){
            return maxStack.peek();
        }else{
            return Integer.MAX_VALUE;
        }
    }
}
```

```
6
down vote
accepted
Your MyDS class has the right idea, in general.

Special values like -1, Integer.MIN_VALUE, and Integer.MAX_VALUE make me suspicious. All of those special values denote what I consider to be error cases. Using special cases that might also be valid data is a dangerous habit that can lead to bugs. Instead of those special numbers, it would be better to throw exceptions — probably NoSuchElementException. You should also offer a size() and/or an isEmpty() method so that users of your data structure can proactively avoid encountering the exception.

The three instance variables should be private. The default access is rarely appropriate. java.util.Stack is to be avoided, due to unfortunate historical design decisions (inappropriately extending java.util.Vector, and being thread-safe by default). The documentation recommends ArrayDeque instead.

Of the four operations in MyDS, I think pop() could use the most work. It's weird that pop() doesn't return a value. The -1 is entirely avoidable: if the main stack is empty, the min and max stacks should surely be empty too.

public int pop() {
    if (s.isEmpty()) {
        throw new NoSuchElementException();
    }
    int popped = s.pop();
    if (popped == min()) {
        minStack.pop();
    }
    if (popped == max()) {
        maxStack.pop();
    }
    return popped;
}
```
```
Excellent comments so far, so this is more about the basic design. I think the source of complexity in this class, even though it's not terribly complex, is the concurrent maintenance of three stacks. It would in my opinion increase clarity to use a single stack instead of three. This requires introducing a class to represent the cumulative state.

I also recommend that you make member fields final unless you need to modify them.
import java.util.ArrayDeque;
public class MinMaxStack {

    private final ArrayDeque<MinMaxState> stack = new ArrayDeque<>();

    private final static class MinMaxState {

        final int min, max, value;

        MinMaxState(int newValue, MinMaxState previous) {
            value = newValue;
            if (previous == null) {
                min = max = newValue;
            } else {
                min = Math.min(newValue, previous.min);
                max = Math.max(newValue, previous.max);
            }
        }
    }

    public int min() {
        requireNonEmpty();
        return stack.peek().min;
    }

    public int max() {
        requireNonEmpty();
        return stack.peek().max;
    }

    public void push(int value) {
        stack.push(new MinMaxState(value, stack.peek()));
    }

    public int pop() {
        requireNonEmpty();
        return stack.pop().value;
    }

    private void requireNonEmpty() {
        if (stack.isEmpty())
            throw new NoSuchElementException("The stack is empty.");
    }
}
```

http://www.geeksforgeeks.org/design-and-implement-special-stack-data-structure/
http://www.cnblogs.com/evasean/archive/2017/07/24/7230112.html
http://www.geeksforgeeks.org/design-and-implement-special-stack-data-structure/
https://stackoverflow.com/questions/3435926/insert-delete-max-in-o1
https://stackoverflow.com/questions/7134129/stack-with-find-min-find-max-more-efficient-than-on
https://stackoverflow.com/questions/685060/design-a-stack-such-that-getminimum-should-be-o1
http://algorithms.tutorialhorizon.com/track-the-maximum-element-in-a-stack/
https://www.cs.bu.edu/teaching/c/stack/array/
https://codereview.stackexchange.com/questions/105424/java-max-stack-implementation-to-return-maximum-value-in-the-stack
http://blog.csdn.net/u014110320/article/details/77677629
http://blog.csdn.net/sinat_19425927/article/details/38518887
http://www.cnblogs.com/xuanxu/archive/2013/08/06/3240416.html
