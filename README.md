# awk_urldecode
awk写的urldecode工具（转自http://www.cnblogs.com/xd502djj/archive/2013/03/08/2949632.html）

# 使用方法：
```
echo "%e8%bf%99%e6%98%af%e4%b8%80%e4%b8%aaurldecode%e7%bc%96%e7%a0%81" |awk_urldecode.sh
这是一个urldecode编码
```
# 闲来无事做了个多语言decode解码能力的对比
## C
```
#include <unistd.h>
#include <stdio.h>
#include <string.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>


void urldecode2(char *dst, const char *src)
{
    char a, b;
    while (*src)
    {
        if ((*src == '%') &&
                ((a = src[1]) && (b = src[2])) &&
                (isxdigit(a) && isxdigit(b)))
        {
            if (a >= 'a')
                a -= 'a'-'A';
            if (a >= 'A')
                a -= ('A' - 10);
            else
                a -= '0';
            if (b >= 'a')
                b -= 'a'-'A';
            if (b >= 'A')
                b -= ('A' - 10);
            else
                b -= '0';
            *dst++ = 16*a+b;
            src+=3;
        }
        else if (*src == '+')
        {
            *dst++ = ' ';
            src++;
        }
        else
        {
            *dst++ = *src++;
        }
    }
    *dst++ = '\0';
}

int main()
{
    char *input = "%e8%bf%99%e6%98%af%e4%b8%80%e4%b8%aaurldecode%e7%bc%96%e7%a0%81";
    char *output = malloc(strlen(input)+1);
    urldecode2(output, input);
    printf("%s\n", output);
}
```
## golang
```
package main

import (
	"net/url"
)

func main() {
	str := "%e8%bf%99%e6%98%af%e4%b8%80%e4%b8%aaurldecode%e7%bc%96%e7%a0%81"
	result, _ := url.QueryUnescape(str)
	println(result)
}
```
# shell
```
#/bin/sh
echo "%e8%bf%99%e6%98%af%e4%b8%80%e4%b8%aaurldecode%e7%bc%96%e7%a0%81" |awk 'BEGIN {
for(i=0;i<10;i++)
hex[i]=i;
hex["A"]=hex["a"]=10;
hex["B"]=hex["b"]=11;
hex["C"]=hex["c"]=12;
hex["D"]=hex["d"]=13;
hex["E"]=hex["e"]=14;
hex["F"]=hex["f"]=15;
}
{
gsub(/\+/," ");
i=$0;
while(match(i,/%../)){
if(RSTART>1);
printf"%s",substr(i,1,RSTART-1);
printf"%c",hex[substr(i,RSTART+1,1)]*16+hex[substr(i,RSTART+2,1)];
i=substr(i,RSTART+RLENGTH);
}
print i;
}'
```
## python
```
#!/usr/bin/env python
from urllib import unquote
str="%e8%bf%99%e6%98%af%e4%b8%80%e4%b8%aaurldecode%e7%bc%96%e7%a0%81"
print(unquote(str))
```
## php
```
#!/usr/bin/php
<?php
echo urldecode("%e8%bf%99%e6%98%af%e4%b8%80%e4%b8%aaurldecode%e7%bc%96%e7%a0%81")."\n";
?>
```
# 单次测试结果
```
[root@server test]# time ./c_decode
这是一个urldecode编码

real    0m0.001s
user    0m0.001s
sys     0m0.000s
[root@server test]# time ./go_decode
这是一个urldecode编码

real    0m0.002s
user    0m0.001s
sys     0m0.000s
[root@server test]# time ./awk_decode
这是一个urldecode编码

real    0m0.004s
user    0m0.002s
sys     0m0.002s
[root@server test]# time ./python_decode
这是一个urldecode编码

real    0m0.034s
user    0m0.028s
sys     0m0.005s
[root@server test]# time ./php_decode
这是一个urldecode编码

real    0m0.047s
user    0m0.040s
sys     0m0.006s
```
# 使用shell for循环执行1000次结果
```
[root@server test]# time for i in {1..1000};do ./c_decode &> /dev/null;done 

real    0m1.196s
user    0m0.137s
sys     0m0.238s
[root@server test]# time for i in {1..1000};do ./go_decode &> /dev/null;done 

real    0m1.604s
user    0m0.480s
sys     0m0.894s
[root@server test]# time for i in {1..1000};do ./awk_decode &> /dev/null;done  

real    0m3.918s
user    0m1.731s
sys     0m1.558s
[root@server test]# time for i in {1..1000};do ./python_decode &> /dev/null;done   

real    0m34.001s
user    0m26.582s
sys     0m7.019s
[root@server test]# time for i in {1..1000};do ./php_decode &> /dev/null;done     

real    0m45.606s
user    0m39.262s
sys     0m6.186s
```
# 使用各语言for循环执行1000次结果
```
[root@server test]# time ./c_decode_1000 &> /dev/null

real    0m0.002s
user    0m0.001s
sys     0m0.001s
[root@server test]# time ./go_decode_1000 &> /dev/null 

real    0m0.003s
user    0m0.002s
sys     0m0.002s
[root@server test]# time ./awk_decode_1000 &> /dev/null    

real    0m1.619s
user    0m0.512s
sys     0m0.871s
[root@server test]# time ./python_decode_1000 &> /dev/null   

real    0m0.046s
user    0m0.039s
sys     0m0.007s
[root@server test]# time ./php_decode_1000 &> /dev/null     

real    0m0.047s
user    0m0.042s
sys     0m0.005s
```
