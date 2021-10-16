# LeetCode题解思路

## 哈希表

### 1、两数之和

题目链接：[两数之和](https://leetcode-cn.com/problems/two-sum/)

解题思路：

1. 遍历一遍数组，先把所有数存进hashmap中
2. 再遍历一遍数组，在hashmap中查找target-nums[i]的值是否存在，若存在，返回i和map.get(target-nums[i])

实现代码：

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

