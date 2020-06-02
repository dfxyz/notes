# KMP算法
问：haystack里面找needle？

首先分析needle的内容，构造一个部分匹配值表，在字符匹配失败时快速移动下标。

例如：
needle: abbaabbabc
values: 0001123420

`values[i]`，即某个以`needle[i]`结尾的子串，与头部的`needle[:values[i]]`相等。构造部分匹配表时，可以利用先前的结果，在一趟循环里完成。
`needle[1]`，第一个b：
* 前一个字符的部分匹配值是0，检查`needle[0]`
* `needle[0]`不是b，所以当前字符的部分匹配值也是0
`needle[3]`，第二个a：
* 前一个字符的部分匹配值是0，检查`needle[0]`
* `needle[0]`是a，所以当前字符的部分匹配值是1
`needle[4]`，第三个a：
* 前一个字符的部分匹配值是1，检查`needle[1]`
* `needle[1]`不是a，根据部分匹配值1得知前一个字符与`needle[0]`相等，所以再检查`values[0]`
* `values[0]`是0，检查`needle[0]`
* `needle[0]`是a，所以当前字符的部分匹配值是1
`needle[5]`，第三个b：
* 前一个字符的部分匹配值是1，检查`needle[1]`
* `needle[1]`是b，所以当前字符的部分匹配值是2
`needle[8]`，最后一个b：
* 前一个字符的部分匹配值是4，检查`needle[4]`
* `needle[4]`不是b，根据部分匹配值4得知前一个字符与`needle[3]`相等，所以再检查`values[3]`
* `values[3]`是1，检查`needle[1]`
* `needle[1]`是b，所以当前字符的部分匹配值是2
`needle[9]`，c：
* 前一个字符的部分匹配值是2，检查`needle[2]`
* `needle[2]`不是c，根据部分匹配值2得知前一个字符与`needle[1]`相等，所以再检查`values[1]`
* `values[1]`是0，检查`needle[0]`
* `needle[0]`不是c，所以当前字符的部分匹配值是0

匹配示例：
  s     x
..abbaabc..
  abbaabbabc

      s x
..abbaabc.. 
      abbaabbabc # shift 3

试图匹配`haystack[i:]`，`haystack[i+6]`与`needle[6]`不匹配，`values[5] = 2`，所以接下来把`needle`右移`6-2`位，即接下里试图匹配`haystack[i+4:]`，继续比较`haystack[i+6]`与`needle[2]`。