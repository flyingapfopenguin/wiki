---
title: "Python"
date: 2023-04-06T22:54:01+02:00
---

## Performance analysis

Source: https://stackoverflow.com/questions/8624779/my-python-program-is-very-slow-how-can-i-speed-it-up-am-i-doing-something-wron

To run the python profiler and store the profiling information in prof.dat
```bash
python -m cProfile -o prof.dat <prog> <args>
```

To start the interactive "profile statistics tool" run
```bash
python -m pstats prof.dat
```

You probably first want to sort the results
```
sort time # sort by tottime, the total time spent in the function alone
sort cumtime # sort by cumtime, the total time spent in the function plus all functions that this function called
```

and afterwards print those
```
stats # print the whole statistic
stats <N> # print the first N entries
```

## Mail via Python

```python
# import necessary packages
from email.mime.multipart import MIMEMultipart
from email.mime.text import MIMEText
from email.mime.application import MIMEApplication
import email.utils
import smtplib
import os.path

def sendMail(server, port, user, password, fromMailadress, fromName, toMailadress, toName, subject, message, attachments=[], replyToMailadress=""):
    # create message object instance
    msg = MIMEMultipart()
    # setup the parameters of the message
    msg['To'] = email.utils.formataddr((toName, toMailadress))
    msg['From'] = email.utils.formataddr((fromName, fromMailadress))
    msg['Date'] = email.utils.formatdate(localtime=True)
    msg['Subject'] = subject
    msg.add_header('reply-to', replyToMailadress)
    # add in the message body
    msg.attach(MIMEText(message, 'plain'))
    # attach files
    for filename in attachments:
        if not os.path.isfile(filename):
            continue
        with open(filename, "rb") as f:
            part = MIMEApplication(f.read(),Name=os.path.basename(filename))
            # After the file is closed
            part['Content-Disposition'] = 'attachment; filename="%s"' % os.path.basename(filename)
            msg.attach(part)
    # create server connection
    server = smtplib.SMTP(server + ": " + port)
    server.starttls()
    server.ehlo()
    # Login Credentials for sending the mail
    server.login(user, password)
    # send the message via the server.
    try:
        server.sendmail(msg['From'], msg['To'], msg.as_string())
    finally:
        server.quit()
```

## Python-venv

Installation:
```bash
apt install python3-venv
```

Setup:
```bash
python3 -m venv myvenv
cd myvenv
source bin/activate
pip install -U pip
```

Install dependency (e.g. django)
```bash
pip install django
```

Install dependency with specific version
```bash
pip install django==1.10
```

Store dependencies
```bash
pip freeze > requirements.txt
```

Retrieve dependencies
```bash
pip install -r requirements.txt
```
