# 为 Twikoo 静态网站评论系统配置邮件通知功能（利用 Mailgun 免费额度）

在部署了 Twikoo 评论系统后，邮件通知功能一直未被启用。直到最近检查文章时，才发现一个月前的用户留言 🤦 尽管已回复，但恐怕为时已晚。为了避免类似问题再次发生，决定为评论系统配置邮件通知功能。

## 操作步骤

### 一、注册 Mailgun 账号并绑定域名

访问 [Mailgun 官网](https://www.mailgun.com/) 注册账号。Mailgun 提供每日 100 封邮件的免费发信额度，完全满足个人博客的需求。

![Mailgun 价格](https://bbtdd.com/img/883339681.webp)

> 提示：免费额度需绑定信用卡。若无信用卡，可通过虚拟信用卡服务轻松解决。👉 [WildCard | 一分钟注册，轻松订阅海外线上服务](https://bbtdd.com/WildCard)

注册完成后，绑定域名。以博客域名 `senjianlu.com` 为例，操作如下：

![添加自定义域名](https://bbtdd.com/img/55253055171629.webp)  
![输入域名](https://bbtdd.com/img/0300622287583287.webp)

根据提供的 DNS 信息验证域名：

![添加 DNS 解析](https://bbtdd.com/img/4629720870.webp)  
![验证域名](https://bbtdd.com/img/1044323547651769.webp)  
![验证成功](https://bbtdd.com/img/081308135108241.webp)

### 二、获取 Mailgun 的 SMTP 信息

在 Mailgun 控制台，进入 `Send` -> `Sending` -> `Domains`，找到对应域名：

![域名列表](https://bbtdd.com/img/666713185866.webp)  
![域名可用状态](https://bbtdd.com/img/178493759833069.webp)

进入域名详情页，点击 `SMTP credentials` 查看 SMTP 信息。默认创建一个 `postmaster` 用户：

![SMTP 用户](https://bbtdd.com/img/89005938251.webp)

重置密码并保存，得到完整的 SMTP 信息。

### 三、测试 SMTP 配置

使用 Python 编写测试脚本，确保发信功能正常：

python
import smtplib
from email.mime.text import MIMEText
from email.header import Header

# 邮件内容
msg = MIMEText('Hello, this is a test email.', 'plain', 'utf-8')
msg['Subject'] = Header('这是一封内部测试邮件', 'utf-8')
msg['From'] = 'postmaster@senjianlu.com'
msg['To'] = '测试邮件接收者'

# SMTP 配置
smtp_server = 'smtp.mailgun.org'
smtp_port = 587
smtp_user = 'postmaster@senjianlu.com'
smtp_password = 'your_password'

# 收件人列表
to_addrs = ['recipient1@example.com', 'recipient2@example.com']

# 发送邮件
try:
    server = smtplib.SMTP(smtp_server, smtp_port)
    server.starttls()
    server.login(smtp_user, smtp_password)
    server.sendmail(smtp_user, to_addrs, msg.as_string())
    print('邮件发送成功')
except Exception as e:
    print('邮件发送失败:', e)
finally:
    server.quit()


> 提示：建议使用多个收件人测试，确保配置无误。

### 四、配置 Twikoo 评论系统的邮件通知功能

进入博客评论模块，点击右上角的设置图标：

![评论模块设置](https://bbtdd.com/img/669919817182.webp)

选择配置管理，并配置邮件通知：

![配置管理](https://bbtdd.com/img/9730862263475.webp)  
![邮件通知配置 01](https://bbtdd.com/img/5343535499843035.webp)  
![邮件通知配置 02](https://bbtdd.com/img/6803440297890.webp)  
![邮件通知配置 03](https://bbtdd.com/img/35151355549.webp)

保存配置后，可通过测试邮件验证功能是否正常：

![发送测试邮件](https://bbtdd.com/img/999413807998.webp)  
![测试邮件成功](https://bbtdd.com/img/042627886019.webp)

### 五、测试邮件通知功能

#### 1、配置管理员邮箱
当用户评论时，管理员邮箱会收到通知：

![管理员邮箱设置](https://bbtdd.com/img/6114713753207553.webp)

#### 2、测试用户评论
用户发布评论后，管理员邮箱收到通知：

![用户评论](https://bbtdd.com/img/219247461721233.webp)  
![评论成功](https://bbtdd.com/img/17089268270651.webp)  
![管理员收到邮件](https://bbtdd.com/img/330634134114890.webp)

#### 3、测试回复评论
管理员回复评论后，用户邮箱收到通知：

![回复评论](https://bbtdd.com/img/2713991766.webp)  
![回复成功](https://bbtdd.com/img/684828130.webp)  
![用户收到邮件](https://bbtdd.com/img/448553420336489.webp)

### 六、设置 Mailgun 发信上限

进入 Mailgun 控制台的 `Manage Account`，设置 `Custom Message Limit`：

![自定义消息限制](https://bbtdd.com/img/1956245468723148.webp)

根据需求设置月度发信上限，最小值为 1000：

![设置发信上限](https://bbtdd.com/img/84183976876871.webp)

至此，Twikoo 评论系统的邮件通知功能已成功配置，确保您不会错过任何用户互动。