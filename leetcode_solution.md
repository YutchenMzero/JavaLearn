
### 解题常用方法、技巧与注意事项
#### **二分法** 
**其使用条件为数组有序且无重复元素**
解题时要明确解所处的区间是[left,right]还是[left,right)
* 循环边界判断
* 区间判断
```java
//以闭区间[left,right]为例
while(left<=right){
    ***
    if(left<right){
        right = mid-1;
        //开区间则为： right = mid;因为下一次循环中nums[mid]不参与比较
    }else{
        left = mid+1;
    }
}
```
#### **双指针**
**该方法即通过双指针遍历的方式，减少一层循环，从而降低时间复杂度**
需注意：
* 如何根据数据特征设计合适的遍历方式，以减少边界的判定，并保证所有的可能解都被考虑。
#### **滑动窗口**
**当需要确定最小连续长度时可考虑该方法**
解题时需要注意：
* 使用外层for循环表示终止位置的移动
* 确定窗口内的所需的数据条件
* 确定如何在保持窗口内条件的情况下移动起始位置
* 
#### **回溯法**
**当需要求出所有的可能的解或者需要在所有解中筛选出最优解时，可采用该方法。**
解题的一般步骤为：
1）针对所给问题，定义问题的解空间；
2）确定易于搜索的解空间结构；
3）以深度优先方式搜索解空间树，并在搜索过程中用剪枝函数避免无效搜索。
其基本框架如下(并不是一定)：
```java
void backtrack(int t){
    if(t > n){ 
        output(x);
    }else {
        for(int i = 0;i <= 1;i++){
            x[t] = i;
            if(constraint(t) && bound(t)){
                backtrack(t+1);//回溯过程隐含在参量传递中，即值的变化直接传给了下一层而不是其本身。
            }
        }//该部分的循环并不是必须的
    }
}
//t：表示递归深度，即当前扩展节点在解空间树中的深度。
//output()：纪录或输出结果的函数
//constaint()：约束函数。返回值=true，则满足约束条件；返回值=false，可剪去相应子树。
//bound()：限界函数。返回值=true，目标函数未越界，可用backtak(t+1)进一步搜索；返回值=false，可剪去相应子树。
```
#### **递归算法**
递归三要素
* **确定递归函数的参数和返回值**： 确定哪些参数是递归的过程中需要处理的，那么就在递归函数里加上这个参数，并且还要明确每次递归的返回值是什么进而确定递归函数的返回类型。
* **确定终止条件**： 即需要明确递归边界的判断条件。
* **确定单层递归的逻辑**： 确定每一层递归需要处理的信息与每次循环结束后的状态。
#### **迭代法实现二叉树深度优先遍历**
即使用栈模拟递归过程，需要注意的是，由于栈是后入先出的，因此树中节点需要按相反顺序入栈，即先序遍历的入栈顺序为：右左中。
* 以下提供一种统一风格的迭代遍历方式，其思想是通过加入空节点做标记来解决中间节点访问和处理不同步的问题。
```java
 public List<Integer> preorderTraversal(TreeNode root) {
        List<Integer> result = new LinkedList<>();
        Stack<TreeNode> st = new Stack<>();
        if (root != null) st.push(root);
        while (!st.empty()) {
            TreeNode node = st.peek();
            if (node != null) {
                st.pop(); // 将该节点弹出，避免重复操作，下面再将右中左节点添加到栈中
                if (node.right!=null) st.push(node.right);  // 添加右节点（空节点不入栈）
                if (node.left!=null) st.push(node.left);    // 添加左节点（空节点不入栈）
                st.push(node);                          // 添加中节点
                st.push(null); // 中节点访问过，但是还没有处理，加入空节点做为标记。
                
            } else { // 只有遇到空节点的时候，才将下一个节点放进结果集
                st.pop();           // 将空节点弹出
                node = st.peek();    // 重新取出栈中元素
                st.pop();
                result.add(node.val); // 加入到结果集
            }
        }
        return result;
    }
```
#### 贪心算法
**当可以通过每一阶段的最优达到全局最优时，考虑使用贪心算法**
值得注意的是，并没有什么固定的策略去判断是否能用贪心，最好的方法先手动模拟，检查是否可以通过局部最优达到全局最优，再尝试举反例，若想不到反例，即可使用。其一般的解题步骤为：
1） 将问题分解为若干个子问题
2） 找出适合的贪心策略
3） 求解每一个子问题的最优解
4） 将局部最优解堆叠成全局最优解

#### 技巧与注意事项
1. 在通过循环或迭代中，可以通过设置哨兵值，来简化边界判定条件；遍历寻找最优值（即符合题意的，常常是最大或最小值）时，结果的初值（也可以理解为哨兵）一定要考虑极限输入产生的边界结果，充分考虑不同输入的影响，否则算法会容易产生错误结果[^1] 
    [^1]:详见LeetCode.16的题目与提交记录
2. 利用`int mid = (l + r) >> 1;`的形式实现整除2；
3. 利用`int sign = ((a ^ b)>>31 & 0x1 ==1)?-1:1`判断两数的符号是否相同；
4. 由于整型在负数域的范围更大，所以可以考虑均转换到负数域上进行操作。
5. 当遇到“判断是否有相同”类型的情况下，可以考虑利用哈希结构实现。
6. 利用`set.stream().mapToInt(i->i).toArray();`将set转换成数组
7. 哈希结构，即可通过索引进行访问的结构，因此数组也是一种哈希结构，当key值的范围确定，且较小时，可使用数组代替Hashmap等，提升访问速度。
8. 对如“在一个字符串中是否出现过另一个字符串”的问题，可考虑使用KMP算法。
9. 通过Set遍历map中的元素
    ```java
    Set<Map.Entry<Integer,Integer>> set = map.entrySet();
        for (Map.Entry<Integer,Integer>entry: set) {
            queue.offer(entry);
        }
    ```
10. 完全二叉树可通过判断其左右遍历深度是否一致，确定其是不是满二叉树
### 例题
#### 中心扩散法求解最长回文串：
```java
public class Solution {
    public String longestPalindrome(String s) {
        int begin_index=0;
        int max_len=1;
        int len=s.length();
        char[] char_array=s.toCharArray();//若使用s.charAt(i)，每次都会判断是否溢出
        int odd_result,even_result;
        for(int i=0;i<len-1;i++){
            odd_result=getPalindrome(char_array,i,i);
            even_result=getPalindrome(char_array,i,i+1);
            if (Math.max(odd_result,even_result)>max_len){
                max_len=Math.max(odd_result,even_result);
                begin_index=i-(max_len-1)/2;
            }
        }
        return s.substring(begin_index,begin_index+max_len);
    }
    private int getPalindrome(char[] char_array,int start,int end){
        int len= char_array.length;
        while(start>=0&&end<len){
            if(char_array[start]!=char_array[end]){
                break;
            }
            start--;
            end++;
        }
        return end-start-1;
    }
    @Test
    public void test(){
        String s=new String("ccc");
        System.out.println(longestPalindrome(s));
    }
}
```
#### 动态规划法求解最长回文串
```java
public class Solution {
    public String longestPalindrome(String s) {
        int len =s.length();
        if (len<2){
            return s;
        }
        int begin = 0,max_len = 1;
        char[] char_array=s.toCharArray();
        boolean [][]dp =new boolean[len][len];
        for(int j=1;j<len;j++){
            for(int i=0;i<j;i++){
               if(char_array[i]==char_array[j]&&(j-i<3||dp[i+1][j-1])){
                   dp[i][j]=true;
                   if(j-i+1>max_len){
                       max_len=j-i+1;
                       begin=i;
                   }
               }
            }
        }
        return s.substring(begin,begin+max_len);
    }
    @Test
    public void test(){
        String s=new String("ccc");
        System.out.println(longestPalindrome(s));
    }
}
```