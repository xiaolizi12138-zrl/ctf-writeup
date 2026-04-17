
# 响应头中的Flag

[![类型](https://img.shields.io/badge/类型-Web-blue)](https://github.com)
[![考点](https://img.shields.io/badge/考点-HTTP响应头-brightgreen)](https://github.com)
[![难度](https://img.shields.io/badge/难度-⭐-green)](https://github.com)

---

## 📝 题目描述

> 访问页面，只显示一行字：`什么也没有。`

页面内容为空，没有任何其他信息。

---

## 🔍 解题过程

### 第一步：查看响应头

在浏览器控制台（<kbd>F12</kbd> → Console）执行：

```javascript
fetch(location.href).then(r => {
    for(let [k,v] of r.headers) {
        console.log(k + ': ' + v);
    }
});

第二步：发现 Flag
<font color="green">✅ 响应头中直接包含 flag 字段：</font>

text
connection: keep-alive
content-encoding: gzip
content-type: text/html
date: Thu, 16 Apr 2026 14:42:25 GMT
flag: flag{521f6a35286f74393169543ed60749bf}
server: Apache/2.4.7 (Ubuntu)
第三步：获取 Flag
text
flag{521f6a35286f74393169543ed60749bf}
