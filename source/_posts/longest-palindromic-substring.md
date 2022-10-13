---
title: Longest Palindromic Substring
tags:
  - algorithm
  - java
categories:
  - leetcode
---

Given a string s, return the longest palindromic substring in s.

A string is called a palindrome string if the reverse of that string is the same as the original string.

Example 1:
```
    Input: s = "babad"
    Output: "bab"
    Explanation: "aba" is also a valid answer.
```

## Solution

There are multiple ways to solve the problem. First we should try to start with the basic brute-force solution.

### Brute-force solution
In this way we will be checking all the possible substring and check if that's a palindrome. Taking the example, `babad` the substrings will be (`b`, `ba`, `bad`, `baba`, `babad`, `a`, `ab`, `aba`, `abad`, `b`, `ba`, `bad`, `a`, `ad`). For all the substrings we have to check if that's a palindrome or not.
```
If palindrome, 
    compare the length if it's the largest

    if largest
      store the value in a variable
```

To check if palindrome

{% codeblock lang:java %}
private static boolean isPalindrome(char[] arr, int start, int end) {
    if(start == end) {
        return true;
    }
    int len = (end - start) / 2;
    for (int i = 0; i < len; i++) {
        if(arr[start++] != arr[end--]) return false;
    }

    return true;
}
{% endcodeblock %}

Now we generate all the substrings and then check if palindrome or not.
{% codeblock lang:java %}
public String longestPalindrome(String s) {
    char[] arr = s.toCharArray();
    
    int max = -1;
    String ans = "";
    
    for(int i = 0; i < arr.length; i++) {
        for(int j = arr.length - 1; j >= i; j--) {
            if(isPalindrome(arr, i, j)) {
                int len = j - i;
                if(len > max) {
                    max = len;
                    ans = new String(arr, i, j-i+1);
                }  
            } 
        }
    }
    return ans;
}
{% endcodeblock %}

To generate the substring the complexity is \\(O(n^2)\\) and for each substring checking palindrome take \\(O(n)\\). In total the complexity of the solution is \\(O(n^3)\\)

### Dynamic programming
The brute-force solution works for solving the problem. Now we have to make the solution a bit better. We can see in our `isPalindrome` funtion we check for same substring if palindrome or not. We are going to store the value and reuse it so that we do not have to check again. We are solving the problem by using the result from our subproblem. Let's say, for string `ababa`, it's a palindrome if first and last character is same and the `bab` substring is palindrome.

We will take a \\((n*n)\\) boolean array to store values for each substring. `start` will be representing row and `end` will represent the column.

```
if start == end, means the same character will set true
if arr[start] == arr[end] 
    if end - start == 1; set true
    else set value from arr[start + 1][end - 1]  // (for `ababa` set value of `bab`)
```

Let's write the solution.

{% codeblock lang:java %}
public static String longestPalindrome(String s) {
    int n = s.length();
    if(n < 2) {
        return s;
    }
    char[] arr = s.toCharArray();

    int max = -1;
    String ans = "";
    boolean[][] dp = new boolean[n][n];

    for(int i = n - 1; i >= 0; i--) {
        for(int j = i; j < n; j++) {
            if(i == j) dp[i][j] = true;
            else if(arr[i] == arr[j]) {
                if(j - i == 1) dp[i][j] = true;
                else {
                    dp[i][j] = dp[i+1][j-1];
                }
            }

            int len = j - i + 1;
            if(dp[i][j] && len > max) {
                max = len;
                ans = new String(arr,i, len);
            }
        }
    }
    return ans;
}
{% endcodeblock %}

In this solution we are building our boolean \\((n*n)\\) in bottom up approach. For our example `cbaba` from last the order of substring will be 
```
(4,4) -> a -> i == j -> true
(3,3) -> b -> i == j -> true
(3,4) -> ba -> arr[i] != arr[j] -> false
(2,2) -> a -> i == j -> true
(2,3) -> ab -> arr[i] != arr[j] -> false
(2,4) -> aba -> arr[i] == arr[j] -> (3, 3) -> true
(1,1) -> b -> i == j -> true
(1,2) -> ba -> arr[i] != arr[j] -> false
(1,3) -> bab -> arr[i] == arr[j] -> (2, 2) -> true
(1,4) -> baba -> arr[i] != arr[j] -> false
(0,0) -> c -> i == j -> true
(0,1) -> cb -> arr[i] != arr[j] -> false
(0,2) -> cba -> arr[i] != arr[j] -> false
(0,3) -> cbab -> arr[i] != arr[j] -> false
(0,4) -> cbaba -> arr[i] != arr[j] -> false
```

if the result value is true, we will then check if the length is longest or not. We will store the longest string in a variable and after the loop is finished we will get the result.

We just removed the use of `isPalindrome` function here but the loop will remain the same. So, for this solution the complexity will be \\(O(n^2)\\). But in this case we are using an extra memory to store the values and that's why we will have an extra \\(O(n^2)\\) memory.

### Expand around the center
In the brute-force approach, we checked if the string is palindrome by starting from both the edges and traverse to center. In this approach we will start from the center expand one by one level to check if the string is palindrome. For example, in `atoyota` we will start with `y` then `oyo`, `toyot` and lastly `atoyota`. Here for every position we will expand and check if palindrome or not.

{% codeblock lang:java %}
private static String expand(char[] arr, int start, int end) {
    while(start >= 0 && end < arr.length && arr[start] == arr[end]) {
        start--;
        end++;
    }
    return new String(arr, start+1, (end - 1) - (start + 1) + 1);
}
{% endcodeblock %}

If we call this `expand` function, we'll get the longest palindrome of that position.
(`atoyota`, 3, 3) -> `atoyota`
(`aba`, 1, 1) -> `aba`
We can see it's working for odd length string with giving the same position. For even length, we have to pass two positions(including the next) to keep it working.
(`abba`, 1, 2) -> `abba`
(`cc`, 0, 1) -> `cc`

The next step will be loop through all the positions in the string to get the longest palindrome.

{% codeblock lang:java %}
public String longestPalindrome(String s) {
    if(s.length() < 2) {
        return s;
    }
    char[] arr = s.toCharArray();

    int max = -1;
    String ans = "";

    for(int i = 0; i < arr.length - 1; i++) {
        String x = expand(arr, i, i);     // Odd 
        String y = expand(arr, i, i+1);   // Even (add the next position i+1)
        
        int len = Math.max(x.length(), y.length());
        if(len > max) {
            max = len;
            ans = len == x.length() ? x : y;
        }
    }
    return ans;
}
{% endcodeblock %}

In this approach, we are only looping through the length of the string and expanding to the edge. For looping through all the positions the complexity will be \\( O(n) \\) and to expand the complexity is \\( O(n) \\). The total complexity of the solution is \\( O(n^2) \\) and having the memory complexity of \\( O(n) \\). The time complexity is similar to our dynamic programming solution but the advantage is the memory complexity.

## Conclusion
There is also another algorithm named [Manacher's Algorithm](https://en.wikipedia.org/wiki/Longest_palindromic_substring#Manacher's_algorithm) where the complexity is \\( O(n) \\).  
