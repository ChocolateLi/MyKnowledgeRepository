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



### 84、柱状图中最大的矩形

**题目链接：**[柱状图中最大的矩形](https://leetcode-cn.com/problems/largest-rectangle-in-histogram/)

**算法思路：**

1、暴力解法

1. 面积=底*高。固定底边，求最大高度不好求；固定高，求最长底边好求
2. 从i向左右两边遍历，找到左边和右边第1个严格小于height[i]的时候停下，中间的长度就是最长底边



2、单调栈

1. 使用一个栈存储数组下标，里面元素是单调递增的；当进栈的元素严格小于栈顶元素时，栈顶元素出栈，求出矩阵的高，矩阵的宽度就是当前下标减去现在的栈顶元素下标再减1，求出矩阵面积；再比较最大的矩阵面积
2. 注意，可以使用哨兵，可以强制使所有进栈元素都出栈，一种设计技巧



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
            //单调栈,存储的是数组下标
            while (!stack.isEmpty() && newHeights[stack.peek()] > newHeights[i]) {
                int curHeight = stack.pop();
                int left = stack.peek();
                int width = i - left - 1;
                res = Math.max(res, newHeights[curHeight] * width);
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



## 四、二叉树

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



### 96、不同的二叉搜索树

**题目链接**：[不同的二叉搜索树](https://leetcode-cn.com/problems/unique-binary-search-trees/)

**解题思路**：

1. 穷举每一个节点作为根节点的情况，它的左右子树使用递归去计算
2. 如何计算该根节点的所能构造的二叉搜索树的个数？固定根节点，计算左右节点的组合数，左右节点的组合数相乘就是该节点的个数

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

### 98、验证二叉搜索树

**题目链接**：[验证二叉搜索树](https://leetcode-cn.com/problems/validate-binary-search-tree/)

**解题思路**：

二叉搜索树的根节点的值大于所有左子树的值，小于所有右子树的值。

明确了根节点要干什么了，就是跟左右子树比较。

注意：根节点不只是比较自己的左右节点，而是比较所有左右节点。根节点就是左子树的最大值，右子树的最小值，可以通过添加两个参数

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



### 101、对称二叉树

**题目链接**：[对称二叉树](https://leetcode-cn.com/problems/symmetric-tree/)

**解题思路**：

**代码实现**：

```java
class Solution {
    public boolean isSymmetric(TreeNode root) {
        if(root==null) return true;
        return isSymmetric(root.left, root.right);
    }

    private boolean isSymmetric(TreeNode left, TreeNode right) {
        if(left==null || right==null) return left==right;
        if(left.val!=right.val) return false;
        return isSymmetric(left.right,right.left) && isSymmetric(left.left,right.right);
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

    int maxDepth = 0;
    public int maxDepth(TreeNode root) {
        return travel(root);
    }

    private int travel(TreeNode root){
        if(root == null){
            return 0;
        }
        int leftDepth = travel(root.left);
        int rightDepth = travel(root.right);
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



### 124、二叉树中的最大路径和

**题目链接**：[二叉树中的最大路径和](https://leetcode-cn.com/problems/binary-tree-maximum-path-sum/)

**解题思路**：

后序遍历。求出根节点的最大单边路径和

后序代码的时候去更新最大路径和

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
        int leftSum = Math.max(0,postOrder(root.left));
        int rightSum = Math.max(0,postOrder(root.right));
        //更新最大路径和
        int pathSum = root.val + leftSum + rightSum;
        res = Math.max(res,pathSum);
        //实现定义，每个节点只能经过一次
        return Math.max(leftSum,rightSum) + root.val;
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

先交换了左右子树，再交换左右节点

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



### 450、删除二叉搜索树中的节点

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

就是在106题中，添加一行代码就可以实现

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



### 652、寻找重复的子树

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
        int maxValue = Integer.MIN_VALUE;
        int maxIndex = -1;
        for(int i=low;i<=high;i++){
            if(nums[i]>maxValue){
                maxValue = nums[i];
                maxIndex = i;
            }
        }
        TreeNode root = new TreeNode(maxValue);
        root.left = constructMaximumBinaryTree(nums,low,maxIndex-1);
        root.right = constructMaximumBinaryTree(nums,maxIndex+1,high);
        return root;
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



## 六、二维数组

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



### 76、最小覆盖子串

**题目链接**：[最小覆盖子串](https://leetcode-cn.com/problems/minimum-window-substring/)

**算法思路**：

1. 使用一个HashMap表示一个窗口，再使用一个HashMap表示辅助窗口，存储需要覆盖的字符串
2. 不断增大窗口，往窗口里不断添加元素，当窗口覆盖辅助窗口时，停止增大窗口开始缩小窗口
3. 在缩小窗口时开始更新结果数据，然后缩小窗口，直到窗口不能覆盖辅助窗口
4. 返回结果值

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









## 三、动态规划

👍思考

```tex
1.动态规划的本质就是穷举状态，然后在选择中选择最优
```



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

### 121.买卖股票的最佳时机（k=1）

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
        //dp[i]表示第i天股票卖出获取的最大利润
        int[] dp = new int[prices.length];
        dp[0] = 0;
        int minPrice = prices[0];
        for(int i=1;i<prices.length;i++){
            if(prices[i]<minPrice){
                minPrice = prices[i];
                dp[i] = 0;
            }else{
                dp[i] = prices[i] - minPrice;
            }
        }
        int res = Integer.MIN_VALUE;
        for (int x:dp){
            res = Math.max(res,x);
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





## 五、回溯法

👍**算法模板**：[回溯框架团灭排列、组合、子集问题](https://blog.csdn.net/weixin_42870497/article/details/119443910)

👍**矩阵搜索算法模板：**[DFS团灭矩阵中的搜索问题](https://blog.csdn.net/weixin_42870497/article/details/120048719)

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

**代码实现：**

```java
class Solution {

    int m, n;

    public boolean exist(char[][] board, String word) {

        m = board.length;
        n = board[0].length;

        boolean[][] visit = new boolean[m][n];
        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                if (dfs(board, word, i, j,0, visit)) {
                    return true;
                }
            }
        }

        return false;
    }

    private boolean dfs(char[][] board, String word, int i, int j, int index,boolean[][] visit) {

        if (i < 0 || i >= m || j < 0 || j >= n || board[i][j]!=word.charAt(index)|| visit[i][j]) {
            return false;
        }

        if (index == word.length()-1) {
            return true;
        }

        visit[i][j] = true;
        boolean res = dfs(board, word, i - 1, j, index+1,visit) ||
                        dfs(board, word, i + 1, j,index+1, visit) ||
                        dfs(board, word, i, j - 1, index+1,visit) ||
                        dfs(board, word, i, j + 1, index+1,visit);
        visit[i][j] = false;
        return res;
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



## 七、二分查找

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





## 九、其他

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



