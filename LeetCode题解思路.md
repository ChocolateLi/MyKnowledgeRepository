# LeetCode题解思路

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







## 三、滑动窗口

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



## 四、排序

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



## 五、字符串

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



## 六、动态规划

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



## 七、双指针

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
        while (left <= right) {
            int mid = left + ((right - left) >> 1);
            if (nums[mid] == target) {
                return mid;
            }
            //右边有序
            if(nums[mid]<nums[right]){
                //目标值在右边
                if(target>nums[mid] && target<=nums[right]){
                    left = mid + 1;
                }else{//目标值在左边
                    right = mid - 1;
                }
            }else{//左边有序
                //目标值在左边
                if(target<nums[mid] && target>=nums[left]){
                    right = mid - 1;
                }else{//目标值在右边
                    left = mid + 1;
                }
            }
            
        }
        return -1;
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

使用双指针边走边算

时间复杂度：O(n)

空间复杂度：O(1)

![](https://labuladong.gitee.io/algo/images/%e6%8e%a5%e9%9b%a8%e6%b0%b4/4.jpg)



**代码实现**：

一、暴力解法

 ```java
 class Solution {
     public int trap(int[] height) {
         int maxResult = 0;
         int n = height.length;
         for(int i=1;i<n-1;i++){
             int maxLeft = 0;
             int maxRight = 0;
             //找左边最高的柱子
             for(int left=i;left >= 0;left--){
                 maxLeft = Math.max(maxLeft, height[left]);
             }
             //找右边最高的柱子
             for (int right = i; right < n; right++) {
                 maxRight = Math.max(maxRight, height[right]);
             }
             maxResult += Math.min(maxLeft,maxRight) - height[i];
         }
         return maxResult;
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
        //leftMax[i]表示第i位柱子的左边最高的柱子
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





## 八、回溯法

👍**算法模板**：[回溯框架](https://blog.csdn.net/weixin_42870497/article/details/119443910)

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
            dfs(nums,target-nums[i],list,i);
            list.removeLast();

        }
    }
}
```







## 九、栈

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

方法二：栈

1. 定义一个栈，先放入-1。这样做的目的是，让当前右括号的下标减去栈顶元素即为要求的长度，也为了保持统一。
2. 遇到 '(' ，将它的下标放入栈中
3. 遇到 ')' ，先弹出栈顶元素表示匹配了当前右括号：如果栈为空，说明当前的右括号没有被匹配，我们将其下标放入栈中

**代码实现**：

```java
class Solution {
    public int longestValidParentheses(String s) {
        int maxLen = 0;
        Stack<Integer> stack = new Stack<>();
        //初始化先放一个-1。目的是让当前右括号的下标减去栈顶元素即为要求的长度
        stack.push(-1);
        for (int i = 0; i < s.length(); i++) {
            //碰到'('，将其下标入栈
            if (s.charAt(i) == '(') {
                stack.push(i);
            }else{//碰到 ')'，弹出栈顶，并计算长度
                stack.pop();
                if (stack.isEmpty()) {
                    stack.push(i);
                }
                maxLen = Math.max(maxLen, i - stack.peek());
            }
        }
        return maxLen;
    }
}
```





## 十、递归

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



## 十一、二分查找

👍**二分查找模板**：[二分查找模板](https://blog.csdn.net/weixin_42870497/article/details/119728363)

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



## 十二、二维数组

**主对角线反转矩阵(按照左上到右下的对角线进行镜像对称)**

![主对角线](/Users/chenli75/Desktop/MyFile/github/MyKnowledgeRepository/picture/主对角线镜像.jpg)

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

![反对角线](/Users/chenli75/Desktop/MyFile/github/MyKnowledgeRepository/picture/反对角线镜像.jpg)

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





### 48、旋转图像

**题目链接**：[旋转图像](https://leetcode-cn.com/problems/rotate-image/)

**解题思路**：

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







## 十三、其他

### 31、下一个排列

**题目链接**：[下一个排列](https://leetcode-cn.com/problems/next-permutation/)

**解题思路**：

1. 定义 i=nums.length-1，从后向前寻找第一个满足nums[i]>nums[i-1]的数，其中nums[i-1]就是要交换的数
2. 对 i 到 nums.length范围的数进行排序
3. 排序后，在 i 到 nums.length中寻找第一个nuns[j] > nums[i-1]的数，其中nums[j] 就是要交换的数
4. 交换nums[i-1] 和 nums[j] ，返回

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



