---
layout: post
title: "LinkedList源码详解"
subtitle: 'LinkedList源码详解'
author: "山大王"
header-style: text
catalog: true
tags:
  - 源码
  - android
---
> “红粉佳人休使老，风流浪子莫教贫。”

## 正文

源码：\sources\android-25

LinkedList和ArrayList都是List，不同的是ArrayList是以数组的形式来存储的，而LinkedList是以链表的形式存储的，ArrayList很简单，没什么可说的，下面就简单看一下LinkedList，他里面有个节点Node，是双向的链表

## Node

```java
    private static class Node<E> {
        E item;
        Node<E> next;
        Node<E> prev;

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
```

并且LinkedList中有两个指针，一个指向头first，一个指向尾last，

```java
    /**
     * Pointer to first node.
     * Invariant: (first == null && last == null) ||
     *            (first.prev == null && first.item != null)
     */
    transient Node<E> first;

    /**
     * Pointer to last node.
     * Invariant: (first == null && last == null) ||
     *            (last.next == null && last.item != null)
     */
    transient Node<E> last;
```

先来看第一个方法linkFirst

## linkFirst

```java
    /**
     * Links e as first element.
     */
    private void linkFirst(E e) {//在第一个节点前面添加一个节点
        final Node<E> f = first;//first节点指向头，把头结点保存在f中
		//创建新节点，新节点的前一个为null，因为新建的节点是要添加链表的头，所以他的前一个是为null，
		//新节点的下一个是f，这是因为新节点是要添加到f的前面的。
        final Node<E> newNode = new Node<>(null, e, f);
        first = newNode;//让新节点等于头结点
		//如果原来头结点为空，说明原来的链表为null，在创建第一个节点的时候first和last要指向同一个节点。
        if (f == null)
            last = newNode;
        else//如果原来链表不是null，让新节点等于原来头结点的前一个，
            f.prev = newNode;
        size++;//链表数量加1
        modCount++;//这个加是对链表操作的时候加，在Iterator遍历的时候是禁止操作的，否则要抛异常
    }
```

既然有添加到第一个，肯定会有添加到最后一个

## linkLast

```java
    /**
     * Links e as last element.
     */
    void linkLast(E e) {
		//添加到最后 
        final Node<E> l = last;
		//创建新节点，新节点的前一个是last，后一个为null，因为新节点就是最后一个，所以后一个要为null
        final Node<E> newNode = new Node<>(l, e, null);
        last = newNode;//让新建节点成为最后一个
        if (l == null)//如果之前的链表是null的，则first和last都要指向同一个节点
            first = newNode;
        else
            l.next = newNode;//让当前节点成为之前链表last的下一个
        size++;//加1
        modCount++;
    }
```

再看下一个方法linkBefore(E e, Node<E> succ)

## linkBefore

```java
    /**
     * Inserts element e before non-null Node succ.
     */
    void linkBefore(E e, Node<E> succ) {
		//他表示插入一个新节点e到succ节点的前面
        // assert succ != null;
		//首先succ节点要存在，否则要抛空指针异常，然后获取他的前一个节点
        final Node<E> pred = succ.prev;
		//创建新节点，新节点是前一个是pred，也就是succ的前一个，后一个是succ
        final Node<E> newNode = new Node<>(pred, e, succ);
        succ.prev = newNode;//新节点等于succ的前一个
		//pred等于null，说明succ是first节点，既然添加到succ的前面，所以就让新节点成为first节点
        if (pred == null)
            first = newNode;
        else
            pred.next = newNode;//让pred的下一个节点指向新节点
        size++;//加1
        modCount++;
    }
```

下一个方法是删除first节点

## unlinkFirst

```java
    /**
     * Unlinks non-null first node f.
     */
    private E unlinkFirst(Node<E> f) {
		//删除first节点，f就是first节点
        // assert f == first && f != null;
        final E element = f.item;
        final Node<E> next = f.next;
        f.item = null;
        f.next = null; // help GC
        first = next;//让f的下一个节点成为first节点，
		//如果next为null，说明之前就一个first节点，删除之后就没有节点，所以只好让last节点指向null
        if (next == null)
            last = null;
        else
            next.prev = null;
        size--;
        modCount++;
        return element;
    }
```

然后下一个方法unlinkLast也都很简单，基本上没什么可说的，看下面一个方法unlink(Node<E> x)

## unlink

```java
    /**
     * Unlinks non-null node x.
     */
    E unlink(Node<E> x) {
        // assert x != null;
		//其实删除x节点很简单，就是让x的前一个后后一个连接就行了，不过还要考虑前一个和后一个是否为空的问题
        final E element = x.item;
        final Node<E> next = x.next;//x的下一个节点
        final Node<E> prev = x.prev;// x的前一个节点

        if (prev == null) {
		//前一个节点为空，说明x是first节点，所以要让next等于first
            first = next;
        } else {//连接
            prev.next = next;
            x.prev = null;
        }

        if (next == null) {//如果next为空，说明x是last节点，
            last = prev;
        } else {//连接
            next.prev = prev;
            x.next = null;
        }

        x.item = null;
        size--;
        modCount++;
        return element;
    }
```

下面再看一个方法remove(Object o) 

## remove

```java
    public boolean remove(Object o) {
        if (o == null) {//o为null的情况
            for (Node<E> x = first; x != null; x = x.next) {//对链表遍历
                if (x.item == null) {//找到之后删除
                    unlink(x);
                    return true;
                }
            }
        } else {//o不为空，和上面的差不多
            for (Node<E> x = first; x != null; x = x.next) {
                if (o.equals(x.item)) {
                    unlink(x);
                    return true;
                }
            }
        }
        return false;
    }
```

下面再看另一个方法node(int index)，根据index查找node

## node

```java
    /**
     * Returns the (non-null) Node at the specified element index.
     */
    Node<E> node(int index) {
        // assert isElementIndex(index);
		//根据index查找节点，如果index在前半部分从前找，如果index在后半部分，从后面查找
        if (index < (size >> 1)) {
            Node<E> x = first;
            for (int i = 0; i < index; i++)
                x = x.next;
            return x;
        } else {
            Node<E> x = last;
            for (int i = size - 1; i > index; i--)
                x = x.prev;
            return x;
        }
    }
```

其他的方法也都很简单，基本上没啥可说的，下面看最后一个方法addAll(int index, Collection<? extends E> c)

## addAll

```java
    /**
     * Inserts all of the elements in the specified collection into this
     * list, starting at the specified position.  Shifts the element
     * currently at that position (if any) and any subsequent elements to
     * the right (increases their indices).  The new elements will appear
     * in the list in the order that they are returned by the
     * specified collection's iterator.
     *
     * @param index index at which to insert the first element
     *              from the specified collection
     * @param c collection containing elements to be added to this list
     * @return {@code true} if this list changed as a result of the call
     * @throws IndexOutOfBoundsException {@inheritDoc}
     * @throws NullPointerException if the specified collection is null
     */
	 //把集合c添加到链表中，从index开始添加
    public boolean addAll(int index, Collection<? extends E> c) {
        checkPositionIndex(index);//检查index，如果越界会抛越界异常

        Object[] a = c.toArray();
        int numNew = a.length;
        if (numNew == 0)//如果c为空，则返回
            return false;
		//succ表示index位置的节点，pred表示succ的前一个节点
        Node<E> pred, succ;
        if (index == size) {//添加到最后
            succ = null;//index位置的节点为null，也就是succ为null
            pred = last;//last是前一个节点，因为要添加造last节点的后面
        } else {
            succ = node(index);//index位置的节点
            pred = succ.prev;//添加到pred和succ之间，所以要保存succ的前一个节点
        }

        for (Object o : a) {
            @SuppressWarnings("unchecked") E e = (E) o;
			//创建节点，前一个是pred，后一个是null，后一个在下面在赋值
            Node<E> newNode = new Node<>(pred, e, null);
			//前一个为null，说明原来链表是null的，让当前节点赋值first
            if (pred == null)
                first = newNode;
            else
			//先连接前面的，后面的先不连，newNode的前一个节点在创建的时候就已经赋值，而他前一个的下一个节点在这地方赋值,
                pred.next = newNode;
            pred = newNode;//让当前节点成为前一个，然后不停的往后添加
        }
		//到目前为止，前面的节点都已经连接完了，后面的都还没连接
        if (succ == null) {
		//这个简单，如果添加到原节点的最后，直接让pred等于last就行的，因为pred是最后一个添加的节点
            last = pred;
        } else {
		//如果不是添加到最后，需要最后添加的节点pred和succ连接起来即可。succ是pred的下一个节点，pred是succ的前一个节点
            pred.next = succ;
            succ.prev = pred;
        }

        size += numNew;//增加size
        modCount++;
        return true;
    }
```

