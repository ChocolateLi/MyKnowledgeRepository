# LeetCode题解思路

## 哈希表

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



## 链表

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

**实现代码**：

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        ListNode newHead = new ListNode();
        ListNode sumNode = newHead;
        int carry = 0;
        while(l1!=null || l2!=null){
            //取出当前位
            int x = l1!=null?l1.val:0;
            int y = l2!=null?l2.val:0;
            //求和
            int sum = x + y + carry;
            if(sum>=10){
                carry = 1;
                sum = sum%10;
            }else{
                carry = 0;
            }
            sumNode.next = new ListNode(sum);
            //三个指针一起动
            l1 = l1!=null?l1.next:null;
            l2 = l2!=null?l2.next:null;
            sumNode = sumNode.next;
        }
        //两数都加完后，最后判断一下进位 carry, 进位不为 0 的话加在前面
        if(carry!=0){
            sumNode.next = new ListNode(carry);
        }
        return newHead.next;

    }
}
```



## 滑动窗口

### 3、无重复字符的最长子串

**题目链接：** [无重复字符的最长子串](https://leetcode-cn.com/problems/longest-substring-without-repeating-characters/)

**解题思路：** 使用滑动窗口操作

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



## 双指针

### 11、盛最多水的容器

题目链接：[盛最多水的容器](https://leetcode-cn.com/problems/container-with-most-water/)

解题思路：

使用双指针，分别指向左右两端。容器的面积的长（right-left）* 宽（Math.min(height[left],height[right])）

只有移动的较短的那个木板才会使得面积有变大的可能，移动较长的那个木板只会使面积变小。

代码实现：

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

