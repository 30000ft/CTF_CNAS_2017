# CNAS/MPS CTF 2017: Identification And Authentication

**Points:** 100

> 网站中只有一个机器人的图片   

![Robot](score-robot.gif.png)

## Write-up

检查了/robots.txt，得到一个URL，是一串（大概是MD5，具体是什么已经忘记了）字符，跟踪进去是一个登录界面，试了一下SQL注入失败，结果弱密码admin/admin登录成功。

## Other write-ups and resources

* <https://ctf-team.vulnhub.com/>
