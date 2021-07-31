# Wget
> wget 抓取整站

**可以将全站下载以本地的当前工作目录，生成可访问、完整的镜像。**

```shell
wget -r -p -np -k -E http://www.xxx.com 抓取整站
wget -l 1 -p -np -k http://www.xxx.com 抓取第一级
wget -m -e robots=off -k -E "http://www.abc.net/"
```

**解释：**
    
    -r 递归抓取
    -k 抓取之后修正链接，适合本地浏览
    -m //镜像，就是整站抓取
    -e robots=off //忽略robots协议，强制、流氓抓取
    -k //将绝对URL链接转换为本地相对URL
    -E //将所有text/html文档以.html扩展名保存