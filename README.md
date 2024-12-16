# NetCTRL
## netcontrol
利用分流设置每天上传常用网站列表可以通过以下方式实现。分流工具可以是 Clash、Surge 或类似的工具。这需要一些自动化脚本和配置文件的协同工作。以下是基本步骤：

---

### **1. 准备一个常用网站列表**
- 创建一个文件（如 `common_sites.txt`），保存常用网站的域名，每行一个域名：
  ```
  google.com
  youtube.com
  twitter.com
  example.com
  ```

---

### **2. 自动生成分流规则**
利用脚本每日生成包含这些域名的分流规则。以下是一个 Python 示例：

```python
from datetime import datetime

def generate_rule_file(input_file, output_file):
    # 读取常用网站列表
    with open(input_file, 'r') as infile:
        domains = infile.readlines()

    # 构建规则内容
    rules = []
    for domain in domains:
        domain = domain.strip()
        if domain:  # 跳过空行
            rules.append(f"DOMAIN-SUFFIX,{domain},Proxy")  # 替换 'Proxy' 为实际策略组

    # 写入规则文件
    with open(output_file, 'w') as outfile:
        outfile.write("# Generated on {}\n".format(datetime.now().strftime("%Y-%m-%d %H:%M:%S")))
        outfile.write("\n".join(rules))

# 示例调用
generate_rule_file('common_sites.txt', 'custom_rules.conf')
```

- **输入文件**：`common_sites.txt`  
- **输出文件**：`custom_rules.conf`

---

### **3. 更新分流规则到代理配置**
假设使用 Clash，需将生成的规则文件合并到配置文件中：

#### 修改 Clash 配置文件（`config.yaml`）
在 `rules` 部分包含生成的规则文件：
```yaml
rules:
  - RULE-SET,custom_rules.conf
  - MATCH,DIRECT  # 确保最后有个兜底规则
```

---

### **4. 自动化脚本上传**
用脚本每天更新生成的规则，并上传到服务器或本地替换代理工具的配置。

以下是一个简单的自动化脚本：
```bash
#!/bin/bash

# 生成分流规则
python3 generate_rules.py

# 上传到远程服务器（若需要）
scp custom_rules.conf user@server:/path/to/clash/config/

# 或本地替换
cp custom_rules.conf /path/to/clash/config/

# 重启 Clash 服务（如需重载配置）
systemctl restart clash
```

将脚本加入到每日计划任务：
```bash
crontab -e
```

添加一行，设置每天运行时间（如凌晨 2 点）：
```bash
0 2 * * * /path/to/script/update_rules.sh
```

---

### **5. 验证**
1. 确保生成的 `custom_rules.conf` 文件语法正确。
2. Clash 的配置文件正确加载规则，并重启生效。
3. 测试代理是否正常分流常用网站。

这样每天的常用网站列表更新都会自动同步到代理规则中。