# LeetCode题解思路

# 数据结构

## 一、哈希表

### 1、两数之和

**题目链接**：[两数之和](https://leetcode-cn.com/problems/two-sum/)

**解题思路**：

1. 遍历一遍数组，先把所有数存进hashmap中。以空间换时间
2. 再遍历一遍数组，在hashmap中查找target-nums[i]的值是否存在，若存在，返回i和map.get(target-nums[i])

**实现代码**：

```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        HashMap<Integer,Integer> map = new HashMap<>();
        for(int i=0;i<nums.length;i++){
            map.put(nums[i],i);
        }
        for(int i=0;i<nums.length;i++){
            int res = target - nums[i];
            if(map.containsKey(res) && map.get(res)!=i){
                return new int[]{i,map.get(res)};
            }
        }
        return new int[0];
    }
}
```

### 41、缺失的第一个正数

**题目链接**：[缺失的第一个正数](https://leetcode-cn.com/problems/first-missing-positive/)

**解题思路**：

思路一：

1.可以使用hashset对数据进行存储

2.遍历1到数组长度，如果不在hashset里面说明就是缺失的第一个正数

3.如果到数组最末尾，则返回数组长度+1

思路二：

1.把数组当成哈希表来使用，遍历每一个元素，把这个元素移动到它减1的索引下标位置。比如数值3移动到index为2的位置

2.直到所有元素排好序，再遍历一遍数组，找出num[i]!=i+1的值

**代码实现**：

实现一：时间复杂度O(n)空间复杂度O(n)

```java
class Solution {
    public int firstMissingPositive(int[] nums) {
        int len = nums.length;
        HashSet<Integer> set = new HashSet<>();
        for(int x:nums){
            set.add(x);
        }
        for(int i=1;i<=len;i++){
            if(!set.contains(i)){
                return i;
            }
        }
        return len+1;
    }
}
```

实现二：时间复杂度O(n)空间复杂度O(1)

```java
class Solution {
    public int firstMissingPositive(int[] nums) {
        int len = nums.length;
        //把数组当哈希表使用。把数值3放到索引为2的位置上
        for(int i=0;i<len;i++){
            //使用while的原因是交换后还不知道num[i]的位置因此还需要继续           
            while(nums[i]>=1 && nums[i]<=len){  
                //nums[i]!=nums[nums[i]-1]表示要交换的元素就是那个位置的话就没要交换了
                if(nums[i]!=nums[nums[i]-1]){
                    swap(nums,i,nums[i]-1);
                }else{
                    break;
                }           
                
            }
        }
        //最后遍历数组
        for(int i=0;i<nums.length;i++){
            if(nums[i]!=i+1){
                return i+1;
            }
        }
        return len+1;
    }

    private void swap(int[] nums,int i,int j){
        int tmp = nums[i];
        nums[i] = nums[j];
        nums[j] = tmp;
    }
}
```



### 49、字母异位词分组

**题目链接**：[字母异位词分组](https://leetcode-cn.com/problems/group-anagrams/)

**解题思路**：

解题关键是异位词排序后是相同的，所以考虑使用HashMap数据结构，以排序后的异位词为key。

**代码实现**：

```java
class Solution {
    public List<List<String>> groupAnagrams(String[] strs) {
        HashMap<String, List<String>> map = new HashMap<>();
        for (String s : strs) {
            char[] c = s.toCharArray();
            //对字符数组排序
            Arrays.sort(c);
            //排序后的字符数组作为hashmap的key
            String key = String.valueOf(c);
            List<String> list = map.getOrDefault(key, new ArrayList<>());
            list.add(s);
            map.put(key, list);
        }
        return new ArrayList<List<String>>(map.values());

    }
}
```



### 128、最长连续序列

**题目链接**：[最长连续序列](https://leetcode-cn.com/problems/longest-consecutive-sequence/)

**解题思路**：

考虑到有重复元素的出现，可以使用HashSet进行去重。使用变量count记录最长连续序列的个数。

注意：从当前元素开始，它的count是从1开始的

**代码实现**：

```java
class Solution {
    public int longestConsecutive(int[] nums) {
        //边界条件
        int len = nums.length;
        if(len==0 || len==1) return len;
        //关键对数组排序
        Arrays.sort(nums);
        HashSet<Integer> set = new HashSet<>();
        set.add(nums[0]);
        int count = 1;
        int maxCount = 0;
        for (int i = 1; i < len; i++) {
            if(set.contains(nums[i])) continue;
            if(set.contains(nums[i]-1)){
                count++;
            }else{
                maxCount = Math.max(maxCount,count);
                count = 1;
            }
            set.add(nums[i]);
        }
        return Math.max(maxCount, count);
    }
}
```



### 146、LRU缓存

**题目链接**：[LRU缓存](https://leetcode-cn.com/problems/lru-cache/)

**算法思路**：

关键点：

1、在makeRecently()使用方法中，先获取到key对应的value，再删除key，最后再插入key

2、get()方法中，先判断key在不在，如果在，先将调用makeRecently()方法，然后再获取

3、在put()方法中，先判断key在不在，如果在就直接put进行覆盖，然后调用makeRecently()方法进行返回；如果不在则先判断是否已经达到容量，如果已经达到容量了，则先删除头部元素(也就是最近最久未使用)，最后再执行插入操作

**代码实现**：

```java
class LRUCache {
    int cap;
    LinkedHashMap<Integer, Integer> cache = new LinkedHashMap<>();
    public LRUCache(int capacity) { 
        this.cap = capacity;
    }
    
    public int get(int key) {
        if (!cache.containsKey(key)) {
            return -1;
        }
        // 将 key 变为最近使用
        makeRecently(key);
        return cache.get(key);
    }
    
    public void put(int key, int val) {
        if (cache.containsKey(key)) {
            // 修改 key 的值
            cache.put(key, val);
            // 将 key 变为最近使用
            makeRecently(key);
            return;
        }
        
        if (cache.size() >= this.cap) {
            // 链表头部就是最久未使用的 key
            int oldestKey = cache.keySet().iterator().next();
            cache.remove(oldestKey);
        }
        // 将新的 key 添加链表尾部
        cache.put(key, val);
    }
    
    private void makeRecently(int key) {
        int val = cache.get(key);
        // 删除 key，重新插入到队尾
        cache.remove(key);
        cache.put(key, val);
    }
}


/**
 * Your LRUCache object will be instantiated and called as such:
 * LRUCache obj = new LRUCache(capacity);
 * int param_1 = obj.get(key);
 * obj.put(key,value);
 */
```



### 208、实现Trie(前缀树)

👍前缀树算法模板：[前缀树算法模板](https://labuladong.gitee.io/algo/2/20/46/)

👍图解前缀树：[图解前缀树](https://blog.csdn.net/m0_46202073/article/details/107253959)

**题目链接**：[实现Trie(前缀树)](https://leetcode-cn.com/problems/implement-trie-prefix-tree/)

**解题思路**：

**代码实现**：

```

```





## 二、链表

👍**链表模式识别**：

- 涉及链表特殊位置，使用快慢指针
- 要删除链表节点，找到节点的前驱节点

### 2、两数相加

**题目链接**：[两数相加](https://leetcode-cn.com/problems/add-two-numbers/)

**解题思路**：

基本思路就是取出两个数的当前位进行相加，判断有无进位等。

两数相加公式

```
当前位 = (A 的当前位 + B 的当前位 + 进位carry) % 10
注意，AB两数都加完后，最后判断一下进位 carry, 进位不为 0 的话加在前面。
```

两数相加算法模板

```
while ( A 没完 || B 没完)
    A 的当前位
    B 的当前位

    和 = A 的当前位 + B 的当前位 + 进位carry

    当前位 = 和 % 10;
    进位 = 和 / 10;

判断还有进位吗
```

注意：注意进位在循环里的第一步，第一步就要加上进位，容易忽略

**实现代码**：

```java
class Solution {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        //两条链表上的指针
        ListNode p1 = l1,p2 = l2;
        //虚拟头节点
        ListNode dummy = new ListNode();
        //指针p负责构建新链表
        ListNode p = dummy;
        //记录进位
        int carry = 0;
        while(p1!=null || p2!=null || carry!=0){
            //先加上上次的进位
            int val = carry;
            if(p1!=null){
                val += p1.val;
                p1 = p1.next;
            }
            if(p2!=null){
                val += p2.val;
                p2 = p2.next;
            }
            //处理进位情况
            carry = val/10;
            val = val%10;
            //构建新的节点
            ListNode newNode = new ListNode(val);
            p.next = newNode;
            p = p.next;
        }
        // 返回结果链表的头结点（去除虚拟头结点）
        return dummy.next;
    }
}
```

### 19、删除链表倒数第N个节点

**题目链接**：[删除链表倒数第N个节点](https://leetcode-cn.com/problems/remove-nth-node-from-end-of-list/)

**解题思路**：

可以使用快慢指针。快指针和慢指针分别指向头节点，先让快指针移动n步，然后快慢指针一起移动。

注意：如果快指针移动完了n步，此时快指针为空，说明头节点就是要删除的节点。

**代码实现**：

```Java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 * int val;
 * ListNode next;
 * ListNode() {}
 * ListNode(int val) { this.val = val; }
 * ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode removeNthFromEnd(ListNode head, int n) {
        if (head == null) {
            return null;
        }
        //快慢指针分别指向头节点
        ListNode fast = head;
        ListNode slow = head;//注意
        //快指针先移动n步
        while (n > 0) {
            fast = fast.next;
            n--;
        }
        //如果此时快指针为空，说明头节点就是要删除的节点
        if(fast==null){
            return head.next;
        }
        //快慢指针同时移动
        while(fast.next!=null){
            fast = fast.next;
            slow = slow.next;
        }
        //删除慢指针后继节点
        slow.next = slow.next.next;//注意
        return head;
    }
}

```

### 21、合并两个有序链表

**题目链接**：[合并两个有序链表](https://leetcode-cn.com/problems/merge-two-sorted-lists/)

**解题思路**：

使用归并排序的思想。

**代码实现**：

迭代

```java
class Solution {
    public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
        if (l1 == null) {
            return l2;
        }
        if(l2==null){
            return l1;
        }
        ListNode head = new ListNode();
        ListNode p1 = head;
        while(l1!=null && l2!=null){
            if(l1.val <= l2.val){
                p1.next = l1;
                l1=l1.next;
            }else{
                p1.next = l2;
                l2 = l2.next;
            }
            p1 = p1.next;
        }
       p1.next = l1==null?l2:l1;
        return head.next;
    }
    
}
```

递归

```java
class Solution {
    public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
        if (l1 == null) {
            return l2;
        }
        if(l2==null){
            return l1;
        }
        if(l1.val<=l2.val){
            l1.next = mergeTwoLists(l1.next,l2);
            return l1;
        }else{
            l2.next = mergeTwoLists(l1,l2.next);
            return l2;
        }
    }
    
}
```



### 23、合并k个升序链表

**题目链接**：[合并k个升序链表](https://leetcode-cn.com/problems/merge-k-sorted-lists/)

**解题思路**：

采用归并排序的思想，分而治之。

**代码实现**：

```java
class Solution {
    public ListNode mergeKLists(ListNode[] lists) {
        if (lists.length == 0 || lists == null) {
            return null;
        }
        return merge(lists, 0, lists.length - 1);
    }

    private ListNode merge(ListNode[] lists,int left,int right){
        if (left == right) {
            return lists[left];
        }
        int mid = left + ((right-left)>>1);
        ListNode l1 = merge(lists, left, mid);
        ListNode l2 = merge(lists, mid + 1, right);
        return mergeTwoLists(l1, l2);
    }

    private ListNode mergeTwoLists(ListNode l1, ListNode l2) {
        if (l1 == null) {
            return l2;
        }
        if (l2 == null) {
            return l1;
        }
        if(l1.val<=l2.val){
            l1.next = mergeTwoLists(l1.next,l2);
            return l1;
        }else{
            l2.next = mergeTwoLists(l1, l2.next);
            return l2;
        }

    }
}

```



### 82、删除排序链表中的重复元素2

**题目链接**：[删除排序链表中的重复元素2](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-list-ii/)

**算法思路**：

**代码实现**：

```java
class Solution {
    public ListNode deleteDuplicates(ListNode head) {
        if(head==null || head.next==null) return head;
        //因为头节点可能重复，所以构建虚拟节点
        ListNode newHead = new ListNode();
        ListNode pre = newHead;
        ListNode cur = head;
        pre.next = cur;
        while(cur!=null && cur.next!=null){
            if(cur.val==cur.next.val){
                while(cur.next!=null && cur.val==cur.next.val){
                    cur = cur.next;
                }
                pre.next = cur.next;
            }else{
                pre = cur;
            }
            cur = cur.next;
        }
        return newHead.next;
    }
}
```





### 83、删除排序链表中的重复元素1

**题目链接**：[删除排序链表中的重复元素1](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-list/submissions/)

**算法思路**：

1.使用快慢指针，快指针的值不等于慢指针的值，即可让慢指针的下一位等于快指针

注意：删除重复元素，但并不是把所有重复元素删除，可以考虑使用快慢指针来实现

**代码实现**：

```java
class Solution {
    public ListNode deleteDuplicates(ListNode head) {
        if(head==null || head.next==null) return head;
        //使用快慢指针
        ListNode slow = head;
        ListNode fast = head;
        while(fast!=null){
            if(fast.val!=slow.val){
                slow.next = fast;
                slow = slow.next;
            }
            fast = fast.next;
        }
        //最后要把链断开
        slow.next = null;
        return head;
    }
}
```





### 141、环形链表

**题目链接**：[环形链表](https://leetcode-cn.com/problems/linked-list-cycle/)

**解题思路**：

定义两个快慢指针，当两个指针相遇时，说明有环。

注意：fast的初始值是 head.next;while循环条件是fast!=null && fast.next!=null

**代码实现**：

```java
public class Solution {
    public boolean hasCycle(ListNode head) {
        if(head==null || head.next==null){
            return false;
        }
        ListNode slow = head;
        ListNode fast = head.next;//注意
        while(fast!=null && fast.next!=null){
            slow = slow.next;
            fast = fast.next.next;
            if(slow==fast){
                return true;
            }
        }
        return false;
    }
}
```



### 142、环形链表2

**题目链接**：[环形链表2](https://leetcode-cn.com/problems/linked-list-cycle-ii/)

**解题思路**：

定义两个快慢指针，当两者相遇时，让慢指针回到起点，两个指针同时往前走一步，当两个指针再次相遇时，这个节点就是环形的入口。

注意：fast的初始化是head，不是head.next;

**代码实现**：

```java
public class Solution {
    public ListNode detectCycle(ListNode head) {
        if(head==null || head.next==null){
            return null;
        }
        ListNode slow = head;
        ListNode fast = head;//注意
        while(fast!=null && fast.next!=null){
            slow = slow.next;
            fast = fast.next.next;
            if(slow==fast){
                slow = head;
                while(slow!=fast){
                    slow = slow.next;
                    fast = fast.next;
                }
                return slow;
            }
        }
        return null;
    }
}
```



### 148、排序链表

**题目链接**：[排序链表](https://leetcode-cn.com/problems/sort-list/)

**解题思路**：

使用归并排序的思想。

1.先定义快慢指针，找到链表的中点

2.从中点处断开，分别对头节点和中点使用递归（归并排序）

3.最后合并两个有序链表



注意：fast的初始值为head.next；

**代码实现**：

```java
class Solution {
    public ListNode sortList(ListNode head) {
        if(head==null || head.next==null){
            return head;
        }
        ListNode slow = head;
        ListNode fast = head.next;//注意
        
        //寻找中间节点
        while(fast!=null && fast.next!=null){
            slow = slow.next;
            fast = fast.next.next;
        }

        //找到中间节点
        ListNode mid = slow.next;
        slow.next = null;

        //归并排序。树的后续遍历
        ListNode l1 = sortList(head);
        ListNode l2 = sortList(mid);
        return merge(l1,l2);

    }

    //合并两个有序链表
    private ListNode merge(ListNode l1,ListNode l2){
        if(l1==null) return l2;
        if(l2==null) return l1;

        if(l1.val<=l2.val){
            l1.next = merge(l1.next,l2);
            return l1;
        }else{
            l2.next = merge(l1,l2.next);
            return l2;
        }
    }
}
```



### 160、相交链表

**题目链接**：[相交链表](https://leetcode-cn.com/problems/intersection-of-two-linked-lists/)

**解题思路**：

1.定义两个指针，他们同时往前，当指针不为空时，指向它们的下一个节点，当指针为空时，指向另一个链表的头节点

2.直到两个指针相等时跳出循环，此时的节点就是两个链表相交的节点



注意：循环里面是p==null而不是p.next==null

**代码实现**：

```java
public class Solution {
    public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
        if(headA==null || headB==null) return null;

        ListNode p1 = headA;
        ListNode p2 = headB;

        while(p1!=p2){
            p1 = (p1==null)?headB:p1.next;
            p2 = (p2==null)?headA:p2.next;
        }
        return p1;
    }
}
```



### 206、反转链表

👍[递归反转链表](https://labuladong.gitee.io/algo/2/17/17/)

**题目链接**：[反转链表](https://leetcode-cn.com/problems/reverse-linked-list/)

**解题思路**：

递归是最简单的实现。相当于树的后序遍历。定义递归函数，表示反转链表。当链表反转完成之后，当前节点要做什么。

![递归反转链表](https://labuladong.gitee.io/algo/images/%e5%8f%8d%e8%bd%ac%e9%93%be%e8%a1%a8/5.jpg)

**代码实现**：

递归

```java
class Solution {
    public ListNode reverseList(ListNode head) {
        //递归结束条件
        if(head==null || head.next==null) return head;
        //相当于树的后序遍历
        ListNode last = reverseList(head.next);
        //当前节点要做的事
        head.next.next = head;//就是让当前的1的next的2的next指向1，即2.next = 1;
        head.next = null;//1.next=null
        return last;
    }
}
```

迭代

```java
class Solution {
    public ListNode reverseList(ListNode head) {
        //定义三个指针，分别表示前置指针、当前指针、后继指针
        ListNode pre = null;
        ListNode cur = head;
        ListNode nxt = head;
        while(cur!=null){
            nxt = cur.next;
            //逐个节点反转
            cur.next = pre;
            //更新指针位置
            pre = cur;
            cur = nxt;
        }
        //返回反转后的头节点
        return pre;

    }
}
```





**扩展：递归反转链表的前n个节点**

解决思路和反转链表差不多

![](https://labuladong.gitee.io/algo/images/%e5%8f%8d%e8%bd%ac%e9%93%be%e8%a1%a8/7.jpg)

```java
ListNode successor = null; // 后驱节点

// 反转以 head 为起点的 n 个节点，返回新的头结点
ListNode reverseN(ListNode head, int n) {
    if (n == 1) {
        // 记录第 n + 1 个节点
        successor = head.next;
        return head;
    }
    // 以 head.next 为起点，需要反转前 n - 1 个节点
    ListNode last = reverseN(head.next, n - 1);

    head.next.next = head;
    // 让反转之后的 head 节点和后面的节点连起来
    head.next = successor;
    return last;
}

```

**扩展：反转以a为头节点的链表，就是反转a到null之间的节点，那么反转a到b之间的节点呢?**

```java
/** 反转区间 [a, b) 的元素，注意是左闭右开 */
ListNode reverse(ListNode a, ListNode b) {
    ListNode pre, cur, nxt;
    pre = null; cur = a; nxt = a;
    // while 终止的条件改一下就行了
    while (cur != b) {
        nxt = cur.next;
        cur.next = pre;
        pre = cur;
        cur = nxt;
    }
    // 返回反转后的头结点
    return pre;
}
```



### 92、反转链表2

**题目链接**：[反转链表2](https://leetcode-cn.com/problems/reverse-linked-list-ii/)

**解题思路**：

1.首先明白反转链表前n个节点的操作

2.基于上面操作，只要让base case等于上面操作就行

3.接下来就是交给递归函数来做就行。如果 `m != 1` 怎么办？如果我们把 `head` 的索引视为 1，那么我们是想从第 `m` 个元素开始反转对吧；如果把 `head.next` 的索引视为 1 呢？那么相对于 `head.next`，反转的区间应该是从第 `m - 1` 个元素开始的；那么对于 `head.next.next` 就是m-2，以此类推...

**代码实现**：

```java
class Solution {
    public ListNode reverseBetween(ListNode head, int m, int n) {
        // base case
        if (m == 1) {
            return reverseN(head, n);
        }
        // 前进到反转的起点触发 base case
        head.next = reverseBetween(head.next, m - 1, n - 1);
        return head;
    }

    ListNode successor = null; // 后驱节点
    // 反转以 head 为起点的 n 个节点，返回新的头结点
    ListNode reverseN(ListNode head, int n) {
        if (n == 1) {
            // 记录第 n + 1 个节点
            successor = head.next;
            return head;
        }
        // 以 head.next 为起点，需要反转前 n - 1 个节点
        ListNode last = reverseN(head.next, n - 1);

        head.next.next = head;
        // 让反转之后的 head 节点和后面的节点连起来
        head.next = successor;
        return last;
    }
}
```



### 25、k个一组反转链表

👍[如何K个一组反转链表](https://labuladong.gitee.io/algo/2/17/18/)

**题目链接**：[K个一组反转链表](https://leetcode-cn.com/problems/reverse-nodes-in-k-group/)

**解题思路**：

1.首先明白反转节点a到节点b该怎么反转

2.每k个一组进行反转

3.递归连接k个一组反转的链表

**代码实现**：

```java
class Solution {
    public ListNode reverseKGroup(ListNode head, int k) {
        if(head==null) return null;
        //区间(a,b],包含k个待反转元素
        ListNode a = head;
        ListNode b = head;
        for(int i=0;i<k;i++){
            if(b==null) return head;
            b = b.next;
        }
        //反转前k个元素
        ListNode newHead = reverse(a,b);
        //递归反转后续链表并链接
        a.next = reverseKGroup(b,k);
        return newHead;
    }

    //反转节点a到节点b之间的链表节点，包含头，但不包含尾.[a,b)
    private ListNode reverse(ListNode a,ListNode b){

        ListNode pre = null;
        ListNode cur = a;
        ListNode nxt = a;
        while(cur!=b){
            nxt = cur.next;
            cur.next = pre;
            pre = cur;
            cur = nxt;
        }
        return pre;
    }
}
```



### 143、重排链表

**题目链接**：[重排链表](https://leetcode-cn.com/problems/reorder-list/)

**解题思路**：

1. 先找出链表的中间节点，将其一分两段
2. 反转后序链表
3. 将两个链表进行合并

**代码实现**：

```java
class Solution {
    public void reorderList(ListNode head) {
        //1.先找到中间链表的节点
        ListNode slow = head;
        ListNode fast = head.next;
        while(fast!=null && fast.next!=null){
            slow = slow.next;
            fast = fast.next.next;
        }
        ListNode mid = slow.next;
        slow.next = null;//注意slow.next要置为空，不然会出现循环
        //2.反转第二个链表的位置
        ListNode tail = reverse(mid);
        //3.合并两个链表
        mergeList(head,tail);
    }

    //递归反转链表的操作
    private ListNode reverse(ListNode head){
        if(head==null || head.next==null){
            return head;
        }
        ListNode last = reverse(head.next);
        head.next.next = head;
        head.next = null;
        return last;
    }

    //合并两个链表
    private void mergeList(ListNode l1,ListNode l2){
        ListNode p1;
        ListNode p2;
        while(l1!=null && l2!=null){
            p1 = l1.next;
            p2 = l2.next;
            l1.next = l2;
            l1 = p1;
            l2.next = l1;
            l2 = p2;
        }
    }
}
```





## 三、栈

👍**单调栈算法模板：**

```java
//单调栈就是保证里面的元素是递增的或者是递减的
//它的应用场景比较多的应用于Next Greater Element（下一个更大数组）
vector<int> nextGreaterElement(vector<int>& nums) {
    vector<int> res(nums.size()); // 存放答案的数组
    stack<int> s;
    // 倒着往栈里放（正放和倒放取决于对题目的要求和理解）
    for (int i = nums.size() - 1; i >= 0; i--) {
        // 判定个子高矮
        while (!s.empty() && s.top() <= nums[i]) {
            // 矮个起开，反正也被挡着了。。。
            s.pop();
        }
        // nums[i] 身后的 next great number
        res[i] = s.empty() ? -1 : s.top();
        // 
        s.push(nums[i]);
    }
    return res;
}

//使不使用哨兵，取决于具体题目
int[] nextGreaterElement(int[] nums){
    int[] res = new int[nums.length];//存放数组答案
    Deque<Integer> stack = new ArrayDeque<>();
    for(int i=nums.length-1;i>=0;i--){
        //单调栈,判定个子高矮
        while(!stack.isEmpty() && stack.peek()<= nums[i]){
             // 矮个起开，反正也被挡着了。。。
            stack.pop();
        }
        // nums[i] 身后的 next great number
        res[i] = s.empty() ? -1 : s.top();
        // 入栈
        s.push(nums[i]);
    }
    
    return res;
}
```





### 20、有效的括号

**题目链接**：[有效的括号](https://leetcode-cn.com/problems/valid-parentheses/)

**解题思路**：

1. 先使用hashmap存储括号匹配
2. 使用栈，碰到左括号就入栈，碰到右括号就出栈与之匹配
3. 最后返回栈是否为空

**代码实现**：

```java
class Solution {
    public boolean isValid(String s) {
        //先用hashmap存储括号匹配问题
        HashMap<Character, Character> map = new HashMap<>(){
            {
                put('(', ')');
                put('[', ']');
                put('{', '}');
            }
        };
		//使用栈来存储括号
        Stack<Character> stack = new Stack<>();
        for (int i = 0; i < s.length(); i++) {
            char c = s.charAt(i);
            //说明是左括号，直接入栈
            if(map.containsKey(c)){
                stack.push(c);
            }else{
                //说明是右括号，弹出左括号跟他匹配
                if(!stack.isEmpty()){
                    char c1 = stack.pop();
                    if(map.get(c1)!=c){
                        return false;
                    }
                }else{
                    return false;
                }
            }
        }
        return stack.isEmpty();

    }
    
}
```



### 32、最长有效括号

**题目链接**：[最长有效括号](https://leetcode-cn.com/problems/longest-valid-parentheses/)

**解题思路**：

方法一：动态规划

1.定义dp数组，dp[i]表示以s[i-1]为结尾的字符串最长有效括号

2.定义栈存储左括号的下标，遍历数组，碰到左括号，进栈，并且当前下标的dp[i+1]为0；碰到右括号，则出栈，计算有效括号的长度，当前有效括号的长度 = i - leftIndex +1 + dp[leftIndex]；之前的dp[leftIndex]不能丢

3.最后遍历dp数组，找出最长的有效括号次数

注意：为什么dp数组要为s.length()+1？用dp[i+1]取存储下标为i的值

以括号 )()()( 为例

1.dp[] = new int[s.length()+1],dp[i+1]的情况（正确答案）

| dp[0] | dp[1] | dp[2] | dp[3] | dp[4] | dp[5] | dp[6] |
| ----- | ----- | ----- | ----- | ----- | ----- | ----- |
|       | 0     | 0     | 2     | 0     | 4     | 0     |

2.dp[] = new int[s.length()],dp[i]的情况（错误答案）

| dp[0] | dp[1] | dp[2] | dp[3] | dp[4] | dp[5] |
| ----- | ----- | ----- | ----- | ----- | ----- |
| 0     | 0     | 2     | 0     | 2     | 0     |





方法二：栈

1. 定义一个栈，先放入-1。这样做的目的是，让当前右括号的下标减去栈顶元素即为要求的长度，也为了保持统一。
2. 遇到 '(' ，将它的下标放入栈中
3. 遇到 ')' ，先弹出栈顶元素表示匹配了当前右括号：如果栈为空，说明当前的右括号没有被匹配，我们将其下标放入栈中

关键：栈存储的是数组下标；先放入-1；碰到左括号就添加到栈，注意放入的是下标；碰到右括号，先弹出栈顶元素，再取栈顶元素进行相减取最大

**代码实现**：

动态规划

```java
class Solution {
    public int longestValidParentheses(String s) {
        //定义数组，dp[i]表示以s[i-1]结尾的字符串的最长有效括号
        int[] dp = new int[s.length()+1];
        //定义栈，存储左括号的下标索引
        Stack<Integer> stack = new Stack<>();
        for(int i=0;i<s.length();i++){
            if(s.charAt(i)=='('){
                stack.push(i);
                //既然是左括号，当前位置的字符串最长有效括号肯定为0
                dp[i+1] = 0;
            }else{
                //遇到右括,栈不为空才能取出左括号索引
                if(!stack.isEmpty()){
                    int leftIndex = stack.pop();
                    int len = i - leftIndex + 1 + dp[leftIndex];
                    dp[i+1] = len;
                }else{
                    dp[i+1] = 0;
                }
            }
        }

        int res = 0;
        for(int x:dp){
            res = Math.max(res,x);
        }

        return res;
    }
}
```



栈

```java
class Solution {
    public int longestValidParentheses(String s) {
        if(s.length()==0) return 0;
        //存储结果
        int res = 0;
        //注意存储的是下标
        Deque<Integer> stack = new ArrayDeque<>();
        //先放入-1
        stack.push(-1);
        for(int i=0;i<s.length();i++){
            if(s.charAt(i)=='('){
                stack.push(i);
            }else{
                stack.pop();
                //如果此时栈为空，要把右括号的下表放进去
                if(stack.isEmpty()){
                    stack.push(i);
                }
                res = Math.max(res,i-stack.peek());
            }
        }
        return res;
    }
}
```



### 921、使括号有效的最少添加

**题目链接**：[使括号有效的最少添加](https://leetcode-cn.com/problems/minimum-add-to-make-parentheses-valid/)

**解题思路**：

判断括号有效性的算法

```java
bool isValid(string str) {

    // 待匹配的左括号数量
    int left = 0;
    for (int i = 0; i < str.size(); i++) {
        if (s[i] == '(') {
            left++;
        } else {
            // 遇到右括号
            left--;
        }
        // 右括号太多
        if (left == -1)
            return false;
    }
    // 是否所有的左括号都被匹配了
    return left == 0;
}

```

这题思路差不多：

1.使用两个变量分别表示需要多少括号的次数

2.遍历字符串，每碰到左括号则右括号需要的次数加1，碰到右括号则右括号需要的次数减1。如果右括号多了，则左括号需要加1

3.最后返回左右括号相加

**代码实现**：

```java
class Solution {
    public int minAddToMakeValid(String s) {

        //记录左括号需要的次数
        int leftNeed = 0;
        //记录右括号需要的次数
        int rightNeed = 0;

        for(int i=0;i<s.length();i++){
            if(s.charAt(i)=='('){
                rightNeed++;
            }
            if(s.charAt(i)==')'){
                rightNeed--;
            }
            //右括号太多
            if(rightNeed==-1){
                //添加左括号
                leftNeed++;
                //达到平衡
                rightNeed = 0;
            }
        }

        //最后返回左括号需要的次数 + 右括号需要的次数
        return leftNeed + rightNeed;

    }
}
```



### 84、柱状图中最大的矩形

**题目链接：**[柱状图中最大的矩形](https://leetcode-cn.com/problems/largest-rectangle-in-histogram/)

**算法思路：**

1、暴力解法

1. 面积=底*高。固定底边，求最大高度不好求；固定高，求最长底边好求
2. 从i向左右两边遍历，找到左边和右边第1个严格小于height[i]的时候停下，中间的长度就是最长底边



2、单调栈

如何想到？因为要找出当前元素左边第一个小于它的元素，以及右边元素第一个小于它的元素，完全符合一个单调递增栈的效果

1. 使用一个栈存储数组下标，里面元素是单调递增的；当进栈的元素严格小于栈顶元素时，栈顶元素出栈，求出矩阵的高，矩阵的宽度就是当前下标减去现在的栈顶元素下标再减1，求出矩阵面积；再比较最大的矩阵面积
2. 注意，可以使用哨兵，可以强制使所有进栈元素都出栈，一种设计技巧

关键：设计哨兵；栈存储的是下标；for循环里面的判断使用的是while循环



**代码实现：**

暴力实现

```java
class Solution {
    public int largestRectangleArea(int[] heights) {
        int len = heights.length;
        if (len == 0) return 0;

        int res = 0;
        for (int i = 0; i < len; i++) {
            int left = i;
            //寻找左边 最后一个 大于等于heights[i]的
            while (left > 0 && heights[left - 1] >= heights[i]) {
                left--;
            }
            int right = i;
            //寻找右边 最后一个 大于等于heights[i]的
            while (right < len - 1 && heights[right + 1] >= heights[i]) {
                right++;
            }
            //求面积，比较大小
            int area = (right - left + 1) * heights[i];
            res = Math.max(res, area);
            
        }

        return res;
    }
}

```

单调栈

```java
class Solution {
    public int largestRectangleArea(int[] heights) {
        int len = heights.length;
        if(len==0) return 0;
        if(len==1) return heights[0];

        //栈，设计成单调栈
        Deque<Integer> stack = new ArrayDeque<>();

        //拷贝一个新的数组，增加数组长度
        int[] newHeights = new int[len + 2];
        //设置哨兵，保证栈不为空，即可以把元素全部出栈
        newHeights[0] = 0;
        newHeights[len+1] = 0;
        System.arraycopy(heights,0,newHeights,1,len);

        //存储结果
        int res = 0;

        for (int i = 0; i < newHeights.length; i++) {
            //单调栈,存储的是数组下标，注意是while循环
            while (!stack.isEmpty() && newHeights[stack.peek()] > newHeights[i]) {
                int curHeight = newHeights[stack.pop()];
                int width = i - stack.peek() - 1;
                res = Math.max(res,curHeight*width);
            }
            stack.push(i);
        }

        return res;

    }
}
```



### 85、最大矩形

**题目链接**：[最大矩形](https://leetcode-cn.com/problems/maximal-rectangle/)

**解题思路**：

将题目这样变形，就变成了84题的柱状图中最大的矩形

![](D:\github\MyKnowledgeRepository\Leetcode_img\85题.png)

依次遍历每一行的矩阵的高，然后代入到上一题的代码中就可以实现最大矩形的计算

**代码实现**：

```java
class Solution {
    public int maximalRectangle(char[][] matrix) {
        int m = matrix.length;
        int n = matrix[0].length;
        if (m == 0) {
            return 0;
        }

        //存储矩阵的最大面积
        int res = 0;

        //存储矩阵的高度
        int[] heights = new int[n];

        for (int i = 0; i < m; i++) {
            //遍历每一个列求出矩阵的高度
            for (int j = 0; j < n; j++) {
                if (matrix[i][j] == '1') {
                    heights[j] += 1;
                }else{
                    heights[j] = 0;
                }
            }
            res = Math.max(res,largeRectangleArea(heights,n));
        }

        return res;
    }

    //上一题的求最大矩阵的面积
    private int largeRectangleArea(int heights[],int len) {
        int res = 0;
        int[] newHeights = new int[len + 2];
        System.arraycopy(heights,0,newHeights,1,len);
        Deque<Integer> stack = new ArrayDeque<>();
        for (int i = 0; i < newHeights.length; i++) {
            while (!stack.isEmpty() && newHeights[stack.peek()] > newHeights[i]) {
                int curHeightIndex = stack.pop();
                int left = stack.peek();
                int curWidth = i - left - 1;
                res = Math.max(res, newHeights[curHeightIndex] * curWidth);
            }
            stack.push(i);
        }
        return res;
    }
}
```



## 四、队列

👍单调递减队列

```java
//构造一个单调队列
class MonotonousQueue{

    LinkedList<Integer> q;

    //构造方法
    MonotonousQueue(){
        this.q = new LinkedList<>();
    }
    //进入队列
    void addValue(int value){
        while(!q.isEmpty() && q.getLast()< value){
            q.pollLast();
        }
        q.offer(value);
    }
    //获取队内最大元素元素
    int getMaxValue(){
        return q.getFirst();
    }
    //删除元素
    void remove(int value){
        if(!q.isEmpty() && q.getFirst()==value){
            q.pollFirst();
        }
    }
} 
```



### 239、滑动窗口的最大值

**题目链接**：[滑动窗口的最大](https://leetcode-cn.com/problems/sliding-window-maximum/)

**解题思路**：

1. 设计一个容器存储窗口内的元素，这个容器就是单调递减队列
2. 遍历元素，先往容器添加 k-1 个元素。之后每添加一个元素就取出队列里的最大元素

**代码实现**：

```java
//构造一个单调队列
class MonotonousQueue{

    LinkedList<Integer> q;

    //构造方法
    MonotonousQueue(){
        this.q = new LinkedList<>();
    }
    //进入队列
    void addValue(int value){
        while(!q.isEmpty() && q.getLast()< value){
            q.pollLast();
        }
        q.offer(value);
    }
    //获取队内最大元素元素
    int getMaxValue(){
        return q.getFirst();
    }
    //删除元素
    void remove(int value){
        if(!q.isEmpty() && q.getFirst()==value){
            q.pollFirst();
        }
    }
} 

public class Solution {
    public int[] maxSlidingWindow(int[] nums, int k) {

        //存储结果
        int[] res = new int[nums.length-k+1];
        if(k==0) return res;

        MonotonousQueue mq = new MonotonousQueue();
        for(int i=0;i<nums.length;i++){
            //先把前k-1个元素填满队列
            if(i<k-1){
                mq.addValue(nums[i]);
            }else{
                mq.addValue(nums[i]);
                res[i-k+1] = mq.getMaxValue();
                mq.remove(nums[i-k+1]);
            }
        }

        return res;

    }
}
```







## 五、二叉树

👍**关键思考和总结**

```tex
1.二叉树算法的关键在于明确根节点要做什么
2.遇到一颗二叉树的通用思考过程：是否可以通过遍历一遍二叉树得到答案？如果不能，是否可以定义一个递归函数，通过子问题（子树）的答案推导出原问题的答案
3.二叉树前序遍历的代码位置只能从函数参数中获取父节点传递来的数据，而后序遍历的代码位置不仅可以获取参数数据，还可以获取到子树通过函数返回值传递回来的数据。中序位置主要用在 BST 场景中，你完全可以把 BST 的中序遍历认为是遍历有序数组。
如果题目是和子树有关，那么大概率是要给函数设置合理的定义和返回值，在后序位置写代码
4.二叉树的一个难点就是如何把题目要求细化成每一个节点要做什么
```

👍**二叉树解题步骤**

```tex
1.是否可以通过遍历一遍二叉树得到答案？如果不能，是否可以通过定义一个递归函数，通过子问题（子树）的答案推到出原问题的答案
2.明确递归函数的定义
3.基于递归框架，明确root节点要做什么，根据题目要求，选择前中后序递归框架

难点：通过题目要求，思考出每个节点要做什么
```





👍**二叉树的非递归遍历模板**

👍前序遍历

```java
/**
     * 迭代前序遍历
     * @param root
     */
private static void pre_travers_iterator(TreeNode root) {
    Deque<TreeNode> stack = new ArrayDeque<>();
    TreeNode cur = root;
    while (cur!=null || !stack.isEmpty()) {
        if (cur != null) {
            System.out.print(cur.val + " ");//前序遍历代码
            stack.push(cur);
            cur = cur.left;
        }else{
            cur = stack.pop();
            //中序遍历代码写这里
            cur = cur.right;
        }
    }

}
```

👍中序遍历

```java
/**
     * 迭代中序遍历
     * @param root
     */
private static void in_travers_itrator(TreeNode root) {
    ArrayDeque<TreeNode> stack = new ArrayDeque<>();
    TreeNode cur = root;
    while (cur!=null || !stack.isEmpty()) {
        if(cur != null) {
            //前序遍历代码写这里
            stack.push(cur);
            cur = cur.left;
        }else{
            cur = stack.pop();
            System.out.print(cur.val + " ");//中序遍历代码写这里
            cur = cur.right;
        }
    }
}
```

👍后序遍历

```java
/**
     * 迭代后序遍历
     * @param root
     */
private static void post_travers_iterator(TreeNode root) {
    ArrayDeque<TreeNode> stack_main = new ArrayDeque<>();
    ArrayDeque<TreeNode> stack_help = new ArrayDeque<>();
    stack_help.push(root);
    while (!stack_help.isEmpty()) {
        TreeNode cur = stack_help.pop();
        stack_main.push(cur);
        if (cur.left != null) {
            stack_help.push(cur.left);
        }
        if (cur.right != null) {
            stack_help.push(cur.right);
        }
    }

    while (!stack_main.isEmpty()) {
        TreeNode node = stack_main.pop();
        System.out.print(node.val + " ");
    }
}
```

👍二叉搜索树常用模板

```java
void BST(TreeNode root, int target) {
    if (root.val == target)
        // 找到目标，做点什么
    if (root.val < target) 
        BST(root.right, target);
    if (root.val > target)
        BST(root.left, target);
}
```



### 94、二叉树的中序遍历

**题目链接**：[二叉树的中序遍历](https://leetcode-cn.com/problems/binary-tree-inorder-traversal/)

**解题思路**：

1、递归

2、迭代

**代码实现**：

递归

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {

    List<Integer> list = new ArrayList<>();
    public List<Integer> inorderTraversal(TreeNode root) {
        inorder(root);
        return list;
    }

    private void inorder(TreeNode root) {
        if (root == null) {
            return;
        }
        inorder(root.left);
        list.add(root.val);
        inorder(root.right);
    }
}
```

迭代

```java
class Solution {
    
    public List<Integer> inorderTraversal(TreeNode root) {
        List<Integer> list = new ArrayList<>();
        Deque<TreeNode> stack = new ArrayDeque<>();
        TreeNode cur = root;
        while (cur != null || !stack.isEmpty()) {
            if (cur != null) {
                stack.push(cur);
                cur = cur.left;
            }else{
                TreeNode node = stack.pop();
                list.add(node.val);
                cur = node.right;
            }
        }
        return list;
    }

}
```



### 96、不同的二叉搜索树（不熟练）

**题目链接**：[不同的二叉搜索树](https://leetcode-cn.com/problems/unique-binary-search-trees/)

**解题思路**：

1. 穷举每一个节点作为根节点的情况，它的左右子树使用递归去计算
2. 如何计算该根节点的所能构造的二叉搜索树的个数？固定根节点，计算左右节点的组合数，左右节点的组合数相乘就是该节点的个数

注意：空子树也是一种情况，所以要技术为1

**代码实现**：

未优化版本,超时

```java
class Solution {
    public int numTrees(int n) {
        return count(1, n);
    }

    private int count(int low, int high) {
        //表示一个空区间，也就是空节点，也是一种情况，返回1
        if (low>high) return 1;

        int res = 0;
        for (int i = low; i <= high; i++) {
            //先算出左右节点有几种组合
            int left = count(low, i - 1);
            int right = count(i + 1, high);
            //左右节点的组合相乘就是BST的总数
            res += left*right;
        }

        return res;
    }
}
```

存在重叠子问题，可以使用备忘录进行优化

```java
class Solution {

    //备忘录
    int  [][] meno;
    public int numTrees(int n) {
        meno = new int[n + 1][n + 1];
        return count(1, n);
    }

    private int count(int low, int high) {
        //表示一个空区间，也就是空节点，也是一种情况，返回1
        if (low>high) return 1;

        if (meno[low][high]!=0) {
            return meno[low][high];
        }

        int res = 0;
        for (int i = low; i <= high; i++) {
            //先算出左右节点有几种组合
            int left = count(low, i - 1);
            int right = count(i + 1, high);
            //左右节点的组合相乘就是BST的总数
            res += left*right;
        }
        //把结果存储起来
        meno[low][high] = res;

        return res;
    }
}
```

### 98、验证二叉搜索树（不熟练）

**题目链接**：[验证二叉搜索树](https://leetcode-cn.com/problems/validate-binary-search-tree/)

**解题思路**：

二叉搜索树的根节点的值大于所有左子树的值，小于所有右子树的值。

明确了根节点要干什么了，就是跟左右子树比较。

注意：根节点不只是比较自己的左右节点，而是比较所有左右节点。根节点就是左子树的最大值，右子树的最小值，可以通过添加两个参数（最大值和最小值）

**代码实现**：

错误代码，只比较了自己的左右子树节点

```java
boolean isValidBST(TreeNode root) {
    if (root == null) return true;
    if (root.left != null && root.val <= root.left.val)
        return false;
    if (root.right != null && root.val >= root.right.val)
        return false;

    return isValidBST(root.left)
        && isValidBST(root.right);
}
```

正确代码，通过添加参数完成最小值和最大值的记录

```java
class Solution {
    public boolean isValidBST(TreeNode root) {
        return isValidBST(root, null, null);
    }

    private boolean isValidBST(TreeNode root, TreeNode min, TreeNode max) {
        if (root == null) {
            return true;
        }
        if (min != null && root.val <= min.val) {
            return false;
        }
        if (max != null && root.val >= max.val) {
            return false;
        }
        return isValidBST(root.left, min, root) && isValidBST(root.right, root, max);
    }
}
```



### 101、对称二叉树（不熟练）

**题目链接**：[对称二叉树](https://leetcode-cn.com/problems/symmetric-tree/)

**解题思路**：

错误思路：如果只是按照前序遍历，只是比较根节点的左右子树是否相同，那么就会出现只比较到了左子树、右子树内部是否对称，而没有比较到外部是否对称。比如left.right要和right.left比较，left.left要和right.right比较

**代码实现**：

```java
class Solution {
    public boolean isSymmetric(TreeNode root) {
        if(root==null) return true;
        //检查两颗子树是否相同
        return isSymmetric(root.left, root.right);
    }

    private boolean isSymmetric(TreeNode left, TreeNode right) {
        if(left==null || right==null) return left==right;
        //两个根节点需要相同
        if(left.val!=right.val) return false;
        //左右节点需要对称相同
        return isSymmetric(left.right,right.left) && isSymmetric(left.left,right.right);
    }
}
```



### 103、二叉树的锯齿形层次遍历

**题目链接**：[二叉树的锯齿层次遍历](https://leetcode-cn.com/problems/binary-tree-zigzag-level-order-traversal/)

**解题思路**：思路依旧是层次遍历框架。唯一不同的是需要定义一个变量，表示是否要反向遍历

**代码实现**：

```java
class Solution {
    public List<List<Integer>> zigzagLevelOrder(TreeNode root) {
        List<List<Integer>> res = new ArrayList<>();
        if(root==null) return res;
        Queue<TreeNode> q = new LinkedList<>();
        q.offer(root);
        //定义一个变量，表示是否要反向
        boolean flag = true;
        while(!q.isEmpty()){
            LinkedList<Integer> list = new LinkedList<>();
            int size = q.size();
            for(int i=0;i<size;i++){
                TreeNode node = q.poll();
                if(flag){//正向遍历
                    list.addLast(node.val);
                }else{//反向遍历
                    list.addFirst(node.val);
                }
                if(node.left!=null){
                    q.offer(node.left);
                }
                if(node.right!=null){
                    q.offer(node.right);
                }
            }
            //别忘了改变变量值
            flag = !flag;
            res.add(list);
        }
        return res;
    }
}
```





### 104、二叉树的最大深度

**题目链接**：[二叉树的最大深度](https://leetcode-cn.com/problems/maximum-depth-of-binary-tree/)

**解题思路**：

1、后序遍历

分别求出左右子树最大的深度，然后比较左右子树，取出最大的深度加1

2、层序遍历

一层一层的遍历，每遍历一层，深度加1

**代码实现**：

递归(后序遍历)

```java
class Solution {

    public int maxDepth(TreeNode root) {
        if(root == null){
            return 0;
        }
        int leftDepth = maxDepth(root.left);
        int rightDepth = maxDepth(root.right);
        return Math.max(leftDepth,rightDepth) + 1;
    }
}
```

广度遍历

```java
class Solution {
    public int maxDepth(TreeNode root) {
        if(root==null) {
            return 0;
        }
        int deep = 0;
        Queue<TreeNode> queue = new LinkedList<>();
        queue.offer(root);
        while(!queue.isEmpty()){
            int size = queue.size();
            for(int i=0;i<size;i++){
                TreeNode node = queue.poll();
                if (node.left != null) {
                    queue.offer(node.left);
                }
                if (node.right != null) {
                    queue.offer(node.right);
                }
            }
            deep++;
        }

        return deep;
    }
}

```



### 105、从前序与中序遍历序列中构造二叉树

**题目链接**：[从前序与中序遍历序列中构造二叉树](https://leetcode-cn.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/)

**解题思路**：



**代码实现**：

```java
class Solution {

    HashMap<Integer,Integer> map;
    public TreeNode buildTree(int[] preorder, int[] inorder) {
        map = new HashMap<>();
        for(int i=0;i<inorder.length;i++){
            map.put(inorder[i],i);
        }
        return buildTree(preorder,0,preorder.length-1,
                        inorder,0,inorder.length-1);
    }

    private TreeNode buildTree(int[] preorder,int pre_start,int pre_end,int[] inorder,int in_start,int in_end){
        if(pre_start>pre_end){
            return null;
        }
        TreeNode root = new TreeNode(preorder[pre_start]);
        //根节点的值在中序遍历数组中的索引
        int in_index = map.get(preorder[pre_start]);
        int left_size = in_index - in_start;
        root.left = buildTree(preorder,pre_start+1,pre_start+left_size,inorder,in_start,in_index-1);
        root.right = buildTree(preorder,pre_start + left_size +1,pre_end,inorder,in_index+1,in_end);
        return root;
    }
                             
}
```

### 112、路径总和

**题目链接**：[路径总和](https://leetcode-cn.com/problems/path-sum/)

**解题思路**：

1. 可以使用回溯法的思路框架进行编写代码
2. 使用前序遍历框架，明白当前节点要做什么

**代码实现**：

```java
class Solution {

    boolean res = false;
    public boolean hasPathSum(TreeNode root, int targetSum) {
        LinkedList<Integer> list = new LinkedList<>();
        traverse(root,targetSum,list);
        return res;
    }

    //回溯法
    private void traverse(TreeNode root,int targetSum,LinkedList<Integer> list){
        if(root==null) return;
        //添加节点
        list.add(root.val);
        targetSum -= root.val;
        //判断是否符合条件，如果符合，直接返回
        if(targetSum==0 && root.left==null && root.right==null){
            res = true;
            return;
        }

        traverse(root.left,targetSum,list);
        traverse(root.right,targetSum,list);
        //还原节点
        int value = list.removeLast();
        targetSum += value;
    }
}
```



### 113、路径总和2

**题目链接**：[路径总和2](https://leetcode-cn.com/problems/path-sum-ii/)

**解题思路**：

1. 可以使用回溯法的思路框架进行编写代码
2. 使用前序遍历框架，明白当前节点要做什么

**代码实现**：

```java
class Solution {

    List<List<Integer>> res = new ArrayList<>();
    public List<List<Integer>> pathSum(TreeNode root, int targetSum) {
        LinkedList<Integer> list = new LinkedList<>();
        traverse(root,targetSum,list);
        return res;
    }

    //回溯法
    private void traverse(TreeNode root,int targetSum,LinkedList<Integer> list){
        if(root==null) return;
        //添加节点
        list.add(root.val);
        targetSum -= root.val;
        //判断是否满足条件
        if(targetSum==0 && root.left==null && root.right==null){
            res.add(new ArrayList<>(list));
        }
        traverse(root.left,targetSum,list);
        traverse(root.right,targetSum,list);
        //还原节点
        int value = list.removeLast();
        targetSum += value;
    }
}
```



### 114、二叉树展开为链表

**题目链接**：[二叉树展开为链表](https://leetcode-cn.com/problems/flatten-binary-tree-to-linked-list/)

**解题思路**：

1. 后序遍历，先左右子树展开铺平，再考虑根节点
2. 明确根节点要做什么；先将左子树作为根节点的右子树；再将根节点的右子树拼接到左子树的后面

**代码实现**：

```java
class Solution {
    public void flatten(TreeNode root) {
        if(root==null) return;
        //后序遍历，先将左右子树拉平
        flatten(root.left);
        flatten(root.right);
        //明确根节点要做什么
        //1.将左子树作为根节点的右子树
        TreeNode tmp = root.right;
        root.right = root.left;
        root.left = null;
        //2.将根节点的右子树拼接到左子树的后面
        TreeNode p = root;
        while(p.right!=null){
            p = p.right;
        }
        p.right = tmp;
    }
}
```



### 116、填充每个节点的下一个右侧节点指针

**题目链接**：[填充每个节点的下一个右侧节点指针](https://leetcode-cn.com/problems/populating-next-right-pointers-in-each-node/)

**解题思路**：

二叉树的难点，如何把题目的要求细化到每一个节点做什么

**代码实现**：

错误实现。不属于同一个父节点，那么按照这段代码的逻辑，它俩就没办法被穿起来，这是不符合题意的

```java
class Solution {
    public Node connect(Node root) {
        if(root == null || root.left==null) return root;
        //前序遍历
        root.left.next = root.right;
        connect(root.left);
        connect(root.right);
        return root;
    }
}
```

正确实现。如果一个节点办不到，那就增加一个节点。即通过增加函数的方式

```java
class Solution {
    public Node connect(Node root) {
        if(root==null) return root;
        connectNode(root.left,root.right);
        return root;
    }
    private void connectNode(Node node1,Node node2){
        if(node1==null || node2 == null) return;
        node1.next = node2;
        connectNode(node1.left,node1.right);
        connectNode(node2.left,node2.right);
        connectNode(node1.right,node2.left);
    }
}
```



### 124、二叉树中的最大路径和（不熟练）

**题目链接**：[二叉树中的最大路径和](https://leetcode-cn.com/problems/binary-tree-maximum-path-sum/)

**解题思路**：

后序遍历。求出根节点的最大单边路径和

后序代码的时候去更新最大路径和

注意：要重新定义一个递归函数，表示从根节点起的最大单边路径。

**代码实现**：

```java
class Solution {

    int res = Integer.MIN_VALUE;
    public int maxPathSum(TreeNode root) {
        if(root==null) return 0;
        postOrder(root);
        return res;
    }

    //定义：从根节点起的最大单边路径和
    private int postOrder(TreeNode root){
        if(root==null) return 0;
        //后序遍历
        //注意如果leftSum或者rightSum小于0时，则返回0
        int leftSum = Math.max(0,postOrder(root.left));
        int rightSum = Math.max(0,postOrder(root.right));
        //更新最大路径和
        int pathSum = root.val + leftSum + rightSum;
        res = Math.max(res,pathSum);
        //实现定义，每个节点只能经过一次，所以左右子树只能选出一个
        return Math.max(leftSum,rightSum) + root.val;
    }
}
```



### 129、求根节点到叶子节点数字之和

**题目链接**：[求根节点到叶子节点数字之和](https://leetcode-cn.com/problems/sum-root-to-leaf-numbers/)

**解题思路**：

1.使用前序遍历即可得到结果

2.像这种字符串拼接得到数字结果的可以考虑使用StringBuilder作为容器

**代码实现**：

```java
class Solution {

    StringBuilder path = new StringBuilder();
    int res = 0;
    public int sumNumbers(TreeNode root) {
        //遍历二叉树就能出结果
        traverse(root);
        return res;
    }

    private void traverse(TreeNode root){
        if(root==null) return;
        //前序遍历
        path.append(root.val);
        //判断是否可以加到结果集里
        if(root.left==null && root.right==null){
            res += Integer.parseInt(path.toString());
        }
        traverse(root.left);
        traverse(root.right);
        path.deleteCharAt(path.length()-1);
    }
}
```





### 144、二叉树的前序遍历

**题目链接**：[二叉树的前序遍历](https://leetcode-cn.com/problems/binary-tree-preorder-traversal/)

**解题思路**：

1、递归

2、迭代

**代码实现**：

递归

```java
class Solution {

    List<Integer> list = new ArrayList<>();
    public List<Integer> preorderTraversal(TreeNode root) {
        travel(root);
        return list;
    }
    private void travel(TreeNode root){
        if(root==null) return;
        list.add(root.val);
        travel(root.left);
        travel(root.right);
    }
}
```

迭代

```java
class Solution {
    public List<Integer> preorderTraversal(TreeNode root) {
        List<Integer> list = new ArrayList<>();
        Deque<TreeNode> stack = new ArrayDeque<>();
        while(!stack.isEmpty() || root!=null){
            if(root!=null){
                list.add(root.val);
                stack.push(root);
                root = root.left;
            }else{
                TreeNode node = stack.pop();
                root = node.right;
            }
        }
        return list;
    }
}
```



### 226、翻转二叉树

**题目链接**：[翻转二叉树](https://leetcode-cn.com/problems/invert-binary-tree/)

**解题思路**：

1、前序遍历

直接交换左右节点

2、后序遍历

先交换了左右子树，再交换根节点的左右节点

**代码实现**：

前序遍历

```java
class Solution {
    public TreeNode invertTree(TreeNode root) {
        if(root==null) return null;
        TreeNode tmp = root.left;
        root.left = root.right;
        root.right = tmp;
        invertTree(root.left);
        invertTree(root.right);
        return root;
    }
}
```

后序遍历

```java
class Solution {
    public TreeNode invertTree(TreeNode root) {
        if(root==null) return null;
        TreeNode left = invertTree(root.left);
        TreeNode right = invertTree(root.right);
        root.left = right;
        root.right = left;
        return root;
    }
}
```



### 230、二叉搜索树中的第k小的元素

**题目链接**：[二叉搜索树中的第k小的元素](https://leetcode-cn.com/problems/kth-smallest-element-in-a-bst/)

**解题思路**：

利用BST的中序遍历的特性（中序遍历是一个有序的数组）

**代码实现**：

```java
class Solution {

    //存储结果
    int res = 0;
    //记录顺序
    int rank = 0;

    public int kthSmallest(TreeNode root, int k) {
       travel(root,k);
       return res;
    }

    private void travel(TreeNode root,int k){
        if(root==null) return;
        travel(root.left,k);
        if(++rank == k){
            res = root.val;
            return;
        }
        travel(root.right,k);
        
    }
}
```



### 236、二叉树的最近公共祖先（困难，不熟练）

**题目链接**：[二叉树的最近公共祖先](https://leetcode-cn.com/problems/lowest-common-ancestor-of-a-binary-tree/)

**解题思路**：

首先清楚递归函数三个问题

```
1.明白函数的定义
2.明白函数的参数变量
3.得到递归结果该干什么
```

1.函数的定义：就是返回节点p和节点q最近公共节点。

2.参数变量只能是root，他才是会变化的，可以缩小规模的。那什么时候才结束递归呢？

情况一：如果root为null，那肯定得返回null

情况二：如果root为p或者root为q，那么不管另一方是否为空，都是返回root

3.得到递归结果，也分为以下情况。

情况一：如果p、q都在以root为根的树中，那么p、q一定分别是left和right

情况二：如果p、q都不在以root为根的树中，那么返回null

情况三：如果有一方不在树中，则直接返回不为null的一方

**代码实现**：

```java
class Solution {
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        
        //递归结束条件，明白函数中参数的变量是哪个
        if(root == null) return null;
        if(root==p || root==q) return root;

        //递归函数调用，一定要明白函数的定义
        TreeNode left = lowestCommonAncestor(root.left,p,q);
        TreeNode right = lowestCommonAncestor(root.right,p,q);

        //得到了递归调用的结果
        //1.情况一：如果p、q都在以root为根的树中，那么p、q一定分别是left和right
        if(left!=null && right!=null) return root;
        //2.情况二：如果p、q都不在以root为根的树中，那么返回null
        if(left==null && right==null) return null;
        //3.情况三：如果有一方不在树中，则直接返回不为null的一方
        return left==null?right:left;
    }
}
```





### 450、删除二叉搜索树中的节点（不熟练）

**题目链接**：[删除二叉搜索树中的节点](https://leetcode-cn.com/problems/delete-node-in-a-bst/)

**解题思路**：

情况1，删除的节点没有左右节点，直接删除

情况2，删除的节点有其中之一的左右节点，返回它的子节点

情况3，删除的节点有左右节点，找到删除节点的右子树的最小节点作为根节点

**代码实现**：

```java
class Solution {
    public TreeNode deleteNode(TreeNode root, int key) {
        if(root==null) return null;
        if(root.val==key){
            //情况1和情况2包含子节点为空或者只有一个节点为空的情况
            if(root.left==null) return root.right;
            if(root.right==null) return root.left;
            //情况3，左右节点都不为空，找到右子树的最小节点，作为删除节点位置的节点
            //获取右子树最小节点
            TreeNode rightMinNode = getRightMinNode(root.right);
            //删除右子树的最小节点
            root.right = deleteNode(root.right,rightMinNode.val);
            //用右子树最小节点替换root节点
            rightMinNode.left = root.left;
            rightMinNode.right = root.right;
            root = rightMinNode;
        }
        if(root.val > key) root.left = deleteNode(root.left,key);
        if(root.val < key) root.right = deleteNode(root.right,key);
        return root;
    }

    private TreeNode getRightMinNode(TreeNode node){
        while(node.left!=null) node = node.left;
        return node;
    }
}
```



### 543、二叉树的直径

**题目链接**：[二叉树的直径](https://leetcode-cn.com/problems/diameter-of-binary-tree/)

**解题思路**：

后序遍历，知道左右子树的最大直径，就可以求出当前节点的直径，当前节点的直径=左子树最大深度+右子树最大深度。

就是在104题中，添加一行代码就可以实现

**代码实现**：

```java
class Solution {

    int maxValue = 0;
    public int diameterOfBinaryTree(TreeNode root) {
        maxDepth(root);
        return maxValue;
    }

    private int maxDepth(TreeNode root){
        if(root==null) return 0;
        int left = maxDepth(root.left);
        int right = maxDepth(root.right);
        maxValue = Math.max(maxValue,left+right);//添加的代码
        return Math.max(left,right) + 1;
    }
}
```



### 652、寻找重复的子树（不熟练）

**题目链接**：[寻找重复的子树](https://leetcode-cn.com/problems/find-duplicate-subtrees/submissions/)

**解题思路**：

需要知道两个点：

1、以我为根节点的二叉树长啥样

2、以其他节点为根的子树长啥样



如何知道自己长啥样？可以通过序列化二叉树，把结果存储起来（使用HashMap）。既然要知道自己长啥样，就得知道左右子树长啥样，所以得通过后序遍历。          

**代码实现**：

```java
class Solution {

    //记录子树是否重复
    HashMap<String,Integer> map = new HashMap<>();

    //存储结果
    List<TreeNode> res = new ArrayList<>();
    public List<TreeNode> findDuplicateSubtrees(TreeNode root) {
        travel(root);
        return res;
    }

    private String travel(TreeNode root){
        if(root==null){
            return "#";
        }
        String left = travel(root.left);
        String right = travel(root.right);
        String subTree = left + "," + right + "," + root.val;
        int count = map.getOrDefault(subTree,0);
        //不管重复多少次，都只存储一次
        if(count==1){
            res.add(root);
        }
        map.put(subTree,count+1);
        return subTree;
    }
}
```



### 654、最大二叉树

**题目链接**：[最大二叉树](https://leetcode-cn.com/problems/maximum-binary-tree/)

**解题思路**：

明确根节点要做什么。从数组中选择最大值构建节点，然后递归遍历。显然采用的是先序遍历

需要通过添加参数来数组下标

**代码实现**：

```java
class Solution {
    public TreeNode constructMaximumBinaryTree(int[] nums) {
        return constructMaximumBinaryTree(nums,0,nums.length-1);
    }

    private TreeNode constructMaximumBinaryTree(int[] nums,int low,int high){
        if(low>high) return null;
        //找出最大值的索引
        int maxIndex = findMaxIndex(nums,low,high);
        int maxValue = nums[maxIndex];
        //构建根节点
        TreeNode root = new TreeNode(maxValue);
        root.left = constructMaximumBinaryTree(nums,low,maxIndex-1);
        root.right = constructMaximumBinaryTree(nums,maxIndex+1,high);
        return root;
    }

    private int findMaxIndex(int[] nums,int low,int high){
        int maxValue = Integer.MIN_VALUE;
        int maxIndex = -1;
        for(int i=low;i<=high;i++){
            if(nums[i]>maxValue){
                maxValue = nums[i];
                maxIndex = i;
            }
        }
        return maxIndex;
    }
}
```



### 700、二叉搜索树中的搜索

**题目链接**：[二叉搜索树中的搜索](https://leetcode-cn.com/problems/search-in-a-binary-search-tree/)

**解题思路**：

根据二叉搜索树的二分查找特性

**代码实现**：

```java
class Solution {
    public TreeNode searchBST(TreeNode root, int val) {
        if(root==null) return null;
        if(root.val>val) return searchBST(root.left,val);
        else if(root.val<val) return searchBST(root.right,val);
        else return root;
    }
}
```



### 701、二叉搜索树中的插入操作

**题目链接**：[二叉搜索树中的插入操作](https://leetcode-cn.com/problems/insert-into-a-binary-search-tree/)

**解题思路**：

根据二叉搜索树的二分查找特性

**代码实现**：

```java
class Solution {
    public TreeNode insertIntoBST(TreeNode root, int val) {
        if(root==null) return new TreeNode(val);
        if(root.val>val) root.left = insertIntoBST(root.left,val);
        if(root.val<val) root.right = insertIntoBST(root.right,val);
        return root;
    }
}
```





## 六、字符串

👍解决回文串的核心是双指针



### 5、最长回文子串

**题目链接**：[最长回文子串](https://leetcode-cn.com/problems/longest-palindromic-substring/)

**解题思路**：

中心扩展法。把每一个字符都当作中心点去扩展，判断是否是回文子串。因为字符串长度为奇数和偶数是两种情况，所以干脆两者都一起判断，最后取最长的字符串。

**代码实现**：

```java
class Solution {
    public String longestPalindrome(String s) {
        String res = "";
        for(int i=0;i<s.length();i++){
            //以s[i]为中心的最长回文子串
            String s1 = palindrome(s,i,i);
            //以s[i]和s[i+1]为中心的最长回文子串
            String s2 = palindrome(s,i,i+1);
            res = res.length()>s1.length()?res:s1;
            res = res.length()>s2.length()?res:s2;
        }
        return res;
    }

    //取出回文串
    private String palindrome(String s,int left,int right){
        //防止索引越界
        while(left>=0 && right<=s.length()-1 && s.charAt(left)==s.charAt(right)){
            left--;
            right++;
        }
        //substring包头不包尾
        return s.substring(left+1,right);
    }
}
```

### 8、字符串转换整数

**题目链接**：[字符串转换整数](https://leetcode-cn.com/problems/string-to-integer-atoi/)

**解题思路**：

1.定义索引，也可以叫指针，index

2.第一步判断前面的是否空格，通过移动指针来判断；注意判断极端情况，本来就是空格字符的情况

3.判断符号。定义sign表示正负

4.接下来就是判断是否是数字，若不是就停止；定义res表示结果集，last_res用于记录上一次的结果集，用于判断是否溢出

5.最后返回带sign的res

注意：判断符号时，是否要对index++的情况。只有碰到'+'或者'-'才需要修改index，否则不用动

**代码实现**：

```java
class Solution {
    public int myAtoi(String s) {
        int len = s.length();
        char[] c = s.toCharArray();
        //1.去空格
        int index = 0;
        while(index<len && c[index]==' ') index++;
        //排除极端情况，s="   "
        if(index==len) return 0;
        //2.设置符号，是正号还是负号
        int sign = 1;
        char first_char = c[index];
        if(first_char=='-'){
            sign = -1;
            index++;
        }else if(first_char=='+'){
            index++;
        }
        //3.记录结果
        int res = 0;
        int last_res = 0;//记录上一次结果，用于判断是否溢出
        while(index < len){
            char tmp = c[index];
            //判断是否是数字
            if(tmp >'9' || tmp < '0') break;
            int val = tmp - '0';
            last_res = res;
            res = res * 10 + val;
            //如果不相等就是溢出
            if(last_res != res/10){
                return sign==1?Integer.MAX_VALUE:Integer.MIN_VALUE;
            }
            index++;
        }
        return res * sign;
    }
}
```



### 43、字符串相乘

**题目链接**：[字符串相乘](https://leetcode-cn.com/problems/multiply-strings/)

**解题思路**：

1.按照乘法运算，从后往前计算

2.存储位置是 i + j和i+j+1，这个需要弄明白

**代码实现**：

```java
class Solution {
    public String multiply(String num1, String num2) {
        int m = num1.length();
        int n = num2.length();
        //存储结果的数组
        int[] res = new int[m+n];
        char[] s1 = num1.toCharArray();
        char[] s2 = num2.toCharArray();
        //从后面往前计算，就和小学乘法一样的过程
        for(int i=m-1;i>=0;i--){
            for(int j=n-1;j>=0;j--){
                int mul = (s1[i]-'0') * (s2[j]-'0');
                //结果存储的位置
                int p1 = i + j;
                int p2 = i + j + 1;
                //记得加上之前这位数上的数值
                int sum = mul + res[p2];
                res[p2] = sum%10;
                res[p1] += sum/10; //别忘了 + 
            }
        }
        //去除前面的0数字
        int k = 0;
        while(k<res.length && res[k]==0) k++;
        //特殊情况，res[]数组全部为0的情况
        if(k==res.length){
            return new String("0");
        }
        //依次将数组的数字转化为string类型
        StringBuilder sb = new StringBuilder();
        while(k<res.length){
            sb.append(res[k]);
            k++;
        }
        return sb.toString();
    }
}
```





### 415、字符串相加

**题目链接**：[字符串相加](https://leetcode-cn.com/problems/add-strings/)

**解题思路**：

1. 使用双指针，分别指向两个字符串的最后一位索引
2. 从后往前相加。注意char字符减'0'是一个整数。注意进位的表示
3. 最后面对所得字符串进行反转就是所得结果了

**代码实现**：

```java
class Solution {
    public String addStrings(String num1, String num2) {
        
        //使用双指针
        int i = num1.length()-1;
        int j = num2.length()-1;
        //进位
        int carry = 0;
        //存储结果
        StringBuilder sb = new StringBuilder();
        while(i>=0 || j>=0){
            int a = (i>=0)?num1.charAt(i)-'0':0;
            int b = (j>=0)?num2.charAt(j)-'0':0;
            //按照加法原则，从后往前加
            int tmp = a + b + carry;
            carry = tmp/10;
            sb.append(tmp%10);
            //别忘了减操作
            i--;
            j--;
        }
        //最后判断一下进位是否为1，如果是要加上
        if(carry==1){
            sb.append(carry);
        }
        //最后对所得的字符串进行反转
        return sb.reverse().toString();
    }
}
```



## 七、数组

**主对角线反转矩阵(按照左上到右下的对角线进行镜像对称)**

![主对角线](D:\github\MyKnowledgeRepository\picture\主对角线镜像.jpg)

```java
public void rotate(int[][] matrix) {
    int n = matrix.length;
    // 先沿对角线镜像对称二维矩阵
    for (int i = 0; i < n; i++) {
        for (int j = i; j < n; j++) {
            // swap(matrix[i][j], matrix[j][i]);
            int temp = matrix[i][j];
            matrix[i][j] = matrix[j][i];
            matrix[j][i] = temp;
        }
    }
}
```

**反对角线反转矩阵**

![反对角线](D:\github\MyKnowledgeRepository\picture\反对角线镜像.jpg)

```java
public void rotate(int[][] matrix) {
    int n = matrix.length;
    // 沿左下到右上的对角线镜像对称二维矩阵
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < n - i; j++) {
            // swap(matrix[i][j], matrix[n-j-1][n-i-1])
            int temp = matrix[i][j];
            matrix[i][j] = matrix[n - j - 1][n - i - 1];
            matrix[n - j - 1][n - i - 1] = temp;
        }
    }
}
```



**Arrays.sort对二维数组进行排序**

```java
//第一位相同，比较第二位
int[][] nums = new int[][]{{1,3},{1,2},{4,5},{3,7}};
Arrays.sort(nums, new Comparator<int[]>() {
  public int compare(int[] a, int[] b){
    if(a[0]==b[0]){
      return a[1] - b[1];
    }else {
      return a[0] - b[0];
    }
  }
});

//只根据第一位进行排序
Arrays.sort(nums,(v1,v2) -> v1[0]-v2[0])
```



### 26、删除有序数组的重复项

**题目链接**：[删除有序数组的重复项](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-array/)

**算法思路**：

1.使用快慢指针，快指针往前找，找到跟慢指针不相同的值，将慢指针加1，并且慢指针的值设为快指针的值

**代码实现**：

```java
class Solution {
    public int removeDuplicates(int[] nums) {
        //使用快慢指针
        int slow = 0;
        int fast = 0;
        while(fast<nums.length){
            //找到跟慢指针不相等的数据
            if(nums[fast]!=nums[slow]){
                //慢指针加1并且等于快指针的值
                nums[++slow] = nums[fast];
            }
            //快指针继续往前推进
            fast++;
        }
        return slow+1;
    }
}
```





### 31、下一个排列

**题目链接**：[下一个排列](https://leetcode-cn.com/problems/next-permutation/)

**解题思路**：

1. 定义 i=nums.length-1，从后向前寻找第一个满足nums[i]>nums[i-1]的数，其中nums[i-1]就是要交换的数
2. 对 i 到 nums.length范围的数进行排序
3. 排序后，在 i 到 nums.length中寻找第一个nuns[j] > nums[i-1]的数，其中nums[j] 就是要交换的数
4. 交换nums[i-1] 和 nums[j] ，返回

注意：注意是取nums[i]还是取nums[i-1]

**代码实现**：

```java
class Solution {
    public void nextPermutation(int[] nums) {

        int len = nums.length;
        for(int i=len-1;i>0;i--){
          	//1.从后往前寻找第一个nums[i]>nums[i-1]的数
            if (nums[i] > nums[i - 1]) {
                //i后面的元素排序
                Arrays.sort(nums,i,len);
                //在后面元素中查找第一个大于nums[i-1]的元素进行交换
                for(int j=i;i<len;j++){
                    if(nums[j]>nums[i-1]){
                        int tmp = nums[i-1];
                        nums[i-1] = nums[j];
                        nums[j] = tmp;
                        return;
                    }
                }
            }
        }
        Arrays.sort(nums);
        return;
    }
}
```





### 48、旋转图像

**题目链接**：[旋转图像](https://leetcode-cn.com/problems/rotate-image/)

**解题思路**：

先沿主对角线反转矩阵，然后将矩阵的每一行进行反转，就可以实现按顺时针旋转图像。

**代码实现**：

```java
class Solution {
    public void rotate(int[][] matrix) {
        int n = matrix.length;
        //沿对角线翻转矩阵
        for (int i = 0; i < n; i++) {
            for (int j = i; j < n; j++) {
                int tmp = matrix[i][j];
                matrix[i][j] = matrix[j][i];
                matrix[j][i] = tmp;
            }
        }
        //将矩阵的每一行进行反转
        for (int[] row : matrix) {
            reverse(row);
        }
    }

    //反转一维数组
    private void reverse(int[] row) {
        int i = 0;
        int j = row.length - 1;
        while (i < j) {
            int tmp = row[i];
            row[i] = row[j];
            row[j] = tmp;
            i++;
            j--;
        }
    }
}

```



### 54、螺旋矩阵

**题目链接**：[螺旋矩阵](https://leetcode-cn.com/problems/spiral-matrix/)

**算法思路**：

1.定义上下左右边界，按照上右下左的顺序进行遍历

2.每遍历完一行或者一列元素，都要进行边界的缩小。每次遍历前都要先判断边界是否合法

**代码实现**：

```java
class Solution {
    public List<Integer> spiralOrder(int[][] matrix) {
        ArrayList<Integer> res = new ArrayList<>();
        int m = matrix.length;
        int n = matrix[0].length;
        //四个边界
        int up = 0;
        int down = m-1;
        int left = 0;
        int right = n-1;
        while(res.size() < m * n){
            //遍历上方
            if(up<=down){
                for(int i=left;i<=right;i++){
                    res.add(matrix[up][i]);
                }
                //遍历完，上边界下移
                up++;
            }
            //遍历右方
            if(left<=right){
                for(int i=up;i<=down;i++){
                    res.add(matrix[i][right]);
                }
                //遍历完，右边界左移
                right--;
            }
            //遍历下方
            if(up<=down){
                for(int i=right;i>=left;i--){
                    res.add(matrix[down][i]);
                }
                //遍历完，下边界上移
                down--;
            }
            //遍历左方
            if(left<=right){
                for(int i=down;i>=up;i--){
                    res.add(matrix[i][left]);
                }
                //遍历完，左边界右移
                left++;
            }
        }
        return res; 
    }
}
```

### 59、螺旋矩阵2

**题目链接**：[螺旋矩阵2](https://leetcode-cn.com/problems/spiral-matrix-ii/)

**解题思路**：跟上题一样的思路

**代码实现**：

```java
class Solution {
    public int[][] generateMatrix(int n) {
        int[][] matrix = new int[n][n];
    int upper_bound = 0, lower_bound = n - 1;
    int left_bound = 0, right_bound = n - 1;
    // 需要填入矩阵的数字
    int num = 1;
    
    while (num <= n * n) {
        if (upper_bound <= lower_bound) {
            // 在顶部从左向右遍历
            for (int j = left_bound; j <= right_bound; j++) {
                matrix[upper_bound][j] = num++;
            }
            // 上边界下移
            upper_bound++;
        }
        
        if (left_bound <= right_bound) {
            // 在右侧从上向下遍历
            for (int i = upper_bound; i <= lower_bound; i++) {
                matrix[i][right_bound] = num++;
            }
            // 右边界左移
            right_bound--;
        }
        
        if (upper_bound <= lower_bound) {
            // 在底部从右向左遍历
            for (int j = right_bound; j >= left_bound; j--) {
                matrix[lower_bound][j] = num++;
            }
            // 下边界上移
            lower_bound--;
        }
        
        if (left_bound <= right_bound) {
            // 在左侧从下向上遍历
            for (int i = lower_bound; i >= upper_bound; i--) {
                matrix[i][left_bound] = num++;
            }
            // 左边界右移
            left_bound++;
        }
    }
    return matrix;
    }
}
```



### 56、合并区间

**题目链接**：[合并区间](https://leetcode-cn.com/problems/merge-intervals/)

**算法思路**：

1、先对二维数组排序，根据开始区间进行排序

2、合并区间元素，根据当前结果集的结束元素和下一个区间的开始元素进行比较，若下一个区间的开始元素大于结果集的结束元素，则添加一个结果集区间，否则更新现有结果集的结束元素，取他们之间的最大值

**代码实现**：

```java
class Solution {
    public int[][] merge(int[][] intervals) {

        //结果数组
        int[][] res = new int[intervals.length][2];
        int index = -1;

        //根据开始区间对数组升序排序
        Arrays.sort(intervals,(v1,v2)->v1[0]-v2[0]);

        //合并区间
        //根据下一个区间的开始元素和结果集的结束元素比较
        for (int[] interval : intervals) {
            //若结果数组为空，或者下一个的开始元素大于结果集的结束元素,形成一个新的结果集
            if (index == -1 || interval[0] > res[index][1]) {
                res[++index] = interval;
            }else{//更新现在的结果集，取最大的区间
                res[index][1] = Math.max(res[index][1],interval[1]);
            }
        }

        return Arrays.copyOf(res, index + 1);
    }
}
```

```java
class Solution {
    public int[][] merge(int[][] intervals) {
        
        int len = intervals.length;
        //根据第一个元素的大小对数组排序
        Arrays.sort(intervals,(a,b)->a[0]-b[0]);
        //存储结果集，先初始化len
        int[][] res = new int[len][2];
        //结果集第一位数先存储数组第一位
        int index = 0;
        res[index] = intervals[0];
        //下标从1开始，以后的每一个
        for(int i=1;i<len;i++){
            //以后的每一个元素都和结果集比较
            //第一个元素比结果集的第二个元素还大，存储到新的结果集中
            if(intervals[i][0]>res[index][1]){
                index++;
                res[index] = intervals[i];
            }else{//否则更新结果集大小
                res[index][1] = Math.max(intervals[i][1],res[index][1]);
            }
        }
        return Arrays.copyOf(res,index+1);
    }
}
```



### 169、多数元素

**题目链接**：[多数元素](https://leetcode-cn.com/problems/majority-element/)

**解题思路**：

既然多数元素是指在数组中出现次数大于n/2次的元素，也就是求它众数，只要对数组排序，数组的中间的中位数就是众数

**代码实现**：

```java
class Solution {
    public int majorityElement(int[] nums) {
        Arrays.sort(nums);
        return nums[nums.length/2];
    }
}
```





## 八、堆

### 215、数组中的第K个最大元素

**题目链接**：[数组中的第K个最大元素](https://leetcode-cn.com/problems/kth-largest-element-in-an-array/)

**解题思路**：使用二叉堆。

**代码实现**：

```java
class Solution {
    public int findKthLargest(int[] nums, int k) {
        //小顶堆
        PriorityQueue<Integer> pq = new PriorityQueue<>();
        for(int x:nums){
            //每个元素都要过一遍二叉堆
            pq.offer(x);
            //堆中元素大于k时，删除堆顶元素
            if(pq.size()>k){
                pq.poll();
            }
        }
        // pq 中剩下的是 nums 中 k 个最大元素，
        // 堆顶是最小的那个，即第 k 个最大元素
        return pq.peek();
    }
}
```





### 295、数据流中的中位数

**题目链接**：[数据流中的中位数](https://leetcode-cn.com/problems/find-median-from-data-stream/)

**算法思路**：

初始化一个大顶堆和一个小顶堆

小顶堆：存储所有元素中较大的一半，堆顶存储的是其中最小的元素
大顶堆：存储所有元素中较小的一半，堆顶存储的是其中最大的元素
相当于把元素分为大小二半，取中位数时，取各自的堆顶的元素即可
如果num数值比小顶堆的堆顶元素大，则放在小顶堆
否则放在大顶堆

两个细节注意点：
当小顶堆的元素个数-大顶堆的元素个数>1时，需要平衡两者的数量。
当大顶堆的元素个数-小顶堆的元素个数>0时，需要平衡两者的数量。

平衡数量的原则：小顶堆的个数 >= 大顶堆的个数，并且不超过1

**代码实现**：

```java
class MedianFinder {

    PriorityQueue<Integer> minQueue;//小顶堆
    PriorityQueue<Integer> maxQueue;//大顶堆
    public MedianFinder() {
        minQueue = new PriorityQueue<>();
        maxQueue = new PriorityQueue<>((a,b)->(b-a));
    }
    
    public void addNum(int num) {
        if(minQueue.isEmpty() || minQueue.peek()<=num){
            minQueue.offer(num);
            //当小顶堆的元素个数-大顶堆的元素个数>1时，需要平衡两者的数量
            if(minQueue.size()-maxQueue.size()>1){
                maxQueue.offer(minQueue.poll());
            }
        }else{
            maxQueue.offer(num);
            //当大顶堆的元素个数-小顶堆的元素个数>0时，需要平衡两者的数量
            if(maxQueue.size()-minQueue.size()>0){
                minQueue.offer(maxQueue.poll());
            }
        }
    }
    
    public double findMedian() {
        int minSize = minQueue.size();
        int maxSize = maxQueue.size();
        if(minSize>maxSize){
            return minQueue.peek().doubleValue();
        }else{
            return (minQueue.peek()+maxQueue.peek())/2.0;
        }
    }
}

/**
 * Your MedianFinder object will be instantiated and called as such:
 * MedianFinder obj = new MedianFinder();
 * obj.addNum(num);
 * double param_2 = obj.findMedian();
 */
```



## 九、图

👍**图的表示**

图的基本表示

```java
// 邻接表
// graph[x] 存储 x 的所有邻居节点
List<Integer>[] graph;

// 邻接矩阵
// matrix[x][y] 记录 x 是否有一条指向 y 的边
boolean[][] matrix;
```



有向加权图

```java
// 邻接表
// graph[x] 存储 x 的所有邻居节点以及对应的权重
List<int[]>[] graph;

// 邻接矩阵
// matrix[x][y] 记录 x 指向 y 的边的权重，0 表示不相邻
int[][] matrix;
```





👍**图的遍历框架**

如果图包含环，遍历框架就要一个 `visited` 数组进行辅助

visited是标记是否访问过，以免重复访问。

onpath是做选择，是否选择这条路径。

```java
// 记录被遍历过的节点
boolean[] visited;
// 记录从起点到当前节点的路径
boolean[] onPath;

/* 图遍历框架 */
void traverse(Graph graph, int s) {
    if (visited[s]) return;
    // 经过节点 s，标记为已遍历
    visited[s] = true;
    // 做选择：标记节点 s 在路径上
    onPath[s] = true;
    for (int neighbor : graph.neighbors(s)) {
        traverse(graph, neighbor);
    }
    // 撤销选择：节点 s 离开路径
    onPath[s] = false;
}

```



### 207、课程表

**题目链接**：[课程表](https://leetcode-cn.com/problems/course-schedule/)

**解题思路**：看到依赖问题，首先把问题转换到有向图这种数据结构，只要图中存在环，那就说明有循环依赖。

首先将题目的输入转化成图，然后遍历图判断是否有环。常用的存储方式是使用邻接表

`List<Integer>[] graph`,graph[s]是邻接表，存储着所有s指向的节点。

**代码实现**：

```java
class Solution {

    boolean[] visted;
    boolean[] path;
    boolean hasCycle = false;
    public boolean canFinish(int numCourses, int[][] prerequisites) {
        List<Integer>[] graph = buildGraph(numCourses,prerequisites);
        visted = new boolean[numCourses];
        path = new boolean[numCourses];
        //遍历图中所有节点
        for(int i=0;i<numCourses;i++){
            traverse(graph,i);
        }
        //只要没有循环依赖就可以完成课程
        return !hasCycle;
    }

    //先构建有向图
    private List<Integer>[] buildGraph(int numCourses,int[][] prerequisites){
        List<Integer>[] graph = new LinkedList[numCourses];
        for(int i=0;i<numCourses;i++){
            graph[i] = new LinkedList<>();
        }
        for(int[] edge:prerequisites){
            //添加一条from指向to的有向边
            int from = edge[0];
            int to = edge[1];
            // 边的方向是「被依赖」关系，即修完课程 from 才能修课程 to
            graph[from].add(to);
        }
        return graph;
    }

    private void traverse(List<Integer>[] graph,int s){
        if(path[s]){
            //出现环
            hasCycle = true;
        }
        if(visted[s] || hasCycle){
            //如果已经找到了环，那么也不用再遍历了
            return;
        }
        //前序代码位置
        visted[s] = true;
        path[s] = true;
        for(int v:graph[s]){
            traverse(graph,v);
        }
        //后序代码位置
        path[s] = false;
    }
}
```



### 797、所有可能的路径

**题目链接**：[所有可能的路径](https://leetcode-cn.com/problems/all-paths-from-source-to-target/)

**解题思路**：

套用图的遍历框架，因为是有向无环图，所以可以不使用visit数组。

注意达到终点时，要移除最后一个节点，不然会出错。

**代码实现**：

```java
class Solution {

    List<List<Integer>> res = new ArrayList<>();
    public List<List<Integer>> allPathsSourceTarget(int[][] graph) {
        LinkedList<Integer> list = new LinkedList<>();
        traverse(graph,0,list);
        return res;
    }

    private void traverse(int[][] graph,int idx,LinkedList<Integer> list){
        list.add(idx);
        //递归结束条件
        if(idx == graph.length-1){
            //到达终点
            res.add(new ArrayList<>(list));
            //移除最后一个节点
            list.removeLast();//注意
            return;
        }
        //递归遍历相邻节点
        for(int v:graph[idx]){
            traverse(graph,v,list);
        }
        //从路径移除节点s
        list.removeLast();
    }
}
```



# 常见算法

## 一、滑动窗口

👍**滑动窗口模板**

```java
int left = 0, right = 0;

while (right < s.size()) {
    window.add(s[right]);
    right++;
    
    while (满足缩小窗口条件) {
        window.remove(s[left]);
        left++;
    }
}
```

使用滑动窗口前需要思考以下4个点：

1. 增大窗口时，要更新哪些数据？
2. 什么时候停止增大窗口，开始缩小窗口？
3. 缩小窗口时，要更新哪些数据？
4. 最后的结果在增大窗口时更新，还是缩小窗口时更新？

应用场景：

1、子字符串。在一个字符串中找出符合另外一个字符串的子串

2、子数组。在一个数组中找到满足条件的子数组



👍Leetcode 76、438、567总结

```java
题目特征：涉及到两个字符串，在一个字符串中寻找包含另一个字符串的子串。
题目思路：
1.使用一个HashMap，存储要找的那个字符串，对这个字符串进行计数。
		HashMap<Character, Integer> need = new HashMap<>();
        for (char c : t.toCharArray()) {
            need.put(c, need.getOrDefault(c, 0) + 1);
        }
2.使用一个变量match，用于判断是否达到缩小窗口。当字符串1中已经包含了字符串2就可以缩小窗口
3.在缩小窗口时就开始更新结果数据
    //代码模板总结，具体题目具体修改
    	//滑动窗口
        HashMap<Character, Integer> window = new HashMap<>();
        HashMap<Character, Integer> need = new HashMap<>();
        for (char c : t.toCharArray()) {
            need.put(c, need.getOrDefault(c, 0) + 1);
        }
		int left = 0;
        int right = 0;
        //存储结果的变量
        int match = 0;
        while (right < s1.length) {
          	//增大窗口
            char c = s.charAt(right);
            window.put(c, window.getOrDefault(c, 0) + 1);
            right++;
            if (window.get(c).equals(need.get(c))) {
                match++;
            }
            //缩小窗口
            while (match == need.size()) {
                //缩小窗口时更新结果
                if (right - left ...) {
                    //更新结果数据
                }
                //对称结构
                char c1 = s.charAt(left);
                window.put(c1, window.get(c1) - 1);
                left++;
                if (need.containsKey(c1)) {
                    if (window.get(c1) < need.get(c1)) {
                        match--;
                    }
                }
            }
        }

```



### 3、无重复字符的最长子串

**题目链接：** [无重复字符的最长子串](https://leetcode-cn.com/problems/longest-substring-without-repeating-characters/)

**解题思路：** 使用滑动窗口操作。注意：缩小窗口之后更新结果

**实现代码：** 

```java
class Solution {
    public int lengthOfLongestSubstring(String s) {
        
        int left = 0;
        int right = 0;
        int res = 0;
        HashMap<Character,Integer> window = new HashMap<>();
        //增大窗口
        while (right < s.length()) {
            //更新数据
            char c = s.charAt(right);
            window.put(c,window.getOrDefault(c,0)+1);
            right++;
            //缩小窗口
            while(window.get(c)>1){              
                //更新数据
                char c1 = s.charAt(left);
                window.put(c1,window.get(c1)-1);
                left++;
            }
             //更新结果
            res = Math.max(right-left,res);
        }
        
        return res;
    }
}
```



### 76、最小覆盖子串

**题目链接**：[最小覆盖子串](https://leetcode-cn.com/problems/minimum-window-substring/)

**算法思路**：

1. 使用一个HashMap表示一个窗口，再使用一个HashMap表示辅助窗口，存储需要覆盖的字符串
2. 不断增大窗口，往窗口里不断添加元素，当窗口覆盖辅助窗口时，停止增大窗口开始缩小窗口
3. 在缩小窗口时开始更新结果数据，然后缩小窗口，直到窗口不能覆盖辅助窗口
4. 返回结果值

注意：缩小窗口时就要更新结果。

**代码实现**：

```java
class Solution {
    public String minWindow(String s, String t) {
        int len1 = s.length();
        int len2 = t.length();
        if (len1 == 0 || len2 == 0 || len1 < len2) {
            return "";
        }
        //滑动窗口
        HashMap<Character, Integer> window = new HashMap<>();
        HashMap<Character, Integer> need = new HashMap<>();
        for (char c : t.toCharArray()) {
            need.put(c, need.getOrDefault(c, 0) + 1);
        }
        int left = 0;
        int right = 0;
        //存储结果的变量
        int start = 0;
        int end = 0;
        int match = 0;
        int minLen = Integer.MAX_VALUE;
        while (right < len1) {
          	//增大窗口
            char c = s.charAt(right);
            window.put(c, window.getOrDefault(c, 0) + 1);
            right++;
            //这一步关键
            if (window.get(c).equals(need.get(c))) {
                match++;
            }
            //缩小窗口
            while (match == need.size()) {
                //缩小窗口时更新结果
                if (right - left < minLen) {
                    start = left;
                    end = right;
                    minLen = right - left;
                }
                //对称结构
                char c1 = s.charAt(left);
                window.put(c1, window.get(c1) - 1);
                left++;
                //这一步关键
                if (need.containsKey(c1)) {
                    if (window.get(c1) < need.get(c1)) {
                        match--;
                    }
                }
            }
        }
        return minLen == Integer.MAX_VALUE ? "" : s.substring(start, end);

    }
}
```



### 438、找到字符串中所有字母异位词

**题目链接**：[找到字符串中所有字母异位词](https://leetcode-cn.com/problems/find-all-anagrams-in-a-string/)

**算法思路**：

使用滑动窗口。需要使用HashMap辅助存储另一个字符串。使用一个变量match判断能否缩小窗口。

注意：缩小窗口时就要更新结果。

**代码实现**：

```java
class Solution {

    public List<Integer> findAnagrams(String s, String p) {

        //滑动窗口
        HashMap<Character,Integer> window = new HashMap<>();
        //辅助工具
        HashMap<Character,Integer> need = new HashMap<>();
        for (char c : p.toCharArray()) {
            need.put(c, need.getOrDefault(c, 0) + 1);
        }

        int left = 0;
        int right = 0;
        int match = 0;

        //存储结果
        List<Integer> res = new ArrayList<>();

        while (right < s.length()) {
            char c = s.charAt(right);
            window.put(c, window.getOrDefault(c, 0) + 1);
            if(need.containsKey(c)){
                if(window.get(c).equals(need.get(c))){
                    match++;
                }
            }
            right++;
            while (match == need.size()) {
                //更新结果
                if (right - left == p.length()) {
                    res.add(left);
                }
                char c1 = s.charAt(left);
                window.put(c1,window.get(c1)-1);
                if (need.containsKey(c1)) {     
                    if (window.get(c1) < need.get(c1)) {
                        match--;
                    }
                }
                left++;

            }
        }
        return res;
    }
}
```



### 567、字符串的排列

**题目链接**：[字符串的排列](https://leetcode-cn.com/problems/permutation-in-string/)

**算法思路**：

使用滑动窗口。使用一个HashMap辅助存储一个字符串。使用变量match判断是否缩小窗口，当且仅当match等于辅助HashMap的大小时，开始缩小窗口。

注意：缩小窗口时就需要更新结果数据

**代码实现**：

```java
class Solution {
    public boolean checkInclusion(String s1, String s2) {

        //滑动窗口
        HashMap<Character,Integer> window = new HashMap<>();
        //辅助窗口
        HashMap<Character,Integer> need = new HashMap<>();
        for (char c : s1.toCharArray()) {
            need.put(c, need.getOrDefault(c, 0) + 1);
        }

        int left = 0;
        int right = 0;
        int match = 0;
        while (right < s2.length()) {
            char c = s2.charAt(right);
            window.put(c, window.getOrDefault(c, 0) + 1);
            if (need.containsKey(c)) {
                if(window.get(c).equals(need.get(c))){
                    match++;
                }
            }
            right++;
            while (match == need.size()) {
                //更新结果
                if (right - left == s1.length()) {
                    return true;
                }
                char c1 = s2.charAt(left);
                window.put(c1, window.get(c1) - 1);
                if(need.containsKey(c1)){ 
                    if (window.get(c1) < need.get(c1)) {
                        match--;
                    }
                }
                left++;
            }
        }
        return false;
    }
}
```



### 209、长度最小的子数组

**题目链接**：[长度最小的子数组](https://leetcode-cn.com/problems/minimum-size-subarray-sum/)

**算法思路**：

使用滑动窗口。当sum累加到大于等于target时就要开始缩小窗口

注意：缩小窗口时就要更新结果

**代码实现**：

```java
class Solution {
    public int minSubArrayLen(int target, int[] nums) {

        int left = 0;
        int right = 0;
        int sum = 0;
        int minLen = Integer.MAX_VALUE;

        while (right < nums.length) {
            int num = nums[right];
            sum += num;
            right++;
            while (sum >= target) {
                //更新结果
                if (right - left < minLen) {
                    minLen = right - left;
                }
                int num1 = nums[left];
                sum -= num1;
                left++;
            }
        }
        return minLen==Integer.MAX_VALUE?0:minLen;
    }
}
```





### 剑指offer57、和为s的连续正数

**题目链接**：[和为s的连续正数](https://leetcode-cn.com/problems/he-wei-sde-lian-xu-zheng-shu-xu-lie-lcof/)

**算法思路**：

1. 定义左右指针从1开始，不断地增大窗口，直到和的值大于等于目标值就开始缩小窗口
2. while循环的结束条件是：target/2 +1。为什么呢？因为超过target/2 +1 的数字和必定大于 target。+1的目的是避免遗漏解，比如 9/2=4，就遗漏了5这个解。

注意：缩小窗口时就要更新结果

**代码实现**：

```java
class Solution {
    public int[][] findContinuousSequence(int target) {

        //存储结果
        List<int[]> res = new ArrayList<>();

        int left = 1;
        int right = 1;
        int sum = 0;
        while (right <= (target / 2 + 1)) {
            int num = right;
            sum += right;
            right++;
            while (sum >= target) {
                //更新结果
                if (sum == target) {
                    int[] temp = new int[right - left];
                    for (int i = 0; i < temp.length; i++) {
                        temp[i] = left + i;
                    }
                    res.add(temp);
                }
                int num1 = left;
                sum -= num1;
                left++;
            }
        }
        return res.toArray(new int[res.size()][]);
    }
}
```





## 二、排序

👍**快排模板**

```java
/**
 * 快速排序
 * @author: 小LeetCode~
 **/
public class QuickSort {

    public static void main(String[] args) {
        int[] nums = {7, 8, 5, 1, 3, 4, 9, 6};
        System.out.println(Arrays.toString(nums));
        quicksort(nums,0,nums.length-1);
        System.out.println(Arrays.toString(nums));
    }

    private static void quicksort(int[] nums, int left, int right) {
        if(left>=right){
            return;
        }
        int mid = partition(nums,left,right);
        quicksort(nums,left,mid-1);
        quicksort(nums,mid+1,right);
    }

    private static int partition(int[] nums, int left, int right) {
        //基准数，一半选取第一个元素
        int x = nums[left];
        int i = left;
        int j = right+1;
        while(true){
            while(nums[++i]<x){
                if(i>=right){
                    break;
                }
            }
            while(nums[--j]>x){

            };
            if(i>=j){
                break;
            }
            swap(nums,i,j);
        }
        swap(nums,left,j);
        return j;
    }

    private static void swap(int[] nums, int i, int j) {
        int temp = nums[i];
        nums[i] = nums[j];
        nums[j] = temp;
    }
}


```

快排的特点

```
一旦找到mid这个数，则意味着在mid的左边的数都是小于它的，在mid的右边的数都是大于它的。
有了这个特点，可以结合二分法，可以快速定位到要找的数是在左边还是右边
```



👍**归并排序模板**

```java
/**
 * 归并排序
 *
 * @author: 小LeetCode~
 **/
public class MergeSort {

    public static void main(String[] args) {
        int[] nums = {7, 8, 5, 1,3, 4, 9, 6};
        System.out.println(Arrays.toString(nums));
        mergesort(nums, 0, nums.length - 1);
        System.out.println(Arrays.toString(nums));
    }

    private static void mergesort(int[] nums, int left, int right) {
        if (left >= right) {
            return;
        }
        int mid = (left+right)/2;//2的1次方
        mergesort(nums, left, mid);
        mergesort(nums, mid + 1, right);
        merge(nums, left, mid, right);
    }

    private static void merge(int[] nums, int left, int mid, int right) {
        int temp[] = new int[right-left+1];
        int i = left;
        int j = mid + 1;
        int k = 0;
        while (i <= mid && j <= right) {
            if (nums[i] <= nums[j]) {
                temp[k++] = nums[i++];
            }else{
                temp[k++] = nums[j++];
            }
        }
        while(i<=mid){
            temp[k++] = nums[i++];
        }
        while(j<=right){
            temp[k++] = nums[j++];
        }

        //把临时数组赋值给原来的数组
        for (int x = 0; x < temp.length ; x++) {
            nums[left+x] = temp[x];
        }
    }
}

```



### 4、 寻找两个正序数组的中位数

**题目链接**：[寻找两个正序数组的中位数](https://leetcode-cn.com/problems/median-of-two-sorted-arrays/)

**解题思路**：

1、使用归并排序，将两个数组合并。数组长度为奇数，则返回中间值。若是偶数，返回中间两个值除以2

2、既然已经知道两个数组的长度，其实也可以不用合并数组。通过指针下标判断是否已经达到中间值。

**实现代码**：

```java
class Solution {
    public double findMedianSortedArrays(int[] nums1, int[] nums2) {
        ArrayList<Integer> list = new ArrayList<>();
        int length1 = nums1.length;
        int length2 = nums2.length;
        int p1 = 0;
        int p2 = 0;
        while(p1<length1 && p2<length2){
            if(nums1[p1]<=nums2[p2]){
                list.add(nums1[p1]);
                p1++;
            }else{
                list.add(nums2[p2]);
                p2++;
            }
        }
        while(p1<length1){
            list.add(nums1[p1]);
            p1++;
        }
        while(p2<length2){
            list.add(nums2[p2]);
            p2++;
        }
        int size = list.size();
        if((size&1)==1){
            return (double)list.get(size/2);
        }else{
            return (list.get(size/2)+list.get(size/2 -1 ))/2.0;
        }
    }
}
```



### 75、颜色分类

**题目链接**：[颜色分类](https://leetcode-cn.com/problems/sort-colors/)

**算法思路**：使用快速排序即可解决问题

**代码实现**：

```java
class Solution {
    public void sortColors(int[] nums) {
        int left = 0;
        int right = nums.length - 1;
        quicksort(nums, left, right);
    }

    private void quicksort(int[] nums, int left, int right) {
        if (left >= right) {
            return;
        }
        int mid = partition(nums, left, right);
        quicksort(nums, left, mid - 1);
        quicksort(nums, mid + 1, right);
    }

    private int partition(int[] nums, int left, int right) {
        int x = nums[left];
        int i = left;
        int j = right + 1;
        while (true) {
            while (nums[++i] < x) {
                if (i >= right) {
                    break;
                }
            }
            while (nums[--j] > x) ;
            if(i>=j) break;
            swap(nums, i, j);
        }
        swap(nums, left, j);
        return j;
    }

    private void swap(int[] nums, int i, int j) {
        int tmp = nums[i];
        nums[i] = nums[j];
        nums[j] = tmp;
    }


}
```



### 剑指offer40、最小的k个数

**题目链接**：[最小的k个数](https://leetcode-cn.com/problems/zui-xiao-de-kge-shu-lcof/)

**算法思路**：

直接套用快排模板。快排的特点：一旦找到mid这个数，则意味着在mid的左边的数都是小于它的，在mid的右边的数都是大于它的。有了这个特点，可以结合二分法，可以快速定位到要找的数是在左边还是右边

当mid==k时，说明当前mid前面的数都是最小的

当mid>k时，说明需要往mid左边去寻找

当mid<k时，说明需要往mid右边去寻找

**代码实现**：

```java
class Solution {
    public int[] getLeastNumbers(int[] arr, int k) {
        int[] res = new int[k];
        quicksort(arr,0,arr.length-1,k);
        for(int i=0;i<k;i++){
            res[i] = arr[i];
        }
        return res;
    }

    private void quicksort(int[] nums,int low,int high,int k){
        if(low>=high) return;
        int mid = partition(nums,low,high);
        if(mid==k) return;
        else if(mid<k) quicksort(nums,mid+1,high,k);
        else quicksort(nums,low,mid-1,k);
    }

    private int partition(int[] nums,int low,int high){
        int x = nums[low];
        int i = low;
        int j = high + 1;
        while(true){
            while(nums[++i]<x){
                if(i>=high) break;
            }
            while(nums[--j]>x);
            if(i>=j) break;
            swap(nums,i,j);
        }
        swap(nums,low,j);
        return j;
    }

    private void swap(int[] nums,int i,int j){
        int tmp = nums[i];
        nums[i] = nums[j];
        nums[j] = tmp;
    }
}
```





### 剑指offer45、把数组排成最小的数

**题目链接**：[把数组排成最小的数](https://leetcode-cn.com/problems/ba-shu-zu-pai-cheng-zui-xiao-de-shu-lcof/)

**算法思路**：

算法的关键在于 x + y > y + x，那么 x 排在 y后面，这里需要使用到排序。

以上成立针对的是字符串，因此第一步需要将int数组转换成字符数组。

然后对字符数组进行排序，使用自定义排序

**代码实现**：

```java
class Solution {
    public String minNumber(int[] nums) {
        String[] s = new String[nums.length];
        for(int i=0;i<nums.length;i++){
            s[i] = String.valueOf(nums[i]);
        }
        //字符串的升序排序
        //x+y>y+x，说明x在y后面
        Arrays.sort(s,(x,y)->(x+y).compareTo(y+x));
        return String.join("",s);
    }
}
```



### 剑指offer51、数组中的逆序对

**题目链接**：[数组中的逆序对](https://leetcode-cn.com/problems/shu-zu-zhong-de-ni-xu-dui-lcof/)

**算法思路**：

直接使用归并排序模板。灵活使用了归并排序的特点

**代码实现**：

```java
class Solution {

    int res = 0;
    public int reversePairs(int[] nums) {
        //归并排序
        mergeSort(nums,0,nums.length-1);
        return res;
    }

    private void mergeSort(int[] nums,int low,int high){
        if(low>=high) return;
        int mid = low + ((high-low)>>1);
        mergeSort(nums,low,mid);
        mergeSort(nums,mid+1,high);
        merge(nums,low,mid,high);
    }

    private void merge(int[] nums,int low,int mid,int high){

        //拷贝数组
        int[] tmp = new int[high-low+1];
        int k = 0;
        int i = low;
        int j = mid+1;
        while(i<=mid && j<=high){
            if(nums[i]<=nums[j]){
                tmp[k++] = nums[i++];
            }else{
                tmp[k++] = nums[j++];
                res += mid - i + 1;//关键代码
            }
        }
        while(i<=mid) tmp[k++] = nums[i++];
        while(j<=high) tmp[k++] = nums[j++];

        //更新原有的数组的排序位置
        for(int x=0;x<tmp.length;x++){
            nums[low+x] = tmp[x];
        }
    }
}
```





## 三、动态规划

👍思考

```tex
动态规划特点：
1.重叠子问题	2.最优子结构 3.状态转移方程(最关键)

题型：求最值

核心：穷举。动态规划的本质就是穷举状态，然后在选择中选择最优

解题套路：
1.明确状态（即在函数中会改变变化的参数）
2.明确选择（即在一个状态中有多少种选择，穷举这个选择，选最优）
3.明确dp数组/函数的定义（即这个dp数组/函数表示什么意思）
3.明确初始化状态（即不能通过状态转移方程得到的）

寻找状态转移方程
1.明白dp数组的含义
2.假设dp[0...i-1]都求出来了，如何求dp[i](即找规律)
```

👍动态规划代码框架

```java
# 初始化 base case
dp[0][0][...] = base
# 进行状态转移
for 状态1 in 状态1的所有取值：
    for 状态2 in 状态2的所有取值：
        for ...
            dp[状态1][状态2][...] = 求最值(选择1，选择2...)
```

👍算法技巧

```
把一个大问题细化到一个点，先研究在这个小点上该如何处理问题，再通过递归或者迭代的方式扩展到整个问题
```



👍**子序列类型**

[最长递增子序列](https://leetcode-cn.com/problems/longest-increasing-subsequence/)

[编辑距离](https://leetcode-cn.com/problems/edit-distance/)

[最长公共子序列](https://leetcode-cn.com/problems/longest-common-subsequence/)

[最大子数组和](https://leetcode-cn.com/problems/maximum-subarray/)



**子序列模板**

一维数组型

```java
int n = array.length;

//dp数组含义：在子数组array[0..i]中，以array[i]结尾的目标子序列（最长递增子序列）的长度是dp[i]。
int[] dp = new int[n];

for (int i = 1; i < n; i++) {
    for (int j = 0; j < i; j++) {
        dp[i] = 最值(dp[i], dp[j] + ...)
    }
}
```

二维数组

```java
int n = arr.length;
/**
2.1 涉及两个字符串/数组时（比如最长公共子序列），dp 数组的含义如下：
在子数组arr1[0..i]和子数组arr2[0..j]中，我们要求的子序列（最长公共子序列）长度为dp[i][j]。

2.2 只涉及一个字符串/数组时（比如本文要讲的最长回文子序列），dp 数组的含义如下：
在子数组array[i..j]中，我们要求的子序列（最长回文子序列）的长度为dp[i][j]。
**/
int[][] dp = new dp[n][n];

for (int i = 0; i < n; i++) {
    for (int j = 1; j < n; j++) {
        if (arr[i] == arr[j]) 
            dp[i][j] = dp[i][j] + ...
        else
            dp[i][j] = 最值(...)
    }
}
```





👍**背包类型**



👍**其他类型**



### 10、正则表达式匹配

题目链接：[正则表达式匹配](https://leetcode-cn.com/problems/regular-expression-matching/)

解题思路：

递归做法比动态规划容易理解。

核心思路通过不断地剪去s和p的首部，直到某一个或者两个都被剪空，就可以得出答案。

有 ‘*’ 和 没 ‘ * ‘ 两种情况进行讨论。

其中有 ’ * ‘ 的又分两种情况，因为它可以匹配零次或者多次：

1. p的第i个元素在s中出现零次：那就s不动，p剪去前面两个字母。如 s=“aac”，p=“b*aac”，把p前面的两个元素剪去
2. p的第i个元素在s中出现一次或多次：比较s和p首元素是否相同，如果相同，p不动，s剪去首元素，继续递归。比如 s=“aabb”，p=“a*bb”，比较首元素相同，s 剪去首元素，即 s=“abb”，p不动，继续匹配

没 ’ * ‘ 的情况是最简单的，这时候只需要扫描一遍s和p，从首部开始比较两元素是否相同，如果相同就剪去，比较下一个即可。

```java
//第i下标位置元素相同，'.'代表任何元素
s.charAt(i) == p.charAt(i) || p.charAt(i)=='.'
```

代码实现：

```java
class Solution {
    public boolean isMatch(String s, String p) {
        if(p.length()==0){
            return s.length()==0;
        }
        //判断首字符是否匹配。这里s.length可能为0的情况，但p.length不可能为0，因为p.length为0是递归出口
        boolean firstIsMatch = (s.length()>0) && 		 (s.charAt(0)==p.charAt(0)||p.charAt(0)=='.');
        //两种情况，p带'*'和不带'*'
        if(p.length()>=2 && p.charAt(1)=='*'){
            return isMatch(s,p.substring(2)) //匹配0次情况
                || (firstIsMatch && isMatch(s.substring(1),p)); //匹配多次或者一次的情况
        }else{
            return firstIsMatch && isMatch(s.substring(1),p.substring(1));
        }
    }
}
```

递归

```java
class Solution {
    public boolean isMatch(String s, String p) {
        return dp(s,0,p,0);
    }

    /**
    若dp(s,i,p,j) = true，则表示s[i..]可以匹配p[j..]；
    若dp(s,i,p,j) = false，则表示s[i..]无法匹配p[j..]
     */
    private boolean dp(String s,int i,String p,int j){
        int m = s.length();
        int n = p.length();
        //j==p.length()意味着模式串p已经被匹配完了，那么应该看看文本串s匹配到哪里了，如果s也恰好被匹配完，则说明匹配成功
        if(j==n) return i==m;
        //当i走到s末尾的时候，j并没有走到p的末尾，但是p依然可以匹配s
        //因为p带'*'可以匹配0次以达到匹配s
        if(i==m){
            // 如果能匹配空串，一定是字符和 * 成对儿出现
            if((n-j)%2==1) return false;
            else{
                // 检查是否为 x*y*z* 这种形式
                for(;j+1<p.length();j+=2){
                    if(p.charAt(j+1)!='*'){
                        return false;
                    }
                }
                return true;
            }
        }
        if(s.charAt(i)==p.charAt(j) || p.charAt(j)=='.'){
            //匹配
            //看j后面是否带'*'
            if(j<p.length()-1 && p.charAt(j+1)=='*'){
                //通配符'*'匹配0次或多次
                return dp(s,i,p,j+2) || dp(s,i+1,p,j);
            }else{
                //通配符'.'匹配一次
                return dp(s,i+1,p,j+1);
            } 
        }else{
            //不匹配
            if(j<p.length()-1 && p.charAt(j+1)=='*'){
                //通配符匹配0次
                return dp(s,i,p,j+2);
            }else{
                //无法继续匹配
                return false;
            }
        }
    }
}
```



### 53、最大子数组和

**题目链接**：[最大子数组和](https://leetcode-cn.com/problems/maximum-subarray/)

**解题思路**：

1. 定义dp数组，dp[i]表示以nums[i]结尾的最大值
2. 初始化dp数组，只要初始化第一个数
3. 状态转移方程。dp[i] = Math.max(nums[i],dp[i-1]+nums[i])

**代码实现**：

```java
class Solution {
    public int maxSubArray(int[] nums) {
        int dp[] = new int[nums.length];
        dp[0] = nums[0];
        for (int i = 1; i < nums.length; i++) {
            dp[i] = Math.max(nums[i],dp[i-1]+nums[i]);
        }
        int res = Integer.MIN_VALUE;
        for(int i=0;i<nums.length;i++){
            res = Math.max(res,dp[i]);
        }
        return res;
    }
}
```



### 62、不同路径

**题目链接**：[不同路径](https://leetcode-cn.com/problems/unique-paths/)

**解题思路**：

1. 定义dp数组，dp[ i ] [ j ]表示到达第 i 行 第 j 列 时有多少路径
2. 初始化dp数组。第一行只能往左移动和第一列只能往下移动，所以路径都为1
3. 定义状态转移方程。除第一行和第一列外，每个位置都可以由上面和左边过来，所以dp[ i ] [ j ] = dp [i-1] [j] + dp [i] [ j-1]

**代码实现**：

```java
class Solution {
    public int uniquePaths(int m, int n) {
        //定义dp数组
        int [][] dp = new int[m][n];
        //初始化dp
        for (int i = 0; i < m; i++) {
            dp[i][0] = 1;
        }
        for (int j = 0; j < n; j++) {
            dp[0][j] = 1;
        }
        //状态转移方程
        for (int i = 1; i < m; i++) {
            for (int j = 1; j < n; j++) {
                dp[i][j] = dp[i-1][j] + dp[i][j-1];
            }
        }
        return dp[m - 1][n - 1];
    }
}
```



### 64、最小路径和

**题目描述**：[最小路径和](https://leetcode-cn.com/problems/minimum-path-sum/)

**算法思路**：

思路一：暴力求解，枚举所有可能的结果，时间复杂度高

思路二：使用动态规划

1. 定义dp数组，dp[ i ] [ j ]表达达到第 i 行 第 j 列时的最短路径
2. 初始化dp数组。第一行只能往左走，第一列只能向下走。
3. 定义状态转移方程，一个位置要么由左边过来，要么由上面下来。 dp[ i ] [j ] = Math.min(dp[ i -1] [ j],dp [i] [j-1]) + grid[i] [j];

**代码实现**：

暴力求解

```java
class Solution {
    int minRes;
    int m,n;
    public int minPathSum(int[][] grid) {
        m = grid.length;
        n = grid[0].length;
        minRes = Integer.MAX_VALUE;
        dfs(grid,0,0,0);
        return minRes;
    }

    private void dfs(int[][] grid, int i, int j,int res) {
        if (i < 0 || i >= m || j < 0 || j >= n) {
            return;
        }
        res += grid[i][j];
        if (i == m - 1 && j == n - 1) {
            minRes = Math.min(minRes,res);
            return;
        }
        dfs(grid,i+1,j,res);
        dfs(grid,i,j+1,res);
        res -= grid[i][j];
    }
}
```

动态规划

```java
class Solution {

    public int minPathSum(int[][] grid) {
        int m = grid.length;
        int n = grid[0].length;
        //定义dp数组
        int[][] dp = new int[m][n];
        //初始化dp数组
        dp[0][0] = grid[0][0];
        //第一列，只能向下走
        for (int i = 1; i < m; i++) {
            dp[i][0] = dp[i-1][0] + grid[i][0];
        }
        //第一行，只能向左走
        for (int j = 1; j < n; j++) {
            dp[0][j] = dp[0][j-1] + grid[0][j];
        }
        //状态转移方程
        for (int i = 1; i < m; i++) {
            for (int j = 1; j < n; j++) {
                dp[i][j] = Math.min(dp[i - 1][j], dp[i][j - 1]) + grid[i][j];
            }
        }
        return dp[m-1][n-1];
    }

}
```



### 70、爬楼梯

**题目链接**：[爬楼梯](https://leetcode-cn.com/problems/climbing-stairs/)

**算法思路**：

1. 定义dp数组。dp[i]表示到达第i层楼梯有多少种方法
2. 初始化dp数组。需要初始化第一二层的楼梯
3. 定义状态转移方程。当前楼层可以由前一层和前两层楼层得到，所以dp[i ] = dp[ i -1] + dp[ i-2]

**代码实现**：

```java
class Solution {
    public int climbStairs(int n) {
        if (n <= 2) {
            return n;
        }
        //定义dp数组
        int[] dp = new int[n+1];
        //初始化dp数组
        dp[1] = 1;
        dp[2] = 2;
        //定义状态转移方程
        for (int i = 3; i <= n; i++) {
            dp[i] = dp[i - 1] + dp[i - 2];
        }
        return dp[n];
    }
}
```



### 72、编辑距离

👍**解决两个字符串的动态规划问题，一般都是用两个指针 `i,j` 分别指向两个字符串的最后，然后一步步往前走，缩小问题的规模**。

**题目链接**：[编辑距离](https://leetcode-cn.com/problems/edit-distance/)

**算法思路**：

1. 定义dp数组。dp[i] [j]表示word1的[0..i]的字符转换成word2[0..j]的字符最少需要多少次
2. 初始化dp数组。如果word1为空，只能插入word2长度的字符；如果word2为空，只能删除word1个字符
3. 定义状态转移方程。当word1[i-1] = word2[j-1]时，dp[i] [j] = dp[i-1] [j-1]；如果不相等的话，则从删除、替换、增加中选择最少次数作为转移dp [i] [j] = Math.min(dp[i-1] [j-1],Math.min(dp[i-1] [j],dp[i] [j-1])) + 1;

**代码实现**：

递归(超时)

```java
class Solution {
    public int minDistance(String word1, String word2) {
        return dp(word1,word2,word1.length()-1,word2.length()-1);
    }

    private int dp(String s1,String s2,int i,int j){
        //递归结束条件
        if(i==-1) return j+1;
        if(j==-1) return i+1;
        if(s1.charAt(i)==s2.charAt(j)){
            //什么也不做，跳过
            return dp(s1,s2,i-1,j-1);
        }else{
            return min(dp(s1,s2,i,j-1),//插入
                       dp(s1,s2,i-1,j),//删除
                       dp(s1,s2,i-1,j-1))+1;//替换
        } 
    }

    private int min(int a,int b,int c){
        return Math.min(a,Math.min(b,c));
    }
}
```

带备忘录的递归

```java
class Solution {
    public int minDistance(String word1, String word2) {
        int[][] memo = new int[word1.length()][word2.length()];
        return dp(word1,word2,word1.length()-1,word2.length()-1,memo);
    }

    private int dp(String s1,String s2,int i,int j,int[][] memo){
        //递归结束条件
        if(i==-1) return j+1;
        if(j==-1) return i+1;
        if(memo[i][j]!=0) return memo[i][j];
        if(s1.charAt(i)==s2.charAt(j)){
            //什么也不做，跳过
            memo[i][j] = dp(s1,s2,i-1,j-1,memo);
        }else{
            memo[i][j] = min(dp(s1,s2,i,j-1,memo),//插入
                             dp(s1,s2,i-1,j,memo),//删除
                             dp(s1,s2,i-1,j-1,memo))+1;//替换
        }
        return memo[i][j]; 
    }

    private int min(int a,int b,int c){
        return Math.min(a,Math.min(b,c));
    }
}
```

动态规划(自顶向下)

```java
class Solution {
    public int minDistance(String word1, String word2) {
        int n1 = word1.length();
        int n2 = word2.length();
        //如果word1为空，只能插入n2个字符
        if(n1==0) return n2;
        //如果word2为空，只能删除n1个字符
        if(n2==0) return n1;

        //1.定义dp数组
        //dp[i][j]表示word1中[0..i]的字符转换到word2的[0..j]的最少次数
        int dp[][] = new int[n1+1][n2+1];
        //2.初始化dp数组
        dp[0][0] = 0;
        //当word1为空时，只能通过插入变成word2
        for (int j = 1; j <= n2 ; j++) {
            dp[0][j] = dp[0][j-1] + 1;
        }
        //当word2为空时，只能通过删除word1来变成word2
        for (int i = 1; i <= n1; i++) {
            dp[i][0] = dp[i-1][0] + 1;
        }
        //3.定义状态转移方程
        for (int i = 1; i <= n1; i++) {
            for (int j = 1; j <= n2; j++) {
                //如果word1[i-1]==word2[j-1]，啥也不做
                if (word1.charAt(i - 1) == word2.charAt(j - 1)) {
                    dp[i][j] = dp[i - 1][j - 1];
                }else{
                    //如果不相等，则从插入、删除、替换中次数最少的选择一个
                    //dp[i-1][j-1]表示替换
                    //dp[i-1][j]表示删除
                    //dp[i][j-1]表示插入
                    dp[i][j] = Math.min(dp[i-1][j-1],Math.min(dp[i-1][j],dp[i][j-1])) + 1;
                }
            }
        }
        return dp[n1][n2];

    }
}
```







### 198、打家劫舍

**题目链接**：[打家劫舍](https://leetcode-cn.com/problems/house-robber/)

**解题思路**：

有两种选择：抢与不抢。跟状态有关，动态规划！

**代码实现**：

暴力递归

```java
// 主函数
public int rob(int[] nums) {
    return dp(nums, 0);
}
// 返回 nums[start..] 能抢到的最大值
private int dp(int[] nums, int start) {
    if (start >= nums.length) {
        return 0;
    }

    int res = Math.max(
            // 不抢，去下家
            dp(nums, start + 1), 
            // 抢，去下下家
            nums[start] + dp(nums, start + 2)
        );
    return res;
}
```

备忘录递归

注意：备忘录要初始化小于0

```java
class Solution {
    public int rob(int[] nums) {
        int[] memo = new int[nums.length];
        Arrays.fill(memo,-1);//注意
        return dp(nums,0,memo);
    }

    private int dp(int[] nums,int idx,int[] memo){
        if(idx>=nums.length){
            return 0;
        }
        if(memo[idx]!=-1){
            return memo[idx];
        }
        //两种选择，偷还是不偷
        int res = Math.max(dp(nums,idx+1,memo),
                dp(nums,idx+2,memo)+nums[idx]);
        memo[idx] = res;
        return res;
    }
}

```

动态规划

```java
class Solution {
    public int rob(int[] nums) {
        if(nums.length == 0){
            return 0;
        }
        int n = nums.length;
        int dp[] = new int[n+1];
        dp[0] = 0;
        dp[1] = nums[0];
        for(int i=2;i<=n;i++){
            dp[i] = Math.max(dp[i-1],dp[i-2]+nums[i-1]);
        }
        return dp[n];
    }
}
```





### 152、乘积最大子数组

**题目链接**：[乘积最大子数组](https://leetcode-cn.com/problems/maximum-product-subarray/)

**解题思路**：

跟53题思路一样，但需要注意的是，对于乘法，负数 * 负数会变成正数，所以解这题时，我们需要维护两个dp数组，当前的最大值和最小值。

**代码实现**：

```java
class Solution {
    public int maxProduct(int[] nums) {
        //定义max_dp数组，max_dp[i]表示以nums[i]结尾的最大乘积
        int[] max_dp = new int[nums.length];
        //定义min_dp数组，min_dp[i]表示以nums[i]结尾的最大乘积
        int[] min_dp = new int[nums.length];
        //初始化dp数组
        max_dp[0] = nums[0];
        min_dp[0] = nums[0];
        //定义状态转移方程
        int res = max_dp[0];
        for(int i=1;i<nums.length;i++){
            max_dp[i] = max(nums[i],max_dp[i-1]*nums[i],min_dp[i-1]*nums[i]);
            min_dp[i] = min(nums[i],max_dp[i-1]*nums[i],min_dp[i-1]*nums[i]);
            res = Math.max(res,max_dp[i]);
        }

        return res;
    }

    private int max(int a,int b,int c){
        return Math.max(a,Math.max(b,c));
    }

    private int min(int a,int b,int c){
        return Math.min(a,Math.min(b,c));
    }
}
```



### 300、最长递增子序列

**题目链接**：[最长递增子序列](https://leetcode-cn.com/problems/longest-increasing-subsequence/)

**解题思路**：

1. 定义dp数组含义，以num[i]结尾的最长递增子序列
2. 初始化dp数组，每个元素都是一个以自己为递增的子序列，所以长度为1
3. 定义状态转移方程。假设知道了dp[0...i-1]的值，如何确定dp[i]？只要知道了num[0..i-1]中小于num[i]的值，就可以知道dp[i] = Math.max(dp[i],dp[j]+1)

**代码实现**：

```java
class Solution {
    public int lengthOfLIS(int[] nums) {
        //1.定义dp数组,dp[i]表示以nums[i]结尾的最长子序列
        int[] dp = new int[nums.length];
        //2.初始化dp数组,每一个数都是以自己为最长递增子序列
        for(int i=0;i<nums.length;i++){
            dp[i] = 1;
        }
        //3.定义状态转移方程
        int res = dp[0];
        for(int i=1;i<nums.length;i++){
            for(int j=0;j<i;j++){
                if(nums[i]>nums[j]){
                    dp[i] = Math.max(dp[i],dp[j]+1);
                }
            }
            res = Math.max(res,dp[i]);
        }
        return res;
        
    }
}
```



### 1143、最长公共子序列

**题目链接**：[最长公共子序列](https://leetcode-cn.com/problems/longest-common-subsequence/)

**解题思路**：

递归：

1.定义dp函数表示最长公共子序列

2.把整个大问题细化到小问题上，求整个字符串细化到每一个字符。

**代码实现**：

递归（超时）

```java
class Solution {
    public int longestCommonSubsequence(String text1, String text2) {
        return dp(text1,0,text2,0);
    }

    private int dp(String s1,int i,String s2,int j){
        //递归结束条件
        if(i==s1.length() || j==s2.length()) return 0;
        if(s1.charAt(i)==s2.charAt(j)){
            return dp(s1,i+1,s2,j+1) + 1;
        }else{
            return max(dp(s1,i+1,s2,j),//s1[i]不在lcs中
                       dp(s1,i,s2,j+1),//s2[j]不在lcs中
                       dp(s1,i+1,s2,j+1));//s1[i]和s2[j]都不在lcs中
        }
    }

    private int max(int a,int b,int c){
        return Math.max(a,Math.max(b,c));
    }
}
```

带备忘录的递归

```java
class Solution {
    public int longestCommonSubsequence(String text1, String text2) {
        int[][] memo = new int[text1.length()][text2.length()];
        return dp(text1,0,text2,0,memo);
    }

    private int dp(String s1,int i,String s2,int j,int[][] memo){
        //递归结束条件
        if(i==s1.length() || j==s2.length()) return 0;
        if(memo[i][j]!=0) return memo[i][j];
        if(s1.charAt(i)==s2.charAt(j)){
            memo[i][j] = dp(s1,i+1,s2,j+1,memo) + 1;
        }else{
            memo[i][j] = max(dp(s1,i+1,s2,j,memo),//s1[i]不在lcs中
                             dp(s1,i,s2,j+1,memo),//s2[j]不在lcs中
                             dp(s1,i+1,s2,j+1,memo));//s1[i]和s2[j]都不在lcs中
        }
        return memo[i][j];
    }

    private int max(int a,int b,int c){
        return Math.max(a,Math.max(b,c));
    }
}
```

动态规划

```java
class Solution {
    public int longestCommonSubsequence(String text1, String text2) {
        int m = text1.length();
        int n = text2.length();
        //1.定义dp数组
        //dp[i][j]表示长度为i的text1和长度为j的text2的最长公共子序列长度
        int[][] dp = new int[m+1][n+1];
        //2.初始化dp数组
        for(int i=0;i<=m;i++){
            dp[i][0] = 0;
        }
        for(int j=0;j<=n;j++){
            dp[0][j] = 0;
        }
        //3.状态转移方程
        for(int i=1;i<=m;i++){
            for(int j=1;j<=n;j++){
                if(text1.charAt(i-1)==text2.charAt(j-1)){
                    dp[i][j] = dp[i-1][j-1] + 1;
                }else{
                    dp[i][j] = max(dp[i-1][j],dp[i][j-1],dp[i-1][j-1]);
                }
            }
        }

        return dp[m][n];

    }

    private int max(int a,int b,int c){
        return Math.max(a,Math.max(b,c));
    }
}
```

### 221、最大正方形

**题目链接**：[最大正方形](https://leetcode-cn.com/problems/maximal-square/)

**解题思路**：

求最值问题，首先想到动态规划问题。

状态转移方程：

```java
// 伪代码
if (matrix(i - 1, j - 1) == '1') {
    dp(i, j) = min(dp(i - 1, j), dp(i, j - 1), dp(i - 1, j - 1)) + 1;
}
```



**代码实现**：

```java
class Solution {
    public int maximalSquare(char[][] matrix) {
        int m = matrix.length;
        int n = matrix[0].length;
        //存储最大的边
        int maxSize = 0;
        //1.定义dp数组，dp[i][j]表示以matric[i-1][j-1]为右下角的最大正方形的边长
        //多设置一行一列的原因在于方便计算边上的值
        int[][] dp = new int[m+1][n+1];
        //2.初始化dp数组，默认dp[m][0]和dp[0][n]的值都为0
        //3.定义状态转移方程
        for(int i=0;i<m;i++){
            for(int j=0;j<n;j++){
                if(matrix[i][j]=='1'){
                    //只跟左边、上边、左上角的数值有关，并且取值最小的那个
                    //木桶理论
                    dp[i+1][j+1] = min(dp[i+1][j],dp[i][j+1],dp[i][j])+1;
                    maxSize = Math.max(maxSize,dp[i+1][j+1]);
                }
            }
        }

        return maxSize * maxSize;

    }

    private int min(int a,int b,int c){
        return Math.min(a,Math.min(b,c));
    }
}
```





### 股票买卖问题

👍动态规划解题步骤

```java
1.定义dp数组
2.初始化
3.状态转移方程

动态规划的本质就是穷举状态，然后在状态选择中选择最优
for 状态1 in 状态1的所有取值：
    for 状态2 in 状态2的所有取值：
        for ...
            dp[状态1][状态2][...] = 择优(选择1，选择2...)
```

[本题讲解思路](https://labuladong.gitee.io/algo/1/12/)

```java
/*
每天都有三种「选择」：买入、卖出、无操作，我们用 buy, sell, rest 表示这三种选择
条件：因为 sell 必须在 buy 之后，buy 必须在 sell 之后。那么 rest 操作还应该分两种状态，一种是 buy 之后的 rest（持有了股票），一种是 sell 之后的 rest（没有持有股票）。而且别忘了，我们还有交易次数 k 的限制，就是说你 buy 还只能在 k > 0 的前提下操作。

「状态」有三个，第一个是天数，第二个是允许交易的最大次数，第三个是当前的持有状态（即之前说的 rest 的状态，我们不妨用 1 表示持有，0 表示没有持有）。考虑用三维数组
dp[i][k][0 or 1]
0 <= i <= n - 1, 1 <= k <= K
n 为天数，大 K 为交易数的上限，0 和 1 代表是否持有股票。
此问题共 n × K × 2 种状态，全部穷举就能搞定。
*/
for 0 <= i < n:
    for 1 <= k <= K:
        for s in {0, 1}:
            dp[i][k][s] = max(buy, sell, rest)
            
//最终答案是 dp[n - 1][K][0]，即最后一天，最多允许 K 次交易，最多获得多少利润。
                
1.定义dp数组
dp数组为dp[i][k][0 or 1]。记住数组表示的能获得的最大利润
                
2.状态转移方程
dp[i][k][0] = max(dp[i-1][k][0], dp[i-1][k][1] + prices[i])
              max( 今天选择 rest,        今天选择 sell       )
/*
解释：今天我没有持有股票，有两种可能，我从这两种可能中求最大利润：
1、我昨天就没有持有，且截至昨天最大交易次数限制为 k；然后我今天选择 rest，所以我今天还是没有持有，最大交易次数限制依然为 k。
2、我昨天持有股票，且截至昨天最大交易次数限制为 k；但是今天我 sell 了，所以我今天没有持有股票了，最大交易次数限制依然为 k。
*/
                
dp[i][k][1] = max(dp[i-1][k][1], dp[i-1][k-1][0] - prices[i])
              max( 今天选择 rest,         今天选择 buy         )
/*
解释：今天我持有着股票，最大交易次数限制为 k，那么对于昨天来说，有两种可能，我从这两种可能中求最大利润：
1、我昨天就持有着股票，且截至昨天最大交易次数限制为 k；然后今天选择 rest，所以我今天还持有着股票，最大交易次数限制依然为 k。
2、我昨天本没有持有，且截至昨天最大交易次数限制为 k - 1；但今天我选择 buy，所以今天我就持有股票了，最大交易次数限制为 k。
*/

3.初始化
dp[-1][...][0] = 0
//解释：因为 i 是从 0 开始的，所以 i = -1 意味着还没有开始，这时候的利润当然是 0。

dp[-1][...][1] = -infinity
//解释：还没开始的时候，是不可能持有股票的。
//因为我们的算法要求一个最大值，所以初始值设为一个最小值，方便取最大值。

dp[...][0][0] = 0
//解释：因为 k 是从 1 开始的，所以 k = 0 意味着根本不允许交易，这时候利润当然是 0。

dp[...][0][1] = -infinity
//解释：不允许交易的情况下，是不可能持有股票的。
//因为我们的算法要求一个最大值，所以初始值设为一个最小值，方便取最大值。

                
//最终代码
//1.定义dp数组
dp[i][k][0 or 1]
//2.初始化
dp[-1][...][0] = dp[...][0][0] = 0
dp[-1][...][1] = dp[...][0][1] = -infinity

//3.状态转移方程
dp[i][k][0] = max(dp[i-1][k][0], dp[i-1][k][1] + prices[i])
dp[i][k][1] = max(dp[i-1][k][1], dp[i-1][k-1][0] - prices[i])
                
```

#### 121.买卖股票的最佳时机（k=1）

**题目链接**：[买卖股票的最佳时机](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock/)

**解题思路**：

```java
dp[i][1][0] = max(dp[i-1][1][0], dp[i-1][1][1] + prices[i])
dp[i][1][1] = max(dp[i-1][1][1], dp[i-1][0][0] - prices[i]) 
            = max(dp[i-1][1][1], -prices[i])
//解释：k = 0 的 base case，所以 dp[i-1][0][0] = 0。

//现在发现 k 都是 1，不会改变，即 k 对状态转移已经没有影响了。
//可以进行进一步化简去掉所有 k：
dp[i][0] = max(dp[i-1][0], dp[i-1][1] + prices[i])
dp[i][1] = max(dp[i-1][1], -prices[i])
```

**代码实现**：

自己的版本，不通用

```java
class Solution {
    public int maxProfit(int[] prices) {
        //定义dp数组，dp[i]表示以nums[i]结尾能获取的最大利润
        int[] dp = new int[prices.length];
        //初始化dp数组,第一天最大利润就是不买
        dp[0] = 0;
        int minPreice = prices[0];
        //定义最大利润
        int res = 0;
        for(int i=1;i<prices.length;i++){
            if(prices[i]<minPreice){
                minPreice = prices[i];
                dp[i] = 0;
            }else{
                dp[i] = prices[i] - minPreice;
                res = Math.max(res,dp[i]);
            }
        }
        return res;

    }
}
```

通用版本

```java
class Solution {
    public int maxProfit(int[] prices) {
        int n = prices.length;
        //1.定义dp数组,k=1可以省略
        int [][] dp = new int[n][2];
        //2.初始化
        dp[0][0] = 0;//第一天没有持有股票，利润为0
        dp[0][1] = -prices[0];//第一天就持有股票，利润当然是负的啦
        //3.状态转移方程
        for(int i=1;i<n;i++){
            dp[i][0] = Math.max(dp[i-1][0],dp[i-1][1] + prices[i]);
            dp[i][1] = Math.max(dp[i-1][1],-prices[i]);
        }
        return dp[n-1][0];
    }
}
```



### 931、下降路径最小和

**题目链接**：[下降路径最小和](https://leetcode.cn/submissions/detail/353045277/)

**解题思路**：

典型的动态规划问题，按照思路。

1.明确状态。在这个求解过程中会变化的量就是矩阵(i,j)的位置，因此将这个作为状态变化量

2.定义dp数组/函数。dp(matrix,i,j)表示从一行开始向下落，达到(i,j)这个位置最小路径和

3.定义base case。第一行自然是不用计算的，既i==0直接返回matrix [i] [j] 的值

4.定义状态转移。dp(i,j)可以由dp(i-1,j-1)、dp(i-1,j)、dp(i-1,j+1)得来



上面求法还存在重叠子问题，比如dp(i-1,j-1)可以得到dp(i-1,j)，再得到dp(i,j)，这里就存在重叠子问题，可以考虑使用备忘录。

**代码实现**：

暴力解法：

```java
class Solution {


    public int minFallingPathSum(int[][] matrix) {
        int n = matrix.length;
        int res = Integer.MAX_VALUE;
        for(int j=0;j<n;j++){
            //1.明确状态。最后肯定达到最后一行，这个是不会变。
            //但是达到哪一列是不知道，所以需要穷举
            res = Math.min(res,dp(matrix,n-1,j));
        }
        return res;
    }


    //2.定义dp函数：表示达到（i，j）这个位置下降得最小和
    private int dp(int[][] matrix,int i,int j){
        if(i<0 || j<0 || i>=matrix.length || j>=matrix[0].length){
            //返回一个不存在的值。可以通过题目所给的范围确定
            return 99999;
        }
        //3.定义base case。第一行的值直接返回
        if(i==0){
            return matrix[0][j];
        }
        //4.做选择
        return matrix[i][j] + min(dp(matrix,i-1,j-1),
                                        dp(matrix,i-1,j),
                                        dp(matrix,i-1,j+1));

    }

    private int min(int a,int b,int c){
        return Math.min(a,Math.min(b,c));
    }


}
```

加上备忘录

```java
class Solution {

    int memo[][];
    public int minFallingPathSum(int[][] matrix) {
        int n = matrix.length;
        int res = Integer.MAX_VALUE;
        //定义一个备忘录
        memo = new int[n][n];
        for(int i=0;i<n;i++){
            Arrays.fill(memo[i],66666);
        }
        for(int j=0;j<n;j++){
            //1.明确状态。最后肯定达到最后一行，这个是不会变。
            //但是达到哪一列是不知道，所以需要穷举
            res = Math.min(res,dp(matrix,n-1,j));
        }
        return res;
    }


    //2.定义dp函数：表示达到（i，j）这个位置下降得最小和
    private int dp(int[][] matrix,int i,int j){
        if(i<0 || j<0 || i>=matrix.length || j>=matrix[0].length){
            //返回一个不存在的值。可以通过题目所给的范围确定
            return 99999;
        }
        //3.定义base case。第一行的值直接返回
        if(i==0){
            return matrix[0][j];
        }
        //如果已经计算过了，则直接返回
        if(memo[i][j]!=66666){
            return memo[i][j];
        }
        //4.做选择
        memo[i][j] =  matrix[i][j] + min(dp(matrix,i-1,j-1),
                                        dp(matrix,i-1,j),
                                        dp(matrix,i-1,j+1));

        return memo[i][j];
    }

    private int min(int a,int b,int c){
        return Math.min(a,Math.min(b,c));
    }


}
```



### 416、分割等和子集

**题目链接**：[分割等和子集](https://leetcode.cn/problems/partition-equal-subset-sum/)

**算法思路**：[0-1背包变种](https://mp.weixin.qq.com/s/OzdkF30p5BHelCi6inAnNg)

这题就是典型的0-1背包问题

**代码实现**：

```java
class Solution {
    public boolean canPartition(int[] nums) {
        int sum = 0;
        for(int num:nums){
            sum += num;
        }
        //和为奇数，肯定不能平分
        if(sum%2!=0) return false;
        sum = sum/2;//和为偶数，将其平分，转换成背包问题
        //定义dp数组，dp[i][j]表示对于前i个物品，当前背包容量为j，若此时为true,说明恰好把背包装满
        boolean[][] dp = new boolean[nums.length+1][sum+1];
        //定义base case
        for(int i=0;i<nums.length;i++){
            dp[i][0] = true;
        }
        for(int i=1;i<=nums.length;i++){
            for(int j=1;j<=sum;j++){
                if(j-nums[i-1]<0){
                    dp[i][j] = dp[i-1][j];
                }else{
                    dp[i][j] = dp[i-1][j] | dp[i-1][j-nums[i-1]];
                }
            }
        }
        return dp[nums.length][sum];
    }
}
```



### 518、兑换零钱2

**题目链接**：[兑换零钱2](https://leetcode.cn/problems/coin-change-2/)

**算法思路**：[完全背包问题](https://mp.weixin.qq.com/s/zGJZpsGVMlk-Vc2PEY4RPw)

**代码实现**：

```java
class Solution {
    public int change(int amount, int[] coins) {
        int n = coins.length;
        //两种状态，一个是amount，一个是硬币的状态
        int dp[][] = new int[n+1][amount+1];
        for(int i=0;i<=n;i++){
            dp[i][0] = 1;
        }
        //两个状态，两个for循环
        for(int i=1;i<=n;i++){
            for(int j=1;j<=amount;j++){
                if(j-coins[i-1]<0){
                    dp[i][j] = dp[i-1][j];
                }else{
                    dp[i][j] = dp[i-1][j] + dp[i][j-coins[i-1]];
                }
            }
        }
        return dp[n][amount];

    }
}
```





## 四、双指针

👍**双指针算法模板**：[双指针技巧框架](https://blog.csdn.net/weixin_42870497/article/details/119893198)

### 11、盛最多水的容器

**题目链接**：[盛最多水的容器](https://leetcode-cn.com/problems/container-with-most-water/)

**解题思路**：

使用双指针，分别指向左右两端。容器的面积的长（right-left）* 宽（Math.min(height[left],height[right])）

只有移动的较短的那个木板才会使得面积有变大的可能，移动较长的那个木板只会使面积变小。

**代码实现**：

```java
class Solution {
    public int maxArea(int[] height) {
        int left = 0;
        int right = height.length-1;
        int res = 0;
        while(left<right){
            res = height[left]<=height[right]?
                Math.max(res,(right-left) * height[left++]):
                Math.max(res,(right-left) * height[right--]);
        }
        return res;
    }
}
```

### 15、三数之和

**题目链接**：[三数之和](https://leetcode-cn.com/problems/3sum/)

**解题思路**：

1. 首先对数组排序，然后遍历数组，固定num[i]，再使用左右指针指向num[i]后面两端
2. 计算三个数的 sum是否为0，如果是则添加进结果集。
3. 如果 nums[i]大于 0，则三数之和必然无法等于 0，结束循环
4. 如果 nums[i] == nums[i-1]，则说明该数字重复，会导致结果重复，所以应该跳过
5. 当 sum0 时，nums[L] == nums[L+1] 则会导致结果重复，应该跳过，L++
6. 当 sum0 时，nums[R]== nums[R-1] 则会导致结果重复，应该跳过，R--

**代码实现**：

```java
class Solution {
    public static List<List<Integer>> threeSum(int[] nums) {
        List<List<Integer>> ans = new ArrayList();
        int len = nums.length;
        if(nums == null || len < 3) return ans;
        Arrays.sort(nums); // 排序
        for (int i = 0; i < len ; i++) {
            if(nums[i] > 0) break; // 如果当前数字大于0，则三数之和一定大于0，所以结束循环
            if(i > 0 && nums[i] == nums[i-1]) continue; // 去重
            int L = i+1;
            int R = len-1;
            while(L < R){
                int sum = nums[i] + nums[L] + nums[R];
                if(sum == 0){
                    ans.add(Arrays.asList(nums[i],nums[L],nums[R]));
                    while (L<R && nums[L] == nums[L+1]) L++; // 去重
                    while (L<R && nums[R] == nums[R-1]) R--; // 去重
                    L++;
                    R--;
                }
                else if (sum < 0) L++;
                else if (sum > 0) R--;
            }
        }        
        return ans;
    }
}

```

通用解法(还存在瑕疵)

[一个函数秒杀 2Sum 3Sum 4Sum 问题](https://mp.weixin.qq.com/s/fSyJVvggxHq28a0SdmZm6Q)

```java
class Solution {

    LinkedList<Integer> tmp = new LinkedList<>();
    public List<List<Integer>> threeSum(int[] nums) {
        //注意：调用nSum之前一定要给数组排序
        Arrays.sort(nums);
        return nSum(nums, 3, 0, 0);
    }

    private List<List<Integer>> nSum(int[] nums, int n, int start, int target) {
        int len = nums.length;
        List<List<Integer>> res = new ArrayList<>();
        //至少是2Sum，且数组大小不能小于n
        if (n < 2 || len < n) {
            return res;
        }
        //2Sum是最基本情况
        if (n == 2) {
            //双指针
            int low = start;
            int high = len - 1;
            while (low < high) {
                int sum = nums[low] + nums[high];
                int left = nums[low];
                int right = nums[high];
                if (sum < target) {
                    while (low < high && nums[low] == left) low++;
                } else if (sum > target) {
                    while (low < high && nums[high] == right) right--;
                }else{
                    tmp.add(left);
                    tmp.add(right);
                    res.add(tmp);
                    tmp.removeLast();
                    tmp.removeLast();
                    left++;
                    right--;
                }
            }
        }else{
            //n>2时，递归计算(n-1)Sum
            for(int i=start;i<len;i++){
                List<List<Integer>> arrs = nSum(nums, n - 1, i + 1, target - nums[i]);
                for(List<Integer> arr:arrs){
                    arr.add(nums[i]);
                    res.add(arr);
                }
                while(i<len-1 && nums[i]==nums[i+1]) i++;
            }

        }
        return res;
    }
}
```



### 42、接雨水

**题目链接**：[接雨水](https://leetcode-cn.com/problems/trapping-rain-water/)

**解题思路**：

一、暴力解法

先考虑局部，仅仅对于位置i，能装多少水？

算出位置i左边的最高柱子leftMax和右边最高的柱子rightMax，则位置 i 能装的水为Math.min(leftMax,rightMax)-height[i]

时间复杂度：O(n^2)

空间复杂度：O(1)

二、备忘录

开两个数组leftMax[]和rightMax[]做备忘录，leftMax[i]表示位置 i  左边最高的柱子高度，rightMax[i] 表示位置 i  右边最高的柱子高度。预先把这两个数组计算好，避免重复计算。

并不需要知道右边或者左边最高的是谁，只要知道当前位置最小的是谁。

时间复杂度：O(n)

空间复杂度：O(n)

![](https://labuladong.gitee.io/algo/images/%e6%8e%a5%e9%9b%a8%e6%b0%b4/3.jpg)

三、双指针

使用双指针边走边算。一直更新记录最右两边的最值情况。

时间复杂度：O(n)

空间复杂度：O(1)

![](https://labuladong.gitee.io/algo/images/%e6%8e%a5%e9%9b%a8%e6%b0%b4/4.jpg)



**代码实现**：

一、暴力解法

```java
class Solution {
    public int trap(int[] height) {
        //注意n
        int n = height.length;
        int resMax = 0;
        //第一个和最后一个都不用计算
        for(int i=1;i<n-1;i++){
            int leftMax = 0;
            int rightMax = 0;
            //求出左边的最高数
            for(int left=i;left>=0;left--){
                leftMax = Math.max(leftMax,height[left]);
            }
            //求出右边的最高数
            for(int right=i;right<=n-1;right++){
                rightMax = Math.max(rightMax,height[right]);
            }
            //容器能装多少水取决于较短的那根
            resMax += Math.min(leftMax,rightMax) - height[i];
        }
        return resMax;
    }
}
```



二、备忘录

```java
class Solution {
    public int trap(int[] height) {
        int maxResult = 0;
        int n = height.length;
        //使用备忘录
        int leftMax[] = new int[n];
        int rightMax[] = new int[n];
        leftMax[0] = height[0];
        rightMax[n-1] = height[n-1];
        //从左向右计算左边最高的柱子
        //leftMax[i]表示第i位柱子的左边最高的柱子
        for (int i=1;i<n;i++){
            leftMax[i] = Math.max(height[i],leftMax[i-1]);
        }
        //从右向左计算右边最高的柱子
        //rightMax[i]表示第i位柱子的有边最高的柱子
        for (int i = n - 2; i >= 0; i--) {
            rightMax[i] = Math.max(height[i],rightMax[i+1]);
        }
        //计算能接的雨水
        for (int i = 0; i < n; i++) {
            maxResult += Math.min(leftMax[i],rightMax[i]) - height[i];
        }
        return maxResult;
    }
}

```

三、双指针

```java
class Solution {
    public int trap(int[] height) {
        int maxResult = 0;
        int left = 0;
        int right = height.length-1;
        int leftMax = 0;
        int rightMax = 0;
        while(left < right){
            leftMax = Math.max(leftMax,height[left]);
            rightMax = Math.max(rightMax,height[right]);
            if (leftMax < rightMax) {
                maxResult += leftMax - height[left];
                left++;
            }else{
                maxResult += rightMax - height[right];
                right--;
            }
        }
        return maxResult;
    }
}
```



## 五、回溯法

👍**算法模板**：[回溯框架团灭排列、组合、子集问题](https://blog.csdn.net/weixin_42870497/article/details/119443910)

👍**矩阵搜索算法模板：**

```java
class Solution {

    int m,n;
    public T function(int[][] grid) {
        //行
        m = grid.length;
        //列
        n = grid[0].length;
        //记录结点是否访问过
        boolean[][] visited = new boolean[m][n];
        for(int i=0;i<m;i++){
            for(int j=0;j<n;j++){
                 dfs(grid,i,j,visited);
            }
        }
        return ..;
    }

    private void dfs(int[][] grid,int i,int j,boolean[][] visited){
        //递归结束条件
        if(i<0 || i>=m || j<0 || j>=n || grid[i][j]== || visited[i][j]){
            return;
        }
        //满足题目条件时的操作
        ...
        //访问过置true
        visite[i][j] = true;
        //向上
        dfs(grid,i+1,j,visited);
        //向下
        dfs(grid,i-1,j,visited);
        //向左
        dfs(grid,i,j-1,visited);
        //向右
        dfs(grid,i,j+1,visited);
        visited[i][j] = false;//这个待定，看题目要求而定
    }
}

---------------------------------------------------------------------------------
// 二维矩阵遍历框架
void dfs(int[][] grid, int i, int j, boolean[] visited) {
    int m = grid.length, n = grid[0].length;
    if (i < 0 || j < 0 || i >= m || j >= n) {
        // 超出索引边界
        return;
    }
    if (visited[i][j]) {
        // 已遍历过 (i, j)
        return;
    }
    // 进入节点 (i, j)
    visited[i][j] = true;
    dfs(grid, i - 1, j, visited); // 上
    dfs(grid, i + 1, j, visited); // 下
    dfs(grid, i, j - 1, visited); // 左
    dfs(grid, i, j + 1, visited); // 右
}

---------------------------------------------------------------------------------
// 方向数组，分别代表上、下、左、右
int[][] dirs = new int[][]{{-1,0}, {1,0}, {0,-1}, {0,1}};

void dfs(int[][] grid, int i, int j, boolean[] visited) {
    int m = grid.length, n = grid[0].length;
    if (i < 0 || j < 0 || i >= m || j >= n) {
        // 超出索引边界
        return;
    }
    if (visited[i][j]) {
        // 已遍历过 (i, j)
        return;
    }

    // 进入节点 (i, j)
    visited[i][j] = true;
    // 递归遍历上下左右的节点
    for (int[] d : dirs) {
        int next_i = i + d[0];
        int next_j = j + d[1];
        dfs(grid, next_i, next_j, visited);
    }
    // 离开节点 (i, j)
}
```

👍回溯算法疑惑

```
如果满足结束条件，就把track加入result，有的时候需要在这里取消选择，有的时候又不需要？
关键是看你结束递归的 base case。
如果 base case 在叶子节点上结束，因为叶子结点也是一个节点，所以 base case 里面也要进行「选择」和「撤销选择」。
如果 base case 在空节点上结束，那就不用考虑这些，直接 return 就好。
```



### 17、电话号码的字母组合

**题目链接**：[电话号码的字母组合](https://leetcode-cn.com/problems/letter-combinations-of-a-phone-number/)

**解题思路**：

应用回溯法的解题思路

1. 先用HashMap初始化数字和字母的组合
2. 使用回溯法开始深度遍历

**代码实现**：

```java
class Solution {

    HashMap<Character,String> map = new HashMap<>(){
        {
            put('2',"abc");
            put('3',"def");
            put('4',"ghi");
            put('5',"jkl");
            put('6',"mno");
            put('7',"pqrs");
            put('8',"tuv");
            put('9',"wxyz");
        }
    };

    List<String> list = new ArrayList<>();
    public List<String> letterCombinations(String digits) {
        if(digits.length()==0){
            return list;
        }
        StringBuilder sb = new StringBuilder();
        dfs(digits,sb,0);
        return list;
    }

    private void dfs(String digits,StringBuilder sb,int index){
        if(index==digits.length()){
            list.add(sb.toString());
            return;
        }
        char c = digits.charAt(index);
        String str = map.get(c);
        for(int i=0;i<str.length();i++){
            sb.append(str.charAt(i));
            dfs(digits,sb,index+1);
            sb.deleteCharAt(sb.length()-1);
        }
    }
}
```



### 39、组合总和

**题目链接：**[组合总和](https://leetcode-cn.com/problems/combination-sum/)

**解题思路：**回溯法。关键点：start这个参数

1. 使用回溯法的思路进行求解
2. 需要先对数组进行排序，方便后续的剪枝操作
3. start 参数的表示之前的元素已经使用过了，当前层的参数还可以继续重复使用。因为组合跟排列不同，[1,2] 和 [2,1]是不同的排列，但是却是相同的组合。

**代码实现：**

```java
class Solution {

    List<List<Integer>> res;
    public List<List<Integer>> combinationSum(int[] candidates, int target) {
        res = new ArrayList<>();
        LinkedList<Integer> list = new LinkedList<>();
        Arrays.sort(candidates);
        dfs(candidates,target,list,0);
        return res;
    }

    private void dfs(int[] nums,int target,LinkedList<Integer> list,int start){
        if (target == 0) {
            res.add(new ArrayList<Integer>(list));
        }
        for (int i = start; i < nums.length; i++) {
            //剪枝
            if (target < nums[i]) {
                return;
            }
            list.add(nums[i]);
            dfs(nums,target-nums[i],list,i);//注意是i，因为数字可以无限制重复被选取
            list.removeLast();

        }
    }
}
```



### 40、组合总和2

**题目链接：**[组合总和2](https://leetcode-cn.com/problems/combination-sum-ii/)

**解题思路：**回溯法。

1. 使用回溯法的思路进行求解
2. 需要先对数组进行排序，方便后续的剪枝操作
3. start 参数的表示之前的元素已经使用过了，当前层的参数还可以继续重复使用。因为组合跟排列不同，[1,2] 和 [2,1]是不同的排列，但是却是相同的组合。在这里 start 还有一个含义，，表示进入下一层递归。
4. 这里不需要再使用 visit数组判断元素是否使用过，因为 start 参数就决定了进入下一层就不会重复使用元素
5. 大剪枝的作用是避免重复元素在同一个位置重复计算

**代码实现：**

```java
class Solution {

    List<List<Integer>> res = new ArrayList<>();
    public List<List<Integer>> combinationSum2(int[] candidates, int target) {
        LinkedList<Integer> list = new LinkedList<>();
        Arrays.sort(candidates);
        backtrack(candidates,target,list,0);
        return res;
    }

    private void backtrack(int[] nums,int target,LinkedList<Integer> list,int start){
        if(target==0){
            res.add(new ArrayList<Integer>(list));
            return;
        }
        for(int i=start;i<nums.length;i++){
            //条件判断，剪枝
            if(target<nums[i]) return;
            //大剪枝。
            if(i>start && nums[i-1]==nums[i]) continue;
            list.add(nums[i]);
            backtrack(nums,target-nums[i],list,i+1);//注意i+1，因为每个数字在每个组合中只能使用一次
            list.removeLast();
        }
    }
}

```





### 46、全排列

**题目链接**：[全排列](https://leetcode-cn.com/problems/permutations/)

**算法思路**：

1. 使用回溯法的思路进行求解
2. 使用 visit 数组的原因是每次进入下一层的递归都是从头开始遍历的，但是元素不能重复使用，所以需要进行剪枝

**代码实现**：

```java
class Solution {

    List<List<Integer>> res = new ArrayList<>();
    public List<List<Integer>> permute(int[] nums) {
        LinkedList<Integer> list = new LinkedList<>();
        boolean [] visit = new boolean[nums.length];
        backtrack(nums,list,visit);
        return res;
    }

    private void backtrack(int[] nums,LinkedList<Integer> list,boolean[] visit){
    	//结束条件
        if(list.size()==nums.length){
            res.add(new ArrayList(list));
            return;
        }
        for(int i=0;i<nums.length;i++){
            //小剪枝
            if(visit[i]) continue;
            //做选择
            list.add(nums[i]);
            visit[i] = true;
            //递归
            backtrack(nums,list,visit);
            //撤销选择
            visit[i] = false;
            list.removeLast();
        }
    }
}

```

### 47、全排列2

**题目链接**：[全排列2](https://leetcode-cn.com/problems/permutations-ii/)

**算法思路**：

1. 使用回溯法的思路进行求解
2. 首先对数组进行排序，排序的原因是数组存在重复的原因，经过排序后，重复的元素就会紧靠在一起，为后续的剪枝做操作
3. 使用 visit 数组的原因是每次进入下一层的递归都是从头开始遍历的，但是元素不能重复使用，所以需要进行剪枝
4. 大剪枝的原因是对重复元素在同一个位置的剪枝

**代码实现**：

```java
class Solution {

    List<List<Integer>> res = new ArrayList<>();
    public List<List<Integer>> permuteUnique(int[] nums) {
        LinkedList<Integer> list = new LinkedList<>();
        boolean [] visit = new boolean[nums.length];
        Arrays.sort(nums);//这个排序很关键
        backtrack(nums,list,visit);
        return res;
    }

    private void backtrack(int[] nums,LinkedList<Integer> list,boolean[] visit){
        if(list.size()==nums.length){
            res.add(new ArrayList(list));
            return;
        }

        for(int i=0;i<nums.length;i++){
        	//小剪枝
            if(visit[i]) continue;
            //大剪枝
            if(i>0 && nums[i-1]==nums[i] && visit[i-1]) continue;
            //做选择
            list.add(nums[i]);
            visit[i]=true;
            //递归
            backtrack(nums,list,visit);
            //撤销选择
            visit[i]=false;
            list.removeLast();
        }
    }
}

```



### 78、子集

**题目链接**：[子集](https://leetcode-cn.com/problems/subsets/)

**算法思路**：

1. 使用回溯法的思路进行求解
2. 注意参数start的作用，它是避免进入下一层递归时，添加到重复的元素

**代码实现**：

```java
class Solution {

    List<List<Integer>> res = new ArrayList<>();
    public List<List<Integer>> subsets(int[] nums) {
        LinkedList<Integer> list = new LinkedList<>();
        dfs(nums, list,0);
        return res;
    }

    private void dfs(int[] nums, LinkedList<Integer> list,int start) {
        res.add(new ArrayList<>(list));
        for (int i = start; i < nums.length; i++) {
            list.add(nums[i]);
            dfs(nums, list, i + 1);
            list.removeLast();
        }
    }
}
```



### 79、单词搜索

**题目链接：**[单词搜索](https://leetcode-cn.com/problems/word-search/)

**算法思路：**

1. 使用回溯法的思路进行矩阵的遍历
2. 注意不能重复访问，所以需要visit变量记录。遍历的过程中，首先单词第一个元素必须匹配上，所以需要一个index变量进行记录
3. 思路可以参考第200题

**代码实现：**

```java
class Solution {

    int m,n;
    public boolean exist(char[][] board, String word) {
        m = board.length;
        n = board[0].length;
        boolean[][] visit = new boolean[m][n];
        for(int i=0;i<m;i++){
            for(int j=0;j<n;j++){
                if(dfs(board,word,i,j,0,visit)){
                    return true;
                }
            }
        }
        return false;
    }

    private boolean dfs(char[][] board,String word,int i,int j,int index,boolean[][] visit){
        if(i<0 || j<0 || i>=m || j>=n) return false;
        if(board[i][j]!=word.charAt(index) || visit[i][j]) return false;
        if(word.length()-1==index) return true;
        visit[i][j] = true;
        boolean flag =  dfs(board,word,i-1,j,index+1,visit)
                    || dfs(board,word,i+1,j,index+1,visit)
                    || dfs(board,word,i,j-1,index+1,visit)
                    || dfs(board,word,i,j+1,index+1,visit);
        visit[i][j] = false;
        return flag;
    }
}
```



### 90、子集2

**题目链接**：[子集2](https://leetcode-cn.com/problems/subsets-ii/)

**算法思路**：

1. 使用回溯法的思路进行求解
2. 回溯前先进行排序，后续剪枝操作需要
3. 注意参数start的作用，它是避免进入下一层递归时，添加到重复的元素

**代码实现**：

```java
class Solution {

    List<List<Integer>> res = new ArrayList<>();
    public List<List<Integer>> subsetsWithDup(int[] nums) {
        LinkedList<Integer> list = new LinkedList<>();
        Arrays.sort(nums);
        backtrack(nums,list,0);
        return res;
    }

    private void backtrack(int[] nums,LinkedList<Integer> list,int start){
        res.add(new ArrayList(list));
        for(int i=start;i<nums.length;i++){
        	//剪枝判断
            if(i>start && nums[i-1]==nums[i]) continue;
            list.add(nums[i]);
            backtrack(nums,list,i+1);
            list.removeLast();
        }
    }
}

```



### 93、复原IP地址

**题目链接**：[复原IP地址](https://leetcode-cn.com/problems/restore-ip-addresses/)

**解题思路**：

1. 使用回溯法 + 剪枝的思路进行求解。需要确定一个变参，那就是切割的字符串位数
2. 回溯的过程中，当list.size()长度为4时，说明都是合法字符串，加入到结果集中
3. 回溯法的关键还是在于for循环如何处置，如果切割完字符串，剩余的字符串大于了合理字符串长度，进行剪枝；如果不大于，再判断切割的字符串是否是合理的字符串，如果不合理，剪枝
4. 最后进行回溯求解

**代码实现**：

```java
class Solution {

    List<String> res = new ArrayList<>();
    public List<String> restoreIpAddresses(String s) {
        int len = s.length();
        //对字符串长度进行判断
        if(len<4 || len>12){
            return res;
        }
        LinkedList<String> list = new LinkedList<>();
        //回溯法
        dfs(s,0,list);
        return res;
    }

    private void dfs(String s,int start,LinkedList<String> list){
        //递归终止条件，分割的四个字符都是合法的
        if(list.size()==4){
            res.add(String.join(".",list));
            return;
        }

        for(int i=start;i<s.length();i++){
            //每次切割后，对剩余字符串的长度进行判断，看是否合理
            //比如你剩下10个字符串，但是合法的字符串最多是9个，因此进行剪枝
            if(s.length()-i-1 > 3*(3-list.size())) continue;
            //判断分割的字符串是否是合理的字符串
            String str = s.substring(start,i+1);
            if(!isValid(str)) continue;
            //把合法的字符串加入到列表中
            list.add(str);
            dfs(s,i+1,list);
            list.removeLast();           
        }
    }

    private boolean isValid(String s){
        //判断前导是否为0，比如01这种就是不合法的
        if(s.charAt(0)=='0' && s.length()>1) return false;
        //判断是否超过255。长度超过4直接返回false
        if(s.length()>4 || Integer.parseInt(s)>255) return false;
        return true;
    }
}
```



### 200、岛屿数量

**题目链接**：[岛屿数量](https://leetcode-cn.com/problems/number-of-islands/)

**算法思路**：把岛屿「淹了」，主要是为了省事，避免维护 `visited` 数组。

**代码实现**：

```java
class Solution {

    int m,n;
    public int numIslands(char[][] grid) {
        m = grid.length;
        n = grid[0].length;
        int res = 0;
        for(int i=0;i<m;i++){
            for(int j=0;j<n;j++){
                //每发现一个岛屿，岛屿数量+1
                if(grid[i][j]=='1'){
                    res++;
                    // 然后使用 DFS 将岛屿淹了
                    dfs(grid,i,j);
                }
            }
        }
        return res;
    }

    private void dfs(char[][] grid,int i,int j){
        //边界条件
        if(i<0 || j<0 || i>=m || j>=n){
            return;
        }
        //已经是海水了
        if(grid[i][j]=='0') return;
        // 将 (i, j) 变成海水
        grid[i][j] = '0';
        // 淹没上下左右的陆地
        dfs(grid,i-1,j);
        dfs(grid,i+1,j);
        dfs(grid,i,j-1);
        dfs(grid,i,j+1);
    }
}
```



### 695、岛屿的最大面积

**题目链接**：[岛屿的最大面积](https://leetcode-cn.com/problems/max-area-of-island/)

**解题思路**：参考第200题思路

**代码实现**：

```java
class Solution {

    int m,n;
    public int maxAreaOfIsland(int[][] grid) {
        m = grid.length;
        n = grid[0].length;
        int res = 0;
        for(int i=0;i<m;i++){
            for(int j=0;j<n;j++){
                if(grid[i][j]==1){
                    // 淹没岛屿，并更新最大岛屿面积
                    res = Math.max(res,dfs(grid,i,j));
                }
            }
        }
        return res;
    }

    private int dfs(int[][] grid,int i,int j){
        if(i<0 || j<0 || i>=m || j>=n) return 0;
        if(grid[i][j]==0) return 0;
        grid[i][j] = 0;
        return dfs(grid,i-1,j)
            + dfs(grid,i+1,j)
            + dfs(grid,i,j-1)
            + dfs(grid,i,j+1)
            + 1; 
    }
}
```





### 1254、统计封闭岛屿的数量

**题目链接**：[统计封闭岛屿的数量](https://leetcode-cn.com/problems/number-of-closed-islands/)

**算法思路**：

还是使用第200题的一样的思路。

只要处理掉边上的岛屿数量，剩下的就是封闭岛屿的数量

**代码实现**：

```java
class Solution {

    int m,n;
    public int closedIsland(int[][] grid) {
        m = grid.length;
        n = grid[0].length;
        int res = 0;

        //把靠边的岛屿排除掉，剩下的就是岛屿的数量
        //去掉左右两边的岛屿
        for(int i=0;i<m;i++){
            dfs(grid,i,0);
            dfs(grid,i,n-1);
        }
        //去掉上下两边的岛屿
        for(int j=0;j<n;j++){
            dfs(grid,0,j);
            dfs(grid,m-1,j);
        }

        //剩下的就是岛屿的数量
        for(int i=0;i<m;i++){
            for(int j=0;j<n;j++){
                //每发现一个岛屿，岛屿数量+1
                if(grid[i][j]==0){
                    res++;
                    // 然后使用 DFS 将岛屿淹了
                    dfs(grid,i,j);
                }
            }
        }
        return res;
    }

    private void dfs(int[][] grid,int i,int j){
        //边界条件
        if(i<0 || j<0 || i>=m || j>=n){
            return;
        }
        //已经是海水了
        if(grid[i][j]==1) return;
        // 将 (i, j) 变成海水
        grid[i][j] = 1;
        // 淹没上下左右的陆地
        dfs(grid,i-1,j);
        dfs(grid,i+1,j);
        dfs(grid,i,j-1);
        dfs(grid,i,j+1);
    }
}
```



### 剑指offer13、机器人的运动范围

**题目链接**：[机器人的运动范围](https://leetcode-cn.com/problems/ji-qi-ren-de-yun-dong-fan-wei-lcof/)

**解题思路**：

使用回溯法进行求解；

注意：下标是从(0,0)开始的；题目要求的是机器人能到达多少个格子，所以它的是它的上下左右能到达格子的总和；

**代码实现**：

```java
class Solution {

    
    public int movingCount(int m, int n, int k) {
        if(k==0) return 1;
        boolean [][] visit = new boolean[m][n];
        return dfs(m,n,k,0,0,visit);
    }

    private int dfs(int m,int n,int k,int i,int j,boolean [][] visit){
        if(i<0 || j<0 || i>=m || j>=n) return 0;
        if(sum(i,j)>k || visit[i][j]) return 0;
        visit[i][j] = true;
        //只能向下走或者向左走
        return dfs(m,n,k,i+1,j,visit)
             + dfs(m,n,k,i,j+1,visit)
             + 1;
    }

    private int sum(int a,int b){
        int sum = 0;
        while(a>0){
            sum += a%10;
            a /= 10;
        }
        while(b>0){
            sum += b%10;
            b /= 10;
        }
        return sum;
    }
}
```





## 六、递归

👍写递归算法的关键是明确递归函数的定义是什么

👍**递归模板**：

```java
public void recur(int level,int param){
    //1.递归终止条件
    if(level>MaxLevel){
        //过程结果
        return;
    }
    //2.处理当前层逻辑
    process...
    //3.进入下一层
    recur(level+1,newParam);
    //4.清理当前层，如果需要的话
    process...
}

参数说明：
level表示来到了第几层
param表示其他参数
```



👍**数据结构递归模板**

```java
//递归遍历二叉树
void traverse(TreeNode root) {
    if (root == null) {
        return;
    }
    // 前序位置
    traverse(root.left);
    // 中序位置
    traverse(root.right);
    // 后序位置
}

/* 迭代遍历数组 */
void traverse(int[] arr) {
    for (int i = 0; i < arr.length; i++) {

    }
}

/* 递归遍历数组 */
void traverse(int[] arr, int i) {
    if (i == arr.length) {
        return;
    }
    // 前序位置
    traverse(arr, i + 1);
    // 后序位置
}

/* 迭代遍历单链表 */
void traverse(ListNode head) {
    for (ListNode p = head; p != null; p = p.next) {

    }
}

/* 递归遍历单链表 */
void traverse(ListNode head) {
    if (head == null) {
        return;
    }
    // 前序位置
    traverse(head.next);
    // 后序位置
}
```

👍**递归三问**

```
1.这个函数是干什么的
2.这个函数的参数中，变量是什么？即缩小规模的变量
3.得到递归结果，应该干什么
```



### 22、括号生成

**题目链接**：[括号生成](https://leetcode-cn.com/problems/generate-parentheses/)

**解题思路**：

使用暴力递归，枚举所有可能的结果。

使用left和right两个参数分别表示左括号和右括号的个数，只要左括号没有达到最大值就可以一直往里面添加左括号。只要右括号小于左括号的个数，就可以往里面添加右括号。最后只要左括号和右括号达到最大个数就可以停止递归。

**代码实现**：

```java
class Solution {

    List<String> res;
    public List<String> generateParenthesis(int n) {
        res = new ArrayList<>();
        recursion(0,0,n,"");
        return res;
    }

    private void recursion(int left,int right,int max,String s){
        //递归结束条件
        if(left==max && right==max){
            res.add(s);
            return;
        }
        //处理逻辑
        if(left<max){
            recursion(left+1,right,max,s+"(");
        }
        if(left>right){
            recursion(left,right+1,max,s+")");
        }
        return;

    }
}
```



### 50、Pow(x,n)

**题目链接**：[Pow(x,n)](https://leetcode-cn.com/problems/powx-n/)

**解题思路**：https://blog.csdn.net/weixin_42870497/article/details/118736946

**代码实现**：

```java
class Solution {
    public double myPow(double x, int n) {
        if(n<0){
            x = 1/x;
            n = -n;
        }
        return pow(x,n);
    }

    private double pow(double x,int n){
        if(n==0) return 1;
        double val = pow(x,n/2);
        if((n&1)==0){
            return val*val;
        }else{
            return val*val*x;
        }
    }
}
```





## 七、二分查找

👍**二分查找模板**：[二分查找模板](https://blog.csdn.net/weixin_42870497/article/details/119728363)

```java
package com.search;

/**
 * 二分查找
 * @author: 小LeetCode~
 **/
public class BinarySearch {

    public static void main(String[] args) {
        //最基本的二分查找
//        int[] nums = {1,2,3,4,5,6};
//        int result_mid = binarySearch_mid(nums,7);
//        System.out.println(result_mid);

        int[] nums = {1,2,2,2,3,4};
        //查找中间边界的2
        int result_mid = binarySearch_mid(nums,2);//查找到索引为2
        System.out.println(result_mid);
        //查找左侧边界的索引
        int result_left = binarySearch_left(nums,2);//查找到的索引为1
        System.out.println(result_left);
        //查找右侧边界的索引
        int result_right = binarySearch_right(nums,2);//查找到的索引为3
        System.out.println(result_right);

    }

    /**
     * 最基本的二分查找
     * @param nums
     * @return
     */
    private static int binarySearch_mid(int[] nums,int target){
        int left = 0;
        int right = nums.length - 1;//注意
        //left=right+1结束循环
        while (left <= right) {
            int mid = left + ((right-left)>>1);
            if (nums[mid] < target) {
                left = mid + 1;
            } else if (nums[mid] > target) {
                right = mid - 1;
            }else{
                return mid;//直接返回
            }
        }
        return -1;
    }

    /**
     * 查找左侧边界
     * @param nums
     * @return
     */
    private static int binarySearch_left(int[] nums,int target){
        int left = 0;
        int right = nums.length - 1;//注意
        //left=right+1结束循环
        while (left <= right) {
            int mid = left + ((right-left)>>1);
            if (nums[mid] < target) {
                left = mid + 1;
            } else if (nums[mid] > target) {
                right = mid - 1;
            }else{
                right = mid-1;//向左收缩边界
            }
        }

        //条件判断
        if(left>=nums.length || nums[left]!=target){
            return -1;
        }

        //这里返回值是根据num[mid]==target来返回的，此时right=mid-1
        //相当于mid = right+1
        //而结束条件right+1=left
        //相当于mid = left
        //所以返回left或者right+1都可以
        return left;
    }

    /**
     * 查找右侧边界
     * @param nums
     * @return
     */
    private static int binarySearch_right(int[] nums,int target){
        int left = 0;
        int right = nums.length - 1;//注意
        while (left <= right) {
            int mid = left + ((right-left)>>1);
            if (nums[mid] < target) {
                left = mid + 1;
            } else if (nums[mid] > target) {
                right = mid - 1;
            }else{
                left = mid + 1;//向右侧收缩边界
            }
        }

        //条件判断
        if (right < 0 || nums[right] != target) {
            return -1;
        }

        //这里返回值是根据num[mid]==target来返回的，此时left=mid+1
        //相当于mid = left-1
        //而结束条件right+1=left,right-left-1
        //相当于mid=right
        //所以返回left-1或者right都可以
        return right;
    }
}
```



### 33、搜索旋转排序数组

**题目链接**：[搜索旋转排序数组](https://leetcode-cn.com/problems/search-in-rotated-sorted-array/)

**解题思路**：

采用二分法的思路进行搜索。

旋转数组后，从中间划分，一定有一边是有序的。

由于一定有一边是有序的，可以根据有序的两个边界值来判断目标值在有序一边还是无序一边。

注意：有序一边的边界值可能等于目标值，所以判断时别忘了等号

**代码实现**：

```java
class Solution {
    public int search(int[] nums, int target) {
        int left = 0;
        int right = nums.length-1;
        while(left<=right){
            int mid = left + ((right-left)>>1);
            if(nums[mid]==target) return mid;
            //右边的数组有序
            if(nums[mid]<nums[right]){
                //目标值在右边有序数组中?
                if(nums[mid]<target && nums[right]>=target){
                    left = mid + 1;
                }else{//不在有边那就在左边
                    right = mid - 1;
                }
            }
            //左边的数组有序
            else{
                //目标值在左边数组中？
                if(nums[left]<=target && nums[mid]>target){
                    right = mid - 1;
                }else{//不在左边那就在右边
                    left = mid + 1;
                }
            }
            
        }
        return -1;
        
    }
}
```





### 34、在排序数组中查找元素的第一个和最后一个位置

**题目链接**：[在排序数组中查找元素的第一个和最后一个位置](https://leetcode-cn.com/problems/find-first-and-last-position-of-element-in-sorted-array/)

**解题思路**：二分查找

**代码模板**：

```java
class Solution {
    public int[] searchRange(int[] nums, int target) {
        if (nums.length == 0) {
            return new int[]{-1,-1};
        }
        int left = binarySearchLeft(nums, target);
        int right = binarySearchRight(nums, target);
        return new int[]{left, right};
    }

    private int binarySearchLeft(int[] nums, int target) {
        int left = 0;
        int right = nums.length-1;
        while (left <= right) {
            int mid = left + ((right - left) >> 1);
            if (nums[mid] < target) {
                left = mid + 1;
            } else if (nums[mid] > target) {
                right = mid - 1;
            } else {
                right = mid - 1;
            }
        }
        if (left >= nums.length || nums[left] != target) {
            return -1;
        }
        return left;
    }

    private int binarySearchRight(int[] nums, int target) {
        int left = 0;
        int right = nums.length-1;
        while (left <= right) {
            int mid = left + ((right - left) >> 1);
            if (nums[mid] < target) {
                left = mid + 1;
            } else if (nums[mid] > target) {
                right = mid - 1;
            } else {
                left = mid + 1;
            }
        }
        if (right < 0 || nums[right] != target) {
            return -1;
        }
        return right;
    }


}
```



### 剑指offer11、旋转数组的最小数字

**题目描述**：[旋转数组的最小数字](https://leetcode-cn.com/problems/xuan-zhuan-shu-zu-de-zui-xiao-shu-zi-lcof/)

**算法思路**：[旋转数组的最小数字](https://blog.csdn.net/weixin_42870497/article/details/118582577)

**代码实现**：

```java
class Solution {
    public int minArray(int[] numbers) {
        int left = 0;
        int right = numbers.length - 1;
        while(left<=right){
            int mid = left + ((right-left)>>1);
            if(numbers[mid]<numbers[right]){
                right = mid;
            }
            else if(numbers[mid]>numbers[right]){
                left = mid+1;
            }
            else{
                right = right-1;
            }
        }
        //找到最小值的那个数时，right会减1，所以返回left
        return numbers[left];
    }
}
```



## 八、贪心算法

### 55、跳跃游戏

**题目描述**：[跳跃游戏](https://leetcode-cn.com/problems/jump-game/)

**算法思路**：

第 i 个元素能跳的最远距离为 i + nums[i]（i从下标0开始），如果这个最远距离大于等于最后一个元素的位置，则说明能跳到。

使用一个变量去记录最远距离，实时更新它，只要有一次能达到就说明能跳到。

**代码实现**：

```java
class Solution {
    public boolean canJump(int[] nums) {

        if (nums == null || nums.length == 0) {
            return false;
        }
        //记录最远距离的变量
        int tmp = 0;
        for (int i = 0; i <= tmp; i++) {
            //第i个元素能够跳的最远距离
            int dis = i + nums[i];
            tmp = Math.max(tmp, dis);
            if(tmp>=nums.length-1){
                return true;
            }
        }
        return false;
    }
}
```



## 九、位运算

👍**算法常用操作**

`n & (n-1)`  作用是消除数字n的二进制表示中的最后一个1

一个数和它本身做异或运算结果为 0，即 `a ^ a = 0`；

一个数和 0 做异或运算的结果为它本身，即 `a ^ 0 = a`



### 136、只出现一次的数字

**题目链接**：[只出现一次的数字](https://leetcode-cn.com/problems/single-number/)

**解题思路**：

一个数和它本身做异或运算结果为 0，即 `a ^ a = 0`；

一个数和 0 做异或运算的结果为它本身，即 `a ^ 0 = a`

**代码实现**：

```java
class Solution {
    public int singleNumber(int[] nums) {
        int res = 0;
        for(int x:nums){
            res ^= x;
        }
        return res;
    }
}
```



### 剑指offer 56、数组中只出现一次的两个数字

**题目链接**：[数组中只出现一次的两个数字](https://leetcode-cn.com/problems/shu-zu-zhong-shu-zi-chu-xian-de-ci-shu-lcof/)

**解题思路**：

一个数和它本身做异或运算结果为 0，即 `a ^ a = 0`；

一个数和 0 做异或运算的结果为它本身，即 `a ^ 0 = a`



我们可以遍历数组，将数组中的数字一起做异或，则相同的数字会约简掉,最后会得到不相同的数字，比如 1 ^ 4 ^ 1 ^ 6 = 4 ^ 6 = 0100 ^ 0110 = 0010



根据异或的性质，说明最后异或出来的那个数至少有一位是1，不然x ^ y就相等等于0

这时候可以通过辅助变量m来保存最后数字中哪一位是1（可能有多个1，但只需要找到最后一个1就行）

```cs
举个例子：z = 10 ^ 2 = 1010 ^ 0010 = 1000,第四位为1
我们将m初始化为1，如果（z & m）的结果等于0说明z的最低为是0
//我们每次将m左移一位然后跟z做与操作，直到结果不为0.
此时m应该等于1000，同z一样，第四位为1
```



然后我们再遍历一遍数组，将每个数跟m进行与操作，结果为0的作为一组，结果不为0的作为一组

```
例如对于数组：[1,2,10,4,1,4,3,3]，我们把每个数字跟1000做与操作，可以分为下面两组：
nums1存放结果为0的: [1, 2, 4, 1, 4, 3, 3]
nums2存放结果不为0的: [10] (碰巧nums2中只有一个10，如果原数组中的数字再大一些就不会这样了)

此时我们发现问题已经退化为数组中有一个数字只出现了一次
分别对nums1和nums2遍历异或就能得到我们预期的x和y
```



**代码实现**：

```java
class Solution {
    public int[] singleNumbers(int[] nums) {
        int z = 0;
        //遍历数组，做异或运算
        for(int n:nums){
            z ^= n;
        }
        //找到z中最后一位为1的数
        int m = 1;
        while((z & m) == 0) m <<= 1;
        //再遍历数组，跟m做&运算，可以分为两组，等于0和不等于0
        int x=0,y=0;
        for(int n:nums){
            if((n & m)!=0){
                x ^= n;
            }else{
                y ^= n;
            }
        }
        return new int[]{x,y};

    }
}

```





