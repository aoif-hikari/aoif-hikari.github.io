---
title: Leetcode / 链表
tags: leetcode
date: 2021-07-07
---

> 常用数据结构--链表

<!-- more -->

```java
// 常用操作
if(head == null || head.next == null) return head;
// 设预指针pre的下一个节点指向head，防止空节点
ListNode pre = new ListNode(); 
pre.next = head;
// 设置动态当前指针cur
ListNode cur = head; 
```

## 基本操作

707 设计链表
剑指 Offer 06. 从尾到头打印链表（栈）

- 删

剑指 Offer 18. 删除链表的节点
83 删除排序链表中的重复元素(相同节点保留一个)
**82 删除排序链表中的重复元素 II(相同节点全部删除)**

- 翻转

24 两两交换链表中的结点
206/剑指 Offer 24. 反转链表
92 反转链表 II
25 k个一组翻转链表

- 合并

21/ 剑指 Offer 25 合并两个有序链表
23 合并K个升序链表
2 两数相加
445 两数相加 II（栈）

- 快慢指针

19/剑指 Offer 22 删除链表的倒数第 N 个结点
61 旋转链表
876 链表的中间结点
234 回文链表(中点+反转)
143 重排链表(中点+反转+合并)

- 拆分

86 分隔链表
328 奇偶链表
**138/剑指 Offer 35. 复杂链表的复制**（hashmap / 链接+拆分）

- 其他

**160/剑指 Offer 52. 两个链表的第一个公共节点**

> - **集合set**
> - **快慢指针（先统计两个链表的长度，如果长度不一样，让链表长的先走，直到两个链表长度一样，这此时两个链表同时每次后移一步，看节点是否一样）**
> - **双指针**