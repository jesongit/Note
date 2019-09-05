## 题目描述
给你一个字符串 s 和一个字符规律 p，请你来实现一个支持 '.' 和 '*' 的正则表达式匹配。
~~~
'.' 匹配任意单个字符
'*' 匹配零个或多个前面的那一个元素
~~~
#### 说明
s 可能为空，且只包含从 a-z 的小写字母。
p 可能为空，且只包含从 a-z 的小写字母，以及字符 . 和 *。
#### 示例
**输入**:
s = "aa"
p = "a"
**输出**: false
## 分析
#### 1.首先来试试正常的字符串匹配（字符串不包含 . 和 * ）
~~~Java
public boolean match(int i, int j, String s, String p) {
    if(j == p.length())  // 模式串匹配完毕，匹配串匹配完则匹配完成
        return i == s.length();
    return s.charAt(i) == p.charAt(j) && match(i+1, j+1, s, p);
}
~~~
#### 2.接下来模式串加入字符 .
~~~Java
public boolean match(int i, int j, String s, String p) {
    if(j == p.length())
        return i == s.length();
    return (s.charAt(i) == p.charAt(j) || p.charAt(j) == '.') && match(i+1, j+1, s, p);
}
~~~
#### 3.再加入符号 *
~~~Java
public boolean match(int i, int j, String s, String p) {
    if(j == p.length())
        return i == s.length();
    boolean flag = i < s.length() && (s.charAt(i) == p.charAt(j) || p.charAt(j) == '.');
    // 前面可以保证 j 不会到 p.length(), 但是 i 有可能到 s.length()
    if(j+1 < p.length() && p.charAt(j+1) == '*') {  // 出现 *
        return (flag && match(i+1, j, s, p)) || match(i, j+2, s, p);
    } else {
        return flag && match(i+1, j+1, s, p);
    }
}
~~~
#### 4.因为回溯的原因可以存在之前已经匹配过的 i 和 j，存起来减少递归次数
> 引用自[官方题解](https://leetcode-cn.com/problems/regular-expression-matching/solution/zheng-ze-biao-da-shi-pi-pei-by-leetcode/)
~~~Java
    enum Result {  // 使用 enum 更方便(可以直接判断空)，更省空间(相较于int)
        TRUE, FALSE
    }
    Result[][] memo;
    public boolean isMatch(String text, String pattern) {
        memo = new Result[text.length() + 1][pattern.length() + 1];
        return dp(0, 0, text, pattern);
    }

    public boolean dp(int i, int j, String text, String pattern) {
        if (memo[i][j] != null) {
            return memo[i][j] == Result.TRUE;
        }
        boolean ans;
        if (j == pattern.length()){
            ans = i == text.length();
        } else{
            boolean first_match = (i < text.length() &&
                    (pattern.charAt(j) == text.charAt(i) ||
                            pattern.charAt(j) == '.'));

            if (j + 1 < pattern.length() && pattern.charAt(j+1) == '*'){
                ans = (dp(i, j+2, text, pattern) ||
                        first_match && dp(i+1, j, text, pattern));
            } else {
                ans = first_match && dp(i+1, j+1, text, pattern);
            }
        }
        memo[i][j] = ans ? Result.TRUE : Result.FALSE;
        return ans;
    }
~~~
dp[i][j] 代表的是当匹配串到 i ， 模式串到 j 时的状态。
其中有一个状态是一定是已知的。dp[s.length()][p.length()]
于是产生了官方题解中的自底向上的方法，不用递归，进一步提高了时间效率
#### 5.自底向上的动态规划（非递归）
> 引用自[官方题解](https://leetcode-cn.com/problems/regular-expression-matching/solution/zheng-ze-biao-da-shi-pi-pei-by-leetcode/)
~~~Java
class Solution {
    public boolean isMatch(String text, String pattern) {
        boolean[][] dp = new boolean[text.length() + 1][pattern.length() + 1];
        dp[text.length()][pattern.length()] = true;

        for (int i = text.length(); i >= 0; i--){
            for (int j = pattern.length() - 1; j >= 0; j--){
                boolean first_match = (i < text.length() &&
                                       (pattern.charAt(j) == text.charAt(i) ||
                                        pattern.charAt(j) == '.'));
                if (j + 1 < pattern.length() && pattern.charAt(j+1) == '*'){
                    dp[i][j] = dp[i][j+2] || first_match && dp[i+1][j];
                } else {
                    dp[i][j] = first_match && dp[i+1][j+1];
                }
            }
        }
        return dp[0][0];
    }
}
~~~