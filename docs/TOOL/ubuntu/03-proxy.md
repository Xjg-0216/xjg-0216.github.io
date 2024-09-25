


# 网络代理
!!! Question
    在Ubuntu系统中，设置网络代理后，通常只有那些配置为使用系统代理的应用程序会通过代理连接网络。对于一些应用程序，比如终端，不会自动遵循全局代理设置，因此，需要为这些应用程序手动配置代理

### 1. git

查看git所有配置
```bash
git config --list
```
添加网络代理

```bash
git config --global http.proxy http://your-proxy-address:port
git config --global https.proxy https://your-proxy-address:port
```

