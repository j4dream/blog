# XSS 与 CSRF

XSS 与 CSRF 是 Web 常见漏洞，现在大部分框架都会处理这些问题，但有时候开发者又会不小心忽略了这个问题，从而导致严重的漏洞。

## XSS
XSS 是由于没有过滤用户输入内容导致的，通常在一些 input, textarea 输入了一些 javascript 代码，这些代码可能会将 cookies，token 发送出去，导致用户信息被盗用。
```javascript
// 输入以下内容保存后，如果浏览器出现 alert 即存在问题。
<script>
  alert('XSS Attact');
</script>
```

### 如何防范
最简单方法，前端过滤所有所有输出，保存数据时，后端也需要处理数据。
```javascript
function encodeHtml(str) {
  return (str + '')
           .replace(/&/g, '&amp;')
           .replace(/</g, '&lt;')
           .replace(/>/g, '&gt;')
           .replace(/\x60/g, '&#96;')
           .replace(/\x27/g, '&#39;')
           .replace(/\x22/g, '&quot;');
};

```
如果有需求允许用户输入 javascipt 怎么办？我曾做过的其中一个系统，就是要可以允许用户通过 编写 javascript 执行一些操作。允许这种操作风险非常大，但又不得不做，这种情况下，只能将风险降到最低，将这部分内容放在另一个域名下，这个域名不含有用户登录信息，由于不遵守同源策略，这就不会影响原有系统，就算出问题的话只会影响这部分。
```javascript
sample.a.com -> 原用户输入
preview.b.com -> 将需要输出 javascipt 放在另外域名下。
```
最后还有一个需要函数注意 eval(str), 这个函数可以将字符串转换成可执行的函数代码。

## CSRF
CSRF 跨站请求伪造，最常见的就是钓鱼网站，界面跟真的一样，用户做相应操作，它也能将数据发送到原始网站，但它可以可以将修改过的数据再传送，这能实现需要一个前提条件，就是你已经登录过真的网站，并且保存在cookies的授权信息未过期。


```
graph LR
A--首先访问---B
A--再访问-->C
C--利用漏洞-->B
```

### 如何防范
在请求数据时，后端生成一个随机字符串 token 给前端，前端可以将 token 放到 Request Headers 或者 url 中，提交数据后再校验 token 是否合法。
