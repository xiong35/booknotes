# 在用js替换字符串内容时

## replace函数只能替换匹配的第一个字符串

将匹配项改写成正则表达式:

```js
"foobar".replace("o", "O");
// "fOobar"

"foobar".replace(/o/g, "O"); // g表示global, 会匹配多项
// "fOObar"
```

# 实现返回顶部效果时

## 自己写js又麻烦又容易错

js 原生窗口滚动:

```js
window.scrollTo({
top: 0,
behavior: "smooth"
});
```