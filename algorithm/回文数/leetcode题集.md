### 回文数汇总

#### 5 最大回文子串

> 详情可见动态规划

代码如下：

```java
/**
 * dp[i][j] 表示字符 s[i,j] 是否是回文串：
 * 由于 dp[i][j] 需要 dp[i + 1][j - 1], 因此从后向前，先计算 i == n - 2 的情况
 */
public String longestPalindrome(String s) {
    int n = s.length();
    if (n < 2) return s;
    int start = 0, end = 0;

    boolean[][] dp = new boolean[n][n];
    for (int i = 0; i < n; ++i) {
        dp[i][i] = true;
    }
    for (int i = n - 2; i >= 0; --i) {
        for (int j = i + 1; j < n; ++j) {
            if (s.charAt(i) == s.charAt(j)) {
                dp[i][j] = j == i + 1 || dp[i + 1][j - 1];
                if (dp[i][j] && (end - start) < (j - i)) {
                    start = i;
                    end   = j;
                }
            }
        }
    }
    return s.substring(start, end + 1);
}
```





#### 131 分割回文串

##### 解法1：在5基础上修改

```java
List<List<String>> result = new LinkedList<>();

public List<List<String>> partition(String s) {
    int n = s.length();
    boolean[][] dp = new boolean[n][n];
    for (int i = 0; i < n; ++i) {
        dp[i][i] = true;
    }

    for (int i = n - 2; i >= 0; --i) {
        for (int j = i + 1; j < n; ++j) {
            if (s.charAt(i) == s.charAt(j)) {
                dp[i][j] = j == i + 1 || dp[i + 1][j - 1];
            }
        }
    }
    // 截取字符串
    helper(dp, s, 0, new LinkedList<>());
    return result;
}


/**
 * 根据 dp 记录，回溯获取结果
 */
void helper (boolean[][] dp, String s, int i, LinkedList<String> temp) {
    if (i >= s.length()) {
        List<String> copy = new LinkedList<>(temp);
        result.add(copy);
        return;
    }
    for (int k = i; k < s.length(); ++k) {
        if (dp[i][k]) {
            temp.add(s.substring(i, k + 1));
            helper(dp, s, k + 1, temp);
            temp.removeLast();
        }
    }
}
```

