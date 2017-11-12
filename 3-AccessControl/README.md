# CNAS/MPS CTF 2017: Access Control 

**Points:** 150

> 这个网站只有管理员才能访问

> [PutResourceHere](PutResourceHere)  

## Write-up

检查header，Cookie里有一串明显的base64字符串，解码之后发现是php的serialize后的，`s:5:"guest"`，是个字符串，于是我们改成`base64_encode(serialize("admin"))`提交回去，页面改变了，给了一段源码。

检查了一下源码，大概是文件上传，源码有点诡异，这里大概回忆一下过程。首先`preg_match("[<>?]", $_POST["data"])`需要返回0，然后`preg_match("[<>?]", implode($_POST["data"]))`需要返回非0（只有1了），所以只需要POST一个`filename=whatever&data[]=<`即可。

## Other write-ups and resources

* <https://ctf-team.vulnhub.com/>
