---
title: 请求不到github.com时修改dns可以解决
tags: 
  - dns
categories: 
  - linux
---
# 常用的可靠DNS服务器地址汇总

如果修改DNS服务器地址就可以访问Google等服务，你还等什么？使用免费DNS解析服务除了去掉了运营商的各种广告，还有个最大的好处就是不会重定向或者过滤用户所访问的地址，这样就防止了很多网站被电信、网通劫持，有利于提供访问一些国外网站的成功率，比如Google Code。网友应该养成不使用默认DNS的习惯，下面是一些常用可靠的DNS服务器地址。

1. **OpenerDNS**: 
   - 42.120.21.30
   - 该DNS是国内服务器，延迟在30ms左右，不掉包，速度非常好，该官方称使用该DNS能够无压力访问Facebook、Twitter、Google等服务，不过要使用https://加密链接。

2. **114DNS**:
   - 114.114.114.114
   - 114.114.115.115
   - 纯净、无劫持，无需再忍受被强扭去看广告或粗俗网站之痛苦。另有用于拦截钓鱼病毒木马网站、保护少年儿童免受网络色情内容毒害的地址。

3. **V2EX DNS**:
   - 199.91.73.222
   - 178.79.131.110
   - 延迟有点高，两个都在250ms以上，偶尔掉包，但这个用起来还是可以的。

4. **阿里DNS**:
   - 223.5.5.5
   - 223.6.6.6
   - 虽然阿里也没什么节操，但是延迟在25ms左右，基本不掉包，速度可以保证，如果不恶心阿的同学是可以考虑的，毕竟是国内的，访问速度可以保证的。

5. **Google DNS**:
   - 8.8.8.8
   - 8.8.4.4
   - 几乎从知道这个DNS之后就一直在用，延迟在50ms左右，速度还可以，就是由于众所周知的原因，掉包比较严重，但我是可以接受的。

6. **OpenDNS**:
   - 208.67.222.222
   - 208.67.220.220
   - 延迟有点高，在180ms左右，偶尔掉包，除了延迟大点，没有什么不能接受的。

7. **OneDNS**:
   - 112.124.47.27
   - 114.215.126.16
   - 官方称可以屏蔽广告，加速Google等网站的访问等，效果还不错，速度也不慢，是一个比较新的DNS。

8. **OpenNIC DNS**:
   - 官网：[OpenNic](https://www.opennic.org/)
   - OpenNic是一个社区化的非营利组织，主张DNS中立、免费服务、用户决议、自由开放、保护隐私、拒绝劫持。访问官网，会自动向你推荐最适合你所在区域的DNS。

9. **Norton（诺顿）DNS**:
   - 198.153.192.50
   - 198.153.194.50
   - 资深安全厂商，安全性和稳定性有保障。

10. **Comodo Secure DNS**:
    - 8.26.56.26
    - 8.20.247.20
    - 资深安全厂商，安全性和稳定性有保障。

最后，选择合适的DNS服务器取决于你所在地区和网络情况。国内用户可以考虑使用114DNS和阿里DNS，如果追求更高质量的DNS，可以选择V2EX DNS和OpenerDNS。对于国外用户，Google DNS是一个不错的选择，但由于掉包问题，也可以尝试其他选项。


（注意：以上DNS地址可能会发生变化，请根据实际情况进行选择。）
```

以上内容是将提供的DNS地址整理为Markdown格式的博客。请注意，DNS地址可能随时发生变化，所以建议在使用之前再次确认地址的准确性。