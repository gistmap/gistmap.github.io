---
title: 菜鸟之见HashMap
date: 2017-11-26 01:15:23
tags: [java,api]
categories: java
---



### 之前的认识
记得刚找工作实习的时候，只要去面试，99%会被问到关于HashMap的问题。比如，Collection下各个容器的特点。被问到了Map，我就像背书一样的说下了以下几点，估计面试官很无语，哈哈。

- 存取快
- 插入删除快
- 数据量大的时候，优点突出

后来工作了发现HashMap的使用场景太多了，无疑是那"80%"的常用技术。

### 我理解的HashMap
我觉得HashMap的结构特别像古代中医相关的电视剧中的`中医柜`，药放的位置和取的位置都是由一定的规则决定的。比方药店刚开张，只有50种药材，OK，那可以让木匠给我们制作个50个药柜小抽屉（其实为了长久考虑，应该多于50个）。每个药放进每一个抽屉并在抽屉脸上写着药名。

随着时代的进步，药材不断被发现。扩展到了100种。药柜放不下了，聪明的店主想出了把药名第一个字相同的放进一个抽屉里并用小隔层隔开。由此，即使来开药的人非常多，拿药人也可以啪啪啪瞬间拿好相应的药。店员纷纷感叹店长的智慧。

可以把计算机科学中的数组想象成每一个药柜，而后面扩容的小隔层想象成链表。这就组成了一个新的数据结构，成为哈希表。哈希表通过按照key计算到一个下标，计算的方法称为hash算法。理想的情况是f(pdd)=index，index唯一。但可能存在f(aab)=index的情况，这种现象称为「碰撞」。想象一下，碰撞次数越多，那么某一个数组下的链表越多，每一次要拿值的时候，我们需要循环+判断key。对神奇的hash算法感兴趣可以自行百度百科，维基百科解惑。

### 前置技能-链表
链表是学习的第一个数据结构，是构成很多数据结构的基础。我写了一个简易版本的链表(只放出部分核心方法)来捡起对链表的认知，为后续理解HashMap做准备。

```java
public class NodeList<T> {

	private Node<T> head;
	private int size;

	class Node<T>{
		T value;
		Node<T> next;

		Node(T value, Node<T> next) {
			this.value = value;
			this.next = next;
		}
	}
	private void add(T item) {
		if (head != null) {
			Node<T> temp = head;
			while ( temp.next != null) {
				temp = temp.next;
			}
			temp.next = new Node<>(item,null);

		} else {
			head = new Node<>(item,null);
		}
		size++;
	}
	private void add(T item, int index) {
		if ( index > size) {
			System.out.println("index error");
			return;
		}
		Node<T> node = head;
		while (--index > 0 ){
			node = node.next;
		}
		//拿到要插入节点的后一个节点
		Node<T> next = node.next;
		//改变前一个节点的指向
		node.next = new Node<>(item,next);
		size++;

	}
	public T get(int index) {
		Node<T> node = head;
		while (--index > 0) {
			node = node.next;

		}
		return node.value;
	}
	private void remove(T item) {
		if (head == null) {
			return ;
		}
		Node<T> node = head;
		while (node != null) {
			//头结点
			if (node.value == item && node == head) {
				head = node.next;
				size--;
				return;
			}
			Node<T> temp = node.next;
			//其他节点
			if ( temp.value == item ) {
				node.next = temp.next;
				size--;
				return;
			}
			node = node.next;
		}

	}
}
```

### 我自己脑洞的简易HashMap的伪代码
```java
class Rmap<T> {
	
	Node[] table ;
	initial size = 6;
	size = 0;
	//无参构造初始化长度的table
	//有参构造初始化固定长度的table

    void put(int key, T value) {
        //index = hash(key)
        //if index > size 扩容

        //if (table[index]==null)
        //  -->新建节点
		//else
        //  -->取出当前数组位置节点
        //  -->遍历找到key相同替换
        //  -->没找到key相同的则追加
	}
    T get(int key) {

        //index = hash(key)
        //取出当前数组位置节点
        //遍历里面的链表判断key输出value

	}
}
```

### 马马虎虎凑合的野路子HashMap

```java

public class Rmap<T> {
	class Node<K>{
		int key;
		K value;
		Node<K> next;

		Node(int key, K value, Node<K> next) {
			this.key = key;
			this.value = value;
			this.next = next;
		}
	}
	private Node[] table ;
	private static final int ORIGINAL_SIZE = 6;
	private int currentSize = 0;
	private int size = 0;

	public Rmap() {
		table = new Node[ORIGINAL_SIZE];
		size = ORIGINAL_SIZE;
	}
	public Rmap(int size) {
		if (size < 0) {
			throw new IllegalArgumentException("size not enough");
		}
		table = new Node[size];
		this.size = size;
	}
	private int getSize() {
		return currentSize;
	}
	private void put(int key, T value) {
		int index = hash(key);
		if (index > size) {
			// 增加容量
		}
		if(table[index]==null){
			table[index]=new Node(key,value,null);
			currentSize++;
		}
		//取出当前数组位置节点
		else {
			Node e = table[index];
			//遍历此Node链表，如果key相同，则替换后返回
			for (; e != null; e=e.next) {
				if (e.key == key) {
					e.value = value;
					return;
				}
			}
			//没有return后分支会走到这里，即追加
			Node news = table[index].next;
			//新建一个节点保存信息
			table[index].next = new Node<T>(key,value,news);
			currentSize++;
		}
	}
	private T get(int key) {
		int index = hash(key);
		Node<T> node = table[index];
		while (node != null) {
			if (node.key == key) {
				return node.value;
			}
			node = node.next;
		}
		return null;
	}
	private int hash(int key) {
		return key % 16;
	}
	public static void main(String[] args) {
		Rmap<String> map = new Rmap<>();
		map.put(1,"zr");
		map.put(2,"zrr");
		map.put(3,"zrrr");
		map.put(3,"zrrrr");
		System.out.println(map.get(1));
		System.out.println(map.get(2));
		System.out.println(map.get(3));
		System.out.println(map.getSize());
	}
}

```

我忧心忡忡的看着我写的代码，真的不会有问题吗。打死我都不信。下篇将会看看jdk中HashMap的细节。另外谁知道那个古代医生题材的电视剧请告诉我，好想再看一遍啊QAQ。我只记得女主很漂亮，有2种人格，白天和晚上不一样。
