# CNAS/MPS CTF 2017: Web Application Comprehensive Protection

**Points:** 300

> Mission Description

> [PutResourceHere](PutResourceHere)  

## Write-up

反而是一个相对容易的SQL注入，是开场做的第二题（我看反题目列表了还不行吗）。首先页面是空的，但是在源码中有个注释，提示`?id=1`，看了一下`?id=2-1`和`?id=1`返回结果一样，显然是注入漏洞。

懒得讲道理，上来就先用sqlmap跑了一波，随手一个tamper用了space2comment，检查出来有盲注漏洞，结果接下去sqlmap就无力了，于是手注。

稍微试了一下果然有waf，首先空格还是被过滤的，看起来space2comment用对了……没有过滤`/**/`，顺便测试了一下waf，过滤了一堆关键字……的小写，所以大小写混用就可以绕过waf了。

题目有一个显位，列数也是1，于是就直接老套路爆表名，爆字段名，爆值。反而比较简单。

## Other write-ups and resources

* <https://ctf-team.vulnhub.com/>
