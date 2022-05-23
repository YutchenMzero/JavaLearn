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
