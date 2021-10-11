- CSDN非登录复制代码:  Array.from(document.querySelectorAll('code')).map(item=> {item.style.userSelect = 'text'})   
- 浏览器退出检测禁止
```javascript
window.onblur=function(){}
document.onkeydown = function() { return true;};
document.body.oncopy = function() { return true;};
document.onselectstart = function() { return true;};
document.oncontextmenu = function() { return true;};                        
```
