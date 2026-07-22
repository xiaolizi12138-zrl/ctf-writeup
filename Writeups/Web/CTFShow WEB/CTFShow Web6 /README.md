CTFShow Web6 题解 (Write-Up)
1. 题目信息
题目名称：ctf.show_web6

题目类型：Web / SQL 注入绕过

考察知识点：空格过滤绕过、联合查询注入 (Union-Based SQL Injection)

2. 题目分析
进入题目后，显示一个标准的登录表单。在常规测试中，输入带有空格的 SQL 语句（如 admin' or 1=1#）会触发后端拦截，并返回固定提示 sql inject error。这表明后端存在 WAF（Web 应用防火墙）或正则表达式黑名单过滤，但经过测试发现，拦截的目标是“空格”，而非 SQL 关键字。reload-alertreload-alertreload-alert

3. 漏洞利用与绕过思路
3.1 空格绕过原理
在 SQL 语法中，空格用于分隔关键字。后端若将用户输入的所有空白字符（%20）替换为空，会导致语句语法错误而无法执行。

绕过技巧：
在 MySQL 中，可以使用 /**/（多行注释符） 或 ()（括号） 来替代空格，且不会影响 SQL 语句的逻辑解析。

正常写法： select flag from flag

绕过写法： select/**/flag/**/from/**/flag 或 (select(flag)from(flag))

把 username 的值修改为下面这一整行（注意这一行里没有空格，全部用了 /**/ 代替）：

sql
admin'union(select/**/1,(select/**/flag/**/from/**/flag),3)#
(如果你不能抓包，直接在网页的用户名框里输入上面的内容，密码随便填，点登录)

原理回顾（你的笔记）：

后端防的是空格，所以我们在 union 和 select 之间，以及 select 和 flag 之间，全用 /**/（MySQL 注释符）代替了空格。

union(select 利用了括号 () 连接，连 /**/ 都省了。
