---
layout: post
title: How to send email using python?
# author: "Himanshu Aggarwal"
# description: "Sending Mail Utility in Python(via SMTP)"
date: 2022-03-01
categories: python tutorials
---

This code will let you fire bunch of emails along with attachments using python via smtp server. <br>
Although in production you may have preset and straightforward ways to do the same. <br>
But, this can be a handy code for sending mail via python <br>
<br>
Before, Sending mails using this script. <br>
Please ensure that you have enabled the access to less secure apps (Help Link : [Google Enable Less Secure Apps](https://myaccount.google.com/lesssecureapps)) <br>
Follow this [Google Help Link](https://support.google.com/mail/answer/7126229?visit_id=1538904444444-1419963073072761751&rd=2#cantsignin), if faces any issue.

```python
import smtplib                                                   #Simple mail Transfer Protocol
from email.mime.application import MIMEApplication               #MIME : Multipurpose Internet Mail Extensions
from email.mime.multipart import MIMEMultipart
from email.mime.text import MIMEText
from os.path import basename
from email import encoders
import logging

logging.basicConfig(level = logging.INFO,format = '%(asctime)s - %(levelname)s: %(message)s',datefmt = '%m/%d/%Y %I:%M:%S %p' ) #Setting up logging


def send_mail(email_from, email_to, password, message, subject, files=[]):
    """
    This function sends the mail via smtp.
    :param: email_from    From Email Address
    :param: email_to      To Email Address
    :param: message       Message bosy of the mail
    :param: files         comma separated list of local file paths

    :return: result, error_message (result is True for success)
    """
    msg = MIMEMultipart()
    msg['From'] = email_from
    msg['To'] = ",".join(email_to)
    msg['Subject'] = subject
    msg.attach(MIMEText(message, 'plain'))
    for file in files:
        with open(file, 'rb') as f:
            part = MIMEApplication(f.read(), Name=basename(file))
            part['Content-Disposition'] =  'attachment; filename='+str(basename(file))
            msg.attach(part)
    try:
        mail_server = smtplib.SMTP('smtp.gmail.com', 587)
    except smtplib.socket.gaierror as e:
        return False, e
    mail_server.ehlo()                              #extended smtp to identify
    mail_server.starttls()                          #tansport layer security
    try:
        mail_server.login(email_from, password)
    except smtplib.SMTPAuthenticationError as e:    #login error
        return False, e
    try:
        mail_server.sendmail(email_from, email_to, msg.as_string())
        mail_server.close()
        return True, None
    except Exception as e:
        return False, e


if __name__ == '__main__': 
    #Sample Usage Of the Function
    #These are only dummy values (substitue with the actual ones)
    email_from = "xyz@gmail.com"
    email_to = "abc@gmail.com"
    password = "password"
    message="Dear All,\nPlease find the attached filess\n\nRegards,\nHimanshu Aggarwal"
    subject = "TEST"
    result, error_message = send_mail(email_from, email_to, password, message, subject, )
    if result == True:
        logging.info('Email Sent Successfully')
    else:
        logging.error('Sending Email Failed Error Message {}'.format(error_message))
```


Some Recommendations : Pick the password from a credentials manager (in production) and use template for message email body
