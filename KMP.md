# KMP算法
问：haystack里面找needle？

首先分析needle的内容，构造一个等同于needle长度的位移表，在字符匹配失败时快速移动下标。

例如：
needle: abbaabbab
values: 000112342

`values[i]`，即`needle[:i+1]`中，相等的真前缀与真后缀的最大长度。计算时，可以利用先前的结果，在一趟大循环里完成。
第一个b：
* 前一个字符的值是0，检查`needle[0]`
* `needle[0]`不是b，所以值也是0
第二个a：
* 前一个字符的值是0，检查`needle[0]`
* `needle[0]`是a，所以值是`0 + 1`
第三个a：
* 前一个字符的值是1，检查`needle[1]`
* `needle[1]`不是a，`values[1]`是0，所以再检查`needle[0]`
* `needle[0]`是a，所以值是`0 + 1`
第三个b：
* 前一个字符的值是1，检查`needle[1]`
* `needle[1]`是b，所以值是`1 + 1`
最后一个b：
* 前一个字符的值是4，检查`needle[4]`
* `needle[4]`不是b，`values[4]`是1，所以再检查`needle[1]`
* `needle[1]`是b，所以最后一个b的值是`1 + 1`

匹配示例：
  s     x
..abbaabc..
  abbaabbab

      s x
..abbaabc.. 
      abbaabbab # shift 3

试图匹配`haystack[i:]`，`haystack[i+6]`，`needle[6]`不匹配，`values[5] = 2`，所以接下来把`needle`右移`6-2`位，即接下里试图匹配`haystack[i+4:]`，继续比较`haystack[i+6]`与`needle[2]`。