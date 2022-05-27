### 中心扩散法求解最长回文串：
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
### 动态规划法求解最长回文串
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