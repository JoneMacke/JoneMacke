import imaplib
import email
from email.header import decode_header

# 账号信息
USERNAME = 'your_email@gmail.com'
PASSWORD = 'your_password'

# 连接到 Gmail 的 IMAP 服务器
mail = imaplib.IMAP4_SSL('imap.gmail.com')

# 登录到邮箱
mail.login(USERNAME, PASSWORD)

# 选择要查看的邮箱标签（例如 INBOX）
mail.select('inbox')

# 搜索所有未读邮件
status, messages = mail.search(None, 'UNSEEN')

# 获取邮件ID列表
email_ids = messages[0].split()

# 打印邮件数量
print(f"你有 {len(email_ids)} 封未读邮件。")

# 遍历所有未读邮件
for email_id in email_ids:
    # 获取邮件数据
    status, msg_data = mail.fetch(email_id, '(RFC822)')
    
    for response_part in msg_data:
        if isinstance(response_part, tuple):
            msg = email.message_from_bytes(response_part[1])
            
            # 获取邮件的主题
            subject, encoding = decode_header(msg['Subject'])[0]
            if isinstance(subject, bytes):
                subject = subject.decode(encoding if encoding else 'utf-8')
            
            # 获取发件人
            from_ = msg.get('From')
            
            # 打印邮件信息
            print(f"发件人: {from_}")
            print(f"主题: {subject}")
            
            # 如果邮件有多个部分（例如包含附件），则遍历所有部分
            if msg.is_multipart():
                for part in msg.walk():
                    content_type = part.get_content_type()
                    content_disposition = str(part.get('Content-Disposition'))
                    
                    # 如果邮件内容是文本
                    if 'attachment' not in content_disposition:
                        body = part.get_payload(decode=True).decode()
                        print(f"邮件内容: {body[:200]}")  # 只打印邮件内容的前 200 个字符
            else:
                body = msg.get_payload(decode=True).decode()
                print(f"邮件内容: {body[:200]}")  # 只打印邮件内容的前 200 个字符

# 关闭连接
mail.logout()
