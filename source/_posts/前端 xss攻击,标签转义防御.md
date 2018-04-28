title: 前端 xss攻击,标签转义防御
date: 2017-10-18 00:00:00
tags: [xss]

---
[TOC]

---
# 一.xss机制
当系统`将用户输入的内容重新加载到浏览器`,且未进行标签字符转义的情况下,就存在xss攻击的风险

# 二.危害:不是alert弹窗那么简单
黑客添加的xss代码,可能会被自己外的任何人访问,如xss代码中保护获取用户token再上报给黑客.
在这种情况下,黑客就能模拟其它用户的账户进行任意操作(且不需要用户名,密码.以为后端只看session,sessionid却在token里)

示例: 免费QQ空间挂件 / 自动发帖 / ...

---
# 三.前端防御
提交数据或展示数据时,对内容进行转义
```
function HTMLEncode(html) {
    var temp = document.createElement("div");
    (temp.textContent != null) ? (temp.textContent = html) : (temp.innerText = html);
    var output = temp.innerHTML;
    temp = null;
    return output;
}
function HTMLDecode(text) { 
    var temp = document.createElement("div"); 
    temp.innerHTML = text; 
    var output = temp.innerText || temp.textContent; 
    temp = null; 
    return output; 
} 

var tagText = "<p><b>123&456</b></p>";
var encodeText = HTMLEncode(tagText);
console.log(encodeText);//<p><b>123&456</b></p>
console.log(HTMLDecode(encodeText)); //<p><b>123&456</b></p>
```

- 调试:
```
$0.innerHTML
"<img src="#" onerror="‘javaScript:alert(2);’/">"

$0.innerText = $0.innerHTML
"<img src="#" onerror="‘javaScript:alert(2);’/">"

$0.innerHTML
"&lt;img src="#" onerror="‘javaScript:alert(2);’/"&gt;"

$0.innerText
"<img src="#" onerror="‘javaScript:alert(2);’/">"
```

---
# 四.后端防御
存储数据,对标签进行转义
## 方法一: StringEscapeUtils.escapeHtml() / HtmlUtils.htmlEscape()
```
System.out.println(StringEscapeUtils.escapeHtml("<img></img>"));// &lt;img&gt;&lt;/img&gt;
System.out.println(StringEscapeUtils.unescapeHtml(StringEscapeUtils.escapeHtml("<img></img>")));// <img></img>

System.out.println(HtmlUtils.htmlEscape("<img></img>"));// &lt;img&gt;&lt;/img&gt;
System.out.println(HtmlUtils.htmlUnescape(StringEscapeUtils.escapeHtml("<img></img>")));// <img></img>
// 风险
System.out.println("------------------风险:{\"key\":\"<img></img>\"}");
System.out.println(HtmlUtils.htmlEscape("{\"key\":\"<img></img>\"}")+" 双引号也被替换,将导致json无法解析");
```

`java后台对前端输入的特殊字符进行转义 - 自行车上的程序员 - 博客园`
http://www.cnblogs.com/yangzhilong/p/5667165.html

`利用StringEscapeUtils对字符串进行各种转义与反转义（Java） - CSDN博客`
http://blog.csdn.net/chenleixing/article/details/43456987

## 方法二: spring filter
`spring mvc xss filter - CSDN博客`
http://blog.csdn.net/xiangtaoxiangtao/article/details/51865808

`在SpringMVC中使用过滤器（Filter）过滤容易引发XSS | zifangsky的个人博客`
https://www.zifangsky.cn/683.html

```
springmvc注解@RequestParam不是通过HttpServletRequest.java的getParameter(String name)方法得到的参数值，而是通过getParameterValues得到的.所以你懂的菊花刀伺候！
```
`SpringMVC如何有效的防止XSS注入？`
https://www.zhihu.com/question/32486587?sort=created

`SpringMVC 重写HttpMessageConverter进行Xss过滤`
https://my.oschina.net/jayhu/blog/653725