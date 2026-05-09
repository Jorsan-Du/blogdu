---
title: LeetCode 刷题日记：哈希表的奇妙运用
date: 2026-05-09 19:00:00
categories:
  - 算法笔记
tags:
  - LeetCode
  - 哈希表
  - Python
excerpt: 通过多个经典题目，深入解析哈希表在算法中的奇妙运用，包括两数之和、字母异位词分组、最长连续序列等场景。
---

# 📘 LeetCode 刷题日记：哈希表的奇妙运用

## 🧠 课前科普：到底什么是哈希表（Hash Table）？

在做题之前，我们需要弄明白哈希表这种数据结构的本质。

### 1. 原理简述
**哈希表**（Hash Table，也叫散列表），是一种根据**键（Key）**直接访问内存存储位置的数据结构。
你可以把它想象成一个拥有无数带编号抽屉的超级大柜子：
- **存数据时**：系统会通过一个**“哈希函数（Hash Function）”**，把你的特征码（Key）通过数学运算变成一个“抽屉编号（索引）”，然后把数据放进那个抽屉里。
- **取数据时**：系统再次把你的 Key 放进哈希函数算一下，直接得出抽屉编号，打开抽屉就能拿到数据。

### 2. 为什么它这么快？
因为它是**通过数学公式直接算出地址**的！
- 如果用数组或链表找一个东西，你需要从头到尾一个个看（时间复杂度 $O(N)$）。
- 而用哈希表，不管柜子里有 10 个还是 1 亿个物品，只要用公式一算就能瞬间定位目标位置。所以它的**查找、插入和删除的时间复杂度平均都是 $O(1)$**。

### 3. 哈希冲突（Hash Collision）
抽屉的数量终究是有限的，如果哈希函数把两个不同的 Key 算出了相同的“抽屉编号”怎么办？这就是“哈希冲突”。
常见的解决方法有：
- **链地址法（拉链法）：** 发现抽屉里已经有东西了，就把新东西跟旧东西用链条拴在一起（在那个位置挂一个链表）。Python 的 `dict` 早期就借鉴了这种思想。
- **开放寻址法：** 发现抽屉被占了，就按一定规律去找下一个空着的抽屉。Python 现在的 `dict` 采用的主要是优化后的开放寻址法。

### 4. 在 Python 中的形态
在 Python 中，我们不需要手写哈希表，因为它已经被完美封装成了两种极其常用的内置数据结构：
- **字典 (`dict`)**：存储键值对映射 (`Key: Value`)，底层是哈希表。
- **集合 (`set`)**：只存储键 (`Key`)，底层也是哈希表，天生具备去重和极速查找功能。

---

## 🟢 场景一：利用哈希表做“空间换时间” (两数之和)
**题目：** [1. 两数之和 (Two Sum)](https://leetcode.cn/problems/two-sum/)  
**难度：** 简单

### 💡 核心思路
- **暴力法痛点：** 双重循环遍历，时间复杂度 $O(N^2)$，数据量大时极慢。
- **哈希表破局：** 遍历数组时，把 `(当前数字, 它的索引)` 存入字典。对于每个数 `n`，只需直接在字典中查找 `target - n` 是否存在即可。
- **特点：** 边遍历、边存入、边查找。只需一次循环，时间复杂度降为 $O(N)$。

### ⌨️ 核心代码
```python
def twoSum(nums: List[int], target: int) -> List[int]:
    lookup = {}
    for i, n in enumerate(nums):
        another = target - n
        if another in lookup:
            return [lookup[another], i]
        lookup[n] = i  # 记录已访问的数字及其索引
```

---

## 🟡 场景二：利用哈希表做“分类与聚合” (字母异位词分组)
**题目：** [49. 字母异位词分组 (Group Anagrams)](https://leetcode.cn/problems/group-anagrams/)  
**难度：** 中等

### 💡 核心思路
- **异位词的本质：** 两个单词无论字母怎么打乱，只要**按字母排序后**，它们必然变成同一个字符串（这就是它们的“唯一特征码”）。
- **哈希表破局：** 把“排序后的字符串”作为字典的 Key，把“原始单词”追加到对应 Key 的列表（Value）里。
- **技巧：** Python 中原生排序是 `Timsort`，极快。为了防止处理不存在的 Key 报错，推荐使用 `collections.defaultdict(list)`。

### ⌨️ 核心代码
```python
from collections import defaultdict

def groupAnagrams(strs: List[str]) -> List[List[str]]:
    hashmap = defaultdict(list)
    for s in strs:
        # 将字符串排序后合并，作为哈希表的键
        key = ''.join(sorted(s))
        hashmap[key].append(s)
    return list(hashmap.values())
```

---

## 🟡 场景三：利用哈希集合做“O(1) 极速查找” (最长连续序列)
**题目：** [128. 最长连续序列 (Longest Consecutive Sequence)](https://leetcode.cn/problems/longest-consecutive-sequence/)  
**难度：** 中等

### 💡 核心思路
- **常规法痛点：** 找连续数字最直观的反应是先排序。但排序（内置 `sorted` 也是 Timsort）的时间复杂度是 $O(N \log N)$，不符合题目 $O(N)$ 的要求。
- **哈希集合破局：** 把所有数字扔进 `set` 去重并实现 $O(1)$ 查找。
- **绝妙巧思（找起点）：** 怎么避免重复遍历？只要判断 `当前数字 - 1` 在不在集合里：
  - 如果**在**：说明它不是序列开头，直接跳过。
  - 如果**不在**：说明它是新序列的起点！从它开始 `while` 循环不断往后找 `n+1`。

### ⌨️ 核心代码（含剪枝优化）
```python
def longestConsecutive(nums: List[int]) -> int:
    num_set = set(nums)
    longest_streak = 0
    total_unique_nums = len(num_set)
    
    for num in num_set:
        # 极致剪枝优化：如果当前记录已经超过总长度的一半，剩下的就不可能更长了
        if longest_streak > total_unique_nums // 2:
            break
            
        # 只有当 num-1 不存在时，num 才是序列的起点
        if num - 1 not in num_set:
            current_num = num
            current_streak = 1
            
            # 顺藤摸瓜找后续连续数字
            while current_num + 1 in num_set:
                current_num += 1
                current_streak += 1
                
            longest_streak = max(longest_streak, current_streak)
            
    return longest_streak
```

---

## 🌟 总结：哈希表使用心法
遇到 LeetCode 题目时，当你心中闪过这三个念头之一，请立刻召唤哈希表：
1. **“需要寻找一个匹配的值”** ➡️ `dict` 记录走过的路。
2. **“需要把一类具有共同特征的东西归在一起”** ➡️ 提炼特征做 Key，用 `dict` 聚合。
3. **“需要极快地判断一个元素存不存在”** ➡️ 直接丢进 `set`。