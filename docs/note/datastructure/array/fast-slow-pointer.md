- 给你一个数组 nums 和一个值 val，你需要 原地 移除所有数值等于 val 的元素。元素的顺序可能发生改变。然后返回 nums 中与 val 不同的元素的数量。
- 假设 nums 中不等于 val 的元素数量为 k，要通过此题，您需要执行以下操作：
- 更改 nums 数组，使 nums 的前 k 个元素包含不等于 val 的元素。nums 的其余元素和 nums 的大小并不重要。
- 返回 k。

示例 1：
输入：nums = [3,2,2,3], val = 3
输出：2, nums = [2,2,_,_]
解释：你的函数函数应该返回 k = 2, 并且 nums 中的前两个元素均为 2。

示例 2：
输入：nums = [0,1,2,2,3,0,4,2], val = 2
输出：5, nums = [0,1,4,0,3,_,_,_]
解释：你的函数应该返回 k = 5，并且 nums 中的前五个元素为 0,0,1,3,4。
注意这五个元素可以任意顺序返回。 

解法:
暴力解法:
``` java
class Solution {
public int removeElement(int[] nums, int val) {
// 暴力法
int n = nums.length;
for (int i = 0; i < n; i++) {
if (nums[i] == val) {
for (int j = i + 1; j < n; j++) {
nums[j - 1] = nums[j];
}
i--;
n--;
}
}
return n;
}
}
```
快慢指针法:
``` java
class Solution {
public int removeElement(int[] nums, int val) {
// 快慢指针
int slowIndex = 0;
for (int fastIndex = 0; fastIndex < nums.length; fastIndex++) {
if (nums[fastIndex] != val) {
nums[slowIndex] = nums[fastIndex];
slowIndex++;
}
}
return slowIndex;
}
}
```
相向双指针法:
```java
class Solution {
public int removeElement(int[] nums, int val) {
int left = 0;
int right = nums.length - 1;
while(right >= 0 && nums[right] == val) right--; //将right移到从右数第一个值不为val的位置
while(left <= right) {
if(nums[left] == val) { //left位置的元素需要移除
//将right位置的元素移到left（覆盖），right位置移除
nums[left] = nums[right];
right--;
}
left++;
while(right >= 0 && nums[right] == val) right--;
}
return left;
}
}
```
相向双指针法（版本二）
```java
class Solution {
public int removeElement(int[] nums, int val) {
int left = 0;
int right = nums.length - 1;
while(left <= right){
if(nums[left] == val){
nums[left] = nums[right];
right--;
}else {
// 这里兼容了right指针指向的值与val相等的情况
left++;
}
}
return left;
}
}
```