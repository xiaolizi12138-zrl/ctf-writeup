

---

## 题目二：无限刷新与Python脚本

**文件路径**：`Writeups/Web/2-无限刷新与Python脚本/README.md`

```markdown
# 无限刷新与Python脚本

## 题目信息
| 项目 | 内容 |
|------|------|
| 类型 | Web |
| 考点 | 页面自动刷新、Python脚本循环请求、概率命中 |
| 难度 | ⭐⭐ |

## 题目描述

页面每 500ms 自动刷新一次，每次刷新图片数字变化。题目提示：**"Stop at panda ! u will get flag"**

## 解题过程

### 第一步：分析页面源码

查看源码，发现关键代码：

```html
<script language="JavaScript">
function myrefresh(){
    window.location.reload();
}
setTimeout('myrefresh()',500); 
</script>

页面每 0.5 秒刷新一次。图片 src 不断变化（6.jpg → 5.jpg → 其他数字），但所有 .jpg 都返回 404。

第二步：理解题目机制
flag 只会在某一帧（恰好是 panda 时）出现在 HTML 中。由于刷新太快，肉眼无法捕捉，需要用脚本循环请求。

第三步：编写 Python 脚本
python
import requests
import re

url = "http://171.80.2.169:16563"
headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36'
}

print("开始抓取，共尝试 200 次...")

for i in range(1, 201):
    try:
        r = requests.get(url, headers=headers, timeout=3)
        html = r.text
        
        # 检查 flag
        flag_match = re.search(r'flag\{[^}]+\}', html)
        if flag_match:
            print(f"\n第 {i} 次请求找到 flag！")
            print(f"Flag: {flag_match.group()}")
            break
        
        # 进度显示
        if i % 20 == 0:
            print(f"已请求 {i} 次")
            
    except Exception as e:
        print(f"请求失败: {e}")
第四步：获取 Flag
运行脚本，在第 20 次请求时成功获取 flag。

text
第 20 次请求找到 flag！
Flag: flag{76bb0d436281167c7726bbc5c4170cc7}
