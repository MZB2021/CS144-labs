# Fetch a Web Page
利用 `telnet` 来获取一个网页, 具体操作如下:
```shell
# 向telnet传入域名和http协议参数
telnet cs144.keithw.org http
# 等待telnet和网站建立稳定的字节流连接, 即TCP
GET /hello HTTP/1.1
Host: cs144.keithw.org

# 注意上面的换行操作, 这个是告诉服务器http require结束
# 等待接受网站信息
```
