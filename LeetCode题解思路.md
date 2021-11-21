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

## 排序

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

## 字符串

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

## 动态规划

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



## 双指针

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



## 回溯法

**算法模板**：[回溯框架](https://blog.csdn.net/weixin_42870497/article/details/119443910)

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



## 链表

👍**链表模式识别**：

- 涉及链表特殊位置，使用快慢指针
- 要删除链表节点，找到节点的前驱节点

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
        ListNode slow = head;
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
        slow.next = slow.next.next;
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





## 栈

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



