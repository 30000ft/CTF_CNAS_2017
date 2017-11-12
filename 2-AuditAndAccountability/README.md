# CNAS/MPS CTF 2017: Audit And Accountability

**Points:** 100

> 网站日志里面放了一个移动互联网的东西

> [64a62e4db6a30fd53b86673f9be56095.apk](64a62e4db6a30fd53b86673f9be56095.apk)  

## Write-up

题目提示了移动设备下载的东西，不过尝试了一下base.apk提示404。

爆破目录发现了/log/，里面有access.log，error.log和other\_vhosts\_access.log三件套，查看了access.log发现有个（又是一堆hash）字符串.apk，下载下来。

直接`unzip`解压apk，拿到classes.dex，`d2j-dex2jar`转jar文件，继续`unzip`解压，发现MainActivity文件，分析smali（我承认工具准备不充分……）得到逻辑。

还好逻辑非常简单，就是一个byte Array全部异或了0xcc然后toString再base64 decode一下。于是扣出这段byte array异或一下base64 decode一下即可。

## Other write-ups and resources

* <https://ctf-team.vulnhub.com/>
