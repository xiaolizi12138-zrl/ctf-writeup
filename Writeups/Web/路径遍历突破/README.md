📝 题目描述
题目是一个文件管理助手，可以读取文件。

目标：读取 /flag.txt 文件。

🔍 解题过程
第一步：探测环境
打开页面，提示：

text
目标flag文件为/flag.txt
尝试直接输入 /flag.txt：

结果：<img width="1919" height="985" alt="image" src="https://github.com/user-attachments/assets/8caa6abc-37ce-4c35-955b-11dda4389641" />


text
禁止以/或者../开头的文件名
说明服务器禁止了以 / 或 ../ 开头的路径。

第二步：读取源码
输入 index.php，成功读取到源码。
<img width="1919" height="1025" alt="image" src="https://github.com/user-attachments/assets/6c85037c-d0a4-4ddd-acb9-a0204c8c6898" />

关键过滤规则：

php
# 禁止以 / 或者 ../ 开头的文件名
if(preg_match('/^(\.|\/)/', $path)){
    echo '禁止以/或者../开头的文件名';
    exit;
}
正则 '/^(\.|\/)/' 匹配以 . 或 / 开头的字符串。

第三步：分析绕过方法
过滤只检查开头，不检查中间内容。

所以我们可以：

在路径前面加一个字母（不是 . 或 /）

然后跟 ../ 向上跳转

例如：a/../../../../../../flag.txt

第四步：构造 Payload
text
a/../../../../../../flag.txt
为什么这样写？

部分	作用
a/	绕过开头检测（以字母开头，不是 . 或 /）
../../../../../../	向上跳转到根目录
flag.txt	目标文件
第五步：获取 Flag
输入 a/../../../../../../flag.txt，成功读取：
<img width="1916" height="1059" alt="image" src="https://github.com/user-attachments/assets/0e77ca09-4ce6-4590-badf-8006358d63ab" />


text
CTF{file_path_bypass_is_fun}
🏁 Flag
text
CTF{file_path_bypass_is_fun}
💡 学到了什么
1. 路径遍历的原理
路径遍历（Path Traversal）利用 ../ 向上跳转目录，读取系统文件。

bash
../../../../etc/passwd
2. 过滤规则的常见缺陷
很多过滤只检查开头，不检查中间。例如：

php
if(preg_match('/^(\.|\/)/', $path)){
    exit;
}
这个正则只拦截以 . 或 / 开头的路径，但 a/../../flag.txt 以字母开头，完美绕过。

3. 路径规范化的特性
当执行 file_get_contents("a/../../flag.txt") 时：

系统会先解析路径中的 ..

a/ 和 .. 相互抵消

最终读取的是 /flag.txt

中间的 a 目录不需要真实存在，因为路径规范化的过程会把它抵消掉。

4. 信息收集的重要性
通过读取 index.php 源码，我们：

知道了过滤规则

找到了绕过方法

读取源码永远是第一步。
