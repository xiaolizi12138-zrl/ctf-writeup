📝 题目描述
题目提供了一个在线 Shell 命令执行环境：


<img width="1919" height="1003" alt="image" src="https://github.com/user-attachments/assets/e259a9b3-dc67-4574-aab6-53bbe87e925c" />
输入命令后显示 execute success!，但看不到命令的输出结果。

目标：读取服务器上的 flag 文件。
🔍 解题过程
第一步：探测环境
输入 ls：

text
ls
结果：

text
execute success!
没有输出内容，说明：

✅ 命令可以执行

❌ 但输出被屏蔽了

第二步：尝试绕过输出屏蔽
使用 || 逻辑或运算符。

原理：|| 左边的命令执行失败时，会执行右边的命令。

构造 Payload：

text
xxx || cat flag.php
xxx：一个不存在的命令，执行失败

cat flag.php：因为左边失败，|| 触发右边命令执行

结果：仍然只显示 execute success!，没有 flag。

说明 cat 命令可能被过滤或输出被屏蔽。

第三步：用 tac 替代 cat
tac 是 cat 的反向输出命令（从最后一行开始输出）。

text
xxx || tac flag.php
<font color="green">✅ 成功！</font>

输出结果：

text
$flag = "CTF{no_space_to_execute_shell_commands}";
🏁 Flag
text
CTF{no_space_to_execute_shell_commands}

❓ 为什么 tac 有效而 cat 无效？
命令	结果	可能原因
cat	❌ 无输出	被过滤规则匹配，返回假成功
tac	✅ 有输出	未被过滤，真正执行并返回结果
核心结论：当常用命令（cat、ls）失效时，尝试它们的变体或替代命令往往能绕过过滤
