---
title: LeetCode-862
author: Ray
top: true
cover: false
date: 2023-05-12 17:14:10
categories: 技术
mathjax: true
tags: 
  - 力扣之旅
  - java
  - 数据结构
---

### Q:

`RandomizedCollection` 是一种包含数字集合(可能是重复的)的数据结构。它应该支持插入和删除特定元素，以及删除随机元素。

实现 `RandomizedCollection` 类:

- `RandomizedCollection()`初始化空的 `RandomizedCollection` 对象。
- `bool insert(int val)` 将一个 `val` 项插入到集合中，即使该项已经存在。如果该项不存在，则返回 `true` ，否则返回 `false` 。
- `bool remove(int val)` 如果存在，从集合中移除一个 `val` 项。如果该项存在，则返回 `true` ，否则返回 `false` 。注意，如果 `val` 在集合中出现多次，我们只删除其中一个。
- `int getRandom()` 从当前的多个元素集合中返回一个随机元素。每个元素被返回的概率与集合中包含的相同值的数量 **线性相关** 。

您必须实现类的函数，使每个函数的 **平均** 时间复杂度为 `O(1)` 。

**注意：**生成测试用例时，只有在 `RandomizedCollection` 中 **至少有一项** 时，才会调用 `getRandom` 。

<!-- more -->

### Input:

```
输入
["RandomizedCollection", "insert", "insert", "insert", "getRandom", "remove", "getRandom"]
[[], [1], [1], [2], [], [1], []]
输出
[null, true, false, true, 2, true, 1]

解释
RandomizedCollection collection = new RandomizedCollection();// 初始化一个空的集合。
collection.insert(1);   // 返回 true，因为集合不包含 1。
                        // 将 1 插入到集合中。
collection.insert(1);   // 返回 false，因为集合包含 1。
                        // 将另一个 1 插入到集合中。集合现在包含 [1,1]。
collection.insert(2);   // 返回 true，因为集合不包含 2。
                        // 将 2 插入到集合中。集合现在包含 [1,1,2]。
collection.getRandom(); // getRandom 应当:
                        // 有 2/3 的概率返回 1,
                        // 1/3 的概率返回 2。
collection.remove(1);   // 返回 true，因为集合包含 1。
                        // 从集合中移除 1。集合现在包含 [1,2]。
collection.getRandom(); // getRandom 应该返回 1 或 2，两者的可能性相同。
```

### S:

因为题目插入时需要查找值是否存在，所以我们需要实现查找为O1，那么有list和hashmap（list通过索引）可以完成。

但是由于list无法直接O1查找元素值，所以可以考虑list和hashmap联合使用。

map存（值，索引），list存值。 

由于list只有在删除尾元素时能够实现O1，所以我们可以将待删除元素与队尾元素互换，然后删除 map中存的是值与索引键值对。

因为一个值可能存在多个索引，所以索引也需要用一个集合封装。

 考虑到一个值不会有2个相同索引，并且在删除交换等操作时需要对值得索引也进行删除等操作，所以使用set来保存索引序列

```java
class RandomizedCollection {
    int n ;//当前集合大小
    HashMap<Integer,Set<Integer>>map;
    ArrayList<Integer>list;
    Random random;
    /** Initialize your data structure here. */
    public RandomizedCollection() {
        this.random = new Random();
        this.map = new HashMap();
        this.n = 0;
        this.list = new ArrayList<>();
    }
    
    /** Inserts a value to the collection. Returns true if the collection did not already contain the specified element. */
    public boolean insert(int val) {
        Set set = map.get(val);
        if(set==null)   set = new HashSet<>();
        set.add(n);//添加索引
        list.add(val);
        map.put(val, set);
        n++;
        return set.size()==1;
    }
    
    /** Removes a value from the collection. Returns true if the collection contained the specified element. */
    public boolean remove(int val) {
        if(map.containsKey(val)){
            int lastIndex = n-1;//得到最后2个值索引
            Set lastset = map.get(list.get(lastIndex));
            Set set = map.get(val);
            int currIndex = (int)set.iterator().next();//得到当前值索引
            //进行删除操作
            swap(list, currIndex, lastIndex);
            list.remove(n-1);//将其在列表中删除
            set.remove(currIndex);//删除原值
            if(set.size()==0)   map.remove(val);//在图中删除
            //修改最后一个值的索引
            lastset.remove(n-1);
            lastset.add(currIndex);
            n--;
        }else{
            return false;
        }
        return true;
    }
    
    /** Get a random element from the collection. */
    public int getRandom() {
        return list.get(random.nextInt(n));
    }
    private void swap(List<Integer> list ,int i,int j){
        int temp = list.get(i);
        list.set(i, list.get(j));
        list.set(j, temp);
    }
}
```

