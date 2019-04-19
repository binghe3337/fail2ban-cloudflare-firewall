# fail2ban-cloudflare-firewall
Fail2ban+CloudFlare实现入门级CC防御

Fail2ban的工作原理很简单：读取日志，使用正则表达式匹配IP地址，只要在规定时间内达到预先设置的访问次数，就会执行ban action。也可以设置在封禁一段时间之后，自动解除封禁。总之这是一款比较灵活的软件。

当我们没有使用CloudFlare的时候，fail2ban通常会把匹配到的ip地址提交给本机的iptables，而使用了CloudFlare之后，我们就可以直接用api把IP地址提交给CloudFlare的防火墙。

下面简单说一下方法：

第一步，配置Nginx。Nginx在编译的时候一定要添加“--with-http_realip_module”模块，并且在http块或server块中引入“set_real_ip_from、real_ip_header”，这样才能把真实的访客IP传递进来。否则，日志里记录到的都是CloudFlare自己的IP。具体方法请看这篇文章：https://support.cloudflare.com/hc/zh-cn/articles/200170706-%E4%BD%BF%E7%94%A8-Nginx-%E6%97%B6%E5%A6%82%E4%BD%95%E6%81%A2%E5%A4%8D%E5%8E%9F%E5%A7%8B%E8%AE%BF%E9%97%AE%E8%80%85%E7%9A%84-IP-

第二步，安装fail2ban。方法看这里：https://github.com/fail2ban/fail2ban 。不要用apt/yum这些包管理器安装，包管理器中的版本太旧了，不支持ipv6。

第三步，配置fail2ban。

修改配置文件，然后覆盖至/etc/fail2ban目录

其中，filter.d目录中的http-get-dos.conf文件匹配所有访问，而http-get-dos-no-static.conf排出了指定后缀的静态文件。因为CloudFlare会缓存静态文件，所以建议使用http-get-dos-no-static.conf这个配置。


全部设置完成后，运行“fail2ban-client reload”重新加载配置文件。

说说这套系统的几个缺点：
第一就是CloudFlare的IP防火墙有延迟，新IP添加进去，大概半分钟左右才会生效。因此我建议在Nginx中设置limit_req_zone，防火墙生效之前能起到临时的防护。


第二，单位时间内的访问量一定要设置合理，网站第一次打开通常会加载大量静态文件，如果设置不合理，很可能会屏蔽正常访客。这里还有一个方法，就是在“http-get-dos.conf”文件中修改正则表达式，只匹配引起高负载的动态文件，这就需要各位自己发挥了。


